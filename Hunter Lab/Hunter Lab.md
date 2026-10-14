author: Azaki
published: published
created: 2026-10-14
tags: CyberDefenders, Lab, Threat Intel

The SOC team got an alert regarding some illegal port scanning activity coming from an employee's system. The employee was not authorized to do any port scanning or any offensive hacking activity within the network. The employee claimed that he had no idea about that, and it is probably a malware acting on his behalf. The IR team managed to respond immediately and take a full forensic image of the user's system to perform some investigations.

There is a theory that the user intentionally installed illegal applications to do port scanning and maybe other things. He was probably planning for something bigger, far beyond a port scanning!

It all began when the user asked for a salary raise that was rejected. After that, his behavior was abnormal and different. The suspect is believed to have weak technical skills, and there might be an outsider helping him!

Your objective as a soc analyst is to analyze the image and to either confirm or deny this theory.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Cyber5W</b> 
</p>

## Q1: What is the computer name of the suspect machine?

**Yêu cầu:** Xác định tên máy tính của đối tượng điều tra

Để xác định tên máy tính Windows tôi tiến hành trích xuất registry hive SYSTEM tại đường dẫn Windows\System32\config và kiểm tra key ControlSet001\Control\ComputerName\ComputerName
![Hunter Lab.png](./images/Hunter%20Lab.png)

>4ORENSICS

## Q2: What is the computer IP?

**Yêu cầu:** Xác định địa chỉ IP của máy tính nghi vấn

Địa chỉ IP cấu hình qua mạng thường được ghi nhận trong hive SYSTEM tại cấu hình card mạng ControlSet001\Services\Tcpip\Parameters\Interfaces
![Hunter Lab-1.png](./images/Hunter%20Lab-1.png)

>10.0.2.15

## Q3: What was the DHCP LeaseObtainedTime?

**Yêu cầu:** Xác định thời gian máy tính nhận cấp phát IP từ DHCP server

Trong cùng subkey card mạng của hive SYSTEM giá trị LeaseObtainedTime lưu timestamp dạng epoch Unix đại diện cho thời điểm cấp phát DHCP thành công
![Hunter Lab-3.png](./images/Hunter%20Lab-3.png)

>21/06/2016 02:24:12 UTC

## Q4: What is the computer SID?

**Yêu cầu:** Xác định định danh bảo mật SID của máy tính

Định danh bảo mật của hệ thống Windows được lưu trữ trong hive SAM tại đường dẫn SAM\Domains\Account với giá trị binary V
![Hunter Lab-4.png](./images/Hunter%20Lab-4.png)

>S-1-5-21-2489440558-2754304563-710705792

## Q5: What is the Operating System(OS) version?

**Yêu cầu:** Xác định phiên bản hệ điều hành đang cài đặt trên máy

Để kiểm tra phiên bản hệ điều hành tôi truy xuất hive SOFTWARE tại khóa Microsoft\Windows NT\CurrentVersion để đọc trường ProductName
![Hunter Lab-5.png](./images/Hunter%20Lab-5.png)

>8.1

## Q6: What was the computer timezone?

**Yêu cầu:** Xác định múi giờ được thiết lập trên máy tính

Thông tin múi giờ được quản lý tại khóa ControlSet001\Control\TimeZoneInformation trong hive SYSTEM
Tại đây giá trị TimeZoneKeyName là Pacific Standard Time với Bias là 480 phút tương đương UTC-08:00
Tuy nhiên hệ thống đang bật chế độ Daylight Saving Time với DaylightBias là -60 phút nên ActiveTimeBias được tính là 420 phút đưa múi giờ hoạt động thực tế về UTC-07:00 tức PDT
![Hunter Lab-15.png](./images/Hunter%20Lab-15.png)

>UTC-07:00

## Q7: How many times did this user log on to the computer?

**Yêu cầu:** Xác định tổng số lần đăng nhập vào hệ thống của tài khoản Hunter

Để đếm số lần đăng nhập của user tôi phân tích hive SAM tại đường dẫn SAM\Domains\Account\Users và kiểm tra thuộc tính của RID tương ứng với tài khoản Hunter
![Hunter Lab-6.png](./images/Hunter%20Lab-6.png)

