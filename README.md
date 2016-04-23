# izOpen
izOpen is a multi protocol bash script to auto opening given URI and auto creating ssh reverse tunnel socks proxy
designed from scratch to be used inside KeePassX as secure URI launcher

## Features
URI handling of: ssh, rdp, vnc, sftp, ftp, http, https, smb and cifs protocols

## Installation
Copy this script into a user or system wide bin directory (suggestion: cp izopen ~/bin), must be into a PATH variable

### OS Dependencies
Download and install the latest version of KeePassX (https://www.keepassx.org).

**KeePassX 0.x integration (Fedora <= 23, CentOS 5/6):**
`yum install -y zenity kdelibs3 proxychains-ng keepassx`

into advanded settings of keepassx select "Custom Browser Command" and insert:
`izopen %1`

**KeePassX 2.x integration (Fedora >= 23):**
`dnf install -y zenity kdelibs3 proxychains-ng keepassx`

create: ~/.local/share/applications/ssh-handler.desktop
```
[Desktop Entry]
Type=Application
Name=SSH Handler
Exec=izopen %u
Icon=utilities-terminal
StartupNotify=false
MimeType=x-scheme-handler/ssh;
```

create: ~/.local/share/applications/rdp-handler.desktop
```
[Desktop Entry]
Type=Application
Name=RDP Handler
Exec=izopen %u
Icon=utilities-terminal
StartupNotify=false
MimeType=x-scheme-handler/rdp;
```

Run:
```
xdg-mime default ssh-handler.desktop x-scheme-handler/ssh
xdg-mime default rdp-handler.desktop x-scheme-handler/rdp
```

## Usage:
`izopen uri://username:password@hostname:port`

### Examples
```
izopen http://www.example.com
izopen http://john:doe@www.example.com
izopen ssh://root:supersecurepassword@www.example.com
izopen smb://Administrator:password@server
izopen rdp://DOMAIN/Administrator:password@server
```


