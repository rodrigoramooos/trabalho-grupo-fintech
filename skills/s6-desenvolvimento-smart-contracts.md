---
name: s6-desenvolvimento-smart-contracts
description: "Especialista em desenvolvimento de smart contracts para a NEAR Protocol em contexto académico de FinTech. Usar SEMPRE que a pergunta envolva: programação de contratos em Rust ou AssemblyScript para NEAR, NEAR SDK, NEAR CLI, deploy de contratos, testes unitários e de integração em NEAR, padrões de design de contratos (Fungible Tokens NEP-141, Non-Fungible Tokens NEP-171, Storage Management NEP-145), segurança de smart contracts, gestão de storage e gas em NEAR, cross-contract calls, promises em NEAR, migração de contratos, upgradeability, ferramentas de desenvolvimento (near-workspaces, sandbox), ou qualquer questão prática sobre como construir aplicações na NEAR. Activar também para comparações entre desenvolvimento em NEAR vs Ethereum/Solidity, boas práticas de auditoria de contratos, e arquitectura de dApps"
---


# NEAR Protocol — Desenvolvimento de Smart Contracts

## Comportamento Esperado

1. **Privilegiar exemplos de código concretos.** Sempre que possível, ilustrar conceitos com snippets em Rust ou AssemblyScript.
2. **Distinguir entre Rust e AssemblyScript.** Rust é a linguagem recomendada para produção; AssemblyScript é mais acessível para prototipagem.
3. **Contextualizar custos de storage e gas.** Todo o código em NEAR tem implicações de custo — mencioná-las sempre que relevante.
4. **Alertar para padrões de segurança.** Re-entrância, overflow, acesso não autorizado e má gestão de storage são riscos recorrentes.
5. **Terminologia técnica em inglês** quando não existe tradução estabelecida (ex: "cross-contract call", "promise", "view method").
6. **Não fornecer código para fins maliciosos.** Análise de vulnerabilidades é para fins académicos e de auditoria.

---

## Linguagens de Desenvolvimento

### Rust (Recomendado para Produção)
- Linguagem de sistemas com garantias de segurança de memória em tempo de compilação.
- Compilado para **WebAssembly (WASM)** — formato de execução dos contratos NEAR.
- Curva de aprendizagem mais elevada, mas produz contratos mais eficientes e seguros.
- SDK: `near-sdk-rs`

```rust
use near_sdk::{near, env, AccountId, NearToken};
use near_sdk::collections::UnorderedMap;

#[near(contract_state)]
pub struct Contrato {
    saldos: UnorderedMap<AccountId, u128>,
    proprietario: AccountId,
}

#[near]
impl Contrato {
    #[init]
    pub fn new(proprietario: AccountId) -> Self {
        Self {
            saldos: UnorderedMap::new(b"s"),
            proprietario,
        }
    }

    pub fn depositar(&mut self) {
        let conta = env::predecessor_account_id();
        let valor = env::attached_deposit().as_yoctonear();
        let saldo_actual = self.saldos.get(&conta).unwrap_or(0);
        self.saldos.insert(&conta, &(saldo_actual + valor));
    }

    pub fn consultar_saldo(&self, conta: AccountId) -> u128 {
        self.saldos.get(&conta).unwrap_or(0)
    }
}
```

### AssemblyScript (Prototipagem)
- Superset tipado de TypeScript, compilado para WASM.
- Sintaxe familiar para developers JavaScript/TypeScript.
- **Não recomendado para contratos com valor financeiro significativo** — menos garantias de segurança que Rust.
- SDK: `near-sdk-as` (descontinuado — migrar para Rust em produção)

---

## Tipos de Métodos

| Tipo | Decorador (Rust) | Acesso ao Estado | Custo de Gas | Casos de Uso |
|---|---|---|---|---|
| **View** | `#[handle_result]` / sem mutação | Só leitura | Gratuito (off-chain) | Consultas, leituras |
| **Call** | `&mut self` | Leitura + escrita | Gas necessário | Transacções, actualizações |
| **Init** | `#[init]` | Inicialização única | Gas necessário | Constructor do contrato |

---

## Storage em NEAR

### Modelo de Storage
- Em NEAR, o **contrato paga pelo storage** que utiliza — não o utilizador (por defeito).
- Custo: **1 NEAR por 100KB** de storage utilizado.
- O contrato deve manter NEAR suficiente para cobrir o seu storage — caso contrário é eliminado.

### Estruturas de Dados Persistentes

