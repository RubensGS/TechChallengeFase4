# Tech Challenge — Fase 4
## Monitoramento Multimodal de Pacientes com IA

Sistema de monitoramento contínuo de pacientes que combina análise de **áudio**, **vídeo** e **texto** para gerar alertas clínicos automáticos. Todos os modelos rodam **localmente**, sem dependência de serviços pagos em nuvem.

---

## Estrutura do projeto

```
fase4/
├── 01_audio_analysis.ipynb     # Análise de voz e transcrição
├── 02_video_analysis.ipynb     # Análise postural por vídeo
├── 03_fusao_multimodal.ipynb   # Fusão dos resultados e alerta final
├── data/
│   ├── ravdess/                # Dataset de áudio (gerado automaticamente)
│   ├── videos/                 # Vídeo baixado do YouTube
│   ├── relatorio_audio_p002.json
│   ├── relatorio_video_p001.json
│   └── relatorios/
│       ├── alerta_p001.json
│       └── relatorio_completo_p001.json
├── .env                        # Credenciais (não versionar)
├── .gitignore
└── README.md
```

---

## Ferramentas utilizadas

| Funcionalidade | Ferramenta | Substitui |
|---|---|---|
| Transcrição de áudio | Whisper (OpenAI) | Azure Speech to Text |
| NER médico | HuggingFace `d4data/biomedical-ner-all` | Azure Text Analytics |
| Análise de sentimento | HuggingFace `cardiffnlp/twitter-roberta-base-sentiment-latest` | Azure Text Analytics |
| Features vocais | librosa + praat-parselmouth | Azure Custom Speech |
| Detecção de pose | MediaPipe Pose | Azure Custom Vision |
| Detecção de objetos | YOLOv8 (ultralytics) | Azure Custom Vision |
| Download de vídeo | yt-dlp | — |
| Relatório clínico | Claude API (claude-opus-4-5) | — |

---

## Pré-requisitos

### Sistema
```bash
sudo apt install ffmpeg -y
```

### Python
```bash
pip install openai-whisper librosa praat-parselmouth \
    transformers torch \
    mediapipe==0.10.14 ultralytics opencv-python \
    yt-dlp kaggle \
    anthropic python-dotenv \
    matplotlib seaborn pandas numpy
```

---

## Configuração

Crie um arquivo `.env` na raiz do projeto:

```env
ANTHROPIC_API_KEY=sua_chave_aqui
KAGGLE_USERNAME=seu_usuario
KAGGLE_KEY=sua_chave_kaggle
```

> ⚠️ O arquivo `.env` já está no `.gitignore`. Nunca suba credenciais para o repositório.

---

## Como executar

Execute os notebooks **em ordem**:

**1. Análise de Áudio**
- Baixa o dataset RAVDESS via Kaggle API
- Extrai features vocais (MFCC, Jitter, Shimmer, HNR)
- Transcreve o áudio com Whisper
- Aplica NER biomédico e análise de sentimento
- Gera `data/relatorio_audio_p002.json`

**2. Análise de Vídeo**
- Baixa vídeo do YouTube via yt-dlp
- Detecta 33 keypoints corporais com MediaPipe Pose
- Detecta objetos e pessoas com YOLOv8
- Identifica anomalias posturais por limiares clínicos
- Gera `data/relatorio_video_p001.json`

**3. Fusão Multimodal**
- Carrega os JSONs dos notebooks anteriores
- Calcula score de risco ponderado (áudio 40%, texto 30%, vídeo 30%)
- Gera dashboard visual consolidado
- Produz relatório clínico em português via Claude API
- Gera alerta para equipe médica em `data/relatorios/`

---

## Lógica de risco

| Score | Nível | Ação |
|---|---|---|
| ≥ 60 | 🔴 Alto | Acionar equipe de plantão imediatamente |
| 30–59 | 🟡 Médio | Reavaliar em 24h |
| < 30 | 🟢 Baixo | Manter monitoramento padrão |

### Limiares clínicos utilizados

- **Jitter** > 1,04% → irregularidade vocal (Parkinson, disartria)
- **Shimmer** > 3,81% → instabilidade de amplitude (disfonia)
- **HNR** < 20 dB → voz rouca ou fatigada
- **Assimetria de joelhos** > 20° → compensação postural
- **Desnível de ombros/quadril** > 5% → desequilíbrio estrutural

---

## Dataset

**RAVDESS** — Ryerson Audio-Visual Database of Emotional Speech and Song  
Kaggle: [uwrfkaggler/ravdess-emotional-speech-audio](https://www.kaggle.com/datasets/uwrfkaggler/ravdess-emotional-speech-audio)  
1.440 gravações · 24 atores · 8 emoções

---

## Curso

PosTech FIAP — Machine Learning Engineering  
Tech Challenge Fase 4
