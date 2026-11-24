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

A entrevista (`Levantamento/transcricaoreuniao.vtt`) confirmou os dois ritmos no Fundamental II. Daniela precisa do número total de lotados, afastados e vagas na hora, e de um relatório mensal completo — cargos, pessoas afastadas e o motivo — que hoje é montado à mão e processa muito dado. A imagem `Levantamento/imagem (5).png` mostra o padrão adotado para esse segundo caso: uma consolidação grava base dedicada, o microsserviço atende a tela de forma síncrona e o arquivo sai para download. Outro front apenas redireciona.

## Decisão

1. **Total simples** (lotados, afastados, vagas e consultas até 3 s): chamada síncrona. A tela mostra o número. A leitura é da base de relatórios, não do schema operacional.
2. **Relatório mensal completo** (cargos, afastados e o motivo do afastamento; também quadro por DRE e histórico): `POST /api/v1/relatorios/` cria um job (`202 Accepted`). O Worker MS-Locus consolida o schema do Locus na base de relatórios e gera XLSX/CSV/PDF. A tela consulta o status e faz o download.
3. A base de relatórios é PostgreSQL de leitura do módulo. O worker é quem grava nela. No desenho de referência esse papel é de um fluxo de ETL; aqui ele fica no Celery, que já é o assíncrono do SIGLA ([ADR-007](ADR-007-assincrono-celery.md)).
4. O Front Locus redireciona para a tela de relatórios. A tela fala com o MS-Locus de forma síncrona no total e acompanha o job no relatório completo.
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
| Relatório pesado lendo o schema operacional | Disputa recurso com o Cadastro na hora do fechamento |
| Ferramenta de BI (Business Intelligence) | Fase futura (painéis analíticos) |

## Referências

- Questionário do PO — §6
- `Levantamento/transcricaoreuniao.vtt` — trecho de relatórios
- `Levantamento/imagem (5).png` — base dedicada, serviço, tela e download
- `SME-SIGLA-LOCUS-Frontend` — `useExportarPdf`
- `SME-SIGLA-MS-Relatorios`
