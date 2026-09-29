LAB 4 — Khảo sát và đánh giá bề mặt mạng bằng Nmap
Thông tin sinh viên
Họ tên:  MA THỊ THU ÁNH  
MSSV: 1150070001
Lớp : 11ĐH_TMĐT
Tên lab: Lab 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap
Môn học: An toàn và bảo mật hệ thống thông tin
Năm học: 2026–2027

Thành phần	Phiên bản / Cấu hình thực tế
Ảo hóa	VMware Workstation (đề bài gốc hướng dẫn VirtualBox, đã thay thế toàn bộ bằng VMware, giữ nguyên logic mạng/topology). 
Mạng	Host-Only (VMnet1), dải mạng thực tế 192.168.240.0/24 (khác với ví dụ 192.168.56.0/24 trong tài liệu). 
Máy tấn công	Kali Linux 2026.2 x64 (VM "MAANH-LINUX-Attacker"), IP: 192.168.240.128 
Máy mục tiêu	Metasploitable 2 (VM "Metasploitable2-Linux"), tải từ SourceForge (Rapid7), IP: 192.168.240.131
Công cụ quét	Nmap 7.99 (cài sẵn trên Kali). 
Tài khoản Metasploitable 2	msfadmin / msfadmin 

Cách dựng môi trường
Sử dụng máy ảo Kali Linux có sẵn trong VMware Workstation, đảm bảo Network Adapter đặt ở chế độ Host-only.
Tải gói Metasploitable 2 (metasploitable-linux-2.0.0.zip) từ SourceForge, giải nén và mở bằng VMware qua file .vmx (chọn "I Copied It" để cấp lại MAC address mới).
Kiểm tra/đặt Network Adapter của Metasploitable 2 về Host-only (VMnet1) để cùng phân đoạn mạng với Kali.
Đăng nhập Metasploitable 2 bằng msfadmin/msfadmin, lấy IP thật bằng ifconfig → 192.168.240.131.
Trên Kali, lấy IP bằng ip -br addr → 192.168.240.128.
Kiểm tra kết nối hai chiều bằng ping -c 4 192.168.240.131 — thành công, 0% packet loss. 

Các bước thực hiện và kết quả
Mục	Lệnh chính	Kết quả
Host discovery	sudo nmap -sn -n 192.168.240.0/24	Tìm được 4 host đang hoạt động trong dải mạng
TCP Connect scan	nmap -sT -n 192.168.240.131	22 cổng TCP mở
SYN scan	sudo nmap -sS -n 192.168.240.131	Cùng 22 cổng mở, thời gian nhanh hơn -sT, cần quyền root
FIN scan	sudo nmap -sF -n 192.168.240.131	Toàn bộ cổng trả về open|filtered (đúng hành vi RFC 793 trên Linux)
Xmas scan	sudo nmap -sX -n 192.168.240.131	Toàn bộ cổng trả về open|filtered
NULL scan	nmap -sN -n 192.168.240.131	Toàn bộ cổng trả về open|filtered
ACK scan	sudo nmap -sA -n 192.168.240.131	Toàn bộ 1000 cổng ở trạng thái unfiltered → không có firewall chặn gói ACK đến máy này
UDP scan	sudo nmap -sU -n --top-ports 20 192.168.240.131	2 cổng UDP open (53-domain, 137-netbios-ns), một số open|filtered, còn lại closed
Version detection	sudo nmap -sV -n 192.168.240.131	Phát hiện nhiều dịch vụ/phiên bản cũ có rủi ro cao: Apache httpd 2.2.8, ProFTPD 1.3.1, UnrealIRCd, Samba smbd 3.x, MySQL 5.0.51a, PostgreSQL 8.3.0
OS detection	sudo nmap -O -n 192.168.240.131	Nhận diện hệ điều hành: Linux 2.6.9 – 2.6.33 (Device type: general purpose)
Aggressive scan	sudo nmap -A -n 192.168.240.131	Tổng hợp version + OS + script mặc định (smb-os-discovery) + traceroute trong 1 lần quét
NSE – SMB info	sudo nmap -p 445 --script smb-os-discovery -n 192.168.240.131	Lộ thông tin: OS Unix (Samba 3.0.20-Debian), computer name metasploitable, domain localdomain
NSE – MS17-010	sudo nmap -p 445 --script smb-vuln-ms17-010 -n 192.168.240.131	Script không trả kết quả VULNERABLE/không xác định (Metasploitable 2 chạy Samba trên Linux, không phải máy Windows nên script không áp dụng được) — không tự suy diễn là đã vá

