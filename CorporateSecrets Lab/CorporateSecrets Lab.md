author: Azaki
published: published
created: 2026-10-12
tags: CyberDefenders, Lab, Endpoint Forensic

A Windows forensics challenge prepared by Champlain College Digital Forensics Association for their yearly CTF.

Windows Image Forensics Case created by AccessData® FTK® Imager 4.2.1.4
Your objective as a soc analyst is to analyze the image and answer the question.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Champlain College Digital Forensics Association</b> 
</p>

## Q1: What is the current build number on the system?

**Yêu cầu:** Xác định số hiệu bản dựng hệ điều hành Windows hiện tại của hệ thống

Kiểm tra Registry hive SOFTWARE tại khóa Microsoft\Windows NT\CurrentVersion trường CurrentBuildNumber ghi nhận số hiệu bản dựng là 16299
![CorporateSecrets Lab.png](./images/CorporateSecrets%20Lab.png)

>16299

## Q2: How many users are there?

**Yêu cầu:** Xác định số lượng người dùng có trên hệ thống

Quan sát cây thư mục Users trong FTK Imager tôi xác định có 6 thư mục tài khoản người dùng gồm admin hansel.apricot jim.tomato miriam.grapes suzy.strawberry và tim.apple
![CorporateSecrets Lab-1.png](./images/CorporateSecrets%20Lab-1.png)

>6

## Q3: What is the CRC64 hash of the file "fruit_apricot.jpg"?

**Yêu cầu:** Tính toán giá trị băm CRC64 của tệp ảnh fruit_apricot.jpg

Điều hướng đến thư mục Users\hansel.apricot\Pictures\Saved Pictures trong FTK Imager trích xuất tệp fruit_apricot.jpg
![CorporateSecrets Lab-2.png](./images/CorporateSecrets%20Lab-2.png)

Nạp tệp ảnh vào công cụ tính mã băm CRC64 thu được kết quả ed865aa6dfd756bf
![CorporateSecrets Lab-3.png](./images/CorporateSecrets%20Lab-3.png)

>ed865aa6dfd756bf

## Q4: What is the logical size of the file "strawberry.jpg" in bytes?

**Yêu cầu:** Xác định kích thước logic của tệp strawberry.jpg theo đơn vị byte

Sử dụng lệnh ls -la kiểm tra thông tin chi tiết của tệp strawberry.jpg ghi nhận dung lượng tệp là 72448 byte
![CorporateSecrets Lab-4.png](./images/CorporateSecrets%20Lab-4.png)

>72448

## Q5: What is the processor architecture of the system? (one word)

**Yêu cầu:** Xác định kiến trúc bộ xử lý của hệ thống

Mở Registry hive SYSTEM tại nhánh ControlSet001\Control\Session Manager\Environment kiểm tra giá trị của biến PROCESSOR_ARCHITECTURE xác nhận kiến trúc hệ thống là AMD64
![CorporateSecrets Lab-5.png](./images/CorporateSecrets%20Lab-5.png)

>AMD64

## Q6: Which user has a photo of a dog in their recycling bin?

**Yêu cầu:** Xác định người dùng sở hữu bức ảnh một chú chó trong thùng rác

Kiểm tra thư mục $Recycle.Bin tìm thấy tệp ảnh chú chó $RGETALS.jpg trong thư mục SID S-1-5-21-2446097003-76624807-2828106174-1002 và thuộc tính bảo mật NTFS Access Control Entry xác nhận chủ sở hữu là hansel.apricot
![CorporateSecrets Lab-6.png](./images/CorporateSecrets%20Lab-6.png)

>hansel.apricot

## Q7: What type of file is "vegetable"? Provide the extension without a dot

**Yêu cầu:** Xác định định dạng tệp của vegetable và cung cấp đuôi tệp không có dấu chấm

Kiểm tra cấu trúc hex của tệp vegetable trong thư mục Pictures của miriam.grapes các magic byte đầu tiên là 37 7A BC AF 27 1C đặc trưng cho tệp lưu trữ nén 7z
![CorporateSecrets Lab-7.png](./images/CorporateSecrets%20Lab-7.png)

