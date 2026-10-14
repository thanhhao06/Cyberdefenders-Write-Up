author: Azaki
published: published
created: 2025-12-29
tags: CyberDefener, Lab, Network Forensics


TechNova Systems’ SOC has detected suspicious outbound traffic from a public-facing IIS server in its cloud platform—activity suggestive of a web-shell drop and covert connections to an unknown host.

As the forensic examiner, you have three critical artefacts in hand: a PCAP capturing the initial traffic, a full memory image of the server, and a malware sample recovered from disk. Reconstruct the intrusion and all of the attacker’s activities so TechNova can contain the breach and strengthen its defenses.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@whitewolfx</b> 
</p>

## PCAP Analysis
### Q1: After flooding the IIS host with rapid-fire probes, the attacker reveals their origin. Which IP address generated this reconnaissance traffic?

**Yêu cầu:** Xác định địa chỉ IP của attacker thực hiện hoạt động reconnaissance đối với IIS server.

Đầu tiên, mở file PCAP bằng **Wireshark** và kiểm tra các conversation giữa những host trong mạng:
```
Statistics → Conversations → IPv4
```

![Pasted image 20260816110655.png](./images/Pasted%20image%2020260816110655.png)

Trong danh sách có thể thấy connection giữa:
```
10.0.2.4 ↔ 10.0.2.15
```
có số lượng packet vượt trội, với hơn `6000` packet được trao đổi.

Do đề bài cho biết attacker thực hiện một lượng lớn probe liên tục vào IIS host, đây là conversation đáng chú ý nhất cần tiếp tục điều tra.

Kết hợp với các traffic SMB và reverse shell ở những bước sau, có thể xác định `10.0.2.4` là hệ thống của attacker, còn `10.0.2.15` là IIS server bị compromise.

>10.0.2.4

### Q2: Zeroing in on a single open service to gain a foothold, the attacker carries out targeted enumeration. Which MITRE ATT&CK technique ID covers this activity?

**Yêu cầu:** Xác định MITRE ATT&CK Technique mô tả hành vi attacker dò tìm các service đang mở trên hệ thống mục tiêu.

Hành vi scan một host nhằm xác định các port và network service đang hoạt động thuộc tactic **Discovery**.

Tra cứu MITRE ATT&CK với các từ khóa liên quan đến:
```
network
service
discovery
port scan
```

có thể xác định technique: **Network Service Discovery**
![Pasted image 20260816110659.png](./images/Pasted%20image%2020260816110659.png)

Technique này mô tả hành vi attacker thu thập danh sách các service đang chạy trên remote host hoặc network infrastructure để tìm những mục tiêu có khả năng bị khai thác.

>T1046

### Q3: While reviewing the SMB traffic, you observe two consecutive Tree Connect requests that expose the first shares the intruder probes on the IIS host. Which two full UNC paths are accessed?

**Yêu cầu:** Xác định hai SMB share mà attacker truy cập trên IIS server.

Tiếp tục phân tích network traffic. Vì hoạt động liên quan đến SMB phiên bản mới nên sử dụng filter:
```
smb2
```

Sau đó tìm các packet có operation:
```
Tree Connect Request
```

![Pasted image 20260816110702.png](./images/Pasted%20image%2020260816110702.png)

Trong các request có thể quan sát hai UNC path:
```
\\10.0.2.15\IPC$
```

và:
```
\\10.0.2.15\Documents
```

`IPC$` là một administrative share thường được sử dụng cho named pipes và các hoạt động IPC trong Windows, trong khi `Documents` là một file share có khả năng chứa hoặc cho phép attacker ghi dữ liệu.

Đây là hai share attacker thăm dò trước khi chuyển sang giai đoạn upload payload

>`\\10.0.2.15\Documents, \\10.0.2.15\IPC$`

### Q4: Inside the share, the attacker plants a web-accessible payload that will grant remote code execution. What is the filename of the malicious file they uploaded, and what byte length is specified in the corresponding SMB2 Write Request?

**Yêu cầu:** Xác định tên malicious file được attacker upload và kích thước dữ liệu trong SMB2 Write Request tương ứng.

Tiếp tục theo dõi SMB2 traffic sau khi attacker kết nối vào share `Documents`.

Tìm các packet có operation:
```
Write Request
```

Trong traffic xuất hiện một file đặc biệt đáng chú ý:
```
shell.aspx
```

![Pasted image 20260816110706.png](./images/Pasted%20image%2020260816110706.png)

Packet Info hiển thị:
```
Write Request Len:1015024 Off:0, File: shell.aspx
```

Như vậy attacker đã ghi một file ASP.NET có tên `shell.aspx` lên server với byte length:
```
1015024
```

File `.aspx` được đặt trên IIS server có thể được server xử lý như một web application. Trong bối cảnh cuộc tấn công, file này đóng vai trò **web shell**, cung cấp cho attacker khả năng thực thi code từ xa trên máy chủ.

>shell.aspx, 1015024

### Q5: The newly planted shell calls back to the attacker over an uncommon but firewall-friendly port. Which listening port did the attacker use for the reverse shell?

**Yêu cầu:** Xác định port mà attacker sử dụng để lắng nghe reverse shell.

Từ các bước trước đã xác định:
```
Attacker: 10.0.2.4
Victim:   10.0.2.15
```

Tiếp tục filter TCP traffic liên quan tới attacker:
```
ip.src == 10.0.2.4 && tcp
```

![Pasted image 20260816110709.png](./images/Pasted%20image%2020260816110709.png)

Có thể thấy một lượng lớn packet từ:
```
10.0.2.4:4443
```

được gửi tới một ephemeral port trên victim:
```
10.0.2.15:49688
```

Trong một reverse shell, victim chủ động thiết lập connection tới listener của attacker. Sau khi kết nối được thiết lập, các packet phản hồi từ attacker sẽ có **source port** chính là listening port.

>4443

## Memory Dump Aanlysis
### Q6: Your memory snapshot captures the system’s kernel in situ, providing vital context for the breach. What is the kernel base address in the dump?

**Yêu cầu:** Xác định kernel base address từ memory dump của hệ thống.

Chuyển sang memory forensics với **Volatility 2**.

Có thể sử dụng plugin:
```
python2 vol.py -f memdump.mem --profile=Win10x64_17134 kdbgscan
```

![Pasted image 20260816110713.png](./images/Pasted%20image%2020260816110713.png)

Trong kết quả `kdbgscan` có thể thấy:
```
KernelBase : 0xfffff80079213000
```

Đây là địa chỉ kernel được Volatility 2 biểu diễn ở dạng canonical 64-bit.

Tuy nhiên, format mà challenge yêu cầu rút gọn phần prefix thành:
```
0xf80079213000
```

>0xf80079213000

### Q7: A trusted service launches an unfamiliar executable residing outside the usual IIS stack, signalling a persistence implant. What is the final full on-disk path of that executable, and which MITRE ATT&CK persistence technique ID corresponds to this behaviour?

**Yêu cầu:** Xác định đường dẫn đầy đủ của persistence implant và MITRE ATT&CK Technique tương ứng.

Tiếp tục phân tích memory dump để tìm command line của các process đáng ngờ.

Sử dụng plugin:
```
python2 vol.py -f memdump.mem --profile=Win10x64_17134 cmdline
```

và filter:
```
grep "updatenow.exe"
```

![Pasted image 20260816110717.png](./images/Pasted%20image%2020260816110717.png)

Kết quả cho thấy process:
```
updatenow.exe
```

được thực thi từ:
```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe
```

Thư mục:
```
...\Programs\Startup\
```
là Windows Startup Folder. Các executable được đặt trong thư mục này có thể tự động được chạy khi người dùng đăng nhập.

Do đó attacker đã lợi dụng cơ chế autostart của Windows để duy trì **Persistence** trên hệ thống.

Technique cấp cao tương ứng là:
```
T1547 – Boot or Logon Autostart Execution
```

Nếu mapping chi tiết theo MITRE ATT&CK, hành vi sử dụng Startup Folder thuộc sub-technique:
```
T1547.001 – Registry Run Keys / Startup Folder
```

