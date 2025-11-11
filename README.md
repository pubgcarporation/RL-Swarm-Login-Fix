# Gensyn RL-Swarm Node - Login Bypass Setup

## Overview
This guide helps you bypass the localhost tunnel login requirement for Gensyn RL-swarm nodes by persisting authentication data and auto-restoring it on each run.

## ⚠️ Important Prerequisites
- **Your RL-swarm node MUST be running** before applying these fixes
- You should have already logged in at least once via the localhost tunnel
- Keep the node running in your terminal during setup

---

## Step 1: Create Persistent Data Directory & Copy Auth Files

Create a directory to store authentication data permanently and copy existing login credentials:

```bash
mkdir -p ~/rl-swarm/persist-data
```

```bash
cp ~/rl-swarm/modal-login/temp-data/userData.json ~/rl-swarm/persist-data/
cp ~/rl-swarm/modal-login/temp-data/userApiKey.json ~/rl-swarm/persist-data/
chmod 600 ~/rl-swarm/persist-data/userData.json
chmod 600 ~/rl-swarm/persist-data/userApiKey.json
```

---

## Step 2: Create Symbolic Links

Remove temporary files and create symbolic links to the persistent data:

```bash
rm -f ~/rl-swarm/modal-login/temp-data/userData.json
rm -f ~/rl-swarm/modal-login/temp-data/userApiKey.json
ln -s ~/rl-swarm/persist-data/userData.json ~/rl-swarm/modal-login/temp-data/userData.json
ln -s ~/rl-swarm/persist-data/userApiKey.json ~/rl-swarm/modal-login/temp-data/userApiKey.json
```

---

## Step 3: Edit Trainer Config File

Open the run script for editing:

```bash
nano ~/rl-swarm/run_rl_swarm.sh
```

### Add Auto-Login Restore Code

Locate the line `set -euo pipefail` and **paste the following code directly below it**:

```bash
# === Auto Login Restore ===
rm -rf ~/rl-swarm/modal-login/temp-data
mkdir -p ~/rl-swarm/modal-login/temp-data
ln -sf ~/rl-swarm/persist-data/userData.json ~/rl-swarm/modal-login/temp-data/userData.json
ln -sf ~/rl-swarm/persist-data/userApiKey.json ~/rl-swarm/modal-login/temp-data/userApiKey.json

rm -rf ~/rl-swarm/web/.modal-data
mkdir -p ~/rl-swarm/web/.modal-data
ln -sf ~/rl-swarm/persist-data/userData.json ~/rl-swarm/web/.modal-data/userData.json
ln -sf ~/rl-swarm/persist-data/userApiKey.json ~/rl-swarm/web/.modal-data/userApiKey.json
# ==========================
```

### Save and Exit

1. Press `Ctrl + O` to save
2. Press `Enter` to confirm
3. Press `Ctrl + X` to exit

---

## Expected File Structure

Your `~/rl-swarm/run_rl_swarm.sh` should now look like this:

```bash
#!/usr/bin/env bash
set -euo pipefail

# === Auto Login Restore ===
rm -rf ~/rl-swarm/modal-login/temp-data
mkdir -p ~/rl-swarm/modal-login/temp-data
ln -sf ~/rl-swarm/persist-data/userData.json ~/rl-swarm/modal-login/temp-data/userData.json
ln -sf ~/rl-swarm/persist-data/userApiKey.json ~/rl-swarm/modal-login/temp-data/userApiKey.json

rm -rf ~/rl-swarm/web/.modal-data
mkdir -p ~/rl-swarm/web/.modal-data
ln -sf ~/rl-swarm/persist-data/userData.json ~/rl-swarm/web/.modal-data/userData.json
ln -sf ~/rl-swarm/persist-data/userApiKey.json ~/rl-swarm/web/.modal-data/userApiKey.json
# ==========================

# General arguments
ROOT=$PWD

...
```

---

## ✅ Testing

Everything is now set! Restart your RL-swarm node and you should **no longer need to login via localhost tunnel**.

The authentication will be automatically restored from the persistent data on each run.

---

## Troubleshooting

- **"File not found" errors**: Make sure your node was running and you had logged in at least once before running these commands
- **Permission denied**: Ensure the `chmod 600` commands were executed successfully
- **Still asking for login**: Verify the symbolic links were created correctly with `ls -la ~/rl-swarm/modal-login/temp-data/`

---

## Notes

- This bypass works by persisting authentication tokens and restoring them automatically
- The `chmod 600` commands ensure your auth files are only readable by you
- Symbolic links allow the system to reference the persistent data without duplication
