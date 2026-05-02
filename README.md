# 🎫 StarPass

A distributed ticketing system built with **Spring Boot** and **RabbitMQ** to demonstrate asynchronous communication, decoupling, and event-driven architecture.

## 📌 Overview

StarPass is a mock ticketing platform where users can purchase tickets for events. The system is split into two microservices to simulate a real-world environment where payment processing is handled independently and asynchronously.

*   **StarPassAPI**: The core service. Manages events, tickets, and purchase requests. It acts as a **Producer** and **Consumer** as well, sending payment tasks to RabbitMQ but also receiving messages from the payment system to set the purchase status.
*   **StarPass_Payment_API**: A dedicated worker service. It acts primarily as a **Consumer**, but also as a consumer, listening for payment requests, "processing" them, and returning the result via messaging.

## 🏗️ Architecture

The project follows a decoupled approach using message queues:

1.  **Client** requests a purchase through `StarPassAPI`.
2.  `StarPassAPI` validates the request and publishes a message to a **RabbitMQ Exchange**.
3.  **RabbitMQ** routes the message to the payment queue.
4.  `StarPass_Payment_API` consumes the message, processes the payment and produces a message with the payment result.
5.  `StarPassAPI` consumes the result and sets the purchase status to DENIED or SOLD.

## 🛠️ Tech Stack

*   **Language:** Java 17+
*   **Framework:** Spring Boot 3.x
*   **Messaging:** RabbitMQ
*   **Database:** PostgreSQL
*   **Build Tool:** Gradle (Kotlin DSL)
*   **Containerization:** Docker & Docker Compose

## 📂 Project Structure

```text
.
├── StarPassAPI/             # Main API (Producer/Consumer)
├── StarPass_Payment_API/     # Payment Service (Producer/Consumer)
└── docker-compose.yml       # Orchestration for Apps, DB, and RabbitMQ
```

## 🚀 Getting Started

### Prerequisites

*   Docker and Docker Compose installed.
*   JDK 26

### Running with Docker

The easiest way to see the system in action is using the provided `docker-compose.yml` file, which sets up the entire infrastructure:

1.  Clone the repository (including submodules):
    ```bash
    git clone --recursive https://github.com/om1cael/starpass.git
    cd starpass
    ```
2.  Build and run the containers:
    ```bash
    docker-compose up --build
    ```

The following services will be available:
*   **StarPass API:** `http://localhost:8080`
*   **RabbitMQ Management:** `http://localhost:15672` (starpass/123)
*   **PostgreSQL:** `localhost:5432`

## 🔌 API Endpoints (StarPassAPI)

### Events & Tickets
*   `POST /event` - Create a new event.
    - `{ "name": "name" }`   
*   `POST /ticket` - Generate tickets for an event.
    - `{ "eventId": 1, "ticketType": "DEFAULT or VIP", "price": 1.0, "amount": 100 }`

### Purchases
*   `POST /purchase` - Initiate a purchase. This triggers the RabbitMQ flow.
    - `{ "ticketId": 1, "amount": 1 }`
*   `GET /purchase/{id}` - Check the current status of a purchase.
