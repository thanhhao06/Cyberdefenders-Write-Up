author: Azaki
published: published
created: 2025-09-08
tags: CyberDefener, Lab, Endpoint Forensic

As a member of the Security Blue team, your assignment is to analyze a memory dump using Redline and Volatility tools. Your goal is to trace the steps taken by the attacker on the compromised machine and determine how they managed to bypass the Network Intrusion Detection System (NIDS). Your investigation will identify the specific malware family employed in the attack and its characteristics. Additionally, your task is to identify and mitigate any traces or footprints left by the attacker.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Ahmed Tammam</b> 
</p>

## Q1: What is the name of the suspicious process?

**Yêu cầu:** Xác định tên của tiến trình đáng ngờ (suspicious process) đang chạy trên hệ thống.

Bắt đầu bằng việc kiểm tra danh sách các tiến trình đang thực thi từ bản sao chép bộ nhớ (memory dump) . Kết quả phân tích ghi nhận một tiến trình lạ mang tên `oneetx.exe` có nhiều dấu hiệu khả nghi
![RedLine Lab.png](./images/RedLine%20Lab.png)


>oneetx.exe

## Q2: What is the child process name of the suspicious process?

**Yêu cầu:** Xác định tên tiến trình con (child process) của tiến trình đáng ngờ đã tìm thấy.

Để điều tra mối quan hệ giữa các tiến trình, ta sử dụng plugin `pstree` của Volatility. Mặc dù liên kết cha-con không hiển thị trực tiếp một cách rõ ràng, nhưng việc phân tích dòng thời gian (timeline) cho thấy điểm bất thường. Tiến trình `oneetx.exe` được khởi tạo vào lúc 22:30:56, và ngay sau đó vào lúc 22:31:53, tiến trình `rundll32.exe` xuất hiện. Chuỗi thời gian này là minh chứng mạnh mẽ cho thấy `rundll32.exe` chính là tiến trình con do mã độc sinh ra để thực thi các tác vụ ngầm
![RedLine Lab-1.png](./images/RedLine%20Lab-1.png)

>rundll32.exe

## Q3: What is the memory protection applied to the suspicious process memory region?

**Yêu cầu:** Cho biết cơ chế bảo vệ bộ nhớ (memory protection) nào được áp dụng cho vùng nhớ của tiến trình đáng ngờ.

Sử dụng plugin `malfind` để kiểm tra các đặc tính bất thường trong không gian bộ nhớ của tiến trình `oneetx.exe` . Quan sát trường "Protection" trong kết quả đầu ra, ta thấy vùng nhớ này được cấp quyền `PAGE_EXECUTE_READWRITE`. Đây là dấu hiệu rất đặc trưng của kỹ thuật chèn mã độc (code injection), cho phép mã độc vừa ghi dữ liệu vừa thực thi mã lệnh trực tiếp trên vùng nhớ đó
![RedLine Lab-2.png](./images/RedLine%20Lab-2.png)

>PAGE_EXECUTE_READWRITE

## Q4: What is the name of the process responsible for the VPN connection?

**Yêu cầu:** Tên của tiến trình chịu trách nhiệm tạo kết nối VPN là gì?

Để tìm các kết nối mạng, plugin `netscan` được sử dụng]. Tiến hành lọc (grep) danh sách để chỉ hiển thị các kết nối sử dụng cổng `443`, một cổng tiêu chuẩn thường được dùng cho lưu lượng VPN và HTTPS. Kết quả cho thấy tiến trình `tun2socks.exe` đang duy trì kết nối mạng
![RedLine Lab-3.png](./images/RedLine%20Lab-3.png)

Tuy nhiên, `tun2socks.exe` chỉ là công cụ thực hiện kết nối. Để tìm ra ứng dụng gốc đứng sau tiến trình này, ta quay lại kiểm tra với plugin `pstree`. Tra cứu sơ đồ cây tiến trình xác nhận rằng `Outline.exe` chính là tiến trình cha đã khởi tạo `tun2socks.exe` để thiết lập đường hầm VPN
![RedLine Lab-4.png](./images/RedLine%20Lab-4.png)

>outline.exe

## Q5: What is the attacker's IP address?

**Yêu cầu:** Địa chỉ IP của kẻ tấn công là gì?

Với việc đã xác định được `oneetx.exe` là tiến trình độc hại chính do kẻ tấn công thả vào máy nạn nhân, ta tiếp tục dùng plugin `netscan` và lọc các kết nối chỉ liên quan đến tiến trình này. Từ đó, ta phát hiện được địa chỉ IP bên ngoài (C2 Server) mà mã độc đang kết nối tới
![RedLine Lab-5.png](./images/RedLine%20Lab-5.png)

>77.91.124.20

## Q6: What is the full URL of the PHP file that the attacker visited?

**Yêu cầu:** Đường dẫn URL đầy đủ của tệp PHP mà kẻ tấn công đã truy cập là gì?

Theo yêu cầu tìm kiếm một tệp `.php`, ban đầu việc sử dụng plugin `filescan` không đem lại kết quả khả quan. Đối mặt với tình huống này, ta chuyển hướng sang việc trích xuất trực tiếp các chuỗi ký tự (strings) từ toàn bộ file bộ nhớ]. Mặc dù lượng dữ liệu là khổng lồ, việc tìm kiếm dựa trên từ khóa kết hợp IP đã nhanh chóng làm lộ diện đường dẫn URL đầy đủ trỏ đến tệp PHP

![RedLine Lab-6.png](./images/RedLine%20Lab-6.png)


>http://77.91.124.20/store/games/index.php

## Q7: What is the full path of the malicious executable?

**Yêu cầu:** Đường dẫn tuyệt đối (full path) của tập tin thực thi độc hại trên ổ đĩa là gì?

Kẻ tấn công có vẻ đã thực hiện các biện pháp xóa dấu vết, khiến plugin `cmdline` (kiểm tra dòng lệnh thực thi) không thể tìm thấy thông tin đường dẫn. Tuy nhiên, mặc dù thất bại ở câu Q6, plugin `filescan` lại hoàn thành xuất sắc vai trò quét cấu trúc đối tượng (object) trong bộ nhớ ở câu này, qua đó khôi phục thành công đường dẫn gốc của mã độc `oneetx.exe`

![RedLine Lab-7.png](./images/RedLine%20Lab-7.png)

>C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe