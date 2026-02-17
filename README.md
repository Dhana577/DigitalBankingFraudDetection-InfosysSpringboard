# Digital Banking Fraud Detection (Spring Boot)

This project is a **simple end‑to‑end digital banking fraud detection demo** built with **Java, Spring Boot, Thymeleaf, and MySQL**.

It lets you:

- Simulate banking transactions (web UI or REST API)
- Automatically calculate a **fraud risk score (0–100)** for each transaction
- Classify transactions as **NORMAL / SUSPICIOUS / FRAUD**
- Log rule violations for audit
- Export data for **machine‑learning training**

The README is written for **beginners** who may be new to Java/Spring Boot and want **step‑by‑step instructions**.

---

## 1. Tech Stack

- **Language**: Java 21
- **Framework**: Spring Boot 4.0.2
- **View Layer**: Thymeleaf + Bootstrap
- **Database**: MySQL (JPA / Hibernate)
- **Build Tool**: Maven

Key Maven configuration: see `pom.xml` (Spring Boot starter parent, JPA, Web MVC, Thymeleaf, MySQL connector, Lombok).

---

## 2. Features Overview

- **Web Dashboard (UI)**
  - Home dashboard to create / simulate individual transactions
  - Fraud report page showing only transactions flagged as fraud
  - All transactions page with search and highlighting

- **REST APIs**
  - `POST /api/transactions` – create a single transaction (JSON)
  - `GET /api/transactions` – list all transactions
  - `POST /api/transactions/simulate/{count}` – auto‑generate random transactions
  - `GET /api/fraud/logs` – view fraud rule violation logs
  - `GET /api/analytics/export` – export CSV training dataset (`fraud-training-data.csv`)

- **Fraud Detection Logic**
  - Rule‑based risk calculation:
    - High amount transactions
    - Rapid multiple transactions in a short time window
    - Location mismatch compared to previous transactions
  - Optional ML‑style plugin (`DummyFraudMLPlugin`) adds extra risk based on simple rules
  - Final status:
    - `NORMAL`, `SUSPICIOUS`, `FRAUD` (see `FraudStatus` enum)

---

## 3. Project Structure (High Level)

Important folders and classes:

- `src/main/java/com/bank/frauddetection/`
  - `FrauddetectionApplication.java` – Spring Boot entry point (`main` method)
  - `controller/`
    - `DashboardController` – routes for web UI (`"/"` and `"/fraud-report"`)
    - `TransactionController` – REST API for transactions (`/api/transactions`)
    - `FraudController` – REST API for fraud logs (`/api/fraud`)
    - `AnalyticsController` – REST API for exporting training data (`/api/analytics`)
    - `TransactionApiGatewayController` – additional API gateway (if used in your environment)
  - `model/`
    - `Transaction` – JPA entity for a banking transaction (amount, location, time, risk score, status, etc.)
    - `FraudLog` – JPA entity for logging triggered fraud rules
    - `FraudStatus` – enum (`NORMAL`, `SUSPICIOUS`, `FRAUD`)
  - `repository/`
    - `TransactionRepository` – JPA repository with custom finders (by account, time, status)
    - `FraudLogRepository` – JPA repository for fraud logs
  - `service/`
    - `TransactionService` – creates transactions, calls fraud detection, saves risk & status
    - `FraudDetectionService` – core fraud rules + optional ML plugin integration
    - `TransactionSimulationService` – generates random realistic transactions
    - `AnalyticsService` – exports data to CSV for ML training
  - `ml/`
    - `FraudMLPlugin` – interface for any ML model plugin
    - `DummyFraudMLPlugin` – simple built‑in implementation (adds risk based on amount and location)
    - `ModelEvaluationResult` – placeholder class for tracking ML metrics
  - `dto/`
    - `TransactionRequestDTO` – DTO for API usage (if wired in controllers)

- `src/main/resources/`
  - `application.properties` – app configuration (server port, DB credentials, Thymeleaf, fraud ML flag)
  - `templates/`
    - `dashboard.html` – main dashboard UI (simulate transaction + list)
    - `fraud-report.html` – table of fraud‑flagged transactions
    - `all-transactions.html` – scrollable, searchable view of all transactions

---

## 4. Prerequisites

Make sure you have the following installed:

