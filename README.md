# Sponsorship Tracker
A public, live portfolio, tracking my UK cloud job hunt, every application, sponsorship status, certifications and interview outcome.
Built to demonstrate AWS skills and to help other international students see what works.

## Architecture

- **Static site:** HTML/CSS/JS hosted in a private S3 bucket
- **CDN + HTTPS:** CloudFront with Origin Access Control (OAC), ACM certificate
- **DNS:** Route 53 with a custom domain
- **API:** API Gateway → Lambda → DynamoDB for visitor counter
- **CI/CD:** GitHub Actions deploying on push to `main` via OIDC federation (no long-lived AWS keys)

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

## Phase C — Continuous deployment ✅

Push to `main` deploys the site automatically:

1. GitHub Actions workflow triggers on push
2. Workflow assumes an AWS IAM role via **OIDC federation** — no long-lived AWS access keys stored in GitHub
3. Role's trust policy is scoped to `repo:sabbirsavvy/cloud-portfolio:ref:refs/heads/main` — only main branch on this repo can assume it
4. Permissions policy scoped to `s3:PutObject`/`DeleteObject`/`ListBucket`/`GetObject` on the one bucket + `cloudfront:CreateInvalidation` on the one distribution
5. Workflow syncs files to S3 and issues a CloudFront invalidation
6. Site is live ~60-90 seconds after `git push`

End-to-end: editing `index.html` locally → live at https://sabbirahmed.uk in under two minutes, no manual steps, no credentials in repo secrets.


## Live Operations Dashboard

Public CloudWatch dashboard: [https://cloudwatch.amazonaws.com/dashboard.html?dashboard=sponsorship-tracker&context=eyJSIjoidXMtZWFzdC0xIiwiRCI6ImN3LWRiLTAwOTA3NTU3NDIyMCIsIlUiOiJ1cy1lYXN0LTFfMUVUWmNBbWFGIiwiQyI6IjU0ZnY5bG1vOHUyZ2tlaXI2ZzliYmtoYW8xIiwiSSI6InVzLWVhc3QtMTo4YTU0YjE1Yi1jYjZiLTRhOGEtYTgzYi1kMWFlMDdlNGMzNjUiLCJNIjoiUHVibGljIn0%3D]

Metrics chosen to answer six questions at a glance:

- **Is anyone visiting?** CloudFront request count.
- **Are visitors getting errors?** CloudFront total error rate.
- **Is the API working?** API Gateway 2xx/4xx/5xx response mix.
- **Is the Lambda healthy?** Invocations, errors, average duration.
- **Is DynamoDB happy and within capacity?** Consumed read/write capacity units.
- **Is anything starting to cost real money?** Estimated charges in USD.

The dashboard is the answer to "how would you operate this if it had real users?"