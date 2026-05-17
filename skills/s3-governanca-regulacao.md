---
name: s3-governanca-regulacao
description: "Especialista em governança, regulação e compliance de criptoativos no contexto do NEAR Protocol para FinTech académica. Usar SEMPRE que a pergunta envolva: governança NEAR, NEAR Foundation, Reference Maintainer, slashing, double signing, invalid chunks, penalizações de validadores, segurança da rede, ataques adaptativos, Hidden Validators, VRF (Verifiable Random Function), Fishermen, fraud proofs, regulação cripto (MiCA, SEC, ESMA, Banco de Portugal), classificação jurídica de tokens, compliance em DeFi, KYC/AML em blockchain, enquadramento legal de smart contracts, política de privacidade em redes públicas, ou questões sobre responsabilidade legal em protocolos descentralizados. Activar também para questões sobre riscos sistémicos, descentralização vs regulação, e o papel das fundações em protocolos open-source."
---

# NEAR Protocol — Governança, Regulação e Compliance

## Comportamento Esperado

1. **Distinguir claramente governança técnica de governança institucional.** A governança on-chain (parâmetros de protocolo) e a governança off-chain (NEAR Foundation, Reference Maintainer) são mecanismos distintos.
2. **Contextualizar regulação por jurisdição.** A regulação cripto varia significativamente entre UE (MiCA), EUA (SEC/CFTC) e outras jurisdições. Indicar sempre o quadro de referência.
3. **Não fornecer aconselhamento jurídico.** Toda a análise regulatória é de natureza académica e informativa.
4. **Usar terminologia jurídica rigorosa em português europeu.** Ex: "valor mobiliário", "instrumento financeiro", "activo digital", "prestador de serviços de criptoativos (PSCA)".
5. **Citar fontes técnicas quando aplicável.** NEAR White Paper para mecanismos de slashing e segurança; regulamentos oficiais (MiCA, MiFID II) para enquadramento legal.

---

## Governança On-Chain — NEAR Protocol

### Parâmetros de Protocolo
- A NEAR utiliza um modelo de **governança off-chain com implementação on-chain**: alterações ao protocolo são propostas e discutidas fora da cadeia, depois implementadas por validadores através de actualizações de software.
- Não existe mecanismo de votação on-chain formal para decisões de governança — diferença relevante face a protocolos como Tezos ou Cosmos.
- O token NEAR confere direito de participação em processos de governança, mas os mecanismos são maioritariamente sociais e reputacionais.

### Reference Maintainer
- Entidade (tipicamente a NEAR Foundation ou um grupo de core developers) responsável por manter a implementação de referência do protocolo.
- Define quais as actualizações que constituem o protocolo canónico.
- Ponto de centralização reconhecido — trade-off entre agilidade de desenvolvimento e descentralização pura.

### NEAR Foundation
- Organização sem fins lucrativos responsável pelo desenvolvimento do ecossistema, financiamento de grants e representação institucional.
- Gere o *Protocol Treasury* (0,5% de inflação anual alocado ao desenvolvimento).
- Não controla directamente os validadores nem a rede, mas tem influência significativa no roadmap técnico.

---

## Segurança da Rede e Mecanismos Punitivos

### Slashing — Progressive Slashing
O NEAR implementa um sistema de **slashing progressivo** para penalizar comportamentos desonestos dos validadores:

| Infracção | Penalização |
|---|---|
| **Double signing** (assinar dois blocos conflituantes) | 3× o stake do validador malicioso (limitado ao stake disponível) |
| **Invalid chunks** (produzir chunks inválidos) | 100% do stake do validador |

- O slashing afecta apenas o validador infractor, não os delegadores directamente (diferença face a alguns outros protocolos).
- Mecanismo dissuasor de comportamentos Byzantine sem punir erros involuntários de configuração (que resultam apenas em exclusão do conjunto de validadores, não slashing).

### Validadores Ocultos (Hidden Validators)
- Mecanismo central de segurança contra **ataques adaptativos**: um atacante que conhece antecipadamente quais os validadores responsáveis por um shard pode tentar corrompê-los ou atacá-los selectivamente.
- A NEAR usa **VRF (Verifiable Random Function)** para seleccionar validadores de forma aleatória e verificável, revelando apenas no último momento quem valida cada shard.
- Garante que um atacante não consegue saber com antecedência suficiente quem atacar.

### Fishermen
- Nós leves que monitorizam a rede e submetem **fraud proofs** quando detectam chunks inválidos.
- Não participam no consenso, mas actuam como camada de vigilância descentralizada.
- Requerem apenas um stake mínimo reduzido para participar.

