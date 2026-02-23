## Explanation Notes – 30 Slides

Plain‑language explanations aimed at onboarding a new team member, slide by slide.

---

### Slide 1 – Title

This slide introduces the project name and what it does.  
Explain that it is a fraud detection prototype for digital banking, built using Java, Spring Boot, Thymeleaf, and MySQL.  
Mention that it covers the full flow from receiving a transaction to deciding whether it is normal or fraud.  
Clarify that the goal of the presentation is to understand both the business problem and the technical solution.

---

### Slide 2 – Problem Statement (Manual)

Here we describe the current pain: many banks still handle fraud detection with a lot of manual work.  
Fraud analysts often check spreadsheets or dashboards after something suspicious happens, which is slow.  
This manual process does not scale when transaction volumes grow, and decisions can differ between analysts.  
We are setting up the motivation for an automated engine that can help prioritize risky transactions quickly.

---

### Slide 3 – Problem Objective

This slide states what our project is trying to achieve.  
We want a system that can take digital transactions as input, calculate a fraud risk score, and classify each transaction.  
The system should be easy to understand so that business stakeholders can see how decisions are made.  
We also want it to be extensible, so that in the future we can plug in more rules or real ML models without rewriting everything.

---

### Slide 4 – Real‑World Relevance

Here we connect the prototype to real use‑cases.  
We show that similar logic is used in banks, FinTech apps, credit card processors, and ATM networks.  
Any time money moves digitally, there is a risk of unauthorized or fraudulent activity.  
By simulating some of these scenarios, our project demonstrates how a fraud engine can fit into real payment flows.

---

### Slide 5 – Backend Roles

This slide lists the main backend building blocks and what each one does.  
Controllers handle incoming HTTP requests and decide which service to call.  
Services implement business workflows such as creating a transaction or calculating fraud risk.  
Repositories are the data access layer, talking to MySQL using Spring Data JPA.  
Models and DTOs represent how data is stored in the database versus how it is passed over APIs.  
The ML plugin layer is a clean place to attach more advanced prediction logic.

---

### Slide 6 – Architecture Style

We explain that the solution follows a layered architecture and is service‑oriented.  
Each layer has a clear responsibility: UI, services, risk/ML engine, repositories, and database.  
Service‑oriented means we expose clear operations, mainly over REST, that other systems can call.  
This separation makes it easier to change one part (for example, swap a database or ML model) without breaking the rest.

---

### Slide 7 – System Architecture Overview

This slide shows the complete system at a high level.  
Users or external systems send transactions to the engine via UI forms or API calls.  
The core engine processes these transactions, calculates risk, updates status, and stores them in the database.  
`Transaction` is the main entity for payment records; `FraudLog` is used to record which fraud rules triggered.  
There is an option to use an ML plugin that adds more risk based on advanced patterns.

---

### Slide 8 – Five‑Layer Architecture

Here we look at each of the five layers individually.  
The UI layer uses Thymeleaf templates to show HTML pages with forms and tables.  
The service layer runs the business processes, like creating transactions and exporting analytics.  
The ML/risk layer contains `FraudDetectionService` and the plugin interfaces used to compute risk.  
The repository layer uses interfaces like `TransactionRepository` to talk to MySQL.  
The database layer physically stores the data in tables for transactions and fraud logs.

---

### Slide 9 – Transaction Execution Flow

This slide explains what happens when a single transaction goes through the system.  
First, the transaction is submitted through the UI form or via a REST API endpoint.  
The controller passes this to `TransactionService.createTransaction`, which sets timestamps and saves it.  
`FraudDetectionService` is then called to compute a risk score based on rules and ML.  
The system sets the status (normal, suspicious, or fraud), updates the transaction in the database, and returns it to the caller.

---

### Slide 10 – Component Interaction

Here we show how the components interact with each other.  
Controllers do not contain business rules; they simply forward requests to services.  
Services orchestrate calls to repositories and to the fraud engine.  
The fraud engine itself uses repositories to query historical data and writes new `FraudLog` records.  
Repositories turn Java method calls into SQL queries using Spring Data JPA, which simplifies data access.

