# Naukri Profile Refresh Automation

Keeps your Naukri profile marked as **"Recently Updated"** so recruiters see your profile at the top of search results.

Every run, the script toggles a trailing period (`.`) at the end of your **Profile Summary** or **Resume Headline**, which Naukri registers as an active profile update.

---

## Key Features

- **Dual Layout Support**: Supports both **Fresher / Naukri Campus** (`Profile Summary`) and **Experienced** (`Resume Headline`) profiles.
- **Anti-Bot & Humanized Delays**: Includes random startup jitter (5–25 seconds) and human-like action pauses to avoid triggering bot detection.
- **Persistent Local Session**: Logs in once using Google Sign-In and saves your browser cookies locally in `.naukri-chrome-profile/`. You don't need to re-login every time.
- **Server Verification**: Reloads and re-reads your profile from Naukri's server to confirm the change actually stuck before logging success.
- **Privacy & Security**: 100% local execution. All personal data lives in `.env`. No credentials or telemetry are ever sent to external servers.

---

## Requirements

- **Windows 10/11** (uses Windows Task Scheduler for background execution)
- **Node.js 18+**
- **Google Chrome** installed
- A Naukri account linked with Google Sign-In

---

## Setup & Quickstart

### 1. Clone & Install Dependencies
```powershell
git clone https://github.com/ankitbaghel01/naukri_update.git
cd naukri_update
npm install
```

### 2. Configure `.env`
Copy `.env.example` to `.env`:
```powershell
copy .env.example .env
```

Open `.env` and fill in your email details:
```env
GOOGLE_EMAIL=your_email@gmail.com
GOOGLE_PASSWORD=your_password  # (Optional: used for automatic sign-in if session expires)
NAUKRI_PROFILE_URL=https://www.naukri.com/mnjuser/profile
```
> **Note**: `.env` is listed in `.gitignore`, so your credentials will never be committed to Git.

### 3. First Login (One-Time Setup)
Run the script in login mode to sign in through a visible browser window:
```powershell
node naukri-profile-refresh.js login
```
- A Chrome window will open. Complete the Google Sign-In (and approve 2-Step Verification if prompted).
- Once logged in, close the window. The session will be saved locally to `.naukri-chrome-profile/`.

### 4. Test a Background Run
```powershell
node naukri-profile-refresh.js
```
Check `naukri-refresh.log` to confirm success:
```log
[18/8/2026, 11:17:25 am] OK: profile summary/headline dot removed (verified) → "Computer Science student..."
```

---

## Automated Background Schedule (Task Scheduler)

To keep your profile safe under Naukri's bot detection guidelines, it is recommended to run the script every **3 to 4 hours**.

Run this command once in PowerShell:

```powershell
$repo = "D:\code\Naukari_automation\naukri_update"
$action  = New-ScheduledTaskAction -Execute "node.exe" -Argument "`"$repo\naukri-profile-refresh.js`"" -WorkingDirectory $repo
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 4)
Register-ScheduledTask -TaskName "NaukriProfileRefresh" -Action $action -Trigger $trigger -Settings (New-ScheduledTaskSettingsSet -StartWhenAvailable) -Force
```

### Useful Management Commands:
```powershell
Get-ScheduledTask NaukriProfileRefresh            # Check task status
Start-ScheduledTask NaukriProfileRefresh          # Run task immediately
Disable-ScheduledTask NaukriProfileRefresh        # Pause task
Enable-ScheduledTask NaukriProfileRefresh         # Resume task
Unregister-ScheduledTask NaukriProfileRefresh     # Remove task
```

---

## How to Switch to a Different Account / ID

To change the Naukri / Google account being automated:

1. **Delete the saved Chrome profile folder**:
   ```powershell
   Remove-Item -Recurse -Force .naukri-chrome-profile
   ```
2. **Update `GOOGLE_EMAIL` in `.env`** with your new account email.
3. **Run the one-time login again**:
   ```powershell
   node naukri-profile-refresh.js login
   ```

---

## Troubleshooting

| Symptom | Cause & Solution |
|---|---|
| `Enable-ScheduledTask: The system cannot find the file specified` | The task wasn't created yet or was deleted. Re-run the `Register-ScheduledTask` command above. |
| `Google login did not complete` | Run `node naukri-profile-refresh.js login` in visible mode and approve Google 2-Step Verification manually once. |
| `save did not stick` | Naukri changed its DOM layout. Check error screenshots in the repository folder (`naukri-refresh-error-*.png`). |
| Want to start fresh | Delete the `.naukri-chrome-profile/` folder and re-run the login command. |

---

## File Overview

| File | Description |
|---|---|
| `naukri-profile-refresh.js` | Main automation script (Playwright) |
| `config.js` | Zero-dependency `.env` configuration loader |
| `.env.example` | Template for environment variables |
| `naukri-refresh.log` | Local execution log file |
| `.naukri-chrome-profile/` | Directory containing saved Chrome session cookies |

---

## Disclaimer

Automating profile activity may be against Naukri's official Terms of Service. This tool is designed for personal convenience, executing at human-like intervals. Use at your own risk.
