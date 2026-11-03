# ADR-004 — MS-Locus: microsserviço de domínio de servidores e lotação


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA / Levantamento LOCUS  |
| **Sistema**      | LOCUS                                   |
| **Relacionadas** | ADR-001, ADR-005, ADR-008, ADR-011      |


---

## Contexto

O frontend LOCUS já consome (com dados estáticos) contratos de `/api/v1/unidades-educacionais/*` e `/api/v1/servidores/*`, mas **nenhum backend implementa esse domínio**. MS-AdminUsuarios (identidade) e MS-ImportaArquivos (ingestão) não são lugares adequados para regras de lotação, vacância e histórico.

## Decisão

Criar o **MS-Locus** (`SME-SIGLA-MS-Locus`, rota `api-sigla.sme.prefeitura.sp.gov.br/ms-locus`), seguindo ADR-001, com:

1. **Apps:** `core` (DRE, componentes curriculares, motivos, portarias), `unidades` (UE, características, módulos, saldo), `servidores` (servidor, vínculo, cargo, situação), `lotacao` (lotação definitiva/precária, exercício, afastamentos), `movimentacoes` (atualizações com motivo, vacância, ato sem efeito), `conciliacao` (divergências), `relatorios`.
2. **APIs públicas** (JWT) espelhando os contratos do frontend; **API interna** `/api/v1/integracao/` (`X-API-Key`) para lotes do MS-ImportaArquivos.
3. **Worker Celery** (fila `locus`) para conciliação, projeção de read model e relatórios.
4. Escrita no write model LOCUS do `DB_SIGLA`; leitura de telas e relatórios no read model (ADR-006).

## Consequências

### Positivas

- Contratos já validados pela UI aceleram a implementação
- Domínio isolado e testável; regras de negócio em `services`

### Negativas

- Mais um deploy, pipeline e worker para operar
- Entidades compartilhadas com o ALVO (UE, cargo) exigem acordo sobre quem é dono (proposta: LOCUS é dono do cadastro de UE e servidor no SIGLA)

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Colocar o domínio no MS-ImportaArquivos | Mistura ingestão com regras de negócio |
| Um MS por agregado (servidor, UE, lotação) | Granularidade excessiva para o time e o volume |
| Reaproveitar MS-Candidatos (ALVO) | Domínio diferente (candidato ≠ servidor) |

## Referências

- `SME-SIGLA-LOCUS-Frontend/src/servicos/*`, `src/paginas/*`
- `diagramas/componente/componente-backend.md`
