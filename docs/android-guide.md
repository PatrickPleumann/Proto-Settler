# Unity für Android – Leitfaden

Nachschlagewerk für dieses Projekt. Jeder Punkt ist ein Einstiegspunkt zum
Recherchieren, keine vollständige Erklärung.

Verhältnis zu `CLAUDE.md`: Dort stehen die **verbindlichen Regeln** dieses Projekts.
Hier steht das **Hintergrundwissen**, aus dem diese Regeln folgen. Wenn eine Regel
im CLAUDE.md unklar wirkt, findet sich hier meist der Grund.

---

## 1. Projekt-Setup und Build

- **Scripting Backend:** IL2CPP, Target Architecture ARM64. Play verlangt 64-Bit; ARMv7 nur mitnehmen, wenn du sehr alte Geräte wirklich brauchst.
- **Minimum API Level** bewusst wählen. Höher heißt weniger Kompatibilitätsarbeit, aber auch weniger erreichbare Geräte.
- **Target API Level:** Google Play erzwingt jährlich ein neues Minimum. Vor jedem Release den aktuellen Stand prüfen.
- **Keystore** früh anlegen, sichern und versionieren (nicht im öffentlichen Repo). Verlust bedeutet, dass die App nie wieder aktualisiert werden kann.
- **Gradle- und Manifest-Templates** im Projekt aktivieren und einchecken, statt sich auf Unity-Defaults zu verlassen.
- **APK** für Direkttests per adb, **AAB** (App Bundle) für Play.
- **Build-Dauer einplanen:** IL2CPP-Builds dauern Minuten. Der gesamte Arbeitsablauf muss darum herum gebaut sein.

## 2. IL2CPP und AOT-Kompilierung

- **Kein JIT.** Dynamische Codegenerierung wie `Reflection.Emit` oder `Expression.Compile()` funktioniert nicht.
- **Managed Code Stripping** entfernt scheinbar ungenutzten Code. Reflektierter Zugriff bricht dadurch erst zur Laufzeit, nicht beim Bauen.
- **`link.xml`** schützt ganze Assemblies oder Typen, `[Preserve]` einzelne Member.
- **Generics mit Value Types** brauchen AOT-generierten Code. Exotische Konstrukte können auf dem Gerät fehlschlagen.
- **Reflection-schwere JSON-Bibliotheken** sind riskant. Source Generators oder handgeschriebene Serialisierung sind der sichere Weg.
- **Api Compatibility Level** (.NET Standard vs. .NET Framework) entscheidet, was überhaupt verfügbar ist.
- **Merke:** Ein grüner Editor beweist über den Gerätebuild gar nichts.

## 3. Speicher und Garbage Collection

- **Incremental GC** in den Player Settings aktivieren. Verteilt Pausen über Frames statt einen Ruckler zu erzeugen.
- **Allokationen in `Update()`** sind die häufigste Ruckelursache: String-Verkettung, LINQ, Boxing, Closures, manche Enumeratoren.
- **Object Pooling** für alles, was häufig entsteht und vergeht.
- **OOM-Kill erfolgt ohne Vorwarnung.** Android meldet keinen sauberen Fehler, die App ist einfach weg. Speicherbudget je Geräteklasse festlegen.
- **Texturen dominieren den Speicher**, nicht der Code.
- **Memory Profiler** (Package) für Snapshots. Nativen und managed Speicher getrennt betrachten.

## 4. Assets und APK-Größe

- **Texturkompression:** ASTC für moderne Android-GPUs, ETC2 als Fallback. Unkomprimierte Texturen sind der klassische Größenfresser.
- **Android-Override im Import-Inspector** nutzen, insbesondere Max Texture Size. Gilt pro Asset.
- **Mipmaps** für Weltgeometrie ja, für UI-Elemente nein.
- **Audio:** Kurzes decompress on load, langes streaming. Vorbis statt WAV.
- **Addressables** oder Play Asset Delivery für Inhalte, die nicht ins Startpaket müssen.
- **Resources-Ordner meiden.** Alles darin landet im Build und wird beim Start indiziert.
- **Build Report** beziehungsweise `Editor.log` auswerten: Unity listet die größten Assets im Build auf.

## 5. Rendering auf Mobile-GPUs

