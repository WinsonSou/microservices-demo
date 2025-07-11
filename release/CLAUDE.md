# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Google Online Boutique microservices demo - a cloud-native application consisting of 11 microservices that communicate via gRPC. It demonstrates Kubernetes, service mesh, and Google Cloud technologies.

## Architecture

The application consists of these services:
- **Frontend** (Go) - Web UI server
- **CartService** (C#) - Shopping cart management (Redis)
- **ProductCatalogService** (Go) - Product listings
- **CurrencyService** (Node.js) - Currency conversion
- **PaymentService** (Node.js) - Payment processing
- **ShippingService** (Go) - Shipping calculations
- **EmailService** (Python) - Order confirmations
- **CheckoutService** (Go) - Checkout orchestration
- **RecommendationService** (Python) - Product recommendations
- **AdService** (Java) - Contextual ads
- **LoadGenerator** (Python/Locust) - Traffic simulation

All services communicate via gRPC with contracts defined in `/protos/demo.proto`.

## Essential Commands

### Development & Deployment
```bash
# Primary development tool
skaffold dev              # Continuous development mode (hot reload)
skaffold run              # Build and deploy once
skaffold delete           # Clean up resources
skaffold debug            # Debug mode with debugger support

# Quick deployment
kubectl apply -f ./release/kubernetes-manifests.yaml

# Local development setup
kubectl port-forward deployment/frontend 8080:8080
```

### Building Services

**Go services** (frontend, productcatalogservice, shippingservice, checkoutservice):
```bash
cd src/[service-name]
go test                   # Run tests
go build                  # Build binary
```

**Java service** (adservice):
```bash
cd src/adservice
./gradlew build           # Build service
./gradlew test            # Run tests
```

**C# service** (cartservice):
```bash
cd src/cartservice
dotnet build              # Build service
dotnet test               # Run tests
```

**Node.js services** (currencyservice, paymentservice):
```bash
cd src/[service-name]
npm install               # Install dependencies
npm test                  # Run tests (if implemented)
```

**Python services** (emailservice, recommendationservice, loadgenerator):
```bash
cd src/[service-name]
pip install -r requirements.txt  # Install dependencies
```

### Testing

Run all Go service tests:
```bash
for SERVICE in "shippingservice" "productcatalogservice" "frontend/validator"; do
  pushd src/$SERVICE && go test && popd
done
```

Run C# tests:
```bash
dotnet test src/cartservice/
```

## Project Structure

- `/src/` - Service source code
- `/kubernetes-manifests/` - Base Kubernetes configs
- `/release/` - Pre-built deployment manifests
- `/helm-chart/` - Helm chart configuration
- `/terraform/` - Infrastructure as Code
- `/kustomize/` - Deployment variations (Istio, Spanner, etc.)
- `/protos/` - gRPC service definitions
- `/docs/` - Documentation

## Key Development Patterns

1. **Service Communication**: All inter-service communication uses gRPC. Service definitions are in `/protos/demo.proto`.

2. **Configuration**: Services use environment variables for configuration. See deployment manifests for available options.

3. **Error Handling**: Services should handle gRPC errors gracefully and provide meaningful error messages.

4. **Logging**: Services use structured logging. Frontend uses Go's log package, other services use language-specific logging.

5. **Health Checks**: All services implement gRPC health checking protocol.

## Common Tasks

### Adding a new feature to a service:
1. Update the proto definition if needed (`/protos/demo.proto`)
2. Regenerate proto code if changed
3. Implement the feature in the service
4. Update tests
5. Test locally with `skaffold dev`

### Debugging a service:
1. Use `skaffold debug` for debugger support
2. Check logs: `kubectl logs deployment/[service-name]`
3. Port forward for direct access: `kubectl port-forward deployment/[service-name] [port]:[port]`

### Updating dependencies:
- Go: Update `go.mod` and run `go mod tidy`
- Java: Update `build.gradle`
- C#: Update `.csproj` file
- Node.js: Update `package.json` and run `npm install`
- Python: Update `requirements.txt`

## Important Notes

- The project uses Skaffold as the primary development tool - avoid manual Docker builds unless necessary
- All services run as non-root users in containers for security
- Redis is used only by CartService for session storage
- The LoadGenerator creates realistic traffic patterns for testing
- Services are designed to be stateless except for CartService (Redis state)