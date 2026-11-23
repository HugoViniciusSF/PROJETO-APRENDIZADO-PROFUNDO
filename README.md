# Monitoramento de Vagas em Estacionamentos com Visão Computacional

Projeto prático da disciplina de Aprendizado Profundo (IMD / UFRN), desenvolvido por Gabriel Costa e Hugo Vinicius.

## Problema e objetivo

O sistema identifica vagas livres e ocupadas em estacionamentos usando câmeras de segurança estáticas já instaladas, sem necessidade de sensores físicos por vaga.

- **Motoristas**: acompanham o mapa público em tempo real com vagas livres em verde, ocupadas em vermelho e contagem disponível por setor.
- **Gestores e operadores**: visualizam a ocupação geral, consultam relatórios diários, desenham ou editam os polígonos das vagas no painel administrativo e registram erros para revisão do modelo.

## Como funciona

1. **Captura e pré-processamento**: os quadros de uma câmera IP estática (`RTSP >= 1280 x 720`) ou vídeo enviado são amostrados a cada 2 a 5 segundos, redimensionados para entrada quadrada `640 x 640 x 3` RGB e normalizados no intervalo `[0, 1]`.
2. **Detecção de veículos**: um modelo detector (`YOLO`) localiza veículos na cena e filtra predições com confiança `>= 0,50`.
3. **Classificação geométrica por vaga (ROI)**: cada vaga possui um polígono demarcado com coordenadas `(x, y)`. A vaga é classificada como **Ocupada (`1`)** quando há sobreposição `IoU > 0,35` ou quando o centroide da caixa está dentro do polígono; caso contrário, recebe estado **Livre (`0`)**.
4. **Estabilidade temporal**: mudanças de estado só são confirmadas após 3 a 5 quadros consecutivos estáveis, evitando marcações falsas causadas por carros em trânsito.

## Modelos, dados e experimentos

### Dados e divisão
- **Pesos iniciais**: transferência de aprendizado a partir do **MS-COCO**.
- **Dataset principal**: **PKLot** (12.417 imagens e 695.899 recortes) complementado por imagens e vídeos locais sob sol, sombra, chuva e período noturno.
- **Separação sem vazamento de dados**: `70%` treino, `15%` validação e `15%` teste divididos por dias distintos.
- **Data Augmentation**: ajustes de brilho e contraste, espelhamento horizontal e variações de escala no treinamento.

### Baselines e arquiteturas comparadas
- **Baseline clássico**: subtração de fundo e detecção de bordas Canny dentro dos polígonos para contrastar regras fixas com Aprendizado Profundo.
- **Baselines neurais por recorte**: CNN simples em PyTorch (`Conv2d`, `BatchNorm2d`, `ReLU`, `MaxPool2d`, `Dropout`, `Linear`) e classificadores baseados em **VGG** e **ResNet**.
- **Detectores principais**: família **YOLO** (`YOLOv8n` e `YOLO11n`) comparada ao detector baseado em Transformer **RT-DETR**, treinados com ajuste fino (`AdamW`, taxas de aprendizado `0,001` e `0,0001`, lotes `16` e `32`).

### Métricas e metas
- **Detecção**: Precisão, Recall, IoU e `mAP` (`mAP@0.5` e `mAP@[.5:.95]`).
- **Classificação por vaga**: Acurácia, Precisão, Recall, F1-score e Matriz de Confusão, priorizando menos falsos negativos (vaga ocupada exibida como livre gera congestionamento).
- **Meta de desempenho**: F1 por vaga `>= 92%` (alvo de `95%`) e latência total abaixo de `200 ms/quadro`.
- **Rastreamento**: todos os treinamentos e comparações são registrados no **MLflow** no Google Colab (GPU T4) com persistência no Google Drive.

## Tecnologias

- **Deep Learning e Visão Computacional**: Python, PyTorch, Ultralytics (YOLO / RT-DETR), OpenCV, Shapely
- **Rastreamento e Análise**: MLflow, scikit-learn, Matplotlib, NumPy
- **Aplicação e Dados**: FastAPI, SQLite / JSON (`vagas` e `historico_ocupacao`)

## Estrutura planejada

```text
.
├── notebooks/      # preparação do PKLot, baselines por recorte e treino YOLO/RT-DETR com MLflow
├── src/            # inferência, módulo geométrico (IoU/centroide), banco e API FastAPI
├── frontend/       # mapa público (vagas em verde/vermelho) e painel administrativo
└── README.md
```
