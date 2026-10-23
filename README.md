TermiAI

A lightweight terminal AI assistant powered by the Google Gemini API.

TermiAI is a Python-based AI chat client designed to run on Android through Termux. Instead of running a large AI model locally, it sends requests to Google's cloud API, keeping the local installation lightweight.

Features

- ☁️ Cloud-powered AI using the Gemini API
- 📱 Runs directly on Android with Termux
- 🐍 Written in Python
- 📦 Uses Python's standard library for HTTP requests and JSON processing
- 💬 Maintains conversation history locally
- 🎨 Colored terminal interface
- ⚙️ Configurable API key and model
- 🚀 Launch TermiAI by typing "TermiAI" from any directory
- 🛡️ Confirmation prompt when launching from an unrelated working directory

Requirements

- An Android device
- Termux
- Python 3
- A Google Gemini API key from Google AI Studio
- An internet connection

1. Install the Required Packages

Open Termux and run:

pkg update && pkg upgrade -y
pkg install python nano -y

TermiAI uses Python's built-in "urllib" and "json" modules for API communication, so additional Python HTTP SDKs are not required.

2. Create the Project Directory

mkdir -p ~/TermiAI
cd ~/TermiAI

The main application file should be located at:

~/TermiAI/main.py

Place the TermiAI Python application code in "main.py".

3. Configure Your Gemini API Key

Obtain an API key through Google AI Studio.

Open the Bash configuration file:

nano ~/.bashrc

Add the following lines at the end of the file:

export GEMINI_API_KEY='YOUR_ACTUAL_API_KEY'
export GEMINI_MODEL='gemini-3.1-flash-lite'

Replace "YOUR_ACTUAL_API_KEY" with your actual API key.

Save the file using "Ctrl + O", press Enter, and exit with "Ctrl + X".

Load the configuration:

source ~/.bashrc

These environment variables will be loaded automatically in new Bash sessions.

Security: Never publish your API key in source code, screenshots, public repositories, or shared configuration files.

4. Run TermiAI

cd ~/TermiAI
python main.py

TermiAI will use the configured model and API key.

5. Create a Global Launcher

Create a directory for personal commands:

mkdir -p ~/bin
nano ~/bin/TermiAI

Paste this launcher script:

'''bash
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
'''
Save the file and exit Nano.

Make the launcher executable:

chmod +x ~/bin/TermiAI

6. Add the Launcher to PATH

Open the Bash configuration file:

nano ~/.bashrc

Add this line if it does not already exist:

export PATH="$HOME/bin:$PATH"

Save the file, then reload the configuration:

source ~/.bashrc

7. Launch TermiAI from Anywhere

You can now launch the application by typing:

TermiAI

The launcher behaves as follows:

- Home directory: Launches immediately.
- TermiAI directory: Launches immediately.
- Any other directory: Displays a confirmation prompt in English.
- Enter "y": Confirms the launch.
- Enter "n": Cancels the launch.

When confirmed, the application changes its working directory to "~/TermiAI". Files in the original directory are not deleted or moved.

For uninterrupted work, open a new Termux session before launching TermiAI.

8. Available Chat Commands

Inside the application:

Command| Description
"/help"| Display available commands
"/clear"| Clear the current conversation history
"/history"| Display recent conversation messages
"/model"| View or change the model for the current session
"/exit"| Exit TermiAI

Conversation history is stored locally in:

~/.termai_history.json

9. Troubleshooting

API error 404

Check that the model ID is valid and available to your API project.

Update "GEMINI_MODEL" in "~/.bashrc", reload the configuration, and restart TermiAI.

API error 503

The model may be experiencing temporary high demand. Try again later or select another available model.

API key not set

Check that "GEMINI_API_KEY" is configured in "~/.bashrc" and reload the configuration:

source ~/.bashrc

Command not found

Check that the launcher is executable and that "~/bin" is included in "PATH".

chmod +x ~/bin/TermiAI
source ~/.bashrc

10. Complete Setup Command Reference

The following commands summarize the installation and configuration process. Run them in order, completing the file-editing steps when prompted.

Install dependencies and create directories

pkg update && pkg upgrade -y
pkg install python nano -y
mkdir -p ~/TermiAI ~/bin

Configure the API key and model

Edit the Bash configuration:

nano ~/.bashrc

Add the following lines, replacing the API key placeholder with your actual key:

export GEMINI_API_KEY='YOUR_ACTUAL_API_KEY'
export GEMINI_MODEL='gemini-3.1-flash-lite'
export PATH="$HOME/bin:$PATH"

Save and reload:

source ~/.bashrc

Create and configure the launcher

nano ~/bin/TermiAI

Paste the launcher script from Section 5, save it, and run:

chmod +x ~/bin/TermiAI

Start TermiAI

Ensure that "main.py" exists in "~/TermiAI", then run:

TermiAI

---

TermiAI is an independent community project concept. Google Gemini API access, model availability, and usage limits are controlled by Google and may change over time.
