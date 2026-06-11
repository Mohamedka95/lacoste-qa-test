# Mobile Testing Strategy — Sauce Labs Sample App

**Application:** Sauce Labs Sample App (React Native)  
**Platforms:** Android 9+ / iOS 14+  
**Reference:** [Android APK](https://github.com/saucelabs/sample-app-mobile/releases) · [iOS .ipa](https://github.com/saucelabs/sample-app-mobile/releases)

---

## 1. Application Analysis

### Core Workflow

The app implements a linear e-commerce checkout funnel in 7 phases:

| Phase | Screen | Core Logic |
|-------|--------|------------|
| 1 | **Login** | Username/password inputs, input validation, biometric (Face ID / Touch ID) alternative |
| 2 | **Catalog** | Product grid, layout toggle (grid/list), sorting (A–Z, Z–A, price asc/desc) |
| 3 | **Product Detail** | Item details, add-to-cart / remove-from-cart, back navigation |
| 4 | **Cart** | Selected items overview, line-item deletion, routing to Checkout or Catalog |
| 5 | **Checkout Info** | Mandatory form (First Name, Last Name, Zip Code), client-side validation |
| 6 | **Order Overview** | Financial summary: subtotal + local tax calculation |
| 7 | **Confirmation** | Transaction success display, reset button → back to Catalog root |

---

### Key User Journeys

**Journey 1 Standard Purchase (Happy Path)**  
`standard_user` logs in → browses catalog → opens product detail → adds 2+ items to cart → fills checkout form → confirms order → sees confirmation screen → resets to catalog.

**Journey 2 Locked Out User**  
`locked_out_user` attempts login → immediate UI error blocks catalog access. Verifies authentication error handling.

**Journey 3 Problem User (Bug Regression)**  
`problem_user` exposes pre-programmed bugs:
- Subtotals double unexpectedly on the overview screen
- "About" link returns HTTP 404
- Cart does not reset after order completion

**Journey 4 Lateral Drawer Navigation**  
Side drawer allows access to: WebView, QR Code Scanner, GPS Map, Canvas Pad. Also exposes logout and global cart clear.

---

### System Interactions

| System / Module | Interaction | QA Impact |
|----------------|-------------|-----------|
| **SyncStorage** | Local session persistence + JSON cart serialization | State consistency across screens; cart must survive backgrounding |
| **react-native-biometrics** | Face ID / Touch ID native API calls | Requires testing cancel and fallback behaviors |
| **Geolocation** | GPS coordinate retrieval | Native permission dialog must be handled at first launch |
| **QR Code Scanner** | Camera video feed capture | Camera permission flow + stream stability |
| **WebView** | In-app browser window | Network-dependent; external page integrity |

---

## 2. Testing Strategy

### Test Level 1 Unit Testing

**Scope:** Pure business logic, isolated from UI and network.

Key test targets:
- `Credentials.verifyCredentials()` all user profiles including edge cases
- `ShoppingCart` model: `addItem()`, `removeItem()`, `resetCart()`, state integrity
- Tax calculator
- Deep link parser: valid and malformed URIs


---

### Test Level 2 Integration Testing

**Scope:** Module-to-module data exchange and navigation routing.

Key test targets:
- Login → Catalog data handoff (session token, user role)
- Cart badge real-time sync across all screens
- Global cart reset from drawer → badge clears immediately
- `problem_user` state propagation across screens


---

### Test Level 3 Functional / End-to-End Testing


Critical E2E flows executed on **both platforms** (iOS + Android):

| Scenario | User Profile | Priority |
|----------|-------------|---------|
| Complete purchase funnel (login → confirmation) | `standard_user` | **Critical** |
| Blocked login attempt | `locked_out_user` | **Critical** |
| Cart add/remove stability | `standard_user` | **Critical** |
| Tax calculation accuracy | `standard_user` | **Critical** |
| Checkout form validation | `standard_user` | **Critical** |
| Bug regression suite | `problem_user` | High |
| Biometric authentication | `standard_user` | High |
| Drawer navigation flows | `standard_user` | High |
| Catalog sort + layout toggle | `standard_user` | Medium |
| WebView / QR / GPS modules | `standard_user` | Medium |


---

### Test Level 4 Non-Functional Testing

#### Performance
- Catalog must be interactive in < 2s on simulated 4G
- Add-to-cart action: UI feedback in < 500ms
- Checkout submission: response in < 3s

#### Compatibility
- iOS: 14, 16, 17 (iPhone SE, 13, 15, 15 Pro)
- Android: 9, 11, 13 (Pixel 6, Samsung Galaxy S22, mid-range device)
- Tablet layouts (iPad, Android 10")

#### Security
- Credentials not stored in plaintext in `SyncStorage`
- Session token invalidated on logout
- No sensitive data in crash logs

---

## 3. Risk Matrix

| Feature / Area | Risk Description | Priority |
|---------------|-----------------|---------|
| Authentication (Form + Biometrics) | Global blocker — users cannot access catalog | **Critical** |
| Cart Operations (Add/Remove) | Breaks transactional flow; direct revenue loss | **Critical** |
| Financial Calculations (Tax + Total) | Incorrect charges — commercial and compliance risk | **Critical** |
| Checkout Form | Order data transmission failure blocks dispatch | **Critical** |
| `problem_user` bug regression | Test gaps where expected failures go undetected | High |
| State Persistence | Cart loss on app backgrounding — user frustration | High |
| Catalog Sort & Layout | UI degradation without blocking checkout | Medium |
| WebView / QR / GPS modules | Annex features fail; UX degrades but shopping unaffected | Medium |

---

## 4. Tooling & QA Ecosystem

| Category | Technology |
|----------|-----------|
| E2E Automation | Appium + WebDriverIO (or Detox for RN-native) |
| Unit & Component | Jest + React Native Testing Library |
| CI/CD | GitHub Actions, Bitrise, or CircleCI |
| Test Case Management | TestRail or Zephyr Scale for Jira |
| Defect Tracking | Jira |
| Physical Devices | iPhone 13, iPhone 15, Pixel 6, Samsung Galaxy S22 |
| Emulators / Simulators | Xcode Simulator (iOS) + Android Emulator (AVD) |

---

## 5. Entry & Exit Criteria

### Entry Criteria (Go-Testing)
- App package compiled, signed, and installable on both platforms (`.apk` and `.ipa`/`.app`)
- Test environment and backend mock APIs fully operational
- Test suite reviewed and signed off by QA lead

### Exit Criteria (Done-Testing)
- 100% of Critical (P0) scenarios pass
- Zero open blocking or critical defects
- ≥ 95% pass rate on all High-priority cases
- Release summary report signed off by engineering stakeholders

---

## 6. QA Deliverables

1. **Test Plan**  global strategic documentation covering web and mobile
3. **Execution Summary Report** pass/fail stats, screenshots, debug logs
4. **Defect Tickets** Jira logs with reproduction steps, actual vs expected, environment
5. **QA Metrics Dashboard** coverage trends, pass rates, flakiness tracking

---
  });
});
```
