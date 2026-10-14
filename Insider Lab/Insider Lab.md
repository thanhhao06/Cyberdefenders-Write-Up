author: Azaki
published: published
created: 2026-09-08
tags: CyberDefener, Lab, Endpoint Forensic

Sau khi Karen bắt đầu làm việc cho 'TAAUSAI', cô ta bắt đầu thực hiện các hoạt động bất hợp pháp trong công ty. 'TAAUSAI' đã thuê bạn làm chuyên viên phân tích an ninh mạng để khởi động cuộc điều tra về vụ việc này.

Bạn đã thu được ảnh đĩa và phát hiện Karen sử dụng hệ điều hành Linux trên máy tính của mình. Hãy phân tích ảnh đĩa máy tính của Karen và trả lời các câu hỏi được cung cấp.

---



## Q1: Which Linux distribution is being used on this machine?

  
**Yêu cầu:** Xác định bản phân phối Linux (Linux distribution) đang được sử dụng trên máy tính này.

Bắt đầu bằng việc kiểm tra thư mục hệ thống quan trọng là `/boot`. Thư mục này thường chứa các tập tin cần thiết để khởi động hệ điều hành, bao gồm kernel và cấu hình bootloader. Phân tích các file cấu hình và kernel image trong `/boot`, chúng ta phát hiện hậu tố `-kali1-amd64` được đính kèm trong tên của chúng. Dấu hiệu này chỉ ra rõ ràng rằng hệ điều hành là một bản phân phối của Kali Linux

![Pasted image 20260908182918.png](./images/Pasted%20image%2020260908182918.png)

>kali

## Q2: What is the MD5 hash of the Apache **access.log** file?

**Yêu cầu:** Tính giá trị mã băm MD5 của tập tin `access.log` của dịch vụ Apache.

Trước tiên, cần xác định vị trí của file log. Theo cấu trúc thư mục mặc định của các hệ thống dựa trên Debian/Ubuntu (như Kali), file log của Apache2 thường nằm tại `/var/log/apache2/`. Truy cập vào đường dẫn này, chúng ta tìm thấy file `access.log`. Điểm đáng chú ý là kích thước của file này là 0 bytes (file rỗng)
![Pasted image 20260908183245.png](./images/Pasted%20image%2020260908183245.png)

Vì đề bài yêu cầu cụ thể giá trị hash của file này, chúng ta tiến hành tính toán MD5. Do file hoàn toàn trống, mã băm thu được chính là MD5 Null Hash tiêu chuẩn
![Pasted image 20260908183328.png](./images/Pasted%20image%2020260908183328.png)

>d41d8cd98f00b204e9800998ecf8427e

## Q3: It is suspected that a credential dumping tool was downloaded. What is the name of the downloaded file?

**Yêu cầu:** Tìm tên của công cụ đánh cắp thông tin đăng nhập (credential dumping tool) bị nghi ngờ đã được tải xuống máy.

Để tìm kiếm các tập tin được người dùng tải về, chúng ta kiểm tra thư mục `/root/Downloads`. Tại đây, một tập tin nén định dạng ZIP đáng ngờ xuất hiện. Tên của file này chỉ đích danh "Mimikatz", một trong những công cụ phổ biến và khét tiếng nhất được dùng để trích xuất mật khẩu dạng plaintext, mã băm (hash), mã PIN và vé Kerberos từ bộ nhớ của Windows
![Pasted image 20260908201112.png](./images/Pasted%20image%2020260908201112.png)

>mimikatz_trunk.zip

## Q4: A super-secret file was created. What is the absolute path to this file?

**Yêu cầu:** Xác định đường dẫn tuyệt đối (absolute path) của một tập tin "siêu bí mật" (super-secret file) đã được tạo ra.

Để theo dõi các hành động của người dùng, phân tích lịch sử dòng lệnh (bash history) là một phương pháp hiệu quả. Bằng cách kiểm tra file `.bash_history` của user `root`, chúng ta phát hiện một chuỗi các lệnh đáng ngờ. Cụ thể, người dùng đã thực hiện điều hướng thư mục và sử dụng các lệnh như `touch` và `cat` kết hợp với toán tử chuyển hướng (redirection `>`) để tạo và ghi dữ liệu vào một file mới. Lệnh này đã ghi rõ đường dẫn tuyệt đối đến file "SuperSecretFile.txt"
![Pasted image 20260908202530.png](./images/Pasted%20image%2020260908202530.png)

>/root/Desktop/SuperSecretFile.txt

## Q5: What program used the file **didyouthinkwedmakeiteasy.jpg** during its execution?

**Yêu cầu:** Cho biết chương trình nào đã sử dụng file ảnh `didyouthinkwedmakeiteasy.jpg` trong quá trình thực thi.

Tiếp tục khai thác thông tin từ file `.bash_history` đã được mở ở câu hỏi trước. Phân tích các dòng lệnh tiếp theo, chúng ta nhận thấy file hình ảnh có tên `didyouthinkwedmakeiteasy.jpg` được truyền vào như một đối số (argument) cho một lệnh cụ thể. Công cụ được gọi đến là một phần mềm phân tích mã nhị phân tĩnh (firmware analysis tool) thường được dùng để trích xuất file ẩn hoặc mã nhúng bên trong các tập tin khác (kỹ thuật Steganography)
![Pasted image 20260908202610.png](./images/Pasted%20image%2020260908202610.png)

>binwalk

