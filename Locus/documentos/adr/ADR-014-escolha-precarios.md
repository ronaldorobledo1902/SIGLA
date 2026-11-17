# ADR-014 — Acesso aos sistemas externos do Fundamental II

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | LOCUS, MS-Locus |
| **Relacionadas** | ADR-012 |

## Contexto

O levantamento nomeia três sistemas de fora do SIGLA:

- EOL: dados funcionais, módulos, lotações e afastamentos.
- Sistema Escolha: envia candidatos e agenda de nomeação; recebe as vagas.
- CI: envia o arquivo de professores precários.

O mesmo documento deixa em aberto a API do EOL, o formato da lista do Sistema Escolha, o layout do arquivo de vagas e o recebimento dos precários. No desenho de containers, o backend é quem acessa esses sistemas.

## Decisão

1. EOL, Sistema Escolha e CI não são microsserviços do SIGLA.
2. A forma da integração não está definida.
3. Até essa definição, o acesso é responsabilidade do **MS-Locus**.

## Consequências

### Positivas

- Nenhum outro serviço assume a troca antes de o mecanismo existir

### Negativas

- O MS-Locus concentra o acesso externo do módulo enquanto o mecanismo não fecha

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Fixar agora arquivo, API ou outro microsserviço como canal | O levantamento não define o mecanismo |

## Referências

- [Fundamental II e Médio](../../visao-arquitetural/proposta-fundamental-ii-medio.md)
- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*, seções de containers e de integrações externas
