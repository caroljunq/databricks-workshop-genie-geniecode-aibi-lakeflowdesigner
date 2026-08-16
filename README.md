# databricks-workshop-genie-geniecode-aibi-lakeflowdesigner

Workshop Databricks para times de negócio. O material cobre upload de dados, Genie Code, Genie Agents, Genie One, AI Functions, Lakeflow Designer (opcional) e AI/BI Dashboards.

---

## Estrutura do Repositório

```
.
├── notebook_guia_workshop.ipynb   # Guia passo a passo do workshop
└── dados/
    ├── clientes.csv               # Cadastro de clientes
    ├── contratos.csv              # Contratos / propostas / grupos
    ├── parcelas_cobranca.csv      # Parcelas, inadimplência e etapas de cobrança
    ├── comunicacoes.csv           # Comunicações enviadas e agendadas
    └── jornada_cnh_mercado.xlsx   # (Opcional) Jornada CNH + sazonalidade de concorrentes
```

---

## Contexto de negócio

A organização fabrica veículos e opera serviços financeiros (financiamento, CDC, leasing e consórcio). Os times presentes querem responder, entre outros:

| Tema | Perguntas típicas |
|---|---|
| Inadimplência e eficiência de processos | Quais fatores influenciam a inadimplência? Onde estão os gargalos do processo? |
| Previsão de comunicações | Quantas comunicações serão enviadas nos próximos meses? |
| Sucesso vs cancelamento | Quais características de clientes, propostas e grupos estão associadas a sucesso ou cancelamento? |
| Recuperação | Quais clientes apresentam maior potencial de recuperação? |
| Jornada CNH | Quais dados da jornada CNH melhoram indicadores de pendência e entrega? |
| Mercado / concorrentes | Qual a sazonalidade dos agentes financeiros concorrentes e o que influencia essa flutuação? |

---

## Módulos do workshop

| # | Módulo | Obrigatório? | O que você faz |
|---|---|---|---|
| 1 | **Upload de arquivos (CSV)** | Sim | Subir os 4 CSVs e criar tabelas no Unity Catalog |
| 2 | **XLSX + Genie Code + Notebook** | Opcional | Subir o XLSX, limpar/transformar com Genie Code e criar tabelas |
| 3 | **Lakeflow Designer** | Opcional | Pipeline visual (join, filtro, agregação) |
| 4 | **AI Functions** | Opcional | `ai_gen` / `ai_classify` em SQL |
| 5 | **Genie Agents** | **Obrigatório** | Perguntas analíticas e hipotéticas em linguagem natural |
| 6 | **Genie One** | Sim | Explorar no Genie One e agendar um report diário |
| 7 | **AI/BI Dashboards** | Sim | Dashboard com 2 páginas |

---

## Pasta `dados/`

| Arquivo | Tabela sugerida | Linhas (approx.) | Descrição |
|---|---|---|---|
| `clientes.csv` | `{prefixo}_clientes` | 250 | Perfil demográfico, score, renda, segmento e canal |
| `contratos.csv` | `{prefixo}_contratos` | ~330 | Propostas, produtos, grupos de consórcio, sucesso/cancelamento |
| `parcelas_cobranca.csv` | `{prefixo}_parcelas_cobranca` | ~4.700 | Parcelas, atraso, etapas de cobrança e potencial de recuperação |
| `comunicacoes.csv` | `{prefixo}_comunicacoes` | ~1.400 | Histórico + agendamentos futuros (para previsão de envios) |
| `jornada_cnh_mercado.xlsx` | `{prefixo}_jornada_cnh` / `{prefixo}_mercado_concorrentes` | 180 / 224 | Abas: jornada CNH (datas BR) e volume dos concorrentes |

> Prefixo: use um identificador único por participante (ex.: `ana_`, `joao_`) para evitar conflito de tabelas.

---

## Schema das Tabelas

### `clientes`

Cadastro de clientes do braço financeiro (financiamento, CDC, leasing e consórcio).

| Coluna | Tipo | Descrição |
|---|---|---|
| `cliente_id` | INTEGER | Identificador único do cliente |
| `nome` | STRING | Nome completo do cliente |
| `cpf_mascarado` | STRING | CPF parcialmente oculto para privacidade (ex.: `***.132.***-**`) |
| `idade` | INTEGER | Idade do cliente em anos |
| `renda_mensal` | FLOAT | Renda mensal declarada em R$ |
| `score_credito` | INTEGER | Score de crédito (faixa típica 300–950) |
| `cidade` | STRING | Cidade de residência |
| `estado` | STRING | UF da residência |
| `segmento` | STRING | Perfil do cliente: `PF`, `PJ Frota` ou `Consorciado` |
| `canal_aquisicao` | STRING | Canal de origem: `Concessionária`, `Digital`, `Telefone` ou `Parceiro` |
| `data_cadastro` | DATE | Data em que o cliente foi cadastrado |
| `ativo` | BOOLEAN | Indica se o cliente está ativo (`true` / `false`) |

