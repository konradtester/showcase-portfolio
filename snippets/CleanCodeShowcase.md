# Code Snippets | Production-Grade Patterns

Real code from production systems. Sensitive values (hostnames, paths, credentials) anonymized.

---

## 1. LangGraph Agent — Safe / Sensitive Tool Split with HITL

The core of the DevOps agent. Tools are classified at compile time into two groups: **safe tools** run autonomously, **sensitive tools** require explicit human approval. The graph compiles with `interrupt_before=["sensitive_tools"]` — a hard architectural constraint, not a runtime check that can be bypassed by a prompt.

```python
from langchain_core.messages import BaseMessage, SystemMessage
from langgraph.graph import END, StateGraph
from langgraph.prebuilt import ToolNode
from langgraph.checkpoint.memory import MemorySaver

from agent.state import AgentState
from agent.tools import (
    read_docker_logs, analyze_nginx_logs,
    check_server_vitals, run_api_tests,     # safe: read-only
    git_pull_updates, cleanup_server_storage # sensitive: destructive
)

# Safe tools: zero side effects — agent runs these autonomously
safe_tools = [read_docker_logs, analyze_nginx_logs, check_server_vitals, run_api_tests]

# Sensitive tools: destructive operations — require human approval before execution
sensitive_tools = [git_pull_updates, cleanup_server_storage]

safe_tool_node = ToolNode(safe_tools)
sensitive_tool_node = ToolNode(sensitive_tools)
model_with_tools = model.bind_tools(safe_tools + sensitive_tools)


def call_model(state: AgentState):
    """Main agent node. Trims message history to control token usage."""
    messages = state["messages"]
    lang = state.get("language", "pl")

    system_prompt = (
        "You are a DevOps AI assistant.\n"
        "Safe tools run automatically. Sensitive tools require user confirmation.\n"
        f"Respond in {'English' if lang == 'en' else 'Polish'}.\n"
        "Default environment: local. Detect 'prod'/'production' keywords to switch."
    )

    # Keep last 4 messages: user prompt → tool call → tool result → AI summary
    trimmed = messages[-4:] if len(messages) > 4 else messages
    if not trimmed or not isinstance(trimmed[0], SystemMessage):
        trimmed = [SystemMessage(content=system_prompt)] + trimmed

    return {"messages": [model_with_tools.invoke(trimmed)]}


def should_continue(state: AgentState) -> str:
    """Routing logic: safe tools → auto-execute, sensitive tools → interrupt."""
    last = state["messages"][-1]
    if not getattr(last, "tool_calls", None):
        return END

    sensitive_names = {t.name for t in sensitive_tools}
    for tc in last.tool_calls:
        if tc["name"] in sensitive_names:
            return "sensitive_tools"
    return "safe_tools"


# Graph construction
workflow = StateGraph(AgentState)
workflow.add_node("agent", call_model)
workflow.add_node("safe_tools", safe_tool_node)
workflow.add_node("sensitive_tools", sensitive_tool_node)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue)
workflow.add_edge("safe_tools", "agent")
workflow.add_edge("sensitive_tools", "agent")

app = workflow.compile(
    checkpointer=MemorySaver(),
    interrupt_before=["sensitive_tools"],  # HITL: hard stop before any destructive op
)
```

---

## 2. Tool Design — Environment-Aware with `_resolve_host`

All tools share a single helper that translates environment keywords (`"local"`, `"prod"`) into concrete targets. This keeps tool signatures clean and makes environment switching transparent to the agent — it passes a keyword, not a hostname.

