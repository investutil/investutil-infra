# InvestUtil Infrastructure

This repository contains all infrastructure and deployment configurations for the InvestUtil project.

## Project Structure

```
.
├── docker/
│   └── backend/
│       ├── Dockerfile
│       ├── docker-compose.yml
│       └── .dockerignore
└── k8s/
    └── backend/
        ├── deployment.yaml
        ├── service.yaml
        ├── configmap.yaml
        ├── secret.yaml.example
        └── postgres.yaml
```

## Docker Deployment

### Using Docker Compose

1. Create a production environment file:
   ```bash
   cp .env.production.example .env.production
   ```

2. Edit the `.env.production` file with your secure credentials:
   ```env
   POSTGRES_USER=your_db_user
   POSTGRES_PASSWORD=your_secure_password
   POSTGRES_DB=investutil
   DATABASE_URL=postgres://your_db_user:your_secure_password@db:5432/investutil
   JWT_SECRET=your_secure_jwt_secret
   ```

3. Start the services:
   ```bash
   docker-compose --env-file .env.production up -d
   ```

4. Run database migrations:
   ```bash
   docker-compose exec app ./investutil-back migrate
   ```

### Manual Docker Build

1. Build the image:
   ```bash
   docker build -t investutil-back .
   ```

2. Run the container:
   ```bash
   docker run -d \
     --name investutil-back \
     -p 8080:8080 \
     -e DATABASE_URL="postgres://user:password@host:5432/investutil" \
     -e JWT_SECRET="your_secure_jwt_secret" \
     investutil-back
   ```

## Kubernetes Deployment

### Prerequisites

- Kubernetes cluster (local or cloud)
- kubectl configured
- helm (optional, for package management)

### Deployment Steps

1. Create namespace (optional):
   ```bash
   kubectl create namespace investutil
   kubectl config set-context --current --namespace=investutil
   ```

2. Create Kubernetes secrets:
   ```bash
   # Copy the example secret file
   cp k8s/backend/secret.yaml.example k8s/backend/secret.yaml
   
   # Edit the secrets with your values
   nano k8s/backend/secret.yaml
   
   # Apply the secret
   kubectl apply -f k8s/backend/secret.yaml
   ```

3. Apply ConfigMap:
   ```bash
   kubectl apply -f k8s/backend/configmap.yaml
   ```

4. Deploy PostgreSQL:
   ```bash
   kubectl apply -f k8s/backend/postgres.yaml
   ```

5. Deploy the application:
   ```bash
   kubectl apply -f k8s/backend/deployment.yaml
   kubectl apply -f k8s/backend/service.yaml
   ```

### Resource Requirements

- Application:
  - CPU: 100m-500m
  - Memory: 128Mi-512Mi
- PostgreSQL:
  - Storage: 10Gi

### Health Checks

The application includes:
- Readiness probe: `/health` endpoint
- Liveness probe: `/health` endpoint

### Scaling

The application is configured with 2 replicas by default. To scale:

```bash
kubectl scale deployment investutil-back --replicas=3
```

### Monitoring and Logs

View application logs:
```bash
kubectl logs -l app=investutil-back
```

View PostgreSQL logs:
```bash
kubectl logs -l app=postgres
```

### Troubleshooting

1. Check pod status:
   ```bash
   kubectl get pods
   ```

2. Describe pod for details:
   ```bash
   kubectl describe pod <pod-name>
   ```

3. Check application logs:
   ```bash
   kubectl logs <pod-name>
   ```

## Security Considerations

1. Never commit sensitive environment variables to version control
2. Use secure passwords and JWT secrets in production
3. Consider using a secrets management service in production
4. Regularly update dependencies and base images
5. Use non-root user in container (already configured)
