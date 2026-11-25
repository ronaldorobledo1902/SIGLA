# Assets (SVG / Draw.io)

Arquivos visuais exportados e editáveis do modelo C4 (Context, Containers, Components, Code) do LOCUS (Lotação e Controle Unificado de Servidores). A legenda (página **C4**) é a mesma do SERAp; elementos TO-BE têm borda tracejada.

```{toctree}
:maxdepth: 1
```

| Arquivo | Página do drawio | Conteúdo |
|---------|------------------|----------|
| `01-c4-legenda.svg` | C4 | Legenda de shapes C4 |
| `02-locus-contexto.svg` | C1 - Contexto LOCUS | Contexto do sistema (C1) |
| `06-locus-componente-ms-importa-arquivos.svg` | C3 - Componentes MS-ImportaArquivos | Componentes de ingestão, uso ALVO (C3) |
| `07-locus-componente-ms-admin-usuarios.svg` | C3 - Componentes MS-AdminUsuarios | Componentes de identidade e permissões (C3) |
| `09-locus-pipeline-cicd.svg` | SISTEMA - Pipeline CI/CD | Esteira CI/CD (Integração Contínua / Entrega Contínua) |
| `10-locus-matriz-risco.svg` | SISTEMA - Matriz Risco de Segurança | Matriz de risco |
| `17-reaproveitamento-visao-sigla.svg` | C2 - Reaproveitamento Visão geral | Front, SME-SIGLA-BFF único, SME-LOCUS-MICROSERVICO ao lado da persistência, ALVO abaixo com a base do ALVO. Access dentro da persistência do Locus |
| `18-reaproveitamento-cadastro-sigla.svg` | C2 - Reaproveitamento Cadastro | Containers do Fluxo 1 neste cenário |
| `19-reaproveitamento-relatorios-sigla.svg` | C2 - Reaproveitamento Relatórios | Síncrono na hora. Assíncrono em DAG do Airflow, gravando na base do Locus |
| `20-reaproveitamento-database-sigla.svg` | C2 - Reaproveitamento Data base | Nomeação lida no ALVO |
| `21-reaproveitamento-vagas-sigla.svg` | C2 - Reaproveitamento Vagas | Vaga calculada publicada no ALVO |
| `22-reaproveitamento-remocao-sigla.svg` | C2 - Reaproveitamento Remoção | Equipe na escola do ALVO |
| `23-reaproveitamento-importacao-eol.svg` | C2 - Reaproveitamento Importação EOL | Airflow busca só o EOL e grava JSON, PDF ou hash na base do Locus |
| `24-locus-contexto-lotacao.svg` | C2 - Contexto Lotação | Tela, SME-SIGLA-BFF, quadro na base do Locus e leitura de unidade, cargo e pessoa no ALVO |
| `ARQUITETURA - LOCUS - C4.drawio` | — | Fonte editável ([diagrams.net](https://app.diagrams.net)) |

## Exportar novamente

Com o draw.io Desktop instalado, cada página é exportada pelo índice (1-based):

```powershell
& "$env:LOCALAPPDATA\Programs\draw.io\draw.io.exe" -x -f svg -b 10 -p 2 -o 02-locus-contexto.svg "ARQUITETURA - LOCUS - C4.drawio"
```