>3

## Q8: When was the last login time for the discovered account? Format: one-space between date and time

**Yêu cầu:** Xác định thời gian đăng nhập lần cuối của tài khoản người dùng

Để xác định thời gian đăng nhập chính xác của tài khoản tôi trích xuất file nhật ký bảo mật Security.evtx từ thư mục Windows\System32\winevt\Logs
![Hunter Lab-18.png](./images/Hunter%20Lab-18.png)

Phân tích các bản ghi sự kiện Event ID 4624 thành công tôi lọc ra phiên đăng nhập tương tác cuối cùng của tài khoản Hunter
Sự kiện diễn ra vào 8:42:40 AM theo giờ PDT tức 01:42 theo chuẩn UTC
![Hunter Lab-7.png](./images/Hunter%20Lab-7.png)

>2016-06-21 01:42

## Q9: There was a “Network Scanner” running on this computer, what was it? And when was the last time the suspect used it? Format: program.exe,YYYY-MM-DD HH:MM:SS UTC

**Yêu cầu:** Xác định công cụ quét mạng đã thực thi trên hệ thống và thời điểm cuối cùng công cụ này được sử dụng

Windows Prefetch lưu lại bằng chứng thực thi của các chương trình nhằm tối ưu tốc độ khởi động tại thư mục Windows\Prefetch
Tôi kiểm tra danh sách file prefetch và phát hiện sự hiện diện của ZENMAP.EXE-56B17C4C.pf
![Hunter Lab-17.png](./images/Hunter%20Lab-17.png)

Phân tích file prefetch này bằng công cụ chuyên dụng thu được thời điểm thực thi cuối cùng được ghi nhận theo chuẩn UTC
![Hunter Lab-16.png](./images/Hunter%20Lab-16.png)

>ZENMAP.EXE,2016-06-21 12:08:13 UTC

## Q10: When did the port scan end? (Example: Sat Jan 23 hh:mm:ss 2016)

**Yêu cầu:** Xác định mốc thời gian hoàn tất đợt quét cổng mạng

Zenmap lưu trữ lịch sử quét và kết quả nmap vào cơ sở dữ liệu SQLite zenmap.db tại thư mục người dùng Users\Hunter\.zenmap
![Hunter Lab-19.png](./images/Hunter%20Lab-19.png)

Mở bảng lưu trữ kết quả quét tôi trích xuất log đầu ra dạng XML của Nmap và quan sát thẻ finished với thuộc tính timestr
Thẻ này ghi nhận chính xác thời điểm tiến trình rà quét kết thúc
![Hunter Lab-20.png](./images/Hunter%20Lab-20.png)

>Tue Jun 21 05:12:09 2016

## Q11: How many ports were scanned?

**Yêu cầu:** Xác định tổng số cổng mạng đã được quét

Kiểm tra chi tiết cấu hình quét trong kết quả XML của Nmap ta thấy tham số mặc định được áp dụng cho toàn bộ 1000 cổng phổ biến
![Hunter Lab-8.png](./images/Hunter%20Lab-8.png)

>1000

## Q12: What ports were found "open"?(comma-separated, ascending)

**Yêu cầu:** Liệt kê các cổng mạng được phát hiện đang mở theo thứ tự tăng dần

Trong bảng kết quả quét chi tiết của Nmap các cổng có trạng thái open được ghi nhận gồm cổng dịch vụ SSH web và hai cổng dịch vụ đặc biệt
![Hunter Lab-11.png](./images/Hunter%20Lab-11.png)

>22,80,9929,31337

## Q13: What was the version of the network scanner running on this computer?

**Yêu cầu:** Xác định phiên bản của trình quét mạng Nmap được cài đặt

Dòng tiêu đề của bản ghi quét mạng trong file log thể hiện rõ phiên bản Nmap đang được giao diện Zenmap gọi thực thi
![Hunter Lab-12.png](./images/Hunter%20Lab-12.png)

>7.12

## Q14: The employee engaged in a Skype conversation with someone. What is the skype username of the other party?

