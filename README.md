# Session-9-Guide-Tic-Tac-Toe

---
# Part 1

## Unity 6.2 • URP • Meta XR All-in-One (v78+) • OpenXR • Quest 2/3

### Student Guide (session-ready, step-by-step)

**Goal of Part 1:** create a clean, *production-ready* Unity project configured for Quest (Android) using **OpenXR** + **Meta XR All-in-One SDK**, apply performance-safe URP defaults, and scaffold the scene/folders we’ll build on in Parts 2–6.
(We’ll add the **PlayerRig**, rays, wall UI, and logic in later parts.)

---
---

## 🧭 Flow map (what window you’ll use each step)

* **Unity Hub** → create project
* **Build Settings / Project Settings** → Android, Player, XR, Graphics
* **Package Manager** → install OpenXR + Meta XR All-in-One
* **Meta XR Project Setup Tool** → Fix All
* **URP Asset & Renderer** → performance defaults
* **Project/Hierarchy** → folders, base scene scaffold

---

## ✅ To-Do Checklist (with “why” + exact clicks)

### 1) Create the project (URP)

1. **Unity Hub** → **New → 3D (URP)** → name **`VR_TicTacToe_Lab`** → Create.
   *URP is the most stable/performant baseline on Quest.*   

### 2) Switch platform & compression (Android + ASTC)

1. **File → Build Settings…** → **Android** → **Switch Platform**.
   *Quest runs Android; do this early to avoid massive reimports later.*   
2. Same dialog → **Texture Compression = ASTC**.
   *Best quality/memory tradeoff for mobile VR.*

### 3) Player Settings (required for Quest stores)

Open **Edit → Project Settings… → Player → Other Settings**:

* **Company/Product**: set your names (needed for package id).
* **Package Name**: `com.company.vr_ttt_lab`
* **Scripting Backend**: **IL2CPP**
* **Target Architectures**: **ARM64** (only)
* **Minimum API Level**: **Android 10 (API 29)** or higher
* **Graphics APIs** (Android): **Vulkan** (remove GLES3 unless you need it)
  *These match current Quest/Unity requirements and typical store checks.*   

### 4) Install XR + Meta packages (OpenXR path)

Open **Window → Package Manager**:

* Install **XR Plug-in Management** (if not installed).
* Install **OpenXR Plug-in** (Unity).
* Install **Meta XR All-in-One SDK (AIO)** (UPM).
  *AIO wraps the latest Meta XR SDKs in one package.*  

> **Heads-up for educators:** If students see “Oculus XR Plug-in”, clarify we’re standardizing on **OpenXR** for Unity 6.2; Meta provides **Unity OpenXR: Meta** support and extensions via OpenXR.   

### 5) Enable OpenXR + Meta Quest support

Open **Edit → Project Settings… → XR Plug-in Management → Android**:

* **Check “OpenXR”**.
* Click **OpenXR** (left pane) → in **Features** enable **Meta / Meta Quest Support** (wording can vary by version; look for Meta-branded OpenXR support).   

> **Quick verify:** In **Package Manager**, you should also see **“Unity OpenXR: Meta”** (com.unity.xr.meta-openxr) listed/installed or pulled as a dependency.   

### 6) Run Meta XR Project Setup / Validation (auto-fix)

1. **Meta XR → Project Setup** (or **Project Settings → Meta XR → Launch Project Setup Tool**).
2. Press **Fix All** until green.
   *This tool applies required flags, permissions, input backends, etc., for Meta Quest targets.*   

### 7) URP performance baseline (mobile VR-safe)

Find your **URP Asset** (Project search: `*UniversalRenderPipelineAsset*`) and **URP Renderer**:

* **URP Asset:**

  * **HDR = OFF**
  * **MSAA = 4×**
  * Shadows modest (e.g., 512 res; distance 20–25m; 2 cascades)
* **URP Renderer:**

  * **Post-processing = OFF** (we’ll add selectively later)
  * **Transparent Receive Shadows = OFF**
  * **Renderer Features** = empty for now
    *These settings avoid common mobile bottlenecks; adjust as you optimize.*   

> **Optional:** Enable **Foveated Rendering** (XR Plug-in Management → provider settings) once you’re stable—good free perf; slight peripheral blur. ([Unity Documentation][6])

### 8) Input handling (sanity)

* **Edit → Project Settings → Player → Active Input Handling = Input System** (or **Both** if you rely on legacy input anywhere).
  *Meta samples and Interaction SDK are Input-System friendly.*

### 9) Project structure & base scene scaffold

**Folders (Project window):**
`Assets/Scenes`, `Assets/Scripts`, `Assets/Prefabs`, `Assets/Materials`, `Assets/Resources`, `Assets/Art`

**Scene:**

1. **File → New Scene** → save as **`Assets/Scenes/01_VR_TicTacToe.unity`**
2. **Hierarchy:** delete **Main Camera** (we’ll spawn the tracked camera from the rig later).
3. **GameObject → 3D Object → Plane** → rename **`Ground`** → **Scale (10,1,10)**, **Position (0,0,0)**
4. Keep **Directional Light**; rotate so shadows are readable in VR.

> **What happens (why this now):** This gives us a visually stable space to stand in during headset tests once the rig is added in Part 2. The camera will come from the rig (OpenXR), not Main Camera.

---

## 🔎 Quick verification (2 minutes)

* **Build Settings** shows **Android**, **ASTC**.
* **XR Plug-in Management → Android** shows **OpenXR** checked; **Meta/Quest features** enabled.   
* **Meta XR Project Setup Tool** reports **No issues** after **Fix All**.   
* **URP Asset** set with **HDR Off**, **MSAA 4×**, **no postFX**.   
* Scene saved as **01_VR_TicTacToe.unity** with **Ground** and **Directional Light** only (camera removed).

---

## 🧩 Notes & Hints (for Students)

* **If you see OpenXR warnings**: reopen **Project Setup Tool → Fix All** and confirm the **Meta OpenXR** feature is enabled.   
* **If the scene is pink** after imports: **Edit → Render Pipeline → URP → Upgrade Project Materials to URP** (only once).
* **Don’t add XR rigs yet**—that’s Part 2; today is “golden project” setup.

---

## 📦 End-of-Part snapshot

(We’ll add the rig/wall/grid next part. For now—clean base.)

**What your Hierarchy should look like now**

```
Directional Light
Ground
```

**What your Project structure should look like now**

```
Assets/
  Art/
  Materials/
  Prefabs/
  Resources/
  Scenes/
    01_VR_TicTacToe.unity
  Scripts/
```

**XR / Graphics configuration summary**

* Android, ASTC • IL2CPP • ARM64 • Vulkan
* OpenXR **enabled** + **Meta/Quest** feature **ON**   
* Meta XR **Project Setup: Fix All** ✅   
* URP: HDR Off, MSAA 4×, no postFX (baseline)   

---
---

# Part 2

## Scene, Rig, Rays, Grab, Teleport, and World-Space UI (OVR path)

**Goal of Part 2:**
Drop in the OVR rig exactly like you teach it, add **controller rays**, **grab**, **stick locomotion**, **teleport**, and a **wall canvas** that rays can click—no custom scripts.

---

## What you’ll touch (flow)

* **Project**: search prefabs
* **Hierarchy**: place/parent objects
* **Inspector**: add components & set values
* **Scene**: position/scale
* **Play**: quick checks with Link/Air Link

> Prereqs from Part 1: platform is Android/ASTC, OVR provider enabled, URP baseline applied, new empty scene with **Ground** + **Directional Light**.

---

## 1) Folders & scene quick prep

* Create folders: `Assets/Scenes`, `Assets/Prefabs`, `Assets/Materials`, `Assets/Resources`.
* Save your scene as: `Assets/Scenes/01_Raycast_Objects_UI.unity`.
* In **Hierarchy**, keep:

  * **Ground** (Plane) at `(0,0,0)`, scaled `(10,1,10)`.
  * **Directional Light**.

---

## 2) Add the rig exactly as you teach it

### A) OVRCameraRig

1. **Project (Search):** `OVRCameraRig` → drag to **Hierarchy (root)**.

### B) OVRInteraction (child of the rig)

2. **Project (Search):** `OVRInteraction` → drag as **child** of `OVRCameraRig`.

### C) OVRController (child of OVRInteraction)

3. **Project (Search):** `OVRController` → drag **inside** `OVRCameraRig/OVRInteraction`.

   * This adds `LeftController` and `RightController` with **ControllerInteractors** children.

> Don’t touch Main Camera—delete it if present. The rig provides the tracked camera.

---

## 3) Rays on both hands (exact objects/places)

### Left hand

* **Hierarchy path:**
  `OVRCameraRig/OVRInteraction/OVRController/LeftController/ControllerInteractors`
* 3.1 Project (Search): type Controller Ray Interactor (or Ray Interactor if that’s the prefab name in your SDK)

⬜ 3.2 Hierarchy Path: OVRCameraRig/OVRInteraction/OVRController/LeftController/ControllerInteractors

Drag the Controller Ray Interactor prefab into this ControllerInteractors object.

Inspector (on the newly added Controller Ray Interactor):

Max Ray Length = 6
Hide When No Interactable = ON (if the property is under a “Visuals/ControllerRay” child, select that child and toggle there)

  * **Controller Ray Interactor**

    * **Max Ray Length:** `6`
    * **Hide When No Interactable:** `On`
  * **Grab Interactor**

### Right hand

* Same path under **RightController** → add the **same two** components with the **same values**.

*(If your SDK version uses a child “Visuals/ControllerRay” for line settings, open it and enable the visual there—no other changes.)*

---

## 4) Locomotion (thumbstick) with zero code

We’ll use the standard movement component that ships with the SDK.

1. Select **OVRCameraRig** (root of the rig), or create an empty **PlayerRig** parent and move OVRCameraRig under it (either is fine—pick your class style).
2. **Inspector:**

   * **Add Component → Character Controller**

     * **Height:** `1.7`
     * **Radius:** `0.3`
     * **Center:** `(0, 0.9, 0)`
   * **Add Component → SimpleCapsuleWithStickMovement** (or its current equivalent in your SDK)

     * No changes needed; default thumbstick locomotion works out of the box.

> Result: left/right stick (depending on bindings) moves the capsule in flat scenes. Keep your rig above the **Ground** so the capsule is not intersecting.

---

## 5) Teleport (aim arc + areas/anchors)

### A) Add a Teleport Interactor on **each** hand

* **Left ControllerInteractors** → **Add Component → Teleport Interactor**

  * If your SDK exposes a **Teleport Arc Visual**, add it too and it will auto-link or give you a slot to assign.
* **Right ControllerInteractors** → repeat the same.

> You can choose which hand does teleport vs. ray/selection later with filters—today we enable both for testing.

### B) Make the scene teleportable

* **Ground** (or any floor mesh you want to allow teleport onto):

  * **Add Component → Teleport Area**

    * Optional: set **Valid/Invalid** visual materials if the component exposes them.
* Optional anchors (specific points):

  * Create empties where you want, name them `TeleportAnchor_*`, add **Teleport Anchor** and place them on platforms, pads, etc.

### C) Typical input (for playtest)

* Aim arc shows while holding your teleport activation (thumbstick press or button per your default mapping).
* Release to teleport.
* If the arc doesn’t show, check the Teleport Interactor’s activation settings and that the **Teleport Area** exists on the target surface.

---

## 6) First 3D target (ray-grab demo cube)

1. **Hierarchy:** **3D Object → Cube** → rename **RayObject**

   * **Transform:** **Scale** `(0.2,0.2,0.2)`, **Position** `(0,0.5,2)`
2. **Inspector (RayObject):**

   * **Add Component → Rigidbody** (Use Gravity = On, Interpolate = Interpolate)
   * **Add Component → ColliderSurface** (if separate) → set **Collider** to its BoxCollider
   * **Add Component → Grabbable** → drag its **Rigidbody** into the slot
   * **Add Component → RayInteractable**

     * Set **Pointable Element = Grabbable**
     * Set **Surface = ColliderSurface**
   * **Add Component → MoveTowardsTargetProvider** (or **MoveFromTargetProvider**)

     * In **RayInteractable → Movement Provider**: assign the provider you added

**Wizard shortcut:**
Right-click **RayObject** → **Interaction SDK → Add Ray Grab Interaction**

* This creates a parent like **`ISDK Ray Grab Interaction`** and moves the cube as its child.
* Configure the **parent** (that’s where most interaction components live).

---

## 7) Wall + world-space Canvas (UI the rays can click)

### A) Wall

* **3D Object → Cube** → rename **Wall**

  * **Position:** `(0,1.5,2.5)`
  * **Scale:** `(3,2,0.1)`

### B) Canvas on the wall

* Right-click **Wall** → **UI → Canvas** → rename **BoardCanvas**

  * **Canvas Render Mode:** `World Space`
  * **Rect:** size `800 × 800`
  * **Local Position:** `(0,0,0.06)` (a few cm off the wall)
  * **Scale:** `(0.001,0.001,0.001)`

### C) Make the UI ray-clickable

* If the scene **doesn’t** have one, create **EventSystem** (any UI object will prompt it).
* **EventSystem** → **Add Component → Pointable Canvas Module** (keep only one input module active).
* **BoardCanvas** → **Add Component → Pointable Canvas** (ensure a **Graphic Raycaster** is also present—UGUI adds it by default).

### D) Grid and placeholders (for Part 3)