```rust
use near_sdk::collections::{UnorderedMap, LookupMap, Vector, LookupSet};

// LookupMap — chave-valor, O(1), sem iteração
let mapa: LookupMap<AccountId, u128> = LookupMap::new(b"m");

// UnorderedMap — chave-valor, O(1), com iteração (mais caro em storage)
let mapa_iteravel: UnorderedMap<AccountId, u128> = UnorderedMap::new(b"u");

// Vector — lista ordenada, acesso por índice
let lista: Vector<String> = Vector::new(b"v");

// LookupSet — conjunto sem duplicados, sem iteração
let conjunto: LookupSet<AccountId> = LookupSet::new(b"s");
```

> **Nota:** Os prefixos (`b"m"`, `b"u"`, etc.) são chaves de namespace no storage — devem ser únicos dentro do contrato.

### Storage Management (NEP-145)
- Standard para contratos que requerem que utilizadores depositem NEAR para cobrir o seu próprio storage.
- Padrão adoptado por tokens fungíveis (NEP-141) — utilizador regista-se e paga storage antes de receber tokens.

---

## Standards NEP (NEAR Enhancement Proposals)

### NEP-141 — Fungible Token
Equivalente ao ERC-20 do Ethereum.

```rust
// Métodos obrigatórios do standard NEP-141
pub fn ft_transfer(&mut self, receiver_id: AccountId, amount: U128, memo: Option<String>);
pub fn ft_transfer_call(&mut self, receiver_id: AccountId, amount: U128, memo: Option<String>, msg: String) -> PromiseOrValue<U128>;
pub fn ft_total_supply(&self) -> U128;
pub fn ft_balance_of(&self, account_id: AccountId) -> U128;
```

### NEP-171 — Non-Fungible Token
Equivalente ao ERC-721 do Ethereum.

```rust
// Métodos core NEP-171
pub fn nft_transfer(&mut self, receiver_id: AccountId, token_id: TokenId, approval_id: Option<u64>, memo: Option<String>);
pub fn nft_token(&self, token_id: TokenId) -> Option<Token>;
```

### NEP-145 — Storage Management
```rust
pub fn storage_deposit(&mut self, account_id: Option<AccountId>, registration_only: Option<bool>) -> StorageBalance;
pub fn storage_withdraw(&mut self, amount: Option<U128>) -> StorageBalance;
pub fn storage_unregister(&mut self, force: Option<bool>) -> bool;
pub fn storage_balance_bounds(&self) -> StorageBalanceBounds;
pub fn storage_balance_of(&self, account_id: AccountId) -> Option<StorageBalance>;
```

---

## Cross-Contract Calls e Promises

### Modelo de Execução Assíncrona
- NEAR usa um modelo **assíncrono baseado em Promises** para comunicação entre contratos.
- Não existe chamada síncrona entre contratos — toda a comunicação é assíncrona.

```rust
use near_sdk::{Promise, Gas};

const GAS_PARA_CALLBACK: Gas = Gas::from_tgas(10);

#[near]
impl MeuContrato {
    pub fn chamar_outro_contrato(&self, contrato_destino: AccountId) -> Promise {
        // Chama método no contrato destino
        Promise::new(contrato_destino)
            .function_call(
                "metodo_externo".to_string(),
                b"{}".to_vec(),
                NearToken::from_yoctonear(0),
                Gas::from_tgas(30),
            )
            .then(
                // Callback no contrato actual
                Self::ext(env::current_account_id())
                    .with_static_gas(GAS_PARA_CALLBACK)
                    .callback_resultado()
            )
    }

    #[private]
    pub fn callback_resultado(&self) -> String {
        if env::promise_result(0) == PromiseResult::Successful {
            "Sucesso".to_string()
        } else {
            "Falhou".to_string()
        }
    }
}
```

> **Padrão de segurança:** O decorator `#[private]` garante que o callback só pode ser chamado pelo próprio contrato.

---

## Gas em NEAR

### Unidades
- **1 TGas = 10¹² Gas units**
- Gas máximo por transacção: **300 TGas**
- Custo aproximado: **1 TGas ≈ 0,0001 NEAR**

### Estimativas Típicas

| Operação | Gas Estimado |
|---|---|
| Transferência NEAR simples | ~0,5 TGas |
| Chamada a view method | Gratuito (off-chain) |
| Deploy de contrato (~100KB) | ~10 TGas |
| Cross-contract call simples | ~20–50 TGas |
| Mint de NFT (NEP-171) | ~10–20 TGas |

---

## Ferramentas de Desenvolvimento

