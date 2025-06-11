# SpawnIt

SpawnIt is an Express-based orchestration tool that automatically deploys services on different providers (AWS via Terraform/OpenTofu or locally via Docker containers). It uses an S3 bucket for state and configuration storage, continuous planning loops for drift detection, and Server-Sent Events (SSE) for real-time log streaming.

> **⚠️ WARNING** This project is **not production ready**. It stores credentials in plain text, lacks authentication/authorization, has minimal error handling, and may expose sensitive data. Use at your own risk and consider security (secret management, encryption, secure networks, strong passwords) before any production deployment.



## Repository Structure

SpawnIt is divided into three repositories that run in concert to deliver the full experience:

### 1. Backend (`backend`)

Houses the Express API, orchestration services (catalog, templates, OpenTofu/Docker executor), SSE log streaming, and backend CI (builds & publishes the Docker image). All core logic lives here.
**Manual Startup:**

```bash
git clone https://github.com/your-org/backend.git
cd backend
npm install
export AWS_ACCESS_KEY_ID="..." AWS_SECRET_ACCESS_KEY="..." AWS_REGION="..."
node index.js
```

*(Listens on port 8000, initializes plan loops, exposes* `*/api/**` *endpoints.)*

### 2. Frontend (`frontend`)

Hosts the React/Next.js dashboard UI that lets users browse services, configure deployments, and stream logs in real time.
**Manual Startup:**

```bash
git clone https://github.com/your-org/frontend.git
cd frontend
npm install
npm run dev
```

*(Runs on* `*http://localhost:3000*`*, communicates with the backend API.)*

**Docker Build & Run:**

```
docker build -t spawnit-frontend:latest .
docker run -d -p 3000:3000 --env API_URL=http://localhost:8000 spawnit-frontend:latest
```



### 3. Infrastructure (`infra`)

Contains Terraform/OpenTofu definitions for networks, S3 state bucket, and Docker instances (MinIO, backend, frontend, Keycloak, etc.), plus automation scripts. A single `all-deploy.sh` script ties everything together for a full local deployment.
**Automated Full Deployment:**

```bash
git clone https://github.com/your-org/infra.git
cd infra
chmod +x all-deploy.sh
./all-deploy.sh apply
```



## Prerequisites

Before you can deploy SpawnIt, ensure you have the following installed on your machine:

1. **Node.js** (v16 or newer) & npm
2. **Docker** (if you intend to run local container services)
3. **Terraform** or **OpenTofu CLI**
   - Install via `brew install hashicorp/tap/terraform` or download from the [OpenTofu releases page](https://github.com/opentofu/opentofu/releases).
4. **AWS CLI**
   - Install via `brew install awscli` or from the [AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
5. **An AWS S3 bucket** named `spawn-it-bucket` (or another name of your choice).
6. **Git**

## Installation

1. **Clone the repositories**

2. **Install dependencies**

   ```
   npm install
   ```

## Configuration

### Environment Variables

SpawnIt reads AWS credentials and region from the environment. Export them in your shell:

```bash
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_ACCESS_KEY"
export AWS_REGION="us-east-1"
```

Alternatively, you can create a `.env` file:

```
AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY
AWS_REGION=us-east-1
```

> **Note:** Do **not** commit `.env` or any credential files to version control.

### S3 Bucket Setup

1. Create the S3 bucket:

   ```
   aws s3 mb s3://spawn-it-bucket --region $AWS_REGION
   ```

2. (Optional) Configure bucket policies/permissions as needed.

## Running Locally

SpawnIt uses the OpenTofu CLI under the hood. To start the server locally, you can launch the provided `all-deploy.sh` script in the infra repository:

```
./scripts/all-deploy.sh
```

The server will:

- Listen on port `8000` by default
- Initialize any existing plan loops for configured clients/services
- Serve API endpoints under `/api/*`

## Docker Deployment

You can build and run SpawnIt in a container:

```
# Build the Docker image
docker build -t spawnit:latest .

# Run the container, mapping port 8000 and passing env vars
docker run -d \
  -p 8000:8000 \
  --env AWS_ACCESS_KEY_ID \
  --env AWS_SECRET_ACCESS_KEY \
  --env AWS_REGION \
  spawnit:latest
```

## API Reference

### Catalog

- **GET** `/api/catalog`
  Returns the list of available service templates from `data/catalog.json`.

### Template

- **GET** `/api/template/:templateName`
  Fetches the Terraform/OpenTofu template archive (zip) by name.

### OpenTofu / Services

- **POST** `/api/clients/:clientId/services/:serviceName/plan`
- **POST** `/api/clients/:clientId/services/:serviceName/apply`
- **POST** `/api/clients/:clientId/services/:serviceName/destroy`
- **GET** `/api/clients/:clientId/services/:serviceName/status`
- **SSE** `/api/clients/:clientId/services/:serviceName/logs`

### Networks

- **POST** `/api/clients/:clientId/network/:provider/apply`
- **POST** `/api/clients/:clientId/network/:provider/destroy`
- **GET** `/api/clients/:clientId/network/:provider/status`

Refer to the code in `routes/` for full details on request bodies and responses.

## Third-Party Software

- **Node.js & npm** – JavaScript runtime and package manager
- **Docker Engine** – For local container-based services
- **Terraform / OpenTofu CLI** – Infrastructure-as-Code tool
- **AWS CLI** – AWS credentials management and S3 interactions
- **GitHub Actions** – CI workflow provided (`.github/workflows/publish.yaml`)

## Security & Warning

> **⚠️ This project is** ***not*** **production ready.**
>
> - **Credentials in plaintext:** AWS keys and secrets are stored in environment variables and S3 without encryption.
> - **No authentication:** The API is unauthenticated; anyone with network access can trigger deployments.
> - **Minimal input validation:** Risk of malicious payloads or misconfigurations.
> - **Sensitive data exposure:** Service configs, state files, and logs are publicly accessible if S3 permissions are misconfigured.
>
> **Before any real-world use**, implement:
>
> - Secret management (Vault, AWS Secrets Manager)
> - HTTPS/TLS for API endpoints
> - Authentication & authorization
> - Cross-account IAM roles and least-privilege policies
> - Input validation and sanitization
> - Encryption at rest & in transit
> - Proper network segmentation (VPCs, subnets, security groups)

## License

This project is licensed under the MIT License.
