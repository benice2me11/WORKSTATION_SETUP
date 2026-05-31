# Workstation Setup

Personal Ubuntu workstation bootstrap guide for local development and agent-driven setup.

Target system: Ubuntu 26.04 LTS or another recent Ubuntu/Debian-based system.

## 1. Update Apt

```bash
sudo apt update
sudo apt upgrade -y
```

## 2. Install Base Packages

```bash
sudo apt install -y \
  git \
  curl \
  wget \
  build-essential \
  make \
  zsh
```

Check the installed tools:

```bash
git --version
curl --version
wget --version
gcc --version
make --version
zsh --version
```

## 3. Configure Git

Set the Git identity once per machine:

```bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_EMAIL"
git config --global init.defaultBranch main
```

Do not guess the name or email for another user. If the machine is configured by an agent, ask the user which identity should be used for commits.

Check the config:

```bash
git config --global --list
```

## 4. Create SSH Key

Create the SSH directory:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Generate an RSA key:

```bash
test ! -f ~/.ssh/id_rsa || {
  echo "SSH key already exists: ~/.ssh/id_rsa"
  echo "Do not overwrite it unless the user explicitly asks for a new key."
  exit 1
}

ssh-keygen -t rsa -b 4096 -C "$(git config --global user.email)" -f ~/.ssh/id_rsa
```

Show the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Add the public key to GitHub or GitLab.

Test GitHub SSH access:

```bash
ssh -T git@github.com
```

The first connection may ask to trust GitHub's host key. After the key is added to GitHub, the command should authenticate successfully.

## 5. Install Oh My Zsh

Install Oh My Zsh:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

For agent-driven or non-interactive setup, install without changing the login shell automatically:

```bash
RUNZSH=no CHSH=no KEEP_ZSHRC=yes sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

If `zsh` is not the default shell after installation:

```bash
chsh -s "$(which zsh)"
```

`chsh` may ask for the user's password. Log out and log back in after changing the default shell.

Check the active login shell:

```bash
getent passwd "$USER" | cut -d: -f7
```

If it still prints `/bin/bash`, Oh My Zsh may be installed correctly, but Bash terminals will not load Zsh plugins. Open a new Zsh shell with:

```bash
zsh
```

Then verify plugin loading from inside Zsh:

```bash
echo "$plugins"
```

## 6. Install Zsh Plugins

Install `zsh-autosuggestions`:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
  "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
```

Install `zsh-syntax-highlighting`:

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
```

Enable plugins in `~/.zshrc`:

```text
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

Or update the plugin line automatically:

```bash
sed -i 's/^plugins=.*/plugins=(git zsh-autosuggestions zsh-syntax-highlighting)/' ~/.zshrc
```

Reload the shell:

```bash
source ~/.zshrc
```

## 7. Install Go

For agent-driven setup without `sudo`, install the official Go archive into the user's home directory:

```bash
GO_VERSION="$(curl -fsSL https://go.dev/VERSION?m=text | awk '{print $1}')"
GO_ARCHIVE="${GO_VERSION}.linux-amd64.tar.gz"
GO_INSTALL_DIR="$HOME/.local"

test ! -d "$GO_INSTALL_DIR/go" || {
  echo "Go already exists: $GO_INSTALL_DIR/go"
  echo "Check the current version before replacing it."
  exit 1
}

mkdir -p "$GO_INSTALL_DIR" "$HOME/go/bin"
curl -fL "https://go.dev/dl/${GO_ARCHIVE}" -o "/tmp/${GO_ARCHIVE}"
tar -C "$GO_INSTALL_DIR" -xzf "/tmp/${GO_ARCHIVE}"
```

If `~/.local/go` already exists, do not overwrite it blindly. Check the current version first:

```bash
~/.local/go/bin/go version
```

Add Go to `~/.profile`, `~/.bashrc`, and `~/.zshrc`:

```bash
export PATH="$HOME/.local/go/bin:$HOME/go/bin:$PATH"
```

Verify Go:

```bash
go version
go env GOPATH GOROOT GOOS GOARCH
```

Official docs: https://go.dev/doc/install

## 8. Install Docker CE And Docker Compose

Use Docker's official APT repository, not the `docker.io` package from Ubuntu's default repositories. This keeps Docker Engine, Buildx, and Docker Compose on the current stable Docker release line.

Official Docker documentation currently supports Ubuntu Resolute 26.04 LTS, Questing 25.10, Noble 24.04 LTS, and Jammy 22.04 LTS.

These commands require `sudo`. If an agent cannot authenticate through `sudo`, run this section manually in a local terminal.

Remove conflicting distro packages if they exist:

```bash
sudo apt remove -y \
  docker.io \
  docker-doc \
  docker-compose \
  docker-compose-v2 \
  podman-docker \
  containerd \
  runc || true
```

Install repository prerequisites:

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

Add Docker's official GPG key:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker's APT repository:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources >/dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update package metadata:

```bash
sudo apt update
```

Optional: check that `docker-ce` will be installed from Docker's repository:

```bash
apt-cache policy docker-ce
```

Install the latest stable Docker packages:

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

Check that Docker is running:

```bash
sudo systemctl status docker
```

If Docker is not running, start it manually:

```bash
sudo systemctl start docker
```

Verify Docker Engine:

```bash
sudo docker run hello-world
```

Verify Docker Compose:

```bash
docker --version
docker compose version
```

## 9. Run Docker Without Sudo

Docker can be used without `sudo` by adding the current user to the `docker` group. This is convenient for a personal dev machine, but the `docker` group effectively grants root-level privileges on the host.

```bash
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker "$USER"
```

Log out and log back in after changing Docker group membership. A terminal that was already open before `usermod` may still not have the `docker` group in its active process groups.

Check the system group database and the current terminal process groups:

```bash
groups "$USER"
id
```

`groups "$USER"` should list `docker`. The plain `id` command must also list `docker` before Docker works without `sudo` in the current terminal.

Check that Docker works without `sudo`:

```bash
docker run hello-world
```

## 10. Final Verification

Run this block after setup:

```bash
git --version
curl --version
wget --version
gcc --version
make --version
zsh --version
test -f ~/.ssh/id_rsa.pub && echo "SSH public key exists"
test -d ~/.oh-my-zsh && echo "Oh My Zsh exists"
test -d ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions && echo "zsh-autosuggestions exists"
test -d ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting && echo "zsh-syntax-highlighting exists"
go version
docker --version
docker compose version
```

If the current login shell is still Bash, switch to Zsh with:

```bash
chsh -s "$(which zsh)"
```

Then log out and log back in.

## Agent Checklist

When an agent prepares or verifies this workstation, it should:

1. Detect the operating system with `cat /etc/os-release`.
2. Install only the base packages listed in this document unless the user asks for more.
3. Avoid overwriting an existing `~/.ssh/id_rsa` key without explicit confirmation.
4. Print only the public SSH key, never the private key.
5. Do not guess Git `user.name` or `user.email`; ask the user for the commit identity.
6. Keep Oh My Zsh plugins limited to:

```text
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

7. Install Oh My Zsh with `RUNZSH=no CHSH=no KEEP_ZSHRC=yes` when running non-interactively.
8. Install Go into `~/.local/go` when `sudo` is unavailable, and add `~/.local/go/bin` plus `~/go/bin` to both Bash and Zsh startup files.
9. If `sudo` requires an interactive password, stop before system package or Docker installation and ask the user to run those commands locally.
10. Install Docker from Docker's official APT repository, not from Ubuntu's `docker.io` package.
11. Install Compose through `docker-compose-plugin`; do not manually download the Compose binary unless the user asks for that.
12. Warn the user that adding someone to the `docker` group grants root-level host privileges.
