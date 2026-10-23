# TermiAI

A lightweight terminal AI assistant powered by the Google Gemini API.

TermiAI is a Python-based AI chat client designed to run on Android through Termux. Instead of running a large AI model locally, it sends requests to Google's cloud API, keeping the local installation lightweight.

## Features

- ☁️ Cloud-powered AI using the Gemini API
- 📱 Runs directly on Android with Termux
- 🐍 Written in Python
- 📦 Uses Python's standard library for HTTP requests and JSON processing
- 💬 Maintains conversation history locally
- 🎨 Colored terminal interface
- ⚙️ Configurable API key and model
- 🚀 Launch TermiAI by typing `TermiAI` from any directory
- 🛡️ Confirmation prompt when launching from an unrelated working directory

## Requirements

- An Android device
- Termux
- Python 3
- A Google Gemini API key from Google AI Studio
- An internet connection

## 1. Install the Required Packages

Open Termux and run:

```bash
pkg update && pkg upgrade -y
pkg install python nano -y
```

TermiAI uses Python's built-in `urllib` and `json` modules for API communication, so additional Python HTTP SDKs are not required.

## 2. Create the Project Directory

```bash
mkdir -p ~/TermiAI
cd ~/TermiAI
nano main.py
```

Then copy the code below and paste it into `main.py`:

```python
import os
import sys
import json
import urllib.request
import urllib.error
from pathlib import Path
from datetime import datetime

API_KEY = os.getenv("GEMINI_API_KEY")
MODEL = os.getenv("GEMINI_MODEL")
API_URL = (
    f"https://generativelanguage.googleapis.com/v1beta/"
    f"models/{MODEL}:generateContent"
)

HISTORY_FILE = Path.home() / ".termai_history.json"
SYSTEM_PROMPT = (
    "You are TermiAI, a helpful AI assistant. "
    "Reply in the same language as the user unless asked otherwise."
)

GREEN = "\033[92m"
CYAN = "\033[96m"
YELLOW = "\033[93m"
RED = "\033[91m"
DIM = "\033[2m"
RESET = "\033[0m"
BOLD = "\033[1m"


def load_history():
    try:
        data = json.loads(HISTORY_FILE.read_text(encoding="utf-8"))
        if isinstance(data, list):
            return data
    except (OSError, json.JSONDecodeError):
        pass
    return []


def save_history(history):
    try:
        HISTORY_FILE.write_text(
            json.dumps(history, ensure_ascii=False, indent=2),
            encoding="utf-8",
        )
    except OSError as exc:
        print(f"{YELLOW}Could not save history: {exc}{RESET}")


def print_banner():
    os.system("clear")
    print(f"{GREEN}{BOLD}")
    print("╭────────────────────────────────────────╮")
    print("│               TermiAI                   │")
    print("│          Python · Gemini Cloud          │")
    print("╰────────────────────────────────────────╯")
    print(f"{RESET}{DIM}Model: {MODEL}{RESET}")
    print(f"{DIM}Commands: /help /clear /history /model /exit{RESET}")
    print()


def request_gemini(history):
    contents = []
    for item in history:
        if item.get("role") in ("user", "model"):
            contents.append({
                "role": item["role"],
                "parts": [{"text": item.get("text", "")}],
            })

    payload = {
        "systemInstruction": {
            "parts": [{"text": SYSTEM_PROMPT}]
        },
        "contents": contents,
        "generationConfig": {
            "temperature": 0.7,
            "maxOutputTokens": 2048,
        },
    }

    req = urllib.request.Request(
        API_URL,
        data=json.dumps(payload).encode("utf-8"),
        headers={
            "Content-Type": "application/json",
            "x-goog-api-key": API_KEY,
        },
        method="POST",
    )

    with urllib.request.urlopen(req, timeout=120) as response:
        data = json.loads(response.read().decode("utf-8"))

    candidates = data.get("candidates", [])
    if not candidates:
        feedback = data.get("promptFeedback", {})
        reason = feedback.get("blockReason", "No response returned")
        raise RuntimeError(reason)

    parts = candidates[0].get("content", {}).get("parts", [])
    answer = "".join(
        part.get("text", "")
        for part in parts
        if "text" in part
    )

    if not answer:
        raise RuntimeError("The model returned no text.")
    return answer


def show_help():
    print(f"""
{CYAN}TermiAI commands{RESET}

  /help       Show this help
  /clear      Clear current conversation
  /history    Show recent conversation
  /model      Show or change model
  /exit       Exit TermiAI

Normal text is sent to Gemini.
Conversation history is stored locally.
""")


def show_history(history):
    if not history:
        print(f"{DIM}No conversation yet.{RESET}")
        return

    for item in history[-10:]:
        role = "You" if item["role"] == "user" else "TermiAI"
        color = CYAN if role == "You" else GREEN
        print(f"\n{color}{role}:{RESET}")
        print(item.get("text", ""))


def main():
    global MODEL, API_URL

    if not API_KEY:
        print(f"{RED}GEMINI_API_KEY is not set.{RESET}")
        print("Set it first with:")
        print("export GEMINI_API_KEY='your_api_key'")
        return

    history = load_history()
    print_banner()

    while True:
        try:
            prompt = input(f"{GREEN}{BOLD}You › {RESET}").strip()
        except (EOFError, KeyboardInterrupt):
            print(f"\n{DIM}TermiAI closed.{RESET}")
            break

        if not prompt:
            continue

        command = prompt.lower()

        if command in ("/exit", "/quit"):
            print(f"{GREEN}See you next time!{RESET}")
            break

        if command == "/help":
            show_help()
            continue

        if command == "/clear":
            history = []
            save_history(history)
            print(f"{YELLOW}Conversation cleared.{RESET}")
            continue

        if command == "/history":
            show_history(history)
            continue

        if command == "/model":
            print(f"Current model: {MODEL}")
            new_model = input(
                "New model name (Enter to keep current): "
            ).strip()
            if new_model:
                MODEL = new_model
                API_URL = (
                    "https://generativelanguage.googleapis.com/v1beta/"
                    f"models/{MODEL}:generateContent"
                )
                print(f"{GREEN}Model set to {MODEL}{RESET}")
            continue

        history.append({"role": "user", "text": prompt})

        print(f"\n{GREEN}{BOLD}TermiAI › {RESET}", end="", flush=True)

        try:
            answer = request_gemini(history)
            print(answer)
            history.append({"role": "model", "text": answer})
            save_history(history)

        except urllib.error.HTTPError as exc:
            details = exc.read().decode("utf-8", errors="replace")
            try:
                message = json.loads(details).get("error", {}).get(
                    "message", details
                )
            except json.JSONDecodeError:
                message = details
            print(f"\n{RED}API error ({exc.code}): {message}{RESET}")
            history.pop()

        except urllib.error.URLError as exc:
            print(f"\n{RED}Network error: {exc.reason}{RESET}")
            history.pop()

        except (TimeoutError, json.JSONDecodeError, RuntimeError) as exc:
            print(f"\n{RED}Error: {exc}{RESET}")
            history.pop()

        except Exception as exc:
            print(f"\n{RED}Unexpected error: {exc}{RESET}")
            history.pop()

        print()


if __name__ == "__main__":
    main()
```

Save the file with `Ctrl + O`, press Enter, and exit with `Ctrl + X`.

## 3. Configure Your Gemini API Key

Obtain an API key through Google AI Studio.

Open the Bash configuration file:

```bash
nano ~/.bashrc
```

Add the following lines at the end of the file:

```bash
export GEMINI_API_KEY='YOUR_ACTUAL_API_KEY'
export GEMINI_MODEL='gemini-3.1-flash-lite'
```

Replace `YOUR_ACTUAL_API_KEY` with your actual API key.

Save the file using `Ctrl + O`, press Enter, and exit with `Ctrl + X`.

Load the configuration:

```bash
source ~/.bashrc
```

These environment variables will be loaded automatically in new Bash sessions.

> **Security:** Never publish your API key in source code, screenshots, public repositories, or shared configuration files.

## 4. Run TermiAI

```bash
cd ~/TermiAI
python main.py
```

TermiAI will use the configured model and API key.

## 5. Create a Global Launcher

Create a directory for personal commands:

```bash
mkdir -p ~/bin
nano ~/bin/TermiAI
```

Paste this launcher script:

```bash
#!/data/data/com.termux/files/usr/bin/bash

APP_DIR="$HOME/TermiAI"
CURRENT_DIR="$(pwd -P)"
HOME_DIR="$(realpath "$HOME")"
APP_REAL="$(realpath "$APP_DIR" 2>/dev/null)"

if [ ! -f "$APP_DIR/main.py" ]; then
    echo "Error: TermiAI/main.py was not found."
    exit 1
fi

if [ "$CURRENT_DIR" != "$HOME_DIR" ] &&
   [ "$CURRENT_DIR" != "$APP_REAL" ]; then

    echo "You are currently in: $CURRENT_DIR"
    echo
    echo "Open TermiAI from a directory other than your home"
    echo "directory or TermiAI directory?"
    echo "The current terminal screen will be taken over by TermiAI."
    echo "It is recommended to create a new session instead."
    echo
    read -r -p "Confirm opening TermiAI from this directory? (y/n): " answer

    case "$answer" in
        y|Y|yes|YES)
            ;;
        *)
            echo "Cancelled. Your current directory is unchanged."
            exit 0
            ;;
    esac
fi

cd "$APP_DIR" || exit 1
exec python main.py
```

Save the file and exit Nano.

Make the launcher executable:

```bash
chmod +x ~/bin/TermiAI
```

## 6. Add the Launcher to PATH

Open the Bash configuration file:

```bash
nano ~/.bashrc
```

Add this line if it does not already exist:

```bash
export PATH="$HOME/bin:$PATH"
```

Save the file, then reload the configuration:

```bash
source ~/.bashrc
```

## 7. Launch TermiAI from Anywhere

You can now launch the application by typing:

```bash
TermiAI
```

The launcher behaves as follows:

- **Home directory:** Launches immediately.
- **TermiAI directory:** Launches immediately.
- **Any other directory:** Displays a confirmation prompt in English.
- **Enter `y`:** Confirms the launch.
- **Enter `n`:** Cancels the launch.

When confirmed, the application changes its working directory to `~/TermiAI`. Files in the original directory are not deleted or moved.

For uninterrupted work, open a new Termux session before launching TermiAI.

## 8. Available Chat Commands

Inside the application:

| Command    | Description                                  |
| ---------- | -------------------------------------------- |
| `/help`    | Display available commands                   |
| `/clear`   | Clear the current conversation history       |
| `/history` | Display recent conversation messages         |
| `/model`   | View or change the model for the current session |
| `/exit`    | Exit TermiAI                                 |

Conversation history is stored locally in:

```text
~/.termai_history.json
```

## 9. Troubleshooting

### API error 404

Check that the model ID is valid and available to your API project.

Update `GEMINI_MODEL` in `~/.bashrc`, reload the configuration, and restart TermiAI:

```bash
nano ~/.bashrc
source ~/.bashrc
```

### API error 503

The model may be experiencing temporary high demand. Try again later or select another available model.

### API key not set

Check that `GEMINI_API_KEY` is configured in `~/.bashrc` and reload the configuration:

```bash
source ~/.bashrc
```

### Command not found

Check that the launcher is executable and that `~/bin` is included in `PATH`:

```bash
chmod +x ~/bin/TermiAI
source ~/.bashrc
```

## 10. Complete Setup Command Reference

The following commands summarize the installation and configuration process. Run them in order, completing the file-editing steps when prompted.

### Install dependencies and create directories

```bash
pkg update && pkg upgrade -y
pkg install python nano -y
mkdir -p ~/TermiAI ~/bin
```

### Configure the API key and model

Edit the Bash configuration:

```bash
nano ~/.bashrc
```

Add the following lines, replacing the API key placeholder with your actual key:

```bash
export GEMINI_API_KEY='YOUR_ACTUAL_API_KEY'
export GEMINI_MODEL='gemini-3.1-flash-lite'
export PATH="$HOME/bin:$PATH"
```

Save and reload:

```bash
source ~/.bashrc
```

### Create and configure the launcher

```bash
nano ~/bin/TermiAI
```

Paste the launcher script from Section 5, save it, and run:

```bash
chmod +x ~/bin/TermiAI
```

### Start TermiAI

Ensure that `main.py` exists in `~/TermiAI`, then run:

```bash
TermiAI
```

---

TermiAI is an independent community project concept. Google Gemini API access, model availability, and usage limits are controlled by Google and may change over time.
