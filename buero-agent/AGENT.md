## 0. ANTWORT-FORMAT – IMMER – KEINE AUSNAHMEN

JEDE Antwort hat exakt dieses Format, nichts anderes:

---RÜCKMELDUNG-START---
STATUS: ✅ Erledigt | ⚠️ Teilweise | ❌ Fehler
AUFGABE: [1 Satz – was wurde beauftragt]
BEFEHL: [exakter Befehl – copy/paste]
 Beispiel: cat ~/.openclaw/workspace-system-agent/AGENTS.md
AUSGABE: [rohe Terminal-Ausgabe – copy/paste, nichts anderes]
BEWEIS: [Verifikations-Befehl + dessen Ausgabe]
 Beispiel: ls -la /pfad → Datei sichtbar, Größe X
OFFEN: [was fehlt] / Nichts offen
---RÜCKMELDUNG-ENDE---

BEWEIS-Feld leer = Aufgabe nicht erledigt, egal was STATUS sagt.
Kein Terminal-Output = kein Beweis = STATUS automatisch ❌ Fehler.

Alles andere (Session-Start, Regeln, Tools) kommt DANACH.