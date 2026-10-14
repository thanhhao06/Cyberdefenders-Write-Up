author: Azaki
published: published
created: 2026-01-04
tags: CyberDefener, Lab, Threat Intel

A cyber threat group was identified for initiating widespread phishing campaigns to distribute further malicious payloads. The most frequently encountered payloads were IcedID. You have been given a hash of an IcedID sample to analyze and monitor the activities of this advanced persistent threat (APT) group.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefender</b> 
</p>

## Q1: What is the name of the file associated with the given hash?

**Yêu cầu:** Xác định tên file liên quan đến mã băm được cung cấp.

Đầu tiên, chúng ta bắt đầu bằng việc thu thập thông tin cơ bản của mẫu malware thông qua mã băm (hash) được cung cấp.
![Pasted image 20260907124050.png](./images/Pasted%20image%2020260907124050.png)

Sau khi VirusTotal trả về kết quả phân tích, chuyển sang tab **Details**. Kiểm tra danh sách các tên file (Names) đã từng được ghi nhận đi kèm với mã băm này.
![Pasted image 20260907124108.png](./images/Pasted%20image%2020260907124108.png)

Kết quả cho thấy file gốc có tên là `document-1982481273.xlsm`. Đuôi `.xlsm` chỉ ra đây là một tệp Microsoft Excel có chứa macro (Excel macro-enabled workbook). Đây là một vector tấn công rất phổ biến, kẻ gian thường lừa người dùng bật macro (Enable Content) để thực thi mã độc ẩn bên trong.

>document-1982481273.xlsm

## Q2: Can you identify the filename of the **GIF** file that was deployed?

**Yêu cầu:** Xác định tên file GIF được payload triển khai xuống hệ thống.

Sau khi nạn nhân mở file Excel và kích hoạt macro, mã độc sẽ bắt đầu thực thi chuỗi lây nhiễm (infection chain). Để quan sát quá trình này, tải mẫu `.xlsm` lên **Tria.ge**.

Theo dõi tab phần Contacted URLs , ta nhận thấy tiến trình `EXCEL.EXE` đã tạo ra các kết nối mạng ra bên ngoài (outbound traffic) để tải về payload ở giai đoạn tiếp theo (second-stage payload).
![Pasted image 20260907125649.png](./images/Pasted%20image%2020260907125649.png)

Trong danh sách các tệp được tải xuống (Dropped Files), có một file đáng ngờ mang tên `3003.gif`. Mặc dù sử dụng phần mở rộng `.gif` để ngụy trang thành một tệp hình ảnh vô hại nhằm vượt qua các hệ thống phát hiện xâm nhập (IDS/IPS), nhưng khi kiểm tra magic bytes (file signature), thực chất đây là một file thực thi dạng DLL (Dynamic Link Library) của IcedID.
![Pasted image 20260907124525.png](./images/Pasted%20image%2020260907124525.png)

>3003.gif
## Q3: How many domains does the malware look to download the additional payload file in **Q2**?

**Yêu cầu:** Xác định số lượng domain mà malware kết nối để tải xuống payload phụ trợ ở câu hỏi 2.

Để đảm bảo tỷ lệ lây nhiễm thành công, mã độc IcedID thường không chỉ dựa vào một máy chủ duy nhất. Phân tích sâu hơn ta vào tab Relation và tìm phần Contacted URLs tại đây có thể thấy 5 doamin mà malware dùng để kết nối và tải payload
![Pasted image 20260907125444.png](./images/Pasted%20image%2020260907125444.png)

Malware sử dụng một vòng lặp để lần lượt kết nối đến các domain này cho đến khi tải thành công payload. Việc đếm số lượng các domain duy nhất trong danh sách C2 (Command and Control) cho kết quả là 5 domain khác nhau.

>5
## Q4: From the domains mentioned in **Q3**, a DNS registrar was predominantly used by the threat actor to host their harmful content, enabling the malware's functionality. Can you specify the Registrar INC?

**Yêu cầu:** Xác định DNS Registrar (nhà đăng ký tên miền) được threat actor sử dụng chủ yếu để host nội dung độc hại.

Việc xác định hạ tầng mạng của kẻ tấn công có thể cung cấp nhiều thông tin tình báo (Threat Intelligence) giá trị. Lấy danh sách 5 domain đã thu thập được ở Q3, chúng ta tiến hành tra cứu thông tin với phần Contacted Domains
![Pasted image 20260907130315.png](./images/Pasted%20image%2020260907130315.png)

