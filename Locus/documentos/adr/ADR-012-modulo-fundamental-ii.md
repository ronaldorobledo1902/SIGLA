# ADR-012 — Fundamental II e Médio no MS-Locus

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | LOCUS |
| **Relacionadas** | ADR-004, ADR-014 |

## Contexto

O levantamento do Fundamental II e Médio (05/11/2026) descreve cinco fluxos e não divide microsserviços. O desenho de containers mostra o backend como o único que acessa a base e os sistemas externos.

EOL e CI são sistemas externos. A forma da integração com eles não está definida no levantamento. O Sistema Escolha ficou fora na entrevista de `transcricaoreuniao.vtt`: é o legado do ALVO, sem carga e sem retorno.

## Decisão

1. O backend do módulo é o **MS-Locus**.
2. O MS-Locus acessa a Base de Dados SIGLA e lê no ALVO cargo, pessoa, nomeação, processo, escola e vaga.
3. Não há integração com o Sistema Escolha. A carga inicial do quadro é o Banco Access, uma vez.
4. O mecanismo da integração com EOL e CI fica em aberto. No cenário de reaproveitamento o MS-Locus não os chama.

## Consequências

### Positivas

- O módulo continua em um backend, e a origem do nomeado deixa de ser o legado

### Negativas

- A troca com EOL e CI não tem contrato até o mecanismo ser definido

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Definir agora o canal da integração com EOL e CI | O levantamento deixa o mecanismo em aberto |
| Manter o Sistema Escolha como sistema externo do módulo | A entrevista removeu essa ligação |

## Referências

- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*
- `Levantamento/transcricaoreuniao.vtt`
