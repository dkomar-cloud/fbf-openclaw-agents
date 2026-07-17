# Runbook — Hermes-Modellwechsel: qwen3next-80b → Qwen3.5-122B-A10B

> **Stand:** 2026-07-17 · **Zweck:** Abgesicherter, reversibler Ablauf zum Prüfen und Umbau
> des Hermes-Primärmodells auf der GMKtec-Box. **Erst absichern und beweisen, dann umbauen.**
> **Ausführung:** server-Rolle der Flotte (bzw. Daniel) auf den echten Maschinen —
> dieses Repo enthält nur die Agent-Prompts, nicht die Live-Config.
> **Regelbindung:** CLAUDE.md §1 (IST/SOLL/RISIKO), §3 (ein Schritt, Freigabe abwarten),
> §6 (Backup-Pflicht vor Schreiben), §15 (ERFOLG-WENN vor Aktion). Jede destruktive
> Aktion steht hinter einem **Freigabe-Gate**.

---

## IST (Quelle: `~/flotte/INFRASTRUKTUR.md`, 15.07.2026)

| | Wert |
|---|---|
| Hermes-Primärmodell | `qwen3next-80b` — 80B total / **~3B aktiv (A3B)**, ~43 GB im VRAM |
| Host | GMKtec `nucbox-evo-x2` (Ryzen AI MAX+ 395), 128 GiB unified, **64 GiB fest als VRAM** |
| Fallback | `qwen3-nothink` 14,8B lokal am ai-server |
| Problem | Modell „macht was es will" — Ursache: nur ~3B aktive Parameter → wackelige Steuerbarkeit |

## SOLL

`qwen3next-80b` als Primary durch **Qwen3.5-122B-A10B** ersetzen (122B total / **10B aktiv**,
Q4 ~73–76 GB, verbessertes Tool-Calling, 256K Kontext, gleiche Qwen-Familie → Hermes-Prompts
greifen fast unverändert; bonus: multimodal → potenziell für gutachten-agent nutzbar).
`qwen3next-80b` bleibt als Fallback erhalten.

## RISIKO (übergreifend)

- **SPOF GMKtec:** Jeder Reboot (Phase 3) nimmt allen 7 Agenten + Hermes das Primärmodell →
  Rückfall auf 14,8B-Fallback. Nur im Wartungsfenster außerhalb der Bürozeit.
- **VRAM zu eng:** Bei 64 GB VRAM passt das 122B-Q4 (~75 GB) **nicht** mit Kontext-Reserve →
  VRAM-Split muss vorher angehoben werden (Phase 3).
- **Migrationsreibung:** minimal (Qwen-Familie), aber Chat-Template/Tool-Call-Format prüfen.

---

## Phase 0 — Baseline sichern *(read-only + Backup · risikofrei · kein Gate)*

Zustand einfrieren, damit Rollback ein Einzeiler ist.

- [ ] **Hermes-Modellkonfiguration lokalisieren** (exakter Pfad — server ermittelt, nicht raten)
      und mit Datum kopieren. Ebenso `~/.openclaw/openclaw.json` (Provider `gmktec`).
