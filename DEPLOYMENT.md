# Deployment Guide - Mountain Sickness Risk Analysis

Complete step-by-step guide for deploying your app to Google Cloud Platform via GitHub.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Initial Setup](#initial-setup)
3. [GitHub Repository Setup](#github-repository-setup)
4. [Google Cloud Setup](#google-cloud-setup)
5. [Automated Deployment Configuration](#automated-deployment-configuration)
6. [First Deployment](#first-deployment)
7. [Continuous Deployment](#continuous-deployment)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Accounts
- ✅ Google Cloud Platform account ([sign up](https://cloud.google.com/))
- ✅ GitHub account ([sign up](https://github.com/))
- ✅ Valid credit card for GCP (free tier available)

### Required Software
```bash
# Install Google Cloud SDK
# macOS (using Homebrew)
brew install --cask google-cloud-sdk

# Windows
# Download from: https://cloud.google.com/sdk/docs/install

# Linux
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Verify installation
gcloud --version
```

### Install Git (if not already installed)
```bash
# macOS
brew install git

# Windows
# Download from: https://git-scm.com/download/win

# Linux (Ubuntu/Debian)
sudo apt-get install git
```

---

## Initial Setup

### 1. Create Project Directory
```bash
# Create a new directory
mkdir mountain-sickness-analyzer
cd mountain-sickness-analyzer

# Copy your index.html file into this directory
# (Your existing HTML file with all the code)
```

### 2. Initialize Git Repository
```bash
# Initialize git
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit: Mountain Sickness Risk Analysis app"
```

---

## GitHub Repository Setup

### 1. Create GitHub Repository

1. Go to [GitHub](https://github.com/new)
2. Repository name: `mountain-sickness-analyzer`
3. Description: "Altitude sickness risk assessment tool"
4. Choose: Public or Private
5. **Do NOT** initialize with README (we already have files)
6. Click "Create repository"

### 2. Push Code to GitHub

```bash
# Add GitHub as remote origin
git remote add origin https://github.com/YOUR-USERNAME/mountain-sickness-analyzer.git

# Push to GitHub
git branch -M main
git push -u origin main
```

Verify: Visit your GitHub repository URL to see your files.

---

## Google Cloud Setup

### 1. Create GCP Project

```bash
# Authenticate with Google Cloud
gcloud auth login

# Create new project (choose a unique PROJECT-ID)
gcloud projects create YOUR-PROJECT-ID --name="Mountain Sickness Analyzer"

# Set as active project
gcloud config set project YOUR-PROJECT-ID

# Verify
gcloud config list project
```

**Note**: PROJECT-ID must be globally unique. Try: `mountain-sickness-YOUR-NAME`

### 2. Enable Billing

1. Go to [GCP Console](https://console.cloud.google.com/)
2. Select your project
3. Go to "Billing" in left menu
4. Link a billing account

**Free Tier Benefits**:
- 28 instance hours per day
- 1 GB data transfer per day
- Usually covers small apps entirely

### 3. Enable Required APIs

```bash
# Enable App Engine Admin API
gcloud services enable appengine.googleapis.com

# Enable Cloud Build API
gcloud services enable cloudbuild.googleapis.com

# Verify APIs are enabled
gcloud services list --enabled
```

### 4. Create App Engine Application

```bash
# Create App Engine app (choose region close to your users)
gcloud app create --region=us-central

# Other region options:
# us-east1, us-west2, europe-west1, asia-northeast1
# Full list: gcloud app regions list
```

---

## Automated Deployment Configuration

### 1. Grant Cloud Build Permissions

```bash
# Get project number
PROJECT_NUMBER=$(gcloud projects describe YOUR-PROJECT-ID --format="value(projectNumber)")

# Grant App Engine Deployer role
gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
  --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
  --role=roles/appengine.deployer

# Grant Service Account User role
gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
  --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
  --role=roles/iam.serviceAccountUser

# Grant App Engine Service Admin role
gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
  --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
  --role=roles/appengine.serviceAdmin

# Grant Cloud Build Service Account role
gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
  --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
  --role=roles/cloudbuild.builds.builder
```

### 2. Connect GitHub to Cloud Build

#### Option A: Using GCP Console (Recommended for first time)

1. Go to [Cloud Build Triggers](https://console.cloud.google.com/cloud-build/triggers)
2. Click "**Connect Repository**"
3. Select source: "**GitHub (Cloud Build GitHub App)**"
4. Click "**Authenticate**" and sign in to GitHub
5. Click "**Install Google Cloud Build**" on GitHub
6. Select your repository
7. Click "**Connect**"

#### Option B: Using Command Line

```bash
# Install GitHub app
gcloud alpha builds connections create github YOUR-CONNECTION-NAME \
  --region=us-central1

# Follow the prompts to authenticate with GitHub
```

### 3. Create Build Trigger

1. After connecting repository, click "**Create Trigger**"
2. Configure:
   - **Name**: `deploy-main-branch`
   - **Description**: "Deploy to App Engine on push to main"
   - **Event**: Push to a branch
   - **Source**: Select your repository
   - **Branch**: `^main$` (regex pattern)
   - **Configuration**: Cloud Build configuration file (yaml or json)
   - **Location**: `/cloudbuild.yaml`
3. Click "**Create**"

**Alternative: Manual trigger creation**
```bash
gcloud builds triggers create github \
  --repo-name=mountain-sickness-analyzer \
  --repo-owner=YOUR-GITHUB-USERNAME \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml
```

---

## First Deployment

### Method 1: Manual Deployment (Test)

```bash
# Ensure you're in project directory
cd mountain-sickness-analyzer

# Deploy manually first to test
gcloud app deploy

# When prompted, type 'y' to continue

# Wait for deployment (2-5 minutes)

# Open your app
gcloud app browse
```

Your app will be available at: `https://YOUR-PROJECT-ID.uc.r.appspot.com`

### Method 2: Trigger GitHub Deployment

```bash
# Make a small change to trigger deployment
echo "# Deployment Test" >> README.md

# Commit and push
git add .
git commit -m "Test automated deployment"
git push origin main

# Monitor build progress
# Go to: https://console.cloud.google.com/cloud-build/builds
```

---

## Continuous Deployment

### Workflow

Now that everything is set up:

1. **Make changes locally** to your code
2. **Commit changes**:
   ```bash
   git add .
   git commit -m "Description of changes"
   ```
3. **Push to GitHub**:
   ```bash
   git push origin main
   ```
4. **Automatic deployment**: Cloud Build detects push and deploys automatically
5. **Monitor**: Check Cloud Build console for deployment status
6. **Verify**: Visit your app URL to see changes live

### Example: Update Application

```bash
# Edit index.html
nano index.html  # or use your preferred editor

# Test locally (optional)
python -m http.server 8000
# Visit http://localhost:8000

# Commit and deploy
git add index.html
git commit -m "Updated risk calculation algorithm"
git push origin main

# Watch deployment
gcloud builds list --ongoing
```

---

## Troubleshooting

### Common Issues

#### 1. Build Fails - Permission Denied

**Error**: "ERROR: (gcloud.app.deploy) Permissions error"

**Solution**:
```bash
# Re-run permission grants
PROJECT_NUMBER=$(gcloud projects describe YOUR-PROJECT-ID --format="value(projectNumber)")

gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
  --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
  --role=roles/appengine.deployer

gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
  --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
  --role=roles/appengine.serviceAdmin
```

#### 2. Billing Not Enabled

**Error**: "Billing must be enabled"

**Solution**: 
1. Visit [GCP Billing](https://console.cloud.google.com/billing)
2. Link billing account to project

#### 3. API Not Enabled

**Error**: "API not enabled"

**Solution**:
```bash
gcloud services enable appengine.googleapis.com
gcloud services enable cloudbuild.googleapis.com
```

#### 4. GitHub Connection Failed

**Solution**:
1. Go to Cloud Build settings
2. Disconnect and reconnect GitHub
3. Ensure GitHub app has repository access

#### 5. Build Timeout

**Error**: "Build timeout"

**Solution**: Update cloudbuild.yaml:
```yaml
timeout: '1200s'  # Increase to 20 minutes
```

### View Logs

```bash
# View recent builds
gcloud builds list --limit=5

# View specific build log
gcloud builds log BUILD-ID

# View app logs
gcloud app logs tail -s default

# View logs in console
# https://console.cloud.google.com/logs
```

### Check App Status

```bash
# View app details
gcloud app describe

# View deployed versions
gcloud app versions list

# View services
gcloud app services list
```

---

## Cost Management

### Monitor Costs

1. Visit [GCP Billing Dashboard](https://console.cloud.google.com/billing)
2. View Reports > Select your project
3. Set up budget alerts:
   ```bash
   # Create budget alert
   gcloud billing budgets create \
     --billing-account=BILLING-ACCOUNT-ID \
     --display-name="Mountain Sickness App Budget" \
     --budget-amount=10USD
   ```

### Free Tier Limits

App Engine Standard includes:
- 28 instance hours per day (F1 or B1)
- 1 GB outbound data per day
- Usually sufficient for personal/small projects

### Reduce Costs

1. **Use automatic scaling** (default in app.yaml)
2. **Set max instances**:
   ```yaml
   automatic_scaling:
     max_instances: 2
   ```
3. **Delete unused versions**:
   ```bash
   gcloud app versions delete OLD-VERSION-ID
   ```

---

## Next Steps

### Custom Domain

1. Go to App Engine > Settings > Custom Domains
2. Add your domain
3. Update DNS records as instructed

### HTTPS (Automatic)

- HTTPS is enabled by default
- Google-managed SSL certificate

### Monitoring

```bash
# Install monitoring
gcloud monitoring dashboards create --config-from-file=dashboard.json
```

### Backup Strategy

- Git serves as backup (code versioned)
- App Engine keeps version history

---

## Quick Reference Commands

```bash
# Deploy manually
gcloud app deploy

# View live app
gcloud app browse

# View logs
gcloud app logs tail

# View builds
gcloud builds list

# Stop app (to save costs)
gcloud app versions stop VERSION-ID

# Delete app (caution!)
# Can only be done via console

# Update gcloud
gcloud components update
```

---

## Support Resources

- **GCP Documentation**: https://cloud.google.com/appengine/docs
- **Cloud Build Docs**: https://cloud.google.com/build/docs
- **Stack Overflow**: Tag: `google-app-engine`
- **GCP Console**: https://console.cloud.google.com

---

## Success Checklist

- ✅ GCP account created and billing enabled
- ✅ GitHub repository created with code
- ✅ Google Cloud SDK installed
- ✅ GCP project created
- ✅ App Engine application created
- ✅ Required APIs enabled
- ✅ Cloud Build permissions granted
- ✅ GitHub connected to Cloud Build
- ✅ Build trigger created
- ✅ First deployment successful
- ✅ App accessible at public URL
- ✅ Automated deployment working

**Congratulations! Your app is now deployed and will auto-update on every push to main branch!** 🎉
