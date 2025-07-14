# 🧠 AI Coding Agent Setup (Part 2)

This document continues the setup for your **local AI agent** based on Mistral and `llama.cpp`, preparing it to serve as a **code-aware development assistant**.

---

## ✅ Prerequisites Recap

Make sure you've completed:

- ✅ Installed `llama.cpp` and compiled the CLI binary
- ✅ Downloaded a suitable Mistral model (e.g. `Q4_K` or `Q8_K`)
- ✅ Successfully tested `local_agent.py` for prompt-based conversation

If not, refer back to the platform-specific setup files (macOS, Windows, Linux).

---

## 🌎 Define Your Developer Workspace

To help the AI agent work on your code:

1. Create a base folder where you store your code projects, e.g.:

   ```bash
   mkdir -p ~/dev
   ```

2. Move your sample projects into that folder:

   ```bash
   mv ~/Downloads/my-python-project ~/dev/
   ```

3. Adjust the Python script to define this base path:

   ```python
   BASE_DEV_PATH = os.path.expanduser("~/dev")
   ```

   > ⚠️ Students must **edit this path** to fit their system.

✅ **Checkpoint**: Run your Python script up to this point and verify that it correctly detects your projects.

---

## 🧹 Project Selection Logic

Add the following to your `local_agent.py` to let users pick which project to load:

```python
# === Select a Project ===
def select_project(base_path):
    if not os.path.exists(base_path):
        print(f"❌ Base dev folder not found: {base_path}")
        sys.exit(1)

    subfolders = [f for f in os.listdir(base_path) if os.path.isdir(os.path.join(base_path, f))]
    if not subfolders:
        print(f"❌ No projects found in: {base_path}")
        sys.exit(1)

    print("\n📂 Available Projects:")
    for i, folder in enumerate(subfolders, start=1):
        print(f"{i}. {folder}")

    while True:
        choice = input("\n🔢 Select a project number: ").strip()
        if choice.isdigit() and 1 <= int(choice) <= len(subfolders):
            selected = subfolders[int(choice) - 1]
            full_path = os.path.join(base_path, selected)
            print(f"✅ Selected project: {selected} ({full_path})\n")
            return full_path
        else:
            print("❌ Invalid selection. Try again.")
```

✅ **Checkpoint**: Run your script and verify that the selection prompt displays your projects correctly.

---

## 🧰 Add SQLite Memory for Persistence

### 🗃️ 1. Initialize the SQLite DB

Create a script at `scripts/init_db.py`:

```python
import sqlite3

conn = sqlite3.connect("memory/agent_memory.db")
c = conn.cursor()

# Create tables
c.execute("""
CREATE TABLE IF NOT EXISTS projects (
    id INTEGER PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    path TEXT NOT NULL,
    summary TEXT
);
""")

c.execute("""
CREATE TABLE IF NOT EXISTS prompts (
    id INTEGER PRIMARY KEY,
    project_id INTEGER,
    prompt TEXT,
    response TEXT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY(project_id) REFERENCES projects(id)
);
""")

conn.commit()
conn.close()
print("✅ SQLite DB initialized.")
```

✅ **Checkpoint**: Run `python3 scripts/init_db.py` and check that `agent_memory.db` is created.

---

### 📂 2. Add DB Connection in `local_agent.py`

```python
import sqlite3

def get_db():
    return sqlite3.connect("memory/agent_memory.db")

def get_or_create_project(name, path):
    conn = get_db()
    c = conn.cursor()
    c.execute("SELECT id FROM projects WHERE name = ?", (name,))
    row = c.fetchone()
    if row:
        conn.close()
        return row[0]
    c.execute("INSERT INTO projects (name, path) VALUES (?, ?) ", (name, path))
    conn.commit()
    project_id = c.lastrowid
    conn.close()
    return project_id
```

✅ **Checkpoint**: Test the database connection by printing the `project_id` after selection.

---

## 📝 Implement the Prompt Builder Function

Add this helper function:

```python
def build_prompt(project_name, recent_history, user_input):
    system_message = (
        f"You are a helpful coding assistant working on the project '{project_name}'. "
        "Answer concisely, focus on technical accuracy, and avoid hallucinations.\n\n"
    )

    history_messages = ""
    for prompt, response in recent_history:
        history_messages += f"Q: {prompt}\nA: {response}\n"

    current_prompt = f"Q: {user_input}\nA:"

    return system_message + history_messages + current_prompt
```

✅ **Checkpoint**: Students can print the generated prompt and validate it before passing to the model.

---

### 🔄 Example: Storing Data

```python
# Save a prompt/response pair
conn = get_db()
c = conn.cursor()
c.execute("INSERT INTO prompts (project_id, prompt, response) VALUES (?, ?, ?)", (PROJECT_ID, prompt, ai_response))
conn.commit()
conn.close()
```

---

## 🚀 You’re Set to Continue

Now your agent supports:

- Persistent memory of project-level prompts
- SQLite-based tracking of interaction history
- Solid base for future tuning or evaluations

From the root of `llama.cpp`, run:

```bash
python3 scripts/local_agent.py
```

