# 🔥 Firecracker MicroVM on MacBook Pro M3

A lightweight, high-performance virtualization setup using Firecracker microVMs on Apple Silicon MacBooks via Lima.

## 🚀 Quick Overview

This project provides a complete setup for running Firecracker microVMs on MacBook Pro M3, leveraging Apple's hardware virtualization capabilities for optimal performance.

## 📋 What's Included

- **🔧 Firecracker Setup** - Latest Firecracker binary installation
- **🐧 Ubuntu MicroVM** - Pre-configured Ubuntu root filesystem
- **🌐 Network Configuration** - Automated networking and internet access
- **🔐 SSH Access** - Secure key-based authentication
- **⚡ Automation Scripts** - One-command microVM startup

## 🎯 Key Features

- ⚡ **Blazing Fast** - Native Apple Silicon virtualization
- 🔒 **Secure Isolation** - Firecracker's minimal attack surface
- 🌐 **Network Ready** - Automatic internet connectivity
- 📱 **Lightweight** - Minimal resource footprint
- 🛠️ **Easy Setup** - Automated configuration scripts

## 🏗️ System Requirements

- **MacBook Pro M3** (or newer Apple Silicon)
- **macOS** with Lima installed
- **Terminal** access
- ** sudo** privileges

## 🚀 Getting Started

### 1. Install Lima

```bash
brew install lima
```

### 2. Run Setup

```bash
# Follow the detailed setup guide
cat instructions.md
```

### 3. Start Your MicroVM

```bash
./start.sh
```

## 📁 Project Structure

```
microVM/
├── README.md              # This file - project overview
├── instructions.md        # 📖 Detailed setup guide
├── start.sh              # 🚀 MicroVM startup script
├── start.py              # 🐍 Python alternative (optional)
├── vm_config.json        # ⚙️ VM configuration
└── *.ext4, *.id_rsa      # 📦 Generated files (after setup)
```

## 🎮 Usage

Once setup is complete:

```bash
# Start Firecracker (Terminal 1)
sudo ./firecracker --api-sock /tmp/firecracker.socket --enable-pci

# Launch MicroVM (Terminal 2)
./start.sh

# Connect to your MicroVM
ssh -i *.id_rsa root@172.16.0.2
```

## 🔧 Configuration

The microVM comes pre-configured with:

- **IP Address**: `172.16.0.2`
- **Host Gateway**: `172.16.0.1`
- **SSH Access**: Key-based authentication
- **Internet**: NAT via host network

## 🐛 Troubleshooting

Common issues and solutions are covered in the [detailed instructions](instructions.md#-troubleshooting).

## 📚 Learn More

- 🔥 [Firecracker Documentation](https://github.com/firecracker-microvm/firecracker)
- 🍎 [Lima Virtualization](https://github.com/lima-vm/lima)
- 🖥️ [Apple Silicon Virtualization](https://developer.apple.com/documentation/virtualization)

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

## 📄 License

This project follows the same license as Firecracker.

---

**🎉 Ready to dive in? Check out [instructions.md](instructions.md) for the complete setup guide!**
