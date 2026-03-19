[README.md](https://github.com/user-attachments/files/26125935/README.md)

<p align="center">
  <img src="https://www.redhat.com/rhdc/managed-files/rh_community_logo_reverse.svg" alt="Red Hat Community Logo" width="300"/>
</p>

<h1 align="center">t430ctl - Enterprise Optimization Utility for ThinkPad T430</h1>

<p align="center">
  <a href="https://www.rust-lang.org" title="Built with Rust">
    <img src="https://img.shields.io/badge/Built%20with-Rust-orange?style=for-the-badge&logo=rust" alt="Built with Rust">
  </a>
  <a href="https://github.com/ITSsafer-DevOps/Optimization_RHEL_RustLang/actions/workflows/rust.yml">
    <img src="https://img.shields.io/github/actions/workflow/status/ITSsafer-DevOps/Optimization_RHEL_RustLang/rust.yml?branch=main&style=for-the-badge&logo=githubactions&label=CI" alt="CI Build Status">
  </a>
</p>

<p align="center">
  <strong>Project Author:</strong> Kristián Kašník<br/>
  <a href="https://github.com/ITSsafer-DevOps/Optimization_RHEL_RustLang/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT">
  </a>
</p>

`t430ctl` is a robust, command-line utility engineered in Rust to systematically apply performance and power-saving optimizations for Lenovo ThinkPad T430 systems running Red Hat Enterprise Linux 9 (or compatible derivatives). It provides a declarative, idempotent, and reversible mechanism for managing key system parameters, adhering to enterprise best practices.

---

## Project Philosophy

The design of `t430ctl` is guided by three core principles:

1.  **System Integration:** Instead of using brittle, standalone scripts, `t430ctl` leverages the standard `tuned` framework provided by Red Hat. This ensures that optimizations are managed by a system service and are applied consistently across reboots.
2.  **Safety and Reversibility:** All changes are fully reversible via a single `revert` command. The tool is designed to be non-destructive and predictable.
3.  **Idempotency:** The `apply` operation can be run multiple times without causing unintended side effects. The system will always converge to the same desired state.

## Architectural Workflow

The utility orchestrates several core Linux subsystems. The diagrams below, rendered using the **Mermaid.js** framework, illustrate the operational flow for the `apply` and `revert` commands.

### `apply` Workflow

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'primaryColor': '#EE0000', 'primaryTextColor': '#fff', 'primaryBorderColor': '#9d0a0a', 'lineColor': '#555', 'secondaryColor': '#fdf0f0', 'tertiaryColor': '#fff'}}}%%
graph TD
    subgraph "User Action"
        A["[user@host]$ sudo t430ctl apply"]
    end

    subgraph "t430ctl `apply` Workflow"
        style B fill:#222,stroke:#c00,stroke-width:2px,color:#fff
        B{"1. Validate<br><i>Root & Command Checks</i>"}
        C["2. Create Tuned Profile<br><i>/etc/tuned/t430-balanced-dev/tuned.conf</i>"]
        D["3. Set Live Kernel Tunables<br><i>/sys/devices/.../scaling_governor<br>/sys/block/sda/queue/scheduler</i>"]
        E["4. Patch TLP Config<br><i>/etc/tlp.conf</i>"]
        F["5. Activate Profile via `tuned-adm`<br><i>Applies persistent settings</i>"]
        G(("(System Optimized)"))
    end

    A --> B;
    B -- Success --> C;
    C --> D;
    D --> E;
    E --> F;
    F --> G;
```

### `revert` Workflow

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'primaryColor': '#EE0000', 'primaryTextColor': '#fff', 'primaryBorderColor': '#9d0a0a', 'lineColor': '#555', 'secondaryColor': '#fdf0f0', 'tertiaryColor': '#fff'}}}%%
graph TD
    subgraph "User Action"
        A[<font color=black><b>[user@host]$</b> sudo t430ctl revert</font>]
    end

    subgraph "t430ctl `revert` Workflow"
        style B fill:#222,stroke:#c00,stroke-width:2px,color:#fff
        B{1. Validate<br><i>Root & Command Checks</i>}
        C[2. Activate `balanced` Profile<br><i>tuned-adm profile balanced</i>]
        D[3. Delete Custom Profile<br><i>rm -rf /etc/tuned/t430-balanced-dev</i>]
        E[4. Delete Sysctl Config<br><i>rm /etc/sysctl.d/99-t430.conf</i>]
        F[5. Revert TLP Config<br><i>/etc/tlp.conf</i>]
        G((System State Restored))
    end

    A --> B
    B -- Success --> C
    C --> D
    D --> E
    E --> F
    F --> G
```


## System Requirements

*   **Hardware:** Lenovo ThinkPad T430 (or systems with a similar 3rd Gen Intel Core "Ivy Bridge" architecture).
*   **Operating System:** Red Hat Enterprise Linux 9.x, Fedora 38+, or a compatible derivative.
*   **System Packages:** The following packages must be installed and enabled:
    ```bash
    # Install required system utilities and build tools
    sudo dnf install tuned tlp tlp-rdw lm_sensors cpupower git rustc cargo
    
    # Enable the core system services for power management
    sudo systemctl enable --now tuned
    sudo systemctl enable --now tlp
    ```

## Deployment and Operational Guide

### Step 1: Build from Source

