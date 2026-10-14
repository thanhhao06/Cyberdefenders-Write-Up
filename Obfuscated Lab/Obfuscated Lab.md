author: Azaki
published: published
created: 2026-10-08
tags: CyberDefenders, Lab, Malware Analyst

While working as a SOC analyst, you may encounter alerts from the enterprise Endpoint Detection and Response (EDR) system regarding unusual activity on an end-user machine. In one instance, a user reported receiving an email containing a DOC file from an unknown sender. The user subsequently submitted the document for analysis to ensure it does not pose a security risk.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Josh Stroschein</b> 
</p>

## Q1: What is the SHA256 hash of the **DOC** file?

**Yêu cầu:** Xác định giá trị băm SHA256 của tệp DOC độc hại

Sử dụng lệnh sha256sum tính toán mã băm cho tệp tài liệu document.doc
![Pasted image 20261008174801.png](./images/Pasted%20image%2020261008174801.png)

>ff2c8cadaa0fd8da6138cce6fce37e001f53a5d9ceccd67945b15ae273f4d751

## Q2: Multiple streams contain macros in this document. Provide the number of the lowest one.

**Yêu cầu:** Tìm số thứ tự của luồng stream nhỏ nhất có chứa macro trong tài liệu

Sử dụng oledump phân tích cấu trúc các stream trong tệp Word độc hại để phát hiện mã macro
![Pasted image 20261008175102.png](./images/Pasted%20image%2020261008175102.png)

Chữ cái M hoặc m đánh dấu luồng chứa mã macro và luồng có chỉ số nhỏ nhất là 8

>8

## Q3: What is the decryption key of the obfuscated code?

**Yêu cầu:** Xác định khóa giải mã được dùng cho đoạn mã bị làm rối

Trích xuất và giải nén mã macro VBA từ stream 8 bằng tham số --vbadecompress của oledump.py tôi ghi nhận chuỗi khóa giải mã được truyền vào lệnh thực thi
![Pasted image 20261008175237.png](./images/Pasted%20image%2020261008175237.png)

>EzZETcSXyKAdF_e5I2i1

## Q4: What is the name of the dropped file?

**Yêu cầu:** Xác định tên của tệp được tạo ra trên hệ thống

Đọc luồng mã VBA xác định đường dẫn tệp script được malware ghi vào thư mục %appdata%\Microsoft\Windows
![Pasted image 20261008175333.png](./images/Pasted%20image%2020261008175333.png)

>maintools.js

## Q5: This script uses what language?

**Yêu cầu:** Xác định ngôn ngữ lập trình của tệp script này

Tệp rơi xuống mang đuôi mở rộng .js và được WScript.Shell gọi thực thi trực tiếp trên hệ thống Windows thông qua engine JScript
![Pasted image 20261008175550.png](./images/Pasted%20image%2020261008175550.png)

>JScript

## Q6: What is the name of the variable that is assigned the command-line arguments?

**Yêu cầu:** Xác định tên biến nhận các tham số truyền vào từ dòng lệnh

Trong mã VBA phát hiện malware trích xuất đoạn mã nhúng từ offset byte 81 với độ dài 16827 byte
![Obfuscated Lab-4.png](./images/Obfuscated%20Lab-4.png)

Dùng lệnh grep với cờ -b định vị vị trí byte của chuỗi dấu hiệu nhận diện trong tệp DOC
![Obfuscated Lab-2.png](./images/Obfuscated%20Lab-2.png)

Sử dụng công cụ dd trích xuất chính xác payload script từ vị trí offset 148158 với kích thước 16827 byte
![Obfuscated Lab-3.png](./images/Obfuscated%20Lab-3.png)

Mở tệp maintools.js vừa bóc tách kiểm tra dòng đầu tiên thấy biến wvy1 được gán nhận mảng tham số dòng lệnh WScript.Arguments
![Obfuscated Lab.png](./images/Obfuscated%20Lab.png)

>wvy1

## Q7: How many command-line arguments does this script expect?

**Yêu cầu:** Xác định số lượng tham số dòng lệnh mà script yêu cầu khi thực thi

Kiểm tra câu lệnh điều kiện rà soát độ dài mảng tham số dòng lệnh wvy1(0) xác nhận script yêu cầu ít nhất 1 tham số
![Obfuscated Lab-5.png](./images/Obfuscated%20Lab-5.png)

>1

## Q8: What instruction is executed if this script encounters an error?

**Yêu cầu:** Xác định lệnh sẽ được thực thi khi script gặp lỗi

Khối lệnh try catch được thiết lập để bắt các ngoại lệ và gọi hàm thoát tiến trình WScript.Quit() khi xảy ra lỗi thực thi
![Obfuscated Lab-6.png](./images/Obfuscated%20Lab-6.png)

>WScript.Quit()

## Q9: What function returns the next stage of code (i.e. the first round of obfuscated code)?

**Yêu cầu:** Xác định hàm trả về đoạn mã cho giai đoạn tiếp theo

