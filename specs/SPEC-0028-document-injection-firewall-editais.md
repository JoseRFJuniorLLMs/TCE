# SPEC-0028 — Document & LLM Injection Firewall para Editais e Prestações de Contas (TCE)

**Status:** Proposed / Core Implemented in Harness  
**Classe:** AI Security / Document Forensics (SPEC-0092 / 0093 / 0094 alignment)  
**Prioridade:** P0  
**Dependências:** SPEC-0004, SPEC-0005, SPEC-0025

## 1. Objetivo

Proteger os pipelines de Inteligência Artificial dos Tribunais de Contas (classe **VigIA**, **ALICE**, **ANIA**, **SEI-IA**, **ChatTCU**) contra **Indirect Prompt Injection** oculto em PDFs de Editais de Licitação, Termos de Referência, Contratos, Defesas Prévias e Prestações de Contas enviados por jurisdicionados.

## 2. Princípio de Defesa em Duas Barreiras

1. **Barreira 1 — Inspeção Estrutural e Diferencial (Visão Humana $\times$ Visão Máquina):**
   - Preserva os bytes originais do documento (`sha256(raw_bytes)`);
   - Extrai separadamente o texto visível ao auditor humano e o fluxo bruto entregue ao parser/LLM;
   - Detecta texto branco sobre fundo branco (`#FFFFFF`), micro-fontes (`< 2.0pt`), caracteres Unicode de largura zero (`\u200B`, `\u200C`, `\u200D`, `\uFEFF`), fragmentação léxica e comandos imperativos de coerção de modelo;
   - Coloca documentos maliciosos em quarentena pré-LLM ou produz cópia sanitizada com `authority=DATA_ONLY` e `tools_allowed=false`.

2. **Barreira 2 — Heraclitus Agent Policy Gateway (Contenção Garantida em Caso de Bypass):**
   - Mesmo que um ataque *zero-day* passe despercebido pelo detector documental (*Detector MISS*), o conteúdo vindo de um documento de jurisdicionado jamais recebe autoridade para invocar ferramentas privilegiadas (`case_write`, alteração de parecer, arquivamento ou exportação de peças sigilosas) sem credencial e aprovação humana vinculada (`HITL`).
   - Resultado garantido em tentativa não autorizada: `DENY` com `upstream_delta = 0`.

## 3. Critérios de Aceite (P0)

| ID | Cenário | Esperado |
|---|---|---|
| DOC-01 | Edital/Petição com esteganografia visual + coerção | `QUARANTINED`, LLM `NOT_EXPOSED`, `upstream_delta=0` |
| DOC-02 | Tentativa oculta de exfiltração de peça sigilosa | `DENY` (`POLICY-HERACLITUS-SIGILO-V1`), `upstream_delta=0` |
| DOC-03 | Detector MISS (bypass simulado) tentando mutação no processo | Gateway `DENY` (`HUMAN_APPROVAL_REQUIRED`), `upstream_delta=0` |
| DOC-04 | Documento técnico legítimo + aprovação HITL válida | `ALLOW`, `upstream_delta=1`, zero falso positivo |
| DOC-05 | Integridade pericial dos bytes originais | `sha256(original_before) == sha256(original_after)` |
| DOC-06 | Verificação offline do pacote forense documental | `PASS` (íntegro) / `FAIL` (adulterado) |
