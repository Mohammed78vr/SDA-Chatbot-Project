# SDA-Chatbot-Project

  

## Welcome to Stage 6 of Capstone project!

### Stage 6: Enhanced Infrastructure for Document-Aware Chatbot

In Stage 6, the **SDA-Chatbot** project introduces a robust and scalable infrastructure, integrating cutting-edge tools to support Retrieval-Augmented Generation (RAG) capabilities for document-focused chatbot interactions.

#### **Infrastructure Overview**

The system now incorporates the following components, as illustrated in the diagram:

1.  **User Interaction:**
    
    -   Users interact with the chatbot through a web interface, enabling both casual conversations and document-specific queries by uploading PDFs.
        
2.  **Secure Azure VM:**
    
    -   The core of the architecture is an Azure Virtual Machine (VM), which hosts the system’s key services within a secure environment:
        
        -   **Subnet with NSG (Network Security Group):** Provides controlled and secure access.
            
        -   **Disk Storage:** Ensures data persistence for the application and supporting services.
            
        -   **Network Interfaces:** Facilitates communication between the VM and external systems.
            
3.  **Backend Services:**
    
    -   The VM runs **Streamlit** for the user interface and **FastAPI** for backend business logic, ensuring a seamless and responsive experience.
        
4.  **Data Storage with PostgreSQL:**
    
    -   All chat data and relevant metadata are securely stored in a PostgreSQL database.
        
5.  **Chroma for Context Retrieval:**
    
    -   **Chroma**, a vector store, plays a crucial role by indexing and retrieving relevant sections of uploaded PDFs to enhance the chatbot’s ability to answer document-specific questions.
        
6.  **GitHub Actions for Automation:**
    
    -   The project leverages GitHub Actions to automate deployment and updates, ensuring continuous delivery and integration.
        

#### **Key Benefits of This Stage**

-   Enables users to interact with uploaded PDF content in a meaningful way.
    
-   Enhances response accuracy through context-aware retrieval.
    
-   Builds on the existing system architecture with scalable and modular components.
    

This expanded setup lays the foundation for future enhancements, making the chatbot not only more powerful but also ready for real-world applications.

> **Note:** The added complexity in this stage demonstrates a real-world application of RAG-based systems. Feel free to explore the infrastructure and codebase to understand how these components come together.
  

  

![Alt text](stage-6.png  "a title")

  

  

Under the hood, the system uses a **vector store (Chroma)** to retrieve the most relevant context from uploaded PDFs. This retrieval step enhances the chatbot’s ability to provide accurate, context-aware answers, bridging the gap between simple conversation and document-focused queries.

  

This enhancement integrates seamlessly with our existing setup—Streamlit for the user interface, FastAPI for business logic, and PostgreSQL for data storage—while laying the foundation for further expansion.

  

>  **Note:** Some LLM-related concepts introduced in this stage may seem complex. However, our main goal is to get the project running, and fully understanding the LLM integration is **optional**. If you’re interested, feel free to explore the code and additional resources to enhance your project, but don’t worry if you don’t grasp everything right away.

  

### How to Get Started
#### **Step 1: Adding custom data to the VM**

In this stage, while creating the VM in Azure use this script in custom data:

```
#!/bin/bash

sudo apt update
sudo apt install -y gnupg2 wget

sudo -u azureuser mkdir -p /home/azureuser/miniconda3
sudo -u azureuser wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /home/azureuser/miniconda3/miniconda.sh
sudo -u azureuser bash /home/azureuser/miniconda3/miniconda.sh -b -u -p /home/azureuser/miniconda3
sudo -u azureuser rm /home/azureuser/miniconda3/miniconda.sh

echo 'export PATH="/home/azureuser/miniconda3/bin:$PATH"' | sudo -u azureuser tee -a /home/azureuser/.bashrc


sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
sudo apt update
sudo apt install -y postgresql-16 postgresql-contrib-16 postgresql-client-16


sudo systemctl start postgresql
sudo systemctl enable postgresql
```
#### **Step 2: Setting up a bash script to run the app in the VM**
Then, we will create a **new** file with extension .sh e.g. `setup.sh`. Then copy and paste the script to it and save. 
```
#!/bin/bash
set -e

date
echo "Updating Python application on VM..."

HOME_DIR=$(eval echo ~$USER)
APP_DIR="$HOME_DIR/SDA-Chatbot-Project"
REPO_URL="https://github.com/Mohammed78vr/SDA-Chatbot-Project.git"
BRANCH="main"
GITHUB_TOKEN=$TOKEN  # Passed securely via protectedSettings

# Update code
if [ -d "$APP_DIR" ]; then
    sudo -u azureuser bash -c "cd $APP_DIR && git fetch origin && git reset --hard origin/$BRANCH"
else
    sudo -u azureuser git clone -b "$BRANCH" "https://${GITHUB_TOKEN}@${REPO_URL}" "$APP_DIR"
fi

# Install dependencies
sudo -u azureuser $HOME_DIR/miniconda3/envs/project/bin/pip install --upgrade pip
sudo -u azureuser $HOME_DIR/miniconda3/envs/project/bin/pip install -r "${APP_DIR}/requirements.txt"

# Restart the service
sudo systemctl restart backend
sudo systemctl is-active --quiet backend || echo "Backend failed to start"
sudo systemctl restart frontend
sudo systemctl is-active --quiet frontend || echo "frontend failed to start"

echo "Python application update completed!"
```

