# transcribe - Audio and Video Transcription

Transcribe audio and video files using AssemblyAI with intelligent speaker identification.

## Overview

`transcribe` processes local audio or video files and generates speaker-diarized transcripts in Markdown format with automatic name detection. It handles the entire pipeline: media normalization, transcription, speaker identification, and formatting.

## Key Features

- **Intelligent Speaker Identification** - Automatically detects speaker names from conversation context
- **Smart Fallback Labels** - Uses "Speaker 1", "Speaker 2" when names can't be detected
- **Universal Language Model** - Supports 99+ languages with high accuracy
- Supports both audio and video files (extracts audio from video automatically)
- Converts media to Speech-to-Text optimized format (16kHz, mono, PCM WAV)
- Advanced speaker diarization (up to 10 speakers)
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
- Speaker labels (detected names or generic labels)
- Full transcript with automatic punctuation

### Speaker Identification

The tool uses AssemblyAI's Speaker Identification feature to automatically detect speaker names from conversation context:

**Example with detected names:**
```markdown
[00:00:13] Mike: Hey, Em.
[00:00:15] Em: Hey, Mike. How are you? Can you hear me clearly?
[00:00:18] Mike: I can. I'm doing all right. How about you?
```

**Example with mixed detection (5-person meeting):**
```markdown
[00:01:26] Jordan: Hey, Lauren.
[00:01:27] Lauren: Hello. How are you?
[00:01:36] Jordan: I totally just realized you probably wanted that doc edited.
[00:02:53] Speaker 1: High rollers here.
[00:03:25] Spencer: Sweet.
```

**Example when names can't be detected:**
```markdown
[00:00:02] Speaker 1: Hello everyone, welcome to today's meeting.
[00:00:08] Speaker 2: Thanks for having me. I wanted to discuss the project timeline.
[00:00:15] Speaker 1: Sure, let's go through the milestones together.
```

### How Speaker Identification Works

The AI model analyzes conversation patterns to detect names, such as:
- Greetings: "Hey, Mike" or "Hi, Jennifer"
- Direct address: "Mike, can you hear me?"
- Introductions: "This is Sarah speaking"

When names are detected, they're used directly. When detection fails, the tool falls back to generic labels (Speaker 1, Speaker 2, etc.) to ensure you always get a usable transcript.

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
│ - Universal model transcription      │
│ - Speaker diarization                │
│ - Speaker identification (names)     │
│ - Automatic punctuation              │
│ - Word-level timestamps              │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Format to Markdown                   │
│ - Use detected names or "Speaker N"  │
│ - Add timestamps [HH:MM:SS]          │
│ - Format: [HH:MM:SS] Name: text      │
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
- Speaker identification: US regions only
- Speaker names limited to 35 characters

## Best Practices for Speaker Identification

To improve the accuracy of speaker name detection:

**Do:**
- Ensure clear audio quality with minimal background noise
- Have speakers introduce themselves or use names naturally
- Allow each speaker at least 30 seconds of speaking time
- Use standard name formats (first names or full names)

**Avoid:**
- Very short conversations (less than 1 minute per speaker)
- Heavy echo or overlapping speech
- Unclear or mumbled introductions
- Expecting detection of nicknames or uncommon name spellings

Note: Even with optimal conditions, some speakers may not be detected. The tool will automatically use "Speaker N" labels as a fallback, ensuring you always get a usable transcript.

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

### Speaker names not detected (showing as "Speaker 1", "Speaker 2")
This is normal behavior and not an error. Speaker name detection works when:
- Speakers introduce themselves or use names naturally ("Hi, I'm Mike")
- Audio quality is clear with distinct voices
- Each speaker has sufficient speaking time (~30 seconds minimum)

If names aren't detected, the tool automatically uses generic labels so your transcript is still fully usable.

## Cost Considerations

AssemblyAI pricing (as of 2026):
- Universal model: $0.15/hour
- Speaker diarization: +$0.02/hour
- Speaker identification: Included (no extra cost)
- **Total for this utility: $0.17/hour** (prorated per second)
- Free tier: $50 in credits for new accounts

**Example costs:**
- 30-minute meeting: ~$0.085
- 1-hour interview: ~$0.17
- 2-hour conference: ~$0.34

Billing is prorated to the exact second. Speaker name detection is included at no additional cost.

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

### Speech Model

This tool uses AssemblyAI's **Universal model** for optimal performance:

**Why Universal?**
- Best for multi-speaker meetings with superior name detection
- Multilingual support - Works with 99+ languages
- Excellent accuracy - ~7% Word Error Rate across languages
- Production-ready - Proven stability and reliability
- Cost-effective at $0.15/hour

### Audio Normalization
- Converts any input to 16kHz, mono, PCM WAV format
- Uses ffmpeg for reliable format conversion
- Handles both audio-only and video files

### Speaker Diarization & Identification
- Powered by AssemblyAI's Universal model with Speaker Identification
- Automatically detects number of speakers (up to 10 supported)
- **Intelligent name detection** from conversation context
- Falls back to generic labels ("Speaker 1", "Speaker 2") when names aren't detected
- 2.9% error rate in speaker count detection (industry-leading)
- Best results when each speaker has ~30 seconds of clear audio
- Handles mixed scenarios (some names detected, others generic)

### Timestamp Accuracy
- Word-level timestamp precision
- Grouped by speaker turns
- Format: `[HH:MM:SS]` for easy readability

## References

- [AssemblyAI Universal Model](https://www.assemblyai.com/blog/announcing-universal-1-speech-recognition-model)
- [Speaker Identification Documentation](https://www.assemblyai.com/docs/speech-understanding/speaker-identification)
- [Speaker Diarization Guide](https://www.assemblyai.com/blog/what-is-speaker-diarization-and-how-does-it-work)
- [AssemblyAI Pricing](https://www.assemblyai.com/pricing)

## License

Personal utilities - use at your own discretion.
