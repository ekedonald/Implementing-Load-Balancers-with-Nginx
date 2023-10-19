# Implementing Load Balancers with Nginx
## Introduction to Load Balancing and Nginx
Load balancing refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a server farm or server pool. 

Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers.

A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures that no one server is overworked, which could degrade performance. If a single server goes down, the load balancer redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer automatically starts to send requests to it.

In this manner, a load balancer performs the following functions:
* Distributes client requests or network load efficiently across multiple servers.
* Ensures high availability and reliability by sending requests only to servers that are online.
* Provides the flexibility to add or subtract servers as demand dictates.

Nginx is a versatile software, it can act like a web server, reverse proxy and a load balancer depending on configuration.

## Implementing Nginx as a Basic Load Balancer between Two Web Servers
The following steps are taken to implement Nginx as a basic load balancer between two web servers:

### Step 1: Provisioning the 1st Apache Web Server
* Open your AWS Management Console, click on EC2.

* Click on the Launch Instance button.

* On the Name Box and Amazon Machine Image, type **Apache_Web_Server_01** and **ubuntu** respectively.

* Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type** as the Amazon Machine Image.

* Click on create new key pair.

* Give the key pair name a name of your choice (i.e web11), select `RSA` as the key pair type and `.pem` as the key file format then click on Create key pair. 

*Note that the `.pem` key will be downloaded into your Downloads directory on your computer*.

* On the Network Settings tab, click on the Edit button to configure the Security Group Inbound Rules.

* Click on Add Security Group Rule on the bottom and on the Security Group Name, Source Type, Port Range type the following: **Apache_Server_Security_Group**, **Anywhere** and **8000** respectively.

* Scroll down to the bottom and click on the Launch Instance Button.

* You will see a prompt shown below, click on the Instance ID highlighted.

* Click on the Intance ID of the **Apache_Web_Server_01** Instance you just created.

* Click on the Connect button.

* Copy the highlighted commands shown below:

* Open your terminal.

* Go to the Downloads directory (i.e `.pem` key pair is stored here) using the command shown below:

```sh
cd Downloads
```

* Run the following command to give read permissions to the `.pem` key pair file:

```sh
chmod 400 <private-key-pair-name>.pem
```

* SSH into the Apache_Web_Server_01 Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

* Update the list of packages in the package manager and install the apache server package installation using the following command:

```sh
sudo apt update && sudo apt install apache2 -y
```

* Verify that Apache is running using the command shown below:

```sh
sudo systemctl status apache2
```

### Step 2: Configuring the 1st Apache Server to Serve Content on Port 8000

*  Run the following command to open the Apache listening ports configuration file:

```sh
sudo vi /etc/apache2/ports.conf
```

* Add a new Listen directive for port 8000 as shown below and save the file:


* Run the following command to open the default configuration file of the Apache.

```sh
sudo vi /etc/apache2/sites-available/000-default.conf
```

* Change the listening port ot the Virtualhost from 80 to 8000 as shown below and save the file:

* Reload the Apache Web Server to load the new configuration changes using the command shown below:

```sh
sudo systemctl reload apache2
```

### Step 3: Creating a Webpage for the 1st Apache Web Server 

* Create and open a new **index.html** file with command shown below:

```sh
sudo vi index.html
```

* Before pasting the html code block shown below, get the **Public IPv4 address** of the **Apache_Web_Server_01** by clicking the Instance ID of the Instance and copying the address.

```sh
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my Apache Web Server 1 EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>
```

* Change the file ownership of the **index.html** file so that the Nginx Load Balancer server can have access to the file using the command shown below:

```sh
sudo chown www-data:www-data ./index.html
```

* Overwrite the default html file of the Apache Web Server by using the command shown below:

```sh
sudo cp -f ./index.html /var/www/html/index.html
```

* Reload the Apache Web Server to load the new configuration changes using the command shown below:

```sh
sudo systemctl reload apache2
```

* Go to your your browser and paste the following URL:

```sh
http://<public_ip_address_of_apache_web_server_1>:8000
```

_The Web Page Should Look Like This_

### Step 4: Provisioning and Configuring the 2nd Apache Web Server

Repeat steps 1 - 3 but ensure the following parameters are changed to these when implementing the steps:

1. Name of Instance: Apache_Web_Server_02

2. Key pair: web11

3. Security Group: Apache_Server_Security_Group

4. When serving content for the 2nd Apache Web Server's webpage, use this as your html code block:

```sh
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my Apache Web Server 2 EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>
```
After completing the steps, go to your your browser and paste the following URL:

```sh
http://<public_ip_address_of_apache_web_server_2>:8000
```

_The Web Page Should Look Like This_

### Step 5: Provisioning the Nginx Load Balancer Server

* On the Instances tab, click on the Launch Instance button.

* On the Name Box and Amazon Machine Image, type **Nginx Load Balancer Server** and **ubuntu** respectively.

* Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type** as the Amazon Machine Image.

* Click on the key pair drop down button and select **web11** as the key pair.

* Create a new security group and select allow HTTP traffic from the internet.

* Click on the Launch Instance button.

* You will see a prompt shown below, click on the Instance ID highlighted.

* Click on the Intance ID of the **Nginx Load Balancer Server** Instance you just created.

* Click on the Connect button.

* Copy the highlighted command shown below:

* Open your terminal.

* Go to the Downloads directory (i.e `.pem` key pair is stored here) using the command shown below:

```sh
cd Downloads
```

* Run the following command to give read permissions to the `.pem` key pair file:

```sh
chmod 400 <private-key-pair-name>.pem
```

* SSH into the **Nginx Load Balancer Server** Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

* Update the list of packages in the package manager and install the apache server package installation using the following command:

```sh
sudo apt update && sudo apt install nginx -y
```

* Verify that Nginx is running using the command shown below:

```sh
sudo systemctl status nginx
```

### Step 6: Configuring Nginx as the Load Balancer between the Two Apache Web Servers

* Run the following command to create and open a load balancer configuration file:

```sh
sudo vi /etc/nginx/conf.d/loadbalancer.conf
```

* Paste the code block shown below and save the file:

```sh
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 127.0.0.1:8000; # public IP and port for webserser 1
            server 127.0.0.1:8000; # public IP and port for webserver 2

        }

        server {
            listen 80;
            server_name <your load balancer's public IP addres>; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
```

* Run the following command to test if the Nginx Load Balancer Server's configuration was successful:

```sh
sudo nginx -t
```

* Reload the Nginx Load Balancer Server to load the new configuration changes using the command shown below:

```sh
sudo nginx -s reload
```

* Go to your browser and paste the URL shown below:

```sh
http://<public_ip_address_of_nginx_load_balancer_server>:80
```

_You will notice that each time you refresh the page, it displays the content of one of the two **Apache Web Servers**. Note that the **Load Balancing Algorithm** used in configuring the **Nginx Load Balancer Server** is the **Round Robin Algorithm**._

