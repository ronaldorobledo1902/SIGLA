# Contexto Geral do Sistema — LOCUS (Lotação e Controle Unificado de Servidores)

O **LOCUS** é o subsistema do **SIGLA** que gerencia o quadro funcional da SME (Secretaria Municipal de Educação): servidores ativos e inativos, Unidades Educacionais, módulos, lotação/exercício, movimentações, vacância, histórico e relatórios. Também concentra a **importação de dados** e as **permissões** de ALVO e LOCUS.

O usuário entra pelo **Portal SIGLA** (credencial única via CoreSSO) e é redirecionado para `/locus/`. O LOCUS consome e confronta os dados oficiais do **EOL (Escola Online / PRODAM)** e do **SIGPEC (SEGES)** e entrega dados importados e permissões ao **ALVO**.

**Premissa de negócio:** uso interno pela DICAR (Divisão de Gestão de Carreiras); o LOCUS não é origem oficial dos dados funcionais.

```{image} ../assets/02-locus-contexto.svg
:width: 100%
:alt: Diagrama de Contexto (C1) — LOCUS
```

Para detalhamento das interações:

```{toctree}
:maxdepth: 1
:caption: Contextos

contexto-acesso
contexto-lotacao
contexto-integracao
```

---

## Atores e sistemas

| Elemento | Tipo | Papel |
|----------|------|-------|
| **Operador DICAR** | Person | Consulta e atualiza servidores, lotação/exercício, UE e módulos; trata divergências |
| **Administrador SIGLA** | Person | Gerencia usuários e permissões do LOCUS e do ALVO; layouts de importação |
| **Consulta (Jurídico / SEGES)** | Person externa | Leitura e relatórios — fase futura, acesso mínimo |
| **Portal SIGLA** | Software System | Login, recuperação de senha, seleção ALVO / LOCUS |
| **LOCUS** | Software System | Sistema em foco |
| **ALVO** | Software System (vizinho) | Convocação e escolha — fora do escopo; consome dados importados e permissões |
| **CoreSSO / API SME Integração** | Software System externo | Credenciais únicas da SME |
| **Keycloak** | Software System externo (TO-BE) | Gestor de identidades em estudo pela SME-COTIC |
| **EOL (PRODAM)** | Software System externo | Origem de servidores, cargos, unidades e DREs |
| **SIGPEC (SEGES)** | Software System externo | Dados funcionais, processos seletivos e lotes de nomeação |
| **E-mail SME (SMTP)** | Software System externo | E-mail de recuperação de senha |

---

## O que não faz parte deste contexto

- Convocação, agendamento e escolha de vagas (ALVO)
- Folha de pagamento e dados financeiros
- Acesso de cidadãos ou dos próprios servidores
