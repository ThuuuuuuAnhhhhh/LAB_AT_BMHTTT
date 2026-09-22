# LAB 3
LAB3 — Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin
Mon hoc: An toan va bao mat he thong thong tin
Sinh vien: Ma Thi Thu Anh - 1150070001 - 11DH_HTTT
Tên lab: Lab 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin
Môn học: An toàn và bảo mật hệ thống thông tin
Năm học: 2026–2027 

TÓM TẮT : 
Phiên bản môi trường thực hành
Thành phần	Phiên bản / Cấu hình thực tế
Ảo hóa	VMware Workstation
Máy ảo	Windows (label "Windows 10 x64 (2)" trong VMware, nhưng hệ thống thực tế nhận diện là bản dựng Windows kiểu Server — winver/whoami trả về build 10.0.20348.169, khác với mô tả Windows 11 25H2 build 26200.9445 trong đề bài. Ghi nhận sai khác này để giảng viên đối chiếu.)
Shell	Windows PowerShell 5.1, chạy với quyền Administrator
Endpoint protection	Microsoft Defender Antivirus (Real-time Protection: bật, Tamper Protection: bật)
Wireshark	Đã cài thủ công qua trình duyệt (tải từ wireshark.org, cài kèm Npcap) do winget không khả dụng trên máy
Python	Chưa cài — bỏ qua do giới hạn thời gian thực hành
Sysmon / Autoruns / Process Explorer	Chưa cài — bỏ qua do giới hạn thời gian thực hành
Gói dữ liệu bài lab	LAB3_Threats_Assets.zip — SHA-256 tính được không khớp với hash công bố trong đề (96236f95ce59d0cc7f9b7ab8fbb04522f21870cd0b53aad6e52da5a23655439); ghi nhận là sai lệch cần giảng viên xác nhận, không chặn tiến độ thực hành
Cách dựng môi trường
Mở máy ảo Windows trong VMware Workstation (mục "Windows 10 x64 (2)" trong thư viện), đăng nhập bằng tài khoản Administrator có sẵn.
Mở PowerShell với quyền Administrator (Run as administrator).
Tạo cấu trúc thư mục làm việc: C:\LAB3 với các thư mục con Evidence, Tools, Downloads, Assets.
Sao chép gói LAB3_Threats_Assets.zip vào C:\LAB3\Downloads, kiểm tra SHA-256 (phát hiện sai lệch — xem ghi chú ở trên), sau đó giải nén vào C:\LAB3 bằng Expand-Archive.
Cài Wireshark thủ công: tải installer từ wireshark.org, chạy qua giao diện (Next → Next → giữ Npcap → Install → Finish) do winget không có sẵn trên máy.
Không cài Python và bộ Sysinternals (Sysmon, Autoruns, Process Explorer) do giới hạn thời gian thực hành trong buổi học.
Các tình huống đã thực hiện và kết quả
Tình huống	Nội dung	Kết quả
Baseline	Kiểm tra Microsoft Defender bằng Get-MpComputerStatus: RealTimeProtectionEnabled : True	PASS
TH1	Xác định tài sản, lỗ hổng, mối đe dọa, rủi ro (risk register 5 dòng) + phân loại 5 tình huống theo 5 nhóm nguồn đe dọa (vô ý, cố ý, thảm họa tự nhiên, lỗi kỹ thuật, lỗi quản lý)	PASS
TH2 — Mã độc (EICAR)	Tải file test EICAR chuẩn từ eicar.org, lưu vào máy, kích hoạt Windows Defender chặn thật (popup "Threats found")	PASS
TH3 — Tấn công mật khẩu / Keylogging	Tạo tài khoản thử nghiệm lab3user, sinh 1 lần đăng nhập thành công (Event ID 4624) và 1 lần đăng nhập thất bại có kiểm soát (Event ID 4625), xác nhận qua Get-WinEvent trên Security log	PASS (một phần) — không thực hiện bước đổi mật khẩu để so sánh trạng thái xác thực trước/sau, và không kiểm tra riêng Event ID 4648, do giới hạn thời gian
TH4 — Backdoor	Không thực hiện	Chưa thực hiện — do giới hạn thời gian trong buổi thực hành
TH5 — Sniffing / MITM / Spoofing	Không thực hiện	Chưa thực hiện — do giới hạn thời gian trong buổi thực hành
TH6 — DoS / DDoS / Mail bombing	Không thực hiện	Chưa thực hiện — do giới hạn thời gian trong buổi thực hành
TH7 — Social Engineering / Phishing	Mở mẫu phishing_email.txt, nhận diện 5 dấu hiệu phishing (khẩn cấp, display name giả tin cậy, domain lạ, Reply-To khác From, yêu cầu credential); phân loại đủ 6 case trong social_engineering_cases.csv (Phishing, Spear Phishing, Pretexting, Baiting, Quid Pro Quo, Watering Hole) kèm biện pháp phòng tránh	PASS
Lỗi gặp phải và cách khắc phục
auditpol /set /subcategory:{GUID} báo lỗi Error 0x00000057: The parameter is incorrect dù cú pháp đúng theo tài liệu Microsoft. → Bỏ qua bước bật audit policy nâng cao này; Windows vẫn ghi nhận đầy đủ Event ID 4624/4625 theo audit policy mặc định của máy, không ảnh hưởng tới kết quả TH3.
New-LocalUser báo lỗi mật khẩu không đạt yêu cầu độ dài/độ phức tạp khi tạo tài khoản lab3user với mật khẩu ngắn. → Khắc phục bằng cách đặt mật khẩu dài hơn, đủ 4 nhóm ký tự (hoa, thường, số, ký tự đặc biệt).
Lệnh runas /user:.\lab3user cmd.exe liên tục báo RUNAS ERROR: Unable to acquire user password, lặp lại cả khi chạy trong PowerShell lẫn Command Prompt thật. → Chuyển hẳn sang cách khác: dùng PowerShell tạo PSCredential bằng ConvertTo-SecureString + New-Object System.Management.Automation.PSCredential, sau đó gọi Start-Process cmd.exe -Credential $cred — cách này chạy thành công và xác nhận được bằng whoami trả về đúng lab3user.
Mất focus bàn phím giữa các cửa sổ trong máy ảo (gõ vào cửa sổ này nhưng không hiện chữ, hoặc ký tự gõ nhầm bị dồn lại). → Khắc phục bằng cách đóng bớt cửa sổ gây xung đột, click trực tiếp vào tiêu đề cửa sổ cần thao tác trước khi gõ, và dùng Ctrl+Alt+Insert để xác nhận VM vẫn nhận input khi nghi ngờ máy bị treo.
Hash SHA-256 của gói LAB3_Threats_Assets.zip không khớp với giá trị công bố trong đề bài. → Ghi nhận là sai lệch cần giảng viên xác nhận (có thể do giảng viên cập nhật lại gói sau khi phát hành tài liệu), không chặn tiến độ, vẫn tiếp tục giải nén và sử dụng gói dữ liệu.
winget không khả dụng trên máy ảo (không có App Installer). → Cài Wireshark thủ công qua trình duyệt (tải .exe từ wireshark.org, cài qua giao diện); bỏ qua việc cài Python và bộ Sysinternals (Sysmon, Autoruns, Process Explorer) do giới hạn thời gian, dẫn tới không thực hiện được TH4, TH5, TH6.
Ghi chú
Toàn bộ ảnh chụp bằng chứng trong báo cáo đều chụp trực tiếp từ máy ảo/màn hình thực hành, không dùng ảnh dựng sẵn.
Không có mật khẩu thật, token, API key hoặc dữ liệu cá nhân nào được đưa vào repository.
Do giới hạn thời gian, TH4/TH5/TH6 chưa được thực hiện; các câu hỏi lý thuyết liên quan (câu 8–13 trong phần Câu hỏi) chưa có bằng chứng thực nghiệm cụ thể từ VM để trích dẫn.
