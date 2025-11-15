# Project: HyperTube_Rush 

Ein leichtgewichtiger, offline-fähiger 3D-Endless-Runner für Android, entwickelt in Unity. Inspiriert von Spielen wie "Tunnel Rush", fokussiert sich dieses Projekt auf reaktionsschnelles Gameplay, hohe Performance auf mobilen Geräten und eine saubere, skalierbare Codebasis.



---

## 🎮 Features

* **Endloses Gameplay:** Ein prozedural generierter Tunnel sorgt für unendliche Wiederspielbarkeit.
* **360°-Steuerung:** Der Spieler rotiert frei um die Z-Achse, um Hindernissen auszuweichen.
* **Dynamische Schwierigkeit:** Die Spielgeschwindigkeit erhöht sich kontinuierlich, je länger der Spieler überlebt.
* **Highscore-System:** Lokale Speicherung des besten Scores (via `PlayerPrefs`).
* **Optimiert für Mobile:** Implementiert mit Object Pooling und Fokus auf Minimierung der Garbage Collection.
* **Einfache State-Machine:** Klar definierte Spielzustände (MainMenu, Running, GameOver).

---

## 🛠️ Technische Umsetzung

* **Engine:** Unity (z.B. 2022.3.x LTS)
* **Sprache:** C#
* **Plattform:** Android
* **Kern-Konzepte:**
    * **Object Pooling:** Der `TunnelSpawner` recycelt Tunnelsegmente, um `Instantiate()`/`Destroy()`-Aufrufe zur Laufzeit zu vermeiden.
    * **Conveyor-Belt-Prinzip:** Der Spieler steht still (Position 0,0,0). Die Welt (`WorldMover`) bewegt sich auf den Spieler zu.
    * **State-Management:** Ein zentraler `GameManager` steuert den Spielfluss.

---

## ⚙️ Systemarchitektur (Kern-Skripte)

Dieses Projekt nutzt ein klares, entkoppeltes System, das über den `GameManager` koordiniert wird.

* **`GameManager.cs` (Core):**
    * Der "Dirigent" des Spiels.
    * Verwaltet die State-Machine (`MainMenu`, `Running`, `GameOver`).
    * Koordiniert `UIManager`, `ScoreManager` und `DifficultyManager`.

* **`PlayerController.cs` (Player):**
    * Verarbeitet Touch-Input (Drag) und wandelt ihn in eine 360°-Rotation des Spielers um.
    * Rotiert nicht direkt, sondern setzt eine `targetRotation`, um Stabilität zu gewährleisten.

* **`PlayerCollision.cs` (Player):**
    * Erkennt Kollisionen mit Objekten auf dem "Obstacle"-Layer.
    * Meldet dem `GameManager` das Spielende (`EndGame()`).

* **`WorldMover.cs` (Gameplay):**
    * Einzelnes Skript auf einem Parent-GameObject (`[WorldMover]`).
    * Bewegt alle Kinder (Tunnel, Hindernisse) mit `currentSpeed` auf den Spieler zu.

* **`TunnelSpawner.cs` (Gameplay):**
    * Implementiert ein **Object Pooling**-System für `TunnelSegment_Prefab`s.
    * Sorgt dafür, dass der Tunnel nahtlos und endlos generiert wird.

* **`SegmentCleanup.cs` (Gameplay):**
    * An jedem Tunnelsegment.
    * Meldet dem `TunnelSpawner`, wenn es weit genug hinter dem Spieler ist, um recycelt zu werden.

* **`DifficultyManager.cs` (Core):**
    * Erhöht linear die `currentSpeed` im `WorldMover` basierend auf der Spielzeit.

* **`ScoreManager.cs` (Core):**
    * Berechnet den Score basierend auf Zeit und aktueller Geschwindigkeit.
    * Speichert und lädt den Highscore.

* **`UIManager.cs` (UI):**
    * Steuert die Sichtbarkeit der UI-Panels (`MainMenu`, `InGame`, `GameOver`).
    * Aktualisiert alle Textanzeigen (Score, Highscore).


##Projektstruktur

* Assets/Audio
* Assets/Audio/Music
* Assets/Audio/SFX
* Assets/Fonts
* Assets/Materials
* Assets/Prefabs
* Assets/Prefabs/Environment
* Assets/Prefabs/Gameplay
* Assets/Scenes
* Assets/Scripts
* Assets/Scripts/Core
* Assets/Scripts/Gameplay
* Assets/Scripts/Player
* Assets/Scripts/UI
* Assets/Settings
* Assets/Textures
* Assets/TextMeshPro



## 🚀 Zukünftige Ideen (Roadmap)

* [ ] Integration von Sounds (Musik, Kollisions-SFX)
* [ ] Partikeleffekte (Kollision, Speed-Lines)
* [ ] Skin-System für den Spieler
* [ ] Power-Ups (z.B. Unverwundbarkeit, Slow-Mo)
* [ ] Verschiedene Tunnel-Themen/Styles
