# pm_helper

`pm_helper` is a POSIX-compliant shell script designed to simplify and automate the process of developing and testing [PortMaster](https://github.com/PortsMaster/PortMaster) ports. It allows you to manage installations, updates, and file transfers across multiple handheld devices running different custom firmware (CFW) with single, simple commands.

## Features

-   **Multi-Device Management**: Run commands on multiple devices simultaneously.
-   **Host Alias System**: Define simple, memorable aliases for your devices so you don't have to remember IP addresses.
-   **CLI Host Management**: Add, update, and delete host aliases directly from the command line.
-   **Target All Devices**: Use the special `ALL` keyword to run commands on every configured device at once.
-   **Simplified Commands**: Wraps common `ssh`, `scp`, and `rsync` tasks into easy-to-use commands (`install`, `update`, `scp`).
-   **CFW Aware**: Knows the correct file paths for various popular custom firmwares (ArkOS, AmberELEC, muOS, JELOS, etc.).
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

`pm_helper` uses a simple configuration file named `pm_devices.cfg` to define your devices. You can manage this file manually or by using the `host_add` and `host_del` commands.

The script checks for this file in two locations:
1.  `$HOME/.config/pm_devices.cfg` (Global configuration)
2.  `./pm_devices.cfg` (Local, project-specific configuration)

**Definitions in the local file will override those in the global file.**

### `pm_devices.cfg` Format

The file uses a simple `alias = CFW,IP_Address` format.

```ini
# This is a comment. Lines starting with # are ignored.

# <alias> = <CFW>,<IP_or_Hostname>
ark353v   = ArkOS2,192.168.0.110
amber351  = AmberELEC,192.168.0.111
mucube    = muOS,192.168.0.112
knulli-dev = Knulli,knulli.local
```

### Supported CFW Names

Use one of the following exact strings for the `<CFW>` value in your config file:

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

### The `<hosts>` String

Most commands use a `<hosts>` argument. This is a colon-separated (`:`) string that can contain any combination of:
-   **An alias** from your `pm_devices.cfg` file (e.g., `ark353v`).
-   **An explicit definition** in the format `CFW,IP` (e.g., `muOS,192.168.0.99`).
-   The special keyword **`ALL`**, which targets every host alias defined in your config files.

**Example `<hosts>` string:** `ark353v:amber351:muOS,192.168.0.99`

---

### `install`
Installs a port `.zip` file onto one or more devices. It copies the zip to the home directory and then runs `harbourmaster` to install it.

**Syntax:**
```sh
pm_helper install <hosts> <zip_file>
```
**Examples:**
```sh
# Install a port on a single device using an alias
pm_helper install ark353v ./MyGame.zip

# Install a port on ALL configured devices
pm_helper install ALL ./AnotherGame.zip
```

---

### `update`
Updates an existing port by syncing a new launch script and data directory. This uses `rsync` for efficient transfers.

**Syntax:**
```sh
pm_helper update <hosts> <port_script.sh> <port_directory/>
```
**Example:**
```sh
# Update a script and its data folder on two devices
pm_helper update amber351:ark353v "Awesome Game.sh" awesomegamedata/
```

---

### `scp`
A wrapper for `scp` to copy files or directories to a specific, predefined location on your devices.

**Syntax:**
```sh
pm_helper scp <hosts> <destination> <file1> [file2...]
```
**Arguments:**
- `<destination>`: A remote path that **must** start with one of the following keywords:
  - `PORT_SCRIPT`: The device's scripts folder (e.g., `/roms/ports`).
  - `PORT_DATA`: The device's ports data folder (e.g., `/roms/ports`).
  - `PORTMASTER`: The `PortMaster` directory itself.
  - `HOME`: The user's home directory (`~`).
- `<file(s)>`: One or more local files or directories to copy.

**Example:**
```sh
# Copy GOG installer files to a specific data folder on all devices
pm_helper scp ALL PORT_DATA/gamename/ gog_installer-*.bin
```

---

### `harbourmaster`
Executes a raw command directly with the `harbourmaster` executable on remote devices.

**Syntax:**
```sh
pm_helper harbourmaster <hosts> <command_and_args>
```
**Example:**
```sh
# Check the status of a runtime file on a device
pm_helper harbourmaster mucube runtime_check "mygame_0.2.squashfs"

**Syntax:**
```sh
pm_helper host_add [-g] <alias> <cfw> <ip_address>
```
**Arguments:**
- `-g`: Use this flag to modify the **global** config file (`~/.config/pm_devices.cfg`). If omitted, the local file (`./pm_devices.cfg`) is used.

**Examples:**
```sh
# Add a new device to the local config
pm_helper host_add rg353v ArkOS2 192.168.1.123

# Add a device to the global config
pm_helper host_add -g odin2 ROCKNIX 192.168.1.124
```

---

### `host_del`
Deletes a host alias from your configuration file.

**Syntax:**
```sh
pm_helper host_del [-g] <alias>
```
**Arguments:**
- `-g`: Use this flag to modify the **global** config file.

**Example:**
```sh
# Remove a device from the local config
pm_helper host_del rg353v
```

---

### `hosts`
Displays all configured device aliases from both global and local `pm_devices.cfg` files.

**Syntax:**
```sh
pm_helper hosts
```

---

### `ssh`
Opens an interactive SSH session to a **single** device. The `ALL` keyword is not supported.

**Syntax:**
```sh
pm_helper ssh <single_host>
```
**Example:**
```sh
pm_helper ssh odin2
```

---

### `help`
Shows the main help message or help for a specific command.

**Syntax:**
```sh
pm_helper help
pm_helper <command> --help
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE.md) file for details.
