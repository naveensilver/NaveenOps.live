# Portfolio Setup

Certainly! Here are the detailed steps to use a domain purchased from [GoDaddy](https://www.godaddy.com/) with your [AWS](https://aws.amazon.com/) account:

### Step 1: Log in to GoDaddy and Get Domain Details
1. Log in to your GoDaddy account.
2. Go to the "My Products" page.
3. Find your domain and click on "DNS" or "Manage DNS" to view the DNS settings.

### Step 2: Create a Hosted Zone in AWS Route 53
1. Log in to your AWS Management Console.
2. Navigate to the [Route 53](https://aws.amazon.com/route53/) service.
3. Click on **Hosted zones** in the left-hand menu.
4. Click the **Create hosted zone** button.
5. Enter your domain name (e.g., example.com) and click **Create**.

### Step 3: Note the AWS Name Servers
1. After creating the hosted zone, click on it to view its details.
2. Note the list of **Name Servers (NS)** provided by AWS. These will be used to update the DNS settings in GoDaddy.

### Step 4: Update DNS Settings in GoDaddy
1. Go back to the DNS settings page for your domain in GoDaddy.
2. Find the section for **Nameservers** and click on **Change**.
3. Select **Enter my own nameservers (advanced)**.
4. Enter the AWS name servers you noted earlier.
5. Save the changes.

### Step 5: Add DNS Records in AWS Route 53
1. Go back to your hosted zone in AWS Route 53.
2. Click on **Create record** to add DNS records (e.g., A, CNAME, MX, etc.) as needed for your domain.
   - For example, to point your domain to an [EC2](https://aws.amazon.com/ec2/) instance, you would create an A record with the public IP of the instance.

### Example: Adding an A Record
```markdown
- **Record name:** (Leave blank for root domain or specify subdomain)
- **Record type:** A
- **Value:** Public IP of your EC2 instance
- **TTL (Seconds):** 300 (default)
```

### Step 6: Verify the Setup
1. It may take some time for DNS changes to propagate. You can use tools like [WhatsMyDNS](https://www.whatsmydns.net/) to check the propagation status.
2. Once propagated, your domain should now be pointing to the resources specified in your AWS Route 53 hosted zone.

### Example: Adding a CNAME Record
```markdown
- **Record name:** www
- **Record type:** CNAME
- **Value:** example.com (or another domain)
- **TTL (Seconds):** 300 (default)
```

### Additional Tips
- Ensure that your AWS resources (like EC2 instances) have the necessary security group settings to allow traffic.
- You can also set up SSL certificates using [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/) for HTTPS support.

By following these steps, you should be able to successfully use your GoDaddy domain with your AWS account. If you encounter any issues, both GoDaddy and AWS have extensive documentation and support services to assist you.

_______________
                    
# Nginx Server Setup 

Setting up an Nginx server on Ubuntu 20.04 and configuring it to serve a website with SSL certificates managed by Certbot. However, there are a few corrections and improvements to be made in your script. Here's a revised version of your setup:

1. **Correct the typo in the `sudo` command.**
2. **Use `echo` to create your HTML file instead of opening an editor. This makes the script non-interactive and easier to automate.**
3. **Ensure proper Nginx configuration and SSL setup.**

Here's the corrected script:

```bash
# Update system repositories and packages
sudo apt update && sudo apt install -y nginx

# Install Certbot for SSL certificate management
sudo apt install -y certbot python3-certbot-nginx

# Create website directory and set permissions
sudo mkdir -p /var/www/naveenops.live/html
sudo chown -R $USER:$USER /var/www/naveenops.live/html
sudo chmod -R 755 /var/www/naveenops.live

# Create the index.html file
echo "<html>
    <head>
        <title>Welcome to naveenops.live!</title>
    </head>
    <body>
        <h1>Success! The naveenops.live server block is working!</h1>
    </body>
</html>" | sudo tee /var/www/naveenops.live/html/index.html

# Create Nginx server block configuration
echo "server {
    listen 80;
    listen [::]:80;

    root /var/www/naveenops.live/html;
    index index.html index.htm index.nginx-debian.html;

    server_name naveenops.live www.naveenops.live;

    location / {
        try_files \$uri \$uri/ =404;
    }
}" | sudo tee /etc/nginx/sites-available/naveenops.live

# Enable the site by creating a symbolic link
sudo ln -s /etc/nginx/sites-available/naveenops.live /etc/nginx/sites-enabled/

# Test Nginx configuration and restart the service
sudo nginx -t
sudo systemctl restart nginx

# Obtain SSL certificate
sudo certbot certonly --agree-tos --email turkapallynaveen@gmail.com --manual --preferred-challenges=dns -d *.naveenops.live --server https://acme-v02.api.letsencrypt.org/directory

# If individual SSL certificates are needed, use the following command:
# sudo certbot --nginx
```

### Explanation:
- **`echo` and `tee`:** This combination is used to write the HTML content directly into the file as root, which avoids having to manually edit the file with `nano`.
- **Nginx configuration:** The configuration is piped directly into the appropriate file.
- **SSL Certificate:** The script includes a command to obtain a wildcard SSL certificate. If individual certificates are needed for each subdomain, you can use `sudo certbot --nginx` to interactively obtain and configure them.

This script should be run with appropriate privileges and assumes that you have DNS properly configured for `*.naveenops.live` to point to your server. Make sure to replace `naveenops.live` with your actual domain and adjust email addresses as necessary.



## To Automate Above Process using Script 

To further automate the setup of your Nginx server and SSL certificates on Ubuntu 20.04, you can create a shell script that requires minimal interaction. This script will handle everything from updating the system, installing necessary packages, setting up the website directory, configuring Nginx, and obtaining SSL certificates using Certbot.

Here’s an enhanced version of the script that includes more automation:

```bash
#!/bin/bash

# Define variables
DOMAIN="naveenops.live"
EMAIL="turkapallynaveen@gmail.com"
WEB_ROOT_DIR="/var/www/${DOMAIN}/html"

# Update system repositories and packages
sudo apt update && sudo apt install -y nginx certbot python3-certbot-nginx

# Create website directory and set permissions
sudo mkdir -p ${WEB_ROOT_DIR}
sudo chown -R $USER:$USER ${WEB_ROOT_DIR}
sudo chmod -R 755 /var/www/${DOMAIN}

# Create the index.html file
cat <<EOF | sudo tee ${WEB_ROOT_DIR}/index.html
<html>
    <head>
        <title>Welcome to ${DOMAIN}!</title>
    </head>
    <body>
        <h1>Success! The ${DOMAIN} server block is working!</h1>
    </body>
</html>
EOF

# Create Nginx server block configuration
cat <<EOF | sudo tee /etc/nginx/sites-available/${DOMAIN}
server {
    listen 80;
    listen [::]:80;

    root ${WEB_ROOT_DIR};
    index index.html index.htm index.nginx-debian.html;

    server_name ${DOMAIN} www.${DOMAIN};

    location / {
        try_files \$uri \$uri/ =404;
    }
}
EOF

# Enable the site by creating a symbolic link
sudo ln -s /etc/nginx/sites-available/${DOMAIN} /etc/nginx/sites-enabled/

# Test Nginx configuration and restart the service
sudo nginx -t && sudo systemctl restart nginx

# Obtain SSL certificate
sudo certbot --nginx --agree-tos --redirect --hsts --staple-ocsp --email ${EMAIL} -d ${DOMAIN} -d www.${DOMAIN}

echo "Setup complete. Your Nginx server is now configured and secured with SSL."
```

### Enhancements in this script:
- **Variables:** The script uses variables for domain and email to make it easier to reuse and modify.
- **Heredoc for file creation:** This approach is used for creating configuration files, making it easier to manage multiline strings.
- **Automated SSL configuration:** The `certbot` command is configured to automatically install the certificate for Nginx, enable HTTPS redirection, HTTP Strict Transport Security (HSTS), and OCSP Stapling.
- **Final message:** A completion message is echoed at the end to confirm that the setup was successful.

### Usage:
1. Save this script as `setup_nginx.sh`.
2. Make it executable with `chmod +x setup_nginx.sh`.
3. Run the script using `sudo ./setup_nginx.sh`.

This script is fully automated and should be run with root privileges to ensure it can perform all operations without manual intervention. Make sure to replace the domain and email variables with your actual details.
