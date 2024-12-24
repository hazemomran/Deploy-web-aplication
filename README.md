# CI/CD Pipeline Project

This project demonstrates the creation of a robust CI/CD pipeline to automate the containerization, deployment, and orchestration of an application across multiple machines using **Docker**, **Jenkins**, **Ansible**, and **Vagrant**.

---

## Project Workflow

1. **Containerize the Application**:
   - A Dockerfile is created to package the application into a container.

2. **Set up Jenkins**:
   - Jenkins is configured to orchestrate the CI/CD pipeline.
   - The pipeline stages:
     1. **Pull Code**: Clone the application repository from GitHub.
     2. **Build Docker Image**: Build a Docker image for the application.
     3. **Push Docker Image**: Push the built image to Docker Hub.
     4. **Run Ansible Playbook**: Deploy the application to two target machines.

3. **Deploy Application Using Ansible**:
   - Install Docker on two target Vagrant machines.
   - Pull the Docker image from Docker Hub to the target machines.
   - Run a Docker container on each target machine.

---

## Technologies Used

- **Docker**: For containerization.
- **Jenkins**: For setting up the CI/CD pipeline.
- **Ansible**: For automating the deployment process.
- **Vagrant**: For creating and managing virtual machines for deployment.
- **GitHub**: For version control and hosting the application code.

---

## Setup and Usage

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```
---

### 2. Dockerize the Application
Create a `Dockerfile` to containerize the application. Below is an example `Dockerfile`:
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

---

### 3. Set Up Jenkins

To orchestrate the CI/CD pipeline:

1. **Install Jenkins**:
   - Download and install Jenkins on your system or server. Follow the [official documentation](https://www.jenkins.io/doc/).

2. **Install Required Plugins**:
   - Navigate to **Manage Jenkins** → **Manage Plugins** and install:
     - **Git Plugin**: For cloning repositories.
     - **Docker Pipeline Plugin**: For Docker image builds and pushes.
     - **Ansible Plugin**: For running Ansible playbooks.

3. **Create a New Pipeline Job**:
   - Open Jenkins and click **New Item**.
   - Select **Pipeline** and configure the job:
     - Add the `Jenkinsfile` script from this repository under the **Pipeline** section.
   - Configure credentials for:
     - GitHub access (e.g., `github-devops`).
     - Docker Hub (e.g., `docker-hub`).

4. **Trigger the Pipeline**:
   - Manually trigger the job or set up a webhook in GitHub to trigger the pipeline automatically on a push event.

---

### 4. Configure Vagrant Machines

To prepare the deployment environment:

1. **Install Vagrant and VirtualBox**:
   - Install [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/) on your system.

2. **Create Vagrant Machines**:
   - Initialize and launch two Ubuntu Vagrant virtual machines:
     ```bash
     vagrant init ubuntu/jammy64
     vagrant up
     ```

3. **Verify Connectivity**:
   - Ensure the machines are running and accessible via SSH:
     ```bash
     vagrant ssh
     ```

4. **Note Machine Details**:
   - Note the private IP addresses assigned to the VMs (e.g., `192.168.56.22`, `192.168.56.16`).

5. **Configure Ansible Inventory**:
   - Create an `inventory` file listing the target machines:
     ```ini
     [targets]
     192.168.56.22 ansible_user=vagrant 
     192.168.56.16 ansible_user=vagrant
     ```

---

### 5. Set Up Ansible

To automate the deployment process:

1. **Install Ansible**:
   - Install Ansible on your Jenkins agent machine:
     ```bash
     sudo apt update
     sudo apt install ansible -y
     ```

2. **Write an Ansible Playbook**:
   - Create a `playbook.yml` to deploy the application:
     ```yaml
     - name: Install Docker, pull image, and run container on target machines
      hosts: all
      become: yes
      tasks:
        - name: Install prerequisites for Docker
          apt:
            name: ["apt-transport-https", "ca-certificates", "curl", "software-properties-common"]
            state: present
        - name: Add Docker GPG key
          ansible.builtin.shell: |
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
        - name: Add Docker repository
          apt_repository:
            repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
            state: present
        - name: Install Docker
          apt:
            name: ["docker-ce", "docker-ce-cli", "containerd.io"]
            state: present
        - name: Ensure Docker service is running
          service:
            name: docker
            state: started
            enabled: yes
        - name: Pull Docker image
          docker_image:
            name: "hazemomran/deploy-webapp"
            source: pull
        - name: Run Docker container
          docker_container:
            name: "flask-app"
            image: "hazemomran/deploy-webapp"
            state: started
            ports:
              - "5000:5000"
     ```

3. **Test the Playbook**:
   - Run the playbook locally to verify its functionality:
     ```bash
     ansible-playbook -i inventory playbook.yml
     ```

4. **Integrate with Jenkins**:
   - Ensure the playbook is invoked as the last stage in the Jenkins pipeline.

---

## Project Structure

The repository structure is as follows:


---

## Running the Pipeline

1. **Push Code to GitHub**:
   - Push your application code to the GitHub repository.

2. **Trigger Jenkins Pipeline**:
   - Start the pipeline manually or via a webhook configured in GitHub.

3. **Deployment**:
   - Jenkins automates the entire process:
     - Pulls the latest code from GitHub.
     - Builds and pushes the Docker image to Docker Hub.
     - Runs an Ansible playbook to deploy the application across the target machines.

4. **Verify Deployment**:
   - Check the application on the target machines:
     ```bash
     curl http://192.168.56.101
     curl http://192.168.56.102
     ```

---

## Future Improvements

To enhance this project:

1. **Add Automated Tests**:
   - Include unit tests and integrate them into the Jenkins pipeline.

2. **Use Dynamic Inventory**:
   - Leverage Ansible's dynamic inventory for scaling deployments.

3. **Automate Vagrant Setup**:
   - Automate the creation of Vagrant machines using Ansible or Jenkins.

4. **Enhance Security**:
   - Use secrets management tools like HashiCorp Vault for credentials.



