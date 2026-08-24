# 🚩 Real-Time Cloud Feature Flag Platform

An AWS-based feature flag platform for **managing, evaluating, and delivering application feature flags in real time**.

Built with **TypeScript, OpenFeature, AWS Lambda, DynamoDB, Kafka/MSK, Centrifugo, ECS/Fargate, and OpenTofu**, the platform demonstrates how application functionality can be controlled **independently from application deployments**.

![CI](https://github.com/dj-berg/real-time-cloud-feature-flag-platform/actions/workflows/ci.yml/badge.svg)

---

## 🎯 Project Overview

Modern applications often need to release, hide, or modify functionality without requiring a new deployment.

This project demonstrates an end-to-end feature flag platform where:

- 🚩 Operators manage feature flags through a centralized dashboard
- 🗄️ Flag state and metadata are persisted in DynamoDB
- ⚡ Changes propagate through an event-driven real-time pipeline
- 📦 Applications evaluate flags through a reusable TypeScript SDK
- 🔌 OpenFeature provides a standardized evaluation interface
- 🔐 Application, tenant, and environment scopes isolate flag access

The result is a system that connects **feature management, cloud infrastructure, real-time delivery, and application consumption** in one platform.

---

## 📸 Feature Flag Management

The **Feature Flag Control** dashboard provides a centralized interface for creating, searching, filtering, enabling, disabling, and deleting application feature flags.

The example below changes the `fraud-alerts` flag from **disabled** to **enabled**.

### Before — Flag Disabled

![Feature Flag Dashboard with Fraud Alerts Disabled](docs/images/dashboard-flag-disabled.png)

### After — Flag Enabled

![Feature Flag Dashboard with Fraud Alerts Enabled](docs/images/dashboard-flag-enabled.png)

The dashboard updates from **7 enabled / 1 disabled** to **8 enabled / 0 disabled**, reflecting the new persisted feature state.

---

## 🏗️ System Architecture

```text
                    ┌──────────────────────┐
                    │   Operator / Admin   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Feature Flag Control │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Feature Flag API   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      DynamoDB        │
                    └──────────┬───────────┘
                               │
                        DynamoDB Streams
                               │
                               ▼
                    ┌──────────────────────┐
                    │  Lambda Publisher    │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Kafka / MSK       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Centrifugo       │
                    │    ECS / Fargate     │
                    └──────────┬───────────┘
                               │
                          WebSocket / WSS
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Consumer Application │
                    └──────────────────────┘
```

When an operator changes a feature flag, the API persists the new state in **DynamoDB**. DynamoDB Streams captures the change, a **Lambda** publisher sends the event through **Kafka/MSK**, and **Centrifugo** delivers the update to connected applications over WebSockets.

The consumer can then evaluate the updated flag and change its behavior without requiring a new application deployment.

---

## ⚡ Feature Flags in Action

The consumer application demonstrates the result of a feature flag change from the application's perspective.

### Before — Feature Disabled

With `fraud-alerts` disabled, **Fraud Alerts is unavailable** in the consumer application.

![Consumer Application Before Feature Flag Change](docs/images/consumer-flag-disabled.png)

### After — Feature Enabled

After the operator enables the flag, **Fraud Alerts becomes available** to the consumer.

![Consumer Application After Feature Flag Change](docs/images/consumer-flag-enabled.png)

> **Application functionality can be controlled through feature flag state rather than requiring a new deployment.**

---

## 🔍 Verification

The demonstration includes visibility into the underlying storage and real-time communication layers.

### 🗄️ Persistent Feature Flag State

Feature flags are stored as structured records in **Amazon DynamoDB**.

![Feature Flag Records Stored in DynamoDB](docs/images/dynamodb-feature-flags.png)

Records contain information such as:

- Flag key
- Enabled/disabled state
- Application scope
- Account scope
- Environment
- Description
- Schema version
- Creation metadata

### 📡 Real-Time Client Connection

Connected applications maintain a persistent real-time connection through **Centrifugo**.

![Active WebSocket Connection](docs/images/websocket-connection.png)

The browser network trace shows a successful WebSocket protocol upgrade with HTTP status **101**, providing the persistent connection used for real-time flag delivery.

Together, the six project screenshots demonstrate the complete path from managing a feature flag to changing consumer application behavior:

```text
Manage Flag
    │
    ▼
Persist State
    │
    ▼
Publish Change
    │
    ▼
Deliver Update
    │
    ▼
Evaluate Flag
    │
    ▼
Change Application Behavior
```

---

## 🛠️ Technology Stack

| Area | Technologies |
| --- | --- |
| **Application** | Node.js, JavaScript, TypeScript, React, Next.js |
| **Feature Flags** | OpenFeature, Custom TypeScript SDK |
| **Cloud Platform** | AWS |
| **Compute** | AWS Lambda, ECS/Fargate |
| **Storage** | DynamoDB, DynamoDB Streams |
| **Messaging** | Kafka / Amazon MSK |
| **Real-Time Delivery** | Centrifugo, WebSockets / WSS |
| **Authentication** | JWT-based application and tenant scoping |
| **Infrastructure as Code** | OpenTofu |
| **Build Tooling** | npm, TypeScript, Babel |
| **CI/CD** | GitHub Actions |
| **Security** | Gitleaks, npm audit |
| **Version Control** | Git, GitHub |

---

## 📦 SDK & OpenFeature Integration

The platform includes a reusable **TypeScript SDK** so consuming applications do not need to directly implement the underlying feature flag API and evaluation behavior.

The SDK provides:

- Feature flag evaluation
- TypeScript interfaces
- React integration
- Browser-compatible builds
- Consumer integration patterns
- OpenFeature provider integration

The **OpenFeature provider** allows applications to evaluate flags through a standardized interface rather than coupling application code directly to platform-specific evaluation logic.

A Next.js integration example is included under:

[`examples/next-openfeature/`](examples/next-openfeature/)

For additional integration details:

- [`consumer-sdk/README.md`](consumer-sdk/README.md)
- [`docs/consumer-integration.md`](docs/consumer-integration.md)

---

## 📁 Project Structure

| Component | Purpose |
| --- | --- |
| `.github/` | GitHub Actions CI workflows |
| `functions/` | API, authorization, and stream-processing Lambda packages |
| `consumer-sdk/` | TypeScript SDK, React integration, and OpenFeature provider |
| `local-app/` | Operator-facing Feature Flag Control dashboard |
| `consumer-app/` | Application demonstrating feature flag consumption |
| `test-consumer/` | Minimal SDK consumer implementation |
| `examples/` | Application integration examples |
| `tofu/` | Core AWS infrastructure definitions |
| `infra/` | Centrifugo, ECS/Fargate, and related infrastructure |
| `scripts/` | Repository validation and utility scripts |
| `shared/` | Shared platform components |
| `docs/` | Detailed architecture and engineering documentation |

---

## 🧪 Testing, CI & Security

The repository includes credential-free validation for the application, SDK, integrations, and infrastructure.

Run the complete repository validation:

```bash
npm run check
```

Automated validation covers:

- ✅ TypeScript SDK builds and type checking
- ✅ SDK tests
- ✅ Consumer integration checks
- ✅ Local application build validation
- ✅ Next.js/OpenFeature integration validation
- ✅ OpenTofu formatting and configuration validation
- ✅ Dependency auditing
- ✅ Secret scanning

Additional checks are available through:

```bash
npm run lint
npm run format
npm run security
```

**GitHub Actions** automatically validates repository changes, while **Gitleaks** and dependency auditing help detect accidentally committed secrets and vulnerable dependencies.

Infrastructure validation does not plan, apply, mutate state, or deploy AWS resources.

---

## 🚀 Getting Started

### Prerequisites

Install:

- Node.js 20+
- npm
- OpenTofu
- Git

### Clone the Repository

```bash
git clone https://github.com/dj-berg/real-time-cloud-feature-flag-platform.git
cd real-time-cloud-feature-flag-platform
```

### Install Dependencies

```bash
npm run setup
```

The setup process installs the required project dependencies and does not deploy AWS resources.

### Configure Environment Variables

Environment templates are provided for the applications.

```bash
cp local-app/.env.example local-app/.env
cp consumer-app/.env.example consumer-app/.env
```

Populate the required values for your environment before starting the applications.

> Never commit `.env` files, credentials, private keys, state files, or environment-specific secrets.

### Run the Feature Flag Dashboard

```bash
npm run dev:local
```

Default address:

```text
http://localhost:3000
```

### Run the Consumer Application

```bash
npm run dev:consumer
```

Default address:

```text
http://localhost:3001
```

### Run the Minimal SDK Consumer

```bash
npm run dev:test-consumer
```

Default address:

```text
http://localhost:3002
```

### Validate the Repository

```bash
npm run check
```

> Running the applications locally does not itself deploy the underlying AWS infrastructure. Live demonstrations require reachable API and real-time services.

---

## 📚 Technical Documentation

Detailed engineering and operational documentation is available throughout the repository.

- [Current Architecture](docs/current-architecture.md) — System architecture and runtime contracts
- [Consumer Integration](docs/consumer-integration.md) — SDK, authentication, API usage, and real-time behavior
- [SDK Documentation](consumer-sdk/README.md) — TypeScript SDK and integration guidance
- [Next.js + OpenFeature](examples/next-openfeature/README.md) — OpenFeature integration example
- [Real-Time Reliability](docs/realtime-reliability-validation.md) — Real-time event pipeline validation
- [Production Readiness](docs/production-readiness.md) — Security, monitoring, rollback, and operational considerations
- [Publication Checklist](docs/publication-checklist.md) — Standalone portfolio publication checks
- [Security](SECURITY.md) — Repository security guidance
- [Contributing](CONTRIBUTING.md) — Contribution guidelines

---

## 💡 Engineering Highlights

This project demonstrates experience with:

- Cloud application architecture
- Event-driven systems
- Feature flag platform design
- REST API development
- TypeScript SDK development
- OpenFeature integration
- Tenant-scoped authentication
- Real-time messaging
- WebSocket communication
- React and Next.js integration
- DynamoDB data modeling
- AWS Lambda
- Kafka / Amazon MSK
- ECS/Fargate
- Infrastructure as Code
- OpenTofu
- CI/CD
- Automated security scanning
- Technical documentation

The central engineering flow demonstrated by the project is:

```text
Manage a Feature
       │
       ▼
Persist its State
       │
       ▼
Propagate its Change
       │
       ▼
Evaluate the Flag
       │
       ▼
Change Consumer Behavior
```

Rather than representing only a management interface, the repository demonstrates the surrounding platform required to **store, distribute, evaluate, and consume feature flags across applications**.

---

## ⚠️ Project Context

This repository is a **standalone portfolio implementation** demonstrating real-time feature flag platform architecture and modern software engineering practices.

It is intended as an architectural and engineering demonstration rather than a production-certified hosted service.

Environment-specific credentials, private configuration, proprietary infrastructure, and other sensitive deployment artifacts are not included.

---

## 📄 License

This project is provided as a portfolio and educational software engineering demonstration.
