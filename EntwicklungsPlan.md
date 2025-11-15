Entwicklungsplan: "Project TunnelRun" (Arbeitstitel)

1. Projektstruktur & Unity-Setup

1.1. Szenen
Wir verwenden eine modulare Multi-Scene-Architektur für bessere Skalierbarkeit und Wartbarkeit.

```
Assets/Scenes/
├── Persistent.unity          // Core-Scene (läuft IMMER, wird nie entladen)
├── MainMenu.unity            // Hauptmenü
├── Game.unity                // Gameplay (bisheriger Tunnel-Runner)
├── Shop.unity                // Platzhalter für zukünftige IAP/Skins
└── Settings.unity            // Platzhalter für Optionen
```

**Beschreibung:**
- **Persistent.unity**: Enthält GameManager, SceneLoader, ServiceLocator, AudioManager (später). Nutzt DontDestroyOnLoad() und bleibt während der gesamten App-Laufzeit aktiv.
- **Additive Scene Loading**: Alle anderen Scenes werden additiv zur Persistent-Scene geladen/entladen.
- **Vorteile**: Saubere Trennung von Core-Logik und Content, bessere Memory-Verwaltung, einfache Erweiterbarkeit.

1.2. Ordnerstruktur
Eine saubere Struktur ist essenziell für die Wartbarkeit.

```
Assets/Audio
Assets/Audio/Music
Assets/Audio/SFX
Assets/Fonts
Assets/Materials
Assets/Prefabs
Assets/Prefabs/Environment
Assets/Prefabs/Gameplay
Assets/Scenes
Assets/Scripts
Assets/Scripts/Core
Assets/Scripts/Gameplay
Assets/Scripts/Player
Assets/Scripts/UI
Assets/Scripts/Services     // Für zukünftige Monetarisierung
Assets/Settings
Assets/Textures
Assets/TextMeshPro
```

1.3. Unity-Projekteinstellungen
Build Settings:
- Plattform: Android.
- Texturkomprimierung: ASTC oder ETC2 (je nach Zielgeräten).
- Scripting Backend: IL2CPP (für Performance).
- API Compatibility Level: .NET Standard 2.1 (oder .NET Framework).

Player Settings (Mobile):
- Default Orientation: Landscape Left. (Auto-Rotation deaktivieren).
- Status Bar: Hidden.

Quality Settings:
- Erstellen Sie ein "Mobile" Quality Level.
- V-Sync: Don't Sync (wir steuern Framerate manuell oder cappen sie).
- Anti-Aliasing: Deaktiviert oder 2x (testen).
- Shadows: Disable Shadows.

Physics Settings:
- Gravity: (0, 0, 0) (Wir benötigen keine Schwerkraft).
- Default Contact Offset: 0.01 (Standard).

Time Settings:
- Fixed Timestep: 0.02 (Standard 50Hz).

2. Alle GameObjects (Hierarchie in GameScene)

