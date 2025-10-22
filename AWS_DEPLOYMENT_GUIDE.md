# AWS Deployment Guide

This guide will help you deploy your simple web page to AWS using S3 static website hosting.

## Prerequisites

- AWS Account with valid credentials
- AWS CLI installed and configured (or use AWS Console)

## Method 1: Deploy to S3 Static Website (Recommended)

### Step 1: Create an S3 Bucket

```bash
# Replace 'your-website-bucket-name' with your unique bucket name
aws s3 mb s3://your-website-bucket-name
```

### Step 2: Configure the bucket for static website hosting

```bash
aws s3 website s3://your-website-bucket-name --index-document index.html --error-document index.html
```

### Step 3: Update bucket policy for public access

Create a file named `bucket-policy.json`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-website-bucket-name/*"
        }
    ]
}
```

Apply the policy:

```bash
aws s3api put-bucket-policy --bucket your-website-bucket-name --policy file://bucket-policy.json
```

### Step 4: Disable block public access

```bash
aws s3api put-public-access-block \
    --bucket your-website-bucket-name \
    --public-access-block-configuration \
    "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```

### Step 5: Upload your files

```bash
# Upload all files to the bucket
aws s3 sync . s3://your-website-bucket-name --exclude ".git/*" --exclude "*.md" --exclude "*.txt"
```

### Step 6: Access your website

Your website will be available at:
```
http://your-website-bucket-name.s3-website-<region>.amazonaws.com
```

Replace `<region>` with your AWS region (e.g., `us-east-1`, `eu-west-1`, etc.)

## Method 2: Using AWS Console (UI)

1. Log in to AWS Console
2. Navigate to S3 service
3. Click "Create bucket"
4. Enter a unique bucket name
5. Uncheck "Block all public access"
6. Create the bucket
7. Go to bucket Properties > Static website hosting > Enable
8. Set index document to `index.html`
9. Go to Permissions > Bucket Policy and add the policy from above
10. Upload `index.html` and `styles.css` files
11. Access your website using the endpoint provided in Static website hosting section

## Method 3: S3 + CloudFront (For HTTPS and Better Performance)

1. Follow Steps 1-5 from Method 1
2. Create a CloudFront distribution:

```bash
aws cloudfront create-distribution \
    --origin-domain-name your-website-bucket-name.s3.amazonaws.com \
    --default-root-object index.html
```

3. Your site will be available at the CloudFront domain (e.g., `d1234567890.cloudfront.net`)
4. Optionally, add a custom domain using Route 53

## Updating Your Website

To update the website content:

```bash
aws s3 sync . s3://your-website-bucket-name --exclude ".git/*" --exclude "*.md" --exclude "*.txt"
```

If using CloudFront, invalidate the cache:

```bash
aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
```

## Cost Estimation

- S3 hosting: ~$0.023 per GB stored + $0.09 per GB transferred
- CloudFront: ~$0.085 per GB transferred (first 10 TB)
- For a small static website, expect costs under $1/month

## Cleanup

To delete your deployment:

```bash
# Empty the bucket first
aws s3 rm s3://your-website-bucket-name --recursive

# Delete the bucket
aws s3 rb s3://your-website-bucket-name
```

## Troubleshooting

**403 Forbidden Error:**
- Check bucket policy is applied correctly
- Verify public access settings
- Ensure files have proper permissions

**404 Not Found:**
- Verify index.html is in the root of the bucket
- Check static website hosting is enabled
- Confirm you're using the website endpoint, not the REST endpoint

## Security Notes

- This setup makes your website publicly accessible
- For production sites, consider:
  - Using CloudFront with HTTPS
  - Setting up a custom domain
  - Implementing AWS WAF for security
  - Using CloudWatch for monitoring