* Under **BoardCanvas** → **UI → Panel** → rename **BoardPanel** (full stretch; subtle background color).
* Under **BoardCanvas** → **UI → Panel** → rename **Grid**

  * **Rect:** `700 × 700`
  * **Add Component → Grid Layout Group**

    * **Cell Size:** `220 × 220`
    * **Spacing:** `20 × 20`
    * **Constraint:** `Fixed Column Count = 3`
* Under **Grid** → **UI → Button (TextMeshPro)** → rename **Cell_0**

  * Clear the label text (we’ll use sprites later).
  * Drag **Cell_0** to `Assets/Prefabs` to create **Cell** prefab; delete **Cell_0** from scene; then drag **9** instances into **Grid** and rename **Cell_0 … Cell_8**.
* Under **BoardCanvas** → **UI → Panel** → rename **GameOverPanel** (disabled for now)

  * Add **Text (TMP)** → **GameOverText**
  * Add **Button (TMP)** → **RestartButton**

---

## 8) Quick playtest (in editor with Link/Air Link)

* **Rays**: point at **Cell** buttons; hover highlights; click presses.
* **Grab**: point at **RayObject**, pull trigger to grab, move it, release to drop.
* **Locomotion**: move using the stick; the capsule glides on the Ground.
* **Teleport**: hold your teleport activate (thumbstick press or assigned button), see the **arc**, release over **Ground** to teleport.

---

## Troubleshooting (fast checks)

* **No ray** → On each controller’s **ControllerInteractors**, confirm **Controller Ray Interactor** is present and not disabled.
* **UI doesn’t click** → Ensure **EventSystem** has **Pointable Canvas Module**, and **BoardCanvas** has **Pointable Canvas** (and a **Graphic Raycaster**). Canvas scale ≈ `0.001`.
* **Can’t grab cube** → On the **host** (wizard parent or cube itself), confirm **RayInteractable + Grabbable + ColliderSurface + MovementProvider** are present and wired.
* **Teleport arc won’t show** → Make sure **Teleport Interactor** is on the hand, and **Ground** has **Teleport Area**. Check the interactor’s activation setting.
* **Stick movement not working** → Confirm **Character Controller** + **SimpleCapsuleWithStickMovement** are on the rig root (or your PlayerRig parent), and the capsule isn’t intersecting the Ground.

---

## End-of-Part snapshot (what you must have now)

### What your **Hierarchy** should look like now

```
OVRCameraRig (or PlayerRig with OVRCameraRig inside)
└─ OVRInteraction
└─ OVRController
   ├─ LeftController
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor
   │     ├─ Grab Interactor
   │     └─ Teleport Interactor (with Arc Visual if separate)
   └─ RightController
      └─ ControllerInteractors
         ├─ Controller Ray Interactor
         ├─ Grab Interactor
         └─ Teleport Interactor (with Arc Visual if separate)

Wall
└─ BoardCanvas (World Space; EventSystem has Pointable Canvas Module)
   ├─ BoardPanel (stretched background)
   ├─ Grid (Grid Layout Group: 3 cols, cell 0.22, spacing 0.02)
   │  ├─ Cell_0 (Prefab instance)
   │  └─ … Cell_8
   └─ GameOverPanel (disabled)
      ├─ GameOverText (TMP)
      └─ RestartButton (TMP Button)

RayObject (cube)  ← if you did the 3D grab demo
EventSystem (with Pointable Canvas Module)
Ground (with Teleport Area)
Directional Light
```

### Key **Inspector** values (recap)

* **Controller Ray Interactor:** Max `6`, Hide When No Interactable `On`.
* **Grab Interactor:** defaults are fine.
* **Teleport Interactor:** defaults are fine; add Arc Visual if separate.
* **Ground:** has **Teleport Area**.
* **OVRCameraRig (root):** **Character Controller** (Height `1.7`, Radius `0.3`, Center `0,0.9,0`) + **SimpleCapsuleWithStickMovement**.
* **BoardCanvas:** World Space, size `800×800`, scale `0.001`, offset `+0.06` on Z; Canvas has **Pointable Canvas**; scene **EventSystem** has **Pointable Canvas Module**.
* **Grid:** cell `220×220`, spacing `20×20`, columns `3`.

---
# Part 3

## Tic-Tac-Toe Board Logic & UI Wiring (OVR path • Unity 6.2 • URP)

**Goal of Part 3:** make the wall board fully playable with rays: click cells → place **X/O**, detect **win/draw**, show **GameOver** UI, **Restart**.

> Prereqs from Part 2: you already have **OVRCameraRig + rays + grab + locomotion + teleport**, and a **Wall/BoardCanvas** with **Grid** containing 9 **Cell** (prefab instances), plus a hidden **GameOverPanel** (with **GameOverText** and **RestartButton**).

---

## What you’ll touch (flow)

* **Project**: create scripts, ScriptableObjects, and sprites
* **Hierarchy**: add managers
* **Inspector**: assign references (cells list, UI fields, button events)
* **Play**: verify turns, win/draw, restart

---

## 1) Assets prep (sprites + folders)

1. **Project** → create folders:
   `Assets/Scripts/Gameplay`, `Assets/Scripts/UI`, `Assets/Resources`, `Assets/Prefabs/Cells`

2. **Sprites (Resources):** add three square images to `Assets/Resources`

   * `x.png` (X icon)
   * `o.png` (O icon)
   * `b.png` (blank)
     **Import settings (each):** Texture Type = **Sprite (2D and UI)**, Mesh Type = **Full Rect**, Mip Maps **Off**.

---

## 2) Create the Cell ScriptableObject (data for each grid cell)

**Project** → `Assets/Scripts/Gameplay` → **Create > C# Script** → **Cell.cs**, paste:

```csharp
using System;
using UnityEngine;

[CreateAssetMenu(fileName = "Cell", menuName = "TicTacToe/Cell")]
public class Cell : ScriptableObject
{
    public int Id;                // 0..8
    public int Value;             // 0=blank, 1=X, 2=O
    public bool IsInteractive { get; private set; } = true;

    public event Action<int, int> OnValueChanged; // (cellId, value)
    public event Action<bool> OnGameFinished;     // mark visuals win/lose

    public void SetValue(int newValue)
    {
        Value = newValue;
        OnValueChanged?.Invoke(Id, Value);
    }

    public void SetResult(bool isWin)
    {
        OnGameFinished?.Invoke(isWin);
        IsInteractive = false;
    }

    public void Reset()
    {
        IsInteractive = true;
        SetValue(0);
    }
}
```

**Create 9 instances:**
Right-click in **Project** → **Create → TicTacToe → Cell** → name them `Cell_0` … `Cell_8`.
Set **Id** on each to match the number.

---

## 3) UICellBehaviour (click handling + icon swap)

**Project** → `Assets/Scripts/UI` → **Create > C# Script** → **UICellBehaviour.cs**, paste:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class UICellBehaviour : MonoBehaviour
{
    [Header("Data")]
    [SerializeField] private Cell cell;

    [Header("UI")]
    [SerializeField] private Button button;
    [SerializeField] private Image image;
    [SerializeField] private Color defaultColor = Color.white;
    [SerializeField] private Color winColor = new Color(0.3f, 1f, 0.3f);
    [SerializeField] private Color failedColor = new Color(1f, 0.3f, 0.3f);

    private Sprite _x; private Sprite _o; private Sprite _blank;

    void Awake()
    {
        _x = Resources.Load<Sprite>("x");
        _o = Resources.Load<Sprite>("o");
        _blank = Resources.Load<Sprite>("b");
    }

    void Start()
    {
        button.onClick.AddListener(OnButtonClick);
        ApplyValue(cell.Value); // in case of serialized state
    }

    void OnEnable()
    {
        cell.OnValueChanged += OnValueChanged;
        cell.OnGameFinished += OnGameFinished;
    }

    void OnDisable()
    {
        cell.OnValueChanged -= OnValueChanged;
        cell.OnGameFinished -= OnGameFinished;
    }

    void OnButtonClick()
    {
        if (!cell.IsInteractive) return;
        if (cell.Value != 0) return;

        bool isXTurn = TurnManager.Instance.GetTurn();
        cell.SetValue(isXTurn ? 1 : 2);   // triggers OnValueChanged
    }

    void OnValueChanged(int cellId, int newValue) => ApplyValue(newValue);

    void ApplyValue(int v)
    {
        if (v == 0)
        {
            image.sprite = _blank;
            image.color = defaultColor;
            button.interactable = true;
        }
        else
        {
            image.sprite = (v == 1) ? _x : _o;
            button.interactable = false;  // locked after play
        }
    }

    void OnGameFinished(bool isWin)
    {
        image.color = isWin ? winColor : failedColor;
        button.interactable = false;
    }

    // Helper for prefab setup in Inspector
    public void Inject(Cell c, Button b, Image i)
    {
        cell = c; button = b; image = i;
    }
}
```

**Attach to the Cell prefab** (not the instances yet):

1. **Project** → open **Cell** prefab.
2. Add **UICellBehaviour**; drag the prefab’s **Button** into **button**, the child **Image** into **image**.
3. Set colors if you like.
4. **Apply** prefab changes (Overrides → Apply All).

---

## 4) TurnManager (singleton that flips X → O → X …)

### A) Base singleton (one-time utility)

**Project** → `Assets/Scripts/Gameplay` → **Create > C# Script** → **PersistentMonoSingleton.cs**, paste:

```csharp
using UnityEngine;

public abstract class PersistentMonoSingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    public static T Instance { get; private set; }

    protected virtual void Awake()
    {
        if (Instance != null && Instance != this) { Destroy(gameObject); return; }
        Instance = this as T;
        DontDestroyOnLoad(gameObject);
        Initialize();
    }

    protected virtual void OnDestroy() { if (Instance == this) Instance = null; }
    protected abstract void Initialize();
}
```

### B) TurnManager

**Project** → `Assets/Scripts/Gameplay` → **Create > C# Script** → **TurnManager.cs**, paste:

```csharp
using UnityEngine;

public class TurnManager : PersistentMonoSingleton<TurnManager>
{
    private bool _xTurn;

    protected override void Initialize() { _xTurn = true; } // X starts

    public bool GetTurn()
    {
        bool current = _xTurn;
        _xTurn = !_xTurn;  // flip for next click
        return current;
    }

    public void ResetTurns() => _xTurn = true;
}
```

**Hierarchy** → **Create Empty** → **TurnManager** → add **TurnManager** component (it will persist).

---

## 5) BoardManager (win/draw detection + reset)

**Project** → `Assets/Scripts/Gameplay` → **Create > C# Script** → **BoardManager.cs**, paste:

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class BoardManager : PersistentMonoSingleton<BoardManager>
{
    public event Action<int, bool> OnGameFinished; // (winnerValue, isWin)
    public event Action OnReset;

    [SerializeField] private List<Cell> cellsList = new(); // assign 9 in Inspector

    private readonly int[][] winConditions = new int[][]
    {
        new[]{0,1,2}, new[]{3,4,5}, new[]{6,7,8}, // rows
        new[]{0,3,6}, new[]{1,4,7}, new[]{2,5,8}, // cols
        new[]{0,4,8}, new[]{2,4,6}                 // diags
    };

    protected override void Initialize() { }

    void Start() => ResetGame();

    void OnEnable()
    {
        foreach (var c in cellsList) c.OnValueChanged += CheckWinCondition;
    }

    void OnDisable()
    {
        foreach (var c in cellsList) c.OnValueChanged -= CheckWinCondition;
    }

    void CheckWinCondition(int cellId, int value)
    {
        if (value == 0) return;

        foreach (var w in winConditions)
        {
            if (cellsList[w[0]].Value == value &&
                cellsList[w[1]].Value == value &&
                cellsList[w[2]].Value == value)
            {
                // mark winners and non-winners
                cellsList[w[0]].SetResult(true);
                cellsList[w[1]].SetResult(true);
                cellsList[w[2]].SetResult(true);

                for (int i = 0; i < cellsList.Count; i++)
                    if (i != w[0] && i != w[1] && i != w[2])
                        cellsList[i].SetResult(false);

                OnGameFinished?.Invoke(value, true);
                return;
            }
        }

        // draw?
        bool allFilled = true;
        foreach (var c in cellsList) if (c.Value == 0) { allFilled = false; break; }
        if (allFilled)
        {
            foreach (var c in cellsList) c.SetResult(false);
            OnGameFinished?.Invoke(0, false);
        }
    }

    public void ResetGame()
    {
        foreach (var c in cellsList) c.Reset();
        TurnManager.Instance.ResetTurns();
        OnReset?.Invoke();
    }
}
```

**Hierarchy** → **Create Empty** → **BoardManager** → add **BoardManager** component.
In **Inspector (BoardManager)**, expand **cellsList** to **9** and **drag the 9 Cell assets** (`Cell_0` … `Cell_8`) in order.

---

## 6) UIBehaviour (status text + restart button)

**Project** → `Assets/Scripts/UI` → **Create > C# Script** → **UIBehaviour.cs**, paste:

```csharp
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class UIBehaviour : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI gameStatusText;
    [SerializeField] private Button restartButton;
    [SerializeField] private GameObject containerPanel; // GameOverPanel

    void OnEnable()
    {
        BoardManager.Instance.OnGameFinished += HandleGameFinished;
        BoardManager.Instance.OnReset += ResetUI;
    }

    void OnDisable()
    {
        if (BoardManager.Instance != null)
        {
            BoardManager.Instance.OnGameFinished -= HandleGameFinished;
            BoardManager.Instance.OnReset -= ResetUI;
        }
    }

    void Start()
    {
        restartButton.onClick.AddListener(RestartGame);
        ResetUI();
    }

    void HandleGameFinished(int value, bool isWin)
    {
        if (isWin)
        {
            string winner = value == 1 ? "X" : "O";
            gameStatusText.text = $"{winner} Player Wins!";
        }
        else
        {
            gameStatusText.text = "It's a Draw! Try Again.";
        }
        containerPanel.SetActive(true);
    }

    void ResetUI()
    {
        gameStatusText.text = "";
        containerPanel.SetActive(false);
    }

    void RestartGame() => BoardManager.Instance.ResetGame();
}
```

**Hierarchy wiring:**

* Select **BoardCanvas/GameOverPanel** → ensure it is **inactive** by default.
* Add **UIBehaviour** to **BoardCanvas** (or to **GameOverPanel**, your choice).
* **Assign fields:**

  * **containerPanel** = `GameOverPanel`
  * **gameStatusText** = `GameOverPanel/GameOverText` (TMP)
  * **restartButton** = `GameOverPanel/RestartButton`
* **OnClick** of `RestartButton` is **already** set via script listener; no extra click event needed.

---

## 7) Wire each Cell instance to a Cell asset

For **each** `Grid/Cell_0 … Cell_8` (prefab instances in the scene):

* Select the **Cell** instance → in **Inspector (UICellBehaviour)**:

  * **cell** = drag the matching **Cell asset** (`Cell_0` → `Cell_0` asset, etc.).
  * **button** = its own **Button** component
  * **image** = its child **Image** component
  * Set **default / win / failed** colors if you want class-specific colors.

*(If any field is missing on multiple instances, open the **Cell prefab** and set defaults there, then **Apply**; only the **cell** reference must be unique per instance.)*

---

## 8) Playtest (exact checks)

* Point a controller **ray** at a cell → hover highlight.
* Click a cell → **X** appears, next click places **O**, and so on.
* Make a 3-in-a-row → the 3 winning cells tint **win color**, others tint **failed color**; **GameOverPanel** appears with the message.
* Click **Play again?** → board clears to blanks; **X** starts again.

---

## Troubleshooting

* **Cell never shows X/O** → On the cell instance, ensure **UICellBehaviour** has **button** and **image** assigned, and the **cell** field points to its unique **Cell asset**.
* **Multiple clicks on one cell** → Ensure **UICellBehaviour** sets **button.interactable = false** on value set (already in code).
* **Win never triggers** → Confirm **BoardManager** has exactly **9** Cell assets in **cellsList**, **ordered** `Cell_0 … Cell_8`.
* **Restart does nothing** → Make sure **UIBehaviour** is in the scene and its references are set; BoardManager and TurnManager must exist.
* **UI doesn’t click** → Confirm **EventSystem** has **Pointable Canvas Module** and **BoardCanvas** has **Pointable Canvas**; Canvas scale `0.001`.

---

## End-of-Part snapshot

### What your **Hierarchy** should look like now

```
OVRCameraRig (OVR path; with rays, grab, teleport from Part 2)
└─ OVRInteraction
└─ OVRController
   ├─ LeftController
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor
   │     ├─ Grab Interactor
   │     └─ Teleport Interactor
   └─ RightController
      └─ ControllerInteractors
         ├─ Controller Ray Interactor
         ├─ Grab Interactor
         └─ Teleport Interactor

Wall
└─ BoardCanvas (World Space; EventSystem has Pointable Canvas Module)
   ├─ BoardPanel
   ├─ Grid (Grid Layout Group)
   │  ├─ Cell_0 (Prefab instance; UICellBehaviour → cell = Cell_0 asset)
   │  └─ … Cell_8 (each bound to its own Cell_X asset)
   └─ GameOverPanel (disabled at start)
      ├─ GameOverText (TMP)
      └─ RestartButton (TMP Button)

BoardManager (component; cellsList has 9 Cell assets in order)
TurnManager  (component; persistent)
EventSystem  (Pointable Canvas Module)
Ground (Teleport Area)
Directional Light
```

### What your **Project** should include now

```
Assets/
  Prefabs/
    Cells/
      Cell.prefab
  Resources/
    x.png
    o.png
    b.png
  Scenes/
    01_VR_TicTacToe.unity
  Scripts/
    Gameplay/
      Cell.cs
      PersistentMonoSingleton.cs
      TurnManager.cs
      BoardManager.cs
    UI/
      UICellBehaviour.cs
      UIBehaviour.cs
```
---
# Part 4
## Snap Interaction (Sockets): grab → hover → precision snap

*(Unity 6.2 • URP • Meta XR AIO v65+ • Your OVR rig path from Parts 1–3 stays exactly as is)*

**Goal of Part 4:** add **Snap Interactions** so a movable 3D object (the **Plug**) snaps perfectly into one or more **Sockets** on the **Wall**. Works with your current **rays**, **grabs**, **locomotion**, and **teleport**. No custom code.

---

## What you’ll touch (flow)

* **Hierarchy**: create Plug and Sockets under your existing scene
* **Inspector**: add **Snap Interactor** (on the Plug) and **Snap Interactable** (on each Socket)
* **Wizards**: use the Interaction SDK **Add Grab Interaction** once for the Plug
* **Play**: tune snap pose live, then copy/paste values after exiting Play

---

## 1) Create the two actors

### A) The target on the wall (Socket)

1. **Hierarchy** → select **Wall** (from Part 2).
2. **Right-click Wall → 3D Object → Cube** → rename **Outlet**.

   * **Transform:** Position `(0.6, 1.3, 2.45)` (a little to the right), Scale `(0.18, 0.18, 0.02)` (thin plate).

### B) The object you will snap (Plug)

1. **Hierarchy** → **3D Object → Cube** → rename **ChargerBrick**.

   * **Transform:** Position `(0, 1.0, 1.2)`, Scale `(0.04, 0.06, 0.025)` (smaller than outlet).
   * Give it a simple **Material** (optional) so it’s easy to see.

> You can use your own models later; shapes here are just clean placeholders to teach snap wiring.

---

## 2) Make the Plug grabbable (wizard, 10 seconds)

1. **Right-click `ChargerBrick` → Interaction SDK → Add Grab Interaction**.
2. In the wizard dialog:

   * Keep defaults (hands/controllers both fine).
   * **Generate Collider = ON**.
   * Click **Fix All** (adds Rigidbody etc.).
   * Click **Create**.

**Result:** the wizard creates a parent host like **`ISDK Hand Grab Interaction`** (name may vary). Your **ChargerBrick** mesh becomes its **child**. The **host** now carries the interaction components (Grabbable/GrabInteractable/ColliderSurface). We’ll attach **Snap Interactor** under this host.

---

## 3) Add the Snap Interactor (on the Plug host)

1. **Hierarchy:** select the wizard **host** object (parent of `ChargerBrick`).
2. **Right-click host → Create Empty** → rename **SnapInteractor** (child of the host).
3. **Inspector (SnapInteractor)** → **Add Component → Snap Interactor**.

   * **Pointable Element:** drag the **host** (it has the Pointable/Grabbable).
   * **Rigidbody:** drag the **Rigidbody** from the **host** (or the child if the wizard placed it there).
   * Leave other defaults for now.

> The Interactor is the “thing that snaps in”.

---

## 4) Add the Snap Interactable (on the Socket)

1. **Hierarchy:** expand **Wall/Outlet**.
2. **Right-click `Outlet` → Create Empty** → rename **Snap_Outlet_A** (child of Outlet).
3. **Inspector (Outlet)** → **Add Component → Rigidbody**

   * **Use Gravity = OFF**, **Is Kinematic = ON** (this outlet stays fixed on the wall).
4. **Inspector (Snap_Outlet_A)** → **Add Component → Box Collider**

   * Resize the Box to match where the plug nose should slide in (a tight box in front of the outlet’s face).
5. **Inspector (Snap_Outlet_A)** → **Add Component → Snap Interactable**

   * **Rigid Body:** drag **Outlet (Rigidbody)** here.
   * **Max Selecting Interactors:** `1` (only one plug per socket).
   * Leave optional visuals empty for now.

> The Interactable is the “hole/seat” that accepts the snap.

---

## 5) Aim the snap pose (live alignment workflow)

1. **Play** with Link/Air Link.
2. Grab **ChargerBrick** (ray or hand), move near **Snap_Outlet_A** → it should **hover/attract** and **snap**.
3. If the snap lands misaligned:

   * While still in **Play**, select **`Snap_Outlet_A`** and **move/rotate** its **Transform** until the snapped plug looks perfect.
   * **Inspector (Transform)** → **⋮** → **Copy Component**.
   * **Exit Play**, select **`Snap_Outlet_A`** again → **⋮ → Paste Component Values**.

> This “tune in Play, paste after” avoids iteration time and gets pixel-perfect placement.

---

## 6) Add a second socket (multi-snap option)

1. **Hierarchy:** **Duplicate** `Snap_Outlet_A` → rename **Snap_Outlet_B**.
2. Move it to the other side of the Outlet plate so you have two sockets.
3. **Play**: the Plug will snap to the **closest** Snap Interactable.

---

## 7) Optional: hover visuals (clear learner feedback)

If your SDK includes a **Snap Interactable Visual**:

1. Select **`Snap_Outlet_A`** (and **B** if you duplicated).
2. **Add Component → Snap Interactable Visual**.

   * **Snap Interactable:** drag same object.
   * **Hover Material:** create a **transparent** material (e.g., Standard Surface, Rendering Mode **Transparent**, low alpha).
   * Assign it so students see a soft ghost where snapping will occur.

---

## Troubleshooting (quick wins)

* **No snapping at all** → Snap Interactor must reference a **Pointable Element** (from the Plug’s wizard host) **and** a **Rigidbody**. The Snap Interactable must also reference a **Rigidbody** (from the Outlet).
* **Snaps but wrong pose** → Adjust **`Snap_Outlet_A`** Transform during **Play**; copy/paste values after Play.
* **Multiple plugs fight** → On each **Snap Interactable**, set **Max Selecting Interactors = 1**.
* **Won’t grab** → Verify the wizard parent exists (host holds Grabbable/GrabInteractable/ColliderSurface) and the Plug mesh has a collider.
* **Falls or jitters** → For fixed sockets, **Outlet Rigidbody** must be **Kinematic**; for the Plug host, keep the wizard defaults (usually non-gravity while grabbed, then sim).
* **UI rays interfere** (optional) → Use your Part 6 mask setup (Tag Set Filter) if you want one hand UI-only and the other 3D-only.

---

## End-of-Part snapshot

### What your **Hierarchy** should look like now

```
OVRCameraRig  (your OVR path from Parts 1–3 + locomotion/teleport)
└─ OVRInteraction
└─ OVRController
   ├─ LeftController
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor
   │     ├─ Grab Interactor
   │     └─ Teleport Interactor
   └─ RightController
      └─ ControllerInteractors
         ├─ Controller Ray Interactor
         ├─ Grab Interactor
         └─ Teleport Interactor

Wall
├─ BoardCanvas   (from Tic-Tac-Toe)
│  ├─ BoardPanel
│  ├─ Grid
│  │  ├─ Cell_0 … Cell_8
│  └─ GameOverPanel (disabled)
└─ Outlet        (Cube + Rigidbody (Kinematic))
   ├─ Snap_Outlet_A (BoxCollider + Snap Interactable)
   └─ Snap_Outlet_B (optional duplicate; BoxCollider + Snap Interactable)

ISDK Hand Grab Interaction   (wizard host for the Plug; exact name may vary)
├─ ChargerBrick              (mesh child; collider from wizard)
└─ SnapInteractor            (Snap Interactor; refs: host Pointable + host Rigidbody)

EventSystem (Pointable Canvas Module)
Ground (Teleport Area)
Directional Light
```

### Key **Inspector** bindings (recap)

* **SnapInteractor (child under Plug host)**

  * **Pointable Element =** the **wizard host** object
  * **Rigidbody =** the **wizard host’s Rigidbody**
* **Snap_Outlet_A / B**

  * **Box Collider** fits socket opening
  * **Snap Interactable → Rigidbody =** **Outlet (Kinematic)**
  * **Max Selecting Interactors = 1**

---

### What students should be able to do now

* Grab the **ChargerBrick** with ray or hand.
* Move near **Snap_Outlet_A/B** → see hover/attract → **release** → object **snaps** perfectly in place.
* Teleport and locomote around while repeating the interaction.

---




\
\

\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\
\\\
\\
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\







































































































🔹 Part 1 — Rig & Foundation (Meta Interaction SDK only)

<span style="color:purple;font-weight:bold;">🟣 </span>

**Goal:** Clean scene with **controller rays**, **grab**, **locomotion** (move + snap turn), and a **world-space button** you can click with the ray — using **current** Meta Interaction SDK patterns.

You will use: **Build Settings**, **Project Settings**, **Package Manager**, **Hierarchy**, **Inspector**, **Project**, **Console**.

---

## 0) One-time Project Baseline (Android • URP • Meta) <span style="color:purple;">🟣</span>

⬜ **0.1** **File → Build Settings → Android → Switch Platform**
 • **Texture Compression = ASTC**

⬜ **0.2** **Project Settings → Player → Other Settings**
 • **Scripting Backend = IL2CPP**, **ARM64** only
 • **Minimum API ≥ 29**, **Target API = Highest Installed**

