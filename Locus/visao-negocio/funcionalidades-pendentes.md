# Funcionalidades Pendentes — LOCUS

Inventário das funcionalidades do LOCUS que **ainda precisam ser construídas**, consolidado a partir do menu do frontend (`src/componentes/layout/MenuLateral.itens.ts`) e do questionário do PO (Product Owner).

O módulo Fundamental II e Médio tem os cinco fluxos levantados (novembro/2026): Cadastro, Relatórios e consultas, Data base, Vagas e Remoção. O texto e os containers estão em [Fundamental II e Médio](../visao-arquitetural/proposta-fundamental-ii-medio.md) e na [página de diagramas](../diagramas/funcionalidades/fundamental-ii-medio.md).

Nesse recorte, F01 a F07 e F10 a F13 já têm fluxo. F08 (Classificação) aparece no Cadastro, com a operação ainda a mapear. Nenhuma dessas telas está construída, salvo a atualização por RF (F06), que segue com dados estáticos. F09, F14, F15 e F16 não fazem parte desse levantamento. Os outros dez módulos seguem sem levantamento.

---

## 1. Situação por funcionalidade

| # | Menu / origem | Funcionalidade | Frontend | Backend | MVP | Fluxo recebido |
|---|---------------|----------------|----------|---------|-----|----------------|
| F01 | Cadastro › Inclusão | Inclusão de servidor / vínculo. Substituição e cargo vago em dois passos; parâmetros a mapear | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F02 | Cadastro › Atualização › Unidade de exercício | Alterar unidade de exercício | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F03 | Cadastro › Atualização › Unidade de lotação | Alterar unidade de lotação (definitiva / precária) | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F04 | Cadastro › Atualização › Sem RF ou código da unidade | Atualização sem RF (Registro Funcional) ou sem código de unidade. Parâmetro a mapear | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F05 | Cadastro › Atualização › Tornar um ato sem efeito | Reverter ato mantendo trilha | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F06 | Cadastro › Atualização › Por RF | Atualização por RF | **Tela pronta** (dados estáticos) | MS-Locus | Sim | Sim (Fund. II) |
| F07 | Cadastro › Atualização › Vacância | Registro de vacância | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F08 | Cadastro › Classificação | Consulta e ajuste da classificação no Cadastro. Operação a mapear | Sem tela | MS-Locus | Sim | Parcial (Fund. II) |
| F09 | Gestão de UE (`/cadastro/gestao-unidades-educacionais`) | Listar, registrar, detalhar, excluir UE; módulos; lotados; afastados; histórico | **Telas prontas** (dados estáticos) | MS-Locus | Sim | Parcial (telas) |
| F10 | Relatórios consultas | Relatórios e consultas do Fund. II: banco de servidores, quadro, cargos, mensal e auditorias. Parte do conteúdo a mapear | Sem tela | MS-Locus | Sim (básicos) | Sim (Fund. II) |
| F11 | Data base | Datas de referência, migração de candidatos, extratos | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F12 | Vagas | Cálculo, EXC, confronto e arquivo para a Escolha | Sem tela dedicada; a gestão de UEs antecipa a consulta | MS-Locus | Sim | Sim (Fund. II) |
| F13 | Remoção | Preparação da remoção (confronto, precários, congelado). Carga ainda a mapear | Sem tela | MS-Locus | Sim | Sim (Fund. II) |
| F14 | Integração | Painel de integrações EOL/SIGPEC e divergências | Sem tela | MS-ImportaArquivos + MS-Locus | Sim | Proposta ([fluxo](../diagramas/funcionalidades/integracao-eol-sigpec.md)) |
| F15 | Importação de dados | Cadastro de layout posicional, upload, validação, histórico | Sem tela | **MS-ImportaArquivos existente** | Sim | Sim |
| F16 | Cadastro de usuários | Usuários e permissões ALVO + LOCUS | Sem tela (portal só tem login) | MS-AdminUsuarios (parcial) | Sim | Sim |

---

## 2. Perguntas em aberto para o PO

| # | Pergunta | Impacto arquitetural |
|---|----------|----------------------|
| Q1 | Como a Data base EOL é atualizada, o que a checagem de módulo verifica e qual a diferença entre os dois botões nominais do extrato? | O fluxo 3 já define datas, migração, controle e extratos. Esses três pontos seguem abertos |
| Q2 | A carga da remoção entra no MS-Locus? Inscrição e classificação do concurso não estão no fluxo | O fluxo 5 prepara o concurso (confronto, precários, congelados e vagas) no MS-Locus. A carga está a mapear |
| Q3 | O que a operação Classificação consulta e grava? O botão está no Cadastro e o parâmetro está a mapear | Regra de ordenação do Fluxo 1 |
| Q4 | "Inclusão" cria servidor no LOCUS antes do EOL? Qual fonte prevalece? | Regra de conciliação. Substituição e cargo vago têm o parâmetro a mapear |
| Q5 | Os relatórios dos outros módulos ficam no MS-Locus ou reaproveitam o MS-Relatorios do ALVO? | No Fund. II o relatório fica no MS-Locus e só lê a base de leitura |
| Q6 | Mecanismo PRODAM e SIGPEC (API, view, arquivo, SFTP)? | Desenho do worker de integração |
| Q7 | Dados de saúde/afastamento são armazenados? | Classificação LGPD (Lei Geral de Proteção de Dados) e mascaramento |
| Q8 | Perfis finais (Administrador, Operador DICAR, Consulta) e abrangência por DRE? | Modelo de permissões |

---

## 3. Como registrar um novo fluxo

1. Copiar [modelo-funcionalidade.md](../diagramas/funcionalidades/modelo-funcionalidade.md) para `diagramas/funcionalidades/Fxx-nome.md`.
2. Adicionar uma página `FUNC - Fxx <nome>` no `ARQUITETURA - LOCUS - C4.drawio` usando a legenda C4 (página 1).
3. Exportar o SVG para `diagramas/assets/` e atualizar a tabela de [assets](../diagramas/assets/index.md).
4. Atualizar a coluna **Fluxo recebido** desta página.

## Referências

- Wiki Azure DevOps *COTIC - DevOps › SIGLA › Visão Arquitetural* — Jornadas › Importação de dados LOCUS e Cadastro de usuários (origem de F15 e F16)
