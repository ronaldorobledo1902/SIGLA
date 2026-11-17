# Assets (SVG / Draw.io)

Arquivos visuais exportados e editáveis do modelo C4 (Context, Containers, Components, Code) do LOCUS (Lotação e Controle Unificado de Servidores). A legenda (página **C4**) é a mesma do SERAp; elementos TO-BE têm borda tracejada.

```{toctree}
:maxdepth: 1
```

| Arquivo | Página do drawio | Conteúdo |
|---------|------------------|----------|
| `01-c4-legenda.svg` | C4 | Legenda de shapes C4 |
| `02-locus-contexto.svg` | C1 - Contexto LOCUS | Contexto do sistema (C1) |
| `03-locus-containers.svg` | C2 - Containers LOCUS | Containers (C2) |
| `04-locus-componente-frontend.svg` | C3 - Componentes LOCUS Frontend | Componentes do frontend (C3) |
| `05-locus-componente-ms-locus.svg` | C3 - Componentes MS-Locus (TO-BE) | Componentes do MS de domínio (C3) |
| `06-locus-componente-ms-importa-arquivos.svg` | C3 - Componentes MS-ImportaArquivos | Componentes de ingestão, uso LOCUS (C3) |
| `07-locus-componente-ms-admin-usuarios.svg` | C3 - Componentes MS-AdminUsuarios | Componentes de identidade e permissões (C3) |
| `08-locus-fluxo-integracao-eol-sigpec.svg` | FLUXO - Integração e conciliação | Fluxo EOL/SIGPEC proposto |
| `09-locus-pipeline-cicd.svg` | SISTEMA - Pipeline CI/CD | Esteira CI/CD (Integração Contínua / Entrega Contínua) |
| `10-locus-matriz-risco.svg` | SISTEMA - Matriz Risco de Segurança | Matriz de risco |
| `11-fund2-contexto.svg` | C1 - Contexto Fund II | Contexto do módulo: front, backend, bases e sistemas externos |
| `12-fund2-containers-cadastro.svg` | C2 - Containers Fund II Cadastro | Containers do Fluxo 1 |
| `13-fund2-containers-relatorios.svg` | C2 - Containers Fund II Relatorios | Containers do Fluxo 2 |
| `14-fund2-containers-database.svg` | C2 - Containers Fund II Data base | Containers do Fluxo 3 |
| `15-fund2-containers-vagas.svg` | C2 - Containers Fund II Vagas | Containers do Fluxo 4 |
| `16-fund2-containers-remocao.svg` | C2 - Containers Fund II Remocao | Containers do Fluxo 5 |
| `ARQUITETURA - LOCUS - C4.drawio` | — | Fonte editável ([diagrams.net](https://app.diagrams.net)) |

## Exportar novamente

Com o draw.io Desktop instalado, cada página é exportada pelo índice (1-based):

```powershell
& "$env:LOCALAPPDATA\Programs\draw.io\draw.io.exe" -x -f svg -b 10 -p 2 -o 02-locus-contexto.svg "ARQUITETURA - LOCUS - C4.drawio"
```
