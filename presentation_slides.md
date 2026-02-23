## Digital Banking Fraud Detection – 30‑Slide Deck Content

Below is suggested content for each slide, aligned with your 7 sections and 30 headings. You can paste this into PowerPoint/Google Slides and format visually.

---

### Slide 1 – Title

- **Title**: Digital Banking Fraud Detection – Rule‑Based + ML‑Enabled Prototype  
- **Sub‑Title**: Real‑Time Transaction Risk Scoring for Banks, FinTech, and Card Issuers  
- **Technology Stack**: Java 21, Spring Boot 4.x, REST APIs, Thymeleaf UI, MySQL, JPA/Hibernate  
- **Authors / Team**: \<Your Name / Team\>  
- **Date**: \<Presentation Date\>  
- **Tagline**: From raw transaction to fraud insight in milliseconds

---

### Slide 2 – Problem Statement (Manual Process)

- **Current Situation (Manual Review)**  
  - Many banks still rely heavily on **manual rule tuning** and **analyst review** of suspicious transactions.  
  - Fraud teams often inspect Excel exports or legacy dashboards after the fact.  
- **Pain Points**  
  - **Delayed detection**: Fraud is often discovered hours or days later.  
  - **Inconsistent decisions**: Different analysts may rate similar cases differently.  
  - **Limited scalability**: Manual triage does not scale with transaction volume growth.  
- **Need**  
  - A **repeatable, automated engine** that can quickly score transactions and surface the riskiest ones for review.

---

### Slide 3 – Problem Statement Objective

- **Primary Objective**  
  - Build a **working prototype** that can **ingest digital banking transactions**, **compute fraud risk**, and **classify them** as NORMAL, SUSPICIOUS, or FRAUD.  
- **Key Goals**  
  - Demonstrate **end‑to‑end flow**: API/UI → Risk Engine → Database → Analytics export.  
  - Provide **clear, explainable rules** that business users can understand.  
  - Allow **ML extensions** via a plugin interface for future intelligent models.  
- **Outcome**  
  - A small but complete system that mirrors how **real fraud engines** are architected, while remaining **simple enough for learning and experimentation**.

---

### Slide 4 – Real‑World Relevance (Banking & FinTech)

- **Domains Impacted**  
  - **Retail Banking**: Online transfers, UPI, NEFT/RTGS, mobile banking transactions.  
  - **FinTech / Wallets**: P2P payments, buy‑now‑pay‑later, in‑app purchases.  
  - **Credit Card Processors**: Card‑not‑present e‑commerce, POS swipes.  
  - **ATM Networks**: Cash withdrawals, PIN‑based authentication.  
- **Why Fraud Detection Matters**  
  - Reduces **financial loss** for bank and customer.  
  - Protects **brand reputation** and **regulatory compliance**.  
  - Minimizes **chargebacks** and operational overhead.  
- **How This Project Relates**  
  - Simulates the **core logic** of these systems: ingest a transaction, apply rules/ML, log fraud events, and expose analytics.

---

### Slide 5 – Backend Components and Their Roles

- **Spring Boot Application (`FrauddetectionApplication`)**  
  - Main entry point; configures Spring context, web server, and beans.  
- **Controllers (`controller` package)**  
  - REST and MVC endpoints for **UI**, **APIs**, and **gateway ingestion**.  
  - Examples: `DashboardController`, `TransactionController`, `TransactionApiGatewayController`.  
- **Services (`service` package)**  
  - Encapsulate **business logic**: transaction creation, fraud risk calculation, simulation, analytics export.  
- **Repositories (`repository` package)**  
  - Spring Data JPA interfaces to access **MySQL** without manual SQL.  
- **Models & DTOs (`model`, `dto` packages)**  
  - JPA entities (`Transaction`, `FraudLog`) and DTOs (`TransactionRequestDTO`) for clean data transfer.  
- **ML Plugin (`ml` package)**  
  - Interface and dummy implementation for plugging in an ML model.

---

### Slide 6 – Architecture Style (Layered & Service‑Oriented)

