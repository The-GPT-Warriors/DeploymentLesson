---
toc: True
comments: True
layout: post
title: Deployment Lesson
description: An in depth deployment lesson
authors: Aliya, Anthony, Emaad, Emma, Ethan T., Tay, Vivian
type: hacks
courses: {'csa': {'week': 19}}
---

## Mini-Guide: Deploying Your Site with AWS

### Prerequisites
1. **GitHub Repository:**
   - Must have a backend repository on GitHub.

2. **Docker Setup:**
   - Verify Backend Docker files are running on localhost.

3. **Domain Configuration:**
   - Have a configured Domain Name pointing to the Public IP of your deployment server using AWS Route 53.

### AWS EC2 Access
1. **Login to AWS Console:**
   - Access AWS Management Console.
   - Navigate to "EC2" and select "Instances."

2. **Instance Selection:**
   - Choose the appropriate instance (CSP or CSA) based on your project.

3. **Terminal Access:**
   - Access the deployment server using either csp.nighthawkcodingsociety.com or csa.nighthawkcodingsociety.com.
   - Use the username "ubuntu" with the password hint "3 Musketeers."

### Application Setup
1. **Finding Port:**
   - Run `docker ps` on AWS EC2 terminal to find an available port starting with 8—.
   - Explanation: Identifies an available port for the application.

2. **Local Docker Setup:**
   - Configure Docker files in VSCode on localhost using the chosen port.
   - Explanation: Sets up the local environment for Docker.

3. **Testing:**
   - Run `docker-compose up` or `sudo docker-compose up` in VSCode terminal.
   - Open `http://localhost:8---` in your browser to check if it runs smoothly.
   - Explanation: Tests the application locally before deployment.

### Server Setup
1. **AWS EC2 Terminal:**
   - Clone your backend repo: `git clone github.com/server/project.git my_unique_name`.
   - Navigate to the repo: `cd my_unique_name`.
   - Explanation: Sets up the server environment and fetches the project code.

2. **Build and Test:**
   - Build the site: `docker-compose up -d --build`.
   - Test your site: `curl localhost:8---` (replace '8---' with your port).
   - Explanation: Builds and tests the application on the server.

### DNS & NGINX Setup
1. **Route 53 DNS:**
   - Set up DNS subdomain for your backend server in AWS Route 53.
   - Explanation: Configures DNS for domain mapping.

2. **NGINX Configuration:**
   - Navigate to `/etc/nginx/sites-available` in the terminal.
   - Create an NGINX config file and configure it accordingly.
   - Explanation: Configures NGINX as a reverse proxy for the application.

3. **Validation and Restart:**
   - Validate with `sudo nginx -t`.
   - Restart NGINX: `sudo systemctl restart nginx`.
   - Test your domain on your desktop browser (http://).
   - Explanation: Validates NGINX configuration and restarts for changes to take effect.

### Certbot Configuration
1. **Run Certbot:**
   - Execute `sudo certbot --nginx` and follow prompts.
   - Choose appropriate options for HTTPS activation.
   - Explanation: Configures SSL certificates for secure communication.

2. **Verify HTTPS:**
   - Test your domain in the browser using HTTPS.
   - Explanation: Ensures successful HTTPS setup.

### Changing Code and Deployment Updates
1. **VSCode Changes:**
   - Before updating, run `git pull` to sync changes.
   - Make code changes and test using Docker Desktop.
   - Explanation: Synchronizes code changes and tests locally.

2. **Deployment Update:**
   - If all goes well, sync changes via UI or `git push` from the terminal.
   - Explanation: Deploys updated code to the server.

### Pulling Changes into AWS EC2
1. **AWS EC2 Terminal:**
   - Navigate to your repo: `cd ~/my_unique_name`.
   - Stop the server: `docker-compose down`.
   - Pull changes: `git pull`.
   - Rebuild and start the container: `docker-compose up -d --build`.
   - Explanation: Pulls and updates code on the server, then restarts the application.

### Optional Troubleshooting Checks on AWS EC2
1. **Check Server Status:**
   - Use commands like `curl localhost:8---` and `docker-compose ps` for verification.
   - Explanation: Checks the status of the running application.

### Cockpit Navigation
1. **Cockpit Features:**
   - Explore Cockpit's features for system monitoring, logs, storage, networking, accounts, services, and software updates.
   - Explanation: Monitors and manages server resources.

### Instance Directory
1. **Purpose**
   - Stores instance specific data
      - ie: configuration files, logs, any data that is specific to that instance.
   - This idea becomes vital when multiple instances are being used simutaneously. 
   - Having a seperate instance directory ensures that configurations and instance data don't get mixed up.

### Database
1. **Purpose**
   - Stores persistent data for the application.
   - Can be SQL or NoSQL.
      - ie: MySQL, PostgreSQL, MongoDB, Redis.
   - Critical for managing transactions, authentication, and more.
