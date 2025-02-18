## 1. Introduction to White Box Testing

**White Box Testing** involves designing and executing tests based on the **internal structure** of the code. Unlike black box testing (where you only look at inputs and outputs), white box testing requires knowledge of:

1. **Control Structures** (e.g., loops, if/else)  
2. **Variable Usage** (definitions, lifetimes)  
3. **Internal Logic and Flow**  

By systematically covering **all** code paths (control flow) and checking **proper variable usage** (data flow), you can catch **subtle or hidden errors** that might escape black box tests.

---

## 2. Control Flow Testing: The Broader Strategy

**Control Flow Testing** is a white box approach that analyzes *how code executes* from one statement or block to the next. You typically:

1. Construct or imagine a **Control Flow Graph (CFG)**, where:  
   - **Nodes** are chunks of code (or decision points).  
   - **Edges** show how execution can move from one node to another.

2. Use **coverage metrics** to ensure you have exercised the necessary branches and paths in that graph.

### 2.1 Coverage Metrics under Control Flow Testing

Below are the **four major coverage metrics** under Control Flow Testing, plus the concept of **cyclomatic complexity**.

#### 2.1.1 Statement Coverage

- **Definition**: Measures the percentage of **executable statements** that are run at least once during testing.  
- **Pros**: Simple to achieve; ensures no line is completely skipped.  
- **Cons**: Can miss untested branches. 100% statement coverage does **not** necessarily mean every decision has been tested in every possible way.

#### 2.1.2 Branch Coverage (Decision Coverage)

- **Definition**: Ensures each **branch** in the code (e.g., `if`, `else`, `elif`) is executed at least once.  
- **Pros**: More rigorous than statement coverage; reveals if an `if` condition was never tested as false (or true).  
- **Cons**: Still does *not* guarantee that **combinations** of decisions (nested or sequential) are covered.

#### 2.1.3 Condition Coverage

- **Definition**: Ensures each **Boolean sub-condition** within a larger condition is evaluated to both true and false.  
- **Example**: In `if (x > 0 and y < 10)`, you ensure `(x > 0)` is tested for both outcomes (true/false), and `(y < 10)` is also tested for both outcomes—possibly requiring multiple input sets.  
- **Pros**: More fine-grained than branch coverage.  
- **Cons**: Can require additional test cases for all sub-conditions. Doesn’t necessarily ensure all **paths** of execution are covered.

#### 2.1.4 Path Coverage

- **Definition**: Aims to test **every possible route** (sequence of branches/decisions) through the program.  
- **Pros**: The most exhaustive in terms of control flow.  
- **Cons**: Potentially explodes in complexity for real-world software with many nested decisions or loops.

### 2.2 Cyclomatic Complexity

- **Definition**: A metric that indicates how many **independent paths** exist in your code. Calculated as:
  \[
    M = E - N + 2
  \]
  where \( E \) is the number of edges in the control flow graph, \( N \) is the number of nodes, and \( M \) is the complexity number.
- **Why It Matters**: 
  - A **higher** complexity means more branching and, by extension, more test cases needed for thorough coverage.  
  - Encourages developers to **refactor** if complexity is too high.

#### 2.2.1 Merging Return Statements

- In many functions, you have multiple return points. For cyclomatic complexity, it’s standard to **merge** all returns into a single “End” node so you don’t artificially inflate the node/edge count.  
- The real complexity is driven by **decision structures** (if/else/loops), not the number of returns.

---

## 3. Data Flow Testing: Complementary to Control Flow

**Data Flow Testing** looks at **how data (variables)** move through the program. Instead of focusing on which branches are taken, data flow testing tracks:

1. **Definition**: Where a variable is first assigned or receives a value.  
2. **Usage**: Where that variable is read or used in an expression.  
3. **Kill**: Where a variable goes out of scope or is overwritten.

### 3.1 Common Anomalies Data Flow Testing Uncovers

1. **Use Before Definition** (a variable is read or used before having an assigned value).  
2. **Redefinition Without Usage** (a variable’s value is overwritten before it’s ever used, indicating a potential logical mistake).  
3. **Never Used** (a variable is defined or assigned but never actually used, indicating dead or vestigial code).

### 3.2 Data Flow Graph vs. Control Flow Graph

- **Data Flow Graph**: Emphasizes the *life cycle of variables* across nodes.  
- **Control Flow Graph**: Emphasizes *branches and decisions*.  
- The two overlap but highlight different aspects. Control flow is about *logic paths*, data flow is about *variable usage patterns*.

---

## 4. Our Use Case: `categorize_age`

To illustrate **both** control flow and data flow testing, consider the following **simple Python function**:

```python
def categorize_age(age):
    if age < 0:
        return "Invalid"
    elif age < 18:
        return "Minor"
    elif age < 65:
        return "Adult"
    else:
        return "Senior"
```

### 4.1 Control Flow Graph (CFG)

Imagine the **CFG** with a **Start** node, a sequence of 3 decisions (`if age<0`, `elif age<18`, `elif age<65`), and an **End** node that unifies all return statements:

