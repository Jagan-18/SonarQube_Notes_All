Here is the installing SonarQube to setting up Nginx and enabling HTTPS.

---

# **SonarQube Installation Guide on Ubuntu**

This guide provides detailed steps for installing and configuring **SonarQube** on an Ubuntu server, along with setting up **Nginx** as a reverse proxy and enabling **HTTPS** for secure access.

---

## **Prerequisites**

Before you begin, ensure that you have the following:

- A server running **Ubuntu** (or other Debian-based distribution).
- **Root** or **sudo** access to execute system commands.
- A basic understanding of terminal commands.
- A registered domain (e.g., `adityatesting.in`) pointing to your server's IP address.

---

## **Step 1: Update the System**

Ensure your system is up-to-date:

```bash
sudo apt update
sudo apt upgrade -y
```

- **Explanation**: 
  - `sudo apt update`: Updates the package lists for upgrades.
  - `sudo apt upgrade -y`: Installs the latest versions of installed packages.

---

## **Step 2: Install Java**

SonarQube requires **Java 11** or **Java 17**. This guide installs **OpenJDK 17**.

```bash
sudo apt install openjdk-17-jdk -y
```

To verify the installation:

```bash
java -version
```

- **Explanation**:
  - Installs **OpenJDK 17**, which is the required version for SonarQube.
  - `java -version` confirms that Java is correctly installed.

---

## **Step 3: Install PostgreSQL**

SonarQube uses **PostgreSQL** as its database. In this step, we will install **PostgreSQL 15** and set it up for SonarQube.

### 3.1 Install Dependencies

```bash
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```

- **Explanation**: Installs necessary dependencies and imports the PostgreSQL repository key.

### 3.2 Add PostgreSQL Repository

```bash
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

- **Explanation**: Adds the PostgreSQL repository to your package manager's sources list.

### 3.3 Install PostgreSQL 15

```bash
sudo apt update
sudo apt install postgresql-15 -y
```

- **Explanation**: Installs **PostgreSQL 15**.

### 3.4 Configure PostgreSQL

```bash
sudo -i -u postgres
createuser sonar
createdb sonar -O sonar
psql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'your_password';
\q
exit
```

- **Explanation**: 
  - Creates a PostgreSQL user and database for SonarQube.
  - Replace `'your_password'` with a secure password.

---

## **Step 4: Install SonarQube**

### 4.1 Download SonarQube

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
```

- **Explanation**: Downloads **SonarQube** from the official repository.

### 4.2 Extract and Move SonarQube

```bash
unzip sonarqube-10.5.1.90531.zip
sudo mv sonarqube-10.5.1.90531 /opt/sonarqube
```

- **Explanation**: 
  - Extracts the downloaded zip file and moves it to `/opt/sonarqube` for proper installation.

### 4.3 Create a SonarQube User and Change Ownership

```bash
sudo adduser --system --no-create-home --group --disabled-login sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

- **Explanation**: Creates a dedicated user to run SonarQube and sets ownership of the SonarQube directory to this user.

### 4.4 Configure SonarQube

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

Edit the following lines:

```properties
sonar.jdbc.username=sonar
sonar.jdbc.password=your_password
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
```

- **Explanation**: Configures SonarQube to use the PostgreSQL user and database.

---

## **Step 5: Create a Systemd Service File**

### 5.1 Create Service File

```bash
sudo vi /etc/systemd/system/sonarqube.service
```

Add the following content:

```ini
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

- **Explanation**: This configuration allows SonarQube to be managed as a system service.

### 5.2 Reload systemd and Start SonarQube

```bash
sudo systemctl daemon-reload
sudo systemctl start sonarqube
sudo systemctl enable sonarqube
```

- **Explanation**: Starts SonarQube and enables it to start at boot.

---

## **Step 6: Configure System Limits**

### 6.1 Check and Increase File Descriptors Limit

```bash
ulimit -n
sudo vi /etc/security/limits.conf
```

Add:

```bash
sonarqube - nofile 65536
sonarqube - nproc 4096
```

- **Explanation**: Increases file descriptor and process limits for the `sonarqube` user.

### 6.2 Set Virtual Memory Limits

```bash
sudo sysctl -w vm.max_map_count=262144
sudo vi /etc/sysctl.conf
```

Add:

```bash
vm.max_map_count=262144
```

Apply the changes:

```bash
sudo sysctl -p
```

- **Explanation**: Adjusts the virtual memory limits for SonarQube’s optimal performance.

---

## **Step 7: Install and Configure Nginx**

### 7.1 Install Nginx

```bash
sudo apt install nginx -y
sudo mkdir -p /var/www/html/.well-known/acme-challenge/
echo "test" | sudo tee /var/www/html/.well-known/acme-challenge/test-file
```

- **Explanation**: Installs **Nginx** and creates a directory for the **Let's Encrypt** ACME challenge to verify your domain.

### 7.2 Create Nginx Configuration for SonarQube

```bash
sudo vi /etc/nginx/sites-available/adityatesting.in
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name adityatesting.in www.adityatesting.in;
    
    # Handle the Let's Encrypt ACME Challenge
    location /.well-known/acme-challenge/ {
        root /var/www/html;
        try_files $uri =404;
    }

    # Proxy pass all other requests to SonarQube
    location / {
        proxy_pass http://127.0.0.1:9000; # SonarQube application
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }
}
```

### 7.3 Enable Nginx Configuration

```bash
sudo ln -s /etc/nginx/sites-available/adityatesting.in /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

- **Explanation**: 
  - Creates a symbolic link to enable the configuration.
  - Verifies the Nginx configuration and restarts the Nginx service.

---

## **Step 8: Configure HTTPS with Let's Encrypt**

### 8.1 Install Certbot and Nginx Plugin

```bash
sudo apt install certbot python3-certbot-nginx -y
```

- **Explanation**: Installs **Certbot** and the **Nginx plugin** for automated SSL certificate issuance.

### 8.2 Obtain SSL Certificate

```bash
sudo certbot --nginx -d adityatesting.in
```

- **Explanation**: Requests and installs an SSL certificate for your domain using **Let’s Encrypt**.

### 8.3 Test and Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

- **Explanation**: Verifies the Nginx configuration and reloads it to apply the changes.

### 8.4 Additional Domain Configuration (Optional)

```bash
sudo certbot --nginx -d adityatesting.in -d www.adityatesting.in
```

- **Explanation**: Configures **HTTPS** for both `adityatesting.in` and `www.adityatesting.in` domains.

---

## **Step 9: Access SonarQube**

Once the setup is complete, SonarQube is accessible at:

```
https://adityatesting.in
```

- **Default Credentials**:
  - **Username**: `admin`
  - **Password**: `admin`

After the first login, you will be prompted to change the password.

---

## **Conclusion**

You have now successfully installed and configured **SonarQube** on Ubuntu, set up **Nginx** as a reverse proxy, and enabled **HTTPS** using **Let’s Encrypt**. Your SonarQube instance is now accessible securely via your domain.