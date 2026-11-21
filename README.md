# SmartRetail P3 — Rule-Based Recommendation & Identity Matcher

## 1. Team & Course Information

* **Course:** Fundamentals of Programming (C++17)

* **Project:** P3 — Rule-Based Recommendation & Identity Matcher

* **Group:** Group 03

* **Lecturer:** `vvvanthe`

* **Repository:** `https://github.com/tryuht08/smartretail-p3-group03`

| Role | Full Name | GitHub Username | 
| ----- | ----- | ----- | 
| Leader | Tran Thu Thuy | `@tryuht08` | 
| Member | Tran Quoc Khang Hy | `@tkhanghy620-gif` | 
| Member | Le Ha Minh Anh | `@Alice-art361` | 
| Member | Nguyen Vu Duc | `@echconvuive` | 

## 2. Problem Specification (Input – Process – Output)

### Input

The program reads 4 CSV files from the specified data directory (`data/sample`, `data/full`, `data/edge/empty`, `data/edge/dirty`):

* `products.csv`: `product_id, name, category, brand, price, rating`

* `customer_events.csv`: `session_id, timestamp, event, product_id` (only `VIEW` events are used to infer preferences)

* `identity_links.csv`: `session_id, customer_id, linked_at, link_source`

* `transactions.csv`: `transaction_id, customer_id, product_id, quantity, total_amount`

### Process

1. **Identity Matching:** Link each `session_id` to a `customer_id` using `identity_links.csv` (if a session is linked to two customers, keep the first valid line). Compute total sessions, linked sessions, and `Identity match rate` (safely handling 0 sessions without division by zero).

2. **Preference Inference (3 Branches):**

   * **Branch (a):** Session is identified and the customer has valid purchase history (excluding `GUEST`) -> infer the most frequent `category` and `brand` from purchased products; `budget = max_unit_price_bought * 120 / 100`.

   * **Branch (b):** Session is anonymous, or identified with no purchase history, and has `VIEW` events -> infer `category` and `brand` from viewed products; `budget = max_price_viewed`.

   * **Branch (c):** Neither purchase history nor `VIEW` events exist -> no preferred `category` or `brand`; `budget = max_price_in_catalogue`.

   * On a tie for `category` or `brand` frequency, select the value that reaches the maximum count earliest when scanning from the start of the list.

3. **Scoring & Top-K Ranking:**

   * Hard constraint: exclude any product where `price > budget`.

   * Score remaining products (max 10 points): same `category` (+3), same `brand` (+3), `rating >= 4.5` (+2), and `price * 10 <= budget * 8` (+2).

   * Sort with a hand-written algorithm by: `score` descending -> `rating` descending -> `price` ascending -> `product_id` ascending, then output the top `K` products (default `K = 3`).

### Output

* `IDENTITY MATCHING` summary: total sessions, linked sessions, `session_id -> customer_id` pairs, and `Identity match rate`.

* Per-session report (all sessions or a specified `session_id`): selected branch, products used for inference, inferred preferences and budget, Top-K recommendations (with scoring reasons and `Excluded (over budget)` list as the Extension).

## 3. Build & Run

Create the `bin/` directory if needed, compile cleanly with C++17 (`-Wall -Wextra`, zero warnings), and run:

```
mkdir -p bin
g++ -std=c++17 -Wall -Wextra src/main.cpp -o bin/p3

```

Example commands:

```
bin/p3 data/sample
bin/p3 data/full
bin/p3 data/full ANON-9100 3
bin/p3 data/edge/empty
bin/p3 data/edge/dirty

```

## 4. Assumptions & Limitations

* All monetary values (`price` and `budget`) use 64-bit integers (`long long`, in VND). The 80% budget threshold is evaluated using integer arithmetic (`price * 10 <= budget * 8`) to avoid floating-point rounding errors.

* Branch (a) assumes a customer's budget is `max_unit_price_bought * 120 / 100` (20% above the highest unit price previously purchased).

* Invalid CSV lines (wrong column count, `quantity <= 0`, unknown `product_id`, empty `customer_id`, or `session_id` absent from `customer_events.csv`) are skipped with a warning and line number printed to `cerr`.

* All `GUEST` transactions are ignored when inferring customer preferences.

