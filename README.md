# ✈️ **Travel Memory: Full-Stack Deployment on AWS**

Travel Memory is a scalable full-stack application built with the **MERN** stack (MongoDB, Express, React, Node.js). This application allows users to document and share their travel experiences, making it a perfect tool for travel enthusiasts. This guide walks you through the steps required to deploy the **Travel Memory** application on **Amazon EC2**, integrate it with **Cloudflare** for DNS management, set up a **reverse proxy** using **Nginx**, and scale the application with an **AWS Load Balancer** and **Auto Scaling Groups**.

---


## 📘 **Project Description**  
**Travel Memory** is a full-stack application using the **MERN** stack. It allows users to document and share travel memories, offering an intuitive platform for organizing travel logs. This guide covers deployment on **Amazon EC2**, Nginx reverse proxy configuration, Cloudflare DNS and SSL integration, and scalability using **AWS ALB** and **Auto Scaling Groups**.

---

## 📁 **Folder Structure**

```plaintext
TravelMemory/
|
├── backend/              # Node.js Express backend code
├── frontend/             # Vite + React frontend code
├── setup-backend.sh      # Backend provisioning script
└── setup-frontend.sh     # Frontend deployment script
```

---

## ⚙️ **Prerequisites**

Ensure the following before deployment:

- ✅ **AWS Account**: Access to create EC2, ALB, ASG, and Target Groups
- ✅ **Cloudflare Domain Access**: Domain (e.g., `example.com`, `api.example.com`) for DNS and SSL
- ✅ **MongoDB Atlas URI**: Cloud-hosted MongoDB for backend `.env`
- ✅ **Security Group Rules**: Allow ports 80 (HTTP), 443 (HTTPS), and 3000 (backend)
- ✅ **Ubuntu EC2 AMI**: With internet access and permission to install dependencies

---

## 🛠 **Installation Instructions**

### 🔀 **Clone the Repository**

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory
```

### 📦 **Required Software on EC2 Instances**

Install the following:

- **Nginx**: Reverse proxy
- **Node.js v22**: Backend (Express) & frontend (Vite) runtime
- **PM2**: Backend process manager

### 🧪 **PM2 Common Commands**

```bash
pm2 start index.js --name "travel-memory-backend"
pm2 list
pm2 logs travel-memory-backend
pm2 restart travel-memory-backend
pm2 stop travel-memory-backend
pm2 delete travel-memory-backend
pm2 save
pm2 startup
```

---

### 🧰 **Backend Setup Script**

```bash
#!/bin/bash
apt-get update -y
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
apt-get install -y nodejs nginx git

cd /home/ubuntu
chmod -R 775 /home/ubuntu/
git clone -b backend https://github.com/your-repo/Travel-Memory.git
cd Travel-Memory

echo "PORT=3000" > .env
echo "MONGO_URI='your mongo url'" >> .env

npm install -y
npm install -g pm2 -y
pm2 start index.js --name "travel-memory-backend"
pm2 startup
pm2 save

rm -f /etc/nginx/sites-enabled/default

cat > /etc/nginx/sites-available/travel-memory << EOF
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

ln -s /etc/nginx/sites-available/travel-memory /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx

echo "✅ Backend deployed and accessible at api.example.com"
```

---

### 🧰 **Frontend Setup Script**

```bash
#!/bin/bash
apt-get update -y
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
apt-get install -y nodejs nginx git

cd /home/ubuntu
chmod -R 775 /home/ubuntu/
git clone -b frontend https://github.com/your-repo/Travel-Memory.git
cd Travel-Memory

echo "VITE_API_BASE_URL=https://api.example.com" > .env

npm install -y
npm run build

