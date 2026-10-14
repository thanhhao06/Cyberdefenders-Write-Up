author: Azaki
published: published
created: 2025-10-18
tags: CyberDefener, Lab, Malware Analysis

An employee accidentally downloaded a suspicious file from a phishing email. The file executed silently, triggering unusual system behavior. As a malware analyst, your task is to analyze the sample to uncover its behavior, persistence mechanisms, communication with Command and Control (C2) servers, and potential data exfiltration or system compromise.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Cyber_Assassin</b> 
</p>

## Q1: What is the compile timestamp (UTC) of the sample?

**Yêu cầu:** Xác định thời điểm sample malware được compile theo múi giờ UTC

Đầu tiên, thực hiện static triage đối với sample bằng **ExifTool**. Trường cần quan tâm là `TimeDateStamp` trong PE header.

Với ExifTool, có thể kiểm tra trực tiếp metadata của executable
![XWorm Lab-1.png](./images/XWorm%20Lab-1.png)
Đổi sang UTC là `2024-02-25 22:53:40`

>2024-02-25 22:53:40
## Q2: Which legitimate company does the malware impersonate in an attempt to appear trustworthy?

**Yêu cầu:** Xác định công ty hợp pháp mà malware giả mạo nhằm khiến file trông đáng tin cậy hơn.

Tiếp tục kiểm tra phần **Version Information** của executable. Có thể sử dụng ExifTool, PEStudio, CFF Explorer hoặc VirusTotal để đọc các trường metadata như `CompanyName`, `FileDescription`, `ProductName` và `LegalCopyright`
![XWorm Lab-2.png](./images/XWorm%20Lab-2.png)
Trong sample xuất hiện các thông tin đáng chú ý:
```text
CompanyName     : Adobe Inc.
FileDescription : Adobe Installer
ProductName     : Adobe Installer
```
Malware đã chèn metadata giả để tạo cảm giác đây là một trình cài đặt hợp lệ của Adobe. Tuy nhiên, các trường version information chỉ là dữ liệu do tác giả executable kiểm soát nên không thể xem chúng như bằng chứng file thực sự do Adobe phát hành

Vậy nên tên công ty bị mạo danh là Adobe

>Adobe

## Q3: How many anti-analysis checks does the malware perform to detect/evade sandboxes and debugging environments?

**Yêu cầu:** Xác định số lượng kỹ thuật kiểm tra môi trường mà malware sử dụng để phát hiện sandbox, máy ảo hoặc debugger.

Vì sample là một executable **.NET/VB.NET**, có thể mở nó bằng **dnSpy** và lần theo entry point. Code đã bị obfuscate bằng các tên class và function ngẫu nhiên, vì vậy cần tập trung vào hành vi của từng function thay vì tên của chúng
![XWorm Lab-3.png](./images/XWorm%20Lab-3.png)
Trong nhánh anti-analysis có tổng cộng năm nhóm kiểm tra:
1. Kiểm tra môi trường máy ảo như **Hyper-V, VMware và VirtualBox**.
2. Kiểm tra xem process có đang chạy dưới debugger hay không.
3. Kiểm tra sự tồn tại của `SbieDll.dll`, DLL đặc trưng của **Sandboxie**.
4. Kiểm tra hệ điều hành có phải **Windows XP** hay không.
5. Truy vấn thông tin IP để xác định máy đang chạy có thuộc **hosting provider** hay không.

Nếu một trong các điều kiện phân tích đáng ngờ được phát hiện, malware có thể thay đổi hoặc dừng luồng thực thi nhằm tránh bị quan sát

>5

## Q4: What is the name of the scheduled task created by the malware to achieve execution with elevated privileges?

**Yêu cầu:** Xác định tên scheduled task được malware tạo để duy trì thực thi và có thể chạy với mức quyền cao.

Trong code persistence, malware xây dựng một lệnh `schtasks` để tạo task định kỳ. Khi có quyền Administrator, task được cấu hình với tùy chọn `/RL HIGHEST`, nghĩa là chạy với mức quyền cao nhất mà tài khoản hiện tại cho phép.
![XWorm Lab-4.png](./images/XWorm%20Lab-4.png)
Điểm cần phân biệt ở câu này là:

```text
Scheduled task name : WmiPrvSE
Executable          : %AppData%\WmiPrvSE.exe
```
`WmiPrvSE.exe` là file được task thực thi, còn **tên của scheduled task** mà đề bài yêu cầu là `WmiPrvSE`.

Tên này cố tình bắt chước `WmiPrvSE.exe` một process hợp lệ của Windows  nhằm khiến persistence artifact ít gây chú ý hơn.

>WmiPrvSE

## Q5: What is the filename of the malware binary that is dropped in the AppData directory?
**Yêu cầu:** Xác định tên file mà malware copy/drop vào thư mục `AppData`.

