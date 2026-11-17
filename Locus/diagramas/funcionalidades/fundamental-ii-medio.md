# Fundamental II e Médio

Recorte do levantamento de 05/11/2026. O texto está em [Fundamental II e Médio](../../visao-arquitetural/proposta-fundamental-ii-medio.md).

O usuário opera pelo front. O MS-Locus aplica a regra e acessa a base e o EOL. A forma dessa integração não está definida.

## Contexto — Fluxo 1 e Fluxo 2

![Contexto do Cadastro e dos Relatórios](../assets/11-fund2-contexto.svg)

## Containers — Cadastro

![Containers do Cadastro](../assets/12-fund2-containers-cadastro.svg)

## Containers — Relatórios

![Containers dos Relatórios](../assets/13-fund2-containers-relatorios.svg)

## Containers — Data base

![Containers da Data base](../assets/14-fund2-containers-database.svg)

## Containers — Vagas

![Containers de Vagas](../assets/15-fund2-containers-vagas.svg)

## Containers — Remoção

![Containers da Remoção](../assets/16-fund2-containers-remocao.svg)

| Fluxo | Entrada | O que faz |
| --- | --- | --- |
| 1 Cadastro | Fundamental II > Cadastro | Inclui e atualiza o professor. Grava na base de escrita. Acessa o EOL no ATIVEOL |
| 2 Relatórios e consultas | Fundamental II > Relatórios / Consultas | Só lê a base de leitura. Não acessa sistema externo |
| 3 Data base | Fundamental II > Data base | Datas de referência, migração de candidatos, controle e extratos. Acessa EOL e Sistema Escolha |
| 4 Vagas | Fundamental II > Vagas | Calcula vagas, exporta ao Sistema Escolha e confronta com o EOL |
| 5 Remoção | Fundamental II > Remoção | Confronto, precários do CI, consultas, congelados e vagas para remoção. A carga está a mapear |
