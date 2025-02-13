# 🚀 Python and Ansible Automation 🤖

## 📌 Introduction
As a Linux administrator with DevOps experience, you can leverage Python to automate repetitive tasks and integrate it with Ansible for enhanced automation. This document covers essential Python scripts for system administration and how to use Python to interact with Ansible effectively.

## 📜 Table of Contents
- [🛠️ System Administration Scripts](#system-administration-scripts)
  - [📊 Monitor System Resources](#monitor-system-resources)
  - [🔑 Automate SSH Login and Command Execution](#automate-ssh-login-and-command-execution)
  - [💾 Backup Important Files](#backup-important-files)
  - [📜 Monitor Log Files for Errors](#monitor-log-files-for-errors)
  - [📧 Send Email Alerts](#send-email-alerts)
  - [👤 Automate User Creation](#automate-user-creation)
  - [🔍 Check Open Ports](#check-open-ports)
- [🤝 Integrating Python with Ansible](#integrating-python-with-ansible)
  - [📜 Running an Ansible Playbook Using Python](#running-an-ansible-playbook-using-python)
  - [🚀 Using Ansible Runner](#using-ansible-runner)
  - [💡 Running an Ansible Ad-hoc Command](#running-an-ansible-ad-hoc-command)
  - [📡 Querying Ansible Facts](#querying-ansible-facts)
  - [🌐 Using Ansible API](#using-ansible-api)
  - [🔌 Exposing Ansible as a REST API](#exposing-ansible-as-a-rest-api)

## 🛠️ System Administration Scripts

### 📊 Monitor System Resources
```python
import psutil

def system_monitor():
    print(f"CPU Usage: {psutil.cpu_percent()}%")
    print(f"Memory Usage: {psutil.virtual_memory().percent}%")
    print(f"Disk Usage: {psutil.disk_usage('/').percent}%")

system_monitor()
```

### 🔑 Automate SSH Login and Command Execution
```python
import paramiko

def ssh_command(host, user, password, command):
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    client.connect(host, username=user, password=password)
    
    stdin, stdout, stderr = client.exec_command(command)
    print(stdout.read().decode())
    client.close()

ssh_command("192.168.1.10", "root", "password", "uptime")
```

### 💾 Backup Important Files
```python
import shutil
import os
from datetime import datetime

def backup(source, destination):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_name = f"backup_{timestamp}.tar.gz"
    shutil.make_archive(os.path.join(destination, backup_name), 'gztar', source)
    print(f"Backup created: {backup_name}")

backup("/etc", "/backup")
```

### 📜 Monitor Log Files for Errors
```python
def monitor_log(logfile):
    with open(logfile, 'r') as file:
        for line in file:
            if "ERROR" in line:
                print(f"Error found: {line.strip()}")

monitor_log("/var/log/syslog")
```

### 📧 Send Email Alerts
```python
import smtplib
from email.mime.text import MIMEText

def send_email(subject, body, to_email):
    msg = MIMEText(body)
    msg["Subject"] = subject
    msg["From"] = "admin@example.com"
    msg["To"] = to_email

    with smtplib.SMTP("smtp.example.com", 587) as server:
        server.starttls()
        server.login("admin@example.com", "password")
        server.sendmail("admin@example.com", to_email, msg.as_string())

send_email("Server Alert", "High CPU usage detected!", "admin@example.com")
```

### 👤 Automate User Creation
```python
import os

def create_user(username, password):
    os.system(f"sudo useradd -m -p $(openssl passwd -1 {password}) {username}")
    print(f"User {username} created.")

create_user("newuser", "password123")
```

### 🔍 Check Open Ports
```python
import socket

def check_open_ports(host, ports):
    for port in ports:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(1)
            if s.connect_ex((host, port)) == 0:
                print(f"Port {port} is open on {host}")

check_open_ports("127.0.0.1", [22, 80, 443])
```

## 🤝 Integrating Python with Ansible

### 📜 Running an Ansible Playbook Using Python
```python
import subprocess

def run_ansible_playbook(playbook, inventory):
    command = ["ansible-playbook", "-i", inventory, playbook]
    result = subprocess.run(command, capture_output=True, text=True)
    print("STDOUT:\n", result.stdout)
    print("STDERR:\n", result.stderr)

run_ansible_playbook("deploy.yml", "inventory.ini")
```

### 🚀 Using Ansible Runner
```python
import ansible_runner

def run_ansible_playbook(playbook, inventory):
    r = ansible_runner.run(private_data_dir='.', playbook=playbook, inventory=inventory)
    print(f"Status: {r.status}")
    print(f"Return Code: {r.rc}")
    print(f"Output: {r.stdout.read()}")

run_ansible_playbook("deploy.yml", "inventory.ini")
```

### ⚡ Running an Ansible Ad-hoc Command
```python
def run_ansible_command(hosts, module, args):
    command = ["ansible", hosts, "-m", module, "-a", args]
    result = subprocess.run(command, capture_output=True, text=True)
    print("STDOUT:\n", result.stdout)
    print("STDERR:\n", result.stderr)

run_ansible_command("webservers", "shell", "df -h")
```

### 📡 Querying Ansible Facts
```python
import json

def get_ansible_facts(host):
    command = ["ansible", host, "-m", "setup"]
    result = subprocess.run(command, capture_output=True, text=True)
    
    facts = json.loads(result.stdout.split("\n", 1)[1])  # Extract JSON part
    print(json.dumps(facts, indent=2))

get_ansible_facts("localhost")
```

### 🌐 Using Ansible API
```python
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.inventory.manager import InventoryManager
from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.playbook.play import Play

def run_ansible_task(hosts, module, args):
    loader = DataLoader()
    inventory = InventoryManager(loader=loader, sources=['inventory.ini'])
    variable_manager = VariableManager(loader=loader, inventory=inventory)

    play_source = {
        'name': "Ansible Play",
        'hosts': hosts,
        'gather_facts': 'no',
        'tasks': [{'action': {'module': module, 'args': args}}]
    }

    play = Play().load(play_source, variable_manager=variable_manager, loader=loader)
    
    tqm = TaskQueueManager(inventory=inventory, variable_manager=variable_manager, loader=loader)
    result = tqm.run(play)
    tqm.cleanup()

    print(f"Task result: {result}")

run_ansible_task("webservers", "ping", "")
```            

### 🔌 Exposing Ansible as a REST API
```python
from flask import Flask, request, jsonify
import subprocess

app = Flask(__name__)

@app.route("/run-playbook", methods=["POST"])
def run_playbook():
    data = request.json
    playbook = data.get("playbook")
    inventory = data.get("inventory", "inventory.ini")
    
    command = ["ansible-playbook", "-i", inventory, playbook]
    result = subprocess.run(command, capture_output=True, text=True)
    
    return jsonify({"stdout": result.stdout, "stderr": result.stderr, "returncode": result.returncode})

if __name__ == "__main__":
    app.run(debug=True)
```

## ✅ Conclusion
By integrating Python with Ansible, you can create powerful automation workflows that enhance system administration and DevOps practices. 🚀💡
