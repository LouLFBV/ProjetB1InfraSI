<<<<<<< HEAD
# Projet VPN 

### Documentation utilisateur :











### Documentation technique :

On va réaliser le troisème sujet : PROJET " VPN (Diculté : 3)".
Puis, si nous avons le temps, on ajoutera des éléments du deuxième sujet : PROJET " ROUTEUR (Diculté : 2)",
dans le but d'améliorer notre projet.












Projet B1 - Infrastructure & Système d’Information - Ynov - LEFEBVRE Lou, CABANES Hugo, CAETANO Maël
=======
# Projet VPN 

### Documentation utilisateur :











### Documentation technique :

on va mettre en place un serveur VPN. On va utiliser Open VPN
Le but : on se connecte à un serveur VPN en tant que client (là ce sera une VM le serveur VPN, mais à l'autre bout du monde ça a le même effet).
Une fois connecté au VPN, on accès à :


on peut désigner le serveur VPN comme passerelle

ainsi, notre traffic ira jusqu'au serveur VPN avant de sortir sur internet


Sur la Machine VPN :

Mise à jour du server :
```
[lou@localhost ~]$ sudo dnf install epel-release -y
sudo dnf update -y
```

Installation d'OpenVPN et Easy-RSA
```
[lou@localhost ~]$ sudo dnf install openvpn easy-rsa -y

```

Génération des certificats et clés : Préparer Easy-RSA

```
[lou@localhost ~]$ mkdir -p ~/openvpn-ca
cp -r /usr/share/easy-rsa/* ~/openvpn-ca/
cd ~/openvpn-ca

```

Inialisation de la PKI :
```
[lou@localhost 3.1.6]$ ./easyrsa init-pki
```
Configurer les paramètres de la PKI :
```
[lou@localhost 3.1.6]$ nano /home/lou/openvpn-ca/3.1.6/pki/vars
set_var EASYRSA_REQ_COUNTRY     "FR"
set_var EASYRSA_REQ_PROVINCE    "Nouvelle-Aquitaine"
set_var EASYRSA_REQ_CITY        "Bordeaux"
set_var EASYRSA_REQ_ORG "Ynov Campus"
set_var EASYRSA_REQ_EMAIL       "projetb1infra@ynov.com"
set_var EASYRSA_REQ_OU          "IT"

```

Appliquer les changements : 
```
[lou@localhost 3.1.6]$ source pki/vars
```
Génération du certificat CA :
```
[lou@localhost 3.1.6]$ ./easyrsa build-ca
Using Easy-RSA 'vars' configuration:
* /home/lou/openvpn-ca/3.1.6/pki/vars

Using SSL:
* openssl OpenSSL 3.2.2 4 Jun 2024 (Library: OpenSSL 3.2.2 4 Jun 2024)

Enter New CA Key Passphrase:

Confirm New CA Key Passphrase:
.+............+....+.........+.....+................+.....+......+.+..+...+....+.........+++++++++++++++++++++++++++++++++++++++*........+...+..+...+++++++++++++++++++++++++++++++++++++++*....+.+......++++++
..+.......+...+...+++++++++++++++++++++++++++++++++++++++*......+.....+...+...+.+...+..................+........+......+.+..+...+++++++++++++++++++++++++++++++++++++++*...+..+......+...+.+...+...........+....+..+..........+...+...........+.+..............+......+.+...+............+...+..+............+.......+.....+......+.........+.+..................+......+........+......+.+......+........+......+...+......+.+...+.....+.......+...+........+.......+............+...+.....+.............+...+...............+...........+.+.........+......+.....+...+............++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

Notice
------
CA creation complete. Your new CA certificate is at:
* /home/lou/openvpn-ca/3.1.6/pki/ca.crt
[hugoetmaelforever]
```

Générer le certificat et la clé du serveur, Générer la clé privée et le certificat du serveur :
```
[lou@localhost 3.1.6]$ ./easyrsa gen-req server nopass
Using Easy-RSA 'vars' configuration:
* /home/lou/openvpn-ca/3.1.6/pki/vars

Using SSL:
* openssl OpenSSL 3.2.2 4 Jun 2024 (Library: OpenSSL 3.2.2 4 Jun 2024)
..+..............+...+...+...+.+...+...........+.......+...+............+..+.......+........+...+............+...+....+++++++++++++++++++++++++++++++++++++++*..........+......+....+..............+.+......+...+...+........+.+........+++++++++++++++++++++++++++++++++++++++*...+.+.....+.............+...+............+..+.+.....+.......+........+..........+..+...+.........+.+...+......+.....+.+...+......++++++
...+...+..........+.........+.....+...+...+.+...........+..................+.+.....+..........+........+...+++++++++++++++++++++++++++++++++++++++*..+.....+......+.+++++++++++++++++++++++++++++++++++++++*..+.+..+.............+..............+...+...+....+.....+......+.+.........+.....+......+...+...+....+.....+....+...............+........................+.....+............+.+..+...............+.+.....+.+...............+.........+.........+............+...............+...+.....+.......+....................+......+.......+..+.+..+.......+.....+.+.....+...+.......+...+...........+...+.......+..............+....+...+.........+...........+...+.......+.....+.......+..+.+...+..+.............+........................+...+............+..+....+.....+.......+.....++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:

Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/lou/openvpn-ca/3.1.6/pki/reqs/server.req
* key: /home/lou/openvpn-ca/3.1.6/pki/private/server.key
```
Signer le certificat du server avec le CA :
```
[lou@localhost 3.1.6]$ ./easyrsa sign-req server server
Using Easy-RSA 'vars' configuration:
* /home/lou/openvpn-ca/3.1.6/pki/vars

Using SSL:
* openssl OpenSSL 3.2.2 4 Jun 2024 (Library: OpenSSL 3.2.2 4 Jun 2024)
You are about to sign the following certificate:
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
Request subject, to be signed as a server certificate
for '825' days:

subject=
    commonName                = server

Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /home/lou/openvpn-ca/3.1.6/pki/openssl-easyrsa.cnf
Enter pass phrase for /home/lou/openvpn-ca/3.1.6/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Jun  6 09:31:14 2027 GMT (825 days)

Write out database with 1 new entries
Database updated

Notice
------
Certificate created at:
* /home/lou/openvpn-ca/3.1.6/pki/issued/server.crt


[lou@localhost 3.1.6]$ ./easyrsa sign-req server server
Using Easy-RSA 'vars' configuration:
* /home/lou/openvpn-ca/3.1.6/pki/vars

Using SSL:
* openssl OpenSSL 3.2.2 4 Jun 2024 (Library: OpenSSL 3.2.2 4 Jun 2024)

EasyRSA version 3.1.6

Error
-----
Cannot sign this request for 'server'.
Conflicting certificate exists at:
* /home/lou/openvpn-ca/3.1.6/pki/issued/server.crt

WARNING
=======
cleanup - remove_secure_session failed
```

Sécuriser les connexions en générant des clés Diffie-Hellman et TLS :
```
[lou@localhost 3.1.6]$ ./easyrsa gen-dh
[lou@localhost 3.1.6]$ openvpn --genkey --secret ta.key
```
✅ Certificat CA (ca.crt)
✅ Clé privée du serveur (server.key)
✅ Certificat du serveur (server.crt)
✅ Clés Diffie-Hellman (dh.pem)
✅ Clé TLS (ta.key)
 
Configurer le serveur OpenVPN, copier les fichiers de certificats et de clés :
```
[lou@localhost 3.1.6]$ sudo mkdir -p /etc/openvpn/server && sudo cp ~/openvpn-ca/3.1.6/pki/ca.crt /etc/openvpn/server/ && sudo cp ~/openvpn-ca/3.1.6/pki/issued/server.crt /etc/openvpn/server/ && sudo cp ~/openvpn-ca/3.1.6/pki/private/server.key /etc/openvpn/server/ && sudo cp ~/openvpn-ca/3.1.6/pki/dh.pem /etc/openvpn/server/ && sudo cp ~/openvpn-ca/3.1.6/ta.key /etc/openvpn/server/
```
Créer le fichier de configuration du serveur :
```
[lou@localhost 3.1.6]$ sudo nano /etc/openvpn/server/server.conf
port 1194
proto udp
dev tun

ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/server.crt
key /etc/openvpn/server/server.key
dh /etc/openvpn/server/dh.pem
tls-auth /etc/openvpn/server/ta.key 0

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt

push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"

keepalive 10 120
data-ciphers AES-256-GCM:AES-128-GCM
data-ciphers-fallback AES-256-CBC
auth SHA256
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
log /var/log/openvpn/openvpn.log
verb 3
```
Configurer le pare-feu et l’IP forwarding :
Activer le forwarding IP et l'appliquer 
```
[lou@localhost 3.1.6]$ echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
[sudo] password for lou:
[lou@localhost 3.1.6]$ sudo sysctl -p
```
Configurer le pare-feu :
```
[lou@localhost 3.1.6]$ sudo firewall-cmd --add-port=1194/udp --permanent && sudo firewall-cmd --add-masquerade --permanent &&
sudo firewall-cmd --reload
success
success
success
```
Créer des fichiers de logs :
```
[lou@localhost 3.1.6]$ sudo mkdir -p /var/log/openvpn/ && sudo touch /var/log/openvpn/openvpn.log && sudo touch /var/log/openvpn/openvpn-status.log && sudo chmod 644 /var/log/openvpn/openvpn.log && sudo chmod 644 /var/log/openvpn/openvpn-status.log
```
Activer et démarrer OpenVPN :
(On doit voir une IP 10.8.0.1 sur le serveur et voir 10.8.0.0/24 dans les routes)
```
[lou@localhost 3.1.6]$ sudo systemctl restart openvpn-server@server
[lou@localhost 3.1.6]$ sudo systemctl status openvpn-server@server
● openvpn-server@server.service - OpenVPN service for server
     Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; preset: disabled)
     Active: active (running) since Mon 2025-03-03 11:37:12 CET; 5s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 2217 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 11097)
     Memory: 1.3M
        CPU: 43ms
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
             └─2217 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-version 2 --suppress-timestamps>

Mar 03 11:37:12 localhost.localdomain systemd[1]: Starting OpenVPN service for server...
Mar 03 11:37:12 localhost.localdomain systemd[1]: Started OpenVPN service for server.
lines 1-16/16 (END)
```
Vérifier la connexion VPN :
```
[lou@localhost 3.1.6]$ ip a show tun0
5: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::b347:be03:aa93:efd7/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
[lou@localhost 3.1.6]$ ip r
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.8.0.0/24 via 10.8.0.2 dev tun0
10.8.0.2 dev tun0 proto kernel scope link src 10.8.0.1
[lou@localhost 3.1.6]$ ping 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=0.357 ms
64 bytes from 10.8.0.1: icmp_seq=2 ttl=64 time=0.154 ms
64 bytes from 10.8.0.1: icmp_seq=3 ttl=64 time=0.112 ms
64 bytes from 10.8.0.1: icmp_seq=4 ttl=64 time=0.107 ms
^C
--- 10.8.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3020ms
rtt min/avg/max/mdev = 0.107/0.182/0.357/0.102 ms
```

 
Sur la Machine Client :













---
Projet B1 - Infrastructure & Système d’Information - Ynov - LEFEBVRE Lou, CABANES Hugo, CAETANO Maël
>>>>>>> 1c51f8d (03/03)