Name (in Hierarchie) | Zweck | Unity-Komponenten | Layer / Tag | Skripte | Wichtige Inspector-Werte
---------------------|-------|------------------|-------------|---------|-------------------------
[SYSTEMS] (Empty GO) | Organisatorischer Container | Transform | - | - | Position: (0,0,0)
├─ GameManager | Singleton. Steuert Spielzustand, Score, Difficulty. | Transform | Untagged | GameManager, ScoreManager, DifficultyManager | -
├─ [UI] (Empty GO) | Organisatorischer Container | Transform | - | - | Position: (0,0,0)
│  ├─ UI_Canvas | Haupt-Canvas für alle UI-Elemente. | Canvas, CanvasScaler, GraphicRaycaster | UI | UIManager | Canvas: Render Mode: Screen Space - Overlay. Scaler: UI Scale Mode: Scale With Screen Size (z.B. 1920x1080)
│  │  ├─ MainMenu_Panel | Container für Hauptmenü-UI. | RectTransform, Image (optional) | UI | - | Aktiv/Inaktiv je nach State
│  │  │  ├─ Title_Text | Spieltitel | RectTransform, TextMeshPro - Text (UI) | UI | - | Font, Text: "TUNNEL RUN"
│  │  │  ├─ Start_Button | Startet das Spiel. | RectTransform, Image, Button | UI | - | OnClick() -> GameManager.StartGame()
│  │  │  └─ HighScore_Text | Zeigt lokalen Highscore an. | RectTransform, TextMeshPro - Text (UI) | UI | - | Text: "Highscore: 0"
│  │  ├─ InGame_Panel | Container für In-Game-UI. | RectTransform | UI | - | Aktiv/Inaktiv je nach State
│  │  │  ├─ Score_Text | Zeigt aktuellen Score an. | RectTransform, TextMeshPro - Text (UI) | UI | - | Text: "0", Oben zentriert
│  │  └─ GameOver_Panel | Container für Game-Over-UI. | RectTransform, Image (optional) | UI | - | Aktiv/Inaktiv je nach State
│  │    ├─ GameOver_Text | "Game Over" Titel. | RectTransform, TextMeshPro - Text (UI) | UI | - | Text: "GAME OVER"
│  │    ├─ FinalScore_Text | Zeigt End-Score an. | RectTransform, TextMeshPro - Text (UI) | UI | - | Text: "Score: 1234"
│  │    └─ Restart_Button | Startet das Spiel neu. | RectTransform, Image, Button | UI | - | OnClick() -> GameManager.RestartGame()
[GAMEPLAY] (Empty GO) | Organisatorischer Container | Transform | - | - | Position: (0,0,0)
├─ Player | Der Spieler. Rotiert nur, bewegt sich nicht. | Transform, Rigidbody, CapsuleCollider | Player | PlayerController, PlayerCollision | Position: (0, 0, 0). Rigidbody: Is Kinematic: true, Use Gravity: false. Collider: Radius/Height passend.
│  └─ Visuals | Das sichtbare Mesh des Spielers (z.B. Pfeil). | Transform, MeshFilter, MeshRenderer | Player | - | Mesh/Material nach Wahl
├─ MainCamera | Folgt der Spielerrotation (First-Person). | Camera, AudioListener | Untagged | - | Position: (0, 0, -0.5) (knapp hinter Spieler). Clear Flags: Solid Color (Schwarz). FOV: 70-90 (für Speed-Gefühl)
└─ WorldMover | Parent für alles, was sich bewegt. | Transform | Untagged | WorldMover | Position: (0,0,0)
    ├─ TunnelSpawner | Spawnt und poolt die Tunnelsegmente. | Transform | Untagged | TunnelSpawner | -
    └─ Lights | (Optional) Simples Licht. | Light | Untagged | - | Type: Directional.

2.1. Prefabs (In Assets/_Project/Prefabs/)

Prefab-Name | Zweck | Unity-Komponenten | Layer / Tag | Skripte | Wichtige Inspector-Werte
-----------|-------|------------------|-------------|---------|-------------------------
TunnelSegment_Prefab | Ein Stück Tunnel (z.B. 20m lang). | Transform | Untagged | SegmentCleanup | -
├─ Walls | Das sichtbare Tunnelmesh (z.B. Zylinder). | MeshFilter, MeshRenderer, MeshCollider (optional) | Default | - | Material: Tunnel_Mat
└─ ObstacleSet_X | (Mehrere) Parent für Hindernisse. | Transform | Untagged | ObstacleSet (optional) | Wird zufällig aktiviert.
    ├─ Obstacle_A | Ein einzelnes Hindernis (z.B. Balken). | Transform, MeshFilter, MeshRenderer, BoxCollider | Obstacle | - | Collider muss präzise sein.
    └─ Obstacle_B | Ein weiteres Hindernis. | (wie oben) | Obstacle | - | (wie oben)

3. Detaillierte Beschreibung der Skripte

3.0. Neue Kernkomponenten

**SceneLoader.cs**
Verantwortlichkeiten: Verwaltet das additive Laden/Entladen von Scenes.

```csharp
public class SceneLoader : MonoBehaviour {
    public static SceneLoader Instance { get; private set; }
    
    private void Awake() {
        if (Instance == null) {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        } else {
            Destroy(gameObject);
        }
    }
    
    public void LoadScene(string sceneName) {
        // Entlade alle aktiven Scenes außer Persistent
        // Lade die neue Scene additiv
        SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);
    }
}
```

**ServiceLocator.cs**
Zentraler Hub für zukünftige Services.

