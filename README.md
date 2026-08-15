# FinSight AI

Assistente Inteligente de Saúde Financeira desenvolvido no **Hackathon ONE G9 Brasil 2026** (Projeto Finance AI).

O sistema utiliza Machine Learning para classificar transações automaticamente em 7 categorias, calcular um **Score de Saúde Financeira (0–1000)** e gerar recomendações personalizadas. O perfil financeiro é classificado como **Saudável**, **Em Observação** ou **Em Risco**.

> 🏆 Equipe: G9-BR-TEAM-20  
> ☁️ Infraestrutura: Oracle Cloud Infrastructure (OCI) Always Free  
> 🤖 ML: TF-IDF + Regressão Logística (categorias) | Random Forest (perfil)

---

## 🎯 Funcionalidades

| Recurso | Descrição |
|---------|-----------|
| 🏷️ Categorização Automática | Classifica despesas em 7 categorias com TF-IDF + ML |
| 📊 Score de Saúde Financeira | Escala 0–1000 com explicações |
| 🔮 Diagnóstico Preditivo | Perfil: Saudável / Em Observação / Em Risco |
| 💡 Recomendações | Dicas contextualizadas por perfil e gastos |
| ⚠️ Alertas Inteligentes | Notificações quando metas são ultrapassadas |
| 📈 Dashboard Visual | Gráficos interativos e evolução do score |
| 🌍 Multi-idioma | PT / EN / ES |
| 📤 Análise em Lote (CSV) | Upload de CSV para processamento massivo |
| 🧪 Simulador de Cenários | Teste "E se...?" sem salvar no histórico |
| 📄 Exportação de Relatório | Relatório HTML pronto para impressão |

---

## 🚀 Deploy na OCI (Recomendado)

Com uma VM Ubuntu na OCI (porta 80 liberada no Security List):

```bash
scp -r BACK-END FRONT-END VM_OCI requirements.txt ubuntu@SEU_IP:/home/ubuntu/finsight-ai/
ssh ubuntu@SEU_IP
cd finsight-ai/VM_OCI
bash ATUALIZAR_VM_v4.sh
```

O script executa automaticamente: backup, instalação de dependências, sync dos modelos do OCI Object Storage, configuração do NGINX + systemd e health check.

Acesse `http://SEU_IP` no navegador.

> ⚠️ O sync dos modelos `.pkl` requer Instance Principals configurado na VM. Se não estiver disponível, o sistema ativa fallback heurístico e continua funcionando.

---

## 🏃 Rodar Localmente

**Pré-requisitos:** Python 3.10+

### 1. Clone o repositório

```bash
git clone https://github.com/No-Country-simulation/G9-BR-TEAM-20.git
cd G9-BR-TEAM-20
```

> Se não tiver Git, baixe o ZIP no GitHub e extraia.

### 2. Crie o ambiente virtual

**Windows:**
```cmd
python -m venv venv
venv\Scriptsctivate
```

**Linux/Mac:**
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Instale as dependências

```bash
pip install -r requirements.txt
```

### 4. Prepare os modelos de ML (opcional, melhora precisão)

Copie os arquivos `.pkl` da pasta `MODELOS/` para a pasta de modelos do sistema:

**Windows:**
```cmd
mkdir %USERPROFILE%insight-ai\models
xcopy MODELOS\*.pkl %USERPROFILE%insight-ai\models```

**Linux/Mac:**
```bash
mkdir -p ~/finsight-ai/models
cp MODELOS/*.pkl ~/finsight-ai/models/
```

> Sem os modelos, a aplicação funciona com regras de palavras-chave (fallback heurístico).

### 5. Configure o frontend para apontar para o backend local

Abra `FRONT-END/index.html` em um editor de texto, localize a linha:

```js
const API_URL = window.location.origin;
```

E troque para:

```js
const API_URL = "http://localhost:8000";
```

Salve o arquivo.

### 6. Inicie o backend

