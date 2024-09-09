# Theo dõi dịch vụ xử lý dữ liệu máy chấm vân tay
>
> Lưu ý: để vào kafka đặt host *192.168.125.7 kafka.utc.edu.vn*

- Mở **Portainer** *192.168.125.155:9000* với ID: *admin*, Password: *CAIT@2020;*
- Truy cập vào menu Stacks->**maychamcong**

1. Bước 1: click vào service: Đối với giảng viên -> ***maychamcong_processor_attendance_teaching*** đối với sinh viên -> ***maychamcong_processor_attendance_studying***
![Xử lý dữ liệu hình 1](../../images/giang-duong/maychamcong-4.png "Xử lý dữ liệu 1")
2. Bước 2: click vào để dòng có chữ  <span style="color:green">**running** </span> để xem log dịch vụ.
![Xử lý dữ liệu hình 2](../../images/giang-duong/maychamcong-5.png "Xử lý dữ liệu 2")
3. Bước 3: vào xem log dịch vụ, nếu log báo <span style="color:green">màu xanh **info** </span>thì là <u>không có lỗi</u>, nếu <span style="color:red">màu đỏ **fail**</span> thì chụp ảnh báo lỗi cho nhóm phần mềm.
![Xử lý dữ liệu hình 3](../../images/giang-duong/maychamcong-6.png "Xử lý dữ liệu 3")
4. Bước 4: Truy cập vào kafka với topics name **attendance_teaching_events** đối với giảng viên, **attendance_studying_events** đối với sinh viên
![Xử lý dữ liệu hình 4](../../images/giang-duong/maychamcong-7.png "Xử lý dữ liệu 4")

- Click vào **attendance_teaching_events**, Tại topic này xem mục Consumers với GroupId: **processor_attendance_teaching** nếu là <span style="color:green">**0** </span> thì đã xử lý hết dữ liệu chấm công, ngược lại là đang tồn số bản ghi chưa được xử lý.
![Xử lý dữ liệu hình 5](../../images/giang-duong/maychamcong-8.png "Xử lý dữ liệu 5")
