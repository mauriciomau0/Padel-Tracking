# Rastreamento de Atletas de Pádel — Pipeline de Visão Computacional

Rastreamento automatizado de quatro atletas de pádel a partir de vídeo de transmissão, projetando suas posições na quadra em um mapa tático de vista zenital (top-down) por meio de estimação de pose e homografia.

https://github.com/user-attachments/assets/PLACEHOLDER_VIDEO_ID

> ⬆️ Substitua o link acima pelo URL real do vídeo após o upload no GitHub.
> Para embutir o vídeo: arraste o `.mp4` diretamente no editor do README no GitHub — ele gera o URL `assets` automaticamente.

---

## Problema

A análise tática no pádel depende de compreender como os quatro jogadores se movimentam e cobrem a quadra ao longo de uma partida. Tradicionalmente isso exige hardware especializado de tracking com múltiplas câmeras ou anotação manual — ambos caros e demorados.

Este projeto extrai as posições dos jogadores a partir de uma única câmera de transmissão convencional (fixa, ângulo oblíquo) e as mapeia para coordenadas reais da quadra, produzindo mapas de calor táticos e sobreposições de trajetória sem nenhum hardware customizado.

## Abordagem

O pipeline processa cada frame do vídeo em quatro estágios:

```
Vídeo → Extração de Frames → Detecção da Quadra → Estimação de Pose → Projeção por Homografia
                                                          ↓
                                                  Mapa Tático Zenital
```

**1 — Extração de Frames**
Decomposição do vídeo em frames individuais via OpenCV `VideoCapture`, preservando metadados de FPS e resolução.

**2 — Detecção da Quadra**
Segmentação da superfície azul da quadra usando thresholding de cor no espaço HSV após filtragem Gaussiana (kernel 35×35). O maior contorno é selecionado e reduzido a quatro pontos de canto via aproximação poligonal (`approxPolyDP`).

**3 — Estimação de Pose e Detecção de Jogadores**
Cada frame é mascarado para isolar a região da quadra (com padding), passando então por dois modelos em sequência:
- **Faster R-CNN** (ResNet-50 + FPN, pré-treinado no COCO) para detecção de pessoas com confiança > 0.3 e NMS
- **HRNet-W32** (top-down heatmap, COCO 256×192) para estimação de 17 keypoints por pessoa detectada

A posição no solo de cada jogador é calculada como: ponto médio dos quadris (eixo X) combinado com a média das posições dos pés (eixo Y), acrescido de um fator de correção de +15px para compensar o deslocamento dos keypoints em relação ao ponto real de contato com o chão.

**4 — Tracking e Projeção por Homografia**
Um algoritmo de atribuição por vizinho mais próximo mantém a identidade de cada jogador entre frames, baseado na distância euclidiana à última posição conhecida. Os quatro cantos detectados e as dimensões reais da quadra de pádel (10m × 20m) definem a matriz de homografia H, que projeta todas as posições da perspectiva da câmera para o plano zenital da quadra.

## Saída

O sistema gera duas saídas sincronizadas:
- **Vídeo anotado** — filmagem original com pontos de trajetória coloridos sobrepostos por jogador
- **Mapa zenital** — vista top-down da quadra mostrando os pontos de posição acumulados dos quatro jogadores, adequado para análise de cobertura tática

## Stack Tecnológico

| Camada | Ferramentas |
|--------|-------------|
| Linguagem | Python 3 |
| Visão Computacional | OpenCV (segmentação HSV, homografia, I/O de vídeo) |
| Detecção de Objetos | MMDetection — Faster R-CNN (ResNet-50 + FPN) |
| Estimação de Pose | MMPose — HRNet-W32 (top-down heatmap, COCO) |
| Deep Learning | PyTorch, TorchVision |
| Framework ML | MMEngine, MMCV |
| Infraestrutura | Google Colab (GPU CUDA), Google Drive |

## Como Usar

Abra o notebook no Google Colab com runtime GPU habilitado:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SEU_USUARIO/padel-tracking/blob/main/Padel_Tracking.ipynb)

1. Faça upload do vídeo da partida de pádel no Google Drive em `/My Drive/Padel/`
2. Execute todas as células sequencialmente — o notebook instala as bibliotecas necessárias (MMPose, MMDetection, MMCV)
3. A detecção da quadra é calibrada automaticamente no primeiro frame; verifique se os quatro cantos detectados estão corretos
4. O vídeo de saída é salvo em `/content/video/`

**Requisitos:** Google Colab com GPU T4 ou equivalente. Todas as dependências são instaladas inline.

## Limitações e Próximos Passos

- A detecção da quadra depende de iluminação estável e superfície azul; outras cores exigiriam recalibração dos thresholds HSV
- O tracker baseado em distância pode perder a identidade durante cruzamentos prolongados entre jogadores — integrar DeepSORT ou ByteTrack melhoraria a robustez
- O fator de correção dos pés é fixo em +15px; um valor adaptativo baseado na distância do jogador à câmera aumentaria a precisão
- Atualmente processa frames sequencialmente; inferência em batch melhoraria significativamente o throughput

## Licença

MIT
