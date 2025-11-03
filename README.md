# SMB RELAY ATTACK
## Explination
An SMB Relay Attack is a network-based attack that exploits the NTLM authentication process in the SMB (Server Message Block) protocol. 
Instead of cracking password hashes, the attacker intercepts and forwards (relays) these credentials to another system in real-time, 
allowing unauthorized access without brute-forcing or hash cracking.
## Requirements
  - The target must not enforce or enable SMB signing.
  - dnsspoof.
  - arpspoof.
  - metasploit.
## Tutorial
Launch an attack using the SMB Relay Exploit in a way that once the Client (172.16.5.5) issues a SMB connection to any hosts on the .sportsfoo.com domain it can be redirected to your Metasploit server, 
and then you can use its credentials to get a shell on the target machine (172.16.5.10*).

This is a graphic that represents how this attack should work:

<img width="875" height="604" alt="Screenshot 2025-11-02 225905" src="https://github.com/user-attachments/assets/ac17c7fc-946d-4c7e-bbff-464e04eb13e8" />

 1.Client (Windows 7) issues a SMB connection to [\\fileserver.sportsfoo.com\finance$] at every 30 seconds or so.

 2.The attacker machine intercepts this request and spoofs the IP address of fileserver.sportsfoo.com.

 3.Then the Windows 7 system issues a SMB connection to [\\172.16.5.101] (attacker machine) instead of using the real IP of the fileserver.sportsfoo.com.

 4.The SMB Relay exploit is already listening, receives the SMB connection, and relays the authentication to the target machine. The payload is a Windows Meterpreter shell.

 5.Once the exploit authenticates on the target machine, a reverse meterpreter session is provided to the attacker.

 
Configure dnsspoof in order to redirect the victim to our Metasploit system every time there's an SMB connection to any host in the domain: sportsfoo.com. 

Create a file with fake dns entry with all subdomains of sportsfoo.com pointing to our attacker machine.

<img width="1025" height="69" alt="Screenshot 2025-11-02 222705" src="https://github.com/user-attachments/assets/44e618c7-0bda-4a16-b9eb-9a77be95fece" />

Then we run dnsspoof as following :

<img width="718" height="72" alt="Screenshot 2025-11-02 230544" src="https://github.com/user-attachments/assets/acab061e-49f0-4901-9dd1-d2f4c0caedac" />

In order to perform an ARP Spoofing attack, we need to enable the IP forwarding as follow:

<img width="782" height="60" alt="Screenshot 2025-11-02 222822" src="https://github.com/user-attachments/assets/ff11b218-34f6-4566-9d55-c86f6e792b6e" />

Activate the MiTM attack using the ARP Spoofing technique. Our goal is to poison the traffic between our victim, Windows 7 at 172.16.5.5, and the default gateway at
172.16.5.1. In this way, we can manipulate the traffic using dnsspoof, which is already running.
In two separate terminals, start the ARP Spoof attack against 172.16.5.5 and 172.16.5.1 using these commands:

<img width="803" height="478" alt="Screenshot 2025-11-02 222505" src="https://github.com/user-attachments/assets/0abbf02e-69fd-49cb-bbf3-315a592d4d2c" />
<img width="844" height="491" alt="Screenshot 2025-11-02 222556" src="https://github.com/user-attachments/assets/d0e8f9d0-78c6-4685-b534-138259323a61" />

dnsspoof aligned with the ARP Spoof attack, forges the DNS replies telling that the searched DNS address is hosted at the attacker machine:

<img width="719" height="354" alt="Screenshot 2025-11-02 222728" src="https://github.com/user-attachments/assets/52074f0d-d10b-4673-ba93-923531154419" />

from the previous results, Windows 7 has started an SMB connection for [\\fileserver.sportsfoo.com\AnyShare]. Then instead of getting a DNS response with the real IP address of 
fileserver.sportsfoo.com, it received the IP of the attacker: 172.16.5.101. Consequently, the SMB connection is hijacked to [\\172.16.5.101\AnyShare
Start msfconsole and configure the SMB Relay exploit:

<img width="859" height="360" alt="Screenshot 2025-11-02 231747" src="https://github.com/user-attachments/assets/60a18634-5712-4ee1-9e65-3bb3547577f0" />

In Metasploit, every time there is an incoming SMB connection, the SMB Relay exploit grab the SMB hashes (credentials) and then uses them to get a shell on the target machine 
(172.16.5.10- since it was set in the SMBHOST field of the smb-relay exploit).

<img width="1066" height="419" alt="Screenshot 2025-11-02 223013" src="https://github.com/user-attachments/assets/f5bdf26f-ef3b-4702-b4a7-f4e1c8d20666" />

The SMB Relay attack was successful, and we were able to obtain a meterpreter session on the target machine.

Interact with the meterpreter session.

<img width="858" height="88" alt="Screenshot 2025-11-02 223101" src="https://github.com/user-attachments/assets/d113ab82-fbda-47ba-95db-93defac6ed6b" />
