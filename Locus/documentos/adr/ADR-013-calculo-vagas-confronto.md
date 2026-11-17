# ADR-013 — Vagas e confronto com o EOL

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | MS-Locus |
| **Relacionadas** | ADR-012, ADR-014 |

## Contexto

O levantamento registra a fórmula e pede validação com a área:

```text
Vagas = Módulo - (Lotados definitivos + Lotados precários) + Afastados
```

LMD, excedentes, NEC e vagas em trânsito aparecem na tela. O papel de cada um está em aberto. O confronto compara o número do Locus com a tabela do EOL. A forma de obter essa tabela não está definida.

## Decisão

1. A fórmula permanece a do levantamento, sujeita à validação da área.
2. O confronto é feito pelo MS-Locus.
3. O mecanismo de acesso ao EOL não está definido (ADR-014).

## Consequências

### Positivas

- O cálculo fica no MS-Locus, como o fluxo de Vagas descreve

### Negativas

- LMD, NEC, excedentes, trânsito e a regra do TRDEF seguem sem fechamento

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Fechar agora o papel de LMD, NEC e TRDEF | O levantamento marca esses pontos como em aberto |

## Referências

- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*, fluxo 4
