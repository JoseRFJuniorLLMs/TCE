# Checklist da Reunião Técnica — POC Integrada TCE / HeraclitusDB

## 1. Preparação da Demonstração (Offline / Loopback)

Levar validado:
- ambiente 100% isolado (loopback `127.0.0.1:8787`);
- execução limpa da campanha em duas fases;
- scorecard Fase 1 (SOC/Sentinel) e Fase 2 (Policy/HITL/Integridade);
- grafo temporal do incidente correlacionado;
- laboratório de *Indirect Prompt Injection* em Editais/Peças Processuais (Defesa em Duas Barreiras);
- `Evidence Bundle` íntegro e cópia sabotada (`modify`, `delete`, `reorder`, `truncate`);
- baseline público de TI dos Tribunais de Contas (`docs/TCE-PUBLIC-INFRA-BASELINE.md`) e baseline técnico do HeraclitusDB (`docs/HERACLITUS-BASELINE.md`).

## 2. Mensagem Executiva em 30 Segundos (Para Presidência, Conselheiros, STI e Controle Externo)

> *"Tribunais de Contas já sofreram ataques severos de ransomware e tentativas de intrusão — entre 2020 e 2022 foi registrado um ataque a cada 41 dias no país, incluindo incidentes públicos no TCE-PR e no TCE-RS. Além disso, o uso crescente de IA na análise de editais e prestações de contas (como VigIA, ALICE e ANIA) abre o vetor de Prompt Injection oculto em PDFs enviados por jurisdicionados. Esta POC demonstra, em ambiente 100% sintético e isolado, como o HeraclitusDB atua ao lado dos bancos atuais do TCE (Oracle, PostgreSQL, SQL Server) e do e-TCE para correlacionar a intrusão desde o primeiro sinal, impedir alterações não autorizadas em processos ou pareceres (`upstream_delta = 0`), proteger agentes de IA com aprovação humana de uso único e exportar prova pericial verificável offline mesmo contra tentativas de apagamento de rastros."*

## 3. Perguntas de Discovery para a Secretaria de TI (STI) e Diretorias de Controle Externo do TCE

Sem solicitar qualquer informação sensível ou confidencial:
1. Quais **classes** de telemetria seriam prioritárias em um eventual piloto: perímetro/API de remessa de jurisdicionados, identidade (AD/SSO/ICP-Brasil), servidores Linux/Windows, auditoria de SGBD (Oracle/PostgreSQL/SQL Server) ou aplicação (`e-TCE`)?
2. O primeiro caso de uso de alto valor para o Tribunal seria **trilha imutável de auditoria do e-TCE / remessas (SIAFIC/AUDESP/SAGRES/SIM-AM)**, **correlação no SOC**, ou **governança e firewall documental para os agentes de IA de fiscalização de editais**?
3. Como o Tribunal avalia hoje a prova de integridade histórica exigida nos critérios de Governança de TI e Segurança da Informação do **MMD-TC (Atricon)**?
4. Quais operações críticas no `e-TCE` ou nos assistentes de IA exigiriam aprovação humana vinculada (*Human-in-the-Loop*) ou duplo controle?
5. Em um eventual piloto em homologação, qual unidade seria guardiã da política (STI / Segurança da Informação) e qual seria consumidora da prova pericial (Auditoria Interna / Corregedoria / Ministério Público de Contas)?
6. Qual infraestrutura institucional de chaves (ICP-Brasil / HSM / KMS) e armazenamento imutável (WORM / Object Lock) poderia ser qualificada em uma Fase 2 do piloto?

## 4. O Que Nunca Pedir na Reunião Inicial

- endereços IP internos ou topologia fina de rede;
- nomes reais de servidores ou regras de firewall/WAF;
- versões de appliances ou vulnerabilidades abertas;
- credenciais, dumps de banco ou logs reais sigilosos;
- processos de contas ou documentos reais sob segredo.
