# Projet VPN / Routeur

## Table des Matières
1. [Documentation utilisateur](#documentation-utilisateur)
2. [Documentation technique](#documentation-technique)
   - [Installation du serveur VPN](#installation-du-serveur-vpn)
   - [Configuration du client VPN](#configuration-du-client-vpn)
   - [Mise en place de l'authentification MFA](#mise-en-place-de-lauthentification-mfa)
   - [Installation d'OPNsense](#installation-dopnsense)

---

## Documentation utilisateur

### 1. Introduction
Ce guide explique comment **se connecter au VPN OpenVPN** et accéder aux ressources distantes en toute sécurité.

### 2. Prérequis
- 📂 Fichiers de connexion VPN : `ca.crt`, `client1.crt`, `client1.key`, `ta.key`
- 🖥️ Client OpenVPN installé
- 🔐 Mot de passe + Code MFA (Google Authenticator)

### 3. Installation du client VPN
#### Sous Windows
1. [Télécharger OpenVPN](https://openvpn.net/community-downloads/)
2. Copier les fichiers `.crt`, `.key`, `.conf` dans `C:\Program Files\OpenVPN\config\`
3. Lancer **OpenVPN GUI en tant qu'administrateur**
4. **Clic droit** sur l'icône OpenVPN et **"Connect"**

#### Sous Linux (Ubuntu/Debian)
```bash
sudo apt update && sudo apt install openvpn -y
sudo openvpn --config /etc/openvpn/client.conf
```
Entrer **nom d’utilisateur** + **mot de passe + code MFA**

### 4. Connexion avec MFA
1. Ouvrir **Google Authenticator** et récupérer le code OTP
2. Lors de la connexion, entrer :
   ```
   [Mot de passe UNIX] + [Code MFA]
   ```
   Exemple : `mypassword123456`
3. La connexion est sécurisée ✅

### 5. Vérifier la connexion VPN
```bash
curl ifconfig.me   # Vérifier l'adresse IP (doit être celle du VPN)
ip a show tun0    # Vérifier que tun0 est actif
ping 10.8.0.1     # Tester la connexion au serveur VPN
```

### 6. Déconnexion du VPN
- **Windows** : Clic droit sur OpenVPN > **Disconnect**
- **Linux** : `CTRL + C` dans le terminal

### 7. Dépannage
| Problème | Solution |
|----------|----------|
| **Impossible de se connecter** | Vérifier que le serveur VPN est actif (`ping 10.8.0.1`) |
| **Mauvais mot de passe ou MFA invalide** | Vérifier le code dans Google Authenticator |
| **Connexion instable** | Relancer OpenVPN et vérifier le réseau |
| **Pas d’Internet après connexion** | Vérifier `redirect-gateway` dans `client.conf` |

🎉 **Vous êtes connecté au VPN en toute sécurité !** 📩 **Contactez l’administrateur en cas de problème.**

---

## Documentation technique

### Installation du serveur VPN
#### 1. Mise à jour et installation des paquets
```bash
sudo dnf install epel-release -y
sudo dnf update -y
sudo dnf install openvpn easy-rsa -y
```

#### 2. Génération des certificats
```bash
mkdir -p ~/openvpn-ca && cp -r /usr/share/easy-rsa/* ~/openvpn-ca/
cd ~/openvpn-ca && ./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key
```

#### 3. Configuration du serveur OpenVPN
```bash
sudo mkdir -p /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/{ca.crt,issued/server.crt,private/server.key,dh.pem} /etc/openvpn/server/
sudo cp ~/openvpn-ca/ta.key /etc/openvpn/server/
```

**Fichier de configuration `/etc/openvpn/server/server.conf`**
```ini
port 1194
proto udp
dev tun
ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/server.crt
key /etc/openvpn/server/server.key
dh /etc/openvpn/server/dh.pem
tls-auth /etc/openvpn/server/ta.key 0
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
auth SHA256
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
log /var/log/openvpn/openvpn.log
verb 3
```

#### 4. Démarrage du service
```bash
sudo systemctl restart openvpn-server@server
sudo systemctl enable openvpn-server@server
sudo systemctl status openvpn-server@server
```

---

### Configuration du client VPN
#### 1. Génération des certificats client
```bash
cd ~/openvpn-ca
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```
Copier les fichiers sur le client :
```bash
scp /etc/openvpn/server/{ca.crt,ta.key} client@client-ip:/home/client/
scp ~/openvpn-ca/pki/{issued/client1.crt,private/client1.key} client@client-ip:/home/client/
```

#### 2. Fichier de configuration client
**`/etc/openvpn/client.conf`**
```ini
client
dev tun
proto udp
remote 10.8.0.1 1194
ca /home/client/ca.crt
cert /home/client/client1.crt
key /home/client/client1.key
tls-auth /home/client/ta.key 1
auth SHA256
persist-key
persist-tun
verb 3
```

#### 3. Connexion
```bash
sudo openvpn --config /etc/openvpn/client.conf
```

---

### Mise en place de l'authentification MFA
#### 1. Installation
```bash
sudo dnf install google-authenticator -y
```
#### 2. Configuration
```bash
google-authenticator -y -y -y -y -y
```
#### 3. Ajout de l'authentification PAM
```bash
sudo nano /etc/pam.d/openvpn
auth required pam_google_authenticator.so nullok
```
#### 4. Ajout au fichier OpenVPN
```ini
plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
verify-client-cert none
username-as-common-name
```
#### 5. Redémarrer le serveur VPN
```bash
sudo systemctl restart openvpn-server@server
```

---

### Installation d'OPNsense
#### Prérequis
- 1 carte réseau **Accès par pont**
- 1 carte **Host-Only**
- ISO OPNsense, 16 Go d'espace libre

#### Installation
1. Démarrer la VM et installer avec **UFS**
2. Retirer l'ISO et redémarrer
3. Configurer l'interface WAN/LAN

🎉 **Votre serveur VPN & routeur sont opérationnels !**



---
Projet B1 - Infrastructure & Système d’Information - Ynov - LEFEBVRE Lou, CABANES Hugo, CAETANO Maël


