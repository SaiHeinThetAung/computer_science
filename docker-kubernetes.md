Let's build a simple **To-Do List** using React and then **Dockerize** it step by step.  

---

## **Step 1: Set Up a React App**
1. Open a terminal and create a new React app:  
   ```sh
   npx create-react-app todo-app
   ```
2. Navigate into the project folder:  
   ```sh
   cd todo-app
   ```
3. Start the development server:  
   ```sh
   npm start
   ```
   Your app should now be running at `http://localhost:3000`.

---

## **Step 2: Create a Simple To-Do List**
Replace the content of `src/App.js` with the following:  

```jsx
import React, { useState } from "react";

function App() {
  const [tasks, setTasks] = useState([]);
  const [input, setInput] = useState("");

  const addTask = () => {
    if (input.trim()) {
      setTasks([...tasks, input]);
      setInput("");
    }
  };

  const removeTask = (index) => {
    setTasks(tasks.filter((_, i) => i !== index));
  };

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h1>Simple To-Do List</h1>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Add a task"
      />
      <button onClick={addTask}>Add</button>
      <ul>
        {tasks.map((task, index) => (
          <li key={index}>
            {task} <button onClick={() => removeTask(index)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```
---

## **Step 3: Create a Dockerfile**
Inside the `todo-app` folder, create a file named **Dockerfile** (without any extension) and add the following content:

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:18

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to install dependencies first
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the app files
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```

---

## **Step 4: Create a .dockerignore File**
Inside the `todo-app` folder, create a file named **.dockerignore** and add:

```
node_modules
npm-debug.log
build
```

This prevents unnecessary files from being copied to the Docker container.

---

## **Step 5: Build the Docker Image**
1. Open a terminal in the `todo-app` directory.
2. Run the following command to build the Docker image:  
   ```sh
   docker build -t todo-app .
   ```
   This will create a Docker image named `todo-app`.

---

## **Step 6: Run the Docker Container**
Once the build is complete, start a container from the image:  
```sh
docker run -p 3000:3000 todo-app
```
Now, go to `http://localhost:3000` in your browser to see your To-Do List running inside Docker!

---

## **Step 7: Stop and Remove the Container**
To stop the running container, first, find its **Container ID** by running:  
```sh
docker ps
```
Then stop it using:  
```sh
docker stop <container_id>
```
To remove it:  
```sh
docker rm <container_id>
```

---

## **Step 8: Push Docker Image to Docker Hub (Optional)**
If you want to share your Docker image:
1. Log in to Docker Hub:  
   ```sh
   docker login
   ```
2. Tag your image:  
   ```sh
   docker tag todo-app your-dockerhub-username/todo-app
   ```
3. Push the image:  
   ```sh
   docker push your-dockerhub-username/todo-app
   ```

---

## **Next Steps**
- **Docker Compose**: To manage multi-container apps (React + Backend).  
- **Kubernetes**: For deploying React apps at scale.  
- **CI/CD Pipelines**: Automate Docker builds and deployments.  

Would you like to explore any of these next? 🚀

Great! Let's start with **Docker Compose**, then move to **Kubernetes**, and finally **CI/CD pipelines** for automated deployments. 🚀  

---

# **Step 9: Using Docker Compose (React + Backend Example)**  
Docker Compose allows you to manage multiple containers together. Let’s say we want to add a **Node.js backend** to our **React To-Do List**.

### **1️⃣ Create a Node.js Backend**  
Inside your `todo-app` folder, create a new folder named `backend` and add the following files:  

**`backend/server.js`**  
```javascript
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

let tasks = [];

app.get("/tasks", (req, res) => {
  res.json(tasks);
});

app.post("/tasks", (req, res) => {
  const { task } = req.body;
  if (task) {
    tasks.push(task);
    res.json({ message: "Task added successfully" });
  } else {
    res.status(400).json({ error: "Task is required" });
  }
});

app.delete("/tasks/:index", (req, res) => {
  const index = req.params.index;
  tasks.splice(index, 1);
  res.json({ message: "Task deleted successfully" });
});

app.listen(5000, () => console.log("Backend running on port 5000"));
```

