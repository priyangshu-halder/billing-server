# 🧾 Billing Server – POS Edge Service

A high-performance billing and printing service built with Node.js & Express, designed for Point-of-Sale (POS) systems.

This service works as a local edge server responsible for barcode scanning, cart management, bill generation, thermal printer integration, offline-first billing, and syncing data to ERP.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Responsibilities](#-responsibilities)
- [Tech Stack](#️-tech-stack)
- [Project Structure](#-project-structure)
- [Installation](#-installation)
- [API Endpoints](#-api-endpoints)
- [Printer Support](#️-printer-support)
- [Authentication](#-authentication)
- [Offline Support](#-offline-support)
- [Environment Variables](#️-environment-variables)
- [Docker Support](#-docker-support)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🚀 Overview

The Billing Server is part of a microservice-based POS system, acting as an edge computing service that handles all billing operations locally while maintaining sync with the central ERP system.

### 📌 Role in System

```
Barcode Scanner → Billing Server → Printer
                          ↓
                    ERP Server
```

---

## ✅ Responsibilities

The Billing Server handles:

- ✔ Scan barcode & manage cart
- ✔ Calculate bill totals
- ✔ Print receipts
- ✔ Store bills locally
- ✔ Sync data to ERP
- ✔ Work without internet

### ❌ Not Responsible For

The following are handled by the ERP Service:

- User authentication
- Inventory management
- Reports & analytics
- Admin dashboard

---

## 🏗️ Tech Stack

| Layer      | Tech                    |
|------------|-------------------------|
| Runtime    | Node.js                 |
| Framework  | Express.js              |
| Database   | SQLite                  |
| Printer    | node-thermal-printer    |
| API        | REST                    |
| Auth       | Service Token           |
| Deployment | Docker                  |

---

## 📁 Project Structure

```
billing-server/
│
├── src/
│   ├── app.js
│   ├── server.js
│
│   ├── config/
│   │   ├── env.js
│   │   ├── printer.config.js
│   │   └── db.config.js
│
│   ├── routes/
│   │   ├── billing.routes.js
│   │   └── health.routes.js
│
│   ├── controllers/
│   │   ├── billing.controller.js
│   │   └── sync.controller.js
│
│   ├── services/
│   │   ├── cart.service.js
│   │   ├── printer.service.js
│   │   ├── erp.service.js
│   │   └── sync.service.js
│
│   ├── models/
│   │   ├── bill.model.js
│   │   └── syncQueue.model.js
│
│   ├── jobs/
│   │   └── sync.job.js
│
│   ├── middlewares/
│   │   ├── auth.middleware.js
│   │   └── error.middleware.js
│
│   ├── utils/
│   │   ├── logger.js
│   │   ├── constants.js
│   │   └── helpers.js
│
│   └── database/
│       └── local.sqlite
│
├── .env
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── package.json
├── package-lock.json
└── README.md
```

---

## 🛠 Installation

### Prerequisites

- Node.js (v16 or higher)
- npm or yarn
- Thermal printer (optional for development)

### Steps

1. **Clone the repository**

```bash
git clone https://github.com/your-username/billing-server.git
cd billing-server
```

2. **Install dependencies**

```bash
npm install
```

3. **Set up environment variables**

```bash
cp .env.example .env
```

Edit `.env` with your configuration.

4. **Run the server**

```bash
# Development
npm run dev

# Production
npm start
```

The server will start on `http://localhost:5001`

---

## 🔌 API Endpoints

### Health Check

Check if the service is running.

```http
GET /api/health
```

**Response:**

```json
{
  "status": "OK",
  "timestamp": "2025-12-29T10:30:00.000Z"
}
```

---

### Scan Barcode

Add item to cart by scanning barcode.

```http
POST /api/billing/scan
```

**Request Body:**

```json
{
  "barcode": "8901234567890"
}
```

**Response:**

```json
{
  "success": true,
  "item": {
    "barcode": "8901234567890",
    "name": "Product Name",
    "price": 99.99,
    "quantity": 1
  },
  "cart": { ... }
}
```

---

### Get Cart

Retrieve current cart items.

```http
GET /api/billing/cart
```

**Response:**

```json
{
  "items": [ ... ],
  "total": 299.97,
  "itemCount": 3
}
```

---

### Checkout & Print

Complete the transaction and print receipt.

```http
POST /api/billing/checkout
```

**Request Body:**

```json
{
  "paymentMode": "CASH",
  "amountPaid": 300.00
}
```

**Response:**

```json
{
  "success": true,
  "billId": "BILL-2025-001",
  "total": 299.97,
  "change": 0.03,
  "printed": true
}
```

---

### Sync Bill to ERP

Manually trigger sync of pending bills to ERP.

```http
POST /api/billing/sync
```

**Response:**

```json
{
  "synced": 5,
  "pending": 0,
  "failed": 0
}
```

---

## 🖨️ Printer Support

This service supports thermal printers with ESC/POS compatibility.

### Supported Printers

- ✔ Thermal printers (58mm, 80mm)
- ✔ ESC/POS compatible
- ✔ USB connection
- ✔ Network printers

### Library Used

```
node-thermal-printer
```

### Configuration

Configure printer settings in `src/config/printer.config.js` or via environment variables.

---

## 🔐 Authentication

| Source          | Method         |
|-----------------|----------------|
| ERP → Billing   | Service Token  |
| Scanner         | Device-based   |
| Frontend        | Not allowed    |

All requests to the Billing Server must include a valid service token in the `Authorization` header.

```http
Authorization: Bearer YOUR_SERVICE_TOKEN
```

---

## 🔁 Offline Support

The Billing Server is designed to work seamlessly in offline mode:

- ✔ Works without ERP connection
- ✔ Stores bills locally in SQLite
- ✔ Auto-syncs when ERP is available
- ✔ No data loss
- ✔ Background sync job runs periodically

When the ERP server is unreachable, bills are queued locally and automatically synced once connectivity is restored.

---

## ⚙️ Environment Variables

Create a `.env` file in the root directory:

```env
# Server Configuration
PORT=5001
NODE_ENV=production

# ERP Service
ERP_BASE_URL=http://erp-service:5000
SERVICE_TOKEN=your_secret_token_here

# Printer Configuration
PRINTER_TYPE=USB
PRINTER_INTERFACE=/dev/usb/lp0

# Database
DB_PATH=./src/database/local.sqlite

# Sync Configuration
SYNC_INTERVAL=300000
MAX_RETRY_ATTEMPTS=3

# Logging
LOG_LEVEL=info
```

---

## 🐳 Docker Support

### Build Image

```bash
docker build -t billing-server .
```

### Run Container

```bash
docker run -p 5001:5001 --env-file .env billing-server
```

### Docker Compose

```bash
docker-compose up -d
```

The service will be available at `http://localhost:5001`

---
