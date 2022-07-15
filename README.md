# izOpen
izOpen is a multi protocol bash script to auto opening given URI and auto creating ssh reverse tunnel socks proxy
izOpen was designed to be used as secure URI launcher in conjunction with any Password Manager (KeePassX, Browser Based etc...) 

## Features
supported URI schemas: ssh, rdp, vnc, sftp, ftp, http, https, smb, cifs

## Installation
```
mkdir -p ~/.local/bin
wget https://raw.githubusercontent.com/ugoviti/izopen/master/izopen -O ~/.local/bin/izopen
chmod 755 ~/.local/bin/izopen
```

### OS Dependencies

### Packages
Download and install the latest version of KeePassXC (https://keepassxc.org/).

**KeePassXC 2.x integration (Fedora / CentOS 8):**  
```
dnf install -y yad xfreerdp proxychains-ng keepassxc sshpass xclip
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
Name=Web Browser via SSH tunnel
Comment=Browse the Web via the specified local SSH tunnnel socks proxy port
Exec=izopen -t
StartupNotify=true
Terminal=false
Type=Application
Icon=web-browser
Categories=GTK;Network;WebBrowser;
MimeType=text/html;x-scheme-handler/http;x-scheme-handler/https;' > ~/.local/share/applications/izopen-http.desktop
```

Add this desktop launcher into you Desktop Bar or Menu

## Usage (with examples):
`izopen -h`
