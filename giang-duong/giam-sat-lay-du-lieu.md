# Theo dõi dịch vụ lấy dữ liệu từ các máy chấm vân tay ở phòng nước
>
> Lưu ý: để vào kafka đặt host *192.168.125.7 kafka.utc.edu.vn*

- Mở **Portainer** *192.168.125.155:9000* với ID: *admin*, Password: *CAIT@2020;*
- Truy cập vào menu Stacks->**maychamcong**

1. Bước 1: click vào service: Đối với giảng viên -> ***maychamcong_receiver_attendance_teaching*** đối với sinh viên -> ***maychamcong_receiver_attendance_studying***
![Nhận dữ liệu 1](../../images/giang-duong/maychamcong-1.png "Nhận dữ liệu 1")
2. Bước 2: click vào để dòng có chữ  <span style="color:green">**running** </span> để xem log dịch vụ.
![Nhận dữ liệu 2](../../images/giang-duong/maychamcong-2.png "Nhận dữ liệu 2")
3. Bước 3: vào xem log dịch vụ, nếu log báo <span style="color:green">màu xanh **info** </span>thì là <u>không có lỗi</u>, nếu <span style="color:red">màu đỏ **fail**</span> thì chụp ảnh báo lỗi cho nhóm phần mềm
![Nhận dữ liệu 3](../../images/giang-duong/maychamcong-3.png "Nhận dữ liệu 3")
4. Bước 4: Sau khi nhận dữ liệu từ các máy chấm vân tay, dịch vụ sẽ xử lý dữ liệu thô rồi bắn lên kafka với topics name **attendance_teaching_events** đối với giảng viên, **attendance_studying_events** đối với sinh viên
