# Deploy-Multiple-Static-Website-in-One-VPS

## ✅ Step 1: Point Domain/Subdomain to VPS
Go to your domain provider account. At first you need to point your domain /subdomain to your VPS. Then, you would create a `A` record for `VPS_IPv4` and a `AAA` record for `VPS_IPv6`

| Type | Name | Target | Proxy |
| --- | --- | --- | --- |
| A | `domain1` | `YOUR_VPS_IPv4` | ✅ Proxied |
| A | `domain2` | `YOUR_VPS_IPv4` | ✅ Proxied |
| AAA | `domain1` | `YOUR_VPS_IPv6` | ✅ Proxied |
| AAA | `domain2` | `YOUR_VPS_IPv6` | ✅ Proxied |

`Note:` If you used main domain you need to change the Name. Used `@` for the `IPv4` and used `www` for the `IPv6`

## ✅ Step 2: Login Into VPS:

```cmd
ssh root@your-vps-ip
```

## ✅ Step 3: Install Nginx

```cmd
sudo apt update
sudo apt install nginx -y
```

## ✅ Step 4: Add SSL (Recommended)

### 🔹 To add SSL install Certbot

```cmd
apt install certbot python3-certbot-nginx -y
```

### 🔹 Then generate SSL for all domain or subdomain

```cmd
certbot --nginx -d your_domain_name
```

## ✅ Step 5: Configure Nginx for multiple sites

### 🔹 Single website configuration:

### 🔹 Step-1 : Create directories on VPS:

```cmd
mkdir -p /var/www/your_domain_name
chown -R $USER:$USER /var/www/your_domain_name
```

`Note:` Recommendation to use domain name for clarity. You can used any name

### 🔹 Step-2 : Upload all:

Open New CMD then, using `cd` go to the the root directory, where your project folder are stored. Then  run this command:

```cmd
scp -r your_folder_name/* root@your_vps_ip:/var/www/your_domain_name/
```

`Note:` Recommendation to use domain name for clarity. You can used any name

### 🔹 Step-3 : Configure Nginx server blocks

```cmd
sudo vim /etc/nginx/sites-available/your_domain_name
```

```cmd
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name your_domain_name;

    # Redirect to HTTPS
    return 301 https://$host$request_uri;
}

# Main HTTPS Server Block
server {
    listen 443 ssl;
    server_name your_domain_name;

    root /var/www/your_domain_name_or_folder_name;
    index index.html index.htm;

    ssl_certificate /etc/letsencrypt/live/your_domain_name/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_domain_name/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Correct MIME types for static files
    location / {
        try_files $uri $uri/ =404;
    }

    # Ensure CSS and JS load with correct MIME types
    location ~* \.(?:css|js|jpg|jpeg|png|gif|ico|svg|ttf|woff|woff2|eot)$ {
        try_files $uri =404;
        access_log off;
        expires 30d;
        add_header Cache-Control "public";
    }

    # Allow access to static/ folder
    location /static/ {
        alias /var/www/your_domain_name_or_folder_name/static/;
    }
}
```

`Note:` Must be change the project directory file and folder name

### 🔹 Step-4 : Set Permissions

```cmd
sudo chown -R www-data:www-data /var/www/your_domain_or_folder_name
```

### 🔹 Step 5: Enable the sites

```bash
sudo ln -s /etc/nginx/sites-available/company.mrshakil.space /etc/nginx/sites-enabled/
```

`Note:` Here `your_domain_name` must be the same as the Nginx config sites-available file name.

### 🔹 Step 6: Reload and Restart the server

```bash
sudo nginx -t
```

```bash
sudo systemctl reload nginx
```

### 🔹`Same way Configure other website that you want to host`