# Todo App - GCP Cloud Run Deployment

A modern, beautiful to-do application built with Vite and vanilla JavaScript, designed for deployment on Google Cloud Platform using Cloud Run with automated CI/CD via Cloud Build.

## 🚀 Features

- ✨ Beautiful, modern UI with dark mode and gradient effects
- ✅ Full CRUD operations (Create, Read, Update, Delete)
- 🔄 Real-time filtering (All, Active, Completed)
- 💾 Local storage persistence
- 📱 Fully responsive design
- 🐳 Docker containerized
- ☁️ Cloud Run ready
- 🔄 Automated CI/CD with GitHub + Cloud Build

## 🛠️ Technology Stack

- **Frontend**: Vite, Vanilla JavaScript, CSS3
- **Server**: nginx (Alpine Linux)
- **Containerization**: Docker (multi-stage build)
- **Deployment**: Google Cloud Run
- **CI/CD**: Google Cloud Build
- **Registry**: Google Container Registry (GCR)

## 📋 Prerequisites

### Local Development
- Node.js 18+ and npm
- Git

### GCP Deployment
- Google Cloud Platform account
- gcloud CLI installed and configured
- GitHub repository
- Required GCP APIs enabled:
  - Cloud Run API
  - Cloud Build API
  - Container Registry API

## 🏃 Local Development

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd proj_todo
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Start Development Server

```bash
npm run dev
```

The app will be available at `http://localhost:3000`

### 4. Build for Production

```bash
npm run build
```

The production build will be in the `dist/` directory.

### 5. Preview Production Build

```bash
npm run preview
```

## 🐳 Docker Local Testing

### Build Docker Image

```bash
docker build -t todo-app .
```

### Run Container

```bash
docker run -p 8080:80 todo-app
```

Access the app at `http://localhost:8080`

## ☁️ GCP Deployment Setup

### Step 1: Create GCP Project

```bash
# Set your project ID
export PROJECT_ID="your-project-id"

# Create new project (or use existing)
gcloud projects create $PROJECT_ID
gcloud config set project $PROJECT_ID
```

### Step 2: Enable Required APIs

```bash
gcloud services enable \
  cloudbuild.googleapis.com \
  run.googleapis.com \
  containerregistry.googleapis.com
```

### Step 3: Configure Environment

```bash
# Copy environment template
cp .env.example .env

# Edit .env and update PROJECT_ID
# Replace 'your-project-id-here' with your actual project ID
```

### Step 4: Grant Cloud Build Permissions

```bash
# Get your project number
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')

# Grant Cloud Run Admin role to Cloud Build
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
  --role=roles/run.admin

# Grant Service Account User role
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
  --role=roles/iam.serviceAccountUser
```

## 🔄 GitHub + Cloud Build Integration

### Step 1: Connect GitHub Repository

