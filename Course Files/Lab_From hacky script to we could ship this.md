LAB | From hacky script to "we could ship this" Lab

Learning Goal

You will build one Python for Low Code: Refactoring with AI workflow path and prove data moves from input to output correctly.
Working Context

This lab turns a manual handoff into a small automated workflow. You define one input, one transformation, and one output a reviewer can inspect. Avoid broad automation. Prove the critical path first.
Proof of Learning

Submit one lab_proof.md with the workflow export or configuration, one execution trace, one input payload, and one output record.
Pseudocode

INPUT: source event, message, ticket, or payload
DECIDE: required fields, routing rule, and destination
BUILD: trigger -> transform -> action
VERIFY: inspect one successful run and output artifact
EXPLAIN: justify the mapping and operational risk

Prompt Discipline

Use AI for review, debugging, or critique. Do not ask it to produce the final artifact or final answer.

Role: Workflow reviewer
Task: Challenge my event-flow plan before I build or submit.
Inputs: trigger, sample payload, destination schema, and proof plan.
Constraints: Do not build the workflow for me. Find brittle mappings and missing checks.
Output format: three workflow risks, two trace checks, and one simplification.
Verification: Name the execution evidence that proves the workflow ran.

Build
Step 1: Select Your Code

Do: Choose which code to refactor.

Do:

    Open your code from Lab M1.05 (OpenAI API) or M1.06 (JSON Validation)
    If you completed both, choose the one that needs more refactoring
    Make a copy of your code to preserve the original:
    Run your original script before refactoring to confirm baseline behavior:

    python product_generator.py

Verify: You have your code ready to refactor.

Checkpoint: Verify you can run your original code successfully.
Step 2: Analyze Your Code

Do: Identify refactoring opportunities in your own code.

Do:

    Read through your code and identify:
        Monolithic functions:
        Functions that do too much (load + validate + API + process all together)
        Repeated code:
        Logic that appears multiple times (could be helper functions)
        Silent failures:
        Places where errors occur but aren't shown
        Mixed concerns:
        Validation logic mixed with API calls, or processing mixed with file operations
        Hardcoded values:
        Strings, numbers, or logic that should be parameters or constants

    Create a list of issues to fix. Use this template:

    ## Refactoring Checklist
     
    ### Issues Found:
    - [ ] Function `X` does too much (loads file AND validates AND calls API)
    - [ ] Error handling missing in function `Y`
    - [ ] Code repeated in functions `A` and `B` (could be helper function)
    - [ ] Hardcoded prompt in function `Z`
    - [ ] No error message when file not found
    - [ ] Validation errors caught but not shown
     
    ### Priority:
    1. [Most critical issue]
    2. [Second priority]
    3. [Third priority]

    Questions to ask yourself:
        Where does my code fail silently?
        What logic is repeated that could be a helper function?
        What functions do multiple things?
        Where is error handling missing?
        What would break if I changed one part?

Verify: A prioritized list of refactoring tasks.

Checkpoint: You have a clear list of what needs to be refactored.
Step 3: Create Helper Functions

Do: Extract reusable logic from your code into helper functions.

Do:

    Identify repeated patterns in your code. Look for:
        JSON loading/parsing (appears multiple times?)
        Data validation (repeated validation logic?)
        Prompt creation (hardcoded prompts?)
        Response parsing (same parsing logic in multiple places?)
        Output formatting (formatting logic repeated?)

    Create helper functions. Here are examples of what you might create:

    Test each helper function independently:

Verify: Helper functions extracted and tested independently.

Checkpoint: Each helper function works on its own.
Step 4: Modularize Your Functions

Do: Break large functions into smaller, focused ones.

Do:

    Identify functions that do multiple things. Look for functions that:
        Load data AND validate AND process
        Validate AND call API AND save results
        Do file operations AND business logic

    Split them into smaller functions. Each function should have ONE responsibility:

    Ensure each function has a single responsibility:
        Loading function:
        Only loads data
        Validation function:
        Only validates data
        API function:
        Only calls API
        Processing function:
        Only processes data
        Saving function:
        Only saves results

Verify: Modular, focused functions where each does one thing.

Checkpoint: Each function has a clear, single purpose.
Step 5: Add Error Handling

Do: Add comprehensive error handling that shows WHERE errors occur.

Do:

    Identify all places where errors could occur:
        File operations (file not found, permission errors)
        JSON parsing (invalid JSON format)
        Data validation (Pydantic validation errors)
        API calls (network errors, API errors, rate limits)
        Response processing (missing fields, wrong format)

    Add try/except blocks with specific error messages. For each error, show:
        Function name
        where error occurred
        Error type
        and message
        Context
        (file path, line number if possible)
        Helpful suggestions
        (what to check, how to fix)

    Test with invalid inputs to verify error messages:

Error message format:

Verify: All errors show WHERE they occurred with helpful context.

Checkpoint: Test with invalid inputs - all errors show clear messages.
Step 6: Test Your Refactored Code

