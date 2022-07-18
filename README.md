# izOpen
izOpen is a multi protocol bash script to auto opening given URI and auto creating ssh reverse tunnel socks proxy
izOpen was designed to be used as secure URI launcher in conjunction with any Password Manager (KeePassX, Browser Based etc...) 

## Features
supported URI schemas: ssh, rdp, vnc, sftp, ftp, http, https, smb, cifs

### Linux OS Dependencies

### Packages
Download and install the latest version of KeePassXC (https://keepassxc.org/).

**Fedora / RHEL 8 DNF/RPM based distro integration:**  
```
sudo dnf install -y proxychains-ng xdg-utils yad openssh-clients sshpass telnet nmap-ncat xsel freerdp remmina keepassxc
```

### Desktop Intergration

#### MIME associations

Be sure the applications directory exist
```
mkdir -p ~/.local/share/applications/
```

Create XDG desktop files
```
echo "[Desktop Entry]
Version=1.0
Name=izOpen SSH Handler
Comment=Open SSH connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=utilities-terminal
Type=Application
Categories=Network;
MimeType=x-scheme-handler/ssh;" > ~/.local/share/applications/izopen-ssh-handler.desktop
 
echo "[Desktop Entry]
Version=1.0
Name=izOpen TELNET Handler
Comment=Open TELNET connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=utilities-terminal
Type=Application
Categories=Network;
MimeType=x-scheme-handler/telnet;" > ~/.local/share/applications/izopen-telnet-handler.desktop
 
echo "[Desktop Entry]
Version=1.0
Name=izOpen RDP Handler
Comment=Open RDP connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=utilities-terminal
Type=Application
Categories=Network;
MimeType=x-scheme-handler/rdp;" > ~/.local/share/applications/izopen-rdp-handler.desktop
 
echo "[Desktop Entry]
Version=1.0
Name=izOpen VNC Handler
Comment=Open VNC connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=vinagre
Type=Application
Categories=Network;
MimeType=x-scheme-handler/vnc;" > ~/.local/share/applications/izopen-vnc-handler.desktop
```

Register the new mime schemas:
```
xdg-mime default izopen-ssh-handler.desktop x-scheme-handler/ssh
xdg-mime default izopen-ssh-handler.desktop x-scheme-handler/sftp
xdg-mime default izopen-telnet-handler.desktop x-scheme-handler/telnet
xdg-mime default izopen-rdp-handler.desktop x-scheme-handler/rdp
xdg-mime default izopen-vnc-handler.desktop x-scheme-handler/vnc
```

Verify the correct associations:
```
xdg-mime query default x-scheme-handler/ssh
xdg-mime query default x-scheme-handler/telnet
xdg-mime query default x-scheme-handler/rdp
xdg-mime query default x-scheme-handler/vnc
```

Rebuild the Desktop MIME database:
```
update-desktop-database ~/.local/share/applications/
update-mime-database    ~/.local/share/mime/
```

#### Open a New Browser Session using the reverse tunnel

Create a new file: `~/.local/share/applications/izopen-http.desktop`
```
echo '#!/usr/bin/env xdg-open
[Desktop Entry]
Version=1.0
Name=izOpen Web Browser
Comment=Browse the Web through the specified local proxy socks port connected via SSH tunnnel
Exec=izopen -f
StartupNotify=true
Terminal=false
Type=Application
Icon=proxytunnel
Categories=GTK;Network;WebBrowser;
MimeType=text/html;x-scheme-handler/http;x-scheme-handler/https;' > ~/.local/share/applications/izopen-http.desktop
```

Add this desktop launcher into you Desktop Bar or Menu

## Windows OS Installation (Windows 10/11) (WORK IN PROGRESS)

### Windows OS Dependencies (using cygwin system)

1. Install Chocolatey Package Manager:  
Run:
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
Go to https://chocolatey.org/install for additional info

2. Install Cygwin:  
Open Power Shell as Administrator and run:
```
choco install cygwin cyg-get -y
```

3. Install Cygwin mandatory components:  
Open "Cygwin64 Terminal":
```
cyg-get.bat vim nc sshpass yad freerdp xorg-server xinit
```

4. Configure Cygwin env:  
Open "Cygwin64 Terminal":
```
echo 'export DISPLAY=:0' >> ~/.bashrc
```

5. Remember to open **XWin Server** Application every time before you want use izopen

6. Download Google Chrome as default izopen browser from https://www.google.com/chrome

7. Create the file `izopen.reg` containing URL handler configuration (WIP: WARNIN DOESN'T WORKS RIGHT NOW)
```
REGEDIT4
[HKEY_CLASSES_ROOT\ssh]
@="izOpen SSH Handler"
"URL Protocol"=""
[HKEY_CLASSES_ROOT\ssh\shell]
[HKEY_CLASSES_ROOT\ssh\shell\open]
[HKEY_CLASSES_ROOT\ssh\shell\open\command]
@="\"c:\\tools\\cygwin\\bin\\bash -l -i -c izopen \"%1\""
```

C:\tools\cygwin\bin\bash -l -i -c "izopen ssh://root@10.1.1.8 -d"

c:\tools\cygwin\bin\bash.exe --login -i -c 'izopen -d ssh://root@10.1.1.8'

8. Continue with izopen installation


### MSYS2 method (DOESN'T WORKS!!! DON'T USE!!! IS HERE ONLY FOR REFERENCE)
Open a Windows Command prompt as Administrator and run:
```
pacman -Syu
pacman -S openssh sshpass openbsd-netcat vim mingw-w64-x86_64-putty mingw-w64-x86_64-freerdp
```

Reboot the system (of course, as always on windows, you are so used to it :) )

#### Manage packages
Update packages info: `pacman -Fy`
Search for available package: `pacman -Ss NAME`
Search for installed package: `pacman -Qs NAME`
Print installed package info: `pacman -Qi NAME`
Print installed package files list: `pacman -Ql NAME`
Install package: `pacman -S NAME`

Addintional info: https://wiki.archlinux.org/title/pacman

## izOpen Installation
```
mkdir -p ~/.local/bin
wget https://raw.githubusercontent.com/ugoviti/izopen/master/izopen -O ~/.local/bin/izopen
chmod 755 ~/.local/bin/izopen
echo 'PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
```

## Usage (with examples):
`izopen -h`

## Customize izopen configuration:
All user custom configurations are located into:
- `$HOME/.config/izopen/izopen.conf`

The file is created on izopen first run (if not exist).  

If you want overwrite the config file with the default options (after upgrade for exampe) run:
- `izopen --mkconfig`

NB. by default this configuration is enabled:
`helper_opts[ssh]="-o StrictHostKeyChecking=no -o CheckHostIP=no"`
WARNING: it involves security concerns, but are required by sshpass for the first time to a new host.
If you dont't want use these options enabled, disable from your user config file.
