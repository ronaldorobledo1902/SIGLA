# Containers — LOCUS

O front do ALVO e o front do Locus falam com um SME-SIGLA-BFF único. O SME-SIGLA-BFF valida o token, consulta os microsserviços, agrega os dados e devolve o DTO da tela. O SME-LOCUS-MICROSERVICO lê no ALVO o que o processo de concurso, convocação e escolha já gravou. O sistema executa a carga inicial do arquivo Access. O Airflow busca só o EOL, de tempos em tempos, e grava o resultado na base do Locus.

O código do ALVO ainda não tem o SME-SIGLA-BFF. Ele entra a partir desta arquitetura ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)). O relato do cenário está em [Containers deste cenário](../../cenarios/reaproveitamento-alvo/containers.md).

## Visão geral

```{image} ../assets/17-reaproveitamento-visao-sigla.svg
:width: 100%
:alt: Visão geral. Front, SME-SIGLA-BFF, SME-LOCUS-MICROSERVICO ao lado da persistência e ALVO abaixo
```

| De | Para | O que passa |
| --- | --- | --- |
| Front do ALVO e Front do Locus | SME-SIGLA-BFF | Requisição da tela. O SME-SIGLA-BFF valida o token |
| SME-SIGLA-BFF | MS-AdminUsuarios, SME-LOCUS-MICROSERVICO e microsserviços do ALVO | Consulta, agregação e DTO da tela |
| SME-LOCUS-MICROSERVICO | Base do Locus | Quadro que nenhum MS do ALVO guarda |
| SME-LOCUS-MICROSERVICO | MS-ProcessosConcursos, MS-Candidatos, MS-Agenda, MS-ProcessosConvocacao, MS-Escolha | Cargo, pessoa, nomeação, processo, escola e vaga já gravados no ALVO |
| Microsserviços do ALVO | Base do ALVO | Leitura e gravação da base do ALVO |
| Banco Access | Base do Locus | Carga inicial executada pelo sistema. Uma vez |

O MS-ImportaArquivos continua sendo a porta de entrada do ALVO. O Locus não recebe lote dele.

## Importação periódica do EOL

O modo de importação roda no Airflow: uma origem, o fluxo que busca, e a base que recebe o resultado. A origem é o EOL. O Airflow grava na base do Locus. O gravado pode ser um arquivo JSON, um PDF ou um arquivo hash.

O confronto automático entre o EOL e o quadro do SIGLA reutiliza o caminho do Fluxo 2, Relatórios.

```{image} ../assets/23-reaproveitamento-importacao-eol.svg
:width: 100%
:alt: Importação periódica. Airflow busca o EOL e grava JSON, PDF ou hash na base do Locus
```

| Passo | O que o container faz |
| --- | --- |
| Agenda | O Airflow dispara a execução |
| Busca | O Airflow consulta só o EOL |
| Gravação | O Airflow salva na base do Locus um JSON, um PDF ou um hash |

## Fluxo 1 — Cadastro

O formulário grava na base do Locus. Unidade e cargo são lidos no ALVO. A pessoa só é lida no MS-Candidatos quando o RF existe lá. ATIVEOL fica de fora: o ALVO não guarda o quadro funcional do EOL.

```{image} ../assets/18-reaproveitamento-cadastro-sigla.svg
:width: 100%
:alt: Containers do Cadastro
```

| Peça | Origem neste cenário |
| --- | --- |
| Local, tipo da unidade, sigla da DRE | MS-Escolha |
| Código e descrição do cargo | MS-ProcessosConcursos |
| Nome, CPF, vínculo e classificação de concursado | MS-Candidatos |
| Cargo atual, atividade, afastamento, exercício, lotação, norteador, motivo | Base do Locus |
| ATIVEOL | Sem fonte. Fica de fora |

## Fluxo 2 — Relatórios

Dois ritmos, pela demanda de processamento.

O relatório de menor processamento volta na hora: a chamada é síncrona e a tela responde de forma dinâmica.

O relatório de grande processamento é assíncrono: o Relatorio dispara uma DAG do Airflow e envia um job ao WORKER. A DAG grava o relatório na base do Locus. O WORKER lê e grava nessa mesma base.

A tela de relatórios chama o SME-SIGLA-BFF. O SME-SIGLA-BFF valida o token. No caminho síncrono, devolve o relatório na hora. No caminho assíncrono, a mesma tela apresenta o relatório quando o Airflow termina de gravar.

```{image} ../assets/19-reaproveitamento-relatorios-sigla.svg
:width: 100%
:alt: Containers dos Relatórios
```

