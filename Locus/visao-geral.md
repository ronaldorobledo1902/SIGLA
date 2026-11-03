# Visão Geral

O **LOCUS (Lotação e Controle Unificado de Servidores)** é o subsistema do **SIGLA** responsável pelo controle do quadro funcional da SME (Secretaria Municipal de Educação) de São Paulo: mais de **100 mil servidores** (ativos e inativos, efetivos, comissionados, contratados e admitidos) e cerca de **1.600 Unidades Educacionais**, além das DREs (Diretorias Regionais de Educação) e órgãos centrais.

Substitui sistemas descontinuados que não recebem mais atualizações de legislação. Os dados **oficiais** continuam no **EOL (Escola Online / PRODAM)** e no **SIGPEC**; o LOCUS consome, confronta, complementa e historiza essas informações.

## O que o sistema oferece (alvo do MVP)

- Cadastro e consulta de servidores (todos os tipos de vínculo) por RF (Registro Funcional), CPF ou nome
- Gestão de Unidades Educacionais: características, módulos por componente curricular, saldo de vagas, lotados e afastados
- Registro de atualizações funcionais (unidade de lotação, unidade de exercício, vacância, ato sem efeito) com motivo, portaria e documento
- Integração com EOL e SIGPEC ao menos 1 vez ao dia, com **confronto de dados** e sinalização de divergências
- Histórico de registros e acesso a **versões anteriores** (revisão e auditoria)
- Relatórios básicos de lotação e quadro funcional
- Administração de usuários e permissões do SIGLA (LOCUS e ALVO)

## Como a solução está organizada

| Camada | Componentes | Situação (out/2026) |
|--------|-------------|---------------------|
| **Portal / login** | Portal SIGLA (React) — login RF/senha, recuperação de senha, seleção ALVO / LOCUS | Existente |
| **Interface LOCUS** | `SME-SIGLA-LOCUS-Frontend` — React 19, Vite 7, Ant Design 5, React Query, servido por Nginx em `/locus/` | Em construção, **dados estáticos** |
| **Identidade e permissões** | `SME-SIGLA-MS-AdminUsuarios` — Django 5.2 / DRF (Django REST Framework), CoreSSO via API SME Integração, JWT (JSON Web Token) | Existente (perfis ALVO) |
| **Importação de dados** | `SME-SIGLA-MS-ImportaArquivos` — Django / DRF + Celery, layouts posicionais, PRODAM, SIGPEC | Existente (uso ALVO), **reaproveitado** pelo LOCUS |
| **Domínio LOCUS** | **MS-Locus** — Django / DRF + Celery + `sigla-sdk` | **Planejado (TO-BE)** |
| **Dados** | PostgreSQL SIGLA (escrita / leitura — CQRS (Command Query Responsibility Segregation)), Redis (broker Celery) | Existente / leitura planejada |
| **Runtime** | Kubernetes (Rancher), Ingress Nginx como API Gateway, Jenkins + Registry SME | A confirmar com Infra Ágil |

## Premissas importantes

- Usuários **internos** (DICAR/COGEP); não há acesso do cidadão nem do próprio servidor
- LOCUS **não é origem oficial**: EOL e SIGPEC continuam sendo a fonte; divergências são resolvidas pela DICAR
- Dados pessoais sob **LGPD (Lei Geral de Proteção de Dados)**: controle por perfil, trilha de auditoria, criptografia e mascaramento em relatórios
- Uso em horário comercial; disponibilidade sugerida de **99,5%**; consultas em até **3 s**, relatórios pesados **assíncronos**
- O LOCUS **gerencia as permissões** de acesso dos dois subsistemas (ALVO e LOCUS)

## Onde aprofundar

| Seção | Conteúdo |
|-------|----------|
| [Visão de negócio](visao-negocio/index.md) | Personas, jornadas, escopo, funcionalidades pendentes e riscos |
| [Visão arquitetural](visao-arquitetural/index.md) | Documento arquitetural, questões consolidadas e integrações EOL / SIGPEC |
| [Documentos / ADRs (Architecture Decision Records)](documentos/index.md) | Decisões arquiteturais, glossário e anexos da wiki |
| [Diagramas C4 (Context, Containers, Components, Code)](diagramas/index.md) | Contexto (C1), containers (C2), componentes (C3) e fluxos |
| [Segurança](seguranca/index.md) | Recomendações prioritárias e matriz de risco |

> Esta documentação descreve o LOCUS **as-is** (outubro/2026: frontend em construção e microsserviços SIGLA existentes) e a arquitetura **TO-BE** proposta (MS-Locus, integrações EOL/SIGPEC, perfis LOCUS). Elementos TO-BE aparecem com **borda tracejada** nos diagramas.
