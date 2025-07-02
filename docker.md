# Experiment 7: Docker Web Hosting on AWS EC2

## Overview
Create a simple web hosting setup on Amazon EC2 using Docker for containerization. This experiment involves creating a React application, containerizing it with Docker, and deploying it on EC2.

## Prerequisites
- AWS Account with EC2 access
- Basic knowledge of React and Docker
- SSH client (PuTTY for Windows)

## Architecture
- **EC2 Instance**: Ubuntu 22.04 LTS
- **Application**: React.js web app
- **Container**: Docker container running the React app
- **Port**: 3000 (HTTP)

## Step-by-Step Instructions

### Step 1: Launch EC2 Instance
1. Go to AWS Console → EC2 → Launch Instance
2. **Instance Configuration:**
   - AMI: **Ubuntu 22.04 LTS**
   - Instance type: **t2.micro**
   - Key pair: Create or use existing
3. **Security Group Settings:**
   - Allow SSH (Port 22) from your IP
   - Allow Custom TCP (Port 3000) from Anywhere (0.0.0.0/0)
4. Launch the instance

### Step 2: Connect to EC2 Instance
```bash
# Using SSH (Linux/Mac)
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>

# Or use PuTTY (Windows) with your .ppk file
```

### Step 3: Install Node.js and npm
```bash
# Install Node.js 16.x
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node -v
npm -v
```

### Step 4: Install Git (if needed)
```bash
sudo apt install git -y
```

### Step 5: Create React Application
```bash
# Create React app
npx create-react-app rollno-app

# Navigate to app directory
cd rollno-app
```

### Step 6: Create Welcome Component
```bash
# Create welcome.js file
nano src/welcome.js
```

**Copy and paste this code:**
```javascript
import React, { useState } from 'react';

const WelcomeMessage = () => {
  const [name, setName] = useState('Ridhi');
  const [showMessage, setShowMessage] = useState(false);

  const handleButtonClick = () => {
    setShowMessage(true);
  };

  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <button onClick={handleButtonClick}>Show Welcome Message</button>
      {showMessage && <p>Hi "{name}!!!" Welcome to Docker Image</p>}
    </div>
  );
};

export default WelcomeMessage;
```

### Step 7: Modify App.js
```bash
# Edit App.js
nano src/App.js
```

**Replace content with:**
```javascript
import React from 'react';
import WelcomeUser from './welcome';

function App() {
  return (
    <div className="App">
      <WelcomeUser />
    </div>
  );
}

export default App;
```

### Step 8: Test React App (Optional)
```bash
# Start development server
npm start

# App will be available at http://<EC2_PUBLIC_IP>:3000
# Press Ctrl+C to stop the server
```

### Step 9: Build React App
```bash
# Create production build
npm run build
```

## Docker Setup

### Step 10: Install Docker
```bash
# Update package index
sudo apt update

# Install Docker
sudo apt install docker.io -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
```

### Step 11: Create Dockerfile
```bash
# Create Dockerfile in project root (not in src/)
nano Dockerfile
```

**Copy and paste this content:**
```dockerfile
# Use Node.js official image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy app source code
COPY . .

# Build the app
RUN npm run build

# Install serve to run the built app
RUN npm install -g serve

# Expose port 3000
EXPOSE 3000

# Start the application
CMD ["serve", "-s", "build", "-l", "3000"]
```

### Step 12: Create .dockerignore
```bash
# Create .dockerignore file
nano .dockerignore
```

**Add these lines:**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.cache
```

### Step 13: Build Docker Image
```bash
# Build Docker image (don't forget the dot at the end)
sudo docker build -t rollno-react-app .
```

### Step 14: Run Docker Container
```bash
# Run container in detached mode
sudo docker run -d -p 3000:3000 rollno-react-app

# Check if container is running
sudo docker ps
```

### Step 15: Access Your Application
1. Open your web browser
2. Navigate to: `http://<EC2_PUBLIC_IP>:3000`
3. You should see:
   - An input field with "Enter your name"
   - A button "Show Welcome Message" 
   - When clicked, displays: "Hi [name]!!! Welcome to Docker Image"

## Useful Docker Commands

```bash
# List running containers
sudo docker ps

# List all containers
sudo docker ps -a

# Stop a container
sudo docker stop <container_id>

# Remove a container
sudo docker rm <container_id>

# List Docker images
sudo docker images

# Remove an image
sudo docker rmi <image_id>

# View container logs
sudo docker logs <container_id>
```

## Troubleshooting

### Port 3000 Not Accessible
- Check EC2 Security Group allows inbound traffic on port 3000
- Verify container is running: `sudo docker ps`
- Check if port is being used: `sudo netstat -tlnp | grep 3000`

### Docker Build Fails
- Ensure you're in the correct directory (where package.json exists)
- Check Dockerfile syntax
- Verify all files are present

### Container Exits Immediately
- Check container logs: `sudo docker logs <container_id>`
- Verify the CMD instruction in Dockerfile
- Ensure build was successful

## Security Considerations
- Restrict Security Group rules to specific IP ranges when possible
- Keep Docker and system packages updated
- Use specific version tags for Docker images in production
- Regularly scan images for vulnerabilities

## Clean Up
To avoid charges:
```bash
# Stop and remove containers
sudo docker stop $(sudo docker ps -q)
sudo docker rm $(sudo docker ps -aq)

# Remove images
sudo docker rmi rollno-react-app

# Terminate EC2 instance from AWS Console
```

## Next Steps
- Add HTTPS with SSL certificates
- Implement CI/CD pipeline
- Use Docker Compose for multi-container setup
- Deploy to Amazon ECS or EKS for production