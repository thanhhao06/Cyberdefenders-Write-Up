author: Azaki
published: published
created: 2025-01-09
tags: CyberDefener, Lab, Threat Intel

A decentralized finance (DeFi) platform recently reported multiple user complaints about unauthorized fund withdrawals. A forensic review uncovered a phishing site impersonating the legitimate PancakeSwap exchange, luring victims into entering their wallet seed phrases. The phishing kit was hosted on a compromised server and exfiltrated credentials via a Telegram bot.

Your task is to conduct threat intelligence analysis on the phishing infrastructure, identify indicators of compromise (IoCs), and track the attacker’s online presence, including aliases and Telegram identifiers, to understand their tactics, techniques, and procedures (TTPs).

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@milannshrestga</b> 
</p>

## Q1: Which wallet is used for asking the seed phrase?

**Yêu cầu:** Xác định loại ví điện tử (wallet) mà trang phishing này nhắm tới để đánh cắp cụm từ khôi phục (seed phrase).

Bắt đầu bằng việc phân tích mã nguồn và giao diện tĩnh của trang lừa đảo. Kiểm tra file `index.html` ở thư mục gốc, trang web hiển thị một danh sách rất nhiều loại ví điện tử khác nhau nhằm dụ dỗ nạn nhân
![GrabThePhisher Lab.png](./images/GrabThePhisher%20Lab.png)

Tuy nhiên, khi đi sâu vào cấu trúc thư mục mã nguồn, chúng ta phát hiện một thư mục con mang tên `metamask` chứa một file `index.html` khác
![GrabThePhisher Lab-1.png](./images/GrabThePhisher%20Lab-1.png)

Mở file HTML này lên để xem giao diện, form yêu cầu người dùng nhập "keyword phrase of 12 words" (cụm từ hạt giống gồm 12 từ) được thiết kế giống hệt với giao diện xác thực của Metamask. Điều này xác nhận mục tiêu chính của phishing kit này là ví Metamask![GrabThePhisher Lab-2.png](./images/GrabThePhisher%20Lab-2.png)

>Metamask

## Q2: What is the file name that has the code for the phishing kit?

**Yêu cầu:** Xác định tên file chứa mã nguồn (logic code) của phishing kit.

Trong cùng thư mục `metamask` vừa phân tích, ngoài file giao diện HTML, còn có một tập tin mã nguồn backend chịu trách nhiệm xử lý dữ liệu
![GrabThePhisher Lab-3.png](./images/GrabThePhisher%20Lab-3.png)

Phân tích tĩnh nội dung file này cho thấy nó đảm nhiệm các hành vi cốt lõi của một phishing kit:
- **Thu thập thông tin:** Tiếp nhận cụm từ khôi phục (seed phrase) do nạn nhân nhập vào thông qua phương thức POST. Đồng thời, thu thập địa chỉ IP, vị trí địa lý (thông qua một API bên thứ ba) và thông tin thiết bị (User-Agent).
- **Đánh cắp dữ liệu (Exfiltration):** Gửi toàn bộ dữ liệu nhạy cảm thu thập được trực tiếp đến tài khoản của kẻ tấn công.
- **Lưu trữ cục bộ:** Ghi lại seed phrase vừa đánh cắp vào một tệp tin văn bản (file log) trên chính máy chủ lưu trữ.

Tên của file xử lý toàn bộ logic độc hại này là `metamask.php`


>metamask.php

## Q3: In which language was the kit written?

**Yêu cầu:** Ngôn ngữ lập trình nào được sử dụng để viết bộ mã độc lừa đảo (phishing kit) này?

Dựa vào phần mở rộng của tập tin đã xác định ở Q2 (`.php`), ta có thể dễ dàng nhận biết ngôn ngữ lập trình. Để chắc chắn hơn về định dạng tệp tin ở cấp độ hệ thống, việc sử dụng lệnh `file metamask.php` trong terminal trả về kết quả là "PHP script, ASCII text"
![GrabThePhisher Lab-4.png](./images/GrabThePhisher%20Lab-4.png)

>PHP

## Q4: What service does the kit use to retrieve the victim's machine information?

**Yêu cầu:** Bộ mã lừa đảo sử dụng dịch vụ nào để trích xuất thông tin máy nạn nhân (ví dụ: vị trí địa lý dựa trên IP)?

Phân tích chi tiết mã nguồn của tệp `metamask.php`. Ngay ở những dòng đầu tiên, mã độc sử dụng hàm `file_get_contents()` để gọi đến một API ngoại vi nhằm truy vấn dữ liệu định vị địa lý (Geolocation) dựa trên địa chỉ IP của nạn nhân (`$_SERVER['REMOTE_ADDR']`)
![GrabThePhisher Lab-5.png](./images/GrabThePhisher%20Lab-5.png)

