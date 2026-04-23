# 00. Getting Started: Global Prerequisites

Welcome to the Sherab project! Before diving into setting up the platform, let's make sure your computer has the necessary tools installed. 

This is the most critical step. If you miss something here, you will face errors later on. Please follow these steps carefully.

## 1. Operating System
- **macOS** or **Linux** (Ubuntu/Debian) is highly recommended. 
- If you are on **Windows**, please install WSL2 (Windows Subsystem for Linux) and use Ubuntu. Do not try to natively install this on Windows.

## 2. Install Docker
Docker is used to run Open edX in isolated "containers" so it doesn't mess up your computer.
- **Mac/Windows**: Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
- Ensure docker is running (you should see the whale icon in your menu bar). Setup Docker to have at least 8GB of RAM and ideally 4+ CPU cores allocated in Docker Desktop settings.

## 3. Install Python
We use Python to run the Tutor command-line tool.
- Check if you have Python by opening a terminal and running `python3 --version`.
- We recommend **Python 3.8 to 3.10** (avoid versions drastically newer if possible, to avoid compatibility issues).
- Need to install it? Download it from [python.org](https://www.python.org/downloads/).

## 4. Install Node.js and npm
Node is required if you are working on the front-end (Micro Frontends - MFEs).
- Check your version: `node -v` and `npm -v`.
- We recommend installing via **NVM (Node Version Manager)**. You will likely need **Node 16 or Node 18**.
- [Install NVM](https://github.com/nvm-sh/nvm), then run `nvm install 18` and `nvm use 18`.

## 5. Install Git
Git tracks the code.
- Check: `git --version`
- [Download Git here](https://git-scm.com/downloads).

---
**Next Step:** Head over to [01-tutor-installation-upgrade-guide.md](01-tutor-installation-upgrade-guide.md) to get the platform installed!
