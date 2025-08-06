# Multi-Stage Docker Deployment for VProfile Application

This guide provides instructions for building and deploying a Java web application using a multi-stage Dockerfile to optimize image size and improve build efficiency.

## Prerequisites
- Docker installed on your system
- Virtual machine with sufficient resources (e.g., 2GB RAM, 2 CPUs)
- Internet connection to download dependencies
- Basic knowledge of Linux and Docker commands

## Deployment Steps

1. **Set Up Virtual Machine**
   - Start your virtual machine (e.g., using VirtualBox, VMware, or a cloud provider).
   - Log in to the virtual machine.

2. **Access Root Privileges**
   ```bash
   sudo -i
   ```

3. **Create Project Directory**
   ```bash
   mkdir multistage
   cd multistage
   ```

4. **Create Multi-Stage Dockerfile**
   Create a file named `Dockerfile` with the following content:
   ```dockerfile
   # Build Stage
   FROM openjdk:8 AS BUILD_IMAGE
   RUN apt update && apt install maven -y
   RUN git clone -b vp-docker https://github.com/example/vprofile-repo.git
   RUN cd vprofile-repo && mvn install

   # Serve Stage
   FROM tomcat:8-jre11
   RUN rm -rf /usr/local/tomcat/webapps/*
   COPY --from=BUILD_IMAGE vprofile-repo/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
   EXPOSE 8080
   CMD ["catalina.sh", "run"]
   ```

5. **Build the Docker Image**
   ```bash
   docker build -t appimg:v1 .
   ```

6. **Verify the Image**
   ```bash
   docker images
   ```

7. **Run the Docker Container**
   ```bash
   docker run -d --name vprofile-app -p 8080:8080 appimg:v1
   ```

8. **Check Running Containers**
   ```bash
   docker ps -a
   ```

9. **Access the Application**
   - Open a browser and navigate to `http://<VM_IP_ADDRESS>:8080`, replacing `<VM_IP_ADDRESS>` with the IP address of your virtual machine.

## Additional Notes
- **Multi-Stage Build Benefits**: The multi-stage Dockerfile uses `openjdk:8` for building the application and `tomcat:8-jre11` for serving it, reducing the final image size by excluding build tools.
- **Security**: The sensitive repository URL has been replaced with a dummy URL (`https://github.com/example/vprofile-repo.git`) for security purposes. Replace it with the actual repository URL.
- **Port Configuration**: Ensure port 8080 is open on your virtual machine's firewall.
- **Troubleshooting**:
  - If the build fails, verify that Maven and Git are correctly installed in the build stage.
  - Check Docker logs with `docker logs vprofile-app` if the container fails to start.
  - Ensure the virtual machine has sufficient resources and an active internet connection during the build.
- **Cleanup**: To stop and remove the container, use:
  ```bash
  docker stop vprofile-app
  docker rm vprofile-app
  ```
