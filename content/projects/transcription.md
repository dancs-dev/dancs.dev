+++
title = 'Transcription: local video subtitling with Whisper'
date = 2025-12-12T13:39:36+00:00
draft = false
summary = "A privacy-first CLI tool that uses on-device machine learning to translate and transcribe videos, generating subtitles in an open format."
[params]
    repo = 'https://github.com/dancs-dev/transcription'
+++

## Introduction

With OpenAI's Whisper local model being as good as it is, there is no need to ship off your family videos and company meeting recordings to a cloud-based company anymore.

After watching The Hobbit, which had questionable audio mixing, ranging from whisper quiet speaking containing key plot points that would transition to booming instruments, I was motivated to build Transcription, a CLI tool that you can run entirely locally on your machine. It takes a video file, extracts the audio, runs it through OpenAI's Whisper model locally, and outputs subtitles (translating into English if necessary). There are no network requests, no API keys, no per-minute billing. Once the tool and model are installed, it can work entirely offline.

You can start transcribing videos and generate a `.srt` with the following:

```bash
transcription -v ./a-video-to-transcribe.mp4

# See all configuration options available with:
transcription --help
```

## How it works

The pipeline is straightforward:

1. **Input:** the user provides path(s) to video files via the CLI.
1. **Audio extraction:** FFmpeg extracts the audio from the video, converting it into a `wav` format for compatibility with Whisper.
1. **Translation and transcription:** the extracted audio is passed to a locally running Whisper model, which translates speech into English and produces timestamped text segments.
1. **Output:** the timestamped segments are formatted into an `.srt` file and optionally embedded into the video's metadata (soft embedding) or onto the stream itself (hard embedding).

## Technology stack

- **Python:** excellent support for Whisper. I used the [SpeechRecognition](https://pypi.org/project/SpeechRecognition/) library to interface with Whisper and the [click](https://pypi.org/project/click/) library to create a robust CLI.
- **uv:** started moving away from other tools like Poetry due to uv's built-in code formatting and linter as well as fast dependency resolution. It also makes packaging the tool incredibly simple.
- **FFmpeg:** handles audio extraction from a large variety of video formats as well as optionally embedding generated subtitles into the video.
- **Whisper:** [OpenAI's general-purpose open source speech recognition model](https://github.com/openai/whisper) supporting both translation and transcription. It was trained on 680,000 hours of audio across 98 languages, with strong performance across 10 languages. There are multiple model sizes available, ranging from `tiny` (39M parameters) to `large` (1.5B parameters).

## Performance

The tool by default uses the 769M parameter `medium` multilingual model requiring around 5GB of VRAM. On my RTX 3080, it translated and transcribed a 27-minute video in 2 minutes and 11 seconds. The quality of the translation itself is good, but does miss some of the subtlety of the manual translation. The `large` model requires around 10GB of VRAM and unfortunately I get an OOM error on my hardware.

If you don't need to translate before transcribing, you can use the 809M parameter `turbo` model requiring around 6GB of VRAM. On my hardware, I could transcribe (but not translate) the same 27-minute video in 36 seconds, which is fairly close to the approximately 4x speedup they claim over the medium model.

The `tiny` model actually took the same amount of time as the medium model to translate the same video, but with noticeably worse translation. When I tested this on a 23-minute English video (i.e., no translation needed), the tiny model took 52s and the medium model took 3 minutes, which is a bit closer but not quite at the 5x relative speedup that OpenAI claimed. The transcription produced by the tiny model was better than the translation it produced.

For transcription tasks, the `turbo` model should be the go-to due to its impressive relative speed over the `medium` model whilst still retaining impressive accuracy. For translation tasks, the `medium` model produced sufficient accuracy, but it may be worth exploring the `large` model if your hardware supports it.

## What's next

I would like to test the tool on weaker hardware such as running on the CPU. Additionally, I suspect that I may be able to get faster performance for transcription-only tasks if I change the flag passed to the whisper model - I have currently fixed this for translation, as it still works well for transcription tasks and keeps the CLI options simple. This would also allow the tool to support transcribing videos in their original language.

## Further reading

- [Ragebait Block: on-device AI content filtering for a calmer feed]({{<ref "/projects/ragebait-block">}}) - another project using on-device ML to create a calmer browsing experience.
