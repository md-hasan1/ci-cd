# 🚀 Guide: Deploying to VPS Using GitHub Actions

This guide outlines the process of deploying a **Node.js application** to a **DigitalOcean VPS** using **GitHub Actions**. Follow the steps below to configure the CI/CD pipeline and deploy your application like a pro.

---

## 🔧 1. Prerequisites

### 💻 On the VPS

- ✅ Node.js and npm installed
- ✅ PM2 installed and configured ([PM2 Docs](https://pm2.keymetrics.io/))
- ✅ Project directory created (e.g., `/var/www/your-project-name`)
- ✅ Public SSH key added to VPS (`~/.ssh/authorized_keys` for the `root` user)

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

**View your private key** (for GitHub secret):

```bash
cat ~/.ssh/id_rsa
```

**Set proper permissions**:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

**Install required dependencies**:

```bash
sudo apt update
sudo apt install git
npm install -g pm2
```

---

### 🌐 On GitHub

1. Create your GitHub repository.
2. Navigate to **Settings > Secrets and variables > Actions**.
3. Add the following secrets:

| 🔐 Secret Name       | 💡 Description                         |
|----------------------|----------------------------------------|
| `HOST`               | Your VPS IP (e.g., `93.188.163.246`)   |
| `USER`               | SSH username (usually `root`)          |
| `SSH_PRIVATE_KEY`    | Your SSH private key (from above)      |

---

## ⚙️ 2. Workflow Configuration

### 🛠️ Create a file: `.github/workflows/deploy.yml`

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup SSH and Deploy
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            set -e
            export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
            cd /var/www/your-project-name
            pm2 stop your-project-name-with-pm2 || true
            git pull origin main
            npm install
            npm run build
            pm2 restart your-project-name-with-pm2 || pm2 start npm --name "your-project-name-with-pm2" -- start
```

📝 **Replace**:
- `/var/www/your-project-name` → your project directory path on VPS
- `your-project-name-with-pm2` → your PM2 process name

---

## 🔍 3. Detailed Workflow Explanation

### 🧠 Step 1: Trigger on Push

```yaml
on:
  push:
    branches: [main]
```

Triggers deployment when you push to the `main` branch.

### 🧱 Step 2: Build Job

- 🛒 Clones the repo
- 🧰 Sets up Node.js v22
- 📦 Installs dependencies
- 🏗️ Runs the `build` script

### 🚚 Step 3: Deploy Job

- Connects via SSH using `appleboy/ssh-action`
- Pulls latest changes
- Installs packages & builds project
- Restarts your app with PM2

---

## 🚀 4. Setting Up PM2

Start your app with a name:

```bash
pm2 start npm --name "your-project-name" -- start
```

Check if it’s running:

```bash
pm2 list
```

---

## 🧯 5. Common Issues & Fixes

### 🔑 SSH Key Issues

- Ensure GitHub secret `SSH_PRIVATE_KEY` matches the VPS public key
- Test locally:

```bash
ssh -i ~/.ssh/id_rsa root@<host>
```

### 🛑 Permissions Issues

Fix permissions if Git pulls or SSH fails:

```bash
sudo chown -R root:root /var/www/your-project-name
```

### 🔁 PM2 Issues

Check process names carefully:

```bash
pm2 restart your-project-name
```

---

## 🎯 6. Running the Workflow

Just push to `main`:

```bash
git push origin main
```

Then sit back and watch the **Actions** tab on GitHub auto-deploy your app 🪄

---

### 🎉 That’s it, Rockstar!

Now you're rocking a full-fledged **CI/CD pipeline** using GitHub Actions & PM2 on a DigitalOcean VPS. If you run into snags — ping the team or your favorite AI assistant (aka me 😎).
