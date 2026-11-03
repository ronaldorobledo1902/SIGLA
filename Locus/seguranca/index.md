# Recomendações Prioritárias — LOCUS (Lotação e Controle Unificado de Servidores)

**Sistema:** LOCUS  
**Documento base:** [documento_arquitetural_sistema.md](../visao-arquitetural/documento_arquitetural_sistema.md)  
**Data:** Outubro/2026  
**Versão:** 1.0  
**Público-alvo:** Equipe técnica, PO (Product Owner), COGEP/DICAR e SME-COTIC

---

## 1. Contexto

O LOCUS tratará dados pessoais de **mais de 100 mil servidores** sob a **LGPD (Lei Geral de Proteção de Dados)**, possivelmente com dados sensíveis (saúde/afastamentos — a confirmar). O PO exige controle de acesso por perfil, trilha de auditoria, criptografia em trânsito e em repouso, mascaramento em relatórios e recomenda MFA (autenticação multifator).

A análise dos repositórios atuais encontrou lacunas que **precisam ser corrigidas antes de qualquer dado real** entrar no LOCUS. Segredos encontrados em arquivos de exemplo **não são reproduzidos** nesta documentação.

```{image} ../diagramas/assets/10-locus-matriz-risco.svg
:width: 100%
:alt: Matriz de risco de segurança — LOCUS
```

---

## 2. Análise de degradação de performance

### 2.1 — Situação atual

| Origem | Padrão | Cache / fila? |
|--------|--------|---------------|
| LOCUS SPA | React Query | Cache em memória do navegador |
| MS-Locus (TO-BE) | REST síncrono + read model | Read model (CQRS (Command Query Responsibility Segregation)) |
| Integração EOL/SIGPEC | Celery (madrugada) | Fila Redis |
| Relatórios | Job assíncrono (TO-BE) | Fila Redis |

### 2.2 — Por que pode degradar

- Listagens de 100 mil+ servidores sem paginação/índices no read model.
- Relatórios pesados executados de forma síncrona.
- Carga integral (não incremental) do EOL concorrendo com o horário comercial.
- Redis compartilhado entre todos os workers do SIGLA.

### 2.3 — Quando o impacto se torna perceptível

| Cenário | Impacto esperado |
|---------|------------------|
| Dia comum (DICAR) | Baixo |
| Início do ano letivo, atribuição e remoção | **Médio/alto** — consultas e relatórios em pico |
| Falha da carga diária | **Médio** — divergências não sinalizadas no dia |

### 2.4 — Conclusão

O volume de usuários é baixo; o risco está no **volume de dados** e nos **relatórios**. Paginação, read model e jobs assíncronos resolvem o requisito de ≤ 3 s.

---

## 3. Curto prazo — segurança e estabilidade

### 3.1 — AuthGuard e guarda por perfil em todas as rotas LOCUS

**Por quê:** as rotas `/cadastro/*` não exigem autenticação; só `modules/sigla` usa AuthGuard. Com dados reais, qualquer pessoa com a URL veria telas carregadas.

### 3.2 — Criar perfis LOCUS e aplicar permissões por endpoint (ADR-003)

**Por quê:** os grupos atuais são do ALVO; sem perfis LOCUS não há privilégio mínimo.

### 3.3 — Restringir CORS

**Por quê:** `CORS_ALLOW_ALL_ORIGINS=True` no MS-AdminUsuarios e no MS-ImportaArquivos anula a lista `CORS_ALLOWED_ORIGINS` (qa/hom-sigla).

### 3.4 — Reduzir validade e separar a chave do JWT (JSON Web Token)

**Por quê:** access token de 7 dias no MS-AdminUsuarios; assinatura HS256 com `SECRET_KEY`/`JWT_SIGNING_KEY` compartilhada entre MS. Alvo: access ≤ 30 min + refresh com rotação, chave dedicada por ambiente; evolução para RS256/JWKS ou Keycloak.

### 3.5 — Remover segredos de exemplo e rotacionar

**Por quê:** o `env.example` do MS-AdminUsuarios traz uma `SECRET_KEY` preenchida; se reutilizada em algum ambiente, permite forjar tokens. Usar placeholders e Secrets do Kubernetes.

### 3.6 — Configuração do frontend por ambiente

**Por quê:** `VITE_LOCUS_API_URL` não chega à imagem e o cliente SIGLA aponta fixo para QA — risco de homologação/produção chamarem QA.

---

## 4. Médio prazo — observabilidade e resiliência

### 4.1 — Armazenamento persistente para arquivos importados

**Por quê:** `media/` local no pod perde arquivos em restart e espalha dados pessoais sem controle; usar PVC (PersistentVolumeClaim) ou object storage com criptografia e expurgo.

### 4.2 — Trilha de auditoria de **acesso** (não só de alteração)

**Por quê:** LGPD exige saber quem consultou dados pessoais; auditlog registra apenas alterações. Logar consultas de detalhe de servidor e exportações.

### 4.3 — Mascaramento de dados pessoais

**Por quê:** CPF e dados sensíveis em listagens e relatórios devem ser mascarados por padrão, sobretudo no perfil Consulta.

### 4.4 — Health checks, probes e alertas

**Por quê:** sem `/health/ready`, pods sem banco/Redis continuam recebendo tráfego; alertar falha da carga diária.

### 4.5 — Rate limiting e limite de upload no Ingress

**Por quê:** proteger login e importação contra abuso e arquivos grandes.

### 4.6 — Versionar a esteira

**Por quê:** Jenkinsfile fica fora dos repositórios; incluir SAST (Static Application Security Testing), auditoria de dependências (`pip-audit`, `npm audit`) e cobertura ≥ 80%.

---

## 5. Longo prazo — modernização

### 5.1 — IdP corporativo com MFA

**Por quê:** concluir o estudo Keycloak (SME-COTIC) e habilitar MFA para perfis com acesso amplo.

### 5.2 — Criptografia em repouso

**Por quê:** requisito do PO; confirmar criptografia de disco/backup do PostgreSQL e avaliar criptografia de colunas sensíveis.

### 5.3 — Política de retenção

**Por quê:** histórico cresce continuamente; definir com o Jurídico a tabela de temporalidade e o expurgo/anonimização.

### 5.4 — Token fora do `localStorage`

**Por quê:** reduzir exposição a XSS (Cross-Site Scripting) com cookie `HttpOnly` / BFF (Backend for Frontend) quando o IdP for definido.

---

## 6. Resumo executivo

| Horizonte | Foco | Qtd. itens | Prioridade geral |
|-----------|------|------------|------------------|
| Curto prazo (antes de dados reais) | Acesso, CORS, JWT, segredos, config | 6 | **Crítica** |
| Médio prazo (MVP em produção) | Arquivos, auditoria de acesso, mascaramento, probes, gateway, esteira | 6 | **Alta** |
| Longo prazo | MFA/IdP, criptografia em repouso, retenção, token | 4 | **Planejada** |

**Destaques de maior impacto imediato:**

1. **AuthGuard + perfis LOCUS**  
2. **CORS restrito**  
3. **JWT curto com chave dedicada**  
4. **Segredos fora do Git e rotacionados**  
5. **Armazenamento persistente e protegido para importações**

---

## 7. Referência

Documento arquitetural: [documento_arquitetural_sistema.md](../visao-arquitetural/documento_arquitetural_sistema.md)  
Integrações: [integracoes_eol_sigpec.md](../visao-arquitetural/integracoes_eol_sigpec.md)

---
