# Cloud AI Agent Control & Lifestyle Management Platform
FastAPI backend + vanilla-JS frontend (served by FastAPI) + database layer that uses **SQLite locally** or **Azure Cosmos DB** when `COSMOS_ENDPOINT` is set. Every record carries `company_id` (Cosmos partition key `/company_id`); the company comes from the signed token, never the request body.

## Run in VS Code (Windows: use `py`/`.venv\Scripts\activate`)
1. File > Open Folder > this folder. Install the Python extension.
2. Terminal:
       cd backend
       python -m venv .venv && .venv\Scripts\activate      (macOS/Linux: source .venv/bin/activate)
       pip install -r requirements.txt
       copy .env.example .env                               (macOS/Linux: cp)
3. Press F5 (Run API) or `uvicorn main:app --reload`, open http://localhost:8000.
4. Log in: demo@companya.test / Demo@12345 (or demo@companyb.test), then Agent Control > start > run task. The Cloud card shows "Local SQLite" until Azure is set up.
`python smoke_test.py` runs the workflow tests (tenant isolation, rules, billing, controls) with no extra packages.

## Connect Azure
**Cosmos DB:** Azure Portal > Create > Azure Cosmos DB for NoSQL (Serverless is cheapest). Keys > copy URI and Primary Key into `.env` as `COSMOS_ENDPOINT` / `COSMOS_KEY`. Restart; the app creates database `aiagents` and 8 containers (companies, users, agents, executions, rules, billing, logs, metrics) itself. The dashboard Cloud card changes to "Azure Cosmos DB". Without a key, it uses `DefaultAzureCredential` (needs the Cosmos data-plane RBAC role).
**Key Vault:** create a vault, add secrets `app-secret` and `cosmos-key`, give your identity "Key Vault Secrets User", set `KEY_VAULT_URL`; values load at start-up.
**App Insights:** create a resource and set `APPLICATIONINSIGHTS_CONNECTION_STRING`.
**App Service deploy:** `cd backend && az webapp up --name <unique-name> --runtime PYTHON:3.12 --sku B1`; set the same variables under Settings > Environment variables; startup command `uvicorn main:app --host 0.0.0.0 --port 8000` (the frontend folder must be deployed beside `backend`, so deploy from the project root with `az webapp up` after moving `requirements.txt` there, or zip-deploy the whole folder). Use ONE instance (agent workers are in-process threads).

## Status: what is and is not verified
- Tested here: the whole backend workflow on SQLite, and the Cosmos adapter against an in-memory fake container. **Not tested against real Azure, a running FastAPI server, or the Azure deployment steps.** Expect to fix small things when you first connect.
- Not implemented: Microsoft Entra External ID sign-in (local email+password with PBKDF2 is used), React, WebSockets (3 s polling), PDF export, roles other than admin. Billing is simulated (no payments). Set a strong `APP_SECRET` before any deployment.
- Real model: set an agent's API config to `{"endpoint":"https://...","key_env":"NAME_OF_ENV_VAR"}`; the key stays in that server env var. The request body is generic; adapt `core.execute` to your provider.
