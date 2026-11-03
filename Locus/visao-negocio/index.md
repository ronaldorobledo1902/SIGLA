# Visão de Negócio — LOCUS (Lotação e Controle Unificado de Servidores)

**Produto:** LOCUS (subsistema do SIGLA)  
**Órgão:** SME (Secretaria Municipal de Educação) de São Paulo — COGEP (Coordenadoria de Gestão de Pessoas) / DICAR (Divisão de Gestão de Carreiras)  
**Data:** Outubro/2026  
**Versão:** 1.0  
**Público-alvo:** Product Owner, DICAR, stakeholders de negócio e time multidisciplinar  

```{toctree}
:maxdepth: 1
:caption: Tópicos

funcionalidades-pendentes
```

---

## 1. Propósito deste documento

Consolidar a **visão de negócio** do LOCUS com base no questionário respondido pelo PO (Product Owner) (`Levantamento/Questionário_PO_LOCUS_respondido.docx`), na wiki de arquitetura do SIGLA (Azure DevOps — *Visão Arquitetural*), no frontend LOCUS em construção (`SME-SIGLA-LOCUS-Frontend`) e nos microsserviços SIGLA já existentes.

O LOCUS **não é portal público** e **não é acessado pelos servidores** cujos dados gerencia: é uma ferramenta interna da DICAR.

---

## 2. Visão do produto

Para a **DICAR/COGEP**, o **LOCUS** é a plataforma única para:

- **gerir os dados** de servidores ativos e inativos (efetivos, comissionados, contratados e admitidos);
- **controlar a lotação e o exercício** nas Unidades Educacionais, DREs (Diretorias Regionais de Educação) e órgãos centrais;
- **controlar movimentações**, vacâncias e disponibilidade (saldo de vagas por módulo);
- **confrontar** continuamente os dados com o **EOL (Escola Online / PRODAM)** e o **SIGPEC (SEGES — Secretaria de Gestão)**;
- **preservar o histórico** de registros com acesso a versões anteriores;
- **apoiar decisões** com relatórios do quadro funcional.

Dentro do SIGLA, o LOCUS também é o **importador de dados** (layouts posicionais, arquivos SIGPEC) e o **gestor de permissões** de usuários dos dois subsistemas (ALVO e LOCUS)

---

## 3. Problema que resolve


| Situação anterior / legado                              | Dor                                              | Solução proposta                                                            |
| ------------------------------------------------------- | ------------------------------------------------ | --------------------------------------------------------------------------- |
| Sistemas descontinuados, sem atualização de legislação  | Regras novas não implementáveis; ajustes manuais | Plataforma nova com regras parametrizáveis (motivos, portarias)             |
| Dados de servidores espalhados (EOL, SIGPEC, planilhas) | Inconsistências e erros de alocação              | Consumo diário EOL/SIGPEC + **conciliação** com sinalização de divergências |
| Alterações sem rastreabilidade                          | Revisões e auditorias difíceis                   | **Histórico de versões** por registro, somente leitura                      |
| Falta de relatórios detalhados                          | Decisões de alocação sem dados atualizados       | Relatórios de lotação e quadro funcional (pesados de forma assíncrona)      |
| Lançamentos manuais repetidos                           | Retrabalho e risco de erro                       | Importação por layout e atualização em lote a partir das fontes oficiais    |


---

## 4. Objetivos de negócio

1. Prover **visão integrada e atualizada** do quadro funcional da SME (mais de 100 mil servidores, ~1.600 UEs).
2. Reduzir **erros de alocação** e **lançamentos manuais**.
3. Garantir **rastreabilidade** (quem, quando, o que mudou e por qual ato).
4. Cumprir a **LGPD (Lei Geral de Proteção de Dados)** e as normas municipais de gestão de pessoal.
5. Substituir os sistemas descontinuados com urgência, sem prazo regulatório formal.

---

## 5. Personas


| Persona                                  | Necessidade principal                                                                                                                                     |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Operador DICAR**                       | Consultar servidor (RF (Registro Funcional), CPF, nome) e unidade; registrar e corrigir lotação, exercício, movimentação e vacância com motivo e portaria |
| **Administrador SIGLA**                  | Cadastrar usuários, atribuir perfis/permissões de ALVO e LOCUS, manter layouts de importação                                                              |
| **Consulta (Jurídico SME, SEGES, DREs)** | Leitura e relatórios com acesso mínimo necessário — **fase futura**, a definir                                                                            |
| **Gestor COGEP**                         | Relatórios do quadro funcional para planejamento de RH com reflexo no planejamento pedagógico                                                             |
| **Operador ALVO** (sistema vizinho)      | Consome dados importados no LOCUS e depende das permissões cadastradas nele                                                                               |


