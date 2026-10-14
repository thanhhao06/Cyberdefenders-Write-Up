author: Azaki
published: published
created: 2026-01-01
tags: CyberDefener, Lab, Threat Intel

A large multinational corporation heavily relies on the 3CX software for phone communication, making it a critical component of their business operations. After a recent update to the 3CX Desktop App, antivirus alerts flag sporadic instances of the software being wiped from some workstations while others remain unaffected. Dismissing this as a false positive, the IT team overlooks the alerts, only to notice degraded performance and strange network traffic to unknown servers. Employees report issues with the 3CX app, and the IT security team identifies unusual communication patterns linked to recent software updates.

As the threat intelligence analyst, it's your responsibility to examine this possible supply chain attack. Your objectives are to uncover how the attackers compromised the 3CX app, identify the potential threat actor involved, and assess the overall extent of the incident. 

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefender</b> 
</p>

## Q1: Understanding the scope of the attack and identifying which versions exhibit malicious behavior is crucial for making informed decisions if these compromised versions are present in the organization. How many versions of 3CX running on Windows have been flagged as malware?

**Yêu cầu:** Xác định có bao nhiêu phiên bản 3CX Desktop App trên Windows bị xác định là độc hại.

Đầu tiên, tra cứu các báo cáo Threat Intelligence liên quan đến sự cố [3CX Supply Chain Attack](https://www.3cx.com/blog/news/desktopapp-security-alert/) để xác định những phiên bản Windows đã bị trojan hóa.
![Pasted image 20260816100335.png](./images/Pasted%20image%2020260816100335.png)

Trong quá trình điều tra, hai phiên bản 3CX Desktop App dành cho Windows thường được xác định có liên quan đến chiến dịch là:
```
18.12.407
18.12.416
```

Đây là các phiên bản chứa chuỗi thực thi bị chỉnh sửa, cho phép attacker lợi dụng cơ chế cập nhật phần mềm hợp lệ của 3CX để phân phối các thành phần độc hại đến hệ thống của người dùng.

Như vậy, tổng cộng có **hai phiên bản Windows** bị xác định trong phạm vi của challenge.


>2

## Q2: Determining the age of the malware can help assess the extent of the compromise and track the evolution of malware families and variants. What's the UTC creation time of the .msi malware?

**Yêu cầu:** Xác định thời gian tạo của file `.msi` độc hại theo múi giờ UTC.

Để xác định thời gian tạo của mẫu malware, lấy hash của file `.msi` và tra cứu trên **VirusTotal**.

Trong tab **Details**, kéo xuống phần **History** để kiểm tra các mốc thời gian liên quan đến mẫu.
![Pasted image 20260816093424.png](./images/Pasted%20image%2020260816093424.png)
Tại trường **Creation Time**, VirusTotal ghi nhận thời điểm tạo của file là:
```
2023-03-13 06:33:26 UTC
```

Thông tin này giúp xác định thời điểm mẫu malware được build trước khi được đưa vào chuỗi cung ứng của 3CX

>2023-03-13 06:33

## Q3: Executable files (.exe) are frequently used as primary or secondary malware payloads, while dynamic link libraries (.dll) often load malicious code or enhance malware functionality. Analyzing files deposited by the Microsoft Software Installer (.msi) is crucial for identifying malicious files and investigating their full potential. Which malicious DLLs were dropped by the .msi file?

**Yêu cầu:** Xác định các DLL độc hại được file `.msi` thả xuống hệ thống.

Tiếp tục phân tích mẫu `.msi` trên VirusTotal và kiểm tra danh sách các file được tạo trong quá trình cài đặt.

Trong thư mục của 3CX Desktop App có thể thấy hai DLL đáng chú ý:
```
d3dcompiler_47.dll
ffmpeg.dll
```

![Pasted image 20260816093428.png](./images/Pasted%20image%2020260816093428.png)

Hai file này đóng vai trò quan trọng trong infection chain của chiến dịch.

`3CXDesktopApp.exe` là executable hợp lệ và được ký số, tuy nhiên attacker đã chỉnh sửa chuỗi dependency để chương trình load `ffmpeg.dll` độc hại.

Sau khi được load, `ffmpeg.dll` tiếp tục đọc dữ liệu được nhúng bên trong `d3dcompiler_47.dll`, giải mã payload và chuyển sang giai đoạn thực thi tiếp theo.

Đây cũng là lý do cuộc tấn công khó bị phát hiện ban đầu: mã độc được thực thi thông qua một ứng dụng 3CX hợp lệ thay vì một executable hoàn toàn xa lạ.

>ffmpeg.dll,d3dcompiler_47.dll

## Q4: Recognizing the persistence techniques used in this incident is essential for current mitigation strategies and future defense improvements. What is the MITRE Technique ID employed by the .msi files to load the malicious DLL?

**Yêu cầu:** Xác định MITRE ATT&CK Technique được sử dụng để load DLL độc hại.

Từ infection chain ở câu trước, có thể thấy `3CXDesktopApp.exe` là một executable hợp lệ nhưng lại load `ffmpeg.dll` độc hại trong quá trình khởi chạy.

Hành vi này là đặc trưng của kỹ thuật **DLL Side-Loading**.
![Pasted image 20260816101228.png](./images/Pasted%20image%2020260816101228.png)

Thay vì trực tiếp khởi chạy một executable độc hại, attacker đặt DLL đã bị chỉnh sửa vào vị trí mà một chương trình hợp lệ sẽ tìm kiếm và load. Khi người dùng chạy ứng dụng, mã độc được thực thi bên trong context của tiến trình đáng tin cậy.

>T1574

## Q5: Recognizing the malware type (threat category) is essential to your investigation, as it can offer valuable insight into the possible malicious actions you'll be examining. What is the threat category of the two malicious DLLs?

**Yêu cầu:** Xác định threat category của hai DLL độc hại.

Tiếp tục tra cứu các mẫu DLL trên **VirusTotal** và quan sát phần **Threat categories**
![Pasted image 20260816093437.png](./images/Pasted%20image%2020260816093437.png)

VirusTotal hiển thị một số nhãn phân loại như:
```
trojan
pua
dropper
```

Vì vậy threat category cần xác định là **Trojan**.

Điều này phù hợp với bản chất của cuộc tấn công: attacker đã trojan hóa các thành phần phần mềm hợp lệ của 3CX để bí mật thực thi mã độc trên hệ thống nạn nhân

>Trojan

## Q6: As a threat intelligence analyst conducting dynamic analysis, it's vital to understand how malware can evade detection in virtualized environments or analysis systems. This knowledge will help you effectively mitigate or address these evasive tactics. What is the MITRE ID for the virtualization/sandbox evasion techniques used by the two malicious DLLs?

**Yêu cầu:** Xác định MITRE ATT&CK Technique mô tả hành vi né tránh môi trường virtualization hoặc sandbox.

Trong quá trình dynamic analysis, tiếp tục kiểm tra phần **Behavior** của mẫu malware và quan sát MITRE ATT&CK mapping.

Kết quả cho thấy malware thực hiện kỹ thuật: **Virtualization/Sandbox Evasion**
![Pasted image 20260816093441.png](./images/Pasted%20image%2020260816093441.png)

Đây là một kỹ thuật anti-analysis thường được malware sử dụng để kiểm tra xem nó có đang chạy trong máy ảo hoặc sandbox của analyst hay không.

Nếu phát hiện môi trường phân tích, malware có thể thay đổi hành vi, trì hoãn thực thi hoặc không kích hoạt payload nhằm tránh bị phát hiện

>T1497

## Q7: When conducting malware analysis and reverse engineering, understanding anti-analysis techniques is vital to avoid wasting time. Which hypervisor is targeted by the anti-analysis techniques in the ffmpeg.dll file?

**Yêu cầu:** Xác định hypervisor mà `ffmpeg.dll` nhắm tới trong quá trình anti-analysis.

Lấy hash của `ffmpeg.dll` và tiến hành phân tích riêng mẫu này trên VirusTotal.

Trong phần **Capabilities → Anti-Analysis**, kết quả cho thấy DLL chứa các chuỗi được sử dụng để kiểm tra môi trường máy ảo
![Pasted image 20260816093445.png](./images/Pasted%20image%2020260816093445.png)

VirusTotal ghi nhận:
```
Reference anti-VM strings targeting VMWare
```

Điều này cho thấy `ffmpeg.dll` có khả năng kiểm tra sự hiện diện của các artifact liên quan đến **VMware**.

Mục đích của kỹ thuật này là xác định liệu malware có đang chạy trong môi trường phân tích hay không, từ đó giúp attacker hạn chế việc payload bị quan sát bởi sandbox hoặc malware analyst


>VMware

## Q8: Identifying the cryptographic method used in malware is crucial for understanding the techniques employed to bypass defense mechanisms and execute its functions fully. What encryption algorithm is used by the ffmpeg.dll file?

**Yêu cầu:** Xác định thuật toán mã hóa được `ffmpeg.dll` sử dụng.

Tiếp tục phân tích capability của `ffmpeg.dll`, đặc biệt là phần liên quan đến **Cryptography**.
![Pasted image 20260816093449.png](./images/Pasted%20image%2020260816093449.png)

Trong kết quả phân tích có thể thấy nhiều dấu hiệu liên quan trực tiếp đến RC4:
```
RC4 PRGA
RC4
RC4 KSA
```

Trong đó:

- **KSA – Key Scheduling Algorithm** được sử dụng để khởi tạo state của RC4 từ encryption key.
- **PRGA – Pseudo-Random Generation Algorithm** tạo keystream phục vụ quá trình mã hóa hoặc giải mã dữ liệu.

Trong infection chain của 3CX, `ffmpeg.dll` sử dụng RC4 để xử lý payload được lưu trong `d3dcompiler_47.dll` trước khi payload được thực thi trong memory

>RC4

## Q9: As an analyst, you've recognized some TTPs involved in the incident, but identifying the APT group responsible will help you search for their usual TTPs and uncover other potential malicious activities. Which group is responsible for this attack?

**Yêu cầu:** Xác định threat actor đứng sau chiến dịch 3CX Supply Chain Attack.

Attribution của một chiến dịch APT không nên chỉ dựa vào một IOC duy nhất mà cần kết hợp nhiều yếu tố như:

- Infrastructure
- Malware family
- Encryption scheme
- Code similarities
- Tactics, Techniques and Procedures
- Các chiến dịch đã được ghi nhận trước đó

Tra cứu thông tin về **3CX Backdoor** trên [Malpedia](https://malpedia.caad.fkie.fraunhofer.de/details/win.3cx_backdoor) cho thấy chiến dịch được liên kết với các threat actor có nguồn gốc từ Triều Tiên.

Trong nguồn tra cứu được sử dụng cho challenge, phần thông tin của **3CX Backdoor** xác định actor liên quan là:
![Pasted image 20260816095606.png](./images/Pasted%20image%2020260816095606.png)

**Lazarus Group** là tên gọi phổ biến được sử dụng để mô tả một hệ sinh thái các hoạt động cyber có liên hệ với Triều Tiên. Trong các báo cáo khác, những cluster cụ thể liên quan tới sự cố 3CX còn được theo dõi bằng các tên như **LABYRINTH CHOLLIMA** hoặc `UNC4736`

>Lazarus

---
# ATTACK CHAIN

![3CX Supply Chain Lab.png](./images/3CX%20Supply%20Chain%20Lab.png)