>7z

## Q8: What type of girls does Miriam Grapes design phones for (Target audience)?

**Yêu cầu:** Xác định đối tượng khách hàng mục tiêu mà Miriam Grapes thiết kế điện thoại hướng tới

Định vị tệp cơ sở dữ liệu lịch sử duyệt web Firefox places.sqlite của người dùng miriam.grapes trong thư mục AppData\Roaming\Mozilla\Firefox\Profiles
![CorporateSecrets Lab-9.png](./images/CorporateSecrets%20Lab-9.png)

Mở places.sqlite bằng DB Browser for SQLite kiểm tra bảng moz_places ghi nhận chuỗi tìm kiếm về loại điện thoại mà các cô gái vsco ưa chuộng
![CorporateSecrets Lab-8.png](./images/CorporateSecrets%20Lab-8.png)

>vsco

## Q9: What is the name of the device?

**Yêu cầu:** Xác định tên của thiết bị máy tính

Truy cập Registry hive SYSTEM tại đường dẫn ControlSet001\Control\ComputerName\ComputerName ghi nhận giá trị khóa ComputerName là DESKTOP-3A4NLVQ
![CorporateSecrets Lab-10.png](./images/CorporateSecrets%20Lab-10.png)

>DESKTOP-3A4NLVQ

## Q10: What is the SID of the machine?

**Yêu cầu:** Xác định mã định danh bảo mật SID của thiết bị

Kiểm tra Registry hive SECURITY tại nhánh Policy\Accounts phát hiện mã định danh tài khoản S-1-5-21-2446097003-76624807-2828106174-501 từ đó suy ra mã SID của máy tính sau khi loại bỏ phần RID phía sau là S-1-5-21-2446097003-76624807-2828106174
![CorporateSecrets Lab-11.png](./images/CorporateSecrets%20Lab-11.png)

>S-1-5-21-2446097003-76624807-2828106174

## Q11: How many web browsers are present?

**Yêu cầu:** Xác định số lượng trình duyệt web có mặt trên hệ thống

Kiểm tra thư mục Windows\Prefetch tìm thấy dấu vết thực thi của Internet Explorer qua tệp IEXPLORE.EXE-908C99F8.pf
![CorporateSecrets Lab-16.png](./images/CorporateSecrets%20Lab-16.png)

Dấu vết thực thi của trình duyệt Google Chrome qua tệp CHROME.EXE-D999B1BA.pf
![CorporateSecrets Lab-15.png](./images/CorporateSecrets%20Lab-15.png)

Dấu vết của Windows Explorer hoặc trình duyệt tích hợp qua tệp EXPLORER.EXE-A80E4F97.pf
![CorporateSecrets Lab-14.png](./images/CorporateSecrets%20Lab-14.png)

Dấu vết thực thi của Mozilla Firefox qua tệp FIREFOX.EXE-20153F0F.pf
![CorporateSecrets Lab-13.png](./images/CorporateSecrets%20Lab-13.png)

Dấu vết thực thi của trình duyệt ẩn danh Tor Browser qua tệp TOR.EXE-4B50033F.pf xác nhận tổng cộng có 5 trình duyệt xuất hiện trên hệ thống
![CorporateSecrets Lab-29.png](./images/CorporateSecrets%20Lab-29.png)

>5

## Q12: How many super secret CEO plans does Tim have? (Dr. Doofenshmirtz Type Beat)

**Yêu cầu:** Xác định số lượng kế hoạch bí mật của CEO Tim

Mở tài liệu kế hoạch của CEO Tim trên máy tính chỉ hiển thị 3 gạch đầu dòng kế hoạch
![CorporateSecrets Lab-17.png](./images/CorporateSecrets%20Lab-17.png)

Bôi đen vùng văn bản và đổi màu chữ thì phát hiện thêm một dòng chữ ẩn nên tổng số kế hoạch bí mật là 4
![CorporateSecrets Lab-18.png](./images/CorporateSecrets%20Lab-18.png)

>4

## Q13: Which employee does Tim plan to fire? (He's Dead, Tim. Enter the full name - two words - space separated)