**Premissa de acesso:** usuários **internos**; login com credencial única da SME (CoreSSO via API SME Integração). Provedor corporativo definitivo (Keycloak) **em estudo** pela SME-COTIC. MFA (autenticação multifator) recomendado.

---

## 6. Escopo do produto

### 6.1 — Em escopo (MVP)


| Capacidade                                                                                                            | Situação (out/2026)                                                    |
| --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Gestão de Unidades Educacionais (listar, filtrar, registrar, detalhar, excluir; módulos, lotados, afastados)          | Tela pronta, **dados estáticos**                                       |
| Atualização por RF (dados funcionais, lotação/exercício, readaptação, encerramento, registrar atualização com motivo) | Tela pronta, **dados estáticos**                                       |
| Histórico e versão anterior da UE (banner de versão histórica)                                                        | Tela pronta, dados estáticos                                           |
| Atualizações de unidade de exercício / lotação, sem RF, ato sem efeito, vacância                                      | **Pendente** (menu sem tela)                                           |
| Inclusão e Classificação                                                                                              | **Pendente**                                                           |
| Integração EOL / SIGPEC com confronto                                                                                 | **Pendente**                                                           |
| Importação por layout posicional e histórico de importações                                                           | Backend existente (MS-ImportaArquivos); telas LOCUS **pendentes**      |
| Usuários e permissões (ALVO + LOCUS)                                                                                  | Backend parcial (MS-AdminUsuarios); perfis LOCUS e telas **pendentes** |
| Relatórios básicos de lotação e quadro funcional                                                                      | **Pendente**                                                           |


O detalhamento por item está em [Funcionalidades pendentes](funcionalidades-pendentes.md).

### 6.2 — Fora de escopo (por desenho)

- Convocação, agendamento e escolha de vagas (subsistema **ALVO**)
- Acesso de cidadãos ou dos próprios servidores
- Ser origem oficial dos dados funcionais (continua EOL / SIGPEC)
- Folha de pagamento

### 6.3 — Fases futuras

- Painéis analíticos e relatórios avançados
- Alertas automáticos de divergência e de vagas/disponibilidade
- Acesso de outras áreas (DREs, SEGES, Jurídico) com perfil Consulta
- Simulações de movimentação para planejamento

### 6.4 — Dependências de dados (visão de negócio)


| Dado                                                     | Origem                       | Mecanismo                                                |
| -------------------------------------------------------- | ---------------------------- | -------------------------------------------------------- |
| Servidores, cargos, vínculos, unidades, DREs             | EOL (PRODAM)                 | **A confirmar** (API, view, arquivo) — ao menos 1x/dia   |
| Dados funcionais, processos seletivos, lotes de nomeação | SIGPEC (SEGES)               | Arquivo TXT/CSV (manual hoje); automação **a confirmar** |
| Credenciais                                              | CoreSSO (API SME Integração) | REST, já em uso no SIGLA                                 |
| Motivos de atualização, portarias, características da UE | LOCUS                        | Cadastro próprio                                         |


---

## 7. Jornadas do usuário

### Jornada A — Integração e conciliação diária (proposta do PO)

1. O sistema importa periodicamente os dados do EOL e do SIGPEC.
2. O sistema confronta as informações e **sinaliza divergências**.
3. O analista DICAR consulta o servidor ou a unidade.
4. Registra ou corrige a lotação/movimentação.
5. O sistema grava a alteração no histórico, **preservando a versão anterior**.
6. O analista ou gestor gera relatórios do quadro funcional.
7. Em revisão ou auditoria, consulta-se o histórico de versões do registro.

### Jornada B — Atualização por RF (tela existente)

Operador busca por RF, CPF ou nome → abre o detalhe do servidor → consulta dados funcionais, lotação/exercício, atividade e readaptação → **registra atualização** (motivo, portaria, data, documento) → a alteração passa a compor o histórico.

### Jornada C — Gestão de Unidade Educacional (tela existente)

Operador filtra unidades por DRE, tipo e características → abre o detalhe → vê módulos por componente curricular, saldo de vagas (disponível / excedente / completo), lotados (definitivo / precário) e afastados → registra ou exclui unidade → consulta versões anteriores.

### Jornada D — Importação de arquivo

Administrador cadastra o **layout posicional** (txt/csv, posição e tamanho de cada campo) → operador faz upload → o MS-ImportaArquivos valida contra o layout e registra erros → dados válidos seguem para a base SIGLA → histórico de importações disponível.

### Jornada E — Usuários e permissões 

Administrador acessa o LOCUS → cadastra/consulta usuário (dados do CoreSSO/EOL) → atribui acessos de ALVO e LOCUS → o backend grava na base de escrita SIGLA.

