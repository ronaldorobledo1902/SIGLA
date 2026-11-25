# Container: frontends

O front do ALVO e o front do Locus enviam a requisição da tela ao SME-SIGLA-BFF. O SME-SIGLA-BFF valida o token e devolve o DTO da tela. O desenho dos dois fronts na visão geral está em [Containers](container.md).

O front do Locus é uma SPA React servida por Nginx em `/locus/`. O login acontece no Portal SIGLA. O código desse portal está temporariamente espelhado em `src/modules/sigla`.

## 1. Quem chama quem

| Cliente | Destino |
| --- | --- |
| Front do ALVO | SME-SIGLA-BFF |
| Front do Locus | SME-SIGLA-BFF |

O código de hoje ainda aponta cada cliente para um microsserviço (`VITE_LOCUS_API_URL`, `VITE_ADMIN_USUARIOS_API_URL` e as URLs do ALVO) e envia `Authorization: Bearer` a partir do `localStorage`. Nesta arquitetura o destino da tela é um só ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)).

## 2. Stack do front do Locus

| Item | Tecnologia |
| --- | --- |
| Framework | React 19 + TypeScript + Vite 7 |
| UI | Ant Design 5 (`temaAntd`), MUI icons, styled-components |
| Estado servidor | React Query |
| Formulários / validação | react-hook-form + Zod |
| HTTP | Axios. O cliente passa a ter o endereço do SME-SIGLA-BFF |
| PDF de tela | jsPDF + html2canvas. Relatório pesado segue o Fluxo 2 |
| Testes | Jest |
| Runtime | `nginx:alpine`; `VITE_BASE_PATH` como build arg; `entrypoint.sh` substitui `SERVER_NAME` |

## 3. Telas do front do Locus

| Fluxo | O que a tela faz | Situação no repositório |
| --- | --- | --- |
| Acesso | Login, senha e seleção de sistemas em `/sigla/*` | Espelho do portal |
| Cadastro | Gestão, registro e detalhe da unidade; busca e atualização do servidor | Páginas prontas com `dadosEstaticos.ts` |
| Relatórios | O front redireciona para a tela de relatórios. O leve aparece na hora. O pesado aparece quando o Airflow conclui | Menu pendente |
| Data base | Mostra Data base DOC e a referência de processo. O usuário confirma “Acrescenta migração” | Menu pendente |
| Vagas | Pede o cálculo e o ajuste de EXC. Mostra módulo, lotados, afastados e vaga da base do Locus | Menu pendente |
| Remoção | Consulta equipe, congelado e vaga final | Menu pendente |

Rotas já existentes no Cadastro:

| Rota | Tela |
| --- | --- |
| `/` | Início |
| `/cadastro/gestao-unidades-educacionais` | Gestão de unidades |
| `/cadastro/registrar-unidade-educacional` | Registrar unidade |
| `/cadastro/unidade-educacional/:codigoLotacao` | Detalhe da unidade |
| `/cadastro/atualizacao/registro-funcional[/:rf]` | Busca, detalhe e atualização do servidor |
| `/sigla/*` | Login, senha e seleção de sistemas |

## 4. O que cada fluxo pede ao SME-SIGLA-BFF

| Fluxo | A tela envia | A tela recebe |
| --- | --- | --- |
| Cadastro | Gravação do quadro e a chave de consulta (código da unidade, cargo, RF) | DTO com a unidade e o cargo do ALVO, a pessoa do MS-Candidatos quando o RF existe, e o quadro da base do Locus |
| Relatórios | Pedido do relatório e os filtros | Leve na hora. Pesado quando a DAG termina de gravar |
| Data base | Consulta da migração e a confirmação do vínculo | Agenda, processo, habilitado e escolha do ALVO. Depois, o vínculo gravado no Locus |
| Vagas | Pedido de cálculo e o ajuste de EXC | Módulo, lotados, afastados e vaga calculada |
| Remoção | Consulta da unidade no período | Equipe, congelado e vaga final do Locus, com a escola do MS-Escolha |

## 5. Pontos de atenção no código atual

1. As rotas LOCUS abrem sem AuthGuard. A guarda passa a valer com o SME-SIGLA-BFF e o perfil ([ADR-003](../../documentos/adr/ADR-003-permissoes.md)).
2. `VITE_LOCUS_API_URL` não chega à imagem: `.env` é ignorado no build Docker e `window.__ENV__` não é preenchido pelo entrypoint. O endereço publicado para o navegador é o do SME-SIGLA-BFF.
3. A URL do MS-AdminUsuarios está fixa em QA no módulo SIGLA. O login da tela passa pelo SME-SIGLA-BFF.
4. O usuário logado é mock (`useUsuarioLogado`).
