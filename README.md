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

---

## Benchmarking

Quick server benchmarking scripts.

**Method 1:**
```bash
wget -qO- bench.sh | bash
```

**Method 2:**
```bash
curl -Lso- bench.sh | bash
```

> **⚠️ Security Note:** Always review scripts before piping to bash. Consider running without `| bash` first to inspect the content.

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

---

## Pastebin Services

### Termbin

Quick text sharing via netcat.

Pipe any output to termbin:
```bash
ls -la | nc termbin.com 9999
```

Website: http://termbin.com/

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
curl -fsSL https://get.docker.com | sudo sh
```

> **⚠️ Note:** Always review scripts before piping to shell.

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

```bash
du -hsx * | sort -rh | head -10
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
```

---

## SSH

### Fix authentication agent error

**Modern syntax (preferred):**
```bash
eval "$(ssh-agent -s)"
```

Check agent socket:
```bash
echo $SSH_AUTH_SOCK
```

**Legacy syntax (still works):**
```bash
eval `ssh-agent`
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

**List loaded keys:**
```bash
ssh-add -l
```

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

### Pathogen (Legacy)

Install Pathogen plugins from file (if you have a `plugins` file listing GitHub repositories):
```bash
while read plugin; do
    git clone https://github.com/$plugin ~/.vim/bundle/
done < plugins
```

### Modern Alternatives

**vim-plug (most popular):**

Add to `~/.vimrc`:
```vim
call plug#begin()
Plug 'tpope/vim-sensible'
Plug 'junegunn/fzf'
call plug#end()
```

**Native package management (Vim 8+):**
```bash
git clone https://github.com/tpope/vim-sensible.git \
    ~/.vim/pack/vendor/start/vim-sensible
```

**Neovim with lazy.nvim:**
Modern Lua-based plugin manager for Neovim users.

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

## Contributing

Feel free to submit pull requests with additional tips and tricks!

## License

MIT License - feel free to use these commands in your own work.
