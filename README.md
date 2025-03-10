# CI-CD-for-Deploying-Multiple-EC2-Instances-with-Ansible-GitHub
🚀 CI/CD for Deploying Multiple EC2 Instances with Ansible &amp; GitHub  
We will:  
  1. Use Terraform to create multiple EC2 instances.
  2. Use Ansible to configure those instances (e.g., install Nginx, Docker, etc.).
  3. Automate it with GitHub Actions.


---

🚀 CI/CD for Deploying Multiple EC2 Instances with Ansible & GitHub

We will:

1. Use Terraform to create multiple EC2 instances.


2. Use Ansible to configure those instances (e.g., install Nginx, Docker, etc.).


3. Automate it with GitHub Actions.




---

my-infra-repo/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ansible/
│   ├── playbook.yml
│   └── inventory
└── app/
    └── app-code/

1️⃣ Prerequisites

✅ AWS IAM User – Needs EC2FullAccess & IAMFullAccess.
✅ SSH Key Pair – For Ansible to connect to EC2 instances.
✅ Terraform Installed – To create EC2 instances.
✅ Ansible Inventory File – Defines the target servers.

---

2️⃣ Create Terraform File to Launch Multiple EC2 Instances (terraform/main.tf)

This script creates 3 EC2 instances.

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {}
variable "instance_type" {
  default = "t2.micro"
}
variable "ami_id" {
  default = "ami-0c55b159cbfafe1f0"
}

resource "aws_instance" "web" {
  count = 3
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = "my-key-pair"

  tags = {
    Name = "Ansible-EC2-${count.index}"
  }
}

output "instance_ips" {
  value = aws_instance.web[*].public_ip
}

👉 This will create 3 EC2 instances.


---

3️⃣ Create Ansible Playbook to Configure Instances (ansible/playbook.yml)

This installs Nginx on all EC2 instances.

- name: Configure EC2 instances
  hosts: all
  become: yes
  tasks:
    - name: Update packages
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes


---

4️⃣ Create Ansible Inventory File (ansible/inventory.ini)

This file will be dynamically updated with EC2 instance IPs.

[web]
# Instances' IPs will be added dynamically here


---

5️⃣ GitHub Actions Workflow (.github/workflows/deploy.yml)

This will deploy EC2 instances & configure them using Ansible.

name: Deploy EC2 with Ansible

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Install Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Initialize Terraform
        run: terraform init
        working-directory: terraform

      - name: Apply Terraform (Create EC2 Instances)
        run: terraform apply -auto-approve
        working-directory: terraform

      - name: Fetch EC2 IPs
        run: |
          terraform output -json instance_ips | jq -r '.[]' > ansible/inventory.ini
        working-directory: terraform

      - name: Install Ansible
        run: sudo apt update && sudo apt install -y ansible

      - name: Run Ansible Playbook
        run: ansible-playbook -i ansible/inventory.ini ansible/playbook.yml --private-key ~/.ssh/my-key.pem


---

6️⃣ Push Code to GitHub to Trigger Deployment

git add .
git commit -m "Deploy multiple EC2 with Ansible"
git push origin main

🚀 GitHub Actions will:

1. Launch 3 EC2 instances using Terraform.


2. Fetch their public IPs and update the Ansible inventory.


3. Run Ansible to install Nginx on all EC2 instances.
---

7️⃣ Verify Deployment

Check deployed instances:

terraform output instance_ips

To test if Nginx is running, open http://<EC2-PUBLIC-IP> in a browser.
