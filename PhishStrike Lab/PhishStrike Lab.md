author: Azaki
published: published
created: 2025-10-30
tags: CyberDefener, Lab, Threat Intel

As a cybersecurity analyst at an educational institution, you receive an alert about a phishing email targeting faculty members. The email appears to be from a trusted contact and claims a $625,000 purchase, providing a link to download an invoice.
Your task is to investigate the email using Threat Intel tools. Analyze the email headers and inspect the link for malicious content. Identify any Indicators of Compromise (IOCs) and document your findings to prevent potential fraud and educate faculty on phishing recognition.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefenders</b> 
</p>

## Q1: Identifying the sender's IP address with specific SPF and DKIM values helps trace the source of the phishing email. What is the sender's IP address that has an SPF value of softfail and a DKIM value of fail?

**Yêu cầu:** Xác định địa chỉ IP của máy chủ gửi email có kết quả:
```
SPF  = softfail
DKIM = fail
```

Đầu tiên, phân tích file email `.eml`. Có thể mở trực tiếp bằng text editor hoặc sử dụng `strings` để đọc phần header của email.

Trong header, tập trung vào trường `Authentication-Results`. **SPF** được sử dụng để kiểm tra máy chủ gửi có được phép gửi email thay mặt domain hay không, trong khi **DKIM** kiểm tra chữ ký số nhằm xác minh tính toàn vẹn và nguồn gốc của nội dung email.

Ở đây, kết quả `spf=softfail` cho thấy máy chủ gửi không nằm trong danh sách được domain cho phép rõ ràng, còn `dkim=fail` cho thấy việc xác minh chữ ký DKIM thất bại. Đây là hai dấu hiệu đáng ngờ thường xuất hiện trong các email giả mạo.
![Pasted image 20260815100930.png](./images/Pasted%20image%2020260815100930.png)

>18.208.22.104

## Q2: Understanding the return path of an email is essential for tracing its origin. What is the return path specified in this email?

**Yêu cầu:** Xác định địa chỉ `Return-Path` được khai báo trong email.

Tiếp tục kiểm tra phần header của file `.eml` và tìm trường `Return-Path:`. Trường này chỉ định địa chỉ nhận các thông báo lỗi hoặc email bounce khi thư không thể được chuyển đến người nhận.

Trong quá trình điều tra phishing, `Return-Path` cũng là một artifact hữu ích để so sánh với trường `From:`. Nếu hai địa chỉ hoặc domain không liên quan với nhau, đây có thể là dấu hiệu giả mạo nguồn gửi.
![Pasted image 20260815100934.png](./images/Pasted%20image%2020260815100934.png)

>erikajohana.lopez@uptc.edu.co

## Q3: Identifying the source of malware is critical for effective threat mitigation and response. What is the IP address of the server hosting the malicious file related to malware distribution?

**Yêu cầu:** Xác định địa chỉ IP của máy chủ đang host file độc hại được phát tán qua email phishing.

Sau khi kiểm tra header, tiếp tục xem phần body của email. Nội dung thư giả mạo một hóa đơn liên quan đến giao dịch trị giá lớn và dụ người nhận tải file từ một liên kết bên ngoài.

Từ URL được nhúng trong email có thể thấy payload `install.exe` được tải trực tiếp từ một địa chỉ IP thay vì một domain thông thường:
```text
http://107.175.247.199/loader/install.exe
```

Việc một email tự nhận là hóa đơn nhưng lại dẫn trực tiếp tới file `.exe` là dấu hiệu rất đáng ngờ và cho thấy liên kết có khả năng được dùng để phát tán malware.
![Pasted image 20260815100938.png](./images/Pasted%20image%2020260815100938.png)

>107.175.247.199

## Q4: Identifying malware that exploits system resources for cryptocurrency mining is critical for prioritizing threat mitigation efforts. The malicious URL can deliver several malware types. Which malware family is responsible for cryptocurrency mining?

