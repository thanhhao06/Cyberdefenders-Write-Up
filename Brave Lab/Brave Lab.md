author: Azaki
published: published
created: 2026-10-05
tags: CyberDefenders, Lab, Endpoint Forensic

A memory image was acquired from a suspected compromised Windows workstation. The system belonged to a user flagged for potentially malicious activities, including unauthorized access attempts and unusual browsing patterns. The security team observed network activity to external IPs associated with encrypted communication services.

Your task is to analyze the provided memory dump to uncover details about the processes involved, identify active connections at the time of the compromise, and trace the usage patterns of specific applications.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Champlain College</b> 
</p>

## Q1: What time was the RAM image acquired according to the suspect system?

**Yêu cầu:** Xác định thời điểm thu thập ảnh bộ nhớ RAM theo giờ hệ thống của máy tình nghi

Plugin windows.info của Volatility 3 cho phép trích xuất thông tin tổng quan về bản ghi bộ nhớ bao gồm cấu trúc thời gian SystemTime của hệ điều hành
![Brave Lab.png](./images/Brave%20Lab.png)

>2021-04-30 17:52

## Q2: What is the SHA256 hash value of the RAM image?

**Yêu cầu:** Tính toán giá trị băm SHA256 của tệp ảnh bộ nhớ RAM

Sử dụng tiện ích sha256sum tính toán giá trị băm toàn vẹn cho tệp ảnh bộ nhớ 20210430-Win10Home-20H2-64bit-memdump.mem
![Brave Lab-1.png](./images/Brave%20Lab-1.png)

>9db01b1e7b19a3b2113bfb65e860fffd7a1630bdf2b18613d206ebf2aa0ea172

## Q3: What is the process ID of **brave.exe**?

**Yêu cầu:** Xác định định danh tiến trình PID của brave.exe

Duyệt danh sách các tiến trình đang hoạt động trong bộ nhớ bằng plugin windows.pslist tôi xác định tiến trình brave.exe có mã PID là 4856
![Brave Lab-2.png](./images/Brave%20Lab-2.png)

>4856

## Q4: How many established network connections were there at the time of acquisition?

**Yêu cầu:** Xác định số lượng kết nối mạng ở trạng thái ESTABLISHED tại thời điểm thu thập bộ nhớ

Plugin windows.netscan quét các cấu trúc mạng TCP và UDP trong bộ nhớ kết hợp lệnh lọc trạng thái ESTABLISHED ghi nhận tổng cộng 10 kết nối đang hoạt động
![Brave Lab-3.png](./images/Brave%20Lab-3.png)

>10

## Q5: Which domain name does Chrome have an established network connection with?

**Yêu cầu:** Xác định tên miền mà trình duyệt Chrome đang duy trì kết nối mạng ở trạng thái ESTABLISHED

Sử dụng windows.netscan lọc các kết nối của chrome.exe phát hiện tiến trình PID 1840 đang kết nối tới địa chỉ IP 185.70.41.130 qua cổng 443 ở trạng thái ESTABLISHED
![Brave Lab-5.png](./images/Brave%20Lab-5.png)

Kiểm tra thông tin địa chỉ IP 185.70.41.130 qua cơ chế Reverse DNS xác định tên miền đích tương ứng là protonmail.ch
![Brave Lab-4.png](./images/Brave%20Lab-4.png)

>protonmail.ch

## Q6: What is the MD5 hash value of the process executable for PID **6988**?

**Yêu cầu:** Tính toán giá trị băm MD5 của tệp thực thi thuộc tiến trình có PID 6988

Sử dụng plugin windows.pslist với tùy chọn --dump để trích xuất tệp thực thi của tiến trình OneDrive.exe PID 6988 ra đĩa rồi tính giá trị băm MD5 bằng lệnh md5sum
![Brave Lab-6.png](./images/Brave%20Lab-6.png)

>0b493d8e26f03ccd2060e0be85f430af

## Q7: Can you identify the word that begins at offset **0x45BE876** and is 6 bytes long?

**Yêu cầu:** Xác định từ có độ dài 6 byte bắt đầu tại vị trí offset 0x45BE876 trong bộ nhớ

Sử dụng công cụ xxd điều hướng trực tiếp tới vị trí offset 0x45BE876 trên tệp bộ nhớ đọc chuỗi ký tự ASCII độ dài 6 byte thu được từ hacker
![Brave Lab-7.png](./images/Brave%20Lab-7.png)

>hacker

## Q8: What is the creation date and time of the parent process of **powershell.exe**?

**Yêu cầu:** Xác định ngày giờ khởi tạo của tiến trình cha của powershell.exe

Kiểm tra powershell.exe bằng windows.pslist xác định mã tiến trình cha PPID là 4352 tương ứng với explorer.exe có thời gian khởi tạo CreateTime ghi nhận là 2021-04-30 17:39:48
![Brave Lab-8.png](./images/Brave%20Lab-8.png)

>2021-04-30 17:39

## Q9: What is the full path and name of the last file opened in notepad?

**Yêu cầu:** Xác định đường dẫn đầy đủ và tên của tệp cuối cùng được mở bằng Notepad

Plugin windows.cmdline trích xuất tham số dòng lệnh từ khối môi trường tiến trình PEB của notepad.exe ghi nhận đường dẫn tệp văn bản được mở
![Brave Lab-9.png](./images/Brave%20Lab-9.png)

>C:\Users\JOHNDO~1\AppData\Local\Temp\7zO4FB31F24\accountNum

## Q10: How long did the suspect use **Brave** browser? (In Hours)

**Yêu cầu:** Xác định thời gian đối tượng tình nghi đã sử dụng trình duyệt Brave tính theo giờ

Khóa UserAssist trong Registry NTUSER.DAT ghi lại thời gian tương tác ứng dụng của người dùng Plugin windows.registry.userassist giải mã dữ liệu cho thấy tổng thời gian chạy của Brave là hơn 4 giờ
![Brave Lab-10.png](./images/Brave%20Lab-10.png)

>4