**Yêu cầu:** Xác định username Skype của đối tượng bên ngoài đã trao đổi với nhân viên

Ứng dụng Skype lưu trữ toàn bộ hồ sơ tin nhắn và danh bạ trong cơ sở dữ liệu SQLite main.db tại thư mục Users\Hunter\AppData\Roaming\Skype
![Hunter Lab-23.png](./images/Hunter%20Lab-23.png)

Truy vấn bảng Messages trong main.db phát hiện các đoạn hội thoại trao đổi với một tài khoản khả nghi
![Hunter Lab-21.png](./images/Hunter%20Lab-21.png)

>linux-rul3z

## Q15: What is the name of the application both parties agreed to use to exfiltrate data and provide remote access for the external attacker in their Skype conversation?

**Yêu cầu:** Xác định phần mềm điều khiển từ xa được hai bên thống nhất sử dụng

Đọc tiếp nội dung hội thoại giữa Hunter và tài khoản linux-rul3z trong bảng Messages ta thấy đối tượng hướng dẫn cài đặt phần mềm truy cập từ xa để hỗ trợ thao tác
![Hunter Lab-22.png](./images/Hunter%20Lab-22.png)

>Teamviewer

## Q16: What is the Gmail email address of the suspect employee?

**Yêu cầu:** Xác định địa chỉ Gmail của nhân viên nghi vấn

Trong cùng cơ sở dữ liệu SQLite main.db của Skype tôi kiểm tra bảng Accounts để tra cứu thông tin hồ sơ tài khoản cục bộ của người dùng trên máy
![Hunter Lab-13.png](./images/Hunter%20Lab-13.png)

>ehptmsgs@gmail.com

## Q17: It looks like the suspect user deleted an important diagram after his conversation with the external attacker. What is the file name of the deleted diagram?

**Yêu cầu:** Xác định tên file sơ đồ quan trọng mà người dùng đã xóa

Trong thư mục người dùng xuất hiện file dữ liệu email Outlook backup.pst chứa hòm thư cá nhân
![Hunter Lab-25.png](./images/Hunter%20Lab-25.png)

Nạp file PST này vào công cụ xem chuyên dụng và kiểm tra mục thùng rác Gmail\Trash tôi tìm thấy email đã bị xóa chứa file ảnh đính kèm là sơ đồ hệ thống mạng
![Hunter Lab-26.png](./images/Hunter%20Lab-26.png)

>home-network-design-networking-for-a-single-family-home-case-house-arkko-1433-x-792.jpg

## Q18: The user Documents' directory contained a PDF file discussing data exfiltration techniques. What is the name of the file?

**Yêu cầu:** Xác định tên tài liệu PDF về kỹ thuật trích xuất dữ liệu trong thư mục Documents

Duyệt cây thư mục người dùng tại Users\Hunter\Documents tôi phát hiện một file PDF nghiên cứu về an toàn thông tin
![Hunter Lab-27.png](./images/Hunter%20Lab-27.png)

Mở xem nội dung trang bìa của file PDF để xác thực tài liệu luận văn về kỹ thuật dữ liệu rò rỉ
![Hunter Lab-28.png](./images/Hunter%20Lab-28.png)

>Ryan_VanAntwerp_thesis.pdf

## Q19: What was the name of the Disk Encryption application Installed on the victim system? (two words space separated)

**Yêu cầu:** Xác định phần mềm mã hóa ổ đĩa đã được cài đặt trên hệ thống

Kiểm tra nhật ký gỡ bỏ cài đặt UnInstall.log trong thư mục phần mềm bảo mật ta thấy shortcut của tiện ích mã hóa phân vùng được ghi nhận
![Hunter Lab-29.png](./images/Hunter%20Lab-29.png)

>Crypto Swap

## Q20: What are the serial numbers of the two identified USB storage?

**Yêu cầu:** Xác định số serial của hai thiết bị lưu trữ USB từng kết nối vào máy

Dấu vết kết nối thiết bị USB được Windows ghi lại trong hive SYSTEM tại khóa ControlSet001\Enum\USBSTOR
Duyệt qua các thiết bị lưu trữ ngoại vi ta trích xuất được số serial của hai ổ đĩa USB
![Hunter Lab-30.png](./images/Hunter%20Lab-30.png)

