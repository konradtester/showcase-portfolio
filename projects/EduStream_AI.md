# EduStream AI | Intelligent Lecture Simplifier

## 🚀 Business Overview

EduStream AI is an educational productivity tool that transforms complex academic lectures into structured, easy-to-digest summaries. By combining Speech-to-Text (STT) technology with Large Language Models, it allows students and professionals to focus on understanding rather than note-taking.

**Key Value Propositions:**
- **Instant Summarization:** Converts 1-hour lectures into 5-minute executive summaries.
- **Key Concept Extraction:** Automatically identifies and defines academic terms.
- **Hallucination Prevention:** Pydantic schema enforcement and a custom audit system ensure deterministic, verifiable output.
- **Multi-Format Export:** Seamlessly export notes to PDF or Markdown.

---

## 🏗️ Engineering Challenges & Solutions

### 1. Handling Long-Context Audio Processing

**Challenge:** Processing large audio files (2h+) without timing out the API or losing context across chunk boundaries.

**Solution:** Asynchronous processing pipeline using FastAPI Background Tasks. Users receive an instant acknowledgement, and the system notifies them via the mobile app once STT and LLM summarization are complete. Audio is chunked with overlap to prevent context loss at boundaries.

---

### 2. Prompt Engineering for Academic Accuracy

**Challenge:** LLMs often miss technical nuances in specialised lectures (Medical, Engineering, Law).

**Solution:** Dynamic Prompting Strategy — the system first classifies the lecture domain, then applies a specialised meta-prompt optimised for that scientific field. This keeps terminology accurate without requiring domain-specific fine-tuning.

---

### 3. Deterministic Output from Non-Deterministic LLMs

**Challenge:** LLM outputs are non-deterministic by nature — summaries can vary in structure, miss required fields, or hallucinate content.

**Solution:** Strict JSON schema enforcement via Pydantic. Every LLM response is validated against a defined schema before being returned to the user. A custom audit system detects hallucinations and broken transcripts in long-form audio — flagging low-confidence outputs rather than silently passing them through.

---

## 🛠️ Technology Stack

| Layer | Technologies |
|---|---|
| Backend | FastAPI · Python |
| AI / ML | Google Gemini · OpenAI GPT-4o · Whisper (STT) · Pydantic (schema validation) |
| Mobile | Flutter · Dart |

---

## 📊 Status

Personal project. Core summarisation pipeline complete with hallucination detection. Multi-format export implemented.
