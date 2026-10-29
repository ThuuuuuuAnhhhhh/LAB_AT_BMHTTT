# LAB 8

Mon hoc: An toan va bao mat he thong thong tin
Sinh vien: Ma Thi Thu Anh - 1150070001 - 11DH_HTTT

I. HẠ TẦNG DNSLAB

Mạng cô lập 10.9.0.0/24, pfSense interface DNSLAB (10.9.0.1), gồm:

DNS_SERVER_LAB9 (10.9.0.10) — chạy BIND9, zone master lab9.test, zone reverse 0.9.10.in-addr.arpa, zone forward external.lab9.test → forward tới 172.16.6.20
VICTIM-LAB9 (10.9.0.100) — nameserver trỏ về 10.9.0.10
MAANH-LINUX-Attacker (10.9.0.200) — chạy scapy/wireshark/tcpdump trên card DNS-LAB
APP-TEST/AUTH-DNS (172.16.6.20, kế thừa DMZ-WEB) — chạy script Python mô phỏng authoritative DNS có độ trễ (auth_dns_delayed.py --delay 0.8)

Các sự cố kỹ thuật đã xử lý trong quá trình dựng lab: xung đột VMware scoreboard file khi clone VM, lỗi tên zone reverse (9.10.in-addr.arpa → sửa thành 0.9.10.in-addr.arpa), named.service thay vì bind9.service, thiếu default gateway trên DNS_SERVER_LAB9, nghẽn backlog do BIND retry nhanh hơn độ trễ phản hồi của script (khắc phục bằng đa luồng + giảm delay xuống 0.8s), DNSSEC validation chặn phản hồi forward-zone không ký (tắt bằng dnssec-validation no), và zone lab9.test che khuất zone forward external.lab9.test (khắc phục bằng thêm bản ghi NS ủy quyền external IN NS ns.lab9.test. trong db.lab9.test).

II. KẾT QUẢ THỰC HIỆN TASK

Task 1 — Hosts file redirect:

echo "10.9.0.200 www.external.lab9.test" | sudo tee -a /etc/hosts
curl http://www.external.lab9.test
→ Kết quả: "FAKE SITE - LAB9 DNS ATTACK" (thành công)

Task 2 — DNS spoofing trực tiếp (mode victim):

sudo python3 dns_spoof_lab9.py --mode victim --iface eth1

Log attacker: [LAB9] forged victim response: www.external.lab9.test. → 10.9.0.200
VICTIM (đã xoá /etc/hosts, dùng DNS thật) vẫn nhận FAKE SITE → spoof thành công qua mạng thật.

Task 3 — DNS cache poisoning (mode resolver):

sudo python3 dns_spoof_lab9.py --mode resolver --iface eth1

Log attacker: [LAB9] forged resolver response: www.external.lab9.test. → 10.9.0.200
Sau khi tắt hẳn attacker, dig @10.9.0.10 www.external.lab9.test vẫn trả 10.9.0.200 → chứng minh cache DNS_SERVER_LAB9 bị đầu độc thật sự, không phải thắng race ngẫu nhiên.
Khôi phục: sudo rndc flush → trả lại đúng 172.16.6.10.

III. 10 CÂU HỎI PHÂN TÍCH

(Se cap nhat noi dung khi thuc hien Lab 8)
