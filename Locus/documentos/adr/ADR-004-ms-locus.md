# ADR-004 — SME-LOCUS-MICROSERVICO: microsserviço do quadro em exercício


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Revisão**      | 2026-11-25                              |
| **Decisores**    | Arquitetura SIGLA                       |
| **Sistema**      | LOCUS                                   |
| **Relacionadas** | ADR-001, ADR-005, ADR-006, ADR-008, ADR-011 |


---

## Contexto

O frontend LOCUS já consome (com dados estáticos) contratos de `/api/v1/unidades-educacionais/*` e `/api/v1/servidores/*`, mas nenhum backend implementa lotação, exercício, vacância e o histórico do quadro em funcionamento.

Parte do que essas telas mostram já tem dono no ALVO:

| Dado | Onde já está |
|------|----------------|
| DRE, escola, vaga e escolha | MS-Escolha |
| Concurso e cargo do concurso | MS-ProcessosConcursos |
| Pessoa habilitada (Registro Funcional (RF), CPF, nome) | MS-Candidatos |
| Login e permissão | MS-AdminUsuarios |
| Arquivo que entra e sai no ALVO | MS-ImportaArquivos. O Locus não recebe esse lote |
| Relatório da escolha e da convocação | MS-Relatorios |

O que não está em nenhum deles é o servidor em exercício: vínculo, cargo base ou sobreposto, situação funcional, lotação, afastamento, módulo e saldo da unidade em funcionamento, vacância, ato sem efeito e o confronto do quadro com o snapshot do EOL.

## Decisão

Criar o **SME-LOCUS-MICROSERVICO** (`SME-SIGLA-SME-LOCUS-MICROSERVICO`, rota `api-sigla.sme.prefeitura.sp.gov.br/ms-locus`), seguindo ADR-001, **somente para o domínio que não tem dono**. Escola, DRE, cargo de concurso e candidato são referenciados nos serviços acima, pelo código da escola e pelo Registro Funcional (RF). Não são cadastrados de novo.

1. **Apps do SME-LOCUS-MICROSERVICO:**
   - `catalogos` — componente curricular, motivo de atualização e portaria (não existem em outro MS)
   - `unidades` — módulo por componente, saldo, lotados e afastados da unidade em funcionamento, apontando para a escola e a DRE do MS-Escolha
   - `servidores` — vínculo, cargo em exercício e situação funcional, apontando para o candidato do MS-Candidatos pelo Registro Funcional (RF) quando a pessoa veio de concurso
   - `lotacao` — lotação definitiva ou precária, unidade de exercício e afastamento
   - `movimentacoes` — atualização com motivo, vacância e ato sem efeito
   - `conciliacao` — divergências do quadro frente ao snapshot do EOL gravado na base do Locus
2. O contrato da tela fica no SME-SIGLA-BFF ([ADR-015](ADR-015-bff-unico.md)). O SME-LOCUS-MICROSERVICO expõe a consulta que o SME-SIGLA-BFF agrega. A API lê escola, DRE, cargo de concurso e candidato nos microsserviços do ALVO, por HTTP com `X-API-Key`. A busca periódica do EOL chega pela DAG do Airflow, que grava na base do Locus ([ADR-005](ADR-005-integracao-importa-arquivos.md)).
3. **Worker Celery** (fila `locus`) recebe o job do relatório de grande processamento e grava na base do Locus. A execução desse relatório e a busca periódica do EOL são DAGs do Airflow ([ADR-011](ADR-011-relatorios-assincronos.md)).
4. Consultas e gravações do LOCUS ficam na **base do Locus**, separada da base do ALVO ([ADR-006](ADR-006-base-unica.md)).

## Consequências

### Positivas

- Contratos já desenhados na UI continuam válidos para o que é do quadro
- Cada cadastro permanece no serviço cuja responsabilidade já é aquela
- Regras de lotação e vacância ficam isoladas do fluxo de concurso e escolha

### Negativas

- Mais um deploy, pipeline e worker para operar
- O SME-LOCUS-MICROSERVICO depende da chave da escola e do Registro Funcional (RF) publicados pelos MS do ALVO

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Recriar DRE, escola, cargo e candidato dentro do SME-LOCUS-MICROSERVICO | Duplica cadastros que já têm dono |
| Estender o MS-Candidatos ou o MS-Escolha com lotação, vacância e conciliação | Muda a responsabilidade desses serviços: um guarda o habilitado do concurso, o outro a vaga escolhida |
| Colocar o domínio no MS-ImportaArquivos | Mistura ingestão com regras de negócio |
| Um MS por agregado (servidor, lotação, movimentação) | Granularidade excessiva para o time e o volume |

## Referências

- `SME-SIGLA-LOCUS-Frontend/src/servicos/*`, `src/paginas/*`
- `SME-SIGLA-MS-Escolha` — `escola` (DRE, Escola), `vagas_escolas`
- `SME-SIGLA-MS-ProcessosConcursos` — `cargos`, `concursos`
- `SME-SIGLA-MS-Candidatos` — `candidatos`
- `SME-SIGLA-MS-Relatorios` — relatórios da escolha e da convocação
- `diagramas/componente/componente-backend.md`
- [ADR-012](ADR-012-modulo-fundamental-ii.md) e [containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
