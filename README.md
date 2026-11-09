# GitHub Actions Ubuntu XFCE RDP

This repository template starts a temporary Ubuntu 24.04 desktop on a GitHub-hosted Actions runner. Connect to it from Windows using **Remote Desktop Connection (`mstsc`)** over a private **Tailscale** network.

## Files

```text
.
├── .github/
│   └── workflows/
│       ├── rdp.yml
│       └── rdp-stop.yml
├── .gitignore
└── README.md
```

## 1. Add the GitHub repository secrets

Open **Repository → Settings → Secrets and variables → Actions → New repository secret**.

Add both:

| Secret | Value |
|---|---|
| `TS_AUTHKEY` | A Tailscale authentication key |
| `RDP_PASSWORD` | A strong password for the Linux account `runner` |

Create a Tailscale auth key at: https://login.tailscale.com/admin/settings/keys

For repeated manual starts, configure the key according to your Tailscale device-management policy. An ephemeral key is suitable for temporary runner devices. Keep the key and RDP password private; never commit them to the repository.

## 2. Start the desktop

1. Push these files to your GitHub repository's default branch.
2. Open the repository's **Actions** tab.
3. Select **Start Linux RDP**.
4. Click **Run workflow** and confirm.
5. Wait for the job's **Show connection information** step.
6. Open the job summary or logs and copy the Tailscale IPv4 address.

The workflow installs XFCE and XRDP, joins the runner to your Tailscale network, and remains active.

## 3. Connect from Windows

1. Install Tailscale for Windows: https://tailscale.com/download/windows
2. Sign in to the same Tailscale network used by the auth key.
3. Press `Win + R`, type `mstsc`, and press Enter.
4. Enter the Tailscale IP shown in the GitHub Actions run summary.
5. Sign in with:

   - **Username:** `runner`
   - **Password:** the value of your `RDP_PASSWORD` repository secret

If Windows displays a certificate warning for the temporary Linux host, verify that you are connecting to the Tailscale IP from your current workflow run before proceeding.

## 4. Stop the desktop

1. Return to the repository's **Actions** tab.
2. Select **Stop Linux RDP**.
3. Click **Run workflow**.

This separate workflow requests cancellation of active runs of `Start Linux RDP`. GitHub then terminates the runner job and its temporary machine.

## Important limitations and security

- This is a **temporary GitHub-hosted runner**, not a permanent VPS.
- The desktop exists only while the start workflow is running. It is also stopped automatically when the job reaches its timeout.
- The workflow uses a 350-minute timeout to stay below the usual six-hour GitHub-hosted job limit. Availability and limits depend on your GitHub plan and repository.
- Both Windows and the runner must be connected to the same Tailscale network.
- RDP is not exposed as a public internet URL. Connect using the private Tailscale IP.
- Do not open port 3389 to the public internet.
- GitHub Actions minutes and runner availability are subject to GitHub's current plan limits and policies.
- The stop workflow requires Actions write permission. If repository or organization policies restrict workflow permissions, an administrator may need to allow the workflow to cancel runs.
- Do not use this setup to bypass GitHub usage limits or run workloads prohibited by GitHub's terms.

## Troubleshooting

### `TS_AUTHKEY` is missing or authentication fails
Check the repository secret name and value. Make sure the key is valid and allowed by your Tailscale policy.

### The Tailscale IP does not respond
Confirm that Tailscale is running and signed in on Windows, and that the Windows device is in the same tailnet. Check that the start workflow is still running.

### RDP connects but shows a blank screen or disconnects
Inspect the `Configure RDP account and XFCE session` and `Start XRDP service` steps in the Actions log. Confirm that the XFCE packages installed successfully.

### Stop workflow cannot cancel the start run
Check that Actions permissions allow the workflow token to cancel workflow runs. The stop workflow needs `actions: write`.

---

**No public RDP URL is created.** The address is a private Tailscale IP, intended for devices authorized in your tailnet.
