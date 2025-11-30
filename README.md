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

## Quick Start (5 Minutes)

### 1. Use this template

Click **"Use this template"** → **"Create a new repository"** (or fork for contributions)

### 2. Configure secrets via CLI

```bash
cd your-repo
echo "YOUR_S3_ACCESS_KEY" | gh secret set S3_ACCESS_KEY_ID
echo "YOUR_S3_SECRET_KEY" | gh secret set S3_SECRET_ACCESS_KEY
echo "YOUR_DB_PASSWORD" | gh secret set POSTGRES_PASSWORD
```

**Alternative:** Set via web UI at **Settings → Secrets → Actions**

### 3. Run the restore

**CLI (Recommended):**
```bash
gh workflow run restore-from-s3.yml \
  -f s3_bucket="my-bucket" \
  -f s3_key="backups/db.dump" \
  -f s3_endpoint="https://account-id.r2.cloudflarestorage.com" \
  -f postgres_host="ep-xxx.neon.tech" \
  -f postgres_database="neondb" \
  -f postgres_user="neondb_owner" \
  -f parallel_jobs=4

# Watch progress
gh run watch
```

**Web UI:**
- **Actions** tab → **Restore PostgreSQL from S3/R2** → **Run workflow**
- Fill in parameters and click **Run**

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
