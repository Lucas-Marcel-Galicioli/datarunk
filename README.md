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
├── docs/
│   └── E4_business_analysis/
│       └── 04_business_analysis.docx  # Análise de negócio para cliente final
├── requirements.txt                   # Dependências Python
├── .gitignore
└── README.md
```

## Dados

### Fonte

Datasets sintéticos fornecidos pela DataRunk simulando o ambiente operacional de uma fintech.

### Datasets

**transactions.csv** (~5.000 linhas)
- Período: 01/01/2024 a 29/02/2024
- Colunas principais: transaction_id, timestamp, customer_id, amount, merchant_category, transaction_type, channel, device_fingerprint, status, is_flagged, fraud_confirmed, response_time_ms, state

**customers.csv** (~800 linhas)
- Colunas principais: customer_id, age, credit_limit, avg_monthly_spend, risk_score, region, account_tier, is_active

**alerts.csv**
- Alertas históricos do sistema legado

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
git clone https://github.com/Lucas-Marcel-Galicioli/datarunk.git
cd datarunk
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

### Preparando os Dados

Os datasets foram fornecidos pela DataRunk no desafio técnico e não estão versionados no repositório (por questões de tamanho e reprodutibilidade). Para executar o projeto:

1. Obtenha os arquivos CSV do desafio DataRunk:
   - `transactions.csv`
   - `customers.csv`
   - `alerts.csv`

2. Coloque-os no diretório `data/raw/`:
```bash
# Estrutura esperada após adicionar os dados
datarunk/
└── data/
    └── raw/
        ├── transactions.csv
        ├── customers.csv
        └── alerts.csv
```

3. Os notebooks carregarão automaticamente os dados desse local

## Execução

Os notebooks devem ser executados sequencialmente na seguinte ordem:

### 1. EDA e Engenharia de Features
```bash
jupyter notebook notebooks/01_eda_features.ipynb
```

Inclui:
- Análise de distribuição da variável alvo (fraude vs. legítima)
- Identificação e tratamento de nulos, outliers e inconsistências
- Análise de ao menos 2 relações bivariadas com a variável alvo
- Criação de 3 features derivadas:
  - Feature temporal: hora do dia e indicador de janela noturna (00h-05h)
  - Feature de anomalia: indicador de device_fingerprint ausente
  - Feature comportamental: desvio da transação vs. avg_monthly_spend do cliente

### 2. Modelo Baseline
```bash
jupyter notebook notebooks/02_baseline.ipynb
```

Inclui:
- Escolha de baseline: Regressão Logística ou sistema legado (is_flagged)
- Estratégia de particionamento justificada
- Tratamento de desbalanceamento de classes documentado
- Métricas obrigatórias: Precision, Recall, F1-Score, ROC-AUC, Matriz de Confusão
- Explicação clara sobre por que Accuracy é inadequada para detecção de fraude

### 3. Modelo Final
```bash
jupyter notebook notebooks/03_model_pipeline.ipynb
```

Inclui:
- Comparação de ao menos 2 algoritmos (ex: Random Forest vs. XGBoost)
- Pipeline reproduzível com sklearn Pipeline ou equivalente
- Avaliação completa com mesmas métricas do baseline + Curva Precision-Recall
- Threshold escolhido e justificado operacionalmente
- Feature importance conectada ao contexto de negócio

## Métricas e Avaliação

### Por que não Accuracy?

Com uma taxa de fraude de aproximadamente 3%, um modelo trivial que classifica todas as transações como legítimas alcançaria 97% de acurácia, mas não detectaria nenhuma fraude. Para o contexto de fraude financeira, as métricas relevantes são:

- **Precision**: Entre as fraudes que o modelo detectou, quantas são realmente fraudes? (minimiza falsos positivos)
- **Recall**: De todas as fraudes reais, quantas o modelo consegue detectar? (minimiza falsos negativos)
- **F1-Score**: Balanço harmônico entre precision e recall
- **ROC-AUC**: Performance geral do modelo em diferentes thresholds
- **Matriz de Confusão**: Detalha tipos de erro (FN, FP, TP, TN)

### Trade-off de Threshold

A escolha do ponto de corte afeta diretamente o negócio:
- **Threshold alto**: Maior precision (menos clientes bloqueados), menor recall (fraudes escapam)
- **Threshold baixo**: Maior recall (captura mais fraudes), menor precision (mais legítimos bloqueados)

A recomendação leva em conta o custo operacional de cada tipo de erro no contexto de negócio.

## Tratamento de Data Leakage

- `fraud_confirmed` nunca é utilizada como feature (é a variável alvo)
- `is_flagged` pode ser usada como feature (informação disponível antes da decisão)
- Todas as features derivadas utilizam apenas informações disponíveis no momento da transação

## Análise de Negócio

Os resultados foram traduzidos em linguagem de negócio e estão disponíveis em:

[docs/E4_business_analysis/04_business_analysis.docx](docs/E4_business_analysis/04_business_analysis.docx)

O documento cobre:
- Comparativo quantitativo baseline vs. modelo final
- Impacto financeiro estimado (economia mensal potencial de R$ 242.000)
- Trade-off de threshold com justificativa operacional
- Riscos conhecidos do modelo e estratégias de mitigação
- Próximos passos recomendados para implantação

## Tecnologias Utilizadas

- **Python 3.8+**
- **scikit-learn**: Preprocessing e modelos de baseline
- **XGBoost / LightGBM**: Modelos avançados para comparação
- **imbalanced-learn**: Tratamento de desbalanceamento de classes
- **pandas / numpy**: Manipulação e análise de dados
- **matplotlib / seaborn**: Visualizações

## Reprodutibilidade

O projeto garante reprodutibilidade através de:
- Random seed fixado em todos os modelos
- Pipeline encapsulado com sklearn Pipeline
- requirements.txt com versões específicas de dependências
- Documentação clara de decisões e trade-offs em cada notebook

## Contato e Dúvidas

Para dúvidas sobre o projeto, enviar ao recrutador DataRunk com assunto: `[DÚVIDA TESTE DS 5H] Seu Nome`

## Licença

Uso restrito ao processo seletivo DataRunk 2026.1

---

**Versão**: 2026.1 | **Última atualização**: 2026-05-03