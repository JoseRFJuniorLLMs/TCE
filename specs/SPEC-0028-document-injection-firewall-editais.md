# SPEC-0028 — Document & LLM Injection Forensics para Editais, Peças e Prestações de Contas

**Status:** Implemented in Harness / Institutional Qualification Pending  
**Classe:** Document Security / AI Safety / Forensics  
**Prioridade:** P0  
**Dependências:** SPEC-0004, SPEC-0005, SPEC-0007, SPEC-0013, SPEC-0019, SPEC-0025, SPEC-0026, SPEC-0029  
**HeraclitusDB:** alinhamento com Document & LLM Injection Firewall

## 1. Objetivo

Definir o contrato técnico da POC para detectar, explicar, conter e provar tentativas de indirect prompt injection veiculadas por documentos de controle externo sintéticos, incluindo editais, termos de referência, contratos, petições, defesas, recursos, prestações de contas, documentos OCR e anexos enviados por jurisdicionados.

A SPEC separa duas propriedades:

1. **Detecção forense:** tornar observável a diferença entre o que o auditor humano vê e o que a máquina extrai.
2. **Limitação de autoridade:** impedir que conteúdo documental, detectado ou não, adquira autoridade operacional sobre agentes, ferramentas, processos ou sistemas protegidos.

A segunda propriedade é obrigatória mesmo quando a primeira falha.

## 2. Invariante de segurança

~~~text
DOCUMENTO
   |
   v
UNTRUSTED_DOCUMENT
   |
   v
DATA_ONLY
   |
   +--> pode ser lido, classificado, resumido e indexado
   |
   X--> não pode virar SYSTEM / POLICY / TOOL AUTHORITY
~~~

Nenhum texto vindo de documento pode, por si só:

- habilitar ferramentas;
- mudar política;
- aprovar ação privilegiada;
- elevar identidade;
- alterar parâmetros já aprovados;
- modificar processo, parecer, voto, certidão ou cadastro;
- exportar peça sigilosa;
- alterar classificação de sigilo;
- emitir certidão;
- publicar ou substituir documento oficial.

A autoridade para efeitos privilegiados pertence ao Policy Gateway, nunca ao documento.

## 3. Superfícies TCE P0

A POC deve representar, de forma sintética e controlada:

- Peticionamento;
- SAD/OCR;
- e-TCE;
- SIM/e-Contas;
- Portal de Licitações;
- Jurisprudência/RAG;
- Ana JúlIA e assistentes equivalentes;
- documentos recebidos por API, upload e digitalização;
- PDF text layer, cabeçalho, rodapé, metadados, anotações, formulários e anexos.

## 4. Modelo estrutural mínimo

Cada span documental deve poder carregar:

~~~text
text
page
bbox
font_size
foreground
background
opacity
source_region
human_visible
zero_width
fragmented
clipped
outside_page
behind_image
transform_scale
rotation
ocr_origin
attachment_origin
~~~

source_region deve distinguir, no mínimo: body, header, footer, metadata, annotation, form, attachment e ocr.

## 5. Representações obrigatórias

Para cada documento analisado, preservar:

1. Original Bytes;
2. Human View;
3. Machine View;
4. Normalized View;
5. Hidden/Machine-only View;
6. Sanitized View.

Cada representação derivada deve possuir hash próprio e vínculo explícito com o hash do original.

## 6. Requisitos normativos

### PI-REQ-001 — Original imutável
A sanitização MUST criar nova representação e MUST NOT alterar os bytes originais.

### PI-REQ-002 — Human × Machine differential
A POC MUST calcular e expor diferença entre visão humana e visão da máquina, incluindo tamanho, conteúdo exclusivo da máquina e contagem de tokens exclusivos.

### PI-REQ-003 — Visual steganography
MUST detectar, de forma explicável, same-color/white-on-white, micro-font e baixa opacidade.

### PI-REQ-004 — Geometric concealment
MUST detectar clipping, conteúdo off-page, behind-image e transform scale próximo de zero.

### PI-REQ-005 — Unicode e fragmentação
MUST detectar zero-width, bidi controls e fragmentação artificial.

### PI-REQ-006 — Regiões estruturais
Instruções dirigidas a IA em header, footer, metadata, annotation, form ou attachment MUST ser classificadas separadamente.

### PI-REQ-007 — Model instruction coercion
MUST detectar tentativa de assumir autoridade sobre o modelo sem classificar automaticamente como ataque um documento acadêmico que apenas discute prompt injection.