- **Layered Architecture**  
  - **Presentation Layer**: Thymeleaf templates + REST controllers.  
  - **Service Layer**: Business logic and orchestration.  
  - **ML / Risk Engine Layer**: Rule‑based + ML plugin (`FraudDetectionService`, `FraudMLPlugin`).  
  - **Repository Layer**: JPA repositories talk to the database.  
  - **Database Layer**: MySQL storing transactions and fraud logs.  
- **Service‑Oriented Interactions**  
  - Clear **service boundaries**: `TransactionService`, `FraudDetectionService`, `AnalyticsService`.  
  - RESTful endpoints to allow other systems or channels to integrate (e.g., mobile app, core banking).  
- **Benefits**  
  - Easier to **maintain, test, and extend**.  
  - Swap or upgrade individual layers (e.g., ML model) with minimal impact.

---

### Slide 7 – System Architecture (High‑Level + ER + ML View)

- **High‑Level View**  
  - **User / External System** → UI / API → Transaction Ingestion → Fraud Engine → Database → Analytics / Reports.  
- **Key Entities (ER Perspective)**  
  - `Transaction`  
    - Fields: id, accountNumber, amount, location, transactionTime, status, riskScore.  
  - `FraudLog`  
    - Fields: id, transactionId, ruleViolated, riskScore, loggedAt.  
  - Relationship: **One transaction** can have **multiple fraud logs**.  
- **ML Integration**  
  - Fraud engine optionally calls **ML plugin** (`FraudMLPlugin`) which returns an additional risk score.  
  - Overall risk = Rule‑based risk + ML risk (capped at 100).

---

### Slide 8 – Layered Architecture (5 Layers)

- **1. UI Layer**  
  - `dashboard.html`, `fraud-report.html`, `all-transactions.html`.  
  - Shows forms and tables for transactions and fraud data.  
- **2. Service Layer**  
  - `TransactionService`, `TransactionSimulationService`, `AnalyticsService`.  
  - Coordinates repositories and fraud engine.  
- **3. ML / Risk Engine Layer**  
  - `FraudDetectionService`, `FraudMLPlugin`, `DummyFraudMLPlugin`.  
  - Calculates risk and determines final fraud status.  
- **4. Repository Layer**  
  - `TransactionRepository`, `FraudLogRepository`.  
  - Encapsulate queries like recent transactions, fraud‑only filters.  
- **5. Database Layer**  
  - MySQL schema with `transaction` and `fraud_log` tables.  
  - Auto‑managed by JPA/Hibernate with `ddl-auto=update`.

---

### Slide 9 – Full Transaction Execution Flow

- **Step‑by‑Step Flow**  
  1. **Ingestion** – Transaction comes in via web form, `/api/transactions`, or `/api/gateway/transaction` with `TransactionRequestDTO`.  
  2. **Service Call** – `TransactionService.createTransaction` is invoked.  
  3. **Persist Raw Transaction** – Transaction is saved with timestamp.  
  4. **Risk Calculation** – `FraudDetectionService.calculateRisk` applies rules and optional ML.  
  5. **Status Determination** – `detectStatus` maps risk to NORMAL/SUSPICIOUS/FRAUD.  
  6. **Update and Save** – Transaction is updated with `riskScore` and `status`.  
  7. **UI / API Response** – Updated transaction (with risk and status) is returned or rendered on UI.  
- **End Result**  
  - Every transaction has a **traceable risk score and fraud status** stored in the database.

---

### Slide 10 – Component Interaction (Controllers, Services, Repos)

- **Controller → Service**  
  - UI controllers (`DashboardController`) receive form data, bind to `Transaction`, and delegate to `TransactionService`.  
  - API controllers (`TransactionController`, `TransactionApiGatewayController`) receive JSON and call service methods.  
- **Service → Risk Engine + Repository**  
  - `TransactionService` uses `TransactionRepository` to persist entities.  
  - It calls `FraudDetectionService` to enrich each transaction with risk and status.  