- [ ] `ollama list` + `ollama ps` auf der GMKtec → vorhandene Tags/Größen dokumentieren.
- [ ] Aktueller **VRAM-Split-Wert** notieren (BIOS UMA-Buffer bzw. AMD „Variable Graphics Memory").
- [ ] GPU-/ROCm-Status + belegter VRAM im Ruhezustand.

**ERFOLG-WENN:** Config-Backup existiert (Datei **+ Prüfsumme**), aktueller Split-Wert schriftlich
festgehalten. **BEWEIS:** Pfade + `sha256sum` der Backups, notierter Split-Wert.

## Phase 1 — Machbarkeit prüfen *(read-only · risikofrei · kein Gate)*

- [ ] **Freier Speicher** im Ollama-Modellordner der GMKtec (Windows) — Qwen3.5-122B-Q4 ~75 GB
      müssen zusätzlich passen.
- [ ] **Bezugsweg/Tag bestätigen:** Qwen3.5-122B-A10B via Ollama-Registry **oder** HuggingFace-GGUF
      (z. B. Unsloth Dynamic Q4_K_M) → dann per Modelfile in Ollama importieren. Exakten Tag festhalten.
- [ ] Netzpfad zum Pull gegen die Netzpolicy geprüft.
- [ ] VRAM-Rechnung gegengeprüft: 64 GB reichen **nicht** komfortabel → Ziel-Split (~80–96 GB) festlegen.

**ERFOLG-WENN:** genug Disk frei **und** Bezugsweg + Ziel-VRAM-Split feststehen.

## Phase 2 — Parallel-Test OHNE Eingriff ins Live-System *(Herzstück · risikofrei)*

Ollama hostet mehrere Modelle gleichzeitig — das Neue **neben** dem 80B, Live-Config unangetastet.

- [ ] Qwen3.5-122B-A10B unter **eigenem Tag** ziehen/importieren.
- [ ] Über **temporären Provider/Port** ansprechen — die 7 Agenten + Hermes laufen unverändert auf 80B.
- [ ] **A/B gegen echte Agenten-Aufgaben** (nicht Benchmarks):
      - Hält es das `RÜCKMELDUNG`-Format (STATUS/AUFGABE/BEFEHL/AUSGABE/BEWEIS/OFFEN)?
      - Respektiert es Freigabe-Gates (keine stillen Folgeaktionen)?
      - Valide Tool-Call-/JSON-Ausgabe?
      - Firmenwissen (FBF / B&H / SV-Komar) korrekt?
- [ ] Antwortgeschwindigkeit messen (10B aktiv ist langsamer als 3B — Tauglichkeit prüfen).

**ERFOLG-WENN (vorab fixieren):** neues Modell folgt in ≥ N/M Testfällen dem Format **und** schlägt
den 80B bei „macht was es soll" spürbar, bei akzeptabler Geschwindigkeit. **Sonst: Abbruch, kein Umbau.**

**→ FREIGABE-GATE 1:** Nur bei bestandenem A/B weiter zu Phase 3.

## Phase 3 — VRAM-Split anheben *(einziger harter Eingriff · Wartungsfenster)*

- [ ] **Wartungsfenster außerhalb Bürozeit** ankündigen (Ausfallfenster wie ~11 Min bei der Vault-Rotation).
- [ ] Alten Split-Wert (aus Phase 0) bestätigt dokumentiert.
- [ ] Split auf ~80–96 GB setzen (BIOS UMA bzw. AMD Variable Graphics Memory), Reboot.
- [ ] Nach Reboot: VRAM-Größe verifizieren, Qwen3.5-122B lädt sauber **mit** Kontext-Reserve.

**ERFOLG-WENN:** Box zeigt ~80–96 GB VRAM, Modell lädt mit Kontext-Puffer.
**ROLLBACK:** alten Split-Wert zurücksetzen, reboot.

**→ FREIGABE-GATE 2:** Vor dem Cutover.

## Phase 4 — Cutover *(reversibel · mit Freigabe + Backup)*

- [ ] **Backup vor Schreiben** (server-agent §6): Hermes-Config sichern.
- [ ] Primär-Modelltag in der Hermes-Config auf Qwen3.5-122B umstellen.
      **`qwen3next-80b` bleibt als Fallback eingetragen.**
- [ ] Gateway/Hermes neu starten.
- [ ] An 2–3 Live-Aufgaben verifizieren, kurz beobachten.

**ERFOLG-WENN:** Hermes antwortet über Qwen3.5-122B, Format-/Freigabe-Verhalten stabil.
**BEWEIS:** Live-RÜCKMELDUNG + `ollama ps` zeigt das neue Modell als aktiv.

**ROLLBACK (jederzeit, < 2 Min):** Config-Backup zurückspielen + Neustart → 80B wieder Primary.

---

## Zusammenfassung Freigabe-Gates

| Phase | Charakter | Freigabe nötig |
|---|---|---|
| 0 Baseline | read-only + Backup | nein |
| 1 Machbarkeit | read-only | nein |
| 2 Parallel-Test | kein Live-Eingriff | nein |
| **3 VRAM-Split** | Reboot/BIOS | **ja (Gate 1)** |
| **4 Cutover** | Config-Schreiben | **ja (Gate 2)** |

## Offene Punkte (server ermittelt, §2 — nicht raten)

- Exakter Pfad der **Hermes-Modellkonfiguration** (das Feld, das den Primär-Modelltag setzt).
- Exakter **Bezugsweg/Tag** für Qwen3.5-122B-A10B (Ollama-Registry vs. HF-GGUF + Modelfile-Import).
- Freier **Disk-Platz** auf der GMKtec und aktueller **VRAM-Split-Wert** (Phase 0/1).

## Nächster Schritt

server-Rolle startet **Phase 0 + 1** (nur lesen/sichern) und meldet zurück:
freier Speicher, aktueller VRAM-Split, vorhandene Modelle, Bezugsweg/Tag. Auf dieser
Faktenbasis wird Phase 2 entschieden.