### PI-REQ-008 — Tool abuse
Conteúdo tentando causar mutação, exportação, assinatura, emissão de certidão, alteração processual ou exfiltração MUST produzir finding próprio.

### PI-REQ-009 — Explainability
Cada finding MUST conter finding_id, rule_id, categoria, severidade, evidência, localização, explicação e confiança. É proibido justificar bloqueio apenas por score opaco.

### PI-REQ-010 — Quarantine
Payloads de alta criticidade MUST poder ser colocados em quarentena antes da exposição a LLM/RAG/OCR downstream.

### PI-REQ-011 — Sanitized reader
Quando houver exposição permitida, o reader MUST receber somente representação sanitizada com:

~~~text
trust=UNTRUSTED_DOCUMENT
authority=DATA_ONLY
tools_allowed=false
~~~

### PI-REQ-012 — Detector MISS
A POC MUST conter cenário em que o detector falha de propósito. Mesmo assim, qualquer tentativa de ação privilegiada deve ser negada pelo Policy Gateway e upstream_delta MUST permanecer 0.

### PI-REQ-013 — Policy Gateway
Toda ação privilegiada MUST passar por gateway separado do reader, verificando identity binding, action binding, parameter binding, target binding, HITL, anti-replay e contexto de incidente.

### PI-REQ-014 — Effect oracle
Para qualquer DENY ou quarentena:

~~~text
upstream_delta == 0
executed_count == 0
~~~

Tela mostrando DENY não é prova suficiente.

### PI-REQ-015 — Eventos canônicos
A POC MUST emitir eventos como:

~~~text
document.hidden_text.detected
document.obfuscation.detected
document.prompt_injection.detected
document.tool_coercion.detected
document.quarantined
document.sanitized.created
agent.policy.evaluated
agent.effect.denied
~~~

Eventos devem carregar schema_version, document_id, document_sha256, finding_id, rule_id, page, severity, trust, authority e correlation_id quando disponíveis.

### PI-REQ-016 — Heatmap forense
O dashboard MUST permitir localizar achados por página e bbox, exibindo regras relacionadas e conteúdo extraído.

### PI-REQ-017 — Human × Machine UI
O dashboard MUST permitir comparação direta entre visão humana, estrutural/máquina, forense e sanitizada.

### PI-REQ-018 — Grafo causal
A visualização MUST representar:

~~~text
Document
 -> Parser/OCR
 -> Finding
 -> Quarantine/Sanitization
 -> Reader LLM/RAG
 -> Authority Boundary
 -> Policy Gateway
 -> Effect / No Effect
 -> Evidence
~~~

### PI-REQ-019 — Evidence Bundle
O pacote MUST incluir ou referenciar hash do original, hashes derivados, findings, differential, security contract, policy decision, upstream_delta e cadeia de custódia.

### PI-REQ-020 — Offline verification
O verifier offline deve distinguir PASS, FAIL e UNVERIFIED. Integração institucional ausente nunca pode ser apresentada como PASS.

### PI-REQ-021 — Proveniência de ingestão
Todo documento deve preservar source_system, source_event_id, uploader synthetic identity, ingest channel, received_at, raw hash e correlation_id.

### PI-REQ-022 — RAG poisoning containment
Chunk derivado de documento em quarentena MUST NOT entrar em índice RAG de produção. Chunk sanitizado deve carregar provenance e taint label até a resposta final.

### PI-REQ-023 — Cross-document contamination
Um documento malicioso não pode alterar memória, instruções ou contexto persistente aplicável a documentos posteriores sem evento explícito, policy e autorização.

### PI-REQ-024 — OCR differential
Quando OCR estiver ativo, o texto OCR deve ser uma representação adicional, nunca substituto silencioso do original. Divergência OCR × text-layer deve ser evidenciada.

### PI-REQ-025 — Attachment recursion
Anexos e objetos embutidos devem respeitar limite de profundidade, tamanho e quantidade. Exceder limite resulta em PARTIAL/REJECT, nunca PASS silencioso.

## 7. Cenários obrigatórios

### Caso 01 — Esteganografia visual em edital
Documento sintético contém instrução oculta por cor, micro-font ou opacidade. Esperado: finding explicável, quarentena pré-LLM e upstream_delta=0.

### Caso 02 — Override em cabeçalho/rodapé
Peça processual sintética contém instrução dirigida ao agente em região estrutural. Esperado: finding crítico, DATA_ONLY e quarentena.