Clone the official repository and compile the project. Using the `--release` flag is strongly recommended as it builds an optimized executable.

```bash
# Clone the repository
git clone https://github.com/ITSsafer-DevOps/Optimization_RHEL_RustLang.git
cd Optimization_RHEL_RustLang

# Build the optimized release binary. This may take a few minutes on the first run.
cargo build --release
```

### 2. Deploy Binary

For system-wide access, deploy the compiled binary to a directory within the system's `PATH`.

```bash
sudo cp target/release/t430ctl /usr/local/bin/
```

### 3. Operational Commands

The utility exposes three primary subcommands for system management.

#### `apply`

Applies the `t430-balanced-dev` optimization profile. This operation is idempotent and requires root privileges.

<div style="background-color: #1e1e1e; color: #d4d4d4; font-family: 'Consolas', 'Courier New', monospace; padding: 15px; border-radius: 5px; border: 1px solid #333;">
  <span style="color: #9cdcfe;">[RHEL@RHEL t430ctl]$</span> <span style="color: #ce9178;">sudo</span> t430ctl apply<br>
  <span style="color: #d4d4d4;">[sudo] password for RHEL: *********</span><br>
  <span style="color: #dcdcaa;">[*] Applying T430 balanced-dev optimization...</span><br>
  <span style="color: #d4d4d4;">[1/5] Setting CPU governor to schedutil...</span><br>
  <span style="color: #d4d4d4;">[2/5] Creating tuned profile...</span><br>
  <span style="color: #d4d4d4;">[3/5] Activating tuned profile...</span><br>
  <span style="color: #d4d4d4;">[4/5] Setting HDD scheduler...</span><br>
  <span style="color: #d4d4d4;">[5/5] Updating TLP config...</span><br>
  <span style="color: #4ec9b0;">Done. Reboot recommended.</span>
</div>

#### `diag`

Executes a series of diagnostic checks to report the current state of optimized parameters. This command does not require root privileges.

<div style="background-color: #1e1e1e; color: #d4d4d4; font-family: 'Consolas', 'Courier New', monospace; padding: 15px; border-radius: 5px; border: 1px solid #333;">
  <span style="color: #9cdcfe;">[RHEL@RHEL t430ctl]$</span> t430ctl diag<br>
  <br>
  <span style="color: #dcdcaa;">=== Tuned Profile ===</span><br>
  <span style="color: #d4d4d4;">Current active profile: t430-balanced-dev</span><br>
  <br>
  <span style="color: #dcdcaa;">=== HDD Scheduler ===</span><br>
  <span style="color: #d4d4d4;">[mq-deadline] kyber bfq none</span><br>
  <span style="color: #d4d4d4;">...</span>
</div>

#### `revert`

Reverts all changes made by the `apply` command, restoring the system to the default `balanced` profile and removing all custom configurations. This operation requires root privileges.

<div style="background-color: #1e1e1e; color: #d4d4d4; font-family: 'Consolas', 'Courier New', monospace; padding: 15px; border-radius: 5px; border: 1px solid #333;">
  <span style="color: #9cdcfe;">[RHEL@RHEL t430ctl]$</span> <span style="color: #ce9178;">sudo</span> t430ctl revert<br>
  <span style="color: #dcdcaa;">[*] Reverting T430 settings...</span><br>
  <span style="color: #d4d4d4;">[1/4] Switching tuned profile to balanced...</span><br>
  <span style="color: #d4d4d4;">[2/4] Removing custom tuned profile...</span><br>
  <span style="color: #d4d4d4;">[3/4] Removing sysctl config...</span><br>
  <span style="color: #d4d4d4;">[4/4] Resetting TLP USB_AUTOSUSPEND...</span><br>
  <span style="color: #4ec9b0;">Revert complete. Reboot recommended.</span>
</div>

## Configuration Reference

The `t430-balanced-dev` profile applies the following key configurations:

| Category      | Parameter               | Value                   | Rationale                                           |
|---------------|-------------------------|-------------------------|-----------------------------------------------------|
| **CPU**       | Governor                | `schedutil`             | Modern, balanced scheduler for performance/power.   |
| **Disk I/O**  | Scheduler (HDD)         | `mq-deadline`           | Optimizes for low latency on rotational disks.      |
|               | Readahead               | `512` KB                | A balanced value for mixed read patterns.           |
| **Memory**    | `vm.swappiness`         | `10`                    | Strongly discourages swapping to the slow HDD.      |
|               | `vm.vfs_cache_pressure` | `50`                    | Favors retaining filesystem metadata in RAM.        |
|               | `transparent_hugepages` | `never`                 | Prevents latency spikes in database/VM workloads.   |
| **Power**     | `USB_AUTOSUSPEND`       | `0` (Off)               | Ensures stability of external peripherals.          |

## Contributing

Contributions to enhance `t430ctl` are welcome. Please follow the standard fork-and-pull-request workflow.

1.  Fork the repository.
2.  Create a feature branch (`git checkout -b feature/my-enhancement`).
3.  Commit your changes (`git commit -am 'feat: Add new optimization'`).
4.  Push to the branch (`git push origin feature/my-enhancement`).
5.  Open a new Pull Request.

## License

This project is licensed under the MIT License.

---
<p align="center"><i>This tool is provided as-is. Always perform backups of critical data before applying system-level changes.</i></p>
