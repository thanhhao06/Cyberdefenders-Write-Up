author: Azaki
published: published
created: 2025-09-13
tags: CyberDefener, Lab, Endpoint Forensics

On May 21, 2021, an intelligence agency intercepted a mobile device suspected of covert operations. The forensic team performed a full disk dump, extracting databases, logs, and application activity. Findings suggest encrypted communications, anonymous browsing, and unauthorized data transfers. Analyze extracted data to determine suspect activities, network connections, and security risks while establishing a timeline of events.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@DFIRScience</b> 
</p>

## Q1: What is the email address of `Zoe Washburne`?


Kiểm tra thư mục agent data của người dùng để lấy file database của phần contacts
![LGDroid Lab.png](./images/LGDroid%20Lab.png)

Sau đó kiểm tra file data base và tìm đến tên người dùng được yêu cầu để lấy mail
![LGDroid Lab-1.png](./images/LGDroid%20Lab-1.png)

>zoewash@0x42.null

## Q2: What was the device's DateTime in UTC at the time of acquisition?

Vì yêu cầu là thời gian tại lúc được ghi nhận nên lúc này mình không tìm kiếm ở agent nữa mà xem các dữ liệu được ghi  trực tiếp lúc kiểm tra trong thư mục này mình thấy có phần thời gian của thiết bị
![LGDroid Lab-2.png](./images/LGDroid%20Lab-2.png)

Kiểm tra file để lấy được thời gian cần tìm
![LGDroid Lab-3.png](./images/LGDroid%20Lab-3.png)

>2021-05-21 18:17

## Q3: What time was the `Tor Browser` downloaded in UTC?

Vì lần cần tìm thời gian tải xuống của browser nên mình cần tìm database của thư mục Downloads để tìm có file này tải xuống không
![LGDroid Lab-4.png](./images/LGDroid%20Lab-4.png)

Kiểm tra file database này thì mình thấy lần gần đây nhất đã có file tor được tải xuống và trong này có cả timestame lúc tải xuống
![LGDroid Lab-5.png](./images/LGDroid%20Lab-5.png)

Mình tiến hành chuyển đổi sang ngày tháng cụ thể
![LGDroid Lab-6.png](./images/LGDroid%20Lab-6.png)

>2021-04-29 19:42

## Q4: At what time did the phone reach a 100% charge after the last reset?

Tìm thời điểm điện thoại sạc đầy 100% pin vì đây là dữ liệu động nên mình cần kiểm tra trong thư mục live data tiếp đến là vào dumpspy để xem nhiều dữ liệu hơn thay vì các thông tin cơ bản
![LGDroid Lab-7.png](./images/LGDroid%20Lab-7.png)

Tại đây mình thấy có file ghi thông tin về pin nên kiểm tra và tim theo từ khóa **full charge** và ghi nhận được 1 lần sau khi hệ thống reset
![LGDroid Lab-8.png](./images/LGDroid%20Lab-8.png)

Nhưng hiện chưa có thời gian cụ thể nên mình cần quay lại phần reset hệ thống để lấy thông tin tại lúc đó
![LGDroid Lab-9.png](./images/LGDroid%20Lab-9.png)

Và vì đã có thời gian là 2021-05-21 13:12:19 sau đó mất tới 5 phút 1 giây mới sạc đầy nên tính thêm khoản thời gian này vào thì sẽ được thời gian lúc sạc được 100%

>2021-05-21 13:17

## Q5: What is the password for the most recently connected WIFI access point?

Kiểm
![LGDroid Lab-10.png](./images/LGDroid%20Lab-10.png)

Kiểm tra file và tìm đến presharekey 
![LGDroid Lab-11.png](./images/LGDroid%20Lab-11.png)

>ThinkingForest!

## Q6: What app was the user focused on at `2021-05-20 14:13:27`?

![LGDroid Lab-12.png](./images/LGDroid%20Lab-12.png)

![LGDroid Lab-13.png](./images/LGDroid%20Lab-13.png)

>youtube




## Q7: How long did the suspect watch YouTube on `2021-05-20`?

![LGDroid Lab-14.png](./images/LGDroid%20Lab-14.png)


![LGDroid Lab-15.png](./images/LGDroid%20Lab-15.png)

14:13:27 to 22:47:57

>08:34



## Q8: What is the structural similarity metric for the image "`suspicious.jpg"` compared to a visually similar image taken with a mobile phone?


![LGDroid Lab-16.png](./images/LGDroid%20Lab-16.png)

>0.99