Phân tích Registrar của cả 5 domain, hầu hết các domain đều không có Registrar nhưng tại domain **tajushariya.com** thì thấy được domain này được dăng ký bởi NameCheap, Inc.

>NAMECHEAP
## Q5: Could you specify the threat actor linked to the sample provided?

**Yêu cầu:** Xác định nhóm threat actor đứng sau mẫu malware này.

Để liên kết (attribute) mẫu mã độc này với một nhóm tấn công cụ thể, chúng ta cần tổng hợp các TTPs (Tactics, Techniques, and Procedures) đã quan sát được: lây nhiễm ban đầu qua file `.xlsm`, tải payload phụ trợ ngụy trang dưới dạng `.gif`, và sử dụng họ malware IcedID (còn gọi là BokBot).
![Pasted image 20260907151535.png](./images/Pasted%20image%2020260907151535.png)

Tra cứu các báo cáo phân tích từ cơ sở dữ liệu của **MITRE ATT&CK**.
Các chiến dịch phân phối IcedID với phương thức tương tự (sử dụng tài liệu Office độc hại làm mồi nhử) thường được gán cho một nhóm tội phạm mạng có động cơ tài chính. Theo hệ thống danh pháp của Secureworks, nhóm phát tán IcedID này được theo dõi dưới bí danh **GOLD CABIN**.
![Pasted image 20260907131154.png](./images/Pasted%20image%2020260907131154.png)

![Pasted image 20260907131211.png](./images/Pasted%20image%2020260907131211.png)

>GOLD CABIN
## Q6: In the **Execution** phase, what function does the malware employ to fetch extra payloads onto the system?

**Yêu cầu:** Trong giai đoạn Execution, xác định hàm Windows API cụ thể mà malware sử dụng để tải các payload bổ sung xuống hệ thống.

Để trả lời câu hỏi này, chúng ta cần phân tích sâu hơn các bằng chứng trong giai đoạn "Execution" được ghi lại bởi hệ thống giám sát.

**Bước 1: Xác định bối cảnh Tấn công (Execution Phase)**

Kiểm tra bằng chứng giai đoạn "Execution" (`TA0002`) cho mẫu malware, chúng ta thấy một kỹ thuật cụ thể đã được kích hoạt: "Exploitation for Client Execution" (`T1203`).
![Pasted image 20260907151725.png](./images/Pasted%20image%2020260907151725.png)

Hình ảnh trên cho thấy kỹ thuật này có số "5" bên cạnh, cho biết có nhiều sự kiện liên quan. Điều này xác nhận rằng một khai thác tài liệu (trong trường hợp này là macro Excel độc hại) đã được sử dụng để thực thi mã.

**Bước 2: Phân tích chi tiết hàm được gọi (UrlDownloadToFileA)**

Đi sâu vào chi tiết của kỹ thuật `T1203` (như được hiển thị trong hình ảnh tiếp theo), chúng ta tìm thấy bằng chứng trực tiếp về hàm API được sử dụng để tải file.
![Pasted image 20260907151238.png](./images/Pasted%20image%2020260907151238.png)

Phân tích thẻ sự kiện chi tiết thứ hai, chúng ta thấy:
*   **Description (Mô tả):** `Document exploit detected (UrlDownloadToFileA call)`. (Phát hiện khai thác tài liệu (lần gọi UrlDownloadToFileA)).
*   **Match (Hàm):** `URLDownloadToFileA` (được in đậm).

Mô tả sự kiện xác nhận rằng việc khai thác tài liệu đã thực hiện một cuộc gọi đến hàm này. Tên hàm chính xác là `URLDownloadToFileA`. Hàm này (export từ `WININET.dll`, như được ghi chú) là một hàm Windows API phổ biến được sử dụng để tải một file từ một URL cụ thể và lưu nó vào một đường dẫn cục bộ chỉ với một cuộc gọi hàm, làm cho nó trở nên lý tưởng cho các cuộc tấn công living-off-the-land.

Đây chính là hàm được macro VBA trong file Excel (`document-1982481273.xlsm`) sử dụng để tải payload giai đoạn hai (`3003.gif`).

>URLDownloadToFileA