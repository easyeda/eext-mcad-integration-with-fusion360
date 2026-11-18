[English](#) | [简体中文](README.md)

# Fusion360 MCAD Integration — EasyEDA Pro Extension

Real-time collaboration between EasyEDA Pro and Fusion360 over a local WebSocket. Supports model export (into a new document or into your existing design), bidirectional position sync, cross-probe, and deletion sync.

> Tested on macOS with Fusion (November 2026 builds). Nothing in the code is platform-specific, but Windows reports are welcome.

## Features

| Feature | Description |
|---|---|
| 3D Model Export | Transfer PCB STEP model to Fusion360 in chunks, supports large files |
| Download Script | One-click save Fusion360 Add-In script to local disk |
| Bidirectional Position Sync | Drag component in EDA → Fusion360 follows, and vice versa |
| Cross-Probe | Click a component on one side, the other side auto-focuses |
| Deletion Sync | Delete component in EDA, Fusion360 removes it simultaneously |
| Export Settings Dialog | Choose import target, up axis, board offset/rotation and Capture Position behaviour before each export |
| Import Into Active Design | Add the PCB as one component to your existing (e.g. enclosure) design, keeping its timeline |
| Board Placement | Y-up/Z-up handling plus X/Y/Z offset and rotation; changing them later re-places the existing board |
| Capture Position Handling | Optional automatic Capture Position, keeping a single capture in the timeline |

***

## Requirements

| Item | Requirement |
|---|---|
| EasyEDA Pro | ≥ 3.0 |
| Fusion360 | Installed and runnable |
| Python | Fusion360 built-in Python environment |
| Network | Localhost available |

***

## Installation

### Step 1: Install EDA Extension

1. Open **EasyEDA Pro**
2. After installation, find **Fusion360 MCAD Integration** in the extension list and confirm it is enabled
3. **Enable external interaction permission**: Go to **Extensions** → **Extension Settings** → Enable **External Interaction** permission (required for WebSocket communication)

### Step 2: Install Fusion360 Script

1. Download the script
   (The script file can also be saved locally via EDA menu **Fusion360 MCAD Integration** → **Download Script**)
   ![](images/下载脚本.png)
2. Open **Fusion360** and create a new document
   ![](images/fusion打开.gif)
3. Create a new add-in module
   ![](images/创建脚本.gif)
4. Open the script file location, replace the script content with the downloaded script file. Note that the script name must match the one used during creation.
   ![](images/6.png)
   ![](images/7.png)
5. Click Run. Note: you must run the module, not the script file.

   ![](images/8.png)
6. **Start automatically with Fusion**: press **Shift+S** → **Add-Ins** tab → select the add-in → tick **Run on Startup**.
   If you created it as a *Script* instead of an *Add-In*, open its `.manifest` file and change `"type": "script"` to `"type": "addin"` (and add `"runOnStartup": true`), then move the folder from `API/Scripts` to `API/AddIns`.
7. **Verify successful startup**: You should see the following in Fusion360's text command window:

```
[EasyEDA:main] CustomEvent registered: easyeda-sync-xxxxxxxx
[EasyEDA:main] Add-in started, version 1.2.6
[EasyEDA:ws] Starting WebSocket server on ws://127.0.0.1:8767
[EasyEDA:ws] WebSocket server listening on 127.0.0.1:8767
```

Everything is also written to `~/easyeda_fusion.log`.

## Usage Guide

### 1. Export 3D Model to Fusion360

1. Make sure the Fusion360 add-in is running
2. In Fusion360, open the design you want the board in (only needed for **Into the active Fusion design**)
3. Open a PCB design file in EDA, enter the **PCB Editor**
4. Click menu **Fusion360 MCAD Integration** → **Export 3D to Fusion360**

![](images/1.png)

5. The **export settings** dialog opens (see below). Click **Export**.
6. The extension connects to Fusion360, retrieves the STEP file and uploads it in chunks.

![](images/2.png)

![](images/11.png)

### Export settings

| Setting | Options |
|---|---|
| Import target | **New Fusion document**: much faster, Direct Design mode. **Into the active Fusion design**: the board is added as one component and your timeline is kept; slow for large boards (minutes) |
| Up axis | **Auto** (follows Fusion's *Default modeling orientation* preference), **Y up** (board is turned so its top faces +Y), **Z up** (as exported) |
| Board placement | Offset (mm) and rotation (°) on the Fusion axes, applied after the up-axis turn. Clicking **Save** re-places a board that is already in the design |
| Capture Position | **Once, when sync stops**, **After a pause in moves** (delay in seconds) or **Never**. *Keep a single capture* replaces the previous add-in capture instead of adding a new one each time |
| Show this dialog on every export | When off, export uses the saved settings; change them via **Fusion 360 export settings** in the menu |

Board placement and part moves are stored in two timeline captures, **EasyEDA placement** and **EasyEDA PCB**. Captures you make yourself are not touched.

> Re-export only when parts are added or removed. After restarting Fusion or EasyEDA, open the design and click **Enable Bidirectional**; the board is found again by designator.

### 2. Enable Bidirectional Interaction

After exporting the model, you can enable bidirectional interaction for real-time sync:

1. Click menu **Fusion360 MCAD Integration** → **Enable Bidirectional**
   ![](images/3.png)
2. EDA will automatically map component designators to Fusion360 3D objects (build\_mapping)
3. After mapping is successful, you can:
   - **Drag a component in EDA** → The 3D model in Fusion360 follows in real-time
   - **Drag an object in Fusion360** → The component in EDA moves synchronously (Fusion pushes position changes over the WebSocket every 2 seconds)
   - **Click a component in EDA** → Fusion360 auto-selects and focuses (flashing highlight)
   - **Click an object in Fusion360** → EDA auto-navigates to the corresponding component

> **Known Limitation:**
> The mapping is one-to-one. Opening multiple pages will cause mapping conflicts. Please operate on a single page.


### 3. Stop Bidirectional Interaction

1. Click menu **Fusion360 MCAD Integration** → **Disable Bidirectional**

![](images/4.png)

2. All mappings and listeners will be cleared.

### 4. Connection Management

| Menu Option | Function |
|---|---|
| Export 3D to Fusion360 | Opens the settings dialog, then auto-connect + chunked upload + import |
| Fusion 360 export settings | Change and save settings without exporting |
| Enable Bidirectional | Enable real-time bidirectional sync |
| Disable Bidirectional | Stop sync and clear mappings |
| Disconnect Fusion360 | Disconnect WebSocket connection |
| Download Script | Save Fusion360 script |

***

## Technical Details

### Communication Architecture

```
EDA ←— WebSocket:8767 (localhost only) —→ Fusion
```

| Direction | Description |
|---|---|
| EDA → Fusion | File upload, export settings, designator mapping, position sync, cross-probe, delete |
| Fusion → EDA | Selection and position changes, pushed every 2 seconds while bidirectional sync is on |

### Key Technical Decisions

| Item | Solution | Reason |
|---|---|---|
| WebSocket Implementation | Pure Python (asyncio) | Fusion360 environment cannot install third-party libraries |
| STEP Import | `importManager.importToNewDocument()` / `importToTarget2()` | Documented API; the old `Translator.Import` text command failed on some file paths and timed out (issue #2) |
| Main-thread work | Queue + `CustomEvent` | All Fusion API calls run on Fusion's main thread; the WebSocket thread only queues work and waits |
| Logging | Main thread → Fusion text window; every line also to `~/easyeda_fusion.log` | `app.log()` is not safe from background threads |
| Part moves | Computed in the board's own frame, keeping each part's flip/tilt | Board can be moved or rotated inside an enclosure; bottom-side parts stay flipped |
| Board never first component | A temporary empty component is created before importing into an empty design | Fusion treats the first component as anchored, and Capture Position drops its move |
| Position Change Threshold | Position 0.1mm, Rotation 0.5° | Filter floating-point noise, avoid feedback loops |

### Thread Safety

Fusion360 API requires all document operations to run on the main thread, while the WebSocket server runs on a background thread. Every Fusion API call (import, mapping, position updates, cross-probe, delete, polling, logging) is queued and executed on the main thread via a `CustomEvent`. Stopping the add-in shuts the WebSocket server down and frees the port, so **Stop / Run** in Fusion reloads the code cleanly.

## FAQ

### Failed to connect to Fusion360

Check in order:

1. **Is Fusion360 running** and the add-in active (check the log for `WebSocket server listening`)
2. **Is port 8767 occupied** — Close Fusion360 and retry
3. **External interaction permission** — Is it enabled in EDA extension settings
4. **Firewall** — Check if a firewall is blocking localhost port 8767

### Import into the active design takes minutes

That time is spent inside Fusion's own STEP import (`importToTarget2`), which is much slower in a parametric design with a timeline than into a new Direct-mode document. It is a one-time cost: moves sync live without re-importing. Use **New Fusion document** for quick checks.

### Fusion360 is unresponsive during import

STEP import runs on Fusion's main thread, so the UI waits until it finishes. The extension waits up to 10 minutes for the result.

### Where is the log?

Fusion's text command window, and `~/easyeda_fusion.log` (every line, including those from the WebSocket thread). If an import fails, the uploaded STEP file is kept in the temp folder so you can try importing it by hand.

### Selecting a component in Fusion360 has no response in EDA

1. Confirm bidirectional interaction is enabled (log should show `Monitor enabled`)
2. Polling interval is 2 seconds, wait 1-2 seconds after selection
3. Check if designator mapping was correctly established (log should show `Mapping: X / Y matched`)

***

## Project Structure

```
pcb-export-to-fusion/
├── src/
│   └── index.ts                         # EDA extension main logic (TypeScript)
├── iframe/
│   └── settings.html                    # Export settings dialog
├── script/
│   ├── Interactive-with-fusion.py       # Python script
├── config/
│   ├── esbuild.common.ts                # Build configuration
│   └── esbuild.prod.ts
├── build/
│   ├── packaged.ts                      # Packaging script
│   └── dist/
├── locales/
│   ├── zh-Hans.json                     # Chinese translations
│   └── en.json                          # English translations
├── images/
│   └── logo.png                         # Extension icon
├── extension.json                       # Extension manifest
├── package.json
└── tsconfig.json
```