```csharp
public class ServiceLocator : MonoBehaviour {
    public static ServiceLocator Instance { get; private set; }
    
    // Referenzen zu allen Services
    public AdService Ads { get; private set; }
    public IAPService IAP { get; private set; }
    
    private void Awake() {
        if (Instance == null) {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            InitializeServices();
        } else {
            Destroy(gameObject);
        }
    }
    
    private void InitializeServices() {
        // Initialisiere alle Services
        Ads = GetComponent<AdService>();
        IAP = GetComponent<IAPService>();
        
        Debug.Log("All services initialized");
    }
}
```

**AdService.cs (Dummy)**
```csharp
// Placeholder für zukünftige Werbung-Integration
public class AdService : MonoBehaviour {
    // Leer lassen, nur Kommentare:
    // - ShowInterstitial()
    // - ShowRewarded()
    // - RemoveAds()
}
```

**IAPService.cs (Dummy)**
```csharp
// Placeholder für zukünftige In-App-Käufe
public class IAPService : MonoBehaviour {
    // Leer lassen, nur Kommentare:
    // - PurchaseRemoveAds()
    // - PurchaseSkin()
    // - RestorePurchases()
}
```

3.1. GameManager.cs (Aktualisiert)
Kategorie: Core
Verantwortlichkeiten: Steuert den globalen Spielzustand (State Machine), verwaltet Score und Schwierigkeit (ruft andere Skripte auf), koordiniert UI-Wechsel.

Wichtige Variablen:
```csharp
public static GameManager Instance { get; private set; } // Singleton
public enum GameState { MainMenu, Playing, GameOver, Shop }
public GameState CurrentState { get; private set; }

public UnityEvent OnGameStart;
public UnityEvent OnGameOver;

// Referenzen (im Inspector ziehen)
[Header("UI References")]
public UIManager uiManager;

[Header("Gameplay References")]
public ScoreManager scoreManager;
public DifficultyManager difficultyManager;
public WorldMover worldMover;
public PlayerController playerController;

// Scene Loader
private SceneLoader sceneLoader;
```

Methoden:
- `Awake()`: Implementiert Singleton-Pattern (Instance = this;).
- `Start()`: Initialisiert das Spiel: ChangeState(GameState.MainMenu).
- `ChangeState(GameState newState)`: Die zentrale State-Machine-Logik.
  ```csharp
  switch (newState) {
      case GameState.MainMenu:
          Time.timeScale = 1; // (für Menü-Animationen) oder 0
          uiManager.ShowMainMenu();
          worldMover.Stop();
          playerController.DisableInput();
          break;
      case GameState.Running:
          Time.timeScale = 1;
          uiManager.ShowInGameUI();
          scoreManager.StartScoring();
          difficultyManager.StartDifficultyScaling();
          worldMover.StartMoving();
          playerController.EnableInput();
          OnGameStart.Invoke();
          break;
      case GameState.GameOver:
          Time.timeScale = 0;
          uiManager.ShowGameOver(scoreManager.CurrentScore);
          worldMover.Stop();
          playerController.DisableInput();
          OnGameOver.Invoke();
          break;
  }
  ```
- `StartGame()`: Lädt die Game-Scene und wechselt in den Playing-State.
  ```csharp
  public void StartGame() {
      sceneLoader.LoadScene("Game");
      ChangeState(GameState.Playing);
  }
  ```
- `ReturnToMenu()`: Lädt das Hauptmenü.
  ```csharp
  public void ReturnToMenu() {
      sceneLoader.LoadScene("MainMenu");
      ChangeState(GameState.MainMenu);
  }
  ```
- `OpenShop()`: Öffnet den Shop (für spätere Verwendung).
  ```csharp
  public void OpenShop() {
      sceneLoader.LoadScene("Shop");
      ChangeState(GameState.Shop);
  }
  ```
- `EndGame()`: (Wird von PlayerCollision aufgerufen) ChangeState(GameState.GameOver).
- `RestartGame()`: (Wird von UI-Button aufgerufen) Time.timeScale = 1; SceneManager.LoadScene(SceneManager.GetActiveScene().name); (Einfachste Reset-Methode).

3.2. PlayerController.cs
Kategorie: Player
Verantwortlichkeiten: Nimmt Spieler-Input (Touch-Drag) entgegen und rotiert das Player-GameObject um die Z-Achse.

Wichtige Variablen:
```csharp
public float rotationSpeed = 200f; // Grad pro Sekunde
private bool inputEnabled = false;
private float targetRotationZ = 0f;
```

