# ADR-014 — Acesso aos sistemas externos do Fundamental II

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Revisão** | 2026-11-25 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | LOCUS, SME-LOCUS-MICROSERVICO |
| **Relacionadas** | ADR-012 |

## Contexto

O levantamento nomeia dois sistemas de fora do SIGLA:

- EOL: dados funcionais, módulos, lotações e afastamentos.
- Sistema Escolha: no levantamento, enviaria candidatos e a agenda de nomeação e receberia as vagas.

A entrevista `Levantamento/transcricaoreuniao.vtt` fechou o segundo item. Jessica: o Escolha é o legado do ALVO; não se traz dado dele para o ALVO; quando o ALVO entrar em produção, o Escolha deixa de ser usado; nada é remetido de volta. No diagrama, pediu para tirar a ligação do backend do Locus com “sistema escolha, vagas e candidatos”. A conversa do Locus é com o ALVO. Daniela: o nomeado que ainda não assumiu é lançado hoje no Banco Access; essa ligação passa a ser com o ALVO.

O Access não é o EOL nem o Escolha. É o banco de servidores e de unidades que geram vaga, e é a carga inicial do Locus. A planilha nomeada na entrevista é a do EOL (Excel, em geral de 15 em 15 dias), que a equipe sobe no Access para confronto. A transcrição não descreve planilha gerada pelo Sistema Escolha.

A busca periódica do EOL é uma DAG do Airflow. O Airflow consulta só o EOL e grava JSON, PDF ou hash na base do Locus. O SME-LOCUS-MICROSERVICO não chama o EOL. O MS-ImportaArquivos não carrega esse arquivo.

## Decisão

1. Não há integração do ALVO nem do Locus com o Sistema Escolha.
2. Nomeação, habilitado, escola e vaga do processo são lidos nos microsserviços do ALVO. A vaga calculada pelo módulo é publicada no MS-Escolha do ALVO, não enviada em arquivo ao legado.
3. A carga inicial do Locus é o conteúdo atual do Banco Access, uma vez.
4. O EOL não é microsserviço do SIGLA. O acesso é a DAG do Airflow, que grava o arquivo na base do Locus. O SME-LOCUS-MICROSERVICO não chama o EOL.

## Consequências

### Positivas

- O legado Escolha sai do desenho antes de existir um contrato que a entrevista já descartou

### Negativas

- O SME-LOCUS-MICROSERVICO não abre conexão com o EOL. O quadro usa o arquivo que a DAG gravou na base do Locus

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Manter o Sistema Escolha como sistema externo do Locus | A entrevista removeu essa ligação |
| Tratar a planilha do EOL como arquivo do Escolha | A transcrição atribui a planilha ao EOL e o Access como destino operacional |

## Referências

- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*, seções de containers e de integrações externas
- `Levantamento/transcricaoreuniao.vtt`
