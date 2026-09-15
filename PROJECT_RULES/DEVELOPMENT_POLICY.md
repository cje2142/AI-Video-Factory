# AI Video Factory Development Policy v1.1

## 1. Roles
- ChatGPT: project planning, overall architecture, step-by-step progress control, validation.
- Gemini: n8n/ComfyUI implementation, modification, technical quality review.
- User: execute tests, confirm results, approve changes.

## 2. Development Principles
- Proceed one stage at a time.
- Do not modify a known-good workflow unnecessarily.
- Test every new stage separately before integration.
- Never change multiple major components at once.
- For code changes, provide/use the complete revised code rather than partial patches.
- Preserve working versions before modification.

## 3. Validation
- Verify actual file creation, not only node success.
- Verify n8n binary fields and file paths.
- Verify ComfyUI History before treating asynchronous generation as complete.
- For 5-scene generation, wait and poll until all expected scene videos are confirmed.
- Confirm final output file and metadata before closing the workflow.

## 4. Backup
- GitHub is the official project backup.
- Keep n8n and ComfyUI backups separated under WORKFLOWS/ and BACKUPS/.
- Do not overwrite a known-good backup without first confirming the new version.
- Version meaningful workflow changes.

## 5. Current Production Route
Manual/automatic trigger → Ollama script → Scene parsing → Image prompts → ComfyUI image/video generation → 5-scene completion check → video composition → TTS narration → subtitles → BGM → final MP4 → /files storage.

## 6. Recovery
If a new change fails validation, return to the last verified version and diagnose one issue at a time.

## 7. Unified Workspace Root
- The default Windows workspace root for ChatGPT + user project work is `D:\chatgpr`.
- New local project files, scripts, temporary working files, test outputs, project utilities, and handoff assets should be created under this root whenever technically possible.
- Each project should use its own subfolder under `D:\chatgpr` rather than scattering files across unrelated Windows paths.
- Existing working applications, model repositories, installed runtimes, and validated environments may remain in their current locations when moving them would create unnecessary risk or break dependencies.
- When an existing external path must be used, keep project-owned generated assets and integration files under `D:\chatgpr` where practical and reference the external dependency explicitly.
- Prefer stable, predictable paths over ad-hoc Desktop, Downloads, or temporary-folder locations.
- Before introducing a new top-level local path outside `D:\chatgpr`, verify that it is required by the tool or runtime.
