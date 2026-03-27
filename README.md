# CD-Hawkeye-Writeup

Description:'''Reconstruct a HawkEye Keylogger data exfiltration incident by analyzing network traffic with Wireshark and CyberChef, identifying IoCs and stolen credentials.'''

We are given a pcap file to investigate:

<img width="2554" height="1071" alt="image" src="https://github.com/user-attachments/assets/11b3d123-5b9a-4dae-9264-d9d61b8bdea0" />

1. How many packets does the capture have?
  Check the lower right corner.
2. At what time was the first packet captured (UTC)?
Go to View- Time Display format and set UTC.
3. What is the duration of the capture?
Statistics - Capture file properties.

<img width="504" height="126" alt="image" src="https://github.com/user-attachments/assets/ba73b48c-1b1a-4a03-ad87-d14f9318599e" />

4. What is the most active computer at the link level?

Statistics - Endpoints

<img width="1033" height="361" alt="image" src="https://github.com/user-attachments/assets/3c2e0f9f-39d3-4d6d-8a75-0e54bdb09a4b" />

5. Manufacturer of the NIC of the most active system at the link level?

This is the first 24 bits of the MAC, I used an online website for lookup:

<img width="893" height="550" alt="image" src="https://github.com/user-attachments/assets/f88a07be-e419-4397-8399-e752d231b29c" />

6. Where is the headquarter of the company that manufactured the NIC of the most active computer at the link level?

Use google, its Palo Alto.

7. The organization works with private addressing and netmask /24. How many computers in the organization are involved in the capture?

Again, this is in Endpoints:

<img width="584" height="143" alt="image" src="https://github.com/user-attachments/assets/0de63470-e146-41ee-a232-ae350cad7fc2" />

8. What is the name of the most active computer at the network level?

I first filtered for that IP to see what traffic we have but it was obviosly too much so I started with dhcp and there are only 2 packets, filtering for DHCP Inform:

<img width="732" height="109" alt="image" src="https://github.com/user-attachments/assets/f26e21de-be20-40f2-bbd0-15eae2dbf78c" />

9. What is the IP of the organization's DNS server?

Simply filter for dns and see how is responding:

<img width="1754" height="176" alt="image" src="https://github.com/user-attachments/assets/2090ad75-2778-4da5-8606-d269b4fb3fc6" />

10. What domain is the victim asking about in packet 204?

Wireshark - Go - Go to packet 204:

<img width="791" height="26" alt="image" src="https://github.com/user-attachments/assets/a5c3d0d7-d0f6-4ca8-a1f3-05fe620736ab" />

11. What is the IP of the domain in the previous question?

Go 2 packets further to see the DNS response:

<img width="863" height="33" alt="image" src="https://github.com/user-attachments/assets/fd4fa207-e038-4132-9adb-6a3e7c26a86a" />

12. Indicate the country to which the IP in the previous section belongs.

<img width="528" height="455" alt="image" src="https://github.com/user-attachments/assets/feea4978-87fe-4ed7-bce2-ef44873606a6" />

13. What operating system does the victim's computer run?

As we already know victim is .132 address we don't even need to filter as right after that DNS starts a long HTTP connection and we can see the headers victim sends:

<img width="247" height="99" alt="image" src="https://github.com/user-attachments/assets/759bb51d-c76e-49c8-9064-20d244a4d1e3" />

14. What is the name of the malicious file downloaded by the accountant?

Again, this is in the HTTP:

<img width="1004" height="105" alt="image" src="https://github.com/user-attachments/assets/cce048fe-da02-4afd-981d-b77698907f93" />

15. What is the md5 hash of the downloaded file?

The fastest way is to export the file and upload to VT to get an understanding of what malware that is:

<img width="2028" height="712" alt="image" src="https://github.com/user-attachments/assets/9cbc0025-eca6-4292-bb20-ba339d488ed1" />

16. What software runs the webserver that hosts the malware?

<img width="215" height="298" alt="image" src="https://github.com/user-attachments/assets/55333e0a-e19a-4255-b573-f6f9e71d890f" />

17. What is the public IP of the victim's computer?

