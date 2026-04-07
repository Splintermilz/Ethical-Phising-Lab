# Ethical Phishing Lab - Debian / Postfix / GoPhish


![Platform](https://img.shields.io/badge/platform-Debian%2012-red?style=flat-square&logo=debian)
![Tool](https://img.shields.io/badge/tool-GoPhish-blue?style=flat-square)
![SMTP](https://img.shields.io/badge/SMTP-Postfix-orange?style=flat-square)
![Context](https://img.shields.io/badge/context-educational%20only-yellow?style=flat-square)

> Projet de sensibilisation au phishing réalisé dans le cadre d'une campagne de sensibilisation.
> Toute utilisation en dehors d'un environnement isolé et avec des cibles non consentantes est illégale.

---

## Contexte

Ce projet simule une campagne de phishing éthique complète, de l'infrastructure d'envoi jusqu'au tracking des interactions.
Il est conçu pour démontrer concrètement les mécaniques d'une attaque par phishing afin de mieux sensibiliser les collaborateurs d'entreprise.

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

---

## Stack

| Couche | Composant | Rôle |
|--------|-----------|------|
| OS | Debian 12 (Bookworm) | Base système |
| Serveur | Postfix 3.7.x | MTA - Mail Transfer Agent |
| Application | GoPhish 0.12.1 | Orchestrateur de campagne |

---

## Documentation

| # | Partie | Statut |
|---|--------|--------|
| 01 | [Serveur SMTP - Postfix](./01-postfix/README.md) | ✅ |
| 02 | [Framework - GoPhish](./02-gophish/README.md) | ✅ |
| 03 | [Campagne & Tracking](./03-campagne/README.md) | ⏳ |

---

## Environnement testé

```
OS       : Debian 12 (Bookworm)
Postfix  : 3.7.x
GoPhish  : 0.12.1
Contexte : VM locale, réseau isolé
```

---

## Avertissement légal

Ce projet est strictement réservé à un usage éducatif en environnement isolé.
Toute simulation de phishing sur des cibles réelles sans consentement explicite
est punissable par la loi.