⬜ **0.3** **Project Settings → Graphics / Quality**
 • **Scriptable Render Pipeline Settings =** your **URP Asset**

⬜ **0.4** **Meta → Tools → Meta XR Project Setup → Fix All**
 • Accept all recommended fixes.

⬜ **0.5** **Player → Active Input Handling = Both**
⬜ **0.6** **Project folders:** `Assets/_Scenes`, `Assets/Scripts`, `Assets/Prefabs`, `Assets/Materials`, `Assets/UI`, `Assets/Audio`

---

## 1) New Scene & Clean Stage <span style="color:purple;">🟣</span>

⬜ **1.1** **File → New Scene** → **Save As** `Assets/_Scenes/S09_TicTacToe.unity`

⬜ **1.2** **Hierarchy:** select **Main Camera** → **Delete**

⬜ **1.3** Keep **Directional Light**; set **Intensity ≈ 0.9**, **Soft Shadows = ON**

---

## 2) Base Rig (modern): `OVRCameraRig` + `OVRManager` <span style="color:purple;">🟣</span>

⬜ **2.1** **Project (Search):** `OVRCameraRig`

 **Drag** **OVRCameraRig** to **Hierarchy (root)** — **rename** **`PlayerRig`** *(optional, clear name; still not “XR”)*

⬜ **2.2** **Create an app settings holder**
 **Hierarchy:** Right-click → **Create Empty** → **rename `AppSettings`** (at root)
 
 **Inspector (AppSettings):** **Add Component → OVRManager**
 
 *(Some AIO versions put OVRManager elsewhere; having it once in the scene is enough.)*
 

> You now have tracked camera anchors under `PlayerRig/TrackingSpace` (Left/Right/Center).

---

## 3) Interaction SDK hands (prefabs) <span style="color:purple;">🟣</span>

> We’ll add the **Interaction SDK** controller prefabs so rays & grabs are standardized.

⬜ **3.1** **Project (Search):** `OVRInteraction`
 **Drag** it **as a child of** **`PlayerRig`** (keep local transform zeroed)

⬜ **3.2** **Project (Search):** `OVRController`

 **Drag** it **under** `PlayerRig/OVRInteraction`
 
 • This prefab (or its variant) spawns/hosts **Left/Right controller** objects and commonly includes **ControllerInteractors** children.
 
 • If your AIO version splits Left/Right into two prefabs, add both under **OVRInteraction**.
 

> If your version doesn’t provide these prefabs, you can still add interactors manually in Step 4 — the rest of the flow stays the same.

---

## 4) Controller Rays + Grab (verify / add) <span style="color:purple;">🟣</span>

> Many `OVRController` variants already include these. If not, add them under each hand.

⬜ **4.1** **Hierarchy path (Left hand):**
`PlayerRig/TrackingSpace/LeftHandAnchor`

 • If you see a child like **`ControllerInteractors`**, select it;
 else: Right-click **LeftHandAnchor → Create Empty → rename `ControllerInteractors`** (reset local transform).

⬜ **4.2** **Add/Verify components on the Left `ControllerInteractors`:**
 **Add Component → Controller Ray Interactor**
 
  – **Max Ray Length = 6**
  
  – **Hide When No Interactable = ON** (optional)
  
  – We’ll assign a **reticle** in step 4.5
  
 **Add Component → Grab Interactor** *(near-grab)*

⬜ **4.3** Repeat **4.1–4.2** for **RightHandAnchor**.

⬜ **4.4** **Grab test target** (we’ll use it in Step 8):
 We’ll prepare a cube later; the hands are ready to interact.

⬜ **4.5** **Reticle prefab**

 **Hierarchy:** 3D Object → **Torus** → rename **`ReticleRing`**
 
 **Transform:** **Scale (0.05,0.05,0.05)**
 
 **Material:** `Assets/Materials/M_Reticle_Unlit` (URP/Lit, Base=white, **Emission ON**) → assign
 
 **Project:** drag **ReticleRing** to **Assets/Prefabs** → **delete from scene**
 
 **Assign to rays:**
 
 - Select each **Controller Ray Interactor** (Left & Right) → if it exposes a **Reticle/Visual** slot, **drag `Prefabs/ReticleRing`** there.
 
 (If your component uses a child “Ray Visual” script, add it and assign the prefab as its reticle.)
 

> Result: both hands have **far-ray** + **near-grab** capability.

---

## 5) Locomotion (modern): CharacterController + tiny driver <span style="color:purple;">🟣</span>

> We won’t use the legacy `OVRPlayerController`. We’ll add a **CharacterController** to `PlayerRig` and drive it with a small, clear script using **OVRInput** sticks.

⬜ **5.1** **Hierarchy:** select **`PlayerRig`**

 **Inspector:** **Add Component → Character Controller**
 
 – **Height = 1.7**, **Radius = 0.3**, **Center Y ≈ 0.85**
 
 (Adjust later if your head starts in the ceiling.)

⬜ **5.2** **Project:** `Assets/Scripts/SimpleRigLocomotion.cs`

```csharp
using UnityEngine;

/// <summary>
/// Modern, prefab-free locomotion:
/// - Left stick = move (relative to HMD forward on XZ)
/// - Right stick = snap turn
/// Requires CharacterController on the same GameObject, and a reference to CenterEyeAnchor.
/// </summary>
[RequireComponent(typeof(CharacterController))]
public class SimpleRigLocomotion : MonoBehaviour
{
    [Header("Assign CenterEyeAnchor from PlayerRig/TrackingSpace")]
    public Transform cameraEye;

    [Header("Move")]
    public float moveSpeed = 1.8f;   // m/s
    public float gravity = -9.81f;   // m/s^2
    private float _yVel = 0f;

    [Header("Snap Turn")]
    public float snapAngleDeg = 45f;
    public float snapDeadzone = 0.7f;
    public float snapCooldown = 0.25f;
    private float _snapTimer = 0f;

    private CharacterController _cc;

    void Awake() => _cc = GetComponent<CharacterController>();

    void Update()
    {
        if (!_cc || cameraEye == null) return;

        // --- Move (Left stick) ---
        Vector2 move = OVRInput.Get(OVRInput.Axis2D.PrimaryThumbstick); // LTouch
        Vector3 fwd = cameraEye.forward; fwd.y = 0; fwd.Normalize();
        Vector3 right = cameraEye.right; right.y = 0; right.Normalize();
        Vector3 planar = (fwd * move.y + right * move.x) * moveSpeed;

        // Gravity
        if (_cc.isGrounded && _yVel < 0) _yVel = -0.5f;
        _yVel += gravity * Time.deltaTime;

        Vector3 motion = new Vector3(planar.x, _yVel, planar.z);
        _cc.Move(motion * Time.deltaTime);

        // --- Snap turn (Right stick X) ---
        _snapTimer -= Time.deltaTime;
        Vector2 turn = OVRInput.Get(OVRInput.Axis2D.SecondaryThumbstick); // RTouch
        if (_snapTimer <= 0f && Mathf.Abs(turn.x) >= snapDeadzone)
        {
            float sign = Mathf.Sign(turn.x);
            transform.Rotate(0f, sign * snapAngleDeg, 0f);
            _snapTimer = snapCooldown;
        }
    }
}
```

⬜ **5.3** **Hierarchy:** select **`PlayerRig`** → **Add Component → SimpleRigLocomotion**
 • **Camera Eye:** drag **`PlayerRig/TrackingSpace/CenterEyeAnchor`**

> ✅ Left stick moves relative to the headset; Right stick snap-turns. No legacy prefab involved.

---

## 6) World-Space Test UI (via Interaction SDK wizard) <span style="color:purple;">🟣</span>

> We’ll use the **Interaction SDK** canvas wizard so the **EventSystem** gets the **Pointable Canvas Module** automatically (modern pipeline).

⬜ **6.1** **Hierarchy:** Right-click → **UI → Canvas** → rename **`TestCanvas`**

 **Inspector (Canvas):** **Render Mode = World Space**
 
 **RectTransform:** **Position (0,1.5,1.5)** • **Size (0.6,0.3)** (meters) • **Scale (1,1,1)**
 

⬜ **6.2** **Run the wizard on the canvas**

 **Hierarchy:** Right-click **`TestCanvas`** → **Interaction SDK → Add Ray Interaction to Canvas**
 
 Follow prompts → **Fix** (adds **Pointable Canvas Module** to EventSystem) → **Create** (adds any required helpers).
 

> You should now have exactly one **EventSystem** in the scene with **Pointable Canvas Module** (modern), not OVR Input Module.

⬜ **6.3** **Button (TMP)**
 **TestCanvas** → **UI → Button (TextMeshPro)** → rename **`TestButton`**
 
 **Button → Transition = Color Tint** (brighter Highlighted)
 
 **Child `Text (TMP)` → “CLICK ME”**, **Font Size 48–72**, **Alignment Center**
 

⬜ **6.4** **Click probe**

 **Project:** `Assets/Scripts/UIButtonDebug.cs`

```csharp
using UnityEngine;
public class UIButtonDebug : MonoBehaviour
{
    public void PrintClick()
    {
        Debug.Log("[UI] Button clicked via Interaction SDK pointer.");
    }
}
```

 **Hierarchy:** select **TestButton** → **Add Component → UIButtonDebug**
 **Button (OnClick)**: **+** → drag **TestButton** → **UIButtonDebug.PrintClick()**

---

## 7) Minimal Room <span style="color:purple;">🟣</span>

⬜ **7.1** **Ground**

 3D Object → **Plane** → rename **`Ground`** → **Scale (4,1,4)**

⬜ **7.2** **Wall**

 3D Object → **Cube** → rename **`Wall`** → **Position (0,1.5,2.2)** • **Scale (3,2,0.08)**

---

## 8) Grabbable test cube (manual or wizard) <span style="color:purple;">🟣</span>

⬜ **8.1** **Hierarchy:** 3D Object → **Cube** → rename **`GrabCube`**

 **Transform:** **Scale (0.2,0.2,0.2)**, **Position (0,0.5,1.8)**
 

**Manual setup (transparent & explicit):**
⬜ **8.2** **Inspector (GrabCube):**

 • **Add Component → Rigidbody** (Use Gravity ON, Interpolate ON)
 
 • **Add Component → Grabbable** → **Rigidbody:** drag this cube’s **Rigidbody**
 
 • **Add Component → ColliderSurface** → **Collider:** drag this cube’s **BoxCollider**
 
 • **Add Component → RayInteractable** →
 
  – **Pointable Element = Grabbable**
  
  – **Surface = ColliderSurface**
  
 • **Add Component → MoveTowardsTargetProvider**
 
  – In **RayInteractable → Movement Provider:** assign this provider

**Wizard alternative (faster):**

⬜ **8.3** Right-click **GrabCube** → **Interaction SDK → Add Ray Grab Interaction**

 • Wizard creates a **parent host** (e.g., `ISDK Ray Grab Interaction`) and moves your cube under it.
 
 • Edit settings on the **parent** (that’s the interactable host).

---

## 9) Play Test (locomotion + rays + grab + UI) <span style="color:purple;">🟣</span>

1. **Enter Play** (Quest Link/AirLink recommended).
2. **Move** with **Left stick**; **snap-turn** with **Right stick**.
3. Aim at **TestButton** → **pull trigger** → Console prints the click message.
4. Aim at **GrabCube** → **ray-grab**; move hand close → **near-grab** with grip; release to drop.

---

## 10) Your Hierarchy should roughly look like

```
PlayerRig  (OVRCameraRig + CharacterController + SimpleRigLocomotion)
└─ TrackingSpace
   ├─ LeftHandAnchor
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor (+ ReticleRing)
   │     └─ Grab Interactor
   ├─ RightHandAnchor
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor (+ ReticleRing)
   │     └─ Grab Interactor
   └─ CenterEyeAnchor
AppSettings (OVRManager)
EventSystem (Pointable Canvas Module)   ← (added by canvas wizard)
TestCanvas (World Space)
└─ TestButton (TMP + UIButtonDebug)
Ground
Wall
GrabCube (or wizard parent host with cube as child)
Directional Light
```

> ✅ Notice: **no** object named “XR”.

---

## 11) Common mistakes & quick fixes

* **No UI clicks:**
  – Make sure you used the **Interaction SDK → Add Ray Interaction to Canvas** wizard. EventSystem must have **Pointable Canvas Module** (not OVR Input Module).
  – Canvas must be **World Space** and reachable by your ray.

* **No rays / no reticle:**
  – Ensure each **hand** has **ControllerInteractors** with **Controller Ray Interactor**.
  – Assign the **ReticleRing** prefab if your component exposes a reticle slot.

* **Can’t move:**
  – `PlayerRig` needs **CharacterController** and **SimpleRigLocomotion** with **Camera Eye** set to **CenterEyeAnchor**.
  – Ensure the capsule height/center makes sense (feet on Ground).

* **Can’t grab:**
  – Hands need **Grab Interactor**.
  – Cube must be fully set up (or via wizard parent host).

* **Two EventSystems present:**
  – Keep **one** (the wizard’s). Delete extra EventSystems.

---

## ✅ End of Part 1 — what to submit

* Screenshot: **PlayerRig** with **CharacterController** + **SimpleRigLocomotion** (Camera Eye assigned).
* Screenshot: **Left/Right** `ControllerInteractors` showing **Ray Interactor** + **Grab Interactor** (and Reticle if exposed).
* Screenshot: **EventSystem** with **Pointable Canvas Module** (from the wizard).
* Short clip: **move**, **snap-turn**, **click the button**, **grab the cube**.

