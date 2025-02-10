# SonarQube Installation Guide
This guide will help you set up SonarQube with PostgreSQL on a Linux system, using OpenJDK 17. Follow the steps below for a smooth installation process.

#### Prerequisites
 - **Operating System:** Ubuntu/Debian-based Linux distributions.
 - **Required Packages:** curl, ca-certificates, wget, unzip, PostgreSQL, OpenJDK 17
-  **SonarQube Version:** 10.5.1 (or similar)



### Step 1: Update the System
```bash
 sudo apt update
```
- Installs the OpenJDK 17 development kit. Sonar qube requires Java 17 or Java 21 and this command ensures you have a compatible version
```bash
sudo apt install openjdk-17-jre-headless -y
```
### Step 2: Install PostgreSQL - SonarQube uses PostgreSQL as its database. Install and configure PostgreSQL 15:
---
#### S-1: . Install dependencies:
- 1.First, install the necessary tools to manage PostgreSQL repositories and certificates:

```bash
sudo apt install curl ca-certificates
```
 ##### Note:
   - **1. Curl:** - Used the downloading the PostgreSQL signing key.
   - **2.ca -certificates:** Ensures secure communication using certificates.

- 2. To Create a directory for the PostgreSQL repository configuration:
```bash
sudo install -d /usr/share/postgresql-common/pgdg
```
- 3. Ensures a directory is created with the correct permissions.
   - Download and add the PostgreSQL signing Key:
```bash
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys ACCC4CF8.asc

(Or)

sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```
---

---
2. Add PostgreSQL repository:
- Add the PostgreSQL repository to the system's sources list:
```bash
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc]
https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc]
https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
---
3. Install PostgreSQL 15:
- To Update the package list to include the new PostgreSQL repository: 
```bash
sudo apt update
```
- Install PostgreSQL 15:
```bash
sudo apt install postgresql-15 -y
```
---
4. Configure PostgreSQL:
- 1. To Switch to the Postgre user (The default database administrator).
```bash
sudo -i -u postgres
```
- 2. To Create a user name sonar:
```bash
createuser sonar
```
- 3. TO Create a database named sonar owned by the sonar user:
```bash
createdb sonar -O sonar
```
- 4. Set a password for the sonar user:
```bash
psql  # Open the PostgreSQL Termanel
```
```bash
ALTER USER sonar WITH ENCRYPTED PASSWORD 'your_password';
```
```bash
\q # Outof the Termal
```
- 5. Exit the postgres user:
```bash
exit
```
---
### Step 3: Install SonarQube
- 1. Download SonarQube Binaries:
```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
```
```bash
sudo apt install unzip

```
- 2. Extract and move SonarQube:
```bash
unzip sonarqube-10.5.1.90531.zip
```
```bash
sudo mv sonarqube-10.5.1.90531 /opt/sonarqube
```
#### Note: 
   - #### 1. unzip - Extracts the sonatQuber archive.
   - #### 2. mv - Moves the SonarQube directory to /opt. a common location for third-party software.

```bash
ls
rm sonarqube-10.5.1.90531.zip  # removeing the file
```
- 3. Create a SonarQube user and change ownership:
- To Add a system user for SonarQube without a home directory or login access:

```bash
sudo adduser --system --no-create-home --group --disabled-login sonarqube
```
- Change Ownership of the Sonarqube directory:
```bash
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

- 4. Configure SonarQube: Edit the SonarQube configuration file:
```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```
- #### Uncomment and set the following properties past inside the vi file:
```bash
sonar.jdbc.username=sonar
sonar.jdbc.password=your_password
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
```
---
---
### Step 4: Create a Systemd Service File
- 1. Create the service file for SonarQube:
```bash
sudo vi /etc/systemd/system/sonarqube.service
```
#### Add the following content:
```bash
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
- 2. Reload the systemd daemon and start SonarQube:
- Below command Start and Enable to the service to start on boot:
```bash
sudo systemctl daemon-reload 
```
```bash
sudo systemctl start sonarqube
```
```bash
sudo systemctl enable sonarqube
```
```bash
sudo systemctl status sonarqube
```
---
---
### Step 5:Configure System Limits 
#### 1 To Check and increase file descriptors limit 
- 1. To check the current file descriptor Limit:
```bash
ulimit -n
```
- 2. Edit the limits configuration file:
```bash
sudo vi /etc/security/limits.conf
```
#### Add the following lines to refe the user's: 
```bash
sonarqube - nofile 65536
sonarqube - nproc 4096
```

- 3. Set virtual memory limits: To Temporarily set the "vm.max_map_count"

```bash
sudo sysctl -w vm.max_map_count=262144
```
- 4. To make the change permanent"
```bash
sudo vi /etc/sysctl.conf
```
- 5 Add:
```bash
vm.max_map_count=262144
```
- 6. Apply changes:

```bash
sudo sysctl -p
```
---

### Step 6:Troubleshooting
- If SonarQube is not starting, check the logs in **/opt/sonarqube/logs**.
- Make sure PostgreSQL is running and accessible from SonarQube.