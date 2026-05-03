# FinSecure Fraud Detection Pipeline

Modelo preditivo de machine learning para detecção de fraudes em transações financeiras, desenvolvido como resposta ao desafio técnico DataRunk.

## Visão Geral

Este projeto implementa um pipeline end-to-end de detecção de fraude, partindo da exploração dos dados através da construção de um modelo final otimizado. O objetivo é substituir um sistema legado baseado em regras fixas por uma solução preditiva que reduz falsos positivos mantendo alta precisão na detecção.

### Contexto de Negócio

A FinSecure Pagamentos necessita melhorar seu sistema de antifraude para:
- Aumentar a precisão na detecção de transações fraudulentas
- Reduzir falsos positivos que geram atrito com clientes legítimos
- Quantificar o impacto financeiro da solução

## Estrutura do Projeto

```
.
├── notebooks/
│   ├── 01_eda_features.ipynb          # Exploração de dados e engenharia de features
│   ├── 02_baseline.ipynb              # Modelo de referência
│   └── 03_model_pipeline.ipynb        # Pipeline final e comparação de modelos
├── data/
│   ├── raw/
│   │   ├── transactions.csv           # Transações (5k registros)
│   │   ├── customers.csv              # Perfil de clientes (800 registros)
│   │   └── alerts.csv                 # Alertas históricos
│   └── processed/                     # Dados preprocessados (gerado durante execução)
├── src/
│   ├── preprocessing.py               # Transformações de dados
│   ├── models.py                      # Definição de modelos
│   └── utils.py                       # Funções auxiliares
├── requirements.txt                   # Dependências Python
├── .gitignore
└── README.md
```

## Dados

### Fonte

Dois datasets sintéticos fornecidos pela DataRunk simulando o ambiente operacional de uma fintech.

### Datasets

**transactions.csv** (~5.000 linhas)
- Período: 01/01/2024 a 29/02/2024
- Colunas principais: transaction_id, timestamp, customer_id, amount, merchant_category, transaction_type, channel, device_fingerprint, status, is_flagged, fraud_confirmed, response_time_ms, state

**customers.csv** (~800 linhas)
- Colunas principais: customer_id, age, credit_limit, avg_monthly_spend, risk_score, region, account_tier, is_active

**Particularidades dos dados:**
- Desbalanceamento acentuado de classes (fraude: ~3%)
- fraud_confirmed possui três estados: 1 (fraude confirmada), 0 (legítima), NULL (não investigada)
- Dados contêm ruídos intencionais: valores nulos, outliers e inconsistências
- device_fingerprint pode estar ausente (NULL)

## Configuração do Ambiente

### Pré-requisitos

- Python 3.8+
- pip ou conda

### Instalação

1. Clone o repositório:
```bash
git clone <seu-repositorio-url>
cd datarunk-fraud-detection
```

2. Crie um ambiente virtual:
```bash
python3 -m venv venv
source venv/bin/activate  # No Windows: venv\Scripts\activate
```

3. Instale as dependências:
```bash
pip install -r requirements.txt
```

## Execução

Os notebooks devem ser executados sequencialmente:

### 1. EDA e Engenharia de Features
```bash
jupyter notebook notebooks/01_eda_features.ipynb
```

Inclui:
- Análise de distribuição da variável alvo
- Identificação e tratamento de nulos, outliers e inconsistências
- Criação de ao menos 3 features derivadas:
  - Feature temporal (hora do dia / indicador noturno)
  - Feature de anomalia (ausência de device_fingerprint)
  - Feature comportamental (desvio de transação vs. avg_monthly_spend)

### 2. Modelo Baseline
```bash
jupyter notebook notebooks/02_baseline.ipynb
```

Inclui:
- Baseline com Regressão Logística ou sistema legado (is_flagged)
- Estratégia de particionamento (justificada)
- Tratamento de desbalanceamento de classes
- Métricas obrigatórias: Precision, Recall, F1-Score, ROC-AUC, Matriz de Confusão

### 3. Modelo Final
```bash
jupyter notebook notebooks/03_model_pipeline.ipynb
```

Inclui:
- Comparação de ao menos 2 algoritmos
- Pipeline reproduzível com sklearn Pipeline
- Avaliação completa com curva Precision-Recall
- Feature importance com conexão ao contexto de negócio

## Métricas e Avaliação

### Por que não Accuracy?

Com uma taxa de fraude de aproximadamente 3%, um modelo trivial que classifica todas as transações como legítimas alcançaria 97% de acurácia, mas não detectaria nenhuma fraude. Para o contexto de fraude financeira, as métricas relevantes são:

- **Precision**: Entre as fraudes que o modelo detectou, quantas são realmente fraudes? (minimiza falsos positivos)
- **Recall**: De todas as fraudes reais, quantas o modelo consegue detectar? (minimiza falsos negativos)
- **F1-Score**: Balanço harmônico entre precision e recall
- **ROC-AUC**: Performance geral do modelo em diferentes thresholds
- **Matriz de Confusão**: Detalha tipos de erro (FN, FP, TP, TN)

### Trade-off de Threshold

A escolha do ponto de corte afeta diretamente:
- **Threshold alto**: Maior precision (menos falsos positivos), menor recall (perde fraudes)
- **Threshold baixo**: Maior recall (detecta mais fraudes), menor precision (mais clientes legítimos bloqueados)

A recomendação leva em conta o custo operacional de cada tipo de erro no contexto de negócio.

## Tratamento de Data Leakage

- `fraud_confirmed` nunca é utilizada como feature (é a variável alvo)
- `is_flagged` pode ser usada como feature (informação disponível antes da decisão)
- Todas as features derivadas utilizam apenas informações disponíveis no momento da transação

## Análise de Negócio

O projeto inclui análise traduzindo os resultados em:
- Comparativo quantitativo (baseline vs. modelo final)
- Impacto financeiro estimado (economia mensal potencial)
- Trade-off de threshold com justificativa operacional
- Limitações do modelo e tipos de fraude que podem não ser capturados

## Tecnologias

- **Python 3.8+**
- **scikit-learn**: Preprocessing e modelos base
- **XGBoost / LightGBM**: Modelos avançados
- **imbalanced-learn**: Tratamento de desbalanceamento
- **pandas / numpy**: Manipulação de dados
- **matplotlib / seaborn**: Visualizações

## Reprodutibilidade

O projeto garante reprodutibilidade através de:
- Random seed fixado em todos os modelos
- Pipeline encapsulado com sklearn Pipeline
- requirements.txt com versões específicas
- Documentação clara de decisões em cada notebook

## Contato e Questões

Para dúvidas sobre o projeto, enviar ao recrutador DataRunk com assunto: `[DÚVIDA TESTE DS 5H] Seu Nome`

## Licença

Uso restrito ao processo seletivo DataRunk 2026.1

---

**Versão**: 2026.1 | **Última atualização**: 2026-05-03