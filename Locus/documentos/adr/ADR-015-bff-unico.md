# ADR-015 — SME-SIGLA-BFF único para ALVO e LOCUS


| Campo            | Valor                                      |
| ---------------- | ------------------------------------------ |
| **Status**       | Proposed                                   |
| **Data**         | 2026-11-24                                 |
| **Decisores**    | Arquitetura SIGLA                          |
| **Sistema**      | SIGLA (ALVO e LOCUS)                       |
| **Relacionadas** | ADR-002, ADR-009                           |


---

## Contexto

O front do ALVO e o front do Locus chamam cada microsserviço pelo próprio endereço, com o JWT no navegador. O código do ALVO ainda não tem um SME-SIGLA-BFF (Backend for Frontend). A partir desta decisão, os dois fronts passam a ter um único SME-SIGLA-BFF. Os microsserviços deixam de ser o destino do navegador.

## Decisão

1. Há um SME-SIGLA-BFF só. Ele atende o front do ALVO e o front do Locus.
2. O navegador envia a requisição da tela somente ao SME-SIGLA-BFF.
3. O SME-SIGLA-BFF valida o token, consulta os microsserviços necessários, agrega a resposta, transforma no DTO da tela e devolve uma resposta só.
4. Login continua no MS-AdminUsuarios, via CoreSSO. O SME-SIGLA-BFF é quem fala com esse serviço em nome da tela.
5. Chamada entre microsserviços continua com `X-API-Key`. O navegador não usa essa chave.
6. O Airflow da importação periódica do EOL não passa pelo SME-SIGLA-BFF. Ele busca o EOL e grava na base do Locus.

## Consequências

### Positivas

- O navegador conhece um endereço, o do SME-SIGLA-BFF
- A tela recebe o dado já no formato em que vai exibir
- A validação do token fica num lugar só, antes da consulta aos microsserviços

### Negativas

- O ALVO passa a ter um componente que o código atual não tem
- Toda tela nova dos dois sistemas publica o contrato no SME-SIGLA-BFF

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| Um SME-SIGLA-BFF para o ALVO e outro para o Locus | A decisão é um SME-SIGLA-BFF único para os dois |
| O front continuar chamando cada microsserviço | O navegador passaria a conhecer e alcançar cada API |

## Referências

- [ADR-002](ADR-002-autenticacao.md)
- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
