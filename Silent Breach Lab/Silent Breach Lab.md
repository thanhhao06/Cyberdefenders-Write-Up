author: Azaki
published: published
created: 2025-12-31
tags: CyberDefener, Lab, Endpoint Forensics

The IMF is hit by a cyber attack compromising sensitive data. Luther sends Ethan to retrieve crucial information from a compromised server. Despite warnings, Ethan downloads the intel, which later becomes unreadable. To recover it, he creates a forensic image and asks Benji for help in decoding the files.

Resources: Windows Mail Artifacts: Microsoft HxStore.hxd (email) Research

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@ph1nx</b> 
</p>

## Q1: What is the MD5 hash of the potentially malicious EXE file the user downloaded?

**Yêu cầu:** Xác định MD5 hash của file thực thi đáng ngờ mà người dùng đã tải xuống.

Đầu tiên, mở forensic image bằng **FTK Imager** và kiểm tra thư mục `Downloads` trong profile của user. Tại đây có một file đáng chú ý tên `IMF-Info.pdf.exe`
![Silent Breach Lab-1.png](./images/Silent%20Breach%20Lab-1.png)
Tên file sử dụng **double extension** `.pdf.exe`, một kỹ thuật thường được dùng để khiến executable trông giống tài liệu PDF nếu người dùng không để ý phần mở rộng thật.

Tiếp theo, sử dụng **md5sum** để lấy giá trị MD5 của file
![Silent Breach Lab-2.png](./images/Silent%20Breach%20Lab-2.png)

>336a7cf476ebc7548c93507339196abb

## Q2: What is the URL from which the file was downloaded?

**Yêu cầu:** Xác định URL nguồn đã được sử dụng để tải file `IMF-Info.pdf.exe` xuống hệ thống.

Với các file được tải từ Internet trên Windows, có thể kiểm tra **Mark-of-the-Web** được lưu trong Alternate Data Stream `Zone.Identifier`. Trong FTK Imager, mở metadata đi kèm file `IMF-Info.pdf.exe` và quan sát trường `HostUrl`
![Silent Breach Lab-3.png](./images/Silent%20Breach%20Lab-3.png)
Thông tin này cho thấy executable được tải trực tiếp từ một HTTP server nội bộ tại địa chỉ `192.168.16.128` và sử dụng port `8000`

Ngoài ra, kết quả này cũng có thể được đối chiếu lại trong database lịch sử download của Microsoft Edge

>http://192.168.16.128:8000/IMF-Info.pdf.exe

## Q3: What application did the user use to download this file?

**Yêu cầu:** Xác định ứng dụng được người dùng sử dụng để tải file độc hại.

Trên hệ thống có cả **Google Chrome** và **Microsoft Edge**, vì vậy cần kiểm tra browser artifacts của từng trình duyệt thay vì chỉ dựa vào danh sách phần mềm đã cài đặt.

Database lịch sử của Chromium-based browsers thường được lưu trong file `History`. Sau khi kiểm tra Chrome không thấy bản ghi download tương ứng, tiếp tục phân tích Microsoft Edge tại:
```text
C:\Users\ethan\AppData\Local\Microsoft\Edge\User Data\Default\History
```

![Silent Breach Lab-4.png](./images/Silent%20Breach%20Lab-4.png)

Mở file bằng **DB Browser for SQLite** và kiểm tra bảng liên quan đến download. Tại đây xuất hiện bản ghi của `IMF-Info.pdf.exe` cùng URL đã xác định ở Q2
![Silent Breach Lab-5.png](./images/Silent%20Breach%20Lab-5.png)

>Microsoft Edge
## Q4: By examining Windows Mail artifacts, we found an email address mentioning three IP addresses of servers that are at risk or compromised. What are the IP addresses?

**Yêu cầu:** Phân tích Windows Mail artifacts và xác định ba địa chỉ IP được nhắc đến trong email liên quan đến các server có nguy cơ hoặc đã bị compromise.

Ban đầu có thể tìm kiếm file `.eml`, tuy nhiên Windows Mail không nhất thiết lưu email dưới dạng từng file riêng lẻ. Một artifact quan trọng của ứng dụng này là `HxStore.hxd`, nằm tại:
```text
C:\Users\ethan\AppData\Local\Packages\microsoft.windowscommunicationsapps_8wekyb3d8bbwe\LocalState\HxStore.hxd
```

![Silent Breach Lab-6.png](./images/Silent%20Breach%20Lab-6.png)
Export `HxStore.hxd` từ forensic image để phân tích. Đây là file binary nhưng vẫn chứa nhiều chuỗi có thể đọc được, bao gồm nội dung và metadata của email.

Có thể mở file bằng **HxD** hoặc sử dụng `strings`, sau đó tìm các từ khóa liên quan như `risk`, `compromised`, `IP` hoặc trực tiếp tìm pattern IPv4
![Silent Breach Lab-7.png](./images/Silent%20Breach%20Lab-7.png)

