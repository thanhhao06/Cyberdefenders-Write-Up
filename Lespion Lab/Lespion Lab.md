author: Azaki
published: published
created: 2026-01-07
tags: CyberDefener, Lab, Threat Intel

You have been tasked by a client whose network was compromised and brought offline to investigate the incident and determine the attacker's identity.

Incident responders and digital forensic investigators are currently on the scene and have conducted a preliminary investigation. Their findings show that the attack originated from a single user account, probably, an insider. Investigate the incident, find the insider, and uncover the attack actions.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@CyberDefender</b> 
</p>

## Q1: File -> Github.txt: What API key did the insider add to his GitHub repositories?

**Yêu cầu:** Tìm API key mà kẻ nội gián (insider) đã vô tình để lộ trên GitHub repositories của mình.

Quá trình điều tra bắt đầu bằng việc kiểm tra trang cá nhân của mục tiêu trên GitHub. Trong danh sách các kho lưu trữ (repositories), có một dự án đáng chú ý mang tên `Project-Build---Custom-Login-Page`, dấu hiệu cho thấy đây có thể là nơi chứa mã nguồn xử lý các chức năng đăng nhập và xác thực.
![Pasted image 20260907162242.png](./images/Pasted%20image%2020260907162242.png)

Tiến hành kiểm tra chi tiết mã nguồn bên trong kho lưu trữ này, cụ thể là file `Login Page.js`. Ngay ở những dòng đầu tiên, có thể thấy mục tiêu đã mắc một lỗi bảo mật nghiêm trọng khi hardcode trực tiếp một API key vào trong file.
![Pasted image 20260907162218.png](./images/Pasted%20image%2020260907162218.png)

>aJFRaLHjMXvYZgLPwiJkroYLGRkNBW

## Q2: File -> Github.txt: What plaintext password did the insider add to his GitHub repositories?

**Yêu cầu:** Xác định mật khẩu (dạng plaintext) được lưu trong repository của mục tiêu.

Tiếp tục phân tích file `Login Page.js` từ câu hỏi trước. Ở phần dưới của mã nguồn, có thể tìm thấy thông tin đăng nhập tĩnh bao gồm Username (`EMarseille99`) và một chuỗi Password. Tuy nhiên, mật khẩu này hiện không ở dạng rõ (plaintext) mà đang được mã hóa bằng Base64
![Pasted image 20260907162730.png](./images/Pasted%20image%2020260907162730.png)

Để lấy được mật khẩu thật, chúng ta cần giải mã chuỗi Base64 này. Sử dụng terminal với lệnh `base64 -d`, ta có thể dễ dàng chuyển đổi chuỗi mã hóa về lại định dạng plaintext ban đầu
![Pasted image 20260907162854.png](./images/Pasted%20image%2020260907162854.png)

> PicassoBaguette99

## Q3: File -> Github.txt: What cryptocurrency mining tool did the insider use?

Quay lại trang tổng quan GitHub của mục tiêu để kiểm tra các repositories khác. Tại đây, xuất hiện một repository liên quan trực tiếp đến tiền điện tử (crypto)
![Pasted image 20260907163231.png](./images/Pasted%20image%2020260907163231.png)
Tên của repository này chỉ đích danh một công cụ mã nguồn mở cực kỳ phổ biến thường được giới tội phạm mạng hoặc những người khai thác sử dụng để đào các đồng tiền ảo (như Monero) bằng sức mạnh của CPU/GPU

> XMRig

## Q4: On which gaming website did the insider have an account?

**Yêu cầu:** Tìm nền tảng chơi game mà mục tiêu có tài khoản.

Vì trang GitHub không cung cấp thêm manh mối trực tiếp nào về các tài khoản khác, bước tiếp theo là thực hiện OSINT dựa trên địa chỉ email của mục tiêu (`emarseille99@gmail.com`). Kết quả tìm kiếm Google cho email này dẫn đến một tài khoản Instagram có username là `@emarseille99`
![Pasted image 20260907164508.png](./images/Pasted%20image%2020260907164508.png)

Kiểm tra các bài đăng trên trang cá nhân Instagram của mục tiêu, phát hiện một bức ảnh có chứa mã QR
![Pasted image 20260907164533.png](./images/Pasted%20image%2020260907164533.png)

Sử dụng công cụ đọc mã QR (QR Reader) để giải mã hình ảnh này, kết quả trả về là một URL rút gọn
![Pasted image 20260907165344.png](./images/Pasted%20image%2020260907165344.png)

Khi truy cập vào đường dẫn rút gọn đó, trình duyệt tự động chuyển hướng đến trang profile cộng đồng của mục tiêu trên một nền tảng phân phối game trực tuyến
![Pasted image 20260907165427.png](./images/Pasted%20image%2020260907165427.png)

> Steam

## Q5: What is the link to the insider Instagram profile?

**Yêu cầu:** Xác định đường dẫn (link) đến trang hồ sơ Instagram của mục tiêu.

Dựa vào thông tin username đã thu thập được ở bước OSINT câu hỏi trước, ta kết hợp tên người dùng với địa chỉ tên miền gốc của mạng xã hội này
![Pasted image 20260907164508.png](./images/Pasted%20image%2020260907164508.png)

Với username là `emarseille99`, đường dẫn đầy đủ và chính xác đến hồ sơ của mục tiêu là
![Pasted image 20260907165455.png](./images/Pasted%20image%2020260907165455.png)

