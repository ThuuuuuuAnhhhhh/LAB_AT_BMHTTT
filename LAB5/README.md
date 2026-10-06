# LAB 5

Mon hoc: An toan va bao mat he thong thong tin
Sinh vien: Ma Thi Thu Anh - 1150070001 - 11DH_TMDT 

Tóm tắt toàn bộ:  ĐÃ HOÀN THÀNH 3 TÌNH HUỐNG 

YOUTUBE : ĐÃ CẬP NHẬT VÀO WORD - HẲN 2 PHẦN LUÔN 

Lỗi	Nguyên nhân	Cách xử lý
1. Lỗi hướng dẫn ban đầu theo kiểu VirtualBox

Trong quá trình thực hiện, tài liệu hướng dẫn ban đầu được xây dựng cho môi trường VirtualBox nên một số thao tác không tương thích với VMware Workstation. Để khắc phục, các bước cấu hình được chuyển đổi sang thao tác tương đương trên VMware Workstation, đặc biệt là phần cấu hình máy ảo và các Network Adapter.

2. Lỗi Network Adapter 2 và 3 cùng sử dụng VMnet3

Khi cấu hình mạng cho máy ảo, Network Adapter 2 và Network Adapter 3 bị đặt cùng sử dụng VMnet3 do chọn nhầm trong danh sách dropdown. Điều này làm sai cấu trúc mạng theo thiết kế ban đầu. Lỗi được khắc phục bằng cách thay đổi Network Adapter 2 sang VMnet2, đảm bảo mỗi adapter kết nối đúng với mạng tương ứng.

3. Lỗi không có file ISO cài đặt

Trong quá trình cài đặt, máy chưa có file ISO cần thiết mà chỉ có file placeholder. Nguyên nhân là file ISO chưa được tải về đầy đủ. Để xử lý, sử dụng shortcut có đuôi .url đi kèm để truy cập nguồn tải và tải file .iso.gz với dung lượng khoảng 548 MB. Sau khi tải hoàn tất, tiến hành giải nén để lấy file ISO phục vụ cài đặt.

4. Lỗi không có 7-Zip để giải nén file .iso.gz

Máy tính không được cài đặt 7-Zip nên không thể thực hiện giải nén file .iso.gz theo hướng dẫn ban đầu. Tuy nhiên, máy đã có WinRAR nên sử dụng WinRAR để thay thế. Thực hiện nhấp chuột phải vào file .iso.gz và chọn Extract Here để giải nén và lấy file ISO.

5. Lỗi ô “Connected” của CD/DVD bị vô hiệu hóa

Trong VMware Workstation, tùy chọn Connected của thiết bị CD/DVD bị làm mờ và không thể thao tác khi máy ảo đang tắt. Trong trường hợp này, sử dụng tùy chọn Connect at power on để thiết bị tự động kết nối với file ISO khi khởi động máy ảo. Lưu ý, tùy chọn Connected chỉ có thể bật hoặc tắt trực tiếp khi máy ảo đang chạy.

6. Lỗi rule “Block LAN còn lại” không giữ đúng vị trí khi kéo-thả

Trong Tình huống 2, khi kéo-thả rule “Block LAN còn lại” để thay đổi thứ tự, pfSense không giữ ổn định vị trí của rule sau khi thao tác. Thay vì tiếp tục sắp xếp thủ công, phương án xử lý là Disable (Toggle disable) rule “Baseline Pass LAN to any”. Khi rule này được tắt, rule Block vẫn được áp dụng và đảm bảo mục tiêu kiểm soát lưu lượng mạng mà không phụ thuộc hoàn toàn vào vị trí hiển thị của rule.

7. Lỗi ping từ DMZ đến LAN bị mất 100% gói tin

Trong Tình huống 3, khi thực hiện ping từ DMZ 172.16.0.2 đến LAN 10.0.0.2, kết quả luôn là 100% packet loss, mặc dù rule Pass DMZ đã được cấu hình đúng. Qua kiểm tra bằng Packet Capture xác nhận interface DMZ vẫn nhận được gói tin, ARP đã phân giải đúng địa chỉ MAC và bảng định tuyến không phát hiện bất thường. Tuy nhiên, firewall log không ghi nhận cả rule Pass lẫn Block, cho thấy gói tin bị loại bỏ ở tầng thấp hơn firewall. Nguyên nhân được xác định là hardware checksum offload không tương thích giữa pfSense (FreeBSD) và card mạng ảo của VMware.

