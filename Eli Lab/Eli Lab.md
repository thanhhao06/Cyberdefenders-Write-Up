author: Azaki
published: published
created: 2026-09-10
tags: CyberDefener, Lab, Threat Intel

A user reported strange browser behavior and missing files on their Chromebook. SOC analysts suspect unauthorized access and data theft. You’ve received a forensic image for review. Investigate user activity, application data, and browsing artifacts to uncover what happened.

Resources:

1. [Hindsight from Ryan Benson: https://github.com/obsidianforensics/hindsight](https://github.com/obsidianforensics/hindsight)
2. [Chromebook Data Locations: https://www.magnetforensics.com/blog/chromebook-data-locations/](https://www.magnetforensics.com/blog/chromebook-data-locations/)
3. [Taking a Byte out of Chromebook Analysis: https://www.magnetforensics.com/resources/taking-a-byte-of-chromebook-analysis-mvs-may-5/](https://www.magnetforensics.com/resources/taking-a-byte-of-chromebook-analysis-mvs-may-5/)
4. [Ross Donnelly on Google Takeout Location Data: https://dfir.pubpub.org/pub/d39u7lg1/release/1](https://dfir.pubpub.org/pub/d39u7lg1/release/1)
5. [Takeout Locations: https://www.magnetforensics.com/resources/targeted-locations-quick-reference-guide-for-android-and-google-takeouts/](https://www.magnetforensics.com/resources/targeted-locations-quick-reference-guide-for-android-and-google-takeouts/)
6. [Takeout Data: https://www.magnetforensics.com/resources/exploring-the-data-available-from-google-takeout-webinar-sep-22-oct-6/](https://www.magnetforensics.com/resources/exploring-the-data-available-from-google-takeout-webinar-sep-22-oct-6/)

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Magnet Forensics, Jordan Kimball, Danlan Navarro,Haykey Froio, Alayna Cash, Jessica Hyde</b> </p>

# Q1: The folder to store all your data in - How many files are in Eli's downloads directory?

**Yêu cầu:** Có bao nhiêu tập tin trong thư mục tải xuống (downloads directory) của Eli?

Để kiểm tra số lượng file trong thư mục download của người dùng Chromebook, chúng ta cần duyệt đến phần dữ liệu đã được giải mã của hệ thống. Đường dẫn lưu trữ mặc định nằm tại `decrypted/mount/user/Downloads`. Mở thư mục này ra và đếm số lượng các mục hiển thị
![Eli Lab.png](./images/Eli%20Lab.png)
>6

## Q2: Smile for the camera - What is the MD5 hash of the user's profile photo?

**Yêu cầu:** Mã băm MD5 của ảnh đại diện (profile photo) của người dùng là gì?

Vì được yêu cầu tìm ảnh đại diện của người dùng (avatar), ta điều hướng đến thư mục quản lý tài khoản theo đường dẫn `decrypted/mount/user/Accounts/Avatar Images`. Tại đây, chúng ta tìm thấy một tập tin hình ảnh gắn liền với địa chỉ email của người dùng
![Eli Lab-1.png](./images/Eli%20Lab-1.png)

Sử dụng lệnh `md5sum` đối với tập tin hình ảnh này để tính toán giá trị băm
![Eli Lab-2.png](./images/Eli%20Lab-2.png)

>5DDD4FE0041839DEB0A4B0252002127B
## Q3: Road Trip! - What city was Eli's destination in?

**Yêu cầu:** Eli có điểm đến là thành phố nào trong chuyến đi đường bộ (road trip)?

Quay lại thư mục `Downloads` đã kiểm tra ở Q1, chúng ta chú ý thấy một số ảnh chụp màn hình. Trong đó, có một bức ảnh chụp giao diện bản đồ đường đi của Google Maps
![Eli Lab-3.png](./images/Eli%20Lab-3.png)

Mở hình ảnh này lên và quan sát kỹ bảng điều khiển lộ trình ở góc trái. Tại ô "Add destination", hệ thống ghi nhận rõ ràng địa điểm điểm đến là một địa chỉ thuộc NY-3. Tên thành phố nằm ngay phía sau tên đường
![Eli Lab-4.png](./images/Eli%20Lab-4.png)


>Plattsburgh

## Q4: Promise Me - How many promises does Wickr make?

**Yêu cầu:** Ứng dụng Wickr đưa ra bao nhiêu "lời hứa" (promises) về bảo mật?

Vẫn tiếp tục kiểm tra các tập tin bên trong thư mục `Downloads`. Tại đây có lưu một tài liệu định dạng PDF mang tên `Wickr-Customer-Security-Promises-November-2020.pdf`
![Eli Lab-5.png](./images/Eli%20Lab-5.png)

Mở tập tin PDF này lên, ta thấy một bảng liệt kê chi tiết các điều khoản được gọi là "Wickr's Customer Security Promises". Chỉ cần đếm số lượng các gạch đầu dòng/hàng trong bảng này
![Eli Lab-6.png](./images/Eli%20Lab-6.png)

>9

## Q5: Key-ty Cat - What are the last five characters of the key for the Tabby Cat extension?

**Yêu cầu:** 5 ký tự cuối cùng của khóa (key) dành cho tiện ích mở rộng Tabby Cat là gì?

Để tìm key của một extension trên Chrome/Chromebook, ta cần định vị thư mục cài đặt của extension đó. Sử dụng tính năng tìm kiếm để dò tìm thư mục chứa dữ liệu của tiện ích mở rộng có tên "Tabby". Kết quả trả về cho thấy thư mục của extension này có chuỗi định danh bắt đầu bằng `mefhakmg...`
![Eli Lab-7.png](./images/Eli%20Lab-7.png)

Truy cập vào đúng đường dẫn thư mục đã tìm được (`decrypted/mount/user/Extensions/mefhakmgclhhfbdadeojlkbllmecialg`)
![Eli Lab-8.png](./images/Eli%20Lab-8.png)

Bên trong thư mục extension, mở tệp tin cấu hình chính là `manifest.json`
![Eli Lab-9.png](./images/Eli%20Lab-9.png)

Dò tìm đến trường dữ liệu `"key":` bên trong cấu trúc JSON. Lấy 5 ký tự cuối cùng của chuỗi giá trị (value) tương ứng với khóa này
![Eli Lab-10.png](./images/Eli%20Lab-10.png)

>DAQAB

## Q6: Time to jam out - How many songs does Eli have downloaded?

**Yêu cầu:** Eli đã tải xuống bao nhiêu bài hát?

Để đếm số lượng bài hát, ta cần truy cập vào khu vực lưu trữ đa phương tiện. Mở thư mục `Music` của hệ thống (`decrypted/mount/user/MyFiles/Music`) lên và đếm số lượng tệp tin âm thanh có trong đó. Dữ liệu hiển thị có 2 file MP3
![Eli Lab-11.png](./images/Eli%20Lab-11.png)

>2

## Q7: Autofill, roll out - Which word was Autofilled the most?

**Yêu cầu:** Từ (word) nào được tính năng tự động điền (Autofill) sử dụng nhiều nhất?

Dữ liệu Autofill là một phần của bộ nhớ đệm trình duyệt web. Do đó, ta cần tìm đến tệp cơ sở dữ liệu `Web Data` của trình duyệt Chrome trong thư mục gốc của người dùng
![Eli Lab-12.png](./images/Eli%20Lab-12.png)

Sử dụng công cụ xem database SQLite để mở tệp này. Truy vấn bảng `autofill` và đếm số lần xuất hiện ở cột `count` của các giá trị. Từ khóa `email` có tần suất xuất hiện cao nhất (3 lần)
![Eli Lab-13.png](./images/Eli%20Lab-13.png)

>email

## Q8: Dress for success - What is this bird's image's logical size in bytes?

**Yêu cầu:** Kích thước logic (tính bằng bytes) của bức ảnh "con chim" là bao nhiêu?

Tìm kiếm các hình ảnh liên quan đến loài chim trong các thư mục lưu trữ của hệ thống. Một trong những tập tin phù hợp là hình ảnh logo của chim cánh cụt Linux (Tux) nằm trong thư mục `Downloads`
![Eli Lab-14.png](./images/Eli%20Lab-14.png)

Kiểm tra thông tin chi tiết qua lệnh `ls -l` hoặc bảng thuộc tính của tệp tin ảnh, ta thấy kích thước logic chính xác của tệp `tux.png` là 46791 bytes
![Eli Lab-15.png](./images/Eli%20Lab-15.png)


>46791

## Q9: Repeat customer - What was Eli's top-visited site?

**Yêu cầu:** Trang web nào được Eli truy cập nhiều nhất?

Để phân tích lịch sử duyệt web, mở dữ liệu hoạt động của người dùng trong Google Takeout (`Takeout/My Activity/Chrome/MyActivity.html`)
![Eli Lab-16.png](./images/Eli%20Lab-16.png)

Rà soát các trang được ghi nhận lịch sử truy cập trên giao diện HTML, ta thấy địa chỉ `mail.protonmail.com` xuất hiện với tần suất và số lượt tương tác dày đặc nhất
![Eli Lab-17.png](./images/Eli%20Lab-17.png)

>protonmail.com

## Q10: Vroom Vroom, What is the name of the car-related theme?

**Yêu cầu:** Tên của chủ đề (theme) trình duyệt có liên quan đến xe hơi là gì?

Việc tìm kiếm bằng tên không khả thi vì thư mục theme thường sử dụng mã ID ngẫu nhiên. Bằng cách tìm kiếm thủ công vào các thư mục quản lý theme, mở tệp `manifest.json` của theme nghi ngờ lên để đọc trường `"name"`. Thông tin này sẽ cung cấp tên chính xác của theme.
![Eli Lab-18.png](./images/Eli%20Lab-18.png)


>Lamborghini Cherry

## Q11: You got mail - How many emails were received from **notification@service.tiktok.com**?

**Yêu cầu:** Có bao nhiêu email được nhận từ địa chỉ `notification@service.tiktok.com`?

Tìm đến thư mục lưu trữ email bên trong dữ liệu Google Takeout (`Takeout/Mail/All mail Including Spam and Trash.mbox`)
![Eli Lab-19.png](./images/Eli%20Lab-19.png)

Thực hiện tìm kiếm nội dung tệp mbox với chuỗi `notification@service.tiktok.com`. Đếm số lượng dòng tiêu đề `From:` khớp với địa chỉ người gửi này để ra tổng số thư
![Eli Lab-20.png](./images/Eli%20Lab-20.png)

>6

## Q12: Hungry for directions - Where did the user request directions on **March 4, 2021, at 4:15:18 AM EDT**?

**Yêu cầu:** Người dùng đã yêu cầu chỉ đường (request directions) đến đâu vào ngày 4 tháng 3 năm 2021, lúc 4:15:18 AM EDT?

Dữ liệu này liên quan đến hoạt động định vị, nên ta kiểm tra tệp hoạt động của Google Maps trong phần Google Takeout (`Takeout/My Activity/Maps/MyActivity.html`)
![Eli Lab-21.png](./images/Eli%20Lab-21.png)

Dò tìm bản ghi ứng với mốc thời gian `Mar 4, 2021, 4:15:18 AM EDT`. Bản ghi ghi nhận thông tin yêu cầu chỉ đường (Directions to) đến một địa điểm cụ thể
![Eli Lab-22.png](./images/Eli%20Lab-22.png)

Xác minh lại thông tin qua bản đồ chi tiết, địa điểm đích được hiển thị rõ ràng là cửa hàng ăn nhanh Chick-fil-A tại Plattsburgh.
![Eli Lab-23.png](./images/Eli%20Lab-23.png)

>Chick-fil-A

## Q13: Who defines essential? - What was searched on **Mar 4, 2021, at 4:09:35 AM EDT?**

**Yêu cầu:** Người dùng đã tìm kiếm (search) từ khóa gì vào lúc 4:09:35 AM EDT ngày 4 tháng 3 năm 2021?

Chuyển sang kiểm tra dữ liệu hoạt động tìm kiếm trong Google Takeout (`Takeout/My Activity/Search/MyActivity.html`)
![Eli Lab-24.png](./images/Eli%20Lab-24.png)

Đối chiếu với mốc thời gian chỉ định (`Mar 4, 2021, 4:09:35 AM EDT`), ta dễ dàng trích xuất được nguyên văn chuỗi từ khóa mà người dùng đã tra cứu trên thanh tìm kiếm.
![Eli Lab-25.png](./images/Eli%20Lab-25.png)

>is travelling to get chicken essential travel

## Q14: I got three subscribers, and counting - How many YouTube channels is the user subscribed to?

**Yêu cầu:** Người dùng đang đăng ký (subscribe) bao nhiêu kênh YouTube?

Truy cập vào bộ dữ liệu liên quan đến YouTube trong Google Takeout (`Takeout/YouTube and YouTube Music/subscriptions/subscriptions.json`)
![Eli Lab-26.png](./images/Eli%20Lab-26.png)

Kiểm tra nội dung của tệp JSON này. Việc mảng dữ liệu bên trong tệp hoàn toàn trống (`[]`) xác nhận người dùng không đăng ký theo dõi bất kỳ kênh YouTube nào
![Eli Lab-27.png](./images/Eli%20Lab-27.png)
>0

## Q15: Time flies when you're watching YT - What date was the first YouTube video the user watched uploaded?

**Yêu cầu:** Ngày đăng tải (upload date) của video YouTube đầu tiên mà người dùng đã xem là ngày nào?

Truy cập vào phần lịch sử xem YouTube trong Takeout (`Takeout/YouTube and YouTube Music/history/watch-history.html`) để tìm video đầu tiên mà Eli đã xem
![Eli Lab-28.png](./images/Eli%20Lab-28.png)

Cuộn xuống mục ghi nhận video cũ nhất, ta xác định được liên kết và tiêu đề của video. 
![Eli Lab-29.png](./images/Eli%20Lab-29.png)

Sau khi dã có URL của video mình tiến hành kiểm tra thông tin siêu dữ liệu (metadata) của video này qua công cụ Fake News Debunker, ngày tải lên (Upload time) hiển thị chính xác là ngày 27 tháng 1 năm 2021
![Eli Lab-30.png](./images/Eli%20Lab-30.png)

>2021-01-27