---

### Slide 11 – Controller Module

This slide focuses on the controller classes.  
`DashboardController` maps web URLs like “/” and “/fraud-report” to pages rendered with Thymeleaf.  
`TransactionController` and `TransactionApiGatewayController` expose REST endpoints for programmatic access.  
These controllers are deliberately kept thin, meaning they do not know the detailed fraud rules.  
New team members should understand that controllers are mainly about routing and not about business logic.

---

### Slide 12 – Service Layer & Analytics

Here we explain the responsibilities of each service.  
`TransactionService` is the central place for creating transactions and ensuring fraud checks are applied.  
`FraudDetectionService` contains the rules and communicates with the ML plugin if enabled.  
`TransactionSimulationService` generates random transactions to test the system under load.  
`AnalyticsService` exports a CSV file with key fields so analysts can study patterns or build ML models.  
The key idea is that services hide complexity from controllers and group related logic together.

---

### Slide 13 – Fraud Detection Logic

This slide goes into detail on the rules.  
Rule 1: if the amount is very high (greater than 50,000), we consider that risky and add many points.  
Rule 2: if there are several transactions in a short time window from the same account, we add more risk.  
Rule 3: if the location suddenly changes compared to past transactions, we treat this as unusual.  
Each time a rule triggers, we create a `FraudLog` entry so we can later see exactly what happened.  
After rules, an ML plugin may add more risk, and finally we cap the total score at 100.

---

### Slide 14 – ML Plugin Architecture

Here we clarify how ML fits into the system.  
We define an interface `FraudMLPlugin` that any ML model class can implement.  
The current dummy implementation just checks for very high amounts and unknown locations, adding risk in a simple way.  
Whether ML is used or not is controlled by a configuration flag in `application.properties`.  
This pattern shows how we can later replace the dummy plugin with a real ML model without rewriting the rest of the code.

---

### Slide 15 – Database Design

This slide describes the schema at a beginner level.  
The `transaction` table holds one row per financial transaction with fields like account number, amount, and status.  
The `fraud_log` table holds one row per triggered rule, with a link back to the related transaction.  
Storing logs separately keeps the main transaction table clean and makes it easy to see a history of rule evaluations.  
Hibernate (JPA) automatically creates or updates these tables based on the Java entity classes.

---

### Slide 16 – Data Flow: DTO, API, and Entity

We show how data moves from outside the system into our entities.  
External callers send JSON that matches a `TransactionRequestDTO` with only three fields: account number, amount, and location.  
The gateway controller converts this DTO into a `Transaction` entity, which has more fields like timestamps and risk score.  
`TransactionService` fills in those extra internal fields before saving to the database.  
This separation allows us to change internal details without breaking external clients, and it’s a common best practice.

---

### Slide 17 – Fraud Percentage / Metrics

This slide explains the idea of basic metrics.  
From the database, we can count how many transactions are labeled as fraud, suspicious, or normal.  
Fraud percentage is calculated as fraud count divided by total transactions, times 100.  
Similarly, we can calculate the suspicious percentage.  
These numbers help the team understand whether the rules are too sensitive or not sensitive enough.

---

### Slide 18 – Analytics Model and Rates

Here we connect the export feature to high‑level analytics.  
`AnalyticsService` writes a CSV file with four columns: amount, location, risk score, and final status.  
With this file, analysts can group transactions by location or amount range and calculate fraud rates in each group.  
They can also see how risk scores relate to final labels and think about alternative thresholds.  
This exported data can become training input for a proper ML model in future iterations.

---

### Slide 19 – Stress Testing

This slide introduces the concept of stress tests.  
In production, systems must handle many transactions per second, so we need to see how our engine behaves under load.  
Using the simulation APIs, we can quickly insert thousands of fake transactions.  
While doing this, we observe application performance and database behavior to find bottlenecks.  
This prepares us to think about optimization and scaling early.

---

### Slide 20 – Simulation Flow

Here we walk through what happens in a simulation run.  
The client calls a simulation endpoint with a number like 1000, meaning we want 1000 simulated transactions.  
The system generates those transactions in memory and sends each one through the same pipeline as real traffic.  
That includes setting timestamps, calculating risk, and writing to the database.  
The end result is a realistic dataset that we can use for demos or analysis.

