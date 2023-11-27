# Node.Js-App-Deployment-running-NGINX-Webserver
> Steps to Deploy a Node.js app to a VPS, Using PM2 manager to maintain app live persistence, Setting up NGINX as a reverse proxy to your node app and Installing an SSL Cert from LetsEncrypt Certbot.


# Node.js App Deployment


## 1. Sign up & purchase a Linux based VPS from your preferred source. It could be Digital Ocean, Hostinger, Contabo, Ovhcloud etc.


## 2. Access your VPS via ssh and make sure all pre-installed packages are updated && upgraded.

TIPS:
i.	| Create a new SSH user => Add user to sudoer list => Migrate to new user account. (This would help in ensuring proper organization and better management with your work, especially if you would be running & managing multiple services(apps, webservers, domains etc.).
ii.	| Generate an SSH Key pair: You could do this using "PuTTYgen tool" which comes pre-packed with the "PuTTY tool". Save your private key securely where you can recover it for reuse on consequent logins, then add your public key to your VPS. With this your server only allows access to anyone who provides it with the corresponding private key on login. This is 100% vital security.
iii.	| You could further tighten security by limiting login access from anywhere to specific hosts or IPs.
 

For this tutoring and test purpose i will be using the root user.


## 3. Install Node/NPM
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
There are a few ways to get your files onto the server either by using a Git Repo having your project files or an SSH SFTP, I would suggest using Git a git repo for ease.
```
git clone yourproject.git
```

### 5. Install dependencies and test app
```
cd yourproject
npm install
npm start (or whatever your start command)
# stop app
ctrl+C
```
## 6. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start app (or whatever your file name)

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```
### You should now be able to access your app using your IP and port. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 7. Setup ufw firewall rules
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 8. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:5000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check/Test NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 80) and see your app. Now let's add a domain

## 9. Adding & Pointing your domain to your VPS by DNS Propagation 
Purchase/register a domain from your preffered registrar (Digital ocean, namecheap etc.). Update your DNS A & CNAME records to carry your vps IP. It may take a very short time to propogate after which you can now access your service via the domain name which would now be running on port 80 (HTTP).

## 10. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```

NOTE: There are other techniques for adding SSL to your domain and one way to go about it is by pointing your nameservers to Cloudflare or any CDN provider, update your DNS records on there and have them manage the hassles, while giving you durable certificates for free and somewhat of a click-through install process.

Now visit https://yourdomain.com and you should see your Node app running securely.
