# Changelog

All notable changes to izOpen will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [3.1.1] - 2025-11-02

### Added
- **Remmina keyboard transparency**: Added `rdp_use_client_keymap=1` to Remmina RDP configuration
  - Enables transparent keyboard layout mapping for Remmina connections
  - Users can now use their local keyboard layout without server-side layout conflicts
  - Works seamlessly when client and server have different keyboard layouts
- **Dialog tool configuration**: Added configurable `dialog_tool` option with auto-detection
  - Supports zenity, kdialog, and yad
  - Auto-detection priority: zenity > kdialog > yad
  - Improves desktop environment integration (GNOME/KDE/XFCE)

### Changed
- **Remmina config refactoring**: Improved `generate_remmina_config()` function
  - Cleaner `extra_settings` handling with individual option appending
  - Domain configuration moved into RDP-specific `extra_settings`
  - Better separation of protocol-specific options
- **krdc keyboard support**: Removed automatic keyboard layout configuration for krdc
  - krdc does not support transparent keyboard mapping like Remmina/xfreerdp
  - Users should use Remmina (with `rdp_use_client_keymap`) or xfreerdp (with `/kbd:unicode:on`) for transparent keyboard support

### Note
For transparent keyboard layout support:
- **Remmina**: Automatically enabled via `rdp_use_client_keymap=1`
- **xfreerdp**: Configure with `/kbd:unicode:on` in `helper_opts[rdp]`
- **krdc**: Not supported - use Remmina or xfreerdp instead

## [3.1.0] - 2025-11-02

### Added
- **JSON database for tunnel management**: Complete refactoring from legacy .conf files to single JSON database
  - `tunnels.json`: Single source of truth for all tunnel port mappings with metadata
  - Metadata tracking: `created`, `last_used` (ISO 8601 timestamps), `connection_count`
  - CRUD functions: `tunnel_json_init()`, `tunnel_json_get()`, `tunnel_json_set()`
  - `jq_update()`: Atomic JSON updates with automatic backup
  - `find_available_port()`: Intelligent port allocation (finds first gap or next sequential)
  - `migrate_legacy_tunnels()`: Automatic migration from 1000+ .conf files to JSON (one-time, non-destructive)
  - Progress tracking during migration with statistics
- **KRDC support**: KDE Remote Desktop Client now supported for both RDP and VNC protocols
  - RDP URL format: `rdp://[DOMAIN\username[:password]@]host[:port]`
  - VNC URL format: `vnc://[password@]host[:port]`
  - Automatic URL encoding for passwords (required by krdc unlike xfreerdp)
- **Wayland-native dialogs**: Zenity and kdialog support with automatic desktop environment detection
  - `detect_dialog_tool()`: Auto-selects zenity (default), kdialog, or yad
  - Dialog wrapper functions: `dialog_form_rdp()`, `dialog_form_vnc()`, `dialog_entry_port()`, `dialog_text_info()`
- **Version option**: `-v/--version` flag to display version information
- **CLAUDE.md**: Comprehensive documentation for Claude Code AI assistant integration
- **Security documentation**: 40+ line SECURITY CONSIDERATIONS section covering:
  - SSHPASS environment variable exposure
  - StrictHostKeyChecking risks
  - Remmina plaintext passwords
  - Password logging (sanitized)
  - Clipboard handling
- **Shared helper infrastructure**: New reusable functions reducing code duplication by ~300 lines
  - `init_helper_connection()`: Common initialization for all protocol helpers
  - `add_helper_opts()`: Unified option parsing
  - `wrap_with_terminal()`: Reusable terminal wrapper
  - `generate_remmina_config()`: Unified Remmina config generation for RDP/VNC
- **URI helper functions**:
  - `get_password_from_clipboard()`: Extract and clear clipboard password
  - `sanitize_uri_for_logging()`: Remove passwords from logged URIs
  - `is_valid_tunnel_port()`: Validate tunnel port range
- **Function documentation**: Comprehensive headers for all major functions with usage examples

### Fixed
- **Password escaping**: Removed incorrect escaping for xfreerdp 3.x, rdesktop, and vncviewer
  - Bash arrays handle special characters correctly without escaping
  - Only krdc requires URL encoding (% → %25, ! → %21, etc.)
- **Proxychains port bug**: Configuration now regenerates with correct port on every connection
  - Moved `make_proxychains_conf` from `helper_tunnel_create` to `helper_tunnel_manage`
  - Uses `$tunnel_port_use` (actual selected port) instead of `$tunnel_port_create`
  - Fixes issue where specifying old tunnel port would use wrong proxy configuration
- **Command execution**: `helper_prepend` (proxychains) now correctly applied before command execution
- **Double parsing**: Fixed `parse_rdp_domain()` using `elif` instead of second `if` statement
- **Debug output**: Now shows complete command array with proper quoting using `printf '%q'`

### Changed
- **Complete refactoring**: All helper functions rewritten using bash arrays instead of string concatenation
  - Eliminates quoting issues and improves security
  - `izopen_connect` array used throughout instead of string building