---
---

# 🔹 Part 2 — Wall Board, BoardCanvas, 3×3 Grid, Cell Prefab (Modern)

<span style="color:purple;font-weight:bold;">🟣 complete every purple step</span>

**Goal:** A wall with a **World-Space canvas** that your controller rays can hover/click, a **3×3 Grid**, and a **Cell** prefab (TMP Button).
**Pipeline:** Interaction SDK with **Pointable Canvas Module** (added via the “**Add Ray Interaction to Canvas**” wizard).   

---

## 0) Pre-flight (must be true from Part 1) <span style="color:purple;">🟣</span>

⬜ **0.1** You have **`PlayerRig`** (OVRCameraRig) with **Controller Ray Interactor** + **Grab Interactor** on both hands.   
⬜ **0.2** Scene has **EventSystem** with **Pointable Canvas Module** (added by the canvas wizard in Part 1).   
⬜ **0.3** If **TestCanvas** from Part 1 is still in the scene, either **Disable** it or move it aside so it won’t overlap the new board.

---

## 1) Build the wall (3D) <span style="color:purple;">🟣</span>

⬜ **1.1** **Hierarchy:** Right-click → **3D Object → Cube** → rename **`Wall`**
⬜ **1.2** **Inspector (Wall → Transform):**

* **Position:** (0, **1.5**, **2.2**)
  
* **Rotation:** (0, 0, 0)
  
* **Scale:** (**3**, **2**, **0.08**)

> Result: a flat wall ~3 m wide × 2 m tall in front of the player.

---

## 2) Create the Board Canvas (World Space, modern UI path) <span style="color:purple;">🟣</span>

⬜ **2.1** **Hierarchy:** Right-click **`Wall`** → **UI → Canvas** → rename **`BoardCanvas`**

⬜ **2.2** **Inspector (BoardCanvas → Canvas):** **Render Mode = World Space**

⬜ **2.3** **Inspector (BoardCanvas → RectTransform):**

* **Position:** (0, **1.3**, **-0.045**) *(a few cm in front of the wall; local Z is negative toward you)*
  
* **Size:** **Width = 0.8**, **Height = 0.8** *(meters)*
  
* **Scale:** (1, 1, 1)

### 2A) Run the Interaction SDK canvas wizard (critical)

⬜ **2.4** **Hierarchy:** 
Right-click **`BoardCanvas`** → **Interaction SDK → Add Ray Interaction to Canvas** → click **Fix** (adds **Pointable Canvas Module** on the **EventSystem**) → **Create** (adds any helpers).   

> This is the **modern** pipeline for ray-cast UI with the Interaction SDK (you don’t add `OVR Input Module` or `OVR Raycaster` for this flow). ( [3])

*(Optional)* Set **Layer = UIWorld** on `BoardCanvas` if you plan to separate UI/3D by layers in Part 5.

---

## 3) Add a subtle board background (Panel) <span style="color:purple;">🟣</span>

⬜ **3.1** **Hierarchy:** Right-click **`BoardCanvas`** → **UI → Panel** → rename **`BoardPanel`**

⬜ **3.2** **Inspector (BoardPanel → RectTransform):**

* **Anchor Preset = Stretch (full)**
  
