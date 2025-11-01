# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

izOpen is a multi-protocol bash script that functions as a secure URI launcher, primarily designed to work with password managers like KeePassXC. It supports the following URI schemas: ssh, rdp, vnc, sftp, ftp, http, https, smb, and cifs.

The key feature is automatic SSH reverse tunnel SOCKS proxy creation and management, allowing secure connections through SSH tunnels.

## Development Commands

**Testing the script:**
```bash
# Basic URI open
./izopen ssh://user@host:22

# Open with debug output
./izopen -d ssh://user@host:22

# Force tunnel usage
./izopen -f http://example.com

# Use specific tunnel port
./izopen -p 2069 http://example.com

# Show help
./izopen -h

# Generate default config file
./izopen --mkconfig
```

**Testing specific protocol handlers:**
```bash
./izopen rdp://DOMAIN/user:password@host:3389
./izopen vnc://password@host:5902
./izopen http://user:password@example.com
./izopen smb://user:password@server/share
```

## Code Architecture

### Single File Structure
The entire application is a single bash script ([izopen](izopen)) with approximately 1250 lines. There are no separate modules or build steps.

### Configuration Management
- **System config**: Lines 143-186 define environment-specific variables based on OS detection
- **User config**: `$HOME/.config/izopen/izopen.conf` (auto-created on first run)
- **Tunnel ports**: Stored per-host in `$HOME/.cache/izopen/` and `$HOME/.config/izopen/hosts/`
- **Remmina profiles**: Generated dynamically in `$HOME/.config/izopen/remmina/`

### Core Components

**1. Environment Detection (lines 38-186)**
- `env_os_linux()` and `env_os_windows()` set OS-specific defaults
- Helper command arrays (`helper_cmd`, `helper_opts`, `helper_conf`, `helper_name`) configure protocol handlers
- Automatic terminal emulator detection based on desktop environment (KDE/GNOME/MATE/XFCE)
- Clipboard support for both X11 (xsel) and Wayland (wl-clipboard)

**2. URI Parser (lines 942-994)**
- Regex-based parser extracting: schema, username, password, host, port, path, query, fragment
- Pattern: `[schema://][user[:password]@]host[:port][/path][?args][#fragment]`
- Function: `uri_parser()` creates global variables with `uri_` prefix

**3. SSH Tunnel Management (lines 790-893)**
- `helper_tunnel_create()`: Creates new SSH SOCKS proxy tunnels
- `helper_tunnel_use()`: Connects to existing tunnels
- `find_available_port()`: Finds free ports in range (default: 2000-65535)
- Per-host tunnel port persistence in `~/.config/izopen/hosts/<sanitized_host>.conf`
- Generates proxychains4 config dynamically at `~/.cache/izopen/proxychains.conf`

**4. Protocol Handlers (lines 261-800)**
Each protocol has a dedicated helper function:
- `helper_ssh()`: Builds SSH command with sshpass for password auth
- `helper_rdp()`: Supports rdesktop, xfreerdp, remmina, and krdc with dynamic config generation
- `helper_vnc()`: Supports vncviewer, remmina, and krdc
- `helper_http()`: Browser-specific options (Chrome/Midori/xdg-open) with proxy settings
- `helper_telnet()`, `helper_sftp()`, `helper_ftp()`, `helper_smb()`, `helper_cifs()`

All helpers populate the `izopen_run` array (bash indexed array) with command and arguments.

**5. Terminal Integration (lines 261-295)**
- `helper_terminal()`: Launches protocol handlers in terminal emulator when `terminal_use=1`
- Terminal-specific option handling (konsole, mintty, generic)
- Tab title customization shows host and tunnel port

### Important Variables

**Global Arrays:**
- `helper_name[]`: Display name of helper command (associative array)
- `helper_cmd[]`: Actual command to execute (associative array)
- `helper_opts[]`: Command-line options (associative array)
- `helper_conf[]`: Protocol-specific config (used for Remmina, associative array)
- `izopen_run=()`: Final command array to execute (indexed array)

**Tunnel Control:**
- `tunnel_create`: Whether to create a new SSH tunnel (default: 1)
- `tunnel_use`: Whether to use existing tunnel (default: 1)
- `tunnel_use_force`: Force tunnel for all URIs (default: 0)
- `tunnel_port_min/max`: Port range (default: 2000-65535)

**Security:**
- Password stored in `$SSHPASS` environment variable (exported only when needed)
- Clipboard integration attempts to clear password after reading
- `uri_safe` variable created for logging (password removed)

### Execution Flow

1. Parse command-line arguments (lines 1190-1235)
2. Import user config from `~/.config/izopen/izopen.conf`
3. Call `uri_open()` (lines 1011-1112):
   - Parse URI with `uri_parser()`
   - Attempt password extraction from clipboard if not in URI
   - Call `helper_tunnel_manage()` to setup/connect tunnel
   - Call protocol-specific `helper_<schema>()` function
   - Execute command array `"${izopen_run[@]}"`
   - Log output to `~/.config/izopen/izopen.log`

### Special Behaviors

