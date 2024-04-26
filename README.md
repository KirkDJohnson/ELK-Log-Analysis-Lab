<h1>ELK Log Analysis</h1>

<br />
<h2>Description</h2>
In this lab, the scenario was there was an alert of possible remtoe access and I was to investigate the logs in ELK. I began with baselining and inventorying the logs, hosts, and connections within the specified timeframe, I reversed the logs to see the events unfold in order. I first uncovered a suspicious PowerShell script masquerading as "Invoice.pdf" with the actual extension ".ps1". Following its execution, various reconnaissance techniques like "whoami", "ipconfig", and "hostname" were employed, indicating the attacker's information gathering. In the following logs, an Invoke-WebRequest downloaded "winPEAS." After researching what this program did I found that it is for privilege escalation. After it was run the attacker altered a registry key to permit elevated MSI downloads. A maliciously crafted "adminshell.msi" was then acquired. Recognizing elevated privileges, I switched to SYSTEM user, revealing the creation of a new user named "backdoor" with administrative privileges. Returning to full logs, another Invoke-WebRequest downloaded "beacon.bat" from the same malicious domain, possibly for Command and Control purposes. The attacker targeted "payroll.server.internal", conducting a successful brute force attack with credentials "bsmith:Password123!" and exfiltrating sensitive data.
<h2>Utility Used</h2>

- <b>Elastic Search/Kibana</b> 


<h2>Environments Used </h2>

- <b>Windows 10

<h2>Lab Overview:</h2>

<p align="center">
<h3>MIITRE ATT&CK: Discovery</h3>
Scenerio/outline for the given lab <br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/ec7d3086-dfa0-49b3-8ebc-b9225a302fe4"  alt="ELK Log Analysis"/>
<br />
<br />
Once connected to the ELK instance, the first thing to do would be to sort the logs for the given time period of the suspected incident to reduce log size and increase search result speed and relevancy<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/78a6f92b-b986-4b46-aa7d-8e066298f234"  alt="ELK Log Analysis"/>
<br />
<br />
Next thing I did was do some basic reconnaissance/baselining. Most the traffic is private except for the IP 84[.]237[.]252[.]156 so I took note of it. Also, for a endpoint user in finance, the top processes being curl and powershell definetely raises redflags. <br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/164f8629-0fcc-496e-821e-e066488c8076"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/9b3c2257-75e2-4d42-a4f4-9f2b023847db"  alt="ELK Log Analysis"/>
<br />
<br />
I then changed date old to newest to retrace the steps of the attacker and filtered with for results for process.command.line to see what commands were used and the first result confirmed the presence of an incdient.<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/9e27d94d-7530-49b1-972f-c2b31ce3a006"  alt="ELK Log Analysis"/>
<br />
<br />
It appears the client was tricked into downloading invoice[.]pdf but was unaware the actual file extenstion was [.]ps1 which is powershell script. In the following logs, commands were input likely by the attacker for reconnaissance efforts such as whoami and ipconfig. These can be benign as an IT admin would be seen using these but given the suspected breach it is likely the threat actor.<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/cdf79fd7-5678-4585-8989-a8b4c4610a9c"  alt="ELK Log Analysis"/>
  <img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/adcc4ec3-5ce3-475d-9342-ff89deccabf3"  alt="ELK Log Analysis"/>
<br />
<br />
Likely a malicious download using Powershell's Invoke-WebRequest from the domain evilparrot[.]thm with the file winPEASany[.exe] renamed as winPEAS[.]exe<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/d857749a-feed-46be-b380-326dc6302b98"  alt="ELK Log Analysis"/>
<br />
<br />
Conducting some research on the program winPEAS[.]exe I found a github page detailing that it is script for privliege escalation.<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/3fe2884a-b20b-4a69-b2b6-03c974999acf"  alt="ELK Log Analysis"/>
<br />
<br />
After the winPEAS[.]exe program was run, the attack changed a registry key "KEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer" to AlwaysInstallElevated. <br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/b5bea594-2ccf-476f-8edc-6d812273be63"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/008cebab-5db1-4578-b5a1-f8d50626dda2"  alt="ELK Log Analysis"/>
<br />
<br />
  <h3>MIITRE ATT&CK: Privilege Escalation</h3>
Now that it is clear that the attacker is moving to Privilege Escalation tactics with tactics such as winPEAS, I decided to also research was AlwaysInstalledElevated means and found:<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/af1bd4a2-7c83-4c9d-a7fb-1843bef3e020"  alt="ELK Log Analysis"/>
<br />
<br />
With my new found knowledge that MSI will likely be leveraged by the attacker I changed the filter to only look for instances of MSI and found some hits. The attacker again used Invoke-WebRequest to download a program from the domain evilparrot[.]thm this program being adminshell[.]msi. (It was downloaded with elevated privileges as mentioned earlier from the "AlwaysInstallElevated" registry change.)<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/a5fdf66c-2479-4232-9499-3d706fb213a3"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/039d3f6b-f5e3-4d17-ad58-130736c33531"  alt="ELK Log Analysis"/>
<br />
<br />
  <h3>MIITRE ATT&CK: Persistence</h3>
Now that we know that attacker has elevated privliges I changed the user from bsmith to SYSTEM to see if if any changes or attackers occured there. It was indeed the case, the attacker forged a new administrative user called backdoor, ensuring ongoing privileged entry to victim's computer, even if the initial infiltration method (Invoice.pdf.ps1) is detected and eradicated. <br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/13b16d8f-c67b-47d5-a5b6-8e3c15250b50"  alt="ELK Log Analysis"/>
 <img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/7f008403-d9bb-4976-968b-1ccc593bd3ae"  alt="ELK Log Analysis"/>
<br />
<br />
After this I removed the user filter for SYSTEM so I could see both bsmith and SYSTEM logs incase the attacker continued the attack. Unfortuantely, I found another download request from the malicious domain, this being bat program. <br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/04d5a418-7d95-485d-844d-66109c9b217a"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/ba3a5f88-1b14-49aa-b890-f7660036cbe5"  alt="ELK Log Analysis"/>
<br />
<br />
The last form of persistance I found by the attack was once again making changes to the registry.<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/30fdc9bb-4bad-42f7-87d5-a56d451e6a6e"  alt="ELK Log Analysis"/>
<br />
<br />
Towards the end of the logs the last at<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/2e47a686-191e-446c-9a74-30084aef7dbe"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/3edffe22-9593-44b2-ab5e-da18718ce3f2"  alt="ELK Log Analysis"/>
<br />
<br />
Latly, the attacker's brute force attack was successful with the credentials bsmith:Password123! and the<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/171b15d1-948d-41ea-ba9e-d6de95d44e16"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/680b13ae-1f89-4f35-97ab-0da4f1c7f9a5"  alt="ELK Log Analysis"/>
<br />
<br />

<h2>Thoughts</h2>
Text
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
