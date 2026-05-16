# Lab: Web Server Log Analysis with CLI Tools

## Learning Outcomes
By completing this lab, you will be able to:
* Understand the structure of an Apache Web Server Combined Log Format.
* Use core Linux command-line tools (`grep`, `awk`, `sort`, `uniq`) to parse and extract data.
* Identify the most active IP addresses and analyze traffic patterns.
* Spot web-based attacks, including Directory Traversal, SQL Injection (SQLi), and automated scanning.
* Perform advanced data extraction to isolate potential brute-force attempts.

## Objective
Use standard Linux terminal commands to parse a web server log file, identify suspicious behavior, isolate specific attack vectors, and trace the attacker's origin.

## Scenario
You are acting as a junior SOC analyst monitoring the infrastructure for **ThreatForge Academy**, an online offensive security training platform. The server has been experiencing performance degradation, and the engineering team suspects malicious actors are running automated vulnerability scanners and attempting to breach the login portal. 

You have been provided with the Apache `access.log` file from the Debian web server. Your task is to analyze the logs via the command line, identify the malicious activity, and generate an incident report.

## Prerequisites
* A Linux environment (Debian, Kali Linux, or Ubuntu VM)
* A terminal emulator
* Basic knowledge of Linux file system navigation

---

## Part 1: Initializing the Investigation

1. **Create an investigation directory:**
   ```bash
   mkdir threatforge-investigation
   cd threatforge-investigation
Download the sample log file:
(In a production environment, logs are typically located in /var/log/apache2/ or /var/log/httpd/)

 ```bash
wget [https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs](https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs) -O access.log
 ```
Verify the file format:

 ```bash
head -n 3 access.log
 ```
Note: The logs follow the Combined Log Format. Key fields include: Column 1 (IP Address), Column 4 (Timestamp), Column 6 (HTTP Method), Column 7 (Requested URL), Column 9 (Status Code), and Column 12+ (User-Agent).

Part 2: Baseline Traffic Analysis
Let's establish a baseline of the traffic before hunting for specific threats.

Exercise 1: Total Request Volume
Goal: Determine the overall size of the log file to understand the traffic volume.
Command:

 ```bash
wc -l access.log
 ```
Explanation: wc -l counts the total number of lines. Each line represents one HTTP request.

Exercise 2: Top 5 Most Active IP Addresses
Goal: Identify the sources generating the most traffic. Disproportionately high requests from a single IP often indicate bots, scrapers, or brute-force tools.
Command:

 ```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -n 5
 ```
Explanation:

awk '{print $1}': Extracts the first column (IP addresses).

sort: Groups identical IP addresses together.

uniq -c: Counts the consecutive occurrences of each IP.

sort -rn: Sorts the output in reverse numerical order (highest count first).

head -n 5: Limits the output to the top 5 lines.

Exercise 3: High-Volume 404 Errors
Goal: Automated vulnerability scanners frequently search for hidden directories or vulnerable files, generating numerous "404 Not Found" errors.
Command:

 ```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -rn | head -n 5
 ```
Explanation: This uses awk to look specifically at the 9th column (status code). If it equals 404, it prints the 7th column (the URL requested), then sorts and counts the most frequently requested missing files.

Part 3: Threat Hunting and Attack Signatures
Now we will search for specific attack payloads within the HTTP requests.

Exercise 4: Detecting SQL Injection (SQLi)
Goal: Attackers often use SQL keywords in the URL to manipulate backend databases.
Command:

 ```bash
grep -iE "(UNION SELECT|DROP TABLE|1=1)" access.log
 ```
Explanation: grep -iE allows for a case-insensitive, extended regular expression search, matching multiple common SQL injection patterns at once.

Exercise 5: Detecting Directory Traversal
Goal: Attackers use ../ to navigate outside the intended web root directory to read sensitive files like /etc/passwd.
Command:

 ```bash
grep -E "(\.\./|\.\.%2F)" access.log
 ```
Explanation: This searches for both standard ../ and its URL-encoded equivalent ..%2F.

Part 4: Advanced Analysis
Let's dig deeper to isolate specific attacker behaviors.

Exercise 6: Isolating Login Brute-Force Attempts
Goal: Find IP addresses that are repeatedly sending POST requests (the HTTP method used for submitting login credentials).
Command:

 ```bash
awk '$6 == "\"POST"' access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -n 3
 ```
Explanation: The first awk command filters out any line that isn't a POST request. The subsequent commands extract and count the IP addresses making those POST requests.

Exercise 7: Identifying Malicious User-Agents
Goal: Sometimes attackers forget to spoof their User-Agent, leaving behind the default name of the hacking tool they are using (like Nikto, Nmap, or DirBuster).
Command:

 ```bash
grep -i "nikto" access.log | awk '{print $1}' | sort -u
 ```
Explanation: This searches the logs for the popular vulnerability scanner "Nikto" and outputs a clean, deduplicated list (sort -u) of the attacking IP addresses.
