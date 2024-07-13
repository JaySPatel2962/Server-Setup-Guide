# Server Setup Guide For EC2

## 1. EC2 Instance Setup
### EC2 Launch and Configuration
1. Launch an EC2 instance. (New one or From AMI)
2. Log in to the EC2 instance using the provided `.pem` file.
3. Add your local machine's SSH key to the server/EC2 instance.
### Environment Setup If not setup already
1. **Install NVM:** [Installation guide](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)
    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    source ~/.bashrc
    ```
2. **Install Node.js:** [Installation guide](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)
    ```sh
    nvm install node
    ```
    or
    ```sh
    sudo apt update
    sudo apt install nodejs
    ```
3. **Install MongoDB (if using local MongoDB):** [Installation guide](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)
4. **Install Nginx:** [Installation guide](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
    ```sh
    sudo apt install nginx
    ```
5. **Install Certbot:** [Installation guide](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04)
    ```sh
    sudo apt-get install -y certbot python3-certbot-nginx
    ```
## 2. Server Setup

### i. Using Ecosystem File
1. **Setup `ecosystem.config.js` file:**
    - Create folders on the server/EC2 as specified in the `path` key of the ecosystem file.
2. **Generate SSH Key on EC2:**
    - Generate an SSH key and configure it with GitHub for SSH-based cloning.
3. **Deploy Application:**
    - Run the following commands locally:
        ```sh
        pm2 deploy ecosystem.config.js <env> setup
        pm2 deploy ecosystem.config.js <env>
        ```
      Example:
        ```sh
        pm2 deploy ecosystem.config.js production setup
        pm2 deploy ecosystem.config.js production
        ```
    - **NOTE**: Just setup .env after `setup` command then hit second command. 
4. **Enable Ports:**
    - Enable the necessary ports (e.g., 3000, 3001) in the security group and verify everything is running.

### ii. Manual Setup
1. **Generate SSH Key on EC2:**
    - Generate an SSH key and configure it with GitHub for SSH-based cloning.
2. **Create Workspace Directory:**
    - Create appropriate directories in the workspace.
3. **Clone Repository:**
    ```sh
    git clone <repository-url>
    ```
4. **Install Dependencies:**
    ```sh
    nvm use
    npm install 
    or
    yarn
    ```
5. **Setup `.env` file:**
    - Manually add the environment variables.
6. **Start Server using PM2:**
    - For a Node.js process:
        ```sh
        pm2 start <filename>.js
        ```
    - To serve a static build:
        - Create Build
            ```sh
            npm run build
            or
            yarn build
            ``` 
       - Serve build using pm2
            ```sh
            pm2 serve build/ <port> --name "process-name" --spa
            ```
7. **Enable Ports:**
    - Enable the necessary ports (e.g., 3000, 3001) in the security group and verify everything is running.

## 3. Domain Setup
### i. Elastic IP
1. **Purchase and Attach Elastic IP:**
    - Attach the Elastic IP to the EC2 instance.
2. **Configure Domain:**
    - List all domain names (e.g., test.google.com, portal.google.com) and configure them with the Elastic IP.
3. **Verify Domain Configuration:**
    ```sh
    nslookup <domainName>
    ```
    Example:
    ```sh
    nslookup test.google.com
    ```
### ii. Setup Nginx for HTTP (Port 80)
1. **Configure Nginx:**
    ```sh
    cd /etc/nginx/conf.d/
    sudo nano <domain>.conf
    ```
    Example file content:
    ```nginx
    server {
        server_name abc.domain.com; # NOTE: server name = domain name
        location / {
            proxy_pass http://localhost:3001; # Proxy requests to the React app's development server
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        listen 80;
    }
    ```
2. **Test and Reload Nginx:**
    ```sh
    sudo nginx -t
    sudo service nginx reload
    ```
3. **Verify Nginx Setup:**
    - Open the domain in a browser and ensure it is properly configured.
### iii. Create SSL Certificate Using Certbot
1. **Generate SSL Certificate:**
    ```sh
    sudo certbot
    ```
    - Follow the instructions and enter the appropriate options.
    - Certbot will automatically create the certificate and update the Nginx configuration.
2. **Verify SSL Certificate:**
    - The SSL certificate should be successfully generated, and HTTP requests should be redirected to HTTPS.

## Verifying SSL Certificate and Completing Server Setup

To verify the successful generation of your SSL certificate, navigate to your domain in a web browser and look for the padlock icon in the address bar. Click on it to view certificate details.

To ensure your server setup is complete, confirm the following:
1. Your application is accessible via HTTPS.
2. PM2 is managing your Node.js processes.
3. Nginx is properly serving your application.

With these steps, your server should be fully operational and secure. Happy coding! 🚀

