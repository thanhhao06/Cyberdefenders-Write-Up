author: Azaki
published: published
created: 2026-09-30
tags: CyberDefener, Lab, Threat Intel

Karen is a security professional looking for a new job. A company called "TAAUSAI"  offered her a position and asked her to complete a couple of tasks to prove her technical competency. As a soc analyst Analyze the provided disk image and answer the questions based on your understanding of the cases she was assigned to investigate.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Champlain College Digital Forensics Association</b> 
</p>

## Q1: What is the administrator's username?

**Yêu cầu:** Xác định tên tài khoản quản trị viên trên máy tính nạn nhân

Trong quá trình điều tra một bản disk image Windows, việc xác định tài khoản quản trị chính giúp định vị đúng thư mục profile để thu thập các artifact người dùng

Kiểm tra thư mục `[root]\Users` trên ảnh đĩa bằng FTK Imager. Bên cạnh các tài khoản hệ thống mặc định (`Default`, `Public`), tài khoản người dùng thực tế và là quản trị viên hệ thống là `Karen`
![Pasted image 20260930200106.png](./images/Pasted%20image%2020260930200106.png)

>Karen

## Q2: What is the OS's build number?

**Yêu cầu:** Xác định số hiệu bản dựng hệ điều hành (Build Number) của máy tính

Thông tin chi tiết về phiên bản và bản build của hệ điều hành Windows được lưu trữ trong registry hive `SOFTWARE`. Truy cập nhánh
```text
SOFTWARE\Microsoft\Windows NT\CurrentVersion
```

Tại đây, trường `CurrentBuild` hoặc `CurrentBuildNumber` lưu số hiệu bản dựng của Windows 10 (bản 1709 - Fall Creators Update) là `16299`
![Pasted image 20260930185001.png](./images/Pasted%20image%2020260930185001.png)

>16299

## Q3: What is the hostname of the computer?

**Yêu cầu:** Xác định tên máy tính (Hostname) của hệ thống

Tên máy tính của Windows được lưu trữ cố định trong registry hive `SYSTEM`. Điều hướng tới nhánh
```text
SYSTEM\ControlSet001\Control\ComputerName\ComputerName
```

Đọc giá trị chuỗi tại `ComputerName`, hostname của máy là `TOTALLYNOTAHACK`
![Pasted image 20260930185327.png](./images/Pasted%20image%2020260930185327.png)

>TOTALLYNOTAHACK

## Q4: A messaging application was used to communicate with a fellow Alpaca enthusiest. What is the name of the software?

**Yêu cầu:** Xác định phần mềm nhắn tin được sử dụng để liên lạc với người có cùng sở thích về Alpaca

Kiểm tra các ứng dụng bên thứ ba được cài đặt và các tệp trao đổi trong hộp thư của Karen. Trong hòm thư có tệp đính kèm `Skype Convo.zip`, đồng thời trên phân vùng ổ đĩa phát hiện bộ cài đặt `Skype-8.41.0.54.exe`

Điều này xác nhận ứng dụng nhắn tin được sử dụng là Skype
![Pasted image 20260930200028.png](./images/Pasted%20image%2020260930200028.png)

>Skype

## Q5: What is the zip code of the administrator's post?

**Yêu cầu:** Xác định mã bưu chính (Zip Code) trong thông tin bài đăng của quản trị viên

Trình duyệt Google Chrome tự động lưu trữ thông tin điền biểu mẫu (Autofill data) vào cơ sở dữ liệu SQLite `Web Data` nằm tại
```text
AppData\Local\Google\Chrome\User Data\Default\Web Data
```

Trích xuất tệp này và mở bằng DB Browser for SQLite, truy vấn các trường địa chỉ autofill của người dùng Karen, xác định mã zip code được lưu lại là `19709`
![Pasted image 20260930190410.png](./images/Pasted%20image%2020260930190410.png)

>19709

## Q6: What are the initials of the person who contacted the admin user from TAAUSAI?

**Yêu cầu:** Xác định chữ cái viết tắt của người đại diện TAAUSAI đã liên hệ với Karen

