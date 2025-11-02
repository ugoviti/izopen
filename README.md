# izOpen

**Multi-protocol URI launcher with SSH SOCKS tunnel integration**

izOpen is a bash script designed to seamlessly open URIs with automatic SSH tunnel creation, perfect for secure remote access through password managers (KeePassXC, browser-based, etc.).

## Features

- ✅ **Multi-protocol support**: SSH, Telnet, RDP, VNC, SFTP, FTP, HTTP/HTTPS, SMB, CIFS
- ✅ **SSH SOCKS proxy tunnels**: Automatic tunnel creation and management with proxychains4 integration
- ✅ **Multiple RDP/VNC clients**: Support for xfreerdp, remmina, krdc, rdesktop, vncviewer
- ✅ **Password manager integration**: Seamless URI handling from KeePassXC and browser extensions
- ✅ **Special character handling**: Proper password encoding for all supported clients
- ✅ **Clipboard integration**: Automatic password reading from clipboard (Wayland & X11)
- ✅ **Desktop integration**: XDG MIME handler registration for all protocols

## Supported Protocols & Clients

### Network Protocols
- **SSH/SFTP**: OpenSSH client with sshpass authentication
- **Telnet**: Standard telnet client
- **FTP**: XDG default handler
- **HTTP/HTTPS**: Google Chrome, Midori, Firefox, or system default browser
- **SMB/CIFS**: XDG default file manager

### Remote Desktop (RDP)
| Client | Status | Features |
|--------|--------|----------|
| **remmina** | ✅ Default | GUI client, config file based, full tunnel support |
| **xfreerdp** | ✅ Supported | FreeRDP 3.x, CLI client, bash array password handling |
| **krdc** | ✅ **NEW v3.1.0** | KDE client, URL-based, automatic password encoding |
| **rdesktop** | ✅ Supported | Legacy client, basic features |

### Remote Desktop (VNC)
| Client | Status | Features |
|--------|--------|----------|
| **remmina** | ✅ Default | GUI client, config file based, full tunnel support |
| **krdc** | ✅ **NEW v3.1.0** | KDE client, URL-based, simple password handling |
| **vncviewer** | ✅ Supported | TigerVNC, CLI client |

## Dependencies

### Required Packages

**Fedora / RHEL / CentOS (DNF/RPM):**
```bash
sudo dnf install -y proxychains-ng xdg-utils yad zenity kdialog openssh-clients sshpass telnet nmap-ncat xsel wl-clipboard freerdp remmina keepassxc krdc
```

**Debian / Ubuntu (APT):**
```bash
sudo apt update
sudo apt install -y proxychains-ng xdg-utils yad zenity kdialog openssh-client sshpass telnet nmap netcat xsel wl-clipboard freerdp2-x11 remmina keepassxc krdc
```