1. Go to [Cloud Build Triggers](https://console.cloud.google.com/cloud-build/triggers)
2. Click **"Connect Repository"**
3. Select **GitHub** as the source
4. Authenticate and select your repository
5. Click **"Connect"**

### Step 2: Create Build Trigger

1. Click **"Create Trigger"**
2. Configure:
   - **Name**: `deploy-todo-app`
   - **Event**: Push to a branch
   - **Source**: Select your repository
   - **Branch**: `^main$` (or `^master$`)
   - **Configuration**: Cloud Build configuration file (yaml or json)
   - **Location**: `/cloudbuild.yaml`
3. Click **"Create"**

### Step 3: Test Automated Deployment

```bash
# Make a change and push to GitHub
git add .
git commit -m "Test deployment"
git push origin main
```

Cloud Build will automatically:
1. Build the Docker image
2. Push to Google Container Registry
3. Deploy to Cloud Run
4. Provide a public URL

## 📦 Manual Deployment (Alternative)

If you prefer manual deployment:

### Build and Push Image

```bash
# Build locally
docker build -t gcr.io/$PROJECT_ID/todo-app:latest .

# Push to GCR
docker push gcr.io/$PROJECT_ID/todo-app:latest
```

### Deploy to Cloud Run

```bash
gcloud run deploy todo-app \
  --image gcr.io/$PROJECT_ID/todo-app:latest \
  --region us-central1 \
  --platform managed \
  --allow-unauthenticated \
  --memory 256Mi \
  --cpu 1 \
  --max-instances 10 \
  --port 80
```

## 🌐 Accessing Your Application

After deployment, Cloud Run provides a URL like:
```
https://todo-app-<hash>-uc.a.run.app
```

You can find it in:
- Cloud Run console
- Cloud Build logs
- Terminal output after deployment

## 🔧 Configuration

### Environment Variables

See `.env.example` for all available configuration options:

- `GCP_PROJECT_ID`: Your GCP project ID
- `GCP_REGION`: Deployment region (default: us-central1)
- `SERVICE_NAME`: Cloud Run service name
- `MEMORY`: Container memory allocation
- `CPU`: Container CPU allocation
- `MAX_INSTANCES`: Maximum auto-scaling instances

### Cloud Run Settings

Modify `cloudbuild.yaml` to adjust:
- Memory allocation (default: 256Mi)
- CPU allocation (default: 1)
- Max instances (default: 10)
- Region (default: us-central1)

### nginx Configuration

Customize `nginx.conf` for:
- Cache headers
- Security headers
- Gzip compression
- Custom routing rules

## 📊 Monitoring & Logs

### View Logs

```bash
# Cloud Build logs
gcloud builds list
gcloud builds log <BUILD_ID>

# Cloud Run logs
gcloud run services logs read todo-app --region us-central1
```

### View Metrics

Visit [Cloud Run Console](https://console.cloud.google.com/run) to see:
- Request count
- Request latency
- Container instance count
- CPU/Memory utilization

## 🐛 Troubleshooting

### Build Fails

**Issue**: Docker build fails
```bash
# Check Cloud Build logs
gcloud builds list --limit=5
gcloud builds log <BUILD_ID>
```

**Common fixes**:
- Ensure `package.json` dependencies are correct
- Check Dockerfile syntax
- Verify build steps in `cloudbuild.yaml`

### Deployment Fails

**Issue**: Cloud Run deployment fails
```bash
# Check service status
gcloud run services describe todo-app --region us-central1
```

**Common fixes**:
- Verify Cloud Build has proper IAM permissions
- Check if required APIs are enabled
- Ensure region matches in all commands

### App Not Loading

**Issue**: Deployed app shows errors

**Common fixes**:
- Check nginx logs: `gcloud run services logs read todo-app`
- Verify nginx.conf syntax
- Test Docker image locally first
- Check build output in `dist/` directory

### Permission Denied

**Issue**: Cloud Build can't deploy to Cloud Run

**Fix**: Grant proper IAM roles:
```bash
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
  --role=roles/run.admin

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
  --role=roles/iam.serviceAccountUser
```

## 🎯 Next Steps

Once you're comfortable with this setup, consider:

1. **Custom Domain**: Map a custom domain to your Cloud Run service
2. **Backend API**: Add a backend service (Node.js, Python, Go)
3. **Database**: Integrate Cloud SQL or Firestore
4. **Authentication**: Add Firebase Auth or Cloud Identity Platform
5. **CDN**: Enable Cloud CDN for global performance
6. **Monitoring**: Set up Cloud Monitoring alerts
7. **Multiple Environments**: Create dev/staging/prod pipelines
8. **Infrastructure as Code**: Use Terraform for resource management

## 📝 Project Structure

```
proj_todo/
├── src/
│   ├── main.js           # Application logic
│   └── style.css         # Styles
├── index.html            # HTML template
├── package.json          # Node dependencies
├── vite.config.js        # Vite configuration
├── Dockerfile            # Multi-stage Docker build
├── nginx.conf            # nginx server config
├── cloudbuild.yaml       # CI/CD pipeline
├── .env.example          # Environment template
├── .gitignore            # Git ignore rules
├── .dockerignore         # Docker ignore rules
└── README.md             # This file
```

## 💡 Tips

- **Cost Optimization**: Cloud Run charges only when serving requests
- **Fast Builds**: Multi-stage Docker build keeps images small
- **Security**: App runs as non-root with minimal nginx Alpine image
- **Performance**: Gzip compression and caching enabled by default
- **Scalability**: Auto-scales from 0 to 10 instances based on traffic

## 📄 License

MIT

## 🤝 Contributing

Contributions are welcome! This is a learning project for GCP deployment.

---

**Built with ❤️ | Deployed on Google Cloud Run**
