---
title: "Mit Homelab: Fra IP-adresser til internt HTTPS med Nginx Proxy Manager"
date: 2026-10-23
description: "Hvordan jeg droppede grimme portnumre og sikrede mit Proxmox- og AdGuard-setup derhjemme i Nørreby."
tags: ["homelab", "proxmox", "netværk", "sikkerhed"]
categories: ["IT-Infrastruktur"]
showTableOfContents: true

draft: true
---

Efter at have kørt mit private tech-laboratorium på mit Lenovo ThinkCentre i et stykke tid, blev jeg træt af at skulle huske rå IP-adresser og grimme portnumre som `:8006` for at tilgå mit setup. 

Derfor har jeg bygget en fuldstændig lukket, intern **Zero Trust-arkitektur**, der omdanner mit private domæne `dalles.it` til min interne genvej via en reverse proxy.

---

## Arkitekturen i mit maskinrum

Hele løsningen er bygget op omkring tre centrale komponenter, som kører uafhængigt på mit LAN, uden at jeg har åbnet en eneste port i min mesh-router mod det farlige internet:

1. **AdGuard Home:** Fungerer som min lokale DNS-server.
2. **Nginx Proxy Manager (NPM):** Køres i en ressourceeffektiv Proxmox LXC-container.
3. **Cloudflare DNS:** Håndterer mit `dalles.it` domæne og validerer mine SSL-certifikater i baggrunden.

---

## Trin 1: Let's Encrypt Wildcard via DNS Challenge

Det geniale ved NPM er, at den kan tale direkte sammen med Cloudflares API. Ved at bruge en **DNS Challenge** i stedet for en traditionel HTTP-challenge, behøver Let's Encrypt ikke at kunne "se" min server udefra. 

Jeg oprettede en sikker API-token i min Cloudflare-konto og smed den direkte ind i min Nginx Proxy Manager for at hente et fuldgyldigt wildcard-certifikat:

```bash
# Et eksempel på hvordan min Cloudflare API-legitimation kaldes i baggrunden
dns_cloudflare_api_token = "MIN_HEMMELIGE_CLOUDFLARE_API_TOKEN_HER"
```

NPM klarede valideringen i kulissen og udstedte et certifikat til `*.dalles.it`. Slut med "Ikke sikker"-advarsler i min browser!

---

## Trin 2: DNS Rewrites i AdGuard Home

For at min Mac mini M1 og mine andre enheder i huset ved, hvor de skal gå hen, når jeg taster en adresse, bruger jeg **DNS Rewrites** i min AdGuard Home.

Jeg har tilføjet en simpel regel:

* **Mønster:** `*.dalles.it`
* **IP-adresse:** `192.168.1.5` (Den faste IP-adresse på min NPM-container)

Hver gang jeg nu taster `pve.dalles.it`, spørger min computer AdGuard om vejen. AdGuard svarer med det samme, at trafikken skal sendes til min reverse proxy.

---

## Det endelige resultat

NPM modtager trafikken på port 443 (HTTPS), tjekker det gyldige wildcard-certifikat og videresender mig til den rigtige maskine og port i baggrunden. Mit interne adressekort spiller nu maks:

| Service | Gammel HTTP-adresse | Ny Sikker HTTPS-adresse |
| :--- | :--- | :--- |
| **Proxmox VE** | `http://192.168.1.100:8006` | `https://dalles.it` |
| **AdGuard Home** | `http://192.168.1.102:80` | `https://dalles.it` |
| **Home Assistant** | `http://192.168.1.101:8123` | `https://dalles.it` |

Dette setup holder min familie og vores enheder fuldstændig sikre på LAN'et, mens jeg har fuld enterprise-komfort, når jeg nørder videre i mit værksted.
