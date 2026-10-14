author: Azaki
published: published
created: 2025-12-31
tags: CyberDefener, Lab, Threat Intel

You are part of the Threat Intelligence team in the SOC (Security Operations Center). An executable file has been discovered on a colleague's computer, and it's suspected to be linked to a Command and Control (C2) server, indicating a potential malware infection.
Your task is to investigate this executable by analyzing its hash. The goal is to gather and analyze data beneficial to other SOC members, including the Incident Response team, to respond to this suspicious behavior efficiently.

<hr>
<p style="text-align:right; font-style:italic;">
  Author: <b>@Sameer_Fakhoury</b> 
</p>

## Q1: Categorizing malware enables a quicker and clearer understanding of its unique behaviors and attack vectors. What category has Microsoft identified for that malware in VirusTotal?

**Yêu cầu:** Xác định category mà Microsoft sử dụng để phân loại mẫu malware trên VirusTotal.

Bắt đầu bằng SHA256 của mẫu malware và tra cứu trên **VirusTotal**. Trong phần kết quả của các security vendor, Microsoft nhận diện mẫu dưới dạng `Trojan:Win32/Redline...`.

Ở đây cần phân biệt giữa **category** và **malware family**: câu hỏi chỉ yêu cầu loại malware, vì vậy phần cần lấy là `Trojan`
![Pasted image 20260816120359.png](./images/Pasted%20image%2020260816120359.png)
>Trojan

## Q2: Clearly identifying the name of the malware file improves communication among the SOC team. What is the file name associated with this malware?

**Yêu cầu:** Xác định tên file được liên kết với mẫu malware.

Tiếp tục kiểm tra thông tin của mẫu trên VirusTotal. Trong phần metadata có thể thấy tên liên quan đến executable là `WEXTRACT.EXE.MUI`.

Do câu hỏi của lab chỉ yêu cầu **file name** và đáp án không bao gồm phần mở rộng, giá trị cần điền là `Wextract`
![Pasted image 20260816120403.png](./images/Pasted%20image%2020260816120403.png)

>Wextract

## Q3: Knowing the exact timestamp of when the malware was first observed can help prioritize response actions. Newly detected malware may require urgent containment and eradication compared to older, well-documented threats. What is the UTC timestamp of the malware's first submission to VirusTotal?

**Yêu cầu:** Xác định thời điểm mẫu được submit lên VirusTotal lần đầu tiên theo UTC.

Trong tab **Details**, kiểm tra phần **History** và tìm trường `First Submission`.

Ảnh kết quả cho thấy timestamp đầy đủ là:
```
2023-10-06 04:41:50 UTC
```

Lab sử dụng định dạng đến phút, vì vậy đáp án được điền là `2023-10-06 04:41`.
![Pasted image 20260816120410.png](./images/Pasted%20image%2020260816120410.png)

>2023-10-06 04:41

## Q4: Understanding the techniques used by malware helps in strategic security planning. What is the MITRE ATT&CK technique ID for the malware's data collection from the system before exfiltration?

**Yêu cầu:** Xác định MITRE ATT&CK Technique ID mô tả hành vi thu thập dữ liệu từ chính hệ thống trước khi exfiltration.

Trong phần mapping **MITRE ATT&CK** của kết quả phân tích, dưới tactic **Collection** xuất hiện technique **Data from Local System**.

Technique này mô tả việc adversary tìm và thu thập dữ liệu từ các nguồn cục bộ như file system, configuration file, local database hoặc process memory trước khi dữ liệu được đưa ra ngoài hệ thống.
![Pasted image 20260816120415.png](./images/Pasted%20image%2020260816120415.png)

>T1005

## Q5: Following execution, which social media-related domain names did the malware resolve via DNS queries?

**Yêu cầu:** Xác định domain liên quan đến mạng xã hội mà mẫu malware đã thực hiện DNS resolution sau khi chạy.

Trong phần network/domain activity của VirusTotal có nhiều domain hợp lệ được resolve, bao gồm các dịch vụ của Google, Microsoft và Facebook.

Domain mạng xã hội phù hợp với yêu cầu của câu hỏi là `facebook.com`.
![Pasted image 20260816120419.png](./images/Pasted%20image%2020260816120419.png)

Cần lưu ý rằng việc malware resolve một domain hợp lệ **không đồng nghĩa domain đó là C2 hoặc là domain độc hại**. Ở câu này mình chỉ xác định domain mạng xã hội xuất hiện trong DNS activity