Xuất kết quả (.txt)	sudo nmap -sV -O -n 192.168.240.131 -oN ket_qua.txt	Đã lưu file ket_qua.txt
Xuất kết quả (.xml)	sudo nmap -sV -O -n 192.168.240.131 -oX ket_qua.xml	Đã lưu file ket_qua.xml
Xuất grepable + lọc	sudo nmap -p 445 -n 192.168.240.0/24 -oG smb.txt rồi grep "445/open" smb.txt	Lọc ra 2 host mở cổng 445: 192.168.240.1 và 192.168.240.131
Chuyển XML → HTML	xsltproc ket_qua.xml -o bao_cao.html	Đã tạo bao_cao.html
Trước/sau hardening	sudo nmap -sV -n -p 80 192.168.240.131 (before) → tắt Apache trên Metasploitable 2 bằng sudo /etc/init.d/apache2 stop → quét lại cùng lệnh (after)	Before: cổng 80/tcp open, Apache httpd 2.2.8. After: đang thực hiện tắt dịch vụ trên đúng máy Metasploitable 2 — cần chạy lại lệnh quét "after" để hoàn tất so sánh và chụp bằng chứng 

Câu hỏi phân tích
1.Sự khác nhau giữa open, closed và filtered? Open: cổng có dịch vụ đang lắng nghe và phản hồi bắt tay (VD: cổng 80 trên Metasploitable 2 lúc Apache đang chạy). Closed: cổng không có dịch vụ, máy trả lời RST. Filtered: có firewall/ACL chặn nên Nmap không nhận được phản hồi rõ ràng, không biết open hay closed (VD: cổng 445 của máy 192.168.240.254 trong bài trả về "filtered").
2.Tại sao -sS cần quyền cao hơn -sT? -sS phải tự tạo gói SYN thô và đọc phản hồi ở tầng thấp (raw socket) nên cần quyền root; -sT dùng lệnh connect() bình thường của hệ điều hành nên không cần quyền cao. -sS chỉ gửi SYN rồi dừng khi có SYN/ACK (không hoàn tất bắt tay), còn -sT hoàn tất đủ 3 bước bắt tay TCP.
3.Vì sao FIN/Xmas/NULL cho kết quả khó diễn giải? Theo RFC 793, cổng closed phải trả RST, cổng open trên hệ Unix/Linux thì im lặng không phản hồi. Nhưng nhiều hệ điều hành (đặc biệt Windows) không tuân đúng chuẩn này, và firewall cũng có thể chặn im lặng cả hai loại cổng → Nmap không phân biệt được, phải báo "open|filtered".
4.ACK scan trả lời câu hỏi gì khác với SYN scan? ACK scan không xác định cổng open/closed, mà chỉ trả lời "có firewall stateful chặn cổng này không". Nhận RST = "unfiltered"; không phản hồi = "filtered". SYN scan thì xác định thẳng open/closed qua SYN-ACK hay RST.
5.Tại sao UDP scan thường chậm và dễ ra open|filtered? UDP không có bắt tay 3 bước; gửi gói đi mà không có phản hồi thì không biết là cổng open hay bị chặn. Chỉ cổng closed mới chắc chắn trả ICMP port-unreachable. Hệ thống cũng thường giới hạn tốc độ gửi ICMP (rate-limit) nên Nmap phải chờ timeout nhiều lần → chậm.
6.-sV đóng vai trò gì trong quản lý lỗ hổng? -sV xác định phần mềm/phiên bản thực sự chạy sau cổng, vì biết "port 80 mở" chưa nói lên được gì — phải biết đó là Apache 2.2.8 (nhiều lỗ hổng) hay bản mới đã vá. Rủi ro gắn với từng phiên bản cụ thể (CVE), nên quản lý lỗ hổng cần -sV chứ không chỉ số cổng.
7.OS fingerprinting có giới hạn gì? Dựa vào các đặc điểm nhỏ trong stack TCP/IP (TTL, window size, thứ tự option...) — có thể bị NAT, firewall, ảo hóa hoặc tùy biến kernel làm sai lệch. Kết quả chỉ là dự đoán theo xác suất/so khớp cơ sở dữ liệu cũ (trong bài, Nmap đoán "Linux 2.6.9–2.6.33" chứ không khẳng định chính xác), nên không được coi là tuyệt đối.
8.NSE script báo timeout có đồng nghĩa "không có lỗ hổng" không? Không. Timeout chỉ có nghĩa là script không kết nối/phản hồi kịp (do mạng, do bị chặn, do máy đích chậm), không có nghĩa là không có lỗ hổng. Không được tự suy diễn là đã vá hay an toàn.
9.So sánh before/after hardening? Trong bài: trước khi tắt dịch vụ, cổng 80/tcp mở với Apache httpd 2.2.8; sau khi chạy sudo /etc/init.d/apache2 stop trên Metasploitable 2 rồi quét lại, cổng 80 được kỳ vọng chuyển sang closed/không còn dịch vụ trả lời. Sự thay đổi trạng thái cổng từ open → closed là bằng chứng biện pháp phòng thủ có hiệu lực thực sự.
10.Ba cấu hình phòng thủ giảm bề mặt tấn công (không dựa vào "ẩn mình"): (1) Gỡ bỏ/tắt hẳn các dịch vụ không cần thiết; (2) Cấu hình firewall theo nguyên tắc least privilege — chỉ cho phép đúng IP/cổng cần thiết; (3) Cập nhật vá lỗi phần mềm lên phiên bản mới nhất để loại bỏ lỗ hổng đã biết.

