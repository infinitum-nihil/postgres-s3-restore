# PostgreSQL S3/R2 Restore via GitHub Actions

Fast PostgreSQL database restoration from S3-compatible storage (AWS S3, Cloudflare R2, DigitalOcean Spaces, etc.) to any PostgreSQL instance (Neon, Vultr, AWS RDS, self-hosted).

## Why This Exists

Restoring large PostgreSQL dumps from object storage to cloud databases is slow when done from local machines with limited upload bandwidth. GitHub Actions provides:

- ✅ **Fast network** to both S3 and target database
- ✅ **Free compute** (2000 minutes/month)
- ✅ **Pre-installed tools** (postgresql-client, awscli)
- ✅ **No local infrastructure** required

## Use Cases

- **Neon migrations:** Restore dumps from R2/S3 to Neon managed PostgreSQL
- **Vultr VM setup:** Bootstrap new Vultr PostgreSQL VMs from backups
- **Disaster recovery:** Restore from S3 backups to any PostgreSQL instance
- **Database cloning:** Copy production to staging via S3 intermediary

## Setup

### 1. Fork or use this repository

### 2. Configure GitHub Secrets

Go to repository **Settings → Secrets and variables → Actions** and add:

- `S3_ACCESS_KEY_ID` - Your S3/R2 access key
- `S3_SECRET_ACCESS_KEY` - Your S3/R2 secret key
- `POSTGRES_PASSWORD` - Target database password

### 3. Trigger restore workflow

**GitHub UI:**
- Go to **Actions** tab
- Select **"Restore PostgreSQL from S3/R2"** workflow
- Click **"Run workflow"**
- Fill in parameters:
  - S3 bucket name
  - Object key (path to dump file)
  - S3 endpoint (for R2: `https://<account-id>.r2.cloudflarestorage.com`)
  - PostgreSQL host, database, username
  - Parallel jobs (1-8, default 4)

**CLI (gh):**
```bash
gh workflow run restore-from-s3.yml \
  -f s3_bucket="my-bucket" \
  -f s3_key="backups/db.dump" \
  -f s3_endpoint="https://account-id.r2.cloudflarestorage.com" \
  -f postgres_host="ep-xxx.neon.tech" \
  -f postgres_database="neondb" \
  -f postgres_user="neondb_owner" \
  -f parallel_jobs=4
```

## Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| `s3_bucket` | Yes | S3/R2 bucket name | `themis-backups` |
| `s3_key` | Yes | Path to dump file | `backups/postgres/db.dump` |
| `s3_endpoint` | No | Endpoint URL (leave empty for AWS S3) | `https://xxx.r2.cloudflarestorage.com` |
| `postgres_host` | Yes | PostgreSQL hostname | `ep-xxx.neon.tech` or `123.45.67.89` |
| `postgres_database` | Yes | Database name | `neondb` |
| `postgres_user` | Yes | Username | `neondb_owner` |
| `parallel_jobs` | No | Restore parallelism (default 4) | `1` to `8` |

## Supported Storage Providers

- ✅ **Cloudflare R2** (use custom endpoint)
- ✅ **AWS S3** (leave endpoint empty)
- ✅ **DigitalOcean Spaces** (use custom endpoint)
- ✅ **Backblaze B2** (use custom endpoint)
- ✅ Any S3-compatible storage

## Supported PostgreSQL Targets

- ✅ **Neon** managed PostgreSQL
- ✅ **Vultr** managed or self-hosted VMs
- ✅ **AWS RDS / Aurora**
- ✅ **DigitalOcean Managed Databases**
- ✅ **Self-hosted** PostgreSQL (publicly accessible)

## Performance

**Example:** 11 GB dump from Cloudflare R2 to Neon
- Download: ~2-3 minutes (GitHub → R2 fast)
- Restore: ~45-60 minutes (parallel jobs)
- **Total: ~1 hour** vs 6+ hours from local machine with 10 Mbps upload

## Workflow Artifacts

Restore logs are saved as artifacts for 7 days:
- Download from **Actions → [workflow run] → Artifacts**
- Contains full pg_restore output for debugging

## Security Notes

- Secrets are never logged or exposed in workflow output
- Dump file is downloaded to ephemeral runner (deleted after job)
- Use GitHub encrypted secrets for all credentials
- Workflow only accessible to repo collaborators (unless public repo)

## License

MIT - Free to use, modify, and distribute

## Credits

Created by [infinitum nihil](https://github.com/infinitum-nihil) as a public service after learning the hard way.
