# 🚀 CRE Bootcamp - Dia 2: Prediction Market

## 📋 Resumo
Neste dia construímos um **Mercado de Previsão** completo usando Chainlink CRE, com smart contracts em Solidity e workflows em TypeScript.

---

## 🔑 Informações do Projeto

| Item | Valor |
|------|-------|
| **Contrato PredictionMarket (Sepolia)** | `0xf548d2EDc1F480C9a6f23De83A516d71bB116860` |
| **MockKeystoneForwarder (Sepolia)** | `0x15fc6ae953e024d975e77382eeec56a9101f9f88` |
| **Transaction Hash (CreateMarket)** | `0x8fb43da6e62a07d95ab91ab976b15a5eda244244d3207574ca036409fae3df01` |
| **Rede** | Ethereum Sepolia Testnet |

---

## 🏗️ Estrutura do Projeto

```
prediction-market/
├── .env                          # Variáveis de ambiente (não commitar!)
├── project.yaml                  # Configurações do projeto CRE
├── secrets.yaml                  # Mapeamento de variáveis secretas CRE
├── my-workflow/                  # Diretório do workflow CRE
│   ├── workflow.yaml             # Configurações do workflow
│   ├── main.ts                   # Ponto de entrada do workflow
│   ├── httpCallback.ts           # Lógica do HTTP Trigger
│   ├── config.staging.json       # Configuração para simulação
│   ├── package.json              # Dependências Node.js
│   └── tsconfig.json             # Configuração TypeScript
└── contracts/                    # Projeto Foundry
    ├── foundry.toml              # Configuração do Foundry
    ├── src/
    │   ├── PredictionMarket.sol  # Contrato principal
    │   └── interfaces/
    │       ├── IReceiver.sol     # Interface do receiver
    │       └── ReceiverTemplate.sol # Template abstrato
    └── test/                     # Testes (opcional)
```

---

## ⚙️ Passo 1: Configuração do Projeto CRE

### Inicializar o projeto
```bash
cre init
# Nome: prediction-market
# Template: TS (TypeScript)
# Starter: Hello World TS
# Workflow name: my-workflow (Enter)
```

### Instalar dependências
```bash
cd prediction-market
bun install --cwd ./my-workflow
```

### Configurar o `.env`
```env
CRE_ETH_PRIVATE_KEY=SUA_CHAVE_PRIVADA_AQUI
CRE_TARGET=staging-settings
GEMINI_API_KEY_VAR=SUA_CHAVE_GEMINI_AQUI
```

> ⚠️ **Nunca commite o arquivo `.env`!** Ele já está no `.gitignore`.

---

## 🏛️ Passo 2: Configuração do Projeto Foundry

### Criar e configurar o projeto
```bash
# Na pasta prediction-market
forge init contracts
cd contracts
forge install OpenZeppelin/openzeppelin-contracts
mkdir -p src/interfaces
```

### `foundry.toml`
```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
remappings = [
    "@openzeppelin/=lib/openzeppelin-contracts/"
]
```

---

## 📄 Passo 3: Smart Contracts

### `src/interfaces/IReceiver.sol`
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {IERC165} from "@openzeppelin/contracts/utils/introspection/IERC165.sol";

interface IReceiver is IERC165 {
    function onReport(bytes calldata metadata, bytes calldata report) external;
}
```

### `src/interfaces/ReceiverTemplate.sol`
Contrato abstrato que fornece:
- ✅ Validação do endereço do forwarder
- ✅ Validação opcional do workflow
- ✅ Suporte ERC165
- ✅ Utilitários de decodificação de metadados

> Ver código completo em `src/interfaces/ReceiverTemplate.sol`

### `src/PredictionMarket.sol`
Contrato principal com 4 ações:
1. **createMarket** → Criar um mercado com pergunta Sim/Não
2. **predict** → Apostar ETH em Sim ou Não
3. **requestSettlement** → Solicitar liquidação (dispara evento para o CRE)
4. **claim** → Resgatar ganhos após liquidação

---

## 🔨 Passo 4: Compilar e Fazer Deploy

### Compilar
```bash
# Na pasta contracts
forge build
# Resultado esperado: Compiler run successful!
```

### Deploy na Sepolia
```bash
# Na pasta contracts
source ../.env

forge create src/PredictionMarket.sol:PredictionMarket \
  --rpc-url "https://ethereum-sepolia-rpc.publicnode.com" \
  --private-key $CRE_ETH_PRIVATE_KEY \
  --broadcast \
  --constructor-args 0x15fc6ae953e024d975e77382eeec56a9101f9f88
