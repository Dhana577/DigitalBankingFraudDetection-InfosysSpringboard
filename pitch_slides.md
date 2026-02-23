## Pitch Script – 30 Slides

Short spoken script you can use in front of a panel, one block per slide.

---

### Slide 1 – Title

“Good \<morning/afternoon\>. Today I’ll walk you through our Digital Banking Fraud Detection prototype. It’s a full‑stack Spring Boot and MySQL application that ingests transactions, scores their fraud risk using rules and an ML‑style plugin, and exposes the results via UI and APIs. I’ll show how this mirrors real‑world fraud engines used in banks and FinTechs.”

---

### Slide 2 – Problem Statement (Manual)

“Most institutions still rely heavily on manual fraud review—analysts looking at spreadsheets or legacy dashboards after suspicious activity occurs. That leads to delays, inconsistent decisions between analysts, and it simply doesn’t scale with growing digital transaction volumes. We need a more systematic and automated way to prioritize risky transactions.”

---

### Slide 3 – Problem Objective

“Our objective with this project is to build a working prototype that can automatically ingest digital banking transactions, compute a fraud risk score, and classify each operation as normal, suspicious, or fraud. The goal is not just to have code running, but to demonstrate a complete, explainable pipeline that can be easily extended with more rules or with real ML models.”

---

### Slide 4 – Real‑World Relevance

“This problem is highly relevant across retail banking, FinTech wallets, credit cards, and ATM networks. Everywhere money moves digitally, there is a risk of fraud. By simulating a smaller version of these environments, this project shows how a fraud engine can sit in the middle of those flows and help reduce financial loss, protect customers, and support compliance.”

---

### Slide 5 – Backend Roles

“On the backend, we follow clear responsibilities. Controllers expose web pages and REST APIs. Services like `TransactionService` and `FraudDetectionService` hold our business logic. Repositories handle database access, models represent data, and an ML plugin layer lets us experiment with predictive risk scoring. This separation makes the system easier to reason about and maintain.”

---

### Slide 6 – Architecture Style

“Architecturally, the solution is layered and service‑oriented. We have a presentation layer on top, followed by services, a risk engine and ML layer, repositories, and MySQL at the bottom. Each service exposes well‑defined operations, mostly over REST, so in the future this engine could be reused by multiple channels like mobile apps or core banking systems.”

---

### Slide 7 – System Architecture Overview

“At a high level, users or external systems send transactions into the engine via UI or API. Those transactions are processed by the fraud engine, stored as `Transaction` records, and any rule violations are logged as `FraudLog` entries. There’s also an optional ML plugin that can add risk on top of the rules, giving us both structure and flexibility in how we score fraud.”

---

### Slide 8 – Five‑Layer View

“This slide zooms into our five layers. The UI layer is built in Thymeleaf and shows dashboards. The service layer coordinates business operations. The ML and risk engine layer encapsulates rules and ML plugins. Repositories map to the database, and the database layer persists all activity. Thinking in these layers keeps the design clean and scalable.”

---

### Slide 9 – Transaction Execution Flow

“Here we walk through a single transaction. A user submits a transaction through the UI or API. The controller passes it to `TransactionService`, which stores it and calls `FraudDetectionService`. That service calculates a risk score using rules and optionally ML, derives a status—normal, suspicious, or fraud—and updates the record. Finally, the enriched transaction is returned to the UI or API caller.”

---

### Slide 10 – Component Interaction

“This interaction diagram shows how the parts talk to each other. Controllers stay thin and forward requests to services. Services orchestrate work across repositories and the fraud engine. The fraud engine, in turn, queries past transactions and writes fraud logs. Repositories encapsulate all database interactions, so the rest of the application never needs to know SQL details.”

---

### Slide 11 – Controller Module

“The controller module is our entry point for outside traffic. The dashboard controller serves HTML pages for human users. The transaction and gateway controllers provide REST endpoints for other systems, including bulk simulation. Importantly, controllers don’t implement rules—they delegate to the service and risk layers, which makes it easy to swap front‑ends without touching core logic.”

---

### Slide 12 – Service & Analytics Modules

“Service classes like `TransactionService` encapsulate the main business workflows: creating transactions, invoking fraud checks, and retrieving fraud‑only lists. `TransactionSimulationService` generates synthetic data for testing, and `AnalyticsService` exports a CSV to support downstream reporting or ML training. By keeping this logic in services, it becomes easy to test and to reuse in different contexts.”

---

### Slide 13 – Fraud Logic Summary

“Our fraud logic starts with three simple, explainable rules: high‑amount transactions, rapid transactions in a short time window, and location mismatch compared to a customer’s history. Each rule adds risk points and writes to `FraudLog`. Then, if enabled, an ML plugin can add extra risk for patterns such as very high amounts or unknown locations. We cap risk at 100 and then map it to a status bucket.”

---

### Slide 14 – ML Plugin Design

“Instead of hard‑coding a specific ML model, we define a `FraudMLPlugin` interface. A dummy implementation shows how to plug in custom logic, and a configuration flag decides whether ML is active. This architecture lets us swap or upgrade ML models over time without touching controllers, services, or the database schema—very close to how production fraud systems evolve.”

---

### Slide 15 – Database Design