* The implementation uses only Sessions 1–7 C++ concepts (`struct`, `vector`, functions, `&` references, recursion, hand-written sorting and searching; no classes, pointers, `std::map`, `std::set`, or `std::sort`).

## 5. What This Project Does NOT Do (Boundary)

* The program uses **only** `identity_links.csv` to link sessions to known customers.

* It does **not** generate new customer behaviour events.

* It does **not** manage inventory levels, stock alerts, or restock orders.

* It does **not** read any data files outside the 4 CSV files specified for P3.

## 6. Task Assignment Table

| ID | Task | Function / file | Assignee | Reviewer | Flowchart | PR | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| T01 | Repo skeleton, `.gitignore`, PR template & README task table | `.gitignore`, `README.md`, `.github/pull_request_template.md` | Tran Thu Thuy (`@tryuht08`) | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | not needed | #1 | 
| T02 | Define structs, read `products.csv` & recursive binary search | `loadProducts()`, `findProductBinary()` | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | Le Ha Minh Anh (`@Alice-art361`) | yes | — | 
| T03 | Read `customer_events.csv`, group sessions & filter `VIEW` | `loadEvents()`, `findOrCreateSession()` | Le Ha Minh Anh (`@Alice-art361`) | Nguyen Vu Duc (`@echconvuive`) | yes | — | 
| T04 | Read `identity_links.csv` (keep first) & `transactions.csv` (skip `GUEST`) | `loadIdentityLinks()`, `loadTransactions()`, `findLink()` | Tran Thu Thuy (`@tryuht08`) | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | yes | — | 
| T05 | Parse CLI arguments & print `IDENTITY MATCHING` summary | `parseArgs()`, `printIdentitySummary()` | Nguyen Vu Duc (`@echconvuive`) | Tran Thu Thuy (`@tryuht08`) | yes | — | 
| T06 | Draft system architecture, function call tree & struct table | `docs/architecture.md` | Tran Thu Thuy (`@tryuht08`) | Le Ha Minh Anh (`@Alice-art361`) | not needed | — | 
| T07 | **[CORE FUNCTION]** Branch selection (a)/(b)/(c), preference & budget inference | `inferPreference()`, `mostFrequentValue()` | Tran Thu Thuy (`@tryuht08`) | Nguyen Vu Duc (`@echconvuive`) | yes | — | 
| T08 | Score products (4 rules), hard budget filter & collect reasons | `scoreProduct()`, `evaluateCatalogue()` | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | Tran Thu Thuy (`@tryuht08`) | yes | — | 
| T09 | 4-level tie-breaker & hand-written sort for Top-K | `comesBefore()`, `sortRecommendations()`, `topK()` | Le Ha Minh Anh (`@Alice-art361`) | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | yes | — | 
| T10 | Print session report, Top-K list & excluded products | `printSessionReport()`, `formatCurrency()` | Nguyen Vu Duc (`@echconvuive`) | Le Ha Minh Anh (`@Alice-art361`) | yes | — | 
| T11 | Design `tests/test_cases.md` (TC01–TC07) with `Expected` outputs | `tests/test_cases.md`, `tests/expected/` | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | Nguyen Vu Duc (`@echconvuive`) | not needed | — | 
| T12 | Run tests on `full`/`edge`, fill `Actual`, write `run_tests.sh` & finalize docs | `tests/test_cases.md`, `tests/run_tests.sh`, `docs/architecture.md` | Le Ha Minh Anh (`@Alice-art361`) | Tran Thu Thuy (`@tryuht08`) | not needed | — | 
| T13 | Prepare presentation slides (max 10 pages PDF) & live demo script | `docs/slides.pdf` | Nguyen Vu Duc (`@echconvuive`) | Tran Quoc Khang Hy (`@tkhanghy620-gif`) | not needed | — | 

## 7. AI Declaration & References

| Tool / Source | Purpose | Affected Files / Scope | 
| ----- | ----- | ----- | 
| Gemini | Structured project requirements, drafted `README.md` and task assignment table, and verified core `inferPreference` flowchart logic | `README.md`, `docs/flowcharts/T07-infer-preference.md` | 
| Course Materials (MiiN-SmartRetail Brief & GitHub Guide) | CSV data schemas, scoring rules, Pull Request workflow, and standard flowchart conventions | Entire repository | 
