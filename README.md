# Mountain Sickness Risk Analysis

A comprehensive web application to assess altitude sickness risk and provide personalized recommendations for mountain travelers.

## Features

- **Risk Assessment**: Calculate personalized altitude sickness risk based on multiple factors
- **Location Presets**: Quick selection of popular mountain destinations
- **Comprehensive Analysis**: Detailed recommendations, symptoms, and prevention strategies
- **Emergency Information**: Critical warning signs and immediate action steps
- **Responsive Design**: Works on desktop, tablet, and mobile devices

## Live Demo

Visit: `https://YOUR-PROJECT-ID.uc.r.appspot.com`

## Technology Stack

- Pure HTML, CSS, and JavaScript (no frameworks required)
- Deployed on Google Cloud Platform (App Engine)
- Automated deployment via GitHub and Cloud Build

## Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/YOUR-USERNAME/mountain-sickness-analyzer.git
   cd mountain-sickness-analyzer
   ```

2. Open `index.html` in your browser:
   ```bash
   open index.html
   # or
   python -m http.server 8000
   # then visit http://localhost:8000
   ```

## Deployment to Google Cloud

### Prerequisites

- Google Cloud Platform account
- GitHub account
- GCP Project with billing enabled
- App Engine enabled in your project

### Automated Deployment via GitHub

1. **Fork/Clone this repository to your GitHub account**

2. **Set up Google Cloud Project**:
   ```bash
   gcloud projects create YOUR-PROJECT-ID
   gcloud config set project YOUR-PROJECT-ID
   gcloud app create --region=us-central
   ```

3. **Connect GitHub to Cloud Build**:
   - Go to [Cloud Build Triggers](https://console.cloud.google.com/cloud-build/triggers)
   - Click "Connect Repository"
   - Select "GitHub" and authenticate
   - Select your repository
   - Create a trigger:
     - Name: `deploy-to-app-engine`
     - Event: Push to branch
     - Branch: `^main$` (or `^master$`)
     - Configuration: Cloud Build configuration file
     - Location: `/cloudbuild.yaml`

4. **Grant Cloud Build permissions**:
   ```bash
   PROJECT_NUMBER=$(gcloud projects describe YOUR-PROJECT-ID --format="value(projectNumber)")
   gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
     --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
     --role=roles/appengine.deployer
   
   gcloud projects add-iam-policy-binding YOUR-PROJECT-ID \
     --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
     --role=roles/appengine.serviceAdmin
   ```

5. **Deploy**:
   - Push changes to the `main` branch
   - Cloud Build will automatically deploy to App Engine
   - View deployment progress in Cloud Build console

### Manual Deployment

```bash
# Install Google Cloud SDK if not already installed
# Visit: https://cloud.google.com/sdk/docs/install

# Authenticate
gcloud auth login

# Set project
gcloud config set project YOUR-PROJECT-ID

# Deploy
gcloud app deploy

# View your app
gcloud app browse
```

## Project Structure

```
mountain-sickness-analyzer/
├── index.html              # Main application file
├── app.yaml               # App Engine configuration
├── cloudbuild.yaml        # Cloud Build CI/CD configuration
├── .gitignore            # Git ignore rules
├── .gcloudignore         # GCloud ignore rules
├── README.md             # This file
└── DEPLOYMENT.md         # Detailed deployment guide
```

## Configuration

### App Engine Settings (app.yaml)

- **Runtime**: Python 3.9 (used for static file serving)
- **Scaling**: Automatic scaling with configurable instances
- **Handlers**: Static file serving for HTML/CSS/JS

### Cost Considerations

- App Engine Standard Environment includes a free tier
- Estimated cost for low traffic: $0-5/month
- Monitor usage in GCP Console

## Environment Variables

No environment variables required for basic deployment.

## Monitoring and Logs

View logs in Google Cloud Console:
```bash
gcloud app logs tail -s default
```

Or visit: https://console.cloud.google.com/logs

## Support

For issues or questions:
- Create an issue in the GitHub repository
- Consult [GCP Documentation](https://cloud.google.com/appengine/docs)

## License

MIT License - feel free to use and modify

## Disclaimer

This tool provides educational information only and is not a substitute for professional medical advice. Always consult with a healthcare provider before traveling to high altitudes, especially if you have pre-existing medical conditions.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgments

- Altitude sickness information based on medical research and guidelines
- Built for educational and informational purposes
