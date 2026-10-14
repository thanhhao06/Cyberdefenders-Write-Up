author: Azaki
published: published
created: 2026-07-04
tags: CyberDefener, Lab, Threat Intel

This exercise focuses on Open-Source Intelligence (OSINT) as a method for mining and analyzing publicly available data. It aims to enhance skills in producing valuable insights when investigating external threats in the role of a security blue team analyst. Through practical application, participants will learn to effectively gather and interpret information to improve overall security measures.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Champlain College Digital Forensics Association</b> 
</p>
## Q1: Who is the Registrar for **jameskainth.com**?

**Yêu cầu:** Xác định nhà đăng ký tên miền (Registrar) cho `jameskainth.com.

Để xem tên công ty đăng ký tên miền của domain `jameskainth.com`, ta có thể sử dụng công cụ `whois` để xem thông tin lúc đăng ký domain
![Pasted image 20260908083301.png](./images/Pasted%20image%2020260908083301.png)

> namecheap

## Q2: What is the Zoom meeting id of the **British Prime Ministers** Cabinet Meeting?

**Yêu cầu:** Xác định ID cuộc họp Zoom của Nội các Thủ tướng Anh.

Đây là một câu hỏi OSINT yêu cầu tìm ID của cuộc họp trên Zoom của British Prime Ministers. Thực hiện tra cứu với các từ khóa liên quan, ta tìm thấy một trang đề cập trực tiếp đến cuộc họp này
![Pasted image 20260908083720.png](./images/Pasted%20image%2020260908083720.png)

Truy cập vào và kiểm tra, ta thấy người họp đã sơ ý để lộ phần Title bar (thanh tiêu đề) có chứa rõ ràng ID của cuộc họp
![Pasted image 20260908083559.png](./images/Pasted%20image%2020260908083559.png)

> 539544323

## Q3: In 2019 UVM’s Ichthyology Class Had to Name their fish for class. Can you find out what the most recently assigned fish name was?

**Yêu cầu:** Tìm tên loài cá được giao gần đây nhất trong lớp học Ngư học (Ichthyology Class) của UVM năm 2019

Đầu tiên, để truy vết được trong lớp học Ichthyology để xem các học sinh đã đặt tên loài cá là gì, ta cần tìm được website của trường này. Sau khi tra cứu, ta thu được URL của trường
![Pasted image 20260908085520.png](./images/Pasted%20image%2020260908085520.png)

Sau khi đã có URL của trường, ta tiến hành dùng tool Wayback Machine để truy ngược về các bài đăng của trường. Tuy nhiên, như hệ thống hiển thị, có rất nhiều bài đăng được lưu trữ trong những năm qua
![Pasted image 20260908085350.png](./images/Pasted%20image%2020260908085350.png)

Vì thế, ta tiến hành lọc các tin có liên quan đến tên của loài cá. Kết quả khá bất ngờ là chỉ còn lại 2 trang nói về nội dung này
![Pasted image 20260908085324.png](./images/Pasted%20image%2020260908085324.png)

Tiến hành truy cập trang, ta tải được file Excel chứa thông tin của từng học sinh ứng với tên loài cá. Vì đề đã nói là tên loài cá gần đây nhất nên ta lấy tên loài cá nằm ở dòng cuối cùng
![Pasted image 20260908085211.png](./images/Pasted%20image%2020260908085211.png)

>saccopharyngiformes

## Q4: Can you identify the state from which this picture was taken? See the attached photo.

**Yêu cầu:** Xác định tiểu bang (state) nơi bức ảnh được chụp

Để tìm kiếm địa điểm của bức tượng khủng long này, trước tiên ta cần tìm hiểu xem đây là loài khủng long gì. Sau khi tra cứu, ta xác định được đây là loài Pteranodon

Sau đó mình tìm và lọc ra các khu vui chơi và công viên có loài khủng long này và mình lọc ra đưuọc 4 địa điểm tìm năng
- Dinosaur World
- The Dinosaur Place
- Dinosaur Land
- The Dinosaur Park

Với việc có được các địa điểm tiềm năng, ta thử đối chiếu các đặc điểm để tìm ra vị trí trong bức ảnh. Khi tìm tới Dinosaur Land, ta bắt gặp hình ảnh ngọn núi màu đỏ đặc trưng
![Pasted image 20260908093408.png](./images/Pasted%20image%2020260908093408.png)

Ngọn núi này tương ứng với ngọn núi phía sau của bức ảnh, điều này làm cơ sở cho việc xác định đây chính là địa điểm cần tìm. Tuy nhiên, trên Google Maps và Google Earth đều không thể xem khu vực này ở chế độ street view
![Pasted image 20260908093455.png](./images/Pasted%20image%2020260908093455.png)

Vì vậy, ta tiếp tục tìm các bức ảnh được người dùng đăng tải lên trên trang của Dinosaur Land này. Sau một hồi tìm kiếm, ta đã tìm được bức ảnh miêu tả rõ về vị trí của bức tượng con khủng long này để xác thực
![Pasted image 20260908092647.png](./images/Pasted%20image%2020260908092647.png)

Sau khi đã có tên địa điểm cần tìm là Dinosaur Land, ta dễ dàng tra cứu được tiểu bang của nó

>Virginia