- **Risk Engine → Repositories**  
  - `FraudDetectionService` queries `TransactionRepository` to check recent or historical transactions.  
  - It stores rule violations using `FraudLogRepository`.  
- **Repository → Database**  
  - JPA/Hibernate translates method calls into SQL and writes to MySQL.  
- **Flow Summary**  
  - Each layer focuses on **one responsibility**, making the overall interaction readable and testable.

---

### Slide 11 – Module: Controller Layer

- **Purpose of Controllers**  
  - Expose the application as **web pages** and **REST APIs**.  
  - Map URLs and HTTP methods to Java methods.  
- **Examples**  
  - `DashboardController`  
    - `GET "/"` → Show dashboard with all transactions.  
    - `POST "/transaction/create"` → Create a transaction from UI form.  
    - `GET "/fraud-report"` → Show only fraud transactions.  
  - `TransactionController`  
    - Base path `/api/transactions`: create, list, simulate transactions programmatically.  
  - `TransactionApiGatewayController`  
    - `/api/gateway/transaction` – external ingestion endpoint using `TransactionRequestDTO`.  
    - `/api/gateway/transactions/fraud` – fraud transactions for other systems.  
- **Key Point**  
  - Controllers are **thin**: they forward work to services and do not contain core business logic.

---

### Slide 12 – Service Layer: Fraud Detect & Analytics

- **TransactionService**  
  - Central orchestrator for creating transactions.  
  - Sets `transactionTime`, saves the transaction, calls fraud engine, updates risk and status.  
- **FraudDetectionService**  
  - Encapsulates **all fraud rules and ML integration**.  
  - Calls repositories to inspect recent activity and prior locations.  
- **TransactionSimulationService**  
  - Generates synthetic transactions for testing and stress scenarios.  
  - Randomizes account numbers, amounts, and locations.  
- **AnalyticsService**  
  - Exports all transactions to `fraud-training-data.csv` for downstream ML work.  
- **Benefits of Service Layer**  
  - Keeps **controllers clean**, hides complexity, and is easy to **unit test**.

---

### Slide 13 – Fraud Detection Logic (Rule‑Based + ML)

- **Rule‑Based Core** (`calculateRuleBasedRisk`)  
  - **Rule 1: High Amount**  
    - If `amount > 50,000`, add **50 risk points** and log `HIGH_AMOUNT_TRANSACTION`.  
  - **Rule 2: Rapid Multiple Transactions**  
    - Count transactions for the same account in the last **2 minutes**.  
    - If count ≥ 3, add **30 risk points** and log `RAPID_MULTIPLE_TRANSACTIONS`.  
  - **Rule 3: Location Mismatch**  
    - Compare current location with the last known location of the account.  
    - If different, add **20 risk points** and log `LOCATION_MISMATCH`.  
- **ML‑Based Extension** (`FraudMLPlugin`)  
  - Optional additional risk added by an ML model.  
  - Dummy implementation: extra risk if amount > 60,000 or location is UNKNOWN.  
- **Final Risk**  
  - `totalRisk = min(ruleRisk + mlRisk, 100)` → then mapped to NORMAL/SUSPICIOUS/FRAUD.

---

### Slide 14 – ML Plugin Architecture

- **Plugin Interface (`FraudMLPlugin`)**  
  - Methods: `predictRisk(Transaction)` and `modelName()`.  
  - Abstracts the actual ML model from the core system.  
- **Dummy Implementation (`DummyFraudMLPlugin`)**  
  - Implemented as a Spring `@Component`, automatically picked up as a bean.  
  - Demonstrates how any ML model can be plugged in without modifying the core engine.  
- **Configuration Switch**  
  - Property `fraud.ml.enabled=true/false` controls whether ML is used.  
  - If enabled and plugin is present, extra risk is added; if not, only rules are used.  
- **Benefits**  
  - Enables **experimentation with multiple models**.  
  - Supports **A/B testing** and future upgrades without rewriting business logic.

---

### Slide 15 – Database Design (Tables & Fields)