Methoden:
- `Update()`: 
  ```csharp
  if (!inputEnabled) return;
  HandleInput();
  ApplyRotation();
  ```
- `HandleInput()`: (Mobile-Fokus)
  ```csharp
  if (Input.touchCount > 0) {
      Touch touch = Input.GetTouch(0);
      if (touch.phase == TouchPhase.Moved) {
          float deltaX = touch.deltaPosition.x;
          targetRotationZ -= deltaX * (rotationSpeed / Screen.width) * Time.deltaTime; // Rotation basierend auf % der Bildschirmbreite
      }
  }
  // (Optional PC-Input für Tests):
  // float mouseX = Input.GetAxis("Mouse X");
  // targetRotationZ -= mouseX * rotationSpeed * Time.deltaTime;
  ```
- `ApplyRotation()`: Rotiert den Spieler.
  ```csharp
  transform.rotation = Quaternion.Euler(0, 0, targetRotationZ); // Direkte Zuweisung, kein Rotate(), um Akkumulationsfehler zu vermeiden
  ```
- `EnableInput()`: inputEnabled = true; (Aufgerufen von GameManager.OnGameStart).
- `DisableInput()`: inputEnabled = false; (Aufgerufen von GameManager.OnGameOver).

3.3. PlayerCollision.cs
Kategorie: Player
Verantwortlichkeiten: Erkennt Kollisionen mit Hindernissen und meldet dies dem GameManager.

Wichtige Variablen:
```csharp
public LayerMask obstacleLayer;
public string obstacleTag = "Obstacle";
private bool isDead = false;
```

Methoden:
- `OnCollisionEnter(Collision collision)`: (Da Player Rigidbody hat)
  ```csharp
  if (isDead) return;
  
  // Check per Tag (performanter als Layer-Check bei Collision)
  if (collision.gameObject.CompareTag(obstacleTag)) {
      HandleDeath();
  }
  ```
- `HandleDeath()`:
  ```csharp
  isDead = true;
  GameManager.Instance.EndGame();
  // (Optional: Partikeleffekt, Sound abspielen)
  ```

3.4. WorldMover.cs
Kategorie: Gameplay
Verantwortlichkeiten: Bewegt das WorldMover-GameObject (und damit alle Kinder: Tunnel, Hindernisse) auf den Spieler zu.

Wichtige Variablen:
```csharp
public float currentSpeed = 10f;
private bool isMoving = false;
```

Methoden:
- `Update()`: 
  ```csharp
  if (!isMoving) return;
  transform.Translate(Vector3.back * currentSpeed * Time.deltaTime);
  ```
- `StartMoving()`: isMoving = true;
- `Stop()`: isMoving = false;
- `SetSpeed(float newSpeed)`: currentSpeed = newSpeed;

3.5. DifficultyManager.cs
Kategorie: Core
Verantwortlichkeiten: Erhöht die Geschwindigkeit (WorldMover.currentSpeed) über die Zeit.

Wichtige Variablen:
```csharp
public WorldMover worldMover;
public float initialSpeed = 10f;
public float speedIncreasePerSecond = 0.2f;
private bool isScaling = false;
```

Methoden:
- `StartDifficultyScaling()`: 
  ```csharp
  worldMover.SetSpeed(initialSpeed);
  isScaling = true;
  ```
- `Update()`: 
  ```csharp
  if (!isScaling) return;
  float newSpeed = worldMover.currentSpeed + speedIncreasePerSecond * Time.deltaTime;
  worldMover.SetSpeed(newSpeed);
  ```
- `StopScaling()`: isScaling = false; (Wird von GameManager.OnGameOver aufgerufen).

3.6. ScoreManager.cs
Kategorie: Core
Verantwortlichkeiten: Zählt den Score basierend auf der Zeit und der Geschwindigkeit.

Wichtige Variablen:
```csharp
public UIManager uiManager;
public WorldMover worldMover;
public float scoreMultiplier = 10f;
public int CurrentScore { get; private set; }
private float scoreBuffer = 0f;
private bool isScoring = false;
```

Methoden:
- `StartScoring()`: 
  ```csharp
  CurrentScore = 0;
  scoreBuffer = 0;
  isScoring = true;
  ```
- `Update()`: 
  ```csharp
  if (!isScoring) return;
  
  // Score basiert auf Zeit * Geschwindigkeit
  scoreBuffer += Time.deltaTime * worldMover.currentSpeed * scoreMultiplier;
  CurrentScore = (int)scoreBuffer;
  uiManager.UpdateScore(CurrentScore);
  ```
