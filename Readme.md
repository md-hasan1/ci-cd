# Github CI/CD In VPS

## Guide: Deploying to VPS Using GitHub Actions

This guide outlines the process of deploying a Node.js application to a DigitalOcean VPS using GitHub Actions. Follow the steps below to configure the CI/CD pipeline and deploy your application.

---

## **1. Prerequisites**

### **On the VPS**

- Ensure you have Node.js and npm installed.
- Install and configure [PM2](https://pm2.keymetrics.io/) to manage your application process.
- Set up your project directory on the VPS (e.g., `/var/www/your-project-name`).
- Add your SSH public key to the `~/.ssh/authorized_keys` file for the `root` user account used to SSH into the VPS.

### **On GitHub**

- Create a repository for your project.
- Add the following secrets to your repository under **Settings > Secrets and variables > Actions**:
    - **HOST**: The IP address of your VPS.
    - **USER**: The username (e.g., `root`) used for SSH access.
    - **SSH_PRIVATE_KEY**: The private SSH key corresponding to the public key added to the VPS.

### **Add the Public Key to `authorized_keys`**

Append the public key to the `authorized_keys` file on the VPS:

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

<pre><code class="language-bash">cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys</code> <button onclick="navigator.clipboard.writeText('cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys')"></button></pre>

### **View the Private Key and Add to GitHub Secret**

Use this command to display the private key (needed for GitHub secrets):

```bash
cat ~/.ssh/id_rsa
```

<pre><code class="language-bash">cat ~/.ssh/id_rsa</code> <button onclick="navigator.clipboard.writeText('cat ~/.ssh/id_rsa')"></button></pre>

### **Set Proper Permissions**

Ensure the `~/.ssh` directory and its files have the correct permissions to prevent access issues:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

<pre><code class="language-bash">chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys</code> <button onclick="navigator.clipboard.writeText('chmod 700 ~/.ssh\nchmod 600 ~/.ssh/authorized_keys')"></button></pre>

### **Install Required Dependencies**

Ensure the VPS has the following installed:

```bash
sudo apt update
sudo apt install git
npm install -g pm2
```

<pre><code class="language-bash">sudo apt update
sudo apt install git
npm install -g pm2</code> <button onclick="navigator.clipboard.writeText('sudo apt update\nsudo apt install git\nnpm install -g pm2')"></button></pre>

---

## **2. Workflow Configuration**

### **Workflow File**

Create a file named `.github/workflows/deploy.yml` in your repository with the following content:

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

            # Load NVM (if needed for future compatibility)
            export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \ . "$NVM_DIR/nvm.sh"

            # Navigate to project directory
            cd /var/www/your-project-name

            # Stop the application using PM2
            pm2 stop your-project-name-with-pm2

            # Pull the latest changes
            git pull origin main
            
            # Install dependencies
            npm install

            # Build the application
            npm run build

            # Start or restart the application using PM2
            pm2 restart your-project-name-with-pm2
```

<pre><code class="language-yaml">name: Deploy to VPS

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

            # Load NVM (if needed for future compatibility)
            export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \ . "$NVM_DIR/nvm.sh"

            # Navigate to project directory
            cd /var/www/your-project-name

            # Stop the application using PM2
            pm2 stop your-project-name-with-pm2

            # Pull the latest changes
            git pull origin main
            
            # Install dependencies
            npm install

            # Build the application
            npm run build

            # Start or restart the application using PM2
            pm2 restart your-project-name-with-pm2</code> <button onclick="navigator.clipboard.writeText('name: Deploy to VPS\n\non:\n  push:\n    branches: [main]\n\njobs:\n  build:\n    runs-on: ubuntu-latest\n\n    steps:\n      - name: Checkout repository\n        uses: actions/checkout@v4\n\n      - name: Setup Node.js\n        uses: actions/setup-node@v4\n        with:\n          node-version: 22\n\n      - name: Install dependencies\n        run: npm install\n\n      - name: Build\n        run: npm run build\n\n  deploy:\n    runs-on: ubuntu-latest\n\n    needs: build\n\n    steps:\n      - name: Checkout repository\n        uses: actions/checkout@v4\n\n      - name: Setup SSH and Deploy\n        uses: appleboy/ssh-action@v1.2.0\n        with:\n          host: ${{ secrets.HOST }}\n          username: ${{ secrets.USER }}\n          key: ${{ secrets.SSH_PRIVATE_KEY }}\n          script: |\n            set -e\n\n            # Load NVM (if needed for future compatibility)\n            export NVM_DIR=\"$HOME/.nvm\"\n            [ -s \"$NVM_DIR/nvm.sh\" ] && \ . \"$NVM_DIR/nvm.sh\"\n\n            # Navigate to project directory\n            cd /var/www/your-project-name\n\n            # Stop the application using PM2\n            pm2 stop your-project-name-with-pm2\n\n            # Pull the latest changes\n            git pull origin main\n            \n            # Install dependencies\n            npm install\n\n            # Build the application\n            npm run build\n\n            # Start or restart the application using PM2\n            pm2 restart your-project-name-with-pm2')"></button></pre>

Replace `/var/www/your-project-name` with the actual path to your project directory on the VPS.

Replace `your-project-name-with-pm2` with the actual application name running on PM2.

---

## **3. Detailed Workflow Explanation**

### **Step 1: Trigger on Push**

The workflow triggers on every `push` to the `main` branch:

```yaml
on:
  push:
    branches: [main]
```

<pre><code class="language-yaml">on:
  push:
    branches: [main]</code> <button onclick="navigator.clipboard.writeText('on:\n  push:\n    branches: [main]')"></button></pre>

### **Step 2: Build Job**

- **Checkout repository**: Clones the repository.
- **Setup Node.js**: Installs Node.js version 22.
- **Install dependencies**: Runs `npm install` to install required packages.
- **Build the project**: Runs the build command specified in your `package.json`.

### **Step 3: Deploy Job**

- **SSH Action**: The `appleboy/ssh-action` connects to the VPS using the provided SSH key and runs deployment commands:
    1. Navigates to the project directory.
    2. Stops the running application using PM2 with the specific project name.
    3. Pulls the latest changes from the repository.
    4. Builds the application using `npm run build`.
    5. Restarts the application with PM2 using the specific project name.

---

## **4. Setting Up PM2**

To ensure your application is managed properly by PM2, start it with a process name:

```bash
pm2 start npm --name "your-project-name" -- start
```

<pre><code class="language-bash">pm2 start npm --name "your-project-name" -- start</code> <button onclick="navigator.clipboard.writeText('pm2 start npm --name \"your-project-name\" -- start')"></button></pre>

Verify the process is running:

```bash
pm2 list
```

<pre><code class="language-bash">pm2 list</code> <button onclick="navigator.clipboard.writeText('pm2 list')"></button></pre>

---

## **5. Common Issues and Troubleshooting**

### **SSH Key Issues**

- Ensure the SSH private key in GitHub secrets matches the public key in the `~/.ssh/authorized_keys` file on the VPS.
- Test the SSH connection locally:
    
    ```bash
    ssh -i ~/.ssh/id_rsa root@<host>
    ```

<pre><code class="language-bash">ssh -i ~/.ssh/id_rsa root@<host></code> <button onclick="navigator.clipboard.writeText('ssh -i ~/.ssh/id_rsa root@<host>')"></button></pre>

### **Permissions Issues**

- Ensure the project directory on the VPS has the correct permissions for the SSH user.
- Example:
    
    ```bash
    sudo chown -R root:root /var/www/your-project-name
    ```

<pre><code class="language-bash">sudo chown -R root:root /var/www/your-project-name</code> <button onclick="navigator.clipboard.writeText('sudo chown -R root:root /var/www/your-project-name')"></button></pre>

### **PM2 Issues**

- If the `pm2 restart` command fails, double-check the process name:
    
    ```bash
    pm2 restart your-project-name
    ```

<pre><code class="language-bash">pm2 restart your-project-name</code> <button onclick="navigator.clipboard.writeText('pm2 restart your-project-name')"></button></pre>

---

## **6. Running the Workflow**

Once everything is set up, push changes to the `main` branch. The GitHub Actions workflow will:

1. Build the application.
2. Deploy it to the VPS.

Monitor the workflow progress in the **Actions** tab of your GitHub repository.

---

#HOST ip
#USER root

#SSH_PRIVATE_KEY

This guide ensures a smooth deployment process for your Node.js application to DigitalOcean VPS using GitHub Actions. Let the team know if there are any questions or concerns!
