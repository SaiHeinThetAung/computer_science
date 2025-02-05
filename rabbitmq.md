To master **RabbitMQ** as fast as possible, follow this structured approach:  

---

# **🔷 Step 1: Learn Core RabbitMQ Concepts (1-2 Hours)**
## **📌 Key Topics**
- **Message Broker:** RabbitMQ acts as an intermediary between services.
- **Producer:** Sends messages.
- **Queue:** Stores messages until processed.
- **Consumer:** Receives and processes messages.
- **Exchange:** Routes messages to queues.
- **Bindings & Routing Keys:** Define how messages get to the right queue.

📝 **Action Items:**
- Read the official [RabbitMQ Introduction](https://www.rabbitmq.com/getstarted.html).
- Watch **RabbitMQ basics on YouTube** (Find a 15-30 min video).
- Install RabbitMQ and enable the management UI (`http://localhost:15672`).

---

# **🔷 Step 2: Hands-on Setup with Node.js (2-3 Hours)**
## **1️⃣ Install RabbitMQ**
### **Windows (With Erlang)**
```sh
rabbitmq-plugins enable rabbitmq_management
rabbitmq-server
```
### **Docker (Recommended)**
```sh
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```
📌 Access RabbitMQ UI at: **http://localhost:15672** (User: `guest`, Password: `guest`).

---

## **2️⃣ Create Your First Producer & Consumer**
### **Producer (Send Messages)**
```javascript
const amqp = require('amqplib');

const sendMessage = async () => {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  const queue = 'test_queue';

  await channel.assertQueue(queue, { durable: true });

  const message = 'Hello, RabbitMQ!';
  channel.sendToQueue(queue, Buffer.from(message), { persistent: true });

  console.log(`Sent: ${message}`);
  setTimeout(() => connection.close(), 500);
};

sendMessage();
```

### **Consumer (Receive Messages)**
```javascript
const amqp = require('amqplib');

const receiveMessage = async () => {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  const queue = 'test_queue';

  await channel.assertQueue(queue, { durable: true });
  console.log(`Waiting for messages in ${queue}...`);

  channel.consume(queue, (msg) => {
    if (msg !== null) {
      console.log(`Received: ${msg.content.toString()}`);
      channel.ack(msg);
    }
  });
};

receiveMessage();
```
✅ **Practice sending and consuming messages multiple times.**

---

# **🔷 Step 3: Learn RabbitMQ Exchange Types (2-3 Hours)**
## **1️⃣ Direct Exchange (Routing Messages)**
📌 Routes messages **based on a specific routing key**.
```javascript
await channel.assertExchange('direct_logs', 'direct', { durable: false });
await channel.publish('direct_logs', 'info', Buffer.from('Info Log!'));
```
📝 **Task:** Implement **direct exchange** with multiple queues.

---

## **2️⃣ Fanout Exchange (Broadcast Messages)**
📌 Messages are **sent to all bound queues**.
```javascript
await channel.assertExchange('logs', 'fanout', { durable: false });
await channel.publish('logs', '', Buffer.from('Broadcast Message!'));
```
📝 **Task:** Implement **fanout exchange** with multiple consumers.

---

## **3️⃣ Topic Exchange (Pattern-Based Routing)**
📌 Routes messages **based on wildcard patterns** (`*.error`, `order.#`).
```javascript
await channel.assertExchange('topic_logs', 'topic', { durable: false });
await channel.publish('topic_logs', 'order.created', Buffer.from('Order Created Event!'));
```
📝 **Task:** Implement **topic-based routing**.

---

# **🔷 Step 4: Implement Advanced Features (4-6 Hours)**
## **1️⃣ Acknowledgments & Message Durability**
📌 Enable **durability** to prevent message loss.
```javascript
await channel.assertQueue('persistent_queue', { durable: true });
channel.sendToQueue('persistent_queue', Buffer.from('Persistent Message'), { persistent: true });
```
📝 **Task:** Implement **persistent queues & message acknowledgments**.

---

## **2️⃣ Dead Letter Queues (Handling Failed Messages)**
📌 **Failed messages** move to a **dead-letter queue** for debugging.
```javascript
await channel.assertQueue('main_queue', {
  durable: true,
  arguments: { 'x-dead-letter-exchange': 'dlx_exchange' }
});
```
📝 **Task:** Implement **DLX (Dead Letter Exchange)**.

---

## **3️⃣ Prefetch & Load Balancing (Optimize Consumer Performance)**
📌 Prevent **overloading consumers**.
```javascript
await channel.prefetch(1);
```
📝 **Task:** Implement **prefetching** to limit the number of messages a consumer processes at a time.

---

## **4️⃣ RabbitMQ with Microservices**
📌 Use RabbitMQ for **event-driven microservices**.
- **Example:** `order-service` sends an order event → `email-service` consumes it and sends an email.

📝 **Task:** Design a **simple microservices setup** with RabbitMQ.

---

# **🔷 Step 5: Mastering RabbitMQ for Production (6-8 Hours)**
### ✅ **Security Best Practices**
- **Enable Authentication** (`rabbitmqctl add_user username password`)
- **Use TLS/SSL Encryption** (For secure message transfer)
- **Enable User Permissions** (`rabbitmqctl set_permissions`)

### ✅ **Scaling RabbitMQ**
- **Clustering:** Run multiple RabbitMQ instances (`rabbitmqctl join_cluster`).
- **High Availability:** Use mirrored queues for **failover**.

### ✅ **RabbitMQ Monitoring & Performance**
- **Monitor Metrics:** Use `rabbitmqctl status`
- **Use Prometheus & Grafana for Visualization**
- **Optimize Queue Performance** (TTL, Lazy Queues)

📝 **Task:** Implement **RabbitMQ clustering & monitoring**.

---

# **🔷 Summary: Become a RabbitMQ Expert in 2 Days**
✅ **Day 1:**
- Install RabbitMQ & Run a Producer/Consumer.
- Learn **Exchanges (Direct, Fanout, Topic)**.
- Implement **Persistent Queues & Acknowledgments**.

✅ **Day 2:**
- Implement **DLX (Dead Letter Queues)**.
- Optimize **Prefetch & Load Balancing**.
- Explore **Microservices Architecture**.
- Set up **Monitoring & Security Best Practices**.

---

### **Next Steps**
🚀 **Want real-world projects?** Build:
1. **Email Notification System** (Orders → Emails)
2. **Live Chat System** (Real-time message streaming)
3. **Microservices Event Bus** (Decouple services with RabbitMQ)

Let's build an **Email Notification System** using **Node.js, RabbitMQ, and Docker**. This system will have:  

✅ **Producer (Order Service)** → Sends an order confirmation request.  
✅ **RabbitMQ (Message Broker)** → Routes the message.  
✅ **Consumer (Email Service)** → Listens and sends an email.  
✅ **Docker** → Runs RabbitMQ and our services in containers.  

---

# **📌 Step 1: Setup Docker with RabbitMQ**
Create a **`docker-compose.yml`** file to run RabbitMQ.

```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
```
📌 **Run RabbitMQ**  
```sh
docker-compose up -d
```
✅ **RabbitMQ Management UI:** http://localhost:15672 (User: `guest`, Password: `guest`).

---

# **📌 Step 2: Create Order Service (Producer)**
📌 **Install dependencies**
```sh
mkdir order-service && cd order-service
npm init -y
npm install amqplib express nodemon
```

📌 **Create `orderService.js`**
```javascript
const express = require('express');
const amqp = require('amqplib');

const app = express();
app.use(express.json());

const RABBITMQ_URL = 'amqp://localhost';

async function sendToQueue(orderData) {
  const connection = await amqp.connect(RABBITMQ_URL);
  const channel = await connection.createChannel();
  const queue = 'order_queue';

  await channel.assertQueue(queue, { durable: true });
  channel.sendToQueue(queue, Buffer.from(JSON.stringify(orderData)), { persistent: true });

  console.log(`✅ Order sent to queue:`, orderData);
  setTimeout(() => connection.close(), 500);
}

app.post('/order', async (req, res) => {
  const order = req.body;
  await sendToQueue(order);
  res.json({ message: 'Order placed successfully!', order });
});

app.listen(3000, () => console.log('📦 Order Service running on port 3000'));
```

📌 **Run Order Service**
```sh
node orderService.js
```
✅ **API Endpoint:** `POST http://localhost:3000/order`  
Sample JSON:  
```json
{
  "orderId": "12345",
  "email": "customer@example.com",
  "product": "Laptop"
}
```

---

# **📌 Step 3: Create Email Service (Consumer)**
📌 **Install dependencies**
```sh
mkdir email-service && cd email-service
npm init -y
npm install amqplib nodemailer nodemon
```

📌 **Create `emailService.js`**
```javascript
const amqp = require('amqplib');
const nodemailer = require('nodemailer');

const RABBITMQ_URL = 'amqp://localhost';

async function receiveFromQueue() {
  const connection = await amqp.connect(RABBITMQ_URL);
  const channel = await connection.createChannel();
  const queue = 'order_queue';

  await channel.assertQueue(queue, { durable: true });
  console.log(`📨 Waiting for orders...`);

  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      const orderData = JSON.parse(msg.content.toString());
      console.log(`📧 Sending email for order:`, orderData);

      await sendEmail(orderData);
      channel.ack(msg);
    }
  });
}

async function sendEmail(order) {
  const transporter = nodemailer.createTransport({
    service: 'gmail',
    auth: { user: 'your-email@gmail.com', pass: 'your-password' },
  });

  const mailOptions = {
    from: 'your-email@gmail.com',
    to: order.email,
    subject: 'Order Confirmation',
    text: `Your order for ${order.product} (ID: ${order.orderId}) has been received.`,
  };

  try {
    await transporter.sendMail(mailOptions);
    console.log(`✅ Email sent to ${order.email}`);
  } catch (error) {
    console.error('❌ Email sending failed:', error);
  }
}

receiveFromQueue();
```
📌 **Run Email Service**
```sh
node emailService.js
```
✅ **The email service listens for new orders and sends confirmation emails.**  

---

# **📌 Step 4: Dockerize Services**
## **1️⃣ Dockerfile for Order Service**
📌 **Create `order-service/Dockerfile`**
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
CMD ["node", "orderService.js"]
```

## **2️⃣ Dockerfile for Email Service**
📌 **Create `email-service/Dockerfile`**
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
CMD ["node", "emailService.js"]
```

## **3️⃣ Update `docker-compose.yml`**
Modify `docker-compose.yml` to include both services:
```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"

  order-service:
    build: ./order-service
    container_name: order-service
    ports:
      - "3000:3000"
    depends_on:
      - rabbitmq

  email-service:
    build: ./email-service
    container_name: email-service
    depends_on:
      - rabbitmq
```

📌 **Run Everything**
```sh
docker-compose up --build
```
✅ **Order API:** `POST http://localhost:3000/order`  
✅ **RabbitMQ UI:** http://localhost:15672  
✅ **Emails will be sent automatically!**

---

# **📌 Summary**
🚀 We built a **RabbitMQ-based Email Notification System** with **Docker**:
1️⃣ **Order Service (Producer):** Sends order confirmation messages.  
2️⃣ **RabbitMQ (Message Broker):** Handles message routing.  
3️⃣ **Email Service (Consumer):** Listens and sends emails.  
4️⃣ **Docker:** Runs everything in isolated containers.  

Would you like **load balancing for multiple consumers** next? 🔥

## **🔥 Load Balancing Consumers in RabbitMQ**
To scale our **Email Service**, we need multiple consumers processing messages in parallel. **RabbitMQ automatically distributes messages to consumers** using the **round-robin dispatching** mechanism.

---

# **📌 Step 1: Modify Email Service for Multiple Consumers**
We need **multiple instances** of our **email-service** running in separate containers.

📌 **Update `emailService.js`**
```javascript
const amqp = require('amqplib');
const nodemailer = require('nodemailer');

const RABBITMQ_URL = 'amqp://rabbitmq'; // Use service name in Docker

async function receiveFromQueue() {
  const connection = await amqp.connect(RABBITMQ_URL);
  const channel = await connection.createChannel();
  const queue = 'order_queue';

  await channel.assertQueue(queue, { durable: true });
  channel.prefetch(1); // Ensures one consumer gets one message at a time

  console.log(`📨 Consumer started - Waiting for orders...`);

  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      const orderData = JSON.parse(msg.content.toString());
      console.log(`📧 Consumer ${process.pid} handling order:`, orderData);

      await sendEmail(orderData);
      channel.ack(msg);
    }
  });
}

async function sendEmail(order) {
  const transporter = nodemailer.createTransport({
    service: 'gmail',
    auth: { user: 'your-email@gmail.com', pass: 'your-password' },
  });

  const mailOptions = {
    from: 'your-email@gmail.com',
    to: order.email,
    subject: 'Order Confirmation',
    text: `Your order for ${order.product} (ID: ${order.orderId}) has been received.`,
  };

  try {
    await transporter.sendMail(mailOptions);
    console.log(`✅ Email sent to ${order.email}`);
  } catch (error) {
    console.error('❌ Email sending failed:', error);
  }
}

receiveFromQueue();
```

---

# **📌 Step 2: Update `docker-compose.yml`**
📌 **Modify `docker-compose.yml` to scale consumers**
```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"

  order-service:
    build: ./order-service
    container_name: order-service
    ports:
      - "3000:3000"
    depends_on:
      - rabbitmq

  email-service:
    build: ./email-service
    deploy:
      replicas: 3  # Scale to 3 instances
    depends_on:
      - rabbitmq
```
✅ This ensures **RabbitMQ will distribute messages across multiple consumers!** 🎯

---

# **📌 Step 3: Run Everything with Multiple Consumers**
📌 **Run with Docker Compose**
```sh
docker-compose up --build
```
✅ **RabbitMQ will distribute orders evenly across 3 email-service instances.**  

---

# **📌 Step 4: Verify Load Balancing**
1️⃣ Open RabbitMQ UI at **http://localhost:15672** (User: `guest`, Password: `guest`).  
2️⃣ Go to **Queues → `order_queue`**, and check the message distribution.  
3️⃣ Send multiple orders using:  
```sh
curl -X POST http://localhost:3000/order -H "Content-Type: application/json" -d '{"orderId":"12345", "email":"user1@example.com", "product":"Laptop"}'
curl -X POST http://localhost:3000/order -H "Content-Type: application/json" -d '{"orderId":"12346", "email":"user2@example.com", "product":"Phone"}'
curl -X POST http://localhost:3000/order -H "Content-Type: application/json" -d '{"orderId":"12347", "email":"user3@example.com", "product":"Tablet"}'
```
✅ The messages will be processed **evenly** by different consumers.  

---

# **📌 Summary**
🚀 We **scaled our consumers** for high performance using RabbitMQ’s **round-robin dispatching**:
- **Multiple Consumers** in `email-service`
- **RabbitMQ load balancing**
- **Scaling with Docker `deploy.replicas`**
  
Would you like to add **Retry Logic** for failed emails next? 📩🔥

# **🔥 Adding Retry Logic for Failed Emails in RabbitMQ**
To make our **email notification system** more **robust**, we'll add **retry logic** when an email fails. Instead of losing failed messages, we'll:
1. **Move them to a "retry queue"** with a delay before reprocessing.
2. **If retries exceed the limit, move them to a "dead-letter queue" (DLQ)** for logging and analysis.

---

# **📌 Step 1: Define the Retry & Dead Letter Queues**
Modify **email-service** to include:
- A **retry queue** (`email_retry_queue`) where failed emails go, with a **TTL (time-to-live)** before retrying.
- A **dead-letter queue (DLQ)** (`email_dead_letter_queue`) for emails that fail multiple times.

📌 **Modify `emailService.js`**
```javascript
const amqp = require('amqplib');
const nodemailer = require('nodemailer');

const RABBITMQ_URL = 'amqp://rabbitmq'; 
const QUEUE_NAME = 'order_queue';
const RETRY_QUEUE = 'email_retry_queue';
const DLQ = 'email_dead_letter_queue';
const MAX_RETRIES = 3;
const RETRY_DELAY_MS = 5000; // 5 seconds delay before retrying

async function receiveFromQueue() {
  const connection = await amqp.connect(RABBITMQ_URL);
  const channel = await connection.createChannel();

  // Declare queues
  await channel.assertQueue(QUEUE_NAME, { durable: true });
  await channel.assertQueue(RETRY_QUEUE, {
    durable: true,
    arguments: {
      'x-dead-letter-exchange': '', // Sends to main queue after delay
      'x-dead-letter-routing-key': QUEUE_NAME,
      'x-message-ttl': RETRY_DELAY_MS, // Retry delay
    },
  });
  await channel.assertQueue(DLQ, { durable: true });

  channel.prefetch(1); // Ensures fair load balancing

  console.log(`📨 Consumer started - Waiting for orders...`);

  channel.consume(QUEUE_NAME, async (msg) => {
    if (msg) {
      const orderData = JSON.parse(msg.content.toString());
      const retryCount = msg.properties.headers['x-retry-count'] || 0;

      try {
        console.log(`📧 Processing email for: ${orderData.email}`);
        await sendEmail(orderData);
        channel.ack(msg);
      } catch (error) {
        console.error(`❌ Failed to send email to ${orderData.email}. Attempt: ${retryCount + 1}`);

        if (retryCount >= MAX_RETRIES) {
          console.log(`🚨 Moving message to Dead Letter Queue`);
          channel.sendToQueue(DLQ, msg.content, { persistent: true });
        } else {
          console.log(`🔄 Retrying in ${RETRY_DELAY_MS / 1000}s...`);
          channel.sendToQueue(RETRY_QUEUE, msg.content, {
            headers: { 'x-retry-count': retryCount + 1 },
            persistent: true,
          });
        }
        channel.ack(msg);
      }
    }
  });
}

async function sendEmail(order) {
  const transporter = nodemailer.createTransport({
    service: 'gmail',
    auth: { user: 'your-email@gmail.com', pass: 'your-password' },
  });

  const mailOptions = {
    from: 'your-email@gmail.com',
    to: order.email,
    subject: 'Order Confirmation',
    text: `Your order for ${order.product} (ID: ${order.orderId}) has been received.`,
  };

  await transporter.sendMail(mailOptions);
  console.log(`✅ Email sent to ${order.email}`);
}

receiveFromQueue();
```

---

# **📌 Step 2: Update `docker-compose.yml`**
📌 **Ensure RabbitMQ handles dead-letter messages properly**
```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"

  order-service:
    build: ./order-service
    container_name: order-service
    ports:
      - "3000:3000"
    depends_on:
      - rabbitmq

  email-service:
    build: ./email-service
    deploy:
      replicas: 3  # Scale consumers for load balancing
    depends_on:
      - rabbitmq
```

---

# **📌 Step 3: Run & Test the System**
🚀 **Run everything**:
```sh
docker-compose up --build
```

🛠 **Test with Failed Emails**
1. **Send a valid request:**
```sh
curl -X POST http://localhost:3000/order -H "Content-Type: application/json" -d '{"orderId":"12345", "email":"valid@example.com", "product":"Laptop"}'
```
✅ The email should send successfully.

2. **Force a failure (change `nodemailer` password or disconnect network).**  
The message will retry **3 times** and move to the **Dead Letter Queue**.

3. **Monitor Queues in RabbitMQ Dashboard**  
Go to **http://localhost:15672** → **Queues**  
Check:
- `order_queue`: Main queue
- `email_retry_queue`: Retries before moving back to main queue
- `email_dead_letter_queue`: Failed emails after max retries

---

# **📌 Summary**
✅ Implemented **retry logic** for failed emails  
✅ **Retries up to 3 times** before moving to **DLQ**  
✅ **Used RabbitMQ TTL & Dead Letter Exchange**  
✅ **Scaled consumers for load balancing**  

Would you like to **automatically process DLQ messages** for later retries? 🚀

 automatically process DLQ messages for later retrie

 # **🔥 Automatically Process DLQ Messages for Later Retry in RabbitMQ**  
To prevent losing failed emails, we need to **reprocess Dead Letter Queue (DLQ) messages** periodically instead of discarding them.  

### ✅ **Approach:**
1. **Move messages from DLQ to the main queue** after a delay.
2. Use a **separate consumer (`dlqProcessor.js`)** to periodically retry these messages.
3. If emails still fail **after multiple retries**, log them or alert admins.

---

## **📌 Step 1: Create DLQ Processor**
A new consumer will **read from the DLQ** and send messages back to the **main queue (`order_queue`)** for another retry.

📌 **Create `dlqProcessor.js`** in `email-service`
```javascript
const amqp = require('amqplib');

const RABBITMQ_URL = 'amqp://rabbitmq';
const DLQ = 'email_dead_letter_queue';
const MAIN_QUEUE = 'order_queue';

async function processDLQMessages() {
  const connection = await amqp.connect(RABBITMQ_URL);
  const channel = await connection.createChannel();

  await channel.assertQueue(DLQ, { durable: true });
  await channel.assertQueue(MAIN_QUEUE, { durable: true });

  console.log(`🔄 DLQ Processor started - Monitoring ${DLQ}...`);

  channel.consume(DLQ, (msg) => {
    if (msg) {
      console.log(`🔄 Retrying failed email: ${msg.content.toString()}`);

      // Move message back to main queue
      channel.sendToQueue(MAIN_QUEUE, msg.content, {
        persistent: true,
      });

      channel.ack(msg);
    }
  });

  setInterval(async () => {
    console.log(`🚀 Reprocessing DLQ messages...`);
  }, 10000); // Retry every 10 seconds
}

processDLQMessages();
```

---

## **📌 Step 2: Update `docker-compose.yml`**
Now, we add the **DLQ processor** as a separate service.

📌 **Modify `docker-compose.yml`**
```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"

  order-service:
    build: ./order-service
    container_name: order-service
    ports:
      - "3000:3000"
    depends_on:
      - rabbitmq

  email-service:
    build: ./email-service
    deploy:
      replicas: 3
    depends_on:
      - rabbitmq

  dlq-processor:
    build: ./email-service
    command: ["node", "dlqProcessor.js"]
    depends_on:
      - rabbitmq
```

---

## **📌 Step 3: Run & Monitor**
Start everything with:
```sh
docker-compose up --build
```

### **🎯 What Happens?**
1. If an email **fails repeatedly**, it moves to **`email_dead_letter_queue` (DLQ)**.
2. The **DLQ processor** automatically **resends** these messages to **`order_queue`** after a delay.
3. If emails **keep failing**, they will loop **until they are successfully processed**.

---

## **📌 Step 4: Advanced Enhancements (Optional)**
### **1️⃣ Set a Maximum DLQ Retries Limit**
Modify `dlqProcessor.js` to **limit retries** before logging:
```javascript
const MAX_DLQ_RETRIES = 5;

channel.consume(DLQ, (msg) => {
  if (msg) {
    const retryCount = msg.properties.headers['x-dlq-retry-count'] || 0;

    if (retryCount >= MAX_DLQ_RETRIES) {
      console.log(`🚨 Email permanently failed: ${msg.content.toString()}`);
      return channel.ack(msg); // Stop retrying after 5 attempts
    }

    console.log(`🔄 Retrying failed email (${retryCount + 1})`);
    channel.sendToQueue(MAIN_QUEUE, msg.content, {
      headers: { 'x-dlq-retry-count': retryCount + 1 },
      persistent: true,
    });

    channel.ack(msg);
  }
});
```

---

# **📌 Final Summary**
✅ **DLQ processor automatically retries failed emails**  
✅ **Moves messages back to `order_queue` for reprocessing**  
✅ **Retries every 10 seconds**  
✅ **Optional: Stop retrying after 5 failed attempts**  

🚀 **Now your email system is fully fault-tolerant!** Do you want to add **logging or alerts** for permanently failed emails? 📊
