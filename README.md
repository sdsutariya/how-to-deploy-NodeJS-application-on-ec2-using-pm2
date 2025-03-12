# how to deploy NodeJS application on ec2 using pm2 with database setup

# Deploying a Node.js Application on AWS EC2 with PM2, Nginx, and a Database

This guide provides a step-by-step walkthrough to deploy a Node.js application on an AWS EC2 instance using PM2 for process management, Nginx as a reverse proxy, and a database (MySQL/PostgreSQL) for data storage. It is beginner-friendly and assumes no prior knowledge of server management.

## Prerequisites

Before proceeding, ensure you have:
- An **AWS EC2 instance** (Ubuntu preferred)
- A **Node.js** application
- **PM2** installed for process management
- **Nginx** installed as a reverse proxy
- A **database** (MySQL or PostgreSQL)
- A **domain name** (optional but recommended)

## Step 1: Connect to Your EC2 Instance

First, use SSH to connect to your EC2 instance:
```sh
ssh -i your-key.pem ubuntu@your-ec2-instance-ip
```

## Step 2: Update and Install Dependencies

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx git
```

## Step 3: Install Node.js and npm

Use Node Version Manager (NVM) to install Node.js:
```sh
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
source ~/.bashrc
nvm install --lts
```
Verify the installation:
```sh
node -v
npm -v
```

## Step 4: Install and Configure the Database

### MySQL Installation
```sh
sudo apt install mysql-server -y
sudo mysql_secure_installation
```
Create a new MySQL database and user:
```sh
sudo mysql
CREATE DATABASE myapp;
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON myapp.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### PostgreSQL Installation
```sh
sudo apt install postgresql postgresql-contrib -y
sudo -u postgres psql
```
Create a new PostgreSQL database and user:
```sql
CREATE DATABASE myapp;
CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE myapp TO myuser;
\q
```

## Step 5: Clone Your Application Repository

Navigate to the working directory and clone your app:
```sh
cd /var/www
sudo git clone https://github.com/your-repo.git
cd your-repo
npm install
```

Create an **.env** file for environment variables:
```sh
nano .env
```
Add the following:
```env
DB_HOST=localhost
DB_USER=myuser
DB_PASSWORD=mypassword
DB_NAME=myapp
PORT=3000
```
Save and exit (`CTRL + X`, then `Y`, then `ENTER`).

## Step 6: Start Your Application with PM2

Install PM2 globally:
```sh
npm install -g pm2
```
Start the application:
```sh
pm2 start app.js --name "myapp"
pm2 save
pm2 startup
```

To check the running process:
```sh
pm2 list
```
To restart the app:
```sh
pm2 restart myapp
```
To monitor logs:
```sh
pm2 logs myapp
```

## Step 7: Configure Nginx as a Reverse Proxy

Edit the Nginx configuration:
```sh
sudo nano /etc/nginx/sites-available/default
```

Replace the content with:
```nginx
server {
    listen 80;
    server_name api.domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save and exit (`CTRL + X`, then `Y`, then `ENTER`).

## Step 8: Restart Nginx

```sh
sudo systemctl restart nginx
sudo systemctl enable nginx
```

## Step 9: Connect Your Domain to AWS EC2

1. **Get your EC2 public IP**:
   ```sh
   curl http://checkip.amazonaws.com
   ```
2. **Go to your domain registrar (GoDaddy, Namecheap, etc.)** and update the DNS settings:
   - Create an **A Record** with `api.domain.com` pointing to your EC2 instance's IP.

3. **Wait for propagation** (can take up to 24 hours).

## Step 10: Secure with SSL (Optional)

```sh
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d api.domain.com
```
Renew SSL automatically:
```sh
sudo certbot renew --dry-run
```

## Step 11: Verify Deployment

Visit `http://api.domain.com` or `http://your-ec2-instance-ip` to confirm the application is running.

## Conclusion

Your Node.js application is now fully deployed on AWS EC2 with a running database, Nginx as a reverse proxy, PM2 for process management, and a custom domain with SSL security. 🎉

---

If this guide was helpful, please ⭐ **star** the repository and follow me on GitHub: [Sumit Sutariya](https://github.com/sdsutariya) 🚀

