# Global API Test Case Architecture & Compliance Matrix
## Enterprise Cross-Cutting Quality Assurance Standards for HCM Attendance Masters

---

### ðŸŒ 1. Global API Quality & Security Standards Overview

Every Master Template in the Attendance Management Platform (**Status Threshold**, **Week-Off Policy**, **Holiday Template**, **Late/Early Policy**, **Attendance Policy**, and **Attendance Shift Master**) is subjected to **7 Global Cross-Cutting Test Suites**:

```
                                  GLOBAL API TEST ARCHITECTURE
  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
  â”‚ 1. Authentication & RBAC Security (Missing/Expired/Tampered Bearer Tokens)               â”‚
  â”‚ 2. Global Input Fuzzing & SQLi/XSS Defense (Quotes, Whitespace, Script Tags)             â”‚
  â”‚ 3. Pagination & Sort Exception Resilience (Negative Pages, Invalid Column JPA Defense)  â”‚
  â”‚ 4. Single-Active-Default & Exclusivity (Max 1 Active Default, 1 Custom per Employee)     â”‚
  â”‚ 5. Parent-to-Child Status Cascade (Deactivation Propagation to Child Assignments)        â”‚
  â”‚ 6. Entity Lifecycle & 404/409 Resilience (Non-Existent ID 404s, Duplicate Name 409s)     â”‚
  â”‚ 7. Relational & Foreign Key Gating (Inactive Master Binding Defense, Date Chronology)    â”‚
  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

### ðŸ“Š 2. Global Test Cases Matrix Across All 6 Masters

| Global Test Category | Test Case Identifier & Objective | Status Threshold | Week-Off Policy | Holiday Template | Late/Early Policy | Attendance Policy | Attendance Shift |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **1. Auth & Security** | `TC-GLOB-01`: Missing Bearer Token (`401 Unauthorized`) | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS |
| | `TC-GLOB-02`: Expired / Tampered JWT Token (`401/403`) | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS |
| **2. Input Fuzzing** | `TC-GLOB-03`: Quotes-Only String Fuzzing (`""""""`) | ðŸž **BUG 2** | âœ… PASS | âœ… PASS | ðŸž **BUG 6** | âœ… PASS | âœ… PASS |
| | `TC-GLOB-04`: Whitespace-Only Name Fuzzing (`"   "`) | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS |
| | `TC-GLOB-05`: XSS & HTML Tag Injection (`<script>`) | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS |
| **3. Pagination & Sort** | `TC-GLOB-06`: Invalid Column Sort Parameter | ðŸž **BUG 1 (500)** | âœ… PASS | âœ… PASS | ðŸž **BUG 7 (500)**| N/A | âœ… PASS |
| | `TC-GLOB-07`: Negative Page Index (`?page=-1&size=-10`) | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | N/A | âœ… PASS |
| | `TC-GLOB-08`: Out-of-Bounds Page (`?page=999999`) | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | N/A | âœ… PASS |
| **4. Invariants** | `TC-GLOB-09`: Single Active Default Rule (`isDefault=Y`) | âœ… PASS | âœ… PASS | N/A | âœ… PASS | N/A | âœ… PASS |
| | `TC-GLOB-10`: Employee Assignment Exclusivity | âœ… PASS | âœ… PASS | âœ… PASS | âœ… PASS | N/A | âœ… PASS |
| **5. Cascade Sync** | `TC-GLOB-11`: Status Deactivation Parent-Child Cascade | âœ… PASS | âœ… PASS | âœ… PASS | ðŸž **BUG 8 (Desync)**| âœ… PASS | âœ… PASS |
| | `TC-GLOB-12`: Action Enum Parameter Fuzzing | N/A | N/A | N/A | N/A | âœ… PASS | âœ… PASS |
| **6. Entity Resilience** | `TC-GLOB-13`: Non-Existent Entity Lookup (`GET /999999`) | âœ… `404` | âœ… `404` | âœ… `404` | âœ… `404` | âœ… `404` | âœ… `404` |
| | `TC-GLOB-14`: Non-Existent Entity Mutation (`PUT /999999`)| âœ… `404` | âœ… `404` | âœ… `404` | âœ… `404` | âœ… `404` | âœ… `404` |
| | `TC-GLOB-15`: Duplicate Entity Name Collision | âœ… `400` | âœ… `400` | âœ… `400` | âœ… `400` | âœ… `409` | âœ… `400` |
| **7. Relational Gating**| `TC-GLOB-16`: Inactive Foreign Key Status Gating | N/A | N/A | N/A | ðŸž **BUG 9 (OD)** | N/A | N/A |
| | `TC-GLOB-17`: Chronological Date Inversion (`To < From`) | âœ… PASS | ðŸž **BUG 3** | âœ… PASS | âœ… PASS | N/A | âœ… PASS |
| | `TC-GLOB-18`: Duplicate Employee in Assignment Array | ðŸž **BUG 4** | âœ… PASS | âœ… PASS | âœ… PASS | N/A | âœ… PASS |

---

### ðŸ† 3. Defect Discovery from Global Test Suites

Applying these standardized global tests systematically across all masters uncovered **all 9 critical defects**:

1. **`BUG-HCM-ATT-001` (`TC-GLOB-06`):** Status Threshold leaks `AttendanceStatusThresholdMaster` on invalid sort (`500 Server Error`).
2. **`BUG-HCM-ATT-002` (`TC-GLOB-03`):** Status Threshold accepts quotes-only name `""""` (`201 Created`).
3. **`BUG-HCM-ATT-003` (`TC-GLOB-17`):** Week-Off Policy accepts inverted assignment dates (`2029-12-31 > 2029-01-01`).
4. **`BUG-HCM-ATT-004` (`TC-GLOB-18`):** Status Threshold creates duplicate employee rows on repeated employee ID in payload.
5. **`BUG-HCM-ATT-005` (Domain Limit):** Status Threshold accepts `50.00` working hours per day.
6. **`BUG-HCM-ATT-006` (`TC-GLOB-03`):** Late/Early Policy accepts quotes-only name `""""""` (`201 Created`).
7. **`BUG-HCM-ATT-007` (`TC-GLOB-06`):** Late/Early Policy leaks `LateEarlyPolicyMaster` on invalid sort (`500 Server Error`).
8. **`BUG-HCM-ATT-008` (`TC-GLOB-11`):** Late/Early Policy deactivation leaves child assignments active (`isActive = 'Y'`).
9. **`BUG-HCM-ATT-009` (`TC-GLOB-16`):** Late/Early Policy accepts deactivated leave type `OD (ID 7)`, causing broken/blank UI state.