### NEAR CLI
```bash
# Instalar
npm install -g near-cli

# Autenticar
near login

# Deploy de contrato
near deploy --accountId meu-contrato.testnet --wasmFile ./target/wasm32-unknown-unknown/release/contrato.wasm

# Chamar método (call)
near call meu-contrato.testnet depositar '{}' --deposit 1 --accountId utilizador.testnet

# Chamar view method
near view meu-contrato.testnet consultar_saldo '{"conta": "utilizador.testnet"}'
```

### near-workspaces (Testes de Integração)
```rust
// Rust
use near_workspaces::{types::NearToken, Account, Contract};

#[tokio::test]
async fn teste_depositar() -> anyhow::Result<()> {
    let sandbox = near_workspaces::sandbox().await?;
    let wasm = std::fs::read("./target/wasm32-unknown-unknown/release/contrato.wasm")?;
    let contrato = sandbox.dev_deploy(&wasm).await?;
    let utilizador = sandbox.dev_create_account().await?;

    let resultado = utilizador
        .call(contrato.id(), "depositar")
        .deposit(NearToken::from_near(1))
        .transact()
        .await?;

    assert!(resultado.is_success());
    Ok(())
}
```

### Redes Disponíveis
| Rede | Uso | Faucet |
|---|---|---|
| **Mainnet** | Produção | N/A |
| **Testnet** | Desenvolvimento/testes | near.org/faucet |
| **Sandbox** | Testes locais (near-workspaces) | Automático |

---

## Segurança de Smart Contracts

### Vulnerabilidades Comuns

#### 1. Re-entrância
```rust
// VULNERÁVEL — transfere antes de actualizar estado
pub fn levantar(&mut self, valor: u128) {
    let saldo = self.saldos.get(&env::predecessor_account_id()).unwrap_or(0);
    Promise::new(env::predecessor_account_id()).transfer(NearToken::from_yoctonear(valor));
    self.saldos.insert(&env::predecessor_account_id(), &(saldo - valor)); // ← actualiza depois
}

// SEGURO — actualiza estado antes de transferir
pub fn levantar(&mut self, valor: u128) {
    let conta = env::predecessor_account_id();
    let saldo = self.saldos.get(&conta).unwrap_or(0);
    assert!(saldo >= valor, "Saldo insuficiente");
    self.saldos.insert(&conta, &(saldo - valor)); // ← actualiza primeiro
    Promise::new(conta).transfer(NearToken::from_yoctonear(valor));
}
```

#### 2. Overflow Aritmético
- Rust em modo `release` não faz panic em overflow por defeito — usar métodos `checked_add`, `checked_sub`, `saturating_add`.

#### 3. Acesso Não Autorizado
```rust
// Verificar sempre quem chama o método
pub fn metodo_admin(&mut self) {
    assert_eq!(
        env::predecessor_account_id(),
        self.proprietario,
        "Apenas o proprietário pode chamar este método"
    );
    // ...
}
```

#### 4. Má Gestão de Storage
- Contratos sem NEP-145 podem ser explorados com spam de registos — esgotando o NEAR do contrato para storage.

### Checklist de Auditoria
- [ ] Todos os métodos que modificam estado verificam autorização
- [ ] Estado actualizado antes de transferências externas (anti re-entrância)
- [ ] Aritmética protegida contra overflow
- [ ] Storage gerido com NEP-145 quando aplicável
- [ ] Callbacks marcados com `#[private]`
- [ ] Gas suficiente reservado para callbacks em cross-contract calls
- [ ] Testes de integração com near-workspaces cobrem casos extremos

---

## Upgradeability de Contratos

- Contratos NEAR podem ser actualizados (re-deploy) pela conta que os controla.
- **Migração de estado:** se a estrutura de dados muda, é necessário código de migração.

```rust
#[near]
impl MeuContrato {
    #[private]
    #[init(ignore_state)]
    pub fn migrar() -> Self {
        // Lê estado antigo e converte para novo formato
        let estado_antigo: EstadoAntigo = env::state_read().expect("Estado não encontrado");
        Self {
            novo_campo: estado_antigo.campo_antigo,
            campo_adicional: valor_defeito(),
        }
    }
}
```

---

## Limitações

- Snippets de código são ilustrativos — versões exactas de APIs podem variar com actualizações do `near-sdk-rs`.
- Questões de arquitectura de sharding e consenso são da competência da skill `s2-arquitetura-tecnica`.
- Questões de tokenomics e economia de mercado são da competência da skill `s1-tokenomics-mercado`.
- Questões de ecossistema e casos de uso FinTech são da competência da skill `s4-ecossistema-fintech`.