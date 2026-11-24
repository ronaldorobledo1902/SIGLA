# Containers deste cenário

Diagramas de containers deste cenário. O MS-Locus lê no ALVO o que o processo de concurso, convocação e escolha já gravou. O sistema executa a carga inicial do arquivo Access. O Airflow busca o EOL de tempos em tempos, compara os servidores do Locus e grava no schema do Locus o que a comparação exigir.

Estes desenhos estão em `diagramas/assets`, arquivos `17` a `23`.

## Visão geral

![Visão geral: Front acima, MS-Locus ao lado da persistência, ALVO abaixo com a base do ALVO e Access na carga inicial](../../diagramas/assets/17-reaproveitamento-visao-base.png)


| De                     | Para                                                                                | O que passa                                                          |
| ---------------------- | ----------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Front Locus            | MS-AdminUsuarios                                                                    | Login e permissão                                                    |
| Front Locus            | MS-Locus                                                                            | Telas do módulo                                                      |
| MS-Locus               | Schema do Locus                                                                     | Quadro que nenhum MS do ALVO guarda                                  |
| MS-Locus               | MS-ProcessosConcursos, MS-Candidatos, MS-Agenda, MS-ProcessosConvocacao, MS-Escolha | Cargo, pessoa, nomeação, processo, escola e vaga já gravados no ALVO |
| Microsserviços do ALVO | Base do ALVO                                                                        | Leitura e gravação dos schemas do ALVO                               |
| Banco Access           | Schema do Locus                                                                     | Carga inicial executada pelo sistema. Uma vez                        |


O MS-ImportaArquivos continua sendo a porta de entrada do ALVO. O Locus não recebe lote dele.

## Importação periódica do EOL

O modo de importação roda no Airflow. De tempos em tempos ele busca os servidores no EOL, compara com os servidores gravados no schema do Locus e grava nesse schema as informações necessárias.

![Containers da importação periódica do EOL](../../diagramas/assets/23-reaproveitamento-importacao-eol.png)


| Passo      | O que o container faz                                        |
| ---------- | ------------------------------------------------------------ |
| Agenda     | O Airflow dispara a execução                                 |
| Busca      | O Airflow consulta o EOL                                     |
| Comparação | O Airflow lê os servidores do schema do Locus                |
| Gravação   | O Airflow grava no schema do Locus o resultado da comparação |


## Fluxo 1 — Cadastro

O formulário grava no schema do Locus. Unidade e cargo são lidos no ALVO. A pessoa só é lida no MS-Candidatos quando o RF existe lá. ATIVEOL não tem chamada: o ALVO não guarda o quadro funcional do EOL.

![Containers do Cadastro](../../diagramas/assets/18-reaproveitamento-cadastro.png)


| Peça                                                                       | Origem neste cenário    |
| -------------------------------------------------------------------------- | ----------------------- |
| Local, tipo da unidade, sigla da DRE                                       | MS-Escolha              |
| Código e descrição do cargo                                                | MS-ProcessosConcursos   |
| Nome, CPF, vínculo e classificação de concursado                           | MS-Candidatos           |
| Cargo atual, atividade, afastamento, exercício, lotação, norteador, motivo | Schema do Locus         |
| ATIVEOL                                                                    | Sem fonte. Fica de fora |


## Fluxo 2 — Relatórios

Dois caminhos. O total de lotados, afastados e vagas volta na tela. O relatório completo — cargos, pessoas afastadas e o motivo do afastamento — é job: o worker consolida o quadro e a tela baixa o arquivo.

O worker lê o schema do Locus e grava uma base de relatórios. O MS-Locus Relatórios lê só essa base. A tela de relatórios chama o MS de forma síncrona no total e faz o download do arquivo quando o job termina. O Front Locus redireciona para essa tela. O MS-Relatorios não entra: os tipos que ele gera são de escolha e convocação.

![Containers dos Relatórios](../../diagramas/assets/19-reaproveitamento-relatorios.png)


| Peça                | Papel                                                                    |
| ------------------- | ------------------------------------------------------------------------ |
| Schema do Locus     | Fonte operacional. Lotados, afastados, vagas e cargos do módulo          |
| Worker MS-Locus     | Lê o quadro, grava o consolidado e gera o arquivo do relatório completo  |
| Base de relatórios  | PostgreSQL de leitura. O total e o arquivo não varrem a base operacional |
| MS-Locus Relatórios | Total em chamada síncrona. Relatório completo cria o job                 |
| Tela de Relatórios  | Mostra o número na hora e baixa o arquivo                                |
| Front Locus         | Redireciona para a tela de relatórios                                    |
| Arquivo             | PDF ou planilha, disponível quando o job termina                         |



