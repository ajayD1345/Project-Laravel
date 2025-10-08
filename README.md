# 🚀 Project Laravel

This project automates the provisioning and deployment of a **LAMP (Linux, Apache, MySQL, PHP)** stack across two Ubuntu-based servers — **Master** and **Slave** — using **Vagrant**, **Bash scripting**, and **Ansible**. It also deploys a **Laravel web application** from GitHub and sets up a **cron job** for daily uptime monitoring.

---

## 🧠 Overview

**Master Node:**
- Provisioned using Vagrant.
- Deploys a complete LAMP stack with a Bash script.
- Clones and configures the Laravel application.

**Slave Node:**
- Configured via an Ansible playbook.
- Executes the Bash deployment script remotely.
- Sets up a daily cron job to log uptime.

---

## 🧩 Key Components

- **Vagrantfile** — Defines and provisions both Master and Slave VMs.  
- **deploy.sh** — Automates the installation and configuration of Apache, MySQL, PHP, and Laravel.  
- **playbook.yml** — Uses Ansible to execute the deployment script and configure uptime monitoring.  
- **ansible.cfg & inventory.ini** — Manage connection and environment settings for automation.  
- **Logs:**  
  - `deploy.log` — Bash deployment log.  
  - `deployerror.log` — Error tracking during deployment.  
  - `uptime.log` — Cron job output log.

---

## ⚙️ Deployment Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/project-laravel.git
   cd project-laravel
   ```
2. Provision servers
   ```
   vagrant up
   ```
3. Run the Ansible playbook
   ```
   ansible-playbook site.yml
   ```
4. Access the Laravel app
   - Open your browser and visit the Master or Slave node IP address (e.g., http://192.168.8.131).

## 🖼️ Screenshots
 
  - Master Node — Laravel app deployed via Bash script.
  - Slave Node — Laravel app deployed via Ansible automation.
  - Playbook Execution — Successful run of Ansible tasks.
  - Cron Job — Automated uptime tracking.
(See /Screenshots folder for details.)

## 📚 Learnings & Highlights

   - Infrastructure-as-Code (IaC) principles using Vagrant and Ansible.
   - End-to-end server provisioning and LAMP stack deployment.
   - Automated PHP (Laravel) application setup.
   - Real-world CI/CD-style workflow simulation.

# 🧩 Conclusion

This project reflects my ability to design and implement end-to-end automation pipelines using multiple DevOps tools. It demonstrates strong fundamentals in system provisioning, configuration management, and deployment automation — key pillars of modern infrastructure engineering.

## 🧑‍💻 Author
  - Hamed Ayojide
