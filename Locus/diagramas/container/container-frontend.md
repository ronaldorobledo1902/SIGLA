# Container: Frontend LOCUS

O frontend do LOCUS é uma SPA (Single Page Application) React servida por Nginx no caminho `/locus/`, independente do frontend do ALVO. O login acontece no Portal SIGLA, cujo código está temporariamente espelhado em `src/modules/sigla`.

```{image} ../assets/04-locus-componente-frontend.svg
:width: 100%
:alt: Componentes do Frontend LOCUS
```

---

## 1. Stack

| Item | Tecnologia |
|------|------------|
| Framework | React 19 + TypeScript + Vite 7 |
| UI | Ant Design 5 (`temaAntd`), MUI icons, styled-components |
| Estado servidor | React Query |
| Formulários / validação | react-hook-form + Zod |
| HTTP | Axios (`src/servicos/http.ts`) |
| PDF | jsPDF + html2canvas |
| Testes | Jest |
| Runtime | `nginx:alpine`; `VITE_BASE_PATH` como build arg; `entrypoint.sh` substitui `SERVER_NAME` |

## 2. Rotas e telas

| Rota | Tela | Situação |
|------|------|----------|
| `/` | Início | Pronta |
| `/cadastro/gestao-unidades-educacionais` | Gestão de UEs (filtros, listagem) | Dados estáticos |
| `/cadastro/registrar-unidade-educacional` | Registrar UE | Dados estáticos |
| `/cadastro/unidade-educacional/:codigoLotacao` | Detalhe da UE (módulos, lotados, afastados, histórico) | Dados estáticos |
| `/cadastro/atualizacao/registro-funcional[/:rf]` | Busca e detalhe do servidor; registrar atualização | Dados estáticos |
| `/sigla/*` | Login, esqueci/nova senha, seleção de sistemas | Espelho do portal |
| Demais itens do menu | Inclusão, atualizações, Classificação, Relatórios, Data base, Vagas, Remoção, Integração | **Pendentes** |

## 3. Integração com backends

| Cliente | Destino nesta arquitetura |
|---------|----------|
| Front do ALVO e Front do Locus | BFF único |

O código de hoje ainda aponta cada cliente para um microsserviço (`VITE_LOCUS_API_URL`, `VITE_ADMIN_USUARIOS_API_URL` e as URLs do ALVO) e envia `Authorization: Bearer` a partir do `localStorage`. A arquitetura passa a ter um destino só: o BFF valida o token, consulta os serviços e devolve o DTO da tela ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)).

## 4. Pontos de atenção

1. Rotas LOCUS **sem AuthGuard** — qualquer pessoa com a URL abre as telas (hoje com dados fictícios).
2. `VITE_LOCUS_API_URL` não chega à imagem: `.env` é ignorado no build Docker e `window.__ENV__` não é preenchido pelo entrypoint.
3. URL do MS-AdminUsuarios fixa em QA no módulo SIGLA.
4. Usuário logado é **mock** (`useUsuarioLogado`).
