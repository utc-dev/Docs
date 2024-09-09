# Theo dõi dịch vụ xử dữ liệu chấm vân tay hiển trên USMART, TIVI, EUTC
>
> Lưu ý: để vào kafka đặt host *192.168.125.7 kafka.utc.edu.vn*
- Mở **Portainer** *192.168.125.85:9000* với ID: *admin*, Password: *X4xD7KBqKca9QGS5*
- Truy cập vào stacks **utc_prod**

1. Bước 1: click vào service: ***utc_prod_giangduong_workers***
![worker hình 1](../../images/giang-duong/worker-1.png "Hình 1")
2. Bước 2: click vào để dòng có chữ  <span style="color:green">**running** </span> để xem log dịch vụ.
![worker hình 2](../../images/giang-duong/worker-2.png "Hình 2")
3. Bước 3: vào xem log dịch vụ, nếu log báo <span style="color:green">màu xanh **info** </span>thì là <u>không có lỗi</u>, nếu <span style="color:red">màu đỏ **fail**</span> thì chụp ảnh báo lỗi cho nhóm phần mềm.
![worker hình 3](../../images/giang-duong/worker-3.png "Hình 3")
4. Bước 4: Truy cập vào kafka với topics name **attendance_teaching_2024** đối với giảng viên, **attendance_studying_2024** đối với sinh viên
![worker hình 4](../../images/giang-duong/worker-4.png "Hình 4")

- Click vào **attendance_teaching_2024**, Tại topic này xem mục Consumers với GroupId: **giangduong_teaching_worker_prod** nếu là <span style="color:green">**0** </span> thì đã xử lý hết dữ liệu để hiển thị chấm công, ngược lại là đang tồn số bản ghi chưa được xử lý.
![worker hình 5](../../images/giang-duong/worker-5.png "Hình 5")
