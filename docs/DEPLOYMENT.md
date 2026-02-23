# Deployment Strategy

Although the project was paused before production rollout, the system architecture is designed for high availability and easy deployment.

## Infrastructure Plan
1. **Application Logic (n8n):**
   - *Current (Staging):* Railway App (Cloud).
   - *Production Target:* Self-hosted VPS using Docker Compose (see [/docker/docker-compose.yml](/docker/docker-compose.yml)) to ensure data residency compliance and cost control.
2. **Database & Vector Store:**
   - *Current:* Google Sheets (Operational CRM) + Supabase (pgvector for RAG).
   - *Production Target:* Migrate Google Sheets state management entirely to Postgres to avoid Google API rate limits (`HTTP 429`) under heavy broadcast loads.
3. **Webhook Provider:**
   - ChatApp configured with IP Whitelisting targeting the n8n VPS static IP.

## Secrets Management
- No secrets are committed to the repository.
- A `.env.example` file is provided. In production, variables are injected via Docker environment files or Railway Secrets Manager.

## Backup Strategy
- **n8n Workflows:** Nightly automated exports to a secure S3 bucket.
- **Database:** Supabase automated daily backups (PITR enabled).
