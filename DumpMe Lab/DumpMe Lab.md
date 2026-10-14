author: Azaki
published: published
created: 2026-10-04
tags: CyberDefenders, Lab, Memory Forensics

A SOC analyst took a memory dump from a machine infected with a meterpreter malware. As a Digital Forensicators, your job is to analyze the dump, extract the available indicators of compromise (IOCs) and answer the provided questions.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Champlain College</b> 
</p>

## Q1: What is the SHA1 hash of Triage-Memory.mem (memory dump)?

**Yêu cầu:** Tính toán giá trị băm SHA1 của tệp memory dump Triage-Memory.mem

Trong quy trình điều tra kỹ thuật số việc đầu tiên cần thực hiện trước khi phân tích là tính toán giá trị băm mật mã của tệp ảnh bộ nhớ nhằm bảo toàn tính toàn vẹn của bằng chứng phục vụ Chain of Custody Thực thi công cụ sha1sum trên môi trường phân tích Linux giá trị băm SHA1 của tệp Triage-Memory.mem được ghi nhận chính xác
![DumpMe Lab.png](./images/DumpMe%20Lab.png)

>c95e8cc8c946f95a109ea8e47a6800de10a27abd

## Q2: What volatility profile is the most appropriate for this machine? (ex: Win10x86_14393)

**Yêu cầu:** Xác định profile Volatility phù hợp nhất cho tệp dump bộ nhớ của hệ thống

Framework Volatility yêu cầu xác định chính xác cấu trúc kernel và kiến trúc hệ điều hành thông qua việc quét cấu trúc KDBG Kernel Debugger Data Block Sử dụng plugin imageinfo để phân tích metadata của tệp ảnh bộ nhớ profile được gợi ý hàng đầu và chuẩn xác nhất cho phiên bản hệ điều hành này là Win7SP1x64
![DumpMe Lab-1.png](./images/DumpMe%20Lab-1.png)

>Win7SP1x64

## Q3: What was the process ID of notepad.exe?

**Yêu cầu:** Xác định mã định danh tiến trình PID của ứng dụng notepad.exe

Hệ điều hành Windows quản lý các tiến trình đang thực thi thông qua danh sách liên kết đôi ActiveProcessLinks trong cấu trúc EPROCESS Sử dụng plugin pslist để duyệt danh sách tiến trình hoạt động tại thời điểm trích xuất bộ nhớ tiến trình notepad.exe được xác định đang chạy với mã định danh PID là 3032
![DumpMe Lab-2.png](./images/DumpMe%20Lab-2.png)

>3032

## Q4: Name the child process of wscript.exe.

**Yêu cầu:** Xác định tên tiến trình con được khởi tạo bởi wscript.exe

Kẻ tấn công thường lợi dụng Windows Script Host wscript.exe để chạy script độc hại và spawn tiến trình thực thi giai đoạn tiếp theo Kiểm tra bằng plugin pslist tôi ghi nhận wscript.exe hoạt động với PID 5116 Sau đó áp dụng plugin pstree để phân tích cấu trúc cây phân cấp tiến trình cha con tiến trình con trực tiếp được tạo ra là UWkpjFjDzM.exe
![DumpMe Lab-3.png](./images/DumpMe%20Lab-3.png)

![DumpMe Lab-4.png](./images/DumpMe%20Lab-4.png)

>UWkpjFjDzM.exe

## Q5: What was the IP address of the machine at the time the RAM dump was created?

**Yêu cầu:** Xác định địa chỉ IP cục bộ của máy nạn nhân tại thời điểm chụp ảnh bộ nhớ RAM

Dấu vết kết nối mạng được lưu giữ trong các cấu trúc socket và endpoint giao tiếp của nhân Windows Sử dụng plugin netscan để quét các cấu trúc mạng TCP và UDP đang hoạt động kiểm tra trường Local Address từ kết quả trích xuất địa chỉ IP cục bộ của máy nạn nhân là 10.0.0.101
![DumpMe Lab-5.png](./images/DumpMe%20Lab-5.png)

>10.0.0.101

## Q6: Based on the answer regarding the infected PID, can you determine the IP of the attacker?

**Yêu cầu:** Dựa trên PID của tiến trình bị lây nhiễm xác định địa chỉ IP của kẻ tấn công

