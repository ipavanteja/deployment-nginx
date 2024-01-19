# Deployment-NGINX

## Setting Up Cloud DNS

- Go to the **Cloud DNS** section under **Network Services**.
- Click on ‘**Create a DNS Zone**’
  - Make zone type **public**
  - Add a Zone name (can be anything)
  - Keep DNSSEC off and hit create
  - Now you have to add a record set for the Domain
  - Click on ‘**Add Record set**’
  - Add your VMInstance’s **IPv4 address** in the IP section (leave everything else empty) and create.
  - After this add another record set with **www** in the DNS Name section, add the IP address again, and hit create.

Now wait for about 10 to 15 mins and then check with your **`<domain-name>:PORT`** number (as of now we haven’t enabled reverse proxying). You should be able to see your app.

Now we will get back to **reverse proxying:** (how to proxy the traffic from PORTs 80 & 443 to your App’s PORT)

- First just do (just for safety) -> **`apt update && apt upgrade`**
- Now do -> **`apt install nginx`**
- After installation, check the status of NGINX
  - **`systemctl status nginx`**
  - The Active should show active (running)
- If it is not active, you need to do -> **`systemctl start nginx`**
- Now check your app again -> it should show an nginx page unless you have put an html file in the **`/var/www/html`** Path.

## Setting Up NGINX Server Blocks

By Default all web servers will point out to **`/var/www/html`**. To override this rule to a custom path, we have to create a server block. Here our new path of the web server block will be -> **`/var/www/<domain.com>/html`**

- **`mkdir -p /var/www/<domain>.com/html`**
- **`nano index.html`**
- From the earlier path (`/var/www/<domain>.com/html`)
  - **`nano /etc/nginx/sites-available/<domain-name>`** (this will create a new file)

Now write the following in the file:

```nginx
server {
    listen 80;
    listen [::]:80;
    root /var/www/<example>.com/html;
    index index.html index.htm index.nginx-debian.html;
    server_name <example>.com www.<example>.com;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/<example>.com.access.log;
    error_log /var/log/nginx/<example>.com.error.log;
}
```

- Add a soft link from sites-enabled to sites-available
  - **ln -s /etc/nginx/sites-available/<domain-name> /etc/nginx/sites-enabled/**
- Now save the file. We also need to test the file for possible errors. For that ->
  - **nginx -t**
- If everything went well, the reply should say -> test is successful. If not, re-check what you wrote in the file.
  - **systemctl restart nginx**
- Now check your domain again. If the server block is working properly, now it should show your app’s page.


{ you have overridden the web server’s path to a custom path which points to your app. }

## Reverse proxying node app with NGINX
- **nano etc/nginx/sites-available/<domain-name>**
- We have to put the location tag in the file that we wrote earlier. Now write this in the location tag ->
```nginx
location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_cache_bypass $http_upgrade;
}
```
- Now you can remove the **PORT 3000** (or whatever your PORT is) from the Firewall Rules section of the VPC networks.
- Now your app is only accessible via the domain, without any port. It won’t be accessible via the PORT number.

## Redirecting IP to domain

```nginx
server {
    listen 80;
    listen 8081;
    server_name <IP address>;
    return 301 https://<domain-name>$request_uri;
}
```
We need to add this server block to the end of the nginx configuration file to redirect IP to the domain.

## Blocking IP address (dropping IP address access)
```nginx
server {
    listen 80;
    server_name <IP address>;
    return 444;
}
```

Add the following at the end of the file to drop the IP.

Don't forget to restart nginx
- **systemctl restart nginx**
