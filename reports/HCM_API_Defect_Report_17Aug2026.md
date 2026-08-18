# HCM Attendance Management — API Defect & Vulnerability Report
**Date:** 17-August-2026  
**Environment:** UAT (`https://uatmcdphcmplatform.omfysgroup.com`)  
**Tested By:** QA API Automation Team  
**Status:** Open / Pending Developer Resolution  

---

## Executive Summary

| Bug ID | Module | Title | Severity | Priority | Status |
|---|---|---|:---:|:---:|:---:|
| **BUG-HCM-ATT-001** | Attendance Status Threshold Master | Unhandled 500 Server Error & Entity Leak on Invalid Sort Query Parameter | Medium | High | **OPEN** |
| **BUG-HCM-ATT-002** | Attendance Status Threshold Master | Missing Input Validation for Special-Character-Only Values in Threshold Name (`""""`) | High | High | **OPEN** |
| **BUG-HCM-ATT-003** | Week Off Policy Master | Missing Chronological Date Order Validation Allows Inverted Dates (`effectiveTo < effectiveFrom`) | High | High | **OPEN** |
| **BUG-HCM-ATT-004** | Attendance Status Threshold Master | Missing Assignment De-Duplication Validation Allows Duplicate Employee Entries (Fan-Out Defect) | High | High | **OPEN** |

---

# Detailed Defect Reports (with Verbatim Raw Request & Response Payloads)

---

## 🐞 BUG 1: Unhandled 500 Internal Server Error & Entity Leak on Invalid Sort Query Parameter

* **Defect ID:** `BUG-HCM-ATT-001`
* **Module:** Attendance Status Threshold Master
* **HTTP Method:** `GET`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/status-threshold/getAll?page=0&size=10&sort=non_existent_column,desc`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```
* **Request Body:**
  ```text
  None (No Body)
  ```

### ❌ Actual Response (HTTP 500 Internal Server Error):
```json
{
  "status": "error",
  "errorCode": "500",
  "message": "No property 'non' found for type 'AttendanceStatusThresholdMaster'",
  "timestamp": "17-Aug-2026 T16:21:09"
}
```

### ✅ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "VALIDATION_ERROR",
  "message": "Invalid sort property 'non_existent_column'. Allowed sort fields: thresholdId, thresholdName, creationDate, effectiveFrom."
}
```

### 🔍 Root Cause & Security Implication:
Spring Data JPA throws an unhandled `PropertyReferenceException`. Instead of returning a clean `400 Bad Request`, it returns `HTTP 500` and leaks the internal backend entity name `"AttendanceStatusThresholdMaster"` (violating OWASP CWE-209).

### 🛠️ Developer Fix:
Add `@ExceptionHandler(PropertyReferenceException.class)` in the `@ControllerAdvice` global exception handler to return `400 Bad Request`.

---

## 🐞 BUG 2: Missing Input Validation for Special-Character-Only Values in Threshold Name

* **Defect ID:** `BUG-HCM-ATT-002`
* **Module:** Attendance Status Threshold Master
* **HTTP Method:** `POST`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/status-threshold/create`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### 📥 Actual Request Body (JSON):
```json
{
  "thresholdCode": "TH_QUOTES_TEST",
  "thresholdName": "\"\"\"\"",
  "description": "Testing if name allows only double quote characters",
  "templateMode": "CUSTOM",
  "shiftTypeApplicability": "Fixed",
  "absentMaxHours": 3.00,
  "halfDayMinHours": 3.01,
  "fullDayMinHours": 6.00,
  "presentMinHours": 8.00,
  "effectiveFrom": "2027-06-01",
  "effectiveTo": "2027-12-31",
  "isDefault": "N",
  "isActive": "Y",
  "templateStatus": "DRAFT",
  "remarks": "Double quote string validation test",
  "assignments": [
    { "employeeId": 3964, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 3585, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 3502, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 3921, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 3504, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 4222, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 3503, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 34, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 4261, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" },
    { "employeeId": 3567, "effectiveFrom": "2027-06-01", "effectiveTo": "2027-12-31", "isActive": "Y" }
  ]
}
```

