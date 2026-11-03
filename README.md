# DaVinci Resolve AI Caption Automation

An end-to-end Python automation tool for **DaVinci Resolve** that automatically transcribes video using **OpenAI Whisper**, formats captions into short, readable multi-line chunks, places title template clips on a dedicated video track, updates Fusion `TextPlus` nodes, and renders the final video.

This automation works on Davinci Resolve Studio only.

---

## 🌟 Key Features

- **Automated AI Transcription**: Transcribes audio directly from MP4 video files using OpenAI Whisper with word-level timestamps.
- **Smart Caption Chunking**: Formats captions into clean, readable lines based on customizable constraints:
  - Max **4 words per line**
  - Max **24 characters per line**
  - Max **2 lines per caption block**
  - Natural sentence breaks on punctuation (`.`, `,`, `?`, `!`)
  - Automatic split on silence gaps exceeding 8 frames
- **Direct DaVinci Resolve Integration**: 
  - Automatically connects to running DaVinci Resolve instances.
  - Imports source video and builds a new timeline (`<filename>_Subtitled`).
  - Creates **Video Track 3** if it does not exist.
  - Placed title template clips (`TFx-Caption2`) synchronized precisely with timeline timecode offsets (`GetStartFrame()`).
  - Supports **100% frame-accurate alignment** using DaVinci Resolve's active timeline FPS (29.97 / 30 / 25 / 24 FPS) or direct Subtitle Track reading.
- **Fusion Engine Text Updates**: Directly injects formatted text into Fusion `TextPlus` tools using `SetInput("StyledText", ...)`.
- **Isolated Queue Rendering**: Clears legacy render jobs, configures H.265 MP4 at 1920x1080 resolution (`<filename>_rel1.mp4`), and monitors export progress in real-time.

---

## 📋 Prerequisites & Requirements

### Software Requirements
1. **DaVinci Resolve** (Studio running on Windows/macOS) <-- this automation project is tested on Windows only
2. **Python 3.8+**

### Python Package Dependencies

Install the required Python libraries:

```bash
pip install openai-whisper torch

Note: Ensure ffmpeg is installed on your system path, as required by OpenAI Whisper. <-- Ask AI how to do this

DaVinci Resolve Scripting API Environment Variables
Make sure your environment variables point to DaVinci Resolve's scripting library:
Windows:
RESOLVE_SCRIPT_API: %PROGRAMDATA%\Blackmagic Design\DaVinci Resolve\Support\Developer\Scripting
RESOLVE_SCRIPT_LIB: C:\Program Files\Blackmagic Design\DaVinci Resolve\fusionscript.dll
PYTHONPATH: %PYTHONPATH%;%RESOLVE_SCRIPT_API%\Modules\
macOS:
RESOLVE_SCRIPT_API: /Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting
RESOLVE_SCRIPT_LIB: /Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Libraries/Fusion/fusionscript.so
PYTHONPATH: $PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/

## Preparation in DaVinci Resolve
Before running the script:
Open DaVinci Resolve and load your project.
Ensure the title template TFx-Caption (or your custom template name) exists in your project's Media Pool Master Bin (or any sub-folder).

## Parameters

if __name__ == "__main__":
    file_path = r"C:\your-own-path\"
    video_file = os.path.join(file_path, "draft1.mp4")
    srt_file = os.path.join(file_path, "captions.srt")

    run_caption_workflow(
        video_path=video_file,
        output_srt=srt_file,
        template_name="TFx-Caption2",
        fps=30,
        model_size="base",
        lines_per_caption=1,     # 1 for single line, 2 for double line
        words_per_line=5,        # Up to 5 words per line
        max_chars_per_line=32,   # Up to 32 characters per line
    )

## Usage
Run the master script directly from your terminal or IDE:
python davinci_caption_automation_v11.py

Technical Details & API Gotchas Solved
- Timeline Start Frame Offset: Resolves default timeline start timecode (01:00:00:00 / 108000 frames) by computing timeline.GetStartFrame() + recordFrame.
- Fusion TextPlus Updating: Uses Fusion's native tool.SetInput("StyledText", formatted_text) to dirty tool inputs and render updated text geometry immediately.
- Render Isolation: Clears pre-existing render jobs via project.DeleteAllRenderJobs() so only the newly generated subtitled timeline is exported.
- Render Loop Polling: Checks project.IsRenderingInProgress() to prevent execution errors during console status polling.

## License
MIT License - feel free to adapt and integrate into your own editing workflows!