**Yêu cầu:** Xác định malware family có chức năng đào tiền mã hóa được phân phối từ hạ tầng độc hại đã tìm thấy.

Sử dụng IOC `107.175.247.199` để pivot sang các nguồn Threat Intelligence như **URLHaus**. Khi tra cứu IP này, có thể thấy cùng hạ tầng từng phân phối nhiều payload khác nhau.

Trong URLHaus, kiểm tra các payload/signature liên quan đến URL độc hại. Một trong các mẫu được gắn nhãn là malware chuyên sử dụng tài nguyên hệ thống để đào cryptocurrency.
![Pasted image 20260815100943.png](./images/Pasted%20image%2020260815100943.png)

>CoinMiner

## Q5: Identifying the specific URLs malware requests is key to disrupting its communication channels and reducing its impact. Based on the previous analysis of the cryptocurrency malware sample, what does this malware request the URL?

**Yêu cầu:** Xác định URL được mẫu **CoinMiner** request trong quá trình thực thi.

Từ kết quả URLHaus ở câu trước, lấy SHA-256 của mẫu được gắn nhãn **CoinMiner**:
```text
453fb1c4b3b48361fa8a67dcedf1eaec39449cb5a146a7770c63d1dc0d7562f0
```

![Pasted image 20260815100946.png](./images/Pasted%20image%2020260815100946.png)
Do không cần thực thi trực tiếp malware trên máy thật, sử dụng hash này để pivot sang **VirusTotal**. Trong phần hành vi mạng, kiểm tra các **HTTP requests** mà mẫu malware thực hiện.

Từ kết quả phân tích, CoinMiner thực hiện request tới một tài nguyên có phần mở rộng `.jpeg`. Tuy nhiên trong ngữ cảnh malware, phần mở rộng file không đảm bảo nội dung thực sự là hình ảnh và có thể được sử dụng để ngụy trang payload
![Pasted image 20260815100951.png](./images/Pasted%20image%2020260815100951.png)

>http://ripley.studio/loader/uploads/Qanjttrbv.jpeg

## Q6: Understanding the registry entries added to the auto-run key by malware is crucial for identifying its persistence mechanisms. Based on the BitRAT malware sample analysis, what is the executable's name in the first value added to the registry auto-run key?

**Yêu cầu:** Xác định tên executable được **BitRAT** thêm vào Registry Run key để duy trì persistence.

Quay lại các payload được phân phối từ hạ tầng độc hại và chọn mẫu được gắn nhãn **BitRAT**. SHA-256 của mẫu liên quan là:
```text
bf7628695c2df7a3020034a065397592a1f8850e59f9a448b555bc1c8c639539
```

![Pasted image 20260815100955.png](./images/Pasted%20image%2020260815100955.png)

Tra cứu hash trên VirusTotal, sau đó kiểm tra phần **Registry actions / Registry Keys Set** để xác định các khóa Registry được malware tạo hoặc chỉnh sửa.
![Pasted image 20260815101000.png](./images/Pasted%20image%2020260815101000.png)
Run key này khiến chương trình được thực thi khi người dùng đăng nhập Windows, vì vậy đây là một cơ chế persistence phổ biến.

>Jzwvix.exe

## Q7: Identifying the SHA-256 hash of files downloaded from a malicious URL is essential for tracking and analyzing malware activity. Based on the BitRAT analysis, what is the SHA-256 hash of the file previously downloaded and added to the autorun keys?

**Yêu cầu:** Xác định SHA-256 của file BitRAT đã được tải xuống và thêm vào Registry autorun key.

Từ câu trước, file được thêm vào Run key là `Jzwvix.exe`. Quay lại kết quả phân tích mẫu hoặc phần **Dropped Files / Relations** trên VirusTotal để xác định hash tương ứng của file này.
![Pasted image 20260815101003.png](./images/Pasted%20image%2020260815101003.png)
Hash này có thể tiếp tục được sử dụng làm IOC để tìm kiếm trên các nền tảng Threat Intelligence, EDR

