# OUI Error Handling Union Strategy

## Table of Contents
1. [Introduction](#introduction)
2. [Why This Strategy?](#why-this-strategy)
3. [Basic Concept](#basic-concept)
4. [Implementation Guide](#implementation-guide)
5. [Basic Example](#basic-example)
6. [ErrorOrVoid for Functions Without Return Values](#errororvoid-for-functions-without-return-values)
7. [Advanced Example with Formatted Messages](#advanced-example-with-formatted-messages)
8. [Complete Real-World Examples](#complete-real-world-examples)
9. [Best Practices](#best-practices)
10. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
11. [Performance Considerations](#performance-considerations)

## Introduction

This is the **default error handling strategy** used by OUI (Our Universal Interface). Instead of using traditional C error handling methods like return codes or global `errno`, we use a union-based approach that encapsulates both successful results and error information in a single return type.

**Key Principle:** A function returns either a successful value OR an error, but never both. This makes error handling explicit and harder to ignore.

## Why This Strategy?

### Problems with Traditional C Error Handling

```c
// Traditional approach - many problems!
int unsafe_divide(float a, float b, float *result) {
    if (b == 0) {
        return -1; // What does -1 mean? Unclear!
    }
    *result = a / b;
    return 0; // Success... maybe?
}

// Usage - error-prone!
float result;
int status = unsafe_divide(10, 0, &result);
// Easy to forget checking status
printf("Result: %f\n", result); // Using garbage value!
```

**Problems:**
- ❌ Easy to forget error checking
- ❌ Error codes are unclear (what does -1 mean?)
- ❌ No error description
- ❌ Can accidentally use uninitialized values
- ❌ Global `errno` can be overwritten

### Benefits of OUI's Union Strategy

```c
// OUI approach - much safer!
ErrorOrFloat safe_divide(float a, float b) {
    if (b == 0) {
        return newErrorOrFloatError(DIVISION_BY_ZERO, "Cannot divide by zero");
    }
    return newErrorOrFloatValue(a / b);
}

// Usage - explicit and safe!
ErrorOrFloat result = safe_divide(10, 0);
if (result.is_error) {
    printf("Error: %s (code: %d)\n", result.error, result.code);
} else {
    printf("Result: %f\n", result.value);
}
```

**Benefits:**
- ✅ **Explicit**: Must check for errors before using value
- ✅ **Self-documenting**: Function signature shows it can fail
- ✅ **Type-safe**: Compiler helps catch mistakes
- ✅ **Descriptive**: Clear error messages and codes
- ✅ **No global state**: Everything is contained in the return value

## Basic Concept

The core idea is simple: create a struct that can hold either:
- A **successful result** of your desired type
- An **error** with description and code

```c
typedef struct {
    float value;         // The successful result (only valid if !is_error)
    const char *error;   // Human-readable error message
    int code;           // Machine-readable error code
    bool is_error;      // Flag: true = error, false = success
} ErrorOrFloat;
```

**Important:** Only access `value` when `is_error` is `false`, and only access `error`/`code` when `is_error` is `true`.

### Special Case: ErrorOrVoid

For functions that don't return a value but can still fail, we use `ErrorOrVoid`:

```c
typedef struct {
    bool is_error;      // Flag: true = error, false = success
    char *error_msg;    // Human-readable error message
} ErrorOrVoid;
```

This is perfect for operations like file writing, configuration changes, or cleanup functions that either succeed or fail without returning data.

## Implementation Guide

### Step 1: Define Error Codes

```c
// Define meaningful error codes for your domain
enum MathErrors {
    DIVISION_BY_ZERO = 1001,
    NEGATIVE_SQUARE_ROOT = 1002,
    OVERFLOW_ERROR = 1003,
    INVALID_INPUT = 1004
};

enum FileErrors {
    FILE_NOT_FOUND = 2001,
    FILE_PERMISSION_DENIED = 2002,
    FILE_TOO_LARGE = 2003
};
```

### Step 2: Define the Error-or-Value Type

```c
typedef struct {
    float value;         // Your actual data type here
    const char *error;   // Static error message
    int code;           // Error code from enum above
    bool is_error;      // true = error, false = success
} ErrorOrFloat;
```

### Step 3: Create Constructor Functions

```c
// Create a successful result
ErrorOrFloat newErrorOrFloatValue(float value) {
    ErrorOrFloat result = {0}; // Initialize everything to 0
    result.is_error = false;
    result.value = value;
    return result;
}

// Create an error result
ErrorOrFloat newErrorOrFloatError(int code, const char *error) {
    ErrorOrFloat result = {0}; // Initialize everything to 0
    result.is_error = true;
    result.error = error;
    result.code = code;
    return result;
}
```

**Why initialize with `{0}`?** This ensures all unused fields are zero, preventing garbage values that could cause bugs.

## Basic Example

Here's the complete basic implementation that OUI uses:

```c
#include <string.h>  
#include <stdlib.h> 
#include <stdio.h>  
#include <stdbool.h> 

// Step 1: Define error codes
enum {
    DIVISION_BY_ZERO = 1001,
    OTHER_ERROR = 1002
};

// Step 2: Define the union type
typedef struct {
    float value;
    const char *error;
    int code;
    bool is_error;
} ErrorOrFloat;

// Step 3: Constructor function declarations
ErrorOrFloat newErrorOrFloatError(int code, const char *error);
ErrorOrFloat newErrorOrFloatValue(float value);
ErrorOrFloat divide(float a, float b);

// Step 4: Implementation of constructors
ErrorOrFloat newErrorOrFloatError(int code, const char *error) {
    ErrorOrFloat e = {0}; // Initialize all fields
    e.is_error = true;
    e.error = error;
    e.code = code;
    return e;
}

ErrorOrFloat newErrorOrFloatValue(float value) {
    ErrorOrFloat e = {0}; // Initialize all fields
    e.is_error = false;
    e.value = value;
    return e;
}

// Step 5: Your business logic using the pattern
ErrorOrFloat divide(float a, float b) {
    if (b == 0) {
        return newErrorOrFloatError(DIVISION_BY_ZERO, "Division by zero is not allowed");
    }
    return newErrorOrFloatValue(a / b);
}

// Step 6: Usage
int main() {
    // Test error case
    ErrorOrFloat result = divide(10, 0);
    if (result.is_error) {
        printf("Error: %s (code: %d)\n", result.error, result.code);
    } else {
        printf("Result: %f\n", result.value);
    }
    
    // Test success case
    ErrorOrFloat result2 = divide(10, 2);
    if (result2.is_error) {
        printf("Error: %s (code: %d)\n", result2.error, result2.code);
    } else {
        printf("Result: %f\n", result2.value);
    }

    return 0;
}
```

**Output:**
```
Error: Division by zero is not allowed (code: 1001)
Result: 5.000000
```

## ErrorOrVoid for Functions Without Return Values

Many functions don't need to return data but still need to report success or failure. For these cases, we use `ErrorOrVoid`:

### Definition

```c
typedef struct {
    bool is_error;      // Flag: true = error, false = success
    char *error_msg;    // Human-readable error message (can be static or dynamic)
} ErrorOrVoid;
```

### Constructor Functions

```c
// Create a success result (no error)
ErrorOrVoid newErrorOrVoidSuccess() {
    ErrorOrVoid result = {0}; // Initialize all fields
    result.is_error = false;
    result.error_msg = NULL;
    return result;
}

// Create an error result with static message
ErrorOrVoid newErrorOrVoidError(const char *error_msg) {
    ErrorOrVoid result = {0}; // Initialize all fields
    result.is_error = true;
    result.error_msg = (char*)error_msg; // Cast away const for simplicity
    return result;
}
```

### Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

// Error codes for file operations
enum FileOperationErrors {
    FILE_WRITE_ERROR = 5001,
    FILE_PERMISSION_ERROR = 5002,
    INVALID_PARAMETER = 5003
};

// ErrorOrVoid definition
typedef struct {
    bool is_error;
    char *error_msg;
} ErrorOrVoid;

// Constructor functions
ErrorOrVoid newErrorOrVoidSuccess() {
    ErrorOrVoid result = {0};
    result.is_error = false;
    result.error_msg = NULL;
    return result;
}

ErrorOrVoid newErrorOrVoidError(const char *error_msg) {
    ErrorOrVoid result = {0};
    result.is_error = true;
    result.error_msg = (char*)error_msg;
    return result;
}

// Example function: Write configuration to file
ErrorOrVoid save_config(const char *filename, const char *config_data) {
    // Validate parameters
    if (filename == NULL || config_data == NULL) {
        return newErrorOrVoidError("Filename and config data cannot be NULL");
    }
    
    if (strlen(filename) == 0) {
        return newErrorOrVoidError("Filename cannot be empty");
    }
    
    // Try to open file for writing
    FILE *file = fopen(filename, "w");
    if (file == NULL) {
        return newErrorOrVoidError("Cannot open file for writing - check permissions");
    }
    
    // Write configuration data
    size_t data_len = strlen(config_data);
    size_t written = fwrite(config_data, 1, data_len, file);
    
    if (written != data_len) {
        fclose(file);
        return newErrorOrVoidError("Failed to write complete configuration data");
    }
    
    // Ensure data is written to disk
    if (fflush(file) != 0) {
        fclose(file);
        return newErrorOrVoidError("Failed to flush configuration data to disk");
    }
    
    fclose(file);
    return newErrorOrVoidSuccess(); // Success!
}

// Example function: Initialize a system component
ErrorOrVoid initialize_logging_system(const char *log_dir) {
    if (log_dir == NULL) {
        return newErrorOrVoidError("Log directory path cannot be NULL");
    }
    
    // Simulate initialization steps
    printf("Initializing logging system in directory: %s\n", log_dir);
    
    // Check if directory exists (simplified check)
    if (strlen(log_dir) > 100) {
        return newErrorOrVoidError("Log directory path too long (max 100 characters)");
    }
    
    // Simulate successful initialization
    printf("Logging system initialized successfully\n");
    return newErrorOrVoidSuccess();
}

// Example function: Cleanup resources
ErrorOrVoid cleanup_resources() {
    printf("Cleaning up allocated resources...\n");
    
    // Simulate cleanup operations
    // In real code, this might free memory, close files, etc.
    
    printf("Resource cleanup completed\n");
    return newErrorOrVoidSuccess();
}

int main() {
    printf("=== ErrorOrVoid Examples ===\n\n");
    
    // Example 1: Save configuration (success case)
    printf("1. Saving configuration to file:\n");
    ErrorOrVoid result1 = save_config("config.txt", "debug=true\nport=8080\n");
    if (result1.is_error) {
        printf("   Error: %s\n", result1.error_msg);
    } else {
        printf("   Configuration saved successfully!\n");
    }
    
    printf("\n");
    
    // Example 2: Save configuration (error case)
    printf("2. Saving configuration with invalid parameters:\n");
    ErrorOrVoid result2 = save_config(NULL, "some config");
    if (result2.is_error) {
        printf("   Error: %s\n", result2.error_msg);
    } else {
        printf("   Configuration saved successfully!\n");
    }
    
    printf("\n");
    
    // Example 3: Initialize logging system (success case)
    printf("3. Initializing logging system:\n");
    ErrorOrVoid result3 = initialize_logging_system("/var/log/myapp");
    if (result3.is_error) {
        printf("   Error: %s\n", result3.error_msg);
    } else {
        printf("   Logging system ready!\n");
    }
    
    printf("\n");
    
    // Example 4: Initialize logging system (error case)
    printf("4. Initializing logging with invalid path:\n");
    ErrorOrVoid result4 = initialize_logging_system(
        "/this/is/a/very/long/path/that/exceeds/the/maximum/allowed/length/for/logging/directory/configuration");
    if (result4.is_error) {
        printf("   Error: %s\n", result4.error_msg);
    } else {
        printf("   Logging system ready!\n");
    }
    
    printf("\n");
    
    // Example 5: Cleanup (typically always succeeds)
    printf("5. Cleaning up resources:\n");
    ErrorOrVoid result5 = cleanup_resources();
    if (result5.is_error) {
        printf("   Error during cleanup: %s\n", result5.error_msg);
    } else {
        printf("   Cleanup completed successfully!\n");
    }
    
    return 0;
}
```

### Output

```
=== ErrorOrVoid Examples ===

1. Saving configuration to file:
   Configuration saved successfully!

2. Saving configuration with invalid parameters:
   Error: Filename and config data cannot be NULL

3. Initializing logging system:
Initializing logging system in directory: /var/log/myapp
Logging system initialized successfully
   Logging system ready!

4. Initializing logging with invalid path:
   Error: Log directory path too long (max 100 characters)

5. Cleaning up resources:
Cleaning up allocated resources...
Resource cleanup completed
   Cleanup completed successfully!
```

### Common Use Cases for ErrorOrVoid

1. **File Operations**: Writing, deleting, moving files
2. **System Initialization**: Setting up components, configurations
3. **Resource Management**: Cleanup, memory deallocation
4. **Validation**: Checking data integrity, parameter validation
5. **Network Operations**: Sending data (when response content isn't needed)
6. **Database Operations**: INSERT, UPDATE, DELETE statements

### ErrorOrVoid with Dynamic Error Messages

For more detailed error reporting, you can use formatted error messages:

```c
#include <stdarg.h>

typedef struct {
    bool is_error;
    char error_msg[200];  // Fixed buffer for formatted messages
} ErrorOrVoidFormatted;

ErrorOrVoidFormatted newErrorOrVoidFormattedSuccess() {
    ErrorOrVoidFormatted result = {0};
    result.is_error = false;
    return result;
}

ErrorOrVoidFormatted newErrorOrVoidFormattedError(const char *format, ...) {
    ErrorOrVoidFormatted result = {0};
    result.is_error = true;
    
    va_list args;
    va_start(args, format);
    vsnprintf(result.error_msg, sizeof(result.error_msg), format, args);
    va_end(args);
    
    return result;
}

// Example usage with formatted messages
ErrorOrVoidFormatted validate_user_age(int age) {
    if (age < 0) {
        return newErrorOrVoidFormattedError(
            "Age cannot be negative: received %d", age);
    }
    if (age > 150) {
        return newErrorOrVoidFormattedError(
            "Age %d seems unrealistic (max: 150)", age);
    }
    return newErrorOrVoidFormattedSuccess();
}
```

**Key Benefits of ErrorOrVoid:**
- ✅ **Explicit**: Cannot ignore that function might fail
- ✅ **Lightweight**: Smaller than full ErrorOrValue types
- ✅ **Clear Intent**: Function signature shows it's a side-effect operation
- ✅ **Consistent**: Follows same pattern as other Error-or-X types

## Advanced Example with Formatted Messages

Sometimes you need dynamic error messages with specific values. Here's how to do it (note: uses more memory):

```c
#include <string.h>  
#include <stdlib.h> 
#include <stdio.h>  
#include <stdbool.h> 
#include <stdarg.h>

enum {
    DIVISION_BY_ZERO = 1001,
    NEGATIVE_INPUT = 1002,
    OUT_OF_RANGE = 1003
};

// Modified struct with fixed-size error buffer
typedef struct {
    float value;
    char error[100];    // Fixed-size buffer for formatted messages
    int code;
    bool is_error;
} ErrorOrFloat;

// Variadic constructor for formatted error messages
ErrorOrFloat newErrorOrFloatError(int code, char *format, ...) {
    ErrorOrFloat e = {0};
    e.is_error = true;
    e.code = code;
    
    // Format the error message with provided arguments
    va_list args;
    va_start(args, format);
    vsnprintf(e.error, sizeof(e.error), format, args);
    va_end(args);
    
    return e;
}

ErrorOrFloat newErrorOrFloatValue(float value) {
    ErrorOrFloat e = {0};
    e.is_error = false;
    e.value = value;
    return e;
}

ErrorOrFloat divide(float a, float b) {
    if (b == 0) {
        return newErrorOrFloatError(DIVISION_BY_ZERO, 
            "Cannot divide %.2f by zero", a);
    }
    return newErrorOrFloatValue(a / b);
}

ErrorOrFloat safe_sqrt(float x) {
    if (x < 0) {
        return newErrorOrFloatError(NEGATIVE_INPUT, 
            "Cannot take square root of negative number: %.2f", x);
    }
    return newErrorOrFloatValue(sqrt(x));
}

ErrorOrFloat validate_range(float x, float min, float max) {
    if (x < min || x > max) {
        return newErrorOrFloatError(OUT_OF_RANGE, 
            "Value %.2f is outside valid range [%.2f, %.2f]", x, min, max);
    }
    return newErrorOrFloatValue(x);
}

int main() {
    // Test division by zero with specific value
    ErrorOrFloat result1 = divide(42.5, 0);
    if (result1.is_error) {
        printf("Error: %s (code: %d)\n", result1.error, result1.code);
    }
    
    // Test negative square root
    ErrorOrFloat result2 = safe_sqrt(-25.0);
    if (result2.is_error) {
        printf("Error: %s (code: %d)\n", result2.error, result2.code);
    }
    
    // Test range validation
    ErrorOrFloat result3 = validate_range(150.0, 0.0, 100.0);
    if (result3.is_error) {
        printf("Error: %s (code: %d)\n", result3.error, result3.code);
    }
    
    // Test successful operation
    ErrorOrFloat result4 = divide(15.0, 3.0);
    if (result4.is_error) {
        printf("Error: %s (code: %d)\n", result4.error, result4.code);
    } else {
        printf("Success: 15.0 / 3.0 = %.2f\n", result4.value);
    }

    return 0;
}
```

**Output:**
```
Error: Cannot divide 42.50 by zero (code: 1001)
Error: Cannot take square root of negative number: -25.00 (code: 1002)
Error: Value 150.00 is outside valid range [0.00, 100.00] (code: 1003)
Success: 15.0 / 3.0 = 5.00
```

## Complete Real-World Examples

### Example 1: String Processing with Memory Management

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

// Error codes for string operations
enum StringErrors {
    STRING_NULL_INPUT = 2001,
    STRING_EMPTY_INPUT = 2002,
    STRING_MEMORY_ERROR = 2003,
    STRING_TOO_LONG = 2004
};

typedef struct {
    char *value;           // Dynamically allocated string
    char error[150];       // Error message buffer
    int code;
    bool is_error;
} ErrorOrString;

ErrorOrString string_success(char *value) {
    ErrorOrString result = {0};
    result.is_error = false;
    result.value = value;
    return result;
}

ErrorOrString string_error(int code, const char *format, ...) {
    ErrorOrString result = {0};
    result.is_error = true;
    result.code = code;
    
    va_list args;
    va_start(args, format);
    vsnprintf(result.error, sizeof(result.error), format, args);
    va_end(args);
    
    return result;
}

ErrorOrString safe_string_duplicate(const char *input) {
    // Validate input
    if (input == NULL) {
        return string_error(STRING_NULL_INPUT, 
            "Input string pointer is NULL");
    }
    
    size_t len = strlen(input);
    if (len == 0) {
        return string_error(STRING_EMPTY_INPUT, 
            "Input string is empty");
    }
    
    if (len > 1000) {
        return string_error(STRING_TOO_LONG, 
            "Input string too long: %zu characters (max: 1000)", len);
    }
    
    // Allocate memory
    char *copy = malloc(len + 1);
    if (copy == NULL) {
        return string_error(STRING_MEMORY_ERROR, 
            "Failed to allocate %zu bytes for string copy", len + 1);
    }
    
    strcpy(copy, input);
    return string_success(copy);
}

ErrorOrString string_to_uppercase(const char *input) {
    // First, safely duplicate the string
    ErrorOrString dup_result = safe_string_duplicate(input);
    if (dup_result.is_error) {
        return dup_result; // Propagate the error
    }
    
    // Convert to uppercase
    char *str = dup_result.value;
    for (size_t i = 0; str[i]; i++) {
        if (str[i] >= 'a' && str[i] <= 'z') {
            str[i] = str[i] - 'a' + 'A';
        }
    }
    
    return string_success(str);
}

// Helper function to safely free string results
void free_string_result(ErrorOrString *result) {
    if (!result->is_error && result->value != NULL) {
        free(result->value);
        result->value = NULL;
    }
}

int main() {
    // Test successful operation
    printf("=== String Processing Examples ===\n\n");
    
    ErrorOrString result1 = string_to_uppercase("hello world");
    if (result1.is_error) {
        printf("Error: %s (Code: %d)\n", result1.error, result1.code);
    } else {
        printf("Original: \"hello world\"\n");
        printf("Uppercase: \"%s\"\n", result1.value);
        free_string_result(&result1);
    }
    
    printf("\n");
    
    // Test error cases
    ErrorOrString result2 = string_to_uppercase(NULL);
    if (result2.is_error) {
        printf("Null input error: %s (Code: %d)\n", result2.error, result2.code);
    }
    
    ErrorOrString result3 = string_to_uppercase("");
    if (result3.is_error) {
        printf("Empty input error: %s (Code: %d)\n", result3.error, result3.code);
    }
    
    return 0;
}
```

### Example 2: File Operations with Detailed Error Reporting

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
#include <errno.h>

enum FileErrors {
    FILE_NOT_FOUND = 3001,
    FILE_READ_ERROR = 3002,
    FILE_MEMORY_ERROR = 3003,
    FILE_TOO_LARGE = 3004,
    FILE_PERMISSION_ERROR = 3005
};

typedef struct {
    char *content;         // File content
    size_t size;          // Size of content
    char error[200];      // Detailed error message
    int code;
    bool is_error;
} ErrorOrFileContent;

ErrorOrFileContent file_success(char *content, size_t size) {
    ErrorOrFileContent result = {0};
    result.is_error = false;
    result.content = content;
    result.size = size;
    return result;
}

ErrorOrFileContent file_error(int code, const char *format, ...) {
    ErrorOrFileContent result = {0};
    result.is_error = true;
    result.code = code;
    
    va_list args;
    va_start(args, format);
    vsnprintf(result.error, sizeof(result.error), format, args);
    va_end(args);
    
    return result;
}

ErrorOrFileContent read_file_safe(const char *filename, size_t max_size) {
    // Validate input
    if (filename == NULL) {
        return file_error(FILE_NOT_FOUND, "Filename is NULL");
    }
    
    // Try to open file
    FILE *file = fopen(filename, "rb");
    if (file == NULL) {
        int err = errno;
        if (err == ENOENT) {
            return file_error(FILE_NOT_FOUND, 
                "File '%s' does not exist", filename);
        } else if (err == EACCES) {
            return file_error(FILE_PERMISSION_ERROR, 
                "Permission denied accessing file '%s'", filename);
        } else {
            return file_error(FILE_READ_ERROR, 
                "Cannot open file '%s': %s", filename, strerror(err));
        }
    }
    
    // Get file size
    fseek(file, 0, SEEK_END);
    long file_size = ftell(file);
    fseek(file, 0, SEEK_SET);
    
    if (file_size < 0) {
        fclose(file);
        return file_error(FILE_READ_ERROR, 
            "Cannot determine size of file '%s'", filename);
    }
    
    if ((size_t)file_size > max_size) {
        fclose(file);
        return file_error(FILE_TOO_LARGE, 
            "File '%s' is too large: %ld bytes (max: %zu)", 
            filename, file_size, max_size);
    }
    
    // Allocate memory
    char *content = malloc(file_size + 1);
    if (content == NULL) {
        fclose(file);
        return file_error(FILE_MEMORY_ERROR, 
            "Cannot allocate %ld bytes for file '%s'", file_size + 1, filename);
    }
    
    // Read file content
    size_t bytes_read = fread(content, 1, file_size, file);
    if (bytes_read != (size_t)file_size) {
        free(content);
        fclose(file);
        return file_error(FILE_READ_ERROR, 
            "Read only %zu of %ld bytes from file '%s'", 
            bytes_read, file_size, filename);
    }
    
    content[file_size] = '\0'; // Null terminate
    fclose(file);
    
    return file_success(content, file_size);
}

void free_file_result(ErrorOrFileContent *result) {
    if (!result->is_error && result->content != NULL) {
        free(result->content);
        result->content = NULL;
        result->size = 0;
    }
}

int main() {
    printf("=== File Operations Examples ===\n\n");
    
    // Try to read existing file
    ErrorOrFileContent result1 = read_file_safe("sample.c", 10000);
    if (result1.is_error) {
        printf("Error reading sample.c: %s (Code: %d)\n", 
               result1.error, result1.code);
    } else {
        printf("Successfully read sample.c (%zu bytes)\n", result1.size);
        printf("First 100 characters:\n%.100s\n", result1.content);
        free_file_result(&result1);
    }
    
    printf("\n");
    
    // Try to read non-existent file
    ErrorOrFileContent result2 = read_file_safe("nonexistent.txt", 1000);
    if (result2.is_error) {
        printf("Expected error: %s (Code: %d)\n", result2.error, result2.code);
    }
    
    // Try to read file that's too large (set very low limit)
    ErrorOrFileContent result3 = read_file_safe("sample.c", 10);
    if (result3.is_error) {
        printf("Size limit error: %s (Code: %d)\n", result3.error, result3.code);
    }
    
    return 0;
}
```

### Example 3: Chaining Operations with Error Propagation

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <stdbool.h>

enum MathErrors {
    MATH_DIVISION_BY_ZERO = 4001,
    MATH_NEGATIVE_SQRT = 4002,
    MATH_INVALID_LOG = 4003,
    MATH_OVERFLOW = 4004
};

typedef struct {
    double value;
    char error[150];
    int code;
    bool is_error;
} ErrorOrDouble;

ErrorOrDouble math_success(double value) {
    ErrorOrDouble result = {0};
    result.is_error = false;
    result.value = value;
    return result;
}

ErrorOrDouble math_error(int code, const char *format, ...) {
    ErrorOrDouble result = {0};
    result.is_error = true;
    result.code = code;
    
    va_list args;
    va_start(args, format);
    vsnprintf(result.error, sizeof(result.error), format, args);
    va_end(args);
    
    return result;
}

// Macro for error propagation - makes code cleaner
#define PROPAGATE_ERROR(result) \
    do { \
        if ((result).is_error) { \
            return (result); \
        } \
    } while(0)

ErrorOrDouble safe_divide(double a, double b) {
    if (b == 0.0) {
        return math_error(MATH_DIVISION_BY_ZERO, 
            "Cannot divide %.2f by zero", a);
    }
    
    double result = a / b;
    if (!isfinite(result)) {
        return math_error(MATH_OVERFLOW, 
            "Division %.2f / %.2f resulted in overflow", a, b);
    }
    
    return math_success(result);
}

ErrorOrDouble safe_sqrt(double x) {
    if (x < 0) {
        return math_error(MATH_NEGATIVE_SQRT, 
            "Cannot take square root of negative number: %.2f", x);
    }
    return math_success(sqrt(x));
}

ErrorOrDouble safe_log(double x) {
    if (x <= 0) {
        return math_error(MATH_INVALID_LOG, 
            "Cannot take logarithm of non-positive number: %.2f", x);
    }
    return math_success(log(x));
}

// Complex operation that chains multiple potentially failing operations
ErrorOrDouble complex_calculation(double a, double b, double c) {
    printf("Computing: sqrt(log(a/b)) where a=%.2f, b=%.2f, c=%.2f\n", a, b, c);
    
    // Step 1: Divide a by b
    ErrorOrDouble step1 = safe_divide(a, b);
    PROPAGATE_ERROR(step1);
    printf("  Step 1: %.2f / %.2f = %.2f\n", a, b, step1.value);
    
    // Step 2: Take logarithm
    ErrorOrDouble step2 = safe_log(step1.value);
    PROPAGATE_ERROR(step2);
    printf("  Step 2: log(%.2f) = %.2f\n", step1.value, step2.value);
    
    // Step 3: Take square root
    ErrorOrDouble step3 = safe_sqrt(step2.value);
    PROPAGATE_ERROR(step3);
    printf("  Step 3: sqrt(%.2f) = %.2f\n", step2.value, step3.value);
    
    return step3;
}

int main() {
    printf("=== Chained Operations Examples ===\n\n");
    
    // Test successful calculation
    printf("Example 1 - Success case:\n");
    ErrorOrDouble result1 = complex_calculation(100.0, 10.0, 0.0);
    if (result1.is_error) {
        printf("Error: %s (Code: %d)\n", result1.error, result1.code);
    } else {
        printf("Final result: %.6f\n", result1.value);
    }
    
    printf("\n");
    
    // Test division by zero
    printf("Example 2 - Division by zero:\n");
    ErrorOrDouble result2 = complex_calculation(100.0, 0.0, 0.0);
    if (result2.is_error) {
        printf("Error: %s (Code: %d)\n", result2.error, result2.code);
    }
    
    printf("\n");
    
    // Test negative logarithm
    printf("Example 3 - Negative logarithm:\n");
    ErrorOrDouble result3 = complex_calculation(1.0, 10.0, 0.0);
    if (result3.is_error) {
        printf("Error: %s (Code: %d)\n", result3.error, result3.code);
    }
    
    return 0;
}
```

## Best Practices

### 1. **Always Initialize with `{0}`**
```c
// ✅ Correct - prevents garbage values
ErrorOrFloat result = {0};
result.is_error = false;
result.value = 42.0;

// ❌ Wrong - may contain garbage in unused fields
ErrorOrFloat result;
result.is_error = false;
result.value = 42.0;
```

### 2. **Use Meaningful Error Codes and Messages**
```c
// ✅ Good - descriptive and actionable
enum {
    INVALID_EMAIL_FORMAT = 1001,
    PASSWORD_TOO_SHORT = 1002,
    USER_ALREADY_EXISTS = 1003
};

return user_error(INVALID_EMAIL_FORMAT, 
    "Email '%s' is invalid: missing @ symbol", email);

// ❌ Bad - vague and unhelpful
enum { ERROR1 = 1, ERROR2 = 2 };
return user_error(1, "Error");
```

### 3. **Check Errors Before Using Values**
```c
// ✅ Always check first
ErrorOrFloat result = risky_operation();
if (result.is_error) {
    printf("Error: %s\n", result.error);
    return; // Handle error appropriately
}
printf("Success: %.2f\n", result.value); // Safe to use

// ❌ Never use value without checking
ErrorOrFloat result = risky_operation();
printf("Result: %.2f\n", result.value); // May be garbage!
```

### 4. **Use Error Propagation for Chained Operations**
```c
// ✅ Clean error propagation
#define PROPAGATE_ERROR(result) \
    do { if ((result).is_error) return (result); } while(0)

ErrorOrFloat complex_operation() {
    ErrorOrFloat step1 = first_operation();
    PROPAGATE_ERROR(step1);
    
    ErrorOrFloat step2 = second_operation(step1.value);
    PROPAGATE_ERROR(step2);
    
    return step2;
}
```

### 5. **Document Memory Ownership**
```c
typedef struct {
    char *value;        // Caller must free() this when done
    const char *error;  // Static string, do not free
    int code;
    bool is_error;
} ErrorOrString;

// Always provide cleanup functions
void free_string_result(ErrorOrString *result) {
    if (!result->is_error && result->value != NULL) {
        free(result->value);
        result->value = NULL;
    }
}
```

### 6. **Choose the Right Error Type for Your Function**
```c
// ✅ Function returns data - use ErrorOrType
ErrorOrFloat calculate_average(float *numbers, size_t count);

// ✅ Function performs action without returning data - use ErrorOrVoid
ErrorOrVoid save_to_file(const char *filename, const char *data);

// ✅ Function returns optional data - consider ErrorOrType with NULL pattern
ErrorOrString find_user_by_id(int user_id); // Returns empty/NULL string if not found
```

## Common Mistakes to Avoid

### 1. **Forgetting to Initialize**
```c
// ❌ Wrong - creates garbage values
ErrorOrFloat create_error() {
    ErrorOrFloat result;      // Uninitialized!
    result.is_error = true;
    result.error = "Failed";
    // code and value contain garbage
    return result;
}

// ✅ Correct - clean initialization
ErrorOrFloat create_error() {
    ErrorOrFloat result = {0}; // All fields zeroed
    result.is_error = true;
    result.error = "Failed";
    return result;
}
```

### 2. **Using Value Without Checking Error Flag**
```c
// ❌ Dangerous - may use garbage data
float calculate() {
    ErrorOrFloat result = divide(10, 0);
    return result.value; // May be garbage if error occurred!
}

// ✅ Safe - always check first
float calculate() {
    ErrorOrFloat result = divide(10, 0);
    if (result.is_error) {
        printf("Calculation failed: %s\n", result.error);
        return NAN; // Or handle error appropriately
    }
    return result.value;
}
```

### 3. **Memory Leaks with Dynamic Content**
```c
// ❌ Memory leak - forget to free on error
void process_file() {
    ErrorOrString content = read_file("data.txt");
    if (content.is_error) {
        return; // Leaked memory if content.value was allocated!
    }
    use_content(content.value);
    free(content.value);
}

// ✅ Proper cleanup
void process_file() {
    ErrorOrString content = read_file("data.txt");
    if (content.is_error) {
        printf("Error: %s\n", content.error);
    } else {
        use_content(content.value);
    }
    // Clean up regardless of success/failure
    free_string_result(&content);
}
```

### 4. **Inconsistent Error Handling**
```c
// ❌ Inconsistent - some functions use different patterns
int old_function() { return -1; } // Returns error code
ErrorOrFloat new_function();      // Returns union type
void other_function();            // No error reporting at all

// ✅ Consistent - all functions use same pattern
ErrorOrInt old_function_fixed();
ErrorOrFloat new_function();
ErrorOrVoid other_function_fixed(); // Now reports errors properly
```

### 5. **Ignoring ErrorOrVoid Results**
```c
// ❌ Dangerous - ignoring potential errors
save_config("config.txt", data);           // What if it failed?
initialize_system();                       // Did initialization work?

// ✅ Proper error handling
ErrorOrVoid save_result = save_config("config.txt", data);
if (save_result.is_error) {
    printf("Failed to save config: %s\n", save_result.error_msg);
    return; // Handle appropriately
}

ErrorOrVoid init_result = initialize_system();
if (init_result.is_error) {
    printf("System initialization failed: %s\n", init_result.error_msg);
    exit(1); // Critical error
}
```

## Performance Considerations

### 1. **Memory Usage**
The union strategy uses more memory than simple return codes:

```c
// Traditional: 4 bytes (just return code)
int traditional_function();

// Union: ~16-20 bytes (depending on architecture)
typedef struct {
    float value;         // 4 bytes
    const char *error;   // 8 bytes (64-bit pointer)
    int code;           // 4 bytes
    bool is_error;      // 1 byte + 3 bytes padding
} ErrorOrFloat;        // Total: ~16-20 bytes
```

**Trade-off:** Slightly higher memory usage for much better safety and clarity.

### 2. **Performance Impact**
- **Minimal**: Structure copying is fast for small types
- **Return Value Optimization**: Modern compilers optimize struct returns
- **Stack Usage**: Slightly more stack space per function call

### 3. **When to Use Each Approach**

**Use static error messages (less memory):**
```c
typedef struct {
    float value;
    const char *error;  // Points to static string
    int code;
    bool is_error;
} ErrorOrFloat;
```

**Use dynamic error messages (more memory, more flexibility):**
```c
typedef struct {
    float value;
    char error[150];    // Fixed buffer for formatted messages
    int code;
    bool is_error;
} ErrorOrFloat;
```

## Conclusion

The OUI Union Strategy provides a robust, type-safe way to handle errors in C. While it uses slightly more memory than traditional approaches, it offers significant benefits:

- **Safety**: Prevents accessing invalid values
- **Clarity**: Makes error handling explicit and visible
- **Maintainability**: Self-documenting code with clear error messages
- **Composability**: Easy to chain operations and propagate errors
- **Flexibility**: Supports both value-returning functions (`ErrorOrType`) and action-performing functions (`ErrorOrVoid`)

This pattern is particularly valuable in systems programming where reliability is crucial and explicit error management is preferred over exceptions or global error states.

**Key Guidelines:**
- Use `ErrorOrType` for functions that return data
- Use `ErrorOrVoid` for functions that perform actions without returning data
- Always check the `is_error` flag before accessing any value fields
- A function returns either a successful result OR an error, never both

Remember: Explicit error handling leads to more reliable and maintainable code!
