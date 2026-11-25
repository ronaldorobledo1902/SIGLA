# ADR-011 — Relatórios: síncronos até 3 s, assíncronos acima


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Revisão**      | 2026-11-25                              |
| **Decisores**    | Arquitetura SIGLA / PO LOCUS            |
| **Sistema**      | SME-LOCUS-MICROSERVICO                                |
| **Relacionadas** | ADR-004, ADR-006, ADR-007               |


---

## Contexto

O PO pede consultas em até 3 s e relatórios pesados processados de forma assíncrona. O frontend já gera PDF de tela (jsPDF + html2canvas). O ALVO tem um MS-Relatorios (WeasyPrint, openpyxl, docx), focado em convocação/escolha.

O corte é a demanda de processamento, não o assunto do relatório. O de menor processamento responde na hora. O de grande processamento dispara uma DAG do Airflow e envia um job ao worker. A DAG e o worker gravam na base do Locus. A tela mostra esse resultado quando a gravação termina. A base do Locus é separada da base do ALVO (ADR-006). Não há uma terceira base só de relatório.

## Decisão

1. **Síncrono:** relatório de menor processamento. A chamada responde na hora e a tela devolve o resultado de forma dinâmica. A leitura é a base do Locus.
2. **Assíncrono:** relatório de grande processamento. O SME-LOCUS-MICROSERVICO dispara uma DAG do Airflow e envia um job ao worker. A DAG e o worker gravam o resultado na base do Locus.
3. Não há base só de relatório. O síncrono e o assíncrono usam a base do Locus, distinta da base do ALVO.
4. O Front Locus redireciona para a tela de relatórios. A tela fala com o SME-SIGLA-BFF. O SME-SIGLA-BFF valida o token, devolve o relatório leve na hora e apresenta o pesado quando o Airflow termina de gravar.
5. Dados pessoais **mascarados** por padrão no perfil Consulta.
6. O MS-Relatorios do ALVO não emite o relatório do módulo. O motor de arquivo pode ser reutilizado; a dependência do quadro funcional não.

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
| Tudo no MS-Relatorios do ALVO | Acopla o relatório do quadro ao domínio de convocação |
| Base separada de relatório | O resultado fica na base do Locus |
| Ferramenta de BI (Business Intelligence) | Fase futura (painéis analíticos) |

## Referências

- Questionário do PO — §6
- `Levantamento/transcricaoreuniao.vtt` — trecho de relatórios
- `Levantamento/imagem (5).png` — padrão de tela e download; a base separada não foi adotada
- `SME-SIGLA-LOCUS-Frontend` — `useExportarPdf`
- `SME-SIGLA-MS-Relatorios`