**Yêu cầu:** Xác định tên nhân viên mà Tim dự định sa thải

Dòng chữ ẩn được tô sáng trong danh sách kế hoạch bí mật của Tim nêu rõ kế hoạch sa thải nhân viên Jim Tomato
![CorporateSecrets Lab-19.png](./images/CorporateSecrets%20Lab-19.png)

>Jim Tomato

## Q14: What was the last used username? (I didn't start this conversation, but I'm ending it!)

**Yêu cầu:** Xác định tên người dùng đăng nhập và thao tác lần cuối trên hệ thống

Trích xuất tệp nhật ký sự kiện bảo mật Security.evtx từ thư mục Windows\System32\winevt\Logs
![CorporateSecrets Lab-21.png](./images/CorporateSecrets%20Lab-21.png)

Mở tệp nhật ký lọc Event ID 4647 ghi nhận phiên đăng xuất gần nhất thuộc về tài khoản jim.tomato
![CorporateSecrets Lab-20.png](./images/CorporateSecrets%20Lab-20.png)

>jim.tomato

## Q15: What was the role of the employee Tim was flirting with?

**Yêu cầu:** Xác định vai trò vị trí của nhân viên mà Tim tán tỉnh

Kiểm tra lịch sử duyệt web Firefox của Tim trong bảng moz_places phát hiện chuỗi tìm kiếm liên quan đến việc tán tỉnh thư ký secretary
![CorporateSecrets Lab-22.png](./images/CorporateSecrets%20Lab-22.png)

>secretary

## Q16: What is the SID of the user "suzy.strawberry"?

**Yêu cầu:** Xác định mã định danh SID của người dùng suzy.strawberry

Mở Registry hive SOFTWARE tại đường dẫn Microsoft\Windows NT\CurrentVersion\ProfileList đối chiếu đường dẫn hồ sơ C:\Users\suzy.strawberry lấy được mã định danh RID là 1004
![CorporateSecrets Lab-23.png](./images/CorporateSecrets%20Lab-23.png)

>1004

## Q17: List the file path for the install location of the Tor Browser

**Yêu cầu:** Xác định đường dẫn thư mục cài đặt của trình duyệt Tor Browser

Tra cứu dấu vết thực thi chương trình trong AppCompatCache ShimCache thuộc Registry hive SYSTEM tìm thấy đường dẫn cài đặt của Tor Browser bắt đầu từ thư mục C:\Program1
![CorporateSecrets Lab-24.png](./images/CorporateSecrets%20Lab-24.png)

>C:\Program1

## Q18: What was the URL for the Youtube video watched by Jim?

**Yêu cầu:** Xác định đường dẫn URL của video Youtube mà Jim đã xem

Điều hướng vào thư mục AppData\Local\Google\Chrome\User Data\Default của người dùng jim.tomato trích xuất tệp History
![CorporateSecrets Lab-26.png](./images/CorporateSecrets%20Lab-26.png)

Mở tệp History bằng DB Browser for SQLite kiểm tra bảng urls ghi nhận liên kết video Youtube hướng dẫn tấn công máy tính
![CorporateSecrets Lab-25.png](./images/CorporateSecrets%20Lab-25.png)

>https://www.youtube.com/watch?v=Y-CsIqTFEyY

## Q19: Which user installed LibreCAD on the system?

**Yêu cầu:** Xác định người dùng đã tải và cài đặt phần mềm LibreCAD

Kiểm tra thư mục Downloads của các người dùng trong FTK Imager tìm thấy bộ cài đặt LibreCAD-Installer-2.1.3.exe nằm trong tài khoản miriam.grapes
![CorporateSecrets Lab-27.png](./images/CorporateSecrets%20Lab-27.png)

>miriam.grapes

## Q20: How many times "admin" logged into the system?

**Yêu cầu:** Xác định số lần tài khoản admin đã đăng nhập vào hệ thống

Mở Registry hive SAM tại nhánh SAM\Domains\Account\Users kiểm tra thông tin tài khoản admin mã RID 1001 tại cột Total Logins ghi nhận số lần đăng nhập là 10
![CorporateSecrets Lab-28.png](./images/CorporateSecrets%20Lab-28.png)

