# Referências Institucionais e Normativas — Uso Responsável (TCE)

Estas referências contextualizam a POC para o ecossistema de Controle Externo brasileiro. Elas **não** significam homologação, certificação ou conformidade institucional automática.

## 1. Ecossistema de Tribunais de Contas (Atricon, IRB, Rede STI e TCU)

- **Atricon — Marco de Medição de Desempenho dos Tribunais de Contas (MMD-TC):** diretrizes de Governança e Gestão de Tecnologia da Informação, Segurança Cibernética, Continuidade de Negócios (Backup/DR) e Inteligência Artificial no Controle Externo.
- **Diretrizes de Governança de IA nos Tribunais de Contas (Atricon / Rede STI):** governança, transparência, supervisão humana e proteção de dados sensíveis em modelos de linguagem e ferramentas de fiscalização automatizada.
- **Acórdãos do TCU sobre Segurança Cibernética e Governança de TI:** levantamentos de maturidade cibernética e prevenção a ataques de *ransomware* na Administração Pública.

## 2. Normativos Federais e de Controle Externo Aplicáveis

- **Decreto Federal nº 10.540/2020 (Padrão Mínimo de Qualidade do SIAFIC):** requisitos de integridade, confiabilidade, rastreabilidade e registro contábil nos sistemas integrados de execução orçamentária, administração financeira e controle fiscalizados pelos TCEs.
- **Lei nº 14.133/2021 (Nova Lei de Licitações e Contratos Administrativos) e PNCP:** fiscalização eletrônica de editais, termos de referência, atas e contratos públicos.
- **Lei nº 13.709/2018 (LGPD):** proteção de dados pessoais no tratamento de folhas de pagamento, cadastros e processos de controle externo.
- **Programa de Privacidade e Segurança da Informação (PPSI / GSI-PR):** controles de referência de segurança da informação e resposta a incidentes para órgãos públicos.
- **ICP-Brasil (MP nº 2.200-2/2001 e Lei nº 14.063/2020):** validade jurídica de documentos eletrônicos, assinaturas qualificadas e carimbos do tempo (RFC 3161).
- **Resolução CNJ nº 615/2025 (alterada pela Resolução nº 674/2026) e Resolução CNJ nº 396/2021:** referências técnicas correlatas de governança de IA e estratégia de segurança cibernética em órgãos colegiados de julgamento.

## 3. Regra de Mapeamento na POC

Mapeamentos normativos no HeraclitusDB utilizam exclusivamente estados verificáveis:
```text
PASS | FAIL | PARTIAL | EXTERNAL | NOT_APPLICABLE | NOT_ASSESSED | UNKNOWN
```
Cada controle técnico aponta para um teste automatizado e uma evidência concreta. A presença de um controle de software nunca é apresentada como `COMPLIANT = true` sem a respectiva qualificação institucional do Tribunal.