### ❌ Actual Response Received (HTTP 201 Created — Unexpected Success):
```json
{
  "status": "success",
  "message": "Attendance Status Threshold created successfully.",
  "data": {
    "thresholdId": 80,
    "thresholdCode": "TH00079",
    "thresholdName": "\"\"\"\"",
    "description": "Testing if name allows only double quote characters",
    "templateMode": "CUSTOM",
    "shiftTypeApplicability": "Fixed",
    "absentMaxHours": 3.00,
    "halfDayMinHours": 3.01,
    "fullDayMinHours": 6.00,
    "presentMinHours": 8.00,
    "effectiveFrom": "2027-06-01",
    "effectiveTo": "2027-12-31",
    "isDefault": "N",
    "isActive": "Y",
    "templateStatus": null,
    "remarks": "Double quote string validation test",
    "creationDate": "2026-08-17T09:56:00.230+00:00",
    "lastUpdateDate": "2026-08-17T09:56:00.230+00:00",
    "assignments": [
      {
        "assignmentId": 68,
        "employeeId": 4261,
        "employeeName": "Abhishek Jalkhare",
        "departmentId": 6,
        "departmentName": "Sales & Marketing",
        "designationId": 42,
        "designationName": "Asst.Manager- IT Infrastructure",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 69,
        "employeeId": 3567,
        "employeeName": "Ajay Dhangar",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 1,
        "designationName": "System Engineer-Trainee",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 60,
        "employeeId": 3964,
        "employeeName": "Mahesh More",
        "departmentId": 5,
        "departmentName": "Finance & Accounts",
        "designationId": 7,
        "designationName": "Jr.Software Architect",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 61,
        "employeeId": 3585,
        "employeeName": "Medhaj Wakchaure",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 1,
        "designationName": "System Engineer-Trainee",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 62,
        "employeeId": 3502,
        "employeeName": "Mohamad Siraj",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 1,
        "designationName": "System Engineer-Trainee",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 63,
        "employeeId": 3921,
        "employeeName": "Mrunal Deshpande",
        "departmentId": 4,
        "departmentName": "HR & Administration",
        "designationId": 12,
        "designationName": "Business Analyst",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 64,
        "employeeId": 3504,
        "employeeName": "Neha Patil",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 1,
        "designationName": "System Engineer-Trainee",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 65,
        "employeeId": 4222,
        "employeeName": "Nikhil Chavan",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 3,
        "designationName": "Jr.Software Engineer",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 66,
        "employeeId": 3503,
        "employeeName": "Om Badakh",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 1,
        "designationName": "System Engineer-Trainee",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 67,
        "employeeId": 34,
        "employeeName": "Sachin Khutwad",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 1,
        "designationName": "System Engineer-Trainee",
        "effectiveFrom": "2027-06-01",
        "effectiveTo": "2027-12-31",
        "isActive": "Y"
      }
    ]
  },
  "timestamp": "17-Aug-2026 T15:26:00"
}
```

### ✅ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "VALIDATION_ERROR",
  "message": "Threshold Name must contain meaningful alphanumeric characters and cannot consist solely of special characters or quotes.",
  "data": [
    "thresholdName: Invalid characters or blank name."
  ]
}
```

### 🛠️ Developer Fix:
Add `@Pattern(regexp = "^(?=.*[a-zA-Z0-9])[a-zA-Z0-9\\s\\-_/().]+$", message = "Threshold Name must contain at least one alphanumeric character")` on `thresholdName` in `AttendanceStatusThresholdRequestDTO`.

---

## 🐞 BUG 3: Missing Chronological Date Order Validation on Week Off Employee Assignments

* **Defect ID:** `BUG-HCM-ATT-003`
* **Module:** Week Off Policy Master
* **HTTP Method:** `POST`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/week-offs/create`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### 📥 Actual Request Body (JSON):
```json
{
  "weekOffName": "Bug Test Inverted Dates Policy 02",
  "description": "Testing inverted chronological date validation",
  "optionModel": "ALLOWED",
  "isDefault": "N",
  "isActive": "Y",
  "details": [
    {
      "dayOfWeek": "SUNDAY",
      "weekNo": 1,
      "isWeekOff": "Y"
    }
  ],
  "assignments": [
    {
      "employeeId": 3502,
      "departmentId": 1,
      "locationId": 1,
      "effectiveFrom": "2029-12-31",
      "effectiveTo": "2029-01-01",
      "isActive": "Y"
    }
  ]
}
```

