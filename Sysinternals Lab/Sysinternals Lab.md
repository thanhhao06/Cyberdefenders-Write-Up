author: Azaki
published: published
created: 2026-10-04
tags: CyberDefenders, Lab, Endpoint Forensics

A user thought they were downloading the SysInternals tool suite and attempted to open it, but the tools did not launch and became inaccessible. Since then, the user has observed that their system has gradually slowed down and become less responsive.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Cyber5W</b> 
</p>

## Q1: What was the malicious executable file name that the user downloaded?

**Yêu cầu:** Xác định tên tệp thực thi độc hại mà người dùng đã tải xuống

Người dùng tải tệp từ Internet về thư mục tải xuống cá nhân trên hệ điều hành Windows Sử dụng công cụ FTK Imager để phân tích cấu trúc hệ thống tệp NTFS tại đường dẫn Users\IEUser\Downloads tôi phát hiện tệp thực thi đáng ngờ mạo danh bộ công cụ quản trị hệ thống nổi tiếng của Microsoft mang tên SysInternals.exe
![Sysinternals Lab.png](./images/Sysinternals%20Lab.png)

>SysInternals.exe

## Q2: When was the last time the malicious executable file was modified?

**Yêu cầu:** Xác định mốc thời gian sửa đổi cuối cùng của tệp thực thi độc hại

Thuộc tính thời gian của tệp trong hệ thống tệp NTFS được lưu trữ trong bản ghi MFT bao gồm các mốc thời gian tạo lập truy cập và sửa đổi Kiểm tra cột Date Modified của tệp SysInternals.exe trên bảng danh sách tệp của FTK Imager mốc thời gian sửa đổi cuối cùng được ghi nhận là 2022-11-15 21:18
![Sysinternals Lab-1.png](./images/Sysinternals%20Lab-1.png)

>2022-11-15 21:18

## Q3: What is the SHA1 hash value of the malware?

**Yêu cầu:** Xác định giá trị băm SHA1 của tệp mã độc

Tệp hive Amcache.hve lưu trữ dấu vết thực thi ứng dụng bao gồm đường dẫn kích thước và đặc biệt là mã băm SHA1 của các chương trình đã từng kích hoạt trên hệ thống Sử dụng công cụ Registry Explorer mở tệp Amcache.hve và điều hướng tới mục Root\InventoryApplicationFile giá trị băm SHA1 của tệp độc hại SysInternals.exe được ghi nhận là 
![Sysinternals Lab-2.png](./images/Sysinternals%20Lab-2.png)

>fa1002b02fc5551e075ec44bb4ff9cc13d563dcf

## Q4: Based on the Alibaba vendor, what is the malware's family?

**Yêu cầu:** Dựa trên kết quả phân tích của hãng bảo mật Alibaba xác định họ mã độc tương ứng

Đối chiếu mã băm SHA1 trích xuất từ Amcache.hve lên nền tảng VirusTotal giúp tra cứu thông tin nhận diện mối đe dọa từ các hãng an ninh mạng quốc tế Tra cứu theo kết quả phát hiện của hãng Alibaba mẫu mã độc này được định danh thuộc về họ Rozena
![Sysinternals Lab-3.png](./images/Sysinternals%20Lab-3.png)

>Rozena

## Q5: What is the first mapped domain's Fully Qualified Domain Name (FQDN)?

**Yêu cầu:** Xác định tên miền FQDN đầu tiên được mã độc ánh xạ

Trong quá trình phân tích hành vi kết nối mạng của mã độc trên VirusTotal thẻ Relations lưu vết các tên miền mà mẫu tệp thực hiện liên lạc Tên miền FQDN đầu tiên xuất hiện trong danh sách và bị cảnh báo độc hại là www.malware430.com
![Sysinternals Lab-4.png](./images/Sysinternals%20Lab-4.png)

>www.malware430.com

## Q6: The mapped domain is linked to an IP address. What is that IP address?

**Yêu cầu:** Xác định địa chỉ IP được liên kết với tên miền đã ánh xạ

Để điều hướng lưu lượng phân giải tên miền về máy chủ của kẻ tấn công mà không cần thông qua máy chủ DNS công cộng mã độc đã can thiệp vào tệp hosts của hệ điều hành Kiểm tra lịch sử câu lệnh PowerShell lưu trong tệp ConsoleHost_history.txt tại đường dẫn AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine câu lệnh ghi vào tệp hosts cho thấy tên miền www.malware430.com được trỏ về địa chỉ IP 192.168.15.10
![Sysinternals Lab-5.png](./images/Sysinternals%20Lab-5.png)

>192.168.15.10

## Q7: What is the name of the executable dropped by the first-stage executable?

**Yêu cầu:** Xác định tên tệp thực thi được thả ra bởi payload giai đoạn 1

Mã độc triển khai kỹ thuật tấn công nhiều giai đoạn bằng cách thả tệp nhị phân tiếp theo vào thư mục hệ thống để thực hiện các hành vi sâu hơn Theo dõi cây tiến trình từ môi trường sandbox tệp SysInternals.exe đã spawn cmd.exe để thực thi và cài đặt một tệp nhị phân được thả vào thư mục C:\Windows có tên là vmtoolsIO.exe
![Sysinternals Lab-6.png](./images/Sysinternals%20Lab-6.png)

>vmtoolsIO.exe

## Q8: What is the name of the service installed by 2nd-stage executable?

**Yêu cầu:** Xác định tên dịch vụ được cài đặt bởi tệp thực thi giai đoạn 2

Kẻ tấn công thiết lập cơ chế duy trì quyền kiểm soát Persistence bằng cách cài đặt một Windows Service để tự động khởi chạy mã độc cùng hệ thống Quan sát chuỗi dòng lệnh của cmd.exe trong cây tiến trình tham số sc config và net start được sử dụng để kích hoạt dịch vụ mang tên VMwareIOHelperService
![Sysinternals Lab-7.png](./images/Sysinternals%20Lab-7.png)

>VMwareIOHelperService