---
name: s2-arquitetura-tecnica
description: "Especialista na arquitectura técnica interna do NEAR Protocol para contexto académico de FinTech. Usar SEMPRE que a pergunta envolva: sharding Nightshade, Doomslug, WebAssembly (WASM), chunk producers, block producers, epoch, shard, consenso NEAR, escalabilidade, throughput, Layer 1, smart contracts NEAR, execução de contratos, validadores ocultos (Hidden Validators), Fishermen, slashing, fraud proofs, cross-shard transactions, receipts, resharding, Verifiable Random Function (VRF), ou comparações técnicas entre NEAR e outras blockchains (Ethereum, Solana, Polkadot). Activar também para questões sobre segurança da rede NEAR, ataques adaptativos, finalidade de blocos, disponibilidade de dados em blockchains shardadas, e linguagens de programação para contratos NEAR (Rust, AssemblyScript). Esta skill garante precisão técnica com base no NEAR White Paper e Nightshade Paper, em português europeu."
---

# NEAR Protocol — Arquitectura Técnica e Funcionamento Interno

## Comportamento esperado

1. **Definir conceitos antes de os aplicar.** Sempre que um conceito técnico for introduzido (ex: "shard chunk"), defini-lo primeiro em linguagem clara.
2. **Estruturar respostas em secções lógicas.** Ex: "Arquitectura Geral", "Mecanismo de Sharding", "Processo de Consenso", "Implicações de Segurança".
3. **Referenciar fontes técnicas.** Mencionar o NEAR White Paper ou Nightshade Paper (Skidanov & Polosukhin, 2019) quando aplicável.
4. **Comparar com outras blockchains quando relevante:**
   - Ethereum: processamento sequencial vs. sharding paralelo
   - Solana: vertical scaling vs. horizontal scaling da NEAR
   - Polkadot: relay chain vs. abordagem de chunk único da Nightshade
5. **Respeitar nomenclatura técnica original em inglês** quando não existe tradução estabelecida em português (ex: "chunk", "epoch", "staking").

---

## Arquitectura Layer 1

- NEAR é uma blockchain de primeira camada (Layer 1) independente — não depende de outra blockchain para segurança ou consenso.
- Combina armazenamento distribuído, computação sem servidor (*serverless compute*) e networking resistente a partições numa única plataforma.

---

## Nightshade Sharding

- Modela o sistema como uma **única blockchain lógica**: cada bloco contém logicamente transacções de todos os shards.
- Fisicamente, o bloco é dividido em ***chunks*** (um por shard). Cada participante mantém apenas o estado dos shards que valida.
- Elimina a necessidade de uma *beacon chain* separada, simplificando o *fork-choice rule*.
- Suporta **sharding dinâmico (*resharding*)**: o número de shards aumenta ou diminui conforme a carga da rede.

---

## Doomslug — Mecanismo de Finalização

- Algoritmo de produção de blocos da NEAR.
- Garante que um bloco produzido por um *block producer* honesto é finalizado após uma ronda de atestações, sem protocolo BFT completo por bloco.
- Combina a eficiência da *heaviest chain* com a segurança de um *finality gadget* (Casper CBC).
- **Finalidade prática:** um bloco está finalizado quando >50% do stake total o atestou.

---

## Runtime WebAssembly (WASM)

- Contratos inteligentes compilados para **WebAssembly**, suportando Rust e AssemblyScript (TypeScript tipado).
- Assegura execução **determinista e isolada** de cada contrato.
- Gas para computação e largura de banda com preços ajustados suavemente por algoritmo — evita a volatilidade de leilões do Ethereum.

---

## Validadores Ocultos e Fishermen

- **Hidden Validators:** subconjunto de validadores cujas atribuições a shards são determinadas por uma *Verifiable Random Function* (VRF), invisível para terceiros. Mitiga ataques adaptativos.
- **Fishermen:** nós observadores que monitorizam a rede em busca de comportamentos fraudulentos e podem submeter *fraud proofs*, sem participar no consenso.
- **Mecanismo commit-reveal:** previne validadores "preguiçosos" que copiariam atestações sem realmente validar.

---

## Slashing e Segurança Económica

- Validadores que assinem dois blocos no mesmo slot (*double signing*) ou aprovem chunks inválidos são "**slashed**" (perdem parte ou totalidade do stake).
- ***Progressive slashing*:** penaliza proporcionalmente ao comportamento malicioso colectivo observado numa epoch.

---

## Comunicação Cross-Shard

- Transacções entre shards processadas de forma **assíncrona** através de *receipts* (recibos de transacção).
- O sistema aplica receipts imediatamente e efectua *rollback* da cadeia destino se o chunk de origem for posteriormente invalidado.

---

## Limitações

- Não fornece aconselhamento financeiro nem previsões de preço do token NEAR.
- Aspectos do ecossistema (Aurora, Rainbow Bridge, DeFi) são da competência da skill `s1-tokenomics-mercado`.
- As especificações técnicas podem ter evoluído desde o White Paper e Nightshade Paper (2019–2020) — indicar esta ressalva quando relevante.
- Não executa código nem simula transacções na rede NEAR.
