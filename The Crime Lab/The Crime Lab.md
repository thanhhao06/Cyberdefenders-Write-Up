author: Azaki
published: published
created: 2025-09-08
tags: CyberDefener, Lab, Endpoint Forensics

We're currently in the midst of a murder investigation, and we've obtained the victim's phone as a key piece of evidence. After conducting interviews with witnesses and those in the victim's inner circle, your objective is to meticulously analyze the information we've gathered and diligently trace the evidence to piece together the sequence of events leading up to the incident.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Infern0o</b> 
</p>

## Q1: Based on the accounts of the witnesses and individuals close to the victim, it has become clear that the victim was interested in trading. This has led him to invest all of his money and acquire debt. Can you identify the `SHA256` of the trading application the victim primarily used on his phone?

**Yêu cầu:** Xác định mã băm SHA256 của ứng dụng giao dịch chứng khoán/trading mà nạn nhân thường sử dụng.

Để trích xuất thông tin về các ứng dụng đã cài đặt trên thiết bị Android, chúng ta cần phân tích cơ sở dữ liệu `gass.db` thông qua module `installedappsGass` trên công cụ ALEAPP. Lọc danh sách các ứng dụng được cài đặt bằng từ khóa "trade", ta tìm thấy một package có tên `com.ticno.olymptrade`. Bảng dữ liệu này cũng cung cấp sẵn mã băm định danh SHA-256 của ứng dụng
![Pasted image 20260908222612.png](./images/Pasted%20image%2020260908222612.png)

>4f168a772350f283a1c49e78c1548d7c2c6c05106d8b9feb825fdc3466e9df3c

## Q2: According to the testimony of the victim's best friend, he said, "`While we were together, my friend got several calls he avoided. He said he owed the caller a lot of money but couldn't repay now`". How much does the victim owe this person?

**Yêu cầu:** Theo lời khai, nạn nhân đang nợ một khoản tiền lớn. Hãy xác định chính xác số tiền mà nạn nhân đang nợ.

Tiến hành kiểm tra dữ liệu tin nhắn SMS của nạn nhân được lưu trữ trong cơ sở dữ liệu `mmssms.db`. Bằng cách đọc nội dung các tin nhắn đến (Received), ta phát hiện một tin nhắn đe dọa đòi nợ được gửi vào ngày 20-09-2023. Tin nhắn này ghi rõ số tiền nạn nhân cần phải chuẩn bị trả là 250,000 EGP
![Pasted image 20260908221625.png](./images/Pasted%20image%2020260908221625.png)

>250000

## Q3: What is the name of the person to whom the victim owes money?

**Yêu cầu:** Xác định tên của người chủ nợ.

Từ tin nhắn đòi nợ ở câu hỏi trước, ta thu thập được số điện thoại của người gửi là `+201172137258`. 
![Pasted image 20260908221523.png](./images/Pasted%20image%2020260908221523.png)

Để biết số điện thoại này thuộc về ai, ta thực hiện đối chiếu chéo (cross-reference) với cơ sở dữ liệu danh bạ `contacts2.db` của điện thoại. Kết quả tra cứu số điện thoại này trong danh bạ hiển thị tên người dùng (Display Name) tương ứng là Shady Wahab
![Pasted image 20260908221547.png](./images/Pasted%20image%2020260908221547.png)

>Shady Wahab

## Q4: Based on the statement from the victim's family, they said that on `September 20, 2023`, he departed from his residence without informing anyone of his destination. Where was the victim located at that moment?

**Yêu cầu:** Nạn nhân đã rời khỏi nhà vào ngày 20/09/2023. Xác định vị trí của nạn nhân vào thời điểm đó.

Kiểm tra lịch sử hoạt động (Recent Activity) của nạn nhân trong ngày 20/09/2023. Dữ liệu cho thấy vào lúc 23:50, nạn nhân có sử dụng hai ứng dụng là Discord và Google Maps
![Pasted image 20260908224349.png](./images/Pasted%20image%2020260908224349.png)

Dựa vào manh mối này, ta trích xuất các tệp tin ảnh chụp màn hình (snapshots) được hệ thống lưu lại tự động khi người dùng chuyển đổi ứng dụng
![Pasted image 20260908224423.png](./images/Pasted%20image%2020260908224423.png)

Trong thư mục snapshots, ta tìm thấy một ảnh chụp màn hình giao diện Google Maps hiển thị vị trí hiện tại của nạn nhân (dấu chấm xanh) đang đứng tại khách sạn The Nile Ritz-Carlton, Cairo
![Pasted image 20260908223303.png](./images/Pasted%20image%2020260908223303.png)


>The Nile Ritz-Carlton

## Q5: The detective continued his investigation by questioning the hotel lobby. She informed him that the victim had reserved the room for 10 days and had a flight scheduled thereafter. The investigator believes that the victim may have stored his ticket information on his phone. Look for where the victim intended to travel.

**Yêu cầu:** Lễ tân khách sạn cho biết nạn nhân có một chuyến bay sau 10 ngày lưu trú. Tìm điểm đến dự định của nạn nhân dựa trên thông tin vé máy bay được lưu trong điện thoại.

Phân tích bộ nhớ cache của ứng dụng Google Photos (`disk_cache`), ta khôi phục được một số hình ảnh mà nạn nhân đã xem hoặc lưu trữ, trong đó có hình ảnh của một chiếc vé máy bay (boarding pass)
![Pasted image 20260908223818.png](./images/Pasted%20image%2020260908223818.png)

Trích xuất hình ảnh vé máy bay này và phóng to để đọc các chi tiết, ta thấy thông tin hành khách là MOHAMED AHMED, bay từ Cairo (Ai Cập) và điểm đến (To) là Las Vegas
![Pasted image 20260908223527.png](./images/Pasted%20image%2020260908223527.png)

>Las Vegas

## Q6: After examining the victim's Discord conversations, we discovered he had arranged to meet a friend at a specific location. Can you determine where this meeting was supposed to occur?

**Yêu cầu:** Qua việc kiểm tra các đoạn hội thoại Discord, xác định địa điểm mà nạn nhân đã hẹn gặp một người bạn.

Từ manh mối ở Q4, chúng ta biết nạn nhân đã sử dụng ứng dụng Discord cùng lúc với Google Maps. Bằng cách phân tích các tin nhắn Discord được lưu trong bộ nhớ đệm (Discord - Cached Messages), ta tìm thấy đoạn hội thoại lịch sử chứa thông tin trao đổi giữa nạn nhân và người bạn. Nội dung tin nhắn đề cập rõ ràng đến địa điểm mà nạn nhân muốn đến và hẹn gặp mặt
![Pasted image 20260908223902.png](./images/Pasted%20image%2020260908223902.png)

>The Mob Museum