author: Azaki
published: published
created: 2025-12-31
tags: CyberDefener, Lab, Network Forensics

The SOC team has detected suspicious activity in the network traffic, revealing that a machine has been compromised. Sensitive company information has been stolen. Your task is to use Network Capture (PCAP) files and Threat Intelligence to investigate the incident and determine how the breach occurred.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Abdelrhman322</b> 
</p>

## Q1: Which IP address was used by the attacker during the initial access?

**Yêu cầu:** Xác định địa chỉ IP được attacker sử dụng trong giai đoạn initial access.

Đầu tiên, mở file PCAP bằng **Wireshark** và kiểm tra các endpoint xuất hiện trong quá trình giao tiếp.

Có thể sử dụng:
```
Statistics → Conversations → IPv4
```
để quan sát những địa chỉ IP có lượng traffic đáng chú ý

![DanaBot Lab-1.png](./images/DanaBot%20Lab-1.png)
Trong kết quả, địa chỉ `10.2.14.101` xuất hiện trong hầu hết các connection nên có thể xác định đây là máy nạn nhân trong network.

Tiếp theo, filter các packet có destination là máy này:
```
ip.dst == 10.2.14.101
```

Tiếp theo, filter các packet có destination là máy này:
```
ip.dst == 10.2.14.101
```

![DanaBot Lab-2.png](./images/DanaBot%20Lab-2.png)

Ngay ở những packet đầu tiên có thể thấy địa chỉ:
```
62.173.142.148
```
thiết lập TCP connection và gửi HTTP response về `10.2.14.101`.

Trước đó, DNS traffic cũng cho thấy domain:
```
portfolio.serveirc.com
```
resolve về chính địa chỉ `62.173.142.148`. CyberDefenders và IOC của Unit 42 cũng xác nhận `portfolio.serveirc.com` là nguồn cung cấp JavaScript ở giai đoạn đầu của infection chain.

>62.173.142.148

## Q2: What is the name of the malicious file used for initial access?

**Yêu cầu:** Xác định tên file độc hại được sử dụng trong giai đoạn initial access.

Từ kết quả ở câu trước, tiếp tục phân tích HTTP traffic bằng filter:
```
http
```

![DanaBot Lab-3.png](./images/DanaBot%20Lab-3.png)

Một request đáng chú ý xuất hiện ngay đầu traffic:
```
GET /login.php HTTP/1.1
```

Request được gửi từ máy nạn nhân `10.2.14.101` tới server `62.173.142.148`.

Tuy nhiên, `login.php` chỉ là tên resource phía server và chưa chắc là tên thực tế của file được trả về.

Để kiểm tra, chọn packet HTTP response tương ứng và sử dụng **Follow HTTP Stream**.
![DanaBot Lab-4.png](./images/DanaBot%20Lab-4.png)

Trong HTTP response header có trường:
```
Content-Disposition: attachment; filename=allegato_708.js
```

Điều này cho thấy mặc dù request truy cập `/login.php`, server thực tế trả về một JavaScript file có tên:
```
allegato_708.js
```

Unit 42 cũng ghi nhận chính file `allegato_708.js` được tải từ `portfolio.serveirc.com/login.php` trong infection chain DanaBot.

>allegato_708.js

## Q3: What is the SHA-256 hash of the malicious file used for initial access?

**Yêu cầu:** Xác định SHA-256 hash của file `allegato_708.js`.

Sau khi xác định được malicious file, có thể extract HTTP object từ PCAP thông qua:
```
File → Export Objects → HTTP
```

Do resource trên server có tên `/login.php`, file được export có thể vẫn mang tên `login.php`, nhưng nội dung chính là JavaScript `allegato_708.js` đã xác định ở câu trước.

Sau khi export, sử dụng `sha256sum` để tính SHA-256:
![DanaBot Lab-5.png](./images/DanaBot%20Lab-5.png)

>847b4ad90b1daba2d9117a8e05776f3f902dda593fb1252289538acf476c4268

## Q4: Which process was used to execute the malicious file?

**Yêu cầu:** Xác định process được sử dụng để thực thi file JavaScript độc hại.

Sau khi extract `allegato_708.js`, tiến hành phân tích nội dung của script.

File JavaScript đã bị **obfuscate**, khiến source code khó đọc trực tiếp. Tuy nhiên, khi kiểm tra các chuỗi và function được sử dụng có thể tìm thấy nhiều tham chiếu tới:
```
WScript
```

![DanaBot Lab-6.png](./images/DanaBot%20Lab-6.png)

`WScript` liên quan đến **Windows Script Host**, thành phần tích hợp sẵn của Windows có khả năng thực thi các script như `.js` và `.vbs`

>wscript.exe

## Q5: What is the file extension of the second malicious file utilized by the attacker?

**Yêu cầu:** Xác định phần mở rộng của malicious file thứ hai được attacker sử dụng.

Từ câu trước, ta biết `allegato_708.js` chỉ là payload ở giai đoạn đầu. Vì vậy cần tiếp tục phân tích HTTP traffic để tìm file được script tải xuống sau khi thực thi.

Sử dụng lại filter:
```
http
```
và kiểm tra các HTTP GET request tiếp theo.

![DanaBot Lab-7.png](./images/DanaBot%20Lab-7.png)

Một request đáng chú ý xuất hiện:
```
GET /resources.dll HTTP/1.1
```

Điều này cho thấy sau JavaScript đầu tiên, hệ thống tiếp tục tải xuống:
```
resources.dll
```

>.dll

## Q6: What is the MD5 hash of the second malicious file?

**Yêu cầu:** Xác định MD5 hash của malicious file thứ hai `resources.dll`.

Sau khi xác định được request tải `resources.dll`, tiếp tục export file từ HTTP traffic:
```
File → Export Objects → HTTP
```

Chọn và lưu:
```
resources.dll
```

Sau đó tính MD5 hash :
![DanaBot Lab-8.png](./images/DanaBot%20Lab-8.png)

>e758e07113016aca55d9eda2b0ffeebe

---

# ATTACK CHAIN

![DanaBot Lab.png](./images/DanaBot%20Lab.png)