### ❌ Actual Response Received (HTTP 200/201 OK — Unexpected Success):
```json
{
  "status": "success",
  "message": "Week Off Template created successfully.",
  "data": {
    "weekOffId": 134,
    "weekOffCode": "LEP00136",
    "weekOffName": "Bug Test Inverted Dates Policy 02",
    "days": null,
    "description": "Testing inverted chronological date validation",
    "optionModel": "ALLOWED",
    "isDefault": "N",
    "isActive": "Y",
    "remarks": null,
    "details": [
      {
        "detailId": 671,
        "dayOfWeek": "SUNDAY",
        "weekNo": 1,
        "isWeekOff": "Y"
      }
    ],
    "assignments": [
      {
        "isActive": "Y",
        "designationId": 1,
        "departmentId": 1,
        "effectiveFrom": "2029-12-31",
        "effectiveTo": "2029-01-01",
        "assignmentId": 2372,
        "employeeId": 3502,
        "departmentName": "Software Development & Delivery ",
        "designationName": "System Engineer-Trainee",
        "employeeName": "Mohamad Siraj"
      }
    ],
    "employeeCount": 1,
    "creationDate": "17-Aug-2026",
    "lastUpdateDate": "17-Aug-2026"
  },
  "timestamp": "17-Aug-2026 T16:42:16"
}
```

### ✅ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "VALIDATION_ERROR",
  "message": "effectiveTo date (2029-01-01) cannot be chronologically earlier than effectiveFrom date (2029-12-31)."
}
```

### 🛠️ Developer Fix:
In `WeekOffService.java`, add validation before persisting:
```java
if (assignment.getEffectiveTo() != null && assignment.getEffectiveTo().isBefore(assignment.getEffectiveFrom())) {
    throw new BadRequestException("effectiveTo date (" + assignment.getEffectiveTo() + ") cannot be earlier than effectiveFrom date (" + assignment.getEffectiveFrom() + ")");
}
```

---

## 🐞 BUG 4: Missing Assignment De-Duplication Validation Allows Duplicate Employee Entries (Calculation Fan-Out Defect)

* **Defect ID:** `BUG-HCM-ATT-004`
* **Module:** Attendance Status Threshold Master
* **HTTP Method:** `POST`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/status-threshold/create`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### 📥 Actual Request Body (JSON):
```json
{
  "thresholdCode": "TH_DUP_REPRODUCE",
  "thresholdName": "Duplicate Employee Test 2035",
  "description": "Testing duplicate employee in same assignment payload",
  "templateMode": "CUSTOM",
  "shiftTypeApplicability": "Fixed",
  "absentMaxHours": 3.00,
  "halfDayMinHours": 3.01,
  "fullDayMinHours": 6.00,
  "presentMinHours": 8.00,
  "effectiveFrom": "2035-01-01",
  "effectiveTo": "2035-12-31",
  "isDefault": "N",
  "isActive": "Y",
  "templateStatus": "DRAFT",
  "remarks": "Testing duplicate employee bug",
  "assignments": [
    {
      "employeeId": 3701,
      "effectiveFrom": "2035-01-01",
      "effectiveTo": "2035-12-31",
      "isActive": "Y"
    },
    {
      "employeeId": 3701,
      "effectiveFrom": "2035-01-01",
      "effectiveTo": "2035-12-31",
      "isActive": "Y"
    }
  ]
}
```

