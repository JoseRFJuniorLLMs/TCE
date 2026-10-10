# Baseline Técnico do HeraclitusDB — POC TCE

**Baseline consultado:** commit `8d3d78d0ef7c59ff1fdae04cc0864b94ddf7636c` (HeraclitusDB v3.0.1).  
**Regra:** fixar novamente o commit imediatamente antes da execução/qualificação em laboratório.

## 1. Posicionamento Arquitetural para o TCE

O HeraclitusDB não substitui o SGBD transacional do Tribunal (Oracle, PostgreSQL ou SQL Server), nem substitui o sistema processual (`e-TCE`), o sistema de remessa de jurisdicionados (`AUDESP` / `SAGRES` / `SIM-AM` / `e-Sfinge`) ou o SIEM corporativo.

Ele opera como **camada soberana de confiança, evidência, auditoria e governança de IA** ao lado desses sistemas para:
1. **Preservar** a história canônica em log append-only (`HRKL` v6);
2. **Provar** integridade e ordem temporal via LSN, HLC, CRC-32C e árvores de Merkle (`BLAKE3` / `SHA-256`);
3. **Correlacionar** sinais de intrusão multi-fonte no `Sentinel` e alimentar a política de controle de acesso com o contexto do incidente aberto;
4. **Controlar** operações privilegiadas e agentes de IA de auditoria via `Agent Policy Gateway`, `HITL` (aprovação humana de uso único), anti-replay e vínculo estrito de identidade e parâmetros (`upstream_delta = 0` em negações);
5. **Inspecionar** documentos submetidos por jurisdicionados (PDFs de editais, contratos e prestações de contas) contra *Indirect Prompt Injection* (SPEC-0092/0093/0094);
6. **Exportar** pacotes forenses (`Evidence Bundle`) verificáveis de forma 100% offline por auditores externos, Ministério Público de Contas ou peritos, sem depender do servidor original.

## 2. Capacidades Relevantes para a Fase 1 (Invasão e SOC)

### Sentinel (`heraclitus-sentinel`)
* Normalização genérica JSON para `SecurityEvent` (L0);
* Preservação de proveniência até o evento bruto (`raw_id` / `source_lsn`);
* Executor determinístico de regras L1 fail-closed e subconjunto Sigma;
* Emissão de `SecuritySignal` explicável;
* Grafo temporal de segurança (L3) e correlação por entidades tipadas (`principal`, `host`, `ip`, `db`, `resource`);
* Exportação de `IncidentContext` para consumo pela camada de política da Fase 2.

### HRKL (`heraclitus-log`)
* Persistência segmentada append-only (formato HRKL v6, modos RAW e PACKED);
* Ordenação monotônica por `LSN` e relógio lógico híbrido (`HLC`);
* Checksums CRC-32C, manifesto `HRKM`, índices `HRKI` e raízes Merkle com `BLAKE3`;
* Reconstrução temporal determinística (`AS OF LSN` / *Time Travel*) e detecção imediata de adulteração (`modify`, `delete`, `reorder`, `truncate`).

## 3. Capacidades Relevantes para a Fase 2 (Pós-Compromisso, Processos de Contas e IA)

### Agent Gateway & Policy (`heraclitus-agent`, `heraclitus-agent-gateway`)
* Proxy MCP e interceptação em modos `observe`, `shadow` e `enforce`;
* Políticas sensíveis ao contexto de incidente aberto no SOC (`OPEN_HIGH_RISK_INCIDENT`);
* Aprovação humana (`HITL`) com consumo atômico de uso único (*single-use*);
* Rejeição estrita de replay (`REPLAY_DETECTED`), troca de identidade (`IDENTITY_BINDING_MISMATCH`), troca de ação (`ACTION_BINDING_MISMATCH`) e adulteração de parâmetros/alvo pós-aprovação (`PARAMETERS_DIGEST_MISMATCH`);
* Oráculo independente de efeitos (`upstream_delta == 0` para toda operação negada).

### Document & LLM Injection Firewall (`SPEC-0092` / `SPEC-0093` / `SPEC-0094`)
* Cálculo diferencial entre visão humana e visão de máquina em documentos processuais, editais de licitação e prestações de contas;
* Detecção explicável de esteganografia visual (`#FFFFFF`, micro-fonte), caracteres Unicode invisíveis (`zero-width`) e instruções de coerção dirigidas a LLMs;
* Sanitização com autoridade `DATA_ONLY` e `tools_allowed=false`;
* Defesa em duas barreiras: mesmo em caso de falha do detector (*detector miss*), o `Agent Policy Gateway` impede que o conteúdo de um edital ou peça processual execute ferramentas privilegiadas ou altere o parecer de auditoria.

## 4. Maturidade e Transparência (Roadmap GOV-BR: SPECs 0086–0094)

A POC segue a regra estrita do `STATUS.md` do HeraclitusDB:
* **SPEC-0086 (HSM/PKCS#11):** envelope autenticado em software presente; HSM físico institucional permanece `NOT_CONFIGURED` na POC.
* **SPEC-0087 (Pacote Forense e Cadeia de Custódia):** verificador offline valida estrutura, cadeia de hashes, Merkle root, manifesto e semântica; assinatura ICP-Brasil e carimbo do tempo ACT externo permanecem `NOT_CONFIGURED` / `UNVERIFIED` enquanto não houver âncora oficial do Tribunal.
* **SPEC-0089 (Trusted Administration):** princípio `NO DURABLE INTENT => NO PRIVILEGED SIDE EFFECT`.
* **SPEC-0090 (Interoperabilidade Aberta):** integração via gRPC, REST, DataFusion/Parquet e adaptação `engine-neutral` para coexistência com Oracle, PostgreSQL e SQL Server.
* **SPEC-0091 (WORM / Legal Hold):** distinção clara entre imutabilidade lógica no HRKL e retenção física externa (WORM/Object Lock).
