
# Test Design Techniques for Web Services Testing Using Python

This study guide covers four critical test design techniques for web services testing using Python and Flask. The topics include **Equivalence Partitioning**, **Boundary Value Analysis**, **State Transition Testing**, and **Decision Table Testing**. Each section includes explanations, examples, code snippets, diagrams, and exercises.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Equivalence Partitioning](#equivalence-partitioning)
   - [Concept and Importance](#concept-and-importance)
   - [Dividing Input Data: Valid and Invalid Groups](#dividing-input-data-valid-and-invalid-groups)
   - [Case Study: Testing API Payloads](#case-study-testing-api-payloads)
   - [Exercises](#exercises-for-equivalence-partitioning)
3. [Boundary Value Analysis](#boundary-value-analysis)
   - [Definition and Importance of Boundary Values](#definition-and-importance-of-boundary-values)
   - [Examples: Determining Edge Cases for API Parameters](#examples-determining-edge-cases-for-api-parameters)
   - [Sample Code and Diagrams](#sample-code-and-diagrams)
   - [Exercises](#exercises-for-boundary-value-analysis)
4. [State Transition Testing](#state-transition-testing)
   - [Concept and Role in Verifying Systems](#concept-and-role-in-verifying-systems)
   - [Designing Tests Based on States and Transitions](#designing-tests-based-on-states-and-transitions)
   - [Flowchart and Sample Test Cases](#flowchart-and-sample-test-cases)
   - [Exercises](#exercises-for-state-transition-testing)
5. [Decision Table Testing](#decision-table-testing)
   - [Definition and Usefulness](#definition-and-usefulness)
   - [Creating Decision Tables for Web Service Scenarios](#creating-decision-tables-for-web-service-scenarios)
   - [Step-by-Step Examples](#step-by-step-examples)
   - [Exercises](#exercises-for-decision-table-testing)
6. [Conclusion](#conclusion)

---

## Introduction

Modern web services often handle complex inputs, multiple states, and numerous conditional rules. Therefore, thorough testing is essential to ensure reliability and security. This guide focuses on four major test design techniques using Python and Flask, complete with practical examples and exercises. By the end of this guide, you should be able to design effective tests for your web service projects.

---

## Equivalence Partitioning

### Concept and Importance

**Equivalence Partitioning** is a black-box testing technique that divides the input domain of a system into classes (or partitions) that are treated similarly by the application. If one test case in a partition passes, the others are likely to pass as well.

- **Why It’s Important:**
  - **Efficiency:** Reduces the total number of test cases by focusing on representative inputs.
  - **Coverage:** Ensures that both valid and invalid input scenarios are considered.
  - **Clarity:** Helps testers understand the behavior of the system by grouping similar inputs.

### Dividing Input Data: Valid and Invalid Groups

When testing a Flask REST API, consider an endpoint that processes user data. For example, imagine an API endpoint that registers a user:

- **Valid Groups:** 
  - A payload with all required fields properly filled.
  - Different valid types for optional fields (e.g., a valid email format, a username of acceptable length).

- **Invalid Groups:**
  - Payloads with missing required fields.
  - Payloads with invalid email formats.
  - Payloads with out-of-bound values (e.g., username too short or too long).

**Example:**

Consider the following Flask endpoint:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    username = data.get('username')
    email = data.get('email')
    
    if not username or not email:
        return jsonify({"error": "Missing required fields"}), 400
    
    if len(username) < 3 or len(username) > 20:
        return jsonify({"error": "Username length must be between 3 and 20 characters"}), 400
    
    if "@" not in email or "." not in email:
        return jsonify({"error": "Invalid email format"}), 400

    # Registration logic here...
    return jsonify({"message": "User registered successfully"}), 201

if __name__ == '__main__':
    app.run(debug=True)
```

**Equivalence Classes for `/register`:**


### Valid Equivalence Class

#### Valid Input
**E1:** A payload where the username is within the valid range (3–20 characters) and the email is in a valid format.  
**Example:**
```json
{
  "username": "john_doe",
  "email": "john@example.com"
}
```

### Invalid Equivalence Classes

#### Missing Required Fields
**E2:** Payload with one or more missing required fields.  
**Example 1 (missing username):**
```json
{
  "email": "john@example.com"
}
```
**Example 2 (missing email):**
```json
{
  "username": "john_doe"
}
```

#### Invalid Username Length
**E3:** Payload where the username is outside the allowed length range (either too short or too long).  
**Example (too short):**
```json
{
  "username": "jo",
  "email": "john@example.com"
}
```
**Example (too long):**
```json
{
  "username": "j" * 25,
  "email": "john@example.com"
}
```

#### Invalid Email Format
**E4:** Payload where the email does not meet the format requirements (e.g., missing "@" or ".").  
**Example:**
```json
{
  "username": "john_doe",
  "email": "johnexample.com"
}
```

### Case Study: Testing API Payloads

**Scenario:** Testing the `/register` endpoint for equivalence classes.

1. **Identify Inputs:** `username` and `email`.
2. **Determine Partitions:**
   - **Username Valid:** Between 3 and 20 characters.
   - **Username Invalid:** Less than 3 or more than 20 characters.
   - **Email Valid:** Contains "@" and a domain.
   - **Email Invalid:** Does not meet the above condition.
3. **Test Cases:** Create test cases for each partition.

### Exercises for Equivalence Partitioning

1. **Exercise 1:**  
   For an API endpoint `/login` that requires a `username` and `password`, define the valid and invalid equivalence classes if:
   - The username must be 5–15 characters long.
   - The password must be at least 8 characters long.
   
2. **Exercise 2:**  
   Create additional test cases for the `/register` endpoint by considering optional fields (e.g., `phone_number`) with a valid pattern.

3. **Exercise 3:**  
   Write a Python script using the Flask test client to automate testing of the `/register` endpoint with payloads representing each equivalence class.

---

## Boundary Value Analysis

### Definition and Importance of Boundary Values

**Boundary Value Analysis (BVA)** focuses on testing the edges of input ranges. Since errors frequently occur at the boundaries of input domains, it is crucial to test the minimum, maximum, just-inside/outside, typical, and error values.

- **Why It’s Critical:**
  - **Edge Case Identification:** Many bugs occur at the boundaries of valid input ranges.
  - **Increased Confidence:** Ensures that the system handles edge cases properly.
  - **Complement to Equivalence Partitioning:** Provides additional assurance that not only typical values are tested but also the limits.

### Examples: Determining Edge Cases for API Parameters

**Scenario:** Testing a login API endpoint where the username must be 5–15 characters.

- **Edge Cases:**
  - **Minimum Valid:** Username of exactly 5 characters.
  - **Just Below Minimum:** Username of 4 characters.
  - **Maximum Valid:** Username of exactly 15 characters.
  - **Just Above Maximum:** Username of 16 characters.
  
For the `/register` endpoint (from the previous section), you would also consider the email format but focus here on the username length.

### Sample Code and Diagrams

**Flask Endpoint Example for Login:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')
    
    if not username or not password:
        return jsonify({"error": "Missing required fields"}), 400
    
    if len(username) < 5 or len(username) > 15:
        return jsonify({"error": "Username must be between 5 and 15 characters"}), 400

    # Login logic here...
    return jsonify({"message": "Login successful"}), 200

if __name__ == '__main__':
    app.run(debug=True)
```

**Diagram: Boundary Values for Username**

```plaintext
        |--- Invalid (4 chars) ---|--- Valid (5 chars) --- ... ---|--- Valid (15 chars) ---|--- Invalid (16 chars) ---|
```

**Sample Python Test Code using Flask's Test Client:**

```python
import unittest
import json
from your_flask_app import app  # Ensure your app is imported properly

class TestLoginBoundaryValues(unittest.TestCase):

    def setUp(self):
        self.client = app.test_client()

    def test_username_min_valid(self):
        payload = {"username": "abcde", "password": "validpassword"}
        response = self.client.post('/login', json=payload)
        self.assertEqual(response.status_code, 200)

    def test_username_below_min(self):
        payload = {"username": "abcd", "password": "validpassword"}
        response = self.client.post('/login', json=payload)
        self.assertEqual(response.status_code, 400)

    def test_username_max_valid(self):
        payload = {"username": "a" * 15, "password": "validpassword"}
        response = self.client.post('/login', json=payload)
        self.assertEqual(response.status_code, 200)

    def test_username_above_max(self):
        payload = {"username": "a" * 16, "password": "validpassword"}
        response = self.client.post('/login', json=payload)
        self.assertEqual(response.status_code, 400)

if __name__ == '__main__':
    unittest.main()
```

### Exercises for Boundary Value Analysis

1. **Exercise 1:**  
   Identify the boundary values for a REST API endpoint that accepts an integer parameter (e.g., `age`) between 18 and 65. List the test cases that should be executed.

2. **Exercise 2:**  
   Modify the `/register` endpoint to include a `phone_number` that must be exactly 10 digits long. Identify and implement test cases for the boundary values.

3. **Exercise 3:**  
   Create a diagram for the boundary values of a field (e.g., `password` length) that requires a minimum of 8 characters and a maximum of 20 characters.

---

## State Transition Testing

### Concept and Role in Verifying Systems

**State Transition Testing** is a method that involves testing the transitions between states of a system. This is especially useful in web services that implement workflows or user session management.

- **Why It’s Important:**
  - **Dynamic Behavior:** Many systems (e.g., authentication, shopping carts) have states that affect functionality.
  - **Comprehensive Coverage:** Testing state transitions helps to catch errors that occur when moving from one state to another.
  - **User Flows:** It validates that the user experience is coherent (e.g., login/logout transitions).

### Designing Tests Based on States and Transitions

**Scenario:** Consider a user authentication system with the following states:
- **Logged Out**
- **Logged In**
- **Locked** (after several failed login attempts)

**Transitions:**
- **From Logged Out to Logged In:** Successful login.
- **From Logged Out to Locked:** Too many failed login attempts.
- **From Logged In to Logged Out:** User logs out.
- **From Locked to Logged Out:** Admin unlocks the account or time expires.

### Flowchart and Sample Test Cases

**Flowchart for a Login/Logout Process:**

```plaintext
             +-----------+
             | Logged Out|
             +-----+-----+
                   |
           (Successful Login)
                   |
             +-----v-----+
             | Logged In |
             +-----+-----+
                   | 
         (User Logs Out)
                   |
             +-----v-----+
             | Logged Out|
                   |
        (Multiple Failed Attempts)
                   |
             +-----v-----+
             |   Locked  |
             +-----------+
```

**Flask Example: User Authentication with State Transitions**

Below is a simplified Flask implementation:

```python
from flask import Flask, request, jsonify, session
from datetime import timedelta

app = Flask(__name__)
app.secret_key = 'secret_key'
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(minutes=30)

# Simulated user database
USERS = {
    "user1": {"password": "password123", "failed_attempts": 0, "locked": False}
}

MAX_FAILED_ATTEMPTS = 3

@app.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')

    user = USERS.get(username)
    if not user:
        return jsonify({"error": "User not found"}), 404

    if user['locked']:
        return jsonify({"error": "Account is locked"}), 403

    if password == user['password']:
        user['failed_attempts'] = 0
        session['username'] = username
        return jsonify({"message": "Login successful"}), 200
    else:
        user['failed_attempts'] += 1
        if user['failed_attempts'] >= MAX_FAILED_ATTEMPTS:
            user['locked'] = True
            return jsonify({"error": "Account locked due to too many failed attempts"}), 403
        return jsonify({"error": "Invalid credentials"}), 401

@app.route('/logout', methods=['POST'])
def logout():
    if 'username' in session:
        session.pop('username')
        return jsonify({"message": "Logged out successfully"}), 200
    return jsonify({"error": "No active session"}), 400

if __name__ == '__main__':
    app.run(debug=True)
```

**Sample Test Cases for State Transitions:**

1. **Test Case 1:**  
   - **Initial State:** Logged Out  
   - **Action:** Successful login  
   - **Expected State:** Logged In  
   - **Test:** POST `/login` with valid credentials.

2. **Test Case 2:**  
   - **Initial State:** Logged Out  
   - **Action:** Three consecutive failed logins  
   - **Expected State:** Locked  
   - **Test:** POST `/login` three times with an invalid password and verify that the account is locked.

3. **Test Case 3:**  
   - **Initial State:** Logged In  
   - **Action:** Logout  
   - **Expected State:** Logged Out  
   - **Test:** POST `/logout` and verify that the session is cleared.

### Exercises for State Transition Testing

1. **Exercise 1:**  
   Draw a detailed state transition diagram for an online shopping cart system (states might include Empty, Active, Checked Out, and Abandoned).

2. **Exercise 2:**  
   Extend the Flask authentication example to include a “password reset” state. Define transitions from “Locked” to “Password Reset” and back to “Logged Out” after a successful reset.

3. **Exercise 3:**  
   Write unit tests using Flask’s test client for the above authentication system to simulate the state transitions.

---

## Decision Table Testing

### Definition and Usefulness

**Decision Table Testing** is a technique used to systematically represent and test combinations of conditions and actions. It is particularly useful when the system has multiple rules and conditions that determine its behavior.

- **Why It’s Useful:**
  - **Clarity:** Helps visualize complex business logic.
  - **Coverage:** Ensures that all possible combinations of inputs (conditions) and corresponding actions are considered.
  - **Maintenance:** Simplifies updates when business rules change.

### Creating Decision Tables for Web Service Scenarios

**Scenario:** Consider an API endpoint `/resource` that enforces access control. The decision to allow access depends on the following conditions:
- **User Role:** `admin`, `user`, `guest`
- **Resource Type:** `public`, `private`
- **Time of Day:** Within working hours (9 AM – 5 PM) or outside

**Decision Table:**

| Condition / Rule            | Rule 1         | Rule 2         | Rule 3         | Rule 4         |
|-----------------------------|----------------|----------------|----------------|----------------|
| **User Role**               | admin          | user           | guest          | user           |
| **Resource Type**           | private        | private        | public         | private        |
| **Within Working Hours?**   | Yes            | Yes            | Yes            | No             |
| **Expected Action**         | Allow access   | Allow access   | Allow access   | Deny access    |

From this decision table, you can derive test cases that cover each rule.

### Step-by-Step Examples

1. **Step 1:** Identify conditions and possible values.
2. **Step 2:** Create the decision table as shown above.
3. **Step 3:** Map each rule to test cases.

**Flask Example:**

```python
from flask import Flask, request, jsonify
from datetime import datetime

app = Flask(__name__)

def is_within_working_hours():
    now = datetime.now().time()
    start = datetime.strptime("09:00", "%H:%M").time()
    end = datetime.strptime("17:00", "%H:%M").time()
    return start <= now <= end

@app.route('/resource', methods=['GET'])
def access_resource():
    user_role = request.args.get('role')
    resource_type = request.args.get('type')
    
    # Conditions based on decision table
    if user_role == "admin" and resource_type == "private":
        return jsonify({"access": "allowed"}), 200
    if user_role == "user" and resource_type == "private" and is_within_working_hours():
        return jsonify({"access": "allowed"}), 200
    if user_role == "guest" and resource_type == "public":
        return jsonify({"access": "allowed"}), 200
    # Additional conditions
    return jsonify({"access": "denied"}), 403

if __name__ == '__main__':
    app.run(debug=True)
```

### Exercises for Decision Table Testing

1. **Exercise 1:**  
   Create a decision table for a payment processing API that considers:
   - Payment method (`credit`, `debit`, `paypal`)
   - Transaction amount (below \$1000, \$1000 and above)
   - User status (`verified`, `unverified`)
   
   Then, derive test cases from your table.

2. **Exercise 2:**  
   Modify the `/resource` endpoint example to include an additional condition (e.g., IP address restrictions). Update your decision table and implement the changes.

3. **Exercise 3:**  
   Write unit tests that iterate through all decision table rules for the `/resource` endpoint to ensure that each combination of conditions is properly handled.

---

## Conclusion

This study guide has explored four critical test design techniques—**Equivalence Partitioning**, **Boundary Value Analysis**, **State Transition Testing**, and **Decision Table Testing**—with a focus on web services testing using Python and Flask. By understanding these techniques and applying them through hands-on examples and exercises, you can design robust test suites for your web applications.

**Next Steps:**
- **Implement:** Use the provided Flask examples and test scripts to build your own test cases.
- **Extend:** Adapt these techniques to more complex scenarios in your projects.
- **Practice:** Complete the exercises to deepen your understanding and practical skills.
