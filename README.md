# Deployment-nginx

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
}
```

- Add a soft link from sites-enabled to sites-available
  - **ln -s /etc/nginx/sites-available/<domain-name> /etc/nginx/sites-enabled/**
- Now save the file. We also need to test the file for possible errors. For that ->
  - **nginx -t**
- If everything went well, the reply should say -> test is successful. If not, re-check what you wrote in the file.
  Then ->
  - systemctl restart nginx
