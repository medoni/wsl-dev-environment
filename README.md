# WSL Dev environment

## Installation

1. Create new Ubuntu wsl distro
   ```bash
       wsl --install -d Ubuntu-24.04
   ```
2. Make as default
   ```bash
       wsl -s Ubuntu-24.04
   ```
3. Start wsl
    ```bash
        wsl
    ```
4. Paste the following commands

```bash
sudo apt update &&
    sudo apt upgrade -y

sudo apt-get install -y \
    libsecret-1-0 \
    gnupg software-properties-common

# Git
# ---
sudo apt install -y git
git config --global credential.helper store

# Python
# ------
sudo apt install -y python3-pip pipx
pipx ensurepath

# Terraform
# ---------
wget -O- https://apt.releases.hashicorp.com/gpg | \
    gpg --dearmor | \
    sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
    https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
    sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt-get -y install terraform

# Aws CLI
# -------
pipx install awscli

# Powershell
# ----------
wget -q https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install -y powershell

# Dotnet
# ------
sudo add-apt-repository ppa:dotnet/backports && \
    sudo apt-get update && \
    sudo apt-get install -y dotnet-sdk-9.0

sudo apt-get update && \
    sudo apt-get install -y dotnet-sdk-8.0

# NPM & NPX
# ---------
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

# Kubectl
# -------
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin

# Docker cli
# ----------
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Trivi
# -----
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy

# Podman
# ------
sudo apt install podman

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update && sudo apt install -y docker-ce-cli

# Other tools
# -----------
sudo apt install -y \
    htop \
    btop \
    bat \
    net-tools \
    cowsay \
    fortunes-bofh-excuses \
    yq \
    lolcat \
    inetutils-traceroute \
    mtr-tiny

```

---

## Setup Powershell $Profile

```powershell
pwsh -c '$ProgressPreference = "SilentlyContinue" ; Install-Module DirColors'
pwsh -c '$ProgressPreference = "SilentlyContinue" ; Install-Module posh-git'
mkdir -p /home/micha/.config/powershell/
cat <<'EOF' > ~/.config/powershell/Microsoft.PowerShell_profile.ps1
Import-Module DirColors
Import-Module posh-git
Set-PSReadlineOption -EditMode Emacs
Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
Set-Variable -Name MaximumHistoryCount 32767

function global:prompt { 
    Write-Host ""
    Write-Host "$(get-location -psprovider filesystem)" -ForegroundColor darkgreen
    Write-Host "$([char]0x03BB)" -NoNewline -ForegroundColor green
    return " "
}
$env:PATH +=";/home/micha/.local/bin"

# basic aliases
function Make-Alias($alias, $target) {
    $fn = "function global:$($alias) { $($target) `$args }"
    Invoke-Expression $fn
}
Make-Alias 'la' 'dir'

# git aliases
Remove-Alias -name gc,gl -f
Make-Alias 'gl'   'git log --decorate=auto --oneline --graph'
Make-Alias 'glfp' 'git log --decorate=auto --oneline --graph --first-parent'
Make-Alias 'gs'   'git status'
Make-Alias 'go'   'git checkout'
Make-Alias 'gb'   'git branch'
Make-Alias 'gbd'  'git for-each-ref --sort=committerdate refs/heads/ --format="%(committerdate:short) %(refname:short)"'
Make-Alias 'gc'   'git commit -a -m'
Make-Alias 'gca'  'git commit --amend'
Make-Alias 'ga'   'git add'
Make-Alias 'ga.'  'git add .'
Make-Alias 'gd'   'git diff'
Make-Alias 'gcp'  'git cherry-pick'

# docker aliases
Make-Alias "d" "podman"
Make-Alias "dc" "podman compose"

EOF
```