rm -rf /var/www/html/*
cp -r dist/* /var/www/html/

rm -f /etc/nginx/sites-enabled/default

cat > /etc/nginx/sites-available/travel-memory-ui << EOF
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files \$uri \$uri/ /index.html;
    }
}
EOF

ln -s /etc/nginx/sites-available/travel-memory-ui /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx

echo "✅ Frontend deployed and accessible at http://example.com"
```

---

## 🧱 **Deployment Architecture Diagram**
<p align="center">
    <img src="https://github.com/user-attachments/assets/70472be8-554a-4b4e-8fbd-d71ea4cfdc1f" alt="Frontend EC2" width="800">
</p>

---

## ⚙️ **Backend Deployment Flow**

### 1. **Launch Templates**
<p align="center">
  <img src="https://github.com/user-attachments/assets/d83be5cb-ac8f-47cc-a6c1-3d5d26864f8c" alt="Frontend EC2" width="800">
  <img src="https://github.com/user-attachments/assets/14aaaefb-5a4c-49cf-8299-ca20c173f5a3" alt="Frontend EC2" width="800">
</p>

### 2. **Target Groups**
<p align="center">
  <img src="https://github.com/user-attachments/assets/7403fdda-5bf6-4471-836a-d3a39acdc3d4" alt="Screenshot 1" width="800">
  <img src="https://github.com/user-attachments/assets/5aab5f5d-dc58-4ef3-bf93-2c1848219324" alt="Screenshot 2" width="800">
  <img src="https://github.com/user-attachments/assets/230792d5-9659-45e8-b1b3-4931668c5e56" alt="Screenshot 3" width="800">
</p>

### 3. **Application Load Balancer (ALB)**
<p align="center">
  <img src="https://github.com/user-attachments/assets/739d382c-1b86-4364-a45a-a1460050c649" alt="ALB Screenshot 1" width="800">
  <img src="https://github.com/user-attachments/assets/fc6987f4-5f81-45bc-95e8-03cbf0fb59fe" alt="ALB Screenshot 2" width="800">
</p>

### 4. **Auto Scaling Groups (ASG)**
<p align="center">
  <img src="https://github.com/user-attachments/assets/f92c3189-1af5-4c76-9778-243a5cde2d00" alt="ASG Screenshot" width="800">
</p>

---

## 🌐 **Frontend Deployment Flow**

### 1. **EC2 Instance for Static Files**
<p align="center">
  <img src="https://github.com/user-attachments/assets/3694814a-82af-48b1-91a1-b087fd7b3f03" alt="EC2 Static Files" width="800">
</p>

### 2. **Cloudflare DNS Configuration (A Record)**
<p align="center">
  <img src="https://github.com/user-attachments/assets/38889044-07e0-4722-9a5e-cde98d41d48a" alt="Cloudflare A Record 1" width="800">
  <img src="https://github.com/user-attachments/assets/dfbb7c60-29c4-41f8-a0a2-a17b025ee9e6" alt="Cloudflare A Record 2" width="800">
</p>

### 3. **Application Live at Example.com**
<p align="center">
  <img src="https://github.com/user-attachments/assets/366ccbe1-4a9c-4041-8a83-83074a1e4565" alt="Application Live Screenshot" width="800">
</p>


---

## 📆 **Usage Instructions**

- Access `http://example.com` for the frontend UI
- Use `https://api.example.com` for backend API calls

---

## 🛠️ **Configuration**  

### ⚖️ **Scaling the Application**  

To ensure high availability, set up **AWS Load Balancer** and **Auto Scaling**:

1. **Create EC2 instances** for both frontend and backend.
2. **Create Target Groups**:
   - One for the frontend servers.
   - One for the backend servers.
3. **Set up the Application Load Balancer**:
   - Create listeners for HTTP/HTTPS.
   - Configure rules to forward traffic to respective target groups.
4. **Enable Auto Scaling**:
   - Set scaling policies based on CPU/memory usage to ensure optimal performance and cost efficiency.

---

### 🌐 **Cloudflare Setup**  

1. **Add your domain** to Cloudflare.
2. **Create DNS Records**:
   - **A Record**: Point to the public IP of the EC2 instance.
   - **CNAME Record**: Point to the AWS Load Balancer DNS endpoint.
     ![Screenshot 2025-04-13 211223](https://github.com/user-attachments/assets/38889044-07e0-4722-9a5e-cde98d41d48a)
     ![Screenshot 2025-04-13 212205](https://github.com/user-attachments/assets/dfbb7c60-29c4-41f8-a0a2-a17b025ee9e6)


---


## 🔐 **Security Best Practices**  
To secure your application, consider the following:
- **SSH Key Authentication**: Disable password-based logins and use SSH keys for secure access to EC2.
- **Restrict EC2 Security Group**: Allow only necessary ports (22 for SSH, 80/443 for HTTP/HTTPS).
- **Environment Variables**: Keep sensitive credentials like database URLs, API keys, etc., in `.env` files and never commit them to version control.
- **Enable SSL**: Use Cloudflare's SSL certificates for secure frontend-backend communication.
- **AWS IAM**: Restrict EC2 access using least-privilege IAM roles and policies.

---

## ❓ **Troubleshooting**  

| Issue                  | Solution                                                   |
|------------------------|------------------------------------------------------------|
| **Nginx failing to start** | Check the syntax using `nginx -t` and ensure no conflicting settings in the config. |
| **PM2 not running the app** | Ensure proper permissions and that the app is correctly configured with PM2. |
| **Backend/Frontend mismatch** | Check the API URL in `.env` and ensure it's accessible. |

---


## 🤝 **Contribution Guidelines**  
Contributions are welcome! Please adhere to the following guidelines:

1. Fork the repository.
2. Create a feature branch.
3. Write tests for new features or bug fixes.
4. Submit a Pull Request with a detailed description of the changes.

---

## 📚 **License**

This project is licensed under the MIT License.

---

✨ Happy Deploying with Travel Memory! ✨