>10

## Q21: What is the name of the DHCP domain the device was connected to?

**Yêu cầu:** Xác định tên miền DHCP mà thiết bị đã kết nối tới

Kiểm tra cấu hình giao diện mạng tại ControlSet001\Services\Tcpip\Parameters\Interfaces trong Registry hive SYSTEM trường DhcpDomain lưu tên miền fruitinc.xyz
![CorporateSecrets Lab-31.png](./images/CorporateSecrets%20Lab-31.png)

>fruitinc.xyz

## Q22: What time did Tim download his background image? (Oh Boy 3AM . Answer in MM/DD/YYYY HH:MM format (UTC).)

**Yêu cầu:** Xác định thời điểm Tim tải ảnh hình nền về máy theo chuẩn UTC

Kiểm tra siêu dữ liệu tệp ảnh hình nền hqdefault.jpg trong thư mục Pictures\Saved Pictures của tim.apple ghi nhận mốc thời gian Date Created là 04/05/2020 03:49
![CorporateSecrets Lab-32.png](./images/CorporateSecrets%20Lab-32.png)

>04/05/2020 03:49

## Q23: How many times did Jim launch the Tor Browser?

**Yêu cầu:** Xác định số lần Jim đã khởi chạy trình duyệt Tor Browser

Trích xuất tệp NTUSER.DAT từ thư mục người dùng jim.tomato để phân tích dấu vết UserAssist
![CorporateSecrets Lab-34.png](./images/CorporateSecrets%20Lab-34.png)

Mở NTUSER.DAT bằng Registry Explorer tra cứu đường dẫn C:\Program1\Browser\firefox.exe trong khóa UserAssist ghi nhận số lần chạy Run Counter là 2
![CorporateSecrets Lab-33.png](./images/CorporateSecrets%20Lab-33.png)

>2

## Q24: There is a png photo of an iPhone in Grapes's files. Find it and provide the SHA-1 hash.

**Yêu cầu:** Tìm tệp ảnh PNG chứa hình iPhone trong dữ liệu của Grapes và cung cấp giá trị băm SHA-1

Kiểm tra phần mở rộng tệp samplePhone.jpg bằng lệnh file và exiftool phát hiện đây thực chất là tệp định dạng WEBP
![CorporateSecrets Lab-35.png](./images/CorporateSecrets%20Lab-35.png)

Sử dụng công cụ binwalk phân tích cấu trúc nhị phân phát hiện một tệp ảnh PNG được nhúng ẩn bên trong tại vị trí offset 5962
![CorporateSecrets Lab-37.png](./images/CorporateSecrets%20Lab-37.png)

Trích xuất tệp ảnh PNG bằng công cụ foremost và tính giá trị băm SHA-1 bằng lệnh sha1sum thu được kết quả 537fe19a560ba3578d2f9095dc2f591489ff2cde
![CorporateSecrets Lab-36.png](./images/CorporateSecrets%20Lab-36.png)

>537fe19a560ba3578d2f9095dc2f591489ff2cde

## Q25: When was the last time a docx file was opened on the device? (An apple a day keeps the docx away. Answer in UTC, YYYY-MM-DD HH:MM:SS)

**Yêu cầu:** Xác định thời điểm gần nhất một tệp docx được mở trên thiết bị

Kiểm tra nhánh Software\Microsoft trong NTUSER.DAT của jim.tomato nhận thấy hệ thống không cài đặt bộ ứng dụng Microsoft Office
![CorporateSecrets Lab-39.png](./images/CorporateSecrets%20Lab-39.png)

Chuyển sang kiểm tra khóa RecentDocs\.docx dưới nhánh Windows Shell trong Registry NTUSER.DAT ghi nhận thời điểm mở tệp gần nhất là 2020-04-11 23:23:36
![CorporateSecrets Lab-38.png](./images/CorporateSecrets%20Lab-38.png)

>2020-04-11 23:23:36

## Q26: How many entries does the MFT of the filesystem have?

**Yêu cầu:** Xác định tổng số lượng bản ghi entry trong bảng $MFT của hệ thống tệp

