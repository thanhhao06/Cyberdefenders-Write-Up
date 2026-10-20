author: Azaki
published: published
created: 2026-10-20
tags: CyberDefenders, Lab, Network Forensic

This lab takes you into the world of voice communications on the internet. VoIP is becoming the de-facto standard for voice communication. As this technology becomes more common, malicious parties have more opportunities and stronger motives to control these systems to conduct nefarious activities. This challenge was designed to examine and explore some of the attributes of the SIP and RTP protocols.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@The Honeynet Project</b> 
</p>

## Q1: What is the transport protocol being used?

**Yêu cầu:** Xác định giao thức tầng giao vận được sử dụng cho lưu lượng VoIP trong file pcap

Để phân tích cấu trúc tổng thể của lưu lượng mạng tôi mở công cụ Wireshark và kiểm tra thống kê phân cấp giao thức Protocol Hierarchy Statistics
Lưu lượng thoại VoIP ưu tiên độ trễ thấp và truyền dữ liệu thời gian thực nên phần lớn các gói tin SIP và RTP đều vận hành trên nền giao thức UDP
![Acoustic Lab-18.png](./images/Acoustic%20Lab-18.png)

>UDP

## Q2: The attacker used a bunch of scanning tools that belong to the same suite. Provide the name of the suite.

**Yêu cầu:** Xác định tên bộ công cụ rà quét được kẻ tấn công sử dụng

Kiểm tra nội dung các gói tin SIP OPTIONS gửi đến hệ thống honeypot tôi nhận thấy trường User-Agent mang giá trị friendly-scanner và header From thể hiện chuỗi sipvicious
![Acoustic Lab.png](./images/Acoustic%20Lab.png)

Tra cứu chữ ký nhận diện này trên tài liệu công khai xác nhận đây là bộ công cụ kiểm thử mã nguồn mở SIPVicious chuyên dụng cho hệ thống SIP VoIP
![Acoustic Lab-1.png](./images/Acoustic%20Lab-1.png)

>SIPVicious

## Q3: What is the User-Agent of the victim system?

**Yêu cầu:** Xác định User-Agent của hệ thống nạn nhân

Áp dụng bộ lọc sip trên Wireshark tôi kiểm tra các gói tin phản hồi Status 200 OK từ địa chỉ IP của nạn nhân 172.25.105.40
Header User-Agent trong gói phản hồi tiết lộ chính xác phiên bản Asterisk đang vận hành
![Acoustic Lab-2.png](./images/Acoustic%20Lab-2.png)

>Asterisk PBX 1.6.0.10-FONCORE-r40

## Q4: Which tool was only used against the following extensions: 100,101,102,103, and 111?

**Yêu cầu:** Xác định công cụ trong bộ SIPVicious chỉ nhắm vào các extension 100, 101, 102, 103 và 111

Kiểm tra mã nguồn của bộ công cụ SIPVicious trên GitHub tôi phân tích chức năng của từng file script Python
Trong đó script svcrack.py hỗ trợ tham số range để thực hiện tấn công dò quét mật khẩu nhắm trực tiếp vào danh sách các extension mục tiêu
![Acoustic Lab-3.png](./images/Acoustic%20Lab-3.png)

>svcrack.py

## Q5: Which extension on the honeypot does NOT require authentication?

**Yêu cầu:** Xác định extension trên hệ thống honeypot không yêu cầu xác thực

Để thuận tiện cho việc rà soát tôi kết xuất toàn bộ log SIP sang định dạng csv
Kiểm tra trường Authorization nhận thấy các extension 101 102 103 và 111 đều yêu cầu Digest authentication trong khi extension 100 hoàn toàn không xuất hiện trường xác thực này
![Acoustic Lab-10.png](./images/Acoustic%20Lab-10.png)

>100

## Q6: How many extensions were scanned in total?

**Yêu cầu:** Xác định tổng số extension đã bị kẻ tấn công rà quét

Kẻ tấn công sử dụng các gói tin SIP REGISTER để dò tìm các số máy nhánh trên tổng đài
Bằng cách lọc các gói tin Request-Line mang phương thức REGISTER và loại bỏ các bản ghi trùng lặp tôi xác định được tổng cộng 2652 extension đã bị quét
![Acoustic Lab-11.png](./images/Acoustic%20Lab-11.png)

>2652

## Q7: There is a trace for a real SIP client. What is the corresponding user-agent? (two words, once space in between)

**Yêu cầu:** Xác định User-Agent của client SIP hợp lệ thực tế trong file pcap

Thực hiện lọc các giá trị User-Agent khác nhau xuất hiện trong toàn bộ phiên liên lạc
Ngoài chuỗi friendly-scanner của công cụ quét lưu lượng còn ghi nhận kết nối từ một client phần mềm VoIP thực tế của người dùng
![Acoustic Lab-9.png](./images/Acoustic%20Lab-9.png)

>Zoiper rev.6751

## Q8: Multiple real-world phone numbers were dialed. What was the most recent 11-digit number dialed from extension 101?

