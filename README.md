# FreeRDP

**Repo:** [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP)

Run a full Windows desktop session from a GitHub Actions runner — accessible anywhere through Tailscale, with VS Code, Node, Python, Windows Terminal and more pre-installed automatically.

The idea is simple: trigger the workflow, wait a moment while Tailscale connects, then RDP in. Everything else — VS Code, extensions, tools — installs in the background while you work. No cloud bills, no VPS to manage.

> ⏳ **Sessions last up to 6 hours** — GitHub's hard limit on free runners. Plan accordingly.

---

## How it works

When you trigger the workflow manually, it:

1. Starts cleaning out pre-installed bloat in the background (Android, Rust, Go, Visual Studio, etc.)
2. Configures Windows Remote Desktop and opens port 3389
3. Creates a local Windows user called `RDP`
4. Enables clipboard history (Win+V multiple items), sets timezone to India Standard Time, and switches to High Performance power scheme
5. Installs and connects Tailscale so you can reach the runner from anywhere
6. Configures Git with your name and email
7. **All major installs run in background:** VS Code + extensions, Node.js, Python, CLI tools, and Windows Terminal
8. Sits in a loop showing completion status — connect within ~60 seconds while installs finish behind you

The connection details (Tailscale IP, username, password) appear in the very first lines of the `Maintain Connection` step — you don't need to wait for installs to finish.

---

## Use This Project

The fastest way to get started is to fork the repo directly from GitHub:

👉 [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP) → click **Fork**

That copies everything into your own account. Then just add your secrets and run it. No cloning needed.

If you'd rather start fresh in an existing repo, grab `.github/workflows/main.yml` and drop it in your own repo at the same path.

---

## Setup

### 1. Fork or copy this repo

