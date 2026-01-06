# CLI Tips and Tricks

A collection of useful command-line one-liners, scripts, and tips for Linux/Unix systems administration and development.

## Contents

- [Benchmarking](#benchmarking)
- [User Management](#user-management)
- [System Configuration](#system-configuration)
- [Date and Time](#date-and-time)
- [Pastebin Services](#pastebin-services)
- [Docker](#docker)
- [AWK](#awk)
- [Find](#find)
- [Sed](#sed)
- [Network Tools](#network-tools)
- [SSH](#ssh)
- [Bash Loops](#bash-loops)
- [Archive Handling](#archive-handling)
- [Vim/Neovim Plugin Management](#vimneovim-plugin-management)
- [Cat with Heredoc](#cat-with-heredoc)
- [Kubernetes](#kubernetes)
- [Git](#git)
- [JSON/YAML Processing](#jsonyaml-processing)
- [System Monitoring](#system-monitoring)
- [Additional Resources](#additional-resources)
- [GitHub Repository Automation](#github-repository-automation)

---

## Benchmarking

Quick server benchmarking scripts.

**Method 1 (wget):**
```bash
wget -qO- https://bench.sh | bash
```

**Method 2 (curl):**
```bash
curl -Lso- https://bench.sh | bash
```

**Alternative - YABS (Yet Another Bench Script):**
```bash
curl -sL yabs.sh | bash
```

> **⚠️ Security Warning:** Always review scripts before piping to bash. Download and inspect first:
> ```bash
> curl -L https://bench.sh -o bench.sh
> less bench.sh  # Review the script
> bash bench.sh  # Run after review
> ```

---

## User Management

### Allow superuser access without password

**Option 1: Add to main sudoers file**
```bash
echo "$USER ALL=(ALL) NOPASSWD: ALL" | sudo tee -a /etc/sudoers
```

**Option 2: Create user-specific sudoers file (recommended)**

For Debian:
```bash
echo 'debian ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/debian
```

For Ubuntu:
```bash
echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/ubuntu
```

For CentOS/RHEL:
```bash
echo 'centos ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/centos
```

> **⚠️ Security Warning:** NOPASSWD is convenient but reduces security. Use only for development/testing environments.

### Colored Bash Prompts

Useful for distinguishing root from regular users.

**Regular user (green prompt):**
```bash
echo "PS1='\[\e[1;32m\][\u@\h \W]\$\[\e[0m\] '" >> ~/.bashrc
```

**Root user (red prompt):**
```bash
echo "PS1='\[\e[1;31m\][\u@\h \W]\$\[\e[0m\] '" >> ~/.bashrc
```

Apply changes:
```bash
source ~/.bashrc
```

---

## System Configuration

### Dropbox inotify fix

Increase the inotify watch limit:

```bash
echo fs.inotify.max_user_watches=100000 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## Date and Time

Useful date formatting for timestamps.

**Basic date format (YYYYMMDD):**
```bash
date +"%Y%m%d"
```
Example output: `20131016`

**ISO date format (YYYY-MM-DD):**
```bash
date +"%Y-%m-%d"
```
Example output: `2013-10-16`

**Full timestamp (YYYYMMDD_HHMMSS):**
```bash
date +"%Y%m%d_%H%M%S"
```
Example output: `20131016_193655`

**ISO 8601 format with timezone:**
```bash
date -Iseconds
```
Example output: `2024-10-16T19:36:55-07:00`

**Unix timestamp (seconds since epoch):**
```bash
date +%s
```
Example output: `1697500615`

---

## Pastebin Services

### Termbin

Quick text sharing via netcat.

Pipe any output to termbin:
```bash
ls -la | nc termbin.com 9999
```

Alternative with timeout:
```bash
echo "Hello World" | timeout 5 nc termbin.com 9999
```

Website: https://termbin.com/

### Paste.rs

Modern Rust-based pastebin service.

Pipe to paste.rs:
```bash
echo "Hello World" | curl --data-binary @- https://paste.rs
```

From file:
```bash
curl --data-binary @file.txt https://paste.rs
```

Website: https://paste.rs

### ix.io

Simple pastebin via curl.

Pipe any output:
```bash
ls -la | curl -F 'f:1=<-' ix.io
```

From file:
```bash
curl -F 'f:1=<-' ix.io < file.txt
```

---

## Docker

### Installation

**Modern method (recommended):**

Ubuntu/Debian:
```bash
sudo apt-get update
sudo apt-get install docker.io docker-compose
```

RHEL/CentOS/Fedora:
```bash
sudo dnf install docker docker-compose
```

Arch Linux:
```bash
sudo pacman -S docker docker-compose
```

**Quick install script (for supported distros):**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

> **⚠️ Security Warning:** Always download and review scripts before execution. Never pipe directly to shell without inspection.

### Allow non-root usage

Add current user to docker group:
```bash
sudo usermod -aG docker $USER
```

Add specific user (e.g., ubuntu):
```bash
sudo usermod -aG docker ubuntu
```

Apply group changes (or logout/login):
```bash
newgrp docker
```

### Container IP addresses

**Get IP of specific container (by ID):**
```bash
docker inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID}
```

**Get IP of specific container (by name):**
```bash
docker inspect --format '{{ .NetworkSettings.IPAddress }}' container_name
```

**Get IPs of all running containers:**
```bash
docker inspect --format '{{ .Name }} - {{ .NetworkSettings.IPAddress }}' $(docker ps -q)
```

**Using jq for JSON parsing:**
```bash
docker inspect container_name | jq -r '.[0].NetworkSettings.IPAddress'
```

---

## AWK

Powerful text processing examples.

**Print the first statement on first line:**
```bash
cat file | awk 'NR==1{print $1}'
```

**Print assigned IP addresses on network interface:**
```bash
arp -n | grep eth0 | awk '{print $1}'
```

**Print all columns:**
```bash
awk '{print $0}' file
```

**Print the 3rd column:**
```bash
awk '{print $3}' file
```

**Print the 1st and 3rd columns:**
```bash
awk '{print $1 $3}' file
```

**Print all columns except the 3rd:**
```bash
awk '{$3=""; print $0}' file
```

**Print all columns except the 1st and 2nd:**
```bash
awk '{$1=$2=""; print $0}' file
```

---

## Find

### Change permissions

**For directories:**
```bash
find . -type d -exec chmod 755 {} +
```

**For files:**
```bash
find . -type f -exec chmod 644 {} +
```

**SSH directory example:**
```bash
find $HOME/.ssh/ -type f -exec chmod 600 {} +
```

### Change ownership

```bash
sudo find . -type d -user root -exec chown idv:idv {} \;
```

### Make scripts executable

```bash
find . -name "*.sh" -execdir chmod u+x {} +
```

### Find largest files

**Top 10 largest files/directories:**
```bash
du -hsx * | sort -rh | head -10
```

**Find largest files only (not directories):**
```bash
find . -type f -exec du -h {} + | sort -rh | head -10
```

**Find files larger than 100MB:**
```bash
find . -type f -size +100M -exec ls -lh {} \; | awk '{print $9 ": " $5}'
```

**Disk usage summary with ncdu (interactive):**
```bash
ncdu  # Install with: brew install ncdu or apt install ncdu
```

---

## Sed

### Common replacements

**Disable GRUB boot animation:**

Ubuntu/Debian:
```bash
sudo sed -i 's/quiet splash//' /etc/default/grub && sudo update-grub
```

RHEL/CentOS/Fedora:
```bash
sudo sed -i 's/quiet splash//' /etc/default/grub && sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

**Locale correction:**
```bash
sed -i 's/uk_UA.UTF-8/en_US.UTF-8/' /etc/default/locale
```

**Replace hostname in /etc/hosts:**
```bash
sed -i "s/ubuntu1404/$(hostname)/" /etc/hosts
```

### Text replacement in multiple files

**Method 1: Using grep and xargs**
```bash
grep -rl "windows" . | xargs sed -i "s/windows/linux/g"
```

**Method 2: Using find**
```bash
find /path -type f -exec sed -i 's/example.com/superexample.com/g' {} +
```

> **💡 Tip:** Use `sed -i.bak` to create backups before replacing.

---

## Network Tools

### ifconfig.me / ifconfig.co

**Get external IP:**
```bash
curl ifconfig.co
```

**Get all network information:**
```bash
curl ifconfig.co/all
```

**Alternative services:**
```bash
curl ifconfig.me
curl icanhazip.com
curl ipinfo.io/ip
curl api.ipify.org
curl checkip.amazonaws.com
```

**Get JSON formatted info:**
```bash
curl ipinfo.io
```

**IPv6 address:**
```bash
curl -6 ifconfig.co
curl -6 icanhazip.com
```

---

## SSH

### Fix authentication agent error

**Modern syntax (recommended):**
```bash
eval "$(ssh-agent -s)"
```

**Check if agent is running:**
```bash
ps aux | grep ssh-agent
echo $SSH_AUTH_SOCK
```

**Kill existing agent:**
```bash
killall ssh-agent
```

**Legacy syntax (deprecated):**
```bash
eval `ssh-agent`  # Use $() instead of backticks
```

### Add SSH keys to agent

**Add default key:**
```bash
ssh-add
```

**Add specific key:**
```bash
ssh-add ~/.ssh/id_ed25519
```

**Add all keys with passphrase timeout (8 hours):**
```bash
ssh-add -t 28800
```

**List loaded keys:**
```bash
ssh-add -l
```

**Remove all keys from agent:**
```bash
ssh-add -D
```

> **💡 Tip:** Use Ed25519 keys (ssh-keygen -t ed25519) for better security and performance compared to RSA.

### SSH Port Forwarding

**Local port forward syntax:**
```bash
ssh -L <local_port>:<destination_server>:<destination_port> user@<ssh_server> -p <ssh_port>
```

**Example: Access router's web interface**

If router's WAN IP is `12.23.34.45`, SSH port is `9999`, and web interface is on port `80`:
```bash
ssh -L 12345:localhost:80 root@12.23.34.45 -p 9999
```

Then access via: http://localhost:12345

**Multiple port forwards:**
```bash
ssh -L 8080:127.0.0.1:80 \
    -L 8443:127.0.0.1:443 \
    -L 8001:127.0.0.1:8001 \
    -L 5901:127.0.0.1:5901 \
    -L 1313:127.0.0.1:1313 \
    user@host
```

---

## Bash Loops

### Simple numeric range

```bash
for i in {1..10}; do command; done
```

### Numeric range with step

```bash
for i in {0..100..10}; do echo $i; done  # 0, 10, 20, ... 100
```

### Specific numbers

```bash
for i in {18,20,21}; do
    sleep 10
    ssh node-$i "tar zcvf - /var/log/ceilometer | cat > /tmp/ceilometer-node$i.tar.gz"
done
```

### Copy files from multiple nodes

```bash
for i in {18,20,21}; do
    scp node-$i:/tmp/ceilometer-node$i.tar.gz .
done
```

### Loop over files

```bash
for file in *.txt; do
    echo "Processing $file"
    # Your commands here
done
```

### Loop over command output

```bash
for pod in $(kubectl get pods -o name); do
    echo "Pod: $pod"
    kubectl describe $pod
done
```

### Parallel execution with xargs

```bash
# Run commands in parallel (4 jobs at a time)
echo {1..10} | xargs -n 1 -P 4 -I {} bash -c 'echo "Processing {}"; sleep 1'
```

---

## Archive Handling

Download and extract archives in one command.

### Using wget

**Uncompressed tar:**
```bash
wget http://example.com/archive.tar -O - | tar -x
```

**Gzipped tar:**
```bash
wget http://example.com/archive.tar.gz -O - | tar -xz
```

**Bzip2 tar:**
```bash
wget http://example.com/archive.tar.bz2 -O - | tar -xj
```

### Using curl

**Uncompressed tar:**
```bash
curl http://example.com/archive.tar | tar -x
```

**Gzipped tar:**
```bash
curl http://example.com/archive.tar.gz | tar -xz
```

**Bzip2 tar:**
```bash
curl http://example.com/archive.tar.bz2 | tar -xj
```

---

## Vim/Neovim Plugin Management

### Modern Plugin Managers (Recommended)

**vim-plug (most popular for Vim):**

Install vim-plug:
```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

Add to `~/.vimrc`:
```vim
call plug#begin()
Plug 'tpope/vim-sensible'
Plug 'junegunn/fzf'
Plug 'junegunn/fzf.vim'
call plug#end()
```

Install plugins:
```vim
:PlugInstall
```

**lazy.nvim (recommended for Neovim):**

Modern Lua-based plugin manager with lazy loading:
```lua
-- ~/.config/nvim/init.lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({"git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git", lazypath})
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({
  "nvim-treesitter/nvim-treesitter",
  "neovim/nvim-lspconfig",
})
```

**Native package management (Vim 8+ / Neovim):**
```bash
# For Vim
git clone https://github.com/tpope/vim-sensible.git \
    ~/.vim/pack/vendor/start/vim-sensible

# For Neovim
git clone https://github.com/tpope/vim-sensible.git \
    ~/.local/share/nvim/site/pack/vendor/start/vim-sensible
```

### Legacy Managers

**Pathogen (deprecated):**

Install Pathogen plugins from file:
```bash
while read plugin; do
    git clone https://github.com/$plugin ~/.vim/bundle/
done < plugins
```

> **💡 Note:** Pathogen is no longer actively maintained. Migrate to vim-plug or native packages.

---

## Cat with Heredoc

### Multiline string to variable

```bash
sql=$(cat <<EOF
SELECT foo, bar FROM db
WHERE foo='baz'
EOF
)
```

Check the variable:
```bash
echo -e "$sql"
```

### Multiline string to file

```bash
cat <<EOF > print.sh
#!/bin/bash
echo \$PWD
echo $PWD
EOF
```

The `print.sh` file will contain:
```bash
#!/bin/bash
echo $PWD
echo /home/user
```

### Multiline string to command/pipe

```bash
cat <<EOF | grep 'b' | tee b.txt | grep 'r'
foo
bar
baz
EOF
```

This creates `b.txt` with both `bar` and `baz` lines but prints only `bar`.

---

## Kubernetes

Essential kubectl commands and patterns.

### Context and Cluster Management

**List all contexts:**
```bash
kubectl config get-contexts
```

**Switch context:**
```bash
kubectl config use-context <context-name>
```

**Set default namespace:**
```bash
kubectl config set-context --current --namespace=<namespace>
```

**View current context:**
```bash
kubectl config current-context
```

### Resource Operations

**Get resources across all namespaces:**
```bash
kubectl get pods -A
kubectl get all -A
```

**Watch resources in real-time:**
```bash
kubectl get pods -w
kubectl get events -w
```

**Describe resource with details:**
```bash
kubectl describe pod <pod-name>
kubectl describe node <node-name>
```

**Get resource YAML:**
```bash
kubectl get pod <pod-name> -o yaml
kubectl get deployment <name> -o json | jq
```

### Logs and Debugging

**Follow logs:**
```bash
kubectl logs -f <pod-name>
kubectl logs -f <pod-name> -c <container-name>
```

**Previous container logs (after crash):**
```bash
kubectl logs <pod-name> --previous
```

**Logs from all pods with label:**
```bash
kubectl logs -l app=nginx --all-containers=true
```

**Execute command in pod:**
```bash
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container> -- sh
```

**Copy files to/from pod:**
```bash
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

### Port Forwarding

**Forward local port to pod:**
```bash
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80
```

**Forward with all addresses (accessible from network):**
```bash
kubectl port-forward --address 0.0.0.0 pod/<pod-name> 8080:80
```

### Resource Management

**Scale deployment:**
```bash
kubectl scale deployment <name> --replicas=3
```

**Restart deployment (rollout restart):**
```bash
kubectl rollout restart deployment/<name>
```

**Check rollout status:**
```bash
kubectl rollout status deployment/<name>
```

**Rollback deployment:**
```bash
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=2
```

### Quick Queries

**Get pod IPs:**
```bash
kubectl get pods -o wide
kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP
```

**Get node resource usage:**
```bash
kubectl top nodes
kubectl top pods
kubectl top pods -A --sort-by=memory
```

**Find pods on specific node:**
```bash
kubectl get pods -A -o wide --field-selector spec.nodeName=<node-name>
```

**Delete all evicted pods:**
```bash
kubectl get pods -A | grep Evicted | awk '{print $1, $2}' | xargs -n2 kubectl delete pod -n
```

### Useful Aliases

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
```

Add to `~/.bashrc` or `~/.zshrc`:
```bash
echo "alias k='kubectl'" >> ~/.bashrc
source <(kubectl completion bash)  # Enable bash completion
```

---

## Git

Common Git operations and workflows.

### Configuration

**Set user identity:**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Set default editor:**
```bash
git config --global core.editor vim
```

**Set default branch name:**
```bash
git config --global init.defaultBranch main
```

**View all config:**
```bash
git config --list
git config --list --show-origin
```

### Branch Management

**Create and switch to new branch:**
```bash
git checkout -b feature/new-feature
# or modern syntax
git switch -c feature/new-feature
```

**List branches:**
```bash
git branch -a  # All branches
git branch -r  # Remote branches
git branch -vv # Verbose with tracking info
```

**Delete branch:**
```bash
git branch -d branch-name  # Safe delete
git branch -D branch-name  # Force delete
git push origin --delete branch-name  # Delete remote
```

**Rename current branch:**
```bash
git branch -m new-branch-name
```

### Commit Operations

**Amend last commit:**
```bash
git commit --amend -m "New message"
git commit --amend --no-edit  # Keep same message
```

**Interactive rebase (clean up commits):**
```bash
git rebase -i HEAD~3  # Last 3 commits
```

**Cherry-pick commit:**
```bash
git cherry-pick <commit-hash>
```

**Revert commit (create new commit):**
```bash
git revert <commit-hash>
```

### Stash Management

**Stash changes:**
```bash
git stash
git stash save "Work in progress on feature X"
```

**List stashes:**
```bash
git stash list
```

**Apply stash:**
```bash
git stash apply
git stash apply stash@{2}
git stash pop  # Apply and remove
```

**Clear all stashes:**
```bash
git stash clear
```

### Remote Operations

**Add remote:**
```bash
git remote add origin https://github.com/user/repo.git
```

**Change remote URL:**
```bash
git remote set-url origin git@github.com:user/repo.git
```

**Fetch and prune deleted branches:**
```bash
git fetch --prune
git remote prune origin
```

**Pull with rebase:**
```bash
git pull --rebase
```

### Useful Queries

**Show changed files:**
```bash
git status -s
git diff --name-only
```

**Show commit history:**
```bash
git log --oneline
git log --graph --oneline --all
git log --since="2 weeks ago"
git log --author="username"
```

**Show file history:**
```bash
git log --follow -p -- <file>
```

**Find who changed a line:**
```bash
git blame <file>
git blame -L 10,20 <file>  # Lines 10-20
```

**Search commits:**
```bash
git log --grep="keyword"
git log -S "function_name"  # Search code changes
```

### Cleanup and Maintenance

**Clean untracked files:**
```bash
git clean -n  # Dry run
git clean -fd  # Remove files and directories
```

**Reset to remote state:**
```bash
git fetch origin
git reset --hard origin/main
```

**Remove file from Git but keep locally:**
```bash
git rm --cached <file>
```

**Optimize repository:**
```bash
git gc --aggressive --prune=now
```

### Aliases

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg 'log --graph --oneline --decorate --all'
```

---

## JSON/YAML Processing

Working with structured data from the command line.

### jq - JSON Processor

**Pretty print JSON:**
```bash
cat file.json | jq .
curl https://api.github.com/users/octocat | jq .
```

**Extract specific field:**
```bash
echo '{"name":"John","age":30}' | jq '.name'
echo '{"user":{"name":"John"}}' | jq '.user.name'
```

**Extract from array:**
```bash
echo '[{"name":"Alice"},{"name":"Bob"}]' | jq '.[0].name'
echo '[{"name":"Alice"},{"name":"Bob"}]' | jq '.[].name'  # All names
```

**Filter array:**
```bash
jq '.[] | select(.age > 25)' users.json
jq '.[] | select(.status == "active")' data.json
```

**Transform data:**
```bash
jq '.users | map({name: .name, email: .email})' data.json
```

**Create new JSON:**
```bash
jq -n '{name: "Alice", age: 30}'
```

**Raw output (without quotes):**
```bash
echo '{"name":"John"}' | jq -r '.name'
```

**Combine with kubectl:**
```bash
kubectl get pods -o json | jq '.items[].metadata.name'
kubectl get nodes -o json | jq '.items[].status.addresses[] | select(.type=="InternalIP") | .address'
```

### yq - YAML Processor

**Read YAML value:**
```bash
yq '.metadata.name' deployment.yaml
yq '.spec.replicas' deployment.yaml
```

**Update YAML value:**
```bash
yq '.spec.replicas = 5' deployment.yaml
yq -i '.spec.replicas = 5' deployment.yaml  # In-place edit
```

**Convert YAML to JSON:**
```bash
yq -o json . file.yaml
```

**Convert JSON to YAML:**
```bash
yq -P . file.json
```

**Merge YAML files:**
```bash
yq eval-all 'select(fileIndex == 0) * select(fileIndex == 1)' file1.yaml file2.yaml
```

**Filter array in YAML:**
```bash
yq '.items[] | select(.metadata.name == "nginx")' resources.yaml
```

### Combining Tools

**Extract, process, and format:**
```bash
kubectl get pods -o json | jq -r '.items[] | "\(.metadata.name) - \(.status.phase)"'
```

**Generate Kubernetes resource from JSON:**
```bash
cat config.json | jq '{apiVersion: "v1", kind: "ConfigMap", metadata: {name: .name}, data: .data}' | kubectl apply -f -
```

**Process Terraform output:**
```bash
terraform output -json | jq -r '.vpc_id.value'
```

---

## System Monitoring

### Process Management

**Find process by name:**
```bash
ps aux | grep process_name
pgrep -fl process_name
```

**Kill process by name:**
```bash
pkill process_name
killall process_name
```

**Kill process by port:**
```bash
lsof -ti:8080 | xargs kill -9
# or
fuser -k 8080/tcp
```

**Monitor processes:**
```bash
top
htop  # Better alternative
btop  # Modern alternative
```

**Sort processes by memory:**
```bash
ps aux --sort=-%mem | head
```

**Sort processes by CPU:**
```bash
ps aux --sort=-%cpu | head
```

### Disk Usage

**Check disk space:**
```bash
df -h
df -h /path/to/mount
```

**Check directory size:**
```bash
du -sh directory/
du -h directory/ | sort -rh | head -10
```

**Find large files:**
```bash
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

**Disk I/O statistics:**
```bash
iostat
iostat -x 2  # Extended stats every 2 seconds
```

### Network Monitoring

**Check open ports:**
```bash
netstat -tulpn
ss -tulpn  # Modern alternative
lsof -i -P -n
```

**Check connections to specific port:**
```bash
netstat -an | grep :80
ss -an | grep :80
```

**Monitor network traffic:**
```bash
iftop
nethogs  # By process
nload    # By interface
```

**Test bandwidth:**
```bash
iperf3 -s  # Server
iperf3 -c server_ip  # Client
```

**DNS lookup:**
```bash
dig example.com
nslookup example.com
host example.com
```

**Trace route:**
```bash
traceroute example.com
mtr example.com  # Better alternative
```

### System Resources

**Memory usage:**
```bash
free -h
vmstat 1  # Every second
```

**CPU information:**
```bash
lscpu
cat /proc/cpuinfo
```

**System load:**
```bash
uptime
w
```

**Check systemd services:**
```bash
systemctl status service_name
systemctl list-units --type=service
systemctl list-units --failed
```

**View system logs:**
```bash
journalctl -xe
journalctl -u service_name
journalctl -f  # Follow
journalctl --since "1 hour ago"
```

### Performance Testing

**Test disk write speed:**
```bash
dd if=/dev/zero of=testfile bs=1G count=1 oflag=direct
```

**Test disk read speed:**
```bash
dd if=testfile of=/dev/null bs=1G count=1 iflag=direct
```

**Stress test CPU:**
```bash
stress --cpu 4 --timeout 60s
```

---

## Additional Resources

### Helpful Tools

**Modern CLI alternatives to classic Unix tools:**
- `bat` - Better `cat` with syntax highlighting
- `exa` / `eza` - Modern `ls` replacement
- `fd` - User-friendly `find` alternative
- `ripgrep` (`rg`) - Faster `grep` replacement
- `htop` / `btop` - Interactive process viewer
- `tldr` - Simplified man pages with examples
- `jq` - JSON processor
- `yq` - YAML processor

**Installation (macOS):**
```bash
brew install bat eza fd ripgrep htop btop tldr jq yq
```

**Installation (Ubuntu/Debian):**
```bash
sudo apt install bat fd-find ripgrep htop jq
```

### Shell Productivity

**History search:**
```bash
# Reverse search
Ctrl + R

# Forward search (after Ctrl+R)
Ctrl + S
```

**Directory navigation:**
```bash
# Go to previous directory
cd -

# Go to home directory
cd ~

# Go up two directories
cd ../..
```

**Command shortcuts:**
```bash
!!          # Repeat last command
!$          # Last argument of previous command
!^          # First argument of previous command
^old^new    # Replace 'old' with 'new' in last command
```

---

## GitHub Repository Automation

### .github/settings.yml

This repository includes a [.github/settings.yml](.github/settings.yml) file for automating GitHub repository configuration using the [GitHub Settings app](https://probot.github.io/apps/settings/).

**What it automates:**
- Repository metadata (name, description, topics, homepage)
- Repository features (issues, projects, wiki, downloads)
- Merge strategies and branch deletion
- Security settings (automated fixes, vulnerability alerts)
- Issue labels with colors and descriptions
- Branch protection rules for `main`
- Pull request review requirements

**Setup:**

1. Install the GitHub Settings app:
   ```
   https://github.com/apps/settings
   ```

2. Grant access to your repository

3. The app will automatically apply settings from `.github/settings.yml` whenever you push changes

**Manual configuration check:**
```bash
# View current settings
cat .github/settings.yml

# Validate YAML syntax
yq eval .github/settings.yml
```

**Key features configured:**

- **Labels**: Pre-configured issue labels including custom ones for Kubernetes, Docker, security, and automation
- **Branch Protection**: Requires 1 approval review, conversation resolution before merging
- **Auto-cleanup**: Automatically deletes head branches after PR merge
- **Security**: Automated security fixes and vulnerability alerts enabled
- **Topics**: Repository tagged with relevant topics for discoverability

**Customization:**

Edit `.github/settings.yml` to modify:
```yaml
repository:
  topics:
    - your-custom-topic

labels:
  - name: custom-label
    color: ff6347
    description: Your custom label

branches:
  - name: main
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 2  # Increase reviews
```

Push changes to apply new configuration automatically.

---

## Contributing

Feel free to submit pull requests with additional tips and tricks!

## License

MIT License - feel free to use these commands in your own work.