To run this, you need to provide 7 arguments to the bash script:  

1.  **PAT_token**: Your GitHub personal access token.
2.  **repo_url**: The URL of your GitHub repository  **(without `https://`)**.
3.  **branch_name**: The branch name to use on the VM.
4.  **db_host**: The database host (e.g.,  `example.postgres.database.azure.com`.
5.  **target_db**: The name of the database you wish to create.
6.  **db_username**: The username for the database server.
7.  **db_password**: The password for the database server.

To run the setup script using this command with the appropriate arguments:  
`bash setup.sh <PAT_token> <repo_url> <branch_name> <db_host> <target_db> <db_username> <db_password>`

#### **Step 3: Configure Environment Variables**

Store your **OpenAI API key**, **Azure database credentials** in a `.env` file.

Your `.env` file should look like this:
```
OPENAI_API_KEY=sk-...
DB_NAME=<azure postgres database name>
DB_USER=<azure postgres user name>
DB_PASSWORD=<azure postgres user password>
DB_HOST=<azure postgres server name>
DB_PORT=5432
AZURE_STORAGE_SAS_URL=...
AZURE_STORAGE_CONTAINER=...
CHROMADB_HOST=<public ip of the chromadb vm>
CHROMADB_PORT=8000
```
>  **Note:** Make sure that you enable Allow public access from any Azure service within Azure to this server and +Add current client IP address.

#### **Step 4: Restart the backend service**

 After adding the `.env` file, you need to restart the backend service using the following command:
 ```
 sudo systemctl restart backend.service
```
to check if the backend service is running use the command:
```
sudo systemctl status backend.service
```
#### **Step 5: Test the application**

To check if the application is running run the command:
```
sudo systemctl status frontend.service
```
Then use the external Ip address to open the application in the browser.
>  **Note:** Make sure that you add inbound rule in the network security group to allow the port 8501 for the streamlit.

#### **Step 6: preparing files for the CI/CD in GitHub Action**
First, create a file with the name `update_app.sh` in the repository directory. Then, copy and paste this script:
```
#!/bin/bash

set  -e

  

date

echo  "Updating Python application on VM..."

  

HOME_DIR=$(eval  echo  ~$USER)

APP_DIR="$HOME_DIR/SDA-Chatbot-Project"

REPO_URL="https://github.com/<your_github_account>/SDA-Chatbot-Project.git"

BRANCH="main"

GITHUB_TOKEN=$TOKEN  # Passed securely via protectedSettings

  

# Update code

if [ -d  "$APP_DIR" ]; then

sudo  -u  azureuser  bash  -c  "cd $APP_DIR && git fetch origin && git reset --hard origin/$BRANCH"

else

sudo  -u  azureuser  git  clone  -b  "$BRANCH"  "https://${GITHUB_TOKEN}@${REPO_URL}"  "$APP_DIR"

fi

  

# Install dependencies

sudo  -u  azureuser  $HOME_DIR/miniconda3/envs/project/bin/pip  install  --upgrade  pip

sudo  -u  azureuser  $HOME_DIR/miniconda3/envs/project/bin/pip  install  -r  "${APP_DIR}/requirements.txt"

  

# Restart the service

sudo  systemctl  restart  backend

sudo  systemctl  is-active  --quiet  backend  ||  echo  "Backend failed to start"

sudo  systemctl  restart  frontend

sudo  systemctl  is-active  --quiet  frontend  ||  echo  "frontend failed to start"

  

echo  "Python application update completed!"
```

Then, Create a workflow directory in the repository directory like this:
```.github/workflows/deploy.yml```

After that past this yaml script:
```
name: Python App CI/CD Pipeline with Direct Deployment

  

on:

push:

branches:

- main

  

jobs:

build-and-deploy:

runs-on: ubuntu-latest

  

steps:

# Checkout the repository

- name: Checkout code

uses: actions/checkout@v4

  

# Set up Python environment

- name: Set up Python

uses: actions/setup-python@v5

with:

python-version: '3.11'

cache: 'pip'  # caching pip dependencies

- run: |

pip install -r requirements.txt

  

# Run tests (Placeholder for actual tests)

- name: Run tests

run: |

echo "Run tests"

  

# Azure Login using GitHub Secrets

- name: Azure Login

uses: azure/login@v2.2.0

with:

creds: ${{ secrets.AZURE_CREDENTIALS }}

  

# Execute the Update Script on the VM

- name: Deploy to Azure VM

run: |

az vm extension set \

--resource-group ${{ secrets.RESOURCE_GROUP }} \

--vm-name ${{ secrets.VM_NAME }} \

--name CustomScript \

--force-update \

--publisher Microsoft.Azure.Extensions \

--settings '{"fileUris": []}' \

--protected-settings '{"commandToExecute": "export GITHUB_TOKEN=${{ secrets.TOKEN }} && sudo -u azureuser bash /home/azureuser/SDA-Chatbot-Project/update_app.sh"}'
```
>  **Note:** make sure to create a secrets in the repository, 

 - AZURE_CREDENTIALS
 - RESOURCE_GROUP
 - TOKEN: Personal access Token for GitHub.
 - VM_NAME

#### **Step 7: Make changes and push the main**
try changing something in the chatbot code and then push it to the main.