* **Left/Right/Top/Bottom = 0**
  ⬜ **3.3** **Inspector (Image):** **Color =** light gray (≈ #D9D9D9, Alpha ~ 200/255)

---

## 4) Make the Grid container (3×3) <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy:** Right-click **`BoardCanvas`** → **Create Empty** → rename **`Grid`**

⬜ **4.2** **Inspector (Grid → RectTransform):**

* **Anchor Preset = Stretch (full)**
  
* **Left/Right/Top/Bottom = 40** *(~4–5 cm margin inside the board)*

⬜ **4.3** **Inspector (Grid):** 
**Add Component → Grid Layout Group**
* **Constraint = Fixed Column Count**
* **Constraint Count = 3**
  
* **Cell Size = (0.22, 0.22)** *(meters)*
* **Spacing = (0.02, 0.02)**
* **Child Alignment = Middle Center**
* **Start Axis = Horizontal**, **Start Corner = Upper Left**

> Any child placed under `Grid` tiles automatically into 3×3.

---

## 5) Create **one** Cell as a proper UI Button (TMP) <span style="color:purple;">🟣</span>

⬜ **5.1** **Hierarchy:** Right-click **`Grid`** → **UI → Button (TextMeshPro)** → rename **`Cell`**
*(If TMP Essentials prompt appears, click **Import TMP Essentials**.)*
⬜ **5.2** **Inspector (Cell → RectTransform):** leave sizing to the **Grid** (don’t resize manually).
⬜ **5.3** **Inspector (Cell → Button):**

* **Transition = Color Tint**
* **Normal:** #FFFFFF
* **Highlighted:** #F3F3F3
* **Pressed:** #E0E0E0
* **Selected:** #F3F3F3
* **Disabled:** #B0B0B0
* **Fade Duration:** **0.06** (snappy)

⬜ **5.4** **Child `Text (TMP)` (inside Cell):**

* **Text =** *(leave empty)* — gameplay will set “X/O” later
* **Font Size = 64** (start), **Auto Size = OFF**
* **Alignment = Center**
* **Color = Black**

*(Optional visual)* Add **Outline** to `Cell` (Effect Color #000000 @ 50% alpha, Distance (2,-2)) for a subtle frame.

---

## 6) Turn Cell into a **Prefab**, then place **nine** cells <span style="color:purple;">🟣</span>

⬜ **6.1** **Project Window:** ensure folder **`Assets/Prefabs`** exists
⬜ **6.2** **Hierarchy:** drag **`Cell`** → **`Assets/Prefabs`** to create **`Cell.prefab`**
⬜ **6.3** **Hierarchy:** delete the scene copy of `Cell` (so `Grid` is empty)
⬜ **6.4** **Project → Prefabs:** drag **`Cell.prefab`** into **`Grid`** **nine** times
⬜ **6.5** **Rename** the instances in **reading order** (top-left to bottom-right):
`Cell_0`, `Cell_1`, `Cell_2`, `Cell_3`, `Cell_4`, `Cell_5`, `Cell_6`, `Cell_7`, `Cell_8`

> This **order matters** for Part 3 when we bind the array.

---

## 7) Prep the Game Over overlay (disabled for now) <span style="color:purple;">🟣</span>

⬜ **7.1** **Hierarchy:** Right-click **`BoardCanvas`** → **UI → Panel** → rename **`GameOverPanel`**
⬜ **7.2** **Inspector (GameOverPanel):**

* **Anchor Preset = Stretch (full)**, **Left/Right/Top/Bottom = 0**
* **Image Color =** black **(#000000, Alpha ~ 140/255)**
* **Active = OFF** *(disable the GameObject now)*

⬜ **7.3** **Hierarchy:** Right-click **`GameOverPanel`** → **UI → Text (TextMeshPro)** → rename **`GameOverText`**

* **Text = “X wins!”** (placeholder)
* **Font Size = 96**, **Alignment = Center**
* **RectTransform:** center; **Width = 0.7**, **Height = 0.15** (m)
* **Color = White**

⬜ **7.4** **Hierarchy:** Right-click **`GameOverPanel`** → **UI → Button (TextMeshPro)** → rename **`RestartButton`**

* **Text = “New Round”** (we’ll wire it in Part 6 as New Round)
* **Font Size = 48**, **Alignment = Center**
* **RectTransform:** under text; **Width = 0.35**, **Height = 0.10** (m)

---

## 8) Quick hover test (no gameplay yet) <span style="color:purple;">🟣</span>

⬜ **8.1** **Enter Play** (Quest Link/AirLink recommended).
⬜ **8.2** Aim the **Right hand** (or either, for now) ray at the cells → you should see **hover** and **press** color changes when squeezing the trigger.

> If hover/press doesn’t work, re-run the **Add Ray Interaction to Canvas** wizard on `BoardCanvas` so the **EventSystem** has the **Pointable Canvas Module** configured correctly.   

---

## 9) What your **Hierarchy** should look like now

```
PlayerRig  (OVRCameraRig + CharacterController + SimpleRigLocomotion)
└─ TrackingSpace
   ├─ LeftHandAnchor
   │  └─ ControllerInteractors
   │     ├─ Controller Ray Interactor
   │     └─ Grab Interactor
   └─ RightHandAnchor
      └─ ControllerInteractors
         ├─ Controller Ray Interactor
         └─ Grab Interactor

Wall
└─ BoardCanvas  (World Space; wizard added Pointable Canvas Module to EventSystem)
   ├─ BoardPanel (stretched background)
   ├─ Grid (Grid Layout Group: 3 cols, cell 0.22, spacing 0.02)
   │  ├─ Cell_0 (Prefab instance)
   │  └─ … Cell_8
   └─ GameOverPanel (disabled)
      ├─ GameOverText (TMP)
      └─ RestartButton (TMP Button)

EventSystem (with Pointable Canvas Module)
Ground
Directional Light
```

---

## 10) Troubleshooting (precise checks) <span style="color:purple;">🟣</span>

* **UI won’t hover/click:**
  – You must run **Interaction SDK → Add Ray Interaction to Canvas** on **this** `BoardCanvas`. That ensures the **EventSystem** has **Pointable Canvas Module** (modern path).   
  – Canvas must be **World Space** and physically reachable by the ray.
* **Cells tile weirdly / wrong spacing:**
  – On `Grid`, confirm **Grid Layout Group** values: **Fixed Column Count=3**, **Cell (0.22, 0.22)**, **Spacing (0.02, 0.02)**, margins **40**.
* **Board appears behind wall:**
  – `BoardCanvas` **local Z** should be **-0.045** under the **Wall** so it floats just in front.
* **Text tiny/huge:**
  – Remember **meters**: adjust **TMP Font Size** and/or the **Canvas Width/Height** (not Canvas scale).

---

## ✅ End of Part 2 — submit

* Screenshot: **`BoardCanvas`** Inspector (**Render Mode = World Space**; confirm you ran the wizard).
* Screenshot: **`Grid`** with **Grid Layout Group** values.
* Screenshot: **nine** `Cell_0 … Cell_8` under `Grid`.
* Short clip: controller ray hovering/pressing cells (no X/O yet).

---
---

# 🔹 Part 3 — Gameplay Logic (tiny code slices + bulletproof wiring)

<span style="color:purple;font-weight:bold;">🟣 complete every purple step</span>

**Goal:** Your wall grid becomes a working Tic-Tac-Toe game controlled by **ray clicks**.

**You will use:** `Assets/Scripts`, **Inspector** (BoardCanvas, Cells), **Hierarchy**, **Console**.

---

## 0) Pre-flight from Part 2 (must be true) <span style="color:purple;">🟣</span>

⬜ **0.1** In **Hierarchy**:

* `Wall/BoardCanvas/BoardPanel`
* `Wall/BoardCanvas/Grid/Cell_0 .. Cell_8` (each is **Button (TMP)** with a **Text (TMP)** child)
* `Wall/BoardCanvas/GameOverPanel` (disabled), with **GameOverText** + **RestartButton**
  ⬜ **0.2** Scene has **EventSystem** (added by *Interaction SDK → Add Ray Interaction to Canvas*) → UI ray clicks already hover/press.

---

## 1) Make the scripts (folder + files) <span style="color:purple;">🟣</span>

⬜ **1.1** **Project Window:** Right-click `Assets/Scripts` → **Create → Folder** → name **`TicTacToe`**
⬜ **1.2** Inside it create **two scripts**:

* **`Space.cs`** (goes on each Cell)
* **`GameController.cs`** (goes on a single manager object)

---

## 2) Script 1 — `Space.cs` (lives on every Cell) <span style="color:purple;">🟣</span>

> Handles one click: writes **X/O** in its TMP, disables its Button, tells the controller “I played!”

⬜ **2.1** Open **`Space.cs`**, paste:

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

/// <summary>
/// Attach to each Cell (the Button GameObject).
/// On click:
///  - Writes current player's mark into its TMP label
///  - Disables its Button so it can't be clicked twice
///  - Notifies GameController to process the turn
/// </summary>
public class Space : MonoBehaviour
{
    [Header("Assign on the Cell")]
    public Button button;       // the Button on this Cell
    public TMP_Text buttonText; // the TMP child inside this Cell

    private GameController gameController;

    public void SetControllerReference(GameController control) => gameController = control;

    /// <summary>Hook this from the Button's OnClick() in the Inspector.</summary>
    public void SetSpace()
    {
        if (gameController == null || buttonText == null || button == null)
            return;

        if (!string.IsNullOrEmpty(buttonText.text))
            return; // already marked

        buttonText.text = gameController.GetSide(); // "X" or "O"
        button.interactable = false;

        gameController.EndTurn(); // evaluate win/tie or change turn
    }
}
```

⬜ **2.2** **Hierarchy:** Select **all 9** `Cell_0..Cell_8` → **Add Component → Space**
⬜ **2.3** Still selected:

* Drag each **Cell’s own Button** into **Space → Button**
* Drag the **child `Text (TMP)`** into **Space → Button Text**
  ⬜ **2.4** **Wire the click:** In **Button (OnClick)** on each Cell → **+** → drag **the same Cell** → pick **`Space.SetSpace()`**

> ✅ Now a click can mark a cell and notify the controller. Next we create the controller.

---

## 3) Script 2 — `GameController.cs` (the brain) <span style="color:purple;">🟣</span>

> Holds references to the **9 TMP labels**, tracks **turns**, checks **win/tie**, shows **Game Over**, and **restarts**.

⬜ **3.1** Open **`GameController.cs`**, paste:

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

/// <summary>
/// Scene-wide manager for TicTacToe:
/// - Holds the 9 cell labels (TMP_Text)
/// - Tracks current side ("X" or "O") and move count
/// - Checks win/tie after each move
/// - Shows GameOver overlay and handles Restart
/// </summary>
public class GameController : MonoBehaviour
{
    [Header("Board cell labels (size = 9; drag Text(TMP) from Cell_0..Cell_8)")]
    public TMP_Text[] spaceList = new TMP_Text[9];

    [Header("Game Over UI (under BoardCanvas)")]
    public GameObject gameOverPanel; // BoardCanvas/GameOverPanel
    public TMP_Text gameOverText;    // BoardCanvas/GameOverPanel/GameOverText
    public Button restartButton;     // BoardCanvas/GameOverPanel/RestartButton

    private string side = "X"; // whose turn
    private int moves = 0;     // how many marks placed

    private static readonly int[][] Wins = new int[][]
    {
        new[]{0,1,2}, new[]{3,4,5}, new[]{6,7,8},   // rows
        new[]{0,3,6}, new[]{1,4,7}, new[]{2,5,8},   // cols
        new[]{0,4,8}, new[]{2,4,6}                  // diagonals
    };

    private void Start()
    {
        // Tell each Space who the controller is
        SetGameControllerReferenceForButtons();

        // Initial state
        side = "X";
        moves = 0;

        if (gameOverPanel) gameOverPanel.SetActive(false);
        if (restartButton) restartButton.gameObject.SetActive(false);
    }

    public string GetSide() => side;

    public void EndTurn()
    {
        moves++;

        if (WinCheck())
        {
            GameOver($"{side} wins!");
            return;
        }

        if (moves >= 9) // all cells filled
        {
            GameOver("Tie!");
            return;
        }

        // next player's turn
        side = (side == "X") ? "O" : "X";
    }

    public void Restart()
    {
        side = "X";
        moves = 0;

        for (int i = 0; i < spaceList.Length; i++)
        {
            if (spaceList[i]) spaceList[i].text = string.Empty;

            var btn = spaceList[i] ? spaceList[i].GetComponentInParent<Button>() : null;
            if (btn) btn.interactable = true;
        }

        if (gameOverPanel) gameOverPanel.SetActive(false);
        if (restartButton) restartButton.gameObject.SetActive(false);
    }

    // --------- Helpers ---------

    private void SetGameControllerReferenceForButtons()
    {
        for (int i = 0; i < spaceList.Length; i++)
        {
            if (!spaceList[i]) continue;
            var space = spaceList[i].GetComponentInParent<Space>();
            if (space != null) space.SetControllerReference(this);
            else Debug.LogWarning($"[GameController] No Space script for index {i}.");
        }
    }

    private bool WinCheck()
    {
        foreach (var w in Wins)
        {
            var a = spaceList[w[0]];
            var b = spaceList[w[1]];
            var c = spaceList[w[2]];
            if (!a || !b || !c) continue;

            if (a.text == side && b.text == side && c.text == side)
                return true;
        }
        return false;
    }

    private void GameOver(string message)
    {
        if (gameOverPanel) gameOverPanel.SetActive(true);
        if (gameOverText)  gameOverText.text = message;

        // lock the board
        SetInteractable(false);

        // show restart and wire it
        if (restartButton)
        {
            restartButton.gameObject.SetActive(true);
            restartButton.onClick.RemoveAllListeners();
            restartButton.onClick.AddListener(Restart);
        }
    }

    private void SetInteractable(bool enabled)
    {
        for (int i = 0; i < spaceList.Length; i++)
        {
            var btn = spaceList[i] ? spaceList[i].GetComponentInParent<Button>() : null;
            if (btn) btn.interactable = enabled;
        }
    }
}
```

---

## 4) Add a manager object & fill references (the “glue”) <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy:** Right-click → **Create Empty** → rename **`SceneManager`**
⬜ **4.2** **Inspector (SceneManager):** **Add Component → GameController**

### 4A) Fill the 9 TMP labels in the correct order

⬜ **4.3** **Inspector (GameController → spaceList):** set **Size = 9**
Drag these **TMP Text** objects to their slots (reading order: top-left → bottom-right):

| Index | Drag from Hierarchy (TMP Text under each cell) |
| ----- | ---------------------------------------------- |
| 0     | `Wall/BoardCanvas/Grid/Cell_0/Text (TMP)`      |
| 1     | `Wall/BoardCanvas/Grid/Cell_1/Text (TMP)`      |
| 2     | `Wall/BoardCanvas/Grid/Cell_2/Text (TMP)`      |
| 3     | `Wall/BoardCanvas/Grid/Cell_3/Text (TMP)`      |
| 4     | `Wall/BoardCanvas/Grid/Cell_4/Text (TMP)`      |
| 5     | `Wall/BoardCanvas/Grid/Cell_5/Text (TMP)`      |
| 6     | `Wall/BoardCanvas/Grid/Cell_6/Text (TMP)`      |
| 7     | `Wall/BoardCanvas/Grid/Cell_7/Text (TMP)`      |
| 8     | `Wall/BoardCanvas/Grid/Cell_8/Text (TMP)`      |

> ⚠️ Order matters for win checks.

### 4B) Hook the Game Over UI

⬜ **4.4** **Game Over Panel:** drag `Wall/BoardCanvas/GameOverPanel`
⬜ **4.5** **Game Over Text:** drag `Wall/BoardCanvas/GameOverPanel/GameOverText`
⬜ **4.6** **Restart Button:** drag `Wall/BoardCanvas/GameOverPanel/RestartButton`

---

## 5) Quick playtest (what you should see) <span style="color:purple;">🟣</span>

1. **Enter Play**.
2. Aim a controller **ray** at a cell → **pull trigger** → cell shows **“X”**, becomes **non-clickable**.
3. Click another cell → shows **“O”**, disables.
4. Force a win (e.g., **X** on 0,1,2): **GameOverPanel** appears: “**X wins!**”.
5. Click **Restart**: board clears, **X** starts again.

---

## 6) Troubleshooting (exact spots to check) <span style="color:purple;">🟣</span>

* **Click does nothing:**
  – On each Cell, **Button → OnClick** must call **`Space.SetSpace()`**.
  – The **Space** component must have **Button** and **Button Text** assigned.
* **Marks appear but win isn’t detected:**
  – Check **spaceList** order: **0..8** top-left → bottom-right.
* **Restart visible but does nothing:**
  – Make sure **RestartButton** field is assigned; code wires it at runtime.
* **Overlay never appears:**
  – **GameOverPanel** must be assigned and initially **disabled** in the Hierarchy.
* **UI doesn’t hover/press at all:**
  – Re-run **Interaction SDK → Add Ray Interaction to Canvas** on **BoardCanvas** to ensure the **EventSystem** has **Pointable Canvas Module**.

---

## 7) (Optional helper) Auto-wire the 9 TMP labels by name

> If students struggle with Step 4A, add this to **GameController** and run it via the component’s context menu.

```csharp
[ContextMenu("AutoWire from Grid (Cell_0..Cell_8)")]
private void AutoWireFromGrid()
{
    var grid = GameObject.Find("Grid");
    if (!grid) { Debug.LogWarning("Grid not found."); return; }

    var texts = grid.GetComponentsInChildren<TMP_Text>(true);
    for (int i = 0; i < 9; i++)
    {
        foreach (var t in texts)
        {
            if (t.transform.parent && t.transform.parent.name == $"Cell_{i}")
            {
                spaceList[i] = t;
                break;
            }
        }
    }
    Debug.Log("AutoWire complete.");
}
```

> Use it by selecting **SceneManager → GameController** → click the component’s **⋮** (or right-click header) → **AutoWire from Grid**.

---

## ✅ End of Part 3/6 — submit

* Screenshot: **SceneManager → GameController** with **spaceList[0..8]** filled + Game Over refs assigned.
* Short clip: play a round, show **Game Over**, click **Restart**, play again.

---
---

# 🔹 Part 4 — VR UX Polish (Reticle, Hover/Press, Haptics, SFX)

<span style="color:purple;font-weight:bold;">🟣 </span>

**Goal:**

* Reticle stays **crisp** and **constant-size** in view.
* Cells show **clear hover/press** feedback (plus a tiny **scale-on-hover**).
* Gentle **haptics** on each move; stronger pulse on **win**.
* **Click** and **win** **sounds**.

**You’ll touch:** `Assets/Scripts`, **BoardCanvas**, **EventSystem** (already has Pointable Canvas Module), **Reticle prefab**, **Cell prefab**.

---

## 0) Pre-flight (from Parts 1–3) <span style="color:purple;">🟣</span>

⬜ UI ray hover/press already works (you ran **Interaction SDK → Add Ray Interaction to Canvas** on `BoardCanvas`).
⬜ `SceneManager` has **GameController** with `spaceList[0..8]` and GameOver refs.
⬜ Reticle prefab **`Prefabs/ReticleRing`** is assigned on both **Controller Ray Interactors** (from Part 1).
⬜ No object named **“XR”** in the scene.

---

## 1) Keep the reticle crisp & constant size <span style="color:purple;">🟣</span>

### 1A) Script: `ReticleScaler.cs`

⬜ **1.1** **Project:** `Assets/Scripts/ReticleScaler.cs`

```csharp
using UnityEngine;

/// <summary>
/// Keeps the reticle ring a constant *apparent* size and facing the HMD.
/// Attach to the Reticle prefab (ReticleRing).
/// </summary>
public class ReticleScaler : MonoBehaviour
{
    [Header("Assign CenterEyeAnchor (PlayerRig/TrackingSpace)")]
    public Transform cameraEye;
    [Header("Visual size (diameter) at 1 meter")]
    public float sizeAt1Meter = 0.05f;

    void LateUpdate()
    {
        if (!cameraEye) return;

        // Face the eye
        transform.rotation = Quaternion.LookRotation(transform.position - cameraEye.position);

        // Scale so it appears constant size with distance
        float d = Vector3.Distance(transform.position, cameraEye.position);
        float s = Mathf.Max(0.001f, sizeAt1Meter * d);
        transform.localScale = Vector3.one * s;
    }
}
```

### 1B) Put it on the reticle prefab (handle the scene reference)

⬜ **1.2** **Project → Prefabs:** double-click **`ReticleRing`** to open it.
⬜ **1.3** **Inspector:** **Add Component → ReticleScaler**.
⬜ **1.4** Assign **Camera Eye** = drag **`PlayerRig/TrackingSpace/CenterEyeAnchor`** from the **scene**.

> If Unity won’t allow scene refs in prefab mode:
>
> * Temporarily **instantiate** `ReticleRing` into the scene, add **ReticleScaler** and assign **CenterEyeAnchor**, then **Apply** to prefab and delete the scene copy.

✅ Result: reticle always faces the HMD and keeps a steady apparent size.

---

## 2) Stronger hover/press feedback on cells <span style="color:purple;">🟣</span>

### 2A) Button color tint (snappy)

⬜ **2.1** **Hierarchy:** select **all 9** `Wall/BoardCanvas/Grid/Cell_0..Cell_8`
⬜ **2.2** **Inspector (Button → Colors):**

* **Normal:** #FFFFFF
* **Highlighted:** #F3F3F3
* **Pressed:** #E0E0E0
* **Selected:** #F3F3F3
* **Disabled:** #B0B0B0
* **Fade Duration:** **0.06**

### 2B) Subtle scale on hover (script)

⬜ **2.3** **Project:** `Assets/Scripts/CellHoverScale.cs`

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

/// <summary>
/// Tiny scale-up on hover via pointer enter/exit (works with Pointable Canvas Module).
/// Attach to each Cell (same GO as the Button).
/// </summary>
public class CellHoverScale : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler
{
    public float hoverScale = 1.05f;
    public float speed = 16f;
    private Vector3 _base;

    void Awake() => _base = transform.localScale;

    public void OnPointerEnter(PointerEventData e)
    {
        StopAllCoroutines();
        StartCoroutine(LerpTo(_base * hoverScale));
    }

    public void OnPointerExit(PointerEventData e)
    {
        StopAllCoroutines();
        StartCoroutine(LerpTo(_base));
    }

    System.Collections.IEnumerator LerpTo(Vector3 target)
    {
        while ((transform.localScale - target).sqrMagnitude > 1e-6f)
        {
            transform.localScale = Vector3.Lerp(transform.localScale, target, Time.deltaTime * speed);
            yield return null;
        }
        transform.localScale = target;
    }
}
```

⬜ **2.4** **Hierarchy:** select **all 9 Cells** → **Add Component → CellHoverScale** (defaults are fine).

✅ Result: cell nudges up 5% on hover; drops back on exit.

---

## 3) Haptics (click & win) — tiny, safe helper <span style="color:purple;">🟣</span>

> We’ll use a small helper inside **GameController** and call it from **Space**. This uses `OVRInput.SetControllerVibration` (simple and fine on Quest).

### 3A) Add the haptics helper inside `GameController.cs`

⬜ **3.1** Open **`Assets/Scripts/TicTacToe/GameController.cs`** and **add** inside the class:

```csharp
// ===== Haptics (OVRInput) =====
private System.Collections.IEnumerator Pulse(float amplitude, float duration)
{
    // amplitude [0..1]; Quest ignores frequency param.
    OVRInput.SetControllerVibration(1f, amplitude, OVRInput.Controller.LTouch);
    OVRInput.SetControllerVibration(1f, amplitude, OVRInput.Controller.RTouch);
    yield return new WaitForSeconds(duration);
    OVRInput.SetControllerVibration(0f, 0f, OVRInput.Controller.LTouch);
    OVRInput.SetControllerVibration(0f, 0f, OVRInput.Controller.RTouch);
}

/// <summary>Called when any cell is successfully clicked.</summary>
public void OnCellClicked()
{
    StartCoroutine(Pulse(0.15f, 0.06f)); // tiny tick
}
```

### 3B) Call it from `Space.cs`

⬜ **3.2** Open **`Assets/Scripts/TicTacToe/Space.cs`** and **add** one line in `SetSpace()` **after** you set the text:

```csharp
buttonText.text = gameController.GetSide();
gameController.OnCellClicked(); // haptic tick on place
```

(Leave the rest of `SetSpace()` as is: disable button → `gameController.EndTurn()`.)

✅ Result: each valid move gives a short vibration.

---

## 4) Audio polish: click & win <span style="color:purple;">🟣</span>

### 4A) Import two clips

⬜ **4.1** **Project:** make `Assets/Audio`
⬜ **4.2** Add two short clips (e.g., **`click.wav`**, **`win.wav`**)
⬜ **4.3** For each **AudioClip → Inspector**: **Load Type = Decompress On Load**, compression low/off.

### 4B) Add an AudioSource to the canvas

⬜ **4.4** **Hierarchy:** select **`Wall/BoardCanvas`** → **Add Component → Audio Source**

* **Spatial Blend = 0.0 (2D)**
* **Play On Awake = OFF**, **Loop = OFF**

### 4C) Add fields + play helpers in `GameController.cs`

⬜ **4.5** Open **`GameController.cs`** and **add** near your other `[Header]` fields:

```csharp
[Header("Audio (on BoardCanvas)")]
public AudioSource audioSource; // drag BoardCanvas AudioSource
public AudioClip clickClip;     // Assets/Audio/click.wav
public AudioClip winClip;       // Assets/Audio/win.wav
```

⬜ **4.6** Add two helpers **inside** the class:

```csharp
private void PlayClick()
{
    if (audioSource && clickClip) audioSource.PlayOneShot(clickClip, 0.8f);
}
private void PlayWin()
{
    if (audioSource && winClip) audioSource.PlayOneShot(winClip, 1.0f);
}
```

⬜ **4.7** Hook calls:

* In **`OnCellClicked()`**, add:

```csharp
PlayClick();
```

* In **`GameOver(string message)`**, **after** setting the text, add:

```csharp
PlayWin();
StartCoroutine(Pulse(0.4f, 0.12f)); // stronger win pulse
```

### 4D) Wire references in the Inspector

⬜ **4.8** **Hierarchy:** select **`SceneManager`** → **GameController**

* **Audio Source** = drag **`Wall/BoardCanvas`** (its AudioSource)
* **Click Clip** = drag `Assets/Audio/click.wav`
* **Win Clip** = drag `Assets/Audio/win.wav`

✅ Result: click sound each move; win sound + stronger haptics on victory.

---

## 5) Playtest (what you should feel/see/hear) <span style="color:purple;">🟣</span>

1. **Enter Play**. Hover a cell → it **brightens** and **scales** slightly.
2. Click a cell → **short haptic** + **click sound**; X/O appears; cell disables.
3. Force a win → **overlay appears**, **win sound**, **stronger vibration**.
4. **Restart** still clears the board.

---

## 6) Troubleshooting (exact places to check) <span style="color:purple;">🟣</span>

* **No hover scale:**
  – `CellHoverScale` must be on the **same GameObject** as the **Button**.
  – EventSystem must be the **Pointable Canvas Module** one (from the wizard).
* **No haptics:**
  – Ensure **OnCellClicked()** is called (you added it in `Space.SetSpace()`).
  – You’re in-headset (Link/Air Link focused).
* **No audio:**
  – **AudioSource** on **BoardCanvas**; fields assigned on **GameController**; headset/system volume up.
* **Reticle not scaling:**
  – `ReticleScaler` is on the **ReticleRing** prefab and **Camera Eye** is **CenterEyeAnchor**.

---

## ✅ End of Part 4 — submit

* Screenshot: **ReticleRing prefab** with **ReticleScaler** (Camera Eye assigned).
* Screenshot: one **Cell** with **CellHoverScale** and Button Colors.
* Screenshot: **GameController** with **AudioSource/Clips** wired.
* Short clip: hover (scale), click (sound + haptic), win (sound + stronger haptic), restart.

---
---

# 🔹 Part 5 — Handedness (Left = 3D only, Right = UI only)

<span style="color:purple;font-weight:bold;">🟣 </span>

**Goal:**

* **Right hand** can **hover/click** the **BoardCanvas** UI (Tic-Tac-Toe cells).
* **Left hand** can **grab / ray-grab** **3D** objects (e.g., your `GrabCube`) but **cannot** click UI.
* Done **entirely** with **Interaction SDK** components (rays, filters, layers), not EventSystem hacks.

**You will touch:** **BoardCanvas** (layer), **Left/Right Controller Ray Interactors** (masks + filters), and **3D hosts** (Tag Set).

---

## 0) Pre-flight (from Parts 1–4) <span style="color:purple;">🟣</span>

⬜ **0.1** You have **`PlayerRig` (OVRCameraRig)** with **Controller Ray Interactor** + **Grab Interactor** on **both** hands.
⬜ **0.2** `Wall/BoardCanvas` is **World Space** and you previously ran **Interaction SDK → Add Ray Interaction to Canvas** (so the **EventSystem** uses the **Pointable Canvas Module** and ray clicks already work).
⬜ **0.3** Game plays fine from Part 3; polish from Part 4 is optional but nice.

---

## 1) Layers for clarity (UI vs 3D) <span style="color:purple;">🟣</span>

> We’ll separate UI and 3D via layers so each hand’s ray only “cares” about its targets.

⬜ **1.1** **Create / confirm a UI layer**

* **Inspector (top-right)** → **Layers → Edit Layers…**
* In **User Layers**, add a new layer: **`UIWorld`** (if you haven’t already).

⬜ **1.2** **Assign the canvas to UIWorld**

* **Hierarchy:** select **`Wall/BoardCanvas`** → **Layer = UIWorld**
  *(Unity prompts to apply to children → choose **Yes** so all child UI uses UIWorld.)*

> 3D objects (like `GrabCube` or your hoop/backboard) should stay on **Default** or other 3D layers (not UIWorld).

---

## 2) RIGHT hand = **UI only** <span style="color:purple;">🟣</span>

> We’ll let the Right ray be visible even when no 3D is around, but it won’t select 3D at all.

⬜ **2.1** **Hierarchy path:**
`PlayerRig/TrackingSpace/RightHandAnchor/ControllerInteractors/Controller Ray Interactor`

⬜ **2.2** **Inspector (Right Controller Ray Interactor):**

* **Hide When No Interactable = OFF** *(keep beam visible for UI hovering)*
* **Collision / Interaction Mask:** **only `UIWorld`**

  * Deselect **Default**, **Grabbable**, or any 3D layers.
  * Leave **UIWorld** checked (or add it if using a LayerMask list).

> ✅ Right-hand ray now ignores all 3D hosts and only “sees” the UI layer.
> The **Pointable Canvas Module** will route UI pointer events from whichever ray is hovering. Because Right is the only one that sees `UIWorld`, **only Right can click UI**.

---

## 3) LEFT hand = **3D only** <span style="color:purple;">🟣</span>

⬜ **3.1** **Hierarchy path:**
`PlayerRig/TrackingSpace/LeftHandAnchor/ControllerInteractors/Controller Ray Interactor`

⬜ **3.2** **Inspector (Left Controller Ray Interactor):**

* **Hide When No Interactable = ON** *(optional; hides beam if nothing 3D is in reach)*
* **Collision / Interaction Mask:** **include 3D layers** (e.g., **Default**, **Grabbable**) and **exclude `UIWorld`**.

> ✅ Left-hand ray won’t interact with UI because it can’t see `UIWorld`. It *will* hit your 3D interactables.

---

## 4) Tag-based filter (belt-and-suspenders, great for juniors) <span style="color:purple;">🟣</span>

> We’ll add a **Tag Set** on every **3D interactable host**, then require that tag on the **Left** ray. This is optional but reinforces “what is 3D” for students.

### 4A) Tag the 3D host

⬜ **4.1** **Pick your 3D host**:

* **Wizard path:** the host is the **parent** the wizard created (e.g., `ISDK Ray Grab Interaction`) that holds **RayInteractable/Grabbable** components.
* **Manual path:** the host is the **same GameObject** that has **RayInteractable + Grabbable + ColliderSurface** (e.g., `GrabCube`).

⬜ **4.2** **Inspector (host):** **Add Component → Tag Set**

* **Tag =** `3DObject` *(type the exact string)*

### 4B) Require that tag on the Left interactor

⬜ **4.3** **Hierarchy:** select **Left Controller Ray Interactor** again

* **Add Component → Tag Set Filter**

  * **Required Tag =** `3DObject`
  * **Exclude Tag =** *(leave empty)*
* In **Controller Ray Interactor → Interactable Filters (list)** → **+** → **drag the Tag Set Filter** you just added into this slot.

> ✅ Left ray now **only** selects interactables whose host is tagged `3DObject`.
> Combined with the Collision Mask (no `UIWorld`), Left can never click UI.

---

## 5) Playtest checklist <span style="color:purple;">🟣</span>

1. **Enter Play.**
2. **Right hand**: aim at **BoardCanvas** → cells **highlight** and **click**. Try to grab `GrabCube` → **no interaction** (good).
3. **Left hand**: aim at **BoardCanvas** → **no hover / no click** (ray should ignore UI). Aim at `GrabCube` → **ray-grab** (or near-grab) works.

---

## 6) Troubleshooting (exact spots) <span style="color:purple;">🟣</span>

* **Right hand still grabs 3D:**
  – **Right Controller Ray Interactor → Collision/Interaction Mask** must be **UIWorld only**.
  – If you also added a Tag Set Filter on Right by mistake, **remove it**.

* **Left hand still hovers UI:**
  – **Left Controller Ray Interactor → Collision/Interaction Mask** must **exclude UIWorld**.
  – If UI still reacts, verify **BoardCanvas (and children)** are on **UIWorld** (Layer). If they’re on **Default**, the Left ray will see them.

* **Left can’t grab anything anymore:**
  – Did you add **Tag Set Filter** on Left? Then **every 3D host** you want to grab must have **Tag Set = `3DObject`** on the **host** (the object with `RayInteractable`, not just the mesh child).
  – Or temporarily **remove** the Tag Set Filter to confirm masks are fine.

* **UI doesn’t click at all now:**
  – Make sure you previously ran **Interaction SDK → Add Ray Interaction to Canvas** on **`BoardCanvas`** so the **EventSystem** has the **Pointable Canvas Module**.
  – Confirm **Right ray** can “see” **UIWorld** (its mask includes UIWorld).

* **Right ray invisible:**
  – Set **Hide When No Interactable = OFF** on the **Right Controller Ray Interactor** so you still see a beam for UI hovering.

---

## 7) Deliverables (submit)

* Screenshot: **BoardCanvas** set to **Layer = UIWorld**.
* Screenshot: **Right Controller Ray Interactor** showing **Hide When No Interactable = OFF** and **Collision/Interaction Mask = UIWorld only**.
* Screenshot: **Left Controller Ray Interactor** with **UIWorld unchecked** and (if you used it) **Interactable Filters** containing a **Tag Set Filter (Required = `3DObject`)**.
* Short clip: **Right** clicks UI but can’t grab; **Left** grabs 3D but can’t click UI.

---
Awesome — here’s your **modern** **Part 6/6**. It adds a clean **HUD (Turn / Score / Round)**, **Undo**, **Reset All**, and a simple **Debug overlay**, all compatible with your **Interaction SDK** setup (Pointable Canvas Module, no `OVRPlayerController`, no object named “XR”). Tiny code slices → where to paste → what to drag in the Inspector → what to test. All purple blocks are homework.

---

# 🔹 Part 6(modern) — HUD, Scores/Rounds, Undo, Reset, Debug Overlay

<span style="color:purple;font-weight:bold;">🟣 </span>

**Goal:**

* Show **whose turn** it is.
* Keep **scores** across **rounds** (X and O).
* **Undo** the last move.
* **Reset All** (scores + round + board).
* Toggle a **Debug overlay** to help juniors self-diagnose.

**You’ll use:** `Assets/Scripts/TicTacToe` (existing scripts), **BoardCanvas** (HUD + buttons), a tiny **DebugOverlay** script.

---

## 0) Pre-flight (from Parts 1–5) <span style="color:purple;">🟣</span>

⬜ **0.1** `Wall/BoardCanvas` exists with `Grid/Cell_0..Cell_8` and a disabled `GameOverPanel` (with `GameOverText` + `RestartButton`).
⬜ **0.2** `SceneManager` has **GameController** wired with `spaceList[0..8]` + GameOver refs.
⬜ **0.3** UI ray clicks work via **Pointable Canvas Module** (you ran “**Add Ray Interaction to Canvas**” on `BoardCanvas`).

---

## 1) Build the HUD (Turn / Score / Round) on the canvas <span style="color:purple;">🟣</span>

⬜ **1.1** **Hierarchy:** Right-click **`Wall/BoardCanvas`** → **UI → Panel** → rename **`HUDPanel`**

* **RectTransform:** **Anchor = Top Stretch** (Alt+click top-stretch preset)
* **Left = 8, Right = 8, Top = 8, Height = 0.12** (meters)
* **Image Color:** black (#000000) with **Alpha ≈ 140/255**

⬜ **1.2** **Turn text**

* **HUDPanel** → **UI → Text (TextMeshPro)** → rename **`TurnText`**
* **RectTransform:** **Left = 12, Width = 0.28, PosY = −0.02**
* **TMP:** **Text = “Turn: X”**, **Font Size = 56**, **Align = Left**, **Color = White**

⬜ **1.3** **Score X**

* **HUDPanel** → **Text (TMP)** → rename **`ScoreXText`**
* **RectTransform:** **Center anchor**, **Width = 0.24, PosX = 0.00, PosY = −0.02**
* **TMP:** **Text = “X: 0”**, **Font Size = 56**, **Center**, **Color = #FFD369**

⬜ **1.4** **Score O**

* Duplicate **ScoreXText** → rename **`ScoreOText`**
* **RectTransform:** **PosX = 0.22**
* **TMP:** **Text = “O: 0”**, **Color = #69C0FF**

⬜ **1.5** **Round**

* **HUDPanel** → **Text (TMP)** → rename **`RoundText`**
* **RectTransform:** **Right anchor**, **Right = 12, Width = 0.22, PosY = −0.02**
* **TMP:** **Text = “Round: 1”**, **Font Size = 56**, **Right align**, **White**

---

## 2) Add HUD action buttons (Undo & Reset All) <span style="color:purple;">🟣</span>

⬜ **2.1** **Undo**

* **HUDPanel** → **UI → Button (TextMeshPro)** → rename **`UndoButton`**
* **RectTransform:** **Left anchor**, **Left = 0.34, Width = 0.14, Height = 0.08, PosY = −0.02**
* **Child TMP:** **Text = “Undo”**, **Font Size = 44**
* **Button Colors:** quick **Color Tint**, **Fade Duration = 0.06**

⬜ **2.2** **Reset All**

* Duplicate **UndoButton** → rename **`ResetAllButton`**
* **RectTransform:** **Left = 0.50**
* **Child TMP:** **Text = “Reset”**

⬜ **2.3** **Rename (visual)** the existing **`RestartButton`** under `GameOverPanel` to **“New Round”** (text only).
*(We’ll wire “New Round” via code in a minute.)*

---

## 3) Update the Cell script so the controller knows *which cell* was played <span style="color:purple;">🟣</span>

> We’ll give each **Space** its index (0..8) and register the move for **Undo**.

⬜ **3.1** Open **`Assets/Scripts/TicTacToe/Space.cs`** and add the index & setter **inside the class**:

```csharp
private int _index;             // which cell am I (0..8)?
public void SetIndex(int i) { _index = i; }
```

⬜ **3.2** In `SetSpace()` **after** `buttonText.text = gameController.GetSide();` add:

```csharp
gameController.RegisterMove(_index); // remember last move for Undo
```

*(Keep the rest of `SetSpace()` the same: disable Button → `gameController.EndTurn();`)*

---

## 4) Extend `GameController` for HUD / Undo / Rounds / Reset <span style="color:purple;">🟣</span>

⬜ **4.1** Open **`Assets/Scripts/TicTacToe/GameController.cs`** and add these fields near your other `[Header]` blocks:

```csharp
[Header("HUD (on BoardCanvas/HUDPanel)")]
public TMP_Text turnText;     // HUDPanel/TurnText
public TMP_Text scoreXText;   // HUDPanel/ScoreXText
public TMP_Text scoreOText;   // HUDPanel/ScoreOText
public TMP_Text roundText;    // HUDPanel/RoundText

[Header("Scores / Round")]
public int scoreX = 0;
public int scoreO = 0;
public int round = 1;

// Undo support
private System.Collections.Generic.List<int> moveHistory =
    new System.Collections.Generic.List<int>();

private bool isGameOver = false;
```

⬜ **4.2** Edit your existing method `SetGameControllerReferenceForButtons()` to also set each cell index:

```csharp
private void SetGameControllerReferenceForButtons()
{
    for (int i = 0; i < spaceList.Length; i++)
    {
        if (!spaceList[i]) continue;
        var space = spaceList[i].GetComponentInParent<Space>();
        if (space != null)
        {
            space.SetControllerReference(this);
            space.SetIndex(i); // tell each Space its index (0..8)
        }
        else
        {
            Debug.LogWarning($"[GameController] No Space script for index {i}.");
        }
    }
}
```

⬜ **4.3** Add small helpers **inside the class**:

```csharp
private void UpdateHUD()
{
    if (turnText)   turnText.text   = $"Turn: {side}";
    if (scoreXText) scoreXText.text = $"X: {scoreX}";
    if (scoreOText) scoreOText.text = $"O: {scoreO}";
    if (roundText)  roundText.text  = $"Round: {round}";
}

private void ClearHistory() => moveHistory.Clear();

public void RegisterMove(int index)
{
    if (!moveHistory.Contains(index))
        moveHistory.Add(index);
}
```

⬜ **4.4** Replace your `Start()` with this version (adds HUD + state init):

```csharp
private void Start()
{
    SetGameControllerReferenceForButtons();

    side = "X";
    moves = 0;
    isGameOver = false;
    ClearHistory();

    if (gameOverPanel) gameOverPanel.SetActive(false);
    if (restartButton) restartButton.gameObject.SetActive(false);

    UpdateHUD();
}
```

⬜ **4.5** Replace your `GameOver(string message)` to award points + show “New Round”:

```csharp
private void GameOver(string message)
{
    isGameOver = true;

    // Award the winner (ties do not score)
    if (message.Contains("wins"))
    {
        if (side == "X") scoreX++;
        else if (side == "O") scoreO++;
    }

    if (gameOverPanel) gameOverPanel.SetActive(true);
    if (gameOverText)  gameOverText.text = message;

    SetInteractable(false);
    UpdateHUD();

    if (restartButton)
    {
        restartButton.gameObject.SetActive(true);
        restartButton.onClick.RemoveAllListeners();
        restartButton.onClick.AddListener(NewRound);
    }

    // (If you added polish in Part 4:)
    PlayWin();
    StartCoroutine(Pulse(0.4f, 0.12f));
}
```

⬜ **4.6** Add **NewRound / ResetAll / UndoLastMove**:

```csharp
public void NewRound()
{
    round++;
    side = "X";
    moves = 0;
    isGameOver = false;
    ClearHistory();

    for (int i = 0; i < spaceList.Length; i++)
    {
        if (spaceList[i]) spaceList[i].text = string.Empty;
        var btn = spaceList[i] ? spaceList[i].GetComponentInParent<Button>() : null;
        if (btn) btn.interactable = true;
    }

    if (gameOverPanel) gameOverPanel.SetActive(false);
    if (restartButton) restartButton.gameObject.SetActive(false);

    UpdateHUD();
}

public void ResetAll()
{
    scoreX = 0;
    scoreO = 0;
    round  = 1;
    NewRound(); // clears board and resets state
}

public void UndoLastMove()
{
    if (isGameOver || moveHistory.Count == 0) return;

    int lastIndex = moveHistory[moveHistory.Count - 1];
    moveHistory.RemoveAt(moveHistory.Count - 1);

    // Clear that cell & re-enable its Button
    if (lastIndex >= 0 && lastIndex < spaceList.Length)
    {
        if (spaceList[lastIndex]) spaceList[lastIndex].text = string.Empty;
        var btn = spaceList[lastIndex] ? spaceList[lastIndex].GetComponentInParent<Button>() : null;
        if (btn) btn.interactable = true;
    }

    // Step back move count and turn
    if (moves > 0) moves--;
    side = (side == "X") ? "O" : "X";

    UpdateHUD();

    // gentle confirmation tick (if Part 4 haptics exist)
    StartCoroutine(Pulse(0.1f, 0.05f));
}
```

> No change needed in `EndTurn()`; it already checks win/tie and swaps turns.

---

## 5) Wire HUD fields and button clicks (Inspector) <span style="color:purple;">🟣</span>

⬜ **5.1** **SceneManager → GameController (component):**

* **Turn Text =** `Wall/BoardCanvas/HUDPanel/TurnText`
* **Score X Text =** `.../ScoreXText`
* **Score O Text =** `.../ScoreOText`
* **Round Text =** `.../RoundText`

⬜ **5.2** **Undo button**

* Select **`HUDPanel/UndoButton`** → **Button (OnClick)** → **+**
* Drag **`SceneManager`** → pick **`GameController.UndoLastMove()`**

⬜ **5.3** **Reset All button**

* Select **`HUDPanel/ResetAllButton`** → **Button (OnClick)** → **+**
* Drag **`SceneManager`** → pick **`GameController.ResetAll()`**

*(“New Round” is auto-wired in code when GameOver shows.)*

---

## 6) Add a Debug overlay (toggle on/off) <span style="color:purple;">🟣</span>

⬜ **6.1** **BoardCanvas** → **UI → Panel** → rename **`DebugPanel`**

* **RectTransform:** **Bottom Stretch**, **Left = 8, Right = 8, Bottom = 8, Height = 0.16**
* **Image:** black with **Alpha ≈ 120/255**
* **Active = OFF** (start hidden)

⬜ **6.2** **Debug text**

* **DebugPanel** → **UI → Text (TextMeshPro)** → rename **`DebugText`**
* **TMP:** **Font Size = 40**, **Top-Left**, **Text = “(debug…)”**

⬜ **6.3** **HUD toggle button**

* **HUDPanel** → **UI → Button (TextMeshPro)** → rename **`DebugToggle`**
* **RectTransform:** **Right anchor**, **Right = 0.10, Width = 0.12, Height = 0.08, PosY = −0.02**
* **Child TMP:** **Text = “Debug”**, **Font Size = 40**

⬜ **6.4** **Project:** `Assets/Scripts/DebugOverlay.cs`

```csharp
using UnityEngine;
using TMPro;

public class DebugOverlay : MonoBehaviour
{
    public GameController controller; // drag SceneManager
    public GameObject panel;          // drag BoardCanvas/DebugPanel
    public TMP_Text debugText;        // drag BoardCanvas/DebugPanel/DebugText

    void Update()
    {
        if (!controller || !debugText) return;
        debugText.text =
            $"Turn: {controller.GetSide()}\n" +
            $"Scores  X:{controller.scoreX}  O:{controller.scoreO}\n" +
            $"Round: {controller.round}\n";
    }

    public void Toggle()
    {
        if (!panel) return;
        panel.SetActive(!panel.activeSelf);
    }
}
```

⬜ **6.5** **Hierarchy:** select **`Wall/BoardCanvas`** → **Add Component → DebugOverlay**

* **Controller =** drag **`SceneManager`**
* **Panel =** drag **`Wall/BoardCanvas/DebugPanel`**
* **Debug Text =** drag **`Wall/BoardCanvas/DebugPanel/DebugText`**

⬜ **6.6** **Wire the toggle**

* Select **`HUDPanel/DebugToggle`** → **Button (OnClick)** → **+**
* Drag **`Wall/BoardCanvas`** → choose **`DebugOverlay.Toggle()`**

---

## 7) Playtest Checklist <span style="color:purple;">🟣</span>

1. **HUD** shows “**Turn: X**” at start.
2. Click alternates **X/O**; **Turn** updates accordingly.
3. Win shows **GameOverPanel**, score **X** or **O** increments, **New Round** appears.
4. Press **New Round** → board clears, **Round** increments, **Turn: X**.
5. **Undo** after a click → last mark disappears, that cell re-enables, **Turn** swaps back.
6. **Reset** → scores to **0–0**, **Round = 1**, fresh board, **Turn: X**.
7. **Debug** button toggles the debug panel text.

---

## 8) Troubleshooting (exact spots) <span style="color:purple;">🟣</span>

* **Undo does nothing:**
  – Ensure `Space.SetSpace()` calls `RegisterMove(_index)` **before** `EndTurn()`.
  – Confirm `SetIndex(i)` is called from controller wiring (`SetGameControllerReferenceForButtons()`).

* **HUD not updating:**
  – `UpdateHUD()` is called in `Start()`, `NewRound()`, `UndoLastMove()`, and inside `GameOver()`; verify these edits.

* **Score not increasing on win:**
  – `GameOver(message)` checks `message.Contains("wins")`. If you changed the win text, keep the word **“wins”** or increment scores manually.

* **Buttons don’t fire:**
  – **UndoButton → GameController.UndoLastMove()**, **ResetAllButton → GameController.ResetAll()** must be in **OnClick**.

* **Debug panel empty:**
  – `Wall/BoardCanvas` must have **DebugOverlay**; its three fields assigned; **DebugToggle** wired to `Toggle()`.

---

## ✅ End-of-course deliverables

* Short video: **play → win → New Round → Undo → Reset → Debug toggle**, plus handedness from Part 5 (Right clicks UI; Left grabs 3D).
* Screenshots: **GameController** with HUD fields wired; **HUDPanel** hierarchy; **DebugOverlay** fields assigned.

---