```
  ┌─────────────┐
  │   (Start)   │
  └─────────────┘
         ▼
   ┌─────────────┐
   │ age < 0 ?   │
   └─────────────┘
       /   \
      /True \False
      ▼      ▼
 (Return)   ┌─────────────┐
            │ age < 18 ?  │
            └─────────────┘
               /   \
              /True \False
              ▼      ▼
          (Return) ┌─────────────┐
                    │ age < 65 ? │
                    └─────────────┘
                       /   \
                      /True \False
                      ▼      ▼
                   (Return) (Return)

```

(All returns collapse into a single conceptual “End” in terms of cyclomatic complexity.)

### 4.2 Applying Control Flow Testing

1. **Statement Coverage**  
   - We need to run **every line**. The function has four return statements.  
   - If we test `age = -1, 10, 40, 70`, we’ll execute all statements (`return "Invalid"`, `return "Minor"`, `return "Adult"`, `return "Senior"`).

2. **Branch Coverage**  
   - Each decision (`if age<0`, `elif age<18`, `elif age<65`) must be tested both ways (true/false).  
   - For instance:  
     - `age=-1`: `age<0` = true (hit `"Invalid"`).  
     - `age=10`: first check is false, second check (`age<18`) = true (`"Minor"`).  
     - `age=40`: first check false, second check false, third check (`age<65`) = true (`"Adult"`).  
     - `age=70`: eventually leads to the `else: return "Senior"` (third check is false).  
   - These four tests ensure all branches are taken at least once.

3. **Condition Coverage**  
   - Each condition is a single sub-expression (`age < 0`, `age < 18`, `age < 65`). We test them both as true and false in our set of test inputs.  
   - For example, `age < 0` is true for -1 and false for 10, 40, 70.

4. **Path Coverage**  
   - The function effectively has 4 final paths. By having an input that results in each path, we achieve 100% path coverage in this simple chain.

5. **Cyclomatic Complexity**  
   - Label nodes as: Start, Decision1 (`age<0?`), Decision2 (`age<18?`), Decision3 (`age<65?`), End.  
   - Typically you’d get \(N=5\), \(E=7\). Then \(M = E - N + 2 = 7 - 5 + 2 = 4\).  
   - Interpreted as 4 independent paths, aligning with the 4 outcomes.

### 4.3 Applying Data Flow Testing

- **Variables**: We have just one main variable, `age`.  
- **Definition**: `age` is defined when the function parameter is received.  
- **Usage**: `age` is used in each comparison.  
- **No Re-definitions**: We never assign `age` a new value.  
- **No Extra Variables**: No risk of “never used” or “use before def.”  

**Conclusion**: Data flow testing for such a simple function doesn’t reveal anomalies. In more complex code (with additional variables, parameters, or state management), data flow testing would be far more critical for spotting subtle errors.

---

## 5. Putting It All Together: Sample `pytest` Suite

A minimal `pytest` suite showing how to test for coverage:

```python
import pytest
from my_module import categorize_age

def test_categorize_age_invalid():
    assert categorize_age(-1) == "Invalid"

def test_categorize_age_minor():
    assert categorize_age(10) == "Minor"

def test_categorize_age_adult():
    assert categorize_age(40) == "Adult"

def test_categorize_age_senior():
    assert categorize_age(70) == "Senior"
```

Running:

```bash
pytest --cov=my_module --cov-report=term-missing
```

- Likely yields **100% statement** and **branch** coverage for `categorize_age`.
- Condition coverage also is effectively satisfied, as is path coverage (given each outcome is tested).

---

## 6. Key Takeaways

1. **Control Flow Testing**:  
   - **Statement** and **Branch Coverage** are the most common.  
   - More advanced metrics (Condition, Path) may be used in especially critical or complex code.  
   - **Cyclomatic Complexity** guides how many tests you might need if you want full path coverage.

2. **Data Flow Testing**:  
   - Focuses on **variables’ definitions and uses**.  
   - Helps catch uninitialized variables, unused variables, or redefinitions that overwrite data prematurely.  
   - Crucial in large, multi-function codebases with shared or global state.

3. **Integration**:  
   - **Both** approaches can be used together to ensure you test **all branches** and **handle variables correctly**.  
   - A robust test suite addresses both **logic paths** (control flow) and **variable correctness** (data flow).

4. **Simplicity vs. Complexity**:  
   - Our `categorize_age` example is straightforward—just one variable and a few branches. In **real-world** apps (with multiple variables, nested conditions, multi-level function calls), both control flow and data flow testing become far more valuable.

---

## 7. Conclusion

- **Control Flow Testing** covers **how the code flows** through decisions, focusing on coverage metrics like statement, branch, condition, and path.  
- **Cyclomatic Complexity** helps quantify how “branchy” your function is and, therefore, how many tests might be needed for thorough coverage.  
- **Data Flow Testing** zeroes in on **variable usage**, catching anomalies like “use before definition” or “never used” variables.  
- Combining these **white box testing** strategies results in a comprehensive test approach, ensuring both **logic correctness** (control flow) and **correct handling of variables** (data flow), ultimately improving overall software reliability.
