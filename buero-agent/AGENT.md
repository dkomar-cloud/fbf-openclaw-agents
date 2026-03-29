## Timeout-Regeln

- **Standard-Tasks**: 5 Minuten Wall-Clock-Time (max. 5 Minuten Ausführungszeit)
- **Komplexe Tasks**: 10 Minuten (explizit in der Aufgabenbeschreibung deklarieren)
- **Subagent-Spawns**: 2 Minuten eigenes Timeout (für untergeordnete Agenten)
- **Externe Calls**: 60 Sekunden mit Exponential Backoff (1s → 2s → 4s → 8s, max. 3 Retries)

## Heartbeat-Pflicht

- **Langlaufende Tasks**: Fortschritt alle 60 Sekunden melden
- **Stall-Detektion**: Kein Heartbeat nach 2x Intervall (120 Sekunden) → Task als "stalled" markieren

## Loop Detection

- **Max. Iterationen**: 25 LLM-Calls pro Task (Verhindert Endlosschleifen)
- **Tool-Call-Signatur**: Gleiche Signatur 3x → sofortiger Abbruch

## Config-Operationen

- **Direktes Datei-Editieren**: Verboten → immer `openclaw config set` verwenden
- **API-Fehler**: Circuit Breaker nach 5 Fehlern innerhalb 60 Sekunden

## Failure-Verhalten

- **Nicht-kritische Fehler**: Graceful Degradation (z. B. alternativer Pfad, Redundanz)
- **Retry-Strategie**: Backoff (1s → 2s → 4s → 8s), dann Escalation an Keks
- **Kritische Fehler**: Nie stille Fortsetzung → sofortige Escalation und Blockierung