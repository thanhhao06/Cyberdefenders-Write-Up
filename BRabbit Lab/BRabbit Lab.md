author: Azaki
published: published
created: 2025-12-31
tags: CyberDefener, Lab, Threat Intel

You are an investigator assigned to assist Drumbo, a company that recently fell victim to a ransomware attack. The attack began when an employee received an email that appeared to be from the boss. It featured the company’s logo and a familiar email address. Believing the email was legitimate, the employee opened the attachment, which compromised the system and deployed ransomware, encrypting sensitive files. Your task is to investigate and analyze the artifacts to uncover information about the attacker.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefenders</b> 
</p>

## Q1: The phishing email used to deliver the malicious attachment showed several indicators of a potential social engineering attempt. Recognizing these indicators can help identify similar threats in the future. What is the suspicious email address that sent the attachment?

**Yêu cầu:** Xác định địa chỉ email đáng ngờ đã gửi file đính kèm độc hại.

Đầu tiên, phân tích file email `.eml`. Mình sử dụng lệnh `strings`đểtrực tiếp đọc phần header của email, sau đó tìm trường `From:` để xác định địa chỉ người gửi.

Quan sát địa chỉ email cho thấy attacker sử dụng một domain có hình thức rất giống với domain thật của công ty. Đây là kỹ thuật **typosquatting**, nhằm khiến nạn nhân khó nhận ra sự khác biệt.

![BRabbit Lab-1.png](./images/BRabbit%20Lab-1.png)

> theceojamessmith@Drurnbo.com

## Q2: The ransomware was identified as part of a known malware family. Determining its family name can provide critical insights into its behavior and remediation strategies. What is the family name of the ransomware identified during the investigation?

**Yêu cầu:** Xác định họ ransomware được sử dụng trong cuộc tấn công.

Dựa trên kết quả phân tích mẫu malware và các dấu hiệu nhận diện, ransomware thuộc họ **Bad Rabbit**.
![BRabbit Lab-2.png](./images/BRabbit%20Lab-2.png)


>BadRabbit

## Q3: Upon execution, the ransomware dropped a file onto the compromised system to initiate its payload. Identifying this file is essential for understanding its infection process. What is the name of the first file dropped by the ransomware?

**Yêu cầu:** Xác định file đầu tiên được ransomware thả xuống hệ thống sau khi thực thi.

Trước tiên, phân tích email phishing để tìm file đính kèm độc hại. Trong email, file được xác định là: **Urgent Contract Action.pdf.exe**
![BRabbit Lab-3.png](./images/BRabbit%20Lab-3.png)

File thực chất là một executable nhưng được đặt tên với phần mở rộng `.pdf.exe` nhằm đánh lừa người dùng nghĩ rằng đây là tài liệu PDF.

Tiếp theo, decode phần dữ liệu Base64 trong email để khôi phục file
![BRabbit Lab-4.png](./images/BRabbit%20Lab-4.png)

Sau khi có mẫu malware, tiến hành phân tích bằng **VirusTotal**. Trong phần thông tin về các file được malware tạo ra hoặc drop xuống hệ thống, có thể xác định file đầu tiên là `infpub.dat`
![BRabbit Lab-5.png](./images/BRabbit%20Lab-5.png)

>infpub.dat

## Q4: Inside the dropped file, the malware contained hardcoded artifacts, including usernames and passwords that could provide clues about its origins or configuration. What is the only person's username found within the dropped file?

**Yêu cầu:** Tìm username của một người được hardcode bên trong file `infpub.dat`.

Trước tiên, phân tích mẫu `app.bin` để xác định tiến trình và các module được malware sử dụng. Trong quá trình thực thi có thể thấy `rundll32.exe` được sử dụng để load payload.
![BRabbit Lab-6.png](./images/BRabbit%20Lab-6.png)

Tiếp tục kiểm tra phần **Modules** để tìm file `infpub.dat` đã được ransomware drop xuống trước đó.
![BRabbit Lab-7.png](./images/BRabbit%20Lab-7.png)