Để khắc phục, truy cập System → Advanced → Networking, chọn “Disable hardware checksum offload”, sau đó lưu cấu hình và khởi động lại pfSense. Sau khi reboot, thực hiện ping lại từ DMZ đến LAN và kết quả thành công với 0% packet loss. 

Tình huống	Nội dung	Trạng thái
1	Block ICMP, Pass DNS/Web	Hoàn thành — ping fail, nslookup/curl OK
2	Giới hạn 1 máy (DC) truy cập Internet	Hoàn thành — DC curl 301, Kali curl 000
3	Cách ly DMZ khỏi LAN	Hoàn thành — baseline OK → block → fail, Internet vẫn OK 

Khó khăn chính gặp phải (theo từng tình huống)

Tình huống 1 — Block ICMP, Pass DNS/Web: không gặp khó khăn đáng kể, làm đúng theo lý thuyết (Block ICMP trên, Pass DNS/Web dưới) là chạy được ngay.

Tình huống 2 — Giới hạn 1 máy truy cập Internet: khó khăn nằm ở sắp xếp thứ tự rule. Kéo-thả (drag-and-drop) để đưa rule "Block LAN còn lại" lên đúng vị trí bị lỗi liên tục — rule cứ nhảy về đầu (chặn nhầm cả DC) hoặc về cuối (vô hiệu vì nằm dưới rule Pass tổng quát), thử cả 4 lần bằng kéo-thả lẫn checkbox+Add đều không giữ đúng thứ tự. → Cách khắc phục: không sắp xếp lại nữa, mà tắt hẳn (Toggle disable) rule "Baseline Pass LAN to any" — khi không còn rule Pass tổng quát nào đứng trước, rule Block phát huy tác dụng bất kể vị trí chính xác.

Tình huống 3 — Cách ly DMZ khỏi LAN: đây là phần khó nhất, mất nhiều bước loại trừ nhất. Ping DMZ (172.16.0.2) → LAN (10.0.0.2) baseline cứ 100% packet loss dù mọi thứ nhìn bề ngoài đều đúng. Đã lần lượt loại trừ:

Card mạng Metasploitable2 bị dư 1 adapter → đã gỡ nhưng không hết lỗi
Windows Firewall trên DC chặn ICMP → đã mở rule allow, thậm chí tắt hẳn firewall → vẫn không hết
Route/gateway trên Metasploitable2 bị mất sau reboot → kiểm tra route -n thấy vẫn đúng → không phải nguyên nhân
Rule pfSense trên DMZ sai/thiếu → kiểm tra lại thấy rule Pass DMZ→any vẫn đúng, không có rule Block nào chặn trước nó
ARP chưa phân giải được MAC của DC → kiểm tra ARP Table thấy pfSense đã có MAC của DC rồi
Bảng định tuyến (routes) sai → kiểm tra thấy route 10.0.0.0/8 qua LAN và 172.16.0.0/16 qua DMZ đều sạch, không có gì bất thường

→ Nguyên nhân thật sự: dùng Packet Capture (Diagnostics → Packet Capture) phát hiện gói ICMP từ DMZ có đến cổng pfSense nhưng không hề được chuyển tiếp sang LAN, và khi bật log cho rule Pass DMZ thì không hề có log nào (không Pass cũng không Block) — chứng tỏ gói bị rớt ở tầng thấp hơn cả bộ lọc firewall. Đây là lỗi kinh điển hardware checksum offload giữa pfSense (FreeBSD) và card mạng ảo của VMware (card ảo "hứa" sẽ tính checksum nhưng không tính kịp khi đi qua switch ảo, khiến FreeBSD coi gói là hỏng và âm thầm loại bỏ). → Cách khắc phục: System → Advanced → Networking → tích "Disable hardware checksum offload" → Save → Reboot pfSense. Sau khi khởi động lại, ping DMZ→LAN thành công ngay lập tức (0% packet loss).

Bài học rút ra: khi pfSense "pass" được ở interface nguồn (thấy packet đến, interface counter tăng) nhưng traffic vẫn không tới được đích, nên nghi ngờ tầng thấp hơn firewall rule (driver/NIC ảo) chứ không chỉ loanh quanh sửa rule — dùng Packet Capture + bật log trên rule là cách xác định nhanh nhất traffic bị rớt ở đâu.
