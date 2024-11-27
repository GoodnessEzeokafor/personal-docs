### Deploying a node application with PM2 on Digital Ocean

## Setting Up a Droplet

1. **Create a Droplet on DigitalOcean**
   - Visit the [DigitalOcean dashboard](https://www.digitalocean.com/) and create a new droplet. Be sure to add your SSH key during the setup process.
   - To view your SSH key, run:  
     ```bash
     cat ~/.ssh/id_rsa.pub
     ```

2. **SSH Into Your Droplet**
   - SSH into your droplet:  
     ```bash
     ssh root@droplet_ip_address
     ```

3. **Create a New User**
   - Run the following command to create a new user:  
     ```bash
     adduser username
     ```

4. **Grant Admin Privileges**
   - Add the user to the `sudo` group:  
     ```bash
     usermod -aG sudo username
     ```

5. **Set Up a Firewall**
   - Allow OpenSSH through the firewall:  
     ```bash
     ufw allow OpenSSH
     ```
   - Enable the firewall:  
     ```bash
     ufw enable
     ```

6. **Transfer SSH Key for New User**
   - Use the following command to enable SSH access for the new user:  
     ```bash
     rsync --archive --chown=username:username ~/.ssh /home/username
     ```
   - Replace `username` with the new user's name.

7. **SSH With Your New User**
   - Exit the root session and SSH back in with your new user:  
     ```bash
     ssh username@your_server_ip
     ```

8. **Further Reference**
   - [DigitalOcean: Initial Server Setup with Ubuntu](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu)

---

## Setting Up Nginx on Ubuntu 22.04 (DigitalOcean)

1. **SSH Into Your Server**
   - Log in using your newly created user:  
     ```bash
     ssh username@your_server_ip
     ```

2. **Update and Upgrade Ubuntu**
   - Run the following commands to update and upgrade the system:  
     ```bash
     sudo apt update
     sudo apt upgrade
     ```

3. **Install Nginx**
   - Install Nginx:  
     ```bash
     sudo apt install nginx
     ```

4. **Adjust the Firewall**
   - Allow Nginx through the firewall:  
     ```bash
     sudo ufw allow 'Nginx HTTP'
     ```

5. **Check Firewall Status**
   - Verify the firewall settings:  
     ```bash
     sudo ufw status
     ```

   - Expected output:
     ```bash
     Status: active

     To                         Action      From
     --                         ------      ----
     OpenSSH                    ALLOW       Anywhere
     Nginx HTTP                 ALLOW       Anywhere
     OpenSSH (v6)               ALLOW       Anywhere (v6)
     Nginx HTTP (v6)            ALLOW       Anywhere (v6)
     ```

6. **Check Nginx Status**
   - Verify if Nginx is running:  
     ```bash
     systemctl status nginx
     ```

7. **Further Reference**
   - [DigitalOcean: How to Install Nginx on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04)

---

## Installing Node.js

1. **Install NVM (Node Version Manager)**
   - Visit the [NVM GitHub page](https://github.com/nvm-sh/nvm) for installation instructions.
   - After downloading NVM using curl or wget, add the following to your `.bashrc` or `.zshrc` to include NVM in your PATH:
     ```bash
     export NVM_DIR="$HOME/.nvm"
     [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
     [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
     ```

2. **Install Node.js (LTS Version)**
   - Install the latest LTS version of Node.js:  
     ```bash
     nvm install v22.11.0
     ```

---

## Installing PM2

1. **Install PM2 Globally**
   - Install PM2 to manage Node.js processes:  
     ```bash
     npm install -g pm2
     ```

2. **PM2 Setup on PM2.io**
   - Log into [PM2](https://app.pm2.io) and create a bucket for your application.
   - Link your PM2 to PM2.io with:  
     ```bash
     pm2 link {{privateKey}} {{secretKey}}
     ```

3. **Unlink PM2 from PM2.io**
   - To unlink PM2 from PM2.io, use:  
     ```bash
     pm2 link delete
     ```

4. **Install PM2 Server Monitor (Optional)**
   - Install the PM2 server monitoring tool:  
     ```bash
     pm2 install pm2-server-monit
     ```

---

## Setting Up Your Application

1. **Create Project Directory**
   - Create and navigate to your project folder:  
     ```bash
     mkdir your-app-name && cd your-app-name
     ```

2. **Initialize Git Repository**
   - Initialize a Git repository in your project folder:  
     ```bash
     git init
     ```

3. **Setup Git Remote**
   - Set up the Git remote with your repository:  
     ```bash
     git remote set-url origin https://$GITHUB_ACTOR:{{GH_TOKEN}}git@github.com:github-user/{{repo-name}}
     ```

4. **Add Git to Known Hosts**
   - Add GitHub to your `known_hosts`:  
     ```bash
     mkdir -p ~/.ssh
     ssh-keyscan github.com >> ~/.ssh/known_hosts
     ```

5. **Checkout and Pull the Relevant Branch**
   - Checkout to the desired branch:  
     ```bash
     git checkout {{branch-name}}
     ```
   - Pull the latest changes from the branch:  
     ```bash
     git pull origin {{branch-name}} -f
     ```

6. **Install Yarn**
   - Install Yarn package manager:  
     ```bash
     npm install --global yarn
     ```

7. **Install Dependencies**
   - Run the following to install all dependencies:  
     ```bash
     yarn install
     ```

8. **Build the Application**
   - Build the application:  
     ```bash
     yarn build
     ```

9. **Start the Application with PM2**
   - Start the app using PM2:  
     ```bash
     pm2 start npm --name {{app-name}} -- start
     ```

10. **Save PM2 Process**
    - Save the process so it will restart on reboot:  
      ```bash
      pm2 save
      ```

11. **Ensure Automatic Restart on Reboot**
    - Set up PM2 to start automatically on server reboot:  
      ```bash
      pm2 startup
      ```

12. **View PM2 Logs**
    - View logs for your application:  
      ```bash
      pm2 logs {{app-name}}
      ```

13. **Setup GitHub Actions**
    - Here's a sample GitHub Actions workflow for deploying your app:  
    ```yaml
    name: Node.js CI

    on:
      push:
        branches: ["production"]
      pull_request:
        branches: ["production"]
      workflow_dispatch:

    jobs:
      build:
        runs-on: ubuntu-20.04
        steps:
          - name: Checkout 🛎️
            uses: actions/checkout@v3

          - name: Set up Node.js
            uses: actions/setup-node@v3
            with:
              node-version: "22.11.0"

          - name: Set up Git credentials
            run: |
              git config --global user.name "{{full name}}"
              git config --global user.email "{{ email }}"
              git remote set-url origin https://$GITHUB_ACTOR:${{ secrets.GH_TOKEN }}@github.com:github-user/{{repo-name}}

          - name: Deploy NodeJS app
            uses: appleboy/ssh-action@v0.1.7
            with:
              host: ${{ secrets.SSH_HOST }}
              key: ${{ secrets.SSH_KEY }}
              username: ${{ secrets.SSH_USERNAME }}
              script: |
                eval $(ssh-agent -s)
                ssh-add <(echo "${{ secrets.SSH_KEY }}")
                export NVM_DIR="$HOME/.nvm"
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                nvm use 22.11.0
                cd ~/gasrail/landing-page
                git pull git@github.com:github-user/{{repo-name}} production -f
                yarn install --ignore-engines
                yarn build
                pm2 update
                pm2 reload {{app-name}}
                echo 'Deployment successful to DigitalOcean'
    ```

---

## Subdomain Configuration

1. **Add an A Record to Your Domain**
   - Add an A record for your subdomain in your domain registrar (e.g., Namecheap, Cloudflare).

2. **Create Directory for Subdomain**
   - Create a directory for your subdomain:  
     ```bash
     sudo mkdir /var/www/html/domain.com
     ```

3. **Set Permissions**
   - Set the correct permissions:  
     ```bash
    sudo chown -R {{user}}:www-data /var/www/html/domain.com
     ```

4. **Create Nginx Configuration File**
   - Create a new configuration file for your subdomain:  
     ```bash
     sudo nano /etc/nginx/sites-available/{{subdomain}}
     ```

5. **Configure the Nginx File**
   - Add the following configuration:  
     ```nginx
    server {  # Ensure Nginx listens on port 80 for HTTP
    server_name www.domain.com domain.com;

      location /{
                #try_files $uri $uri/ =404;
                # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.

              # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        }
    }
     ```

6. **Enable the Site**
   - Create a symbolic link to enable the site:  
     ```bash
     sudo ln -s /etc/nginx/sites-available/{{subdomain}} /etc/nginx/sites-enabled/
     ```

7. **Test Nginx Configuration**
   - Test the Nginx configuration:  
     ```bash
     sudo nginx -t
     ```

8. **Restart Nginx**
   - Restart Nginx to apply the changes:  
     ```bash
     sudo systemctl restart nginx
     ```

9. **Further Reference**
   - [How To Serve NGINX Subdomains or Multiple Domains](https://adamtheautomator.com/nginx-subdomain/)


---

## Setting Up SSL with Certbot (Nginx)

1. **Install Certbot**
   - First, install Certbot and the Nginx plugin:  
     ```bash
     sudo apt install certbot python3-certbot-nginx
     ```

2. **Obtain SSL Certificate**
   - Run the following command to obtain an SSL certificate for your domain:  
     ```bash
     sudo certbot --nginx -d yourdomain.com
     ```
   - Replace `yourdomain.com` with your actual domain. Certbot will automatically configure Nginx to use SSL.

3. **Verify the SSL Installation**
   - Certbot will attempt to automatically configure Nginx to redirect HTTP to HTTPS. After the installation, check that your site is now accessible over HTTPS by visiting `https://yourdomain.com`.

4. **Further Reference**
   - [DigitalOcean: How To Secure Nginx with Let's Encrypt on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)