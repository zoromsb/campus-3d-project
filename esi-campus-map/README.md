# ESI Campus Map

## Run it
1. Open a terminal in this folder.
2. `node server.cjs`
3. Open http://localhost:3000  (needs internet: three.js loads from unpkg)

Opening index.html by double-click also works now, but campus.json fetching and the
satellite image need the server, so always prefer localhost:3000.

## Which AI for what
| Tool | Use it for | Don't use it for |
|---|---|---|
| **opencode** (main builder) | Every coding phase in PROMPTS.md. It already lives in this folder and reads AGENTS.md. | Big vague requests ("build the editor") — give it one phase at a time. |
| **Claude chat** (brain + debugger) | Planning, debugging (paste the console error + the function), reviewing what opencode changed, turning your room notes into campus.json. | Editing the project files directly, it can't see your folder. |
| **Antigravity** (tester, optional) | Its browser agent can open localhost:3000 and click through the app to find bugs (test prompt in PROMPTS.md). Also a fallback builder if opencode hits limits. | Editing at the same time as opencode. |

Rule: only ONE agent edits the code at a time.

## Safety net (do this once)
    git init
    git add . && git commit -m "working baseline"
After every phase that works: `git add . && git commit -m "phase X done"`.
If an agent breaks something: `git restore index.html` and try again with a smaller prompt.

## Order of work
0 Baseline check -> 1 Load campus.json -> 2 Satellite underlay -> 3 Editor (a,b,c,d) ->
4 Real data -> 5 Polish + publish -> 6 (optional) roads / blocked zones.
Full prompts: PROMPTS.md