- **Java Development Kit (JDK) 21**
  - Check: `java -version`
- **Maven 3.9+** (optional if you prefer `mvnw.cmd` wrapper)
  - Check: `mvn -version`
- **MySQL 8.x**
  - Able to create a database and user
- **Git** (if cloning from a repository)

You are on **Windows**, so commands below that use `mvnw` should be run as `mvnw.cmd` from a terminal opened in the project root.

---

## 5. Database Setup (MySQL)

The application expects a MySQL database named `frauddb` and credentials defined in `application.properties`.

Open `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/frauddb?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=harsh
spring.jpa.hibernate.ddl-auto=update
```

### 5.1. Create the database

In MySQL (e.g., MySQL Shell, Workbench, or CLI):

```sql
CREATE DATABASE frauddb;
```

### 5.2. Configure username/password

Either:

- Use the existing `root` user and **set its password** to match `spring.datasource.password`, **or**
- Create a dedicated user and update `application.properties`.

Example of creating a dedicated user:

```sql
CREATE USER 'frauduser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON frauddb.* TO 'frauduser'@'localhost';
FLUSH PRIVILEGES;
```

Then update `application.properties`:

```properties
spring.datasource.username=frauduser
spring.datasource.password=StrongPassword123!
```

> **Important:** Do not commit real passwords to a public repository. The password in this demo should be changed for real deployments.

---

## 6. How Fraud Detection Works (Conceptual)

When you create a transaction (via UI or API):

1. The request is handled by `TransactionService.createTransaction`.
2. The transaction is saved (with a timestamp).
3. `FraudDetectionService.calculateRisk` is called to compute a risk score.
4. Rule‑based checks are applied:
   - **Rule 1 – High Amount**
     - If `amount > 50,000`, adds 50 points to the risk score.
   - **Rule 2 – Rapid Transactions**
     - Looks at recent transactions by the same account in the last **2 minutes**.
     - If there are at least **3** transactions in that window, adds 30 risk points.
   - **Rule 3 – Location Mismatch**
     - Compares the current transaction location to previous transactions for that account.
     - If the location has changed, adds 20 risk points.
   - A `FraudLog` entry is created for each rule triggered.
5. If the ML plugin is enabled (`fraud.ml.enabled=true`) and available:
   - `DummyFraudMLPlugin` adds extra risk:
     - `+40` if amount > 60,000
     - `+30` if location is `"UNKNOWN"`
6. The **total risk** is capped at 100.
7. `FraudDetectionService.detectStatus` maps the score:
   - `risk >= 70`  → `FRAUD`
   - `risk >= 30`  → `SUSPICIOUS`
   - else          → `NORMAL`
8. The transaction is updated with `riskScore` and `status` and saved again.

You can see the results in the **dashboard UI** and via the **REST APIs**.

---

## 7. Running the Application

### 7.1. Clone or copy the project

If you haven’t already:

```bash
cd d:\
git clone <your-repo-url> DigitalBankingFraudDetection
cd DigitalBankingFraudDetection
```

If you already have the folder (e.g., `d:\clone_pro\DigitalBankingFraudDetection`), just open it in your IDE or terminal.

### 7.2. Build (optional but recommended)

From the project root (where `pom.xml` is located):

```bash
mvnw.cmd clean package
```

Or, if you have Maven in PATH:

```bash
mvn clean package
```

This will compile the project and run tests.

### 7.3. Run the app

You can run it in one of two simple ways:

**A. Using Maven wrapper**

```bash
mvnw.cmd spring-boot:run
```

**B. Using the built JAR**

After `mvnw.cmd clean package`, run:

```bash
java -jar target/frauddetection-0.0.1-SNAPSHOT.jar
```

The app will start on the port defined in `application.properties`:

```properties
server.port=8082
```

So the application is accessible at:

- `http://localhost:8082/`

---

## 8. Using the Web UI

### 8.1. Dashboard (`/`)

Open:

- `http://localhost:8082/`

You will see:

- A form to **simulate a transaction**:
  - `Account Number`
  - `Amount`
  - `Location`
- A table of existing transactions, including:
  - Account
  - Amount
  - Location
  - Status (`NORMAL` / `SUSPICIOUS` / `FRAUD`) with colored badges
  - Risk score