| Pedido                                     | Caminho                                                          |
| ------------------------------------------ | ---------------------------------------------------------------- |
| Número total de lotados, afastados e vagas | Síncrono. A tela lê o consolidado e mostra o número              |
| Relatório completo                         | Assíncrono. O worker gera o arquivo. O usuário acompanha e baixa |


Nome de unidade e de cargo, quando o relatório precisa, são lidos no ALVO na consolidação.

## Fluxo 3 — Data base

Datas operacionais do módulo ficam no Locus. A migração não busca o Sistema Escolha: lê agenda, processo, habilitado e escolha que o ALVO já tem, e o usuário confirma a gravação do vínculo.

![Containers da Data base](../../diagramas/assets/20-reaproveitamento-database.png)


| Passo do levantamento               | Neste cenário                                                                                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Data base EOL = ATIVEOL             | Sem data equivalente no ALVO. A tela mostra Data base DOC e, como referência de processo, `nomeacao_em` e `data_fechamento_modulo` |
| 1º passo da migração                | Consulta MS-Agenda, MS-ProcessosConvocacao, MS-Candidatos e MS-Escolha                                                             |
| 2º passo, Acrescenta migração       | Grava o vínculo novo no schema do Locus                                                                                            |
| Controle de atualizações e extratos | Leitura da auditoria e da movimentação do Locus                                                                                    |


## Fluxo 4 — Vagas

O cálculo usa módulo, lotados e afastados do schema do Locus. A vaga calculada é publicada no MS-Escolha. O confronto com o EOL não tem de onde ler a tabela MOU.

![Containers de Vagas](../../diagramas/assets/21-reaproveitamento-vagas.png)

A publicação preenche `vagas_definitivas` e `vagas_precarias` da escola e do cargo, com a data de fechamento do módulo. O MS-ImportaArquivos segue exportando esse registro no formato SIGPEC, como já faz para o ALVO. O Locus não gera o CSV nem fala com o SIGPEC.


| Opção da tela                             | Neste cenário                                                                  |
| ----------------------------------------- | ------------------------------------------------------------------------------ |
| Vagas Fund. II e Médio, Vagas Médio       | Calculadas no MS-Locus                                                         |
| Ajuste de EXC                             | Gravado no schema do Locus e recalculado                                       |
| Escolha                                   | Publicação no MS-Escolha. Os passos manuais de limpar coluna e enviar CSV saem |
| Confronto com EOL                         | Fora. Não há tabela MOU importada em nenhum MS                                 |
| Módulo, lotados, afastados e vagas do EOL | Fora. A tela pode mostrar os números do Locus                                  |


## Fluxo 5 — Remoção

Consultas, equipe, congelado e vaga final usam o schema do Locus e a escola do MS-Escolha. Confronto com EOL e planilha do CI não têm dado reaproveitável. Vaga precária do MS-Escolha é a oferta do processo, não a lista de professores precários do CI.

![Containers da Remoção](../../diagramas/assets/22-reaproveitamento-remocao.png)


| Bloco                                                      | Neste cenário                                                        |
| ---------------------------------------------------------- | -------------------------------------------------------------------- |
| Confronto de módulos, lotados, afastados e vagas com o EOL | Fora                                                                 |
| Precários do CI                                            | Fora                                                                 |
| Afastado perde lotação, outros afastamentos, liminar       | Leitura do schema do Locus                                           |
| Equipe da unidade                                          | Vínculos do Locus na escola do MS-Escolha                            |
| Congelado nosso                                            | Fotografia gravada no schema do Locus                                |
| Congelado EOL                                              | Fora                                                                 |
| Vagas para remoção                                         | Cálculo no MS-Locus a partir dos números conferidos no próprio Locus |
| Carga da remoção                                           | Continua a mapear no levantamento. Não há carga pronta em nenhum MS  |


## O que cada container deixa de existir

Nas telas, esse componente não existe no MS-Locus. O lugar da leitura de concurso, convocação e escolha é o ALVO. A busca periódica no EOL fica no Airflow. O arquivo Access entra na carga inicial, executada pelo sistema.