# Componente: Frontend LOCUS

O frontend segue a cadeia **páginas → hooks (React Query) → serviços → cliente HTTP**, com validação Zod e layout comum (`LayoutBase`). Os serviços já expressam o contrato esperado do MS-Locus, mas as páginas ainda leem `dados/dadosEstaticos.ts`.

```{image} ../assets/04-locus-componente-frontend.svg
:width: 100%
:alt: Componentes (C3) do Frontend LOCUS
```

---

## 1. Visão geral (C3)

Requisições síncronas HTTPS ao MS-Locus (TO-BE) e ao MS-AdminUsuarios. Exportação de PDF feita no navegador.

---

## 2. Componentes e responsabilidades

| Componente | Local | Responsabilidade |
|------------|-------|------------------|
| Nginx + entrypoint | `nginx.conf`, `entrypoint.sh` | Serve `/locus/` com rewrite e fallback SPA |
| Rotas | `src/rotas/index.tsx`, `caminhos.ts` | Rotas LOCUS e `/sigla/*`; breadcrumb |
| Módulo SIGLA espelhado | `src/modules/sigla` | Login, senha, seleção de sistemas, AuthGuard |
| LayoutBase | `src/componentes/layout` | MenuLateral, PainelSubmenu, Cabeçalho, Rodapé |
| Páginas de Unidades | `src/paginas/GestaoUnidadesEducacionais`, `RegistrarUnidadeEducacional`, `DetalheUnidadeEducacional` | Listagem, cadastro, detalhe, histórico e versão |
| Páginas de Servidor | `src/paginas/AtualizacaoRegistroFuncional`, `DetalheServidor` | Busca RF/CPF/nome, dados funcionais, registrar atualização |
| Dados estáticos | `src/paginas/*/dados/dadosEstaticos.ts` | Respostas simuladas enquanto não há backend |
| Hooks de página | `src/paginas/*/hooks/use*` | Queries/mutações React Query |
| Validações | `src/paginas/validacoes` (Zod) | `registrarUnidadeEducacional`, `registrarAtualizacao`, `buscarServidor` |
| Serviços HTTP | `src/servicos/recursos/*` | `unidadesEducacionais`, `servidores`, `autenticacao` |
| Hooks compartilhados | `src/hooks` | `useUsuarioLogado` (mock), `useExportarPdf`, `useNotificacao` |
| Cliente HTTP LOCUS | `src/servicos/http.ts` | Axios + Bearer + tratamento 401/403 |
| Cliente HTTP SIGLA | `src/modules/sigla/servicos/http.ts` | Axios para MS-AdminUsuarios |

---

## 3. Contrato esperado do MS-Locus (extraído dos serviços)

| Método | Endpoint | Uso |
|--------|----------|-----|
| GET | `/api/v1/unidades-educacionais/lotacao/{codigo}/` | Consulta por código de lotação |
| POST | `/api/v1/unidades-educacionais/` | Registrar UE |
| GET / DELETE | `/api/v1/unidades-educacionais/{codigo}/` | Detalhe / exclusão |
| GET | `/api/v1/unidades-educacionais/{codigo}/modulos/` | Módulos e saldo |
| GET | `/api/v1/unidades-educacionais/{codigo}/componentes/{id}/lotados/` | Lotados por componente |
| GET | `/api/v1/unidades-educacionais/{codigo}/componentes/{id}/afastados/` | Afastados por componente |
| GET | `/api/v1/unidades-educacionais/{codigo}/historico/` | Histórico e versões |
| GET | `/api/v1/servidores/` | Pesquisa de servidor (RF, CPF, nome) |

Ainda **sem endpoint definido** no frontend: listagem/filtros de UE, detalhe do servidor e registro de atualização (hoje só em dados estáticos) — devem ser desenhados no OpenAPI do MS-Locus.

---

## 4. Diretrizes para o desenvolvedor

1. **Proteger todas as rotas LOCUS** com AuthGuard e guarda por perfil (ADR-003).
2. **Não fixar URLs** no código: usar `VITE_*` no build ou `window.__ENV__` injetado pelo entrypoint.
3. Trocar `dadosEstaticos.ts` por chamadas reais mantendo os schemas Zod como contrato.
4. Não registrar dados pessoais em `console`/logs; mascarar CPF em listagens.
5. Relatórios pesados devem usar o fluxo de job assíncrono (ADR-011), não `html2canvas`.
6. Branches: `test` → `homolog` → `master` (ADR-010).