- **Tile-Based Rendering** hat eine andere Kostenstruktur als Desktop. Speicherbandbreite ist teurer als Rechenleistung.
- **Overdraw ist der Hauptfeind**, vor allem bei großflächiger transparenter UI und Partikeln.
- **URP** statt Built-in Pipeline, dazu SRP Batcher und GPU Instancing.
- **MSAA ist auf Tile-GPUs vergleichsweise günstig**, Fullscreen-Post-Processing dagegen teuer, weil es Bandbreite kostet.
- **Framebuffer nicht zurücklesen.** Alles, was den Tile-Speicher zwingt, in den Hauptspeicher zu schreiben, ist teuer.
- **Render Scale senken** ist oft wirksamer als Effekte zu streichen.
- **`Application.targetFrameRate` explizit setzen.** 30 fps sind auf Mobilgeräten häufig die klügere Wahl als 60.
- **Draw Calls und SetPass Calls** im Frame Debugger prüfen. Batching bricht an Materialwechseln und Canvas-Rebuilds.

## 6. App-Lifecycle

- **`OnApplicationPause(true)`** ist der verlässliche Speicherzeitpunkt. `OnApplicationQuit` kommt auf Android oft nie.
- **Der Prozess kann im Hintergrund beendet werden.** Alles Ungespeicherte ist verloren.
- **Beim Zurückkehren** kann die Activity neu erzeugt und der GL-Kontext verloren sein. Render Textures und ähnliche Ressourcen müssen das überstehen.
- **Auflösungswechsel zur Laufzeit** durch Splitscreen, Multi-Window oder Foldables abfangen.
- **Focus und Pause unterscheiden** (`OnApplicationFocus` vs. `OnApplicationPause`).
- **Kein Verlass auf Timer**, die nur laufen, während die App im Vordergrund ist.

## 7. Dateisystem und Persistenz

- **`Application.persistentDataPath`** ist der einzige zuverlässig beschreibbare Ort. `dataPath` ist es nicht.
- **StreamingAssets liegt im APK.** Zugriff nur über `UnityWebRequest`, klassisches File-IO schlägt fehl.
- **PlayerPrefs** ist für Kleinigkeiten gedacht, nicht für Spielstände. Keine Struktur, keine Atomarität.
- **Atomar speichern:** in temporäre Datei schreiben, dann umbenennen. Sonst entstehen kaputte Saves, wenn das OS mitten im Schreiben zuschlägt.
- **Save-Versionierung ab dem ersten Tag.** Migrationspfade nachzurüsten ist deutlich teurer.
- **Android-Backup-Verhalten** (`allowBackup`) beeinflusst, was Nutzer nach einem Gerätewechsel wiederbekommen.

## 8. Input und UI

- **Kein Hover, kein Rechtsklick, kein Scrollrad, keine Tastatur.** Jede darauf gebaute Affordanz muss ersetzt werden.
- **Multitouch und Gesten** wie Pinch-Zoom selbst umsetzen oder das Input System nutzen.
- **Zurück-Geste und Zurück-Taste** abfangen, sonst beendet der Nutzer versehentlich die App.
- **Bildschirmtastatur** verdeckt Eingabefelder. `TouchScreenKeyboard`-Verhalten auf echten Geräten prüfen.
- **Canvas-Rebuilds sind teuer.** Statische und häufig aktualisierte Elemente auf getrennte Canvases legen.
- **Raycast Target** überall abschalten, wo nichts angeklickt werden muss.
- **Mindestens 48 dp** für jedes Touch-Ziel.

## 9. Bildschirmvielfalt

- **Seitenverhältnisse** reichen von 16:9 bis über 21:9, dazu kommen Foldables mit wechselnder Geometrie.
- **`Screen.safeArea`** auswerten für Notch, Punch-Hole und Gestenleiste.
- **Canvas Scaler** auf Scale With Screen Size, Match-Wert bewusst wählen. In dp denken, nicht in Pixeln.
- **Device Simulator** (Package) ist gut für Layoutprüfungen, ersetzt aber keine Messung auf echter Hardware.
- **Auf mindestens zwei Geräteklassen** prüfen, idealerweise ein schmales und ein sehr breites Gerät.

## 10. Profiling und Debugging auf dem Gerät