- `StopScoring()`: isScoring = false;

3.7. TunnelSpawner.cs (Object Pooling)
Kategorie: Gameplay
Verantwortlichkeiten: Verwaltet einen Pool von TunnelSegment_Prefabs. Platziert Segmente nahtlos aneinander.

Wichtige Variablen:
```csharp
public GameObject tunnelSegmentPrefab;
public int poolSize = 10;
public float segmentLength = 20f; // Länge des Prefabs in Z
public int initialSegments = 5; // Anzahl Segmente zu Beginn

private Queue<GameObject> segmentPool = new Queue<GameObject>();
private List<GameObject> activeSegments = new List<GameObject>();
private Vector3 nextSpawnPoint = Vector3.zero;
```

Methoden:
- `Start()`: 
  ```csharp
  InitializePool();
  SpawnInitialSegments();
  ```
- `InitializePool()`: Füllt die segmentPool Queue mit poolSize instanziierten, aber deaktivierten Prefabs.
- `SpawnInitialSegments()`: Ruft SpawnSegment() initialSegments mal auf.
- `SpawnSegment()`: (Wird von Start und SegmentCleanup aufgerufen)
  ```csharp
  GameObject segment = segmentPool.Dequeue(); // Hole aus Pool
  segment.transform.SetParent(this.transform); // Kind von TunnelSpawner machen
  segment.transform.localPosition = nextSpawnPoint;
  segment.SetActive(true);
  activeSegments.Add(segment);
  nextSpawnPoint.z += segmentLength;
  ```
- `DespawnSegment(GameObject segment)`: (Wird von SegmentCleanup aufgerufen)
  ```csharp
  activeSegments.Remove(segment);
  segment.SetActive(false);
  segmentPool.Enqueue(segment);
  SpawnSegment(); // Sofort ein neues am Ende anfügen
  ```

3.8. SegmentCleanup.cs
Kategorie: Gameplay
Verantwortlichkeiten: An TunnelSegment_Prefab. Meldet dem Spawner, wenn es weit genug hinter dem Spieler ist.

Wichtige Variablen:
```csharp
public float cleanupDistanceZ = -30f; // Muss < 0 sein
private TunnelSpawner spawner;
```

Methoden:
- `Start()`: 
  ```csharp
  spawner = FindObjectOfType<TunnelSpawner>(); // Caching
  ```
- `Update()`:
  ```csharp
  // Position in Weltkoordinaten prüfen (da Parent sich bewegt)
  if (transform.position.z < cleanupDistanceZ) {
      spawner.DespawnSegment(this.gameObject);
  }
  ```

3.9. UI Management (Aktualisiert)

Anstatt eines zentralen UIManagers verwenden wir nun scene-spezifische UI-Skripte:

**MainMenuUI.cs** (in MainMenu.unity):
```csharp
public class MainMenuUI : MonoBehaviour {
    [Header("UI Elements")]
    public Button playButton;
    public Button shopButton; // Deaktiviert für später
    public TextMeshProUGUI highscoreText;

    private void Start() {
        playButton.onClick.AddListener(OnPlayClicked);
        shopButton.interactable = false; // Noch nicht implementiert
        
        // Highscore laden
        int highscore = PlayerPrefs.GetInt("HighScore", 0);
        highscoreText.text = $"Highscore: {highscore}";
    }
    
    private void OnPlayClicked() {
        GameManager.Instance.StartGame();
    }
}
```

**GameUI.cs** (in Game.unity):
```csharp
public class GameUI : MonoBehaviour {
    public TextMeshProUGUI scoreText;
    
    public void UpdateScore(int score) {
        scoreText.text = $"{score}";
    }
}
```

**GameOverPanel.cs** (in Game.unity):
```csharp
public class GameOverPanel : MonoBehaviour {
    [Header("UI Elements")]
    public TextMeshProUGUI finalScoreText;
    public Button restartButton;
    public Button menuButton;
    
    private void Start() {
        restartButton.onClick.AddListener(OnRestartClicked);
        menuButton.onClick.AddListener(OnMenuClicked);
    }
    
    public void Show(int score) {
        gameObject.SetActive(true);
        finalScoreText.text = $"Score: {score}";
    }
    
    private void OnRestartClicked() {
        GameManager.Instance.StartGame();
    }
    
    private void OnMenuClicked() {
        GameManager.Instance.ReturnToMenu();
    }
}
```
Kategorie: UI
Verantwortlichkeiten: Aktiviert/Deaktiviert UI-Panels. Aktualisiert Text-Elemente.

