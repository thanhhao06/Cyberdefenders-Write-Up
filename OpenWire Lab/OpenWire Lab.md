author: Azaki
published: published
created: 2026-09-09
tags: CyberDefener, Lab, Threat Intel

During your shift as a tier-2 SOC analyst, you receive an escalation from a tier-1 analyst regarding a public-facing server. This server has been flagged for making outbound connections to multiple suspicious IPs. In response, you initiate the standard incident response protocol, which includes isolating the server from the network to prevent potential lateral movement or data exfiltration and obtaining a packet capture from the NSM utility for analysis. Your task is to analyze the pcap and assess for signs of malicious activity.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@quixote</b> 
</p>


## Q1: By identifying the C2 IP, we can block traffic to and from this IP, helping to contain the breach and prevent further data exfiltration or command execution. Can you provide the IP of the C2 server that communicated with our server?

**Yêu cầu:** Xác định địa chỉ IP của C2 server đã giao tiếp với máy chủ của chúng ta.

Kiểm tra các phiên giao tiếp (conversations) giữa các IP trong file PCAP, ta phát hiện có một địa chỉ IP đang tương tác với máy chủ qua một lượng lớn gói tin (hơn 4800 packets). Địa chỉ IP đáng ngờ này là `146.190.21.92`
![OpenWire Lab.png](./images/OpenWire%20Lab.png)

Để khẳng định, ta tiến hành lọc (filter) các gói tin có địa chỉ IP này. Kết quả cho thấy chúng trao đổi với nhau bằng giao thức TCP thông qua port `443` (HTTPS). Căn cứ vào hành vi và dung lượng dữ liệu trao đổi đủ lớn cho một kênh Command & Control (C2), ta có thể xác nhận đây chính là IP cần tìm
![OpenWire Lab-1.png](./images/OpenWire%20Lab-1.png)

>146.190.21.92

## Q2: Initial entry points are critical to trace the attack vector back. What is the port number of the service the adversary exploited?

**Yêu cầu:** Xác định số hiệu cổng (port number) của dịch vụ đã bị kẻ tấn công khai thác.

Để tìm port của dịch vụ bị khai thác, ta không thể chỉ nhìn vào port giao tiếp C2 (443) ở câu trên. Thay vào đó, tiến hành lọc các gói tin theo phương thức `openwire` trên Wireshark. Khi kiểm tra chi tiết một gói tin OpenWire, tại phần thông tin giao thức TCP, ta xác định được Destination Port (cổng đích) của dịch vụ
![OpenWire Lab-2.png](./images/OpenWire%20Lab-2.png)

>61616

## Q3: Following up on the previous question, what is the name of the service found to be vulnerable?

**Yêu cầu:** Xác định tên của dịch vụ tồn tại lỗ hổng dựa trên kết quả của câu hỏi trước.

Tiếp tục phân tích payload của gói tin `openwire` vừa tìm được. Ở phần WireFormatInfo, tại trường `Magic`, ta có thể thấy rõ chuỗi ký tự nhận diện là `ActiveMQ`. Tên đầy đủ của dịch vụ mạng này là Apache ActiveMQ
![OpenWire Lab-3.png](./images/OpenWire%20Lab-3.png)

>Apache ActiveMQ

## Q4: The attacker's infrastructure often involves multiple components. What is the IP of the second C2 server?

**Yêu cầu:** Hạ tầng của kẻ tấn công thường bao gồm nhiều thành phần. Hãy xác định IP của C2 server thứ hai.

Để tìm được C2 server tiếp theo, ta cần kiểm tra xem C2 server ban đầu có thả file hay tương tác gì với máy nạn nhân không. Lọc các gói tin HTTP, ta phát hiện một request GET tải xuống file cấu hình có tên `invoice.xml`
![OpenWire Lab-4.png](./images/OpenWire%20Lab-4.png)

Kiểm tra sâu vào luồng dữ liệu (TCP stream) của file XML này, ta thấy nó chứa mã lệnh để buộc máy chủ tương tác với một URL bên ngoài. URL này chứa địa chỉ IP của C2 server giai đoạn 2
![OpenWire Lab-5.png](./images/OpenWire%20Lab-5.png)

>128.199.52.72

## Q5: Attackers usually leave traces on the disk. What is the name of the reverse shell executable dropped on the server?

**Yêu cầu:** Xác định tên của tập tin thực thi (reverse shell) đã được thả (drop) xuống máy chủ.

Phân tích đoạn mã bên trong file `invoice.xml` ở câu trước, URL tương tác thực chất đang gọi lệnh `curl` để tải xuống một tập tin có tên là `docker`. Ngay sau đó, nó cấp quyền thực thi (`chmod +x /tmp/docker`) và chạy tập tin này (`./tmp/docker`). Đây chính là payload reverse shell nhằm thiết lập kết nối ngược
![OpenWire Lab-6.png](./images/OpenWire%20Lab-6.png)

>docker

## Q6: What Java class was invoked by the XML file to run the exploit?

**Yêu cầu:** Lớp Java (Java class) nào đã được file XML gọi để thực thi mã khai thác?

Tiếp tục quan sát cấu trúc của file `invoice.xml`. Tại phần cấu hình `<bean>`, thuộc tính `class` đã tiết lộ chính xác tên Java class được sử dụng để khởi tạo tiến trình hệ thống và chạy các lệnh độc hại
![OpenWire Lab-7.png](./images/OpenWire%20Lab-7.png)

>java.lang.ProcessBuilder

## Q7: To better understand the specific security flaw exploited, can you identify the CVE identifier associated with this vulnerability?

**Yêu cầu:** Xác định mã định danh CVE liên quan đến lỗ hổng bảo mật này.

Sau khi thu thập đủ các đặc điểm và hành vi của malware (Khai thác Apache ActiveMQ qua cổng 61616, sử dụng giao thức OpenWire để tải file XML cấu hình Spring framework, qua đó gọi `java.lang.ProcessBuilder` để RCE), ta tiến hành tra cứu các từ khóa này trên các kho lưu trữ bảo mật để tìm ra CVE tương ứng
![OpenWire Lab-8.png](./images/OpenWire%20Lab-8.png)

>CVE-2023-46604

## Q8: The vendor addressed the vulnerability by adding a [validation step](https://github.com/apache/activemq/pull/1098/commits/3eaf3107f4fb9a3ce7ab45c175bfaeac7e866d5b) to ensure that only valid `Throwable` classes can be instantiated, preventing exploitation. In which **Java class** and **method** was this validation step added?

**Yêu cầu:** Nhà phát triển đã vá lỗ hổng bằng cách thêm một bước xác thực. Bước xác thực này được thêm vào Java class và method nào?

Truy cập vào đường dẫn commit trên GitHub đã được cung cấp trong câu hỏi, ta có thể xem các thay đổi mã nguồn (diff) mà nhà phát triển đã thực hiện. Đọc đoạn code được sửa đổi, ta dễ dàng xác định được tên class và method đã được bổ sung thêm hàm xác thực `validateIsThrowable
![OpenWire Lab-9.png](./images/OpenWire%20Lab-9.png)

>BaseDataStreamMarshaller.createThrowable