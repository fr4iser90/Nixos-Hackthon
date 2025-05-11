# Nixos-Hackthon

Ja, das geht – und deine Idee klingt nach einem richtig praktischen Setup für Hackathons! Du kannst ein dediziertes NixOS-System aufbauen, das automatisch Code einsammelt, containerisiert (z. B. per Docker oder Nix-basiertem Build-System), deployed und dann nach 48 Stunden automatisch aufräumt. Hier ist ein grober Überblick, wie du das umsetzen könntest:

---

## ✅ Zielsystem-Funktionen:

1. **Code-Upload & Trigger**

   * GitHub Push (Webhook)
   * Oder direkte Uploads via Web UI/API (z. B. über eine einfache Upload-Seite mit FastAPI oder Node)

2. **Automatisierte Containerisierung**

   * Dockerfile automatisch erkennen oder generieren (`buildpacks`, `nixpacks`, oder `Dockerfile-Templates`)
   * Build in isoliertem Container oder Builder-VM (z. B. mit BuildKit)

3. **Deployment**

   * Traefik mit dynamischen Hostnamen (z. B. `projektname.hack.local`)
   * Container wird mit Namen & URL deployed
   * Domain Routing über Traefik

4. **Laufzeit-Management**

   * Max. Laufzeit: 48h → dann Cleanup (per Systemd-Timer, Cronjob, oder Lifecycle-Manager)
   * Optionale manuelle Verlängerung via Webinterface

5. **Isolierung / Sicherheit (für kurzzeitiges Vertrauen)**

   * Netzwerk-isolierte Container
   * Keine Persistenz außerhalb der Containerlaufzeit
   * Rate Limits / Basic Auth optional via Traefik-Middleware

6. **Zentrale UI / Übersicht**

   * Zeigt alle aktuell laufenden Projekte an
   * Bietet Upload-Möglichkeit
   * Kill-Button / Timer-Anzeige

---

## 🛠 Stack-Vorschlag (Minimal):

* **System:** NixOS
* **Web/API:** FastAPI, Next.js oder Astro mit Upload-Funktion
* **Container:** Docker oder Podman
* **Build-Tooling:** Buildpacks / Nixpacks / Custom Dockerfile-Support
* **Routing:** Traefik mit ACME off oder interner CA
* **Jobs:** Systemd-Timer oder `docker container prune --filter "label=expire=..."`

---

## 🧠 Containerisierung automatisieren

Wenn du **keine Dockerfiles** erwartest:

* Verwende [**Buildpacks**](https://buildpacks.io) oder [**nixpacks**](https://github.com/railwayapp/nixpacks)
* Diese erkennen die Sprache (Node, Python, Go, etc.) automatisch und bauen einen Container
* Du kannst Templates für häufige Setups hinterlegen

Beispiel (CLI):

```bash
nixpacks build . -o type=docker -t meinprojekt:latest
```

---

## 🧼 Automatischer Cleanup

Du kannst Container mit Ablaufdatum starten:

```bash
docker run -d --label expire_at=2025-05-12T15:00 --name projekt123 meinprojekt
```

Dann alle 10 Minuten per Cron:

```bash
docker ps --filter "label=expire_at" --format '{{.ID}} {{.Label "expire_at"}}' \
| while read id time; do
    if [ "$(date -u +%s)" -ge "$(date -u -d "$time" +%s)" ]; then
        docker rm -f "$id"
    fi
done
```

---

Willst du ein einfaches Webinterface dafür oder eher ein CLI-Skript-System mit Webhook-Support?
