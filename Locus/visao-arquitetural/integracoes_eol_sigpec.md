# Integrações EOL e SIGPEC — LOCUS (proposta)

**Sistema:** LOCUS  
**Data:** Outubro/2026  
**Versão:** 1.0 (proposta — mecanismos a confirmar com PRODAM e SEGES)  

## 1. Contexto

O MVP do LOCUS depende do **consumo contínuo** dos dados oficiais do **EOL (Escola Online, mantido pela PRODAM)** e do **SIGPEC (mantido pela SEGES — Secretaria de Gestão)** para **confrontar** o quadro funcional e sinalizar divergências. O PO (Product Owner) define como premissa **ao menos uma sincronização por dia**; o ideal é refletir toda alteração da origem.

A forma de disponibilização (API, acesso a banco/view, arquivo, SFTP (SSH File Transfer Protocol)) **ainda não foi confirmada**. Por decisão do time, as integrações **reaproveitam o MS-ImportaArquivos** (ADR-005), que já possui layouts, validação, registro de erros, cliente PRODAM e Celery beat.

```{image} ../diagramas/assets/08-locus-fluxo-integracao-eol-sigpec.svg
:width: 100%
:alt: Fluxo de integração e conciliação EOL/SIGPEC
```

---

## 2. Responsabilidades

| Responsabilidade | Componente |
|------------------|------------|
| Agendar e executar a coleta | Worker ImportaArquivos (Celery beat) |
| Ler a origem (API PRODAM, arquivo SIGPEC) | `api_prodam` / upload de arquivo |
| Validar formato (layout posicional, tipos) | `validacao_*` + `Layout` |
| Registrar importação e erros | `Importacao*`, `ImportacaoErro`, `LogRequestHttp` |
| Entregar lote válido ao domínio | Cliente `api_locus` (REST + `X-API-Key`) → `MS-Locus /api/v1/integracao/` |
| Confrontar com o quadro LOCUS | `ConciliacaoService` (Worker MS-Locus) |
| Persistir divergências e aplicar o que for automático | MS-Locus (write model + histórico) |
| Decidir divergências não automáticas | Operador DICAR (tela de divergências) |

---

## 3. Fluxo EOL (PRODAM)

1. Beat dispara a carga diária fora do horário comercial.
2. Worker consulta a PRODAM (mecanismo a confirmar) — preferencialmente **incremental** (alterados desde a última carga).
3. Valida estrutura e registra a importação.
4. Envia lotes paginados ao MS-Locus (idempotência por `importacao_id` + chave natural RF/vínculo ou código da unidade).
5. MS-Locus enfileira a conciliação; cada registro resulta em **igual**, **atualizado automaticamente** (campos de origem oficial) ou **divergente** (exige decisão).
6. Resumo da execução fica disponível no menu **Integração**.

## 4. Fluxo SIGPEC

1. SEGES gera arquivo TXT/CSV (hoje **manual**, conforme wiki).
2. Operador faz upload na tela de importação do LOCUS com o layout cadastrado.
3. Segue os passos 3 a 6 do fluxo EOL.
4. Automação (SFTP/API) fica como evolução, quando a SEGES disponibilizar.

---

## 5. Regras de conciliação (a validar com o PO)

| Situação | Tratamento sugerido |
|----------|---------------------|
| Servidor no EOL e ausente no LOCUS | Criar registro com origem EOL e marcar para revisão |
| Servidor no LOCUS e ausente no EOL | Divergência "sem correspondência na origem" |
| Campo de origem oficial diferente (cargo, situação) | Atualizar a partir da origem e versionar |
| Lotação/exercício diferente | **Divergência** para decisão da DICAR (pode haver ato ainda não refletido na origem) |
| Unidade nova/encerrada no EOL | Atualizar cadastro de UE e sinalizar impacto em módulos |

---

## 6. Requisitos não funcionais

| Requisito | Valor |
|-----------|-------|
| Frequência mínima | 1x/dia |
| Janela | Fora do horário comercial |
| Volume | 100 mil+ servidores, ~1.600 UEs; preferir carga incremental |
| Idempotência | Reprocessar a mesma importação não duplica registros |
| Rastreabilidade | Cada alteração aponta para a importação de origem |
| Falha | Retry com backoff; alerta; dados do dia anterior permanecem válidos |
| LGPD (Lei Geral de Proteção de Dados) | Arquivos com dados pessoais em armazenamento persistente e protegido; expurgo após processamento conforme retenção definida |

---

## 7. Perguntas em aberto

1. PRODAM oferece API ou view para servidores, vínculos, cargos e unidades? Há carga incremental?
2. SIGPEC pode ser disponibilizado via SFTP/API? Qual layout oficial?
3. Qual fonte prevalece em cada campo (EOL × SIGPEC × LOCUS)?
4. Quanto tempo os arquivos importados devem ser mantidos?