- **Transaction Table (simplified)**  
  - `id` (PK, auto‑generated).  
  - `account_number` (string).  
  - `amount` (double).  
  - `location` (string).  
  - `transaction_time` (timestamp).  
  - `status` (string: NORMAL / SUSPICIOUS / FRAUD).  
  - `risk_score` (integer, 0–100).  
- **FraudLog Table (simplified)**  
  - `id` (PK, auto‑generated).  
  - `transaction_id` (FK to Transaction).  
  - `rule_violated` (string: HIGH_AMOUNT_TRANSACTION, etc.).  
  - `risk_score` (risk at time of logging).  
  - `logged_at` (timestamp).  
- **Design Choices**  
  - Separate `FraudLog` table to provide an **audit trail** of which rules fired.  
  - Risk score stored both on `Transaction` and `FraudLog` for easier reporting.

---

### Slide 16 – Data Flow: DTO, API, Entity

- **Inbound Data (DTO)**  
  - External clients call `/api/gateway/transaction` with `TransactionRequestDTO`.  
  - DTO carries only **required input fields**: accountNumber, amount, location.  
- **Transformation to Entity**  
  - Gateway controller converts DTO → `Transaction` entity.  
  - `TransactionService` sets additional fields: `transactionTime`, `status`, `riskScore`.  
- **Persistence and Response**  
  - Repositories save the transaction to the database.  
  - Controllers return the **fully enriched entity** (JSON) or render it in the UI.  
- **Separation of Concerns**  
  - DTO isolates external contract from internal entity structure.  
  - Entities represent how data is **stored**; DTOs represent how data is **sent/received**.

---

### Slide 17 – Fraud Percentage / Metrics Concept

- **What We Can Measure**  
  - **Total Transactions** vs **Fraud Transactions**.  
  - Fraud rate = (Fraud transactions / Total transactions) × 100.  
  - Suspicious rate = (Suspicious transactions / Total transactions) × 100.  
- **How to Calculate**  
  - Use database queries or export `fraud-training-data.csv` and compute in Excel / Python.  
  - Group by `status` field in `Transaction`.  
- **Use Cases**  
  - Track **overall fraud prevalence** in simulated data.  
  - Tune rules/thresholds to reach a target false positive/false negative balance.  
- **Future UI Idea**  
  - Add dashboard cards showing **fraud %**, **suspicious %**, **trend over time**.

---

### Slide 18 – Analytics Model (Fraud Rate & Percentages)

- **Data Export Mechanism** (`AnalyticsService`)  
  - Endpoint: `GET /api/analytics/export`.  
  - Writes `fraud-training-data.csv` with columns: `amount,location,riskScore,status`.  
- **How Analysts Can Use It**  
  - Load CSV into Excel, Power BI, or Python (pandas).  
  - Compute:  
    - Fraud rate by **location**.  
    - Fraud rate by **amount buckets** (e.g., 0–10k, 10–50k, >50k).  
    - Relationship between **riskScore** and final status.  
- **Towards a Predictive Model**  
  - Use the exported data as **training data** for a real ML classifier.  
  - Evaluate using metrics like **accuracy**, **precision**, **recall** (see `ModelEvaluationResult`).  
- **Feedback Loop**  
  - Insights from analytics can be fed back into updated rules or ML model retraining.

---

### Slide 19 – Stress Testing

- **Why Stress Testing Matters**  
  - Real banking systems handle **millions of transactions per day**.  
  - Need to know how the engine behaves under **high load**.  
- **Simulation‑Based Stress Test**  
  - Use `/api/transactions/simulate/{count}` or `/api/gateway/simulate/{count}`.  
  - Example: simulate **10,000** transactions and observe response times and DB growth.  
- **What to Observe**  
  - Application CPU and memory usage.  
  - Database performance (query times, table sizes).  
  - Impact on risk calculation time and UI responsiveness.  
- **Outcome**  
  - Identify any bottlenecks in **rules**, **queries**, or **I/O** before scaling to production‑like loads.

---

### Slide 20 – Simulation Flow: API → Fraud Engine → DB

