# Contexto — Servidores, Unidades Educacionais e lotação

## Contexto

Núcleo funcional do LOCUS: a DICAR consulta e mantém o quadro de **servidores** (RF (Registro Funcional), vínculos, cargos, situação) e de **Unidades Educacionais** (características, módulos, saldo de vagas, lotados e afastados), registrando cada alteração com motivo e ato. A tela fala com o SME-SIGLA-BFF. O SME-LOCUS-MICROSERVICO grava o quadro na base do Locus e lê, no ALVO, a unidade, o cargo e a pessoa que o concurso, a convocação e a escolha já gravaram.

```{image} ../assets/24-locus-contexto-lotacao.svg
:width: 100%
:alt: Contexto de lotação. Tela, SME-SIGLA-BFF, quadro na base do Locus e leitura do ALVO
```

---



## Objetivos

- Visão única e atualizada de lotação e exercício
- Registrar movimentações com fundamento legal (portaria) e preservar versões
- Controlar saldo de vagas por módulo/componente curricular

---



## Escopo



### Em escopo

- Gestão de UE: listar/filtrar, registrar, detalhar, excluir; características (escola municipalizada, EF I, EJA modular, São Paulo Integral, contabilizar UE + motivo); módulos por componente (base comum, linguagens adicionais); lotados (definitivo/precário); afastados
- Servidor: busca por RF/CPF/nome; dados funcionais; lotação/exercício; atividade; readaptação; encerramento → vacância
- Atualizações: unidade de exercício, unidade de lotação, sem RF/código, ato sem efeito, vacância
- Histórico e versões



### Fora de escopo

- Escolha de vaga por candidato (ALVO)
- Cálculo de folha

---



## Sistemas envolvidos


| Sistema | Interação |
| --- | --- |
| Telas de UE e servidor | Requisição da tela ao SME-SIGLA-BFF |
| SME-SIGLA-BFF | Valida o token, agrega e devolve o DTO |
| SME-LOCUS-MICROSERVICO | Grava lotação, exercício, afastamento, módulo e histórico |
| Base do Locus | Quadro do módulo |
| MS-Escolha | Unidade, tipo e sigla da DRE já gravados |
| MS-ProcessosConcursos | Código e descrição do cargo já gravados |
| MS-Candidatos | Nome, CPF e vínculo, quando o RF existe |


---



## Considerações

- Contratos da API estão definidos em `SME-SIGLA-LOCUS-Frontend/src/servicos/*` e schemas Zod — usar como base do OpenAPI do SME-LOCUS-MICROSERVICO.
- Afastamentos podem conter dados de saúde (a confirmar) — tratar como dado sensível.

