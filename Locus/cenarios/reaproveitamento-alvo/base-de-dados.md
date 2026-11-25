# Base do Locus

Sim. O MS-Locus precisa de um schema próprio, no mesmo PostgreSQL `db_sigla` dos outros microsserviços, com credencial que não lê o schema dos demais.

Reaproveitar o ALVO não elimina essa base. O ALVO guarda o processo de escolha. O Locus guarda o que acontece com o professor depois que ele entra no módulo, e o cálculo que só o módulo faz.

## Por que não basta a base que já existe

| Dado do levantamento | Já está no ALVO? |
| --- | --- |
| Escola, DRE, cargo, habilitado, agenda, escolha, vaga ofertada | Sim. O MS-Locus consulta. Não copia o cadastro. |
| Ficha do Cadastro: cargo atual, atividade, tipo de vaga ocupada, exercício, lotação, afastamento, substituição, vacância, norteador, motivo e autor da atualização | Não |
| Data base DOC e checagem de módulo | Não |
| Módulo por unidade e componente, EXC e vaga calculada | Não. A vaga do MS-Escolha é a oferta do processo, não este cálculo |
| Fotografia congelada da remoção | Não |
| Auditoria de negócio do Cadastro (autor, data, motivo, valor anterior e novo) | Não. O auditlog de cada MS registra a tabela daquele serviço |

Copiar escola, cargo e candidato para dentro do Locus duplicaria dono. O [ADR-004](../../documentos/adr/ADR-004-ms-locus.md) já recusa isso. Este cenário mantém a recusa e acrescenta a premissa da reunião: a cópia também não vem de uma nova importação de EOL ou SIGPEC.

## O que fica no schema do Locus

O MS-Locus grava só o que não tem dono. A referência ao ALVO é por chave natural, sem chave estrangeira entre schemas.

| Agregado | Conteúdo | Chave para o ALVO |
| --- | --- | --- |
| Vínculo do módulo | Cargo atual, atividade, tipo de vaga ocupada, situação funcional, classificação e norteador do vínculo | Registro Funcional, quando a pessoa existe no MS-Candidatos |
| Lotação e exercício | Unidade de lotação, unidade de exercício, título, item, realocado, excedente | `codigo_eol` da escola no MS-Escolha |
| Afastamento | Tipo, retorno, causa, documento | Registro Funcional do vínculo |
| Movimentação | Inclusão, atualização, vacância, ato sem efeito, motivo, autor e data | — |
| Datas do módulo | Data base DOC e data da checagem | — |
| Módulo da unidade | Quantidade por componente, lotados definitivos, lotados precários, afastados, EXC | `codigo_eol` e código do cargo |
| Congelado | Fotografia dos números no período da remoção | — |

A migração do fluxo de Data base escreve o vínculo na primeira vez em que o usuário aciona “Acrescenta migração”. A partir daí a ficha é do Locus. Escolha, classificação e data de nomeação continuam sendo lidas no ALVO. Não há sincronização de volta para o MS-Candidatos.

## O que não entra neste schema

- Escola, DRE e endereço. Continuam no MS-Escolha.
- Catálogo de cargo e concurso. Continuam no MS-ProcessosConcursos.
- Pessoa, CPF e classificação do concurso. Continuam no MS-Candidatos.
- Arquivo importado, layout e log da PRODAM. Continuam no MS-ImportaArquivos.
- Tabela MOU, planilha do CI e fila de divergência com o EOL. Este cenário não as recebe. Se a comparação de cenários decidir que o confronto continua obrigatório, a importação volta ao desenho atual e estas tabelas passam a ter dono no Locus.

## Como os serviços se falam

Leitura do ALVO: HTTP com `X-API-Key`, no padrão que MS-ImportaArquivos, MS-Relatorios e MS-ProcessosConvocacao já usam. O MS-Locus não abre conexão no banco dos outros.

Escrita no ALVO: um único caso. No fluxo de Vagas, o MS-Locus publica a vaga calculada no MS-Escolha (`vagas definitivas` e `vagas precárias` por escola e cargo). Hoje essa vaga também pode entrar por arquivo no MS-ImportaArquivos. A regra deste cenário é: vaga calculada pelo Fundamental II tem o MS-Locus como publicador; arquivo de vaga continua valendo para processo que o Locus não calcula. Os dois não gravam a mesma vaga do mesmo processo.

A carga inicial copia o Banco Access para este schema, uma vez. Não há sincronismo contínuo com o Access nem com o Sistema Escolha.

O relatório leve lê este schema na hora. O relatório pesado é gravado aqui pela DAG do Airflow. Não há base separada de relatório, como o [ADR-011](../../documentos/adr/ADR-011-relatorios-assincronos.md) descreve.
