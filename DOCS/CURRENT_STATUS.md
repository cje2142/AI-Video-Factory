# AI Video Factory — Current Status

## Project
Local AI Shorts automation pipeline centered on n8n + ComfyUI.

## Verified
- n8n Docker environment is operational.
- ComfyUI connection is operational.
- AI image generation has been verified.
- PNG output can be transferred to n8n as Binary field `data`.
- n8n `/files` access and Windows bind-mount storage have been verified.
- PNG-to-MP4 generation has been verified.
- The workflow is designed to wait for all 5 scene videos to finish, verify ComfyUI History, then output the final selected MP4.

## Next Pipeline
Scene videos → video composition → narration/TTS → subtitles → BGM → final Shorts MP4.

## Operating Principle
One step at a time. Test each stage separately before connecting it to the main pipeline. Preserve known-good versions before modifications.
