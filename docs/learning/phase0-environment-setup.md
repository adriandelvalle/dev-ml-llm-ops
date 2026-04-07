# Phase 0: Environment Setup

## Objective
Establish a secure, reproducible foundation for DevOps/MLOps/LLMOps learning on local hardware.

## Hardware & Base OS
| Component | Specification |
|-----------|--------------|
| Device | ACEMAGIC Mini PC |
| CPU | AMD Ryzen 7 6800H (8 cores / 16 threads) |
| GPU | AMD Radeon 680M (iGPU, shared VRAM) |
| RAM | 32 GB DDR5 (shared with GPU) |
| Storage | NVMe SSD |
| OS | Ubuntu 24.04 LTS (minimal install) |

## Setup Steps

### 1. Ubuntu Post-Install Basics
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget git build-essential htop tmux net-tools

# Configure timezone & locale
sudo timedatectl set-timezone Europe/Madrid
sudo localectl set-locale LANG=en_US.UTF-8

# Create non-root user with sudo (if not done during install)
sudo adduser jota
sudo usermod -aG sudo jota
``` 

---

### 2. SSH Hardening
```bash
# Generate SSH key on local machine (NOT on server)
ssh-keygen -t ed25519 -C "adriandelvallecv@gmail.com"

# Copy public key to server
ssh-copy-id jota@192.168.0.21

# On server: disable password auth, enforce key-only
sudo nano /etc/ssh/sshd_config
# Ensure these lines:

#   PasswordAuthentication no
#   PubkeyAuthentication yes
#   PermitRootLogin no

# Restart SSH service
sudo systemctl restart sshd
```

---

### 3. Git Configuration
```bash
# Global config
git config --global user.name "jota"
git config --global user.email "adriandelvallecv@gmail.com"
git config --global init.defaultBranch main
git config --global pull.rebase true

# Conventional Commits template (optional)
git config --global commit.template ~/.gitmessage
```

---

### 4. VS Code Remote SSH Workflow
- Install "Remote - SSH" extension on local VS Code.
- Add host to ~/.ssh/config (local machine):

```
Host acemagic
    HostName 192.168.0.21
    User jota
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes
```
- Connect via command palette: ```Remote-SSH: Connect to Host... → acemagic.```
- Port forwarding: VS Code auto-forwards ports used by processes (e.g., 8000). ```Manual: Ports tab → Forward Port → 8000.```


---

### 5. Basic Security Baseline
```bash
# Enable UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 8000/tcp  # FastAPI dev
sudo ufw enable

# Install fail2ban
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban

# Audit script (custom)
# See: brewery-app/scripts/audit-permissions.sh
```

### ✅ Execution & Verification Log
| Date       | Action                  | Status   | Verified By          |
|------------|-------------------------|----------|----------------------|
| 2026-04-07 | UFW + fail2ban applied  | ✅ Live  | `jota@jotasrv`       |
| 2026-04-07 | Firewall & jail rules   | ✅ Active| `ufw status` + `systemctl is-active` |


---

## Verification Checklist
- ssh jota@192.168.0.21 connects without password
- git commit uses correct name/email
- VS Code Remote SSH shows server filesystem
- sudo ufw status shows firewall active
- htop shows system resources correctly


---

## Troubleshooting
| Issue | Solution |
|:--- |:---|
| **SSH key rejected** | Check permissions: `chmod 700 ~/.ssh, chmod 600 ~/.ssh/authorized_keys` |
| **Port forwarding not working** | VS Code → Ports tab → right-click → "Change Local Port" or restart tunnel |
| **Git push asks for password** | Ensure SSH key is loaded: `ssh-add -l` or use `git remote set-url origin git@github.com:...` |

