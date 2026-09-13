# AI Shorts Automation Pipeline v0.71 — Full Automation Baseline

Status: FIRST FULL END-TO-END AUTOMATION BASELINE

This backup preserves the exact original uploaded workflow bytes before the FLUX → LTX I2V redesign.

Original filename:
`AI Shorts Automation Pipeline v0.71-full-run-t2v-ok.json`

Original size:
`124893 bytes`

Archive method:
- gzip compressed with mtime=0
- split into 4 binary parts
- parts must be concatenated in numeric order

Restore on Linux/macOS:
```bash
cat AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json.gz.part01 \
    AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json.gz.part02 \
    AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json.gz.part03 \
    AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json.gz.part04 \
    > AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json.gz

gzip -dc AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json.gz \
    > AI_Shorts_Automation_Pipeline_v0.71_FULL_AUTOMATION_BASELINE.json
```

Verified milestone:
- Production planning
- Scene split
- Prompt engine
- ComfyUI generation
- History polling
- Scene collection
- TTS
- Continuous narration
- Whisper
- SRT
- FFmpeg final compose
- Final MP4

Note:
Visual generation quality is not final. This version is preserved as the first complete end-to-end automation baseline before pipeline redesign.
