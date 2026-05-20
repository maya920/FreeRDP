# FreeRDP

**Repo:** [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP)

Run a full Windows desktop session from a GitHub Actions runner — accessible anywhere through Tailscale, with VS Code, Node, Python, Docker and more pre-installed automatically.

The idea is simple: trigger the workflow, wait a few minutes while everything sets up, then RDP in and get to work. No cloud bills, no VPS to manage. Just a free GitHub runner doing its thing.

> ⏳ **Sessions last up to 6 hours** — that's GitHub's hard limit on free runners. Plan accordingly.

---

## How it works

When you trigger the workflow manually, it:

1. Clears out the junk GitHub pre-installs (Android Studio, Miniconda, Haskell, etc.) to free up disk space
2. Configures Windows Remote Desktop and opens port 3389
3. Creates a local Windows user called `RDP` using your chosen password from secrets
4. Installs and connects Tailscale so you can reach the runner from anywhere
5. Installs Node.js, Python, Docker, GitHub CLI, and Bun
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

1. Go to **Actions** tab in your repo
2. Select **RDP** from the left sidebar
3. Click **Run workflow**
4. Wait roughly 5–8 minutes for setup to finish
5. Open the workflow logs and look for the `Maintain Connection` step — it prints the Tailscale IP, username, and password
6. Open your RDP client, connect to that IP, log in with `RDP` and your password

On Windows you can use the built-in Remote Desktop Connection app. On Mac, grab [Microsoft Remote Desktop](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466) from the App Store. On Android/iOS the same app is available too.

---

## What gets removed

The runner comes with a lot of things pre-installed that most people don't need. These get deleted at the start to free up around 30–40 GB of disk space:

- Android Studio and Android SDK
- Miniconda / Anaconda
- Haskell (GHCup)
- Rust and Cargo
- Go
- Boost C++ libraries
- PHP
- Azure CLI
- Chocolatey package cache
- Windows SDK extras
- .NET SDK
- Java
- Microsoft SQL Server
- Microsoft Visual Studio (the full IDE, not VS Code)
- Selenium WebDrivers
- Temp files

After cleanup you should have roughly 40–50 GB free on C:.

---

## What gets installed

### Node.js
Installed via winget. These global packages come with it:
`yarn`, `pnpm`, `typescript`, `ts-node`, `nodemon`

### Python 3.12
Installed via winget. These pip packages come with it:
`requests`, `numpy`, `pandas`, `flask`, `fastapi`, `uvicorn`, `django`, `python-dotenv`, `httpx`, `pytest`, `black`, `pylint`

### Docker Desktop
Installed but needs to be started manually once you're in the RDP session. Docker requires either WSL2 or Hyper-V — if it doesn't start, check the Docker Desktop UI for setup prompts.

### CLI Tools
- **GitHub CLI** (`gh`) — for working with GitHub from the terminal
- **Wget** — for downloading files
- **Bun** — fast JavaScript runtime and package manager

### VS Code
Installed silently. These extensions come pre-installed:

**GitHub**
- GitHub Copilot
- GitHub Copilot Chat
- GitHub Actions
- GitHub Pull Requests

**Python**
- Python, Pylance, Debugpy, Black Formatter
- Jupyter, Jupyter Keymap, Jupyter Renderers

**HTML / CSS / JS**
- Auto Rename Tag, Auto Close Tag
- Code Runner
- ESLint
- Prettier
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

**Docker might need manual setup.** On first launch inside RDP, Docker Desktop may ask you to enable WSL2 or Hyper-V. Just follow the prompts.

**Nothing persists between runs.** Every time you trigger the workflow, you get a fresh runner. Anything you create or download during the session is gone when it ends. If you need files to persist, push them to Git or upload them somewhere before the session ends.

**The password needs to meet Windows complexity rules.** If `RDP_PASSWORD` doesn't have uppercase + lowercase + number + special character, the user creation step will fail and the whole workflow stops there.

---

## Stopping the session

Cancel the workflow run from the Actions tab. That's it — the runner shuts down and Tailscale removes the ephemeral node automatically if you used an ephemeral auth key.

---

## File structure

```
FreeRDP/
└── .github/
    └── workflows/
        └── main.yml    ← the whole thing lives here
```

That's intentionally minimal. The workflow is self-contained — one file does all the setup, connection, and cleanup.

---

## Contributing

Found a bug, have a better cleanup path, or want to add something useful? Pull requests are open.

A few things that would be good additions if anyone wants to take a shot:

- Auto-detect and clean up more pre-installed runner bloat
- Optional step to auto-clone a specific repo into the session on startup
- A way to push files back to GitHub or somewere automatically before the session ends
- PostgreSQL / MongoDB install as optional steps

To contribute, fork the repo, make your changes in a branch, and open a PR against `main`. Keep changes focused — one thing per PR is easier to review.

---

## Known issues

**winget sometimes needs a moment.** On rare occasions winget on GitHub runners is slow to initialize and a package install fails on the first attempt. If a run fails at an install step, just re-trigger it — it usually works on the second try.

**VS Code extensions install under the runner user.** When you RDP in as the `RDP` user, VS Code may not show the extensions immediately. Open VS Code once and they should sync up. If not, running `code --list-extensions` in the terminal will confirm they're installed.

**Tailscale auth key must be valid.** If the key is expired or revoked, the connection step will fail and the workflow will exit early. Always double-check the key expiry in your Tailscale admin panel before a run.

---

## Credits

Built and maintained by [@maya920](https://github.com/maya920).

Uses [Tailscale](https://tailscale.com) for secure networking and [GitHub Actions](https://github.com/features/actions) free runners for compute. Both are free at the tier this project uses.