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
| **Remoção** | Processo de mudança de unidade a pedido (a detalhar com o PO) |

## Sistemas externos

| Termo | Significado |
|-------|-------------|
| **EOL** | Escola Online — sistema da SME mantido pela PRODAM, origem de servidores, cargos e unidades |
| **PRODAM** | Empresa de Tecnologia da Informação e Comunicação do Município de São Paulo |
| **SIGPEC** | Sistema Integrado de Gestão de Pessoas e Competências — mantido pela SEGES |
| **SEGES** | Secretaria Municipal de Gestão |
| **CoreSSO** | Single Sign-On corporativo da SME, acessado via API SME Integração |
| **Keycloak** | Gestor de identidades em estudo pela SME-COTIC |
| **SME-COTIC** | Coordenadoria de Tecnologia da Informação e Comunicação da SME |

## Técnicos

| Termo | Significado |
|-------|-------------|
| **C4** | Context, Containers, Components, Code — modelo de diagramas |
| **ADR** | Architecture Decision Record |
| **CQRS** | Command Query Responsibility Segregation — modelos de escrita e leitura separados |
| **DRF** | Django REST Framework |
| **JWT** | JSON Web Token |
| **MFA** | Autenticação multifator |
| **LGPD** | Lei Geral de Proteção de Dados (Lei 13.709/2018) |
| **MS** | Microsserviço |
| **TO-BE** | Estado futuro proposto (borda tracejada nos diagramas) |
| **`sigla-sdk`** | Biblioteca comum dos MS SIGLA (HTTP client, middlewares, ApiKey, logs JSON) |
| **Read model / Write model** | Estruturas de leitura e de escrita no CQRS |
| **PVC** | PersistentVolumeClaim — volume persistente no Kubernetes |
