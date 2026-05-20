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

## Phase A — Static site infrastructure ✅

Live at: https://sabbirahmed.uk

- Private S3 bucket holding HTML/CSS/JS assets
- CloudFront distribution with Origin Access Control (OAC) reading from S3
  - HTTPS via ACM certificate (us-east-1)
  - TLSv1.2_2021 security policy
  - HTTP/2 enabled, global edge distribution
- Route 53 hosted zone with alias A-records for apex and www
- No public access to S3; all reads go through CloudFront

## Phase B — Serverless backend ✅

Visitor counter implemented as:
- DynamoDB table (`visitor-count`) — single item, atomic increment via `ADD` UpdateExpression
- Lambda function (`visitor-counter`, Python 3.14) — boto3 against DynamoDB
- API Gateway HTTP API — `GET /visit`, CORS configured for browser clients
- IAM execution role scoped to `dynamodb:GetItem` and `dynamodb:UpdateItem` on the specific table ARN (least privilege)

Client-side `fetch` from `index.html` updates the count on page load.