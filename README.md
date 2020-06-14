# # VPN par Wireguard!
[toc]

## Présentation
Ce tutoriel est valable pour **_Linux_** et **_Windows_**. Bien que les configurations soit proches, il existe bien des différences.

---
### Le projet Wireguard 
Wireguard est un programme Open-Source de tunnelage VPN.
Il est simple d'utilisation et permet de créer un ``Point-to-Point VPN`` en tres peu de temps.
Il ne necessite que d'un hote et d'un client, les 2 étants des ``"PEERs"``  




### Les avantages
 - Ce logiel marche sur toutes les plateformes (Linux/OSX/Windows/Android/IOS etc...).
 - Open-source, donc n'importe qui peut contribuer au projet.
 - Installation et utilisation simple.
### Explication d'un VPN
![](https://i.imgur.com/xuQuTA7.png)
> Réalisé par Samuel ADONE 




## Prérequis
Pour cela, vous aurez besoin de:
- Wireguard disponible [ici](https://www.wireguard.com/install/)
- Un hote (ou "serveur")
- Un Client
- Ouvrir les ports sur votre Box internet/routeur que vous allez utiliser (ici, il sera question du port 51820 en TCP et UDP): http://192.168.1.1/

[à noter que tout peut se faire en environement virtuel]

---

## Installation: 

### Linux:

Il vous faudra en premier lieu installer Wireguard, sous les distributions basées sur
 - Debian/Ubuntu (19 ou plus):
    ```shell
    $ sudo apt install wireguard
    ```
 - Fedora:
    ```shell
    $ sudo dnf install wireguard-tools
    ```
 - Arch:
    ```shell
    $ sudo pacman -S wireguard-tools
    ```
 - CentOS 7:
    ```shell
    $ sudo yum install epel-release https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
    $ sudo yum install yum-plugin-elrepo
    $ sudo yum install kmod-wireguard wireguard-tools
    ```
 - CentOS 8: 
    ```shell
    $ sudo yum install elrepo-release epel-release
    $ sudo yum install kmod-wireguard wireguard-tools
    ```
### Windows:
Lien de téléchargement [disponible ici](https://download.wireguard.com/windows-client/wireguard-amd64-0.1.0.msi)

---

## Configuration

### Serveur Linux:
1. Une fois Wireguard installé, la premiere chose a faire sera de généré ce que l'on appel une **clé privée**. Cette clé sera propre à notre ordinateur. ***Il ne faudra la divulguer à personne***, meme pas votre gentille mamie qui vous cuisine des bons petits plats car elle pourrait etre un agent de la NSA.
    Pour ce faire:
    ```shell
    pi@raspberry:~ $ wg genkey
    4H2VqKGxE3jkERITIqIKJRvBszn2SFeqkvaDUXj4BFQ=
    ```
2. Il vous faudra maintenant creer le ficher de configuration de Wireguard
    ```shell
    pi@raspberry:~ $ sudo vim /etc/wireguard/wg0.conf
    ```
3. Dedans, vous aller coller le code suivant:
    ```
    [Interface]
    Address =    # Il s'agira de l'adresse de votre serveur dans la lan virtuel que va creer WG
    SaveConfig = # Enregistre les configs
    ListenPort = # Port de WG
    PrivateKey = # Copier coller la clé générée juste avant
    DNS =        # Il s'agira du DNS
    ```
    Le fichier devrais donc ressembler à quelque chose comme ca:
    ```
    [Interface]
    Address = 10.0.6.1/24
    SaveConfig = true 
    ListenPort = 51820
    PrivateKey = 4H2VqKGxE3jkERITIqIKJRvBszn2SFeqkvaDUXj4BFQ=
    DNS = 9.9.9.9, 149.112.112.112
    ```
4. Ouverture des ports:
    Dependant de votre **distribution**, il vous faudra ouvrir le port choisi pour **WG**. Ici, sous debian ca sera comme tel:    
    ```bash
    pi@raspberry:~ $ sudo ufw enable
    Firewall is active and enabled on system startup
    
    pi@raspberry:~ $ sudo ufw allow 51820/udp
    Rules updated
    Rules updated (v6)
    
    pi@raspberry:~ $ sudo ufw reload
    Firewall reloaded    
    ```
    
### Client Linux 
1. La configuration est la meme, ou presque:
    fichier ``wg0.conf``:
    
   ```bash 
    [Interface]
    Address = 10.0.6.3/24 # une autre IP dans le meme reseau que le serveur
    SaveConfig = true 
    ListenPort = 51820
    PrivateKey = CC4srDYhjcJd1uxw5vYV/+yWG0VrCVum3mzhXRP+RW8=
    DNS = 9.9.9.9, 149.112.112.112
    ```
    Le ``[Peer]`` sera le serveur dans le cas suivant.

2. Nous allons maintenant lancer le VPN afin d'obtenir une clé publique, il y a 2 facons de le faire:
    1. ```bash 
       pi@raspberry:~ $ sudo wg-quick up wg0
        [#] ip link add wg0 type wireguard
        [#] wg setconf wg0 /dev/fd/63
        [#] ip -4 address add 10.0.6.4/24 dev wg0
        [#] ip link set mtu 1420 up dev wg0
        [#] ip -4 route add 0.0.0.0/0 dev wg0 table 51820
        [#] ip -4 rule add not fwmark 51820 table 51820
        [#] ip -4 rule add table main suppress_prefixlength 0
        [#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
        [#] iptables-restore -n 
       ```
    2. NB: si toutefois cela ne marche pas, il faudra alors verifier que le processus tourne donc:
         ```bash 
            pi@raspberry:~ $ systemctl status wg-quick@wg0
            ● wg-quick@wg0.service - WireGuard via wg-quick(8) for wg0
               Loaded: loaded (/lib/systemd/system/wg-quick@.service; enabled; vendor preset
               Active: active (exited) since Mon 2020-05-04 20:22:47 BST; 4s ago
                 Docs: man:wg-quick(8)
                       man:wg(8)
                       https://www.wireguard.com/
                       https://www.wireguard.com/quickstart/
                       https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8
                       https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8
              Process: 4065 ExecStart=/usr/bin/wg-quick up wg0 (code=exited, status=0/SUCCES
                     Main PID: 4065 (code=exited, status=0/SUCCESS)
         ```
         La ligne ``Active: active`` montre que le vpn marche bien
          

3. la clé public peut s'obtenir en faisant:

    ```
    pi@raspberry:~ $ sudo wg show
    interface: wg0
      public key: 5AIZWb7mPTHP/VmszU+5qJg2aa7N2yqYWdp6upjKrR0=
      private key: (hidden)
      listening port: 51820
      fwmark: 0xca6c
    ```

    3.5 Il faut désactiver le VPN:
    - Soit par la commande ``wg-quick``:
    ```bash
    pi@raspberry:~ $  sudo wg-quick down wg0
    [#] wg showconf wg0
    [#] ip -4 rule delete table 51820
    [#] ip -4 rule delete table main suppress_prefixlength 0
    [#] ip link delete dev wg0
    [#] iptables-restore -n
    ```
    - Soit par la commande ``systemctl``:
    ```bash
    pi@raspberry:~ $ systemctl stop wg-quick@wg0
    ```
:warning: IL EST IMPORTANT de répéter l'opération avec le serveur pour obtenir sa clé publique aussi.

### Configuration des Pairs (Peers) Linux

1. Apres avoir obtenu la clef publique du Serveur et du Client [voir CLIENT LINUX - ETAPE 3], nous rajouterons les Peers, c'est à dire les coordonnées de chaque bout du tunnel que nous allons créer
    Pour cela:
    1. le fichier de configuration du serveur doit ressembler à cela:
    ```bash
    [Interface]
    PrivateKey = YMxbC0rv3DHtRm/VW9JUedFLjg5UTnwFCNMgIgCdB0k=
    Address = 10.0.6.1/24
    DNS = 9.9.9.9, 149.112.112.112

    [Peer]
    PublicKey = 5AIZWb7mPTHP/VmszU+5qJg2aa7N2yqYWdp6upjKrR0= #Clé public du client
    AllowedIPs = 0.0.0.0/0
    ```
    > ``AllowedIPs`` correspond à l'adresse IP locale qui aura le droit de se connecter au serveur, ici ``0.0.0.0/0`` signifie que toutes les IPs sont autorisées, Si une seule IP est souhaitée, alors un ``10.0.6.4/32`` signifie que SEULE cette adresse est autorisée, le ``/32`` définie l'exclusivité de cette adresse. 
    
:warning: Si vous voulez aussi rediriger le trafic vers l'interface, il vous faudra ajouter ces lignes à ``[Interface]``
    et remplacer ``eth0`` par le nom de votre carte réseau
      - - 
      `PostUp = sysctl -w net.ipv4.ip_forward=1; iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`
      -  - 
      `PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE`
    
2. Le fichier de configuration du client doit ressembler à cela:

    ```bash
    [Interface]
    Address = 10.0.6.3/24 # une autre IP dans le meme reseau que le serveur
    SaveConfig = true 
    ListenPort = 51820
    PrivateKey = CC4srDYhjcJd1uxw5vYV/+yWG0VrCVum3mzhXRP+RW8=
    DNS = 9.9.9.9, 149.112.112.112
    
    [Peer]
    PublicKey = /MXugIG88DZEdLq3EWhke72hfLeRD1pWKw+36w45yk8= #clé pulique serveur
    Endpoint = [adressePublicServeur]:51820
    AllowedIPs = 0.0.0.0/0
    ```
---
#### Et voila! il ne reste plus qu'à demarrer le client et le serveur! 
Pour cela, c'est comme nous l'avons vu précédement avec ``$ systemctl start wg-quick@wg0`` ou ``$ sudo wg-quick up wg0``

- Pour s'assurer de la connexion, on peut:
    - effectuer un ``ping``


---


### Serveur Windows

La configuration est plus simple sous Windows, meme si elle reste presque identique.

1. Preparer le Serveur et le fichier de configuration

    ```bash
    [Interface]
    Address = 192.168.200.1/24
    PrivateKey = #Replace with server private key
    ListenPort = 51820
    DNS = 9.9.9.9, 149.112.112.112
    ```
    
A sauvegarder dans `C:\wireguard\wg_server.conf`
### Utilisation
La configuration des pairs est identique à Linux.

- Pour demarer le serveur, voici un script:

```powershell
@echo off
"C:\Program Files\WireGuard\wireguard.exe" /installtunnelservice "C:\wireguard\wg_server.conf"
"%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe" Set-NetConnectionSharing "wg_server" $true
```
> remplacer la ligne 2 par l'executable de votre Wireguard

Configuration de la NAT:

Dans le powershell:
```bash
Set-NetConnectionSharing "wg_server" $true
```


Nous allons maintenant enregistrer ce script sous `start_server.bat`

- Pour arreter le serveur:

```powershell
@echo off
"C:\Program Files\WireGuard\wireguard.exe" /installtunnelservice "C:\wireguard\wg_server.conf"
"%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe" Set-NetConnectionSharing "wg_server" $true
```
> remplacer la ligne 2 par l'executable de votre Wireguard

:warning: Toujours lancer ces scripts en tant qu'administrateur.
