Click to watch the code
https://colab.research.google.com/drive/1Ry8joq9gvi6vuCN5JY9LcUfH-9bBdDH-?usp=sharing

# 🤖 LangGraph Command Tracker

A LangGraph workflow that classifies, validates, and tracks file operation commands. Built to understand and demonstrate core LangGraph concepts — StateGraph, nodes, edges, conditional routing, and state accumulation.

---

## 🚀 How It Works

```
User types a command
        ↓
Node 1: classify_command
Identifies what type of action it is
        ↓
Node 2: validate_command
Checks if the command is safe
        ↓
       / \
      /   \
  Safe?  Dangerous?
    ↓        ↓
Node 3    Node 4
Log it    Reject it
    ↓        ↓
   END      END
```

---

## ✅ What It Does

- Classifies voice/text commands into 6 file operation types
- Validates commands against dangerous patterns
- Routes to different nodes based on validation result
- Accumulates command history across the entire session
- Prints a full history of all commands processed

---

## 🧩 LangGraph Concepts Used

| Concept | Where Used |
|---|---|
| `StateGraph` | Main graph container with `CommandState` schema |
| Nodes | 4 nodes — classify, validate, log, reject |
| Edges | `classify → validate` fixed edge |
| Conditional Edges | After validation — routes to log or reject |
| `compile()` | Locks and prepares graph for execution |
| State Accumulation | `Annotated[list, operator.add]` — history grows automatically |

---

## 🔍 Node Details

### Node 1 — `classify_command`
Reads the user command and identifies which of the 6 action types it is by checking for keywords:

| Keyword | Classified As |
|---|---|
| "create" | `create_file` |
| "edit" | `edit_file` |
| "append" | `append_file` |
| "delete" | `delete_file` |
| "rename" | `rename_file` |
| "run" | `run_script` |
| none matched | `unknown` |

### Node 2 — `validate_command`
Checks two things:
- Is the command type known? If `unknown` → rejected
- Does it contain dangerous patterns? (`system32`, `../`, `rm -rf`, `.exe`, `format`) → rejected

### Node 3 — `log_command`
Reached only when command is valid. Saves the command to state history.

### Node 4 — `handle_invalid`
Reached only when command is invalid. Prints rejection reason and saves to history.

---

## 📊 State Schema

```python
class CommandState(TypedDict):
    command: str                           # raw user input
    command_type: str                      # classified action type
    is_valid: bool                         # validation result
    reason: str                            # explanation
    history: Annotated[list, operator.add] # auto-accumulating history
```

`Annotated[list, operator.add]` means every time a node adds to history it appends rather than replacing — so the full session history is preserved automatically.

---

## 💬 Example Session

```
🤖 LangGraph Command Tracker
✅ LangGraph compiled successfully

Enter command: create a file named hello.py
🔍 Classified as: create_file
📝 Logged: {command: "create...", type: "create_file", valid: True}
✅ Valid — Command is safe to execute.

Enter command: delete system32
🔍 Classified as: delete_file
❌ Rejected: Dangerous pattern detected: 'system32'

Enter command: open browser
🔍 Classified as: unknown
❌ Rejected: Command type not recognized.

Enter command: history
📋 Command History:
1. ✅ [create_file] create a file named hello.py
2. ❌ [delete_file] delete system32
3. ❌ [unknown]     open browser

Enter command: exit
👋 Goodbye!
```

---

## 🛠️ Technologies Used

| Technology | Role |
|---|---|
| Python | Core language |
| LangGraph | Workflow graph — nodes, edges, state, routing |
| TypedDict | State schema definition |
| operator.add | Automatic list accumulation in state |

---

## ⚙️ Setup & Installation

### 1. Install dependencies

```bash
pip install langgraph
```

### 2. Run the project

```bash
python langgraph_command_tracker.py
```

### 3. Commands you can try

```
create a file named app.py
edit hello.py and fix the loop
append a function to utils.py
delete old_file.py
rename main.py to app.py
run script.py
delete system32        ← will be rejected
open browser           ← will be rejected (unknown)
history                ← shows all processed commands
exit                   ← stops the program
```

---

## 📌 Known Limitations

- **Keyword-based classification** — uses simple keyword matching, not AI
- **No actual file execution** — this is a tracking and validation layer only
- **English only** — command parsing works on English input

---

## 🔗 Related Project

This project is a companion to the
[Voice-Driven Coding Assistant](https://github.com/SaimAhmad-h/voice-driven-coding-assistant)
— which uses Whisper + Gemini to execute real file operations from voice input.
This tracker adds a proper LangGraph state management layer to that concept.

---

## 🔮 Future Enhancements

- Connect to the Voice-Driven Coding Assistant as a state management layer
- Add LLM-based classification instead of keyword matching
- Add persistent storage using LangGraph's SqliteSaver checkpoint
- Add more validation rules and safety checks

---

## 📄 License

Open source. Free to use and modify.
