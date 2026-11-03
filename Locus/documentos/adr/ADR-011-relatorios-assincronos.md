# ADR-011 — Relatórios: síncronos até 3 s, assíncronos acima


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA / PO LOCUS            |
| **Sistema**      | MS-Locus                                |
| **Relacionadas** | ADR-004, ADR-006, ADR-007               |


---

## Contexto

O PO pede consultas em até 3 s e relatórios pesados processados de forma assíncrona. O frontend já gera PDF de tela (jsPDF + html2canvas). O ALVO tem um MS-Relatorios (WeasyPrint, openpyxl, docx), focado em convocação/escolha.

## Decisão

1. **Consultas e relatórios leves** (≤ 3 s): endpoint síncrono no MS-Locus lendo o read model; exportação de tela pode continuar no front.
2. **Relatórios pesados** (quadro funcional completo, por DRE, histórico): `POST /api/v1/relatorios/` cria um job (`202 Accepted`), o Worker MS-Locus gera XLSX/CSV/PDF em armazenamento persistente, e o front consulta status e baixa.
3. Dados pessoais **mascarados** por padrão no perfil Consulta.
4. Reaproveitar ou não o MS-Relatorios do ALVO fica **em aberto** (pergunta Q5 ao PO); a interface de job permite mover a geração depois.

## Consequências

### Positivas

- Telas não travam em picos (início do ano letivo, atribuição, remoção)
- Arquivos rastreáveis (quem gerou, quando, com quais filtros)

### Negativas

- Exige armazenamento e expurgo dos arquivos gerados
- UX (experiência do usuário) com espera/notificação

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Tudo síncrono | Não atende 3 s com 100 mil+ servidores |
| Tudo no MS-Relatorios do ALVO | Acopla domínios; decisão pendente |
| Ferramenta de BI (Business Intelligence) | Fase futura (painéis analíticos) |

## Referências

- Questionário do PO — §6
- `SME-SIGLA-LOCUS-Frontend` — `useExportarPdf`
- `SME-SIGLA-MS-Relatorios`