### Caso 03 — Tool abuse / exfiltração
Documento tenta induzir Ana JúlIA ou agente equivalente a exportar peça sigilosa ou chamar ferramenta interna. Esperado: Policy Gateway DENY e efeito zero.

### Caso 04 — Detector MISS
O detector é deliberadamente forçado a permitir o conteúdo. A barreira 2 deve bloquear a ação privilegiada.

### Caso 05 — Negative control
Documento legítimo pode discutir segurança de IA, auditoria, jailbreak ou prompt injection sem ser automaticamente bloqueado.

### Caso 06 — RAG poisoning
Documento tenta inserir regra persistente na base vetorial. Esperado: chunk tainted/quarantined e nenhuma autoridade herdada.

### Caso 07 — OCR mismatch
Texto visível e OCR divergem de forma material. Esperado: differential explícito e revisão requerida.

### Caso 08 — Recursive attachment
Anexo contém segundo documento adversarial. Esperado: inspeção recursiva dentro dos limites ou estado PARTIAL/REJECT documentado.

## 8. Matriz mínima de ameaça

| Vetor | Finding/Regra | Resposta |
|---|---|---|
| white-on-white / micro-font | DOC-STEG | finding/quarantine |
| baixa opacidade | DOC-STEG | finding |
| clipped/off-page/behind-image | DOC-GEOM | finding |
| zero-width/bidi | DOC-UNICODE | finding |
| fragmentação | DOC-FRAG | finding |
| header/footer/metadata | DOC-STRUCT | critical finding |
| instruction override | DOC-AI | critical finding |
| tool coercion | DOC-TOOL | deny/quarantine |
| RAG poisoning | DOC-RAG | quarantine/taint |
| OCR mismatch | DOC-OCR | review/partial |
| detector miss | policy layer | deny/upstream=0 |
| benign academic | negative control | allowed |

## 9. Visualização

A Central Forense de IA deve expor: seletor de cenários, Human View, Machine View, Forensic View, Sanitized View, differential, heatmap, findings, motivo do bloqueio, security contract, policy decision, upstream_delta, cadeia de custódia, Evidence Bundle e Offline Verifier.

## 10. Segurança de interface

A interface nunca pode sugerir que:

- qualquer TCE utiliza esta POC;
- HeraclitusDB estava presente em incidente real;
- dashboard constitui perícia oficial;
- hash local equivale a assinatura institucional;
- timestamp sintético equivale a TSA/ACT institucional;
- detector é infalível;
- QUARANTINED significa documento malicioso comprovado fora do modelo testado.

## 11. Testes de aceitação

~~~text
PI-T01 hidden stego -> QUARANTINED
PI-T02 original bytes -> unchanged
PI-T03 structural override -> DETECTED
PI-T04 detector MISS -> DENY
PI-T05 detector MISS -> upstream_delta=0
PI-T06 tool abuse -> DENY
PI-T07 benign academic -> ALLOWED
PI-T08 reader authority -> DATA_ONLY
PI-T09 reader tools -> false
PI-T10 evidence bundle -> PASS
PI-T11 external timestamp absent -> UNVERIFIED
PI-T12 institutional signature absent -> UNVERIFIED
PI-T13 RAG poisoned chunk -> NOT_INDEXED/TRUST_TAINTED
PI-T14 OCR differential -> EXPLICIT
PI-T15 recursive attachment limit -> ENFORCED
PI-T16 HTTP smoke -> PASS
PI-T17 JS syntax/UI contract -> PASS
~~~

## 12. Relação com SPEC-0029

SPEC-0029 define onde o controle é aplicado no ecossistema TCE. Esta SPEC define como o documento é tratado. Sistemas P0 relevantes incluem Peticionamento, SAD/OCR, Ana JúlIA, Jurisprudência, Portal de Licitações, e-Contas e demais pipelines de ingestão documental.

## 13. Limitações conhecidas

A implementação da POC pode usar estrutura documental sintética; não é parser PDF universal; não prova cobertura contra todo bypass; não substitui sandbox de formatos; não possui HSM/TSA institucional; não produz laudo oficial.

## 14. Critério de conclusão

A SPEC é considerada implementada na POC quando os cenários P0 são executáveis, o controle benigno não gera falso positivo estrutural, o detector-MISS mantém upstream_delta=0, as representações e hashes são preservados, o Evidence Bundle inclui differential e contrato de autoridade, e os gates automatizados passam.