## 🧩 Adding Automatic Project Summary to Your AI Agent

To make your AI agent more aware of the project it works on, you can **automatically summarize the project folder structure**. This summary provides context to the AI before any interaction.

---

### ✅ Step 1: Generate a Project Summary

Add this function to your `local_agent.py`:

```python
import os

def summarize_project_structure(base_path, max_depth=3):
    summary = []

    def walk(dir_path, depth):
        if depth > max_depth:
            return
        try:
            entries = os.listdir(dir_path)
        except Exception:
            return
        for entry in entries:
            full_path = os.path.join(dir_path, entry)
            rel_path = os.path.relpath(full_path, project_path)
            if os.path.isdir(full_path):
                summary.append(f"📁 {rel_path}/")
                walk(full_path, depth + 1)
            else:
                summary.append(f"📄 {rel_path}")

    walk(project_path, 1)
    return "\n".join(summary)
```

This function explores your project up to 3 levels deep and formats a readable list of folders and files.

---

### ✅ Step 2: Store the Summary in the Database

After selecting the project, generate and store its summary:

```python
project_summary = summarize_project_structure(PROJECT_PATH)

conn = get_db()
c = conn.cursor()
c.execute("UPDATE projects SET summary = ? WHERE id = ?", (project_summary, PROJECT_ID))
conn.commit()
conn.close()
print("📂 Project summary stored in database.")
```

---

### ✅ Step 3: Use the Summary in Your Prompts

In your `build_prompt()` function, you can now include the project summary:

```python
def build_prompt(project_name, project_summary, recent_history, user_input):
    system_message = (
        f"You are a helpful coding assistant working on the project '{project_name}'.\n"
        f"Project structure:\n{project_summary}\n"
        "Answer concisely, focus on technical accuracy, and avoid hallucinations.\n\n"
    )
    ...
```

Remember to pass `project_summary` into `build_prompt()` and update the call in your main loop accordingly.

---

### 🚀 Result

Your AI agent will now:

- ✅ Automatically scan and understand the project structure
- ✅ Use this awareness in all conversations
- ✅ Store project details persistently in the database

This makes your AI coding assistant more **project-aware** from the start, helping it provide **better, more relevant answers**.

## 🟣 Continuing the Project Summary Integration

Now that your AI agent can collect and store a simple project summary, let's **improve it step by step** to make it more useful and structured for coding purposes.

---

### ✅ Step 4: Improve the Summary Focus

👉 **Add key file detection inside your existing function**.

In your current `summarize_project_structure()`, after the line where you collect files, add this filtering logic **(only add these lines inside your existing loop):**

```python
key_files = [
    f for f in files if f.endswith((
        '.py', '.js', '.ts', '.ipynb', '.java'
    )) or f in ["requirements.txt", "Dockerfile", "package.json"]
]
for f in key_files:
    summary_lines.append(f"{indent}  - 📄 {f}")
```

📌 **Why**: This highlights important code files and build files, avoiding clutter.

✅ **Hint**: Leave the rest of the function unchanged — you are just adding the file filtering inside your loop.

---

### ✅ Step 5: Limit the Summary Length

After your function returns the string summary, limit the number of characters to avoid overflowing the model context.

📌 **In your main script after generating the summary**, change this line:

```python
project_summary = summarize_project_structure(PROJECT_PATH)
```

➡️ to this:

```python
project_summary = summarize_project_structure(PROJECT_PATH)[:1000]
```

✅ **Why**: This ensures the summary remains concise and fits within prompt limits.

---

### ✅ Step 6: Use a Silent Prompt to Improve Summary Quality

After generating the structure summary, **let the LLM rephrase it into a coherent natural language summary**.

👉 **Add this code after `project_summary` generation:**

```python
silent_prompt = (
    f"<|system|>\nYou are a code assistant. Here's the structure of the project '{project_name}':\n"
    f"{project_summary}\n"
    "Provide a short summary of what this project seems to be about, mentioning the main technologies if visible.\n"
    "<|user|>\nSummarize this project.\n<|assistant|>\n"
)

silent_response = llm(silent_prompt, max_tokens=300)
project_summary = silent_response["choices"][0]["text"].strip()
```

✅ **Why**: This produces cleaner summaries focused on project purpose and technologies.

✅ **Reminder**: Leave your `summarize_project_structure()` function unchanged except where you added `key_files`.

---

### ✅ Step 7: Store the Improved Summary

You already have this part done:

```python
conn = get_db()
c = conn.cursor()
c.execute("UPDATE projects SET summary = ? WHERE id = ?", (project_summary, PROJECT_ID))
conn.commit()
conn.close()
```

✅ **Keep this part unchanged**. The summary now stored is more useful for prompt conditioning.

---

### 🚀 **Final Outcome**

Your AI agent now:

- ✅ Extracts project structure up to a given depth
- ✅ Focuses on relevant source and config files
- ✅ Sends a short version to the LLM for summarization
- ✅ Stores the improved summary in the database
- ✅ Uses the refined summary in subsequent prompts

Students learn step-by-step **refinement** of project awareness using LLM prompting techniques.
