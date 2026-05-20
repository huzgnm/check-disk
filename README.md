# 🩺 Disk Health Check

Script bash kiểm tra toàn diện sức khỏe và thông số ổ đĩa trên Linux server/VPS. Tự động phát hiện loại disk (SSD/HDD/NVMe/virtio/Xen), môi trường ảo hóa và OS family để chạy đúng lệnh.

![Bash](https://img.shields.io/badge/bash-5.x-green) ![Linux](https://img.shields.io/badge/platform-Linux-blue) ![License](https://img.shields.io/badge/license-MIT-lightgrey)

## ✨ Tính năng

- 🔍 **Auto-detect** OS (Debian/Ubuntu, RHEL/CentOS/Rocky, Arch, Alpine, openSUSE) và gợi ý đúng lệnh cài
- 🖥️ **Auto-detect** môi trường ảo hóa (KVM, Xen, OpenVZ, LXC, Docker, VMware) và skip SMART trên virtual disk
- 💾 **Phân loại disk** tự động: SATA SSD, SATA HDD, NVMe, virtio (KVM), xvd (Xen)
- 📊 **Disk usage** với cảnh báo theo ngưỡng (80% warning, 90% critical) + inode usage
- 🌡️ **SMART data**: status, power-on hours, nhiệt độ, bad sectors, SSD wear, NVMe percentage used
- ⚡ **I/O performance** qua `iostat` + top process tiêu thụ I/O
- 🗂️ **Filesystem health**: mount options, ext4 fsck status, mount count
- 🚨 **Kernel errors**: quét `dmesg` tìm lỗi disk gần nhất
- 🔗 **RAID/ZFS**: tự động hiển thị status nếu có mdadm hoặc zpool
- 🎨 **Color output** với option `--no-color` khi pipe ra file

## 📋 Yêu cầu

| Công cụ | Mục đích | Bắt buộc |
|---------|----------|----------|
| `smartctl` (smartmontools) | Đọc SMART data | Khuyến nghị |
| `iostat` (sysstat) | I/O statistics | Khuyến nghị |
| `lsblk`, `df`, `mount` (util-linux) | Thông tin cơ bản | ✅ Bắt buộc |
| `iotop` | Top process I/O | Tuỳ chọn |

### Cài đặt dependencies

**Debian/Ubuntu:**
```bash
sudo apt update && sudo apt install -y smartmontools sysstat util-linux
```

**RHEL/CentOS/Rocky/Alma:**
```bash
sudo dnf install -y smartmontools sysstat util-linux
# hoặc yum nếu hệ cũ
```

**Arch/Manjaro:**
```bash
sudo pacman -S smartmontools sysstat util-linux
```

**Alpine:**
```bash
sudo apk add smartmontools sysstat util-linux
```

## 🚀 Cài đặt

```bash
# Clone repo
git clone https://github.com/huzgnm/disk-health-check.git
cd disk-health-check

# Set executable
chmod +x disk_health_check.sh

# Hoặc tải trực tiếp
wget https://raw.githubusercontent.com/huzgnm/disk-health-check/main/disk_health_check.sh
chmod +x disk_health_check.sh
```

## 💻 Sử dụng

### Cơ bản
```bash
sudo ./disk_health_check.sh
```

### Các option
```bash
sudo ./disk_health_check.sh --full       # Quét thêm top thư mục lớn (chậm)
sudo ./disk_health_check.sh --no-color   # Tắt màu (cho log file)
sudo ./disk_health_check.sh --help       # Xem hướng dẫn
```

### Ví dụ output

```
========================================================================
  1. THÔNG TIN HỆ THỐNG
========================================================================
Hostname     : web-vpn-de01
OS           : Debian GNU/Linux 12 (bookworm)
OS Family    : debian
Kernel       : 6.1.0-13-amd64
Virtualization: kvm (một số check SMART có thể không khả dụng)
Uptime       : up 42 days, 7 hours, 12 minutes

========================================================================
  3. DUNG LƯỢNG DISK
========================================================================
Filesystem    Type    Size     Used     Avail    Use%   Mounted [OK]
/dev/vda1     ext4    78G      24G      51G      32%    /       [OK]
/dev/vdb1     ext4    100G     89G      6G       94%    /data   [CRITICAL]
```

## ⚙️ Cấu hình ngưỡng cảnh báo

Mở file script và chỉnh các biến ở đầu:

```bash
WARN_THRESHOLD=80   # % disk usage cảnh báo
CRIT_THRESHOLD=90   # % disk usage nguy hiểm
INODE_WARN=80       # % inode cảnh báo
TEMP_WARN=50        # Nhiệt độ disk cảnh báo (°C)
TEMP_CRIT=60        # Nhiệt độ disk nguy hiểm (°C)
```

## 🤖 Chạy định kỳ qua cron

Thêm vào `/etc/cron.d/disk_check`:

```cron
# Chạy mỗi 6 giờ, lưu log không màu
0 */6 * * * root /usr/local/bin/disk_health_check.sh --no-color > /var/log/disk_health.log 2>&1
```

Hoặc gửi email khi có critical:

```bash
#!/bin/bash
OUTPUT=$(/usr/local/bin/disk_health_check.sh --no-color)
if echo "$OUTPUT" | grep -q "CRITICAL"; then
    echo "$OUTPUT" | mail -s "[ALERT] Disk issue on $(hostname)" admin@example.com
fi
```

## 🐳 Lưu ý môi trường ảo

| Môi trường | SMART data | Ghi chú |
|------------|------------|---------|
| Bare-metal dedicated | ✅ Đầy đủ | OVH dedi, Hetzner SX/AX, etc. |
| KVM VPS | ⚠️ Không có | Script tự skip virtio disk |
| Xen VPS | ⚠️ Không có | Script tự skip xvd disk |
| OpenVZ/LXC | ❌ Không truy cập được block device | Chỉ check được df, không có lsblk thật |
| Docker container | ❌ Bị giới hạn | Nên chạy trên host |

## 🛡️ Bảo mật

Script chỉ đọc thông tin, **không ghi/sửa/xoá** gì trên hệ thống. Tất cả các lệnh dọn dẹp ở phần "Khuyến nghị" chỉ là gợi ý hiển thị bằng text, không tự động thực hiện.

## 📝 Roadmap

- [ ] Output JSON đầy đủ (`--json`)
- [ ] Telegram bot alert
- [ ] Prometheus exporter format
- [ ] Hỗ trợ BTRFS scrub status
- [ ] LVM volume health check

## 🤝 Đóng góp

Pull requests welcome. Mở issue trước nếu muốn thay đổi lớn.

## 📄 License

MIT License - tự do dùng cho mục đích cá nhân và thương mại.

## 👤 Tác giả

Built for managing VPS fleet across OVH, Hetzner, Netcup, HostHatch, IONOS, EdgeCenter và các provider khác.

---

**⭐ Nếu thấy hữu ích, hãy star repo để ủng hộ!**