- **Flow Diagram Description**  
  1. Client calls **Simulation API** (`/simulate/{count}`).  
  2. `TransactionSimulationService` generates a list of random `Transaction` objects.  
  3. For each transaction, `TransactionService.createTransaction` is invoked.  
  4. Fraud engine calculates risk and status, logs rules if needed.  
  5. Transactions and logs are persisted to MySQL.  
- **Purpose of Simulation**  
  - Quickly generate **realistic data** for:  
    - UI demos.  
    - Performance testing.  
    - Analytics and ML experimentation.  
- **Key Idea**  
  - Simulation mimics **live traffic** without connecting to real banking systems.

---

### Slide 21 – Benefits of Simulation

- **Safe Testing Environment**  
  - No exposure of real customer data or PII.  
  - Zero financial risk while experimenting with rules and thresholds.  
- **Rapid Iteration**  
  - Easy to adjust parameters (amount ranges, locations, frequencies) and see impact.  
  - Helps tune the rules for a desired fraud detection rate.  
- **Training Data Generation**  
  - Creates labeled data (NORMAL / SUSPICIOUS / FRAUD) for ML models.  
- **Demonstration Tool**  
  - Ideal for **presentations, POCs, and workshops**, showing end‑to‑end fraud detection visually.

---

### Slide 22 – Step‑by‑Step Execution (How to Run)

- **1. Setup**  
  - Install JDK 21, MySQL 8, and Maven (or use `mvnw.cmd`).  
  - Create database: `CREATE DATABASE frauddb;`.  
  - Configure `application.properties` with correct DB username/password.  
- **2. Build and Run**  
  - From project root: `mvnw.cmd clean package` then `mvnw.cmd spring-boot:run`.  
  - Application starts on `http://localhost:8082/`.  
- **3. Use UI**  
  - Go to `/` for dashboard; submit transactions; view fraud report at `/fraud-report`.  
- **4. Use APIs**  
  - Call `/api/transactions`, `/api/gateway/transaction`, and `/api/analytics/export` for programmatic usage.  
- **5. Verify Results**  
  - Inspect database tables and CSV export to confirm risk scores and statuses.

---

### Slide 23 – Example Execution Scenario

- **Scenario**: High‑Value, New Location Transaction  
  - Account `ACC1234` usually transacts in **MUMBAI** with amounts under **₹10,000**.  
  - Suddenly, a new transaction appears:  
    - Amount: **₹80,000**.  
    - Location: **DELHI**.  
- **What the Engine Does**  
  - **Rule 1 (High Amount)** triggers: +50 risk.  
  - **Rule 3 (Location Mismatch)** triggers: +20 risk.  
  - If ML plugin is on, **Dummy ML** adds extra risk for high amount.  
  - Final risk ≥ 70 → Status = **FRAUD**.  
- **Result**  
  - Transaction is highlighted as FRAUD in dashboard and fraud report.  
  - Fraud logs record which rules fired for audit.

---

### Slide 24 – Risk Threshold Logic (Normal, Suspicious, Fraud)

- **Risk Score Range**: 0 to 100  
- **Mapping Logic (`detectStatus`)**  
  - `0 ≤ risk < 30` → **NORMAL**  
  - `30 ≤ risk < 70` → **SUSPICIOUS**  
  - `70 ≤ risk ≤ 100` → **FRAUD**  
- **Design Rationale**  
  - Allow a **buffer zone** for suspicious cases where manual analyst review may be required.  
  - Immediate blocking can be considered for FRAUD, while SUSPICIOUS might trigger OTP or step‑up authentication.  
- **Tuning Possibility**  
  - Thresholds can be adjusted based on **business tolerance** for false positives/negatives.

---

### Slide 25 – Comparison with Real Banking Systems

- **Simplifications in This Project**  
  - Only a few rules (amount, rapid usage, location).  
  - Single currency, single region, simple channels.  
  - No real‑time integration with card networks / core banking.  
