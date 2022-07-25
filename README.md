# Windows
Following are instructions to setup a WSL2 environment in Windows tailored to zoops specific needs. Only ment for members working on projects with him. And personal documentation.  
WSL is basically VMs out of docker images with some perks such as having the windows file system mounted and networking setup for you. It runs great in conjunction with Docker Desktop and VSCode.

 ## Windows setup
 We want to install whatever we need for WSL2 to work optimally.
 1. Install the windows terminal for better experience when working with the console.  
 Use these instructions: https://docs.microsoft.com/en-us/windows/terminal/install
 2. Install [a nerdfont](https://www.nerdfonts.com/font-downloads) for use in terminal. I recommend the standard p10k meslo font though:  
    Download the [regular](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf), [bold](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf), [italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf) and [bold italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf) fonts and install them.  
    Change the default font in Settings -> Defaults -> Appearance -> Font face -> *Meslo LGS NF*
 3. Launch the terminal as administrator and [install wsl](https://docs.microsoft.com/en-us/windows/wsl/install) if not already installed. If yes make sure it's [updated](https://docs.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package).
 4. Install [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline), we will use this to install Linux distros not packaged by Microsoft.
 5. Install [VSCode](https://code.visualstudio.com/download).

## Install WSL Distro
You can use the default Ubuntu distro, skip this step if you do. But it's usually quite outdated. Debian Bullseye should be better supported too. (for now)
1. Select a folder you want to install the WSL Distro into. I have chosen "F:\Virtual Machines".
2. Download the image. I have selected Debian Bookworm but there's more available. See [this link](https://github.com/DDoSolitary/LxRunOffline/wiki) for more. In powershell:  

    ```
    wget "https://lxrunoffline.apphb.com/download/Debian/bookworm" -outfile "rootfs.tar.xz"
    ```
3. Create the WSL VM with *-n \<Name> -d \<Folder>* as parameter:
   ```
   LxRunOffline i -n Bookworm -d "F:\Virtual Machines\Bookworm" -f "rootfs.tar.xz" -s
   ```
4. Update it to WSL2 as it installs as WSL1 by default:
   ```
   wsl --set-version Bookworm 2
   ```
Additional Tipps:
* Use `wsl -l -v` to display your installed distros.
* In Windows Terminal you can hide unwanted ones from the dropdown and set a default startup distro. It also has json a json config option.
* Use `wsl -l -o` to show available distros from MS and install them with `wsl --install -d <distro>`
* Uninstall a distro with `wsl --unregister <name>` or `LxRunOffline ui -n <name>`

## Linux Setup
Install some often used software, remove & add as you wish. Some can most likely be skipped if you go with default Debian.
```
apt update && apt install -y sudo nano git curl wget jq
```
### User Setup
It's up to you if you want to setup a user or keep being root. Warning: if you remain root don't change shell to zsh.  
Set a password with `passwd root`  
For setting up a user first follow these steps in distro and take note of the uid:  
```
adduser zoop
usermod -aG sudo zoop
su zoop
id
sudo visudo -f /etc/sudoers.d/zoop
#add line for passwordless sudo
#zoop ALL=(ALL) NOPASSWD: ALL
```
Then for automatically logging in as that user (my uid was 1000) run this in powershell:
```
lxrunoffline su -n Bookworm -v 1000
```
### Shell setup
We will go for zsh with p10k.
```
sudo apt install zsh zsh-doc
chsh -s /bin/zsh
zsh
#go trough zsh wizard
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
source ~/.zshrc
#go through p10k wizard
```
These are the [color changes](https://github.com/romkatv/powerlevel10k#how-do-i-change-prompt-colors) I made in my .p10k.zsh file:
```
  typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND=208
  typeset -g POWERLEVEL9K_OS_ICON_BACKGROUND=232
  typeset -g POWERLEVEL9K_DIR_BACKGROUND=172
  typeset -g POWERLEVEL9K_DIR_FOREGROUND=016
  typeset -g POWERLEVEL9K_DIR_SHORTENED_FOREGROUND=016
  typeset -g POWERLEVEL9K_DIR_ANCHOR_FOREGROUND=016
```
to print color palette use `for i in {0..255}; do print -Pn "%K{$i}  %k%F{$i}${(l:3::0:)i}%f " ${${(M)$((i%6)):#3}:+$'\n'}; done`

### Install tools
1. exa
2. kubectl
3. kubens
4. k9s
5. z