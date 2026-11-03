# F14/F15 — Integração e conciliação EOL / SIGPEC (proposta)

| Campo | Valor |
|-------|-------|
| **Código** | F14 (Integração) / F15 (Importação de dados) |
| **Menu / rota** | Integração; telas de importação e layouts (a criar) |
| **Perfis** | Administrador (layouts), Operador DICAR (upload, divergências) |
| **MVP** | Sim |
| **Fluxo informado por** | Questionário do PO (jornada sequencial) + wiki (Importação de dados LOCUS) |
| **ADRs relacionadas** | ADR-004, ADR-005, ADR-007, ADR-008 |

---

## 1. Objetivo

Manter o LOCUS alinhado ao EOL (Escola Online / PRODAM) e ao SIGPEC (SEGES) ao menos uma vez por dia, sinalizando divergências para a DICAR.

## 2. Fluxo

```{image} ../assets/08-locus-fluxo-integracao-eol-sigpec.svg
:width: 100%
:alt: Fluxo de integração e conciliação EOL/SIGPEC
```

1. Celery beat dispara a consulta diária ao EOL/PRODAM (mecanismo a confirmar).
2. MS-ImportaArquivos valida layout/schema e registra a importação.
3. Lote validado é enviado ao MS-Locus (`/api/v1/integracao/`, `X-API-Key`).
4. MS-Locus enfileira a conciliação.
5. Worker confronta, grava divergências e histórico.
6. SEGES envia o arquivo SIGPEC (processo manual).
7. Operador faz upload do TXT/CSV.
8. Telas de importação enviam ao MS-ImportaArquivos (seguem os passos 2 a 5).
9. Operador analisa divergências e corrige.
10. Correção gravada no MS-Locus com histórico.

## 3. Regras de negócio

Ver [Integrações EOL / SIGPEC — §5](../../visao-arquitetural/integracoes_eol_sigpec.md).

## 4. Componentes envolvidos

| Camada | Componente | Responsabilidade |
|--------|------------|------------------|
| Frontend | Telas de importação, layouts, divergências (TO-BE) | Upload, histórico, decisão |
| MS-ImportaArquivos | API Layouts, API Importações, Importações LOCUS, Tasks, Cliente PRODAM, `api_locus` | Ingestão e validação |
| MS-Locus | API Integração interna, ConciliacaoService, Histórico | Domínio e decisão |

## 5. Pendências

- Mecanismo PRODAM e SIGPEC (API, view, arquivo, SFTP)
- Precedência de campos entre EOL, SIGPEC e LOCUS
- Retenção dos arquivos importados
