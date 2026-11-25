# Glossário — LOCUS

Termos de negócio e técnicos usados na documentação do LOCUS.

## Negócio

| Termo | Significado |
|-------|-------------|
| **SIGLA** | Sistema da SME composto pelos subsistemas ALVO e LOCUS |
| **ALVO** | Alocação de Vagas Online — convocação, agendamento e escolha de vagas (fora do escopo desta documentação) |
| **LOCUS** | Lotação e Controle Unificado de Servidores |
| **SME** | Secretaria Municipal de Educação de São Paulo |
| **COGEP** | Coordenadoria de Gestão de Pessoas da SME |
| **DICAR** | Divisão de Gestão de Carreiras (COGEP) — principal usuária do LOCUS |
| **DRE** | Diretoria Regional de Educação |
| **UE** | Unidade Educacional |
| **RF** | Registro Funcional do servidor |
| **Vínculo** | Relação funcional do servidor com a PMSP (Prefeitura Municipal de São Paulo): efetivo, comissionado, contratado, admitido |
| **Lotação** | Unidade à qual o cargo do servidor pertence; pode ser **definitiva** ou **precária** |
| **Exercício** | Unidade onde o servidor efetivamente atua (pode diferir da lotação) |
| **Módulo** | Quantidade de cargos/aulas previstos por componente curricular na UE, base do saldo de vagas |
| **Saldo de vagas** | Comparação entre módulo e lotados: disponível, excedente ou completo |
| **Afastamento** | Ausência temporária (ex.: licença médica, afastamento para estudo) |
| **Readaptação** | Mudança de função por restrição de saúde |
| **Vacância** | Cargo vago por encerramento do vínculo (exoneração, aposentadoria, falecimento etc.) |
| **Ato sem efeito** | Anulação de um ato anterior, preservando o histórico |
| **Portaria** | Ato administrativo que fundamenta uma alteração funcional |
| **Motivo de atualização** | Catálogo (~90 itens) que classifica a alteração registrada |
| **Divergência** | Diferença entre o dado do LOCUS e o da origem oficial (EOL/SIGPEC) |
| **Conciliação** | Processo de confrontar LOCUS × origem e tratar divergências |
| **Remoção** | Preparação do concurso de remoção do Fund. II e Médio: confronto, congelado e vagas. A carga ainda está a mapear |
| **Data base** | Datas de referência do módulo (EOL, DOC, agenda), migração de candidatos e extrato de entrada e saída |
| **Norteador** | Código associado ao vínculo do professor. No exemplo do Fund. II tem o formato de código de unidade |
| **ATIVEOL** | Conferência dos dados funcionais do EOL no cadastro. O mecanismo de acesso não está definido; fica com o SME-LOCUS-MICROSERVICO |
| **MOU / Mourão** | Extrato do EOL por componente, importado como `<COMPONENTE>MOU`. Mourão é o nome legado dessa referência, não um sistema |
| **EXC** | Excedente ajustado à mão na vaga da unidade. É gravado e auditado |
| **Fund. II e Médio** | Primeiro módulo do LOCUS. Valor `fund2-medio` no SME-LOCUS-MICROSERVICO |

## Sistemas externos

| Termo | Significado |
|-------|-------------|
| **EOL** | Escola Online — sistema externo da SME, mantido pela PRODAM. Origem de dados funcionais, módulos e unidades. Não é microsserviço do SIGLA |
| **Sistema Escolha** | Legado do ALVO (convocação e escolha). Fora de integração: o ALVO não importa dado dele e o Locus não o chama. Não é o MS-Escolha |
| **MS-Escolha** | Microsserviço do ALVO (convocação e escolha de vagas). Não é o Sistema Escolha do módulo Fundamental II |
| **PRODAM** | Empresa de Tecnologia da Informação e Comunicação do Município de São Paulo |
| **SIGPEC** | Sistema Integrado de Gestão de Pessoas e Competências — mantido pela SEGES |
| **SEGES** | Secretaria Municipal de Gestão |
| **CoreSSO** | Autenticação externa chamada pelo MS-AdminUsuarios na API SME Integração (`/api/v1/autenticacao/externa`) |
| **SME-COTIC** | Coordenadoria de Tecnologia da Informação e Comunicação da SME |

## Técnicos

| Termo | Significado |
|-------|-------------|
| **C4** | Context, Containers, Components, Code — modelo de diagramas |
| **ADR** | Architecture Decision Record |
| **DRF** | Django REST Framework |
| **JWT** | JSON Web Token |
| **SME-SIGLA-BFF** | Backend for Frontend. Um só para o ALVO e o Locus: recebe a tela, valida o token, consulta os microsserviços, agrega e devolve o DTO |
| **MFA** | Autenticação multifator |
| **LGPD** | Lei Geral de Proteção de Dados (Lei 13.709/2018) |
| **MS** | Microsserviço |
| **TO-BE** | Estado futuro proposto (borda tracejada nos diagramas) |
| **`sigla-sdk`** | Biblioteca comum dos MS SIGLA (HTTP client, middlewares, ApiKey, logs JSON) |
| **Base de Dados SIGLA (BASE SIGLA)** | PostgreSQL único do SIGLA (`db_sigla`). Consultas e gravações usam a mesma base |
| **PVC** | PersistentVolumeClaim — volume persistente no Kubernetes |
