






















# SovereignEngine 888

Sistema de integração e benchmarking automatizado para modelos Text-to-Speech (TTS) com validação de latência e monitoramento de VRAM.

## Características

- **Benchmark Robusto**: Warmup + mediana de múltiplas runs elimina variância de medição
- **FileLock**: Operações thread-safe para produção em massa
- **Validação de Áudio**: Detecta arrays vazios, NaN e clipping
- **Monitoramento VRAM**: Tracking de pico e média durante inferência
- **Real-Time Factor (RTF)**: Métrica de capacidade de tempo real
- **Blacklist Inteligente**: Expiração automática de entradas antigas
- **Cleanup Seguro**: Liberação garantida de recursos GPU

## Instalação

```bash
pip install -r requirements.txt
```

## Uso Básico

```python
from sovereign_engine import SovereignEngine

# Context manager garante cleanup
with SovereignEngine(latency_threshold_ms=888.0) as engine:
    # Integra modelo
    if engine.integrate_tts_model("microsoft/speecht5_tts"):
        # Gera fala
        audio = engine.generate_speech("Olá mundo!", play=True)
```

## Uso Avançado

### Processamento em Batch

```python
import numpy as np
import wave

def save_wav(audio, filename, sr=16000):
    audio_int16 = (audio * 32767).astype(np.int16)
    with wave.open(filename, 'w') as f:
        f.setnchannels(1)
        f.setsampwidth(2)
        f.setframerate(sr)
        f.writeframes(audio_int16.tobytes())

with SovereignEngine() as engine:
    engine.integrate_tts_model("microsoft/speecht5_tts")
    
    textos = ["Texto 1", "Texto 2", "Texto 3"]
    for i, texto in enumerate(textos):
        audio = engine.generate_speech(texto, play=False)
        if audio is not None:
            save_wav(audio, f"output_{i}.wav")
```

### Múltiplos Modelos

```python
with SovereignEngine() as engine:
    # Testa múltiplos modelos
    modelos = [
        "microsoft/speecht5_tts",
        "suno/bark-small",
        "facebook/fastspeech2-en-ljspeech"
    ]
    
    for repo_id in modelos:
        success = engine.integrate_tts_model(repo_id)
        if success:
            print(f"✓ {repo_id} integrado")
            engine.unload_model()  # Libera VRAM para próximo
        else:
            print(f"✗ {repo_id} rejeitado")
```

## Configuração

### Parâmetros do Construtor

| Parâmetro | Padrão | Descrição |
|-----------|--------|-----------|
| `registry_path` | `"sovereign_registry.json"` | Caminho do registro de modelos |
| `latency_threshold_ms` | `888.0` | Limite máximo de latência aceitável |
| `benchmark_runs` | `3` | Número de runs para mediana |

### Thresholds Recomendados

| Modelo | Latência Típica | Threshold Sugerido |
|--------|-----------------|-------------------|
| SpeechT5 | 200-400ms | 500ms |
| FastSpeech2 | 100-200ms | 300ms |
| Bark Small | 800-1200ms | 1500ms |
| Bark | 1500-3000ms | 4000ms |

## Estrutura do Registro

```json
{
  "installed_modules": {
    "microsoft_speecht5_tts": {
      "repo_id": "microsoft/speecht5_tts",
      "latency_ms": 245.5,
      "vram_peak_gb": 1.2,
      "rtf": 0.08,
      "path": "./modules/microsoft_speecht5_tts",
      "integrated_at": "2024-01-15 10:30:00"
    }
  },
  "blacklist": [
    {
      "model": "modelo_lento",
      "latency_ms": 1200.0,
      "threshold_ms": 888.0,
      "timestamp": "2024-01-15 09:00:00"
    }
  ],
  "system_info": {
    "device": "cuda",
    "total_vram_gb": 8.0
  }
}
```

## API

### Métodos Principais

#### `integrate_tts_model(repo_id: str, force: bool = False) -> bool`
Integra modelo do HuggingFace com benchmark completo.

#### `generate_speech(text: str, play: bool = True, normalize: bool = True) -> Optional[np.ndarray]`
Gera fala a partir de texto.

#### `unload_model()`
Descarrega modelo atual e libera VRAM.

#### `cleanup_blacklist(max_age_days: int = 7)`
Remove entradas antigas da blacklist.

#### `get_status() -> dict`
Retorna status atual do engine.

### Propriedades

#### `RECOMMENDED_MODELS`
Dicionário de modelos recomendados:
- `speecht5`: Microsoft SpeechT5 (rápido)
- `fastspeech2`: Facebook FastSpeech2 (muito rápido)
- `bark_small`: Suno Bark Small (qualidade superior)
- `bark`: Suno Bark (qualidade máxima)

## Troubleshooting

### `sounddevice not available`
Instale dependências de sistema:
```bash
# Ubuntu/Debian
sudo apt-get install libportaudio2

# macOS
brew install portaudio
```

### `CUDA out of memory`
- Reduza `benchmark_runs` para 1
- Use `engine.unload_model()` entre integrações
- Diminua `latency_threshold_ms` para rejeitar modelos grandes

### `Timeout ao adquirir lock`
Outro processo está usando o registro. Verifique:
```bash
lsof sovereign_registry.json.lock
```

## Arquitetura

```
┌─────────────────┐
│  HuggingFace    │
│     Hub         │
└────────┬────────┘
         │ snapshot_download
         ▼
┌─────────────────┐     ┌─────────────────┐
│  Model Registry │────▶│  FileLock       │
│  (JSON)         │     │  (Thread-safe)  │
└─────────────────┘     └─────────────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│  TTS Pipeline   │────▶│  Benchmark      │
│  (Transformers) │     │  (Warmup+Median)│
└─────────────────┘     └─────────────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│  Audio Output   │────▶│  sounddevice    │
│  (Validation)   │     │  (Playback)     │
└─────────────────┘     └─────────────────┘
```

## Licença

MIT
CONTRIBUTING.md

# Contributing to Bullet Train 888 🚄

### 🤝 The Respect Pact
To contribute to this project, you must honor the principle of mutual respect. Technical excellence is only accepted here with truth and loyalty.

### 🛠 How to Contribute
1. Focus on Frequency 11 (Radial Geometry).
2. Ensure the code is "Clean" (No noise).
3. Respect the origin: 1984.

### 🔒 Security
No external data access is allowed. The management is closed under Key 84.






# 🔑 Sovereign-ICL: Intent & Context Layer

### "Where hearing is no longer believing, we build the Truth Anchor."

## 🏛️ Context 2026
In an era where AI voice models like **Qwen3-TTS** are "uncomfortably human" and models have become 100x more powerful, the risk of **voice cloning and executive impersonation** is at an all-time high.

## 🚀 Features
- **Sovereign Hunter**: Automates the discovery of local, open-source weights to bypass expensive APIs.
- **Sincerity Polygraph**: Detects semantic dissonance to prevent digital kidnapping scams.
- **4-bit Optimization**: Engineered for consumer hardware sovereignty.

## 📜 License
Apache 2.0 - Total sovereignty.