>07B20C03C80830A9,AAI6UXDKZDV8E9OU

## Q21: One of the installed applications is a file shredder. What is the name of the application? (two words space separated)

**Yêu cầu:** Xác định tên ứng dụng tiêu hủy dữ liệu đã được cài đặt

Trong thư mục Program Files (x86) xuất hiện thư mục cài đặt Jetico\BCWipe
![Hunter Lab-31.png](./images/Hunter%20Lab-31.png)

Kiểm tra file ReadMe.txt bên trong thư mục phần mềm xác nhận đây là tiện ích xóa vĩnh viễn dữ liệu chuyên dụng
![Hunter Lab-34.png](./images/Hunter%20Lab-34.png)

>Jetico BCWipe

## Q22: How many prefetch files were discovered on the system?

**Yêu cầu:** Xác định tổng số file prefetch hợp lệ được tìm thấy trên hệ thống

Khi kiểm tra thư mục Windows\Prefetch qua phân tích index $I30 nhận thấy có nhiều file kích thước 0 byte do dấu vết xóa hoặc tạo lỗi để lại
![Hunter Lab-36.png](./images/Hunter%20Lab-36.png)

Để đếm chính xác số lượng file prefetch thực tế tôi dùng lệnh shell lọc bỏ toàn bộ các file có dung lượng bằng 0 byte

```bash
find . -type f -name "*.pf" -size +0c | wc -l
```

![Hunter Lab-35.png](./images/Hunter%20Lab-35.png)

>174

## Q23: How many times was the file shredder application executed?

**Yêu cầu:** Xác định số lần phần mềm tiêu hủy file đã được thực thi

Định vị file prefetch BCWIPE.EXE-36F3F2DF.pf của ứng dụng tiêu hủy dữ liệu trong thư mục Windows\Prefetch
![Hunter Lab-32.png](./images/Hunter%20Lab-32.png)

Phân tích cấu trúc header của file prefetch bằng WinPrefetchView trường Run Counter cho biết số lần thực thi của tiến trình
![Hunter Lab-33.png](./images/Hunter%20Lab-33.png)

>5

## Q24: Using prefetch, determine when was the last time ZENMAP.EXE-56B17C4C.pf was executed?

**Yêu cầu:** Xác định thời điểm thực thi cuối cùng của Zenmap dựa trên phân tích prefetch

Dấu vết thực thi của công cụ quét mạng đã được ghi nhận khi phân tích file ZENMAP.EXE-56B17C4C.pf ở câu hỏi trước
Đối chiếu trường Last Run Time trên công cụ phân tích prefetch ta thu được mốc thời gian thực thi cuối cùng của tiến trình
![Hunter Lab-37.png](./images/Hunter%20Lab-37.png)

>06/21/2016 12:08:13 PM

## Q25: A JAR file for an offensive traffic manipulation tool was executed. What is the absolute path of the file?

**Yêu cầu:** Xác định đường dẫn tuyệt đối của file JAR công cụ can thiệp lưu lượng mạng

Kiểm tra thư mục tải về của người dùng tại Users\Hunter\Downloads tôi phát hiện file thực thi Java của công cụ Burp Suite
![Hunter Lab-38.png](./images/Hunter%20Lab-38.png)

>C:\Users\Hunter\Downloads\burpsuite_free_v1.7.03.jar

## Q26: The suspect employee tried to exfiltrate data by sending it as an email attachment. What is the name of the suspected attachment?

**Yêu cầu:** Xác định tên file đính kèm nghi vấn được dùng để trích xuất dữ liệu qua email

Mở rộng phân tích hòm thư backup.pst tại mục [Gmail]\Sent Mail ta phát hiện email gửi tới địa chỉ linux-rul3z@hotmail.com chứa file nén dữ liệu kèm thông báo mật khẩu giải nén sẽ được gửi qua Skype
![Hunter Lab-39.png](./images/Hunter%20Lab-39.png)

>Pictures.7z

