author: Azaki
published: published
created: 2026-10-04
tags: CyberDefenders, Lab, Threat Intel

Threat actors often use living-off-the-land (LOTL) techniques, such as executing PowerShell, to advance their attacks and move away from macro code. This challenge aims to demonstrate how you can quickly analyze artifacts to extract important indicators of compromise (IOCs). The focus of this exercise is on static analysis techniques. As a security blue team analyst, analyze the provided artifacts and answer the questions.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Josh Stroschein</b> 
</p>

## Q1: Multiple streams contain macros in this document. Provide the number of the highest one

**Yêu cầu:** Xác định số hiệu index lớn nhất của stream có chứa mã macro VBA trong tài liệu độc hại

Định dạng tài liệu OLE Structured Storage tổ chức dữ liệu thành các stream riêng biệt và công cụ oledump là giải pháp hàng đầu để phân tích cấu trúc này Khi chạy oledump phân tích tệp sample.bin công cụ gắn cờ chữ cái m hoặc M cho các stream có chứa code macro VBA Quan sát danh sách stream được liệt kê tôi ghi nhận stream có chỉ số index cao nhất chứa mã macro VBA là stream 16
![MalDoc101 Lab.png](./images/MalDoc101%20Lab.png)

>16

## Q2: What event is used to begin the execution of the macros?

**Yêu cầu:** Xác định sự kiện được sử dụng để kích hoạt thực thi macro khi mở tài liệu

Để kích hoạt mã độc tự động ngay khi người dùng mở tệp mà không đòi hỏi thao tác kích hoạt phức tạp kẻ tấn công thường lợi dụng các AutoExec event tích hợp sẵn của Microsoft Office Sử dụng công cụ olevba để bóc tách luồng thực thi tôi phát hiện hàm Document_open được thiết lập để tự động kích hoạt mã macro ngay tại thời điểm tài liệu Word được mở
![MalDoc101 Lab-1.png](./images/MalDoc101%20Lab-1.png)

>Document_open

## Q3: What malware family was this maldoc attempting to drop?

**Yêu cầu:** Xác định họ mã độc mà tài liệu độc hại này cố gắng tải xuống và cài đặt

Tra cứu mã hash của mẫu tệp trên nền tảng VirusTotal giúp đối chiếu chữ ký nhận diện từ hàng loạt hệ thống chống mã độc toàn cầu Kết quả tổng hợp chỉ ra mẫu tài liệu Word này là dropper được thiết kế để phân phối họ trojan ngân hàng khét tiếng emotet
![MalDoc101 Lab-2.png](./images/MalDoc101%20Lab-2.png)

>emotet

## Q4: What stream is responsible for the storage of the base64-encoded string?

**Yêu cầu:** Xác định stream chịu trách nhiệm lưu trữ chuỗi mã hóa Base64

Kẻ tấn công không lưu trực tiếp chuỗi lệnh độc hại trong code VBA mà phân tách và cất giấu dữ liệu trong các thuộc tính của form hoặc luồng đối tượng OLE nhằm lẩn tránh các giải pháp dò quét tĩnh Dựa vào kết quả trích xuất từ olevba kết hợp rà soát danh sách stream của oledump luồng lưu trữ chuỗi Base64 bị làm rối được định vị chính xác tại stream 34
![MalDoc101 Lab-3.png](./images/MalDoc101%20Lab-3.png)

![MalDoc101 Lab-4.png](./images/MalDoc101%20Lab-4.png)

![MalDoc101 Lab-5.png](./images/MalDoc101%20Lab-5.png)

>34

## Q5: This document contains a user-form. Provide the name

**Yêu cầu:** Xác định tên của user-form có trong tài liệu độc hại

Các user-form trong tệp Office thường được đính kèm để chứa các form control hoặc được lợi dụng làm nơi ẩn giấu chuỗi biến môi trường và payload độc hại Khi sử dụng olevba phân tích thành phần nhúng và lọc các tệp form tệp form được khai báo trong tài liệu mang tên roubhaol
![MalDoc101 Lab-6.png](./images/MalDoc101%20Lab-6.png)

>roubhaol

## Q6: This document contains an obfuscated Base64 encoded string; what value is used to pad (or obfuscate) this string?

**Yêu cầu:** Xác định giá trị chuỗi được chèn vào để làm rối chuỗi mã hóa Base64

Kỹ thuật chèn ký tự rác vào giữa chuỗi Base64 phá vỡ định dạng nhận diện thông thường của các hệ thống phát hiện chữ ký Khi phân tích mã nguồn VBA bằng olevba tôi nhận thấy các đoạn chuỗi được nối với nhau và chứa một chuỗi đệm đồng nhất được dùng làm mốc cắt lọc là 2342772g3&*gs7712ffvs626fq
![MalDoc101 Lab-7.png](./images/MalDoc101%20Lab-7.png)

>2342772g3&*gs7712ffvs626fq

## Q7: What is the program executed by the Base64 encoded string?

**Yêu cầu:** Xác định chương trình được thực thi bởi chuỗi mã hóa Base64

Đưa chuỗi Base64 thu thập được vào CyberChef và loại bỏ các chuỗi ký tự đệm gây nhiễu tôi thu được câu lệnh PowerShell hoàn chỉnh Kết quả trích xuất cho thấy chương trình đích được gọi ra để chạy kịch bản độc hại là powersheLL
![MalDoc101 Lab-8.png](./images/MalDoc101%20Lab-8.png)

>powersheLL

## Q8: What WMI class is used to create the process to launch the Trojan?

**Yêu cầu:** Xác định WMI class được sử dụng để tạo tiến trình khởi chạy Trojan

Kỹ thuật Living off the Land thông qua Windows Management Instrumentation cho phép kẻ tấn công tách biệt cây tiến trình con khỏi ứng dụng Office nhằm lẩn tránh sự giám sát của hệ thống EDR Trong đoạn mã PowerShell đã giải mã đối tượng WMI được gọi qua phương thức Create để khởi tạo tiến trình là win32_Process
![MalDoc101 Lab-9.png](./images/MalDoc101%20Lab-9.png)

>win32_Process

## Q9: Multiple domains were contacted to download a Trojan. Provide the first FQDN as per the provided hint

**Yêu cầu:** Xác định tên miền FQDN đầu tiên trong danh sách các domain được liên hệ để tải Trojan

Mã độc triển khai danh sách tên miền dự phòng nhằm đảm bảo tải thành công payload ngay cả khi một số hạ tầng C2 bị triệt hạ Phân tích mảng danh sách URL trong script PowerShell giải mã từ CyberChef tên miền đầy đủ đầu tiên được sử dụng để tải trojan về hệ thống là haoqunkong.com
![MalDoc101 Lab-10.png](./images/MalDoc101%20Lab-10.png)

>haoqunkong.com