Wichtige Variablen:
```csharp
public GameObject mainMenuPanel, inGamePanel, gameOverPanel;
public TextMeshProUGUI scoreText, finalScoreText, highscoreText;
private int highscore = 0;
```

Methoden:
- `Start()`: 
  ```csharp
  highscore = PlayerPrefs.GetInt("HighScore", 0);
  highscoreText.SetText("Highscore: {0}", highscore);
  ```
- `ShowMainMenu()`: 
  ```csharp
  mainMenuPanel.SetActive(true);
  inGamePanel.SetActive(false);
  gameOverPanel.SetActive(false);
  ```
- `ShowInGameUI()`: 
  ```csharp
  mainMenuPanel.SetActive(false);
  inGamePanel.SetActive(true);
  gameOverPanel.SetActive(false);
  ```
- `ShowGameOver(int finalScore)`:
  ```csharp
  mainMenuPanel.SetActive(false);
  inGamePanel.SetActive(false);
  gameOverPanel.SetActive(true);
  
  finalScoreText.SetText("Score: {0}", finalScore);
  
  if (finalScore > highscore) {
      highscore = finalScore;
      PlayerPrefs.SetInt("HighScore", highscore);
  }
  
  highscoreText.SetText("Highscore: {0}", highscore);
  ```
- `UpdateScore(int score)`: 
  ```csharp
  scoreText.SetText("{0}", score);
  ```

4. State-Machine / Spielfluss

Der GameManager steuert den Fluss über das GameState Enum:

**Start (MAIN MENU)**
- GameManager.Start() -> ChangeState(GameState.MainMenu)
- UIManager zeigt MainMenu_Panel.
- WorldMover steht. Time.timeScale ist 1 (oder 0).
- Spiel wartet auf Input (Start_Button).

**Transition: Start Game**
- Start_Button.OnClick() -> GameManager.StartGame()
- GameManager.ChangeState(GameState.Running)

**Spiel läuft (RUNNING)**
- UIManager zeigt InGame_Panel.
- PlayerController ist aktiv.
- WorldMover bewegt sich.
- ScoreManager zählt.
- DifficultyManager erhöht WorldMover.speed.
- TunnelSpawner recycelt Segmente.
- PlayerCollision prüft auf Kollisionen.

**Transition: Game Over**
- PlayerCollision.OnCollisionEnter() (mit "Obstacle")
- PlayerCollision.HandleDeath() -> GameManager.EndGame()
- GameManager.ChangeState(GameState.GameOver)

**Ende (GAME OVER)**
- Time.timeScale = 0 (friert Spiel ein).
- WorldMover stoppt.
- UIManager zeigt GameOver_Panel (inkl. finalScore).
- ScoreManager speichert ggf. Highscore.
- Spiel wartet auf Input (Restart_Button).

**Transition: Restart**
- Restart_Button.OnClick() -> GameManager.RestartGame()
- SceneManager.LoadScene(...) (lädt die GameScene neu).
- Zyklus beginnt bei Schritt 1.

5. Game Loop & Difficulty Design

**Tunnelerzeugung:**
- Das Spiel ist ein "Conveyor Belt".
- Der TunnelSpawner nutzt Object Pooling (siehe 3.7).
- Er recycelt TunnelSegment_Prefabs.

**Hindernis-Generierung:**
- Die TunnelSegment_Prefabs enthalten verschiedene "Sets" von Hindernissen (Kinder-GameObjects, z.B. ObstacleSet_A, ObstacleSet_B).
- Wenn ein Segment aus dem Pool geholt wird (OnEnable()-Methode im Segment-Prefab), wählt es zufällig eines seiner Hindernis-Sets aus und aktiviert es.
  ```csharp
  void OnEnable() {
      DeactivateAllObstacleSets();
      int index = Random.Range(0, obstacleSets.Length);
      obstacleSets[index].SetActive(true);
  }
  ```

**Schwierigkeitskurve:**
- Gesteuert durch DifficultyManager.cs.
- Nur ein Parameter: Geschwindigkeit.
  ```
  currentSpeed = initialSpeed + (timeInGame * speedIncreasePerSecond)
  ```