## Q27: Shellbags shows that the employee created a folder to include all the data he will exfiltrate. What is the full path of that folder?

**Yêu cầu:** Xác định đường dẫn đầy đủ của thư mục chứa dữ liệu trích xuất được phát hiện qua Shellbags

Dấu vết tương tác thư mục trong Windows Explorer được ghi nhận trong hive Shellbags UsrClass.dat tại đường dẫn Users\Hunter\AppData\Local\Microsoft\Windows
![Hunter Lab-42.png](./images/Hunter%20Lab-42.png)

Nạp hive vào Shellbags Explorer để khôi phục cấu trúc các thư mục người dùng đã truy cập phát hiện folder Exfil được tạo ra trong thư mục Pictures
![Hunter Lab-41.png](./images/Hunter%20Lab-41.png)

Kiểm tra cấu trúc file thực tế trên ảnh đĩa tại \Users\Hunter\Pictures\Exfil xác nhận thư mục này chứa các sơ đồ và tài liệu trích xuất dữ liệu
![Hunter Lab-40.png](./images/Hunter%20Lab-40.png)

>\Users\Hunter\Pictures\Exfil

## Q28: The user deleted two JPG files from the system and moved them to $Recycle-Bin. What is the file name that has the resolution of 1920x1200?

**Yêu cầu:** Xác định tên gốc của file ảnh có độ phân giải 1920x1200 bị chuyển vào thùng rác

Khi file bị xóa vào thùng rác NTFS hệ thống đổi tên file thành định dạng $R<random>.<ext> và tạo file metadata $I<random>.<ext> tương ứng
Kiểm tra thư mục thùng rác của tài khoản Hunter tại $Recycle.Bin\S-1-5-21-2489440558-2754304563-710705792-1001 phát hiện file ảnh $RP3TBNW.jpg
![Hunter Lab-43.png](./images/Hunter%20Lab-43.png)

Dùng exiftool kiểm tra thông số kỹ thuật của file $RP3TBNW.jpg xác nhận kích thước hình ảnh đạt chuẩn 1920x1200

```bash
exiftool \$RP3TBNW.jpg
```

![Hunter Lab-45.png](./images/Hunter%20Lab-45.png)

Đối chiếu với danh sách file ảnh gốc trong thư mục Pictures hoặc metadata khôi phục ta xác định được tên file ban đầu
![Hunter Lab-44.png](./images/Hunter%20Lab-44.png)

>ws_Small_cute_kitty_1920x1200.jpg

## Q29: Provide the name of the directory where information about jump lists items (created automatically by the system) is stored?

**Yêu cầu:** Xác định tên thư mục lưu trữ các mục jump list được hệ thống tạo tự động

Cơ chế Windows Jump Lists giúp người dùng truy cập nhanh các tệp mở gần đây
Các mục do hệ điều hành tự động khởi tạo được lưu trữ dưới dạng các file *.automaticDestinations-ms bên trong thư mục AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations
![Hunter Lab-46.png](./images/Hunter%20Lab-46.png)

>AutomaticDestinations

## Q30: Using JUMP LIST analysis, provide the full path of the application with the AppID of "aa28770954eaeaaa" used to bypass network security monitoring controls.

**Yêu cầu:** Xác định đường dẫn đầy đủ của ứng dụng có AppID aa28770954eaeaaa qua phân tích Jump List

Truy cập thư mục AppData\Roaming\Microsoft\Windows\Recent để kiểm tra các kho lưu trữ Jump List
![Hunter Lab-48.png](./images/Hunter%20Lab-48.png)

Trong thư mục CustomDestinations tôi tìm thấy file aa28770954eaeaaa.customDestinations-ms tương ứng với AppID cần điều tra
![Hunter Lab-49.png](./images/Hunter%20Lab-49.png)

Sử dụng Jump List Explorer phân tích cấu trúc lnk stream bên trong file customDestinations trích xuất đường dẫn tuyệt đối của trình duyệt Tor Browser
![Hunter Lab-47.png](./images/Hunter%20Lab-47.png)

>C:\Users\Hunter\Desktop\Tor Browser\Browser\firefox.exe