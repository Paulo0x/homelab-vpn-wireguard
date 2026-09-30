# 🔐 Déploiement WireGuard -- Homelab 

> **En bref**
> - **Contexte :** permettre le télétravail en accédant au réseau interne depuis l'extérieur, sans exposer le LAN.
> - **Ce que j'ai fait :** VPN WireGuard sur pfSense, redirection du port UDP 51820 sur la box, règles de pare-feu, accès limité au LAN, configuration du poste client.
> - **Résultat :** accès distant chiffré au LAN et connexion RDP à un serveur, sans aucun port du LAN exposé.
> - **Compétences :** WireGuard, pfSense, NAT, pare-feu, VPN.

## 🎯 Objectif

Mettre en place un accès distant sécurisé permettant :

-   Accès au LAN PME : `10.0.10.0/24`
-   Connexion RDP vers `10.0.10.101`
-   Administration sécurisée
-   Simulation télétravail professionnel

------------------------------------------------------------------------

# 🏗 Architecture VPN

    PC Distant (Internet)
            │ UDP 51820
            ▼
    Freebox (NAT)
            │
            ▼
    pfSense WAN (192.168.1.92)
            │
    WireGuard Interface (10.10.10.1/24)
            │
            ▼
    LAN PME (10.0.10.0/24)

------------------------------------------------------------------------

# 🔧 1️⃣ Installation WireGuard sur pfSense

### Étape 1 -- Installation du package

1.  `System`
2.  `Package Manager`
3.  `Available Packages`
4.  Rechercher **WireGuard**
5.  Installer

Menu `VPN → WireGuard` disponible après installation.

------------------------------------------------------------------------

# ⚙ 2️⃣ Configuration du Tunnel

## Création du Tunnel

`VPN → WireGuard → Tunnels → Add`

  Paramètre           Valeur
  ------------------- ---------------
  Enable              ✔
  Listen Port         51820
  Interface Address   10.10.10.1/24
  Keys                Generate

Save → Apply

------------------------------------------------------------------------

## Création du Peer (Client)

`VPN → WireGuard → Peers → Add`

  Paramètre     Valeur
  ------------- ------------------------
  Enable        ✔
  Public Key    Clé publique du client
  Allowed IPs   10.10.10.2/32
  Description   PC-Distant

Save → Apply

------------------------------------------------------------------------

# 🌐 3️⃣ Configuration NAT (Freebox)

Redirection de port :

-   Protocole : UDP
-   Port externe : 51820
-   IP destination : 192.168.1.92
-   Port interne : 51820

------------------------------------------------------------------------

# 🔥 4️⃣ Règles Firewall pfSense

## Règle WAN

`Firewall → Rules → WAN → Add`

-   Action : Pass
-   Protocol : UDP
-   Destination Port : 51820

## Règle WireGuard

`Firewall → Rules → WG_INTERFACE → Add`

-   Action : Pass
-   Source : Any
-   Destination : Any

------------------------------------------------------------------------

# 💻 5️⃣ Configuration Client WireGuard

Installer WireGuard (Windows).

Configuration :

    [Interface]
    PrivateKey = CLE_PRIVEE_CLIENT
    Address = 10.10.10.2/24
    DNS = 10.0.10.10

    [Peer]
    PublicKey = CLE_PUBLIQUE_PFSENSE
    Endpoint = IP_PUBLIQUE:51820
    AllowedIPs = 10.0.10.0/24
    PersistentKeepalive = 25

------------------------------------------------------------------------

# 🧪 6️⃣ Tests de Validation

Après activation du tunnel :

    ipconfig
    ping 10.0.10.1
    ping 10.0.10.101

Connexion RDP :

    mstsc → 10.0.10.101
    Utilisateur : monlab\paulo.rosa

------------------------------------------------------------------------

# 🧠 Justification Technique

  Choix                      Justification
  -------------------------- ------------------------------
  WireGuard                  Moderne, performant, simple
  /32 pour client            Attribution unique sécurisée
  AllowedIPs limité          Accès restreint au LAN
  Port UDP 51820             Standard WireGuard
  Tunnel séparé 10.10.10.0   Séparation logique VPN/LAN

------------------------------------------------------------------------

# ✅ Résultat

✔ Accès distant sécurisé\
✔ Aucun port LAN exposé\
✔ Architecture PME réaliste\
✔ Simulation télétravail professionnelle

------------------------------------------------------------------------

**Projet Homelab -- Paulo Rosa**