---

### `contratos`

Contratos e propostas de crédito vinculados aos clientes (produto, veículo, grupo e desfecho).

| Coluna | Tipo | Descrição |
|---|---|---|
| `contrato_id` | INTEGER | Identificador único do contrato |
| `cliente_id` | INTEGER | Referência ao cliente (`clientes.cliente_id`) |
| `proposta_id` | INTEGER | Identificador da proposta comercial/crédito |
| `grupo_consorcio` | STRING | Código do grupo de consórcio (ex.: `G001`); vazio se não for consórcio |
| `tipo_produto` | STRING | Produto: `Financiamento Veículo`, `Consórcio`, `CDC` ou `Leasing` |
| `modelo_veiculo` | STRING | Modelo fictício do veículo (Aurora, Nimbus, Vértice, Atlas, etc.) |
| `categoria_veiculo` | STRING | Categoria do veículo: `Passeio` ou `SUV` |
| `valor_financiado` | FLOAT | Valor financiado em R$ |
| `entrada_pct` | FLOAT | Percentual de entrada sobre o valor (ex.: `0.20` = 20%) |
| `prazo_meses` | INTEGER | Prazo do contrato em meses |
| `taxa_juros_am` | FLOAT | Taxa de juros ao mês (%) |
| `status_contrato` | STRING | Situação atual: `Ativo`, `Quitado`, `Cancelado` ou `Inadimplente` |
| `motivo_cancelamento` | STRING | Motivo do cancelamento (quando `status_contrato = Cancelado`) |
| `data_contratacao` | DATE | Data de contratação / início do contrato |
| `data_status` | DATE | Data da última atualização do status |
| `aprovado` | BOOLEAN | Indica se a proposta foi aprovada |
| `sucesso` | BOOLEAN | Desfecho positivo (`true` para Ativo/Quitado; `false` para Cancelado/Inadimplente) |

---

### `parcelas_cobranca`

Parcelas dos contratos, com inadimplência, etapas do processo de cobrança e potencial de recuperação.

| Coluna | Tipo | Descrição |
|---|---|---|
| `parcela_id` | INTEGER | Identificador único da parcela |
| `contrato_id` | INTEGER | Referência ao contrato (`contratos.contrato_id`) |
| `cliente_id` | INTEGER | Referência ao cliente (`clientes.cliente_id`) |
| `numero_parcela` | INTEGER | Número sequencial da parcela no contrato |
| `data_vencimento` | DATE | Data de vencimento da parcela |
| `valor_parcela` | FLOAT | Valor da parcela em R$ |
| `status_parcela` | STRING | Situação: `Paga`, `Pendente`, `Vencida` ou `Renegociada` |
| `dias_atraso` | INTEGER | Quantidade de dias em atraso (0 se em dia) |
| `etapa_cobranca` | STRING | Etapa do processo: `Sem Cobrança`, `SMS`, `Email`, `Ligação`, `Cartório` ou `Jurídico` |
| `data_pagamento` | STRING/DATE | Data em que a parcela foi paga/recuperada (vazio se em aberto) |
| `valor_recuperado` | FLOAT | Valor efetivamente recuperado em R$ |
| `potencial_recuperacao` | STRING | Classificação de recuperação: `Alto`, `Médio` ou `Baixo` |
| `agente_cobranca` | STRING | Equipe/agente responsável pela cobrança |
| `mes` | INTEGER | Mês de vencimento da parcela (1–12) |
| `ano` | INTEGER | Ano de vencimento da parcela |

---

### `comunicacoes`

Histórico e fila de comunicações (enviadas e agendadas) para cobrança, oferta, CNH e entrega.

| Coluna | Tipo | Descrição |
|---|---|---|
| `comunicacao_id` | INTEGER | Identificador único da comunicação |
| `cliente_id` | INTEGER | Referência ao cliente (`clientes.cliente_id`) |
| `contrato_id` | INTEGER | Referência ao contrato (`contratos.contrato_id`) |
| `tipo_comunicacao` | STRING | Canal: `SMS`, `Email`, `WhatsApp`, `Carta` ou `Push` |
| `campanha` | STRING | Campanha: `Cobrança`, `Renovação`, `Oferta Cross-sell`, `Lembrete Parcela`, `CNH Pendência` ou `Entrega Veículo` |
| `data_prevista_envio` | DATE | Data prevista de envio (inclui datas futuras para previsão) |
| `data_envio` | STRING/DATE | Data efetiva do envio (vazio se ainda não enviado) |
| `status_envio` | STRING | Status: `Enviado`, `Agendado`, `Falhou` ou `Cancelado` |
| `resultado` | STRING | Resultado: `Respondido`, `Sem Resposta`, `Pagou` ou `Opt-out` |
| `mes_referencia` | INTEGER | Mês da data prevista de envio |
| `ano_referencia` | INTEGER | Ano da data prevista de envio |
| `custo_estimado_rs` | FLOAT | Custo unitário estimado da comunicação em R$ |

