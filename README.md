# Docker Monitoring Stack

Monitoramento completo de **CPU, Memória, Disco e Rede** de todos os containers Docker, com dashboard Grafana pré-configurado.

Ideal para VPS com **Coolify**.

## Stack

| Serviço          | Porta | Função                                   |
|------------------|-------|------------------------------------------|
| **cAdvisor**     | 8080  | Coleta métricas de cada container Docker |
| **Node Exporter**| 9100  | Coleta métricas do host (servidor)       |
| **Prometheus**   | 9090  | Armazena e consulta métricas (30 dias)   |
| **Grafana**      | 3001  | Dashboard visual com gráficos            |

## Deploy via Coolify (recomendado)

### 1. Crie o repositório no GitHub

```bash
git init
git add .
git commit -m "monitoring stack"
git remote add origin git@github.com:SEU_USUARIO/docker-monitoring.git
git push -u origin main
```

### 2. No Coolify

1. Vá em **Projects** → escolha seu projeto (ou crie um novo)
2. Clique em **+ New Resource** → **Docker Compose**
3. Selecione **GitHub** como source
4. Conecte o repositório `docker-monitoring`
5. Em **Environment Variables**, adicione:

```
GRAFANA_USER=admin
GRAFANA_PASSWORD=SuaSenhaSegura!
GRAFANA_PORT=3001
GRAFANA_ROOT_URL=https://monitor.seudominio.com
PROMETHEUS_PORT=9090
PROMETHEUS_RETENTION=30d
CADVISOR_PORT=8080
NODE_EXPORTER_PORT=9100
```

6. Clique em **Deploy**

### 3. Acesse o Grafana

- URL: `http://SEU_IP:3001` ou o domínio configurado no Coolify
- Login: valores de `GRAFANA_USER` / `GRAFANA_PASSWORD`
- O dashboard **"Docker Containers - Monitoramento"** já é a home page

## Deploy manual (sem Coolify)

```bash
git clone git@github.com:SEU_USUARIO/docker-monitoring.git
cd docker-monitoring
cp .env.example .env
# edite o .env com suas configurações
nano .env
docker compose up -d
```

## O que o dashboard mostra

### Visão geral do host
- Gauges: CPU, Memória, Disco
- Uptime, RAM total/disponível, containers rodando, tráfego total

### Por container
- **CPU**: gráfico temporal com média, máx e valor atual na legenda
- **Memória**: gráfico temporal + ranking Top 10 (bar gauge)
- **Rede**: download (RX) e upload (TX) separados
- **Disco**: I/O leitura e escrita

### Alertas configurados
- CPU do host > 85% por 5min
- Memória do host > 90% por 5min
- Disco > 85% por 5min
- Container usando > 512MB RAM por 5min
- Container usando > 80% CPU por 5min
- Container possivelmente parado

## Segurança

Proteja as portas internas com firewall:

```bash
# Só permite Grafana externamente
ufw allow 3001/tcp

# Bloqueia acesso externo aos outros serviços
ufw deny 9090/tcp
ufw deny 8080/tcp
ufw deny 9100/tcp
```

Ou melhor: configure um **proxy reverso** no Coolify com HTTPS para o Grafana e não exponha as outras portas.

## Consumo estimado

| Serviço        | RAM        |
|----------------|------------|
| cAdvisor       | ~50-100 MB |
| Prometheus     | ~100-300 MB|
| Node Exporter  | ~20 MB     |
| Grafana        | ~50-100 MB |
| **Total**      | **~250-500 MB** |

## Comandos úteis

```bash
# Logs
docker logs grafana -f
docker logs prometheus -f

# Reiniciar
docker compose restart

# Parar
docker compose down

# Atualizar imagens
docker compose pull && docker compose up -d

# Recarregar config do Prometheus (sem restart)
curl -X POST http://localhost:9090/-/reload
```

## Personalização

- Edite `prometheus/alerts.yml` para ajustar thresholds dos alertas
- Edite `prometheus/prometheus.yml` para adicionar novos scrape targets
- Importe dashboards extras no Grafana em [grafana.com/dashboards](https://grafana.com/grafana/dashboards/)
  - Dashboard **193** (Docker) e **1860** (Node Exporter Full) são ótimos complementos
