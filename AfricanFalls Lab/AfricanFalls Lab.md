author: Azaki
published: published
created: 2026-10-05
tags: CyberDefenders, Lab, Endpoint Forensic

Instructions:
- Uncompress the lab (pass: **cyberdefenders.org**)

John Doe was accused of doing illegal activities. A disk image of his laptop was taken. Your task as a soc analyst is to analyze the image and understand what happened under the hood.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@DFIRScience</b> 
</p>

## Q1: What is the MD5 hash value of the suspect disk?

**Yêu cầu:** Xác định giá trị băm MD5 của tệp ảnh đĩa của đối tượng tình nghi

Khi tạo tệp ảnh đĩa DiskDrigger.ad1 công cụ FTK Imager tự động lưu lại nhật ký thu thập kèm giá trị băm kiểm tra Mở tệp văn bản này tôi lấy được giá trị băm MD5 của bằng chứng đĩa
![AfricanFalls Lab.png](./images/AfricanFalls%20Lab.png)

>9471e69c95d8909ae60ddff30d50ffa1

## Q2: What phrase did the suspect search for on 2021-04-29 18:17:38 UTC? (three words, two spaces in between)

**Yêu cầu:** Xác định cụm từ đối tượng đã tìm kiếm vào thời điểm 2021-04-29 18:17:38 UTC

Định vị tệp cơ sở dữ liệu SQLite lưu lịch sử duyệt web của Google Chrome tại đường dẫn User Data\Default\History trên cây thư mục FTK Imager
![AfricanFalls Lab-1.png](./images/AfricanFalls%20Lab-1.png)

Mở tệp bằng DB Browser for SQLite và kiểm tra bảng keyword_search_terms tôi ghi nhận cụm từ tìm kiếm tương ứng với mốc thời gian yêu cầu là password cracking lists
![AfricanFalls Lab-2.png](./images/AfricanFalls%20Lab-2.png)

>password cracking lists

## Q3: What is the IPv4 address of the FTP server the suspect connected to?

**Yêu cầu:** Xác định địa chỉ IPv4 của máy chủ FTP mà đối tượng đã kết nối tới

FileZilla Client lưu thông tin các máy chủ từng kết nối trong tệp recentservers.xml tại thư mục AppData\Roaming\FileZilla Mở tệp này qua trình xem văn bản của FTK Imager thẻ RecentServers ghi nhận địa chỉ IPv4 của máy chủ FTP là 192.168.1.20
![AfricanFalls Lab-3.png](./images/AfricanFalls%20Lab-3.png)

>192.168.1.20

## Q4: What date and time was a password list deleted in UTC? (YYYY-MM-DD HH:MM:SS UTC)

**Yêu cầu:** Xác định ngày giờ danh sách mật khẩu bị xóa theo chuẩn thời gian UTC

Trong thư mục $Recycle.Bin tệp $RW9BJ2Z.txt chứa nội dung danh sách mật khẩu và tệp $IW9BJ2Z.txt lưu siêu dữ liệu của tệp bị xóa Kiểm tra thuộc tính Date Changed trên bản ghi MFT của tệp siêu dữ liệu này mốc thời gian xóa theo chuẩn UTC là 2021-04-29 18:22:17
![AfricanFalls Lab-4.png](./images/AfricanFalls%20Lab-4.png)

>2021-04-29 18:22:17

## Q5: How many times was Tor Browser ran on the suspect's computer? (number only)

**Yêu cầu:** Xác định số lần trình duyệt Tor Browser đã được thực thi trên máy tính của nghi phạm

Kiểm tra thư mục Windows\Prefetch để tìm kiếm dấu vết thực thi của trình duyệt Tor
![AfricanFalls Lab-5.png](./images/AfricanFalls%20Lab-5.png)

Tại đây tôi thấy tệp TORBROWSER-INSTALL-WIN64-10.0-F3C4DF19.pf có số lần chạy Run Counter là 1 nhưng đây là bộ cài đặt chứ không phải trình duyệt web
![AfricanFalls Lab-6.png](./images/AfricanFalls%20Lab-6.png)

Kiểm tra ShimCache trong hive SYSTEM có ghi nhận đường dẫn firefox.exe của Tor Browser trên Desktop nhưng dấu vết này chỉ xác nhận tệp từng tồn tại trên đĩa chứ không chứng minh tiến trình đã chạy
![AfricanFalls Lab-7.png](./images/AfricanFalls%20Lab-7.png)

Rà soát lại toàn bộ thư mục Prefetch không hề có tệp prefetch nào của firefox.exe hay tor.exe chứng tỏ trình duyệt Tor chưa từng được khởi chạy lần nào
![AfricanFalls Lab-8.png](./images/AfricanFalls%20Lab-8.png)

>0

## Q6: What is the suspect's email address?

**Yêu cầu:** Xác định địa chỉ email của đối tượng tình nghi