```python
import docker
import subprocess
import os
from langchain_core.tools import tool


def _resolve_host(env: str = None) -> str:
    """Resolves environment keyword to a concrete host.

    'local' / 'dev'        → local Docker socket
    'prod' / 'production'  → VPS_HOST from environment
    anything else          → treated as a direct hostname or IP
    """
    if not env:
        return os.getenv("VPS_HOST")
    env_str = str(env).lower().strip()
    if env_str in ("dev", "local"):
        return "local"
    if env_str in ("prod", "production"):
        return os.getenv("VPS_HOST")
    return env


@tool
def read_docker_logs(container_name: str, tail: int = 50, env: str = "local") -> str:
    """Reads the last N lines of logs from a Docker container.
    Works across environments: local Docker socket or remote via SSH.
    """
    try:
        host = _resolve_host(env)
        if host == "local":
            client = docker.from_env()
            container = client.containers.get(container_name)
            logs = container.logs(tail=tail).decode("utf-8", errors="ignore")
        else:
            result = subprocess.run(
                ["ssh", f"root@{host}", f"docker logs --tail {tail} {container_name}"],
                capture_output=True, text=True
            )
            if result.returncode != 0:
                return f"SSH error from {host}: {result.stderr}"
            logs = result.stdout

        return logs.strip() or f"No logs found for '{container_name}' on {host}."
    except Exception as e:
        return f"Error reading logs for '{container_name}' on {env}: {e}"


@tool
def analyze_nginx_logs(
    env: str = "prod",
    log_path: str = "/var/log/nginx/access.log",
    lines: int = 1000
) -> str:
    """Analyzes NGINX access logs for anomalies: 404 spikes, DDoS patterns, top IPs."""
    try:
        host = _resolve_host(env)
        if host == "local":
            result = subprocess.run(["tail", "-n", str(lines), log_path],
                                    capture_output=True, text=True)
            log_data = result.stdout
        else:
            result = subprocess.run(
                ["ssh", f"root@{host}", f"tail -n {lines} {log_path}"],
                capture_output=True, text=True
            )
            if result.returncode != 0:
                return f"SSH error: {result.stderr}"
            log_data = result.stdout

        if not log_data.strip():
            return f"No log data on {env}."

        lines_list = log_data.strip().split("\n")
        stats = {"total": len(lines_list), "404s": 0, "5xxs": 0, "ips": {}}

        for line in lines_list:
            parts = line.split()
            if len(parts) < 9:
                continue
            ip, status = parts[0], parts[8]
            stats["ips"][ip] = stats["ips"].get(ip, 0) + 1
            if status == "404":
                stats["404s"] += 1
            elif status.startswith("5"):
                stats["5xxs"] += 1

        top_ip = max(stats["ips"], key=stats["ips"].get) if stats["ips"] else "N/A"
        top_count = stats["ips"].get(top_ip, 0)

        summary = (
            f"NGINX Analysis [{env}] — {stats['total']} requests\n"
            f"  404s: {stats['404s']}  |  5xxs: {stats['5xxs']}"
            f"  |  Top IP: {top_ip} ({top_count} reqs)\n"
        )
        if stats["404s"] > stats["total"] * 0.3:
            summary += "  ⚠️  High 404 volume — possible directory scanning.\n"
        if top_count > stats["total"] * 0.5:
            summary += f"  ⚠️  {top_ip} accounts for >50% of traffic — possible DDoS.\n"

        return summary
    except Exception as e:
        return f"Error during log analysis: {e}"
```

---

## 3. Discord Bot — HITL Approval Flow

The bot exposes the agent via Discord. When a sensitive tool is requested, the graph pauses and asks for explicit confirmation. Approval state persists via `MemorySaver` — the channel ID acts as the checkpoint key, so each channel has isolated conversation state.

```python
import discord
from langchain_core.messages import HumanMessage, ToolMessage
from agent.graph import app


class DevOpsAgentBot(discord.Client):
    async def on_message(self, message):
        if message.author.id == self.user.id:
            return
        if not (self.user.mentioned_in(message) or
                message.content.lower().startswith("agent")):
            return

        async with message.channel.typing():
            config = {"configurable": {"thread_id": str(message.channel.id)}}
            user_content = self._clean_input(message.content)

            try:
                state = app.get_state(config)

                # HITL: graph is paused waiting for approval
                if state.next and "sensitive_tools" in state.next:
                    approval = user_content.lower().strip()

                    if approval in ("tak", "yes", "y", "t", "potwierdzam"):
                        await message.reply("✅ Approved. Executing...")
                        result = app.invoke(None, config)

                    elif approval in ("nie", "no", "n", "anuluj", "odrzucam"):
                        await message.reply("❌ Operation cancelled.")
                        app.update_state(
                            config,
                            {"messages": [HumanMessage(content="Action cancelled.")]},
                            as_node="agent"
                        )
                        return

                    else:
                        await message.reply(
                            "⚠️ **Confirmation required.**\n"
                            "Reply `tak` to approve or `nie` to cancel."
                        )
                        return

                else:
                    result = app.invoke(
                        {"messages": [HumanMessage(content=user_content)]},
                        config
                    )

                # Check if graph paused after invocation
                state = app.get_state(config)
                if state.next and "sensitive_tools" in state.next:
                    last = state.values["messages"][-1]
                    tools_requested = [tc["name"] for tc in last.tool_calls]
                    await message.reply(
                        f"⚠️ **Confirmation required**\n"
                        f"Agent wants to run: `{', '.join(tools_requested)}`\n\n"
                        f"Reply `tak` to approve or `nie` to cancel."
                    )
                    return

                # Extract and send final response
                messages = state.values.get("messages", [])
                response = "Agent completed the task without generating a text response."
                for msg in reversed(messages):
                    if (hasattr(msg, "content") and msg.content.strip()
                            and not isinstance(msg, ToolMessage)):
                        response = msg.content
                        break

                for i in range(0, len(response), 2000):
                    await message.reply(response[i:i + 2000])

            except Exception as e:
                await message.reply(f"❌ Agent error: {e}")

    def _clean_input(self, content: str) -> str:
        content = content.strip()
        if content.lower().startswith("agent"):
            content = content[5:].lstrip(",").strip()
        return content
```
