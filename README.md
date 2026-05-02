# Algorithmic Trading Infrastructure & Edge Gateway

## 1. Project Objective
In algorithmic trading, developing a winning strategy is only half the battle; the other half is building a highly reliable **Execution Infrastructure**. 
This project transforms a standard machine into a dedicated, bare-metal trading server to achieve the following:
* **Absolute Uptime (24/7):** Trading bots require a dedicated machine that continuously listens to market data without interruptions (sleep modes, Wi-Fi drops).
* **Low Latency & Traffic Routing:** Running multiple bots (e.g., Mean Reversion, Statistical Arbitrage) requires a single entry point (Edge Gateway) to receive market data streams (Webhooks/WebSockets) and route them to the correct bot with minimal latency. 
* **Data Gravity:** Centralizing historical tick data and OHLCV logs into a stable, on-premise database (PostgreSQL) rather than scattered local CSV files.

## 2. System Architecture
The system enforces a strict separation between the development environment and the production server.

### Node A: Dev Machine (Local)
* Used for writing code (IDE: CLion / PyCharm).
* Runs the unit testing suite (Pytest).
* Handles version control and utilizes Remote Development to compile/debug directly on the Target Server.

### Node B: Target Server (Bare-Metal Production)
* **OS:** Ubuntu Server LTS (Headless, SSH-only access).
* **Layer 1: C++ Edge Gateway:** A custom Systemd Daemon listening on a public port. It parses incoming network requests (TCP/HTTP) and routes them internally using asynchronous I/O and zero-copy principles.
* **Layer 2: Dockerized Microservices:**
  * `trading_bot_1` (Python): Receives signals from the C++ Gateway and executes trades via Exchange APIs.
  * `postgres_db`: Stores historical data, order logs, and execution states.
  * `telemetry_dashboard`: A Web UI displaying real-time PnL, Gateway latency metrics, and system health.

## 3. Tech Stack & Tooling
* **Version Control:** GitHub (Monorepo structure).
* **CI/CD:** GitHub Actions (Automated `pytest` runs on every push to ensure trading logic remains intact).
* **Deployment:** Docker & Docker Compose.
* **Core Networking:** Modern C++ (C++17/20) utilizing Linux `epoll` for asynchronous socket management.
* **Application Layer:** Python (for trading logic, data analysis, and system verification).

## 4. Required Skills & Knowledge Base
* Linux System Administration (SSH, Systemd, Network Interfaces, Process Management).
* POSIX Network Programming (Non-blocking I/O, Socket lifecycles).
* Advanced C++ Concurrency (Lock-free structures, Memory Pools, Event Loops).
* Python Automation & System Testing (Pytest, Network Mocking).

## 5. Potential Pitfalls & Mitigation
1. **Memory Leaks in C++:** Improper dynamic allocation during traffic spikes will crash the Gateway. *Mitigation: Mandatory `Valgrind` checks before deployment.*
2. **Zombie Sockets:** Clients or exchanges dropping connections without proper TCP teardown can exhaust File Descriptors. *Mitigation: Strict timeout policies and robust socket state machines.*
3. **Database Connection Pool Exhaustion:** Concurrent read/writes from multiple bots locking the database. *Mitigation: Implementing proper Connection Pooling.*
