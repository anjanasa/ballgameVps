# VPS Setup Guide for Ball Game Application

## Overview

This guide will help you deploy your Node.js application to a VPS and access it from the internet.

## Server Configuration

### Current Setup

- **Port**: 4000 (can be changed via `PORT` environment variable)
- **Static Files**: Served from root directory
- **Socket.IO**: Integrated with Express server
- **Database**: MySQL (configured in server.js)

## Step-by-Step VPS Deployment

### 1. Connect to Your VPS

```bash
ssh root@YOUR_VPS_IP
# or
ssh username@YOUR_VPS_IP
```

### 2. Install Required Software

#### Install Node.js (v18 or later recommended)

```bash
# Update package manager
sudo apt update

# Install Node.js (using NodeSource repository)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

#### Install MySQL (if not already installed)

```bash
sudo apt install mysql-server
sudo mysql_secure_installation
```

### 3. Upload Your Application

#### Option A: Using Git (Recommended)

```bash
cd /var/www  # or your preferred directory
git clone https://github.com/anjanasa/ballgameVps.git
cd ballgameVps
npm install
```

#### Option B: Using SCP/FTP

```bash
# From your local machine
scp -r /path/to/your/project/* username@YOUR_VPS_IP:/var/www/ballgame/
```

### 4. Configure Environment Variables

Create a `.env` file (optional, for production):

```bash
nano .env
```

Add:

```
PORT=4000
DB_HOST=localhost
DB_USER=your_db_user
DB_PASS=your_db_password
MYSQL_DB=nadeera_game
```

### 5. Set Up MySQL Database

```bash
sudo mysql -u root -p
```

```sql
CREATE DATABASE nadeera_game;
CREATE USER 'remoteuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON nadeera_game.* TO 'remoteuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 6. Install PM2 (Process Manager)

```bash
npm install -g pm2
```

### 7. Start Your Application

```bash
# Navigate to project directory
cd /var/www/ballgameVps

# Start with PM2
pm2 start server.js --name "ballgame"

# Or using npm
npm start

# Save PM2 configuration
pm2 save

# Set PM2 to start on boot
pm2 startup
```

### 8. Configure Firewall

```bash
# Allow port 4000
sudo ufw allow 4000/tcp

# Or if using specific IP
sudo ufw allow from YOUR_CLIENT_IP to any port 4000

# Enable firewall
sudo ufw enable
```

## Accessing Your Application

### Using IP Address

Once deployed, access your application at:

```
http://YOUR_VPS_IP:4000
```

**Example**: `http://192.168.1.100:4000`

### Using Domain Name (Optional)

#### If you have a domain:

1. **Point your domain to VPS IP**

   - Add an A record in your DNS settings
   - Domain: `ballgame.yourdomain.com` → `YOUR_VPS_IP`

2. **Install Nginx as Reverse Proxy** (Recommended)

```bash
sudo apt install nginx
```

Create Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/ballgame
```

Add:

```nginx
server {
    listen 80;
    server_name ballgame.yourdomain.com;  # or YOUR_VPS_IP

    location / {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/ballgame /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Then access at: `http://ballgame.yourdomain.com` (port 80, no need for :4000)

### For HTTPS (SSL Certificate)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d ballgame.yourdomain.com
```

Then access at: `https://ballgame.yourdomain.com`

## Update Client Configuration

After deploying to VPS, you have two options:

### Option 1: Auto-detect (Current setup)

The script.js automatically detects the current host and port. No changes needed if you access via the same URL.

### Option 2: Hardcode VPS URL

In `script.js`, update the socket connection:

```javascript
const socket = io("http://YOUR_VPS_IP:4000");
// or
const socket = io("https://ballgame.yourdomain.com");
```

## Useful Commands

### Check if server is running

```bash
pm2 status
pm2 logs ballgame
```

### Restart application

```bash
pm2 restart ballgame
```

### Stop application

```bash
pm2 stop ballgame
```

### View logs

```bash
pm2 logs ballgame
```

### Monitor resources

```bash
pm2 monit
```

## Troubleshooting

### Check if port is open

```bash
sudo netstat -tulpn | grep 4000
# or
sudo ss -tulpn | grep 4000
```

### Check firewall status

```bash
sudo ufw status
```

### Test database connection

```bash
mysql -u remoteuser -p -h localhost nadeera_game
```

### Check Node.js application logs

```bash
pm2 logs
# or
tail -f /var/log/pm2.log
```

## Security Recommendations

1. **Change default passwords** in server.js
2. **Use environment variables** for sensitive data
3. **Set up firewall rules** to restrict access
4. **Use HTTPS** in production
5. **Regular updates**: `sudo apt update && sudo apt upgrade`
6. **Database security**: Don't expose MySQL port 3306 publicly

## URL Summary

- **Local Development**: `http://localhost:4000`
- **VPS with IP**: `http://YOUR_VPS_IP:4000`
- **VPS with Domain (HTTP)**: `http://yourdomain.com`
- **VPS with Domain (HTTPS)**: `https://yourdomain.com`

## Need Help?

- Check PM2 logs: `pm2 logs ballgame`
- Check Nginx logs: `sudo tail -f /var/log/nginx/error.log`
- Check system logs: `sudo journalctl -u nginx`