| Peça | Papel |
| --- | --- |
| Base do Locus | Única base do módulo. O WORKER lê e grava nela. A DAG grava o relatório nela |
| Relatorio | Responde na hora o de menor processamento. Dispara a DAG e envia o job ao WORKER |
| WORKER | Recebe o job. Lê e grava na base do Locus |
| Airflow | DAG do relatório pesado. Executa e grava o relatório na base do Locus |
| SME-SIGLA-BFF | Valida o token. Devolve o relatório leve na hora e o pesado quando a DAG termina |
| Tela de Relatórios | Mostra o síncrono na hora e o assíncrono quando o Airflow conclui |
| Front Locus | Redireciona para a tela de relatórios |

| Ritmo | Caminho |
| --- | --- |
| Síncrono | Menor processamento. A tela recebe a resposta na hora |
| Assíncrono | Grande processamento. O Relatorio dispara a DAG e envia o job. A DAG grava o relatório. A tela mostra quando estiver pronto |

## Fluxo 3 — Data base

Datas operacionais do módulo ficam no Locus. A migração lê agenda, processo, habilitado e escolha que o ALVO já tem, e o usuário confirma a gravação do vínculo.

```{image} ../assets/20-reaproveitamento-database-sigla.svg
:width: 100%
:alt: Containers da Data base
```

| Passo do levantamento | Neste cenário |
| --- | --- |
| Data base EOL = ATIVEOL | Sem data equivalente no ALVO. A tela mostra Data base DOC e, como referência de processo, `nomeacao_em` e `data_fechamento_modulo` |
| 1º passo da migração | Consulta MS-Agenda, MS-ProcessosConvocacao, MS-Candidatos e MS-Escolha |
| 2º passo, Acrescenta migração | Grava o vínculo novo na base do Locus |
| Controle de atualizações e extratos | Leitura da auditoria e da movimentação do Locus |

## Fluxo 4 — Vagas

A tela pede o cálculo ao SME-SIGLA-BFF. O SME-SIGLA-BFF chama o SME-LOCUS-MICROSERVICO. O serviço lê, na base do Locus, o módulo da unidade, os lotados e os afastados, e calcula a vaga.

```{image} ../assets/21-reaproveitamento-vagas-sigla.svg
:width: 100%
:alt: Containers de Vagas
```

| Opção da tela | Neste cenário |
| --- | --- |
| Vagas Fund. II e Médio, Vagas Médio | Calculadas no SME-LOCUS-MICROSERVICO com os números da base do Locus |
| Ajuste de EXC | Gravado na base do Locus. O cálculo é refeito |
| Confronto com o EOL | A tela usa o número do Locus |
| Módulo, lotados, afastados e vagas do EOL | A tela mostra o que está gravado no Locus |

## Fluxo 5 — Remoção

Consultas, equipe, congelado e vaga final usam a base do Locus e a escola do MS-Escolha. O confronto com o EOL não tem dado reaproveitável.

```{image} ../assets/22-reaproveitamento-remocao-sigla.svg
:width: 100%
:alt: Containers da Remoção
```

| Bloco | Neste cenário |
| --- | --- |
| Confronto de módulos, lotados, afastados e vagas com o EOL | Fora |
| Afastado perde lotação, outros afastamentos, liminar | Leitura da base do Locus |
| Equipe da unidade | Vínculos do Locus na escola do MS-Escolha |
| Congelado nosso | Fotografia gravada na base do Locus |
| Congelado EOL | Fora |
| Vagas para remoção | Cálculo no SME-LOCUS-MICROSERVICO a partir dos números conferidos no próprio Locus |
| Carga da remoção | Continua a mapear no levantamento. Não há carga pronta em nenhum MS |

## Resumo arquitetural

O navegador fala com o SME-SIGLA-BFF. O SME-SIGLA-BFF valida o token, consulta os microsserviços e devolve um DTO da tela. O front do ALVO e o front do Locus usam esse mesmo SME-SIGLA-BFF. O login continua no MS-AdminUsuarios, chamado pelo SME-SIGLA-BFF.

O SME-LOCUS-MICROSERVICO guarda o quadro do módulo na base do Locus: lotação, exercício, afastamento, módulo e vaga calculada. Concurso, convocação e escolha são lidos nos microsserviços do ALVO. O MS-ImportaArquivos segue como porta de entrada do ALVO.

Duas cargas entram na base do Locus. O Access entra uma vez, na carga inicial. O Airflow, de tempos em tempos, busca só o EOL e grava um JSON, um PDF ou um hash.

As telas seguem esse desenho. O Cadastro grava o quadro e lê unidade, cargo e pessoa no ALVO. O relatório leve volta na hora; o pesado dispara uma DAG do Airflow e um job ao WORKER, e os dois gravam na mesma base. A Data base lê a nomeação que o ALVO já tem e confirma o vínculo no Locus. A vaga é calculada com módulo, lotados e afastados da base do Locus. A Remoção lê o quadro do Locus e a escola no MS-Escolha. A carga da remoção continua a mapear.

O detalhe de cada front está em [Frontend](container-frontend.md). O detalhe dos serviços, do worker e das bases está em [Backend](container-backend.md).
