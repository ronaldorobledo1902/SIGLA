# Contexto — Importação e integração EOL / SIGPEC

## Contexto

O LOCUS precisa consumir os dados oficiais do **EOL (PRODAM)** e do **SIGPEC (SEGES)** ao menos 1x/dia e confrontá-los com o quadro funcional. Isso é proposta (ADR-005). Hoje o **MS-ImportaArquivos** atende o ALVO: layout de arquivo (tipo e estrutura JSON, padrão `HABILITADOS`), importação de escolhas da PRODAM e exportação SIGPEC. A conciliação do quadro ficaria no **SME-LOCUS-MICROSERVICO**.

```{image} ../assets/06-locus-componente-ms-importa-arquivos.svg
:width: 100%
:alt: Componentes do MS-ImportaArquivos (uso ALVO)
```

---

## Objetivos

- Manter o LOCUS alinhado às fontes oficiais
- Sinalizar divergências para decisão da DICAR
- Rastrear cada importação (arquivo, autor, erros)

---

## Escopo

### Em escopo da proposta

- Novos tipos de importação para servidores e unidades
- Entrega de lotes ao SME-LOCUS-MICROSERVICO e conciliação
- Histórico de importações e erros (o registro de erro já existe no serviço)

### O que o serviço já faz (ALVO)

- Layout de arquivo e importação de habilitados
- Carga de escolhas da PRODAM
- Exportação de vagas e lotes no formato SIGPEC

---

## Fluxo resumido

```text
Beat → Worker ImportaArquivos → escolhas PRODAM → MS-Escolha (comportamento atual)
Proposta: EOL/PRODAM → valida → SME-LOCUS-MICROSERVICO /integracao → conciliação → divergências
```

---

## Sistemas envolvidos

| Sistema | Interação |
|---------|-----------|
| MS-ImportaArquivos + Worker | Ingestão, validação, histórico |
| SME-LOCUS-MICROSERVICO + Worker | Conciliação e aplicação |
| EOL / PRODAM | Origem (API/view a confirmar) |
| SIGPEC / SEGES | Destino da exportação atual de vagas e lotes. Origem do quadro do LOCUS é proposta |
| Redis | Broker Celery |

---

## Considerações

- Mudanças no MS-ImportaArquivos exigem regressão dos fluxos do ALVO.
- Arquivos com dados pessoais precisam de armazenamento persistente e política de expurgo.
