# Componentes do frontend LOCUS

O front do Locus segue a cadeia **páginas → hooks (React Query) → serviços → cliente HTTP**. O cliente HTTP tem um destino: o SME-SIGLA-BFF. A validação Zod acompanha o DTO da tela. O layout comum é o `LayoutBase`.

As páginas de Cadastro ainda leem `dados/dadosEstaticos.ts`. Relatórios, Data base, Vagas e Remoção seguem no menu, sem página.

## 1. Componentes

| Componente | Local | Responsabilidade |
| --- | --- | --- |
| Nginx + entrypoint | `nginx.conf`, `entrypoint.sh` | Serve `/locus/` com rewrite e fallback de SPA. Publica o endereço do SME-SIGLA-BFF |
| Rotas | `src/rotas/index.tsx`, `caminhos.ts` | Rotas do Locus e `/sigla/*`; breadcrumb |
| Módulo SIGLA espelhado | `src/modules/sigla` | Login, senha, seleção de sistemas. A chamada de login sai para o SME-SIGLA-BFF |
| LayoutBase | `src/componentes/layout` | MenuLateral, PainelSubmenu, Cabeçalho, Rodapé |
| Páginas de Cadastro — unidade | `GestaoUnidadesEducacionais`, `RegistrarUnidadeEducacional`, `DetalheUnidadeEducacional` | Listagem, cadastro, detalhe, histórico e versão |
| Páginas de Cadastro — servidor | `AtualizacaoRegistroFuncional`, `DetalheServidor` | Busca por RF, CPF ou nome; dados funcionais; registrar atualização |
| Dados estáticos | `src/paginas/*/dados/dadosEstaticos.ts` | Resposta simulada enquanto o SME-SIGLA-BFF não atende a tela |
| Hooks de página | `src/paginas/*/hooks/use*` | Queries e mutações React Query |
| Validações | `src/paginas/validacoes` (Zod) | `registrarUnidadeEducacional`, `registrarAtualizacao`, `buscarServidor`. O schema descreve o DTO da tela |
| Serviços HTTP | `src/servicos/recursos/*` | Montam a requisição da tela (`unidadesEducacionais`, `servidores`, `autenticacao`) |
| Hooks compartilhados | `src/hooks` | `useUsuarioLogado`, `useExportarPdf`, `useNotificacao` |
| Cliente HTTP | `src/servicos/http.ts` | Axios + Bearer. Destino: SME-SIGLA-BFF. Tratamento de 401 e 403 |

O front do ALVO é o outro cliente do mesmo SME-SIGLA-BFF. Os componentes internos dele ficam no repositório do ALVO.

## 2. O que cada fluxo pede

O contrato da tela é o DTO devolvido pelo SME-SIGLA-BFF ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)). A origem de cada campo está em [Containers](../container/container.md).

| Fluxo | Componente de tela | DTO |
| --- | --- | --- |
| Cadastro | Páginas de unidade e de servidor | Unidade e DRE do MS-Escolha, cargo do MS-ProcessosConcursos, pessoa do MS-Candidatos quando o RF existe, quadro gravado na base do Locus |
| Relatórios | Tela de relatórios. O front redireciona para ela | Leve na resposta da chamada. Pesado quando a DAG do Airflow termina |
| Data base | Tela de data base | Data base DOC, `nomeacao_em`, `data_fechamento_modulo`, e o vínculo depois de “Acrescenta migração” |
| Vagas | Tela de vagas | Módulo, lotados, afastados, EXC e vaga calculada |
| Remoção | Tela de remoção | Equipe, congelado e vaga final, com a escola do MS-Escolha |

ATIVEOL, o congelado do EOL e o confronto de vagas com o EOL ficam de fora destas telas.

## 3. Diretrizes

1. Proteger as rotas do Locus com AuthGuard e com a guarda de perfil ([ADR-003](../../documentos/adr/ADR-003-permissoes.md)).
2. O navegador usa o endereço do SME-SIGLA-BFF, via `VITE_*` no build ou `window.__ENV__` injetado pelo entrypoint.
3. Trocar `dadosEstaticos.ts` pela chamada ao SME-SIGLA-BFF, mantendo o schema Zod como contrato do DTO.
4. Não registrar dado pessoal em `console` ou log. Mascarar CPF nas listagens.
5. Relatório de grande processamento usa o job assíncrono ([ADR-011](../../documentos/adr/ADR-011-relatorios-assincronos.md)). O `html2canvas` permanece para o PDF da tela leve.
6. Branches: `test` → `homolog` → `master` ([ADR-010](../../documentos/adr/ADR-010-branches.md)).
