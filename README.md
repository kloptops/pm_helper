# pm_helper

`pm_helper` is a POSIX-compliant shell script designed to simplify and automate the process of developing and testing [PortMaster](https://github.com/PortsMaster/PortMaster) ports. It allows you to manage installations, updates, and file transfers across multiple handheld devices running different custom firmware (CFW) with single, simple commands.

## Features

-   **Multi-Device Management**: Run commands on multiple devices simultaneously.
-   **Alias System**: Define simple, memorable aliases for your devices so you don't have to remember IP addresses.
-   **Simplified Commands**: Wraps common `ssh`, `scp`, and `rsync` tasks into easy-to-use commands (`install`, `update`, `scp`).
-   **CFW Aware**: Knows the correct file paths for various popular custom firmwares (ArkOS, AmberELEC, muOS, JELOS, etc.).
-   **Direct `harbourmaster` Access**: Pass commands directly to the `harbourmaster` executable on remote devices.
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

`pm_helper` uses a simple configuration file named `pm_devices.cfg` to define your devices and their aliases.

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

-   `muOS`
-   `muOS2`
-   `AmberELEC`
-   `ROCKNIX`
-   `JELOS`
-   `UnofficialOS`
-   `ArkOS`
-   `ArkOS2`
-   `Knulli`
-   `Batocera`

---

## Command Usage

### The `<hosts>` String

Most commands use a `<hosts>` argument. This is a colon-separated (`:`) string that can contain any combination of:
-   **An alias** from your `pm_devices.cfg` file (e.g., `ark353v`).
-   **An explicit definition** in the format `CFW,IP` (e.g., `muOS,192.168.0.99`).

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

# Install on multiple devices, including one defined explicitly
pm_helper install amber351:mucube:ArkOS,192.168.0.120 ./AnotherGame.zip
```

---

### `update`
Updates an existing port by syncing a new launch script and data directory. This uses `rsync` for efficient transfers.

**Syntax:**
```sh
pm_helper update <hosts> <port_script.sh> <port_directory/>
```
**Arguments:**
- `<port_script.sh>`: The path to the port's launch script (e.g., `"My Game.sh"`).
- `<port_directory/>`: The path to the port's data directory (e.g., `mygamedata/`).

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

**Examples:**
```sh
# Copy a new icon into a port's asset directory on one device
pm_helper scp mucube PORT_DATA/mygame/assets/ icon.png

# Copy GOG installer files to a specific data folder on multiple devices
pm_helper scp ark353v:amber351 PORT_DATA/gamename/ gog_installer-*.bin
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

# List all installed ports on multiple devices
pm_helper harbourmaster ark353v:amber351 list
```

---

### `ssh`
Opens an interactive SSH session to a **single** device.

**Syntax:**
```sh
pm_helper ssh <single_host>
```
**Note:** This command will fail if the `<single_host>` resolves to more than one device.

**Example:**
```sh
# SSH into a device using its alias
pm_helper ssh ark353v
```

---

### `hosts`
Displays all configured device aliases from both global and local `pm_devices.cfg` files.

**Syntax:**
```sh
pm_helper hosts
```

---

### `help`
Shows the main help message or help for a specific command.

**Syntax:**
```sh
pm_helper help
pm_helper <command> --help
```
**Example:**
```sh
pm_helper install --help
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE.md) file for details.