Kiểm tra thuộc tính của tệp $MFT trong FTK Imager ghi nhận dung lượng tệp là 225181696 byte
![CorporateSecrets Lab-41.png](./images/CorporateSecrets%20Lab-41.png)

Mỗi bản ghi MFT entry trong định dạng NTFS có kích thước cố định 1024 byte lấy tổng dung lượng chia cho 1024 tính ra 219904 bản ghi
![CorporateSecrets Lab-40.png](./images/CorporateSecrets%20Lab-40.png)

>219904

## Q27: Tim wanted to fire an employee because they were ......?(Be careful what you wish for)

**Yêu cầu:** Xác định lý do Tim muốn sa thải nhân viên

Trích xuất tệp History trình duyệt Chrome của người dùng tim.apple trong FTK Imager
![CorporateSecrets Lab-43.png](./images/CorporateSecrets%20Lab-43.png)

Mở cơ sở dữ liệu kiểm tra bảng urls phát hiện Tim đã tìm kiếm cách sa thải nhân viên bốc mùi stinky
![CorporateSecrets Lab-42.png](./images/CorporateSecrets%20Lab-42.png)

>stinky

## Q28: What cloud service was a Startup item for the user admin?

**Yêu cầu:** Xác định dịch vụ đám mây nằm trong mục khởi động của người dùng admin

Kiểm tra thư mục Startup trong AppData của tài khoản admin phát hiện lối tắt OneDrive.lnk được thiết lập tự khởi chạy cùng hệ thống
![CorporateSecrets Lab-44.png](./images/CorporateSecrets%20Lab-44.png)

>OneDrive

## Q29: Which Firefox prefetch file has the most runtimes? (Flag format is [filename]/[number])

**Yêu cầu:** Xác định tệp prefetch của Firefox có số lần thực thi nhiều nhất

Trích xuất các tệp prefetch liên quan đến Firefox từ thư mục Windows\Prefetch
![CorporateSecrets Lab-45.png](./images/CorporateSecrets%20Lab-45.png)

Phân tích các tệp prefetch ghi nhận tệp FIREFOX.EXE-A606B53C.pf có số lần thực thi Run Counter cao nhất là 21 lần
![CorporateSecrets Lab-30.png](./images/CorporateSecrets%20Lab-30.png)

>FIREFOX.EXE-A606B53C.pf/21

## Q30: What was the last IP address the machine was connected to?

**Yêu cầu:** Xác định địa chỉ IP cuối cùng mà máy tính đã nhận kết nối

Kiểm tra Registry hive SYSTEM tại nhánh Tcpip\Parameters\Interfaces ghi nhận giá trị khóa DhcpIPAddress được cấp phát cho card mạng là 192.168.2.242
![CorporateSecrets Lab-46.png](./images/CorporateSecrets%20Lab-46.png)

>192.168.2.242

## Q31: Which user had the most items pinned to their taskbar?

**Yêu cầu:** Xác định người dùng ghim nhiều mục nhất vào thanh tác vụ Taskbar

Rà soát đường dẫn User Pinned\TaskBar trong AppData của tất cả tài khoản ghi nhận người dùng admin có số lượng mục ghim nhiều nhất với 3 lối tắt
![CorporateSecrets Lab-47.png](./images/CorporateSecrets%20Lab-47.png)

>admin

## Q32: What was the last run date of the executable with an MFT record number of 164885? (Format: MM/DD/YYYY HH:MM:SS (UTC).)

**Yêu cầu:** Xác định thời điểm chạy cuối cùng của tệp thực thi có số bản ghi MFT là 164885 theo chuẩn UTC

Sử dụng công cụ MFTECmd truy vấn trực tiếp số bản ghi 164885 trong $MFT xác định tên tệp thực thi tương ứng là 7zG.exe
![CorporateSecrets Lab-48.png](./images/CorporateSecrets%20Lab-48.png)

Định vị tệp prefetch tương ứng 7ZG.EXE-0F8C4081.pf trong thư mục Windows\Prefetch
![CorporateSecrets Lab-50.png](./images/CorporateSecrets%20Lab-50.png)

