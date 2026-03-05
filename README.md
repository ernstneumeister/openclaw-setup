Ich möchte OpenClaw auf einem eigenen VPS-Server installieren, damit ich einen KI-Assistenten habe, der rund um die Uhr über Telegram erreichbar ist. Ich bin kein Server-Experte. Führe mich Schritt für Schritt durch die Installation.

Bei manuellen Schritten: Warte auf meine Bestätigung, bevor du weitergehst.
Erkläre kurz, WARUM wir jeden Schritt machen.
Wenn ich irgendwo nicht weiterkomme, bitte mich um einen Screenshot von dem, was ich sehe — dann kann ich dir helfen.

Alles was du automatisch erledigen kannst (Server-Konfiguration, Software-Installation, Sicherheit), mach bitte selbstständig. Nur bei Schritten, die meine Eingabe brauchen (Account-Erstellung, Bezahlung, Bot-Erstellung), führe mich durch.

---

## Schritt 1: Arbeitsbereich einrichten

Erstelle auf meinem Desktop einen Ordner `openclaw-setup` mit folgender Struktur:
- `.env` — hier werden alle Zugangsdaten gesammelt (einzige Datei mit Secrets)
- `CLAUDE.md` — Projektdoku, damit du in zukünftigen Sessions sofort Kontext hast

Starte die `.env` mit diesen leeren Feldern:

```env
# OpenClaw Management — Alle Zugangsdaten an einem Ort
# =====================================================

# Hetzner Cloud
HETZNER_API_TOKEN=
SERVER_IP=
SERVER_NAME=openclaw-server

# SSH
SSH_PUBLIC_KEY=

# Telegram Bot
TELEGRAM_BOT_NAME=
TELEGRAM_BOT_USERNAME=
TELEGRAM_BOT_TOKEN=
TELEGRAM_USER_ID=

# Gateway Dashboard
DASHBOARD_PORT=18789
DASHBOARD_TOKEN=
DASHBOARD_URL=
```

---

## Schritt 2: SSH-Key vorbereiten

Prüfe, ob auf meinem Computer schon ein SSH-Key existiert. Falls nicht, erstelle einen.
Passe die Befehle automatisch an mein Betriebssystem an (macOS, Windows, Linux).
Trage den Public Key in die `.env` ein und zeige ihn mir — den brauche ich gleich.

---

## Schritt 3: Hetzner-Account und Server erstellen (manuell)

Leite mich durch folgende Schritte. Ich mache sie selbst im Browser:

### 3a: Hetzner-Account
1. Gehe zu https://console.hetzner.cloud
2. Erstelle einen Account (falls noch keiner vorhanden)
3. Füge eine Zahlungsmethode hinzu

### 3b: SSH-Key hinzufügen
1. In der Hetzner Console: Auf den Server klicken → **Sicherheit** (unten links) → **SSH Keys → Add SSH Key**
2. Füge den Public Key von eben ein
3. Benenne ihn (z.B. "mein-computer")

### 3c: Server erstellen
1. **Servers → Add Server**
2. **Standort**: Nürnberg (oder der nächstgelegene)
3. **Image**: Ubuntu 24.04
4. **Typ**: Shared vCPU → **Cost-Optimized** → **CX33**
   - 4 vCPU, 8 GB RAM, 80 GB SSD, Intel/AMD
   - WICHTIG: Mindestens 8 GB RAM! Kleinere Server reichen nicht.
5. **Netzwerk**: "Public IPv4" muss aktiviert sein
6. **SSH Key**: den gerade hinzugefügten auswählen
7. **Name**: openclaw-server
8. Klicke auf "Kostenpflichtig erstellen"

**Kosten: ca. 5,99 EUR/Monat** (5,49 Server + 0,50 IPv4). Jederzeit kündbar, keine Vertragsbindung.

Wenn der Server läuft, gib mir die **Server-IP-Adresse** (steht in der Hetzner Console).

### 3d: API-Token erstellen
1. In der Hetzner Console: **Security → API Tokens → Generate API Token**
2. Name: "openclaw"
3. Berechtigung: **Read & Write**
4. Token kopieren und mir geben

Ich trage beides in deine `.env` ein.

---

## Schritt 4: Server einrichten (automatisch)

Ab hier übernimmst wieder du. Mit der Server-IP und dem SSH-Key kannst du dich verbinden und alles einrichten:

1. **SSH-Config** erstellen (`~/.ssh/config`) für einfachen Zugriff mit `ssh openclaw-server`:
   ```
   Host openclaw-server
     HostName [SERVER_IP]
     User root
     Port 22
     IdentityFile ~/.ssh/id_ed25519
   ```

2. **SSH-Verbindung testen**: `ssh openclaw-server`

3. **Firewall** konfigurieren:
   - `ufw default deny incoming && ufw default allow outgoing`
   - `ufw allow ssh && ufw enable`
   - Verifizieren mit `ufw status`

4. **Sicherheits-Tools** installieren:
   - `apt install -y fail2ban unattended-upgrades`
   - `systemctl enable --now fail2ban`

5. **Automatische Sicherheitsupdates** aktivieren

6. **Node.js 22** installieren

7. **OpenClaw** installieren (`npm install -g openclaw@latest`)

8. **OpenClaw Onboarding** durchführen
    - Workspace: `/root/clawd` (oder wie beim Onboarding gewählt)
    - Config: `/root/.openclaw/openclaw.json`