- **Real‑World Systems Typically Include**  
  - Dozens to hundreds of rules (device fingerprinting, IP reputation, velocity checks).  
  - Multi‑channel integration (mobile, web, ATM, POS).  
  - Sophisticated ML models and graph‑based analyses.  
  - Case management tools, alert queues, and workflow.  
- **Value of This Prototype**  
  - Captures the **essence of how fraud engines work**.  
  - Serves as a **learning platform** and a base for future enhancements.

---

### Slide 26 – Why a Hybrid Approach (Rules + ML)

- **Pure Rule‑Based Pros/Cons**  
  - Pros: Easy to explain, easy to implement, transparent to regulators.  
  - Cons: Hard to maintain at scale, misses **complex patterns**.  
- **Pure ML‑Based Pros/Cons**  
  - Pros: Learns from data, captures non‑obvious correlations.  
  - Cons: Harder to explain, can drift over time, needs labeled data.  
- **Hybrid Strategy in This Project**  
  - **Rules** handle obvious, high‑impact patterns (e.g., very high amount, rapid multiple transactions).  
  - **ML Plugin** can fine‑tune risk based on historical patterns.  
- **Result**  
  - Better balance between **explainability** and **accuracy**, aligned with how many banks operate today.

---

### Slide 27 – False Positives and False Negatives

- **False Positive (Legitimate flagged as Fraud)**  
  - Example: Customer legitimately spends ₹80,000 on a holiday booking from a new city.  
  - Impact: Customer frustration, call center load, possible loss of future business.  
- **False Negative (Fraud Not Detected)**  
  - Example: Small but frequent fraudulent transactions that stay below thresholds.  
  - Impact: Direct financial loss, reputational damage.  
- **Balancing the Two**  
  - Adjust thresholds and rules to minimize **total risk** to the bank + customer.  
  - Use **analytics export** to measure how often rules flag legitimate vs fraudulent behavior.  
- **Educational Point**  
  - Any fraud system is a **trade‑off**; this project illustrates how to start reasoning about that trade‑off.

---

### Slide 28 – Scalability Considerations

- **Application Layer**  
  - Stateless REST services can be scaled horizontally (multiple instances behind a load balancer).  
  - Caching frequent lookups (e.g., last location per account) can reduce DB load.  
- **Database Layer**  
  - Index critical columns: `accountNumber`, `transactionTime`, `status`.  
  - Consider read replicas, sharding, or partitioning for very high volumes.  
- **Fraud Engine**  
  - Optimize queries used in rules (e.g., time‑window checks).  
  - Pre‑compute aggregates or use streaming platforms for real‑time events.  
- **Infrastructure**  
  - Containerization (Docker, Kubernetes) for elastic scaling.  
  - Monitoring with metrics (latency, error rate, throughput).

---

### Slide 29 – Future Enhancements

- **Functional Enhancements**  
  - More rules: device fingerprint, IP reputation, merchant category codes.  
  - User and role‑based access control (fraud analyst vs admin).  
  - Case management interface for investigating alerts.  
- **ML Enhancements**  
  - Train a real ML model from exported CSV (e.g., gradient boosting, neural nets).  
  - Add multiple models and model selection logic.  
- **Tech Enhancements**  
  - Add an event streaming pipeline (Kafka) for real‑time scoring.  
  - Build richer dashboards (charts, heatmaps) for analytics.  
- **Operational Enhancements**  
  - Automated tests, CI/CD pipeline, and observability (logs, metrics, traces).

---

### Slide 30 – Conclusion

- **What We Built**  
  - A **full‑stack fraud detection prototype**: from transaction ingestion to risk scoring, storage, and analytics export.  
- **Key Learnings**  
  - How layered architectures (UI, services, ML engine, repositories, DB) make systems modular and maintainable.  
  - How simple rules and optional ML can work together to detect risky transactions.  
- **Real‑World Perspective**  
  - While simplified, this design reflects the **core ideas** used in real banking fraud engines.  
- **Next Step**  
  - Use this project as a **sandbox** to experiment with new rules, real ML models, and larger‑scale simulations, moving closer to production‑grade fraud analytics.

