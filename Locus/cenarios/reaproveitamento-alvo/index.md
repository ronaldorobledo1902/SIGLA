# Cenário — Locus reaproveita o que o ALVO já importou

**Status:** cenário de reaproveitamento do ALVO. O front do ALVO e o do Locus usam um BFF único ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)). A entrevista de `transcricaoreuniao.vtt` alterou a ligação com o Sistema Escolha no [ADR-012](../../documentos/adr/ADR-012-modulo-fundamental-ii.md) e no [ADR-014](../../documentos/adr/ADR-014-escolha-precarios.md). O [ADR-005](../../documentos/adr/ADR-005-integracao-importa-arquivos.md) permanece para o MS-ImportaArquivos: o Locus não passa a importar SIGPEC por esse serviço. A busca periódica do EOL fica no Airflow, que grava JSON, PDF ou hash na base do Locus.

**Premissa da reunião com os devs:** as telas leem o que o ALVO já importou e guardou nos microsserviços existentes. A busca periódica do EOL é feita pelo Airflow, fora da tela.

**Entrevista posterior** (`Levantamento/transcricaoreuniao.vtt`): o ALVO e o Locus não integram com o Sistema Escolha. Esse sistema é o legado do ALVO e deixa de ser usado. A carga inicial do Locus é o Banco Access (servidores e unidades). A planilha citada na entrevista é a do EOL, carregada hoje no Access; não é um arquivo do Escolha. O nomeado que a equipe lança à mão no Access passa a ser lido no ALVO.

**Recorte:** os cinco fluxos do levantamento *Locus – Fundamental II e Médio* (Cadastro, Relatórios, Data base, Vagas, Remoção).

```{toctree}
:maxdepth: 1

analise-microservicos
base-de-dados
containers
```

## O que muda em relação ao desenho atual

| Tema | Levantamento de 05/11 | Depois da entrevista |
| --- | --- | --- |
| Quem fala com EOL, PRODAM, SIGPEC e CI | MS-Locus | O Airflow busca só o EOL e grava na base do Locus. PRODAM, SIGPEC e CI ficam fora do MS-Locus |
| Sistema Escolha (legado) | Sistema externo, mecanismo em aberto | Fora. Sem chamada, sem carga e sem retorno. O MS-Escolha do ALVO permanece como dono de escola, DRE e vaga do processo |
| Banco Access | Não aparecia | Carga inicial do schema do Locus, uma vez |
| Dado que o MS-Locus grava | Tudo do módulo, inclusive o que veio de fora | Só o que nenhum MS do ALVO tem dono |
| Base | PostgreSQL `db_sigla`, credencial do MS-Locus | Igual. O schema do Locus continua necessário |

## Leitura

1. [O que cada microsserviço atende](analise-microservicos.md)
2. [Por que o Locus ainda precisa da própria base](base-de-dados.md)
3. [Containers de cada fluxo](containers.md)

## O que este cenário não cobre

O ALVO importa e guarda o processo de concurso, convocação e escolha. Não guarda o professor que já está na rede.

Ficam sem fonte neste cenário, e por isso não entram no desenho como integração:

- Quadro funcional de quem não passou pelo concurso atual: lotação, exercício, afastamento, vacância, norteador e módulo da unidade.
- ATIVEOL e as tabelas de confronto do EOL (sufixo MOU).
- Planilha de precários do CI (RF, VINC, NOME, CODCAR, TPLOT).
- Uma data única de “data base EOL”.

Esses quatro itens existem no levantamento. Este cenário não inventa uma origem para eles. A comparação com o desenho atual é justamente se o módulo vive sem esses confrontos ou se a importação direta volta a ser necessária.
