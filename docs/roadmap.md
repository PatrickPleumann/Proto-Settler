# Fahrplan

Die Etappen vom Projektstart bis zur Definition of Done (`CLAUDE.md`), geordnet nach
Risiko, nicht nach Features. Aufgaben stehen hier bewusst grob; die feinen Aufgaben
entstehen bei der Standortbestimmung in `current.md`.

Agile Einordnung (Kanban): Der Fahrplan entspricht dem Product Backlog, `current.md`
der Spalte „In Arbeit", `done.md` der Spalte „Erledigt".

**Testgeräte**

| Gerät | Rolle | Android | Seitenverhältnis | Besonderheit |
|---|---|---|---|---|
| Samsung Galaxy Z Flip 6 | primär | 14+ | 22:9 | Foldable, Außendisplay |
| Samsung Galaxy S20 FE | Leihgerät | 13 (letzte) | 20:9 | schwächere Hardware |

---

## ✅ Etappe 0 — Projektgerüst

Ordnerstruktur laut `CLAUDE.md`, Unity-Projekt in `unity/`, `docs/` mit Platzhaltern.

## ⬜ Etappe 1 — Leere APK auf dem Gerät

Die größte Unbekannte zuerst: Kommt überhaupt etwas aufs Handy?

- `rules-android-build.md` befüllen
- Player Settings: Portrait, IL2CPP, ARM64, Paketname; minimale Android-Version
  höchstens Android 13, damit das S20 FE mitläuft
- `BuildScript.cs` + `tools/build-android.ps1` und `tools/deploy-android.ps1`
- Unity-Vorlagenreste entfernen (TutorialInfo, Readme)
- Budgets festlegen und Ausgangswert messen: APK-Größe, Speicher

**Fertig, wenn:** Ein Befehl baut die APK, installiert und startet sie. Die App läuft
hochkant auf dem Z Flip 6, und eine Log-Zeile ist über `adb logcat` sichtbar.

## ⬜ Etappe 2 — Kern-Gerüst, Tests, CI

- `core/` und `core.tests/` anlegen, `dotnet test core.tests` läuft
- Entscheiden, wie Unity den Kern einbindet (→ `decisions.md`)
- GitHub Actions führt die Kern-Tests bei jedem Push aus, Badge ins README

**Fertig, wenn:** Ein Wert aus `Game.Core` erscheint in der App auf dem Gerät (beweist,
dass der Kern den IL2CPP-Build übersteht), und die CI ist grün.
· DoD: CI

## ⬜ Etappe 3 — Dünner Durchstich: Zustand, Befehl, Speichern

- `rules-time-persistence.md` befüllen
- Minimaler Zustand: ein Feld, eine Pflanzenart aus JSON, ein Command, ein Event,
  ein Knopf
- Mitgelieferte Daten aus StreamingAssets über `UnityWebRequest`
- Serialisierung ohne Reflection, Speichern bei `OnApplicationPause(true)`

**Fertig, wenn:** Auf dem Gerät säen, App hart beenden, neu starten — die Pflanze ist
noch da. Ebenso nach Zuklappen und Aufklappen des Z Flip 6.
· DoD: Save überlebt App-Kill

## ⬜ Etappe 4 — Zeit, Offline-Fortschritt, Migration

- Absolute UTC-Zeitpunkte, Offline-Berechnung mit Obergrenze
- Schutz gegen zurückgestellte Uhr
- Golden-Hash: Prüfsumme über den Spielzustand; gleich auf Desktop und Gerät heißt,
  die Regeln rechnen auf beiden identisch
- Eine echte Save-Migration v1 → v2

**Fertig, wenn:** Tests für 5 min, 8 h, 3 Tage, Obergrenze und Uhr-zurück sind grün, und
der Golden-Hash ist auf Desktop und Gerät identisch.
· DoD: Offline-Fortschritt, Migration, zurückgestellte Uhr, Golden-Hash

## ⬜ Etappe 5 — Interaktionsmodell und Layout

- `rules-ui-layout.md` befüllen: Touch-Gesten, Daumenzonen, Layout hochkant
- Kamera: Verschieben und Zoomen per Touch
- Foldable: Gesten über den Knick hinweg prüfen; Außendisplay und Flex-Modus
  (halb aufgeklappt) bewusst nicht unterstützen (→ `decisions.md`)

**Fertig, wenn:** Die Bedienung funktioniert einhändig auf dem Z Flip 6 und auf dem
S20 FE.
· DoD: Portrait auf Testgerät und zweitem Seitenverhältnis

## ⬜ Etappe 6 — Die Spielschleife

- `rules-economy.md` befüllen
- Säen → Wachsen → Ernten → Verkaufen → Ausbauen, vollständig aus Daten
- Content-Hot-Reload über `tools/push-data.ps1`

**Fertig, wenn:** Eine erste Session trägt 10–15 Minuten, und der Offline-Fortschritt
macht das Wiederkommen lohnend.

## ⬜ Etappe 7 — Harte Währung und Time-Skip

- Shop gegen einen Mock, kein echtes Bezahlsystem
- Gutschrift idempotent: doppelt ausgelöst wird trotzdem nur einmal gutgeschrieben

**Fertig, wenn:** Ein Time-Skip funktioniert Ende zu Ende, auch bei App-Kill mitten im
Kauf.
· DoD: Time-Skip

## ⬜ Etappe 8 — Onboarding

**Fertig, wenn:** Eine fremde Person versteht die Schleife ohne Textwand.
· DoD: Onboarding

## ⬜ Etappe 9 — Messen und Abschluss

- `performance.md`: APK-Größe, Speicherverlauf über 30 Minuten, eine belegte
  Optimierung; Gegenprüfung auf dem S20 FE als schwächerem Gerät
- README mit Video, Architekturdiagramm und drei Absätzen Begründung
- `learnings.md` vollständig, Definition of Done durchgehen

**Fertig, wenn:** Alle Häkchen der Definition of Done sind gesetzt.
