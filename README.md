# Cloud Computing & DevOps – Task 1
### Static Portfolio hosted on AWS S3 + CloudFront (HTTPS)

**Live URL:** _fill in after deployment, e.g. https://d123abcxyz.cloudfront.net_
**GitHub repo:** _fill in, e.g. https://github.com/yourname/portfolio_

## What this is
A single-page responsive portfolio (Hero, About, Projects, Contact) built with
plain HTML/CSS, using Google Fonts and Font Awesome. Hosted as a static website
on Amazon S3 and served through CloudFront for HTTPS + caching.

## Files
- `index.html` – page markup
- `styles.css` – all styling
- `assets/` – any images (add your own headshot/screenshots here if used)

## Steps I took

### 1. Local project + GitHub
```bash
cd portfolio
git init
git add .
git commit -m "Initial portfolio"
git branch -M main
git remote add origin https://github.com/<yourname>/portfolio.git
git push -u origin main
```

### 2. Create the S3 bucket
- AWS Console → S3 → **Create bucket**
- Bucket name: `<yourname>-portfolio` (must be globally unique)
- Uncheck "Block all public access" (needed for a public static site)
- Enable **Static website hosting** (Properties tab) → index document: `index.html`

### 3. Upload files
```bash
aws s3 sync . s3://<yourname>-portfolio --exclude ".git/*"
```

### 4. Bucket policy (allow public read)
Properties → Permissions → Bucket policy → paste, replacing the bucket name:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<yourname>-portfolio/*"
    }
  ]
}
```
Apply with the CLI if you prefer:
```bash
aws s3api put-bucket-policy --bucket <yourname>-portfolio --policy file://policy.json
```

### 5. Test the S3 website endpoint
Found under Properties → Static website hosting, looks like:
`http://<yourname>-portfolio.s3-website-<region>.amazonaws.com`

### 6. Create a CloudFront distribution
- Origin: the S3 **static website hosting endpoint** (not the bucket REST endpoint)
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Minimum TLS version: TLSv1.2
- Default root object: `index.html`

### 7. (Optional) Custom domain
Route 53 (or your DNS provider) → create an **A record (alias)** pointing to the
CloudFront distribution. Request/attach an ACM certificate in `us-east-1` for HTTPS.

### 8. Verify
Visit the CloudFront domain (`https://dxxxxxxx.cloudfront.net`) and confirm the
padlock/HTTPS works and the site loads correctly.

## Screenshots to include in the submission
- [ ] S3 bucket → Properties → Static website hosting enabled
- [ ] S3 bucket → Permissions → Bucket policy
- [ ] CloudFront distribution → General settings (domain name, status: Deployed)
- [ ] Live site open in a browser showing the padlock/HTTPS

## Notes
- If you get **403 Forbidden**: check the bucket policy and confirm "Block public access" is off.
- If CloudFront shows an old version after you re-upload: create an invalidation
  (`aws cloudfront create-invalidation --distribution-id <ID> --paths "/*"`).
