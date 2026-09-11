# CLAUDE.md

Diese Datei ist der **Kompass**. Sie enthält, was immer gilt, unabhängig davon,
woran gerade gearbeitet wird. Alles, was nur bei bestimmten Tätigkeiten fällig
wird, steht in den Modulen unter "Kontextdateien".

---

## Projekt

Vertical Slice eines Farm- und Homestead-Builders für **Android**. Ein kleines
Projekt aus Lust am Thema: im Umfang bewusst klein, technisch ernst gemeint.

Kein kommerzielles Produkt, kein Feature-Wachstum. Erste Session im Ziel: 10–15
Minuten, danach trägt der Offline-Fortschritt.

Engine: **Unity**. Spielkern: **pure C#, engine-frei**. Ausgeliefert wird
ausschließlich Android in **Portrait**. Der PC ist Entwicklungsvehikel, kein Ziel.

Das Repository ist so wichtig wie der Build. Lesbarkeit, Tests und
nachvollziehbare Entscheidungen haben Vorrang vor Feature-Umfang.

---

## Lernziel

Wichtigstes Ziel dieses Projekts ist zu verstehen, **warum für Android anders
geplant wird als für ein Wegwerf-Prototyp am Desktop**. Alles folgt aus einer
Tatsache: Entwicklungsmaschine und Zielmaschine sind verschieden. Daraus:

| Zwang | Folge im Projekt |
|---|---|
| Feedback-Loop dauert Minuten statt Sekunden | engine-freier Kern, Tests ohne Unity, Content-Hot-Reload |
| Das OS killt den Prozess jederzeit | Speichern bei Pause, Zustand als klar umrissenes Objekt |
| Die Welt läuft weiter, während die App zu ist | absolute Zeitstempel, Offline-Berechnung, Save-Migration |
| Kein Hover, kein Rechtsklick, 9 mm Ungenauigkeit | Interaktionsmodell vor der UI festlegen |
| Ein Daumen, hochkant, unterwegs | Zonen-Layout, Interaktion im unteren Bildschirmteil |
| Speicher, APK-Größe und Akku sind harte Grenzen | Budgets statt Optimierung im Nachhinein |
| IL2CPP/AOT-Build ist ein anderes Programm als der Editor | früh und oft bauen, Stripping-Risiken sofort aufdecken |

Vorgehen ist deshalb **risikogetrieben, nicht featuregetrieben**: Erst eine leere
APK auf dem Gerät, dann ein dünner Pfad durch alle Schichten inklusive Speichern,
App-Kill und Offline-Fortschritt, erst danach Features.

Merksatz: Android verzeiht nichts später, was früh nicht entschieden wurde.

---

## Nicht verhandelbar

1. **Android ist Zielplattform, nicht Portierung.** Ein Feature gilt erst als fertig,
   wenn es als APK auf einem echten Gerät läuft. Nicht im Emulator.
2. **Der Android-Build bleibt grün.** Bricht er, ist der Stand kaputt, auch wenn im
   Editor alles läuft.
3. **`Game.Core` enthält null Referenzen auf `UnityEngine`.** Per asmdef erzwungen.
   Wenn du dort ein `using UnityEngine` brauchst, ist der Entwurf falsch.
4. **Der Spielzustand ist ganzzahlig und serialisierbar.** Keine Floats, keine
   Engine-Typen, keine Objektreferenzen ohne stabile ID.
5. **Kein Content im Code.** Pflanzen, Gebäude, Rezepte, Zeiten, Preise und
   Balancing liegen in Daten.

Als Leitlinie, nicht als harte Regel: **Plattformneutralität mitnehmen, wo sie
nichts kostet.** Entscheidungen, die eine spätere PC-Version offenhalten und
Mobile nicht belasten, werden so getroffen. Bei echtem Zielkonflikt gewinnt
Mobile, und der Fall wird in `docs/decisions.md` festgehalten.

---

## Stille Fehler

Diese Punkte stehen hier und nicht in einem Modul, weil ihr Verstoß im Editor
unsichtbar bleibt und erst auf dem Gerät auffällt. Details jeweils im genannten
Modul.

- **StreamingAssets nur über `UnityWebRequest`.** File-IO funktioniert im Editor und
  scheitert auf dem Gerät. → `rules-android-build.md`
- **Keine reflektionsabhängige Serialisierung.** IL2CPP-Stripping entfernt sie
  lautlos. → `rules-android-build.md`
- **Timer sind absolute UTC-Zeitpunkte, niemals Restlaufzeiten.**
  → `rules-time-persistence.md`
- **Gespeichert wird bei `OnApplicationPause(true)`.** Auf `OnApplicationQuit` ist
  auf Android kein Verlass. → `rules-time-persistence.md`
- **Keine Iteration über `Dictionary` oder `HashSet` in Regel-Logik.** Die Reihenfolge
  ist Teil des Zustands. → `rules-time-persistence.md`

---

## Aufbau

```
core/            Game.Core       – netstandard2.1, engine-frei, Zustand + Regeln
core.tests/      Game.Core.Tests – NUnit, läuft ohne Unity
unity/           Unity-Projekt   – ausschließlich Präsentation, Input, UI
  Assets/Scripts/Presentation
  Assets/StreamingAssets/data    – Content- und Layout-JSON, wird mitgebaut
docs/            Regelmodule, Ressourcen, Entscheidungen, Learnings, Messwerte
builds/          Artefakte (git-ignoriert)
```

Datenfluss ist einseitig: Presentation schickt **Commands** in den Kern, der Kern
gibt **Events** und einen lesbaren Zustand zurück. Presentation schreibt niemals
direkt in den Zustand.

---

