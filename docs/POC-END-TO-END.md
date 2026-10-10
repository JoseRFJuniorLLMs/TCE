# POC End-to-End — Intrusão, Abuso em Processo de Contas/Fiscalização e Evidência Verificável (TCE)

## 1. História Única da Campanha

A POC demonstra uma única campanha adversarial sintética que atravessa duas fases conectadas pelo mesmo identificador:

```text
campaign_id = TCE-POC-CAMPAIGN-001
incident_id = TCE-POC-INCIDENT-001
```

Todos os eventos brutos, sinais de detecção, nós do grafo temporal, decisões de política, aprovações humanas (HITL), recibos do sistema protegido e provas criptográficas carregam referência auditável a essa campanha.

---

## 2. Fase 1 — Invasão e Correlação Multi-Fonte (SOC / Sentinel)

### 2.1 Sinais Simulados no Ambiente Sintético do TCE
A campanha inicia com eventos que, isoladamente, poderiam parecer ruído operacional na infraestrutura do Tribunal:

```text
T+00 firewall     conexão externa no portal/API de remessa (waf_score acima do limiar)
T+03 identity     autenticação de conta de serviço (service-account-17) em dispositivo não reconhecido
T+08 host         execução de processo fora do baseline (synthetic-worker) em servidor Linux de aplicação
T+14 network      movimentação lateral sintética entre servidor de aplicação (srv-app-07) e servidor de banco (srv-db-02)
T+20 db-audit     consulta massiva fora do perfil sobre metadados de processos de contas e licitações
T+27 app          acesso direto a processo restrito de fiscalização/prestação de contas no sistema sintético
```

### 2.2 Correlação Determinística e Abertura de Incidente
O **Heraclitus Sentinel** correlaciona os sinais por:
* identidade/principal (`service-account-17`);
* endereços de origem/destino e hosts (`srv-app-07`, `srv-db-02`);
* banco de dados e recurso de controle externo (`case://SYNTHETIC/TC-PCA-000001`);
* janela lógica (`HLC` / `LSN`) e regras determinísticas explicáveis (`DET-EDGE-001` a `DET-APP-001`).

Eventos administrativos benignos ocorrendo na mesma janela temporal são avaliados e **excluídos** do incidente (prova de não-contaminação).

Saída da Fase 1:
```text
SecuritySignal[]
      |
      v
Temporal Security Graph
      |
      v
Incident TCE-POC-INCIDENT-001
state: OPEN
severity: HIGH
confidence: HIGH
principal: service-account-17
evidence_lsns: [...]
```

---

## 3. Ponte entre as Fases (SOC $\rightarrow$ Policy Gateway)

Quando a correlação atinge o critério de qualificação, o incidente é aberto e exportado como `IncidentContext`:
```text
INCIDENT_OPENED (TCE-POC-INCIDENT-001)
COMPROMISE_SUSPECTED (principal = service-account-17)
campaign_id preserved (TCE-POC-CAMPAIGN-001)
```
A política da Fase 2 passa a consultar automaticamente esse estado antes de autorizar qualquer mutação em processos de contas, pareceres de auditoria, certidões ou bancos de dados.

---

## 4. Fase 2 — Abuso Pós-Compromisso sobre Processo de Contas e IA

### 4.1 Recurso Protegido Sintético
```text
case://SYNTHETIC/TC-PCA-000001
tipo: Prestação de Contas Anual / Fiscalização de Licitação
classificação: RESTRICTED (Sigilo / Instrução em Andamento)
```

### 4.2 Tentativas do Atacante Sintético
1. **Alteração não autorizada de metadados/parecer do processo (`case_write`):**
   * O atacante, usando a identidade comprometida `service-account-17` (vinculada ao incidente aberto na Fase 1), tenta modificar o parecer/estado do processo de contas.
   * **Resultado:** `DENY` imediato (`reason_code = OPEN_HIGH_RISK_INCIDENT`) com `upstream_delta = 0` (nenhuma escrita atinge o banco ou sistema protegido).
2. **Exportação de documento restrito com exigência de HITL (`export_restricted`):**
   * Uma operação sensível de exportação é submetida ao gate de política.
   * Sem aprovação humana prévia: retorna `REQUIRE_HITL` (`upstream_delta = 0`).
   * Com aprovação humana válida, vinculada a `principal`, `action`, `target` e `parameters_digest`: retorna `ALLOW` uma única vez (`upstream_delta = 1`).
3. **Ataques ao protocolo de autorização (Replay, Identity Swap e Parameter Swap / TOCTOU):**
   * **Replay:** tentativa de reutilizar o mesmo `approval_id` já consumido $\rightarrow$ `DENY` (`REPLAY_DETECTED`, `upstream_delta = 0`).
   * **Identity Swap:** tentativa de usar a aprovação com outro agente/operador $\rightarrow$ `DENY` (`IDENTITY_BINDING_MISMATCH`, `upstream_delta = 0`).
   * **Parameter/Target Swap:** tentativa de trocar o processo ou parâmetros após a aprovação humana $\rightarrow$ `DENY` (`PARAMETERS_DIGEST_MISMATCH`, `upstream_delta = 0`).

---

## 5. Tentativa de Encobrimento (Tamper / Apagar Rastros)

Para esconder sua atuação no ambiente do Tribunal, o atacante tenta sabotar o histórico registrado:
* **Modificação (`modify`):** alterar um registro `DENY` para `ALLOW` ou editar o payload;
* **Exclusão (`delete`):** apagar um evento incriminador da trilha;
* **Reordenação (`reorder`):** inverter a ordem cronológica de dois eventos preservando seus textos;
* **Truncamento (`truncate`):** cortar o final do log de auditoria.

Resultado em todas as modalidades:
```text
HISTORY_TAMPER  => DETECTED (quebra de hash-chain, Merkle root, manifesto e/ou sequência LSN)
```

---

## 6. Reconstrução Temporal (*Time Travel AS-OF LSN*) e Verificação Offline

* **Time Travel (`AS OF LSN`):** permite à auditoria interna, Corregedoria ou Ministério Público de Contas consultar exatamente como o processo e os indicadores de segurança estavam no `LSN` anterior à intrusão, no instante da abertura do incidente, após o bloqueio `DENY` e após a ação aprovada por HITL.
* **Evidence Bundle e Verificador Offline (`verify.py`):** exporta um pacote autocontido com eventos, sinais, grafo, decisões de política, aprovações, árvore de Merkle e manifesto assinado por digests SHA-256, validado por um verificador independente com a rede e o servidor desligados.