Do: Verify your refactored code works correctly.

Do:

    Test with valid data:
        Should work exactly as before
        Should produce same results

    Test with invalid data:
        Missing file: should show file path and suggestion
        Invalid JSON: should show line and column of error
        Invalid product data: should show which fields are invalid
        API errors: should show API error details

    Test error scenarios:

    Compare behavior:
        Does refactored code work the same as original?
        Are error messages better?
        Is code easier to understand?

Verify: Refactored code works correctly with better error handling.

Checkpoint: All test cases pass with appropriate error messages.
Step 7: Create API Wrapper (Optional - Advanced)

    Note: This is a spoiler for tomorrow's class. This step is optional and advanced.

Do: Create a reusable API wrapper class.

Do:

    Extract OpenAI API calls into a wrapper class:

    Add retry logic with exponential backoff

    Standardize error responses

    Use wrapper in your refactored code

Verify: Reusable API wrapper that handles errors and retries.

Checkpoint: API wrapper works and handles errors gracefully.
Step 8: Add Logging (Optional - Advanced)

Do: Add logging for debugging and monitoring.

Do:

    Set up logging configuration:

    Log key operations:
        File loading (success/failure)
        Validation results
        API calls (requests, responses, errors)
        Processing progress

    Use logging throughout your code:

Verify: Complete audit trail of operations in log file.

Checkpoint: Log file contains useful information for debugging.
Path 2: Refactor Provided Starter Code
Step 1: Analyze the Starter Code

Do: Identify all the issues in the provided starter code.

Do:

    Read through the starter code below
    Identify where it fails silently
    List all the concerns mixed together
    Note where error handling is missing

Starter Code:

Tip: if you are not using the Field helper from Pydantic, this code will need to be adapted.

Analysis Questions:

    What happens if
    products.json
    doesn't exist?
    What happens if JSON is invalid?
    What happens if product validation fails?
    What happens if API call fails?
    What functions do multiple things?
    Where is code repeated that could be a helper function?

Verify: A list of issues to fix.

Checkpoint: You've identified at least 5 major issues in the starter code.
Step 2: Create Helper Functions

Do: Extract reusable logic into helper functions.

Do:

    Create helper functions for common operations:

    Test each helper function independently before using in main code.

Verify: Helper functions that can be tested independently.

Checkpoint: Each helper function works and handles errors properly.
Step 3: Modularize Functions

Do: Break monolithic function into focused modules.

Do:

    Split the big function into smaller, focused functions:

    Ensure each function has a single responsibility.

Verify: Each function has single responsibility.

Checkpoint: Main function now just orchestrates, doesn't do the work.
Step 4: Add Error Handling (CRITICAL)

Do: Add comprehensive error handling that shows WHERE errors occur.

Do:

    FileNotFoundError: Show file path and suggest checking file location

    JSONDecodeError: Show line number and character position

    Pydantic ValidationError: Show which fields are invalid and why

    OpenAI API errors: Show error type, status code, and message

    Network errors: Show timeout/connection details

Error message format:

Verify: All errors show WHERE they occurred, not silent failures.

Checkpoint: Test with invalid inputs - all errors show clear, helpful messages.
Step 5: Test Your Refactored Code

Do: Verify refactored code handles all scenarios.

Do:

    Test with valid JSON file:
        Should process products successfully
        Should generate descriptions
        Should save results

    Test with missing file:

    Test with invalid JSON:

    Test with invalid product data:

    Test with API errors:

Verify: All test cases show clear, helpful error messages.

Checkpoint: All error scenarios handled with clear messages.
Step 6: Create OpenAI API Wrapper (Optional - Advanced)

Do: Create a reusable API wrapper class.

Do:

    Create OpenAIWrapper class:

    Include retry logic with exponential backoff

    Standardize error responses

    Add timeout handling

    Use wrapper in refactored code

Verify: Clean API wrapper that can be reused.

Checkpoint: API wrapper handles errors and retries automatically.
Step 7: Add Logging (Optional - Advanced)

Do: Add logging for debugging and monitoring.

Do:

    Set up logging configuration:

    Log key operations:

    Log file operations, validation results, API calls, and errors

Verify: Complete audit trail of operations.

Checkpoint: Log file contains useful debugging information.
Step 8: Refactor Previous Project Code (Optional - Advanced)

Do: Apply refactoring to code from M1.05 or M1.06.

Do:

    Take code from previous lab (M1.05 or M1.06)
    Apply same refactoring principles:
        Create helper functions
        Modularize code
        Add error handling
    Compare before/after

Verify: Improved version of previous lab code.

Checkpoint: Previous lab code is now more maintainable.
Sample JSON File

Create a file named products.json with this structure:

