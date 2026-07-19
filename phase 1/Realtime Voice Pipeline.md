SECTION 2 & 3: Realtime Voice Architecture

1. The Latency Problem

Traditional voice assistants process sequentially: Record -> STT -> LLM -> TTS -> Play. This yields 2-4 second latencies. VOXY requires human-like conversation (< 500ms).

2. Pipeline Architecture & Benchmarks

A. Wake Word Engine (Section 3)

Alternatives: Porcupine (Picovoice), Snowboy, openWakeWord.

Porcupine: 99% accuracy, <1MB RAM, proprietary license.

openWakeWord: Apache 2.0, ONNX-based, ~20MB RAM, excellent accuracy, highly customizable.

Recommendation: openWakeWord. The open-source license prevents vendor lock-in. We will train a custom VOXY model on diverse acoustic environments.

B. Voice Activity Detection (VAD) & Barge-in

Silero VAD: State-of-the-art neural VAD. Runs in <1ms on CPU.

WebRTC AEC: Required to subtract the TTS output from the microphone input so the user can interrupt VOXY (Barge-in).

Architecture: Microhpone -> WebRTC AEC -> Silero VAD. If VAD detects human speech while TTS is playing, instantly send a cancellation token to the TTS audio buffer.

C. Streaming STT (Speech-to-Text)

Options: Whisper.cpp, Whisper-streaming, Kaldi.

Recommendation: Whisper.cpp with stream chunking. By processing audio in 500ms overlapping chunks, text is fed to the LLM before the user finishes speaking.

D. Streaming LLM Inference

Model: Llama-3-8B-Instruct (GGUF format) or Microsoft Phi-3-Mini (ONNX).

Execution: Llama.cpp with Vulkan/DirectML backend to support a wide range of AMD/NVIDIA/Intel integrated graphics without requiring CUDA installation.

E. Streaming TTS (Text-to-Speech)

Options: Piper (Fast, robotic), XTTSv2 (High quality, slow), StyleTTS2 (Near Siri/ElevenLabs quality, fast).

Recommendation: StyleTTS2 (ONNX export). Generate audio chunk-by-chunk as the LLM streams tokens. The first audio chunk can begin playing before the LLM has finished the sentence.