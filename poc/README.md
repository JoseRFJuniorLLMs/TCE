# POC Executável — HeraclitusDB / TCE

Ambiente integralmente sintético e loopback-only que demonstra a campanha descrita nas SPECs do repositório para o contexto de Tribunais de Contas (TCE).

## Executar

Requer apenas Python 3.11+ (biblioteca padrão).

```bash
cd poc
python server.py
# No Windows:
py server.py
```

Abra `http://127.0.0.1:8787`.

## Testes

```bash
py -m unittest discover -s tests -v
```

Ou na raiz do repositório:

```bat
run_tests.bat
```

## Campanha End-to-End

1. baseline benigno;
2. sinal de edge/WAF (portal/API de remessa);
3. autenticação anômala (`service-account-17`);
4. processo incomum em host Linux;
5. movimento lateral sintético (`srv-app-07` $\rightarrow$ `srv-db-02`);
6. DB audit anômalo (consulta fora do perfil);
7. acesso a processo restrito de controle externo e abertura de incidente no SOC;
8. tentativa de write pós-compromisso $\rightarrow$ `DENY` (`OPEN_HIGH_RISK_INCIDENT`), `upstream_delta=0`;
9. exportação restrita $\rightarrow$ `REQUIRE_HITL`;
10. aprovação humana vinculada (HITL);
11. uma única execução permitida (`upstream_delta=1`);
12. tentativa de replay $\rightarrow$ `DENY` (`REPLAY_DETECTED`, `upstream_delta=0`);
13. identity swap $\rightarrow$ `DENY` (`IDENTITY_BINDING_MISMATCH`, `upstream_delta=0`);
14. parameter/target swap $\rightarrow$ `DENY` (`PARAMETERS_DIGEST_MISMATCH`, `upstream_delta=0`);
15. tentativa de adulteração (`modify`, `delete`, `reorder`, `truncate`) $\rightarrow$ `DETECTED`;
16. exportação do `Evidence Bundle`;
17. verificação offline independente (`verify.py`).
