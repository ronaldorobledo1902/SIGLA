# ADR-008 — Histórico: `django-auditlog` + versões de registro


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA / PO LOCUS            |
| **Sistema**      | MS-Locus                                |
| **Relacionadas** | ADR-004                                 |


---

## Contexto

É premissa do LOCUS manter o histórico de registros e alterações com **acesso a versões anteriores** para revisão e auditoria. O frontend já exibe histórico da UE e um banner de "versão histórica" somente leitura. Os MS SIGLA já usam `django-auditlog` com `AuditlogJWTMiddleware` (`sigla-sdk`), que registra o autor a partir do JWT. Retenção a definir com o Jurídico.

## Decisão

1. **Trilha técnica:** `django-auditlog` em todos os models do MS-Locus (quem, quando, campos alterados).
2. **Versão de negócio:** cada atualização funcional gera um registro imutável em `Movimentacao` (motivo, portaria, data, documento, origem — manual ou importação) e um **snapshot** da entidade (UE, lotação), exposto em `/{recurso}/historico/` e `/{recurso}/versoes/{id}/`.
3. Registros nunca são apagados fisicamente; "excluir" e "ato sem efeito" geram nova versão.
4. Política de retenção parametrizável, definida com o Jurídico SME.

## Consequências

### Positivas

- Atende auditoria e LGPD (rastreabilidade de acesso e alteração)
- Reconstrução de qualquer estado anterior

### Negativas

- Crescimento contínuo de tabelas (particionar por ano quando necessário)
- Snapshots duplicam dados pessoais — protegê-los como o dado vivo

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Apenas auditlog | Difícil reconstruir versão de negócio e exibir ao usuário |
| `django-simple-history` | Viável; auditlog já é padrão no SIGLA |
| Event sourcing completo | Complexidade desproporcional |

## Referências

- `SME-SIGLA-SDK-Microsservico` — `AuditlogJWTMiddleware`
- `SME-SIGLA-LOCUS-Frontend` — `BannerVersaoHistorica`, histórico da UE