## Q6: What is the third goal from the checklist Karen created?

**Yêu cầu:** Xác định mục tiêu thứ ba trong danh sách kiểm tra (checklist) do Karen tạo.

Kiểm tra nội dung của thư mục `/root/Desktop`, ta phát hiện một tập tin văn bản thuần túy (plaintext) có tên `Checklist`. Khi đọc nội dung của file này, nó hiển thị một danh sách gồm 3 mục tiêu. Theo thứ tự từ trên xuống, mục tiêu ở dòng thứ ba chính là câu trả lời
![Pasted image 20260908202716.png](./images/Pasted%20image%2020260908202716.png)

>profit

## Q7: How many times was Apache run?

**Yêu cầu:** Dịch vụ web Apache đã được chạy bao nhiêu lần?

Quay trở lại thư mục log của Apache tại `/var/log/apache2/` mà chúng ta đã phân tích ở Q2. Tại đây có chứa các file log tiêu chuẩn như `access.log`, `error.log`, và `other_vhosts_access.log`. Nếu dịch vụ Apache đã từng được khởi động và xử lý bất kỳ request nào, hoặc gặp lỗi trong quá trình chạy, các file này sẽ ghi nhận dữ liệu. Tuy nhiên, quan sát kích thước của cả 3 file này cho thấy tất cả đều có kích thước là 0 KB. Điều này cung cấp bằng chứng chắc chắn rằng dịch vụ Apache chưa từng được khởi chạy trên hệ thống này kể từ khi cài đặ
![Pasted image 20260908203749.png](./images/Pasted%20image%2020260908203749.png)

>0

## Q8: This machine was used to launch an attack on another. Which file contains the evidence for this?

**Yêu cầu:** Xác định file chứa bằng chứng cho thấy máy tính này đã được dùng để tấn công một máy tính khác.

Rà soát lại thư mục home của user `root` (`/root`), chúng ta tìm thấy một tập tin hình ảnh có tên `irZLAohL.jpeg`. Khi mở bức ảnh này, nó hiển thị ảnh chụp màn hình (screenshot) của một phiên làm việc trên hệ điều hành Windows. Màn hình Command Prompt đang thực thi một công cụ có tên `aylmao.exe`, được sử dụng để mô phỏng và tạo lưu lượng mạng độc hại (network flight simulator). Ảnh chụp màn hình này đóng vai trò như một bằng chứng (Artifact) cho thấy máy chủ Linux của Karen đã tham gia vào quá trình kiểm soát hoặc tấn công máy mục tiêu chạy Windows đó
![Pasted image 20260908203717.png](./images/Pasted%20image%2020260908203717.png)

>irZLAohL.jpeg
## Q9: It is believed that Karen was taunting a fellow computer expert through a bash script within the Documents directory. Who was the expert that Karen was taunting?

**Yêu cầu:** Xác định tên vị chuyên gia máy tính mà Karen đã chế nhạo thông qua một bash script trong thư mục Documents.

Để giải quyết câu hỏi này, ta cần xem lại file bash script trong thư mục Documents. Kiểm tra nội dung của file `firstscript_fixed`, ta thấy có các lệnh echo in ra màn hình. Dòng cuối cùng in ra: "Heck yeah! I can write bash too Young". Phân tích kỹ nội dung này sẽ tiết lộ danh tính của chuyên gia đang bị nhắm đến.
![Pasted image 20260908204104.png](./images/Pasted%20image%2020260908204104.png)

>Young

## Q10: A user executed the **su** command to gain root access multiple times at 11:26. Who was the user?

**Yêu cầu:** Xác định user nào đã nhiều lần thực thi lệnh `su` để chiếm quyền root vào lúc 11:26.

Mọi nỗ lực xác thực (authentication), bao gồm cả việc chuyển đổi người dùng bằng lệnh `su` (substitute user), đều được hệ thống ghi nhận (log). Trên các hệ thống Debian/Ubuntu, file log đảm nhiệm chức năng này là `/var/log/auth.log`. Mở file này và tìm kiếm các sự kiện xảy ra vào mốc thời gian 11:26. Các bản ghi log cho thấy một tài khoản dịch vụ cụ thể đã thực hiện nhiều nỗ lực leo thang đặc quyền
![Pasted image 20260908210107.png](./images/Pasted%20image%2020260908210107.png)

>postgres

## Q11: Based on the bash history, what is the current working directory?

**Yêu cầu:** Dựa trên lịch sử bash (bash history), thư mục làm việc hiện tại (current working directory) là gì?

Kiểm tra lại nội dung của file `.bash_history` để theo dõi các lệnh điều hướng thư mục (`cd`). Lệnh `cd` cuối cùng được ghi lại trong lịch sử là thao tác chuyển đến thư mục `../Documents/myfirsthack/`. Để xác nhận việc điều hướng này thành công và đây thực sự là thư mục làm việc hiện hành (CWD), ta kiểm tra các lệnh được thực thi ngay sau đó. Sau lệnh `cd`, người dùng đã sử dụng lệnh `echo` và `touch` để tạo tập tin `bob.txt`. Việc tập tin `bob.txt` tồn tại bên trong `/root/Documents/myfirsthack/` chứng tỏ lệnh `cd` cuối cùng đã thành công và xác định chính xác vị trí thư mục hiện tại![Pasted image 20260908204635.png](./images/Pasted%20image%2020260908204635.png)

>/root/Documents/myfirsthack/