# Samsung Note 20 Ultra: Auto Boot When Charged

Tài liệu hướng dẫn và Script tự động dành cho các dòng Samsung (đã thử nghiệm thành công trên Note 20 Ultra).

## ⚠️ Điều kiện tiên quyết (Bắt buộc)
* **Thiết bị phải được Root bằng Magisk:** Đây là điều kiện tiên quyết để có thể chiếm quyền Superuser (`su`) và ghi dữ liệu vào phân vùng hệ thống (`/system`).
* **Môi trường:** Máy tính đã cài đặt ADB Drivers và điện thoại đã bật *USB Debugging*.

---

## 1. Script tự động thiết lập (`autoboot_setup.sh`)

Bạn có thể tạo file này trên máy tính, đẩy vào điện thoại và chạy để tiết kiệm thời gian.

```bash
#!/system/bin/sh
# Script thiết lập Autoboot cho Samsung (Tested on Note 20 Ultra)
# Yêu cầu: Root bằng Magisk
# Tác giả: Van - Embedded Developer

echo "[+] Đang kiểm tra quyền Root..."
if [ "$(id -u)" -ne 0 ]; then
    echo "[Lỗi] Vui lòng chạy script bằng quyền Root (su)!"
    exit 1
fi

echo "[+] Đang cấu hình hệ thống..."
setenforce 0
mount -o remount,rw /

# Đường dẫn file sạc của Samsung
LPM_PATH="/system/bin/lpm"

if [ -f "$LPM_PATH" ]; then
    if [ ! -f "$LPM_PATH.bak" ]; then
        echo "[+] Đang sao lưu file lpm gốc tại $LPM_PATH.bak"
        mv "$LPM_PATH" "$LPM_PATH.bak"
    else
        echo "[!] Bản sao lưu lpm.bak đã tồn tại, tiến hành ghi đè script mới."
    fi

    echo "[+] Đang nạp script Autoboot..."
    echo "#!/system/bin/sh" > "$LPM_PATH"
    echo "/system/bin/reboot" >> "$LPM_PATH"

    echo "[+] Cấp quyền thực thi..."
    chmod 755 "$LPM_PATH"
    chown root:shell "$LPM_PATH"
    
    echo "[OK] Hoàn tất! Hãy tắt nguồn và cắm sạc để kiểm tra."
else
    echo "[Lỗi] Không tìm thấy binary lpm tại $LPM_PATH. Có thể dòng máy này dùng cơ chế khác."
fi
```

---

## 2. Hướng dẫn kỹ thuật (README.md)

### Cơ chế hoạt động
Khi tắt nguồn và cắm sạc, bootloader của Samsung sẽ gọi tiến trình `/system/bin/lpm` để hiển thị màn hình sạc. Việc thay thế binary này bằng một script chứa lệnh `/system/bin/reboot` sẽ đánh lừa hệ thống khởi động lại ngay lập tức thay vì dừng lại ở màn hình sạc.

### Các bước thực hiện thủ công nhanh
1. **Truy cập:** `adb shell` -> `su`.
2. **Mở khóa ghi:** `mount -o remount,rw /`.
3. **Đánh tráo:**
   ```bash
   mv /system/bin/lpm /system/bin/lpm.bak
   echo -e "#!/system/bin/sh\n/system/bin/reboot" > /system/bin/lpm
   chmod 755 /system/bin/lpm
   ```

### Lưu ý quan trọng cho dân Embedded
* **Tính bền vững:** Cập nhật OTA sẽ ghi đè file này. Cần chạy lại script sau mỗi lần cập nhật firmware.
* **Nguồn cấp:** Đảm bảo dòng ra của sạc ổn định (tối thiểu **2A**). Khi máy tự boot từ 0%, nếu nguồn yếu sẽ gây ra hiện tượng sập nguồn liên tục (Brown-out) do CPU tiêu thụ điện năng đạt đỉnh (Peak) lúc khởi động.
* **An toàn:** Luôn giữ file `/system/bin/lpm.bak` để có thể khôi phục trạng thái xuất xưởng khi cần.

---