**Password Handling:**
- URI embedded passwords take priority
- If password missing and `clipboard_use=1`, reads from clipboard
- Clipboard automatically cleared after read (security feature)
- For RDP/VNC: if credentials incomplete, prompts with YAD dialog
- **Special characters in passwords**:
  - **xfreerdp 3.x**: No escaping needed (line 551 commented). Bash arrays handle special chars correctly
  - **krdc**: URL encoding required (line 650). Password `D%m41n4dm` → `D%25m41n4dm` in URL
  - **rdesktop/vncviewer**: No escaping (lines 526, 646 commented)

**Domain Parsing (RDP):**
- Supports `DOMAIN/username` format
- Supports `username@DOMAIN` format
- Extracts and passes to RDP client appropriately

**Remmina Config Generation:**
- Creates `.remmina` files on-the-fly at `~/.config/izopen/remmina/<host>_<schema>.remmina`
- Includes tunnel proxy settings when `tunnel_use=1`
- Helper config from `helper_conf[rdp]` merged into profile

**KRDC (KDE Remote Desktop Client):**
- Supports both RDP and VNC protocols via URL format
- RDP URL format: `rdp://[DOMAIN\username[:password]@]host[:port]`
- VNC URL format: `vnc://[password@]host[:port]`
- No config file needed, all parameters passed via URL
- Supports `--fullscreen` option for fullscreen mode
- **IMPORTANT**: krdc REQUIRES URL encoding for passwords (unlike xfreerdp). Special characters like `%`, `!`, `@` are automatically encoded (`%` → `%25`, `!` → `%21`, etc.)

## Testing & Debugging

**Enable debug mode:**
```bash
./izopen -d <uri>
```
Debug output (lines 911-931) shows all parsed URI components, tunnel settings, and final command.

**View connection logs:**
```bash
tail -f ~/.config/izopen/izopen.log
```

**Check tunnel ports:**
```bash
ls -la ~/.config/izopen/hosts/
cat ~/.cache/izopen/tunnelport.conf  # Last used port
```

**Test tunnel creation:**
```bash
# Creates tunnel on first SSH connection
./izopen ssh://user@host

# Check if tunnel is active
nc -z 127.0.0.1 <port>
```

## Common Modifications

**Adding a new protocol handler:**
1. Add to supported schemas in case statement at line 1066
2. Create `helper_<schema>()` function following existing pattern
3. Populate `izopen_run` array with command and arguments
4. Add default command in `env_os_linux()` or `env_os_windows()`

**Changing default applications:**
Edit `~/.config/izopen/izopen.conf` (or set before sourcing):
```bash
# RDP clients
helper_cmd[rdp]="xfreerdp"    # FreeRDP 3.x (default for CLI)
helper_cmd[rdp]="remmina"     # Remmina (default for GUI)
helper_cmd[rdp]="krdc"        # KDE Remote Desktop Client
helper_cmd[rdp]="rdesktop"    # Legacy rdesktop

# VNC clients
helper_cmd[vnc]="remmina"     # Remmina (default for GUI)
helper_cmd[vnc]="krdc"        # KDE Remote Desktop Client
helper_cmd[vnc]="vncviewer"   # TigerVNC viewer

# Other
helper_cmd[http]="firefox"
helper_cmd[terminal]="alacritty"
```

**Modifying RDP/VNC defaults:**
Use `helper_conf[]` for Remmina, `helper_opts[]` for command-line clients and krdc:
```bash
# Remmina specific config
helper_conf[rdp]="window_width=1920 window_height=1080"

# xfreerdp, rdesktop, krdc options
helper_opts[rdp]="-g 1920x1080"           # rdesktop
helper_opts[rdp]="/drive:/tmp /kbd:..."  # xfreerdp
helper_opts[rdp]="--fullscreen"          # krdc
```

## Security Considerations

- SSH option `-o StrictHostKeyChecking=no` is enabled by default (security risk)
- Passwords logged to `~/.config/izopen/izopen.log` are sanitized (removed from URIs)
- `SSHPASS` variable cleared on script exit via trap (line 1250)
- **Special characters in passwords**:
  - **xfreerdp 3.x**: Password escaping disabled (line 551 commented). Passwords passed as-is without encoding
  - **krdc**: Requires URL encoding (line 650 active). Special chars like `%` → `%25`, `!` → `%21`, `@` → `%40`
  - **rdesktop/vncviewer**: Password escaping disabled (lines 526, 646 commented)

## Platform Support

- **Linux**: Full support (primary platform)
- **Windows (Cygwin/MSYS2)**: Partial support, some features disabled (sshpass, clipboard)
- **macOS/BSD/Solaris**: Failback to Linux environment settings (untested)

## Integration Points

**KeePassXC Integration:**
- Set custom browser command: `izopen %1`
- URI format in entry: `schema://{USERNAME}:{PASSWORD}@host:port`
- Placeholders `{USERNAME}` and `{PASSWORD}` are replaced by KeePassXC

**XDG Desktop Integration:**
See README.md for MIME handler registration to make izopen the default handler for ssh://, rdp://, vnc:// URIs system-wide.
