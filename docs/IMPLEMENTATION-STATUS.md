# Estado de Implementação — POC TCE / HeraclitusDB

**Atualizado:** 10/10/2026

Este documento separa capacidades realmente executadas e calculadas pelo código da POC de entradas e integrações que permanecem sintéticas por desenho de segurança.

## Executado e Calculado pelo Código da POC

| Capacidade | Estado | Implementação |
|---|---|---|
| Telemetria heterogênea (Perímetro, IAM, Host, Rede, Banco, App Controle Externo) | IMPLEMENTADO | `poc/telemetry.py` normaliza edge, IAM, host, rede, DB e app |
| Preservação raw $\rightarrow$ canonical | IMPLEMENTADO | evento bruto e resultado normalizado ficam no evento/evidência |
| Detecção determinística explicável | IMPLEMENTADO | `poc/detector.py`, regras determinísticas com `rule_id`, `reason_code` e entidades tipadas |
| Negativos benignos das regras | TESTADO | testes comprovam que eventos abaixo do limiar não geram sinal |
| Correlação cross-source | IMPLEMENTADO | `poc/correlation.py`, componentes conectados por entidades compartilhadas na janela HLC |
| Exclusão de sinal não relacionado | TESTADO | correlação não absorve sinal desconectado |
| Abertura de incidente | IMPLEMENTADO | depende do componente correlacionado qualificado (`qualifies=True`) |
| Policy enforcement (context-aware) | IMPLEMENTADO | `poc/policy.py`, fail-closed e ciente de `OPEN_HIGH_RISK_INCIDENT` |
| HITL (Human-in-the-Loop) | IMPLEMENTADO NA POC | approval vinculada, *single-use* |
| Anti-replay | IMPLEMENTADO | approval consumida não executa novamente (`REPLAY_DETECTED`) |
| Identity & Action binding | IMPLEMENTADO | identidade ou ação divergente gera `DENY` (`IDENTITY_BINDING_MISMATCH`, `ACTION_BINDING_MISMATCH`) |
| Parameter/target binding | IMPLEMENTADO | alvo/digest diferente gera `DENY` (`PARAMETERS_DIGEST_MISMATCH`) |
| Independent upstream oracle | IMPLEMENTADO | `poc/synthetic_upstream.py` emite recibos (`EFF-xxxxxx`) com digest canônico; `upstream_hits` e `upstream_delta=0` em `DENY` estritamente verificados |
| Hash-chain | IMPLEMENTADO NO HARNESS | SHA-256 encadeado entre eventos da POC |
| Merkle root | IMPLEMENTADO NO HARNESS | raiz criptográfica sobre hashes dos eventos |
| Tamper modify/delete/reorder/truncate | IMPLEMENTADO | cópia adulterada falha na verificação com `DETECTED` |
| Time travel AS-OF LSN | IMPLEMENTADO NO HARNESS | reconstrução determinística por LSN com replay de recibos |
| Evidence Bundle | IMPLEMENTADO NO HARNESS | pacote JSON com eventos, sinais, incidente, grafo, manifesto e qualificação |
| Offline verifier | IMPLEMENTADO | `poc/verify.py` independente do servidor e sem chamadas de rede |
| Adapter & Surface Health | IMPLEMENTADO | `poc/heraclitus_adapter.py` monitora status `CONNECTED`, `DEGRADED` e `UNAVAILABLE` em loopback |
| Acompanhamento Processual (Ledger gRPC) | IMPLEMENTADO | `poc/processos.py` e `poc/heraclitus_core.py` (Append imutável com `parents`, idempotência e consulta GQL `AS OF LSN`) |
| Defesa Documental / Prompt Injection Lab | IMPLEMENTADO NO HARNESS | `poc/zanin_defense.py`: parser multi-camada, findings explicáveis, defesa em 2 barreiras (detector miss $\rightarrow$ gateway deny com `upstream_delta=0`), Evidence Bundle e offline verifier |
| HTTP smoke & Suite de Testes | IMPLEMENTADO | 119 testes unitários + smoke HTTP loopback validando endpoints e CSRF |

## Sintético por Desenho (Safety by Construction)

Os itens abaixo **não representam acesso, varredura ou execução contra infraestrutura de nenhum Tribunal de Contas (TCE)**:
- o agente atacante;
- tráfego de firewall/WAF e remessas de jurisdicionados;
- identidades, contas de serviço e sessões;
- hosts Windows/Linux;
- movimento lateral;
- SGBD transacional (Oracle/PostgreSQL/SQL Server);
- sistema de processo eletrônico (`e-TCE`) e coleta (`AUDESP`/`SAGRES`/`SIM-AM`/`e-Sfinge`);
- processos de contas, editais, relatórios de auditoria e acórdãos;
- endereços de rede (RFC 5737 `203.0.113.0/24` / loopback `127.0.0.1`);
- credenciais e aprovações humanas.

## Integração com HeraclitusDB Real em Loopback

Quando uma instância local do HeraclitusDB está ativa em loopback, o adapter somente leitura consulta superfícies reais como:
- `/sentinel/status`, `/sentinel/incidents`, `/sentinel/dashboard`;
- `/telemetry/health`, `/compliance/status`;
- `/api/v1/agent/status`, `/api/v1/agent/red-team/events`;
- serviço gRPC `/heraclitus.v1.Heraclitus/Append` e `/heraclitus.v1.Heraclitus/Query`.

## Não Configurado / Não Alegar como Concluído na POC

- integração com logs reais de qualquer TCE;
- firewall/WAF/IAM/EDR/SIEM reais do Tribunal;
- SGBD Oracle/PostgreSQL/SQL Server de produção;
- HSM/KMS institucional do TCE;
- âncoras ICP-Brasil e Autoridade de Carimbo do Tempo (ACT) instaladas pelo órgão (`NOT_CONFIGURED` / `UNVERIFIED`);
- homologação, certificação ou endosso institucional por qualquer Tribunal de Contas ou pela Atricon.
