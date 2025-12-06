# Telegram Bot API Server

[![Build and Push Telegram Bot API](https://github.com/reloadlife/tg-api/actions/workflows/multiarch-build.yml/badge.svg)](https://github.com/reloadlife/tg-api/actions/workflows/multiarch-build.yml)

A self-hosted Telegram Bot API server implementation that allows you to run your own local instance of the Telegram Bot API. This project provides Docker images and deployment configurations for running the [Telegram Bot API](https://github.com/tdlib/telegram-bot-api) server in containerized environments.

## 🌟 Features

- **Self-hosted**: Run your own Telegram Bot API server instead of relying on `api.telegram.org`
- **Docker Support**: Pre-built multi-architecture Docker images
- **Kubernetes Ready**: Complete Kubernetes manifests and Helm charts included
- **Docker Compose**: Easy local deployment with Docker Compose
- **High Availability**: Support for multiple replicas and load balancing
- **SSL/TLS Support**: Integrated with Traefik and NGINX Ingress for HTTPS
- **Persistent Storage**: Data persistence with volume mounts and PVCs
- **Monitoring**: Built-in statistics endpoint support
- **Lightweight**: Based on Alpine Linux for minimal image size

## 📋 Prerequisites

Before deploying this project, ensure you have:

- **Telegram API Credentials**: 
  - `TELEGRAM_API_ID`: Your API ID from [my.telegram.org](https://my.telegram.org)
  - `TELEGRAM_API_HASH`: Your API Hash from [my.telegram.org](https://my.telegram.org)
- **Docker** (for Docker/Docker Compose deployments)
- **Kubernetes Cluster** (for Kubernetes deployments)
- **Helm 3.x** (for Helm chart deployments)

### Getting Telegram API Credentials

1. Visit [my.telegram.org](https://my.telegram.org)
2. Log in with your phone number
3. Go to "API Development Tools"
4. Create a new application
5. Copy your `api_id` and `api_hash`

## 🚀 Quick Start

### Using Docker

```bash
docker run -d \
  --name telegram-bot-api \
  -e TELEGRAM_API_ID=your_api_id \
  -e TELEGRAM_API_HASH=your_api_hash \
  -p 8081:8081 \
  -v telegram-bot-api-data:/var/lib/telegram-bot-api \
  ghcr.io/reloadlife/tg-api:latest
```

### Using Docker Compose

1. Clone this repository:
   ```bash
   git clone https://github.com/reloadlife/tg-api.git
   cd tg-api
   ```

2. Create a `.env` file:
   ```bash
   cat > .env << EOF
   TELEGRAM_API_ID=your_api_id
   TELEGRAM_API_HASH=your_api_hash
   HOSTNAME=tg-api.yourdomain.com
   EOF
   ```

3. Start the services:
   ```bash
   docker compose up -d
   ```

The API will be available at `https://tg-api.yourdomain.com` with automatic SSL/TLS via Let's Encrypt.

### Using Kubernetes (kubectl)

1. Create a namespace:
   ```bash
   kubectl create namespace tg-api
   ```

2. Create a secret with your Telegram credentials:
   ```bash
   kubectl create secret generic tg-api-secret \
     --from-literal=apiId=your_api_id \
     --from-literal=apiHash=your_api_hash \
     -n tg-api
   ```

3. Apply the manifests:
   ```bash
   # Option 1: Using the all-in-one manifest
   kubectl apply -f manifest.yaml

   # Option 2: Using individual manifests
   kubectl apply -f k8s/namespace.yaml
   kubectl apply -f k8s/pvc.yaml
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   kubectl apply -f k8s/ingress.yaml
   ```

4. Update the ingress host in `k8s/ingress.yaml` to match your domain.

### Using Helm

1. Add required values:
   ```bash
   cat > values.yaml << EOF
   env:
     TELEGRAM_API_ID: "your_api_id"
     TELEGRAM_API_HASH: "your_api_hash"
   
   ingress:
     enabled: true
     hosts:
       - host: tg-api.yourdomain.com
         paths:
           - path: /
             pathType: Prefix
     tls:
       - secretName: tg-api-tls
         hosts:
           - tg-api.yourdomain.com
   EOF
   ```

2. Install the chart:
   ```bash
   helm install tg-api ./helm/tg-api -f values.yaml -n tg-api --create-namespace
   ```

3. Upgrade the deployment:
   ```bash
   helm upgrade tg-api ./helm/tg-api -f values.yaml -n tg-api
   ```

## ⚙️ Configuration

### Environment Variables

The following environment variables can be used to configure the Telegram Bot API server:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `TELEGRAM_API_ID` | Yes | - | Your Telegram API ID from my.telegram.org |
| `TELEGRAM_API_HASH` | Yes | - | Your Telegram API Hash from my.telegram.org |
| `TELEGRAM_LOG_FILE` | No | - | Path to log file (if empty, logs to stdout) |
| `TELEGRAM_STAT` | No | - | Enable statistics endpoint on port 8082 |
| `TELEGRAM_FILTER` | No | - | Filter requests by IP address |
| `TELEGRAM_MAX_WEBHOOK_CONNECTIONS` | No | 40 | Maximum webhook connections per bot |
| `TELEGRAM_VERBOSITY` | No | 0 | Logging verbosity level (0-10) |
| `TELEGRAM_MAX_CONNECTIONS` | No | - | Maximum total connections |
| `TELEGRAM_PROXY` | No | - | Proxy server address (e.g., socks5://proxy:1080) |
| `TELEGRAM_LOCAL` | No | - | Enable local mode for Bot API |
| `TELEGRAM_HTTP_IP_ADDRESS` | No | - | HTTP server IP address |

### Ports

- **8081**: Main HTTP API endpoint
- **8082**: Statistics endpoint (when `TELEGRAM_STAT` is set)

### Storage

The server stores data in the following directories:
- `/var/lib/telegram-bot-api`: Persistent data storage
- `/tmp/telegram-bot-api`: Temporary files

Make sure these directories are properly mounted for data persistence.

## 📝 Usage

### Setting the Webhook

Once your server is running, you can use it with your bots by changing the API endpoint:

Instead of:
```
https://api.telegram.org/bot<token>/METHOD_NAME
```

Use:
```
https://your-domain.com/bot<token>/METHOD_NAME
```

### Example with Python

```python
import requests

BOT_TOKEN = "your_bot_token"
API_URL = "https://tg-api.yourdomain.com"

# Set webhook
webhook_url = f"{API_URL}/bot{BOT_TOKEN}/setWebhook"
response = requests.post(webhook_url, json={
    "url": "https://your-bot-webhook.com/webhook"
})
print(response.json())
```

### Example with Node.js (node-telegram-bot-api)

```javascript
const TelegramBot = require('node-telegram-bot-api');

const bot = new TelegramBot(token, {
  polling: true,
  baseApiUrl: 'https://tg-api.yourdomain.com'
});

bot.on('message', (msg) => {
  console.log(msg);
});
```

## 🏗️ Architecture

```
┌─────────────┐
│   Client    │
│   (Bot)     │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Load Balancer  │
│  (Traefik/NGINX)│
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌────────┐ ┌────────┐
│ TG-API │ │ TG-API │  (Multiple replicas)
│  Pod 1 │ │  Pod N │
└───┬────┘ └───┬────┘
    │          │
    └────┬─────┘
         │
         ▼
┌──────────────────┐
│  Persistent      │
│  Volume          │
│  (Bot Data)      │
└──────────────────┘
```

## 🔒 Security Considerations

1. **API Credentials**: Always store `TELEGRAM_API_ID` and `TELEGRAM_API_HASH` in secrets, never in plain text
2. **HTTPS**: Always use HTTPS in production to protect API tokens
3. **Token Filtering**: The included Traefik configuration automatically redacts bot tokens from logs
4. **Network Policies**: Consider implementing Kubernetes network policies to restrict access
5. **Resource Limits**: Configure appropriate resource limits to prevent resource exhaustion
6. **Regular Updates**: Keep the Docker image updated to receive security patches

## 📊 Monitoring

### Statistics Endpoint

Enable the statistics endpoint by setting `TELEGRAM_STAT=1`:

```bash
curl http://your-domain.com:8082/
```

### Health Checks

The deployment includes liveness and readiness probes:
- **Liveness**: Checks if the container is running (TCP check on port 8081)
- **Readiness**: Checks if the service is ready to accept traffic (TCP check on port 8081)

## 🔧 Troubleshooting

### Container fails to start

Check the logs:
```bash
# Docker
docker logs telegram-bot-api

# Kubernetes
kubectl logs -n tg-api deployment/tg-api

# Docker Compose
docker compose logs api
```

### Cannot connect to Telegram servers

1. Check your API credentials are correct
2. Verify network connectivity
3. If behind a proxy, set the `TELEGRAM_PROXY` environment variable
4. Check firewall rules

### High memory usage

1. Reduce the number of replicas
2. Adjust resource limits in the deployment
3. Consider using `TELEGRAM_MAX_CONNECTIONS` to limit connections

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📜 License

This project packages the [Telegram Bot API](https://github.com/tdlib/telegram-bot-api) server, which is licensed under the Boost Software License 1.0.

## 🔗 Links

- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [Telegram Bot API Server Source](https://github.com/tdlib/telegram-bot-api)
- [Get API Credentials](https://my.telegram.org)
- [Docker Hub](https://ghcr.io/reloadlife/tg-api)

## 📞 Support

If you encounter any issues or have questions:
- Open an issue on [GitHub](https://github.com/reloadlife/tg-api/issues)
- Check the [Telegram Bot API documentation](https://core.telegram.org/bots/api)

## 🙏 Acknowledgments

- [Telegram](https://telegram.org) for the Bot API
- [tdlib](https://github.com/tdlib/telegram-bot-api) for the Telegram Bot API server implementation
- All contributors to this project