- Empfohlene Startwerte: initialSpeed = 10, speedIncreasePerSecond = 0.2.
- Indirekte Schwierigkeit: Komplexere Hindernis-Sets können später im Pool freigeschaltet werden (z.B. basierend auf Score), dies ist aber für v1 optional.

6. Mathematisch/Technische Details

**Player-Rotation:**
- Wir rotieren nicht den Player selbst, sondern nur einen Ziel-Rotationswert (targetRotationZ).
  ```csharp
  transform.rotation = Quaternion.Euler(0, 0, targetRotationZ); // in PlayerController.Update()
  ```
- Dies verhindert Probleme mit transform.Rotate() (Gimbal Lock oder unkontrollierte Drehung).
- Die Rotation ist immer absolut.

**Hindernis-Bewegung:**
- Die Hindernisse bewegen sich nicht. Sie sind statische Kinder von TunnelSegment.
- Das TunnelSegment ist Kind von WorldMover.
- Der WorldMover bewegt sich auf Z-Negativ (Vector3.back).

**Kollision:**
- Player: Rigidbody (Kinematic) + CapsuleCollider.
- Obstacle: BoxCollider (Static).
- Die Kollision wird von Unitys Physik-Engine erkannt und löst OnCollisionEnter im PlayerCollision.cs Skript aus.

**Score-System:**
- Score = (Zeit * Geschwindigkeit * Multiplikator).
- Dies belohnt das Überleben bei höheren Geschwindigkeiten.

7. Mobile Optimierungen (WICHTIG):

1. **Object Pooling:** (Implementiert in TunnelSpawner).
   - Kein Instantiate() oder Destroy() während des Spiels.

2. **Garbage Collection (GC):**
   - Vermeiden Sie String-Operationen in Update(). (Nutzen Sie TextMeshPro.SetText("{0}", scoreValue) statt scoreText.text = "Score: " + score).
   - Cachen Sie alle Referenzen in Awake() oder Start() (z.B. GetComponent<Rigidbody>()).
   - Kein FindObjectOfType in Update().

3. **Draw Calls (Batching):**
   - Verwenden Sie ein Material für alle Tunnelwände und ein Material für alle Hindernisse (Texture Atlas).
   - Aktivieren Sie "Static Batching" für die Hindernis-Meshes (da sie sich relativ zu ihrem Parent-Segment nicht bewegen).

8. UI & UX

**Elemente:** (Siehe Sektion 2). Minimalistisch.

**Fonts:**
- TextMeshPro verwenden.
- Importieren Sie einen klaren Font (z.B. "Roboto" oder "Montserrat").

**Layout:**
- CanvasScaler auf "Scale With Screen Size" setzen.
- Alle UI-Elemente mit Ankern versehen (z.B. Score oben mittig, Buttons unten mittig).

**UX-Fluss:**
1. User öffnet App -> MainMenu_Panel.
2. Klick Start_Button -> InGame_Panel.
3. Spieler rotiert durch Touch-Drag (links/rechts).
4. Kollision -> GameOver_Panel.
5. Klick Restart_Button -> Zurück zu MainMenu_Panel (nach Neuladen der Szene).

9. Erweiterungen (Optional, nach v1)

- **Highscore:** (Bereits im Plan, via PlayerPrefs).
- **Skins:** Manager, der das Mesh und Material im Player/Visuals-GameObject austauscht.
- **Sound:** AudioManager (Singleton), der OnGameStart (Musik), OnGameOver (Fail-Sound) und Kollisionen (SFX) abspielt.
- **Partikel:** Ein ParticleSystem (gepoolt), das bei PlayerCollision.HandleDeath() an der Kollisionsstelle abgespielt wird.

10. Finaler Schritt-für-Schritt-Umsetzungsplan

**Setup (Tag 1):**
- Unity-Projekt erstellen (Target: Android).
- Alle Einstellungen (Player, Quality, Physics, Time) gemäß Sektion 1 vornehmen.
- Ordnerstruktur (Sektion 1.2) anlegen.
- GameScene erstellen und speichern.
- TextMeshPro importieren.

