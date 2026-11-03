# Anexos — Wiki SIGLA (recorte LOCUS)

Imagens da wiki *COTIC - DevOps › Arquitetura › Sistemas › SIGLA › Visão Arquitetural* (Azure DevOps) usadas como base desta documentação. Mantidas aqui para rastreabilidade; a fonte oficial continua sendo a wiki.

```{toctree}
:maxdepth: 1
```

| Arquivo | Página da wiki | Uso nesta documentação |
|---------|----------------|------------------------|
| `wiki/Visao_Geral_1.png` | Visão Arquitetural | Visão geral SIGLA (ALVO + LOCUS) |
| `wiki/Contextos_de_Negocio_1.png` | Contextos de Negócio | Fronteira ALVO × LOCUS |
| `wiki/Visao_FRONTEND_1.png` | Diagramas › Visão FRONTEND | Referência do C3 frontend |
| `wiki/Visao_BACKEND_1.png` | Diagramas › Visão BACKEND | Referência do C2 / CQRS (Command Query Responsibility Segregation) |
| `wiki/Login_1.png` | Jornadas › Login | Fluxo CoreSSO / Keycloak (em estudo) |
| `wiki/Cadastro_de_usuarios_do_sistema_1.png` | Jornadas › Cadastro de usuários | Container de permissões ALVO + LOCUS |
| `wiki/Cadastro_de_usuarios_do_sistema_2.png` | Jornadas › Cadastro de usuários | Componente MS-AdminUsuarios |
| `wiki/importacao_1.png` | Jornadas › Importação de dados LOCUS | Contexto da importação |
| `wiki/Importacao_de_dados_LOCUS_1.png` | Jornadas › Importação de dados LOCUS | Contexto |
| `wiki/Importacao_de_dados_LOCUS_2.png` | Jornadas › Importação de dados LOCUS | Container (layout posicional, upload, validação) |
| `wiki/Importacao_de_dados_LOCUS_3.png` | Jornadas › Importação de dados LOCUS | Componente MS-ImportaArquivos |

## Principais pontos extraídos da wiki

- SIGLA = ALVO + LOCUS; o LOCUS concentra **importação de dados** e **cadastro de permissões** dos dois subsistemas.
- Microsserviços atribuídos ao LOCUS: **MS-ImportaArquivos** e **MS-AdminUsuarios**.
- Rotas: `sigla.sme.prefeitura.sp.gov.br/locus` → `api-sigla.sme.prefeitura.sp.gov.br/ms-importa-arquivos` e `/ms-admin-usuarios`.
- Login pelo **CoreSSO** (credenciais do sistema antigo ESCOLHAS); **Keycloak** em estudo pela SME-COTIC.
- Ingress do Kubernetes como API Gateway; Rancher; Jenkins; Grafana + Elastic.
- `DB_SIGLA` com **CQRS** (write models por domínio; read models atualizados de forma assíncrona).
- Backend Python (Django escolhido no ADR-01 da wiki); frontends React; cobertura de testes mínima de **80%**.
- Importação: tela de cadastro de layout posicional (txt/csv, posições e tamanhos), tela de upload, validação contra o layout; principal fonte é o **SIGPEC (SEGES)**.

## Questionário do PO

O questionário respondido está em `SIGLA/Levantamento/Questionário_PO_LOCUS_respondido.docx` e é a fonte da [Visão de negócio](../../visao-negocio/index.md).
