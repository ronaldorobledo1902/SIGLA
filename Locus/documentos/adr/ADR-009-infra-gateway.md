# ADR-009 — Infra: Ingress como API Gateway, Kubernetes Rancher, Jenkins


| Campo            | Valor                                       |
| ---------------- | ------------------------------------------- |
| **Status**       | Accepted (as-is) — a confirmar para o LOCUS |
| **Data**         | 2026-10-27                                  |
| **Decisores**    | Arquitetura SIGLA (wiki) / Infra Ágil       |
| **Sistema**      | SIGLA                                       |
| **Relacionadas** | ADR-002, ADR-010                            |


---

## Contexto

A wiki define Kubernetes via Rancher, Ingress Controllers como API Gateway (com possibilidade de Keycloak, TLS, circuit breaker e rate limiting), Jenkins para CI/CD e Grafana + Elastic para observabilidade. Os repositórios LOCUS têm Dockerfile, mas nenhum Jenkinsfile versionado (só `CODEOWNERS`).

## Decisão

1. LOCUS roda no **Rancher SME**, com front em `sigla.sme.prefeitura.sp.gov.br/locus` e APIs em `api-sigla.sme.prefeitura.sp.gov.br/ms-*`.
2. **Ingress Nginx** como gateway: TLS, roteamento por path, **rate limiting** e limites de upload para importação.
3. **Jenkins** (Infra Ágil) + **Registry SME**; deploy por branch (ADR-010).
4. Configuração por ConfigMap/Secret; o frontend recebe `VITE_LOCUS_API_URL` por build arg ou por injeção em runtime (`window.__ENV__`).

## Consequências

### Positivas

- Mesmo runtime do restante do SIGLA
- Gateway sem novo produto

### Negativas

- Pipeline fora do repositório dificulta revisão e reprodutibilidade
- Funções de gateway (auth centralizada, quotas) limitadas ao que o Ingress oferece

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| API Gateway dedicado (Kong etc.) | Fora do padrão SME |
| GitHub Actions | Esteira oficial é Jenkins |

## Referências

- Wiki SIGLA — API Gateway, Tecnologias Utilizadas
- `SME-SIGLA-LOCUS-Frontend/Dockerfile`, `nginx.conf`, `entrypoint.sh`