```bash
cd BACK-END
uvicorn main:app --host 0.0.0.0 --port 8000
```

Deixe esse terminal **aberto**.

### 7. Inicie o frontend

Abra um **novo terminal** e execute:

```bash
cd FRONT-END
python -m http.server 8080
```

### 8. Acesse no navegador

Abra: `http://localhost:8080`

---

## 📡 API Endpoints

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `GET` | `/health` | Status do sistema |
| `POST` | `/analise-financeira` | Análise completa + salva no histórico |
| `POST` | `/simular` | Simula cenário sem salvar |
| `GET` | `/classificar` | Classifica uma única transação |
| `POST` | `/analise-batch-csv` | Processa CSV em lote |
| `GET` | `/historico` | Lista análises anteriores |
| `GET` | `/relatorio/{id}` | Obtém relatório específico |
| `DELETE` | `/historico/{id}` | Remove análise |
| `POST` | `/historico/limpar` | Limpa todo o histórico |
| `POST` | `/reload-models` | Recarrega modelos sem reiniciar |

### Exemplo de requisição

```bash
curl -X POST http://localhost:8000/analise-financeira \
  -H "Content-Type: application/json" \
  -d '{
    "renda_mensal": 4500,
    "nivel_endividamento": 25,
    "frequencia_poupanca": "Media",
    "transacoes": [
      {"descricao": "Supermercado", "valor": 420},
      {"descricao": "Combustivel", "valor": 300}
    ],
    "metas": {"alimentacao": 600}
  }'
```

### Exemplo de resposta

```json
{
  "perfil_financeiro": "Saudavel",
  "probabilidade": 0.8316,
  "score_saude": 825,
  "resumo_gastos": {
    "alimentacao": 420.0,
    "transporte": 300.0,
    "saude": 0.0,
    "moradia": 0.0,
    "educacao": 0.0,
    "lazer": 0.0,
    "servicos": 0.0
  },
  "recomendacoes": [
    "Continue mantendo hábitos saudáveis de organização financeira",
    "Considere diversificar investimentos em renda fixa e variável",
    "Avalie metas de longo prazo (aposentadoria, imóvel, intercâmbio)",
    "Com sua margem confortável, estude opções de independência financeira"
  ],
  "alertas": [],
  "explicabilidade": [
    "Endividamento em 25% (ideal)",
    "Poupança classificada como Media",
    "Gastos consomem 16.0% da renda (saudável)",
    "Maior categoria de gasto: alimentacao (R$ 420.00)"
  ],
  "total_gasto": 720.0,
  "percentual_renda_gasta": 16.0,
  "comprometimento_gastos": 41.0,
  "transacoes_classificadas": [
    {"descricao": "Supermercado", "valor": 420.0, "categoria": "alimentacao", "confianca": 0.9825},
    {"descricao": "Combustivel", "valor": 300.0, "categoria": "transporte", "confianca": 0.8909}
  ]
}
```

---

## 🏗️ Arquitetura

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│   Usuário   │────▶│   NGINX     │────▶│  Frontend       │
│  (Browser)  │     │  (Porta 80) │     │  (HTML/CSS/JS)  │
└─────────────┘     └─────────────┘     └─────────────────┘
                                               │
                                               ▼
                                        ┌─────────────────┐
                                        │  FastAPI        │
                                        │  (Porta 8000)   │
                                        └─────────────────┘
                                               │
                          ┌────────────────────┼────────────────────┐
                          ▼                    ▼                    ▼
                   ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐
                   │   SQLite    │    │   Modelos   │    │  OCI Object     │
                   │  (Histórico)│    │   ML (.pkl) │    │  Storage        │
                   └─────────────┘    └─────────────┘    └─────────────────┘