Lỗi gặp phải và cách khắc phục
Nmap treo vô thời hạn khi quét theo hostname/IP (sudo nmap -sn 192.168.240.0/24, nmap -sT 192.168.240.131) do mạng Host-Only không có DNS server, Nmap cố phân giải DNS ngược. → Khắc phục bằng cách luôn thêm cờ -n (bỏ qua DNS resolution) vào mọi lệnh quét từ đó về sau.

Chạy nhầm lệnh ip -br addr trên máy Metasploitable 2 (máy này dùng bản ip cũ không hỗ trợ -br) do gõ nhầm tab cửa sổ VM. → Chuyển đúng sang tab Kali để chạy lệnh, dùng ifconfig cho Metasploitable 2.

Gõ nhầm IP mục tiêu: dùng IP ví dụ trong tài liệu gốc (192.168.56.101) thay vì IP thật của máy (192.168.240.131), khiến Nmap báo lỗi setup_target: failed to determine route. → Luôn dùng đúng IP thật lấy được từ ifconfig/ip -br addr, không copy IP mẫu trong tài liệu.

Gõ sai cú pháp lệnh nhiều lần: thiếu dấu - trước tên kỹ thuật quét (gõ sN thay vì -sN), gõ sai tên NSE script (smb-os-disscovery, smd-os-discovery thay vì smb-os-discovery). → Soát lại từng ký tự trước khi Enter, đặc biệt với các lệnh dài có --script.

Chạy lệnh tắt dịch vụ Apache nhầm trên máy Kali (máy tấn công) thay vì máy Metasploitable 2 (máy mục tiêu) khi thực hiện tình huống trước/sau hardening. → Xác nhận lại đúng tab/cửa sổ VM đang thao tác trước khi gõ lệnh có tác động tới hệ thống (đặc biệt các lệnh sudo, stop, start).

Script smb-vuln-ms17-010 không trả kết quả rõ ràng trên Metasploitable 2. → Ghi nhận đúng theo tài liệu: không được tự suy diễn là "đã vá" khi script không xác định được, chỉ ghi là không áp dụng được do mục tiêu không phải Windows/SMB tương thích.

Ghi chú
Toàn bộ lệnh Nmap trong bài đều thay IP mẫu (192.168.56.x) bằng IP thật của môi trường VMware (192.168.240.x) do đổi từ VirtualBox sang VMware.
Không upload mật khẩu, token, API key hay dữ liệu cá nhân thật vào repository.
Phần tình huống trước/sau hardening (mục 11) cần hoàn tất bước quét "after" và chụp ảnh so sánh trước khi nộp bài.
Danh mục ảnh minh chứng bắt buộc (mục 12.1) đã có đủ: ip -br addr trên Kali, ifconfig trên Metasploitable 2, kết quả -sn, kết quả -sS/-sT, kết quả -sV, kết quả -O/-A, một NSE script kèm kết luận, và các tệp kết quả đã lưu (.txt/.xml/.html).
