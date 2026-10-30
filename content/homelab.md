---
title: "Dalles Homelab Setup"
description: "Dokumentation af min private infrastruktur og netværksarkitektur."
draft: true
---

# Velkommen til maskinrummet

Mit homelab kører som et fuldt, isoleret enterprisestruktur derhjemme i Nørreby. Det er bygget til at være redundant, strømsvagt og topsikret.

## 🖥️ Hardwaren
* **Compute:** Lenovo ThinkCentre (Hovedserver til al virtualisering)
* **Netværk:** Eget Mesh-router-setup koblet direkte til fiberboksen uden om udbyderens hardware.
* **Automation & Smarthome:** Raspberry Pi (Dedikeret til Home Assistant)

## 🌐 Netværksarkitektur & Sikkerhed
Hele netværket er struktureret med streng adgangskontrol for at beskytte min families enheder. 
* **Lokal DNS:** Al intern trafik filtreres og routes via **AdGuard Home**.
* **Zero Trust adgang:** Ingen porte er åbne i min router. Ekstern adgang sker via en krypteret **Cloudflare Tunnel** sikret med Cloudflare Access (One-Time-Passcode validering).
