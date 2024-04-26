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
Powershell command was input and the next reconnisance efforts by the attacker appear to be happening, these can be benign as an IT admin would be seen using these but given the suspected breach it is likely the threat actor<br/>
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