### ❌ Actual Response Received (HTTP 201 Created — Unexpected Success):
```json
{
  "status": "success",
  "message": "Attendance Status Threshold created successfully.",
  "data": {
    "thresholdId": 96,
    "thresholdCode": "TH00095",
    "thresholdName": "Duplicate Employee Test 2035",
    "description": "Testing duplicate employee in same assignment payload",
    "templateMode": "CUSTOM",
    "shiftTypeApplicability": "Fixed",
    "absentMaxHours": 3.00,
    "halfDayMinHours": 3.01,
    "fullDayMinHours": 6.00,
    "presentMinHours": 8.00,
    "effectiveFrom": "2035-01-01",
    "effectiveTo": "2035-12-31",
    "isDefault": "N",
    "isActive": "Y",
    "templateStatus": null,
    "remarks": "Testing duplicate employee bug",
    "creationDate": "2026-08-17T12:45:43.393+00:00",
    "lastUpdateDate": "2026-08-17T12:45:43.393+00:00",
    "assignments": [
      {
        "assignmentId": 80,
        "employeeId": 3701,
        "employeeName": "Gautam Gehani",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 86,
        "designationName": "Assistant Manager-Business Analysis & Delivery",
        "effectiveFrom": "2035-01-01",
        "effectiveTo": "2035-12-31",
        "isActive": "Y"
      },
      {
        "assignmentId": 79,
        "employeeId": 3701,
        "employeeName": "Gautam Gehani",
        "departmentId": 1,
        "departmentName": "Software Development & Delivery ",
        "designationId": 86,
        "designationName": "Assistant Manager-Business Analysis & Delivery",
        "effectiveFrom": "2035-01-01",
        "effectiveTo": "2035-12-31",
        "isActive": "Y"
      }
    ]
  },
  "timestamp": "17-Aug-2026 T18:15:43"
}
```

### ✅ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "VALIDATION_ERROR",
  "message": "Duplicate employee assignments detected. Employee ID 3701 appears more than once in the assignment list."
}
```

### 🛠️ Developer Fix:
In `AttendanceStatusThresholdService.java`, validate uniqueness of employee IDs in the assignment list:
```java
Set<Long> uniqueEmpIds = new HashSet<>();
for (ThresholdAssignmentDTO dto : request.getAssignments()) {
    if (!uniqueEmpIds.add(dto.getEmployeeId())) {
        throw new BadRequestException("Duplicate employee assignment for employeeId: " + dto.getEmployeeId());
    }
}
```

---

## ðŸž BUG 5: Missing 24-Hour Upper Bound Limit Validation on Threshold Working Hours

* **Defect ID:** `BUG-HCM-ATT-005`
* **Module:** Attendance Status Threshold Master
* **HTTP Method:** `POST`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/status-threshold/create`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### ðŸ“¤ Actual Request Body (JSON):
```json
{
  "thresholdCode": "TH_EXT_HRS_01",
  "thresholdName": "Extreme Boundary Hours 2028",
  "description": "Testing hours exceeding 24h limit",
  "templateMode": "CUSTOM",
  "shiftTypeApplicability": "Fixed",
  "absentMaxHours": 25.00,
  "halfDayMinHours": 30.00,
  "fullDayMinHours": 40.00,
  "presentMinHours": 50.00,
  "effectiveFrom": "2028-01-01",
  "effectiveTo": "2028-12-31",
  "isDefault": "N",
  "isActive": "Y",
  "templateStatus": "DRAFT",
  "remarks": "Extreme hours limit test",
  "assignments": [
    {
      "employeeId": 34,
      "effectiveFrom": "2028-01-01",
      "effectiveTo": "2028-12-31",
      "isActive": "Y"
    }
  ]
}
```

### âŒ Actual Response Received (HTTP 201 Created â€” Unexpected Success):
```json
{
  "status": "success",
  "message": "Attendance Status Threshold created successfully.",
  "data": {
    "thresholdId": 106,
    "thresholdCode": "TH00105",
    "thresholdName": "Extreme Boundary Hours 2028",
    "absentMaxHours": 25.00,
    "halfDayMinHours": 30.00,
    "fullDayMinHours": 40.00,
    "presentMinHours": 50.00,
    "isActive": "Y"
  },
  "timestamp": "18-Aug-2026 T13:18:38"
}
```

