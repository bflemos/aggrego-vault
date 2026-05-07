# 🦞 Definitive Guide: Turning Your WSL into a Personal Assistant with OpenClaw 🦞

![If I were a Lobster](lobster-me.png "Myself as a Lobster")

## 🥸 About the Author
Bruno is a Generative Artificial Intelligence enthusiast and Tech Lead specializing in application support and Site Reliability Engineering (SRE). At 39 years old and living in Campinas - SP, he divides his time between technical leadership in complex corporate environments, exploring new technologies (like Python, TypeScript, and React), and the continuous development of projects focused on custom AI assistants. This guide was born from his practical experiences testing, failing, and building his own local AI ecosystems.

## 📝 Introduction 
The concept of having a virtual assistant is not new, but the possibility of having a completely private one, running on your own machine and directly accessible through your favorite messaging apps, is a gamechanger. This practical guide documents the exact path to install and configure OpenClaw within your Windows Subsystem for Linux (WSL) environment.
In this guide, you won’t just find commands, but also practical insights from someone who has already walked this path (and had to reinstall everything from scratch to learn the right way). The goal is that, by the end of these pages, you will have an assistant running securely, intelligently, and perfectly integrated into your workflow.

## 🖥 A note about your Operating System
All the step-by-step instructions built in this guide were designed running on WSL2 (Ubuntu on Windows). However, OpenClaw and its dependencies (Node.js, Ollama, Homebrew) are native tools of the Unix ecosystem. If you already have a machine running a Linux distribution directly on the hardware, you are good to go. All installation commands, port configurations, and service activations (systemd) detailed in the next chapters apply fully to your system, often delivering even better performance for Artificial Intelligence. 

## 📌 What is OpenClaw?
OpenClaw is a program that transforms Artificial Intelligence into your own private personal assistant, running directly on your own computer (inside your WSL/Ubuntu) to guarantee total privacy. You will talk to it directly through Telegram or WhatsApp.

## 🛠️ Preparing the Groundwork (Prerequisites)
Let’s prepare the "engine" that OpenClaw needs to run and the "brain" it will use to think.

> I'll assume you are already running a Linux distribution either directly or via WSL. If not, you can check this link on how to configure WSL on your Win 11: https://learn.microsoft.com/en-us/windows/wsl/install

### Installing the "Engine" (Node.js)
OpenClaw requires Node.js (version 24). In your Ubuntu terminal (WSL), run the commands below:

**Command A:**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

> 🛑 VERY IMPORTANT: After running the command above, close the Ubuntu window and open a new one so it recognizes the installation.

**Command B:** 
```bash
nvm install 24
```

### Choosing the "Brain" (Intelligence)
You have two options to provide intelligence to your assistant:

#### 🟢 OPTION 1: Local Usage via Ollama (100% Free and Private) 🏆

Ollama allows you to run AI on your own computer without needing internet access for thinking.
**Step A:** Install Ollama
```bash
curl -fsSL https://ollama.com/install.sh | bash
```

**Step B:** Download the AI Model (Llama 3)
> 💡 Note: Here we are using llama3 because it is a model that runs well on most machines. If you prefer another model, lighter or more powerful, simply replace the model name in the command below and remember your choice during the configuration process. You can see all models available in Ollama at: https://ollama.com/search

```bash
ollama run llama3
```

> 💡 Note: This may take a few minutes depending on your internet connection. When it finishes and a chat screen appears, type /bye to exit.


#### 🔵 OPTION 2: Cloud Usage (API Key)

If you already have a developer account with AI providers (such as OpenAI/ChatGPT, Google, or Anthropic/Claude):
1. Access your favorite AI provider dashboard.
2. Generate an API Key.
3. Copy this long code and store it safely to use in the next phase.

### Homebrew (The "Tool Installer")
Homebrew is a tool that OpenClaw uses to install some of its **skills**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 🦞 Phase 2: Installation
Now we are going to install OpenClaw and configure it all at once.

**Start the Magic Installation** 
Copy and paste into your WSL:
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

> **What is happening in my terminal?**
> While the lines of text scroll by, the installer is:
> 1. Downloading the core: Bringing the latest OpenClaw files to your machine.
> 2. Configuring the Daemon: Think of the daemon as an invisible butler. It is a process that stays awake in the background, ensuring OpenClaw responds to your messages even if you close the terminal window.

## 🦞 Automatic Configuration
** The Setup Assistant (Onboarding)** 
At the end of the installation, OpenClaw will automatically run a diagnostic (called doctor) and ask if you want to configure the gateway (its central system).

* Select **Yes**
* Setup Mode: Choose **QuickStart**. This automatically configures basic security for you (creating a "shield" so nobody from outside can access your assistant).
* Select all default options until you reach the AI Provider/Model (LLM Model) option.

### AI Provider
> 💡 Note: We are going to configure a 100% local and free setup using Ollama and running a local model. If you have a subscription to a premium model (Google Gemini, OpenAI ChatGPT, Anthropic Claude, etc.) you will need to generate an API key from your provider and configure a new model after completing the setup, or continue with the API-based setup instead of the local one, described below.

1. Select Ollama (to run 100% free and private on your PC).
2. Base URL: Just press Enter to keep the default text.
3. Mode: Select Local.
4. Default Model: Select Keep Current.

### Channel
OpenClaw becomes much more useful when you can talk to it from anywhere. Let’s give "ears and a mouth" to your assistant so you can chat with it from your phone. 📱

