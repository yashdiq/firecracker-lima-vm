# 🚀 Firecracker MicroVM Setup on MacBook Pro M3

This guide walks you through setting up Firecracker microVMs on your MacBook Pro M3 using Lima virtualization.

## 📋 Prerequisites

- **MacBook Pro M3** (or newer Apple Silicon)
- **Lima** installed (`brew install lima`)
- **Basic command line knowledge**

---

## 🏗️ Step 1: Create Lima VM with Nested Virtualization

Start a Lima instance with nested virtualization enabled:

```bash
limactl start --set '.nestedVirtualization=true' --name=mvm template://default
```

> 💡 **Note**: This enables hardware virtualization support needed for Firecracker.

---

## 🔧 Step 2: Enter Lima VM & Install Firecracker

```bash
limactl shell mvm

# Add user to kvm group for proper permissions
sudo usermod -aG kvm $USER

# Exit the Lima VM to apply group changes
exit

# Shell back into the Lima VM with new group permissions
limactl shell mvm
```

Once inside the Lima VM install Firecracker:

```bash
# Set version and architecture
FC_VERSION="v1.13.0"
ARCH="aarch64"

# Download Firecracker
wget https://github.com/firecracker-microvm/firecracker/releases/download/${FC_VERSION}/firecracker-${FC_VERSION}-${ARCH}.tgz

# Extract and install
tar -xzf firecracker-${FC_VERSION}-${ARCH}.tgz
sudo mv release-${FC_VERSION}-${ARCH}/firecracker-${FC_VERSION}-${ARCH} /usr/local/bin/firecracker
sudo chmod +x /usr/local/bin/firecracker

# Cleanup
rm -rf firecracker-${FC_VERSION}-${ARCH}.tgz release-${FC_VERSION}-${ARCH}

# Verify installation
firecracker --version
```

---

## 📦 Step 3: Download Kernel and RootFS

Create workspace and download the latest supported kernel:

```bash
mkdir -p ~/microvm && cd ~/microvm

# Get latest kernel
latest_kernel_key=$(wget "http://spec.ccfc.min.s3.amazonaws.com/?prefix=firecracker-ci/v1.13/${ARCH}/vmlinux-5.10&list-type=2" -O - 2>/dev/null | grep "(?<=<Key>)(firecracker-ci/v1.13/${ARCH}/vmlinux-5\.10\.[0-9]{3})(?=</Key>)" -o -P)
wget "https://s3.amazonaws.com/spec.ccfc.min/${latest_kernel_key}"
```

Download Ubuntu rootfs:

```bash
# Get latest Ubuntu rootfs
latest_ubuntu_key=$(curl "http://spec.ccfc.min.s3.amazonaws.com/?prefix=firecracker-ci/v1.13/${ARCH}/ubuntu-&list-type=2" \
    | grep -oP "(?<=<Key>)(firecracker-ci/v1.13/${ARCH}/ubuntu-[0-9]+\.[0-9]+\.squashfs)(?=</Key>)" \
    | sort -V | tail -1)
ubuntu_version=$(basename $latest_ubuntu_key .squashfs | grep -oE '[0-9]+\.[0-9]+')
wget -O ubuntu-$ubuntu_version.squashfs.upstream "https://s3.amazonaws.com/spec.ccfc.min/$latest_ubuntu_key"
```

---

## 🛠️ Step 4: Prepare Root Filesystem

Extract and configure the root filesystem:

```bash
# Extract squashfs
unsquashfs ubuntu-$ubuntu_version.squashfs.upstream

# Generate SSH keys
ssh-keygen -f id_rsa -N ""

# Setup SSH access
cp -v id_rsa.pub squashfs-root/root/.ssh/authorized_keys
mv -v id_rsa ./ubuntu-$ubuntu_version.id_rsa
sudo chown -R root:root squashfs-root

# Create ext4 filesystem
truncate -s 1G ubuntu-$ubuntu_version.ext4
sudo mkfs.ext4 -d squashfs-root -F ubuntu-$ubuntu_version.ext4

# Verify setup
echo "📋 Setup Summary:"
KERNEL=$(ls vmlinux-* | tail -1)
[ -f $KERNEL ] && echo "✅ Kernel: $KERNEL" || echo "❌ ERROR: Kernel $KERNEL does not exist"
ROOTFS=$(ls *.ext4 | tail -1)
e2fsck -fn $ROOTFS &>/dev/null && echo "✅ Rootfs: $ROOTFS" || echo "❌ ERROR: $ROOTFS is not a valid ext4 fs"
KEY_NAME=$(ls *.id_rsa | tail -1)
[ -f $KEY_NAME ] && echo "✅ SSH Key: $KEY_NAME" || echo "❌ ERROR: Key $KEY_NAME does not exist"
```

---

## 🔐 Step 5: Configure SSH Access (Alternative Method)

If you need to manually configure SSH access:

```bash
# Create mount point
mkdir -p mnt
sudo mount -o loop ubuntu-24.04.ext4 mnt

# Setup SSH directory
sudo mkdir -p mnt/root/.ssh
sudo cp id_rsa.pub mnt/root/.ssh/authorized_keys
sudo chmod 700 mnt/root/.ssh
sudo chmod 600 mnt/root/.ssh/authorized_keys

# Cleanup
sudo umount mnt
```

---

## 🚀 Step 6: Start Firecracker

```bash
# Set socket path
API_SOCKET="/tmp/firecracker.socket"

# Clean up any existing socket
sudo rm -f $API_SOCKET

# Start Firecracker with PCI support
sudo firecracker --api-sock "${API_SOCKET}" --enable-pci
```

> 🎯 **Firecracker is now running!** Leave this terminal open.

---

## 🖥️ Step 7: Launch MicroVM

Open a **new terminal** and connect to the Lima VM:

```bash
limactl shell mvm
cd ~/microvm
./start.sh
```

The `start.sh` script will:

- 🌐 Configure network interfaces
- 🔥 Set up IP forwarding and NAT
- ⚙️ Configure the microVM via Firecracker API
- 🚀 Start the microVM
- 📡 Configure guest networking
- 🔐 Provide SSH connection instructions

---

## 🎮 Usage

Once the script completes, you'll see connection instructions similar to:

```bash
To connect to the microVM, run:
ssh -i ./ubuntu-24.04.id_rsa root@172.16.0.2

Use 'root' for both the login and password.
Run 'reboot' to exit.
```

---

## 🐛 Troubleshooting

### Common Issues

1. **"No route to host" error**

   - The script now includes better timing and error handling
   - Wait a few more seconds before connecting

2. **iptables errors**

   - These are now suppressed and harmless
   - Network setup continues normally

3. **SSH connection failures**
   - The script waits up to 30 seconds for SSH to be available
   - Check if the microVM started successfully

### Performance Tips for M3

- 🚀 The M3's hardware virtualization provides excellent performance
- 💾 Allocate sufficient memory to Lima VM if needed
- 🌡️ Monitor temperature under heavy load

---

## 📚 Additional Resources

- [Firecracker Documentation](https://github.com/firecracker-microvm/firecracker)
- [Lima Documentation](https://github.com/lima-vm/lima)
- [Apple Silicon Virtualization](https://developer.apple.com/documentation/virtualization)

---

## 🎉 Success!

You now have a running Firecracker microVM on your MacBook Pro M3! Enjoy the lightweight, secure virtualization experience.

---

_Last updated: $(date '+%Y-%m-%d')_
