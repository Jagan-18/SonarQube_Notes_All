Here’s a clean and well-structured README file for your Jenkins installation instructions:

---

# Jenkins Installation Guide (LTS)

This guide will help you install **Jenkins** Long Term Support (LTS) release on a **Linux** system. This process assumes you're using a Debian-based distribution (like Ubuntu).

---

## Prerequisites

- **Operating System**: Debian/Ubuntu-based Linux distributions.
- **Required Packages**: `wget`, `openjdk-17`
- **Jenkins Version**: LTS (Long Term Support)

---

## Step 1: Update the System

Start by updating your system's package list:

```bash
sudo apt update
```

---

## Step 2: Install Java

Jenkins requires Java to run. In this guide, we will install **OpenJDK 17**.

```bash
sudo apt install openjdk-17-jre-headless -y
```

You can verify Java installation with:

```bash
java -version
```

---

## Step 3: Install Jenkins (Long Term Support Release)

Jenkins provides a stable release that is recommended for production environments. To install the Jenkins LTS version, follow the steps below:

### S-1: Add Jenkins Repository Key

To ensure that Jenkins packages are securely signed, download and add the Jenkins repository key:

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
```

### S-2: Add Jenkins Repository

Add the Jenkins stable repository to your package sources:

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### S-3: Update the Package List

Update the package list to include the Jenkins repository:

```bash
sudo apt-get update
```

### S-4: Install Jenkins

Install Jenkins:

```bash
sudo apt-get install jenkins -y
```

---

## Step 4: Start and Enable Jenkins Service

After the installation is complete, Jenkins will automatically be set up as a system service. You can start Jenkins and enable it to start on boot with the following commands:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Check the status of Jenkins to ensure it's running:

```bash
sudo systemctl status jenkins
```

---

## Step 5: Access Jenkins

Once Jenkins is up and running, you can access it via your web browser.

The default URL to access Jenkins is:

```
http://<your_server_ip>:8080
```

---

## Step 6: Unlock Jenkins

The first time you access Jenkins, you will be prompted to unlock it. Follow these steps:

1. Open the terminal on your Jenkins server.
2. Retrieve the Jenkins unlock key:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

3. Copy the unlock key and paste it in the web interface to unlock Jenkins.

---

## Step 7: Set Up Jenkins

Once unlocked, you can proceed with the Jenkins setup wizard:

1. Install the suggested plugins.
2. Create an admin user (optional).
3. Jenkins is now ready for use!

---

## Troubleshooting

- **Jenkins not starting**: If Jenkins is not starting, check the logs located at `/var/log/jenkins/jenkins.log` for errors.
- **Port 8080 is occupied**: If another service is using port 8080, you can change the port by modifying the `jenkins.xml` file located at `/etc/default/jenkins`.

---



