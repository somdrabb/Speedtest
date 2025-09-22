# SPEEDTEST
Internetschnelltest.de
# Internetschnelltest – Richtiger Speedtest

Diese Repo enthält alles, was du brauchst, um internetschnelltest.de auf einen echten LibreSpeed-Server umzuziehen. Folg einfach den drei Schritten.

## A) Domain auf den VPS zeigen
1. Ermittel die öffentliche IPv4 (und optional IPv6) deines VPS, z. B. `203.0.113.12`.
2. Setz im DNS für `internetschnelltest.de` einen `A`-Record auf diese IP. Falls du IPv6 hast, leg zusätzlich einen `AAAA`-Record an.
3. Wenn GitHub Pages momentan direkt auf dem Apex (`internetschnelltest.de`) läuft, verschieb die Pages auf eine Subdomain wie `app.internetschnelltest.de`, damit der Apex zum VPS zeigen kann.
4. Warte auf DNS-Propagation (einige Minuten bis ~1 h). Prüfe mit `dig internetschnelltest.de` vom lokalen Rechner oder einem Online-DNS-Checker.

## B) LibreSpeed + Caddy mit HTTPS betreiben
1. Logg dich auf deinem VPS ein und leg einen Projektordner an, z. B. `/opt/internetschnelltest`.
2. Kopier `docker-compose.yml` und `Caddyfile` aus diesem Repo in den Ordner.

   ```bash
   sudo mkdir -p /opt/internetschnelltest
   sudo chown "$USER" /opt/internetschnelltest
   cd /opt/internetschnelltest
   # Dateien aus dem Repo kopieren oder per git clone holen
   ```

3. Stell sicher, dass Docker (inkl. Compose v2) installiert ist und dein User in der `docker`-Gruppe steckt.

   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker "$USER"
   newgrp docker
   ```

4. Starte den Stack:

   ```bash
   docker compose up -d
   ```

   - `librespeed` läuft im internen Netzwerk `web`.
   - `caddy` lauscht auf Port 80/443, holt ein Let’s-Encrypt-Zertifikat für `internetschnelltest.de` und proxyt zum LibreSpeed-Container.

5. Prüf die Logs, falls etwas hakt:

   ```bash
   docker compose logs -f
   ```

6. Besuche `https://internetschnelltest.de` und führe einen Speedtest aus. Du solltest echte Download-/Upload-Transfers und Ping-Messungen sehen (DevTools → Network zur Kontrolle).

**Alternative:** Wenn du lieber Nginx + Certbot nutzt, starte LibreSpeed allein auf `127.0.0.1:8080`, setz das im Nginx-Vhost als Proxy ein und hol dir TLS über `certbot --nginx`.

## C) GitHub Pages weiterleiten
1. Das neue `index.html` in diesem Repo leitet sofort per Meta-Refresh auf `https://internetschnelltest.de` weiter und zeigt einen Fallback-Link.
2. Push diese Änderung zu GitHub Pages, damit Besucher deiner alten statischen Seite automatisch beim echten Test landen.
3. Optional kannst du stattdessen ein `<iframe>` verwenden (genaue Zeile in `index.html` austauschen), beachte aber mögliche Genauigkeitseinbußen und Mixed-Content-Einschränkungen.

## Betrieb & Wartung
- **Updates:** `docker compose pull && docker compose up -d`
- **Status:** `docker compose ps`
- **Stoppen:** `docker compose down`
- **Telemetrie:** Aktiviere `TELEMETRY=true` in der Compose-Datei, wenn du Ergebnisse anonymisiert speichern willst.
- **Skalierung:** Für mehrere Server kannst du LibreSpeed mit einer `servers.json` betreiben und `MODE=server` nutzen.

Viel Erfolg beim Go-Live deines echten Speedtests!