>bf7628695c2df7a3020034a065397592a1f8850e59f9a448b555bc1c8c639539

## Q8: Analyzing the HTTP requests made by malware helps in identifying its communication patterns. What is the URL in the HTTP request used by the loader to retrieve the BitRAT malware?

**Yêu cầu:** Xác định URL mà loader sử dụng để tải payload **BitRAT**.

Tiếp tục phân tích mẫu BitRAT bằng SHA-256 đã xác định ở trên. Trong phần **Network / HTTP requests**, tìm request tải một executable từ hạ tầng `107.175.247.199`
![Pasted image 20260815101007.png](./images/Pasted%20image%2020260815101007.png)
Đây là một IOC quan trọng vì nó mô tả trực tiếp đường dẫn được sử dụng trong chuỗi tải malware

>http://107.175.247.199/loader/server.exe

## Q9: Introducing a delay in malware execution can help evade detection mechanisms. What is the delay (in seconds) caused by the PowerShell command according to the BitRAT analysis?

**Yêu cầu:** Xác định thời gian delay được PowerShell tạo ra trong quá trình thực thi BitRAT.

Trong process tree hoặc phần **Shell commands** của báo cáo phân tích, có thể thấy malware gọi `powershell.exe` với tham số `-enc`, cho biết command đã được encode bằng Base64
![Pasted image 20260815101011.png](./images/Pasted%20image%2020260815101011.png)

Decode chuỗi Base64 bằng **CyberChef** hoặc Terminal, sau đó loại bỏ các null byte do chuỗi được mã hóa theo UTF-16LE. Nội dung thu được tương ứng với lệnh
![image](https://hackmd.io/_uploads/HkXzJBeyZe.png)

>50

## Q10: Tracking the command and control (C2) domains used by malware is essential for detecting and blocking malicious activities. What is the C2 domain used by the BitRAT malware?

**Yêu cầu:** Xác định domain Command and Control (**C2**) được mẫu BitRAT sử dụng.

Sử dụng SHA-256 của BitRAT để tra cứu trên **VirusTotal** và kiểm tra các community report, sandbox report hoặc IOC liên quan tới mẫu
![Pasted image 20260815101015.png](./images/Pasted%20image%2020260815101015.png)
Trong các kết quả phân tích có thể xác định domain mà malware sử dụng để liên lạc với hạ tầng điều khiển từ xa
Domain này nên được đưa vào danh sách IOC để phục vụ hunting và chặn kết nối outbound trong môi trường bị ảnh hưởng

>gh9st.mywire.org

## Q11: Understanding how malware exfiltrates data is essential for detecting and preventing data breaches. According to the AsyncRAT analysis, what is the Telegram Bot ID used by this malware?

**Yêu cầu:** Xác định Telegram Bot ID được mẫu **AsyncRAT** sử dụng trong hoạt động liên quan đến exfiltration/C2.

Quay lại danh sách payload được phân phối từ hạ tầng ban đầu và chọn mẫu được gắn nhãn **AsyncRAT**. SHA-256 của mẫu là
```text
5ca468704e7ccb8e1b37c0f7595c54df4e2f4035345b6e442e8bd4e11c58f791
```

Sử dụng hash để tra cứu trên **tria.ge** và mở behavioral report của mẫu
![Pasted image 20260815101019.png](./images/Pasted%20image%2020260815101019.png)

Trong phần network activity, có thể thấy các request hướng tới Telegram Bot API. Bot ID nằm ngay trong đường dẫn request, sau tiền tố `bot`
![Pasted image 20260815101024.png](./images/Pasted%20image%2020260815101024.png)
Vì vậy tìm được Telegram Bot ID được sử dụng
>bot5610920260

---

# ATTACK CHAIN

![PhishStrike Lab.png](./images/PhishStrike%20Lab.png)