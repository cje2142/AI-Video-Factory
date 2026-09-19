# AI Shorts Automation Pipeline v2.32-Final — Backup Manifest

_Date: 2026-09-19_

## Artifact Identity

- Workflow name: `AI Shorts Automation Pipeline v2.32-Final`
- Source export filename: `AI Shorts Automation Pipeline v2.32-Final.json`
- JSON size: **413,600 bytes**
- SHA-256: `462c44975d5b6d1b7d07f4349188c9a33a43501e38d2fc0c86d6f83b53cd0adf`
- Nodes: **68**
- Connection source entries: **66**
- `pinData`: empty object (`{}`)

## Verified Content-Branch State

- `Fact Pack Input Expanded` = v1.7
- `Fact Pack Validator Expanded` = v1.6.2
- `Script Writer Input Expanded` = v1.2
- `Script Validator Expanded` = v1.6
- second Evidence Expansion disabled
- duration failure after Expansion routes to `stop`
- `Expanded Script Ready?` false route reaches `Stop - Insufficient Verified Content`

## Runtime Test Evidence

The 2026-09-19 cat-zoomies test verified:

- factual validation PASS
- causal integrity PASS
- unsupported filler not generated
- 15.9 s estimated script vs 40 s target
- one Evidence Expansion pass
- expanded script still insufficient
- safe STOP route completed correctly

## Restore / Verification

After restoring the workflow JSON into n8n:

1. confirm the workflow name is `AI Shorts Automation Pipeline v2.32-Final`
2. confirm `pinData` is empty
3. confirm the four Expanded node versions above
4. confirm `Script Validator Expanded` contains:
   - `needs_evidence_expansion = false` after expanded duration FAIL
   - `ready_for_scene_adapter = false`
   - `next_route = stop`
5. do not overwrite `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

## Documentation

Detailed architecture/progress checkpoint:

- `DOCS/AI_VIDEO_FACTORY_AUTOMATION_CHECKPOINT_2026-09-19.md`

Current project status:

- `DOCS/CURRENT_STATUS.md`

## Note

This manifest records the exact validated artifact identity and restore checks. The exported JSON remains the authoritative workflow artifact for v2.32-Final.


## GitHub Artifact Backup Completed

The exact validated workflow JSON is now stored in two repository locations:

- Canonical workflow:
  - `WORKFLOWS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final.json`
- Dated archive backup:
  - `BACKUPS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final_2026-09-19.json`

Verification:

- both files: **413,600 bytes**
- both Git blob SHA: `78e9279113330fc9abdecb92ead4136d659fa0e1`
- canonical create commit: `6ed40ff1fc33d50ce969054b12b8e7d8cc5273d7`
- dated archive create commit: `d45793db7a27be51849bb037f639932842b1000d`
- identical blob SHA confirms both repository copies contain identical bytes.