Trích xuất tệp History của Chrome tại thư mục AppData\Local\Google\Chrome\User Data\Default để kiểm tra lịch sử truy cập
![AfricanFalls Lab-9.png](./images/AfricanFalls%20Lab-9.png)

Mở tệp bằng BrowsingHistoryView và lọc các liên kết ProtonMail tiêu đề trang hộp thư đến hiển thị địa chỉ email cá nhân của đối tượng là dreammaker82@protonmail.com
![AfricanFalls Lab-10.png](./images/AfricanFalls%20Lab-10.png)

>dreammaker82@protonmail.com

## Q7: What is the FQDN did the suspect port scan?

**Yêu cầu:** Xác định tên miền FQDN mà đối tượng đã thực hiện quét cổng

Mở tệp nhật ký ConsoleHost_history.txt của PSReadLine tại thư mục AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine qua FTK Imager tôi phát hiện đối tượng đã chạy lệnh nmap để quét cổng nhắm vào tên miền dfir.science
![AfricanFalls Lab-11.png](./images/AfricanFalls%20Lab-11.png)

>dfir.science

## Q8: What country was picture "20210429_152043.jpg" allegedly taken in?

**Yêu cầu:** Xác định quốc gia nơi bức ảnh 20210429_152043.jpg được chụp lại

Định vị bức ảnh 20210429_152043.jpg trong thư mục Pictures\Contact trên cây thư mục FTK Imager
![AfricanFalls Lab-12.png](./images/AfricanFalls%20Lab-12.png)

Đọc siêu dữ liệu EXIF bằng công cụ exiftool tôi thu được tọa độ GPS Position là 16 deg 0' 0.00" S 23 deg 0' 0.00" E
![AfricanFalls Lab-13.png](./images/AfricanFalls%20Lab-13.png)

Nhập tọa độ này lên Google Maps kết quả vị trí tương ứng thuộc địa phận quốc gia Zambia
![AfricanFalls Lab-14.png](./images/AfricanFalls%20Lab-14.png)

>Zambia

## Q9: What is the parent folder name picture "20210429_151535.jpg" was in before the suspect copy it to "contact" folder on his desktop?

**Yêu cầu:** Xác định tên thư mục cha chứa bức ảnh 20210429_151535.jpg trước khi đối tượng sao chép vào thư mục contact trên Desktop

Định vị tệp ảnh 20210429_151535.jpg trong thư mục Pictures\Contact
![AfricanFalls Lab-15.png](./images/AfricanFalls%20Lab-15.png)

Kiểm tra thẻ Camera Model Name qua exiftool tôi ghi nhận thiết bị chụp là LM-Q725K
![AfricanFalls Lab-16.png](./images/AfricanFalls%20Lab-16.png)

Tra cứu mã model này xác định đây là dòng điện thoại thông minh LG Q7
![AfricanFalls Lab-17.png](./images/AfricanFalls%20Lab-17.png)

Khi thiết bị MTP kết nối vào hệ thống Windows lưu lại dấu vết điều hướng thư mục trong ShellBags tại hive UsrClass.dat
![AfricanFalls Lab-18.png](./images/AfricanFalls%20Lab-18.png)

Mở tệp UsrClass.dat bằng ShellBags Explorer và kiểm tra cây thư mục của thiết bị LG Q7 đường dẫn tuyệt đối cho thấy bức ảnh ban đầu nằm trong thư mục gốc Camera
![AfricanFalls Lab-19.png](./images/AfricanFalls%20Lab-19.png)

>Camera

## Q10: A Windows password hashes for an account are below. What is the user's password? Anon:1001:aad3b435b51404eeaad3b435b51404ee:3DE1A36F6DDB8E036DFD75E8E20C4AF4:::

**Yêu cầu:** Bẻ khóa chuỗi băm mật khẩu Windows của tài khoản Anon để tìm mật khẩu dạng văn bản rõ

Tách phần mã băm NTLM 3DE1A36F6DDB8E036DFD75E8E20C4AF4 và tra cứu trên Hashes.com mật khẩu văn bản rõ của tài khoản Anon được giải mã là AFR1CA!
![AfricanFalls Lab-20.png](./images/AfricanFalls%20Lab-20.png)

>AFR1CA!

## Q11: What is the user "John Doe's" Windows login password?

**Yêu cầu:** Xác định mật khẩu đăng nhập Windows của người dùng John Doe

Trích xuất hai hive registry SAM và SYSTEM rồi chạy lệnh mimikatz để giải mã mật khẩu cục bộ
```bash
lsadump::sam /sam:SAM /system:SYSTEM
```

Từ kết quả trích xuất tôi lấy được mã băm NTLM ecf53750b76cc9a62057ca85ff4c850e của người dùng John Doe
![AfricanFalls Lab-21.png](./images/AfricanFalls%20Lab-21.png)

Đưa giá trị băm này lên Hashes.com mật khẩu đăng nhập Windows được tìm thấy là ctf2021
![AfricanFalls Lab-22.png](./images/AfricanFalls%20Lab-22.png)

>ctf2021