- **Editor-Profiling sagt fast nichts** über die Laufzeit auf dem Gerät aus. Immer per Profiler über ADB oder WLAN am Gerät messen.
- **Development Build** plus Autoconnect Profiler. Deep Profiling verzerrt stark, nur punktuell einsetzen.
- **`adb logcat -s Unity`** für Laufzeitfehler. Für Crashes beim Build Symbole erzeugen lassen, sonst ist der Stacktrace unlesbar.
- **Frame Debugger** für Draw Calls, **Android GPU Inspector** oder Herstellerwerkzeuge für GPU-Details.
- **Perfetto** für die Systemebene: CPU-Takt, Scheduling, Throttling.
- **Release-Build separat prüfen.** Er verhält sich anders als der Development-Build.

## 11. Thermik und Akku

- **Leistung sinkt im Betrieb.** Nach zehn Minuten messen, nicht nach zehn Sekunden.
- **Die ersten Minuten lügen**, weil das Gerät auf Boost-Takt läuft.
- **Framerate begrenzen** spart mehr Akku als jede Mikrooptimierung im Code.
- **Nicht rendern, wenn sich nichts ändert.** On-Demand Rendering oder reduzierte Framerate im Leerlauf.
- **Akkusparmodus** drosselt zusätzlich. In diesem Zustand mindestens einmal testen.
- **Kaltstartzeit messen.** Sie ist für Nutzer und Store-Bewertung relevant.

## 12. Auslieferung über Google Play

- **AAB ist Pflicht** für neue Uploads, APK bleibt für lokale Tests.
- **Play App Signing:** Upload-Key und Signing-Key sind verschiedene Dinge. Den Unterschied vor dem ersten Upload verstehen.
- **Größenlimits** für Download und Installation beachten. Play Asset Delivery für großes Zusatzmaterial.
- **Jede Berechtigung im Manifest** muss begründbar sein. Unity fügt manche automatisch hinzu, prüfe das erzeugte Manifest.
- **Datenschutz- und Datensicherheitsangaben** sind Pflichtfelder und verzögern Releases, wenn sie fehlen.
- **Interne Testspur** nutzen, statt sich nur auf lokal installierte Builds zu verlassen.

## 13. Klassiker: im Editor grün, auf dem Gerät rot

- Reflektierter Code wurde durch **Stripping** entfernt.
- Shader-Variante fehlt, weil sie beim **Shader Stripping** herausgefallen ist. Variant Collections prüfen.
- **Pfade sind auf Android case-sensitiv**, im Editor unter Windows nicht.
- **Fehlende Laufzeitberechtigung**, die im Editor niemand braucht.
- **Plattform-Defines** (`UNITY_EDITOR` vs. `UNITY_ANDROID`) verdecken echte Unterschiede.
- **Natives Plugin** liegt nicht für die gebaute ABI vor.
- **Float-Verhalten** unterscheidet sich zwischen x86-Editor und ARM-Gerät. Für alles Deterministische ein echtes Problem.

## 14. Arbeitsweise

- **Erste APK am ersten Tag**, nicht am Ende des Projekts.
- **Jeder Meilenstein wird auf dem Gerät verifiziert**, nicht nur im Editor.
- **Ein günstiges Altgerät im Bestand halten.** Teure Geräte verstecken Probleme, die deine Nutzer haben.
- **Emulator nur für Funktionstests**, niemals für Performanceaussagen.
- **Spiellogik engine-frei und testbar halten**, damit du nicht für jeden Logikfehler einen Gerätebuild brauchst.

---

## Reihenfolge fürs Durcharbeiten

Für dieses Projekt zahlen sich zuerst aus: **2** (IL2CPP), **6** (Lifecycle),
**7** (Persistenz), **8** und **9** (Touch und Bildschirme). Sie betreffen die
Kernarchitektur.

Zurückstellen kannst du **5** (Rendering) und **11** (Thermik), weil das Genre nicht
performancekritisch ist. **12** (Play) wird erst relevant, wenn du tatsächlich
veröffentlichst.

## Hinweis zur Aktualität

Stand der Zusammenstellung: September 2026. Versionsabhängige Angaben, insbesondere
zu Google-Play-Anforderungen, Unity-Paketnamen und Player Settings, vor Gebrauch
gegen das Unity-Handbuch (Platform development / Android) und die Google Play
Console prüfen. Konkrete API-Level-Zahlen stehen bewusst nicht drin, sie veralten
jährlich.
