# ADR-012 — Fundamental II e Médio no MS-Locus

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | LOCUS |
| **Relacionadas** | ADR-004 |

## Contexto

O levantamento do Fundamental II e Médio (05/11/2026) descreve cinco fluxos e não divide microsserviços. O desenho de containers mostra o backend como o único que acessa a base e os sistemas externos.

EOL, Sistema Escolha e CI são sistemas externos. A forma da integração não está definida no levantamento.

## Decisão

1. O backend do módulo é o **MS-Locus**.
2. O MS-Locus acessa a base de escrita, a base de leitura e, neste momento, os sistemas externos.
3. O mecanismo da integração com EOL, Sistema Escolha e CI fica em aberto.

## Consequências

### Positivas

- O desenho do levantamento se mantém: um backend acessa base e sistemas externos

### Negativas

- A troca com EOL, Sistema Escolha e CI não tem contrato até o mecanismo ser definido

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Definir agora o canal da integração (arquivo, API ou outro MS) | O levantamento deixa o mecanismo em aberto |

## Referências

- [Fundamental II e Médio](../../visao-arquitetural/proposta-fundamental-ii-medio.md)
- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*
