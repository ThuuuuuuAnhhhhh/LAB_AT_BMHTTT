# LAB 7

Mon hoc: An toan va bao mat he thong thong tin
Sinh vien: Ma Thi Thu Anh - 1150070001 - 11DH_HTTT
EM CHỈ MỚI CÀI ĐẶT THÔI THẦY , BÀI CỦA EM CHƯA CÓ GÌ MẤY THẦY DỪNG TRẢ LẠI NHA THẦY 
Kiến trúc
DC01 (DNS, 10.6.0.10) → DMZ-WEB (nginx, website giả lập lab7.test, 172.16.6.10)
Client (10.6.0.21/22) → pfSense (10.6.0.1) → WEB-PROXY (10.6.0.30, Squid+E2Guardian+ClamAV, cổng 8080) → Internet/DMZ-WEB 

 Đã hoàn thành
- DNS: 13 bản ghi A cho *.lab7.test trên DC01 (script Add-LAB7-DNS.ps1), trỏ về 172.16.6.10
- DMZ-WEB: nginx virtual host cho từng domain, file test (EICAR, 10/100/110MB)
- WEB-PROXY: Squid (127.0.0.1:3128) + E2Guardian (10.6.0.30:8080) + ClamAV
- pfSense: alias LAB7_PROXY/POLICY_CLIENTS/WEB_PORTS có sẵn từ LAB6; rule chặn direct 80/443 + allow client→proxy + allow proxy→Any đã đúng, không cần sửa
- Policy chặn domain: dùng Squid ACL (dstdomain + http_access deny), test thành công: normal (allow), china/weapons (block)
- Test end-to-end từ client thật (CLIENT_A 10.6.0.21): đúng hành vi

 Vấn đề đã gặp & cách fix
1. DNS resolv.conf trên WEB-PROXY bị NetworkManager revert mất nameserver → set lại qua nmcli
2. e2guardian.conf bị duplicate "filterports=8080" do sed patch lỗi → gây bind EADDRINUSE → xoá dòng trùng
3. ClamAV "Access denied" khi scan: clamd chạy user `clamav` không đọc được file tmp do e2guardian (chạy root) tạo → comment `User clamav` trong clamd.conf để chạy root (chỉ chấp nhận được trong môi trường lab cô lập)
4. E2Guardian site-list (bannedsitelist) KHÔNG hoạt động dù config đúng, list đúng, process mới — nguyên nhân gốc chưa tìm ra (đã loại trừ: CRLF, process cũ, quyền file, group mapping). → Workaround: chặn domain bằng Squid ACL (dstdomain + http_access deny) thay vì dựa vào E2Guardian site-list
5. Squid ACL phải đặt TRƯỚC rule "http_access allow localhost/localnet" (Squid match rule đầu tiên khớp, không phải rule cụ thể nhất)

    Hạn chế / chưa làm (so với ma trận kiểm thử)
- Warning/temporary bypass (weapons/politics/religion/streaming): hiện đang HARD BLOCK thay vì cảnh báo+bypass tạm, do phải dùng Squid ACL thay cho E2Guardian site-list bị lỗi
- Chưa test riêng: mideast, jobs, politics, religion, blockedgame (cùng ACL với china/weapons nên về lý thuyết hoạt động giống, nhưng chưa confirm từng domain)
- Antivirus qua HTTP thật: chưa test (ClamAV standalone đã confirm detect EICAR OK)
- Entertainment/Game theo schedule 60 phút: chưa làm
- Giới hạn file >100MB (reply_body_max_size): chưa làm
- Custom block page: chưa làm (đang dùng trang lỗi mặc định Squid)
- HTTPS inspection (CA/MITM): chưa làm (optional)
(Se cap nhat noi dung khi thuc hien Lab 7)
