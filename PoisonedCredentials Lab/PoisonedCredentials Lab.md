author: Azaki
published: published
created: 2026-10-02
tags: CyberDefener, Lab, Network Forensic

Your organization's security team has detected a surge in suspicious network activity. There are concerns that LLMNR (Link-Local Multicast Name Resolution) and NBT-NS (NetBIOS Name Service) poisoning attacks may be occurring within your network. These attacks are known for exploiting these protocols to intercept network traffic and potentially compromise user credentials. Your task is to investigate the network logs and examine captured network traffic.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefenders</b> 
</p>

## Q1: In the context of the incident described in the scenario, the attacker initiated their actions by taking advantage of benign network traffic from legitimate machines. Can you identify the specific mistyped query made by the machine with the IP address 192.168.232.162?

**Yêu cầu:** Xác định truy vấn bị gõ sai chính tả xuất phát từ máy nạn nhân có địa chỉ IP `192.168.232.162`

Khi người dùng trên Windows gõ sai tên một máy chủ nội bộ hoặc chia sẻ tệp, hệ thống không tìm thấy bản ghi tương ứng trong bộ nhớ đệm DNS nên sẽ tự động kích hoạt cơ chế phân giải tên dự phòng qua mạng cục bộ như LLMNR cổng UDP 5355, NetBIOS Name Service cổng UDP 137 và mDNS cổng UDP 5353

Mở file lưu lượng mạng `PoisonedCredentials.pcap` bằng Wireshark và áp dụng bộ lọc theo địa chỉ IP nguồn của máy nạn nhân
```text
ip.src == 192.168.232.162
```

Kiểm tra các gói tin truy vấn phân giải tên broadcast và multicast. Tại gói tin số 47 (NBNS), gói 48, 49 (MDNS) cùng gói 52, 53 (LLMNR), hệ thống liên tục gửi các truy vấn tìm kiếm hostname bị gõ sai là `fileshaare` thay vì `fileshare`
![Pasted image 20261002232020.png](./images/Pasted%20image%2020261002232020.png)

>fileshaare

## Q2: We are investigating a network security incident. To conduct a thorough investigation, We need to determine the IP address of the rogue machine. What is the IP address of the machine acting as the rogue entity?

**Yêu cầu:** Xác định địa chỉ IP của máy độc hại thực hiện hành vi đầu độc mạng

Trong các cuộc tấn công LLMNR và NBT-NS Poisoning thường thực hiện bằng công cụ Responder, kẻ tấn công liên tục lắng nghe các gói tin truy vấn broadcast trong cùng subnet và lập tức gửi phản hồi giả mạo tự nhận mình nắm giữ tài nguyên đó để lừa máy nạn nhân gửi dữ liệu xác thực về máy của mình

Lọc các gói tin thuộc giao thức NetBIOS Name Service bằng bộ lọc
```text
nbns
```

Ngay sau truy vấn tìm kiếm `FILESHAARE<20>` của nạn nhân ở gói 47, tại gói tin số 51 xuất hiện phản hồi `Name query response` với cờ Authoritative khẳng định hostname `FILESHAARE<20>` tương ứng với địa chỉ IP `192.168.232.215`
![Pasted image 20261002232930.png](./images/Pasted%20image%2020261002232930.png)

>192.168.232.215

## Q3: As part of our investigation, identifying all affected machines is essential. What is the IP address of the second machine that received poisoned responses from the rogue machine?

**Yêu cầu:** Xác định địa chỉ IP của máy nạn nhân thứ hai đã nhận các gói tin phản hồi đầu độc từ máy rogue

Để xác định các máy trạm khác trong mạng bị ảnh hưởng bởi cuộc tấn công đầu độc, tiến hành lọc toàn bộ các phản hồi NBNS do máy tấn công `192.168.232.215` gửi đi

Áp dụng bộ lọc tìm các gói tin NBNS có địa chỉ liên quan đến máy rogue
```text
nbns.addr == 192.168.232.215
```

Bên cạnh nạn nhân đầu tiên là `192.168.232.162` ở gói tin số 51, từ gói tin số 145 trở đi xuất hiện hàng loạt gói `Name query response` được máy rogue gửi tới địa chỉ đích `192.168.232.176` để đầu độc các truy vấn CYBERCACTUS và PRINETR
![Pasted image 20261002233028.png](./images/Pasted%20image%2020261002233028.png)

>192.168.232.176

## Q4: We suspect that user accounts may have been compromised. To assess this, we must determine the username associated with the compromised account. What is the username of the account that the attacker compromised?

**Yêu cầu:** Xác định tên tài khoản người dùng đã bị kẻ tấn công chiếm đoạt

Sau khi lừa nạn nhân tin rằng máy rogue là máy chủ tệp hợp lệ, nạn nhân tự động khởi tạo kết nối SMB qua cổng TCP 445 và thực hiện quá trình bắt tay xác thực NTLMSSP để truy cập chia sẻ mạng. Thông tin tài khoản cùng hash NetNTLMv2 sẽ được truyền tải qua phiên này

Sử dụng bộ lọc giao thức SMB2 để quan sát luồng xác thực giữa máy nạn nhân `192.168.232.176` và máy rogue `192.168.232.215`
```text
smb2
```

Kiểm tra gói tin số 242 ứng với lệnh `Session Setup Request, NTLMSSP_AUTH`. Trong phần chi tiết `SMB2 Header`, trường `Session Id` hiển thị tài khoản thực hiện xác thực là `Acct: janesmith` thuộc domain `cybercactus.local`
![Pasted image 20261002233608.png](./images/Pasted%20image%2020261002233608.png)

>janesmith

## Q5: As part of our investigation, we aim to understand the extent of the attacker's activities. What is the hostname of the machine that the attacker accessed via SMB?

**Yêu cầu:** Xác định hostname của máy tính mà kẻ tấn công đã truy cập thông qua dịch vụ SMB

Trong quá trình thương lượng xác thực NTLMSSP qua SMB, gói tin phản hồi `Session Setup Response, Error: STATUS_MORE_PROCESSING_REQUIRED, NTLMSSP_CHALLENGE` chứa cấu trúc `Target Info` với các thuộc tính định danh chi tiết của máy trạm bao gồm NetBIOS computer name và DNS computer name

Lọc các gói tin SMB2 gửi tới máy rogue bằng cú pháp
```text
ip.dst == 192.168.232.215 && smb2
```

Quan sát gói tin số 241, mở rộng trường `NTLM Secure Service Provider -> Target Info`. Thuộc tính `Attribute: NetBIOS computer name` ghi nhận rõ tên máy tính là `AccountingPC`
![Pasted image 20261002234635.png](./images/Pasted%20image%2020261002234635.png)

>AccountingPC