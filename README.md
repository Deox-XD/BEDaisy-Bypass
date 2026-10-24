<div align="center">

# 🛡️ BattlEye Handler

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Status](https://img.shields.io/badge/status-active-success.svg)

**A kernel-mode driver for intercepting and modifying BattlEye callback operations**

### 🚀 Built with Passion by [Deox](https://github.com/deox)

*Pushing the boundaries of C# into the kernel*

[![Discord](https://img.shields.io/badge/Discord-cool.deox-5865F2?style=flat&logo=discord&logoColor=white)](https://discord.gg/cool.deox)

---

**If you found this interesting, check out my other projects!**

⭐ [Star this repo](https://github.com/deox/battleye-handler) • 🍴 [Fork it](https://github.com/deox/battleye-handler/fork) • 💬 [Discord: cool.deox](https://discord.gg/cool.deox)

</div>

---

## 📋 Overview

This project implements a Windows kernel driver that hooks BattlEye's object callback routines to modify process handle permissions. By intercepting pre-operation callbacks, the driver grants elevated access rights to protected game processes, bypassing standard access restrictions.

### ⚡ Key Features

- **Kernel-Mode Operation** - Runs at Ring 0 for maximum privileges
- **Callback Hooking** - Intercepts BattlEye's `ObRegisterCallbacks` 
- **Dynamic Process Targeting** - Configurable target process and driver names
- **Thread-Safe Implementation** - Utilizes system threads for safe initialization
- **Minimal Footprint** - Lightweight design with efficient memory usage

---

## 🏗️ Architecture

```
┌─────────────────────────────────────┐
│      User-Mode Application          │
│    (RainbowSix.exe / Custom)        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│       Windows Kernel Layer          │
│  ┌───────────────────────────────┐  │
│  │   BattlEye Driver (BEDaisy)   │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │  ObRegisterCallbacks    │  │  │
│  │  └───────────┬─────────────┘  │  │
│  └──────────────┼────────────────┘  │
│                 │                    │
│  ┌──────────────▼────────────────┐  │
│  │   DEOX BattlEye Handler       │  │
│  │   ├─ Callback Interceptor     │  │
│  │   ├─ Access Mask Modifier     │  │
│  │   └─ Pre/Post Operation Hooks │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

---

## 🚀 Getting Started

### Prerequisites

- Windows 10/11 (x64)
- Visual Studio 2019 or later
- Windows Driver Kit (WDK) 10
- Test signing enabled or valid code signing certificate

### Building

1. **Clone the repository**
   ```bash
   git clone https://github.com/deox/battleye-handler.git
   cd battleye-handler
   ```

2. **Open the solution**
   ```bash
   start bypass.sln
   ```

3. **Configure build settings**
   - Set to `Release` configuration
   - Platform: `x64`

4. **Build the driver**
   - Build → Build Solution (Ctrl+Shift+B)
   - Output: `bypass.sys`

### Installation

1. **Enable test signing** (if not using a valid certificate)
   ```cmd
   bcdedit /set testsigning on
   ```
   Restart your system after enabling test signing.

2. **Load the driver**
   ```cmd
   sc create BEBypass type= kernel binPath= C:\path\to\bypass.sys
   sc start BEBypass
   ```

3. **Verify driver status**
   ```cmd
   sc query BEBypass
   ```

---

## ⚙️ Configuration

Edit the configuration in `main.cpp` before building:

```cpp
// Target BattlEye driver altitude (usually "363220")
global::target_altitude = L"363220";

// BattlEye driver name
global::target_driver = "BEDaisy";

// Target game process (e.g., "RainbowSix." for Rainbow Six Siege)
global::target_process = "RainbowSix.";

// Your usermode application process name
global::usermode_process = "YourApp";
```

### Supported Games

While designed for flexibility, this handler has been tested with:
- Rainbow Six Siege
- Other BattlEye-protected titles (requires configuration)

---

## 📂 Project Structure

```
battleye-handler/
├── bypass/
│   ├── main.cpp              # Driver entry point and thread initialization
│   ├── defs.hpp              # Type definitions and Windows structures
│   ├── global/
│   │   └── global.hpp        # Global configuration variables
│   ├── hook/
│   │   └── hook.hpp          # Callback hooking implementation
│   └── io/
│       ├── io.hpp            # Debug print declarations
│       └── io.cpp            # I/O utility implementations
└── bypass.sln                # Visual Studio solution file
```

---

## 🔬 Technical Details

### How It Works

1. **Driver Initialization**
   - Creates a system thread on `DriverEntry`
   - Waits for BattlEye driver (`BEDaisy`) to load

2. **Callback Location**
   - Locates process/thread callback lists via `PsProcessType` + 0xC8
   - Iterates through registered callbacks

3. **Hook Installation**
   - Identifies BattlEye callbacks by altitude string (`363220`)
   - Replaces pre/post operation callbacks with custom handlers

4. **Access Modification**
   - Intercepts `ObPreOperationCallback` calls
   - Modifies `DesiredAccess` to `0x1FFFFF` (PROCESS_ALL_ACCESS)
   - Returns control to original callback chain

### Security Considerations

⚠️ **This project is for educational purposes only**

- Requires kernel-mode access (Ring 0)
- Bypasses anti-cheat protection mechanisms
- May violate Terms of Service for online games
- Use at your own risk

---

## 🛠️ Debugging

Enable kernel debugging for development:

```cpp
// Debug output via DbgPrint
io::dbgprint("Your debug message here: %d", value);
```

View debug output using:
- **DebugView** (SysInternals)
- **WinDbg** (Kernel debugging)

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## ⚠️ Disclaimer

This software is provided for **educational and research purposes only**. The authors and contributors are not responsible for any misuse or damage caused by this program. Use of this software to gain unauthorized access to systems or to cheat in online games may violate local laws and Terms of Service agreements.

**Using this driver may result in:**
- Game bans (temporary or permanent)
- Legal consequences
- System instability

**Use at your own risk.**

---

## 🌟 Credits

- **DEOX** - Primary development and maintenance
- **Vergilius Project** - Windows structure research
- Community contributors and testers

---

## 📞 Contact

For questions, suggestions, or issues:

- **GitHub**: [@deox](https://github.com/deox)
- **Issues**: [Report a bug](https://github.com/deox/battleye-handler/issues)

---

<div align="center">

### Made with 💻 and ⚡ by Deox • © 2024

*"Code that shouldn't exist, but does."*

---

**If you found this interesting, check out my other projects!**

⭐ [Star this repo](https://github.com/deox/battleye-handler) • 🍴 [Fork it](https://github.com/deox/battleye-handler/fork) • 💬 [Discord: cool.deox](https://discord.gg/cool.deox)

</div>
