# O que cada microsserviço atende

Cada serviço abaixo foi lido pelo código que está nos repositórios. “Atende” significa que o dado já está persistido e o SME-LOCUS-MICROSERVICO pode lê-lo por HTTP, com a mesma chave de API que os serviços já usam entre si. O SME-LOCUS-MICROSERVICO não lê tabela de outro serviço.

A chave de ligação, quando a pessoa veio de concurso, é o Registro Funcional no MS-Candidatos, o código EOL da escola no MS-Escolha e o código do cargo no MS-ProcessosConcursos.

O **Sistema Escolha** não é o MS-Escolha. A entrevista (`Levantamento/transcricaoreuniao.vtt`) tirou os dois lados dessa integração: o ALVO não recebe carga do legado e não devolve nada para ele; o Locus não chama o legado. Jessica pediu para remover, do diagrama, a ligação “sistema escolha, vagas e candidatos”. O que o Locus consulta é o ALVO. O Banco Access, que hoje guarda servidores e unidades, vira a carga inicial do Locus. A planilha que a equipe sobe no Access é a do EOL, não um arquivo do Escolha.

## Resumo

| Microsserviço | Papel para o SME-LOCUS-MICROSERVICO | Fluxos |
| --- | --- | --- |
| MS-Escolha | Escola, DRE, vaga definitiva ou precária e escolha feita. Não é o Sistema Escolha | 1, 3, 4, 5 |
| MS-Candidatos | Pessoa habilitada, classificação e lote SIGPEC | 1, 3 |
| MS-ProcessosConcursos | Catálogo de cargo e concurso | 1, 4 |
| MS-ProcessosConvocacao | Processo, data de convocação e corte de vagas | 3 |
| MS-Agenda | Data de escolha e data de nomeação | 3 |
| MS-AdminUsuarios | Login e permissão. Sem dado do módulo | todos |
| sigla-sdk | Cliente HTTP, correlation id e auditoria de JWT. Não é fonte de dado | — |
| MS-ImportaArquivos | Porta de entrada do ALVO. O Locus não recebe lote dele | — |
| MS-Relatorios | Relatório de escolha e convocação. Não emite os relatórios do módulo | 2, em aberto |
| MS-Convocacao | CRUD genérico, substituído pelo MS-ProcessosConvocacao. Não integrar | — |

## MS-Escolha

Dono da unidade e da vaga ofertada no processo de escolha.

Já persiste:

- DRE: código, nome e sigla.
- Escola: `codigo_eol`, nome oficial, tipo de unidade, endereço e capacidade. A carga da DRE e da escola já sai da SME Integração (`/api/DREs` e escola por código EOL). O Locus não repete essa chamada.
- Vaga da escola: cargo, vagas definitivas, vagas precárias, data de fechamento do módulo e escola.
- Escolha: candidato, processo, situação (escolha, não escolha, reconvocação) e tipo de vaga (definitiva ou precária).

Atende no Fundamental II:

- Busca da unidade no Cadastro (código, tipo, nome, sigla da DRE).
- Lista de candidatos que já escolheram, no passo de migração da Data base.
- Destino das vagas calculadas. A publicação entra em `vagas_escolas` do ALVO. O CSV manual para o Sistema Escolha deixa de existir.
- Nome da unidade nos relatórios, na equipe e nos congelados.

Não atende: módulo da unidade por componente, lotados, afastados e o saldo usado na fórmula de vagas. `data_fechamento_modulo` e as quantidades definitiva e precária são a oferta do processo de escolha, não o módulo do quadro em exercício.

## MS-Candidatos

Dono da pessoa que entrou por concurso.

Já persiste:

- Candidato: nome, CPF, Registro Funcional e vínculo.
- Habilitado (`ConcursoCandidato`): código e descrição do cargo, classificação geral, NNA e PCD, convocação, eliminação, reclassificação e dados de lote SIGPEC (número do lote, código da empresa, número da vaga).

Atende:

- Abertura do Cadastro por RF quando a pessoa veio de concurso: nome, CPF, vínculo e classificação.
- Lista que o usuário confere antes de “Acrescenta migração”.

Não atende o banco inteiro de professores do módulo. Quem já está na rede e não foi importado como habilitado não existe aqui. Também não há lotação, exercício, afastamento, cargo base, norteador nem motivo de atualização.

## MS-ProcessosConcursos

Dono do cargo e do concurso.

Já persiste cargo (código e nome), concurso e autorização publicada. Cargo e tipo de concurso já são buscados na SME Integração (`/api/cargos` e `/api/concurso/tipos`). O Locus usa o catálogo gravado. Não chama a SME Integração.

Atende o código e a descrição do cargo nas telas de Cadastro e de Vagas (os componentes do Fund. II e do Médio são cargos desse catálogo, quando o código coincidir).

Não atende atividade, tipo de vaga ocupada, situação funcional nem jornada.

## MS-ProcessosConvocacao

