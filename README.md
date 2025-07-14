# 🧠 Local AI Code Agent — Full-Day Workshop

## 🗕️ Purpose

This full-day hands-on workshop introduces students to building and deploying a **local AI coding assistant**. The session covers the **architecture, tools, and prompt framing** of a personal AI agent capable of understanding and improving software projects — entirely **offline**, with **local models**.

It’s designed to give participants a practical understanding of:

- The local AI agent workflow
- Open-source LLMs (e.g., Mistral)
- Prompt engineering with project-specific context
- Offline-first coding support using lightweight DB
- 🕘️ **(Optional)**: Post-workshop pathways for API comparisons or advanced features

---

## 🕘️ Morning — Foundations & Framing

### 1. 🧡 Introduction to AI Agents & Mistral

- What are AI agents?
- The Mistral ecosystem and open models
- Use cases: developer productivity, internal code assistance
- Overview of `llama.cpp` and quantized model formats (GGUF)

### 2. 🔍 Choosing the Right LLM for the Job

- Evaluating models for **local execution**
- RAM, CPU/GPU requirements, quantization (Q4_K, Q6_K, Q8_0…)
- ✅ Focused on **offline local models**
- 🕘️ **(Optional Mention)**: API-based augmentation (Claude, GPT-4) as an advanced topic

### 3. 🎯 Prompt Engineering Basics

- How prompt framing improves model usefulness
- Instruction-tuned models vs. base models
- Techniques: **zero-shot**, **few-shot**, **role conditioning**

### 4. ⚖️ Ethical Framing for Local AI

- Why local-first matters (privacy, IP protection)
- Limits of LLM reasoning (hallucination awareness)
- ✅ Logging interactions in SQLite for transparency
- 🕘️ **(Mention)**: Corporate code safety considerations

---

## 🔧 Afternoon — Build Your Local Agent

### 👨‍💻 Hands-On Objective

Build a **local AI code assistant** that:

- ✅ Summarizes project structure
- ✅ Provides **project-aware answers** through prompt framing
- ✅ Tracks session history in a local database
- ✅ Works **entirely offline** with `llama.cpp` and Mistral
- 🕘️ **Optional extension topics** discussed, but **not part of base build**

---

## ✨ Key Features You’ll Build

- 🧱 **Local LLM** (Mistral-7B GGUF) via `llama.cpp`
- 🔍 **Context-aware project parsing**
- 📁 **Per-project prompt memory** (SQLite)
- 🧠 **Code-aware answering** via structured prompts
- 🛠️ **Modular terminal agent**
- 🕘️ **Optional API comparisons (presented, not built)**

---

## 🏗️ Architecture Summary

```text
Student Codebase
       │
       ▼
[ Local Agent ]
(llama.cpp + prompt loader + SQLite memory)
       │
       ├─→ Parses project structure
       ├─→ Summarizes context with LLM
       ├─→ Provides coding suggestions offline
       └─→ Stores prompt-response in DB
```

---

## 🛠️ Tools Used

| Purpose             | Tool                                 | Description                         |
| ------------------- | ------------------------------------ | ----------------------------------- |
| Local inference     | `llama.cpp`, Mistral GGUF            | Local LLM execution engine          |
| Project parsing     | Python `os.walk`, structured prompts | Summarize file structure            |
| Session memory      | SQLite                               | Local DB storage of interactions    |
| (Optional examples) | GPT-4, Claude (presentation only)    | Shown in slides, not built in agent |

---

## 📦 Portability

- Works on **macOS, Windows, Linux**
- Requires **Python 3.10+**
- No Docker or web server required for core functionality

---

## ✅ Workshop Outcomes

Students will:

- ✅ Run a **local Mistral-based code assistant** offline
- ✅ Understand prompt framing using project structure
- ✅ Track sessions via **SQLite memory**
- ✅ Identify use cases for offline LLM coding support
- 🕘️ **(Optional learning)**: API comparisons for future exploration

---

## 💡 Why Local?

> Because private code should stay private — and **offline agents** let you benefit from AI without leaking your IP, exposing user data, or relying on external APIs.

