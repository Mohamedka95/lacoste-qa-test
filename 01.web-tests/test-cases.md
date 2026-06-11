# Web Test Cases — Order Management

**Feature:** Order Management (WEB)  
**Total cases:** 31  
**Priority levels:** P0 (Critical) · P1 (High) · P2 (Medium)

---

## Priority Matrix

| ID     | Module              | Description                                        | Priority | Expected Result |
|--------|---------------------|----------------------------------------------------|----------|-----------------|
| TC001  | Order Search        | Search with valid Order ID + FR region             | **P0**   | Order found, items displayed correctly |
| TC002  | Order Search        | Search with non-existent Order ID                  | **P0**   | Clear "Order not found" error, no items |
| TC003  | Order Search        | Search with empty Order ID                         | **P0**   | Button disabled OR validation message |
| TC004  | Order Search        | Search with special characters (`ORD-123#@!`)      | **P0**   | No 500 error; graceful rejection or "not found" |
| TC005  | Order Search        | Search with 100+ char Order ID                     | **P0**   | No UI break; maxlength enforced or graceful rejection |
| TC006  | Release Order       | Successful Release Order                           | **P0**   | All items status → RELEASED |
| TC007  | Release Order       | Release an already released order                  | **P0**   | Button grayed out; no duplicate action |
| TC008  | Release Order       | Verify status update after release                 | **P0**   | Status visually updates instantly |
| TC009  | Release Order       | Double-click on Release Order                      | **P0**   | Only 1 request sent; no error or duplicate |
| TC010  | Cancel Order        | Successful cancellation (non-shipped)              | **P0**   | Confirmation pop-up → status = CANCELLED |
| TC011  | Cancel Order        | Cancel a shipped order                             | **P0**   | Cancel button hidden/grayed — impossible |
| TC012  | Cancel Order        | Verify status persists after cancellation          | **P0**   | CANCELLED state persists after page refresh |
| TC013  | Cancel Order        | Double-click on Cancel Order                       | **P0**   | First click handled; second ignored; no crash |
| TC014  | Ship                | Ship a released item                               | **P0**   | Item status → SHIPPED |
| TC015  | Ship                | Ship a non-released item                           | **P0**   | Ship button blocked/grayed |
| TC016  | Ship                | Verify `Shipped: Yes` indicator                    | **P0**   | Indicator clearly shows shipped state |
| TC017  | Ship                | Double-click on Ship                               | **P0**   | Single shipping notification; no error |
| TC018  | Return              | Return a shipped item                              | **P1**   | Return workflow initiated; status → RETURNED |
| TC019  | Return              | Return a non-shipped item                          | **P1**   | Return action unavailable |
| TC020  | Return              | Verify status after return                         | **P1**   | Status changes to RETURNED |
| TC021  | Short               | Declare an item as Short                           | **P1**   | Item marked SHORT; available qty → 0 |
| TC022  | Short               | Verify quantity impact after Short                 | **P1**   | Qty drops to 0 or logistics alert triggered |
| TC023  | Short               | Short after Ship                                   | **P1**   | Action blocked/grayed — cannot short shipped item |
| TC024  | Multi-items         | Release item 1 only (partial)                     | **P1**   | Item A = RELEASED, Item B = CREATED, order = PARTIAL |
| TC025  | Multi-items         | Ship item 2 only                                   | **P1**   | Item B = SHIPPED, Item A unchanged |
| TC026  | Multi-items         | Simultaneous actions on selected items             | **P1**   | All selected items update simultaneously |
| TC027  | UX / Validation     | Readable error messages                            | **P2**   | No raw stack traces; human-friendly text |
| TC028  | UX / Validation     | Mobile responsive (375px)                          | **P2**   | Table adapts; no button overlaps |
| TC029  | UX / Validation     | Keyboard navigation (no mouse)                     | **P2**   | All fields/actions reachable via Tab + Enter |
| TC030  | UX / Validation     | Response time < 3s                                 | **P2**   | Data displayed in under 3 seconds |
| TC031  | UX / Validation     | Loading indicator on slow network                  | **P2**   | Spinner displayed during fetch |

---

## Detailed Test Cases

### TC001 — Search with valid Order ID

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Order Search |
| **Pre-cond**  | Order `GFR35150_12356` exists in DB for region FR |
| **Steps**     | 1. Enter `GFR35150_12356` in the Order ID field. 2. Select `FR` in the country dropdown. 3. Click **LOOK FOR THE ORDER**. |
| **Expected**  | Order is found. Header shows `Order: GFR35150_12356 (FR)` and `Release(s) found from MAO`. Items table displays 2 rows with correct itemIds, qty, location, releaseId. |

