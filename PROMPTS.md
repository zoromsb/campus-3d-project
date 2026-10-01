# Prompts, in order

Paste ONE prompt at a time into **opencode** (run it inside this folder). Test in the browser
(http://localhost:3000). Commit with git. Then move to the next prompt.
Tool tags: [opencode] [Claude chat] [Antigravity]

---

## Phase 0 — Baseline check

Run `node server.cjs`, open http://localhost:3000. You should see 4 glowing buildings with labels.
Search "Amphi", click a result: the camera should fly to the building and the side panel should open.
If it works, run `git init`, then commit.

**[opencode] Prompt 0 — make sure the agent understands the project**
```
Read AGENTS.md and index.html. Do NOT change anything yet. Reply with: (1) a 10-line summary of how index.html is organized, (2) the list of main functions and what they do, (3) any bugs or risky code you notice.
```

---

## Phase 1 — Load campus.json (with fallback)

**[opencode] Prompt 1**
```
Change how campus data is loaded. The app should try fetch('campus.json') first and fall back to the embedded CAMPUS_DATA if the fetch fails. Make CAMPUS_DATA a `let` so everything that reads it (search, side panel, fly-to, loadCampus) uses the loaded data. Make loadCampus async and start animate() after it. Then add a function rebuildCampus(data) that removes all existing building groups from the scene (dispose geometries and materials, remove the CSS2D label elements from the DOM), clears buildingMeshes, and rebuilds everything from the given data. I will use rebuildCampus from an editor later. Do not change any visuals. Tell me how to test.
```

---

## Phase 2 — Satellite underlay

Save your top-down screenshot as `assets/satellite.png` first.

**[opencode] Prompt 2**
```
Add an optional satellite image underlay. Load assets/satellite.png with THREE.TextureLoader (silently ignore errors if the file is missing). Show it as a flat plane just under the buildings (y = -0.02), darkened and about 60% opacity so the cyan buildings stay readable. Add a small "Satellite" toggle button in the top-right corner to show/hide it. Add a compact calibration panel (collapsible) with number inputs for: image width, image depth, offset X, offset Z, rotation in degrees. They must update the plane live and be saved in localStorage so they persist after reload. Defaults: 200 x 200 world units, offset 0, rotation 0. Keep the existing style. Tell me how to test.
```

Then calibrate by eye: adjust the numbers until the real buildings in the image are roughly where the
blue boxes will go. Roughly is enough.

---

## Phase 3 — Level editor (4 small steps)

**[opencode] Prompt 3a — edit mode + drawing buildings**
```
Add an "Edit" toggle button (top-right, next to Satellite). When edit mode is ON: switch the camera to a top-down view looking straight down and restrict OrbitControls to pan and zoom only (no rotation); show the satellite plane if available; snap everything to a 1-unit grid. Click and drag on the ground (raycast against the plane y = 0) to draw a rectangle. On release, create a new building with a unique id "bloc-N", name "New building", height 8, x/z/width/depth taken from the drag, and no rooms. Add it to the buildings data and show it immediately using the existing createBuildingMesh / registerBuildingMesh (or rebuildCampus). When edit mode is OFF, restore the normal 3D camera and controls and normal click-to-inspect behavior. Do not build selection, moving or forms yet. Tell me how to test.
```

**[opencode] Prompt 3b — select, move, resize, delete**
```
In edit mode, add selection and editing of buildings. Clicking a building selects it: highlight it in orange and show 4 small corner handles. Dragging the body moves it (snap to 1 unit); dragging a corner handle resizes it (min size 2). Pressing Delete or Backspace removes the selected building (not while typing in an input). Clicking empty ground deselects. All changes must update the buildings data and the meshes and labels live, without breaking search or the side panel. Tell me how to test.
```

**[opencode] Prompt 3c — edit form (name, height, rooms)**
```
In edit mode, when a building is selected, show an edit form in a side panel (same visual style as the existing side panel). Fields: name (also updates the label), id (auto-generated slug from the name, editable), height, x, z, width, depth (number inputs that update the building live). Below that, a rooms editor: a list of rooms, each with name (text), type (dropdown: amphitheater, classroom, lab, office, library, study, gym, locker, other) and floor (number), a remove button per room, and an "Add room" button. Every change updates the data so the search and the normal side panel see it right away. Tell me how to test.
```

**[opencode] Prompt 3d — save / export / import**
```
Add persistence to edit mode. (1) Autosave the whole buildings data to localStorage key "esi-campus-draft" after every change, and load it on startup if it exists (it takes priority over campus.json). (2) "Export" button: downloads the current data as campus.json (pretty printed, 2 spaces). (3) "Import" button: lets me pick a JSON file and loads it with rebuildCampus. (4) "Reset" button: asks for confirmation, clears the draft and reloads from campus.json. Also show the edit buttons only when the URL has ?edit=1, so a public version stays clean. Tell me how to test.
```

Now build the real campus in edit mode over the satellite picture. When done, click Export and replace
`campus.json` in this folder with the downloaded file.

---

## Phase 4 — Real room data

**[Claude chat] Prompt 4 — turn your notes into JSON**
```
I'm building a campus map app. Convert my notes below into a JSON array of rooms using exactly this shape, one object per room:
{ "id": "kebab-case-unique-id", "name": "Salle 101", "type": "amphitheater|classroom|lab|office|library|study|gym|locker|other", "floor": 0 }
Floor 0 is the ground floor. Return ONLY valid JSON, nothing else. If something is ambiguous, list your questions after the JSON instead of guessing.

Building: <name>
My notes:
<paste your list of rooms here>
```
Paste the result into the room list of that building in campus.json (or type the rooms in the editor form).

---

## Phase 5 — Polish and publish

**[opencode] Prompt 5a — search UX**
```
Improve the search dropdown: keyboard navigation (arrow up/down to move, Enter to select, Escape to close), highlight the matching part of the text, and show a small colored tag per room type. Keep the existing style. Tell me how to test.
```

**[opencode] Prompt 5b — mobile**
```
Make the app usable on a phone: the side panel becomes a bottom sheet on narrow screens, the search bar fits the width, tapping a building selects it, and pinch/drag work with OrbitControls. Do not change the desktop layout. Tell me how to test using the browser's device emulation.
```

**Publish:** the app is fully static. Drag the folder (index.html, campus.json, assets/) into Netlify Drop,
or push it to a GitHub repo and enable GitHub Pages. Share the link WITHOUT ?edit=1.

---

## Phase 6 (optional) — Roads and non-accessible zones

**[opencode] Prompt 6**
```
Add "zones" to the data: an optional top-level array in campus.json, each item { "id", "type": "road" | "blocked" | "green", "x", "z", "width", "depth" }. Render them as flat rectangles on the ground (road = dim white, blocked = dim red, green = dim green) with low opacity and no height, following the same holographic style. In edit mode add a "Draw:" selector (Building / Road / Blocked / Green) so the same drag-to-draw tool creates zones, and let zones be selected, moved, resized and deleted like buildings. Include zones in export/import/autosave. Do not implement pathfinding. Tell me how to test.
```
Later idea, only if you want it: pathfinding ("route me to Amphi B") using roads as walkable area.

---

## Testing helper

**[Antigravity] Prompt T — let its browser agent test the app (read-only)**
```
Open http://localhost:3000 and test the app like a user. Do NOT edit any files. Check: (1) the page loads and the loading text disappears, (2) searching "amphi" shows results, (3) clicking a result flies the camera to the building and opens the side panel with the room highlighted, (4) clicking a building directly opens its panel, (5) clicking empty space closes it. If ?edit=1 exists, open http://localhost:3000/?edit=1 and test drawing a building, moving it, resizing it, editing its rooms, Export and reload. Report every bug with exact steps to reproduce and what you saw in the browser console.
```

## Debug helper

**[Claude chat] Prompt D — when something breaks**
```
My Three.js single-file app (index.html) has a problem. What I did: <steps>. What I expected: <...>. What happened: <...>. Browser console error (F12 > Console): <paste>. Here is the relevant code: <paste the function(s), not the whole file>. Explain the cause first, then give the smallest fix.
```
