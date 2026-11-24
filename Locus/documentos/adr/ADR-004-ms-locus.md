# ADR-004 — MS-Locus: microsserviço do quadro em exercício


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Revisão**      | 2026-11-16                              |
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
| Arquivo que entra e sai | MS-ImportaArquivos |
| Relatório da escolha e da convocação | MS-Relatorios |

O que não está em nenhum deles é o servidor em exercício: vínculo, cargo base ou sobreposto, situação funcional, lotação, afastamento, módulo e saldo da unidade em funcionamento, vacância, ato sem efeito e o confronto do quadro com EOL/SIGPEC.

## Decisão

Criar o **MS-Locus** (`SME-SIGLA-MS-Locus`, rota `api-sigla.sme.prefeitura.sp.gov.br/ms-locus`), seguindo ADR-001, **somente para o domínio que não tem dono**. Escola, DRE, cargo de concurso e candidato são referenciados nos serviços acima, pelo código da escola e pelo Registro Funcional (RF). Não são cadastrados de novo.

1. **Apps do MS-Locus:**
   - `catalogos` — componente curricular, motivo de atualização e portaria (não existem em outro MS)
   - `unidades` — módulo por componente, saldo, lotados e afastados da unidade em funcionamento, apontando para a escola e a DRE do MS-Escolha
   - `servidores` — vínculo, cargo em exercício e situação funcional, apontando para o candidato do MS-Candidatos pelo Registro Funcional (RF) quando a pessoa veio de concurso
   - `lotacao` — lotação definitiva ou precária, unidade de exercício e afastamento
   - `movimentacoes` — atualização com motivo, vacância e ato sem efeito
   - `conciliacao` — divergências do quadro frente a EOL/SIGPEC
2. **APIs públicas** (JWT) espelhando os contratos do frontend. A API lê escola, DRE, cargo de concurso e candidato nos MS já existentes. **API interna** `/api/v1/integracao/` (`X-API-Key`) recebe lotes do MS-ImportaArquivos (ADR-005).
3. **Worker Celery** (fila `locus`) para conciliação e relatórios assíncronos. Relatório do quadro funcional fica em aberto: reaproveitar o MS-Relatorios ou expor no MS-Locus (ADR-011).
4. Consultas e gravações do LOCUS na **Base de Dados SIGLA**, o mesmo PostgreSQL dos demais MS (ADR-006).

## Consequências

### Positivas

- Contratos já desenhados na UI continuam válidos para o que é do quadro
- Cada cadastro permanece no serviço cuja responsabilidade já é aquela
- Regras de lotação e vacância ficam isoladas do fluxo de concurso e escolha

### Negativas

- Mais um deploy, pipeline e worker para operar
- O MS-Locus depende da chave da escola e do Registro Funcional (RF) publicados pelos MS do ALVO

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Recriar DRE, escola, cargo e candidato dentro do MS-Locus | Duplica cadastros que já têm dono |
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
