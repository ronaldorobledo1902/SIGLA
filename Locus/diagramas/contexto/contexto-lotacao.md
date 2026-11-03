# Contexto — Servidores, Unidades Educacionais e lotação

## Contexto

Núcleo funcional do LOCUS: a DICAR consulta e mantém o quadro de **servidores** (RF (Registro Funcional), vínculos, cargos, situação) e de **Unidades Educacionais** (características, módulos, saldo de vagas, lotados e afastados), registrando cada alteração com motivo e ato. O frontend já possui as telas de UE e de atualização por RF com dados estáticos; o backend é o **MS-Locus (TO-BE)**.

```{image} ../assets/05-locus-componente-ms-locus.svg
:width: 100%
:alt: Componentes do MS-Locus (TO-BE)
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

## Fluxo resumido

```text
Operador → LOCUS SPA → MS-Locus API (JWT) → Services de domínio → write model + histórico
                                                      └── Worker → read model → listagens/relatórios
```

---

## Sistemas envolvidos

| Sistema | Interação |
|---------|-----------|
| LOCUS SPA | Telas de UE, servidor e atualização |
| MS-Locus (TO-BE) | Regras, persistência, histórico |
| PostgreSQL SIGLA | Write model LOCUS + read model |
| MS-ImportaArquivos | Abastece dados oficiais (EOL/SIGPEC) |

---

## Considerações

- Contratos da API estão definidos em `SME-SIGLA-LOCUS-Frontend/src/servicos/*` e schemas Zod — usar como base do OpenAPI do MS-Locus.
- Afastamentos podem conter dados de saúde (a confirmar) — tratar como dado sensível.