Dono do processo que convoca habilitados.

Já persiste concurso, tipo de escolha, status, data de convocação, data de corte de vagas e percentuais de NNA e PCD. Também envia a carta de convocação.

Atende o recorte do processo na migração: qual concurso, qual data de convocação e qual corte de vagas.

Não é o cadastro funcional e não substitui o MS-Convocacao antigo: este último é um CRUD genérico (título, número, tipo, datas) e não participa do fluxo real. O SME-LOCUS-MICROSERVICO não integra com ele.

## MS-Agenda

Dono da sessão de escolha.

Já persiste processo, cargo, `escolha_em`, `nomeacao_em`, classificação, sessão e a lista de habilitados convocados para a sessão.

Atende a “Data base agenda” do fluxo de Data base: a data de nomeação que o levantamento descreve como vinda do Sistema Escolha está em `nomeacao_em`. A data de escolha está em `escolha_em`.

Não grava a Data base DOC nem a checagem de módulo. Essas duas datas são operação do Locus.

## MS-AdminUsuarios

Login no CoreSSO, JWT, usuário, grupo e permissão. O SME-SIGLA-BFF único consulta este serviço em nome do front do ALVO e do front do Locus. Não há dado de professor, unidade ou vaga.

## sigla-sdk

Biblioteca. O SME-LOCUS-MICROSERVICO usa o cliente HTTP, o correlation id e o middleware de auditoria do JWT, no mesmo padrão dos outros serviços. Não guarda domínio.

## MS-ImportaArquivos

Hoje ele recebe o CSV de habilitados, o arquivo de vagas e o TXT de lotes SIGPEC, busca a escolha na PRODAM e entrega o resultado ao MS-Candidatos e ao MS-Escolha. Também exporta vagas e lotes no formato SIGPEC.

Neste cenário ele continua sendo a porta de entrada do ALVO. O SME-LOCUS-MICROSERVICO não ganha cliente `api_locus`, não cria layout de EOL ou SIGPEC e não lê a tabela de importação. O dado útil já está no serviço de domínio depois que a importação termina.

A exportação SIGPEC que já existe continua saindo das vagas do MS-Escolha. Se o Locus publicar a vaga calculada nesse serviço, o arquivo SIGPEC segue sendo gerado por quem já o gera hoje.

## MS-Relatorios

Gera ata, súmula, lauda de vagas, lauda de convocação, listagem e resultado da escolha. Cada relatório chama os outros microsserviços e grava o arquivo.

Nenhum tipo atual é banco de servidores, quadro da unidade, cargos do módulo, mensal ou auditoria do Cadastro. O motor (job, planilha, documento) pode ser reutilizado depois. Neste cenário os relatórios do módulo nascem no SME-LOCUS-MICROSERVICO.

O relatório de menor processamento volta na hora. O de grande processamento dispara uma DAG do Airflow, que grava na base do Locus. A tela mostra esse resultado quando a DAG termina. Não há base separada de relatório. O MS-Relatorios não passa a depender do quadro funcional. O desenho está em [Containers](containers.md), fluxo 2.

## Matriz por fluxo

| Necessidade do levantamento | Onde já está | O que o SME-LOCUS-MICROSERVICO faz |
| --- | --- | --- |
| Unidade, tipo e DRE | MS-Escolha | Consulta por código EOL |
| Cargo (código e nome) | MS-ProcessosConcursos | Consulta o catálogo |
| Nome, CPF, RF, classificação de quem veio de concurso | MS-Candidatos | Consulta por RF |
| Data de nomeação e lista da sessão | MS-Agenda e MS-Candidatos | Lê para o 1º passo da migração |
| Escolha definitiva ou precária | MS-Escolha | Lê para saber quem escolheu |
| Inclusão do professor no Cadastro do módulo | em lugar nenhum | Grava na base do Locus |
| Lotação, exercício, afastamento, vacância, norteador | em lugar nenhum | Grava na base do Locus |
| Data base DOC e checagem de módulo | em lugar nenhum | Grava na base do Locus |
| Módulo, lotados, afastados e EXC | em lugar nenhum | Calcula e grava na base do Locus |
| Arquivo de vagas para o Sistema Escolha | Não há integração com o legado | Publica a vaga calculada no MS-Escolha do ALVO |
| Nomeado que ainda não assumiu | Hoje lançado à mão no Access | Lê MS-Candidatos, MS-Agenda e MS-Escolha |
| Carga inicial de servidores e unidades | Banco Access | Importa uma vez para o schema do Locus |
| Relatório de menor processamento | em lugar nenhum | Síncrono, lendo a base do Locus |
| Relatório de grande processamento | em lugar nenhum | DAG do Airflow grava na base do Locus. A tela mostra quando termina |
| ATIVEOL, tabela MOU, confronto com EOL | em lugar nenhum | Fora deste cenário |
| Login | MS-AdminUsuarios | Front autentica lá |