### âœ”ï¸ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "VALIDATION_ERROR",
  "message": "Working threshold hours cannot exceed 24.00 hours per day (presentMinHours: 50.00 is invalid)."
}
```

### ðŸ”§ Developer Fix:
In `AttendanceStatusThresholdRequestDTO.java`, add maximum 24.00 hours constraint validation on all numeric hour properties:
```java
@DecimalMin(value = "0.00", message = "Hours must be greater than or equal to 0")
@DecimalMax(value = "24.00", message = "Hours cannot exceed 24.00 hours per day")
private BigDecimal presentMinHours;
```

---

## ðŸž BUG 6: Missing Input Validation for Special-Character-Only Values in Policy Name (Accepted Quotes-Only Name)

* **Defect ID:** `BUG-HCM-ATT-006`
* **Module:** Late / Early Policy Master
* **HTTP Method:** `POST`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/late-early-policies/create`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### ðŸ“¤ Actual Request Body (JSON):
```json
{
  "policyCode": null,
  "policyName": "\"\"\"\"\"\"",
  "description": "Testing quotes-only policy name fuzzing acceptance",
  "templateMode": "CUSTOM",
  "eventCountMinutes": 30,
  "graceMinutes": 10,
  "graceEvent": 2,
  "allowedEvent": 3,
  "deductionType": "LEAVE",
  "leaveDeductDays": 0.50,
  "leaveTypeId": 1,
  "effectiveFrom": "2027-01-01",
  "effectiveTo": "2027-12-31",
  "isDefault": "N",
  "isActive": "Y",
  "assignments": [
    {
      "employeeId": 3503,
      "effectiveFrom": "2027-01-01",
      "effectiveTo": "2027-12-31",
      "isActive": "Y"
    }
  ]
}
```

### âŒ Actual Response Received (HTTP 201 Created â€” Unexpected Success):
```json
{
  "status": "success",
  "message": "Late/Early Policy created successfully.",
  "data": {
    "policyId": 154,
    "policyCode": "LEP00153",
    "policyName": "\"\"\"\"\"\"",
    "description": "Testing quotes-only policy name fuzzing acceptance",
    "templateMode": "CUSTOM",
    "eventCountMinutes": 30,
    "graceMinutes": 10,
    "graceEvent": 2,
    "allowedEvent": 3,
    "deductionType": "LEAVE",
    "leaveDeductDays": 0.50,
    "leaveTypeId": 1,
    "leaveTypeName": "CL",
    "effectiveFrom": "2027-01-01T00:00:00.000+00:00",
    "effectiveTo": "2027-12-31T00:00:00.000+00:00",
    "isDefault": "N",
    "isActive": "Y"
  },
  "timestamp": "18-Aug-2026 T14:51:00"
}
```

### âœ”ï¸ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "VALIDATION_ERROR",
  "message": "Policy name must contain valid alphanumeric characters."
}
```

### ðŸ”§ Developer Fix:
In `LateEarlyPolicyRequestDto.java`, add `@NotBlank` and regex pattern constraint validation on `policyName`:
```java
@NotBlank(message = "Policy Name is required")
@Pattern(
    regexp = "^(?=.*[a-zA-Z0-9])[a-zA-Z0-9\\s\\-_/().]+$", 
    message = "Policy Name must contain alphanumeric characters and cannot consist solely of whitespace or special symbols"
)
private String policyName;
```

---

## ðŸž BUG 7: Unhandled 500 Internal Server Error & JPA Entity Leak on Invalid Sort Query Parameter in Late/Early Policy

* **Defect ID:** `BUG-HCM-ATT-007`
* **Module:** Late / Early Policy Master
* **HTTP Method:** `GET`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/late-early-policies/getAll?page=0&size=10&sort=non_existent_column,desc`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```
* **Request Body:** *None (GET request)*

### âŒ Actual Response Received (HTTP 500 Internal Server Error â€” Unhandled Exception & Entity Leak):
```json
{
  "status": "error",
  "errorCode": "500",
  "message": "No property 'non' found for type 'LateEarlyPolicyMaster'",
  "timestamp": "18-Aug-2026 T14:59:09"
}
```

### âœ”ï¸ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "400",
  "message": "Invalid sort property: 'non_existent_column'",
  "timestamp": "18-Aug-2026 T14:59:09"
}
```

