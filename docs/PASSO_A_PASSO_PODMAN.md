# Passo a passo — localstack-web no Podman (WSL Ubuntu nova)

> Objetivo: levantar **localstack-web** + **LocalStack** via **Podman rootless** no WSL, com **S3/SQS/DynamoDB/etc + API Gateway**, e depois você validar na UI.
>
> Observação: este passo a passo parte de uma WSL Ubuntu recém-instalada e configura tudo do zero.

---

## 0) Premissas / decisões

- Rodar com **Podman rootless** (recomendado).
- Clonar o projeto **dentro do filesystem do WSL** (ex.: `~/src`) para evitar problemas de mount em `/mnt/*`.
- No Podman deste ambiente, **short-names** (ex.: `nginx:alpine`, `localstack/localstack:3.1`) podem falhar; por isso vamos usar imagens **fully-qualified** (`docker.io/...`).
- Vamos **remover** o mount `docker.sock` do compose, porque em Podman rootless geralmente não existe / não equivale a Docker.

---

## 1) Instalar WSL Ubuntu (referência)

> Esta etapa depende do Windows. Deixo apenas como guia (não é comando dentro do Ubuntu).

- No PowerShell (Admin):
  - `wsl --install -d Ubuntu --name Podman`
- Abrir a distro Ubuntu e criar usuário.

---

## 2) Atualizar Ubuntu (dentro do WSL)

```bash
sudo apt update
sudo apt -y upgrade
sudo apt -y autoremove
```

---

## 3) Instalar dependências (Podman + podman-compose + git)

```bash
sudo apt install -y git podman podman-compose
```

Validar:
```bash
podman --version
podman info --debug | sed -n '1,120p'
podman-compose --version
```

---

## 4) Clonar o projeto dentro do WSL

```bash
mkdir -p ~/src
cd ~/src
rm -rf localstack-web

git clone --depth 1 https://github.com/dantasrafael/localstack-web.git
cd localstack-web
```

Arquivos relevantes:
```bash
ls -la Dockerfile docker-compose.yml nginx.conf
```

---

## 5) Ajuste A — Qualificar imagens (docker.io/...) para evitar erro de short-name

### 5.1) Dockerfile (nginx)

```bash
cp -a Dockerfile Dockerfile.bak
sed -i 's|^FROM nginx:alpine|FROM docker.io/library/nginx:alpine|g' Dockerfile
```

### 5.2) docker-compose.yml (localstack)

```bash
cp -a docker-compose.yml docker-compose.yml.bak
sed -i 's|image: localstack/localstack:3.1|image: docker.io/localstack/localstack:3.1|g' docker-compose.yml
```

Checar:
```bash
grep -n '^FROM ' Dockerfile
grep -n 'image:' docker-compose.yml
```

---

## 6) Ajuste B — Remover docker.sock do compose (Podman rootless)

O `docker-compose.yml` original monta:

```yaml
- "/var/run/docker.sock:/var/run/docker.sock"
```

No Podman rootless isso é problemático. Remover a linha:

```bash
cp -a docker-compose.yml docker-compose.yml.bak2
sed -i '/\/var\/run\/docker\.sock:\/var\/run\/docker\.sock/d' docker-compose.yml

grep -n "docker.sock" docker-compose.yml || echo "OK: docker.sock não está no compose"
```

---

## 7) Ajuste C — Habilitar API Gateway nos serviços do LocalStack

Garanta que `SERVICES=` inclui `apigateway`:

```bash
sed -i -E 's/(^\s*-\s*SERVICES=).*/\1sns,sqs,s3,ses,lambda,dynamodb,kinesis,kms,apigateway/' docker-compose.yml

grep -n 'SERVICES=' docker-compose.yml
```

---

## 8) Override (opcional, recomendado) — LAMBDA_EXECUTOR=local

Crie `docker-compose.override.yml` (aplica automaticamente junto do compose):

```bash
cat > docker-compose.override.yml <<'EOF'
services:
  localstack:
    environment:
      - LAMBDA_EXECUTOR=local

```
Para encerrar digite: EOF
---

## 9) Validar e subir

Validar que o compose parseia:

```bash
podman-compose config >/dev/null
echo "OK: compose parseou"
```

Subir:

```bash
podman-compose up -d --build
```

---

## 10) Smoke tests (validar que está tudo ok)

### 10.1) Containers e portas

```bash
podman ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Esperado (exemplo):
- LocalStack em `127.0.0.1:4566`
- UI em `0.0.0.0:3000->80`

### 10.2) Health do LocalStack

```bash
curl -s http://127.0.0.1:4566/_localstack/health | head -c 400; echo
curl -s http://127.0.0.1:4566/_localstack/health | grep -o '"apigateway": *"[^"]*"' || true
```

Esperado:
- `"apigateway": "available"` (ou `running`, depende da versão)

### 10.3) Abrir UI

- Abrir no browser: `http://localhost:3000`
- Verificar que a UI conecta e lista serviços.

---

## 11) Parar / limpar

Parar:

```bash
podman-compose down
```

Dados persistem em:
- `/tmp/localstack` (por padrão, via `${TMPDIR:-/tmp}/localstack`)

---

## Troubleshooting rápido

### Erro: short-name não resolve (nginx/localstack)
- Sintoma: `short-name ... did not resolve ... no unqualified-search registries`
- Fix: usar `docker.io/library/nginx:alpine` e `docker.io/localstack/localstack:3.1` (etapa 5).

### API Gateway aparece disabled
- Fix: garantir `SERVICES=...,apigateway` e recriar stack (`podman-compose down && podman-compose up -d --build`).

### Montagens estranhas / warning de mount no rootless
- Garanta que o repo está em `~/...` e não em `/mnt/c/...`.
