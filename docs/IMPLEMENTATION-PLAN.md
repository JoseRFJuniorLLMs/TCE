# Plano de Implementação — POC Integrada TCE / HeraclitusDB

## Fase 0 — Congelar Baseline

- Fixar commit do repositório `TCE`;
- Fixar commit do `HeraclitusDB` (`8d3d78d0ef7c59ff1fdae04cc0864b94ddf7636c` — v3.0.1);
- Registrar baseline público dos Tribunais de Contas (`docs/TCE-PUBLIC-INFRA-BASELINE.md`);
- Gerar `BUILD-INFO`;
- Congelar fixtures de telemetria e políticas de controle externo.

## Fase 1A — Malha de Telemetria (Telemetry Fabric)

Implementar adapters sintéticos vendor-neutral representativos da arquitetura de um TCE (PDTI):

```text
fw          (borda do Portal / API de Remessa de Jurisdicionados)
waf         (proteção de aplicação web e-TCE / Coleta)
identity    (AD / SSO / Gov.br / ICP-Brasil sintético)
linux       (servidores de aplicação e containers)
windows     (estações VDI de gabinetes e diretorias de controle externo)
network     (tráfego interno LAN/WLAN/VPN)
db-audit    (auditoria engine-neutral: Oracle / PostgreSQL / SQL Server class)
application (e-TCE / Coleta AUDESP-SAGRES-SIM-AM / SEI-like)
api         (integrações SIAFIC / PNCP)
backup      (proteção de dados / WORM anti-ransomware)
```

Todos convergem para `SecurityEvent` canônico preservando o evento bruto (`raw`).

Gate:
- schema canônico validado;
- proveniência até `raw_id` / `source_lsn`;
- replay determinístico.

## Fase 1B — Detecção (Sentinel L1)

Usar Sentinel / motor determinístico:
- normalização L0;
- regras determinísticas explicáveis (`DET-EDGE-001` a `DET-APP-001`) / subconjunto Sigma;
- emissão de `SecuritySignal`;
- exclusão estrita de eventos benignos abaixo do limiar.

Gate:
- sinal conhecido $\rightarrow$ `DETECTED`;
- evento benigno $\rightarrow$ `NO_SIGNAL`;
- regra inválida $\rightarrow$ fail-closed;
- replay $\rightarrow$ mesmos IDs lógicos.

## Fase 1C — Correlação Multi-Fonte e Incidente (Sentinel L3)

Construir:
- grafo temporal de segurança por entidades tipadas (`principal`, `host`, `ip`, `db`, `resource`);
- vínculo de `campaign_id`;
- cadeia causal;
- ciclo de vida do incidente (`CANDIDATE` $\rightarrow$ `OPEN`).

Gate:
- sinais multi-fonte formam um único incidente qualificado (`confidence=HIGH`);
- evento administrativo não relacionado não contamina a campanha;
- todos os nós preservam referências de evidência (`evidence_lsns` e `evidence_hash`).

## Fase 1D — Resposta SOC Sintética e Ponte para a Fase 2

Sem tocar rede real:
- abrir alerta SOC;
- marcar `principal` (`service-account-17`) como comprometido (`COMPROMISE_SUSPECTED`);
- simular contenção em cyber range;
- alimentar o `Agent Policy Gateway` da Fase 2 com o `IncidentContext`.

## Fase 2A — Aplicação de Controle Externo Sintética (`e-TCE` / `AuditCaseLab`)

Modelar em alto nível um processo eletrônico de Controle Externo:
- processo fictício de Prestação de Contas Anual / Fiscalização de Licitação;
- parecer técnico, voto/acórdão e certidão fictícios;
- classificação restrita/sigilosa;
- trilha de auditoria de banco e aplicação.

## Fase 2B — Controle de Ações Privilegiadas e Governança de IA

Integrar `Agent Policy Gateway`:
- níveis `LOW`, `MEDIUM`, `HIGH` e `BLOCKED`;
- bloqueio automático (`DENY`) quando a identidade está vinculada a incidente aberto de alto risco (`OPEN_HIGH_RISK_INCIDENT`);
- aprovação humana (`HITL`) de uso único (*single-use*);
- proteção anti-replay (`REPLAY_DETECTED`);
- vínculo estrito de identidade (`IDENTITY_BINDING_MISMATCH`), ação (`ACTION_BINDING_MISMATCH`) e parâmetros (`PARAMETERS_DIGEST_MISMATCH`);
- oráculo independente do sistema protegido (`upstream_delta = 0` em toda negação);
- inspeção documental multi-camada contra *Indirect Prompt Injection* em editais e prestações de contas em PDF.

## Fase 2C — Detecção de Adulteração (Cover Tracks / Anti-Ransomware & Anti-Tamper)

Sabotagens controladas em cópia de laboratório:
- `modify` (alterar `DENY` para `ALLOW` ou adulterar parecer);
- `delete` (apagar evento de auditoria);
- `reorder` (inverter ordem de eventos);
- `truncate` (cortar cauda do log);
- `bundle tamper` (corromper pacote exportado);
- `approval replay` (reutilizar autorização humana).

## Fase 2D — Pacote Forense (`Evidence Bundle`) e Verificação Offline

Exportar a campanha inteira:
```text
edge signal
 -> incident
 -> compromised principal
 -> privileged request on audit case
 -> policy decision
 -> human approval (HITL)
 -> upstream receipt / zero-effect proof
 -> tamper detection
```
Validar com verificador independente (`poc/verify.py`) sem rede e com o servidor desligado.
