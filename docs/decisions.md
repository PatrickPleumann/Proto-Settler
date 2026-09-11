# Entscheidungen

Nennenswerte Entwurfsentscheidungen als kurzer Eintrag: Entscheidung, Alternative,
Grund. Drei bis fünf Sätze. Neueste unten.

---

## Hilfsskripte in PowerShell statt Bash

*2026-09-10*

Die Skripte unter `tools/` werden in PowerShell geschrieben. Alternative war Bash
über Git Bash, weil die meisten Unity-Batchmode-Anleitungen so geschrieben sind und
eine spätere CI üblicherweise auf Linux läuft. Den Ausschlag gab, dass PowerShell
die Shell ist, die auf der Entwicklungsmaschine ohnehin benutzt wird, während Git
Bash ein zweites, fremdes Werkzeug mit eigenen Pfad-Eigenheiten wäre. Die Skripte
bleiben bewusst dünn: Die Build-Logik liegt in C# im Unity-Projekt, sodass eine
Bash-Variante für CI später ohne doppelte Logik nachziehbar ist.