```

### Resultado do Deploy
```
Deployer: 0x93Da27ecDE39B1f0c2b9DC9F7c0F544D2368AAda
Deployed to: 0xf548d2EDc1F480C9a6f23De83A516d71bB116860
Transaction hash: 0x2d06a0c04f03c835e6a0c2b7e91b795507cb1629e06c849b19535c35184eaa6d
```

---

## ⚙️ Passo 5: Configurar o Workflow

### `my-workflow/config.staging.json`
```json
{
  "geminiModel": "gemini-2.0-flash",
  "evms": [
    {
      "marketAddress": "0xf548d2EDc1F480C9a6f23De83A516d71bB116860",
      "chainSelectorName": "ethereum-testnet-sepolia",
      "gasLimit": "500000"
    }
  ]
}
```

---

## 📝 Passo 6: HTTP Trigger Workflow

### `my-workflow/httpCallback.ts`
Workflow completo com 6 etapas:
1. **Parse e validação** do payload HTTP
2. **Configuração da rede** EVM (Sepolia)
3. **Codificação** dos dados em ABI
4. **Geração** do relatório CRE assinado
5. **Escrita** no smart contract via `writeReport`
6. **Verificação** do resultado e retorno do tx hash

### `my-workflow/main.ts`
Registra o HTTP Trigger e conecta ao `onHttpTrigger`.

---

## 🧪 Passo 7: Simulações

### Simulação simples (sem broadcast)
```bash
# Na pasta prediction-market
cre workflow simulate my-workflow
# Payload: {"question": "Will Argentina win the 2022 World Cup?"}
```

### Simulação com broadcast (escreve na blockchain)
```bash
cre workflow simulate my-workflow --broadcast
# Payload: {"question": "Will Argentina win the 2022 World Cup?"}
```

### Resultado esperado
```
[Step 1] Received market question: "Will Argentina win the 2022 World Cup?"
[Step 2] Target chain: ethereum-testnet-sepolia
[Step 2] Contract address: 0xf548d2EDc1F480C9a6f23De83A516d71bB116860
[Step 3] Encoding market data...
[Step 4] Generating CRE report...
[Step 5] Writing to contract: 0xf548d2EDc1F480C9a6f23De83A516d71bB116860
[Step 6] ✓ Transaction successful: 0x8fb43da6e62a07d95ab91ab976b15a5eda244244d3207574ca036409fae3df01

Workflow Simulation Result:
"0x8fb43da6e62a07d95ab91ab976b15a5eda244244d3207574ca036409fae3df01"
```

---

## ✅ Passo 8: Verificar o Mercado na Blockchain

```bash
export MARKET_ADDRESS=0xf548d2EDc1F480C9a6f23De83A516d71bB116860

cast call $MARKET_ADDRESS \
  "getMarket(uint256) returns ((address,uint48,uint48,bool,uint16,uint8,uint256,uint256,string))" \
  0 \
  --rpc-url "https://ethereum-sepolia-rpc.publicnode.com"
```

### Resultado
```
(0x15fC6ae953E024d975e77382eEeC56A9101f9F88, 1776909912, 0, false, 0, 0, 0, 0, "Will Argentina win the 2022 World Cup?")
```

---

## 🔄 Fluxo Completo do Sistema

```
HTTP Request
    ↓
CRE Workflow (httpCallback.ts)
    ↓
KeystoneForwarder (valida assinaturas)
    ↓
PredictionMarket.onReport()
    ↓
_processReport()
    ↓
createMarket() → Mercado criado na blockchain! ✅
```

---

## 📚 Conceitos Aprendidos

| Conceito | Descrição |
|----------|-----------|
| **IReceiver** | Interface que o contrato deve implementar para receber dados do CRE |
| **ReceiverTemplate** | Contrato abstrato com segurança e validações prontas |
| **HTTP Trigger** | Dispara o workflow via requisição HTTP |
| **EVM Write** | Escreve dados em smart contracts via relatório assinado |
| **KeystoneForwarder** | Valida assinaturas e entrega dados ao contrato |
| **forge build** | Compila os contratos Solidity |
| **forge create** | Faz deploy do contrato na blockchain |
| **cast call** | Lê dados de um smart contract |

---

## 🔗 Links Úteis

- [Bootcamp CRE Dia 2](https://smartcontractkit.github.io/cre-bootcamp-2026/pt/day-2/)
- [Sepolia Etherscan - Contrato](https://sepolia.etherscan.io/address/0xf548d2EDc1F480C9a6f23De83A516d71bB116860)
- [MockKeystoneForwarder](https://sepolia.etherscan.io/address/0x15fc6ae953e024d975e77382eeec56a9101f9f88#code)
- [Google AI Studio (Gemini)](https://aistudio.google.com/app/apikey)
- [Foundry Docs](https://www.getfoundry.sh)

---

*Manual criado durante o CRE Bootcamp 2026 - Dia 2*
