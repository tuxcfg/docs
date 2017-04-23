Software
========

System update: 

```bash
sudo apt update && sudo apt upgrade
```

Basic set:

```bash
sudo apt install acpid openssh-server bash-completion mc htop git curl tmux vim
```

Desktop set:

```bash
sudo apt install synaptic chromium-browser clipit geany geany-plugin-addons geany-plugin-markdown geany-plugins-common gthumb gimp inkscape meld mpv ncdu nfs-common openvpn whois nmap
```

[Albert](https://github.com/albertlauncher/albert) launcher:

```bash
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt update
sudo apt install albert
```

Oracle Java:

```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt update
sudo apt install oracle-java8-installer
```

[Atom](https://github.com/atom/atom):

```bash
sudo add-apt-repository ppa:webupd8team/atom
sudo apt-get update
sudo apt install atom
```

[NodeJS](https://nodejs.org/en/download/package-manager/):

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt install nodejs build-essential
```

Skype:

```bash
curl -s https://repo.skype.com/data/SKYPE-GPG-KEY | sudo apt-key add -
wget https://repo.skype.com/latest/skypeforlinux-64.deb
sudo dpkg -i skypeforlinux-64.deb
rm skypeforlinux-64.deb
```

Remove unnecessary programs:

```bash
apt remove command-not-found
```