Đường dẫn URL được gọi đến là `http://api.sypexgeo.net/json/`. Từ đó suy ra, kẻ tấn công đang sử dụng dịch vụ của Sypex Geo

>Sypex Geo

## Q5: How many seed phrases were already collected?

**Yêu cầu:** Có bao nhiêu cụm từ khôi phục (seed phrases) đã bị thu thập (đánh cắp) và lưu lại?

Kiểm tra phần logic lưu trữ cục bộ ở nửa cuối file `metamask.php`, ta thấy mã độc gọi hàm `file_put_contents()` để nối (`FILE_APPEND`) dữ liệu thu thập được vào một file tên là `log.txt` nằm trong thư mục `/log/`
![GrabThePhisher Lab-6.png](./images/GrabThePhisher%20Lab-6.png)

Mở file `log.txt` này ra để kiểm tra, nội dung bên trong chứa các đoạn văn bản lưu trữ các cụm 12 từ. Đếm số dòng trong file, ta thấy có tổng cộng 3 chuỗi tương ứng với 3 cụm từ khôi phục ví của các nạn nhân khác nhau
![GrabThePhisher Lab-7.png](./images/GrabThePhisher%20Lab-7.png)

>3

## Q6: Could you please provide the seed phrase associated with the most recent phishing incident?

**Yêu cầu:** Cung cấp cụm từ khôi phục (seed phrase) của sự cố lừa đảo gần đây nhất.

Do mã độc sử dụng cờ `FILE_APPEND` khi ghi file `log.txt`, dữ liệu mới nhất sẽ luôn được ghi vào cuối cùng của tệp tin. Đọc dòng cuối cùng của file log này, ta sẽ thu được cụm từ khôi phục của nạn nhân bị tấn công gần nhất
![GrabThePhisher Lab-8.png](./images/GrabThePhisher%20Lab-8.png)

>father also recycle embody balance concert mechanic believe owner pair muffin hockey

## Q7: Which medium was used for credential dumping?

**Yêu cầu:** Phương tiện (medium) nào được sử dụng để tuồn thông tin xác thực (credential dumping) ra ngoài?

Kiểm tra hàm `sendTel()` trong tệp `metamask.php`, đây là hàm chịu trách nhiệm gửi dữ liệu ra bên ngoài (exfiltration). Mã nguồn cho thấy kẻ tấn công đã xây dựng một URL truy vấn đến API `https://api.telegram.org/bot`. Chúng sử dụng một con Bot để tự động đẩy tin nhắn chứa dữ liệu đánh cắp được vào một kênh chat cụ thể
![GrabThePhisher Lab-9.png](./images/GrabThePhisher%20Lab-9.png)

>Telegram

## Q8: What is the token for accessing the channel?

**Yêu cầu:** Token được sử dụng để truy cập kênh bot là gì?

Ngay bên trong hàm `sendTel()`, kẻ tấn công đã hardcode (nhúng trực tiếp) các thông tin xác thực cần thiết để gọi Telegram Bot API. Biến `$token` chứa giá trị khóa API của con bot lừa đảo
![GrabThePhisher Lab-10.png](./images/GrabThePhisher%20Lab-10.png)

> 5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10

## Q9: What is the Chat ID for the phisher's channel?

**Yêu cầu:** Chat ID của kênh mà kẻ tấn công sử dụng để nhận dữ liệu là gì?

Tương tự như Token, ID của đoạn chat/kênh nhận dữ liệu cũng được định nghĩa rõ ràng trong biến `$id` thuộc cùng khối mã lệnh
![GrabThePhisher Lab-11.png](./images/GrabThePhisher%20Lab-11.png)

>5442785564

## Q10: What are the allies of the phish kit developer?

**Yêu cầu:** Bí danh (alias) của nhà phát triển bộ mã lừa đảo (phish kit developer) này là gì?

Trong phân tích phần mềm độc hại, việc tìm kiếm các thông tin do chính tác giả để lại trong mã nguồn (như comments, metadata) là một phương pháp tình báo quan trọng. Đọc kỹ toàn bộ mã nguồn của file `metamask.php`, ở đoạn giữa file có một khối bình luận (comment block) mà tác giả dùng để gửi lời chào. Tại phần chữ ký kết thúc ("Regards,"), tác giả đã để lại bí danh trực tuyến của mình
![GrabThePhisher Lab-12.png](./images/GrabThePhisher%20Lab-12.png)

>j1j1b1s@m3r0