Biến ES3c được gán kết quả trả về từ hàm y3zb chứa chuỗi mã độc đã làm rối cho giai đoạn xử lý kế tiếp
![Obfuscated Lab-7.png](./images/Obfuscated%20Lab-7.png)

>y3zb

## Q10: The function **LXv5** is important, what variable is assigned a key string value in determining what this function does?

**Yêu cầu:** Xác định tên biến chứa chuỗi khóa định hình chức năng của hàm LXv5

Kiểm tra hàm LXv5 thấy biến LUK7 được khởi tạo một chuỗi bảng chữ cái đặc trưng
![Obfuscated Lab-8.png](./images/Obfuscated%20Lab-8.png)

>LUK7

## Q11: What encoding scheme is this function responsible for decoding?

**Yêu cầu:** Xác định cơ chế mã hóa mà hàm này đảm nhận giải mã

Chuỗi ký tự được gán cho biến LUK7 chính là bảng mã tiêu chuẩn của thuật toán Base64
```
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
```

>Base64

## Q12: In the function **CpPT**, the first two **for()** loops are responsible for what important part of this function?

**Yêu cầu:** Xác định chức năng của hai vòng lặp for đầu tiên trong hàm CpPT

Hai vòng lặp for đầu tiên thực hiện khởi tạo mảng trạng thái 256 phần tử và hoán vị các giá trị dựa trên khóa giải mã tương ứng với thuật toán sinh lịch trình khóa Key-Scheduling Algorithm
![Obfuscated Lab-9.png](./images/Obfuscated%20Lab-9.png)

```js
for (var i = 0; i < 256; i++) {
    AWy7[i] = i;
}
for (var i = 0; i < 256; i++) {
    V2Vl = (V2Vl + AWy7[i] + bOe3.charCodeAt(i % bOe3.length)) % 256;
    qyCq = AWy7[i];
    AWy7[i] = AWy7[V2Vl];
    AWy7[V2Vl] = qyCq;
}
```

>Key-Scheduling Algorithm

## Q13: The function CpPT requires two arguments, where does the value of the first argument come from?

**Yêu cầu:** Xác định nguồn gốc giá trị của đối số thứ nhất trong hàm CpPT

Đối số đầu tiên ssWZ được truyền vào từ phần tử đầu tiên của mảng tham số dòng lệnh command-line argument
![Obfuscated Lab-10.png](./images/Obfuscated%20Lab-10.png)

>command-line argument

## Q14: For the function **CpPT**, what does the first argument represent?

**Yêu cầu:** Xác định ý nghĩa thực sự của đối số thứ nhất trong hàm CpPT

Trong script tệp maintools.js đối số đầu tiên ssWZ lấy trực tiếp từ tham số dòng lệnh wvy1(0)
![Obfuscated Lab-13.png](./images/Obfuscated%20Lab-13.png)

Đối chiếu lại lệnh thực thi trong mã VBA chuỗi ký tự EzZETcSXyKAdF_e5I2i1 được truyền vào làm tham số chính là khóa giải mã key cho thuật toán
![Obfuscated Lab-14.png](./images/Obfuscated%20Lab-14.png)

>key

## Q15: What encryption algorithm does the function **CpPT** implement in this script?

**Yêu cầu:** Xác định thuật toán mã hóa được hàm CpPT triển khai

Cấu trúc khởi tạo mảng hoán vị 256 byte qua KSA kết hợp với vòng lặp tạo dòng khóa PRGA để giải mã payload là thuật toán mã hóa dòng đối xứng RC4

>RC4

## Q16: What function is responsible for executing the deobfuscated code?

**Yêu cầu:** Xác định hàm chịu trách nhiệm thực thi đoạn mã đã được giải mã

Sau khi chuỗi mã độc trải qua giải mã Base64 và RC4 nó được chuyển thẳng vào hàm eval để nạp và thực thi trực tiếp trong bộ nhớ
![Obfuscated Lab-16.png](./images/Obfuscated%20Lab-16.png)

>eval

## Q17: What Windows Script Host program can be used to execute this script in command-line mode?

**Yêu cầu:** Xác định chương trình Windows Script Host dùng để chạy script này ở chế độ dòng lệnh

Nạp chuỗi làm rối vào CyberChef giải mã tuần tự qua Base64 và thuật toán RC4 với chuỗi khóa EzZETcSXyKAdF_e5I2i1 thu được mã nguồn sử dụng đối tượng WScript Để chạy tệp script này ở chế độ dòng lệnh trên hệ thống Windows cần sử dụng chương trình cscript.exe
![Obfuscated Lab-11.png](./images/Obfuscated%20Lab-11.png)

>cscript.exe

## Q18: What is the name of the first function defined in the deobfuscated code?

**Yêu cầu:** Xác định tên hàm đầu tiên được định nghĩa trong đoạn mã sau khi giải mã hoàn chỉnh

Từ kết quả giải mã ở trên quan sát dòng đầu tiên của mã nguồn hàm đầu tiên được khai báo là UspD
![Obfuscated Lab-12.png](./images/Obfuscated%20Lab-12.png)

>UspD