Wenn etwas schiefgeht, sage mir Bescheid. Ansonsten melde dich, wenn du fertig bist.

---

## Schritt 5: OAuth-Credentials einrichten (teilweise manuell)

OpenClaw nutzt mein Claude-Abo über OAuth — KEINEN Anthropic API Key.

Suche die Claude OAuth-Credentials auf meinem Computer:
- macOS: `~/Library/Application Support/Claude/`, `~/.claude/`, Keychain
- Windows: `%APPDATA%/Claude/`, Windows Credential Store
- Linux: `~/.claude/`, `~/.config/claude/`

Ich brauche den Access-Token (`sk-ant-oat01-...`) und falls vorhanden den Refresh-Token (`sk-ant-ort01-...`).
Falls du nichts findest: Führe `claude setup-token` aus und gib mir den Output.

Die Credentials speicherst du auf dem Server in:
`/root/.openclaw/agents/main/agent/auth-profiles.json`

---

## Schritt 6: Telegram-Bot erstellen (manuell)

Diesen Schritt machst du in Telegram:

1. Öffne Telegram und suche **@BotFather** (mit blauem Haken!)
2. Sende: `/newbot`
3. Wähle einen **Namen** für deinen Assistenten (z.B. "Mein KI-Assistent")
4. Wähle einen **Username** (muss auf "bot" enden, z.B. "mein_assistent_bot")
5. BotFather gibt dir einen **Token** — kopiere ihn und gib ihn mir

Ich konfiguriere dann Telegram auf dem Server und starte den Dienst.

---

## Schritt 7: Pairing (manuell)

1. Sende eine beliebige Nachricht an deinen neuen Bot in Telegram
2. Du bekommst einen **Pairing-Code** (z.B. "VVSHLNG8")
3. Gib mir den Code — ich genehmige dich auf dem Server

---

## Schritt 8: Dashboard und Abschluss (automatisch)

Erledige den Rest:

1. **Dashboard-Token** auslesen und in `.env` eintragen
2. **OpenClaw Dashboard-Launcher-Script** erstellen und im Projektordner `openclaw-setup` in einem Unterordner `openclaw-dashboard` ablegen
3. **CLAUDE.md** mit allen relevanten Infos befüllen, insbesondere:
   - Workspace: `/root/clawd`
   - Config: `/root/.openclaw/openclaw.json`
   - Service-Name und wie man ihn startet/stoppt
4. **Sicherheits-Check** durchführen:
   - Nur Port 22 von außen erreichbar
   - Keine unerwarteten Dienste
   - OAuth funktioniert
   - Telegram verbunden und getestet

Am Ende zeige mir eine Zusammenfassung mit dem Status aller Komponenten.

---

## Wichtige Regeln für dich (Claude)

- Warne IMMER vor Schritten, die Geld kosten
- Speichere ALLE Zugangsdaten ausschließlich in der `.env`
- Verwende `botToken` (nicht `token`) für die Telegram-Konfiguration
- OAuth-Credentials gehören in `~/.openclaw/agents/main/agent/auth-profiles.json` im folgenden Format:
`{"version":1,"profiles":{"anthropic:claude-cli":{"type":"oauth","provider":"anthropic","access":"ACCESS_TOKEN_HIER","refresh":"REFRESH_TOKEN_HIER_FALLS_VORHANDEN","expires":4102444800000}},"lastGood":{"anthropic":"anthropic:claude-cli"}}`
- Das Feld heißt `"type": "oauth"` — NICHT `authType`, NICHT `manual`, NICHT `setupToken`, NICHT `claudeAiOauth`
- Falls du sowohl Access-Token (`sk-ant-oat01-...`) als auch Refresh-Token (`sk-ant-ort01-...`) findest, übertrage BEIDE (Refresh-Token als `"refresh"` Feld)
- `ANTHROPIC_API_KEY` darf NICHT in der `openclaw.json` Config stehen — nur OAuth verwenden
- Firewall: NUR Port 22 darf offen sein — prüfe das am Ende
- Wenn der Nutzer nicht weiterkommt: Bitte um einen Screenshot
- Erkläre bei manuellen Schritten kurz, was passiert und warum
- Erkenne automatisch das Betriebssystem des Nutzers (macOS/Windows/Linux) und passe alle Befehle entsprechend an

- Erstelle am Ende eine vollständige CLAUDE.md, damit zukünftige Sessions sofort Kontext haben
- Frage den Nutzer NIEMALS nach einem Anthropic API Key. Schlage NIEMALS vor, einen API Key auf console.anthropic.com zu erstellen. Wir nutzen AUSSCHLIESSLICH OAuth über das bestehende Claude Pro/Max Abo. Wenn du einen Authentifizierungsfehler bekommst, prüfe und repariere das OAuth-Setup eigenständig — frage nicht nach einem API Key als Alternative.
- Führe alle Terminal-Befehle SELBST aus. Bitte den Nutzer NICHT, etwas im Terminal einzugeben oder zu kopieren. Du hast Zugriff auf das Terminal — nutze es. Der Nutzer soll nur bei Schritten aktiv werden, die explizit als "(manuell)" markiert sind (Hetzner-Account, Telegram-Bot, Pairing).

---

## Lizenz

[CC BY-NC 4.0](LICENSE) — © 2026 Ernst Neumeister / Neumeister Consulting GmbH
