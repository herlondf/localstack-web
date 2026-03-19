# Passo a passo — localstack-web no Docker (WSL Ubuntu com systemd)

> Objetivo: levantar **localstack-web** + **LocalStack** via **Docker Engine dentro do WSL** (dockerd), com **S3/SQS/DynamoDB/etc + API Gateway**, e depois validar na UI.

---

## 0) Premissas / decisões

- Docker Engine rodando **dentro** do WSL (não Docker Desktop).
- **systemd habilitado** no WSL (necessário para serviço do Docker sem gambiarras).
- Rodar o projeto no filesystem do Linux (ex.: `~/src`) para evitar dor/performance ruim em `/mnt/*`.

---

## 1) (Se necessário) habilitar systemd no WSL

Se `ps -p 1 -o comm=` não retornar `systemd`, habilite:

1) Editar `/etc/wsl.conf`:

```ini
[boot]
systemd=true
```

2) No Windows (PowerShell/CMD):

```powershell
wsl --shutdown
```

3) Abrir o Ubuntu de novo e validar:

```bash
ps -p 1 -o comm=
systemctl is-system-running
```

---

## 2) Atualizar Ubuntu

```bash
sudo apt update
sudo apt -y upgrade
sudo apt -y autoremove
```

---

## 3) Instalar Docker Engine + Compose v2 (dentro do WSL)

No Ubuntu 24.04 (apt):

```bash
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
```

(Usar sem sudo — opcional, recomendado)

```bash
sudo usermod -aG docker "$USER"
# Fechar o terminal e abrir de novo (ou newgrp docker)
```

Validar:

```bash
docker --version
docker compose version
docker ps
```

---

## 4) Clonar o projeto

```bash
sudo apt install -y git

mkdir -p ~/src
cd ~/src
rm -rf localstack-web-docker

git clone --depth 1 https://github.com/dantasrafael/localstack-web.git localstack-web-docker
cd localstack-web-docker
```

---

## 5) Habilitar API Gateway no LocalStack

O `docker-compose.yml` pode não listar `apigateway` em `SERVICES=`. Garanta que inclui:

```bash
sed -i -E 's/(^\s*-\s*SERVICES=).*/\1sns,sqs,s3,ses,lambda,dynamodb,kinesis,kms,apigateway/' docker-compose.yml

grep -n 'SERVICES=' docker-compose.yml
```

---

## 6) Subir a stack

```bash
docker compose up -d --build
```

---

## 7) Smoke tests

### 7.1) Containers e portas

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

### 7.2) Health do LocalStack (incluindo API Gateway)

```bash
curl -s http://127.0.0.1:4566/_localstack/health | head -c 400; echo
curl -s http://127.0.0.1:4566/_localstack/health | grep -o '"apigateway": *"[^"]*"' || true
```

Esperado:
- `"apigateway": "available"` (ou `running`, depende da versão)

### 7.3) Abrir UI

- Abrir no browser: `http://localhost:3000`

---

## 8) Parar / limpar

```bash
docker compose down
```

Dados do LocalStack persistem em:
- `${TMPDIR:-/tmp}/localstack` (default: `/tmp/localstack`)

---

## Troubleshooting rápido

### API Gateway aparece disabled
- Reconfira `SERVICES=...apigateway` e recrie stack:
  - `docker compose down && docker compose up -d --build`

### Permissão no /run/docker.sock
- Confirme que seu usuário está no grupo `docker` e reabriu a sessão.
