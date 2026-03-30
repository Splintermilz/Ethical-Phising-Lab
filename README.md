# Ethical-Phising-Lab : Stack Debian / PostFix / GoPhish
Projet de sensibilisation à destination d'entreprise pour lutter contre le phishing.


![Platform](https://img.shields.io/badge/platform-Debian%2012-red?style=flat-square&logo=debian)
![Tool](https://img.shields.io/badge/tool-GoPhish-blue?style=flat-square)
![SMTP](https://img.shields.io/badge/SMTP-Postfix-orange?style=flat-square)
![Context](https://img.shields.io/badge/context-educational%20only-yellow?style=flat-square)

> **Contexte académique : usage local uniquement.**
> Ce projet est réalisé dans l'optique de sensibiliser des collaborateurs d'entreprises aux risques liés au phishing.
> Toute utilisation en dehors d'un environnement isolé et avec des cibles non consentantes
> est illégale et contraire à l'éthique. Ne jamais déployer cette stack sur Internet.

---

## Architecture

```
┌─────────────┐        ┌──────────────────┐        ┌──────────────────┐
│   GoPhish   │──SMTP──▶    Postfix        │──────▶ │  Cible (interne) │
│  :3333 UI   │        │  localhost:25    │        │  boîte mail test │
│  :80 landing│        │                  │        │                  │
└─────────────┘        └──────────────────┘        └──────────────────┘
      │                                                      │
      │◀──────────── {{.Tracker}} ouverture mail ────────────│
      │◀──────────── {{.URL}} clic sur lien ─────────────────│
```

**Flux détaillé :**

1. GoPhish construit l'email (template + variables) et l'envoie via Postfix sur `localhost:25`.
2. Postfix route le message vers la boîte de destination (locale ou jetable).
3. Le destinataire ouvre le mail → `{{.Tracker}}` déclenche une requête vers GoPhish → **Email Opened**.
4. Le destinataire clique le lien `{{.URL}}` → redirigé vers la Landing Page GoPhish → **Clicked Link**.
5. Si la Landing Page capture un formulaire → **Submitted Data**.

---

## Stack

| Couche | Composant | Rôle |
|--------|-----------|------|
| OS | Debian 12 | Base système |
| Serveur | Postfix 3.7.x | MTA : Mail Transfer Agent |
| Application | GoPhish 0.12.1 | Orchestrateur de campagne |

> **Go n'est pas requis.** J'utilise le binaire pré-compilé de la release officielle GoPhish.

---

## Prérequis

```bash
sudo apt update && sudo apt install -y wget unzip swaks
```

---
> **MAIL n'est pas disponible sur ma version Debian, j'ai donc opté pour SWAKS qui se prête parfaitement à cette opération.**

## 1. Installation & configuration de Postfix

### 1.1 Installation

```bash
sudo apt install -y postfix
```

Lors de l'assistant de configuration interactif, choisir :
- **Type de configuration** : `Local only`
- **System mail name** : `donotclick.com` (ou le domaine fictif de ton choix)

> 📸 **Capture recommandée :** l'écran de sélection du type de configuration Postfix.

### 1.2 Fichier `main.cf`

```bash
sudo vim /etc/postfix/main.cf
```

Ajouter ou modifier les lignes suivantes :

```ini
myhostname = mail.donotclick.com
mydomain   = donotclick.com
myorigin   = $mydomain
inet_protocols = ipv4
```

> **Pourquoi ces valeurs ?**
>
> | Directive | Valeur | Rôle |
> |-----------|--------|------|
> | `myhostname` | `mail.donotclick.com` | Identité du serveur annoncée lors du EHLO SMTP |
> | `mydomain` | `donotclick.com` | Domaine de référence pour les mails sortants |
> | `myorigin` | `$mydomain` | Domaine affiché dans l'adresse expéditeur |
> | `inet_protocols` | `ipv4` | Désactive IPv6 : évite les comportements inattendus en lab |
>
> Les autres directives (`inet_interfaces = loopback-only`, `mynetworks = 127.0.0.0/8`,
> `default_transport = error`) sont déjà correctement configurées par Debian lors
> de la sélection `Local only`.

> **Note :** toutes les commandes d'édition nécessitent `sudo`.
> Ne pas modifier les permissions des fichiers Postfix / On sauvegarde et on recharge :


```bash
sudo systemctl reload postfix
```

### 1.3 Vérification du service

```bash
sudo systemctl status postfix
```

```bash
# Vérifier les valeurs appliquées
postconf myhostname
postconf mydomain
postconf myorigin
postconf inet_protocols
```

> 📸 **Capture recommandée :** `systemctl status postfix` avec le statut `active (running)`.

### 1.4 Validation — test d'envoi SMTP

#### Vérifier que Postfix écoute sur le port 25

Le port 25 est le port par défaut du protocole SMTP.
Postfix l'ouvre automatiquement à l'installation, aucune configuration supplémentaire n'est nécessaire.

```bash
ss -tlnp | grep :25
```

Résultat attendu : `127.0.0.1:25`.

#### Tester avec swaks

`swaks` (Swiss Army Knife SMTP) est l'outil de référence pour tester un serveur SMTP.
Il affiche le handshake SMTP complet.

```bash
swaks --to root@localhost --from test@donotclick.com --server 127.0.0.1:25
```

> **Pourquoi `root@localhost` et pas `root@donotclick.com` ?**
> `root@localhost` est une boîte mail qui existe réellement sur le système Debian
> (stockée dans `/var/mail/root`). Postfix peut y livrer sans résolution DNS.
> `donotclick.com` est un domaine fictif, il n'y a pas de vraie boîte derrière,
> Postfix échouerait à résoudre le domaine.
>
> - Test Postfix → `root@localhost` (boîte réelle, livraison garantie)
> - Campagne GoPhish → `cible@tempmail.com` (adresse jetable externe)

La sortie doit se terminer par :

```
<-  250 2.0.0 Ok: queued as XXXXXXXXX
->  QUIT
<-  221 2.0.0 Bye
```

> 📸 **Capture recommandée :** sortie complète de swaks avec le `250 Ok: queued`.

#### Lire le mail reçu

```bash
sudo cat /var/mail/root
```

#### Vérifier les logs (Debian 12 — journald)

> Sur Debian 12, les logs mail passent par journald et non `/var/log/mail.log`.

```bash
sudo journalctl -t postfix --no-pager | tail -30
```

---

## 2. Installation de GoPhish

> *En cours de rédaction, sera publié dans la prochaine release.*

---

## Environnement testé

```
OS       : Debian 12 (Bookworm)
Postfix  : 3.7.x
Contexte : VM locale, réseau isolé
```

---

## Avertissement légal

Ce projet est strictement réservé à un usage éducatif en environnement isolé.
Toute simulation de phishing sur des cibles réelles sans consentement explicite
est punissable par la loi.