Submitting the form creates a transaction, runs fraud detection, and reloads the dashboard.

### 8.2. Fraud Report (`/fraud-report`)

Open:

- `http://localhost:8082/fraud-report`

You will see only the transactions with status `FRAUD`, highlighted and including:

- ID
- Account
- Amount
- Location
- Status
- Risk score
- Time

### 8.3. All Transactions (`/all-transactions` or similar template)

If you have a controller mapping for `all-transactions.html`, it will display:

- Scrollable, searchable table of all transactions
- Highlighting for fraud rows
- Filter by account number (simple JavaScript search)

If you do not yet have a mapping, you can create one in a controller to return `"all-transactions"` with `transactions` in the model.

---

## 9. Using the REST APIs (Examples)

Below are example calls using `curl`. Replace `localhost:8082` if you run on a different host or port.

### 9.1. Create a transaction

```bash
curl -X POST "http://localhost:8082/api/transactions" ^
  -H "Content-Type: application/json" ^
  -d "{\"accountNumber\":\"ACC1234\",\"amount\":75000,\"location\":\"DELHI\"}"
```

(Use `^` line continuation in PowerShell or put everything on one line.)

### 9.2. Get all transactions

```bash
curl "http://localhost:8082/api/transactions"
```

### 9.3. Simulate multiple random transactions

```bash
curl -X POST "http://localhost:8082/api/transactions/simulate/50"
```

This will generate 50 random transactions and persist them.

### 9.4. Get fraud logs

```bash
curl "http://localhost:8082/api/fraud/logs"
```

You will get a JSON array of `FraudLog` objects indicating which rules were triggered for which transaction.

### 9.5. Export training data to CSV

```bash
curl "http://localhost:8082/api/analytics/export"
```

This will:

- Write a file named `fraud-training-data.csv` in the **working directory** where the app is running.
- Return the file path as a string in the response.

The CSV will contain:

```text
amount,location,riskScore,status
...
```

You can load this into tools like Excel, Python (pandas), or ML frameworks for experimentation.

---

## 10. Enabling / Disabling the ML Plugin

In `application.properties`:

```properties
fraud.ml.enabled=true
```

- If **`true`** and `DummyFraudMLPlugin` (or any other bean implementing `FraudMLPlugin`) is present, the ML risk component is added.
- If **`false`**, only the rule‑based engine is used.

For a real ML model, you would:

1. Implement `FraudMLPlugin` with your prediction logic.
2. Annotate it with `@Component` or configure it as a Spring bean.
3. Optionally log `modelName()` and `ModelEvaluationResult` somewhere.

---

## 11. Common Issues and Tips (Beginner Friendly)

- **Port already in use (8082)**:
  - Another process is using port 8082.
  - Either stop that process or change `server.port` in `application.properties`.

- **Database connection errors**:
  - Check that:
    - MySQL is running.
    - The `frauddb` database exists.
    - `spring.datasource.username` and `spring.datasource.password` are correct.
    - The user has privileges on `frauddb`.

- **`Table 'frauddb.transaction' doesn't exist` (or similar)**:
  - Hibernate will auto‑create tables because `spring.jpa.hibernate.ddl-auto=update`.
  - If something goes wrong, you can:
    - Drop and recreate the `frauddb` database.
    - Restart the app.

- **Java version mismatch**:
  - The project targets Java 21 (`<java.version>21</java.version>` in `pom.xml`).
  - Make sure your JDK and IDE are configured for Java 21 (or adjust the POM if you intentionally want another version).

---

## 12. Next Steps and Extensions

Ideas to extend this project:

- Replace `DummyFraudMLPlugin` with a real ML model (e.g., load a model from file).
- Add authentication and user roles (admin vs. analyst).
- Add more detailed dashboards / charts (e.g., fraud by location, time of day).
- Integrate with a message queue (Kafka/RabbitMQ) for streaming transactions.
- Containerize with Docker and add docker‑compose for MySQL + app.

---

## 13. Summary

- This project demonstrates **practical fraud detection** using a combination of:
  - Rule‑based checks
  - Optional plug‑in ML scoring
  - A simple web UI and REST APIs
- It is designed to be **easy to run on a local machine** and **easy to understand** for beginners exploring Spring Boot and fraud analytics.

