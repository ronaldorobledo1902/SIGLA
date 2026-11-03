# ADR-010 — Branches: padrão `test` / `homolog` / `master`


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Accepted (as-is)                        |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Time SIGLA / DevOps                     |
| **Sistema**      | Repositórios LOCUS                      |
| **Relacionadas** | ADR-009                                 |


---

## Contexto

Os repositórios do SIGLA usados pelo LOCUS já trabalham com `test`, `homolog` e `master`, alinhados aos ambientes QA, homologação e produção.

## Decisão

1. `feature/*` e `fix/*` saem de `test` e voltam por PR (Pull Request) revisado (`CODEOWNERS`).
2. Promoção por PR: `test` → `homolog` → `master`.
3. Cada branch publica no ambiente correspondente: `qa-*`, `hom-*` e produção.
4. Hotfix sai de `master` e é propagado de volta para `homolog` e `test`.

## Consequências

### Positivas

- Padrão único com o restante da SME
- Ambiente previsível por branch

### Negativas

- Exige disciplina para não divergir `test` de `master`

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| GitFlow com `develop`/`release` | Padrão antigo da SME, em migração |
| Trunk-based | Sem ambientes efêmeros na infra atual |

## Referências

- Repositórios `SME-SIGLA-*` (branches remotas)
