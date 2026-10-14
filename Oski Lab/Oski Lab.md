author: Azaki
published: published
created: 2026-01-07
tags: CyberDefener, Lab, Threat Intel

The accountant at the company received an email titled "Urgent New Order" from a client late in the afternoon. When he attempted to access the attached invoice, he discovered it contained false order information. Subsequently, the SIEM solution generated an alert regarding downloading a potentially malicious file. Upon initial investigation, it was found that the PPT file might be responsible for this download. Could you please conduct a detailed examination of this file?

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefender</b> 
</p>

## Q1: Determining the creation time of the malware can provide insights into its origin. What was the time of malware creation?

**Yêu cầu:** Xác định thời điểm malware được tạo.

Đầu tiên, lấy hash của mẫu malware được cung cấp và tiến hành tra cứu trên **VirusTotal**.

Trong tab **Details**, kéo xuống phần **History** để kiểm tra các timestamp liên quan đến mẫu.
![Pasted image 20260816112050.png](./images/Pasted%20image%2020260816112050.png)

Tại trường **Creation Time**, VirusTotal hiển thị:
```
2022-09-28 17:40:46 UTC
```

Cần phân biệt `Creation Time` với `First Seen In The Wild`. Trong trường hợp này, mẫu có thời gian tạo từ năm 2022 nhưng chỉ được VirusTotal ghi nhận xuất hiện ngoài thực tế vào năm 2023.

>2022-09-28 17:40

## Q2: Identifying the command and control (C2) server that the malware communicates with can help trace back to the attacker. Which C2 server does the malware in the PPT file communicate with?

**Yêu cầu:** Xác định Command-and-Control server mà malware liên lạc sau khi thực thi.

Tiếp tục phân tích mẫu trên VirusTotal và chuyển sang tab **Relations**.

Trong phần **Contacted URLs**, có thể thấy malware kết nối tới hai URL đáng chú ý.
![Pasted image 20260816112054.png](./images/Pasted%20image%2020260816112054.png)

URL đầu tiên là:
```
http://171.22.28.221/5c06c05b7b34e8e6.php
```

Đây cũng là endpoint được sandbox ANY.RUN extract trực tiếp trong phần **Malware Configuration** của mẫu Stealc.

Endpoint này đóng vai trò C2 để malware trao đổi dữ liệu với infrastructure của attacker.

>http://171.22.28.221/5c06c05b7b34e8e6.php

## Q3: Identifying the initial actions of the malware post-infection can provide insights into its primary objectives. What is the first library that the malware requests post-infection?

**Yêu cầu:** Xác định library đầu tiên malware request sau khi infect hệ thống.

Tiếp tục kiểm tra các URL mà mẫu liên lạc.

Ngoài C2 endpoint ở câu trước, VirusTotal còn ghi nhận request tới:
```
http://171.22.28.221/9e226a84ec50246d/sqlite3.dll
```

![Pasted image 20260816112058.png](./images/Pasted%20image%2020260816112058.png)

Từ URL có thể xác định library được malware tải xuống là:
```
sqlite3.dll
```

ANY.RUN cũng ghi nhận `sqlite3.dll` trong configuration và các artifact của mẫu.

Việc một information stealer sử dụng SQLite là đáng chú ý vì nhiều trình duyệt lưu dữ liệu như history, cookies hoặc login information trong các SQLite database. Đây cũng phù hợp với những hành vi credential collection được quan sát ở các bước tiếp theo.

>sqlite3.dll

## Q4: By examining the provided Any.run report, what RC4 key is used by the malware to decrypt its base64-encoded string?

**Yêu cầu:** Xác định RC4 key được malware sử dụng để giải mã các chuỗi đã được Base64 encode.

Đối với câu này, mở **ANY.RUN report** của mẫu và tìm phần:
```
Malware configuration
```

Sandbox nhận diện configuration thuộc malware family **Stealc** và hiển thị trực tiếp phần encryption key.
![Pasted image 20260816112102.png](./images/Pasted%20image%2020260816112102.png)

