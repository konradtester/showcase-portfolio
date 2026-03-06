# EduStream AI | Intelligent Lecture Simplifier

## 🚀 Business Overview

EduStream AI is an educational productivity tool that transforms complex academic lectures into structured, easy-to-digest summaries. By combining Speech-to-Text (STT) technology with Large Language Models (LLMs), it allows students and professionals to focus on understanding rather than note-taking.

### Key Value Propositions:

- **Instant Summarization**: Converts 1-hour lectures into 5-minute executive summaries.
- **Key Concept Extraction**: Automatically identifies and defines academic terms.
- **Multi-Format Export**: Seamlessly export notes to PDF or Markdown.

---

## 🏗️ Engineering Challenges & Logical Solutions

### 1. Handling Long-Context Audio Processing

**Challenge**: Processing large audio files (2h+) without timing out the API.
**Solution**: Implemented an asynchronous processing pipeline using **FastAPI Background Tasks**. Users receive an instant response that the file is processing, and the system notifies them via the mobile app once the STT and LLM summarization are complete.

### 2. Prompt Engineering for Academic Accuracy

**Challenge**: LLMs often miss technical nuances in specialized lectures (e.g., Medical or Engineering).
**Solution**: Developed a **Dynamic Prompting Strategy**. The system first classifies the lecture's domain and then applies a specialized meta-prompt optimized for that specific scientific field to ensure terminology remains accurate.

---

## 🛠️ Technology Stack

- **Backend**: FastAPI (Python).
- **AI/ML**: OpenAI GPT-4 / Gemini, Whisper (STT).
- **Frontend**: Flutter.
