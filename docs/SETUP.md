# Issue: localstack

## Contexto
Estamos trabalhando na implementação/integração da UI **localstack-web** dentro do repo DocFiscAll.
Diretório alvo: `/mnt/docfiscall/backend/Source/Utilitarios/Localstack/localstack-web`.

## Objetivo
- Ter uma UI web para inspeção/gerenciamento de recursos do LocalStack (S3, SQS, DynamoDB, Lambda, Kinesis, KMS, SNS, SES, API Gateway).
- Rodar via Docker Compose local e evitar problemas de CORS no browser.

## Estado atual (alto nível)
- Projeto Vue 3 + Vuetify 3 + Vite + Pinia + AWS SDK v3.
- Execução via `docker-compose.yml` com `localstack`, `mock-api`, `localstack-web`.
- Proxy via Nginx: `/localstack/` -> `http://localstack:4566/` para same-origin.

## Referências internas
- Checkpoint detalhado: `checkpoint.md`
- Plano incremental: `plan.md`
- Prompt de retomada: `resume_prompt.md`
