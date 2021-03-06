#### Configuring HTTPS Certificate

Now let’s protect our application with a nice HTTPS certificate provided by [Let’s Encrypt](https://letsencrypt.org/).

Setting up HTTPS has never been this easy. And better, we can get it for free nowadays. They provide a solution called **certbot** which takes care of installing and renewing the certificates for us. It’s very straightforward:

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

Now install the certs:

```
sudo certbot --nginx
```

Just follow the prompts. When asked about:

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
```

Choose `2` to redirect all HTTP traffic to HTTPS.

With that the site is already being served over HTTPS:

![HTTPS](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-7/https.png)

Setup the auto renew of the certs. Run the command below to edit the crontab file:

```
sudo crontab -e
```

Add the following line to the end of the file:

```
0 4 * * * /usr/bin/certbot renew --quiet
```

This command will run every day at 4 am. All certificates expiring within 30 days will automatically be renewed.

* * *