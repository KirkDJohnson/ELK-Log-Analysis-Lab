<h1>ELK Log Analysis</h1>

<br />
<h2>Description</h2>
This lab I analyzed a known malicious packet capture on the exploit Log4J. Due to the traffic in capture using http and not https (Hyper Text Transfer Protocol...[Secure] encrypts data after the TCP handshake making it unreadable unless in possession of the decryption key), I was able to identify the exploit in clear text. Once the attack was identified I checked to see if the server responsed to the attacker, i.e opened a command and control (c2) channel confirming whether or not the exploit was successful; which was fortunately not the case. Once it was clear the attack was unsuccessful I conducted threat intelligence/research to learn more about attack. My approach to this was to initally see who (what IP addresses and where they were from) was communicating with the target server. Then it was crucial to determine whether the server that was attacked (198.71.247.91) begun any new outbound connections.

<h2>Utility Used</h2>

- <b>Elastic Search/Kibana</b> 


<h2>Environments Used </h2>

- <b>Kali Linux through Oracle Virtual Box

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
Next thing I did was do some basic reconnaissance/baselining. Most the traffic is private except for the IP 84[.]237[.]252[.]156 so I took note of it. Alao, for a endpoint user in finance, the top processes being curl and powershell definetely raises redflags. <br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/164f8629-0fcc-496e-821e-e066488c8076"  alt="ELK Log Analysis"/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/9b3c2257-75e2-4d42-a4f4-9f2b023847db"  alt="ELK Log Analysis"/>
<br />
<br />
changed date old to newest to retrace the steps of the attacker and filtered with for results for process.command.line to see what commands were ued<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/9e27d94d-7530-49b1-972f-c2b31ce3a006"  alt="ELK Log Analysis"/>
<br />
<br />
It appears the client was tricked into downloading invoice[.]pdf but was unaware the actual file extenstion was [.]ps1 which is powershell script. In the following logs, commands were input likely by the attacker for reconnisance efforts such as whoami and ipconfig. These can be benign as an IT admin would be seen using these but given the suspected breach it is likely the threat actor<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/cdf79fd7-5678-4585-8989-a8b4c4610a9c"  alt="ELK Log Analysis"/>
  <img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/adcc4ec3-5ce3-475d-9342-ff89deccabf3"  alt="ELK Log Analysis"/>
<br />
<br />
Suspicious, likely malicious downloaded using Powershells Invoke-WebRequest from the domain evilparrot[.]thm with the file winPEASany[.exe] renamed as winPEAS[.]exe<br/>
<img src="https://github.com/KirkDJohnson/ELK-Log-Analysis-Lab/assets/164972007/d857749a-feed-46be-b380-326dc6302b98"  alt="ELK Log Analysis"/>
<br />
<br />
Conducting some research on the program winPEAS[.]exe I found a github page detailing that it is script for finding privliege escalation<br/>
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
  Text<br/>
<img src=""  alt="ELK Log Analysis"/>
<br />
<br />
  Text<br/>
<img src=""  alt="ELK Log Analysis"/>
<br />
<br />
  Text<br/>
<img src=""  alt="ELK Log Analysis"/>
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