Tiếp tục theo dõi phần code chịu trách nhiệm cho quá trình cài đặt và persistence. Malware thêm đường dẫn trong AppData vào danh sách Microsoft Defender exclusions, sau đó copy chính nó tới
![XWorm Lab-5.png](./images/XWorm%20Lab-5.png)
Tên `WmiPrvSE.exe` được lựa chọn để giả mạo process **WMI Provider Host** hợp lệ của Windows. Tuy nhiên, binary hợp lệ của Windows không nằm trong thư mục AppData của user, vì vậy vị trí file là một IOC đáng chú ý

>WmiPrvSE.exe

## Q6: Which cryptographic algorithm does the malware use to encrypt or obfuscate its configuration data?

**Yêu cầu:** Xác định thuật toán mã hóa được malware sử dụng để bảo vệ hoặc obfuscate configuration data.

Trong dnSpy, lần theo function được gọi khi malware khởi tạo các biến cấu hình. Các chuỗi cấu hình được lưu ở dạng Base64 và được đưa qua một hàm decrypt trước khi sử dụng
![XWorm Lab-6.png](./images/XWorm%20Lab-6.png)
Trong hàm này có thể quan sát việc khởi tạo **Rijndael/AES** cùng key được tạo từ một giá trị hardcode. Sau khi Base64 decode, dữ liệu ciphertext được giải mã bằng AES để lấy các thông số như C2 hosts, port và các giá trị cấu hình khác.

Ở sample này, code decompile cho thấy AES được sử dụng ở **ECB mode** cho phần configuration decryption

>AES

## Q7: To derive the parameters for its encryption algorithm (such as the key and initialization vector), the malware uses a hardcoded string as input. What is the value of this hardcoded string?

**Yêu cầu:** Xác định chuỗi hardcode được malware sử dụng làm input cho quá trình tạo key phục vụ giải mã configuration.

Không nên xác định giá trị này chỉ dựa vào độ dài của chuỗi. Thay vào đó, lần theo function decrypt trong dnSpy và tìm biến được truyền vào quá trình tạo cryptographic key
![XWorm Lab-7.png](./images/XWorm%20Lab-7.png)
Code cho thấy chuỗi:
```text
8xTJ0EKPuiQsJVaT
```

được đưa qua **MD5**. Digest thu được sau đó được sử dụng để dựng AES key phục vụ việc giải mã các biến configuration.

Chuỗi này còn được malware sử dụng làm **mutex name**, qua đó bảo đảm chỉ có một instance của malware hoạt động tại một thời điểm

>8xTJ0EKPuiQsJVaT

## Q8: What are the Command and Control (C2) IP addresses obtained after the malware decrypts them?

**Yêu cầu:** Giải mã configuration và xác định các địa chỉ IP của Command and Control server.

Sau khi xác định được AES key ở câu trước, tiếp tục quan sát các giá trị trả về từ hàm decrypt. Một trong các biến configuration chứa danh sách C2 hosts, được lưu dưới dạng chuỗi phân tách bởi dấu phẩy
![XWorm Lab-8.png](./images/XWorm%20Lab-8.png)

Sau khi giải mã, malware thu được ba địa chỉ:
```text
185.117.250.169
66.175.239.149
185.117.249.43
```

Trong luồng network, malware lần lượt thử các host trong danh sách cho tới khi thiết lập được kết nối thành công

>185.117.250.169,66.175.239.149,185.117.249.43
 
## Q9: What port number does the malware use for communication with its Command and Control (C2) server?

**Yêu cầu:** Xác định port được XWorm sử dụng để kết nối đến C2 server.

Port cũng được lưu trong phần configuration đã mã hóa. Sau khi đi qua cùng hàm decrypt, giá trị port được khôi phục thành
![XWorm Lab-9.png](./images/XWorm%20Lab-9.png)
Trong network routine, malware kết hợp một trong ba C2 IP ở Q8 với port này để thiết lập kết nối TCP. Khi kết nối thành công, nó có thể gửi thông tin hệ thống bị nhiễm và chờ command từ operator

>7000

## Q10: The malware spreads by copying itself to every connected removable device. What is the name of the new copy created on each infected device?

**Yêu cầu:** Xác định tên executable mà malware tạo trên các removable drive như USB.

Trong phần code xử lý removable drives, malware duyệt qua các ổ đĩa đang kết nối, kiểm tra loại drive và tìm các thiết bị removable
![XWorm Lab-10.png](./images/XWorm%20Lab-10.png)
Khi phát hiện một thiết bị phù hợp, malware copy chính nó vào thiết bị với tên
```text
USB.exe
```