- **Modular helper functions**:
  - `helper_rdp`: Split from 200 lines to 20 lines (-90%)
    - `parse_rdp_domain()`, `validate_rdp_credentials()`
    - `helper_rdp_rdesktop()`, `helper_rdp_xfreerdp()`, `helper_rdp_remmina()`, `helper_rdp_krdc()`
  - `helper_vnc`: Split from 120 lines to 16 lines (-87%)
    - `validate_vnc_credentials()`
    - `helper_vnc_vncviewer()`, `helper_vnc_remmina()`, `helper_vnc_krdc()`
- **Chrome proxy handling**: Now uses native `--proxy-server` option, skips proxychains wrapper
  - New `helper_prepend_skip` flag to bypass proxychains for apps with native proxy support
  - Chrome tunnel options grouped into `chrome_tunnel_opts` array
- **Credential validation**: Replaced 9 awk process spawns with single `IFS='|' read -r` operations
  - `validate_rdp_credentials()`: 6 awk → 1 IFS read
  - `validate_vnc_credentials()`: 3 awk → 1 IFS read
- **Dialog simplification**: Removed redundant KDE detection from `detect_dialog_tool()`
  - Zenity now default for all environments (Wayland-native)
- **URI sanitization**: Replaced sed with bash parameter expansion in `sanitize_uri_for_logging()`
  - `echo "$uri" | sed ...` → `echo "${uri//:$password/}"`
- **Return early pattern**: Applied in `get_password_from_clipboard()` for cleaner flow
- **Tunnel port configuration refactoring**: Eliminated redundant configuration files
  - Removed `tunnel.conf` (contained only port number, redundant with `proxy.conf`)
  - `proxy.conf` now single source of truth for last used tunnel port
  - Smart fallback system with priority: `proxy.conf` → `tunnels.json` (sorted by last_used) → empty field
  - `helper_tunnel_create()`: Auto-generates `proxy.conf` on SSH tunnel creation
  - `helper_tunnel_use()`: Reads from `proxy.conf` first, falls back to most recently used tunnel from JSON
  - Legacy cleanup: Automatic removal of deprecated `tunnel.conf`, `socks_port_current.conf`
- **Config file locations**:
  - `~/.cache/izopen/tunnelport.conf` → `port.conf` → **removed** (redundant)
  - `~/.cache/izopen/tunnel.conf` → **removed** (replaced by `proxy.conf` + `tunnels.json`)
  - `~/.cache/izopen/proxychains.conf` → `proxy.conf`
  - `~/.cache/izopen/izopen.remmina` → `rdp.remmina` / `vnc.remmina`
  - `~/.config/izopen/hosts/*.conf` → `~/.config/izopen/tunnels.json` (migrated automatically)
- **Remmina config**: Changed from per-host files in `~/.config/izopen/remmina/` to single cached file
- **README**: Complete reorganization for v3.1.0
  - Moved Desktop Integration section before Configuration
  - Added krdc to dependencies
  - Updated installation instructions

### Security
- **File permissions**: Enforced on every execution, not just creation
  - Directories: 700 (user read/write/execute only)
  - Files: 600 (user read/write only)
  - Applied to: `~/.config/izopen/`, `~/.config/izopen/hosts/`, `~/.cache/izopen/`, log file
- **Improved trap**: Changed from signal `0` to `EXIT ERR` for guaranteed password cleanup
  - `trap "unset password SSHPASS" EXIT ERR`
- **Password sanitization**: Always removed from logged URIs via `sanitize_uri_for_logging()`
- **Remmina config security**: Plaintext passwords in `.remmina` files now have 600 permissions

### Performance
- **Reduced process spawns**: Eliminated 9 external process calls
  - 6 awk in `validate_rdp_credentials()` → 1 IFS read
  - 3 awk in `validate_vnc_credentials()` → 1 IFS read
  - 1 sed + 1 printf in `sanitize_uri_for_logging()` → bash parameter expansion
- **Code reduction**: Net reduction of ~50 lines despite adding extensive documentation
  - Eliminated ~300 lines of duplication through DRY principles
  - Added ~250 lines of documentation and new features

### Technical Notes
- **Tunnel port management architecture**:
  - `tunnels.json`: Per-host tunnel mappings with metadata (SSH tunnel hosts only)
  - `proxy.conf`: Last globally used tunnel port (highest priority for dialog pre-fill)
  - Priority system: `proxy.conf` (last actual use) → `tunnels.json` (most recent by timestamp) → empty
  - Migration: Process substitution `< <(find ...)` avoids subshell scope issues
  - Atomic writes: `.tmp` → `mv` → `chmod 600` pattern throughout
- **Password handling by client**:
  - xfreerdp 3.x: NO escaping (bash arrays handle special chars)
  - rdesktop: NO escaping
  - vncviewer: NO escaping
  - krdc: YES URL encoding required (% → %25, ! → %21, @ → %40, etc.)
- **Proxychains bypass**: Chrome uses `--proxy-server`, sets `helper_prepend_skip=1`
- **URI parser**: Documented 13 capture groups with detailed regex breakdown

## [3.0.7] - Previous Release

(Earlier changelog entries to be added)

---

**Legend:**
- Added: New features
- Fixed: Bug fixes
- Changed: Changes in existing functionality
- Security: Security improvements
- Performance: Performance improvements
- Technical Notes: Implementation details