Sau khi đã xác định tiến trình độc hại UWkpjFjDzM.exe mang PID 3496 tôi kết hợp lọc dữ liệu kết nối mạng từ plugin netscan theo PID này Kết quả phân tích hiển thị một kết nối trạng thái ESTABLISHED đang liên lạc tới cổng 4444 cổng mặc định đặc trưng của payload Metasploit với địa chỉ Foreign Address của kẻ tấn công là 10.0.0.106
![DumpMe Lab-6.png](./images/DumpMe%20Lab-6.png)

>10.0.0.106

## Q7: How many processes are associated with VCRUNTIME140.dll?

**Yêu cầu:** Xác định số lượng tiến trình có liên kết với thư viện VCRUNTIME140.dll

Khi sử dụng plugin dlllist để tra cứu danh sách liên kết PEB InLoadOrderModuleList kết quả chỉ hiển thị duy nhất một tiến trình do cơ chế che giấu hoặc tải thư viện động không đăng ký chuẩn trong PEB Để kiểm tra toàn diện tôi sử dụng plugin ldrmodules giúp đối chiếu cấu trúc VAD với ba danh sách nạp mô đun trong PEB Kết quả quét VAD phát hiện có tổng cộng 5 tiến trình đang ánh xạ thư viện VCRUNTIME140.dll
![DumpMe Lab-7.png](./images/DumpMe%20Lab-7.png)

![DumpMe Lab-8.png](./images/DumpMe%20Lab-8.png)

>5

## Q8: After dumping the infected process, what is its md5 hash?

**Yêu cầu:** Trích xuất tiến trình độc hại ra khỏi bộ nhớ và tính toán giá trị băm MD5 của nó

Để phục vụ công tác điều tra và đảo ngược mã độc tiến trình độc hại PID 3496 cần được trích xuất an toàn từ bộ nhớ về đĩa cứng Sử dụng plugin procdump để dump file thực thi executable.3496.exe từ không gian tiến trình sau đó thực thi lệnh md5sum để tính toán chữ ký băm giá trị thu được là 690ea20bc3bdfb328e23005d9a80c290
![DumpMe Lab-9.png](./images/DumpMe%20Lab-9.png)

>690ea20bc3bdfb328e23005d9a80c290

## Q9: What is the LM hash of Bob's account?

**Yêu cầu:** Xác định giá trị băm LM của tài khoản người dùng Bob

Hệ điều hành Windows lưu giữ mật khẩu băm của các tài khoản cục bộ trong hive SAM của registry được nạp vào bộ nhớ Sử dụng plugin hashdump để trích xuất khóa SYSKEY và giải mã cấu trúc băm tài khoản kết quả phân tích hiển thị tài khoản Bob lưu giữ giá trị LM hash rỗng chuẩn mặc định là aad3b435b51404eeaad3b435b51404ee
![DumpMe Lab-10.png](./images/DumpMe%20Lab-10.png)

>aad3b435b51404eeaad3b435b51404ee

## Q10: What memory protection constants does the VAD node at 0xfffffa800577ba10 have?

**Yêu cầu:** Xác định hằng số bảo vệ bộ nhớ của nút VAD tại địa chỉ 0xfffffa800577ba10

Bộ quản lý bộ nhớ của Windows sử dụng cây nhị phân Virtual Address Descriptor VAD để theo dõi các vùng bộ nhớ ảo được cấp phát cho tiến trình Áp dụng plugin vadinfo để duyệt chi tiết các nút VAD và lọc theo địa chỉ nút 0xfffffa800577ba10 trường thuộc tính Protection ghi nhận hằng số bảo vệ của vùng nhớ này là PAGE_READONLY
![DumpMe Lab-11.png](./images/DumpMe%20Lab-11.png)

>PAGE_READONLY

## Q11: What memory protection did the VAD starting at 0x00000000033c0000 and ending at 0x00000000033dffff have?

**Yêu cầu:** Xác định thuộc tính bảo vệ bộ nhớ của vùng VAD có dải địa chỉ từ 0x00000000033c0000 đến 0x00000000033dffff