> https://www.instagram.com/emarseille99/

## Q6: Which country did the insider visit on her holiday?

**Yêu cầu:** Xác định quốc gia mà mục tiêu đã đến thăm trong kỳ nghỉ.

Tiếp tục khai thác thông tin từ các bài đăng còn lại trên Instagram, có một số bức ảnh chụp các địa danh du lịch. Dựa vào manh mối hình ảnh (chiếc áo có in chữ tượng hình), ta có thể khoanh vùng vị trí nằm ở các quốc gia hoặc vùng lãnh thổ sử dụng hệ thống chữ viết này như Trung Quốc, Nhật Bản, Hàn Quốc, Hong Kong, Đài Loan hoặc Singapore
![Pasted image 20260907170019.png](./images/Pasted%20image%2020260907170019.png)

Sử dụng phương pháp tìm kiếm hình ảnh ngược (Reverse Image Search), kiến trúc đặc trưng trong một bức ảnh khác khớp hoàn toàn với Marina Bay Sands, một tổ hợp khu nghỉ dưỡng và sòng bạc nổi tiếng
![Pasted image 20260907165840.png](./images/Pasted%20image%2020260907165840.png)

>Singapore

## Q7: Which city does the insider family live in?

**Yêu cầu:** Xác định thành phố nơi gia đình mục tiêu sinh sống.

Để tìm kiếm vị trí sinh sống, ta phân tích bức ảnh cuối cùng trên Instagram hiển thị một khu vực dân cư hoặc một ngôi nhà. Quan sát kỹ các chi tiết nhỏ trong bức ảnh, ở góc trên bên phải có treo quốc kỳ của Các Tiểu vương quốc Ả Rập Thống nhất (UAE)
![Pasted image 20260907170806.png](./images/Pasted%20image%2020260907170806.png)

Kết hợp manh mối này với phần bình luận (comments) của bài đăng, có một người bạn của mục tiêu đã để lại bình luận nhắc trực tiếp đến một thành phố cụ thể thuộc quốc gia này
![Pasted image 20260907170924.png](./images/Pasted%20image%2020260907170924.png)

>Dubai

## Q8: File -> office.jpg: You have been provided with a picture of the building in which the company has an office. Which city is the company located in?

**Yêu cầu:** Dựa trên bức ảnh `office.jpg`, xác định thành phố nơi công ty đặt văn phòng.

Phân tích bức ảnh, điểm mấu chốt đầu tiên là biển chỉ dẫn giao thông hướng về phía Nhà hát Alexandra (Alexandra Theatre)
![Pasted image 20260907171418.png](./images/Pasted%20image%2020260907171418.png)

Xác định được vị trí của nhà hát này trên bản đồ, ta bắt đầu đối chiếu và tìm kiếm các điểm mốc xuất hiện trên cột chỉ dẫn. Bằng cách lấy vị trí các điểm mốc nằm bên trái và bên phải, ta có thể khoanh vùng tương đối (triangulate) đoạn đường thẳng mà bức ảnh được chụp
![Pasted image 20260907173547.png](./images/Pasted%20image%2020260907173547.png)

Tiếp theo, phía sau cột chỉ đường có hình ảnh một trung tâm với chữ "Grand" hiện ra và phần sau bị che khuất. Theo suy luận logic về cách đặt tên các khu vực lớn, đây có thể là "Grand Central". Tìm kiếm Grand Central trên bản đồ cho thấy nó nằm ở vị trí hoàn toàn khớp với đường thẳng đã phỏng đoán
![Pasted image 20260907174207.png](./images/Pasted%20image%2020260907174207.png)

Sử dụng Google Street View để kiểm tra trực quan khu vực này, vị trí cột chỉ đường và tòa nhà văn phòng đã được xác nhận chính xác
![Pasted image 20260907172812.png](./images/Pasted%20image%2020260907172812.png)

> Birmingham

## Q9: File -> Webcam.png: With the intel, you have provided, our ground surveillance unit is now overlooking the person of interest suspected address. They saw them leaving their apartment and followed them to the airport. Their plane took off and landed in another country. Our intelligence team spotted the target with this IP camera. Which state is this camera in?

**Yêu cầu:** Xác định tiểu bang (state) nơi đặt chiếc IP camera đã ghi hình mục tiêu.

Kiểm tra bức ảnh `Webcam.png`, ở góc dưới có hiển thị watermark của **EarthCam** (một mạng lưới cung cấp webcam trực tiếp trên toàn cầu). Ở góc trên bên trái, có một đoạn văn bản đáng chú ý: **"A View from the Dome"**.
Bám sát vào hai từ khóa quan trọng này để truy vấn trên công cụ tìm kiếm, kết quả chỉ ra rằng đây là một camera được đặt tại Đại học Notre Dame (University of Notre Dame)
![Pasted image 20260907175115.png](./images/Pasted%20image%2020260907175115.png)

Truy cập trực tiếp vào hệ thống EarthCam để đối chiếu, hình ảnh trả về trùng khớp hoàn toàn với vị trí cần tìm
![Pasted image 20260907175049.png](./images/Pasted%20image%2020260907175049.png)

Vì đã có tên chính xác của trường đại học, chỉ cần tra cứu thông tin địa lý của cơ sở giáo dục này để trích xuất tên tiểu bang tương ứng tại Mỹ
![Pasted image 20260907175831.png](./images/Pasted%20image%2020260907175831.png)

>Indiana