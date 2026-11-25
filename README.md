# VVS AWS Migration: case study

Moving a small plumbing firm's website from an office PC to AWS.

## The problem

The firm (12 employees) runs its website on an office PC. After a power cut the site was down for two days. Nobody at the firm knows IT, and the budget is at most 300 SEK per month. The site has 5 text pages and 400 images (about 2 GB), roughly 3,000 visits per month mostly from the Mälardalen area, and a quote form (about 40 requests per month).

## Architecture

```mermaid
flowchart LR
  V[Visitor] --> CF[CloudFront]
  CF --> S3[(S3 private bucket)]
  V -->|quote form| API[API Gateway]
  API --> L[Lambda]
  L --> SES[SES]
  SES --> O[Owner email]
  E[Employee] -->|upload images/| S3
```

| Part | Service | Why | If removed |
| --- | --- | --- | --- |
| Site and images | S3 (private bucket) | Cheap, durable, no server to patch | No place to store the site |
| Delivery | CloudFront with Origin Access Control | HTTPS, caching, fast in Sweden, bucket stays private | The bucket would have to be public and there is no HTTPS on a custom domain |
| Contact form | API Gateway + Lambda | Static hosting cannot process forms; pay only per submission | The form cannot work |
| Email to owner | SES | Sends each request to the owner | The owner never sees requests |
| Employee uploads | IAM user limited to `images/` | One person can add images and nothing else | Uploading would need broader access |

## Key decisions

- **Hosting:** S3 + CloudFront, because there is no server that can go down in a power cut, nothing to patch, and the cost is very low.
- **Contact form:** API Gateway + Lambda + SES, because a static site cannot process a form and Lambda only costs money when someone submits.
- **Region:** Stockholm (eu-north-1). Visitors are nearby and the form holds personal data (GDPR). In my own test, the connection time from Sweden was 0.136 s to N. Virginia versus 0.033 s to Stockholm, about four times faster.
- **Access:** Visitors can read the site and submit the form. They cannot read the bucket directly or upload. One employee can upload and view files in `images/` only. The owner receives requests by email and needs no AWS access.

## Cost estimate (Stockholm)

| Item | Monthly cost |
| --- | --- |
| S3 (2 GB) | [from Pricing Calculator] |
| CloudFront (about 3,000 visits) | [from Pricing Calculator] |
| API Gateway + Lambda (about 40 requests) | [from Pricing Calculator] |
| SES (about 40 emails) | [from Pricing Calculator] |
| **Total** | **[total] (limit: 300 SEK)** |
| For comparison: one t3.micro EC2 instance running all month | about 7.88 USD (0.0108 USD/hour in Stockholm) |

## Risks and mitigations

| Risk | Fix |
| --- | --- |
| Form spam or abuse | Hidden honeypot field, API Gateway throttling, validate and cap input length |
| Bucket accidentally made public | Keep Block Public Access on, only CloudFront can read (OAC), set a billing alert |
| Employee account stolen | Limited policy, MFA, access keys not shared |
| Personal data in emails | Collect only needed fields, delete old emails, state the purpose |
| Site down again | No server to fail; S3 and CloudFront are managed and replicated |

## What I built and learned

In AWS Learner Lab I built an EC2 web server and an S3 static site. I tested:

- **EC2 vs serverless:** an EC2 server runs and bills all the time and needs patching; Lambda runs only on requests.
- **Security groups:** why the page did not load until port 80 was allowed, and why SSH (port 22) should be limited to My IP. My SSH log showed an outside IP probing port 22 within hours.
- **S3 access models:** a public bucket policy compared with a private bucket and a presigned URL.
- **Region latency:** my own measurements, Virginia vs Stockholm (see Key decisions).

## Limitations

Learner Lab limits which regions and services I can use, so [what was blocked, in your own words]. The full VVS solution (CloudFront, the form Lambda, SES) is a design; [what you actually built, if anything].
