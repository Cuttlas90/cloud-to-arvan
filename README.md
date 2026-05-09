# Remote Downloader to S3 / Arvan Object Storage

> 📖 [فارسی](README_FA.md) | English

Download large files on GitHub Actions and automatically transfer them to S3-compatible Object Storage (like ArvanCloud).

---

# Why Was This Project Built?

In some countries or network conditions, direct downloading of large files from the international internet:
- Is very slow
- Is unstable
- Requires VPN/VPS
- Or has high costs

This project leverages GitHub Actions to download files on GitHub's infrastructure and transfer them to the user's personal Object Storage.

---

# Project Architecture

```text
Internet
   ↓
GitHub Actions
   ↓
Chunked Upload
   ↓
S3-Compatible Object Storage
(Arvan / MinIO / ...)
```

---

# Features

- Download files on GitHub Actions
- Supports direct links
- Upload to S3-compatible Object Storage
- Split files into small chunks
- Resume capability
- Prevents re-uploading already uploaded chunks
- Resilient to GitHub Actions' 6-hour timeout
- Suitable for very large files
- No VPS required

---

# Why Chunk Files?

GitHub Actions has a time limit and usually stops the job after about 6 hours.

If the file is uploaded as a single piece:
- If the workflow is interrupted
- The entire upload is lost

But in this project:

```text
8GB file
↓
16 × 512MB chunks
```

Each chunk is uploaded separately.

On subsequent runs:
- Previous chunks are detected
- Only remaining chunks are uploaded

So the workflow is fully resumable.

---

# File Structure Inside Bucket

Example:

```text
needed_file.iso/
  needed_file.iso.part-00000
  needed_file.iso.part-00001
  needed_file.iso.part-00002
  needed_file.iso.sha256
```

---

# How to Assemble the File

After downloading all chunks:

```bash
cat needed_file.iso.part-* > needed_file.iso
```

Then to verify file integrity:

```bash
sha256sum -c needed_file.iso.sha256
```

If the hash is correct:
- The file has been successfully reconstructed.

---

# Setup

## 1. Fork Repository

First, fork the repository.

---

## 2. Create Object Storage

This project works with any S3-compatible storage:

- Arvan Object Storage
- MinIO
- AWS S3
- Backblaze B2
- Cloudflare R2

---

## 3. Create GitHub Secrets

Path:

```text
Repository → Settings → Secrets and variables → Actions
```

Required Secrets:

```text
ARVAN_ACCESS_KEY
ARVAN_SECRET_KEY
ARVAN_BUCKET
ARVAN_ENDPOINT
```

---

# Running a Download

Path:

```text
Actions → Download and Upload to Arvan → Run workflow
```

Parameters:

| Parameter | Description |
|---|---|
| url | File URL |
| filename | Output file name |
| chunk_size | Size of each chunk (e.g., 512M) |

---

# Resume Upload

If the workflow:
- Stops
- Times out
- Fails

Just run the same workflow again with the same filename.

Previous chunks:
- Will not be re-uploaded
- Are detected and skipped

---

# Limitations

- GitHub Actions is designed for personal and reasonable use.
- This project should not be used for abuse, heavy scraping, or massive public mirroring.
- Upload to infrastructures inside Iran may be slow.
- Cloudflare and some providers may not be accessible through certain ISPs.

## Responsible Use

This project is designed for:
- Lawful downloading of files you have permission to access
- Transferring files to the user's personal storage
- Personal, research, educational, and archival use

---

## Important Notes

- This project is not a public relay service.
- Each user must use their own storage and GitHub account.
- Heavy, abusive, or automated mass scraping may result in GitHub Actions restrictions.
- The user is responsible for their usage of this project.
- The author is not responsible for illegal use or violation of provider policies.

---

## Using GitHub Actions

GitHub Actions is designed for CI/CD.

This project strives to follow:
- Conservative usage
- Resumable uploads
- Chunked transfers
- Prevention of unreasonable resource consumption

In case of heavy or unusual usage:
- GitHub may restrict workflows
- Or rate-limit the account

# GitHub Timeout

GitHub-hosted runners typically have a timeout of about 6 hours.

This project works around this limitation by using:
- Chunked uploads
- Resumable uploads
- Part detection

---

# Security

Never:
- Make your Access Key public.
- Place secrets inside the repository.

Use GitHub Secrets instead.

---

# Core Idea of the Project

This project aims to:
- Without VPS
- Without a permanent server
- Without a public relay
- Without violating provider policies

Provide users with the ability to transfer large files.

---

# License

MIT