---

### Slide 21 – Benefits of Simulation

This slide summarizes why simulation is useful.  
We don’t need real customer data, so there are no privacy or security concerns.  
We can quickly generate many different scenarios to see how the rules behave.  
It’s also an inexpensive way to generate labeled data (normal vs. fraud) for teaching or ML experiments.  
New team members can safely practice with the system without any production risk.

---

### Slide 22 – Step‑by‑Step Execution

Here we provide simple instructions a new developer can follow.  
First, they install the required tools: JDK 21, Maven, and MySQL, and they create the `frauddb` database.  
Next, they configure the database username and password in `application.properties`.  
Then they run the Maven build and start the Spring Boot app.  
Finally, they open the dashboard in a browser and try creating some transactions to see the system in action.

---

### Slide 23 – Example Execution Scenario

This slide illustrates the engine with a concrete example.  
We describe a customer with a known behavior pattern who suddenly performs a high‑value transaction from a new city.  
The rules see that the amount is above the configured high‑amount threshold and that the location has changed.  
They add risk points, and the final status becomes fraud, which appears in the fraud report.  
This makes the abstract logic more intuitive for newcomers.

---

### Slide 24 – Risk Threshold Logic

Here we clarify the mapping from numeric risk score to label.  
The risk value is between 0 and 100, and we cut this range into three segments.  
Scores under 30 are treated as normal; scores between 30 and 69 are suspicious; and scores 70 and above are fraud.  
We explain that these thresholds are design choices that can be tuned over time.  
Understanding this mapping is important, because business teams will often ask to adjust these cut‑offs.

---

### Slide 25 – Real Banking Comparison

This slide reminds the audience that the project is a simplified model.  
Real banks use many more signals, such as device fingerprints, IP addresses, and external risk feeds.  
They also have complex workflows for analysts, case management, and regulatory reporting.  
However, the architectural principles—ingestion, scoring, storage, and analytics—are very similar.  
This helps new team members see where our prototype fits in the bigger picture.

---

### Slide 26 – Why Hybrid Rules and ML

Here we explain why we don’t rely only on rules or only on ML.  
Rules are easy to understand and explain to non‑technical stakeholders, but they can miss complex behavior.  
ML can find non‑obvious patterns in data but can be harder to explain.  
By combining both, we keep transparency while still gaining some benefits of data‑driven models.  
Our plugin interface is the technical mechanism that makes this hybrid approach possible.

---

### Slide 27 – False Positives and Negatives

This slide introduces two key quality metrics for fraud systems.  
A false positive is when the system flags a legitimate transaction as fraud, causing customer annoyance.  
A false negative is when the system misses an actual fraud, causing financial loss.  
We explain that changing thresholds or rules will affect the rate of both types of errors.  
Understanding this trade‑off is essential for designing and tuning any fraud detection system.

---

### Slide 28 – Scalability Considerations

Here we talk about what would be needed if the system grew larger.  
We would likely run multiple application instances behind a load balancer to handle more traffic.  
We’d optimize queries and add database indexes on important columns to keep lookups fast.  
We might introduce caching or streaming platforms if we needed near real‑time scoring at scale.  
New team members should understand that the current design is compatible with standard scaling patterns.

---

### Slide 29 – Future Enhancements

This slide lists ideas for continuing the project.  
We can add more rules and integrate additional data sources, such as device information.  
We can implement a real ML model using the exported CSV as training data.  
We can build richer dashboards, add user roles, and create a simple case management UI.  
These enhancements show how the prototype can gradually evolve toward a more realistic system.

---

### Slide 30 – Conclusion

The final slide recaps the key achievements.  
We built an end‑to‑end system that simulates many core ideas of fraud detection engines.  
The architecture is layered, modular, and ready for further extensions in both rules and ML.  
For new team members, this project is an excellent sandbox to learn about Spring Boot, databases, and fraud analytics.  
It also provides a common reference point for future discussions about improvements and production‑grade design.

