# Lacoste QA Technical Test
### by Hamza BOUNAAMANE

---

## 📁 Repository Structure

```
lacoste-qa-test/
│
├── 📂 web-tests/
│   └── test-cases.md          ← 31 prioritized test cases (P0/P1/P2)
│
├── 📂 postman/
│   ├── lacoste-order-management.postman_collection.json   ← Full API test suite
│   └── lacoste-qa-local.postman_environment.json         ← Environment variables
│
├── 📂 mobile-tests/
│   └── mobile-strategy.md     ← App analysis, user journeys, test strategy
│
└── README.md                  ← You are here
```

---

## 🌐 Exercise 1 — Web: Order Management

### What's covered

**Feature under test:** Order Management UI (search, release, cancel, ship, return, short)

→ **[View all test cases](./web-tests/test-cases.md)**

The test matrix covers **31 test cases** across 7 modules:

| Module | Cases | Priority |
|--------|-------|---------|
| Order Search | TC001–TC005 | P0 |
| Release Order | TC006–TC009 | P0 |
| Cancel Order | TC010–TC013 | P0 |
| Ship | TC014–TC017 | P0 |
| Return | TC018–TC020 | P1 |
| Short (Stock Shortage) | TC021–TC023 | P1 |
| Multi-items & UX | TC024–TC031 | P1/P2 |

---

## 📬 Postman Collection

### How to import

1. Open **Postman**
2. Click **Import** → drag and drop both files from the `/postman/` folder:
   - `lacoste-order-management.postman_collection.json`
   - `lacoste-qa-local.postman_environment.json`
3. Select the **"Lacoste QA — Local / Demo"** environment in the top-right dropdown
4. Run the collection via **Collection Runner** or individually

> **Note:** The collection uses `https://postman-echo.com` as a proxy since the real API is not publicly accessible. Mock response objects are embedded in each test script to simulate real assertions.

### Collection structure

```
📁 Lacoste — Order Management API
  ├── 🔍 Order Search
  │   ├── TC001 — Valid order (Happy Path)       → asserts structure, items, fields
  │   ├── TC002 — Non-existent order             → expects 404
  │   ├── TC003 — Empty orderId                  → expects 400/422
  │   └── TC004 — Special characters injection   → no 500 error
  │
  ├── 📦 Release Order
  │   ├── TC006 — Release (Happy Path)           → status = RELEASED, timestamp
  │   └── TC007 — Release already released       → idempotency check
  │
  ├── ❌ Cancel Order
  │   ├── TC010 — Cancel active order            → status = CANCELLED
  │   └── TC011 — Cancel shipped order           → 403/409 blocked
  │
  ├── 🚚 Ship Item
  │   ├── TC014 — Ship released item             → shipped=true, shippedAt
  │   └── TC015 — Ship non-released item         → 409/422 blocked
  │
  ├── ↩️ Return Item
  │   ├── TC018 — Return shipped item            → status = RETURNED
  │   └── TC019 — Return non-shipped item        → 409/422 blocked
  │
  ├── ⚠️ Short (Stock Shortage)
  │   ├── TC021 — Declare Short                  → status = SHORT, qty = 0
  │   └── TC023 — Short after Ship               → 409/422 blocked
  │
  └── 🔒 Security & Edge Cases
      ├── SEC01 — Missing Authorization header   → 401 expected
      ├── SEC02 — SQL Injection in orderId       → no 500, valid JSON
      └── PERF01 — Response time < 3s            → performance baseline
```

### Key design decisions

- **Collection variables** carry state between requests (e.g. `releaseId` is extracted from the search response and reused in ship/return/short calls)
- **Mock responses** embedded in test scripts to enable assertions without the real backend
- **`X-Request-ID: {{$guid}}`** header added to mutation requests for traceability
- **Security tests** included as a dedicated folder (injection, missing auth, performance)
- Each request has a **saved example response** showing the expected payload shape

---

## 📱 Exercise 2 — Mobile: Sauce Labs Sample App

→ **[View full strategy](./mobile-tests/mobile-strategy.md)**

### Summary

| Section | Content |
|---------|---------|
| App Analysis | 7-phase workflow, 4 user journeys, 5 system integrations |
| Test Levels | Unit → Integration → E2E → Non-Functional |
| Risk Matrix | 8 areas rated Critical / High / Medium |
| Tooling | Appium + WebDriverIO, Jest, GitHub Actions, TestRail |
| Sample Code | Appium E2E test for Happy Path purchase flow |

The strategy deliberately includes:
- **`problem_user` regression suite** — catching the pre-programmed bugs
- **Biometric authentication** test paths (Face ID / Touch ID cancel + fallback)
- **Accessibility** requirements (testID attributes, touch targets, VoiceOver)
- **Security** checks (credentials storage, session invalidation)

---

## 🛠️ Tech Stack Covered

| Layer | Tools |
|-------|-------|
| API Testing | Postman (collection + environment) |
| Web Manual QA | Test case matrix (Markdown) |
| Mobile E2E | Appium + WebDriverIO |
| Unit Testing | Jest + React Native Testing Library |
| CI/CD | GitHub Actions |
| Defect Tracking | Jira |
| TMS | TestRail / Zephyr Scale |

---

## 📄 Author

**Hamza BOUNAAMANE**  
QA Engineer  
*Lacoste Technical Test — June 2026*
