# **Installation Guide for Docker and SonarQube**

## **Prerequisites**

Before starting the installation, ensure that you have:

- A machine running Ubuntu or any Debian-based Linux distribution.
- Root or sudo privileges to execute the commands.

## **Step 1: Update System Packages**

First, make sure your system's package list is up-to-date.

```bash
sudo apt update
```

- **Explanation**: This command updates the local package cache to ensure you're installing the latest available software.

## **Step 2: Install Docker**

Next, install Docker, which will allow you to run SonarQube in a containerized environment.

```bash
sudo apt install docker.io -y
```

- **Explanation**: This installs Docker on your system. The `-y` flag automatically answers 'yes' to the installation prompts.

## **Step 3: Add User to Docker Group**

To allow the `ubuntu` user to run Docker commands without using `sudo`, add the user to the `docker` group.

```bash
sudo usermod -aG docker ubuntu
```

- **Explanation**: This command adds the user `ubuntu` to the `docker` group, which grants the user permission to run Docker commands without needing superuser privileges.

## **Step 4: Apply the New Group Permissions**

After adding the user to the Docker group, you'll need to apply the group changes. You can either log out and log back in or run:

```bash
newgrp docker
```

- **Explanation**: This command applies the new group changes for the current session.

## **Step 5: Verify Docker Installation**

Ensure that Docker is correctly installed and running by pulling and running a test image:

```bash
docker pull hello-world
```

- **Explanation**: This command pulls the `hello-world` image from Docker Hub. After pulling, Docker will try to run this image. If everything is working fine, you will see a confirmation message.

## **Step 6: Install SonarQube Using Docker**

Now you can run SonarQube using Docker.

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

- **Explanation**:
  - `docker run`: This command runs a container from the specified image.
  - `-d`: Runs the container in detached mode (background).
  - `--name sonar`: Assigns the container the name "sonar" for easy reference.
  - `-p 9000:9000`: Maps the SonarQube container's port 9000 to your machine's port 9000. This is the default port for SonarQube's web interface.
  - `sonarqube:lts-community`: Specifies the image to use. The `lts-community` tag refers to the Long-Term Support version of SonarQube's community edition.

## **Step 7: Verify Running Docker Containers**

You can verify that the SonarQube container is running by using the following command:

```bash
docker ps
```

- **Explanation**: This command lists all running Docker containers. If SonarQube is up and running, you should see it listed in the output with the name `sonar`.

---

### **Accessing SonarQube**

Once SonarQube is running, open a browser and go to:

```
http://<YOUR_IP>:9000
```

Replace `<YOUR_IP>` with your server's IP address (or `localhost` if running locally).

- **Default Credentials**:
  - **Username**: `admin`
  - **Password**: `admin`

You will be prompted to change the default password on first login.

---

## **Troubleshooting**

- If you encounter issues accessing SonarQube via `localhost`, ensure that port 9000 is not blocked by a firewall.
- If the Docker command gives permission issues, verify that the user has been correctly added to the `docker` group by running `groups ubuntu` and checking for `docker` in the list.

---

This completes the setup for Docker and SonarQube. Let me know if you need further assistance!