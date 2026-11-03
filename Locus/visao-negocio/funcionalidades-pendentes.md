# Funcionalidades Pendentes — LOCUS

Inventário das funcionalidades do LOCUS que **ainda precisam ser construídas**, consolidado a partir do menu do frontend (`src/componentes/layout/MenuLateral.itens.ts`), do questionário do PO (Product Owner) e da wiki SIGLA.

Os **fluxos de cada funcionalidade serão informados futuramente** pelo PO. Quando chegarem, cada item ganha uma página em [diagramas/funcionalidades](../diagramas/funcionalidades/index.md) a partir do [modelo de detalhamento](../diagramas/funcionalidades/modelo-funcionalidade.md), com diagrama C3 (componentes) / fluxo no drawio.

---

## 1. Situação por funcionalidade

| # | Menu / origem | Funcionalidade | Frontend | Backend | MVP | Fluxo recebido |
|---|---------------|----------------|----------|---------|-----|----------------|
| F01 | Cadastro › Inclusão | Inclusão de servidor / vínculo | Sem tela | MS-Locus (TO-BE) | Sim | Não |
| F02 | Cadastro › Atualização › Unidade de exercício | Alterar unidade de exercício | Sem tela | MS-Locus | Sim | Não |
| F03 | Cadastro › Atualização › Unidade de lotação | Alterar unidade de lotação (definitiva / precária) | Sem tela | MS-Locus | Sim | Não |
| F04 | Cadastro › Atualização › Sem RF ou código da unidade | Atualização sem RF (Registro Funcional) ou sem código de unidade | Sem tela | MS-Locus | Sim | Não |
| F05 | Cadastro › Atualização › Tornar um ato sem efeito | Reverter ato mantendo trilha | Sem tela | MS-Locus | Sim | Não |
| F06 | Cadastro › Atualização › Por RF | Atualização por RF | **Tela pronta** (dados estáticos) | MS-Locus | Sim | Parcial (tela) |
| F07 | Cadastro › Atualização › Vacância | Registro de vacância | Sem tela | MS-Locus | Sim | Não |
| F08 | Cadastro › Classificação | Classificação | Sem tela | A definir | A definir | Não |
| F09 | Gestão de UE (`/cadastro/gestao-unidades-educacionais`) | Listar, registrar, detalhar, excluir UE; módulos; lotados; afastados; histórico | **Telas prontas** (dados estáticos) | MS-Locus | Sim | Parcial (telas) |
| F10 | Relatórios consultas | Relatórios de lotação e quadro funcional | Sem tela | MS-Locus (assíncrono) | Sim (básicos) | Não |
| F11 | Data base | A definir com o PO | Sem tela | A definir | A definir | Não |
| F12 | Vagas | Saldo/disponibilidade de vagas por UE e módulo | Sem tela | MS-Locus | A definir | Não |
| F13 | Remoção | Processo de remoção | Sem tela | A definir | A definir | Não |
| F14 | Integração | Painel de integrações EOL/SIGPEC e divergências | Sem tela | MS-ImportaArquivos + MS-Locus | Sim | Proposta ([fluxo](../diagramas/funcionalidades/integracao-eol-sigpec.md)) |
| F15 | Wiki — Importação de dados | Cadastro de layout posicional, upload, validação, histórico | Sem tela | **MS-ImportaArquivos existente** | Sim | Sim (wiki) |
| F16 | Wiki — Cadastro de usuários | Usuários e permissões ALVO + LOCUS | Sem tela (portal só tem login) | MS-AdminUsuarios (parcial) | Sim | Sim (wiki) |

---

## 2. Perguntas em aberto para o PO

| # | Pergunta | Impacto arquitetural |
|---|----------|----------------------|
| Q1 | O que é "Data base" (data-base de referência do quadro? carga inicial?) | Modelo de snapshot / versionamento |
| Q2 | "Remoção" é o concurso de remoção da rede (com inscrição e classificação)? | Pode exigir novo domínio ou microsserviço |
| Q3 | "Classificação" refere-se a classificação de servidores para remoção/atribuição? | Regras e ordenação |
| Q4 | "Inclusão" cria servidor no LOCUS antes do EOL? Qual fonte prevalece? | Regra de conciliação |
| Q5 | Relatórios: ficam no MS-Locus ou reaproveitam o MS-Relatorios do ALVO? | Separação de domínios |
| Q6 | Mecanismo PRODAM e SIGPEC (API, view, arquivo, SFTP)? | Desenho do worker de integração |
| Q7 | Dados de saúde/afastamento são armazenados? | Classificação LGPD (Lei Geral de Proteção de Dados) e mascaramento |
| Q8 | Perfis finais (Administrador, Operador DICAR, Consulta) e abrangência por DRE? | Modelo de permissões |

---

## 3. Como registrar um novo fluxo

1. Copiar [modelo-funcionalidade.md](../diagramas/funcionalidades/modelo-funcionalidade.md) para `diagramas/funcionalidades/Fxx-nome.md`.
2. Adicionar uma página `FUNC - Fxx <nome>` no `ARQUITETURA - LOCUS - C4.drawio` usando a legenda C4 (página 1).
3. Exportar o SVG para `diagramas/assets/` e atualizar a tabela de [assets](../diagramas/assets/index.md).
4. Atualizar a coluna **Fluxo recebido** desta página.
