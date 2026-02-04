# Production Deployment Guide - Vila Pinheiro

Complete guide to deploy the Vila Pinheiro website to production with Vercel, TiDB Cloud, and AWS S3.

## 📋 Prerequisites Checklist

Before starting, ensure you have:
- [ ] GitHub account (for code repository)
- [ ] Vercel account (free tier available)
- [ ] TiDB Cloud account (free tier available)
- [ ] AWS account (for S3 storage)
- [ ] Your domain name registered (optional, can use Vercel's free domain initially)
- [ ] Manus OAuth credentials (from manus.im)

---

## 🚀 Step 1: Prepare GitHub Repository

### 1.1 Create GitHub Repository

```bash
# Navigate to your project
cd vila-pinheiro-site

# Initialize git if not already done
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit: Vila Pinheiro production ready"

# Create main branch
git branch -M main

# Add remote (replace with your GitHub URL)
git remote add origin https://github.com/YOUR_USERNAME/vila-pinheiro-site.git

# Push to GitHub
git push -u origin main
```

### 1.2 Create `.env.production` File

Create a file named `.env.production` in the root directory:

```env
# Database (will be configured in TiDB Cloud)
DATABASE_URL=mysql://user:password@host:3306/vila_pinheiro

# AWS S3 Configuration
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=YOUR_AWS_ACCESS_KEY
AWS_SECRET_ACCESS_KEY=YOUR_AWS_SECRET_KEY
AWS_S3_BUCKET=vila-pinheiro-prod

# Manus OAuth
MANUS_CLIENT_ID=YOUR_MANUS_CLIENT_ID
MANUS_CLIENT_SECRET=YOUR_MANUS_CLIENT_SECRET
MANUS_REDIRECT_URI=https://your-domain.com/api/oauth/callback
OAUTH_SERVER_URL=https://manus.im

# Session & Security
SESSION_SECRET=YOUR_RANDOM_SECRET_KEY_HERE
JWT_SECRET=YOUR_RANDOM_JWT_SECRET_HERE

# Node Environment
NODE_ENV=production

# Owner ID (for admin access)
OWNER_OPEN_ID=YOUR_OPEN_ID

# Public URL
PUBLIC_URL=https://your-domain.com

# Vite Configuration
VITE_APP_ID=vila-pinheiro
VITE_OAUTH_PORTAL_URL=https://manus.im
```

**⚠️ IMPORTANT:** Do NOT commit this file to GitHub. Add to `.gitignore`:

```bash
echo ".env.production" >> .gitignore
git add .gitignore
git commit -m "Add .env.production to gitignore"
git push
```

---

## 🗄️ Step 2: Setup TiDB Cloud Database

### 2.1 Create TiDB Cloud Account

1. Go to [tidb.cloud](https://tidb.cloud)
2. Sign up for a free account
3. Create a new cluster:
   - **Cluster Name:** vila-pinheiro-prod
   - **Region:** Select closest to your users
   - **Tier:** Free tier (sufficient for starting)

### 2.2 Get Connection String

1. In TiDB Cloud dashboard, go to your cluster
2. Click "Connect"
3. Copy the connection string (looks like: `mysql://user:password@host:3306/database`)
4. Save this as your `DATABASE_URL`

### 2.3 Create Database

1. In TiDB Cloud, go to "SQL Editor"
2. Run these commands:

```sql
CREATE DATABASE vila_pinheiro;
USE vila_pinheiro;
```

### 2.4 Run Migrations

Once you have the connection string, run migrations locally:

```bash
DATABASE_URL="your-connection-string" pnpm db:push
```

This will create all necessary tables for users and images.

---

## 🪣 Step 3: Setup AWS S3

### 3.1 Create S3 Bucket

1. Go to [AWS Console](https://console.aws.amazon.com)
2. Navigate to S3
3. Click "Create bucket"
4. **Bucket name:** vila-pinheiro-prod
5. **Region:** us-east-1 (or your preferred region)
6. **Block Public Access:** Uncheck "Block all public access" (images need to be public)
7. Click "Create bucket"

### 3.2 Create IAM User

1. Go to IAM in AWS Console
2. Click "Users" → "Create user"
3. **Username:** vila-pinheiro-app
4. Click "Create user"

### 3.3 Create Access Keys

1. Select the newly created user
2. Go to "Security credentials"
3. Click "Create access key"
4. Choose "Application running outside AWS"
5. Copy:
   - **Access Key ID** → `AWS_ACCESS_KEY_ID`
   - **Secret Access Key** → `AWS_SECRET_ACCESS_KEY`

### 3.4 Attach S3 Policy

1. In the user's "Permissions" tab, click "Add inline policy"
2. Choose "JSON" and paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::vila-pinheiro-prod",
        "arn:aws:s3:::vila-pinheiro-prod/*"
      ]
    }
  ]
}
```

3. Click "Create policy"

### 3.5 Configure Bucket CORS

1. Go to your S3 bucket
2. Click "Permissions" → "CORS"
3. Add this configuration:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
    "AllowedOrigins": ["https://your-domain.com"],
    "ExposeHeaders": ["ETag"]
  }
]
```

---

## 🔐 Step 4: Setup Manus OAuth

### 4.1 Create Manus Application

