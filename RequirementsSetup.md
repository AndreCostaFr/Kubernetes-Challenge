# 🛠️ Environment Setup Guide (Windows + WSL2)

This guide provides a step-by-step walkthrough to set up the development environment from scratch on a Windows machine. It ensures the correct versions of tools (specifically Ansible > 2.17) are installed within the WSL subsystem.

---

## 1. Install Windows Subsystem for Linux (WSL)

If you do not have Linux installed on your Windows machine yet:

1.  Open **PowerShell** as Administrator.
2.  Run the following command:
    ```powershell
    wsl --install
    ```
3.  **Restart your computer**.
4.  Upon restarting, a window will open prompting you to create a **Username** and **Password** for Ubuntu. Create them.

---

## 2. Install and Configure Docker Desktop

Docker Desktop will manage the containers and serve as the driver for Minikube.

1.  Download and install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/).
2.  Open Docker Desktop.
3.  Navigate to **Settings (Gear Icon) ⚙️** > **Resources** > **WSL Integration**.
4.  **Enable the "Ubuntu" option** (Check the box).
5.  Click **Apply & Restart**.

---

## 3. Install Minikube (Local Cluster)

From this point on, open your **Ubuntu (WSL) terminal** to run the following commands:

1.  Download the binary:
    ```bash
    curl -LO [https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64)
    ```
2.  Install it to the system:
    ```bash
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```
3.  Start the cluster:
    ```bash
    minikube start
    ```

---

## 4. Install Kubectl (Kubernetes Control CLI)

Still in the Ubuntu terminal, install the command-line tool:

1.  Download the latest stable release:
    ```bash
    curl -LO "[https://dl.k8s.io/release/$(curl](https://dl.k8s.io/release/$(curl) -L -s [https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl](https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl)"
    ```
2.  Install the binary:
    ```bash
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```
3.  Verify the connection:
    ```bash
    kubectl get nodes
    ```

---

## 5. Install Ansible (Modern Version 2.19+)

**WARNING:** This project requires a modern version of Ansible (above 2.17) to correctly support the AWS Collections. Do not use `apt install ansible` as it installs outdated versions.

1.  Update lists and install Python Pip:
    ```bash
    sudo apt update
    sudo apt install python3-pip -y
    ```
2.  Install Ansible for the current user:
    ```bash
    pip install --user ansible
    ```
3.  Add the local bin path to your terminal (so the command can be found):
    ```bash
    export PATH=$PATH:~/.local/bin
    ```
4.  **Verify the version** (Must be >= 2.17):
    ```bash
    ansible --version
    ```

---

## 6. Configure Addons

### Enable Ingress
For external access to work (`challenge.local`), enable the Ingress controller in Minikube:

```bash
minikube addons enable ingress
```

---