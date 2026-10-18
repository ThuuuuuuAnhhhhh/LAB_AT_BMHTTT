# LAB 6

Mon hoc: An toan va bao mat he thong thong tin
Sinh vien: Ma Thi Thu Anh - 1150070001 - 11DH_HTTT

# LAB6 - pfSense + AD + Access Policies
ĐÃ HOÀN THÀNH LAB ĐỦ 7 KỊCH BẢN 
ĐÃ CẬP NHẬT LINK YOUTUBE 

1. Mục tiêu
Dựng pfSense làm firewall/router phân vùng mạng, kết hợp Active Directory domain controller.
Cô lập và bảo mật hệ thống theo từng khu vực mạng (LAN / MGMT / DMZ / WAN).
Áp dụng chính sách firewall theo nhóm người dùng (alias) và theo khung giờ (schedule).
Mô phỏng một DMZ web server đa site để kiểm chứng các rule kiểm soát truy cập.

2.Sự cố gặp phải & cách xử lý
Full Clone quá chậm → chuyển sang Linked Clone, chỉ mất vài giây.
Quên mật khẩu admin pfSense → reset qua console VM, menu option 3 “Reset webConfigurator password”.
VMware Workstation “Not Responding” do chạy 5 VM + quay màn hình cùng lúc → chờ đợi, không ép tắt.
DMZ-WEB mất IP phụ sau khi dừng/khởi động lại (do ip addr add không lưu qua reboot) → cấu hình lại và xác nhận bằng curl.
Rule KB4 bị tạo sai vị trí (nhảy lên đầu, chặn nhầm cả KB1/KB2/KB3) → phát hiện qua ping OK nhưng curl luôn timeout → xóa, tạo lại bằng nút Add cuối danh sách rồi kéo-thả lên đúng vị trí.
Rule KB3 Pass mất gắn Schedule LUNCH_WINDOW → SOCIAL_SITE luôn Pass bất kể giờ → phát hiện qua test thực tế, gắn lại Schedule.
Nút “Add” không chèn theo vị trí checkbox — bên trái luôn thêm vào đầu, bên phải luôn thêm vào cuối → phải dùng kéo-thả (icon mỏ neo) để sắp xếp đúng.

5Các bước thực hiện
1 Dựng pfSense router

Cài pfSense 2.7.2-RELEASE trên VMware Workstation, cấu hình 4 interface WAN/LAN/MGMT/DMZ. Truy cập WebGUI qua https://10.6.0.1.

2 Dựng DC01 (Active Directory)

Cài Windows Server, dựng AD Domain Services, gán IP tĩnh 10.6.0.10 trong LAN.

3 Dựng Client-A, Client-B

Hai máy Kali Linux (10.6.0.21 và 10.6.0.22) mô phỏng người dùng cuối, test bằng curl/ping.

4 Dựng DMZ-WEB
Linked Clone từ máy Kali có sẵn (nhanh hơn Full Clone rất nhiều).
Gán 4 IP trên cùng eth0 để mô phỏng 4 site:
bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses 172.16.6.10/24 ipv4.gateway 172.16.6.1 ipv4.method manual
sudo nmcli con up "Wired connection 1"
sudo ip addr add 172.16.6.20/24 dev eth0
sudo ip addr add 172.16.6.30/24 dev eth0
sudo ip addr add 172.16.6.40/24 dev eth0
Cấu hình nginx 4 server {} block, mỗi block nghe 1 IP riêng; xóa site default. Kiểm tra bằng curl tới cả 4 IP.
5 Cấu hình LDAP Authentication

Không triển khai — bỏ qua do giới hạn thời gian.

6 Cấu hình Captive Portal

Không triển khai — tương tự mục 3.5.

7 Firewall Aliases & Schedules

Aliases:

Tên	Giá trị
CLIENT_A	10.6.0.21
CLIENT_B	10.6.0.22
POLICY_CLIENTS	10.6.0.21, 10.6.0.22
GENERAL_SITE	172.16.6.10
JOB_SITE	172.16.6.20
SOCIAL_SITE	172.16.6.30
RESTRICTED_SITE	172.16.6.40

