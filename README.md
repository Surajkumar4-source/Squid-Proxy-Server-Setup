# Squid-Proxy-Server-Setup



## Proxy server
*A proxy is an intermediary server that acts as a gateway between a client and the internet. It intercepts requests from the client, processes them, and forwards them to the destination server. The proxy can modify requests or responses based on specific rules.*

## Reverse Proxy
*A reverse proxy is a type of proxy that works on behalf of backend servers, handling incoming client requests and forwarding them to the appropriate backend server. Unlike a regular proxy, which hides the client, a reverse proxy hides the identity of the backend servers.*

## Forward Proxy
*A forward proxy is a server that sits between the client (such as a browser) and the destination server (such as a website). It forwards the client's requests to the destination server, often used for tasks like caching, filtering, and security enforcement.*

## Key Differences Between Forward Proxy and Reverse Proxy



| Feature               | Forward Proxy                                    | Reverse Proxy                                     |
|-----------------------|--------------------------------------------------|---------------------------------------------------|
| **Direction of Traffic** | Client to Server (client-side proxy)            | Client to Proxy to Server (server-side proxy)     |
| **Primary Use**        | Used by clients to access the internet or hide their identity | Used by servers to manage traffic to backend servers |
| **Visibility**         | Hides client information from the internet      | Hides backend server information from clients     |
| **Common Use Cases**   | Browsing restrictions, anonymity, caching       | Load balancing, security, SSL termination, caching |






## Use Cases
### 1. Forward Proxy Use Cases:
  - Anonymity: Hides the client's IP address, making it harder for websites to track the user.
 Access Control: Filters access to specific websites or restricts internet use in organizations.
  - Content Filtering: Blocks inappropriate content or enforces internet usage policies.
  - Caching: Caches frequently accessed content to speed up user access and reduce bandwidth.

### 2. Reverse Proxy Use Cases:

  - Load Balancing: Distributes client requests across multiple backend servers to optimize resource use and improve availability.
  - Security and Anonymity: Hides the backend servers from direct client access, protecting them from attacks.
  - SSL Termination: Handles encryption/decryption of HTTPS traffic, offloading the task from backend servers.
  - Caching: Caches content to reduce load on backend servers and improve response times.
  - Web Acceleration: Optimizes and compresses responses, speeding up web delivery.




# Setting Up Squid Proxy Server with NGINX Reverse Proxy
  - To set up a Squid proxy server and integrate it with NGINX as a reverse proxy, follow the steps below:

1. Install NGINX
  - Start by installing NGINX, which will act as a reverse proxy.

```yml
sudo apt update
sudo apt install nginx
```

2. Configure NGINX as a Reverse Proxy
  - Edit the NGINX configuration to forward traffic to a backend server. This configuration is typically located in /etc/nginx/sites-available/default.

```yml
sudo nano /etc/nginx/sites-available/default
```
  *Add the following configuration:*

```yml
server {
    listen 80;

    location / {
        proxy_pass http://backend_server_ip;  # Replace with the actual backend server IP or domain
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
  - proxy_pass: Forwards the client request to the backend server.
  - proxy_set_header: Passes headers such as the client's IP and scheme (HTTP/HTTPS) to the backend server.
    
    - After saving the file, restart NGINX to apply the changes:

```yml
sudo systemctl restart nginx
```

3. Install Squid Proxy Server
Step 1: Update System Packages
```yml
sudo apt update
```
Step 2: Install Squid
```yml
sudo apt install squid -y
```
Step 3: Backup the Original Squid Configuration
  - Create a backup of the Squid configuration file before making any changes:

```yml
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.bak
```
Step 4: Edit Squid Configuration
  - Edit the Squid configuration file:

```yml
sudo nano /etc/squid/squid.conf
```
  *Add the following configurations*

  - http_port: Specify the port on which Squid will listen (default is 3128).
```yml
http_port 3128
```

  - ACL: Set up Access Control Lists (ACLs) to define which clients can use the proxy.
```yml
acl allowed_clients src 192.168.1.0/24  # Modify with your network range
http_access allow allowed_clients
```
  - This configuration allows clients from the 192.168.1.0/24 subnet to use the proxy. Adjust the IP range according to your network setup.

Step 5: Save and Exit
 
4. Restart Squid to Apply Changes
  - Restart the Squid service to apply the new configuration:

```yml
sudo systemctl restart squid    OR

# To reload Squid without restarting:

squid -k reconfigure
```
5. Enable Squid to Start on Boot
  - Ensure that Squid starts automatically on boot:

```yml
sudo systemctl enable squid
```
6. Verify Squid is Running
  - Check if Squid is running correctly:

```yml
sudo systemctl status squid   OR

sudo systemctl restart squid
```
7. Configure Firewall (If Applicable)
  - If you're using a firewall (like UFW ,or Firewalld), allow traffic on the Squid proxy port (default 3128):

```yml
sudo ufw allow 3128/tcp
```
8. Testing the Proxy Server
  - Test the Squid proxy by using a web browser or the curl tool. Replace your-squid-server-ip with your Squid server's actual IP address:

```yml
curl --proxy http://your-squid-server-ip:3128 http://example.com
```
   - This command will send a request through the Squid proxy to access example.com.

<br>





### Configuration Examples:
1, Allow Local Network Access: To allow a specific IP or network range to access Squid:

  *Edit /etc/squid/squid.conf:*

```yml
acl internet_allow src 192.168.206.121/32
http_access allow internet_allow
```

  - For a subnet:
```yml
acl internet_allow src 192.168.206.0/24
http_access allow internet_allow
```
2. Block Specific IP Addresses: To block a range of IP addresses:

```yml
acl banned1 src 172.18.90.100-109
http_access deny banned1
```
3. Block Specific Websites: To block domains like facebook.com and youtube.com:

```yml
acl blocksite1 dstdomain .facebook.com .youtube.com
http_access deny blocksite1
```

4. Restrict by URI Pattern: To block URLs containing reddit.com:

```yml
acl banned_reddit url_regex ^http://.*reddit.com/.*$
http_access deny banned_reddit
```
5, Block List of Websites:

  - Create a file /etc/squid/blockedsites.squid:
```yml
.tesla.com
.gmail.com
.cdac.in
```
  *Edit /etc/squid/squid.conf:*
```yml
acl blocksites dstdomain "/etc/squid/blockedsites.squid"
http_access deny blocksites
```




<br>

<br>

## Conclusion

  - NGINX Reverse Proxy: Handles the traffic from clients and forwards it to backend servers.
  - Squid Proxy: Acts as a caching and filtering proxy, managing traffic between clients and external servers.

*This setup will have Squid handle client requests via NGINX's reverse proxy, allowing caching, security, and improved performance for your server infrastructure.*














