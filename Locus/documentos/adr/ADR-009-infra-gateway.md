# ADR-009 — Infra observada nos repositórios


| Campo            | Valor                          |
| ---------------- | ------------------------------ |
| **Status**       | Accepted (as-is)               |
| **Data**         | 2026-11-17                     |
| **Decisores**    | Arquitetura SIGLA              |
| **Sistema**      | SIGLA                          |
| **Relacionadas** | ADR-002, ADR-010               |


---

## Contexto

Os repositórios do SIGLA trazem Dockerfile. O cliente Elastic APM está nas settings e permanece desligado enquanto `ELASTIC_APM_ENABLED` não for `1`. O README do `SME-SIGLA-MS-ProcessosConvocacao` cita dois deployments no Rancher/Kubernetes (QA e homologação).

Não há, nestes repositórios, manifesto de Ingress, rate limiting, circuit breaker, Grafana nem Jenkinsfile.

## Decisão

1. Ingress, rate limiting, circuit breaker e um gateway de identidade **não** fazem parte da descrição do sistema atual.
2. A observabilidade presente no código é o cliente Elastic APM (desligado por padrão) e os logs JSON do `sigla-sdk`.
3. A esteira de deploy não está versionada nos repositórios do LOCUS.
4. A URL do frontend continua no build (`VITE_LOCUS_API_URL` / `VITE_BASE_PATH`).

## Consequências

### Positivas

- A documentação descreve o que os repositórios contêm
- Não se assume quota, TLS de borda nem um segundo provedor de identidade

### Negativas

- O caminho real de publicação (host, TLS, orquestração do LOCUS) continua fora destes repositórios

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Descrever um API Gateway com rate limit e identidade na borda | Não há manifesto nem código correspondente |

## Referências

- `SME-SIGLA-LOCUS-Frontend/Dockerfile`, `nginx.conf`, `entrypoint.sh`
- Settings `ELASTIC_APM_*` dos microsserviços
- `SME-SIGLA-MS-ProcessosConvocacao/README.md` — menção a Rancher