Schedules:
Tên	Khung giờ
LUNCH_WINDOW	T2-T6, 11:30-13:00
WORK_HOURS	T2-T6, 08:00-17:00
8 Kịch bản chính sách (1-7)

Rule tổng hợp trên LAN (thứ tự từ trên xuống):

1. Anti-Lockout Rule
2. KB3 Pass  CLIENT_B -> SOCIAL_SITE      (Schedule: LUNCH_WINDOW)
3. KB3 Block CLIENT_B -> SOCIAL_SITE      (always)
4. KB2 Pass  POLICY_CLIENTS -> GENERAL_SITE
5. KB2 Block POLICY_CLIENTS -> RESTRICTED_SITE
6. KB1 Pass  CLIENT_A -> GENERAL_SITE
7. KB1 Block CLIENT_A -> JOB_SITE
8. KB4 Pass  POLICY_CLIENTS -> Any        (Schedule: WORK_HOURS)
9. KB4 Block POLICY_CLIENTS -> Any        (always)
10. Default allow LAN to any

Rule trên DMZ:

1. KB5 Block DMZ subnets -> LAN subnets   (protocol: Any)

Bảng kết quả kiểm thử:

Nguồn	Đích	Trong schedule	Ngoài schedule	Kết quả thực tế
Client-A	GENERAL_SITE .10	Allow	Allow	Allow — curl thành công
Client-A	JOB_SITE .20	Block	Block	Block — timeout ~130s (Reject: ~18ms)
Client-A	RESTRICTED_SITE .40	Block	Block	Block — timeout
Client-B	SOCIAL_SITE .30	Allow trong LUNCH	Block	Block (test ngoài LUNCH_WINDOW)
Client-B	RESTRICTED_SITE .40	Block	Block	Block (suy ra từ KB2)
DMZ-WEB	LAN	Block	Block	Block — ping 100% mất gói

Tóm tắt từng kịch bản:

Kịch bản	Dự đoán	Thực tế	Nhận xét
1. JOB_SITE	Chặn JOB_SITE, cho GENERAL_SITE	Đúng	Hoạt động đúng ngay từ đầu
2. Restricted	Chặn RESTRICTED_SITE	Đúng	Không gặp lỗi
3. Social Schedule	Cho SOCIAL_SITE trong giờ trưa	Ban đầu sai (luôn Pass) do mất gắn Schedule, đã sửa	Phát hiện qua test thực tế
4. Work hours	Cho truy cập trong WORK_HOURS	Đúng (sau khi sửa vị trí)	Ban đầu rule nhảy lên đầu danh sách, chặn nhầm; đã xóa và tạo lại đúng vị trí
5. DMZ isolation	DMZ không vào được LAN	Đúng	Hoạt động đúng ngay
6-7. Reject/Log/State	Reject nhanh hơn Block; log ghi nhận	Đúng (18ms vs 130s; log có “Default deny rule”)	Có bằng chứng số liệu rõ ràng
   
4.Kết luận
Qua bài thực hành LAB 6, nhóm đã dựng thành công mô hình pfSense phân vùng mạng LAN/MGMT/DMZ/WAN, triển khai web server đa site trong DMZ, và cấu hình hoàn chỉnh 7 Kịch bản chính sách firewall dựa trên Aliases và Schedules. Quá trình thực hành giúp hiểu rõ nguyên tắc first-match, tầm quan trọng của thứ tự rule và việc gắn đúng Schedule, cũng như sự khác biệt giữa Block và Reject. Các sự cố gặp phải đều được phát hiện qua kiểm thử thực tế (curl/ping) và khắc phục thành công, cho thấy tầm quan trọng của việc luôn kiểm chứng cấu hình bằng test thực tế. Phần LDAP Authentication và Captive Portal chưa được triển khai do giới hạn thời gian.