---

## 8. Roadmap de produto (proposto)


| Fase                   | Entregas                                                                                                                           | Dependências                        |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| **0 — Fundação**       | MS-Locus (esqueleto padrão SIGLA), perfis LOCUS no AdminUsuarios, AuthGuard no frontend, integração das telas existentes com a API | ADR-004, ADR-003                    |
| **1 — MVP cadastro**   | UE, servidor por RF, atualizações (lotação, exercício, vacância, ato sem efeito), histórico                                        | Contratos já desenhados no frontend |
| **2 — MVP integração** | Importação EOL/SIGPEC via MS-ImportaArquivos, conciliação diária, tela de divergências                                             | Mecanismo PRODAM/SIGPEC confirmado  |
| **3 — MVP relatórios** | Lotação e quadro funcional, exportação assíncrona                                                                                  | Read model (CQRS)                   |
| **4 — Evolução**       | Painéis, alertas, perfil Consulta para outras áreas, simulações                                                                    | Definições do PO                    |


---

## 9. Stakeholders e governança


| Stakeholder                  | Papel                                                          |
| ---------------------------- | -------------------------------------------------------------- |
| COGEP / DICAR                | Dona do produto e principal usuária                            |
| PO (Product Owner) LOCUS     | Prioriza backlog e define fluxos das funcionalidades pendentes |
| PRODAM                       | Disponibiliza dados do EOL                                     |
| SEGES (Secretaria de Gestão) | Responsável pelo SIGPEC                                        |
| Jurídico SME                 | Retenção de dados, normas de gestão de pessoal, LGPD           |
| SME-COTIC                    | Infraestrutura, identidade corporativa (Keycloak), padrões     |
| Infra Ágil                   | Jenkins, Rancher, Registry                                     |
| Squad SIGLA (Spassu)         | Desenvolvimento ALVO e LOCUS                                   |


---

## 10. Regras e premissas de negócio relevantes

- LOCUS **não é origem oficial**; em divergência, a correção é decidida pela DICAR e registrada com ato (portaria).
- Toda alteração funcional exige **motivo** (catálogo de ~90 motivos no frontend), **data** e, quando aplicável, **portaria/documento**.
- Vacância decorre de encerramento do vínculo; "tornar ato sem efeito" reverte um ato anterior mantendo a trilha.
- Lotação pode ser **definitiva** ou **precária**; afastamentos (ex.: licença médica, afastamento para estudo) impactam o saldo de vagas.
- Saldo de vagas por módulo/componente: **disponível**, **excedente** ou **completo**.
- Unidade pode ser marcada para **não contabilizar** no quadro, com motivo obrigatório.
- Integração com EOL/SIGPEC **ao menos 1 vez ao dia**.
- Histórico preservado; prazo de retenção a definir com o Jurídico (tabela de temporalidade).

---

## 11. Riscos de negócio


| Risco                                                | Impacto                                            | Mitigação                                                         |
| ---------------------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------------- |
| Mecanismo de integração com PRODAM/SIGPEC indefinido | Conciliação (núcleo do MVP) bloqueada              | Priorizar reunião técnica com PRODAM e SEGES; começar por arquivo |
| Perfis de acesso não definidos                       | Exposição de dados pessoais de 100 mil+ servidores | ADR-003; perfis mínimos antes de produção                         |
| Dados sensíveis (saúde/afastamento) não confirmados  | Requisitos LGPD subdimensionados                   | Validar com Jurídico; mascaramento por padrão em relatórios       |
| Dependência de fornecedores externos                 | Atrasos fora do controle do time                   | Contratos de interface versionados; mocks no MS-ImportaArquivos   |
| Fluxos das funcionalidades pendentes não descritos   | Retrabalho de modelagem                            | Modelo de detalhamento em `diagramas/funcionalidades`             |
| Picos em início de ano letivo, atribuição e remoção  | Lentidão em consultas/relatórios                   | Read model + relatórios assíncronos                               |


---

## 12. Critérios de sucesso (negócio)

KPIs não foram definidos formalmente; propostos pelo PO:


| Indicador                                 | Meta sugerida (a validar com o PO)                                |
| ----------------------------------------- | ----------------------------------------------------------------- |
| Servidores com lotação atualizada         | ≥ 98% após a 1ª conciliação estável                               |
| Divergências LOCUS × EOL/SIGPEC           | Tendência de queda; tempo médio de resolução monitorado           |
| Correções e lançamentos manuais           | Redução mensurável frente ao sistema atual                        |
| Tempo de geração de relatórios gerenciais | Consultas ≤ 3 s; relatórios pesados entregues de forma assíncrona |
| Disponibilidade em horário comercial      | 99,5%                                                             |


