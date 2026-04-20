# Samsung-Note20-Ultra-boot-When-changre

Chào Van, để tiện cho việc lưu trữ và tái sử dụng cho các máy Samsung khác sau này, tôi đã tổng hợp toàn bộ quy trình thành một **Script tự động (.sh)** và một **File hướng dẫn (.md)**.

Dưới đây là nội dung chi tiết:

### 1. Script tự động (autoboot_setup.sh)
Bạn có thể copy đoạn này, lưu thành file `.sh` và chạy trực tiếp từ máy tính qua lệnh `adb push`.

```bash
#!/system/bin/sh
# Script thiết lập Autoboot cho Samsung Rooted
# Tác giả: Van - Embedded Developer

echo "[+] Đang bắt đầu thiết lập Autoboot..."

# 1. Chiếm quyền và remount
setenforce 0
mount -o remount,rw /

# 2. Xử lý file lpm
if [ -f /system/bin/lpm ]; then
    if [ ! -f /system/bin/lpm.bak ]; then
        echo "[+] Đang sao lưu file lpm gốc..."
        mv /system/bin/lpm /system/bin/lpm.bak
    else
        echo "[!] File lpm.bak đã tồn tại, bỏ qua bước sao lưu."
    fi

    # 3. Tạo file lpm mới để reboot
    echo "[+] Đang tạo script reboot mới..."
    echo "#!/system/bin/sh" > /system/bin/lpm
    echo "/system/bin/reboot" >> /system/bin/lpm

    # 4. Cấp quyền
    chmod 755 /system/bin/lpm
    chown root:shell /system/bin/lpm
    echo "[OK] Thiết lập thành công! Hãy tắt máy và thử cắm sạc."
else
    echo "[Lỗi] Không tìm thấy file /system/bin/lpm. Kiểm tra lại dòng máy."
fi
```

### 2. File hướng dẫn kỹ thuật (README.md)


---
# Hướng dẫn Kỹ thuật: Autoboot Samsung Note 20 Ultra

## Cơ chế hoạt động
Trên các dòng máy Samsung, khi cắm sạc lúc tắt nguồn, hệ thống sẽ thực thi file binary `/system/bin/lpm` để hiển thị hoạt ảnh sạc pin (LPM - Low Power Mode). Bằng cách thay thế binary này bằng một shell script thực hiện lệnh `reboot`, chúng ta ép thiết bị khởi động lại ngay khi nhận được nguồn cấp.

## Các bước thực hiện nhanh (Quick Start)

1. **Kết nối thiết bị:** `adb shell`
2. **Chiếm quyền Root:** `su`
3. **Mở khóa phân vùng:** `mount -o remount,rw /`
4. **Thực thi lệnh đánh tráo:**
   ```bash
   mv /system/bin/lpm /system/bin/lpm.bak
   echo -e "#!/system/bin/sh\n/system/bin/reboot" > /system/bin/lpm
   chmod 755 /system/bin/lpm
   ```

## Lưu ý cho nhà phát triển hệ thống nhúng
- **Tính bền vững:** Sau khi cập nhật hệ điều hành (OTA), file `lpm` sẽ bị Samsung ghi đè về mặc định. Cần chạy lại script này.
- **Dòng điện:** Khi máy tự boot từ 0% pin, hãy đảm bảo nguồn cấp (Adapter/USB Port) có dòng ra tối thiểu **2A**. Nếu dòng quá yếu, máy sẽ rơi vào tình trạng sập nguồn liên tục do thiếu điện năng lúc khởi động (Peak power consumption).
- **SELinux:** Luôn đảm bảo `setenforce 0` khi thực hiện thao tác để tránh các lỗi từ chối quyền truy cập từ kernel.

---

Bản tổng hợp này giúp bạn có cái nhìn hệ thống hơn. Nếu bạn cần tích hợp script này vào một quy trình tự động hóa lớn hơn cho xưởng hoặc dự án Lab, cứ cho tôi biết nhé!