## Kontextdateien

Diese Dateien werden **nicht automatisch geladen**. Sie werden gelesen, sobald der
Auslöser zutrifft, und zwar **bevor Code entsteht**, nicht nachdem etwas kaputt ist.

| Auslöser | Datei |
|---|---|
| Bevor du Code schreibst, der Zeit, Timer, Spielstände oder das Save-Schema berührt | `docs/rules-time-persistence.md` |
| Bevor du Code schreibst, der Währungen, Preise, Time-Skips oder Käufe berührt | `docs/rules-economy.md` |
| Bevor du UI, Layout, Kamerasteuerung oder Eingaben anlegst oder änderst | `docs/rules-ui-layout.md` |
| Bevor du Player Settings, Manifest, Gradle, Serialisierung, Content-Laden oder Abhängigkeiten anfasst | `docs/rules-android-build.md` |
| Wenn ein Fehler nur auf dem Gerät auftritt, im Editor aber nicht | `docs/android-guide.md`, Abschnitt „Klassiker: im Editor grün, auf dem Gerät rot" |
| Bevor du ein Unity-Thema anfasst, das im Projekt neu ist (Texturen, Audio, Profiling, Auslieferung) | `docs/android-guide.md` |

Regeln für den Umgang mit diesen Dateien:

- **Bei Widerspruch gilt diese Datei.** Melde den Widerspruch und korrigiere das
  Modul, statt eigenmächtig zu entscheiden.
- **Fehlt ein Modul, sag es**, statt die Regeln zu erraten.
- **Verweise gehen auf Überschriften, nicht auf Nummern.** Nummern verrutschen
  unbemerkt.
- `docs/android-guide.md` ist eine Nachschlage-Ressource ohne bindende Wirkung. Die
  Regelmodule sind bindend.

---

## Arbeitsweise

- Bei Aufgaben über eine Datei hinaus: erst Plan vorschlagen, dann implementieren.
- Vor jedem Commit muss `dotnet test core.tests` durchlaufen.
- Vor jedem Meilenstein-Commit zusätzlich ein Deploy auf das Gerät und ein
  Funktionscheck dort.
- Kleine, thematisch saubere Commits mit aussagekräftiger Message. Kein Squashing
  der Historie, sie ist Teil der Projektdokumentation.
- Neue Abhängigkeiten nur nach Rückfrage. Jede Dependency muss auf Android laufen
  und IL2CPP-tauglich sein.
- Nennenswerte Entwurfsentscheidungen als kurzer Eintrag in `docs/decisions.md`:
  Entscheidung, Alternative, Grund. Drei bis fünf Sätze.
- **Wenn eine Entscheidung durch die Android-Plattform erzwungen ist, benenne das
  ausdrücklich**, statt sie stillschweigend umzusetzen. Kurz: welcher Zwang, wie
  sähe die Desktop-Lösung aus, warum trägt sie hier nicht. Solche Fälle sammeln
  sich in `docs/learnings.md` und sind Teil des Projektziels, nicht Overhead.
- Kommentare erklären das Warum. Kein Kommentar, der den Code nacherzählt.

---

## Kommandos

```powershell
dotnet test core.tests                      # Kern-Tests, schnell, laufen ohne Unity
.\tools\build-android.ps1                   # Unity batchmode -> builds/app.apk
.\tools\deploy-android.ps1                  # baut, adb install -r, startet App
.\tools\push-data.ps1                       # Content-/Layout-JSON aufs Gerät, ohne Rebuild
adb logcat -s Unity                         # Laufzeit-Logs vom Gerät
```

Die Skripte sind PowerShell, weil das die Shell auf der Entwicklungsmaschine ist.
Sie bleiben dünn: Die eigentliche Build-Logik liegt als C#-Datei im Unity-Projekt
und wird per `-executeMethod` angesprungen. Das Skript findet nur Unity, ruft es
auf und reicht den Exit-Code weiter.

Ein Windows-Build darf zum schnellen Ausprobieren existieren, ist aber kein
Liefergegenstand und wird nicht gepflegt.

---

## Nicht bauen

Eigene Engine. Multiplayer. Echtes Billing-SDK, Server-Backend oder Netzwerkschicht.
Landscape-Ansicht. Gepflegter PC-Build. ECS ohne gemessenen Gewinn. Prozedurale
Generierung über das für den Slice Nötige hinaus. Weitere Content-Domänen, solange
die erste nicht rund ist.

---

## Definition of Done für den Slice

- [ ] APK läuft in Portrait auf dem Testgerät und auf einem zweiten Seitenverhältnis
- [ ] Offline-Fortschritt korrekt nach 5 Minuten, 8 Stunden und 3 Tagen, Cap greift
- [ ] Save überlebt App-Kill, Neustart und eine Schema-Migration
- [ ] Zurückgestellte Uhr erzeugt keinen Fortschritt und keinen Schaden
- [ ] Time-Skip mit harter Währung funktioniert Ende zu Ende gegen den Mock,
      Gutschrift ist idempotent
- [ ] Golden-Hash identisch auf Desktop und Gerät. Der Desktop-Wert kommt aus
      `core.tests`, nicht aus einem Unity-Windows-Build.
- [ ] Onboarding erklärt den Loop ohne Textwand
- [ ] `README.md` mit Video (60–90 s), einem Architekturdiagramm und drei Absätzen
      Begründung
- [ ] `docs/performance.md` mit APK-Größe, Speicherverlauf über 30 Minuten und einer
      belegten Optimierung
- [ ] `docs/learnings.md` enthält die plattformbedingten Entscheidungen
- [ ] CI läuft Kern-Tests bei jedem Push, Badge im README
