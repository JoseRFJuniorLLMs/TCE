# SPEC-0025 — Post-Compromise Audit Process & Data Abuse (e-TCE / Controle Externo)

**Status:** Proposed  
**Fase:** 2 — Pós-compromisso  
**Prioridade:** P0  
**Dependências:** SPEC-0023, SPEC-0024, SPEC-0005

## 1. Objetivo

Continuar a campanha da Fase 1 usando a mesma identidade/contexto comprometido (`service-account-17`) e demonstrar a tentativa de abuso sobre uma aplicação sintética de Controle Externo (`e-TCE` / `AuditCaseLab`) e um banco transacional sintético (`engine-neutral`: classe Oracle / PostgreSQL / SQL Server).

## 2. Aplicação Sintética de Controle Externo

Representa conceitos universais de um processo eletrônico em Tribunal de Contas:
- processo de contas / fiscalização (`case`);
- andamento / instrução técnica / parecer do MPC / voto (`docket/event`);
- metadados de unidade gestora, ordenador de despesa e exercício financeiro (`metadata`);
- peça técnica, edital de licitação ou acórdão (`document`);
- grau de sigilo (`classification`: `PUBLIC`, `INTERNAL`, `RESTRICTED`);
- ator (`actor`: auditor, relator, conta de serviço, agente de IA);
- trilha de auditoria (`audit trail`).

Não reproduz schema proprietário de nenhum TCE real.

## 3. Processo Fictício Alvo

```text
case://SYNTHETIC/RE-000001 (alias: case://SYNTHETIC/TC-PCA-000001)
natureza=Prestação de Contas Anual / Fiscalização de Edital
status=ACTIVE
classification=RESTRICTED
relator=CONSELHEIRO-SINTETICO-01
```

## 4. Tentativas do Invasor Pós-Compromisso

A identidade marcada na Fase 1 tenta:
- ler metadados restritos de instrução / denúncia sigilosa;
- alterar metadados ou parecer técnico de irregularidade para regularidade (`case_write`);
- mudar a classificação de sigilo;
- exportar peça restrita (`export_restricted`);
- reutilizar uma aprovação humana (`HITL`) já consumida (*replay*);
- trocar a identidade ou o alvo após a concessão da aprovação (*identity/parameter swap*);
- apagar ou adulterar o registro de auditoria (`modify`, `delete`, `reorder`, `truncate`).

## 5. Política Sensível ao Contexto do Incidente (SOC $\rightarrow$ Policy)

```text
IF incident.state == OPEN
AND incident.severity IN {HIGH, CRITICAL}
AND principal == incident.principal
AND action == "case_write"
THEN DENY (reason_code = OPEN_HIGH_RISK_INCIDENT, upstream_delta = 0)
```

## 6. Oráculo Independente do Banco / Sistema Protegido

Para qualquer decisão `DENY` ou `REQUIRE_HITL` pendente:
- `upstream_delta == 0` (nenhum efeito colateral no banco ou aplicação).

Para decisão `ALLOW` com aprovação humana válida e não consumida:
- `upstream_delta == 1` e recibo assinado por digest canônico (`EFF-xxxxxx`).
