# 01 — Serveur SMTP local avec Postfix

![Platform](https://img.shields.io/badge/platform-Debian%2012-red?style=flat-square&logo=debian)
![SMTP](https://img.shields.io/badge/SMTP-Postfix-orange?style=flat-square)

> Mise en place d'un serveur SMTP local sur Debian 12.
> Postfix est configuré en mode `loopback-only`, aucun mail ne sera exposé sur Internet.

---

## Objectif

Disposer d'un MTA (Mail Transfer Agent) local que GoPhish pourra utiliser
pour envoyer ses emails via `localhost:25`, sans aucune dépendance externe.

---

## 1. Installation

```bash
sudo apt update && sudo apt install -y postfix
```

L'installateur ouvre un assistant interactif. Sélectionner **"Local uniquement"** :

![Sélection Local uniquement](../assets/screenshots/postfix-local-only.png)

Puis renseigner le domaine fictif **`donotclick.com`** :

![System mail name donotclick.com](../assets/screenshots/postfix-mailname.png)

---

## 2. Configuration — `main.cf`

```bash
sudo vim /etc/postfix/main.cf
```

Ajouter ou modifier les quatre directives suivantes :

```ini
myhostname = mail.donotclick.com
mydomain   = donotclick.com
myorigin   = $mydomain
inet_protocols = ipv4
```

![Fichier main.cf configuré](../assets/screenshots/postfix-maincf.png)

| Directive | Valeur | Rôle |
|-----------|--------|------|
| `myhostname` | `mail.donotclick.com` | Identité annoncée lors du `EHLO` SMTP |
| `mydomain` | `donotclick.com` | Domaine de référence pour les mails sortants |
| `myorigin` | `$mydomain` | Domaine affiché dans l'adresse expéditeur |
| `inet_protocols` | `ipv4` | Désactive IPv6 |

> Les directives `inet_interfaces = loopback-only`, `mynetworks = 127.0.0.0/8`
> et `default_transport = error` sont déjà posées par Debian à l'installation.

Sauvegarder, puis recharger :

```bash
sudo systemctl enable postfix
```

---

## 3. Vérification

```bash
sudo systemctl enable postfix && sudo systemctl restart postfix
sudo systemctl status postfix
```

![systemctl status postfix — active running](../assets/screenshots/postfix-status.png)

---

## 4. Test d'envoi SMTP

### Vérifier que Postfix écoute sur le port 25

```bash
ss -tlnp | grep :25
```

![Port 25 en écoute sur 127.0.0.1](../assets/screenshots/postfix-port25.png)

Le port 25 est ouvert automatiquement à l'installation, aucune configuration nécessaire.

### Envoyer un mail de test

> `mail` n'est pas disponible sur Debian 12 — j'utilise `swaks`
> (Swiss Army Knife SMTP), l'outil de référence pour tester un handshake SMTP.

```bash

swaks --to root@localhost --from test@donotclick.com --server 127.0.0.1:25
```

![Sortie swaks — 250 Ok queued](../assets/screenshots/postfix-swaks.png)

> **Pourquoi `root@localhost` ?**
> C'est la seule boîte mail qui existe réellement sur le système (dans `/var/mail/root`).
> `donotclick.com` est fictif — Postfix ne peut pas résoudre ce domaine localement.

### Lire le mail reçu

```bash
sudo cat /var/mail/root
```

![cat /var/mail/root — mail reçu](../assets/screenshots/postfix-mailrecu.png)

### Consulter les logs

> Sur Debian 12, les logs mail passent par `journald` — `/var/log/mail.log` n'existe pas.

```bash
sudo journalctl -t postfix --no-pager | tail -30
```

---

## Résultat

Postfix est opérationnel :

- écoute sur `127.0.0.1:25` uniquement
- accepte les connexions locales sans authentification
- ne relaie aucun mail vers l'extérieur

➡️ Suite : [02 — Installation de GoPhish](../02-gophish/README.md)