### Optional Dependencies
- **KeePassXC**: Password manager with URI integration (https://keepassxc.org/)
- **Google Chrome**: For enhanced proxy tunnel support
- **wl-clipboard**: For Wayland clipboard support

## Installation

### Linux Installation

```bash
# Create local bin directory
mkdir -p ~/.local/bin

# Download izOpen
wget https://raw.githubusercontent.com/ugoviti/izopen/master/izopen -O ~/.local/bin/izopen

# Make executable
chmod 755 ~/.local/bin/izopen

# Add to PATH (if not already present)
echo 'PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc

# Reload shell
source ~/.bashrc
```

### First Run Configuration

On first run, izOpen creates a default configuration file at:
```
~/.config/izopen/izopen.conf
```

To regenerate the config file after upgrades:
```bash
izopen --mkconfig
```

## Desktop Integration

### MIME Type Registration

Register izOpen as the default handler for remote protocols:

```bash
# Create applications directory
mkdir -p ~/.local/share/applications/

# Create desktop files
cat > ~/.local/share/applications/izopen-ssh-handler.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Name=izOpen SSH Handler
Comment=Open SSH connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=utilities-terminal
Type=Application
Categories=Network;
MimeType=x-scheme-handler/ssh;
EOF

cat > ~/.local/share/applications/izopen-rdp-handler.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Name=izOpen RDP Handler
Comment=Open RDP connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=preferences-desktop-remote-desktop
Type=Application
Categories=Network;
MimeType=x-scheme-handler/rdp;
EOF

cat > ~/.local/share/applications/izopen-vnc-handler.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Name=izOpen VNC Handler
Comment=Open VNC connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=vinagre
Type=Application
Categories=Network;
MimeType=x-scheme-handler/vnc;
EOF

cat > ~/.local/share/applications/izopen-telnet-handler.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Name=izOpen TELNET Handler
Comment=Open TELNET connection to remote URI
Exec=izopen %U
StartupNotify=false
Terminal=false
Icon=utilities-terminal
Type=Application
Categories=Network;
MimeType=x-scheme-handler/telnet;
EOF

# Register MIME handlers
xdg-mime default izopen-ssh-handler.desktop x-scheme-handler/ssh
xdg-mime default izopen-ssh-handler.desktop x-scheme-handler/sftp
xdg-mime default izopen-rdp-handler.desktop x-scheme-handler/rdp
xdg-mime default izopen-vnc-handler.desktop x-scheme-handler/vnc
xdg-mime default izopen-telnet-handler.desktop x-scheme-handler/telnet

# Rebuild MIME database
update-desktop-database ~/.local/share/applications/
update-mime-database ~/.local/share/mime/
```

### Verify Registration

```bash
xdg-mime query default x-scheme-handler/ssh
xdg-mime query default x-scheme-handler/rdp
xdg-mime query default x-scheme-handler/vnc
xdg-mime query default x-scheme-handler/telnet
```

### Browser Launcher (Optional)

Create a desktop launcher for opening browsers through SSH tunnels:

```bash
cat > ~/.local/share/applications/izopen-browser.desktop <<'EOF'
#!/usr/bin/env xdg-open
[Desktop Entry]
Version=1.0
Name=izOpen Web Browser
Comment=Browse the Web through SSH tunnel SOCKS proxy
Exec=izopen -f
StartupNotify=false
Terminal=false
Type=Application
Icon=applications-internet
Categories=GTK;Network;WebBrowser;
MimeType=text/html;x-scheme-handler/http;x-scheme-handler/https;
EOF

chmod +x ~/.local/share/applications/izopen-browser.desktop
```

Add this launcher to your desktop bar or application menu.

## Configuration

### Default Client Selection

Edit `~/.config/izopen/izopen.conf` to change default clients:

```bash
# RDP Clients
helper_cmd[rdp]="remmina"     # Default: Remmina GUI client
#helper_cmd[rdp]="xfreerdp"   # FreeRDP 3.x CLI
#helper_cmd[rdp]="krdc"       # KDE Remote Desktop Client
#helper_cmd[rdp]="rdesktop"   # Legacy rdesktop

# VNC Clients
helper_cmd[vnc]="remmina"     # Default: Remmina GUI client
#helper_cmd[vnc]="krdc"       # KDE Remote Desktop Client
#helper_cmd[vnc]="vncviewer"  # TigerVNC CLI

# Web Browser
helper_cmd[http]="google-chrome"
#helper_cmd[http]="firefox"

# Terminal Emulator
helper_cmd[terminal]="konsole"
#helper_cmd[terminal]="gnome-terminal"
```

### Client-Specific Options

#### xfreerdp Options
```bash
helper_opts[rdp]="/drive:/tmp /kbd:0x00020409 /sound:sys:pulse"
```

#### KRDC Options
```bash
helper_opts[rdp]="--fullscreen"
helper_opts[vnc]="--fullscreen"
```

#### Remmina Configuration
```bash
helper_conf[rdp]="drive=/tmp window_width=1920 window_height=1080 tls-seclevel=0"
helper_conf[vnc]="window_maximize=1 ignore-tls-errors=1"
```

### Security Settings

⚠️ **Warning**: By default, SSH host key checking is disabled for sshpass compatibility:
```bash
helper_opts[ssh]="-o StrictHostKeyChecking=no -o CheckHostIP=no"
```

To disable this (more secure but requires manual host key acceptance):
```bash
# In ~/.config/izopen/izopen.conf
helper_opts[ssh]=""
```

## Usage Examples

### Basic Usage

```bash
# Display help
izopen -h

# Enable debug mode
izopen -d <uri>
```

### SSH Connections

```bash
# Basic SSH
izopen ssh://user@example.com

# SSH with port
izopen ssh://user@example.com:2222

# SSH with password in URI
izopen ssh://user:password@example.com

# SSH with tunnel creation (opens browser with SOCKS proxy)
izopen -f ssh://user@example.com
```

### RDP Connections

```bash
# Basic RDP
izopen rdp://administrator@10.1.1.100

# RDP with domain
izopen rdp://DOMAIN/administrator@10.1.1.100

# RDP with password
izopen rdp://administrator:P@ssw0rd!@10.1.1.100

# RDP with port
izopen rdp://administrator@10.1.1.100:3390

# RDP through SSH tunnel (using xfreerdp)
izopen -t rdp://administrator@10.1.1.100
```

### VNC Connections

```bash
# Basic VNC
izopen vnc://10.1.1.100

# VNC with password
izopen vnc://password@10.1.1.100:5901

# VNC with port
izopen vnc://10.1.1.100:5902
```

### Other Protocols

```bash
# Telnet
izopen telnet://192.168.1.1

# FTP
izopen ftp://user:password@ftp.example.com

# SFTP (uses SSH)
izopen sftp://user@example.com

# HTTP through tunnel
izopen -f http://internal.site.local

# SMB/CIFS
izopen smb://server/share
```

### URL Format Specifications

#### SSH/Telnet/SFTP
```
ssh://[username[:password]@]host[:port]
telnet://host[:port]
sftp://[username[:password]@]host[:port]
```

#### RDP
```
rdp://[DOMAIN/username[:password]@]host[:port]
rdp://[username@DOMAIN[:password]@]host[:port]
```

**Supported formats:**
- `rdp://host` - Anonymous connection
- `rdp://user@host` - Username only
- `rdp://user:pass@host` - Username and password
- `rdp://DOMAIN/user:pass@host` - With domain (format 1)
- `rdp://user@DOMAIN:pass@host` - With domain (format 2)
- `rdp://user:pass@host:3390` - Custom port

#### VNC
```
vnc://[password@]host[:port]
```

#### HTTP/HTTPS
```
http://url
https://url
```

### Password Manager Integration

#### KeePassXC Configuration

1. Open KeePassXC Settings → Browser Integration
2. Enable "Browser Integration"
3. For custom URI handling, use izOpen as the browser:
```
Command: izopen
Arguments: %1
```

#### Example Entry URLs in KeePassXC
```
ssh://admin@server.example.com
rdp://WORKGROUP/administrator:P@ssw0rd@10.1.1.50
vnc://secretpass@192.168.1.100:5901
```

### Tunnel Management

#### Create SSH Tunnel with Browser
```bash
# Opens SSH tunnel and launches browser with SOCKS proxy
izopen -f ssh://user@jumphost.example.com

# The browser will be configured to route all traffic through the tunnel
# Default SOCKS proxy: 127.0.0.1:dynamic_port
```

#### Use Existing Tunnel
```bash
# Connect to RDP through existing tunnel
izopen -t rdp://10.1.1.100

# izOpen will prompt for tunnel port or use last used port
```

#### Tunnel Persistence
Tunnel ports are saved per-host in:
```
~/.config/izopen/hosts/<sanitized_hostname>.conf
```

## Windows Installation (W.I.P)

izOpen can run on Windows using WSL or Cygwin.

### Method 1: WSL (Windows Subsystem for Linux) - Recommended

#### Prerequisites
1. Verify virtualization is enabled in BIOS
2. Check with PowerShell: `Systeminfo.exe`

#### Installation Steps

```powershell
# Open PowerShell as Administrator

# Enable WSL
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux

# Reboot system
Restart-Computer

# After reboot, update WSL kernel
# Download: https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

# Set WSL 2 as default
wsl --set-default-version 2

# Install Ubuntu from Microsoft Store
# https://apps.microsoft.com/store/detail/ubuntu-on-windows/9NBLGGH4MSV6
```

#### Continue with Linux Installation
Once Ubuntu is installed, follow the standard Linux installation steps above.

### Method 2: Cygwin

#### Prerequisites

1. **Install Chocolatey Package Manager:**
```powershell
# Open PowerShell as Administrator
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

More info: https://chocolatey.org/install

2. **Install Cygwin:**
```powershell
choco install cygwin cyg-get -y
```

3. **Install Required Packages:**
```bash
# Open Cygwin64 Terminal
cyg-get.bat vim nc sshpass yad freerdp xorg-server xinit
```

4. **Configure Cygwin Environment:**
```bash
echo 'export DISPLAY=:0' >> ~/.bashrc
source ~/.bashrc
```

5. **Start XWin Server** before using izOpen (required for GUI clients)

6. **Install Google Chrome** (recommended):
https://www.google.com/chrome

7. **Continue with izOpen installation** using Linux steps

#### Windows Registry Integration (Work in Progress)

**Note:** Direct URI handling from Windows is not fully functional yet.

Example registry file (`izopen.reg`):
```reg
REGEDIT4
[HKEY_CLASSES_ROOT\ssh]
@="izOpen SSH Handler"
"URL Protocol"=""

[HKEY_CLASSES_ROOT\ssh\shell]
[HKEY_CLASSES_ROOT\ssh\shell\open]
[HKEY_CLASSES_ROOT\ssh\shell\open\command]
@="\"c:\\tools\\cygwin\\bin\\bash\" -l -i -c \"izopen \"%1\"\""
```

#### Test Commands
```cmd
C:\tools\cygwin\bin\bash -l -i -c "izopen ssh://root@10.1.1.8 -d"
C:\tools\cygwin\bin\bash.exe --login -i -c 'izopen -d ssh://root@10.1.1.8'
```

## Troubleshooting

### Debug Mode
Enable debug output to see parsed URI components and commands:
```bash
izopen -d rdp://user@host
```

Debug output shows:
- Parsed URI components (schema, username, password, host, port)
- Selected helper application
- Tunnel configuration
- Final command being executed

### Common Issues

**Problem:** Password with special characters fails authentication

**Solution:**
- **xfreerdp 3.x**: No escaping needed, passwords work as-is
- **krdc**: Automatic URL encoding (% → %25, ! → %21, @ → %40)
- **remmina**: Passwords stored in config file without encoding

**Problem:** SSH tunnel not working

**Solution:**
```bash
# Check if tunnel is active
ps aux | grep ssh | grep "\-D"

# Check proxychains configuration
cat ~/.cache/izopen/proxychains.conf

# Verify port
cat ~/.cache/izopen/tunnel_port_current.conf
```

**Problem:** MIME handler not registered

**Solution:**
```bash
# Rebuild databases
update-desktop-database ~/.local/share/applications/
update-mime-database ~/.local/share/mime/

# Verify registration
xdg-mime query default x-scheme-handler/ssh
```

## Files and Directories

```
~/.local/bin/izopen                          # Main script
~/.config/izopen/izopen.conf                 # User configuration
~/.config/izopen/izopen.log                  # Connection log
~/.config/izopen/hosts/<host>.conf           # Per-host tunnel ports
~/.config/izopen/google-chrome/<port>/       # Per-tunnel Chrome profiles
~/.cache/izopen/proxychains.conf             # Generated proxychains config
~/.cache/izopen/tunnel_port_current.conf     # Last used tunnel port
~/.cache/izopen/izopen.remmina               # Cached remmina RDP config
```

## Contributing

Contributions are welcome! Please submit pull requests or open issues at:
https://github.com/ugoviti/izopen

## License

Written by Ugo Viti <u.viti@wearequantico.it>
