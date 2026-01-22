# transcribe - Audio and Video Transcription

Transcribe audio and video files using AssemblyAI with speaker diarization.

## Overview

`transcribe` processes local audio or video files and generates speaker-diarized transcripts in Markdown format. It handles the entire pipeline: media normalization, transcription, and formatting.

## Key Features

- Supports both audio and video files (extracts audio from video automatically)
- Converts media to Speech-to-Text optimized format (16kHz, mono, PCM WAV)
- Speaker diarization (identifies different speakers)
- Automatic punctuation and word timestamps
- Outputs clean Markdown transcripts with speaker labels and timestamps
- Handles long files (up to 10 hours)
- No cloud storage management required - AssemblyAI handles everything

## Installation

### System Dependencies

Install ffmpeg:
```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg
```

### Python Dependencies

```bash
pip install assemblyai python-dotenv
```

### AssemblyAI API Setup

1. **Get an API Key**
   - Go to https://www.assemblyai.com/
   - Sign up for a free account
   - Copy your API key from the dashboard

2. **Add to Environment Variables**

   Add to your `.env` file in the repository root:
   ```bash
   ASSEMBLYAI_API_KEY=your-api-key-here
   ```

   Or set in your shell (add to `~/.zshrc` or `~/.bashrc`):
   ```bash
   export ASSEMBLYAI_API_KEY="your-api-key-here"
   ```

## Usage

```bash
transcribe /path/to/audio_or_video.mp4
```

By default, the transcript is saved to the **current directory** with the same name as the input file but with a `.md` extension:

```bash
cd ~/Documents
transcribe /path/to/videos/interview.mp4
# Creates: ~/Documents/interview.md (in current directory)
```

You can specify a custom output location with the `--output` flag:

```bash
transcribe audio.mp3 --output transcript.md
transcribe video.mp4 -o ~/Desktop/output.md
```

## Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `input_file` | Path to audio or video file (required) | - |
| `--output`, `-o` | Path to save markdown transcript (optional) | `<filename>.md` in current directory |
| `--language`, `-l` | Language code (e.g., en, es, fr) | Auto-detect |
| `--title`, `-t` | Custom title for the transcript | Input filename |
| `--keep-wav` | Keep the normalized WAV file after transcription | False |

## Supported File Formats

**Audio:** WAV, MP3, FLAC, OGG, AAC, M4A, WMA

**Video:** MP4, MOV, AVI, MKV, FLV, WEBM, WMV

Any format supported by ffmpeg will work.

## Output Format

The generated Markdown file includes:
- Timestamps in `[HH:MM:SS]` format
- Speaker labels (Speaker 1, Speaker 2, etc.)
- Full transcript with automatic punctuation

Example output:
```markdown
[00:00:02] Speaker 1: Hello everyone, welcome to today's meeting.
[00:00:08] Speaker 2: Thanks for having me. I wanted to discuss the project timeline.
[00:00:15] Speaker 1: Sure, let's go through the milestones together.
```

## Flowchart

```
┌─────────────────────────────────────┐
│ Input: Audio/Video File             │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Probe Media (ffprobe)                │
│ - Detect audio/video streams         │
│ - Get duration, codecs               │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Normalize to WAV (ffmpeg)            │
│ - Extract audio from video           │
│ - Convert to 16kHz, mono, PCM        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Transcribe (AssemblyAI)              │
│ - Upload audio directly to API       │
│ - Speaker diarization                │
│ - Automatic punctuation              │
│ - Word-level timestamps              │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Format to Markdown                   │
│ - Speaker labels with timestamps     │
│ - Format: [HH:MM:SS] Speaker N: text│
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Save to: output.md                   │
│ (specified or default location)      │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Cleanup Temporary Files              │
│ - Delete normalized WAV (if not kept)│
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Output: transcript.md                │
└─────────────────────────────────────┘
```

## Examples

### Basic transcription (saves to current directory)
```bash
cd ~/Documents
transcribe ~/Downloads/podcast_episode.mp3
# Creates: ~/Documents/podcast_episode.md
```

### Transcribe with custom output location
```bash
transcribe ~/Videos/team_meeting.mp4 --output meeting_notes.md
```

### Specify language explicitly
```bash
transcribe ~/audio/spanish_interview.mp3 --language es
```

### Keep the normalized WAV file
```bash
transcribe ~/media/interview.mp4 --keep-wav
```

### Save to specific location
```bash
transcribe /path/to/audio.mp3 -o ~/Desktop/transcript.md
```

## Processing Times

AssemblyAI processes transcriptions asynchronously. The vast majority of files complete in under 45 seconds to 1 minute, regardless of audio duration, with a Real-Time-Factor (RTF) as low as .008x.

## Limitations

- Maximum file size: 5GB
- Maximum duration: 10 hours
- Minimum duration: 160ms
- Input files must contain an audio stream
- Requires active internet connection
- API rate limits apply based on your AssemblyAI plan

## Troubleshooting

### "ffmpeg not found in PATH"
Install ffmpeg using your system's package manager (see Installation section).

### "ASSEMBLYAI_API_KEY environment variable not set"
Make sure your `.env` file contains `ASSEMBLYAI_API_KEY=your-key` and is in the repository root, or set it in your shell environment.

### "No audio stream found"
The input file must contain an audio track. Verify with:
```bash
ffprobe /path/to/your/file.mp4
```

### "ModuleNotFoundError: No module named 'assemblyai'"
Install the AssemblyAI SDK:
```bash
pip install assemblyai
```

Or if using a virtual environment:
```bash
source .venv/bin/activate
pip install assemblyai
```

### Transcription fails or returns empty results
- Check that the audio quality is clear
- Verify the audio file isn't corrupted
- Ensure your API key is valid and has available credits
- Check AssemblyAI status page for service issues

## Cost Considerations

AssemblyAI pricing (as of 2026):
- Base transcription: $0.15/hour
- Speaker diarization: +$0.02/hour
- **Total for this utility: $0.17/hour** (prorated per second)
- Free tier: $50 in credits for new accounts

Billing is prorated to the exact second. For example, a 30-minute transcription costs approximately $0.085.

Check https://www.assemblyai.com/pricing for current rates and additional feature pricing.

## Directory Structure

```
assemblyai/
├── executable_scripts/
│   └── transcribe           # Main executable script
├── utilities_data/
│   └── transcribe/
│       ├── media_probe.py   # Media probing (ffprobe wrapper)
│       ├── ffmpeg_audio.py  # Audio normalization
│       ├── format_md.py     # Transcript formatting
│       └── transcribe_aai.py# AssemblyAI API integration
└── README/
    └── README_transcribe.md # This file
```

## Technical Details

### Audio Normalization
- Converts any input to 16kHz, mono, PCM WAV format
- Uses ffmpeg for reliable format conversion
- Handles both audio-only and video files

### Speaker Diarization
- Powered by AssemblyAI's speaker identification
- Automatically detects number of speakers
- Labels speakers as "Speaker 1", "Speaker 2", etc.
- No limit on number of speakers (typically accurate up to 8-10)

### Timestamp Accuracy
- Word-level timestamp precision
- Grouped by speaker turns
- Format: `[HH:MM:SS]` for easy readability

## License

Personal utilities - use at your own discretion.