> 💡 Note: Let’s start with the easiest setup, using Telegram. If you do not have the app installed yet, download it from your App Store and create an account. After completing the process, you can configure the communication channel through WhatsApp.

1. Select the option: **Telegram (Bot API)**
2. Talk to the "Father of Bots" Open your Telegram app (on mobile or PC) and search for the contact @BotFather (make sure it is the official one, with the blue verification badge).
3. Create your Robot
  1. Send the message /newbot to BotFather.
  2. It will ask you to choose a Name and a "Username" (which must end with the word "bot").
  3. At the end, it will give you an API Token (a very long code with numbers and letters). Copy this code!
4. Back in the OpenClaw settings 💻, select the option: **Enter Telegram bot token**
5. Paste the token generated by BotFather.

Done! Initial Telegram configuration completed! 🚀
After the configuration is completed, we will activate and authorize the Telegram chat.

> 💡 Survival Tip (Memory)
> Whenever you radically change the topic, type /new in the WhatsApp/Telegram conversation. This clears the assistant’s "short-term memory", making it faster and, if you are using the Cloud model option, **saving a lot of credits!**

### 🔍 Search Provider
Here select the option **skip for now**, this will use OpenClaw’s default settings for basic internet search, website access, etc.

In the future, you can add a search provider, which usually requires an API key, if you have any preference or subscription.

### 🤹 Skills
"Skills" are like the apps you install on your phone. OpenClaw already comes with "Core Tools" that handle the basics, such as reading and writing files on your PC or browsing the web.

For specific tasks that will help turn OpenClaw into a personal assistant, here are the Skills I recommend selecting with the space bar right now:
* 🧩 clawhub (Required): It is OpenClaw’s "App Store". By installing this now, we can install or remove any other skill later using just one line in the chat.
* 🧾 summarize: Extremely useful. Helps the assistant read giant articles, news, and files and give you a clean summary without using too much memory.
* 💎 obsidian: It is currently the best option for helping you create, format, and organize notes and texts in a structured way.
* 🐦 xurl: Teaches the assistant how to "read" links to products, news, or websites that you paste for it.
* 🎤 openai-whisper: Since you will use Telegram (and maybe WhatsApp), this skill will give it the fantastic ability to listen to and transcribe voice messages!

**Homebrew recommended** and **Show Homebrew install command?**
If these options appear, select **No** so the installation command is not shown, we already completed this part in phase 1, at the beginning of the tutorial.

**Preferred node manager for skill installs**
Select the option **npm**

💡 The selected skills will be installed. After completion, several options for setting API Keys for skill usage will appear, select **No** for all of them.

### 🪝 Hooks
Hooks are like "Automations" executed under certain conditions.

At this moment we will select two options:
* 💾 session-memory - Whenever the agent’s conversation is cleared with the `/new` command, this hook will automatically create a "little summary" of the entire conversation and save it into a secure notes file on your computer (a long-term memory). This way, it clears short-term memory (keeping it fast), but maintains a searchable file with the history of your conversations for future reference if you ask about something from the past.
* 📝 command-logger - Ensures that every command executed by the agent is saved into a log. This way, if there is any issue or if you want to know exactly what the agent did after you asked it to fix your photo folder, you can retrieve everything that was executed.


## 🦞🚀 Final Stretch
We have reached the final stage for your new favorite crustacean to be born! 🎉

At this stage the Gateway will be started and you will receive several pieces of information about how to access the local web console (something like the address http://127.0.0.1:18789/#token=...)

When you see the option "How do you want to hatch your bot?" select **Hatch in TUI (recommended)**

At this moment, the setup assistant will be completed and the chat with your agent will begin.

Here you can already talk to your agent as if it were ChatGPT, Gemini, or another AI you are already used to!

> 💡 Notice the message "Enabled systemd lingering", this is excellent news and means that your Linux (WSL) has been configured to keep the OpenClaw engine running forever, even if you close the Ubuntu terminal.

> ⚠️ Save the OpenClaw access token carefully, it is what guarantees that only you can access your agent inside your network. If you lose the token and need to discover it again, run the command `openclaw config get gateway.auth.token` in the terminal.


## 📱 Telegram
Now that the installation and configuration assistant is complete, it is time to activate your Telegram chat.
With all configurations already completed, this step will be very simple!

1. Go to Telegram and start a conversation with your Bot.
  * You can find the bot by searching for its name in your contact list or, in the conversation with BotFather, it sent you a direct contact link to your new bot after creation.
2. Say hello, or any other message.
  * The bot will reply with a message saying that access has not been configured. In this same message it will inform your Telegram user ID and the OpenClaw authentication code. And to make it even easier, it will provide the exact command you need to run in your terminal, something like `openclaw pairing approve telegram XXXXXXXX`
3. Copy this command and run it in your terminal.

Now everything is done and configured! You can already talk to your crustacean agent from anywhere! 🦞🚀

# Final Considerations
I hope this guide was useful, it was built while I was configuring OpenClaw on my machine for the second time, since on the first attempt I made so many mistakes that I preferred deleting my WSL instance and starting from scratch! 😅

> 💡 Tip: basically every `openclaw` command has a help page by adding `--help`. So when in doubt, abuse this help. You can also ask your agent how to do things and normally it can answer you. Example: `openclaw --help` `openclaw models --help` `openclaw skills --help`
> 💡 Another thing is to keep the OpenClaw documentation page saved in your favorites, it is very useful: https://docs.openclaw.ai/

---
*This guide was created by Bruno Lemos and is part of [aggrego-vault](link-do-repo). The content here is licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.en).*
