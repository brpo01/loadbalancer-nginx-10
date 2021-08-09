# **Load Balancer Solution With Nginx & Securing your Website using SSL/TLS**

In this project, i'll be covering how to configure a load balancer in nginx. Nginx is a high web server and reverse proxy server. It is a popular web service used for serving content of whatever sort on the internet/web. Also what is a load-balancing?. Loadbalancing refers to the process of distributing network/traffic between several servers to recieve requests and send back responses to clients, and this is based on a lot of factors/use cases which have to be considered such as the no. of servers, loadbalancing algorithms e.t.c. A very practical example will be to think about how google can keep their search engines running and serve content to users with minimum downtime, this is possible based on the loadbalancing concepts that allows distribution of networ to the millions of servers they have running.

Next thing we'll be implementing after the loadbalancer is how to secure connections to a website using SSL/TLS. TLS(Transport Layer Security) is a security technology that uses encryption to protect connections/sensitive data between clients and the web servers from Man-In-The-Middle Attacks. We'll be implementing how to create an SSL/TLS digital certificate that protects our website.

![12](https://user-images.githubusercontent.com/47898882/128687800-2e53b3fd-dba1-4ec1-bf40-b3399ff9c19e.JPG)



Now that all the concepts have been explained..Lets Begin!!

# **Configure Nginx as a Loadbalancer**
- Spin up an EC2 instance that is going to act as a loadbalancer. Make sure to open inbound connections for ports 22(ssh), 80(http), 443(https).
- Install Nginx & Enable the Service

```
$ sudo apt install nginx
$ sudo systemctl enable nginx
$ sudo systemctl start nginx
```
- Edit the configuration file in /etc/hosts and create an alias for the webservers, also map the alias to the server's ip address

![4](https://user-images.githubusercontent.com/47898882/128644757-889ba747-01f1-4587-8674-8ef2e84dfc12.JPG)

- Configure the loadbalancer by editing the configuration file in /etc/nginx/nginx.conf.

*Note: Do not forget to comment out the /etc/nginx/sites-enabled directory as it contains the default configuration for nginx*

```
upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

```

![5](https://user-images.githubusercontent.com/47898882/128644817-a2332781-4db9-4125-a46e-12985c5a8e99.JPG)

- Restart your nginx webservice and check the status 

```
$ sudo systemctl restart nginx
$ sudo systemctl status nginx
```

- At this point you can test your load balancing configuration on the browser using your public ip address
![6](https://user-images.githubusercontent.com/47898882/128645056-294530a2-cd25-4a2d-a965-a1be2b4a69d9.JPG)


# **Register a domain name and configure secure connection using SSL/TLS**

- Register a domain name for your website. If you need a free domain, just go to [freenom](https://freenom.com).

- After registration, we have to find a way of mapping the Domain name to our ip address. This wil be made possible by the Route 53 service on AWS. Roure 53 allows for routing of network/traffic to your domain name b mapping the ip address to the domian name.

- Go to the Route 53 service on the AWS console and create a new hosted zone. 

![7](https://user-images.githubusercontent.com/47898882/128646772-07490a02-a3fc-4134-8399-329b672ed1e6.JPG)

- Copy the DNS names where traffic will be routed to and paste in the nameservers section of your domain name on the freenom website.

![8](https://user-images.githubusercontent.com/47898882/128646889-0b247656-52e7-4cea-be63-e474074d87f7.JPG)

![9](https://user-images.githubusercontent.com/47898882/128646891-09191b63-de39-4f84-9d98-be8aebfc216d.JPG)


- Create new records and add your ipaddress, so it can be mapped to the domain name.

![10](https://user-images.githubusercontent.com/47898882/128646968-1c03acc4-e77f-40fb-8dc3-2f0178ebff3a.JPG)


- Install certbot and request for an SSL/TLS certificate. Before you install chec to see if the `snapd` service is running
 
```
$ sudo systemctl status snapd
$ sudo snap install --classic certbot
```
![1](https://user-images.githubusercontent.com/47898882/128647101-4c520799-c6df-4b28-9592-429550c3451c.JPG)

- Link the /snap/bin/certbot directory to /usr/bin/certbot directory 

```
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

- Request for a SSL/TLS certificate using the command below and follow all the prompts.

```
$ sudo certbot --nginx
```

![2](https://user-images.githubusercontent.com/47898882/128647070-bec9678c-148b-48eb-a94c-5e9ef205d978.JPG)

- Test your setup on the browser using the domain name https://<your-domain-name.com>. You'll find out that your website has been secured when you see the padloc icon at top left corner of your browser. You can also view the digital certificate.

![3](https://user-images.githubusercontent.com/47898882/128684760-257da76c-06d0-4c6d-b822-d1a3d9cf5ecc.JPG)

![11](https://user-images.githubusercontent.com/47898882/128647143-27482544-9648-44ae-8f46-884ec3c8cfa0.JPG)

- Your Certificate has to be renews after 90 days. Configure a cronjob in the crontab file that automates the renewal to twice a day. You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

```
$ crontab -e
```

- Add the following line to the file:

```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

## Congratulations!!. You have configured a load balancer using nginx and secured connections to your website using SSL/TLS digital certificates.