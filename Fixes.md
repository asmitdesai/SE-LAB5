# **Static Analysis Report and Fix Explanation**

**File:** inventory\_system.py

This document details the issues found in inventory\_system.py by Pylint, Bandit, and Flake8. It explains why each was a problem and how it was remediated.

## **1\. Security Vulnerabilities**

### **Issue: Use of eval()**

* **What:** The line eval("print('eval used')") was present in the main function.  
* **Why it was found:** Flagged by **Bandit (B307)** and **Pylint (W0123)** as a critical security risk. The eval() function can execute any text string as Python code. If that string ever came from user input, it would allow a malicious user to run any code they want (e.g., delete files, steal data).  
* **How it was fixed:** The entire line was **deleted**. It was a dangerous example and was not essential to the program's logic.

## **2\. Bugs and Logical Errors**

### **Issue: Dangerous Default Value (logs=\[\])**

* **What:** The addItem function signature was def addItem(..., logs=\[\]).  
* **Why it was found:** Flagged by **Pylint (W0102)**. In Python, default arguments are created *once* when the function is first defined. This meant all calls to addItem that didn't provide their own list would **share the exact same list**. This would cause logs to accumulate incorrectly across separate calls.  
* **How it was fixed:**  
  1. The function signature was changed to def add\_item(..., logs=None). None is an immutable default.  
  2. Inside the function, if logs is None: logs \= \[\] was added. This creates a new, fresh list *every time* the function is called, which is the correct behavior.

### **Issue: Bare except**

* **What:** The removeItem function used a generic try...except: block.  
* **Why it was found:** Flagged by **Pylint (W0702)**, **Flake8 (E722)**, and **Bandit (B110)**. A bare except: "swallows" all errors, including system-exiting ones (like KeyboardInterrupt) or unexpected logic errors (like TypeError). This hides bugs and makes the code extremely difficult to debug.  
* **How it was fixed:** The except: was replaced with except KeyError:. This specifically catches the *one* error we expect to happen (trying to remove an item that isn't in the dictionary) while letting all other unexpected errors crash the program, making them visible.

## **3\. Resource Management**

### **Issue: Missing with for File Operations**

* **What:** The loadData and saveData functions used f \= open(...) and then manually called f.close().  
* **Why it was found:** Flagged by **Pylint (R1732)** (consider-using-with). If an error occurred after open() but before f.close() (e.g., a json.loads error), the program would crash and the file would be left open, leaking system resources.  
* **How it was fixed:** Both functions were refactored to use the with open(...) as f: syntax. This construct guarantees that the file is automatically and safely closed the moment the block is exited, even if an error occurs.

### **Issue: Unspecified File Encoding**

* **What:** The open() calls in loadData and saveData did not specify an encoding.  
* **Why it was found:** Flagged by **Pylint (W1514)**. This causes the code to rely on the system's default encoding, which can vary between machines (e.g., UTF-8 on a Mac, cp1252 on a Windows machine). This can lead to crashes or data corruption.  
* **How it was fixed:** Explicitly added encoding="utf-8" to all open() calls. UTF-8 is a universal standard and ensures the code behaves consistently everywhere.

## **4\. Code Quality and Style (PEP 8\)**

### **Issue: Unused Import (logging)**

* **What:** The logging module was imported at the top of the file but was never used.  
* **Why it was found:** Flagged by **Pylint (W0611)** and **Flake8 (F401)**. Unused imports are dead code that clutter the namespace.  
* **How it was fixed:** The line import logging was **deleted**.

### **Issue: Missing Docstrings**

* **What:** The module itself and every function (e.g., addItem, removeItem) were missing documentation strings (docstrings).  
* **Why it was found:** Flagged by **Pylint (C0114, C0116)**. Code without documentation is difficult to understand, use, and maintain.  
* **How it was fixed:** A module-level docstring was added at the top of the file, and a simple docstring (e.g., """Adds an item...""") was added inside every function to explain its purpose.

### **Issue: Invalid Naming Convention**

* **What:** Functions used camelCase (e.g., addItem, loadData).  
* **Why it was found:** Flagged by **Pylint (C0103)**. The official Python style guide (PEP 8\) recommends snake\_case (e.g., add\_item, load\_data) for function and variable names for better readability.  
* **How it was fixed:** All function names were **renamed** to use snake\_case (e.g., addItem \-\> add\_item). All calls to these functions in main() were also updated.

### **Issue: Missing Main Execution Block**

* **What:** The main() function was called directly at the global scope.  
* **Why it was a problem:** This is a common Python convention. Code at the global level will run *even if the file is imported* as a module into another script, which is almost never desired.  
* **How it was fixed:** The final call to main() was wrapped in the standard if \_\_name\_\_ \== "\_\_main\_\_": block. This ensures the main() function only runs when the script is executed directly.

