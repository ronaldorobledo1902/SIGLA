# Contexto — Importação e integração EOL / SIGPEC

## Contexto

O LOCUS precisa consumir os dados oficiais do **EOL (PRODAM)** e do **SIGPEC (SEGES)** ao menos 1x/dia e confrontá-los com o quadro funcional. A ingestão reaproveita o **MS-ImportaArquivos** (ADR-005), que já tem layout posicional, validação e histórico; a conciliação fica no **MS-Locus**.

```{image} ../assets/06-locus-componente-ms-importa-arquivos.svg
:width: 100%
:alt: Componentes do MS-ImportaArquivos (uso LOCUS)
```

---

## Objetivos

- Manter o LOCUS alinhado às fontes oficiais
- Sinalizar divergências para decisão da DICAR
- Rastrear cada importação (arquivo, autor, erros)

---

## Escopo

### Em escopo

- Cadastro de layout posicional (txt/csv; posição, tamanho e tipo de cada campo)
- Upload de arquivo SIGPEC e validação contra o layout
- Carga agendada a partir da PRODAM (mecanismo a confirmar)
- Entrega de lotes ao MS-Locus e conciliação
- Histórico de importações e erros

### Fora de escopo

- Exportações para o ALVO (vagas SIGPEC, lotes de nomeação) — continuam no MS-ImportaArquivos, sem mudança

---

## Fluxo resumido

```text
Beat → Worker ImportaArquivos → EOL/PRODAM → valida → MS-Locus /integracao → conciliação → divergências
Operador → upload SIGPEC → MS-ImportaArquivos → (mesmo caminho)
```

Fluxo completo em [Integração EOL / SIGPEC](../funcionalidades/integracao-eol-sigpec.md).

---

## Sistemas envolvidos

| Sistema | Interação |
|---------|-----------|
| MS-ImportaArquivos + Worker | Ingestão, validação, histórico |
| MS-Locus + Worker | Conciliação e aplicação |
| EOL / PRODAM | Origem (API/view a confirmar) |
| SIGPEC / SEGES | Origem (arquivo TXT/CSV) |
| Redis | Broker Celery |

---

## Considerações

- Mudanças no MS-ImportaArquivos exigem regressão dos fluxos do ALVO.
- Arquivos com dados pessoais precisam de armazenamento persistente e política de expurgo.
