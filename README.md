# 🚩 Real-Time Cloud Feature Flag Platform

A cloud-based feature flag platform for controlling application features in real time without requiring a new application deployment.

I built the platform using AWS, TypeScript, OpenFeature, DynamoDB, Kafka/MSK, Centrifugo, and OpenTofu. It includes a feature flag dashboard, real-time update pipeline, reusable TypeScript SDK, consumer application, and cloud infrastructure.

![CI](https://github.com/dj-berg/real-time-cloud-feature-flag-platform/actions/workflows/ci.yml/badge.svg)

---

## 🛠️ Technologies Used

- TypeScript
- JavaScript
- React
- Next.js
- Node.js
- AWS
- AWS Lambda
- DynamoDB & DynamoDB Streams
- Kafka / Amazon MSK
- Centrifugo
- WebSockets
- ECS / Fargate
- OpenFeature
- OpenTofu
- GitHub Actions

---

## 🚩 1. Build the Feature Flag Dashboard

I created a dashboard for managing application feature flags from one place.

The dashboard allows an operator to:

- Create feature flags
- Search and filter flags
- Enable and disable features
- Delete flags
- View application and environment information

For example, the `fraud-alerts` feature can be switched from disabled to enabled through the dashboard.

### Before — Feature Disabled

![Feature Flag Dashboard with Fraud Alerts Disabled](docs/images/dashboard-flag-disabled.png)

### After — Feature Enabled

![Feature Flag Dashboard with Fraud Alerts Enabled](docs/images/dashboard-flag-enabled.png)

The dashboard reflects the new state after the change is saved.

---

## 🗄️ 2. Store Feature Flags in DynamoDB

Feature flag information is stored in Amazon DynamoDB.

Each flag can include information such as:

- Flag key
- Enabled or disabled state
- Application
- Account
- Environment
- Description
- Schema version
- Creation information

![Feature Flag Records Stored in DynamoDB](docs/images/dynamodb-feature-flags.png)

This gives the platform a central source for the current state and configuration of each feature flag.

---

## ⚡ 3. Send Flag Changes Through the Real-Time Pipeline

When a feature flag changes, the update moves through an event-driven pipeline instead of requiring the consumer application to constantly check for changes.

The platform works like this:

```text
Feature Flag Dashboard
        │
        ▼
 Feature Flag API
        │
        ▼
     DynamoDB
        │
        ▼
 DynamoDB Streams
        │
        ▼
 Lambda Publisher
        │
        ▼
   Kafka / MSK
        │
        ▼
    Centrifugo
   ECS / Fargate
        │
        ▼
 WebSocket / WSS
        │
        ▼
Consumer Application
```

The API first saves the new flag state in DynamoDB.

DynamoDB Streams captures the change, a Lambda function publishes the event through Kafka/MSK, and Centrifugo delivers the update to connected applications through WebSockets.

This allows feature changes to reach applications in real time.

---

## 📡 4. Maintain a Real-Time Client Connection

The consumer application maintains a WebSocket connection through Centrifugo so it can receive feature flag updates.

![Active WebSocket Connection](docs/images/websocket-connection.png)

The browser network trace shows an HTTP **101 Switching Protocols** response, confirming that the WebSocket connection was successfully established.

Once connected, the application can receive new flag information without requiring a page refresh or new deployment.

---

## 📦 5. Create a Reusable TypeScript SDK

I created a TypeScript SDK so applications can work with the feature flag platform without directly implementing the underlying API and evaluation logic themselves.

The SDK includes:

- Feature flag evaluation
- TypeScript interfaces
- React integration
- Browser-compatible builds
- Consumer integration patterns
- OpenFeature provider integration

I also added an **OpenFeature provider**, allowing applications to evaluate flags through a standardized feature flag interface.

A Next.js integration example is included in:

```text
examples/next-openfeature/
```

Additional SDK and integration documentation is available in:

```text
consumer-sdk/README.md
docs/consumer-integration.md
```

---

## 💻 6. Connect a Consumer Application

I built a consumer application to demonstrate how an application can respond to feature flag changes.

With `fraud-alerts` disabled, the feature is unavailable:

![Consumer Application Before Feature Flag Change](docs/images/consumer-flag-disabled.png)

After the flag is enabled from the dashboard, the updated state reaches the consumer application and Fraud Alerts becomes available:

![Consumer Application After Feature Flag Change](docs/images/consumer-flag-enabled.png)

The complete flow is:

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

This demonstrates the main purpose of the platform: **application functionality can change through feature flag state without requiring a new application deployment.**

---

## ☁️ 7. Define the Cloud Infrastructure

I used **OpenTofu** to define the AWS infrastructure as code.

The infrastructure covers the cloud resources needed for areas such as:

- DynamoDB
- Lambda
- Kafka / Amazon MSK
- Centrifugo
- ECS / Fargate
- Networking and supporting infrastructure

Keeping infrastructure definitions in the repository makes the cloud architecture easier to track, validate, and reproduce.

---

## 🧪 8. Add Testing, CI, and Security Checks

I added automated validation to check the application, SDK, integrations, and infrastructure.

The main repository check can be run with:

```bash
npm run check
```

Validation includes:

- TypeScript SDK builds and type checking
- SDK tests
- Consumer integration checks
- Application build validation
- Next.js and OpenFeature integration validation
- OpenTofu formatting and configuration validation
- Dependency auditing
- Secret scanning

Additional checks include:

```bash
npm run lint
npm run format
npm run security
```

GitHub Actions runs automated repository checks, while Gitleaks and dependency auditing help identify accidentally committed secrets and vulnerable dependencies.

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

### Configure Environment Variables

Environment templates are included for the applications:

```bash
cp local-app/.env.example local-app/.env
cp consumer-app/.env.example consumer-app/.env
```

Add the required values for your environment before starting the applications.

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

Running the applications locally does not automatically deploy the AWS infrastructure. A live end-to-end demonstration requires reachable API and real-time services.

---

## 🧠 What I Learned

This project gave me experience building a system across frontend development, backend services, cloud infrastructure, and real-time communication.

I practiced:

- Designing cloud application architecture
- Building REST API functionality
- Working with AWS Lambda and DynamoDB
- Using DynamoDB Streams for event-driven processing
- Working with Kafka / Amazon MSK
- Delivering real-time updates through WebSockets
- Running services with ECS / Fargate
- Building a reusable TypeScript SDK
- Integrating OpenFeature
- Connecting React and Next.js applications to feature flags
- Defining AWS infrastructure with OpenTofu
- Building CI workflows with GitHub Actions
- Adding dependency and secret scanning
- Testing integrations across multiple parts of a system

The biggest takeaway was understanding how multiple services can work together to take one change and deliver it across an application in real time:

**Manage → Store → Publish → Deliver → Evaluate → Update**

---

## 📁 Repository Structure

```text
real-time-cloud-feature-flag-platform/
│
├── .github/
├── consumer-app/
├── consumer-sdk/
├── docs/
├── examples/
├── functions/
├── infra/
├── local-app/
├── scripts/
├── shared/
├── test-consumer/
├── tofu/
│
├── .gitignore
├── CONTRIBUTING.md
├── README.md
├── SECURITY.md
└── package.json
```

### Main Components

| Folder | Purpose |
|---|---|
| `consumer-app/` | Application demonstrating feature flag consumption |
| `consumer-sdk/` | TypeScript SDK and OpenFeature provider |
| `functions/` | API, authorization, and event-processing Lambda packages |
| `local-app/` | Feature Flag Control dashboard |
| `examples/` | Application integration examples |
| `test-consumer/` | Minimal SDK consumer |
| `tofu/` | Core AWS infrastructure definitions |
| `infra/` | Centrifugo, ECS/Fargate, and related infrastructure |
| `shared/` | Shared platform components |
| `scripts/` | Validation and utility scripts |
| `docs/` | Architecture and engineering documentation |

---

## 📚 Documentation

More detailed technical documentation is available throughout the repository:

- [Current Architecture](docs/current-architecture.md)
- [Consumer Integration](docs/consumer-integration.md)
- [SDK Documentation](consumer-sdk/README.md)
- [Next.js + OpenFeature Example](examples/next-openfeature/README.md)
- [Real-Time Reliability](docs/realtime-reliability-validation.md)
- [Production Readiness](docs/production-readiness.md)
- [Security](SECURITY.md)
- [Contributing](CONTRIBUTING.md)

---

## ⚠️ Project Context

This repository is a portfolio implementation of a real-time feature flag platform.

It demonstrates the architecture and engineering behind storing, distributing, evaluating, and consuming feature flags across applications. It is not presented as a production-certified hosted service.

Environment-specific credentials, private configuration, and sensitive deployment information are not included.