Fork from [github.com/maya920/FreeRDP](https://github.com/maya920/FreeRDP) or add `.github/workflows/main.yml` to your own repository.

### 2. Add your secrets

Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Required | Description |
|--------|----------|-------------|
| `RDP_PASSWORD` | ✅ | Password for the `RDP` user. Must meet Windows complexity rules — 8+ chars with upper, lower, digit, and special character |
| `TAILSCALE_AUTH_KEY` | ✅ | Auth key from your [Tailscale admin panel](https://login.tailscale.com/admin/settings/keys). Use an ephemeral key |
| `GIT_USERNAME` | Optional | Your Git display name |
| `GIT_EMAIL` | Optional | Your Git email address |
| `GH_PAT` | Optional | GitHub PAT with `read:org`/`read:user` scope — shows remaining Actions billing minutes |

If `GIT_USERNAME` or `GIT_EMAIL` are missing, Git config is skipped. The workflow handles all missing secrets gracefully.

### 3. Install Tailscale on your device

Download Tailscale on whatever device you'll RDP from — [tailscale.com/download](https://tailscale.com/download). Log in with the same account you used to generate the auth key.

---

## Running it

1. Go to the **Actions** tab in your repo
2. Select **RDP** from the left sidebar
3. Click **Run workflow**
4. Optionally toggle optional installs:
   - `install_vscode` — install VS Code in background
   - `install_extensions` — install VS Code extensions
   - `install_node` — install Node.js + npm global packages
   - `install_cli` — install GitHub CLI, Wget, Bun
   - `install_python` — install Python pip packages
5. Open the workflow logs — the `Maintain Connection` step prints the Tailscale IP, username, and password immediately
6. Open your RDP client, connect to that IP, and log in with `RDP` and your password
7. VS Code, extensions, and tools continue installing in the background — you can start working right away

On Windows, use Remote Desktop Connection (`mstsc`). On Mac, use [Microsoft Remote Desktop](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466). Available on Android and iOS too.

---

## What gets removed

GitHub runners come loaded with ~40 GB of pre-installed tools. The workflow removes all of it upfront in the background. The full list:

**Development tools & runtimes:** Android SDK, Miniconda, Haskell (GHCup), Rust/Cargo, Go, PHP, Java, R/RTools, Strawberry Perl, .NET SDK, Boost C++, LLVM/Clang, CMake

**Microsoft tools:** Visual Studio (full IDE), SQL Server, Windows Kits, Microsoft SDKs, Azure CLI, Azure Cosmos Emulator, Service Fabric

**Other:** AWS tools, Unity Hub, Firefox, Selenium, Chocolatey cache, temp files

After cleanup: ~40-50 GB free on `C:`.

---

## What gets installed

### VS Code

Installed with `--scope machine` so the `RDP` user has it. Desktop shortcut and taskbar pin (for both `runneradmin` and `RDP` users). VS Code settings pre-configured:
- Font size: **16**
- Auto save: **afterDelay** (1s)
- Compact folders in explorer: **off**

| Category | Extensions |
|----------|-----------|
| GitHub | Copilot, GitHub Actions, Pull Requests |
| Python/AI | Python, Pylance, Black, Jupyter, Google Colab, OpenCode |
| HTML/CSS/JS | Auto Rename Tag, Auto Close Tag, Code Runner, ESLint, Prettier, CSS Snippets, CSS Peek |
| Tailwind/React | Tailwind Fold, Tailwind CSS IntelliSense, ES7+ React/Redux Snippets, **Turbo Console Log** |
| IntelliSense | Path IntelliSense, npm IntelliSense, Quokka.js, Wallaby.js, **Color Highlight** |
| Productivity | Material Icon Theme, GitLens, Live Share, Edit CSV, DotENV, REST Client, Thunder Client |
| Full Stack | Prisma, Docker, Live Server, Code Spell Checker |

### Node.js

Installed via winget. Global packages: `yarn`, `pnpm`, `typescript`, `ts-node`, `nodemon`

### Python

Uses the pre-installed Python. Pip packages: `requests`, `numpy`, `pandas`, `flask`, `fastapi`, `uvicorn`, `django`, `python-dotenv`, `httpx`, `pytest`, `black`, `pylint`

### CLI Tools

- **GitHub CLI** (`gh`)
- **Wget**
- **Bun** — fast JS runtime & package manager

### Windows Terminal

Installed with desktop shortcut for the `RDP` user and pinned to the taskbar.

---

## Background installs & notifications

All installs run in parallel as background PowerShell processes. The `Maintain Connection` step shows completion status as each component finishes:

```
  ▶ [14:02:30] INSTALL_STARTED: VS Code
  ✅ [14:05:10] INSTALL_DONE: VS Code
  ✅ [14:06:45] INSTALL_DONE: CLI Tools
```

When each batch completes, a Windows popup notification fires (except individual extensions — they finish too fast to notify per-extension).

You can RDP in within ~60 seconds of the workflow starting. Everything continues installing while you work.

---

## Secure File Sharing

The workflow automatically sets up **two secure ways** to transfer files between your RDP machine and your phone, PC, or other devices — all through Tailscale.

### SMB (Network File Share)

- **Share folder:** `C:\TailscaleShare`
- **Access from your PC/phone:** `\\<TAILSCALE_IP>\TailscaleShare`
- **User:** `RDP`
- **Password:** your RDP password
- **Permission:** full read/write

Perfect for Windows PCs and any device with SMB support. The share is created automatically during workflow setup.

**Mobile apps that support SMB:**
- Android: `Solid Explorer`, `ES File Explorer`, `Total Commander`
- iOS: `FileMerge` or other SMB clients

### SFTP (SSH File Transfer Protocol)

For devices where SMB doesn't work easily (especially phones), use SFTP:

- **Host:** `<TAILSCALE_IP>` (same Tailscale IP as RDP)
- **Port:** `22`
- **User:** `RDP`
- **Password:** your RDP password
- **Home folder symlink:** `C:\Users\RDP\TailscaleShare` → points to the shared folder for convenience

**Mobile apps that support SFTP:**
- Android: `Termius`, `Solid Explorer`, `AndFTP`
- iOS: `Prompt 3`, `Termius`, `iSH Shell`
- Desktop: `WinSCP`, `FileZilla`, `Cyberduck`

The OpenSSH server is installed and configured automatically during workflow setup.

### How it works

1. Both services are configured during the workflow run
2. Firewall rules open port `445` (SMB) and port `22` (SSH)
3. All communication is encrypted and isolated to your Tailscale network
4. No file leaves your secure private network
5. Use your RDP user credentials for both

### Access priority

- **PC → PC:** Use SMB (faster, simpler)
- **Phone → Machine:** Try SMB first; fall back to SFTP if needed
- **All platforms:** SFTP always works as a fallback

---

## Session warnings

The maintain loop prints warnings at key points and sends a Windows popup:

| Time | Warning |
|------|---------|
| **5h30m elapsed** | ⚠ Session expires in 30 minutes — push your work |
| **5h50m elapsed** | ⚠ Final warning — ~10 minutes remaining |

---

## Private repo billing

If this workflow runs on a **private repo**, GitHub Actions charges against the 2000 min/month free tier. Each 6-hour session uses ~360 minutes.

To track remaining minutes, add a `GH_PAT` secret (classic PAT with `read:org` or `read:user` scope). The maintain loop checks the billing API once and prints:

```
📊 Billing (private): 450/2000m used (22.5%) — 1550m remaining
```

Without the PAT, a friendly reminder prints instead.

---

## Secrets summary

| Secret | Required | Description |
|--------|----------|-------------|
| `RDP_PASSWORD` | ✅ | Windows password for the `RDP` user |
| `TAILSCALE_AUTH_KEY` | ✅ | Tailscale auth key (ephemeral recommended) |
| `GIT_USERNAME` | Optional | Git global user.name |
| `GIT_EMAIL` | Optional | Git global user.email |
| `GH_PAT` | Optional | GitHub PAT for billing API (shows remaining minutes) |

---

## File structure

```
FreeRDP/
└── .github/
    └── workflows/
        └── main.yml    ← the whole thing (one file, ~840 lines)
```

Everything lives in one file. That's intentional — fork, add secrets, run.

---

## Contributing

Pull requests are open. Things worth adding:

- Auto-clone a specific repo on startup
- PostgreSQL / MongoDB / Redis as optional installs
- Auto-push files to GitHub before session ends
- Support for self-hosted runners (no 6-hour limit)

To contribute, fork the repo, make changes in a branch, and open a PR against `main`.

---

## Known issues

**winget sometimes needs a moment.** On rare occasions winget on GitHub runners is slow to initialize and a package fails on the first attempt. If a run fails at an install step, just re-trigger it.

**Tailscale auth key must be valid.** Expired or revoked keys cause the connection step to fail. Check key expiry before a run.

**VS Code extensions install under the runner user.** They're copied to the RDP user's profile automatically. If they don't show immediately, open VS Code once and they sync.

**Only one RDP session at a time.** Windows Server 2025 on GitHub runners only allows one interactive RDP session. If you're already connected and someone else tries to connect, you'll be disconnected.

---

## Things to know

**The 6-hour limit is real.** GitHub free runners stop after 360 minutes. `timeout-minutes: 365` means the workflow outlasts the runner.

**Nothing persists between runs.** Every trigger gives you a fresh VM. Push work to GitHub before the session ends.

**Private repos use your Actions quota.** Each 6-hour session burns ~360 minutes from your 2000 min/month free tier.

---

## Credits

Built and maintained by [@maya920](https://github.com/maya920).

Uses [Tailscale](https://tailscale.com) for secure networking and [GitHub Actions](https://github.com/features/actions) free runners.
