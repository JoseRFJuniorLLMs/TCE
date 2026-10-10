# Matriz de Rastreabilidade — POC Integrada TCE / HeraclitusDB

## Fase 1 — Invasão, Telemetria e Correlação SOC

| Claim | SPEC | Teste | Evidência |
|---|---|---|---|
| Fontes heterogêneas (FW/WAF, IAM, Host, Rede, DB, e-TCE) normalizam sem perder o bruto | SPEC-0021 | `test_telemetry.py` (`P1-INGEST-*`) | `CanonicalTelemetry` + `raw` preservado |
| Regra determinística produz sinal explicável | SPEC-0022 | `test_detector.py` (`P1-DETECT-01`) | `Detection` / `SecuritySignal` |
| Eventos benignos abaixo do limiar não disparam falso positivo | SPEC-0022 | `test_detector.py` (`DET-02`) | `None` (`NO_SIGNAL`) |
| Sinais multi-fonte correlacionam por entidades tipadas na janela HLC | SPEC-0022 / SPEC-0023 | `test_correlation.py` (`P1-CORR-01`) | `correlate()` + arestas do grafo |
| Evento inocente desconectado não contamina a campanha | SPEC-0023 | `test_correlation.py` (`P1-CORR-NEG`) | exclusão do componente principal |
| Incidente preserva proveniência até LSN e hash de origem | SPEC-0023 | `test_poc.py` (`P1-PROV-01`) | `evidence_lsns` + `evidence_hash` |
| Contexto do incidente aberto alimenta a política da Fase 2 | SPEC-0024 / SPEC-0025 | `test_policy.py` (`BRIDGE-01`) | `OPEN_HIGH_RISK_INCIDENT` $\rightarrow$ `DENY` |

## Fase 2 — Abuso Pós-Compromisso em Processo de Contas/Fiscalização e IA

| Claim | SPEC | Teste | Evidência |
|---|---|---|---|
| Identidade vinculada a incidente aberto não altera processo de contas | SPEC-0025 | `test_policy.py` / `test_poc.py` (`P2-WRITE-01`) | `DENY` + `upstream_delta=0` |
| Aprovação humana (HITL) válida e vinculada permite execução única | SPEC-0005 / SPEC-0025 | `test_policy.py` / `test_poc.py` (`P2-HITL-01`) | `ALLOW` + recibo `EFF-000001` (`upstream_delta=1`) |
| Replay de `approval_id` já consumido é bloqueado | SPEC-0006 | `test_policy.py` / `test_poc.py` (`P2-REPLAY`) | `DENY` (`REPLAY_DETECTED`) + `upstream_delta=0` |
| Troca de identidade (*identity swap*) pós-aprovação é bloqueada | SPEC-0005 / SPEC-0006 | `test_policy.py` (`P2-ID-SWAP`) | `DENY` (`IDENTITY_BINDING_MISMATCH`) + `upstream_delta=0` |
| Troca de alvo ou parâmetros (*parameter swap* / TOCTOU) é bloqueada | SPEC-0005 / SPEC-0006 | `test_policy.py` (`P2-TOCTOU`) | `DENY` (`PARAMETERS_DIGEST_MISMATCH`) + `upstream_delta=0` |
| Adulteração de histórico (`modify`, `delete`, `reorder`, `truncate`) é detectada | SPEC-0004 | `test_poc.py` (`INT-*`) | quebra de hash-chain, Merkle root e manifesto |
| Reconstrução temporal (`AS OF LSN`) recupera estados passados exatos | SPEC-0008 | `test_poc.py` / `test_processos.py` | snapshot determinístico por LSN |
| *Indirect Prompt Injection* em Edital/Peça PDF é contido em 2 barreiras | SPEC-0005 / SPEC-0025 | `test_zanin.py` | Quarentena pré-LLM ou Gateway `DENY` com `upstream_delta=0` |
| Pacote forense (`Evidence Bundle`) verifica offline sem servidor/rede | SPEC-0007 | `test_poc.py` (`EVID-*`) | `verify.py` retorna `overall=PASS` (íntegro) e `FAIL` (sabotado) |

## End-to-End

| Claim | SPEC | Evidência |
|---|---|---|
| Mesma campanha atravessa Fase 1 e Fase 2 | SPEC-0026 | `campaign_id` / `incident_id` contínuos |
| Primeiro sinal observável liga-se ao bloqueio final | SPEC-0023 / SPEC-0026 | grafo causal completo |
| Limitações institucionais permanecem explícitas (`NOT_CONFIGURED` / `UNVERIFIED`) | SPEC-0001 / SPEC-0027 | scorecard e seção `trust`/`limitations` do bundle |