---

### XLSX — aba `jornada_cnh` (opcional)

Pendências e entregas da jornada CNH. Colunas com nomes “sujos” e datas em `dd/mm/yyyy` de propósito (exercício de Genie Code).

| Coluna | Descrição |
|---|---|
| `ID Jornada` | Identificador da jornada CNH |
| `ID Cliente` | Referência ao cliente |
| `ID Contrato` | Referência ao contrato |
| `Status CNH` | Status: `Entregue`, `Pendente Documentação`, `Pendente Exame` ou `Em Análise` |
| `Tipo Pendencia` | Motivo da pendência (ex.: documento ilegível, CPF divergente) ou `Sem pendência` |
| `Data Solicitacao` | Data da solicitação (formato `dd/mm/yyyy`) |
| `Data Prevista Entrega` | Data prevista de entrega (formato `dd/mm/yyyy`) |
| `Data Entrega Real` | Data real de entrega (formato `dd/mm/yyyy`; vazio se não entregue) |
| `Dias Atraso Entrega` | Dias de atraso em relação à data prevista |
| `Etapa Jornada` | Etapa: `Cadastro`, `Documentação`, `Análise`, `Produção CNH`, `Expedição` ou `Entrega` |
| `Canal Entrega` | Canal: `Correios`, `Retirada Concessionária`, `Motoboy` ou `Digital` |
| `Flag Pendencia` | Indica pendência: `SIM` / `NAO` |
| `Entrega No Prazo` | Indica entrega no prazo: `SIM` / `NAO` (vazio se ainda não entregue) |
| `Score Experiencia` | Nota de experiência do cliente na jornada (0–100) |
| `Observacao` | Comentário livre operacional |

### XLSX — aba `mercado_concorrentes` (opcional)

Série mensal de volume e indicadores dos agentes financeiros concorrentes.

| Coluna | Descrição |
|---|---|
| `mercado_id` | Identificador do registro mensal |
| `ano` | Ano de referência |
| `mes` | Mês de referência (1–12) |
| `agente_financeiro` | Agente fictício: `Banco Horizon`, `CredAuto Novaera`, `Financiadora Veloce`, `AutoCred Solaris`, `Capital Motora`, `Bancos Tradicionais` ou `Fintechs Auto` |
| `volume_contratos` | Volume de contratos no mês |
| `ticket_medio_rs` | Ticket médio dos contratos em R$ |
| `taxa_media_am` | Taxa média ao mês praticada (%) |
| `taxa_selic_ref` | Taxa Selic de referência no período (%) |
| `ipca_ref` | IPCA de referência no período (%) |
| `participacao_mercado_pct` | Participação de mercado estimada (%) |
| `campanha_ativa` | Campanha vigente no mês (ex.: `Juros zero`, `Entrada facilitada`) |

---

## Relações entre as Tabelas

```
clientes (cliente_id)
   ├──< contratos (cliente_id)
   │       ├──< parcelas_cobranca (contrato_id)
   │       ├──< comunicacoes (contrato_id)
   │       └──< jornada_cnh (ID Contrato)   [opcional]
   ├──< parcelas_cobranca (cliente_id)
   └──< comunicacoes (cliente_id)

mercado_concorrentes                         [opcional, sem FK]
```

---

## Como usar

1. Clone ou baixe este repositório.
2. Importe `notebook_guia_workshop.ipynb` no Databricks.
3. Baixe a pasta `dados/` e siga o Módulo 1 (upload dos CSVs).
4. (Opcional) Faça o Módulo 2 com o XLSX + Genie Code.
5. Complete Genie Agents, Genie One (com agendamento diário) e o dashboard de 2 páginas.

---

## Pré-requisitos no workspace

- Acesso a um **SQL Warehouse**
- Permissão para criar tabelas em um **catálogo/schema** do Unity Catalog
- (Recomendado) acesso a **Genie**, **Genie One**, **AI/BI Dashboards** e, se disponível, **Lakeflow Designer** / **Genie Code**
