# 🩺 Disk Health Check

Script bash kiểm tra toàn diện sức khỏe và thông số ổ đĩa trên Linux. **Output tiếng Việt, tự cài deps, tự scan.**

## ⚡ Chạy ngay (1 lệnh)

```bash
curl -sL https://raw.githubusercontent.com/huzgnm/disk-health-check/main/disk_health_check.sh | sudo bash
```

Lệnh trên sẽ tự động:
- ✅ Phát hiện OS (Debian/Ubuntu/RHEL/CentOS/Arch/Alpine...)
- ✅ Tự cài `smartmontools`, `sysstat` nếu thiếu
- ✅ Phát hiện môi trường ảo (KVM/Xen/OpenVZ)
- ✅ Phân loại disk (SSD/HDD/NVMe/virtio)
- ✅ Quét đầy đủ và in báo cáo tiếng Việt

## 📊 Script kiểm tra những gì

1. **Thông tin hệ thống** — OS, kernel, uptime, virtualization
2. **Danh sách disk** — SSD/HDD/NVMe/virtio, model, vendor, size
3. **Dung lượng** — `df -h` với cảnh báo màu (80% vàng, 90% đỏ) + inode
4. **SMART health** — status, power-on hours, nhiệt độ, bad sectors, SSD wear, NVMe % used
5. **I/O performance** — `iostat` + top process tiêu thụ I/O
6. **Filesystem** — mount options, ext4 fsck status
7. **Kernel errors** — quét `dmesg` tìm lỗi disk
8. **RAID / ZFS** — tự hiện status nếu có
9. **Tóm tắt** — số partition critical/warning + gợi ý dọn dẹp

## 🔧 Cách dùng khác

**Tải về dùng lâu dài:**
```bash
curl -sL https://raw.githubusercontent.com/huzgnm/disk-health-check/main/disk_health_check.sh -o /usr/local/bin/disk-check && chmod +x /usr/local/bin/disk-check
```

Sau đó chỉ cần gõ:
```bash
sudo disk-check
```

**Quét đầy đủ (kèm top thư mục lớn):**
```bash
sudo disk-check --full
```

**Chạy không màu (cho cron log):**
```bash
sudo disk-check --no-color > /var/log/disk_health.log
```

## 🤖 Chạy định kỳ qua cron

```cron
# /etc/cron.d/disk_check - chạy mỗi 6 giờ
0 */6 * * * root /usr/local/bin/disk-check --no-color > /var/log/disk_health.log 2>&1
```

## ⚙️ Tuỳ chỉnh ngưỡng cảnh báo

Sửa các biến ở đầu script:

```bash
WARN_THRESHOLD=80   # % disk usage cảnh báo
CRIT_THRESHOLD=90   # % disk usage nguy hiểm
TEMP_WARN=50        # Nhiệt độ disk cảnh báo (°C)
TEMP_CRIT=60        # Nhiệt độ disk nguy hiểm (°C)
```

## 🐳 Lưu ý môi trường ảo

| Môi trường | SMART data | Ghi chú |
|------------|------------|---------|
| Bare-metal dedicated | ✅ Đầy đủ | OVH dedi, Hetzner SX/AX |
| KVM VPS | ⚠️ Không có | Script tự skip virtio disk |
| Xen VPS | ⚠️ Không có | Script tự skip xvd disk |
| OpenVZ/LXC | ❌ Bị giới hạn | Chỉ check được df |

## 🛡️ Bảo mật

Script chỉ **đọc** thông tin, không ghi/sửa/xoá gì. Phần "khuyến nghị dọn dẹp" chỉ hiển thị text gợi ý, không tự thực hiện.

## 📄 License

MIT — tự do dùng cho cá nhân và thương mại.