Sau đó nó có thể thay đổi thuộc tính của các file/thư mục gốc và kết hợp với shortcut `.lnk` nhằm khiến người dùng vô tình kích hoạt `USB.exe`

>USB.exe

## Q11: To ensure its execution, the malware creates specific types of files. What is the file extension of these created files?

**Yêu cầu:** Xác định phần mở rộng của các file được malware tạo trên removable drive để kích hoạt payload.

Tiếp tục phân tích routine lây lan qua USB. Sau khi copy malware thành `USB.exe`, XWorm tạo shortcut cho các file hoặc thư mục trên thiết bị
![XWorm Lab-11.png](./images/XWorm%20Lab-11.png)
Các shortcut này có phần mở rộng **.ink**

Ý tưởng của kỹ thuật này là ẩn nội dung thật trên USB rồi tạo các shortcut có tên/icon tương tự. Khi nạn nhân mở shortcut, nó có thể kích hoạt malware trước khi mở nội dung mà người dùng mong đợi
>lnk

## Q12: What is the name of the DLL the malware uses to detect if it is running in a sandbox environment?
**Yêu cầu:** Xác định DLL mà malware kiểm tra để phát hiện môi trường sandbox.

Trong nhóm anti-analysis ở Q3, một function sử dụng cơ chế kiểm tra module/DLL đã được load vào process.
![XWorm Lab-12.png](./images/XWorm%20Lab-12.png)
Tên DLL được tìm kiếm là:
```text
SbieDll.dll
```

`SbieDll.dll` là thành phần gắn liền với **Sandboxie**. Việc phát hiện DLL này trong process là một dấu hiệu mạnh cho malware rằng nó có thể đang bị chạy trong môi trường sandbox để phân tích

>SbieDll.dll

## Q13: What is the name of the registry key manipulated by the malware to control the visibility of hidden items in Windows Explorer?

**Yêu cầu:** Xác định registry value được malware thay đổi để kiểm soát khả năng hiển thị các mục ẩn/protected trong Windows Explorer.

Trong phần xử lý hệ thống file, malware mở registry path
```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced
```
![XWorm Lab-13.png](./images/XWorm%20Lab-13.png)

Tại đây, nó thao tác value:
```text
ShowSuperHidden
```

Malware đặt giá trị này về `0`, làm giảm khả năng hiển thị các protected operating system files trong Explorer. Hành vi này hỗ trợ việc che giấu các artifact mà XWorm tạo ra, đặc biệt trong quá trình lây nhiễm removable drive

>ShowSuperHidden

## Q14: Which API does the malware use to mark its process as critical in order to prevent termination or interference?

**Yêu cầu:** Xác định Windows API được malware gọi để đánh dấu process của nó là critical.

Tiếp tục xem các unmanaged API được import/call từ code .NET. Malware sử dụng một API thuộc `ntdll.dll` để thay đổi trạng thái critical của process
![XWorm Lab-14.png](./images/XWorm%20Lab-14.png)

API được gọi là:
```text
RtlSetProcessIsCritical
```

Khi một process được đánh dấu là critical, việc terminate process đó có thể khiến Windows bugcheck và dẫn tới **BSOD**. XWorm lợi dụng đặc điểm này nhằm gây khó khăn cho việc can thiệp hoặc kết thúc process của nó

>RtlSetProcessIsCritical

## Q15: Which API does the malware use to insert keyboard hooks into running processes in order to monitor or capture user input?

**Yêu cầu:** Xác định API được malware sử dụng để cài keyboard hook và thu thập thao tác bàn phím của người dùng.

Trong phần collection functionality, malware sử dụng Windows hook mechanism để theo dõi keyboard events
![XWorm Lab-15.png](./images/XWorm%20Lab-15.png)
API được sử dụng là:
```text
SetWindowsHookEx
```

Malware đăng ký một low-level keyboard hook, cho phép nó nhận các sự kiện phím bấm và ghi lại input cùng thông tin process/window liên quan

>SetWindowsHookEx

## Q16: Given the malware’s ability to insert keyboard hooks into running processes, what is its primary functionality or objective?

**Yêu cầu:** Từ hành vi cài keyboard hook ở câu trước, xác định chức năng chính mà malware đang thực hiện.

`SetWindowsHookEx` cho phép XWorm theo dõi các phím mà người dùng nhập. Trong code, dữ liệu thu thập được còn được ghi vào một file log, thường là:
```text
Log.tmp
```

![XWorm Lab-16.png](./images/XWorm%20Lab-16.png)
Chức năng này có thể thu thập các dữ liệu nhạy cảm được nhập qua bàn phím, chẳng hạn username, password, nội dung chat hoặc thông tin khác trước khi dữ liệu được gửi tới C2

>Keylogger

# ATTACK CHAIN

![XWorm Lab.png](./images/XWorm%20Lab.png)