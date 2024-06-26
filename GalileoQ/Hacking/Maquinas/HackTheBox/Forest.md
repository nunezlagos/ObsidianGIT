#ActiveDirectory #Easy #windows 

### Ping
```python
ping -c 1 10.10.10.161
PING 10.10.10.161 (10.10.10.161) 56(84) bytes of data.
64 bytes from 10.10.10.161: icmp_seq=1 ttl=127 time=151 ms

--- 10.10.10.161 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 151.126/151.126/151.126/0.000 ms
```

### TTL 127=Windows
### nmap
```python
PORT      STATE SERVICE      VERSION
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-03-16 15:37:07Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49674/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc        Microsoft Windows RPC
49680/tcp open  msrpc        Microsoft Windows RPC
49700/tcp open  msrpc        Microsoft Windows RPC
49911/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
|_clock-skew: mean: 2h25m55s, deviation: 4h02m29s, median: 5m54s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-03-16T15:38:02
|_  start_date: 2024-03-16T14:08:55
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2024-03-16T08:37:58-07:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.82 seconds
```

### Enumeración del puerto 445 (SMB)
con crackmapexec enumeramos el puerto `smb` donde podemos obtener el nombre de la maquina y el nombre del dominio, tratamos de enumerar los recursos compartidos pero no tenemos éxito. aplicamos un nuevo reconocimiento con smbclient pero no tenemos resultado
![[Pasted image 20240316114408.png]]

### rpcclient
con `rpcclient` podemos enumerar usuarios a nivel de dominio. obtenemos un listado de posibles usuarios
![[Pasted image 20240316115705.png]]

### Enumeración de usuarios
```python
rpcclient -U "" 10.10.10.161 -N -c 'enumdomusers' | grep -oP "\[.*?\]" | grep -v 0x | tr -d '[]' > users.txt
```
crearemos un archivo para guardar todos los usuarios que hemos enumerado
![[Pasted image 20240316120801.png]]

### impacket-GetNPUsers
con `impacket-GetNPUsers` podemos realizar un ataque ASREPRoast para intentar obtener un `TGT`(Ticket Granting Tickets) de algunos de los usuarios que hemos logrado enumerar 
![[Pasted image 20240316121631.png]]

### john
con john podemos intentar crakear el TGT que hemos conseguido que no es mas que un hash
![[Pasted image 20240316123151.png]]

### crackmapexec
validamos las credenciales que hemos conseguido con y validamos que este usuario tenga conexión con `evil-winrm` 
![[Pasted image 20240316130621.png]]

### evil-winrm

![[Pasted image 20240316130915.png]]

### ldapdomaindump
con `ldapdomaindump` podemos hacer una enumeración a nivel de grupos pertenecientes al usuario del sistema.
![[Pasted image 20240316133840.png]]

### servidor con python
creamos un servidor de python para poder ver los archivos desde el `localhost` 
![[Pasted image 20240316133533.png]]

### BloodHound
```python
sudo apt install neo4j BloodHound
```
primero debemos iniciar el neo4j para que luego el bloodhound pueda conectarse a la base de datos 
![[Pasted image 20240316134850.png]]

`BloodHound`
ingresamos las credenciales y abrimos el bloodhound
![[Pasted image 20240316135001.png]]

`BloodHound`
![[Pasted image 20240316135100.png]]

### Sharphound.ps1
vamos a descargar esta herramienta desde aquí [SharpHound.ps1](https://raw.githubusercontent.com/puckiestyle/powershell/master/SharpHound.ps1)  
![[Pasted image 20240316140513.png]]

`ahora vamos a subir la herramienta a la maquina victima` 
![[Pasted image 20240316140733.png]]

`Descargamos el archivo zip que hemos generado`
![[Pasted image 20240316144032.png]]

`descomprimimos el zip`
![[Pasted image 20240316144334.png]]

### BloodHound
en el bloodhound vamos a upload Data seleccionamos todos los archivos y lo abrimos
![[Pasted image 20240316144426.png]]