### ðŸ”§ Developer Fix:
In global `@ControllerAdvice` (`GlobalExceptionHandler.java`), add an exception handler for Spring Data's `PropertyReferenceException` to sanitize the error and return HTTP `400 Bad Request`:
```java
@ExceptionHandler(PropertyReferenceException.class)
public ResponseEntity<ErrorResponseDto> handlePropertyReferenceException(PropertyReferenceException ex) {
    ErrorResponseDto error = new ErrorResponseDto(
        "error", 
        "400", 
        "Invalid sort property: '" + ex.getPropertyName() + "'"
    );
    return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
}
```

---

## ðŸž BUG 8: Missing Status Cascade on Policy Deactivation Leaves Child Employee Assignments Active (Cascade Desynchronization)

* **Defect ID:** `BUG-HCM-ATT-008`
* **Module:** Late / Early Policy Master
* **HTTP Method:** `PATCH` & `GET`
* **Request URLs:**
  * **Step A (Deactivate):** `PATCH https://uatmcdphcmplatform.omfysgroup.com/api/attendance/late-early-policies/138/status?status=N`
  * **Step B (Inspect):** `GET https://uatmcdphcmplatform.omfysgroup.com/api/attendance/late-early-policies/getById/138`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### ðŸ“‹ Business & Architectural Impact:
When an administrator deactivates a Late/Early Policy (`PATCH status?status=N`), the backend only updates the parent policy record to `isActive = 'N'` and fails to cascade the deactivation to child employee assignments and deduction priorities. As a result, employee assignments remain active (`'Y'`) under an inactive policy, causing automated payroll and attendance deduction batch jobs to incorrectly penalize employees under a policy that HR had disabled.

### âŒ Actual Response Received (HTTP 200 OK â€” Desynchronized Child Assignment State):
```json
{
  "status": "success",
  "message": "Late/Early Policy fetched successfully.",
  "data": {
    "policyId": 138,
    "policyCode": "LEP00137",
    "policyName": "Standard Late Early Policy 6530",
    "isActive": "N",
    "assignments": [
      {
        "assignmentId": 1498,
        "employeeId": 34,
        "employeeName": "Sachin Khutwad",
        "isActive": "Y"
      }
    ]
  },
  "timestamp": "18-Aug-2026 T15:38:12"
}
```

### âœ”ï¸ Expected Response (HTTP 200 OK â€” Cascaded Deactivation to Child Assignments):
```json
{
  "status": "success",
  "message": "Late/Early Policy fetched successfully.",
  "data": {
    "policyId": 138,
    "policyCode": "LEP00137",
    "policyName": "Standard Late Early Policy 6530",
    "isActive": "N",
    "assignments": [
      {
        "assignmentId": 1498,
        "employeeId": 34,
        "employeeName": "Sachin Khutwad",
        "isActive": "N"
      }
    ]
  },
  "timestamp": "18-Aug-2026 T15:38:12"
}
```

### ðŸ”§ Developer Fix:
In `LateEarlyPolicyServiceImpl.java` inside `changeStatus(Long policyId, String status)`:
```java
// 1. Update parent entity status
policy.setIsActive(status);
lateEarlyPolicyRepository.save(policy);

// 2. Cascade status update to all child assignments and deduction priorities
assignmentRepository.updateStatusByPolicyId(policyId, status);
deductionPriorityRepository.updateStatusByPolicyId(policyId, status);
```

---

## ðŸž BUG 9: Missing Foreign Active-State Validation on Leave Type Allows Creation of Corrupted Policies Linked to Deactivated Leaves

* **Defect ID:** `BUG-HCM-ATT-009`
* **Module:** Late / Early Policy Master
* **HTTP Method:** `POST`
* **Request URL:** `https://uatmcdphcmplatform.omfysgroup.com/api/attendance/late-early-policies/create`
* **Request Headers:**
  ```http
  Authorization: Bearer {{authToken}}
  Content-Type: application/json
  ```