![Pasted image 20260816111923.png](./images/Pasted%20image%2020260816111923.png)

>C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe, T1547

### Q8: The reverse shell’s outbound traffic is handled by a built-in Windows process that also spawns the implanted executable. What is the name of this process, and what PID does it run under?

**Yêu cầu:** Xác định process xử lý reverse shell và PID của process đó.

Tiếp tục kiểm tra process list trong memory dump.

Trong kết quả có thể thấy:
```
w3wp.exe       PID 4332
updatenow.exe  PID 900
```

![Pasted image 20260816110720.png](./images/Pasted%20image%2020260816110720.png)

Điểm quan trọng là `updatenow.exe` có **PPID 4332**, trùng với PID của:
```
w3wp.exe
```

Điều này cho thấy `updatenow.exe` là child process được spawn từ `w3wp.exe`.

`w3wp.exe` là **IIS Worker Process**, chịu trách nhiệm xử lý request của web application trên Microsoft IIS.

Trong bối cảnh này, việc IIS Worker Process spawn một executable lạ từ thư mục Startup là dấu hiệu rất đáng ngờ và phù hợp với việc attacker đã thực thi code thông qua `shell.aspx`.

>w3wp.exe, 4332

## Malware Sample Analysis
### Q9: Static inspection reveals the binary has been packed to hinder analysis. Which packer was used to obfuscate it?

**Yêu cầu:** Xác định packer được sử dụng để đóng gói `updatenow.exe`.

Tiến hành static analysis mẫu malware bằng **Detect It Easy (DIE)**.
![Pasted image 20260816110725.png](./images/Pasted%20image%2020260816110725.png)

DIE nhận diện:
```
Packer: UPX(3.91)[NRV, brute]
```

Ngoài ra, heuristic cũng phát hiện các đặc điểm của binary đã bị compress hoặc packed.

**UPX – Ultimate Packer for eXecutables** là một executable packer phổ biến. Trong malware analysis, packer có thể được sử dụng để làm thay đổi cấu trúc binary và gây khó khăn hơn cho quá trình static analysis

>UPX

### Q10: Threat-intel analysis shows the malware beaconing to its command-and-control host. Which fully qualified domain name (FQDN) does it contact?

**Yêu cầu:** Xác định FQDN của Command-and-Control server mà malware liên lạc.

Sau khi có mẫu `updatenow.exe`, lấy hash và tra cứu trên **VirusTotal**.

Trong tab:
```
Relations
```

kiểm tra phần:
```
Contacted Domains
```

![Pasted image 20260816110729.png](./images/Pasted%20image%2020260816110729.png)

Một domain nổi bật là:
```
cp8nl.hyperhost.ua
```

Domain này có detection khác với các domain hợp lệ như:
```
microsoft.com
sectigo.com
```

Điều đó khiến `cp8nl.hyperhost.ua` trở thành IOC cần được ưu tiên điều tra.

Kết hợp với hành vi của mẫu malware, đây là FQDN được sử dụng cho hoạt động Command-and-Control.

>cp8nl.hyperhost.ua

### Q11: Open-source intel associates that hash with a well-known commodity RAT. To which malware family does the sample belong?

**Yêu cầu:** Xác định malware family của `updatenow.exe`.

Cuối cùng, sử dụng hash của mẫu malware để thực hiện Threat Intelligence lookup.

Trong phần community analysis có kết quả:
```
Verdict: Malware
Score: 100/100
Threat: Family.AGENTTESLA
```

![Pasted image 20260816110733.png](./images/Pasted%20image%2020260816110733.png)

Tên family được xác định trực tiếp là:
```
AGENTTESLA
```

Agent Tesla thường được phân loại là một commodity information-stealing malware/RAT với khả năng thu thập thông tin từ hệ thống nạn nhân và truyền dữ liệu về infrastructure của attacker.

>AgentTesla

---

# ATTACK CHAIN

![Lockdown Lab.png](./images/Lockdown%20Lab.png)