**`backend/package.json`**  
```json
{
  "name": "todo-backend",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  },
  "scripts": {
    "start": "node server.js"
  }
}
```

### **2️⃣ Create a Dockerfile for the Backend**
Inside the `backend` folder, create a **Dockerfile**:

```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```

### **3️⃣ Update the React App (`src/App.js`) to Use the Backend**
Modify `src/App.js` to fetch tasks from the backend:

```jsx
import React, { useState, useEffect } from "react";

function App() {
  const [tasks, setTasks] = useState([]);
  const [input, setInput] = useState("");

  useEffect(() => {
    fetch("http://localhost:5000/tasks")
      .then((res) => res.json())
      .then((data) => setTasks(data));
  }, []);

  const addTask = () => {
    if (input.trim()) {
      fetch("http://localhost:5000/tasks", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ task: input }),
      }).then(() => {
        setTasks([...tasks, input]);
        setInput("");
      });
    }
  };

  const removeTask = (index) => {
    fetch(`http://localhost:5000/tasks/${index}`, { method: "DELETE" }).then(() =>
      setTasks(tasks.filter((_, i) => i !== index))
    );
  };

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h1>Simple To-Do List</h1>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Add a task"
      />
      <button onClick={addTask}>Add</button>
      <ul>
        {tasks.map((task, index) => (
          <li key={index}>
            {task} <button onClick={() => removeTask(index)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

### **4️⃣ Create a Docker Compose File**
Inside the `todo-app` folder, create a new file named **`docker-compose.yml`**:

```yaml
version: "3"
services:
  frontend:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
```

---

### **5️⃣ Run Everything Using Docker Compose**
Now, from the `todo-app` folder, run:

```sh
docker-compose up --build
```

Your **React frontend (port 3000)** and **Node.js backend (port 5000)** will run inside Docker! 🎉  

To stop everything:  
```sh
docker-compose down
```

---

# **Step 10: Kubernetes Deployment (Basic Setup)**
Now let’s deploy this on Kubernetes.

### **1️⃣ Install Minikube (If You Don’t Have Kubernetes)**
```sh
minikube start
```

### **2️⃣ Create Kubernetes Deployment Files**
Inside `todo-app`, create a **k8s** folder and add:

#### **frontend-deployment.yml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-frontend
  template:
    metadata:
      labels:
        app: todo-frontend
    spec:
      containers:
        - name: todo-frontend
          image: todo-app:latest
          ports:
            - containerPort: 3000
```

#### **backend-deployment.yml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-backend
  template:
    metadata:
      labels:
        app: todo-backend
    spec:
      containers:
        - name: todo-backend
          image: todo-backend:latest
          ports:
            - containerPort: 5000
```

### **3️⃣ Apply Kubernetes Configuration**
Run:
```sh
kubectl apply -f k8s/frontend-deployment.yml
kubectl apply -f k8s/backend-deployment.yml
```

This will deploy the **React frontend** and **Node.js backend** in Kubernetes.

---

# **Step 11: CI/CD for Automated Deployments**
Let’s automate the **Docker build & push** using GitHub Actions.

### **1️⃣ Create a GitHub Actions Workflow**
Inside your repository, create:  
📁 `.github/workflows/docker-ci.yml`

```yaml
name: Docker CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build and push frontend
        run: |
          docker build -t your-dockerhub-username/todo-frontend:latest .
          docker push your-dockerhub-username/todo-frontend:latest

      - name: Build and push backend
        run: |
          docker build -t your-dockerhub-username/todo-backend:latest ./backend
          docker push your-dockerhub-username/todo-backend:latest
```

### **2️⃣ Add Secrets in GitHub**
1. Go to **GitHub Repository → Settings → Secrets**.
2. Add:
   - `DOCKER_USERNAME`
   - `DOCKER_PASSWORD`
   
Now, when you **push to `main`**, GitHub will **automatically build & push** the Docker images!

---

# **Next Steps**
✅ **Docker Compose** for managing multi-container apps  
✅ **Kubernetes** for deploying at scale  
✅ **CI/CD Pipelines** for automated Docker builds  

Would you like to go deeper into **AWS deployment** or **monitoring with Prometheus/Grafana** next? 🚀
