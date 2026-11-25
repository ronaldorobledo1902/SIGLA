# ADR-013 — Vagas e confronto com o EOL

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Revisão** | 2026-11-25 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | SME-LOCUS-MICROSERVICO |
| **Relacionadas** | ADR-012, ADR-014 |

## Contexto

O levantamento registra a fórmula e pede validação com a área:

```text
Vagas = Módulo - (Lotados definitivos + Lotados precários) + Afastados
```

LMD, excedentes, NEC e vagas em trânsito aparecem na tela. O papel de cada um está em aberto. O confronto usa o número gravado na base do Locus. A DAG do Airflow consulta o EOL e grava nessa base um JSON, um PDF ou um hash.

## Decisão

1. A fórmula permanece a do levantamento, sujeita à validação da área.
2. O confronto é feito pelo SME-LOCUS-MICROSERVICO com os números da base do Locus.
3. O acesso ao EOL é a DAG do Airflow, que grava o arquivo na base do Locus ([ADR-014](ADR-014-escolha-precarios.md)). O SME-LOCUS-MICROSERVICO não chama o EOL.

## Consequências

### Positivas

- O cálculo fica no SME-LOCUS-MICROSERVICO, como o fluxo de Vagas descreve

### Negativas

- LMD, NEC, excedentes, trânsito e a regra do TRDEF seguem sem fechamento

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Fechar agora o papel de LMD, NEC e TRDEF | O levantamento marca esses pontos como em aberto |

## Referências

- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*, fluxo 4
