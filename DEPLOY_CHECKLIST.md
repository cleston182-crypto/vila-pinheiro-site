# Vila Pinheiro - Production Deployment Checklist

Quick checklist to deploy your website to production in 30 minutes.

## ✅ Pre-Deployment (5 minutes)

- [ ] You have a GitHub account
- [ ] You have a Vercel account (free at vercel.com)
- [ ] You have an AWS account (free tier available)
- [ ] You have your domain name ready (or will use Vercel's free domain)

## 🔧 Step 1: Push Code to GitHub (5 minutes)

```bash
cd vila-pinheiro-site
git init
git add .
git commit -m "Initial commit: Vila Pinheiro"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/vila-pinheiro-site.git
git push -u origin main
```

**Save your GitHub URL:** `https://github.com/YOUR_USERNAME/vila-pinheiro-site`

## 🗄️ Step 2: Setup TiDB Cloud Database (5 minutes)

1. Go to https://tidb.cloud
2. Sign up and create a free cluster
3. Wait for cluster to be ready (~2 minutes)
4. Click "Connect" and copy the connection string
5. In SQL Editor, run:
   ```sql
   CREATE DATABASE vila_pinheiro;
   ```

**Save your DATABASE_URL:** `mysql://user:password@host:3306/vila_pinheiro`

## 🪣 Step 3: Setup AWS S3 (5 minutes)

1. Go to https://console.aws.amazon.com
2. Search for "S3" and create bucket:
   - Name: `vila-pinheiro-prod`
   - Region: `us-east-1`
   - Uncheck "Block public access"
3. Go to IAM → Users → Create user: `vila-pinheiro-app`
4. Create access key and copy:
   - Access Key ID
   - Secret Access Key
5. Attach policy (inline):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Action": ["s3:*"],
       "Resource": ["arn:aws:s3:::vila-pinheiro-prod", "arn:aws:s3:::vila-pinheiro-prod/*"]
     }]
   }
   ```

**Save your AWS credentials:**
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

## 🔐 Step 4: Setup Manus OAuth (2 minutes)

1. Go to https://manus.im
2. Log in and go to Applications
3. Create new application:
   - Name: Vila Pinheiro
   - Redirect URI: `https://your-domain.com/api/oauth/callback`
4. Copy Client ID and Secret

**Save your Manus credentials:**
- `MANUS_CLIENT_ID`
- `MANUS_CLIENT_SECRET`

## 🚀 Step 5: Deploy to Vercel (5 minutes)

1. Go to https://vercel.com
2. Click "New Project" → "Import Git Repository"
3. Select your GitHub repository
4. Click "Deploy"
5. Wait for build to complete (~2 minutes)

**Your temporary URL:** `https://vila-pinheiro-site.vercel.app`

## ⚙️ Step 6: Add Environment Variables to Vercel (3 minutes)

In Vercel project settings → Environment Variables, add:

```
DATABASE_URL=mysql://user:password@host:3306/vila_pinheiro
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=YOUR_KEY
AWS_SECRET_ACCESS_KEY=YOUR_SECRET
AWS_S3_BUCKET=vila-pinheiro-prod
MANUS_CLIENT_ID=YOUR_CLIENT_ID
MANUS_CLIENT_SECRET=YOUR_CLIENT_SECRET
MANUS_REDIRECT_URI=https://your-domain.com/api/oauth/callback
OAUTH_SERVER_URL=https://manus.im
SESSION_SECRET=generate_random_string_here
JWT_SECRET=generate_random_string_here
NODE_ENV=production
OWNER_OPEN_ID=your_manus_open_id
PUBLIC_URL=https://your-domain.com
VITE_APP_ID=vila-pinheiro
VITE_OAUTH_PORTAL_URL=https://manus.im
```

Then click "Redeploy" in Deployments tab.

## 🌐 Step 7: Connect Your Domain (Optional)

1. In Vercel → Settings → Domains
2. Add your domain (e.g., `vilapinheiro.com.br`)
3. Follow DNS setup instructions
4. SSL certificate auto-provisions in ~5 minutes

## ✅ Step 8: Test Everything

- [ ] Visit your domain - page loads
- [ ] Click on Cabanas - images show
- [ ] Go to Reservas - form works
- [ ] Fill form and test WhatsApp redirect
- [ ] Go to `/admin` - login works
- [ ] Upload a test image in Admin
- [ ] Image appears in gallery
- [ ] Check browser console - no errors

## 🎉 Done!

Your website is now live in production! 

### What's Next:
1. Share your domain with friends/family
2. Start accepting bookings via WhatsApp
3. Upload photos via Admin panel
4. Monitor performance

### Quick Links:
- Your Website: `https://your-domain.com`
- Admin Panel: `https://your-domain.com/admin`
- Vercel Dashboard: https://vercel.com/dashboard
- TiDB Cloud: https://tidb.cloud
- AWS Console: https://console.aws.amazon.com

---

**Questions?** Check PRODUCTION_DEPLOYMENT.md for detailed instructions.