### ðŸ§­ UI Navigation Path to Observe Corrupted State:
1. Navigate to **Attendance Management** $\rightarrow$ **Late / Early Policy** list table.
2. In the search box, search for Template Code: **`LEP00175`** (or `Deactivated Leave Gating Test 2038`).
3. Click the **Edit (âœï¸ icon)** button to open the *Edit Late/Early Policy Template* modal.
4. Observe the **"Leave Type for Deduction *"** field: It renders completely **BLANK (`Select leave type`)** because the frontend dropdown hides inactive leaves (`OD`), failing to bind the backend's `leaveTypeId: 7`.
5. Attempting to click **"Update"** blocks the user with a mandatory field error, making the policy permanently un-editable from the web UI.

### ðŸ“‹ Technical & Architectural Root Cause:
In the Leave Master API (`/admin/leaves/getleavestypes`), Leave Type ID 7 (`OD`) has `is_active = "N"`. However, the Late/Early Policy creation endpoint only verifies if `leaveTypeId` exists in the database table and fails to check if `is_active == 'Y'`. Consequently, the API allows creating an active policy bound to a disabled leave type, desynchronizing the frontend dropdown and crashing payroll deduction cron jobs.

### ðŸ“¤ Actual Request Body (JSON):
```json
{
  "policyCode": null,
  "policyName": "Deactivated Leave Gating Test 2038",
  "description": "Testing foreign active-state gating on leaveTypeId: 7 (OD)",
  "templateMode": "CUSTOM",
  "eventCountMinutes": 30,
  "graceMinutes": 10,
  "graceEvent": 2,
  "allowedEvent": 3,
  "deductionType": "LEAVE",
  "leaveDeductDays": 0.50,
  "leaveTypeId": 7,
  "effectiveFrom": "2038-01-01",
  "effectiveTo": "2038-12-31",
  "isDefault": "N",
  "isActive": "Y",
  "assignments": [
    {
      "employeeId": 4223,
      "effectiveFrom": "2038-01-01",
      "effectiveTo": "2038-12-31",
      "isActive": "Y"
    }
  ]
}
```

### âŒ Actual Response Received (HTTP 201 Created â€” Unexpected Success):
```json
{
  "status": "success",
  "message": "Late/Early Policy created successfully.",
  "data": {
    "policyId": 175,
    "policyCode": "LEP00175",
    "policyName": "Deactivated Leave Gating Test 2038",
    "description": "Testing foreign active-state gating on leaveTypeId: 7 (OD)",
    "templateMode": "CUSTOM",
    "templateStatus": null,
    "eventCountMinutes": 30,
    "graceMinutes": 10,
    "graceEvent": 2,
    "allowedEvent": 3,
    "deductionType": "LEAVE",
    "leaveDeductDays": 0.50,
    "leaveTypeId": 7,
    "leaveTypeCode": null,
    "leaveTypeName": "OD",
    "fineAmount": null,
    "effectiveFrom": "2038-01-01T00:00:00.000+00:00",
    "effectiveTo": "2038-12-31T00:00:00.000+00:00",
    "isDefault": "N",
    "isActive": "Y"
  },
  "timestamp": "18-Aug-2026 T15:54:04"
}
```

### âœ”ï¸ Expected Response (HTTP 400 Bad Request):
```json
{
  "status": "error",
  "errorCode": "400",
  "message": "Leave Type 'OD' (ID 7) is deactivated and cannot be selected for late/early deduction rules.",
  "timestamp": "18-Aug-2026 T15:54:04"
}
```

### ðŸ”§ Developer Fix:
In `LateEarlyPolicyServiceImpl.java`:
```java
LeaveTypeMaster leaveType = leaveTypeRepository.findById(requestDto.getLeaveTypeId())
    .orElseThrow(() -> new ResourceNotFoundException("Leave Type not found"));

if (!"Y".equalsIgnoreCase(leaveType.getIsActive())) {
    throw new BadRequestException("Leave Type '" + leaveType.getLeaveTypeName() + "' is inactive and cannot be assigned.");
}
```
