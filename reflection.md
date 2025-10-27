# **Lab 5: Static Code Analysis Reflection**

### **1\. Which issues were the easiest to fix, and which were the hardest? Why?**

* **Easiest:** The easiest issues to fix were the ones that were simple to understand and required just a one-line change.  
  * **eval-used (Bandit/Pylint):** This was the easiest, as the fix was to simply delete a dangerous, non-essential line of code.  
  * **unused-import (Flake8/Pylint):** This was also very easy, requiring only the deletion of the import logging line.  
  * **PEP 8 Styling (Flake8/Pylint):** Issues like invalid-name (e.g., addItem to add\_item) were easy, just a matter of renaming.  
* **Hardest:** The hardest issues were those that involved Python's specific scoping rules or potential logic errors.  
  * **dangerous-default-value (Pylint):** This was tricky because it's a non-obvious bug. You have to understand *why* a mutable default (logs=\[\]) is bad (it's shared across all calls) and how to fix it correctly (using logs=None and re-initializing).  
  * **global variable SyntaxError:** The SyntaxError we encountered with the global stock\_data placement was the hardest part. It required understanding that the global declaration must come at the very top of a function, *before* any assignments happen, even within try...except blocks.

### **2\. Did the static analysis tools report any false positives? If so, describe one example.**

Yes, there was one warning that could be considered a "false positive" in this specific context:

* **global-statement (Pylint W0603):** Pylint warns against using the global keyword in load\_data. While this is generally good advice (modifying global state is often bad practice), it was a *necessary* part of this simple script's design. The entire point of load\_data was to fetch data from a file and update the single, module-level stock\_data dictionary. In this case, the warning was technically correct but not actionable, so we had to accept its use.

### **3\. How would you integrate static analysis tools into your actual software development workflow? Consider continuous integration (CI) or local development practices.**

I would integrate these tools at both the local and CI levels:

* **Local Development:**  
  1. **Editor Integration:** I would use a code editor (like VS Code) with extensions for Pylint and Flake8. This provides real-time feedback, highlighting errors and style issues as I type, which is the fastest way to fix them.  
  2. **Pre-Commit Hooks:** I would set up a pre-commit hook (using a tool like pre-commit) that automatically runs flake8, bandit, and pylint on my changed files *before* I'm allowed to make a commit. This stops bad code from ever even entering the repository.  
* **Continuous Integration (CI):**  
  1. **GitHub Actions:** I would create a GitHub Actions workflow that runs on every push or pull request.  
  2. **Workflow Steps:** This workflow would (1) check out the code, (2) install dependencies, and (3) run all three static analysis tools (flake8, bandit \-r ., pylint inventory\_system.py).  
  3. **Fail the Build:** If any of the tools (especially Flake8 for style or Bandit for security) find an issue, the workflow would "fail." This would block the pull request from being merged, forcing the developer to fix the issues first.

### **4\. What tangible improvements did you observe in the code quality, readability, or potential robustness after applying the fixes?**

The improvements were significant and tangible:

* **Robustness (Most Important):** The code is much safer and more reliable.  
  * It no longer has a critical **security vulnerability** (eval()).  
  * It no longer crashes if you try to removeItem that doesn't exist (fixed bare except).  
  * It no longer crashes if the inventory.json file is missing (added FileNotFoundError catch).  
  * It won't leak file resources (fixed with with open()).  
  * It won't produce strange, hard-to-debug logging errors (fixed dangerous-default-value).  
* **Readability:** The code is much easier to read.  
  * All functions follow a **consistent naming style** (snake\_case).  
  * Every function now has a **docstring**, so I know what it does without having to read its internal logic.  
  * Using f-strings (f"{item}: {qty}") makes the print statements clearer.  
* **Maintainability:** The code is now much easier to change or fix in the future. Because it's readable and robust, a new developer (or me, in six months) can confidently make changes without worrying about breaking hidden logic.

