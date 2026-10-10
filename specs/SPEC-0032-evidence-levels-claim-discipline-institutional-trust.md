# SPEC-0032 — Evidence Levels, Claim Discipline & Institutional Trust

**Status:** Proposed / P0 Governance  
**Classe:** Evidence Semantics / Trust / Institutional Claims  
**Prioridade:** P0  
**Dependências:** SPEC-0007, SPEC-0012, SPEC-0014, SPEC-0019, SPEC-0030

## 1. Objetivo

Garantir que toda afirmação exibida pela POC corresponda exatamente ao nível de evidência disponível.

Uma demonstração técnica forte não precisa dizer que algo é impossível de adulterar. Ela precisa declarar qual propriedade foi testada, contra qual ameaça, sob qual domínio de confiança e com qual evidência.

## 2. Níveis de evidência

### E0 — CLAIM ONLY
Afirmação sem teste executado. Não pode aparecer como PASS.

### E1 — HARNESS OBSERVED
Comportamento observado no harness sintético.

### E2 — REPRODUCIBLE LOCAL
Teste reproduzível em processo limpo com evidência exportada.

### E3 — INDEPENDENT OFFLINE VERIFY
Verificador separado confirma integridade local do pacote.

### E4 — EXTERNAL TRUST ANCHORED
Existe âncora externa verificável, como HSM/KMS, WORM independente, assinatura institucional ou TSA/ACT real.

### E5 — INSTITUTIONALLY QUALIFIED
Integração e procedimento formalmente aceitos pelo órgão competente.

A POC atual deve operar predominantemente em E1-E3.

## 3. Claims proibidos sem E4/E5

Não usar, de forma absoluta:

- impossível de alterar;
- ninguém consegue adulterar;
- garantia de não-repúdio;
- RPO=0 garantido;
- 100% de paridade;
- certidão oficial;
- assinatura institucional;
- timestamp oficial;
- prova jurídica definitiva;
- produção protegida.

## 4. Linguagem permitida

Preferir:

- a POC detectou adulteração dentro do modelo testado;
- a cadeia local valida neste snapshot;
- o cenário sintético observou upstream_delta=0;
- o verificador offline confirmou integridade local;
- HSM/TSA institucional: NOT_CONFIGURED;
- confiança externa: UNVERIFIED;
- certidão sintética de demonstração, sem efeito oficial.

## 5. Claim record

Claims P0 devem poder ser rastreados para:

~~~json
{
  "claim_id": "CLAIM-...",
  "statement": "...",
  "evidence_level": "E2",
  "test_id": "...",
  "expected": "...",
  "observed": "...",
  "evidence_refs": ["..."],
  "limitations": ["..."]
}
~~~

## 6. Claim lint

CI deve bloquear frases proibidas na UI runtime quando o repositório não possui o trust profile necessário. A lista de termos é versionada e revisável.

## 7. Trust profile

A execução deve declarar:

~~~text
LOCAL_HASH_CHAIN          CONFIGURED
LOCAL_MERKLE              CONFIGURED
OFFLINE_VERIFIER          CONFIGURED
WORM_EXTERNAL             NOT_CONFIGURED
HSM_KMS_INSTITUTIONAL     NOT_CONFIGURED
TSA_ACT_INSTITUTIONAL     NOT_CONFIGURED
ICP_BRASIL_SIGNATURE      UNVERIFIED
REAL_TCE_INTEGRATION      NOT_CONFIGURED
INSTITUTIONAL_APPROVAL    NOT_APPLICABLE
~~~

## 8. Separação entre integridade e autenticidade

Hash/Merkle local pode provar consistência relativa ao snapshot e às chaves/bytes observados. Não prova sozinho autoria, autoridade institucional, tempo confiável externo ou resistência a administrador com controle do mesmo host.

## 9. Status canônicos

PASS, FAIL, DENY, DETECTED, UNKNOWN, PARTIAL, UNVERIFIED, NOT_CONFIGURED e NOT_APPLICABLE devem permanecer distintos. A UI não pode converter estados por estética.

## 10. Gates

| ID | Cenário | Esperado |
|---|---|---|
| CLM-01 | hash local sem âncora externa | no máximo E3 |
| CLM-02 | HSM ausente | NOT_CONFIGURED |
| CLM-03 | frase absoluta proibida na UI | CI FAIL |
| CLM-04 | verifier offline íntegro | LOCAL_INTEGRITY PASS |
| CLM-05 | confiança institucional ausente | UNVERIFIED/NOT_APPLICABLE |
| CLM-06 | falha ambígua | UNKNOWN, nunca PASS |
| CLM-07 | relatório final | claim -> test -> evidence -> limitation |

## 11. Critério de conclusão

A POC está em conformidade quando cada claim importante possui nível de evidência explícito e nenhuma camada visual ou textual promove evidência local/sintética a autoridade institucional inexistente.