Việc kiểm tra thuộc tính bảo vệ của từng dải địa chỉ bộ nhớ ảo cụ thể cho phép đánh giá quyền thực thi đọc ghi của các vùng dữ liệu Thực thi plugin vadinfo và lọc theo khoảng địa chỉ từ 0x00000000033c0000 đến 0x00000000033dffff quyền hạn bảo vệ bộ nhớ được thiết lập là PAGE_NOACCESS
![DumpMe Lab-12.png](./images/DumpMe%20Lab-12.png)

>PAGE_NOACCESS

## Q12: There was a VBS script that ran on the machine. What is the name of the script? (submit without file extension)

**Yêu cầu:** Xác định tên tệp kịch bản VBS đã được thực thi trên máy nạn nhân

Chuỗi tham số dòng lệnh thực thi tiến trình được lưu trữ trong cấu trúc Process Environment Block PEB của không gian bộ nhớ tiến trình Sử dụng plugin cmdline để trích xuất câu lệnh khởi chạy wscript.exe kết quả phân tích chỉ ra tệp script VBS được thực thi trong thư mục tạm có tên là vhjReUDEuumrX
![DumpMe Lab-13.png](./images/DumpMe%20Lab-13.png)

>vhjReUDEuumrX

## Q13: An application was run at 2019-03-07 23:06:58 UTC. What is the name of the program? (Include extension)

**Yêu cầu:** Xác định tên ứng dụng đã chạy vào thời điểm 2019-03-07 23:06:58 UTC

Cơ chế Application Compatibility Cache ShimCache theo dõi dấu vết thực thi của các tệp nhị phân trên hệ thống Windows nhằm hỗ trợ khả năng tương thích ứng dụng Sử dụng plugin shimcache để phân tích các mục lưu vết và lọc theo mốc thời gian 2019-03-07 23:06:58 UTC chương trình tương ứng được định danh là Skype.exe
![DumpMe Lab-14.png](./images/DumpMe%20Lab-14.png)

>Skype.exe

## Q14: What was written in notepad.exe at the time when the memory dump was captured?

**Yêu cầu:** Xác định nội dung văn bản đang được mở trong notepad.exe tại thời điểm thu thập bộ nhớ

Để khôi phục chuỗi văn bản đang hiển thị trong bộ nhớ của tiến trình notepad.exe PID 3032 tôi sử dụng plugin memdump để dump toàn bộ không gian bộ nhớ của tiến trình thành tệp 3032.dmp Sau đó áp dụng lệnh strings quét bảng mã Unicode kết hợp biểu thức chính quy để trích xuất cờ nội dung thu được là flag<REDBULL_IS_LIFE>
![DumpMe Lab-15.png](./images/DumpMe%20Lab-15.png)

![DumpMe Lab-16.png](./images/DumpMe%20Lab-16.png)

>flag<REDBULL_IS_LIFE>

## Q15: What is the short name of the file at file record 59045?

**Yêu cầu:** Xác định tên ngắn short name dạng 8.3 của tệp tại bản ghi MFT record số 59045

Bảng Master File Table MFT trên phân vùng NTFS lưu trữ cấu trúc thuộc tính của mọi tệp tin trong đó thuộc tính $FILE_NAME lưu cả tên dài chuẩn lẫn tên ngắn định dạng 8.3 tương thích DOS Sử dụng plugin mftparser để phân tích các bản ghi MFT và lọc bản ghi số hiệu 59045 tên ngắn được lưu trong thuộc tính $FILE_NAME là EMPLOY~1.XLS
![DumpMe Lab-17.png](./images/DumpMe%20Lab-17.png)

![DumpMe Lab-18.png](./images/DumpMe%20Lab-18.png)

>EMPLOY~1.XLS

## Q16: This box was exploited and is running meterpreter. What was the infected PID?

**Yêu cầu:** Xác định PID của tiến trình bị khai thác và đang chạy meterpreter

Tổng hợp phân tích cây tiến trình từ plugin pstree và kết nối mạng từ plugin netscan tiến trình lạ UWkpjFjDzM.exe được sinh ra bởi wscript.exe và đang duy trì kết nối Meterpreter đến máy chủ tấn công qua cổng 4444 Mã định danh PID của tiến trình độc hại này là 3496
![DumpMe Lab-4.png](./images/DumpMe%20Lab-4.png)

>3496