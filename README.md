# Sponsorship Tracker
A public, live portfolio, tracking my UK cloud job hunt, every application, sponsorship status, certifications and interview outcome.
Built to demonstrate AWS skills and to help other international students see what works.

## Architecture
Static site:HTML/CSS/JS hosted in a private S3 bucket
CDN + HTTPS: CloudFront with Origin Access Control (OAC), ACM certificate
DNS: Route 53 with a custom domain
API: API Gateway → Lambda → DynamoDB for visitor counter
CI/CD: GitHub Actions deploying on push to `main` via OIDC federation (no long-lived AWS keys)

## Status

Work in progress. See commits for build log.