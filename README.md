# 🛍️ Shopify Backend Integration with NestJS

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Node.js Version](https://img.shields.io/badge/node-%3E%3D14.0.0-blue.svg)
![NestJS Version](https://img.shields.io/badge/nestjs-8.x-brightgreen.svg)

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Demo](#demo)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Application](#running-the-application)
- [Testing](#testing)
- [API Documentation](#api-documentation)
- [Deployment](#deployment)
- [Security Enhancements](#security-enhancements)
- [Monitoring and Logging](#monitoring-and-logging)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Introduction

This project is a robust NestJS backend designed for seamless integration with Shopify. It provides a powerful API for managing Shopify resources such as products, orders, and collections. Built with scalability and performance in mind, it leverages Mikro-ORM for efficient database management, Redis for lightning-fast caching, and BullMQ for reliable background job processing. The architecture ensures scalability, security, and monitoring using modern DevOps practices.

## Features

- **🔗 Shopify API Integration**: Manage Shopify products, collections, and orders using both REST and GraphQL APIs
- **🌐 Contentful Integration**: Manage and synchronize content with Contentful CMS
- **⚡ Redis Caching**: Optimize performance with intelligent Redis caching
- **📊 Background Job Processing**: Efficiently handle long-running tasks with BullMQ
- **🗄️ Mikro-ORM**: Streamlined database operations with PostgreSQL or MySQL
- **📈 Prometheus Monitoring**: Real-time system performance metrics
- **📝 Winston Logging**: Advanced, structured logging for better debugging
- **🚨 Sentry Error Tracking**: Proactive error monitoring and performance tracking
- **🔒 Enhanced Security**: Implements rate limiting, HTTPS, and secure environment variable management
- **☸️ Kubernetes-Ready**: Pre-configured for scalable Kubernetes deployment
- **🧪 Comprehensive Testing**: Includes unit, integration, and E2E tests

## Demo

[Include screenshots or a link to a live demo if available]

![Demo Screenshot](./screenshots/demo.png)

## Installation

### Prerequisites

- Node.js (v14 or higher)
- PostgreSQL or MySQL
- Redis
- Shopify Partner Account
- Contentful Account
- Docker & Docker Compose

### Steps

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/yourusername/your-repo-name.git
   cd your-repo-name
   ```

2. **Install Dependencies:**
   ```bash
   pnpm install
   ```

3. **Set Up Environment Variables:**
   Copy the `.env.example` file to `.env` and configure it with your credentials:
   ```bash
   cp .env.example .env
   ```
   Update the `.env` file with your Shopify API keys, Redis config, database details, and other relevant information.

4. **Database Setup:**
   Run migrations to set up the database:
   ```bash
   pnpm mikro-orm migration:up
   ```

## Configuration

The project supports both PostgreSQL and MySQL:

- **PostgreSQL**: Install PostgreSQL locally and set `type: 'postgresql'` in `mikro-orm.config.ts`.
- **MySQL**: Keep the `@mikro-orm/mysql` package and set `type: 'mysql'` in the configuration file.

## Running the Application

To start the app in development mode:

```bash
pnpm run start:dev
```

Your application will be available at `http://localhost:3000`.

## Testing

### Running Unit Tests
```bash
pnpm run test
```

### Running End-to-End Tests
```bash
pnpm run test:e2e
```

### Linting
```bash
pnpm run lint
```

## API Documentation

The project includes interactive API documentation using Swagger. Access it at `http://localhost:3000/api-docs` after starting the application.

## Deployment

### Docker Deployment

To run the application with Docker:

1. Build the Docker Image:
   ```bash
   docker-compose up --build
   ```

2. Access the Application:
   The app should be running on `http://localhost:3000`.

### Kubernetes Deployment

1. Apply Secrets:
   Configure your secrets in `k8s/secrets.yaml`:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: app-secrets
   type: Opaque
   data:
     SHOPIFY_API_KEY: <base64_encoded_api_key>
     SHOPIFY_API_SECRET_KEY: <base64_encoded_api_secret>
     CONTENTFUL_SPACE_ID: <base64_encoded_space_id>
     CONTENTFUL_ACCESS_TOKEN: <base64_encoded_access_token>
     SENTRY_DSN: <base64_encoded_sentry_dsn>
     REDIS_PASSWORD: <base64_encoded_redis_password>
     AWS_ACCESS_KEY_ID: <base64_encoded_aws_access_key_id>
     AWS_SECRET_ACCESS_KEY: <base64_encoded_aws_secret_access_key>
   ```

2. Apply the secrets:
   ```bash
   kubectl apply -f k8s/secrets.yaml
   ```

3. Deploy Application and Redis:
   ```bash
   kubectl apply -f k8s/redis-deployment.yaml
   kubectl apply -f k8s/deployment.yaml
   ```

4. Set up auto-scaling:
   ```bash
   kubectl apply -f k8s/hpa.yaml
   ```

## Security Enhancements

### Rate Limiting with Throttler

To protect your API from abuse, rate limiting is implemented using the `@nestjs/throttler` package.

1. Install Throttler:
   ```bash
   pnpm add @nestjs/throttler
   ```

2. Configure in App Module:
   ```typescript
   @Module({
     imports: [
       ThrottlerModule.forRoot({
         ttl: 60,
         limit: 10,
       }),
       // other imports...
     ],
   })
   ```

### HTTPS Configuration

Ensure that your application is using HTTPS in production. Here's an example Nginx configuration for redirecting HTTP to HTTPS:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/ssl/certs/yourdomain.com.crt;
    ssl_certificate_key /etc/ssl/private/yourdomain.com.key;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Secret Management

Manage sensitive data using Kubernetes Secrets or AWS Secrets Manager for production environments. Example usage:

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { SecretManagerService } from '../../common/services/secret-manager.service';

@Injectable()
export class SomeService implements OnModuleInit {
  private shopifyApiKey: string;
  private shopifyApiSecret: string;

  constructor(private readonly secretManagerService: SecretManagerService) {}

  async onModuleInit() {
    this.shopifyApiKey = await this.secretManagerService.getSecret('SHOPIFY_API_KEY');
    this.shopifyApiSecret = await this.secretManagerService.getSecret('SHOPIFY_API_SECRET');
    // Use the secrets as needed
  }

  // ... other methods
}
```

## Monitoring and Logging

### Prometheus and Grafana for Monitoring

1. Install Prometheus:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install prometheus prometheus-community/prometheus
   ```

2. Install Grafana:
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm install grafana grafana/grafana
   ```

3. Expose Metrics:
   Ensure your NestJS app exposes metrics for Prometheus:
   ```typescript
   @Controller('/metrics')
   export class MetricsController {
     @Get()
     getMetrics() {
       return Prometheus.register.metrics();
     }
   }
   ```

### Centralized Logging

Use the ELK Stack (Elasticsearch, Logstash, Kibana) for centralized logging. Install it using Helm:

```bash
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
```

## Contributing

To contribute to this project:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature-branch`).
3. Make your changes and commit (`git commit -am 'Add new feature'`).
4. Push to the branch (`git push origin feature-branch`).
5. Open a Pull Request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

## Contact

For support, questions, or more information, contact the project maintainer at [email@example.com].

---

<p align="center">
  Made with ❤️ by [Your Name/Organization]
</p>