>facebook.com

## Q6: Once the malicious IP addresses are identified, network security devices such as firewalls can be configured to block traffic to and from these addresses. Can you provide the IP address and destination port the malware communicates with?

**Yêu cầu:** Xác định địa chỉ IP và destination port của hạ tầng C2 mà malware liên lạc tới.

Kiểm tra phần behavioral/network indicators của mẫu. Trong **Memory Pattern URLs** xuất hiện cả URL HTTP và TCP socket tới cùng một địa chỉ:
```
http://77.91.124.55:19071
tcp://77.91.124.55:19071
```

![Pasted image 20260816120423.png](./images/Pasted%20image%2020260816120423.png)

>77.91.124.55:19071

## Q7: YARA rules are designed to identify specific malware patterns and behaviors. Using MalwareBazaar, what's the name of the YARA rule created by "Varp0s" that detects the identified malware?

**Yêu cầu:** Xác định tên YARA rule do `Varp0s` tạo và được MalwareBazaar match với mẫu malware.

Tra cứu MalwareBazaar bằng SHA256 của mẫu. Với MalwareBazaar nên sử dụng đúng cú pháp:
```
sha256:248fcc901aff4e4b4c48c91e4d78a939bf681c9a1bc24addc3551b32768f907b
```

![Pasted image 20260816120428.png](./images/Pasted%20image%2020260816120428.png)
![image](https://hackmd.io/_uploads/SyEL6sM4-x.png)
Sau khi mở entry của mẫu, kéo xuống phần **YARA Signatures** và tìm rule có `Author: Varp0s`

>detect_Redline_Stealer

## Q8: Understanding which malware families are targeting the organization helps in strategic security planning for the future and prioritizing resources based on the threat. Can you provide the different malware alias associated with the malicious IP address according to ThreatFox?

**Yêu cầu:** Xác định malware alias mà ThreatFox gán cho hạ tầng C2 đã tìm được ở Q6.

Từ IOC `77.91.124.55:19071`, pivot sang **ThreatFox** và tra cứu theo IOC. Entry lịch sử của IOC này được liên kết với **RedLine Stealer**; malware alias được sử dụng là `RECORDSTEALER`.

Threat Intelligence là dữ liệu thay đổi theo thời gian nên một IOC cũ có thể bị expire hoặc không còn hiển thị trong danh sách mặc định. Vì vậy cần phân biệt giữa việc tìm một IOC đang hoạt động hiện tại và việc tra cứu historical intelligence của IOC trong lab.
![Pasted image 20260816120440.png](./images/Pasted%20image%2020260816120440.png)

Kiểm tra trường **Malware alias** mà ThreatFox sử dụng cho RedLine Stealer
![Pasted image 20260816120447.png](./images/Pasted%20image%2020260816120447.png)

>RECORDSTEALER

## Q9: By identifying the malware's imported DLLs, we can configure security tools to monitor for the loading or unusual usage of these specific DLLs. Can you provide the DLL utilized by the malware for privilege escalation?

**Yêu cầu:** Xác định DLL được malware import có các chức năng liên quan đến privilege/token manipulation.

Quay lại phần **Imports** của mẫu và kiểm tra các DLL được executable sử dụng. Trong `ADVAPI32.dll` có các API đáng chú ý như:
```
AdjustTokenPrivileges
GetTokenInformation
LookupPrivilegeValueA
OpenProcessToken
```

![Pasted image 20260816120452.png](./images/Pasted%20image%2020260816120452.png)

`ADVAPI32.dll` là DLL hợp lệ của Windows, **không phải file do malware tự tạo**.

DLL này cung cấp các API quản lý access token và privilege. Ví dụ, `AdjustTokenPrivileges` có thể enable hoặc disable các privilege đã tồn tại trong access token, trong khi `OpenProcessToken` cho phép mở access token gắn với một process.

Việc sample import các API này phù hợp với hành vi **privilege/token manipulation**. Tuy nhiên, chỉ riêng việc import một DLL hoặc API **chưa đủ để chứng minh malware đã privilege escalation thành công**; muốn kết luận chắc chắn hơn cần quan sát API call hoặc runtime behavior

>ADVAPI32.dll

---

# ATTACK CHAIN

![Red Stealer Lab.png](./images/Red%20Stealer%20Lab.png)