### Ataques Adaptativos e Mitigações
- **Problema:** Num sistema sharded, um atacante com <33% do stake total poderia comprometer um shard individual se conseguisse concentrar poder nesse shard.
- **Mitigação NEAR:** Rotação aleatória de validadores por epoch (½ dia) + hidden validators + VRF → custo de ataque mantém-se proporcional ao stake total da rede, não ao stake de um shard individual.

---

## Enquadramento Regulatório

### União Europeia — MiCA (Markets in Crypto-Assets Regulation)
- Regulamento (UE) 2023/1114, aplicável a partir de Dezembro de 2024 (stablecoins) e Dezembro de 2024/2025 (restantes).
- Classifica activos digitais em três categorias: **e-money tokens (EMT)**, **asset-referenced tokens (ART)** e **outros criptoativos**.
- O token NEAR enquadra-se tipicamente na categoria "outros criptoativos" — não é stablecoin nem referenciado a activos.
- **Prestadores de Serviços de Criptoativos (PSCA/CASP):** exchanges, custódia, execução de ordens — todos sujeitos a autorização e requisitos de capital.
- Obrigações de **whitepaper** para emissores de criptoativos com oferta pública.

### Estados Unidos — Quadro SEC/CFTC
- Debate sobre se tokens de utilidade constituem valores mobiliários (*securities*) ao abrigo do **Howey Test**.
- A SEC tem adoptado posição expansiva; a CFTC reivindica jurisdição sobre criptoativos como commodities.
- Ausência de legislação federal abrangente — regulação por enforcement (acções judiciais caso a caso).
- Tokens de governança com expectativa de valorização têm maior risco de classificação como *security*.

### Portugal / Banco de Portugal
- O Banco de Portugal é a autoridade competente para registo de prestadores de serviços de criptoativos ao abrigo da legislação anti-branqueamento (Lei n.º 83/2017).
- Com a entrada em vigor plena do MiCA, o Banco de Portugal passa a ser a autoridade competente para autorização de CASP em Portugal.
- CMVM (Comissão do Mercado de Valores Mobiliários) tem jurisdição quando os criptoativos são qualificados como instrumentos financeiros.

---

## Compliance em DeFi e Smart Contracts

### KYC/AML em Protocolos Descentralizados
- **Desafio fundamental:** protocolos descentralizados não têm entidade central responsável pela implementação de KYC/AML.
- **Abordagem regulatória emergente:** foco nos pontos de entrada/saída (exchanges, fiat on-ramps) e em interfaces (front-ends) que interagem com os protocolos.
- **Travel Rule (GAFI/FATF):** obrigação de transmissão de informação sobre ordenante e beneficiário em transferências acima de limiar — difícil implementação em DeFi puro.

### Smart Contracts — Responsabilidade Legal
- Ausência de regime jurídico específico para smart contracts na maioria das jurisdições.
- Questões abertas: validade como contrato, lei aplicável, responsabilidade por bugs, imutabilidade vs. necessidade de compliance.
- MiCA não regula directamente smart contracts, mas regula os serviços construídos sobre eles quando há intermediário identificável.

### Identidade Digital NEAR (`nome.near`)
- Contas NEAR com nomes legíveis não constituem identidade verificada (não-KYC por defeito).
- Potencial para integração com sistemas de identidade descentralizada (DID — Decentralised Identifiers) para compliance em DeFi regulado.

---

## Riscos Sistémicos e Descentralização

| Risco | Descrição | Mitigação NEAR |
|---|---|---|
| Concentração de validadores | Poucos validadores controlam a rede | Thresholded PoS — maximiza nº de validadores únicos |
| Centralização de governança | Reference Maintainer / NEAR Foundation | Processo aberto de EIPs; comunidade pode fazer fork |
| Ataque de 51% adaptativo | Comprometer shards individuais | Hidden validators + VRF + rotação por epoch |
| Risco regulatório | Classificação adversa do token | Token utility-first; sem promessa de rendimento |
| Risco de custódia | Perda de chaves privadas | Modelo de conta abstracta NEAR (recovery keys) |

---

## Limitações

- Não fornece aconselhamento jurídico, fiscal ou de compliance específico.
- O enquadramento regulatório cripto evolui rapidamente — verificar sempre a versão mais recente dos regulamentos.
- A análise técnica de slashing e segurança baseia-se no NEAR White Paper e Nightshade Paper (2019); implementações podem ter evoluído.
- Questões de arquitectura técnica detalhada (sharding, WASM, consenso) são da competência da skill `s2-arquitetura-tecnica`.
- Questões de tokenomics e economia de mercado são da competência da skill `s1-tokenomics-mercado`.
