# SPEC-0030 — Persistence Truthfulness, Failover, Durability & Reconciliation

**Status:** Proposed / P0 Hardening  
**Classe:** Durability / Failover / Evidence Truthfulness  
**Prioridade:** P0  
**Dependências:** SPEC-0004, SPEC-0007, SPEC-0017, SPEC-0018, SPEC-0019

## 1. Objetivo

Impedir que falha, memória volátil, fallback ou resposta ambígua sejam apresentados como persistência real no HeraclitusDB.

A regra central é simples:

~~~text
FAILED / UNKNOWN / VOLATILE
          X
          |
          v
PERSISTED / DURABLE / VERIFIED
~~~

Nenhuma camada de API ou UI pode atravessar essa fronteira sem evidência.

## 2. Estados normativos

Toda escrita relevante deve expor, quando aplicável:

~~~json
{
  "accepted": false,
  "persisted": false,
  "storage_mode": "HERACLITUS_CORE|MEMORY_FALLBACK|NONE",
  "durability": "DURABLE|VOLATILE|UNKNOWN",
  "fallback_active": false,
  "core_connected": true,
  "lsn": null,
  "reason_code": "..."
}
~~~

accepted=true não significa persisted=true. persisted=true exige recibo verificável da camada durável esperada.

## 3. Invariantes

1. Falha do core MUST NOT retornar accepted=true por conveniência.
2. MEMORY_FALLBACK MUST ser explicitamente VOLATILE.
3. LSN de fallback MUST NOT ser confundido com namespace do core real.
4. Query inválida, protocolo incompatível ou decode failure MUST NOT virar lista vazia silenciosa.
5. Append ambíguo MUST produzir UNKNOWN até reconciliação.
6. A UI MUST mostrar o modo de storage atual.
7. Evidência exportada MUST registrar se cada item veio do core real, fallback ou fixture.
8. Fallback de leitura MUST NOT autorizar escrita.
9. Escrita processual P0 deve preferir fail-closed se não houver journal durável e reconciliação formal.

## 4. Failover de append

Cenário obrigatório:

~~~text
query real PASS
append real FAIL/CoreUnavailable
~~~

Resultado esperado:

- não fabricar continuidade;
- não gerar parent inexistente;
- não declarar persistência;
- não emitir PASS;
- storage_mode e durability visíveis;
- estado reconciliável.

## 5. Journal de failover

Se escrita offline for requisito futuro, deve existir:

- journal durável;
- namespace de IDs separado;
- snapshot de base consistente;
- origem FALLBACK explícita;
- fila de reconciliação;
- detecção de conflito;
- regra de merge;
- evento de reconciliação que preserva o estado original.

Sem esses elementos, fallback de escrita processual é proibido.

## 6. Reconciliation

UNKNOWN original é imutável. Reconciliação cria novo evento:

~~~text
operation.unknown
 -> reconcile.query_upstream
 -> reconcile.result.confirmed_effect | confirmed_no_effect | still_unknown
~~~

Nunca reescrever UNKNOWN para SUCCEEDED.

## 7. Gates

| ID | Cenário | Esperado |
|---|---|---|
| DUR-01 | core append falha | persisted=false |
| DUR-02 | fallback RAM ativo | durability=VOLATILE |
| DUR-03 | query decode inválido | INVALID_RESPONSE |
| DUR-04 | timeout após possível efeito | UNKNOWN |
| DUR-05 | restart após RAM fallback | perda reconhecida, nunca mascarada |
| DUR-06 | core retorna receipt | persisted=true + receipt verificado |
| DUR-07 | deny policy | upstream_delta=0 |
| DUR-08 | UI em fallback | badge explícito |
| DUR-09 | bundle mistura fontes | provenance por item |
| DUR-10 | reconciliação | novo evento, histórico original preservado |

## 8. Critério de conclusão

Nenhum teste adversarial deve conseguir transformar falha, incerteza ou memória volátil em claim de persistência durável.