Sau đó tải `infpub.dat` về để thực hiện phân tích tĩnh
![image](https://hackmd.io/_uploads/rkFf0MCRge.png)

Vì một số chuỗi trong file được lưu dưới dạng Unicode little-endian, sử dụng `strings` với tùy chọn `-el`
```
strings -el infpub.dat
```

Từ kết quả thu được, có thể tìm thấy username gồm bốn ký tự
![BRabbit Lab-8.png](./images/BRabbit%20Lab-8.png)

>alex

## Q5: After execution, the ransomware communicated with a C2 server. Recognizing its communication techniques can assist in mitigation. What MITRE ATT&CK sub-technique describes the ransomware’s use of web protocols for sending and receiving data?

**Yêu cầu:** Xác định MITRE ATT&CK Sub-Technique mô tả việc malware sử dụng giao thức web để gửi và nhận dữ liệu với C2 server.

Upload và tra cứu mẫu `infpub.dat` lên Any.run để phân tích malware, sau đó kiểm tra phần mapping **MITRE ATT&CK**
![BRabbit Lab-9.png](./images/BRabbit%20Lab-9.png)

Theo gợi ý của đề bài, cần tìm kỹ thuật liên quan đến việc sử dụng **web protocols** cho hoạt động Command and Control.
![BRabbit Lab-10.png](./images/BRabbit%20Lab-10.png)

Kết quả tương ứng với: **Application Layer Protocol: Web Protocols**
![BRabbit Lab-11.png](./images/BRabbit%20Lab-11.png)

> T1071.001

## Q6: Persistence mechanisms are a hallmark of sophisticated ransomware. Identifying how persistence was achieved can aid in recovery and prevention of reinfection. What is the MITRE ATT&CK Sub-Technique ID associated with the ransomware’s persistence technique?

**Yêu cầu:** Xác định MITRE ATT&CK Sub-Technique được malware sử dụng để duy trì persistence trên hệ thống.

Trong trường hợp này, thông tin persistence không xuất hiện rõ trong phân tích `infpub.dat`, vì vậy cần quay lại phân tích mẫu `app.bin`

Trong phần **Persistence**, malware tạo scheduled task nhằm đảm bảo payload tiếp tục được thực thi trên hệ thống.
![BRabbit Lab-12.png](./images/BRabbit%20Lab-12.png)

Sub-Technique tương ứng là: **Scheduled Task/Job: Scheduled Task**
![BRabbit Lab-13.png](./images/BRabbit%20Lab-13.png)

>T1053.005

## Q7: As part of its infection chain, the ransomware created specific tasks to ensure its continued operation. Recognizing these tasks is crucial for system restoration. What are the names of the tasks created by the ransomware during execution?

**Yêu cầu:** Xác định tên hai scheduled task được ransomware tạo trong quá trình thực thi.

Do scheduled task liên quan trực tiếp đến việc thực thi payload, kiểm tra phần **Execution** trong kết quả phân tích malware.
![BRabbit Lab-14.png](./images/BRabbit%20Lab-14.png)

Task đầu tiên được xác định là
![BRabbit Lab-15.png](./images/BRabbit%20Lab-15.png)

Task thứ hai là
![BRabbit Lab-16.png](./images/BRabbit%20Lab-16.png)
Đây đều là tên các con rồng trong _Game of Thrones_

>rhaegal, drogon

## Q8: the malicious binary dispci.exe displayed a suspicious message upon execution, urging users to disable their defenses. This tactic aimed to evade detection and enable the ransomware's full execution. What suspicious message was displayed in the Console upon executing this binary?

**Yêu cầu:** Xác định thông báo đáng ngờ được hiển thị khi thực thi `dispci.exe`

Tiến hành chạy `dispci.exe` trong môi trường sandbox để quan sát hành vi của chương trình.

Khi thực thi, chương trình hiển thị thông báo yêu cầu người dùng tắt các giải pháp bảo mật trên hệ thống
![BRabbit Lab-17.png](./images/BRabbit%20Lab-17.png)
Thông báo này là một dấu hiệu rõ ràng cho thấy chương trình đang cố gắng khiến người dùng vô hiệu hóa các cơ chế phòng vệ, từ đó tạo điều kiện cho payload độc hại thực thi mà không bị phát hiện hoặc ngăn chặn

>Disable your anti-virus and anti-malware programs

## Q9: To modify the Master Boot Record (MBR) and encrypt the victim’s hard drive, the ransomware utilized a specific driver. Recognizing this driver is essential for understanding the encryption mechanism. What is the name of the driver used to encrypt the hard drive and modify the MBR?

**Yêu cầu:** Xác định driver được ransomware sử dụng để mã hóa ổ cứng và sửa đổi **Master Boot Record (MBR)**.

Từ quá trình phân tích trước đó, ta biết ransomware tạo ra hai thành phần đáng chú ý:
```
C:\Windows\dispci.exe
C:\Windows\cscc.dat
```

Tiếp tục tra cứu các báo cáo Threat Intelligence về [Bad Rabbit](https://www.varonis.com/blog/bad-rabbit-ransomware) cho thấy malware không tự xây dựng toàn bộ cơ chế mã hóa ổ đĩa mà tận dụng **DiskCryptor**, một phần mềm mã hóa ổ đĩa mã nguồn mở, một số bài báo sẽ không nói về phần mềm này

Trong đó, `cscc.dat` thực chất là driver hợp lệ `dcrypt.sys` của DiskCryptor được đổi tên. Thành phần `dispci.exe` sẽ giao tiếp với driver này thông qua device `\\.\dcrypt`, gửi các IOCTL command để bắt đầu quá trình mã hóa ổ đĩa. Đồng thời, `dispci.exe` chứa chức năng thay thế MBR bằng bootloader của ransomware
![BRabbit Lab-18.png](./images/BRabbit%20Lab-18.png)

>DiskCryptor
## Q10: Attribution is key to understanding the threat landscape. The ransomware was tied to a known attack group through its tactics, techniques, and procedures (TTPs). What is the name of the threat actor responsible for this ransomware campaign?

**Yêu cầu:** Xác định threat actor đứng sau chiến dịch ransomware dựa trên các Tactics, Techniques and Procedures (TTPs).

Để thực hiện attribution, mình tra cứu malware family **Bad Rabbit** trên các nguồn Threat Intelligence và MITRE ATT&CK.

MITRE ATT&CK định danh Bad Rabbit là:
```
S0606 – Bad Rabbit
```

![BRabbit Lab-19.png](./images/BRabbit%20Lab-19.png)

Trong phần **Groups That Use This Software**, Bad Rabbit được liên kết với:
```
G0034 – Sandworm Team
```

![BRabbit Lab-20.png](./images/BRabbit%20Lab-20.png)
MITRE cũng liệt kê **TeleBots** là một trong các tên liên quan đến Sandworm Team

>Sandworm
## Q11: The ransomware rendered the system unbootable by corrupting critical system components. Identifying the technique used provides insight into its destructive capabilities. What is the MITRE ATT&CK ID for the technique used to corrupt the system firmware and prevent booting?

**Yêu cầu:** Xác định MITRE ATT&CK Technique mô tả hành vi làm hỏng firmware khiến hệ thống không thể boot.

Dựa vào các từ khóa quan trọng trong câu hỏi như **firmware**, **corrupt** và **prevent booting**, tiến hành tra cứu trên MITRE ATT&CK
MITRE ATT&CK cũng liệt kê **Bad Rabbit (S0606)** là một procedure example của T1495, do malware cài đặt bootloader đã bị chỉnh sửa khiến hệ thống không thể khởi động bình thường
![BRabbit Lab-21.png](./images/BRabbit%20Lab-21.png)
Vì vậy technique phù hợp là **Firmware Corruption**

>T1495

---

# ATTACK CHAIN

![BRabbit Lab.png](./images/BRabbit%20Lab.png)