Từ nội dung email, xác định được ba địa chỉ IP được nhắc đến
![Silent Breach Lab-8.png](./images/Silent%20Breach%20Lab-8.png)

>145.67.29.88, 212.33.10.112, 192.168.16.128

## Q5: By examining the malicious executable, we found that it uses an obfuscated PowerShell script to decrypt specific files. What predefined password does the script use for encryption?

**Yêu cầu:** Phân tích executable độc hại và xác định password được hardcode trong PowerShell script dùng cho quá trình mã hóa file.

Trước tiên, kiểm tra hành vi của mẫu `IMF-Info.pdf.exe` trên các nguồn malware analysis. Trong kết quả behavioral analysis có thể thấy một PowerShell script đáng ngờ với tên:
```text
Gz3m6mG3j2TyAqF2Zx4v.ps1
```

![Silent Breach Lab-9.png](./images/Silent%20Breach%20Lab-9.png)

![Silent Breach Lab-10.png](./images/Silent%20Breach%20Lab-10.png)

Tiếp tục phân tích executable bằng `strings` và tìm các chuỗi liên quan đến PowerShell hoặc tên script. Từ đó có thể tìm thấy một đoạn dữ liệu Base64 bị obfuscate
![Silent Breach Lab-11.png](./images/Silent%20Breach%20Lab-11.png)
Điểm đáng chú ý là script sử dụng `[Array]::Reverse(...)`, cho thấy dữ liệu cần được **đảo ngược trước khi decode Base64**. Vì vậy có thể đưa chuỗi vào CyberChef với thứ tự xử lý:
```text
Reverse
From Base64
```

Sau khi deobfuscate, script lộ ra các tham số mã hóa, trong đó password được hardcode là `Imf!nfo#2025Sec$`. Script sử dụng password này cùng salt cố định và PBKDF2 để sinh key/IV cho AES
![Silent Breach Lab-12.png](./images/Silent%20Breach%20Lab-12.png)

>Imf!nfo#2025Sec$

## Q6: After identifying how the script works, decrypt the files and submit the secret string.

**Yêu cầu:** Dựa trên cơ chế mã hóa đã phân tích ở Q5, giải mã các file bị mã hóa và tìm secret string.

Từ PowerShell script sau khi deobfuscate có thể thấy malware nhắm đến hai file trên Desktop:
```text
C:\Users\ethan\Desktop\IMF-Secret.pdf
C:\Users\ethan\Desktop\IMF-Mission.pdf
```

Sau khi mã hóa, các file gốc bị xóa và dữ liệu ciphertext được lưu lại với phần mở rộng `.enc`. Vì vậy, quay lại **FTK Imager**, export hai file sau từ Desktop:
```text
IMF-Secret.enc
IMF-Mission.enc
```

Các tham số cần giữ nguyên khi đảo ngược quá trình mã hóa gồm:
```text
Password   : Imf!nfo#2025Sec$
Salt       : 01 02 03 04 05 06 07 08
Iterations : 10000
Key size   : 32 bytes (AES-256)
IV size    : 16 bytes
Mode       : AES-CBC
Padding    : PKCS7
```

Trước tiên cần tìm KEY và IV và vì đã có kích thước của KEY và IV nên kích thước chuỗi cần tìm là 48 byte tương ứng với 384 bit
![Silent Breach Lab-13.png](./images/Silent%20Breach%20Lab-13.png)

Sau khi chạy script, hai file PDF gốc sẽ được khôi phục. Mở `IMF-Mission.pdf` để đọc **secret string** được yêu cầu bởi challenge
![Silent Breach Lab-14.png](./images/Silent%20Breach%20Lab-14.png)

Ngoài ra có thể dùng một PowerShell script để thực hiện quá trình giải mã với cùng password, salt và PBKDF2 parameters:
```powershell
$password = "Imf!nfo#2025Sec$"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32
$ivSize = 16

$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes(
    $password, $salt, $iterations
)

$key = $deriveBytes.GetBytes($keySize)
$iv  = $deriveBytes.GetBytes($ivSize)

$inputFiles = @(
    "IMF-Secret.enc",
    "IMF-Mission.enc"
)

foreach ($inputFile in $inputFiles) {
    $outputFile = $inputFile -replace '\.enc$', '.pdf'

    $aes = [System.Security.Cryptography.Aes]::Create()
    $aes.Key = $key
    $aes.IV = $iv
    $aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

    $decryptor = $aes.CreateDecryptor()
    $cipherBytes = [System.IO.File]::ReadAllBytes($inputFile)
    $plainBytes = $decryptor.TransformFinalBlock(
        $cipherBytes, 0, $cipherBytes.Length
    )

    [System.IO.File]::WriteAllBytes($outputFile, $plainBytes)
    $aes.Dispose()
}
```

>`CyberDefenders{N3v3r_eX3cuTe_F!l3$_dOwnL0ded_fr0m_M@lic10u5_$erV3r}`

---

# ATTACK CHAIN

![Silent Breach Lab.png](./images/Silent%20Breach%20Lab.png)