**Yêu cầu:** Xác định số điện thoại 11 chữ số gần đây nhất được quay từ extension 101

Tôi thiết lập bộ lọc các gói tin INVITE gửi từ extension 101 để tìm kiếm các cuộc gọi đi
Trong danh sách kết quả hiển thị 3 bản ghi cuộc gọi và số điện thoại 11 chữ số tại mốc thời gian mới nhất được ghi nhận
![Acoustic Lab-12.png](./images/Acoustic%20Lab-12.png)

>00112524021

## Q9: What are the default credentials used in the attempted basic authentication? (format is username:password)

**Yêu cầu:** Xác định thông tin đăng nhập mặc định được sử dụng trong nỗ lực xác thực Basic Auth

Sử dụng bộ lọc tìm kiếm các frame chứa từ khóa Authorization trên giao diện quản trị web HTTP
Tại gói tin HTTP GET trang admin tôi trích xuất được chuỗi Base64 của trường Authorization và giải mã thu được cặp tài khoản mật khẩu
![Acoustic Lab-13.png](./images/Acoustic%20Lab-13.png)

>maint:password

## Q10: Which codec does the RTP stream use? (3 words, 2 spaces in between)

**Yêu cầu:** Xác định codec âm thanh được sử dụng trong luồng dữ liệu RTP

Áp dụng bộ lọc rtp trên Wireshark để kiểm tra cấu trúc tiêu đề của các gói tin truyền tải giọng nói
Trường Payload type trong header Real-Time Transport Protocol chỉ định rõ chuẩn nén âm thanh được áp dụng
![Acoustic Lab-14.png](./images/Acoustic%20Lab-14.png)

>ITU-T G.711 PCMU

## Q11: How long is the sampling time (in milliseconds)?

**Yêu cầu:** Xác định chu kỳ lấy mẫu thời gian tính bằng mili giây

Tra cứu thông số kỹ thuật của chuẩn nén G.711 trên tài liệu viễn thông xác định tần số lấy mẫu của codec này đạt 8000 Hz
![Acoustic Lab-19.png](./images/Acoustic%20Lab-19.png)

Áp dụng công thức vật lý tính chu kỳ lấy mẫu từ tần số f = 8000 Hz

$$T = \frac{1}{f} = \frac{1}{8000\text{ Hz}} = 0.000125\text{ s} = 0.125\text{ ms}$$

Thực hiện phép chia trên máy tính thu được khoảng thời gian giữa các lần lấy mẫu là 0.125 mili giây

>0.125

## Q12: What was the password for the account with username 555?

**Yêu cầu:** Xác định mật khẩu của tài khoản người dùng có username 555

Để tìm mật khẩu nội bộ tôi sử dụng tính năng Export Objects HTTP của Wireshark nhằm trích xuất các file cấu hình được truyền qua mạng
Lọc theo từ khóa conf phát hiện file cấu hình sip_custom.conf được gửi về trình duyệt
![Acoustic Lab-16.png](./images/Acoustic%20Lab-16.png)

Mở nội dung file cấu hình vừa trích xuất và tìm đến khối định danh của người dùng 555 tôi đọc được giá trị trường secret
![Acoustic Lab-15.png](./images/Acoustic%20Lab-15.png)

>1234

## Q13: Which RTP packet header field can be used to reorder out of sync RTP packets in the correct sequence?

**Yêu cầu:** Xác định trường trong tiêu đề gói tin RTP được sử dụng để tái sắp xếp các gói tin bị đảo lộn thứ tự về đúng trình tự

Giao thức RTP truyền tải dữ liệu đa phương tiện thời gian thực qua mạng và sử dụng trường timestamp trong header để phản ánh mốc thời gian lấy mẫu của khối dữ liệu
Nhờ đặc tính thời gian tuyến tính này bên nhận có thể căn cứ vào timestamp để đồng bộ và phát lại các gói tin theo đúng trình tự thời gian ban đầu
![Acoustic Lab-17.png](./images/Acoustic%20Lab-17.png)

>timestamp

## Q14: The trace includes a secret hidden message. Can you hear it?

**Yêu cầu:** Tìm thông điệp bí mật ẩn giấu bên trong luồng âm thanh của file pcap

Để khôi phục âm thanh cuộc gọi tôi truy cập tính năng Telephony trên thanh công cụ Wireshark và chọn mục RTP Player
![Acoustic Lab-8.png](./images/Acoustic%20Lab-8.png)

Wireshark phân tích luồng RTP và hiển thị dạng sóng âm thanh của cuộc gọi thoại cho phép nghe trực tiếp và trích xuất ra file wav
![Acoustic Lab-7.png](./images/Acoustic%20Lab-7.png)

Sau khi trích xuất file âm thanh tôi đưa file vào công cụ chuyển đổi âm thanh sang văn bản AudioToText để đọc toàn bộ nội dung hội thoại và tìm thấy mật khẩu bí mật
![Acoustic Lab-6.png](./images/Acoustic%20Lab-6.png)

>Mexico
