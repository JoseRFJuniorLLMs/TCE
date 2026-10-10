# SPEC-0027 — TCE Public Environment Approximation Profile

**Status:** Proposed  
**Classe:** Environment Profile (Controle Externo / Tribunais de Contas)  
**Prioridade:** P0  
**Dependências:** Pesquisa pública (PDTI, PNCP, Atricon MMD-TC)

## 1. Objetivo

Orientar fixtures, adapters e componentes da POC com base em classes de tecnologia publicamente documentadas nos Planos Diretores de TI (PDTI) e contratações dos Tribunais de Contas dos Estados (TCE), sem reconstruir topologia sensível nem expor ativos reais de nenhum Tribunal.

## 2. Classes Confirmadas Publicamente nos TCEs

- **Processo Eletrônico de Controle Externo (`e-TCE` / `e-Contas`):** tramitação de Prestações de Contas Anuais (PCA), Tomadas de Contas Especiais, Atos de Pessoal, Denúncias, Representações, Alertas LRF e Acórdãos;
- **Sistemas de Coleta e Remessa Eletrônica de Jurisdicionados:** classe `AUDESP` (TCE-SP), `SAGRES` (TCE-PB/PE/PI/MA), `SIM-AM`/`SIT` (TCE-PR), `e-Sfinge` (TCE-SC), `CidadES` (TCE-ES), `SICOM` (TCE-MG), `SIGA` (TCE-RJ/BA) e `Geo-Obras`;
- **Integrações Estruturantes:** `SIAFIC` (Decreto nº 10.540/2020), `PNCP` (Lei nº 14.133/2021), `Siconfi`/`MSC` e `SEI` administrativo;
- **SGBDs Transacionais (OLTP):** coexistência típica entre **Oracle Database**, **PostgreSQL** e **Microsoft SQL Server**;
- **Plataforma Analítica, Data Lake e BI:** Data Warehouse / Data Lake para malhas de fiscalização contínua (folha de pagamento, empenhos, NF-e, obras e licitações) consumidos por **Power BI**, **Qlik Sense** ou **Superset**;
- **Segurança Cibernética e SOC 24x7:** Firewall NGFW, WAF de borda (Portal e API de Remessa), IAM/AD/SSO + **ICP-Brasil**, SIEM, EDR/XDR e **Backup Imutável / Appliance anti-ransomware**;
- **Inteligência Artificial aplicada ao Controle Externo:** análise automatizada de editais e licitações (**VigIA**, **ALICE**, **LicitIA**) e assistentes processuais/generativos (**ANIA**, **SEI-IA**, **ChatTCU**, **Ana JúlIA**, **Sofia**).

## 3. Perfil Sintético (`tce-public-approx-v1`)

```yaml
profile: tce-public-approx-v1
edge:
  firewall: vendor-neutral
  waf: vendor-neutral
  jurisdicionado_gateway: synthetic
identity:
  provider: vendor-neutral
  icp_brasil_class: synthetic
compute:
  linux: true
  windows_vdi: true
  ai_fiscalizacao_class: true
network:
  lan_wlan_vpn: true
data:
  transactional_db: engine-neutral # Oracle / PostgreSQL / SQL Server class
  warehouse_lake_malhas: true
applications:
  etce_audit_app: synthetic
  coleta_remessa_jurisdicionados: synthetic
  admin_sei_like: synthetic
  siafic_pncp_integration: synthetic
ai:
  edital_auditor_agent_class: true # VigIA / ALICE / ANIA class
protection:
  immutable_backup_appliance_class: true
```

## 4. Regra de Não-Inferência

Não fixar fabricante específico de firewall, WAF, SIEM, EDR, hypervisor ou topologia interna de rede sem confirmação pública e sem necessidade arquitetural. A POC opera por contratos de classe e adapters configuráveis.

## 5. Fontes

Consultar [`docs/TCE-PUBLIC-INFRA-BASELINE.md`](../docs/TCE-PUBLIC-INFRA-BASELINE.md).
