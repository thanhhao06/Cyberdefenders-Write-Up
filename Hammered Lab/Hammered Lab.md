author: Azaki
published: published
created: 2026-10-01
tags: CyberDefener, Lab, Threat Intel

This challenge takes you into virtual systems and confusing log data. In this challenge, as a SOC Analyst figure out what happened to this webserver honeypot using the logs from a possibly compromised server.

Thanks, [th3c0rt3x](https://cyberdefenders.org/accounts/profile/th3c0rt3x) for reviewing the challenge.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@The Honeynet Project</b> 
</p>

## Q1: Which service did the attackers use to gain access to the system?

**Yêu cầu:** Xác định dịch vụ mà kẻ tấn công đã sử dụng để truy cập vào hệ thống

Kiểm tra tệp nhật ký xác thực hệ thống `/var/log/auth.log`. Nhận thấy các bản ghi liên tục xuất phát từ tiến trình daemon `sshd`

Như vậy dịch vụ mà kẻ tấn công khai thác để xâm nhập vào hệ thống là `ssh`
![Pasted image 20261001135430.png](./images/Pasted%20image%2020261001135430.png)

>ssh

## Q2: What is the operating system version of the targeted system?

**Yêu cầu:** Xác định phiên bản hệ điều hành của hệ thống mục tiêu

Kiểm tra log khởi động của kernel trong `syslog` hoặc `dmesg`. Tại dòng thông báo phiên bản kernel:
```text
Linux version 2.6.24-26-server (buildd@crested) (gcc version 4.2.4 (Ubuntu 4.2.4-1ubuntu3)) #1 SMP ...
```

Phiên bản hệ điều hành được chỉ định chi tiết qua bản phát hành build là `4.2.4-1ubuntu3`
![Pasted image 20261001135526.png](./images/Pasted%20image%2020261001135526.png)

>4.2.4-1ubuntu3

## Q3: What is the name of the compromised account?

**Yêu cầu:** Xác định tên tài khoản bị xâm phạm

Quan sát các nỗ lực đăng nhập trong `auth.log`. Mặc dù kẻ tấn công thử nghiệm nhiều tên người dùng khác nhau (`cyrus`, `cyan`, `hailey`, `dannica`...), mục tiêu tài khoản hợp lệ bị brute-force và đăng nhập thành công là tài khoản `root`
![Pasted image 20261001135725.png](./images/Pasted%20image%2020261001135725.png)

>root

## Q4: How many attackers, represented by unique IP addresses, were able to successfully access the system after initial failed attempts?

**Yêu cầu:** Xác định số lượng kẻ tấn công (theo các IP duy nhất) đã truy cập thành công vào hệ thống sau khi đã có các lần thử đăng nhập thất bại trước đó

Để lọc chính xác các IP brute force thành công, ta cần tìm các IP vừa có bản ghi `Failed password` vừa có bản ghi `Accepted password`. Có thể sử dụng chuỗi lệnh sau:

```bash
grep "Accepted password" auth.log | grep -oP '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort -u > success.txt
```

```bash
grep "Failed password" auth.log | grep -oP '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c > fail_counts.txt
```

```bash
grep -f success.txt fail_counts.txt | sort -nr
```

Kết quả trả về danh sách gồm 6 địa chỉ IP có số lần brute force lớn rồi đăng nhập thành công
- `219.150.161.20` (9259 fails)
- `222.66.204.246` (1573 fails)
- `121.11.66.70` (1435 fails)
- `222.169.224.197` (646 fails)
- `122.226.202.12` (513 fails)
- `61.168.227.12` (213 fails)

![Pasted image 20261001130147.png](./images/Pasted%20image%2020261001130147.png)

>6

## Q5: Which attacker's IP address successfully logged into the system the most number of times?

**Yêu cầu:** Xác định IP của kẻ tấn công đã đăng nhập thành công vào hệ thống nhiều lần nhất

Từ kết quả thống kê các phiên đăng nhập thành công, địa chỉ IP `219.150.161.20` chiếm số lượng lớn nhất với hơn 9,200 lần thử sai và là kẻ tấn công tích cực nhất trên hệ thống
![Pasted image 20261001130227.png](./images/Pasted%20image%2020261001130227.png)

>219.150.161.20

## Q6: How many requests were sent to the Apache Server?

**Yêu cầu:** Xác định số lượng request đã được gửi tới máy chủ web Apache

Kiểm tra tệp nhật ký truy cập `www-access.log`. Mỗi dòng log đại diện cho một HTTP request gửi tới máy chủ web

Sử dụng lệnh `wc -l` để đếm tổng số dòng:
```bash
wc -l www-access.log
```

Kết quả cho ra 365 dòng, tương ứng 365 requests
![Pasted image 20261002155900.png](./images/Pasted%20image%2020261002155900.png)

>365

## Q7: How many rules have been added to the firewall?

**Yêu cầu:** Xác định số lượng quy tắc đã được thêm vào tường lửa

Lọc các câu lệnh liên quan đến `iptables` được thực thi trong `auth.log`:
```bash
grep "iptables" auth.log
```

Quan sát thấy người dùng root đã thực thi 6 lệnh thêm quy tắc vào chuỗi `INPUT` (`-A INPUT`)
- `iptables -A INPUT -p ssh -dport 2424 -j ACCEPT`
- `iptables -A INPUT -p tcp -dport 53 -j ACCEPT`
- `iptables -A INPUT -p udp -dport 53 -j ACCEPT`
- `iptables -A INPUT -p tcp --dport ssh -j ACCEPT`
- `iptables -A INPUT -p tcp --dport 53 -j ACCEPT`
- `iptables -A INPUT -p tcp --dport 113 -j ACCEPT`

Tổng cộng có 6 quy tắc đã được thêm vào tường lửa
![Pasted image 20261001132143.png](./images/Pasted%20image%2020261001132143.png)

>6

## Q8: One of the downloaded files on the target system is a scanning tool. What is the name of the tool?

**Yêu cầu:** Xác định tên của công cụ quét mạng được tải về và cài đặt trên máy mục tiêu

Trình quản lý gói Debian/Ubuntu ghi nhận toàn bộ lịch sử cài đặt phần mềm vào `/var/log/dpkg.log`

Kiểm tra `dpkg.log` vào ngày 24/04/2010, phát hiện bản ghi cài đặt công cụ quét cổng mạng:
```text
2010-04-24 19:38:16 status installed nmap 4.53-3
```

Tên công cụ là `nmap`
![Pasted image 20261001133814.png](./images/Pasted%20image%2020261001133814.png)
![Pasted image 20261001133831.png](./images/Pasted%20image%2020261001133831.png)

>nmap

## Q9: When was the last login from the attacker with IP 219.150.161.20?

**Yêu cầu:** Xác định thời gian lần đăng nhập cuối cùng của kẻ tấn công có IP 219.150.161.20

Lọc các sự kiện `Accepted password` của IP `219.150.161.20` trong `auth.log`:
```bash
grep "Accepted password" auth.log | grep "219.150.161.20"
```

Bản ghi cuối cùng được ghi nhận vào:
```text
Apr 19 05:56:05 app-1 sshd[13218]: Accepted password for root from 219.150.161.20 port 36585 ssh2
```

Định dạng thời gian theo chuẩn năm 2010 là `2010-04-19 05:56`
![Pasted image 20261001134326.png](./images/Pasted%20image%2020261001134326.png)

>2010-04-19 05:56

## Q10: The database showed two warning messages. Please provide the most critical and potentially dangerous one.

**Yêu cầu:** Cung cấp thông điệp cảnh báo nguy hiểm và nghiêm trọng nhất từ cơ sở dữ liệu

Kiểm tra các thông điệp cảnh báo dịch vụ trong `/var/log/daemon.log`:
```bash
grep "WARNING" daemon.log
```

Đầu ra hiển thị 2 thông báo:
- `WARNING: mysqlcheck has found corrupt tables`
- `WARNING: mysql.user contains 2 root accounts without password!`

Trong đó, việc tài khoản root MySQL không có mật khẩu là cảnh báo nghiêm trọng nhất về mặt bảo mật
![Pasted image 20261001134745.png](./images/Pasted%20image%2020261001134745.png)

>mysql.user contains 2 root accounts without password!

## Q11: Multiple accounts were created on the target system. Which account was created on April 26 at 04:43:15?

**Yêu cầu:** Xác định tài khoản nào được tạo trên hệ thống vào ngày 26 tháng 4 lúc 04:43:15

Tiến hành kiểm tra tiến trình tạo người dùng `useradd` được ghi nhận lại trong `auth.log`. Tìm kiếm theo mốc thời gian `04:43:15`:
```bash
grep "04:43:15" auth.log | grep "new user"
```

Kết quả hiển thị:
```text
Apr 26 04:43:15 app-1 useradd[20115]: new user: name=wind3str0y, UID=1004, GID=1005, home=/home/wind3str0y, shell=/bin/bash
```

Tên tài khoản mới được tạo là `wind3str0y`
![Pasted image 20261001134922.png](./images/Pasted%20image%2020261001134922.png)

>wind3str0y

## Q12: Few attackers were using a proxy to run their scans. What is the corresponding user-agent used by this proxy?

**Yêu cầu:** Xác định chuỗi User-Agent tương ứng được proxy sử dụng để quét hệ thống

Trích xuất các chuỗi User-Agent xuất hiện trong nhật ký truy cập `www-access.log`:
```bash
cat www-access.log | cut -d ' ' -f 12 | sort | uniq
```

Trong danh sách các User-Agent, phát hiện công cụ quét proxy chuyên dụng:
```text
"pxyscand/2.1"
```

User-Agent tương ứng là `pxyscand/2.1`
![Pasted image 20261001135056.png](./images/Pasted%20image%2020261001135056.png)

>pxyscand/2.1
