# Rastreamento de Atletas de Padel

Rastreamento automatizado de quatro atletas de padel a partir de vídeo de transmissão, projetando suas posições na quadra em um mapa tático de vista zenital (top-down) por meio de estimação de pose e homografia.

[https://github.com/user-attachments/assets/PLACEHOLDER_VIDEO_ID](https://www.youtube.com/watch?v=fum6uSU7SPA&t=1s)

## Problema

Este projeto extrai as posições dos jogadores a partir de uma única câmera de transmissão convencional (fixa, ângulo oblíquo) e as mapeia em coordenadas reais da quadra, produzindo mapas de calor táticos e sobreposições de trajetória.

## Saída

O sistema gera duas saídas sincronizadas:
- **Vídeo anotado** — filmagem original com pontos de trajetória coloridos sobrepostos por jogador
- **Mapa zenital** — vista top-down da quadra mostrando os pontos de posição acumulados dos quatro jogadores, adequado para análise de cobertura tática

## Como Usar

Abra o notebook no Google Colab com GPU habilitado:

1. Faça upload do vídeo da partida de padel no Google Drive em `/My Drive/Padel/`
2. Execute todas as células sequencialmente — o notebook instala as bibliotecas necessárias (MMPose, MMDetection, MMCV)
3. A detecção da quadra é calibrada automaticamente no primeiro frame; verifique se os quatro cantos detectados estão corretos
4. O vídeo de saída é salvo em `/content/video/`
