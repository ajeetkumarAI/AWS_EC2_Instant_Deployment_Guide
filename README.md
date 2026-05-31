# AWS_EC2_Instant_Deployment_Guide

## Deploy on AWS EC2
Step-by-step guide to deploy the AI/ML/GenAI on an AWS EC2 instance and make it publicly accessible.

Step 1: Launch an EC2 Instance
Log in to the AWS Management Console.
Navigate to EC2 → Instances → Launch Instances.
Configure:
AMI: Ubuntu Server 22.04 LTS (or latest Ubuntu LTS)
Instance type: t3.micro (free tier eligible) or larger
Key pair: Create or select an existing .pem key pair (e.g., aws-instant-api-key-1-hr-document-assistant.pem)
Network: Allow SSH (port 22) in the security group for initial access
Click Launch Instance.
Step 2: Connect to the EC2 Instance via SSH

# Open a terminal (Git Bash on Windows, Terminal on macOS/Linux)
 
# Navigate to where your .pem key file is stored
```BASH
cd Downloads/
 ```
# Secure the key file — required by SSH, prevents "permissions too open" error
```BASH
chmod 400 "aws-instant-api-key-1-hr-document-assistant.pem"
 ```
# Connect to the EC2 instance using its public DNS
# Replace the hostname with your instance's Public IPv4 DNS from the EC2 console
```bash
ssh -i "aws-instant-api-key-1-hr-document-assistant.pem" ubuntu@ec2-<your-ip>.compute-1.amazonaws.com
 ```
# When prompted "Are you sure you want to continue connecting?", type: yes
Tip: Find your instance's Public IPv4 DNS in the EC2 console under
Instances → select your instance → Details → Public IPv4 DNS.

Step 3: Update System Packages

# Update the package index and upgrade all installed packages to latest versions
```bash
sudo apt update && sudo apt upgrade -y
```
Step 4: Install Required System Dependencies

# Install pip (Python package manager)
```bash
sudo apt install python3-pip -y
 ```

# Install Git and Python 3 (usually pre-installed on Ubuntu, but ensuring they're present)
```bash
sudo apt install git python3 python3-venv -y
```
Step 5: Clone the Repository

# Clone the project repository from GitHub
```bash
git clone https://github.com/ajeetkumarAI/HR_Document_Assistant_Copilot.git
 ```
# Change into the project directory
```bash
cd HR_Document_Assistant_Copilot/
```
Step 6: Create and Activate a Virtual Environment

# Create a Python virtual environment named 'venv'
```bash
python3 -m venv venv
 ```
# Activate the virtual environment
# All pip installs will now go into venv/ instead of system Python
```bash
source venv/bin/activate
```
Step 7: Install Python Dependencies

# Verify the dependencies that will be installed
```
cat requirements.txt
 ```
# Install all required Python packages
```bash
pip install -r requirements.txt
```
Step 8: Configure the OpenAI API Key

# Create the .env file and add your OpenAI API key
```bash
nano .env
```
In the nano editor:

Type: OPENAI_API_KEY=sk-your-actual-api-key-here
Save: Press Ctrl+O, then press Enter
Exit: Press Ctrl+X
Important: Replace sk-your-actual-api-key-here with your real OpenAI API key
from https://platform.openai.com/api-keys.

Step 9: Start the Streamlit Application

# Run Streamlit on port 8501, binding to all network interfaces (0.0.0.0)
# --server.address 0.0.0.0  →  allows external access (not just localhost)
# --server.port 8501        →  the port the app listens on
```bash
streamlit run app.py --server.port 8501 --server.address 0.0.0.0
```
Keep it running after SSH disconnect: Use nohup or tmux so the app
survives when you close the terminal:


# Option A: nohup (runs in background, logs to app.log)
```bash
nohup streamlit run app.py --server.port 8501 --server.address 0.0.0.0 > app.log 2>&1 &
 ```
# Option B: tmux (lets you reattach later to see live output)
tmux new -s app
streamlit run app.py --server.port 8501 --server.address 0.0.0.0
# Detach: press Ctrl+B, then D
# Reattach later: tmux attach -t app
Step 10: Configure Security Group (Allow Port 8501)
Go to the EC2 Console → select your instance → Security tab.
Click on the Security Group link.
Click Edit inbound rules → Add rule:
Type: Custom TCP
Port range: 8501
Source: 0.0.0.0/0 (Anywhere-IPv4)
Click Save rules.
Step 11: Access the Application
Open your browser and go to:


http://<your-ec2-public-ipv4>:8501/
Find your current Public IPv4 address in the EC2 console:
Instances → select your instance → Details → Public IPv4 address

Example: http://54.163.205.248:8501/

⚠ Important Notes:

Use http:// (not https://) — Streamlit does not serve HTTPS by default.
The Public IPv4 address changes every time you stop/start the instance.
To get a fixed IP, attach an Elastic IP to your instance (free while the instance is running).
If you see "connection refused", verify:
The Streamlit process is still running (curl http://localhost:8501 from SSH)
The Security Group inbound rule for port 8501 is correctly configured
You're using the current Public IPv4 address (not an old one)
Troubleshooting
Problem	Solution
ERR_CONNECTION_REFUSED when accessing the public URL	1. Check Streamlit is running: curl http://localhost:8501 on the EC2. 2. Verify Security Group allows port 8501. 3. Confirm you're using the current Public IPv4 address.
App stops when SSH session closes	Use nohup or tmux (see Step 9 above).
Public IP changed after restart	Attach an Elastic IP to your instance.
ModuleNotFoundError	Ensure virtual environment is activated: source venv/bin/activate
OPENAI_API_KEY not found	Verify .env file exists in the project root with the correct key.
No documents have been ingested yet	Upload a PDF and click "Ingest/Process Document" before asking questions.
Port 8501 already in use	Kill the existing process: sudo lsof -ti:8501 | xargs kill -9