Trong mục **Keys → RC4**, giá trị được xác định là:
```
5329514621441247975720749009
```

ANY.RUN cũng extract cùng RC4 key từ configuration của `VPN.exe`.

RC4 là symmetric stream cipher, do đó cùng một key có thể được sử dụng trong quá trình mã hóa và giải mã các chuỗi configuration của malware.

>5329514621441247975720749009

## Q5: By examining the MITRE ATT&CK techniques displayed in the Any.run sandbox report, identify the main MITRE technique (not sub-techniques) the malware uses to steal the user’s password.

**Yêu cầu:** Xác định **main MITRE ATT&CK Technique**, không phải sub-technique, mô tả hành vi malware đánh cắp password.

Trong ANY.RUN, chuyển sang phần **ATT&CK** và quan sát tactic:
```
Credential Access
```

![Pasted image 20260816112106.png](./images/Pasted%20image%2020260816112106.png)

Một trong những hành vi nổi bật là:
```
Credentials from Password Stores
```

Malware còn có hành vi cụ thể hơn là thu thập credentials được lưu trong các web browser.
![image](https://hackmd.io/_uploads/Syq6-toNZe.png)

MITRE ATT&CK định danh technique chính này là:
```
T1555 – Credentials from Password Stores
```

Trong đó hành vi lấy credential từ browser thuộc sub-technique:
```
T1555.003 – Credentials from Web Browsers
```

MITRE mô tả T1555 là kỹ thuật attacker tìm kiếm các vị trí lưu password để thu thập credential, còn T1555.003 cụ thể hóa việc đọc credential từ browser storage

>T1555

## Q6: By examining the child processes displayed in the Any.run sandbox report, which directory does the malware target for the deletion of all DLL files?

**Yêu cầu:** Xác định thư mục mà malware nhắm tới để xóa toàn bộ file `.dll`.

Tiếp tục kiểm tra **Malware Configuration** và các command được ANY.RUN extract từ mẫu.

Trong configuration xuất hiện hành vi xóa các DLL với wildcard:
```
C:\ProgramData\*.dll
```

Điều này cho thấy malware không chỉ xóa một DLL cụ thể mà nhắm tới **tất cả file có phần mở rộng** `**.dll**` **trong thư mục** `**C:\ProgramData**`.

ANY.RUN extract trực tiếp chuỗi command có thao tác `del` đối với `C:\ProgramData\*.dll`.
![Pasted image 20260816114430.png](./images/Pasted%20image%2020260816114430.png)

Đây có thể được xem là một bước cleanup nhằm loại bỏ những dependency hoặc artifact được malware sử dụng trong quá trình thực thi

>C:\ProgramData

## Q7: Understanding the malware's behavior post-data exfiltration can give insights into its evasion techniques. By analyzing the child processes, after successfully exfiltrating the user's data, how many seconds does it take for the malware to self-delete?

**Yêu cầu:** Xác định thời gian delay trước khi malware tự xóa chính nó sau khi hoàn thành quá trình exfiltration.

Tiếp tục phân tích command line được malware sử dụng ở giai đoạn cleanup.
![Pasted image 20260816114520.png](./images/Pasted%20image%2020260816114520.png)

ANY.RUN extract được command chứa:
```
timeout /t 5
```

sau đó mới thực hiện thao tác:
```
del /f /q
```

Điều này cho thấy malware tạo một khoảng delay trước khi thực hiện self-deletion.

ANY.RUN xác nhận chuỗi self-delete của mẫu chứa `timeout /t 5` trước lệnh xóa file
![Pasted image 20260816112116.png](./images/Pasted%20image%2020260816112116.png)

Tham số:
```
/t 5
```
có nghĩa tiến trình chờ **5 giây** trước khi tiếp tục thực thi lệnh tiếp theo.

Hành vi tự xóa sau khi hoàn thành nhiệm vụ giúp giảm số lượng artifact còn lại trên filesystem và gây khó khăn hơn cho quá trình forensic investigation

>5

---

# ATTACK CHAIN

![Oski Lab.png](./images/Oski%20Lab.png)