```

| Camada | Tecnologia |
|--------|-----------|
| Frontend | HTML5, Tailwind CSS, Chart.js, Lucide Icons |
| Backend | Python 3.10, FastAPI, Uvicorn |
| ML | scikit-learn, pandas, numpy, joblib |
| Banco de Dados | SQLite |
| Infraestrutura | OCI VM (Ubuntu 24.04), NGINX |
| Storage | OCI Object Storage (modelos .pkl) |

---

## 📁 Estrutura do Projeto

```
G9-BR-TEAM-20/
├── BACK-END/
│   ├── main.py                 # Backend FastAPI
│   └── (models/ e data/ criados em ~/finsight-ai/)
├── DOCS/
│   ├── FinSight_AI_Documentacao_Tecnica_Completa.docx    # Documentação
│   └── documentacao.html
├── FRONT-END/
│   └── index.html              # Frontend completo
├── datasets/                   # Dados de treinamento
├── EDA__/                      # Notebooks de análise exploratória
├── MODELOS/                    # Artefatos ML (.pkl)
│   ├── vetorizador_tfidf.pkl
│   ├── modelo_categoria_producao.pkl
│   ├── codificador_categorias.pkl
│   ├── modelo_perfil_producao.pkl
│   └── codificador_perfil.pkl
├── VM_OCI/
│   └── ATUALIZAR_VM_v4.sh      # Script de deploy automático
├── requirements.txt            # Dependências Python
└── README.md                   # Este arquivo
```

---

## 📊 Modelos de Machine Learning

| Modelo | Algoritmo | Arquivo |
|--------|-----------|---------|
| Vetorização de Texto | TF-IDF | `vetorizador_tfidf.pkl` |
| Classificação de Categorias | Regressão Logística Multinomial | `modelo_categoria_producao.pkl` |
| Classificação de Perfil | Random Forest (200 estimadores) | `modelo_perfil_producao.pkl` |

> 💡 **Fallback Heurístico:** Se os modelos não estiverem disponíveis, o sistema ativa regras de palavras-chave inteligentes. A aplicação nunca para.

---

## 🆘 Troubleshooting

| Problema | Solução |
|----------|---------|
| `Could not import module "main"` | Certifique-se de estar na pasta `BACK-END/` ao rodar `uvicorn` |
| Frontend não conecta à API | Edite `API_URL` em `FRONT-END/index.html` para `http://localhost:8000` |
| `mkdir -p` dá erro no Windows | Use `mkdir %USERPROFILE%insight-ai\models` |
| Health check falha | `sudo journalctl -u finsight -f` (OCI) ou verifique o terminal do uvicorn |
| Modelos não carregam | Confirme se os `.pkl` estão em `~/finsight-ai/models/` |
| Porta 80 bloqueada | Verifique o Security List da VM na OCI |
| VM trava | O script cria SWAP de 2GB automaticamente |

---

## 👥 Equipe

| Nome | Função | LinkedIn | GitHub |
|------|--------|----------|--------|
| Fabio C. Zinetti | Data Scientist | [LinkedIn](https://www.linkedin.com/in/fabiozinetti) | [GitHub](https://github.com/fabinhoz) |
| João P. R. Deodato | Data Analyst | [LinkedIn](https://www.linkedin.com/in/jpdeodato) | [GitHub](https://github.com/jpdeodato) |
| Edson H. F. da Silva | Data Analyst | [LinkedIn](https://www.linkedin.com/in/henriquesilvatech) | [GitHub](https://github.com/86HenriqueSilva) |
| Luciano R. da Silva | Back-End | [LinkedIn](https://www.linkedin.com/in/luciano-ribeiro-da-silva-26a2802a6) | [GitHub](https://github.com/siilverado) |

---

<p align="center"><sub>FinSight AI v5.0 — Hackathon OCI 2026 🚀</sub></p>
<p align="center">
  <img src="https://img.shields.io/badge/version-6.0-blue?style=flat-square" alt="Version">
  <img src="https://img.shields.io/badge/python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/fastapi-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/oci-F80000?style=flat-square&logo=oracle&logoColor=white" alt="OCI">
</p>

