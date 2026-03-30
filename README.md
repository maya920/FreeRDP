# FreeRDP

**Repo:** [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP)

Run a full Windows desktop session from a GitHub Actions runner — accessible anywhere through Tailscale, with VS Code, Node, Python and more pre-installed automatically.

The idea is simple: trigger the workflow, wait a few minutes while everything sets up, then RDP in and get to work. No cloud bills, no VPS to manage. Just a free GitHub runner doing its thing.

> ⏳ **Sessions last up to 6 hours** — that's GitHub's hard limit on free runners. Plan accordingly.

---

## How it works

When you trigger the workflow manually, it:

1. Clears out everything GitHub pre-installs that you don't need — this frees up a serious amount of disk space
2. Configures Windows Remote Desktop and opens port 3389
3. Creates a local Windows user called `RDP` using your chosen password from secrets
4. Installs and connects Tailscale so you can reach the runner from anywhere on your network
5. Installs Node.js, Python, GitHub CLI, and Bun
6. Configures Git with your name and email
7. Installs VS Code with a full set of extensions for web and full-stack development
8. Sits in a loop printing the connection details every 5 minutes until you cancel it

---

## Use This Project

The fastest way to get started is to fork the repo directly from GitHub:

👉 [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP) → click **Fork** in the top right

That copies everything into your own account. Then just add your secrets and you're good to go. No cloning, no setup beyond that.

If you'd rather start fresh in an existing repo, just grab the `.github/workflows/main.yml` file and drop it in your own repo at the same path.

---

## Setup

### 1. Fork or copy this repo

Fork from [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP) or add the `.github/workflows/main.yml` file to your own repository.

### 2. Add your secrets

Go to your repo → **Settings → Secrets and variables → Actions → New repository secret** and add these:

