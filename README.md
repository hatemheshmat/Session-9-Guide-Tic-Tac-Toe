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

# Part 2

## Player Rig, Rays, Wall, and World-Space UI (Unity 6.2 • URP • Meta XR AIO • OpenXR)

### Student Guide — session-ready, step-by-step

**Goal of Part 2:** put a VR rig in the scene, add controller rays (plus grab), build a wall with a world-space Canvas, and make the UI ray-clickable using the Meta Interaction SDK canvas pipeline.
*(We’ll add Tic-Tac-Toe logic and cell scripts in later parts.)*

---

## Learning outcomes (today)

* Create a **PlayerRig** with **OVRCameraRig**, **CharacterController**, and **SimpleRigLocomotion**.
* Add **Controller Ray Interactor** and **Grab Interactor** to both hands.
* Build a **Wall** and attach a **BoardCanvas** (world-space UI).
* Enable VR UI interactions using **Pointable Canvas Module** + **Pointable Canvas**.
* Leave the scene clean and organized for the next parts.

---

## Flow map (which window you’ll use at each step)

* **Project**: find prefabs/scripts, create folders and prefabs.
* **Hierarchy**: create and parent objects to form the rig and board.
* **Inspector**: add components and set property values.
* **Scene**: position/scale objects.
* **Game**: quick play tests with Link/Air Link.

---

## Prereqs (from Part 1)

* Project on **Android**, **ASTC**, **IL2CPP/ARM64**, **Vulkan**.
* **OpenXR** enabled with **Meta Quest Support**.
* **Meta XR Project Setup / Validation** already “Fix All.”
* **URP** baseline set (HDR off, MSAA 4×, minimal PostFX).
* Scene saved as **01_VR_TicTacToe.unity** with **Ground** and **Directional Light** only.

---

## Step 1 — PlayerRig (Rig + CharacterController + Locomotion)

### A) Create the PlayerRig root

1. **Hierarchy** → **Create Empty** → rename **PlayerRig**.
2. **Inspector (PlayerRig)** → **Add Component → CharacterController**

   * **Height:** `1.7`
   * **Radius:** `0.3`
   * **Center:** `(0, 0.9, 0)`
   * **Slope Limit:** `45`
   * **Step Offset:** `0.3`
3. **Inspector (PlayerRig)** → **Add Component → SimpleRigLocomotion** (we’ll create it now).

### B) SimpleRigLocomotion script (thumbstick move in place)

1. **Project** → `Assets/Scripts` → **Create > C# Script** → **SimpleRigLocomotion.cs**
2. Paste:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

[RequireComponent(typeof(CharacterController))]
public class SimpleRigLocomotion : MonoBehaviour
{
    public float moveSpeed = 2.0f;
    public float gravity = -9.81f;

    [Header("Camera Root (OVRCameraRig or XR camera parent)")]
    public Transform head; // assign to the head/camera transform under the rig
    private CharacterController _cc;
    private float _yVel;

    void Awake()
    {
        _cc = GetComponent<CharacterController>();
    }