Kiểm tra hòm thư đến của Karen, tìm email có tiêu đề `Follow Up Email` gửi từ `Alpaca Activists <13919c46dffe3fe3bc0e41c115892ba7@reply.craigslist.org>`. Ở cuối email, người gửi ký tên viết tắt
```text
Best,
M.S.
--
The Alpaca Association of USA International (TAAUSAI)
```

Hai chữ cái viết tắt là `MS`
![Pasted image 20260930200006.png](./images/Pasted%20image%2020260930200006.png)

>MS

## Q7: How much money was TAAUSAI willing to pay upfront?

**Yêu cầu:** Xác định số tiền mà TAAUSAI sẵn sàng trả trước cho Karen

Đọc nội dung trao đổi công việc trong email từ TAAUSAI gửi tới Karen. Trong thư, phía tuyển dụng đề nghị
```text
"We're willing to pay $150,000 USD upfront, and more at the completion of the job."
```

Karen cũng nhắc lại con số `$150,000` trong email phản hồi. Số tiền trả trước là `150000`
![Pasted image 20260930195807.png](./images/Pasted%20image%2020260930195807.png)

>150000

## Q8: What country is the admin user meeting the hacker group in?

**Yêu cầu:** Xác định quốc gia nơi quản trị viên được hẹn gặp nhóm tin tặc

Trong email có tiêu đề `RE: Interested in the job` gửi từ `taausai@gmail.com` vào ngày 22/03/2019, người gửi không ghi tên quốc gia mà cung cấp tọa độ địa lý
```text
Meet us here "27°22'50.10"N, 33°37'54.62"E"
```

Tra cứu tọa độ này trên bản đồ, vị trí tương ứng thuộc bờ biển El Gouna của Ai Cập (`Egypt`)
![Pasted image 20260930195904.png](./images/Pasted%20image%2020260930195904.png)
![Pasted image 20260930195708.png](./images/Pasted%20image%2020260930195708.png)

>Egypt

## Q9: What is the machine's timezone? (Use the three-letter abbreviation)

**Yêu cầu:** Xác định múi giờ của máy trạm (Sử dụng tên viết tắt 3 chữ cái)

Múi giờ của hệ thống Windows được cấu hình trong registry hive `SYSTEM`. Duyệt tới nhánh
```text
SYSTEM\ControlSet001\Control\TimeZoneInformation
```

Kiểm tra giá trị `TimeZoneKeyName`, múi giờ được thiết lập là `UTC`
![Pasted image 20260930195638.png](./images/Pasted%20image%2020260930195638.png)

>UTC

## Q10: When was AlpacaCare.docx last accessed?

**Yêu cầu:** Xác định mốc thời gian tệp `AlpacaCare.docx` được truy cập lần cuối cùng

Kiểm tra siêu dữ liệu tệp trong hệ thống tệp NTFS tại phân vùng `Horcrux.E01: Partition 3 [3122MB]:PacaLady [NTFS]`

Tìm tệp `AlpacaCare.docx`, cột `Date Accessed` ghi nhận
```text
3/17/2019 9:52:20 PM
```

Quy đổi sang định dạng `YYYY-MM-DD HH:MM` là `2019-03-17 21:52`
![Pasted image 20260930195551.png](./images/Pasted%20image%2020260930195551.png)

>2019-03-17 21:52

## Q11: There was a second partition on the drive. What is the letter assigned to it?

**Yêu cầu:** Xác định ký tự ổ đĩa được gán cho phân vùng thứ hai

Khi người dùng mở tệp từ các phân vùng hoặc ổ đĩa khác, Microsoft Office và Windows sẽ tạo tệp shortcut (.LNK) trong thư mục Recent

Duyệt vào `[root]\Users\Karen\AppData\Roaming\Microsoft\Office\Recent`, phát hiện tệp shortcut có tên `PacaLady (A).LNK`. Ký tự `(A)` chỉ ra phân vùng `PacaLady` được gán ký tự ổ đĩa là `A`
![Pasted image 20260930195431.png](./images/Pasted%20image%2020260930195431.png)

>A

## Q12: What is the answer to the question Company's manager asked Karen?

**Yêu cầu:** Xác định câu trả lời cho câu đố mà người quản lý công ty đã đặt ra cho Karen