---

### TC002 — Search with non-existent Order ID

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Order Search |
| **Pre-cond**  | ID does not exist in DB |
| **Steps**     | 1. Enter `INVALID_ORDER_999`. 2. Select `FR`. 3. Click **LOOK FOR THE ORDER**. |
| **Expected**  | Clear error message: "Order not found". No items table displayed. No 500 error. |

---

### TC003 — Search with empty Order ID

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Order Search |
| **Pre-cond**  | User on search page |
| **Steps**     | 1. Leave ID field empty. 2. Select `FR`. 3. Click search or press Enter. |
| **Expected**  | Search button is disabled OR inline validation message shown ("This field is required"). No API call made. |

---

### TC006 — Successful Release Order

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Release Order |
| **Pre-cond**  | Order in CREATED or PENDING status |
| **Steps**     | 1. Load order `GFR35150_12356 (FR)`. 2. Click **RELEASE ORDER**. |
| **Expected**  | Success message displayed. All items status → RELEASED. Release button grays out. |

---

### TC007 — Release an already released order (idempotency)

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Release Order |
| **Pre-cond**  | Order already in RELEASED status |
| **Steps**     | 1. Load already-released order. 2. Observe **RELEASE ORDER** button state. 3. Attempt to click it. |
| **Expected**  | Button is grayed out / disabled. No API call fired. No duplicate release created. |

---

### TC009 — Double-click on Release Order

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Release Order |
| **Pre-cond**  | Order in PENDING status, slow network simulated (Fast 3G) |
| **Steps**     | 1. Click **RELEASE ORDER** rapidly twice in < 300ms. |
| **Expected**  | Only 1 HTTP POST request sent (verify in Network tab). No duplicate release. No error message. |

---

### TC010 — Successful cancellation

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Cancel Order |
| **Pre-cond**  | Order in CREATED or RELEASED status (not shipped) |
| **Steps**     | 1. Load order. 2. Click **CANCEL ORDER**. 3. Confirm if pop-up appears. |
| **Expected**  | Confirmation pop-up shown. Status changes to CANCELLED. All action buttons become unavailable. |

---

### TC011 — Cancel a shipped order

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Cancel Order |
| **Pre-cond**  | All items in SHIPPED status |
| **Steps**     | 1. Load a shipped order. 2. Observe **CANCEL ORDER** button. |
| **Expected**  | Button is hidden or grayed out. Impossible to cancel a fully shipped order. |

---

### TC014 — Ship a released item

| Field         | Value |
|---------------|-------|
| **Priority**  | P0 |
| **Module**    | Ship |
| **Pre-cond**  | Item is in RELEASED status |
| **Steps**     | 1. Load order. 2. Click **SHIP** on item row `3666165426757`. |
| **Expected**  | Row status → SHIPPED. `Shipped: Yes` indicator visible. SHIP button grays out. |

---

### TC021 — Declare item as Short

| Field         | Value |
|---------------|-------|
| **Priority**  | P1 |
| **Module**    | Short |
| **Pre-cond**  | Item in CREATED or RELEASED, not yet shipped |
| **Steps**     | 1. Load order. 2. Click **SHORT** on item `3666165426658`. |
| **Expected**  | Item flagged as SHORT. Available qty → 0. SHIP button disabled for this item. |

---

### TC024 — Partial release (multi-items)

| Field         | Value |
|---------------|-------|
| **Priority**  | P1 |
| **Module**    | Multi-items |
| **Pre-cond**  | Order has 2 items (A and B) in CREATED status |
| **Steps**     | 1. Click **RELEASE** only on Item A row. |
| **Expected**  | Item A → RELEASED. Item B → still CREATED. Global order status → PARTIALLY RELEASED. |

---

## Non-Functional Tests

### TC028 — Mobile Responsive (375px)

| Field         | Value |
|---------------|-------|
| **Priority**  | P2 |
| **Steps**     | Open Chrome DevTools → Device Toolbar → set to 375px. Load the Order Management page. |
| **Expected**  | Table uses horizontal scroll or card view. No button overlaps. All actions reachable. |

### TC030 — Response time SLA

| Field         | Value |
|---------------|-------|
| **Priority**  | P2 |
| **Steps**     | Trigger order search on standard connection. Measure time to first render in Network tab. |
| **Expected**  | Order data displayed in < 3 seconds. |
