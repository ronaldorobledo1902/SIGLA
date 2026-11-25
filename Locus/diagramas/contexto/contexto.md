# Contexto Geral do Sistema — LOCUS (Lotação e Controle Unificado de Servidores)

O **LOCUS** é o subsistema do **SIGLA** que gerencia o quadro funcional da SME (Secretaria Municipal de Educação): servidores ativos e inativos, Unidades Educacionais, módulos, lotação/exercício, movimentações, vacância, histórico e relatórios.

O usuário entra pelo **Portal SIGLA**. O **MS-AdminUsuarios** valida a credencial na API SME Integração e emite o JWT. O **MS-ImportaArquivos**, usado pelo ALVO, importa escolhas da PRODAM e exporta vagas e lotes no formato SIGPEC. O domínio de servidores e lotação é o **SME-LOCUS-MICROSERVICO**, ainda proposta.

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
| **Administrador SIGLA** | Person | Cadastro de usuários no MS-AdminUsuarios (grupos do ALVO). Perfis do LOCUS são proposta |
| **Consulta (Jurídico / SEGES)** | Person externa | Leitura e relatórios — fase futura, acesso mínimo |
| **Portal SIGLA** | Software System | Login, recuperação de senha, seleção ALVO / LOCUS |
| **LOCUS** | Software System | Sistema em foco |
| **ALVO** | Software System (vizinho) | Convocação e escolha |
| **CoreSSO / API SME Integração** | Software System externo | Validação da credencial (`/api/v1/autenticacao/externa`) |
| **EOL (PRODAM)** | Software System externo | Origem pretendida de servidores, cargos, unidades e DREs. O cliente PRODAM que existe hoje alimenta escolhas do ALVO |
| **SIGPEC (SEGES)** | Software System externo | No código, destino da exportação de vagas e lotes. O confronto do quadro com o SIGPEC é proposta |
| **E-mail SME (SMTP)** | Software System externo | E-mail de recuperação de senha |

---

## O que não faz parte deste contexto

- Convocação, agendamento e escolha de vagas (ALVO)
- Folha de pagamento e dados financeiros
- Acesso de cidadãos ou dos próprios servidores
