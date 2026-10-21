# doc_copilot

This repository collects small Copilot/Codex workflow notes and troubleshooting guides.

## Documents

| File | Topic | Use When |
| --- | --- | --- |
| [Copilot_Hisotry_Export_Import.md](./Copilot_Hisotry_Export_Import.md) | VS Code Copilot Chat history export/import | You need to move or restore a Copilot Chat session. |
| [Vite_Dev_Server_Chunk_Error_Workaround.md](./Vite_Dev_Server_Chunk_Error_Workaround.md) | Vite dev server chunk error workaround | `npm run start` opens a page with `/node_modules/.vite/deps/chunk-*.js` errors or `504 Outdated Optimize Dep`. |
| [vite-dev-server-exception-vite-001.pptx](./vite-dev-server-exception-vite-001.pptx) | VITE-001 slide deck: why the page crashed after Vite optimization (MUI Popper deep import, `styled_default is not a function`) and the fix (`@mui/base/PopperUnstyled`) | You want the root cause and fix of the chunk error explained as slides, as a companion to the workaround guide above. |
| [Hooks注入Skill覆寫實測紀錄.zip](./Hooks%E6%B3%A8%E5%85%A5Skill%E8%A6%86%E5%AF%AB%E5%AF%A6%E6%B8%AC%E7%B4%80%E9%8C%84.zip) | Hooks test report (extract and open `index.html`; includes 4 screenshots in `img/`): using `UserPromptSubmit` / `PreToolUse` hooks to inject a skill override before the skill runs, without editing the repo's skill files. Example: a local `verify-pr` wrapper | You want a worked example of overriding skill behavior with hooks, including how to prove the result came from the hook. |
| [CloudeDirectorSystemtooldirector-system-evaluatordirector-system-evaluator.md](./CloudeDirectorSystemtooldirector-system-evaluatordirector-system-evaluator.md) | Plan for adopting Grill (`mattpocock/skills`) in a Director-style workflow: install (Part A), auto-use on project tasks (Part B), rolling reinforcement with mistake tracking and Lv0–Lv5 question levels (Part C) | You want to see how `grill-me` / `grill-with-docs` were introduced to clarify requirements before coding. |

## Maintenance Rules

- Keep each guide as one Markdown file with a clear title.
- Use stable English filenames so links work across platforms.
- Keep commands and error messages in code blocks so they are easy to copy and search.
- Update this README whenever a new `.md` file is added.
