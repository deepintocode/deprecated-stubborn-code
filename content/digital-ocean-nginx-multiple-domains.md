---
title: "Multiple Static Websites on a Digital Ocean Droplet using Nginx (+ SSL Certification)"
date: 2018-07-06T10:46:06+01:00
draft: false
type: "post"
---

## HostGator to Digital Ocean

I created my first website about six years ago and hosted it on a shared hosting plan through HostGator. Minimal setup, minimal configuration, a high price and a desire to learn more about servers led me to switch from HostGator to Digital Ocean. While I am happy with my decision, there hasn't a lack of frustrating and confusing moments.

## Things you take for granted

Sometimes, when you are used to having certain things, you don't even think about them. That's what happened to me when I switched.

*Now, I need to sort out an email server...*

*Now, I need a way to configure multiple domains on the same IP*

*Now, I need to acquire an SSL certificate*

Granted, the SSL issue is not such a big problem nowadays.
Also, I ended up not creating an email server, but that's a story for another time.

This is a guide for multiple domains on a single Digital Ocean droplet and I'm assuming that you have acquired your desired domain names, successfully setup your Digital Ocean droplet and are familiar with the Linux terminal.

### 1. Point to the Digital Ocean nameservers from your Domain Registrar

Since this differs based on the registrar you used, you can find a detailed guide [on Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars)

### 2. Add the domain and create A records on Digital Ocean

On the Digital Ocean page, click on the create domain option, found on the top navigation bar. You will be faced with an input field to add your domain name. After, you need to create A records for your domain name. There is a debate regarding the best choice between A and C names. Based on my research, I decided that ultimately, it won't make a lot of difference. Especially for a website of this caliber. 

Depending on how you want your domain name to appear, you have a few choices. It's quite common to have a domain with both the plain name, such as myamazingwebsite.com and the www subdomain, such as www.myamazingwebsite.com and have one redirect to the other for SEO purposes. Which one you decide to keep is your choice. On the Digital Ocean side you would register both as shown below. Nginx will handle the redirection.

![Non-www Domain](/img/do-nginx-non-www.png)
![www Domain](/img/do-nginx-www.png)

And that's it for the Digital Ocean part.

### 3. Add the Nginx service on the Digital Ocean droplet

SSH into your Digital Ocean droplet and install the Nginx service.

```bash
sudo apt-get update
sudo apt-get install nginx
```

### 4. Transfer your website's files to the Digital Ocean droplet

By default your website files go into the www directory, which is located at /var/www. Create a directory that will contain your website's files, such as /var/www/myamazingwebsite.com and transfer the files. You can either use an FTP client of your choice, such as [FileZilla](https://filezilla-project.org/), or using the terminal:

```bash
scp -r * root@100.100.100.100:/var/www/myamazingwebsite.com
```

Be sure to replace 100.100.100.100 with the IP of your droplet and myamazingwebsite.com with your domain.

### 5. Configure Nginx

Nginx's magic happens at /etc/nginx.

nginx.conf is the general configuration, but we won't deal with that. The directories we need are 'sites-available' and 'sites-enabled'. The typical Nginx workflow is creating the configuration file in 'sites-available' and having a soft link in 'sites-enabled' that references the aforementioned configuration file. That means that you can keep all your sites configuration files in 'sites-available', but they won't be live until you create that link in 'sites-enabled'.

Create a file in 'sites-available'. For organisational purposes it's better to name these files after your website domain names.

```bash
cd /etc/nginx/sites-available
sudo nano myamazingwebsite.com
```

Inside the file add the following:

```
server {
  listen 80;
  listen [::]:80;
  root /var/www/myamazingwebsite.com;
  index index.html;
  server_name myamazingwebsite.com www.myamazingwebsite.com;
  return  301 http://$server_name$uri;
  location / {
    try_files $uri $uri/ =404;
  }
}
```

```
listen 80;
listen [::]:80;
```

These two lines tell Nginx to listen for IPv4 and IPv6 connections on port 80.

```
root /var/www/myamazingwebsite.com;
```

Which content to serve.

```
index index.html
```

What's the entry point.

```
server_name myamazingwebsite.com www.myamazingwebsite.com;
```

The server names it's looking for.

```
return  301 http://$server_name$uri;
```

Redirect www to non-www.

```
location / {
    try_files $uri $uri/ =404;
  }
```

Show a 404 page if the URL does not exist.

Now, you need to create the link in 'sites-enabled'

This is done by the following command.

```
ln -s /etc/nginx/sites-available/myamazingwebsite.com /etc/nginx/sites-enabled/myamazingwebsite.com
```

Remember to change the domain name!

Now, if you restart the Nginx service, you should be able to see your website live.

```
sudo systemctl restart nginx
```

### 6. Enable HTTPS

If you are only now starting your Internet journey, consider yourself lucky. It's only recent that you can get a free SSL certificate in a couple of minutes typing less than ten Shell commands.

Run the commands below and you will have installed Certbot. [Let's Encrypt's](https://letsencrypt.org/) automated solution for free SSL certificates. 

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

The command below will do all the work for you

```
sudo certbot --nginx
```

The bot will ask you which domains you want the certificate for. Then, it will proceed to ask if you want it to include the option to redirect HTTP traffic to HTTPS. I chose to do that and I did not face any issues.

After that's done, you have a working Nginx configuration for one static website with SSL support!!

While I mentioned multiple domains in the title, we just managed one, right? Well, if you need to add more, you just need to repeat steps 1 to 6. Nginx is a very powerful service, but it can get a bit complicated if you need to add reverse proxy configuration, in order to deal with websites that need to work with the back-end. That's another post on its own though...

**Enjoy your new shiny websites!**