“The database has two key tables: `transaction` and `fraud_log`. `transaction` stores the core payment details along with a risk score and final status. `fraud_log` links back to transactions and records which rule was triggered, when, and at what risk level. This structure gives us a clean transactional history plus a separate, auditable trail of fraud reasoning.”

---

### Slide 16 – Data Flow: DTO to Entity

“When external systems send data, they use a lightweight DTO that only contains what’s needed—account number, amount, and location. The gateway controller converts that DTO into a full `Transaction` entity. The service then enriches it with timestamps, risk score, and status before persisting. This clear separation between DTOs and entities keeps our external contracts stable even if the internal model grows.”

---

### Slide 17 – Fraud Percentages

“Once the engine is running, we can look at metrics such as the percentage of transactions labeled as fraud or suspicious. These simple ratios—fraud rate and suspicious rate—tell us whether our rules are too strict or too lenient. Even in a demo setting, these metrics are useful to demonstrate how tuning thresholds changes the overall behavior of the system.”

---

### Slide 18 – Analytics View

“The analytics export feature creates a CSV file with amount, location, risk score, and status. Analysts can load that file into Excel, Power BI, or Python to study patterns: which locations are riskier, how risk score correlates with final labels, and how often our rules trigger. This is the bridge between a rule engine and a future, more data‑driven ML model.”

---

### Slide 19 – Stress Testing Story

“To approximate real‑world load, we use the simulation APIs to generate thousands of synthetic transactions. We then observe performance—how quickly transactions are scored, how the database behaves, and whether risk calculations still feel real time. This gives us an early sense of where we might need optimization before deploying such a design at production scale.”

---

### Slide 20 – Simulation Flow

“In simulation mode, the client calls a single endpoint with a count parameter. The system internally generates that many random transactions and passes each one through the exact same pipeline as live traffic—service layer, fraud engine, and database. This ensures that our test data exercises all the same code paths, which is essential for meaningful validation.”

---

### Slide 21 – Benefits of Simulation

“Simulation has three big benefits: it’s safe, fast, and flexible. Safe, because we’re not using real customer data. Fast, because we can generate large volumes of traffic in seconds. And flexible, because we can vary the distributions of amounts and locations to see how the fraud engine behaves under different conditions. This is a powerful way to iterate on rules and models.”

---

### Slide 22 – Execution Steps

“Here we summarize how to actually run the system: set up MySQL and the `frauddb` schema, configure credentials, build with Maven, and start Spring Boot on port 8082. From there, you can interact with the dashboard in a browser or use the REST endpoints directly. These steps are deliberately simple so that new team members can get the system running quickly on their laptops.”

---

### Slide 23 – Example Scenario

“To make this concrete, we walk through a sample: a customer who usually spends small amounts in Mumbai suddenly makes an ₹80,000 transaction in Delhi. Our rules immediately pick up the high value and the location change, assign a high risk score, and mark the transaction as fraud. This example shows how intuitive rules can still capture many real‑world fraud patterns.”

---

### Slide 24 – Threshold Logic

“The risk score between 0 and 100 is translated into three buckets: normal under 30, suspicious from 30 to 69, and fraud at 70 and above. These thresholds are simple but powerful, and they’re configurable. They allow us to distinguish between transactions that should proceed, those that might need extra verification, and those that should be blocked or escalated immediately.”

---

### Slide 25 – Real‑World Comparison

“Compared to full‑scale bank systems, our project is intentionally simplified: fewer rules, one region, and no multi‑channel integration. Real platforms include hundreds of signals, sophisticated ML, and case management tools. But even with this simplified scope, our design mirrors how those systems are structured and provides a strong foundation for learning and experimentation.”

---

### Slide 26 – Why Hybrid Rules + ML

“We deliberately combine rules and ML because each has strengths. Rules are transparent and regulator‑friendly, while ML can detect complex, evolving patterns. A hybrid approach lets us keep the business comfortable—because they can see and tune rules—while still benefiting from data‑driven improvements as we plug in smarter models over time.”

---

### Slide 27 – False Positives vs False Negatives

“Every fraud system balances two types of mistakes. A false positive is when we block a legitimate customer—hurting experience. A false negative is when we miss fraud—costing money. This slide highlights how our thresholds and rules influence that balance, and how analytics from the exported data can help us tune the system toward an acceptable trade‑off.”

---

### Slide 28 – Scalability

“If we wanted to scale this engine, we’d horizontally scale the stateless API layer, optimize the most frequent queries in the fraud engine, and add indexes and possibly read replicas on the database. Containerization and monitoring would help us handle spikes in volume. This design is intentionally compatible with typical cloud‑native scaling patterns.”

---

### Slide 29 – Future Enhancements

“Looking ahead, we can enrich the rule set, plug in real trained ML models, build more powerful dashboards, and add alert workflows for fraud analysts. We can also introduce message queues or streams to score events in near real time. In other words, this prototype is a starting point that can grow into a more production‑like architecture.”

---

### Slide 30 – Conclusion

“To summarize, we’ve implemented a complete fraud detection pipeline: from transaction ingestion, through risk scoring and persistence, to analytics export. The architecture is layered, explainable, and extensible, making it suitable both as a learning tool and as a foundation for more advanced work. We believe this approach closely reflects how modern banking fraud engines are built, while remaining easy to understand.”