**Basis-GameObjects (Tag 1):**
- Alle GameObjects aus Sektion 2 in der GameScene-Hierarchie anlegen ([SYSTEMS], [GAMEPLAY], Player, Camera, WorldMover, UI_Canvas etc.).
- Player (mit Rigidbody + Collider) und Camera korrekt positionieren.
- Simples Directional Light hinzufügen.

**Player-Movement (Tag 2):**
- PlayerController.cs Skript erstellen.
- Input-Logik (HandleInput) implementieren, um targetRotationZ zu ändern.
- Rotations-Logik (ApplyRotation) implementieren.
- Im Editor testen (Maus-Input hinzufügen).

**Welten-Erzeugung (Tag 2-3):**
- Einfaches Tunnel-Segment-Modell erstellen (z.B. Zylinder ohne Endkappen, 20m lang) -> Als TunnelSegment_Prefab speichern.
- WorldMover.cs erstellen (Bewegt WorldMover-GameObject).
- TunnelSpawner.cs (mit Pooling) und SegmentCleanup.cs erstellen.
- Logik implementieren, sodass 5-10 Segmente spawnen und sich endlos bewegen/recyceln.

**Hindernisse & Kollision (Tag 4):**
- Einfache Hindernis-Meshes (z.B. Würfel/Balken) erstellen.
- Dem TunnelSegment_Prefab Hindernis-Sets als Kinder hinzufügen (z.B. ObstacleSet_A).
- Obstacle-Layer und Obstacle-Tag erstellen und den Hindernissen zuweisen.
- PlayerCollision.cs erstellen (OnCollisionEnter).

**Game-Loop & State (Tag 5):**
- GameManager.cs erstellen (mit GameState Enum und ChangeState-Logik).
- PlayerCollision muss GameManager.EndGame() aufrufen.
- DifficultyManager.cs erstellen, um WorldMover.speed zu erhöhen.
- ScoreManager.cs erstellen, um CurrentScore zu berechnen.

**UI (Tag 6):**
- Alle UI-Panels und Text-Elemente (Sektion 2) im UI_Canvas aufbauen.
- UIManager.cs erstellen.
- GameManager mit UIManager verknüpfen, um Panels je nach GameState zu wechseln.
- Buttons (Start, Restart) mit GameManager (StartGame, RestartGame) verknüpfen.
- Highscore-Logik (PlayerPrefs) im UIManager und ScoreManager implementieren.

**Testen & Polish (Tag 7):**
- Werte anpassen (Speed, Rotation, Score-Multiplikator).
- Auf Android-Gerät builden und testen.
- (Optional) Sound-Effekte und Partikel hinzufügen.
- Sicherstellen, dass keine GC Spikes auftreten (Profiler verwenden).

## 🏗️ Architektur-Prinzipien
* **Multi-Scene-Design:** Saubere Trennung von Core-Logik und Content.
* **Additive Scene Loading:** Persistent.unity läuft immer, andere Scenes werden dynamisch geladen.
* **Service Locator Pattern:** Zentraler Zugriffspunkt für zukünftige Services (Ads, IAP, Analytics).
* **Scene-spezifische UI:** Jede Scene hat eigene UI-Skripte (keine monolithischen Manager).
* **Erweiterbarkeit:** Vorbereitet für Monetarisierung, ohne bestehenden Code zu ändern.

## 🚀 Entwicklungs-Roadmap

### Phase 1: Core-Architektur ✅ (Aktuell)
- [x] Multi-Scene-Struktur (Persistent, MainMenu, Game)
- [x] SceneLoader für Additive Loading
- [x] ServiceLocator als Service-Hub
- [x] Dummy-Services (AdService, IAPService) als Platzhalter

### Phase 2: Gameplay-Polish
- [ ] Audio-System (Musik, SFX)
- [ ] Partikeleffekte (Kollisionen, Speed-Lines)
- [ ] Visuelles Feedback (Screen Shake, etc.)

### Phase 3: Monetarisierung (später)
- [ ] Unity Ads / AdMob Integration
- [ ] In-App-Käufe (Remove Ads, Skins)
- [ ] Shop-Scene mit UI

### Phase 4: Content-Erweiterung
- [ ] Skin-System für Spieler
- [ ] Verschiedene Tunnel-Themes
- [ ] Power-Ups (Unverwundbarkeit, Slow-Mo)

### Phase 5: Analytics & Retention
- [ ] Analytics-Integration
- [ ] Daily Rewards
- [ ] Achievements/Leaderboards
