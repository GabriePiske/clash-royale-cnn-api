# Clash Classifier — CNN Inference API

API REST que identifica personagens do **Clash of Clans** em imagens usando um modelo CNN treinado no PyTorch. O modelo foi treinado no Google Colab e a inferência é executada localmente via Node.js + Python.

### Classes suportadas

| Personagem | Classe |
|------------|--------|
| 👺 Goblin  | `goblin` |
| 🤖 P.E.K.K.A | `pekka` |
| 🧙 Witch (Bruxa) | `witch` |
| 🧝 Wizard (Mago) | `wizard` |

Se a imagem não pertencer a nenhuma dessas classes com confiança acima de 80%, a API retorna `"desconhecido"`.

---

## Como funciona

```
Google Colab          Máquina local
─────────────         ──────────────────────────────────────
Treina a CNN    →     API Node.js recebe a imagem via HTTP
Exporta .pth    →     Worker Python carrega o .pth e faz inferência
                      Retorna JSON com a classe prevista
```

---

## Estrutura do projeto

```
cnn-node-api/
├── server.js              # API Node.js + worker Python embutido
├── package.json
├── requirements.txt
├── README.md
├── models_saved/
│   └── model.pth          # Modelo treinado (gerado no Colab)
├── clash_royale/
│   ├── train/             # Imagens de treino
│   │   ├── goblin/
│   │   ├── pekka/
│   │   ├── witch/
│   │   └── wizard/
│   └── test/              # Imagens de teste
│       ├── goblin/
│       ├── pekka/
│       ├── witch/
│       └── wizard/
└── frontend/
    ├── index.html         # Interface web
    └── README.md          # Documentação do endpoint
```

---

## Pré-requisitos

- Node.js instalado
- Python 3.12 instalado
- Modelo treinado no formato `.pth`

---

## Instalação

**1. Clone o repositório**

```bash
git clone https://github.com/GabriePiske/clash-royale-cnn-api.git
cd clash-royale-cnn-api
```

**2. Instale as dependências Node.js**

```bash
npm install
```

**3. Crie o ambiente virtual Python**

Windows CMD:
```cmd
python -m venv .venv
.venv\Scripts\activate.bat
```

Windows Git Bash:
```bash
python -m venv .venv
source .venv/Scripts/activate
```

Linux/macOS:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

**4. Instale as dependências Python**

```bash
python -m pip install torch torchvision pillow --index-url https://download.pytorch.org/whl/cpu
```

**5. Adicione o modelo treinado**

Copie o arquivo `.pth` gerado no Google Colab para a pasta `models_saved/` e renomeie para `model.pth`:

```
models_saved/model.pth
```

> A API não inicia sem esse arquivo.

---

## Executando o projeto

O projeto precisa de **dois terminais abertos ao mesmo tempo** — um para a API e outro para o frontend.

### Terminal 1 — API

Windows CMD:
```cmd
.venv\Scripts\activate.bat
npm start
```

Windows Git Bash:
```bash
source .venv/Scripts/activate
npm start
```

Linux/macOS:
```bash
source .venv/bin/activate
npm start
```

Saída esperada:

```
Inicializando API de inferência CNN...
Modelo esperado em: models_saved/model.pth

API iniciada com modelo carregado.
Dispositivo usado pelo PyTorch: cpu
Classes carregadas: ["goblin","pekka","witch","wizard"]
Servidor rodando em: http://localhost:3000
Endpoint de inferência: POST http://localhost:3000/infer
```

> Mantenha esse terminal aberto enquanto usar o projeto.

### Terminal 2 — Frontend

Abra um **novo terminal** e execute:

```bash
npx serve frontend
```

Acesse o endereço exibido no terminal (ex: `http://localhost:64491`) no navegador. A interface permite enviar uma imagem e visualizar o resultado da inferência.

---

## Testando via curl

Com a API rodando, abra um novo terminal e execute:

**Linux/macOS:**
```bash
curl -X POST http://localhost:3000/infer \
  -F "image=@./sua-imagem.jpg"
```

**Windows CMD/PowerShell:**
```cmd
curl.exe -X POST http://localhost:3000/infer -F "image=@./sua-imagem.png"
```

**Testando imagens da pasta de teste:**
```bash
curl.exe -X POST http://localhost:3000/infer -F "image=@./clash_royale/test/goblin/img01.png"
```

---

## Resposta da API

**Imagem reconhecida:**
```json
{
  "ok": true,
  "predictedClass": "goblin",
  "predictedIndex": 0,
  "confidence": 0.9956,
  "topPredictions": [
    { "class": "goblin", "index": 0, "confidence": 0.9956 },
    { "class": "wizard", "index": 3, "confidence": 0.0042 },
    { "class": "witch",  "index": 2, "confidence": 0.0000 }
  ]
}
```

**Imagem não reconhecida (confiança abaixo de 80%):**
```json
{
  "ok": true,
  "predictedClass": "desconhecido",
  "predictedIndex": 2,
  "confidence": 0.3812,
  "topPredictions": [...]
}
```

---

## Observações

- O modelo foi treinado no Google Colab com imagens das 4 classes acima.
- A arquitetura da CNN no `server.js` deve ser idêntica à usada no treinamento.
- Imagens muito diferentes do estilo de treino podem retornar `"desconhecido"`.
- As pastas `.venv/`, `node_modules/` e `.runtime/` não são incluídas no repositório, precisam ser criadas localmente seguindo os passos acima.
