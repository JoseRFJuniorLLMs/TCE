# SPEC-0031 — Repository Contracts, CI Qualification & Deploy Provenance

**Status:** Proposed / P0-P1 Hardening  
**Classe:** QA / Repository Integrity / Deployment  
**Prioridade:** P0/P1  
**Dependências:** SPEC-0009, SPEC-0015, SPEC-0016, SPEC-0019

## 1. Objetivo

Fazer o repositório provar que código, dashboard, documentação e ambiente demonstrado pertencem ao mesmo baseline.

CI verde de sintaxe não é suficiente. O gate deve validar contratos estruturais e impedir que uma feature exista em um arquivo, mas desapareça da UI, do deploy ou da documentação.

## 2. Repository contracts P0

Testes automatizados devem validar:

- toda main-tab possui view correspondente;
- toda view crítica é navegável;
- arquivos JavaScript do dashboard passam syntax gate;
- não há handlers dinâmicos inseguros quando IDs podem ser externos;
- termos técnicos P0 possuem ajuda contextual;
- estados de fallback são explicitamente visíveis;
- claims proibidos não aparecem na UI runtime;
- links/anchors internos críticos existem;
- SPECs referenciadas pelo README existem;
- arquivos declarados no runbook/deploy existem;
- nenhum material de private key aparece no repositório.

## 3. CI mínimo

~~~text
repository hygiene          PASS
python compileall           PASS
unit tests                  PASS
repository contract tests   PASS
HTTP smoke                  PASS
all dashboard JS syntax     PASS
negative policy assertions  PASS
fault injection subset      PASS
dependency inventory        PASS
secret scan                 PASS
qualification artifacts     GENERATED
~~~

Actions externas devem ser fixadas por commit SHA quando viável.

## 4. Supply chain

O pipeline deve produzir ou registrar:

- commit fonte;
- commit HeraclitusDB usado;
- versão da toolchain;
- SBOM ou inventário equivalente;
- hashes dos artefatos;
- workflow/run;
- dependências opcionais;
- scanner status separado de vulnerability status.

Erro de scanner = NOT_ASSESSED/ERROR, nunca NO_VULNERABILITIES.

## 5. Deploy por identidade

Deploy manual por lista de arquivos é proibido como caminho qualificado.

O deploy deve usar:

~~~text
repository + exact SHA
ou
versioned artifact + digest
~~~

O ambiente deve expor DEPLOYED_SHA e, quando houver frontend estático, um build manifest verificável.

## 6. Configuração operacional versionada

Quando houver Nginx/systemd/container/orchestrator no ambiente qualificado, a configuração efetiva deve estar versionada ou exportada por digest. Sem isso, o repositório não pode alegar que provou headers, TLS, limits, timeouts, privileges ou sandboxing reais.

## 7. Paths de CI

Mudanças em poc/**, specs/**, docs/**, scripts raiz, workflow, README e arquivos de deploy devem disparar qualificação apropriada. Path filter não pode esconder alteração de requisito ou deploy.

## 8. UI/DOM gate

P1: teste headless deve validar troca de abas, ausência de erro de console, elementos críticos, acessibilidade básica por teclado e renderização dos estados PASS/DENY/UNKNOWN/FALLBACK.

## 9. Artifacts

Gerar:

- test-results;
- qualification-summary;
- dependency inventory/SBOM;
- evidence sample;
- verifier output;
- fault report;
- repository-contract report;
- deployed SHA record quando aplicável.

## 10. Critério de conclusão

Uma mudança que quebre navegação, provenance, contratos do dashboard, claims ou deploy deve derrubar o CI antes de chegar à demonstração.
