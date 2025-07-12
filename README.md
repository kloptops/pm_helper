# pm_helper

`pm_helper` is a POSIX-compliant shell script designed to simplify and automate the process of developing and testing [PortMaster](https://github.com/PortsMaster/PortMaster) ports. It allows you to manage installations, updates, file transfers, and device configurations across multiple handheld devices running different custom firmware (CFW) with single, simple commands.

## Features

-   **Multi-Device Management**: Run commands on multiple devices simultaneously.
-   **Host Alias System**: Define simple, memorable aliases for your devices so you don't have to remember IP addresses.
-   **CLI Host Management**: Add, update, and delete host aliases directly from the command line.
-   **Target All Devices**: Use the special `ALL` keyword to run commands on every configured device at once.
-   **Alias Scoping**: Use flags (`-l`, `-g`) to specify whether to use local, global, or combined host configurations.
-   **Extensible CFW Support**: Add definitions for your own custom firmware via a local config file.
-   **Simplified Commands**: Wraps common `ssh`, `scp`, and `rsync` tasks into easy-to-use commands (`install`, `update`, `scp`).
-   **Portable**: Written in POSIX-compliant `sh` for maximum compatibility across systems (including macOS).

## Prerequisites

-   A POSIX-compliant shell (e.g., `bash`, `zsh`, `sh`).
-   Standard command-line utilities: `ssh`, `scp`, `rsync`.
-   SSH key-based authentication to your devices is highly recommended to avoid typing passwords for every operation.

## Installation

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/your-username/pm_helper.git
    cd pm_helper
    ```

2.  **Make the script executable:**
    ```sh
    chmod +x pm_helper
    ```

3.  **Place it in your PATH (Recommended):**
    For easy access from any directory, move the script to a location in your system's PATH.
    ```sh
    sudo mv pm_helper /usr/local/bin/
    ```

## Configuration

### Host Aliases (`pm_devices.cfg`)

`pm_helper` uses a configuration file named `pm_devices.cfg` to define your devices. You can manage this file manually or by using the `host_add` and `host_del` commands.

The script checks for this file in two locations:
1.  `$HOME/.config/pm_devices.cfg` (Global configuration for all projects)
2.  `./pm_devices.cfg` (Local, project-specific configuration)

By default, both are loaded, with local definitions overriding global ones. You can change this behavior with the `-l` and `-g` flags (see Command Usage).

**Format:**
```ini
# This is a comment.

# <alias> = <CFW>,<IP_or_Hostname>
ark353v   = ArkOS2,192.168.0.110
amber351  = AmberELEC,192.168.0.111
```

### Advanced Configuration: Custom CFWs

You can add or override CFW definitions by creating a file at `$HOME/.config/pm_cfw.cfg`. This file is sourced by the script, allowing you to define new `CFW_` variables or add to the `KNOWN_CFWS` list.

**Example `~/.config/pm_cfw.cfg`:**
```sh
# Add a new, custom CFW to the list of known firmwares
KNOWN_CFWS="$KNOWN_CFWS MyCustomOS"

# Define its variables
CFW_MyCustomOS_USERNAME="user"
CFW_MyCustomOS_PORTMASTER="/opt/PortMaster"
CFW_MyCustomOS_PORT_DATA="/opt/ports"
CFW_MyCustomOS_PORT_SCRIPT="/opt/ports"
```

### Built-in Supported CFWs

- `AmberELEC`
- `ArkOS`
- `ArkOS2` - 2 sdcard install
- `Batocera`
- `JELOS`
- `Knulli`
- `muOS`
- `muOS2` - 2 sdcard install
- `RetroDECK`
- `ROCKNIX`
- `UnofficialOS`

---

## Command Usage

**Syntax:** `pm_helper [-l|-g] <command> [options...]`

### Global Scope Flags (Optional)

You can specify a scope flag **before** the command to control which `pm_devices.cfg` file is used for resolving aliases.

-   `-l`: **Local Only**. Uses only `./pm_devices.cfg`.
-   `-g`: **Global Only**. Uses only `~/.config/pm_devices.cfg`.
-   **(No Flag)**: **Default**. Loads global, then local (local overrides global).

### The `<hosts>` String

Most commands use a `<hosts>` argument. This is a colon-separated (`:`) string that can contain any combination of:
-   **An alias** from your loaded `pm_devices.cfg` file(s) (e.g., `ark353v`).
-   **An explicit definition** in the format `CFW,IP` (e.g., `muOS,192.168.0.99`).
-   The special keyword **`ALL`**, which targets every host alias defined in the loaded configuration scope.

---

### Available Commands

#### `install`
Installs a port `.zip` file onto one or more devices.

**Syntax:** `pm_helper install <hosts> <zip_file>`
```sh
# Install a port on a single device using an alias
pm_helper install ark353v ./MyGame.zip

# Install a port on ALL configured devices (using the default config scope)
pm_helper install ALL ./AnotherGame.zip

# Install a port on devices defined ONLY in the local config
pm_helper -l install my-local-device ./ProjectSpecificPort.zip
```

#### `update`
Updates an existing port by syncing a new launch script and data directory.

**Syntax:** `pm_helper update <hosts> <port_script.sh> <port_directory/>`
```sh
# Update a script and its data folder on two devices
pm_helper update amber351:ark353v "Awesome Game.sh" awesomegamedata/
```

#### `scp`
A wrapper for `scp` to copy files or directories to a specific, predefined location.

**Syntax:** `pm_helper scp <hosts> <destination> <file1> [file2...]`
- `<destination>`: Must start with `PORT_SCRIPT`, `PORT_DATA`, `PORTMASTER`, or `HOME`.
```sh
# Copy GOG installer files to a specific data folder on all devices
pm_helper scp ALL PORT_DATA/gamename/ gog_installer-*.bin
```

#### `harbourmaster`
Executes a raw command directly with the `harbourmaster` executable.

**Syntax:** `pm_helper harbourmaster <hosts> <command_and_args>`
```sh
# List all installed ports on all devices
pm_helper harbourmaster ALL list
```

#### `host_add`
Adds a new host alias or updates an existing one.

**Syntax:** `pm_helper host_add [-g] <alias> <cfw> <ip_address>`
- `-g`: Modifies the **global** config file (`~/.config/pm_devices.cfg`). If omitted, the local file (`./pm_devices.cfg`) is used.
```sh
# Add a new device to the local config
pm_helper host_add rg353v ArkOS2 192.168.1.123

# Add a device to the global config
pm_helper host_add -g odin2 AmberELEC 192.168.1.124
```

#### `host_del`
Deletes a host alias from a configuration file.

**Syntax:** `pm_helper host_del [-g] <alias>`
- `-g`: Modifies the **global** config file.
```sh
# Remove a device from the local config
pm_helper host_del rg353v
```

#### `hosts`
Displays all configured device aliases from the loaded configuration scope.

**Syntax:** `pm_helper hosts`
```sh
# Show default (global + local) aliases
pm_helper hosts

# Show only aliases from the global config
pm_helper -g hosts
```

#### `cfws`
Lists all supported CFWs (including custom ones) and their defined path variables.

**Syntax:** `pm_helper cfws`

#### `ssh`
Opens an interactive SSH session to a **single** device. The `ALL` keyword is not supported.

**Syntax:** `pm_helper ssh <single_host>`

#### `help`
Shows the main help message or help for a specific command.

**Syntax:** `pm_helper help` or `pm_helper <command> --help`

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE.md) file for details.
