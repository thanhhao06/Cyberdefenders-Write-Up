author: Azaki
published: published
created: 2026-01-07
tags: CyberDefener, Lab, Network Forensics

An alert from the Intrusion Detection System (IDS) flagged suspicious lateral movement activity involving PsExec. This indicates potential unauthorized access and movement across the network. As a SOC Analyst, your task is to investigate the provided PCAP file to trace the attacker’s activities. Identify their entry point, the machines targeted, the extent of the breach, and any critical indicators that reveal their tactics and objectives within the compromised environment.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefender</b> 
</p>

## Q1: To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?

**Yêu cầu:** Xác định địa chỉ IP của máy mà attacker sử dụng làm điểm xuất phát cho hoạt động lateral movement.

Đầu tiên, mở file PCAP bằng **Wireshark** và kiểm tra các IPv4 conversation:
```
Statistics → Conversations → IPv4
```

![Pasted image 20260816114626.png](./images/Pasted%20image%2020260816114626.png)

Trong kết quả, địa chỉ:
```
10.0.0.130
```

có lượng giao tiếp rất lớn với hai máy nội bộ:
```
10.0.0.133
10.0.0.131
```

Đáng chú ý nhất là connection giữa `10.0.0.130` và `10.0.0.133` với hơn **38.000 packet**, vượt xa các conversation khác.

Vì lab tập trung vào hoạt động lateral movement bằng PsExec, đây là dấu hiệu cho thấy `10.0.0.130` đang chủ động giao tiếp với các host khác trong mạng và cần được xem là điểm xuất phát của cuộc điều tra.

>10.0.0.130

## Q2: To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?

**Yêu cầu:** Xác định hostname của máy đầu tiên mà attacker pivot sang.

Do PsExec hoạt động thông qua SMB, filter traffic bằng:
```
smb2
```

Sau đó quan sát quá trình:
```
Session Setup
```
giữa `10.0.0.130` và `10.0.0.133`.

![Pasted image 20260816114631.png](./images/Pasted%20image%2020260816114631.png)

Trong SMB2 Session Setup Response, mở:
```
Security Blob
→ GSS-API
→ NTLM Secure Service Provider
→ NTLMSSP_CHALLENGE
```

Tại trường:
```
Target Name
```

Wireshark hiển thị:
```
SALES-PC
```

>Sales-PC

## Q3: Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?

**Yêu cầu:** Xác định username được attacker sử dụng để authenticate tới máy mục tiêu.

Tiếp tục theo dõi SMB2 Session Setup giữa:
```
10.0.0.130 → 10.0.0.133
```

Trong packet chứa:
```
NTLMSSP_AUTH
```

Wireshark hiển thị trực tiếp username được sử dụng cho quá trình authentication
![Pasted image 20260816114635.png](./images/Pasted%20image%2020260816114635.png)

Tại cột Info có thể thấy:
```
User: \ssales
```

Điều này cho thấy attacker đang sử dụng credential của tài khoản:
```
ssales
```
để xác thực tới `Sales-PC`.

Việc PsExec được sử dụng với một tài khoản hợp lệ là đặc trưng thường gặp trong lateral movement, vì attacker cần quyền thích hợp trên remote system để truy cập administrative share và tạo service.

>ssales

## Q4: After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?

**Yêu cầu:** Xác định executable được PsExec triển khai lên máy mục tiêu để phục vụ remote execution.

Tiếp tục kiểm tra SMB2 traffic sau khi authentication thành công.

Trong các SMB file operation xuất hiện một file đáng chú ý:
```
PSEXESVC.exe
```

![Pasted image 20260816114643.png](./images/Pasted%20image%2020260816114643.png)

Có thể quan sát chuỗi thao tác:
```
Create Request  → PSEXESVC.exe
Create Response → PSEXESVC.exe
Write Request   → PSEXESVC.exe
```

Điều này cho thấy file được tạo và ghi dữ liệu lên remote system.

PsExec sử dụng một temporary Windows service trên máy đích để thực thi command từ xa; MITRE cũng ghi nhận PsExec sử dụng temporary Windows service cho **Service Execution**.

>psexesvc

## Q5: We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?

**Yêu cầu:** Xác định network share được PsExec sử dụng để đưa service executable lên máy mục tiêu.

Quan sát các SMB2 **Tree Connect Request** xuất hiện ngay trước hoạt động tạo `PSEXESVC.exe`.
![Pasted image 20260816114646.png](./images/Pasted%20image%2020260816114646.png)

Có thể thấy máy `10.0.0.130` kết nối tới:
```
\\10.0.0.133\ADMIN$
```

`ADMIN$` là một Windows administrative share. PsExec có thể sử dụng share này để copy executable sang remote host trước khi đăng ký và khởi chạy service.

MITRE ATT&CK cũng ghi nhận PsExec có khả năng ghi chương trình vào `ADMIN$` để thực hiện remote execution.

>ADMIN$

## Q6: We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?

**Yêu cầu:** Xác định network share được PsExec sử dụng cho hoạt động giao tiếp giữa hai máy.

Quay lại các SMB2 Tree Connect Request trước khi attacker truy cập `ADMIN$`.
![Pasted image 20260816114649.png](./images/Pasted%20image%2020260816114649.png)

Có thể thấy một connection được thiết lập tới:
```
\\10.0.0.133\IPC$
```

`IPC$`, hay **Inter-Process Communication share**, là một special share của Windows được sử dụng cho các hoạt động giao tiếp từ xa, bao gồm named pipes và RPC.

Trong hoạt động PsExec, SMB và IPC có thể được sử dụng để thiết lập communication channel giữa máy điều khiển và service chạy trên remote host.

MITRE liệt kê cả `ADMIN$` và `IPC$` là các Windows administrative shares có thể được sử dụng trong hoạt động remote access qua SMB.

>IPC$

## Q7: Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?

**Yêu cầu:** Xác định hostname của máy thứ hai mà attacker tiếp tục nhắm tới sau `Sales-PC`.

Từ phần IPv4 Conversations ban đầu, ngoài connection giữa:
```
10.0.0.130 ↔ 10.0.0.133
```

còn có lượng traffic đáng kể giữa:
```
10.0.0.130 ↔ 10.0.0.131
```

Điều này cho thấy attacker tiếp tục tương tác với một host nội bộ khác.

Để xác định hostname tương ứng với `10.0.0.131`, filter các packet liên quan đến SMB/NetBIOS Browser traffic.
![Pasted image 20260816114654.png](./images/Pasted%20image%2020260816114654.png)

Trong các packet **Host Announcement**, Wireshark hiển thị hostname:
```
MARKETING-PC
```

và source IP của các announcement này là:
```
10.0.0.131
```

Như vậy có thể ánh xạ:
```
10.0.0.131 → MARKETING-PC
```

Đây là hệ thống thứ hai mà attacker nhắm tới trong quá trình pivot.

CyberDefenders cũng mô tả cuộc điều tra kết thúc bằng việc phát hiện một nỗ lực pivot tiếp theo sang máy thứ hai.

>Marketing-PC

---

# ATTACK CHAIN

![PsExec Hunt Lab.png](./images/PsExec%20Hunt%20Lab.png)