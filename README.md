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
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log
   

Verify the file format:
head -n 3 access.log