Phân tích tệp prefetch đọc mốc thời gian thực thi gần nhất quy đổi sang chuẩn UTC là 04/12/2020 02:32:09
![CorporateSecrets Lab-49.png](./images/CorporateSecrets%20Lab-49.png)

>04/12/2020 02:32:09

## Q33: What is the log file sequence number for the file "fruit_Assortment.jpg"?

**Yêu cầu:** Xác định số thứ tự chuỗi tệp nhật ký Log File Sequence Number của fruit_Assortment.jpg

Sử dụng công cụ MFTECmd trích xuất toàn bộ dữ liệu $MFT ra định dạng bảng CSV
![CorporateSecrets Lab-58.png](./images/CorporateSecrets%20Lab-58.png)

Mở bảng kết quả tìm kiếm bản ghi của tệp fruit_Assortment.jpg tại dòng 199073
![CorporateSecrets Lab-57.png](./images/CorporateSecrets%20Lab-57.png)

Tra cứu cột Log File Sequence Number của bản ghi này lấy được giá trị 1276820064
![CorporateSecrets Lab-59.png](./images/CorporateSecrets%20Lab-59.png)

>1276820064

## Q34: Jim has some dirt on the company stored in a docx file. Find it, the flag is the fourth secret, in the format of <"The flag is a sentence you put in quotes">. (Secrets, secrets are no fun)

**Yêu cầu:** Tìm bí mật thứ tư được cất giấu trong tệp docx của Jim

Kiểm tra tài liệu docx có sẵn trong hồ sơ của Jim nhưng phần nội dung chỉ chứa thông điệp ngụy trang
![CorporateSecrets Lab-53.png](./images/CorporateSecrets%20Lab-53.png)

Mở thư mục $Recycle.Bin của Jim phát hiện các tệp tài liệu đã bị xóa gồm $RQ1FSDY.docx và $RK6JCRY.docx
![CorporateSecrets Lab-54.png](./images/CorporateSecrets%20Lab-54.png)

Giải nén tệp $RQ1FSDY.docx phát hiện tệp tin bất thường mang tên Content.xml khác với cáu trúc của các file docs còn lại
![CorporateSecrets Lab-55.png](./images/CorporateSecrets%20Lab-55.png)

Sử dụng lệnh file kiểm tra cấu trúc của Content.xml xác định đây thực chất là một tài liệu Microsoft Word hoàn chỉnh
![CorporateSecrets Lab-56.png](./images/CorporateSecrets%20Lab-56.png)

Đổi tên tệp thành phần mở rộng .docx và giải nén để tiếp cận cấu trúc tệp word/document.xml
![CorporateSecrets Lab-52.png](./images/CorporateSecrets%20Lab-52.png)

Mở document.xml đọc danh sách các bí mật công ty trong đó bí mật thứ tư là Customer data is not stored securely
![CorporateSecrets Lab-51.png](./images/CorporateSecrets%20Lab-51.png)

>Customer data is not stored securely

## Q35: In the company Slack, what is threatened to be deactivated if the user gets their email deactivated?

**Yêu cầu:** Xác định bộ phận cơ thể bị đe dọa vô hiệu hóa nếu email của người dùng bị hủy kích hoạt trong Slack công ty

Điều hướng vào thư mục lưu trữ LevelDB của ứng dụng Slack trong AppData trích xuất tệp nhật ký 000003.log
![CorporateSecrets Lab-61.png](./images/CorporateSecrets%20Lab-61.png)

Do đã biết từ cần tìm nằm trong 1 đoạn chat nên cần phải lọc các ký tự của hệ thống và chỉ hiển thị các đoạn có 4 từ trở lên

```
strings 000003.log | grep -Eo '([a-zA-Z]+[[:space:]]+){3,}[a-zA-Z.,?]+'
```

Tại đây tìm thấy 1 đoạn chữ in hoa trông như 1 thông báo quan trọng và phía sau là 1 đoạn về email vì thế đoạn giữa chính là đoạn chứa từ cần tìm
![CorporateSecrets Lab-60.png](./images/CorporateSecrets%20Lab-60.png)

>kneecaps
