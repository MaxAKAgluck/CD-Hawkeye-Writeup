# HawkEye Keylogger — Network Forensics Writeup

> **Challenge:** Reconstruct a HawkEye Keylogger data exfiltration incident by analyzing network traffic with Wireshark and CyberChef, identifying IoCs and stolen credentials.
> https://cyberdefenders.org/blueteam-ctf-challenges/hawkeye/

---

## Overview

We are given a pcap file to investigate:

<img width="2554" height="1071" alt="image" src="https://github.com/user-attachments/assets/11b3d123-5b9a-4dae-9264-d9d61b8bdea0" />

---

## Questions & Analysis

### 1. How many packets does the capture have?

Check the lower right corner.

### 2. At what time was the first packet captured (UTC)?

Go to **View → Time Display Format** and set UTC.

### 3. What is the duration of the capture?

**Statistics → Capture File Properties.**

<img width="504" height="126" alt="image" src="https://github.com/user-attachments/assets/ba73b48c-1b1a-4a03-ad87-d14f9318599e" />

### 4. What is the most active computer at the link level?

**Statistics → Endpoints.**

<img width="1033" height="361" alt="image" src="https://github.com/user-attachments/assets/3c2e0f9f-39d3-4d6d-8a75-0e54bdb09a4b" />

### 5. Manufacturer of the NIC of the most active system at the link level?

This is the first 24 bits of the MAC address. I used an online OUI lookup website:

<img width="893" height="550" alt="image" src="https://github.com/user-attachments/assets/f88a07be-e419-4397-8399-e752d231b29c" />

### 6. Where is the headquarters of the company that manufactured the NIC of the most active computer at the link level?

Use Google — it's Palo Alto.

### 7. The organization works with private addressing and netmask /24. How many computers in the organization are involved in the capture?

Again, this is in **Endpoints**:

<img width="584" height="143" alt="image" src="https://github.com/user-attachments/assets/0de63470-e146-41ee-a232-ae350cad7fc2" />

### 8. What is the name of the most active computer at the network level?

I first filtered for that IP to see what traffic we had, but it was obviously too much, so I started with DHCP — there are only 2 packets. Filtering for **DHCP Inform**:

<img width="732" height="109" alt="image" src="https://github.com/user-attachments/assets/f26e21de-be20-40f2-bbd0-15eae2dbf78c" />

### 9. What is the IP of the organization's DNS server?

Simply filter for `dns` and see who is responding:

<img width="1754" height="176" alt="image" src="https://github.com/user-attachments/assets/2090ad75-2778-4da5-8606-d269b4fb3fc6" />

### 10. What domain is the victim asking about in packet 204?

**Wireshark → Go → Go to Packet 204:**

<img width="791" height="26" alt="image" src="https://github.com/user-attachments/assets/a5c3d0d7-d0f6-4ca8-a1f3-05fe620736ab" />

### 11. What is the IP of the domain in the previous question?

Go 2 packets further to see the DNS response:

<img width="863" height="33" alt="image" src="https://github.com/user-attachments/assets/fd4fa207-e038-4132-9adb-6a3e7c26a86a" />

### 12. Indicate the country to which the IP in the previous section belongs.

<img width="528" height="455" alt="image" src="https://github.com/user-attachments/assets/feea4978-87fe-4ed7-bce2-ef44873606a6" />

### 13. What operating system does the victim's computer run?

As we already know the victim is the `.132` address, we don't even need to filter — right after that DNS exchange, a long HTTP connection starts and we can see the headers the victim sends:

<img width="247" height="99" alt="image" src="https://github.com/user-attachments/assets/759bb51d-c76e-49c8-9064-20d244a4d1e3" />

### 14. What is the name of the malicious file downloaded by the accountant?

Again, this is in the HTTP stream:

<img width="1004" height="105" alt="image" src="https://github.com/user-attachments/assets/cce048fe-da02-4afd-981d-b77698907f93" />

### 15. What is the MD5 hash of the downloaded file?

The fastest way is to export the file and upload it to VirusTotal to get an understanding of what malware it is:

<img width="2028" height="712" alt="image" src="https://github.com/user-attachments/assets/9cbc0025-eca6-4292-bb20-ba339d488ed1" />

### 16. What software runs the web server that hosts the malware?

<img width="215" height="298" alt="image" src="https://github.com/user-attachments/assets/55333e0a-e19a-4255-b573-f6f9e71d890f" />

### 17. What is the public IP of the victim's computer?

My first thought was NAT, so I filtered for the `nat-pmp` protocol — but this capture doesn't contain it, meaning we're seeing public IP → private IP pairs. I then thought about protocols that might leak this IP, with no luck either. Then I checked what happened after the victim finished the HTTP download of the malware — and bingo:

<img width="1590" height="312" alt="image" src="https://github.com/user-attachments/assets/e683a5aa-c95c-4b7c-a895-37adf8156e38" />

The GET request returned the IP:

<img width="763" height="335" alt="image" src="https://github.com/user-attachments/assets/6f4bbc9d-6cad-414c-98c8-435fd3305d62" />

### 18. In which country is the email server to which the stolen information is sent?

I quickly spotted the email server's IP and domain in the SMTP traffic:

<img width="1181" height="57" alt="image" src="https://github.com/user-attachments/assets/a1390203-0cfc-4197-8ee4-6bfda376030e" />

After using an IP lookup tool, the country showed "Belgium", whereas the answer field expects 2 words. I then used another service and the majority returned "United States", so I guess these tools can sometimes present very different results:

<img width="1108" height="1133" alt="image" src="https://github.com/user-attachments/assets/81033180-01f8-478d-8a6b-6ed3216a9aa7" />

### 19. Analyzing the first extraction of information — what software runs the email server to which the stolen data is sent?

This can be found by following the TCP stream of the SMTP packets:

<img width="198" height="120" alt="image" src="https://github.com/user-attachments/assets/c4940360-8975-429c-869a-8a1d04c1f32d" />

### 20. To which email account is the stolen information sent?

Again, same method:

<img width="762" height="261" alt="image" src="https://github.com/user-attachments/assets/0980efd7-685b-4d1c-aa6d-e7aa391af28b" />

Login details encapsulated in Base64 give us the [sales.del@macwinlogistics.in](mailto:sales.del@macwinlogistics.in) address.

### 21. What is the password used by the malware to send the email?

The password input appears right after the login details — Base64 `U2FsZXNAMjM=` decodes to `Sales@23` in plain text.

### 22. Which malware variant exfiltrated the data?

I checked VirusTotal again and confirmed this is HawkEye Reborn, but the question was: which specific answer does the challenge site expect? After a bit more research I found there are multiple versions labeled v8 and v9. This Talos report is a great reference: https://blog.talosintelligence.com/hawkeye-reborn/ — after cross-referencing the article with the timestamp of the capture, the answer is **Reborn v9**.

### 23. What are the Bank of America access credentials? (username:password)

I searched for `frame contains bankofamerica` — no result. I then realized there's a large amount of SMTP data, all in Base64, which makes manual inspection impractical. I delegated this to GPT and got a script in about 1.5 minutes that neatly extracted the data and actually revealed the answer to the previous question as well:

<img width="482" height="355" alt="image" src="https://github.com/user-attachments/assets/f923c63d-13ec-49bd-a005-6f0741325956" />

<img width="783" height="323" alt="image" src="https://github.com/user-attachments/assets/2575f071-d1d1-4cbe-a7cd-17b47c9e6caf" />

### 24. Every how many minutes does the collected data get exfiltrated?

There are 7 conversations in total:

<img width="376" height="357" alt="image" src="https://github.com/user-attachments/assets/01a9e408-a41d-4292-8a8f-8ac0d0c36ab1" />

By checking the timestamps at the beginning of each conversation, we can deduce the interval is **10 minutes**.

---

## Conclusion

Overall, I really enjoyed this lab. Some questions were a little self-repeating, and judging by the first part I thought this was an easy, not a medium, challenge — but then it got interesting. It was a great experience to remind yourself how to navigate Wireshark and make use of some threat intelligence tooling.