{
  "products": [
    {
      "id": "P001",
      "name": "Wireless Bluetooth Headphones",
      "category": "Electronics",
      "price": 99.99,
      "features": ["Bluetooth 5.0", "Noise Cancelling", "30hr Battery", "Comfortable Fit"]
    },
    {
      "id": "P002",
      "name": "Smart Watch",
      "category": "Wearables",
      "price": 249.99,
      "features": ["Heart Rate Monitor", "GPS", "Water Resistant", "Sleep Tracking"]
    },
    {
      "id": "P003",
      "name": "Laptop Stand",
      "category": "Accessories",
      "price": 49.99,
      "features": ["Adjustable Height", "Aluminum Construction", "Cable Management"]
    }
  ]
}

For testing error handling, also create:

    invalid_products.json
    - Products with validation errors (negative prices, missing fields)
    malformed.json
    - Invalid JSON syntax
    Test with missing file (use non-existent filename)

Verification Criteria

Before submitting, verify:

    Code is refactored into helper functions
    Code is modular (separate functions for separate concerns)
    Error handling shows WHERE errors occur (not silent failures)
    All error types are handled explicitly:
        FileNotFoundError
        JSONDecodeError
        Pydantic ValidationError
        OpenAI APIError
        Network errors
    Code works with provided JSON data
    Error messages include:
        Function name where error occurred
        Error type and message
        Location/context
        Helpful suggestions
    (If extras) API wrapper created with retry logic
    (If extras) Logging implemented and working

Submission Guidelines
What to Submit

Required deliverables:

    Your refactored code file (.py or .ipynb)
    Before/after comparison (can be comments in code or separate document)
    Screenshots or output showing:
        Successful processing with valid data
        Error messages for invalid data (showing WHERE errors occur)
        Error messages for missing files
        Error messages for API errors
    lab_summary.md at the repository root with one short paragraph covering: what you refactored (Path 1 vs Path 2), how you modularized and improved error handling, the main challenge you hit, and what you learned. Use code comments, screenshots, or checklists for detail. Do not write a separate multi-page report.

If you completed extras:

    API wrapper class code
    Logging examples (screenshot of log file)
    Refactored previous lab code (if applicable)

How to Submit

Upload your work:

    For code files:
    Upload your Python script or notebook
    For screenshots:
    Upload PNG or JPG images showing error messages
    For reports:
    Upload PDF or text file

Important notes:

    DO NOT include your API key in submitted files
    Make sure your code uses environment variables for the API key
    Test your code before submitting
    Include comments explaining your refactoring approach
    Show examples of error messages in screenshots or in your README

Due date: Check with your instructor for the specific due date.
Troubleshooting

Common issues and solutions:

Issue 1: "My code still fails silently"

    Solution:
        Make sure you're printing error messages before raising exceptions
        Check that all try/except blocks include error messages
        Test with invalid inputs to verify error messages appear

Issue 2: "Error messages don't show WHERE the error occurred"

    Solution:
        Include function name in error message
        Include file path for file operations
        Include line numbers for JSON errors
        Use traceback module for more detailed location info

Issue 3: "I don't know what to refactor"

    Solution:
        Look for functions longer than 20-30 lines
        Look for repeated code patterns
        Look for functions that do multiple things
        Ask: "What would break if I changed this part?"

Issue 4: "Helper functions are too small"

    Solution:
        Helper functions should be small and focused
        It's better to have many small functions than few large ones
        Small functions are easier to test and reuse

Issue 5: "Refactoring broke my code"

    Solution:
        Test each helper function independently
        Test after each refactoring step
        Keep original code as backup
        Use version control (git) to track changes

Additional Resources

Reference materials:

    Morning lesson: "Automation with Python" (refactoring concepts)
    Python Error Handling Best Practices
    Pydantic Validation Error Handling
    OpenAI API Error Handling

Tools and software:

    Python Debugger (pdb)
    Python Logging

Learning Reflection

Optional self-check: not part of the graded submission beyond the single paragraph above.

After completing this lab, reflect on what you've learned:

    Refactoring:
    What makes code easier to maintain?
    Helper Functions:
    How do helper functions improve code quality?
    Error Handling:
    Why is it important to show WHERE errors occur?
    Modularity:
    How does separating concerns make code better?
    Testing:
    How does refactoring make code easier to test?

Key takeaways:

    Refactoring is essential for managing technical debt
    Helper functions make code reusable and testable
    Error handling must show WHERE errors occur, not just that they occurred
    Modular code is easier to understand, test, and maintain
    These skills are crucial for professional AI-assisted development

These skills ensure your code is production-ready and maintainable, even when using AI tools to generate code.

Great work on refactoring your code! You've learned essential skills for professional development that you'll use throughout your career.
Verify

The reviewer can inspect one run trace and see the observable output in the destination.
Explain

What is the first failure mode you would monitor if this workflow ran daily?
Submission Sanity Check

Before you submit:

    Keep only files for this lab.
    Name files clearly.
    Remove secrets, API keys, tokens, and unrelated data.
    Make lab_proof.md the entry point for the reviewer.

Optional Stretch

    Add one operational control: retry handling, error notification, duplicate detection, audit logging, or a second route.
    Create a small UI with vibecoding only if it safely submits or previews the workflow input. Deploy only if no credentials are exposed.