1. Go to [manus.im](https://manus.im)
2. Log in or create account
3. Go to "Applications"
4. Click "New Application"
5. Fill in:
   - **Name:** Vila Pinheiro
   - **Redirect URI:** `https://your-domain.com/api/oauth/callback`
6. Copy:
   - **Client ID** → `MANUS_CLIENT_ID`
   - **Client Secret** → `MANUS_CLIENT_SECRET`

---

## 🚀 Step 5: Deploy to Vercel

### 5.1 Connect Vercel to GitHub

1. Go to [vercel.com](https://vercel.com)
2. Sign up or log in
3. Click "New Project"
4. Select "Import Git Repository"
5. Search for and select `vila-pinheiro-site`
6. Click "Import"

### 5.2 Configure Environment Variables

1. In Vercel project settings, go to "Environment Variables"
2. Add all variables from your `.env.production`:
   - `DATABASE_URL`
   - `AWS_REGION`
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_S3_BUCKET`
   - `MANUS_CLIENT_ID`
   - `MANUS_CLIENT_SECRET`
   - `MANUS_REDIRECT_URI`
   - `OAUTH_SERVER_URL`
   - `SESSION_SECRET`
   - `JWT_SECRET`
   - `NODE_ENV=production`
   - `OWNER_OPEN_ID`
   - `PUBLIC_URL`
   - `VITE_APP_ID`
   - `VITE_OAUTH_PORTAL_URL`

3. Click "Deploy"

### 5.3 Wait for Build

Vercel will automatically:
- Build your React frontend
- Build your Express backend
- Deploy everything to their CDN

This typically takes 2-5 minutes.

---

## 🌐 Step 6: Connect Your Domain

### 6.1 Get Vercel Domain

Once deployed, Vercel gives you a temporary domain like:
`vila-pinheiro-site.vercel.app`

### 6.2 Connect Custom Domain

1. In Vercel project, go to "Settings" → "Domains"
2. Click "Add Domain"
3. Enter your domain (e.g., `vilapinheiro.com.br`)
4. Follow the DNS setup instructions:
   - If using Vercel nameservers: Change your domain registrar's nameservers
   - If using CNAME: Add CNAME record pointing to `cname.vercel-dns.com`

### 6.3 SSL/HTTPS

Vercel automatically provisions SSL certificates via Let's Encrypt. Your site will be HTTPS within minutes.

---

## ✅ Step 7: Verify Everything Works

### 7.1 Test the Website

1. Visit your domain: `https://vilapinheiro.com.br`
2. Check that:
   - [ ] Home page loads
   - [ ] Images display correctly
   - [ ] Navigation works
   - [ ] Responsive design works on mobile

### 7.2 Test Reservations Form

1. Go to "Reservas" page
2. Fill in the form
3. Click "Enviar pelo WhatsApp"
4. Verify WhatsApp opens with pre-filled message

### 7.3 Test Admin Panel

1. Go to `/admin`
2. Click "Fazer Login"
3. Authenticate with Manus
4. Upload a test image:
   - Select image file
   - Choose category "Cabanas"
   - Select "Ipê"
   - Click "Enviar Imagem"
5. Verify image appears in gallery
6. Go back to cabanas page and verify new image displays

### 7.4 Check Logs

In Vercel dashboard:
1. Go to "Deployments"
2. Click latest deployment
3. Go to "Logs" → "Runtime Logs"
4. Look for any errors

---

## 🔧 Step 8: Post-Deployment Configuration

### 8.1 Update MANUS_REDIRECT_URI

If you changed your domain, update in Manus:
1. Go to [manus.im](https://manus.im)
2. Edit your application
3. Update **Redirect URI** to your new domain
4. Save changes

### 8.2 Update S3 CORS

Update your S3 bucket CORS configuration with your actual domain:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
    "AllowedOrigins": ["https://vilapinheiro.com.br"],
    "ExposeHeaders": ["ETag"]
  }
]
```

### 8.3 Enable Backups

For production database:
1. In TiDB Cloud, enable automated backups
2. Set retention to at least 7 days

---

## 📊 Step 9: Setup Monitoring

### 9.1 Vercel Analytics

1. In Vercel project settings, enable "Web Analytics"
2. This tracks page performance and user metrics

### 9.2 Error Tracking (Optional)

Consider adding Sentry for error tracking:
1. Create account at [sentry.io](https://sentry.io)
2. Create new project for "Node.js"
3. Add `SENTRY_DSN` to environment variables

---

## 🆘 Troubleshooting

### Issue: "Database connection failed"

**Solution:**
```bash
# Test connection locally
DATABASE_URL="your-connection-string" pnpm db:push

# Check Vercel logs
vercel logs
```

### Issue: "AWS S3 access denied"

**Solution:**
1. Verify IAM user has correct policy
2. Check bucket name matches exactly
3. Verify AWS credentials in Vercel env vars

### Issue: "WhatsApp redirect not working"

**Solution:**
1. Verify `MANUS_REDIRECT_URI` matches your domain
2. Update in both Manus dashboard and Vercel env vars
3. Redeploy after changes

### Issue: "Images not uploading"

**Solution:**
1. Check S3 bucket exists and is accessible
2. Verify CORS configuration includes your domain
3. Check AWS credentials have S3 permissions
4. Look at browser console for specific errors

---

## 📝 Important Notes

1. **Environment Variables:** Never commit `.env.production` to GitHub
2. **Secrets:** Keep AWS keys and secrets safe
3. **Database:** Regular backups are essential
4. **Monitoring:** Set up alerts for errors and downtime
5. **Updates:** Keep dependencies updated for security

---

## 🎉 You're Live!

Your Vila Pinheiro website is now live in production! 

### Next Steps:
1. Share the link with friends and family
2. Start collecting bookings through the WhatsApp form
3. Upload photos via the Admin panel
4. Monitor analytics and performance
5. Gather feedback and make improvements

### Support:
- Vercel docs: https://vercel.com/docs
- TiDB docs: https://docs.tidb.cloud
- AWS S3 docs: https://docs.aws.amazon.com/s3
- Manus docs: https://manus.im/docs

---

**Congratulations! Your permanent website is ready! 🚀**
