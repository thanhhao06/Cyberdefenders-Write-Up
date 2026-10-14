author: Azaki
published: published
created: 2026-09-10
tags: CyberDefener, Lab, Endpoint Forensic

A multinational corporation has suffered a cyber attack, resulting in the theft of sensitive data. The attack employed a previously unseen variant of the BlackEnergy v2 malware. The company's security team has obtained a memory dump from the infected machine and is seeking your expertise as a SOC analyst to analyze the dump in order to understand the scope and impact of the attack.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@HouseOfStark</b> 
</p>

## Q1: Which volatility profile would be best for this machine?

**Yêu cầu:** Xác định profile Volatility phù hợp nhất cho file memory dump này.

Để xác định hệ điều hành và profile tương ứng cho tệp bộ nhớ, chúng ta sử dụng plugin `imageinfo` của Volatility. Kết quả phân tích từ plugin này sẽ quét các cấu trúc KDBG và đưa ra các đề xuất. Dựa vào đầu ra `Suggested Profile(s)`, cấu hình tốt nhất và phù hợp nhất được liệt kê đầu tiên là `WinXPSP2x86`
![Pasted image 20260910111111.png](./images/Pasted%20image%2020260910111111.png)

>WinXPSP2x86

## Q2: How many processes were running when the image was acquired?

**Yêu cầu:** Có bao nhiêu tiến trình (processes) đang chạy tại thời điểm thu thập bộ nhớ?

Tiếp theo, để liệt kê các tiến trình đang hoạt động trên hệ thống khi bộ nhớ được kết xuất, ta sử dụng plugin `pslist`. Bằng cách đếm tổng số dòng tiến trình được in ra trong danh sách, ta có thể xác định được số lượng tiến trình đang chạy. Kết quả ghi nhận là 19 tiến trình
![Pasted image 20260910111714.png](./images/Pasted%20image%2020260910111714.png)

>19

## Q3: What is the process ID of `cmd.exe`?

**Yêu cầu:** Process ID (PID) của tiến trình `cmd.exe` là gì?

Kiểm tra kỹ hơn trong danh sách tiến trình vừa trích xuất từ lệnh `pslist`, ta dò tìm cột Name để định vị `cmd.exe`. Đối chiếu sang cột PID tương ứng, ta tìm được Process ID của nó
![Pasted image 20260910111745.png](./images/Pasted%20image%2020260910111745.png)

>1960

## Q4: What is the name of the most suspicious process?

**Yêu cầu:** Tên của tiến trình đáng ngờ nhất là gì?

Quét qua toàn bộ danh sách các tiến trình hệ thống hợp lệ thông thường, ta nhanh chóng nhận thấy một cái tên vô cùng bất thường và độc hại ngay từ tên gọi. Tiến trình này được đặt tên là `rootkit.exe`, đây rõ ràng là một dấu hiệu của malware
![Pasted image 20260910112036.png](./images/Pasted%20image%2020260910112036.png)

>rootkit.exe

## Q5: Which process shows the highest likelihood of code injection?

**Yêu cầu:** Tiến trình nào có khả năng bị chèn mã (code injection) cao nhất?

Để phát hiện các hành vi chèn mã độc vào bộ nhớ (như DLL injection hoặc hollowing), ta sử dụng plugin `malfind`. Plugin này sẽ rà soát các vùng nhớ có đặc quyền thực thi và đọc/ghi (`PAGE_EXECUTE_READWRITE`) nhưng không được map vào một tệp hợp lệ trên ổ đĩa. Kết quả cho thấy tiến trình `svchost.exe` có các đặc điểm này, chứng tỏ nó đã bị attacker lạm dụng để chèn mã
![Pasted image 20260910114607.png](./images/Pasted%20image%2020260910114607.png)
>svchost.exe

## Q6: There is an odd file referenced in the recent process. Provide the full path of that file.

**Yêu cầu:** Có một tập tin kỳ lạ được tham chiếu trong tiến trình vừa phát hiện. Hãy cung cấp đường dẫn đầy đủ của tệp đó.

Để kiểm tra các tài nguyên đang được tiến trình `svchost.exe` (PID 880) mở và thao tác, ta dùng lệnh `handles -p 880 -t file`. Trong danh sách các Handles trả về, có một đường dẫn trỏ tới tệp `.sys` (driver) trông rất đáng ngờ nằm trong thư mục system32. Tên của tệp này là `str.sys`
![Pasted image 20260910113734.png](./images/Pasted%20image%2020260910113734.png)

>C:\WINDOWS\system32\drivers\str.sys

## Q7: What is the name of the injected DLL file loaded from the recent process?

**Yêu cầu:** Tên của tập tin DLL bị chèn (injected) được load từ tiến trình vừa phân tích là gì?

Để phân tích sâu hơn về các module (DLL) được load ngầm, ta sử dụng plugin `ldrmodules`. Plugin này giúp phát hiện các DLL đã bị giấu khỏi danh sách module liên kết kép tiêu chuẩn của hệ điều hành. Lọc dữ liệu theo tiến trình `svchost.exe`, ta phát hiện tập tin `msxml3r.dll`. Điều bất thường là cả ba trường `InLoad`, `InInit`, và `InMem` của nó đều mang giá trị `False`, một minh chứng kinh điển cho thấy DLL này đã được nạp một cách lén lút (injected)
![Pasted image 20260910114311.png](./images/Pasted%20image%2020260910114311.png)

>msxml3r.dll

## Q8: What is the base address of the injected DLL?

**Yêu cầu:** Địa chỉ nền (base address) của DLL bị chèn là gì?

Cuối cùng, để tìm địa chỉ nền của đoạn mã độc đã được nạp vào bộ nhớ của `svchost.exe`, ta quay lại xem xét kết quả đầu ra của plugin `malfind` ở câu Q5. Plugin này chỉ định rõ vị trí vùng nhớ bị nhiễm mã độc thông qua trường `Address`
![Pasted image 20260910114629.png](./images/Pasted%20image%2020260910114629.png)

>0x980000 