| Secret | Required | What to put |
|--------|----------|-------------|
| `RDP_PASSWORD` | ✅ Yes | Password for the RDP user. Must meet Windows complexity rules — at least 8 characters with uppercase, lowercase, a number, and a special character |
| `TAILSCALE_AUTH_KEY` | ✅ Yes | An auth key from your [Tailscale admin panel](https://login.tailscale.com/admin/settings/keys). Use an ephemeral key so it auto-cleans up |
| `GIT_USERNAME` | Optional | Your Git display name |
| `GIT_EMAIL` | Optional | Your Git email address |

If `GIT_USERNAME` or `GIT_EMAIL` are missing, the Git config step just skips — nothing breaks.

### 3. Install Tailscale on your device

Download Tailscale on whatever device you'll RDP from — [tailscale.com/download](https://tailscale.com/download). Log in with the same account you used to generate the auth key. That's it.

---

## Running it

1. Go to the **Actions** tab in your repo
2. Select **RDP** from the left sidebar
3. Click **Run workflow**
4. Wait roughly 10–15 minutes for setup to finish
5. Open the workflow logs and look for the `Maintain Connection` step — it prints the Tailscale IP, username, and password
6. Open your RDP client, connect to that IP, and log in with `RDP` and your password

On Windows, use the built-in Remote Desktop Connection app (`mstsc`). On Mac, grab [Microsoft Remote Desktop](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466) from the App Store. The same app is available on Android and iOS too.

---

## What gets removed

GitHub runners come loaded with things most people will never touch. The workflow removes all of it upfront to free up around **40–50 GB** of disk space. Here's everything that gets cleaned out:

**Development tools & runtimes**
- Android Studio and Android SDK
- Miniconda / Anaconda
- Haskell (GHCup)
- Rust and Cargo
- Go
- PHP
- Java
- R and RTools
- Strawberry Perl
- .NET SDK
- Boost C++ libraries
- LLVM / Clang

**Microsoft tools**
- Microsoft Visual Studio (the full IDE — not VS Code, that stays)
- Microsoft SQL Server
- Microsoft SDKs and Windows Kits
- Azure CLI
- Azure Cosmos DB Emulator
- Azure Service Fabric

**Other**
- Amazon / AWS tools and config
- CMake
- Unity Hub and Unity
- Mozilla Firefox
- Selenium WebDrivers
- Chocolatey package cache
- Temp files

After cleanup you should have around 40–50 GB free on C:.

---

## What gets installed

### Node.js
Installed via winget. These global packages come with it:
`yarn`, `pnpm`, `typescript`, `ts-node`, `nodemon`

### Python
Pre-installed on the runner. These pip packages come with it:
`requests`, `numpy`, `pandas`, `flask`, `fastapi`, `uvicorn`, `django`, `python-dotenv`, `httpx`, `pytest`, `black`, `pylint`

### CLI Tools
- **GitHub CLI** (`gh`) — for working with GitHub from the terminal
- **Wget** — for downloading files
- **Bun** — fast JavaScript runtime and package manager

### VS Code
Installed silently. These extensions come pre-installed:

**GitHub**
- GitHub Copilot
- GitHub Actions
- GitHub Pull Requests

**Python / AI / Notebooks**
- Python, Pylance, Black Formatter
- Jupyter
- Google Colab — connects VS Code notebooks directly to Colab runtimes, including free GPUs and TPUs

**HTML / CSS / JS**
- Auto Rename Tag, Auto Close Tag
- Code Runner
- ESLint, Prettier
- CSS Snippets, CSS Peek
- Tailwind Fold, Tailwind CSS IntelliSense
- ES7+ React/Redux Snippets

**IntelliSense**
- Path IntelliSense
- npm IntelliSense
- Quokka.js

**Productivity**
- Material Icon Theme
- GitLens
- Live Share
- Edit CSV
- DotENV
- REST Client
- Thunder Client (Postman alternative)

**Full Stack Extras**
- Prisma
- Docker (by Microsoft)
- Live Server
- Code Spell Checker

> **Note on Docker:** Docker Desktop is not installed in this workflow. It requires WSL2 or Hyper-V to run properly, and setting that up inside an RDP session is more trouble than it's worth for most use cases. If you need containers, the Docker VS Code extension is still there for syntax support and Dockerfile editing.

---

## Secrets summary

| Secret | Required | Description |
|--------|----------|-------------|
| `RDP_PASSWORD` | ✅ | Windows password for the `RDP` user |
| `TAILSCALE_AUTH_KEY` | ✅ | Tailscale auth key (ephemeral recommended) |
| `GIT_USERNAME` | Optional | Git global user.name |
| `GIT_EMAIL` | Optional | Git global user.email |

---

## Things to know

**The 6-hour limit is real.** GitHub free runners stop after 360 minutes. `timeout-minutes: 365` in the workflow means it'll hit GitHub's limit before the workflow timeout fires. If you need longer sessions, you'd need a paid GitHub plan or a self-hosted runner.

**Tailscale auth keys expire.** If your key is expired, the Tailscale step will fail. Generate a new one from the Tailscale admin panel and update the secret.

**Nothing persists between runs.** Every time you trigger the workflow, you get a fresh runner. Anything you create or download during the session is gone when it ends. If you need files to persist, push them to Git or upload them somewhere before the session ends.

**The password needs to meet Windows complexity rules.** If `RDP_PASSWORD` doesn't have uppercase + lowercase + number + special character, the user creation step will fail and the whole workflow stops there.

**VS Code extensions install under the runner user.** When you RDP in as the `RDP` user, VS Code may not show the extensions immediately. Open VS Code once and they should sync up. Running `code --list-extensions` in the terminal will confirm they're installed.

---

## Stopping the session

Cancel the workflow run from the Actions tab. The runner shuts down and Tailscale removes the ephemeral node automatically if you used an ephemeral auth key.

---

## File structure

```
FreeRDP/
└── .github/
    └── workflows/
        └── main.yml    ← the whole thing lives here
```

One file does all the setup, connection, and cleanup. That's intentional.

---

## Contributing

Found a bug, have a better cleanup path, or want to add something useful? Pull requests are open.

A few things that would be worth adding:

- Auto-detect and clean up more pre-installed runner bloat
- Optional step to auto-clone a specific repo into the session on startup
- A way to push files back to GitHub automatically before the session ends
- PostgreSQL / MongoDB install as optional steps

To contribute, fork the repo, make your changes in a branch, and open a PR against `main`. One thing per PR is easier to review.

---

## Known issues

**winget sometimes needs a moment.** On rare occasions winget on GitHub runners is slow to initialize and a package install fails on the first attempt. If a run fails at an install step, just re-trigger it — it usually works on the second try.

**Tailscale auth key must be valid.** If the key is expired or revoked, the connection step will fail and the workflow exits early. Check the key expiry in your Tailscale admin panel before a run.

---

## Credits

Built and maintained by [@maya920](https://github.com/maya920).

Uses [Tailscale](https://tailscale.com) for secure networking and [GitHub Actions](https://github.com/features/actions) free runners for compute. Both are free at the tier this project uses.