    void Update()
    {
        if (head == null) return;

        // Input System: left stick on any XR controller
        Vector2 move = Vector2.zero;
        var gamepad = Gamepad.current;
        if (gamepad != null) move = gamepad.leftStick.ReadValue();

        // XR Controller bindings (if you use XRI actions, map them here)
        // For a simple lab, Gamepad.leftStick via Link works fine.

        // Move relative to head's horizontal forward
        Vector3 fwd = new Vector3(head.forward.x, 0, head.forward.z).normalized;
        Vector3 right = new Vector3(head.right.x, 0, head.right.z).normalized;
        Vector3 planar = (fwd * move.y + right * move.x) * moveSpeed;

        // Gravity
        if (_cc.isGrounded) _yVel = -0.5f;
        else _yVel += gravity * Time.deltaTime;

        Vector3 motion = planar + Vector3.up * _yVel;
        _cc.Move(motion * Time.deltaTime);
    }
}
```

*(This gives dependable “flat world” locomotion for labs. In advanced parts you can switch to action-based movement.)*

### C) Add the OVRCameraRig under PlayerRig

1. **Project (Search):** `OVRCameraRig`
2. **Drag** into **Hierarchy** as a **child of PlayerRig**.
3. **Set PlayerRig position** to `(0, 0, 0)` so it starts on the **Ground**.
4. In **OVRCameraRig**, find the head/camera transform (usually under **TrackingSpace**).

   * **Select PlayerRig → SimpleRigLocomotion → head**: **drag the head/camera transform here**.

---

## Step 2 — Left/Right controller interactors (Ray + Grab)

### A) Create interactor containers per hand

For both **LeftHandAnchor** and **RightHandAnchor**:

1. **Hierarchy Path:**
   `PlayerRig / OVRCameraRig / TrackingSpace / LeftHandAnchor`
   → **Create Empty** → rename **ControllerInteractors**.
   Repeat under **RightHandAnchor**.

### B) Add interactors to each ControllerInteractors

Under **LeftHandAnchor/ControllerInteractors**:

1. **Add Component → Controller Ray Interactor** (Meta Interaction SDK).

   * **Max Ray Length:** `6`
   * **Hide When No Interactable:** `On`
2. **Add Component → Grab Interactor** (Meta Interaction SDK).

Repeat the same two components under **RightHandAnchor/ControllerInteractors**.

### C) Optional visual line

If available as a separate component:

* **Add Component → Line Visual** to each **Controller Ray Interactor**.

  * **Width:** `0.005` to `0.01`
  * **Max Distance:** `6`

*(Depending on SDK version, line visuals may be embedded or provided via a child. Use the default visuals in your SDK if present.)*

---

## Step 3 — Wall and the BoardCanvas (world-space UI)

### A) Wall

1. **Hierarchy** → **3D Object → Cube** → rename **Wall**

   * **Position:** `(0, 1.5, 2.5)`
   * **Scale:** `(3, 2, 0.1)`
     *(A large, flat surface to mount the board.)*

### B) BoardCanvas (world-space)

1. **Hierarchy (right-click Wall)** → **UI → Canvas** → rename **BoardCanvas**
2. **Inspector (BoardCanvas → Canvas):**

   * **Render Mode:** `World Space`
   * **RectTransform Size:** `800 × 800`
   * **Position:** local `(0, 0, 0.06)` *(a few cm in front of the wall)*
   * **Scale:** `(0.001, 0.001, 0.001)` *(crisp UI in VR)*

### C) EventSystem + Meta canvas pipeline

1. If there’s no **EventSystem** in the scene: **UI → EventSystem** (auto-created).
2. **Select EventSystem** → **Add Component → Pointable Canvas Module**.
   *(This routes controller rays to UI; keep only one UI module active.)*
3. **Select BoardCanvas** → **Add Component → Pointable Canvas**.

   * Ensure **Graphic Raycaster** is present (Unity adds it by default to UGUI canvases).

### D) Board background

1. **Hierarchy (right-click BoardCanvas)** → **UI → Panel** → rename **BoardPanel**

   * **RectTransform:** stretch to `0,0,0,0` (full)
   * **Image Color:** low-alpha dark or light tint as a background.

---

## Step 4 — Grid container and Cell prefab (UI only for now)

### A) Grid container

1. **Hierarchy (right-click BoardCanvas)** → **Create UI → Panel** → rename **Grid**
2. **Inspector (Grid → RectTransform):**

   * **Anchor/Pivot:** center
   * **Size:** `700 × 700` *(square)*
3. **Inspector (Grid)** → **Add Component → Grid Layout Group**

   * **Cell Size:** `220 × 220`
   * **Spacing:** `20 × 20`
   * **Constraint:** `Fixed Column Count = 3`

### B) Create the Cell prefab

1. **Hierarchy (right-click Grid)** → **UI → Button (TextMeshPro)** → rename **Cell_0**
2. **Inspector (Cell_0 Button):**

   * **Transition:** Color Tint (keep defaults; we’ll polish later)
3. **Child Text (TMP):** clear the text (empty) — we’ll use sprites or TMP later.
4. **Project:** drag **Cell_0** from Hierarchy to `Assets/Prefabs` to make a **prefab** named **Cell**.
5. Back in **Hierarchy**, delete **Cell_0**, then **drag the Cell prefab** into **Grid** nine times, renaming:

   * **Cell_0 … Cell_8**
     *(The Grid Layout Group will auto-tile them into 3×3.)*

### C) GameOver UI (hidden for now)

1. **Hierarchy (right-click BoardCanvas)** → **UI → Panel** → rename **GameOverPanel**

   * **Set Active:** `Off` (disabled)
   * Place centered over the grid; size ~ `700 × 200`
2. **Right-click GameOverPanel** → **UI → Text (TextMeshPro)** → rename **GameOverText**

   * **Alignment:** `Center`
   * **Font Size:** `72`
   * **Text:** *(leave empty for now)*
3. **Right-click GameOverPanel** → **UI → Button (TextMeshPro)** → rename **RestartButton**

   * **Text:** `Play again?`
   * **Font Size:** `48`

*(We’ll wire these in Part 3 when we add the board logic.)*

---

## Step 5 — Quick smoke test (editor with Link/Air Link)

* **Enter Play**: you should see the wall ahead.
* Point either controller toward the **BoardCanvas**: the **ray** should land on cells and show button hover states.
* Pull trigger: the button should press visually (no gameplay yet).
* Move with the thumbstick (SimpleRigLocomotion) to confirm the CharacterController is responding.

---

## Notes & Hints (for Students)

* If you don’t see hover/press on the buttons, **EventSystem** probably doesn’t have **Pointable Canvas Module**, or **BoardCanvas** is missing **Pointable Canvas**.
* If rays seem to go *through* the UI, move **BoardCanvas** a few cm off the wall (we used `+0.06` Z local).
* If UI is blurry, verify **BoardCanvas scale = 0.001** and the **Grid** area is big enough (we used `700 × 700`).
* If the player sinks or floats, adjust **CharacterController Height/Center** and place **PlayerRig.y** near `0`.

---

## Instructor Tips

* Have students **collapse** and **name** interactor containers exactly as shown; consistency prevents wiring mistakes later.
* If some students still have the old Input Module on the EventSystem, ask them to remove it and keep **only** the **Pointable Canvas Module** to avoid double input.
* Keep the interactor defaults; we’ll layer in filters and tags in a later part.

---

## End-of-Part Snapshot

### What your Hierarchy should look like now

```
PlayerRig  (OVRCameraRig + CharacterController + SimpleRigLocomotion)
└─ OVRCameraRig
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
└─ BoardCanvas  (World Space; EventSystem has Pointable Canvas Module)
   ├─ BoardPanel (stretched background)
   ├─ Grid (Grid Layout Group: 3 cols, cell 0.22, spacing 0.02)
   │  ├─ Cell_0 (Prefab instance)
   │  ├─ Cell_1
   │  ├─ Cell_2
   │  ├─ Cell_3
   │  ├─ Cell_4
   │  ├─ Cell_5
   │  ├─ Cell_6
   │  ├─ Cell_7
   │  └─ Cell_8
   └─ GameOverPanel (disabled)
      ├─ GameOverText (TMP)
      └─ RestartButton (TMP Button)

EventSystem (with Pointable Canvas Module)
Ground
Directional Light
```

### Key Inspector settings (quick recap)

* **PlayerRig → CharacterController:** Height `1.7`, Radius `0.3`, Center `y=0.9`.
* **PlayerRig → SimpleRigLocomotion:** head = camera transform under the rig.
* **Controller Ray Interactor:** Max Ray Length `6`, Hide When No Interactable `On`.
* **BoardCanvas:** World Space, Size `800×800`, Scale `0.001`, Z offset `0.06`.
* **EventSystem:** has **Pointable Canvas Module** (only).
* **BoardCanvas:** has **Pointable Canvas** (+ default **Graphic Raycaster**).
* **Grid:** Grid Layout Group (Cell `220×220`, Spacing `20×20`, Columns `3`).

---

### Next (Part 3)

We’ll add the **ScriptableObject-based** Tic-Tac-Toe logic (`Cell`, `UICellBehaviour`, `BoardManager`, `TurnManager`, `UIBehaviour`), wire the nine cells, and make **Restart** and **GameOver** work.












































































































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