Trong email gửi ngày 16/03/2019, người quản lý gửi bài test kiểm tra nhanh bằng một chuỗi Base64
```text
VGhlQ2FyZENyaWVzTm9Nb3Jl`
```

Karen đã giải mã chuỗi này và trả lời qua email vào ngày 17/03/2019
```text
"The answer is TheCardCriesNoMore"
```
![Pasted image 20260930195326.png](./images/Pasted%20image%2020260930195326.png)
![Pasted image 20260930195347.png](./images/Pasted%20image%2020260930195347.png)

>TheCardCriesNoMore

## Q13: What is the job position offered to Karen? (3 words, 2 spaces in between)

**Yêu cầu:** Xác định vị trí công việc được đề nghị cho Karen (3 từ, cách nhau bởi 2 dấu cách)

Trong email xác nhận kết quả bài test gửi ngày 22/03/2019 (`Re: Interested in the job`), người quản lý thông báo vị trí
```text
"The job position we think you'll be an awesome fit for is an entry level cyber security analysts."
```

Tên vị trí công việc gồm 3 từ là `Cyber Security Analyst`
![Pasted image 20260930195243.png](./images/Pasted%20image%2020260930195243.png)

>Cyber Security Analyst

## Q14: When was the admin user password last changed?

**Yêu cầu:** Xác định thời điểm mật khẩu của tài khoản quản trị viên được thay đổi lần cuối cùng

Trong registry hive `SAM`, mỗi tài khoản người dùng được lưu trong một khóa theo RID (Karen có RID `000003E9`). Dữ liệu nhị phân `F` lưu mốc thời gian đổi mật khẩu `Last Password Change Time`

Mở hive `SAM` tại `SAM\Domains\Account\Users\000003E9`, trường này hiển thị
```text
3/21/2019 19:13:09 UTC
```

Định dạng lại là `03/21/2019 19:13:09`
![Pasted image 20260930195153.png](./images/Pasted%20image%2020260930195153.png)

>03/21/2019 19:13:09

## Q15: What version of Chrome is installed on the machine?

**Yêu cầu:** Xác định phiên bản Google Chrome được cài đặt trên máy trạm

Google Chrome lưu trữ chuỗi phiên bản hiện tại trong tệp văn bản `Last Version` tại
```text
AppData\Local\Google\Chrome\User Data\Last Version
```

Mở tệp này để đọc chuỗi phiên bản chính xác là `72.0.3626.121`
![Pasted image 20260930194954.png](./images/Pasted%20image%2020260930194954.png)

>72.0.3626.121

## Q16: What is the URL used to download Skype?

**Yêu cầu:** Xác định URL đã được dùng để tải phần mềm Skype về máy

Cơ sở dữ liệu SQLite `History` của Chrome lưu trữ toàn bộ chuỗi URL chuyển hướng khi tải tệp trong bảng `downloads_url_chains`

Mở `History` trong DB Browser for SQLite, truy vấn bảng `downloads_url_chains` tìm bản ghi tải file `Skype-8.41.0.54.exe`. URL tải tệp nhị phân trực tiếp là
```text
https://download.skype.com/s4l/download/win/Skype-8.41.0.54.exe
```
![Pasted image 20260930195038.png](./images/Pasted%20image%2020260930195038.png)

>https://download.skype.com/s4l/download/win/Skype-8.41.0.54.exe

## Q17: What is the domain name of the website Karen browsed on Alpaca care that the file AlpacaCare.docx is based on?

**Yêu cầu:** Xác định tên miền trang web về chăm sóc Alpaca mà tệp `AlpacaCare.docx` dựa trên đó

Tệp Word định dạng OOXML (`.docx`) lưu trữ các siêu liên kết bên ngoài (External Relationships) trong cấu trúc XML. Sử dụng công cụ `oleobj` để phân tích tệp `AlpacaCare.docx`
```bash
oleobj AlpacaCare.docx
```

Kết quả trích xuất các liên kết ngoài đều trỏ về cùng một tên miền
```text
http://palominoalpacafarm.com/?page_id=24
```

Tên miền trang web là `palominoalpacafarm.com`
![Pasted image 20260930194812.png](./images/Pasted%20image%2020260930194812.png)
![Pasted image 20260930194825.png](./images/Pasted%20image%2020260930194825.png)

>palominoalpacafarm.com