# C Naming Conventions Guide 📋

## Table of Contents
1. [Introduction](#introduction)
2. [Why Naming Conventions Matter](#why-naming-conventions-matter)
3. [Library Namespace Collision Prevention](#library-namespace-collision-prevention)
4. [Constants](#constants)
5. [Global Variables](#global-variables)
6. [Local Variables](#local-variables)
7. [Functions](#functions)
8. [Structs and Types](#structs-and-types)
9. [Macros](#macros)
10. [Complete Examples](#complete-examples)
11. [Common Mistakes and How to Avoid Them](#common-mistakes-and-how-to-avoid-them)
12. [Quick Reference Chart](#quick-reference-chart)

## Introduction

Welcome to the C Naming Conventions Guide! This document will teach you how to name your code elements consistently and professionally. If you're new to C programming, don't worry - we'll explain everything step by step with plenty of examples.

**What are naming conventions?** They are rules that tell us how to name things in our code (like variables, functions, constants, etc.) so that:
- Our code is easy to read and understand
- Different programmers can work together on the same project
- We avoid conflicts between different libraries
- Our code looks professional and maintainable

## Why Naming Conventions Matter

### 🤔 The Problem Without Conventions

Imagine you're working on a project and you see code like this:

```c
// BAD: Inconsistent and confusing naming
int X;                    // What does X represent?
float SpeedOfCar;         // Mixed case style
char* user_name;          // Snake case
int getUserAge();         // Camel case
#define max 100           // Lowercase constant
int list();               // Generic name that could conflict
```

**Problems with this code:**
- Hard to understand what each thing does
- Mixing different naming styles makes it look unprofessional
- Generic names like `list()` might conflict with other libraries
- `X` doesn't tell us anything about its purpose

### ✅ The Solution With Conventions

Now look at the same code following our conventions:

```c
// GOOD: Consistent and clear naming
int current_speed_kmh;              // Clear variable name
float car_max_speed;                // Consistent snake_case
char* user_name;                    // Consistent style
int get_user_age();                 // Clear function purpose
#define MYLIB_MAX_SPEED 100         // Clear constant with namespace
int mylib_list_files();             // Namespaced to avoid conflicts
```

**Benefits of this approach:**
- Easy to understand what each element does
- Consistent style throughout the codebase
- No conflicts with other libraries
- Professional and maintainable code

## Library Namespace Collision Prevention

### 🚨 What is Namespace Collision?

**Namespace collision** happens when two different libraries use the same name for different things. This causes conflicts and can break your program.

**Example of the problem:**
```c
// Library A defines:
int list_files();

// Library B also defines:
int list_files();

// When you include both libraries:
#include "library_a.h"
#include "library_b.h"

// Compiler error: Which list_files() do you want?
list_files(); // ERROR: Ambiguous!
```

### ✅ The Solution: Library Prefixes

**All global elements** (constants, function names, global variables, macros) **MUST** have the library prefix in their name.

**What are global elements?**
- **Functions**: Can be called from anywhere in your program
- **Global variables**: Variables declared outside of any function
- **Constants**: `#define` values and `const` variables
- **Macros**: `#define` preprocessor directives
- **Enums**: Enumeration values

### 📝 How to Choose a Library Prefix

1. **Use 2-4 letters** that represent your library name
2. **Make it unique** - check that other libraries don't use it
3. **Keep it short** but meaningful

**Examples:**
- `dtw` for [DoTheWorld](https://github.com/OUIsolutions/DoTheWorld) library
- `cweb` for [CWebStudio](https://github.com/OUIsolutions/CWebStudio) library
- `json` for a JSON parsing library
- `db` for a database library

### 🔧 Function Examples with Prefixes

```c
// DoTheWorld library functions (prefix: dtw)
dtw_list_files_recursively(const char *path);
dtw_create_directory(const char *path);
dtw_file_exists(const char *filename);
dtw_read_file_content(const char *filename);

// CWebStudio library functions (prefix: cweb or structured)
CwebHttpRequest_get_param(CwebHttpRequest *request, const char *name);
CwebHttpRequest_get_method(CwebHttpRequest *request);
CwebHttpResponse_set_status(CwebHttpResponse *response, int status);
CwebHttpResponse_add_header(CwebHttpResponse *response, const char *key, const char *value);
```

**Why this works:**
- `dtw_list_files_recursively()` clearly belongs to DoTheWorld library
- `CwebHttpRequest_get_param()` clearly belongs to CWebStudio library
- No chance of collision between different libraries

### 🔧 Constants and Macros with Prefixes

```c
// DoTheWorld library constants
#define DTW_MAX_PATH_LENGTH 4096
#define DTW_FILE_READ_MODE "r"
#define DTW_SUCCESS 0
#define DTW_ERROR -1

// CWebStudio library constants
#define CWEB_OK 200
#define CWEB_BAD_REQUEST 400
#define CWEB_NOT_FOUND 404
#define CWEB_INTERNAL_ERROR 500

// JSON library constants
#define JSON_PARSE_SUCCESS 0
#define JSON_PARSE_ERROR -1
#define JSON_MAX_DEPTH 100
```

### 🔧 Global Variables with Prefixes

```c
// DoTheWorld library global variables
char dtw_current_directory[DTW_MAX_PATH_LENGTH];
int dtw_debug_enabled;
FILE* dtw_log_file;

// CWebStudio library global variables
int cweb_server_port;
char* cweb_server_host;
bool cweb_logging_enabled;
```

## Constants

### 📐 What are Constants?

Constants are values that **never change** during program execution. In C, we create them using:
- `#define` preprocessor directives
- `const` keyword with variables
- `enum` values

### 📝 Naming Rule: UPPER_SNAKE_CASE

**All constants must use UPPER_SNAKE_CASE with library prefix**

**Format:** `LIBRARY_CONSTANT_NAME`

### 🔧 Examples

```c
// ✅ GOOD: Proper constant naming
#define MYLIB_MAX_USERS 1000
#define MYLIB_DEFAULT_TIMEOUT 30
#define MYLIB_ERROR_CODE -1
#define MYLIB_SUCCESS_CODE 0
#define MYLIB_PI 3.14159
#define MYLIB_MAX_BUFFER_SIZE 4096

// Using const keyword
const int MYLIB_MAX_CONNECTIONS = 100;
const char* MYLIB_VERSION = "1.0.0";
const float MYLIB_DEFAULT_SPEED = 25.5f;

// Enums
enum {
    MYLIB_STATE_IDLE,
    MYLIB_STATE_RUNNING,
    MYLIB_STATE_STOPPED,
    MYLIB_STATE_ERROR
};
```

```c
// ❌ BAD: Wrong constant naming
#define max_users 1000          // Missing prefix, wrong case
#define MaxUsers 1000           // Missing prefix, wrong case  
#define mylib_max_users 1000    // Wrong case
#define MAX_USERS 1000          // Missing prefix
```

### 🎯 When to Use Constants

```c
// Instead of magic numbers in your code:
// ❌ BAD
if (user_count > 500) {
    return -1;
}

// ✅ GOOD  
#define MYLIB_MAX_USERS 500
if (user_count > MYLIB_MAX_USERS) {
    return MYLIB_ERROR_TOO_MANY_USERS;
}
```

**Benefits:**
- Easy to change values in one place
- Code is self-documenting
- Prevents typos in repeated values
- Makes code more maintainable

## Global Variables

### 🌍 What are Global Variables?

Global variables are variables declared **outside of any function** that can be accessed from anywhere in your program.

### 📝 Naming Rule: lower_snake_case with prefix

**Format:** `library_variable_name`

### 🔧 Examples

```c
// ✅ GOOD: Proper global variable naming
int mylib_debug_mode;           // Debug flag
char* mylib_config_file_path;   // Configuration file path  
FILE* mylib_log_file;           // Log file handle
int mylib_connection_count;     // Active connections
float mylib_current_version;    // Current version number
bool mylib_is_initialized;      // Initialization flag

// Real-world examples from libraries:
int dtw_verbose_mode;           // DoTheWorld debug mode
char* cweb_server_root;         // CWebStudio server root directory
int json_parse_depth;           // JSON parser current depth
```

```c
// ❌ BAD: Wrong global variable naming
int debugMode;                  // Missing prefix, wrong case
int Debug_Mode;                 // Missing prefix, wrong case
int debug_mode;                 // Missing prefix
int MYLIB_DEBUG_MODE;           // Wrong case (this is constant style)
```

### ⚠️ Global Variables Best Practices

```c
// 1. Always initialize global variables
int mylib_connection_count = 0;
bool mylib_is_running = false;

// 2. Use descriptive names
// ❌ BAD
int mylib_x;        // What is x?
int mylib_temp;     // What kind of temp?

// ✅ GOOD  
int mylib_active_connections;     // Clear purpose
int mylib_cpu_temperature;        // Specific type of temperature

// 3. Consider using functions instead of direct access
// Instead of: mylib_debug_mode = 1;
// Provide: mylib_enable_debug_mode();
void mylib_enable_debug_mode() {
    mylib_debug_mode = 1;
}
```

## Local Variables

### 🏠 What are Local Variables?

Local variables are variables declared **inside functions** that can only be used within that function.

### 📝 Naming Rule: lower_snake_case (no prefix needed)

Since local variables are only visible inside functions, they don't need library prefixes.

**Format:** `variable_name`

### 🔧 Examples

```c
void calculate_area() {
    // ✅ GOOD: Clear local variable names
    float rectangle_width;        // Width of rectangle
    float rectangle_height;       // Height of rectangle  
    float total_area;            // Calculated area
    int user_input;              // Input from user
    char file_name[256];         // Name of file to process
    bool is_valid_input;         // Input validation flag
    int loop_counter;            // Loop iteration counter
    
    // Short variables for simple loops are OK
    int i, j;                    // Loop counters
    char c;                      // Single character
    
    // Use the variables...
    rectangle_width = 5.0f;
    rectangle_height = 3.0f;
    total_area = rectangle_width * rectangle_height;
}
```

```c
// ❌ BAD: Poor local variable naming
void calculate_area() {
    float w;                     // Too short, unclear
    float RectangleHeight;       // Wrong case style
    float Total_Area;            // Wrong case style  
    int x;                       // Meaningless name
    char fn[256];                // Abbreviation unclear
    bool flag;                   // Generic name
}
```

### 🎯 Local Variable Best Practices

```c
// 1. Use descriptive names that explain purpose
// ❌ BAD
int calc(int a, int b) {
    int temp = a * b;
    return temp;
}

// ✅ GOOD
int calculate_rectangle_area(int width, int height) {
    int total_area = width * height;
    return total_area;
}

// 2. Initialize variables when you declare them
// ✅ GOOD
int user_age = 0;
bool is_valid = false;
char* file_path = NULL;

// 3. Use specific names instead of generic ones
// ❌ BAD
char* data;
int value;
float number;

// ✅ GOOD  
char* user_name;
int student_grade;
float account_balance;
```

## Functions

### ⚙️ What are Functions?

Functions are blocks of code that perform specific tasks. They can be called from other parts of your program.

### 📝 Naming Rules

The naming rule depends on whether your function is part of OOP emulation or not:

1. **OOP Emulation Functions**: Follow [POO_emulation.md](POO_emulation.md) conventions
2. **Regular Functions**: Use `library_prefix_function_name` in lower_snake_case

### 🔧 Regular Functions (Non-OOP)

```c
// ✅ GOOD: Regular function naming
int dtw_create_directory(const char* path);
char* dtw_read_file_content(const char* filename);
bool dtw_file_exists(const char* filename);
void dtw_initialize_library();
int dtw_get_file_size(const char* filename);

// Math library example
float math_calculate_distance(float x1, float y1, float x2, float y2);
int math_find_greatest_common_divisor(int a, int b);
bool math_is_prime_number(int number);

// String utility library
char* str_trim_whitespace(char* input);
int str_count_occurrences(const char* text, char character);
bool str_starts_with(const char* text, const char* prefix);
```

### 🔧 OOP Emulation Functions

When you're following the OOP emulation pattern, use these conventions:

```c
// Constructor functions (create new objects)
Car* newCar(const char* model, int year, float price);
Student* newStudent(const char* name, int age);
Database* newDatabase(const char* connection_string);

// Method functions (operate on objects)  
void Car_set_speed(Car* self, float speed);
float Car_get_speed(Car* self);
void Car_start_engine(Car* self);
bool Car_is_running(Car* self);

void Student_set_grade(Student* self, float grade);
float Student_calculate_gpa(Student* self);
void Student_add_course(Student* self, const char* course_name);

// Destructor functions (clean up objects)
void Car_free(Car* self);
void Student_free(Student* self);
void Database_free(Database* self);
```

**See [POO_emulation.md](POO_emulation.md) for complete details on OOP function naming.**

### 🎯 Function Naming Best Practices

```c
// 1. Use verbs that describe what the function does
// ✅ GOOD
bool mylib_validate_email(const char* email);      // Validates something
char* mylib_convert_to_uppercase(char* text);      // Converts something  
void mylib_print_error_message(const char* msg);   // Prints something
int mylib_count_words(const char* text);           // Counts something

// 2. Be specific about parameters and return values
// ❌ BAD
int mylib_process(void* data);           // What kind of processing?

// ✅ GOOD
int mylib_compress_image_data(unsigned char* image_data, int size);

// 3. Use consistent verb patterns
// ✅ GOOD - Consistent "get/set" pattern
int mylib_get_user_age(int user_id);
void mylib_set_user_age(int user_id, int age);

char* mylib_get_config_value(const char* key);
void mylib_set_config_value(const char* key, const char* value);

// 4. Indicate if function can fail
// ✅ GOOD - Return codes indicate success/failure
int mylib_connect_to_server(const char* host, int port);  // Returns 0 on success
bool mylib_file_exists(const char* filename);             // Returns true/false
char* mylib_read_file(const char* filename);              // Returns NULL on failure
```

## Structs and Types

### 🏗️ What are Structs and Types?

Structs are user-defined data types that group related data together. Types are names we give to data structures.

### 📝 Naming Rule: PascalCase with optional prefix

**Format:** `LibraryStructName` or `StructName`

### 🔧 Examples

```c
// ✅ GOOD: Struct naming

// Simple structs without library prefix (for small projects)
typedef struct {
    char* name;
    int age;
    float grade;
} Student;

typedef struct {
    float x;
    float y;
    float z;
} Point3D;

// Structs with library prefix (for libraries)  
typedef struct {
    char* model;
    int year;
    float price;
    bool is_running;
} CwebCar;

typedef struct {
    char* host;
    int port;
    bool is_connected;
    int timeout_seconds;
} DtwConnection;

// Complex nested structs
typedef struct {
    char* first_name;
    char* last_name;
    int age;
} PersonInfo;

typedef struct {
    PersonInfo personal_info;
    char* employee_id;
    float salary;
    int years_of_service;
} Employee;
```

### 🔧 Enum Naming

```c
// ✅ GOOD: Enum naming
typedef enum {
    MYLIB_STATUS_SUCCESS,
    MYLIB_STATUS_ERROR,
    MYLIB_STATUS_PENDING,
    MYLIB_STATUS_TIMEOUT
} MylibStatus;

typedef enum {
    CAR_TYPE_SEDAN,
    CAR_TYPE_SUV,  
    CAR_TYPE_TRUCK,
    CAR_TYPE_MOTORCYCLE
} CarType;

// Usage
MylibStatus result = mylib_connect_to_server("localhost", 8080);
if (result == MYLIB_STATUS_SUCCESS) {
    printf("Connected successfully!\n");
}
```

### 🔧 Function Pointer Types

```c
// ✅ GOOD: Function pointer naming
typedef int (*MylibCompareFn)(const void* a, const void* b);
typedef void (*MylibErrorCallback)(const char* error_message);
typedef bool (*MylibValidatorFn)(const char* input);

// Usage
void mylib_sort_array(void* array, size_t count, MylibCompareFn compare_func);
void mylib_set_error_handler(MylibErrorCallback callback);
```

## Macros

### 🔧 What are Macros?

Macros are preprocessor directives that replace text in your code before compilation. They're defined with `#define`.

### 📝 Naming Rule: UPPER_SNAKE_CASE with prefix

**Format:** `LIBRARY_MACRO_NAME`

### 🔧 Simple Value Macros

```c
// ✅ GOOD: Simple macro naming
#define MYLIB_VERSION "1.2.3"
#define MYLIB_MAX_BUFFER_SIZE 4096
#define MYLIB_DEFAULT_TIMEOUT 30
#define MYLIB_PI 3.14159265359
#define MYLIB_DEBUG_ENABLED 1

// File operation macros
#define DTW_READ_MODE "r"
#define DTW_WRITE_MODE "w"
#define DTW_APPEND_MODE "a"

// HTTP status codes
#define CWEB_HTTP_OK 200
#define CWEB_HTTP_NOT_FOUND 404
#define CWEB_HTTP_SERVER_ERROR 500
```

### 🔧 Function-like Macros

```c
// ✅ GOOD: Function-like macro naming
#define MYLIB_MAX(a, b) ((a) > (b) ? (a) : (b))
#define MYLIB_MIN(a, b) ((a) < (b) ? (a) : (b))
#define MYLIB_ABS(x) ((x) < 0 ? -(x) : (x))
#define MYLIB_SQUARE(x) ((x) * (x))

// Debugging macros
#define MYLIB_DEBUG_PRINT(msg) \
    do { \
        if (MYLIB_DEBUG_ENABLED) { \
            printf("[DEBUG] %s\n", msg); \
        } \
    } while(0)

#define MYLIB_ASSERT(condition, message) \
    do { \
        if (!(condition)) { \
            fprintf(stderr, "Assertion failed: %s\n", message); \
            exit(1); \
        } \
    } while(0)

// Memory allocation helper
#define MYLIB_MALLOC(type, count) \
    ((type*)malloc(sizeof(type) * (count)))

#define MYLIB_FREE_AND_NULL(ptr) \
    do { \
        free(ptr); \
        (ptr) = NULL; \
    } while(0)
```

### ⚠️ Macro Best Practices

```c
// 1. Always wrap multi-token replacements in parentheses
// ❌ BAD
#define MYLIB_DOUBLE(x) x * 2        // Can cause issues: MYLIB_DOUBLE(3+4) = 3+4*2 = 11

// ✅ GOOD
#define MYLIB_DOUBLE(x) ((x) * 2)    // Correct: MYLIB_DOUBLE(3+4) = ((3+4)*2) = 14

// 2. Use do-while(0) for multi-statement macros
// ✅ GOOD
#define MYLIB_LOG_ERROR(msg) \
    do { \
        fprintf(stderr, "Error: %s\n", msg); \
        fflush(stderr); \
    } while(0)

// 3. Avoid side effects in macros
// ❌ BAD
#define MYLIB_NEXT(x) (++x)          // Changes x twice: MYLIB_NEXT(i) + MYLIB_NEXT(i)

// ✅ GOOD - Use inline functions instead
static inline int mylib_next(int x) {
    return x + 1;
}
```

## Complete Examples

### 🎯 Example 1: Math Library

```c
// mathlib.h - A complete example following naming conventions

#ifndef MATHLIB_H
#define MATHLIB_H

#include <stdbool.h>

// Constants
#define MATHLIB_PI 3.14159265359
#define MATHLIB_E 2.71828182846
#define MATHLIB_MAX_ITERATIONS 1000
#define MATHLIB_EPSILON 0.000001

// Error codes
#define MATHLIB_SUCCESS 0
#define MATHLIB_ERROR_INVALID_INPUT -1
#define MATHLIB_ERROR_DIVISION_BY_ZERO -2

// Macros
#define MATHLIB_MAX(a, b) ((a) > (b) ? (a) : (b))
#define MATHLIB_MIN(a, b) ((a) < (b) ? (a) : (b))
#define MATHLIB_ABS(x) ((x) < 0 ? -(x) : (x))
#define MATHLIB_SQUARE(x) ((x) * (x))

// Types
typedef struct {
    float x;
    float y;
} MathPoint;

typedef struct {
    float real;
    float imaginary;
} MathComplex;

typedef enum {
    MATHLIB_ANGLE_RADIANS,
    MATHLIB_ANGLE_DEGREES
} MathAngleUnit;

// Global variables
extern bool mathlib_debug_enabled;
extern int mathlib_precision_digits;

// Basic math functions
float mathlib_calculate_distance(float x1, float y1, float x2, float y2);
float mathlib_calculate_circle_area(float radius);
float mathlib_calculate_triangle_area(float base, float height);
bool mathlib_is_prime_number(int number);
int mathlib_calculate_factorial(int n);
float mathlib_power(float base, float exponent);
float mathlib_square_root(float number);

// Trigonometric functions
float mathlib_sine(float angle, MathAngleUnit unit);
float mathlib_cosine(float angle, MathAngleUnit unit);
float mathlib_tangent(float angle, MathAngleUnit unit);
float mathlib_convert_degrees_to_radians(float degrees);
float mathlib_convert_radians_to_degrees(float radians);

// Point operations (OOP-style)
MathPoint* newMathPoint(float x, float y);
void MathPoint_set_coordinates(MathPoint* self, float x, float y);
float MathPoint_get_x(MathPoint* self);
float MathPoint_get_y(MathPoint* self);
float MathPoint_calculate_distance_to(MathPoint* self, MathPoint* other);
void MathPoint_move(MathPoint* self, float dx, float dy);
void MathPoint_free(MathPoint* self);

// Complex number operations
MathComplex* newMathComplex(float real, float imaginary);
MathComplex* MathComplex_add(MathComplex* a, MathComplex* b);
MathComplex* MathComplex_multiply(MathComplex* a, MathComplex* b);
float MathComplex_get_magnitude(MathComplex* self);
void MathComplex_free(MathComplex* self);

// Utility functions
void mathlib_enable_debug_mode();
void mathlib_disable_debug_mode();
void mathlib_set_precision(int digits);
int mathlib_get_version_major();
int mathlib_get_version_minor();
const char* mathlib_get_version_string();

#endif // MATHLIB_H
```

### 🎯 Example 2: File Processing Library

```c
// filelib.h - File processing library example

#ifndef FILELIB_H
#define FILELIB_H

#include <stdio.h>
#include <stdbool.h>

// Constants
#define FILELIB_MAX_PATH_LENGTH 4096
#define FILELIB_MAX_FILENAME_LENGTH 256
#define FILELIB_BUFFER_SIZE 8192
#define FILELIB_MAX_FILES_PER_DIRECTORY 10000

// Error codes
#define FILELIB_SUCCESS 0
#define FILELIB_ERROR_FILE_NOT_FOUND -1
#define FILELIB_ERROR_PERMISSION_DENIED -2
#define FILELIB_ERROR_DISK_FULL -3
#define FILELIB_ERROR_INVALID_PATH -4

// File types
typedef enum {
    FILELIB_TYPE_UNKNOWN,
    FILELIB_TYPE_TEXT,
    FILELIB_TYPE_BINARY,
    FILELIB_TYPE_IMAGE,
    FILELIB_TYPE_EXECUTABLE
} FileType;

// File information structure
typedef struct {
    char file_path[FILELIB_MAX_PATH_LENGTH];
    char file_name[FILELIB_MAX_FILENAME_LENGTH];
    long file_size_bytes;
    FileType file_type;
    bool is_readable;
    bool is_writable;
    bool is_executable;
    time_t creation_time;
    time_t modification_time;
} FileInfo;

// Directory listing structure  
typedef struct {
    FileInfo* files;
    int file_count;
    int max_capacity;
} DirectoryListing;

// Global configuration
extern char filelib_default_directory[FILELIB_MAX_PATH_LENGTH];
extern bool filelib_verbose_logging;
extern int filelib_default_buffer_size;

// Basic file operations
bool filelib_file_exists(const char* file_path);
bool filelib_directory_exists(const char* directory_path);
int filelib_create_file(const char* file_path);
int filelib_create_directory(const char* directory_path);
int filelib_delete_file(const char* file_path);
int filelib_delete_directory(const char* directory_path);
int filelib_copy_file(const char* source_path, const char* destination_path);
int filelib_move_file(const char* source_path, const char* destination_path);

// File reading/writing
char* filelib_read_entire_file(const char* file_path);
int filelib_write_string_to_file(const char* file_path, const char* content);
int filelib_append_string_to_file(const char* file_path, const char* content);
long filelib_get_file_size(const char* file_path);

// File information
FileInfo* newFileInfo(const char* file_path);
FileType FileInfo_get_type(FileInfo* self);
bool FileInfo_is_text_file(FileInfo* self);
bool FileInfo_is_image_file(FileInfo* self);
const char* FileInfo_get_extension(FileInfo* self);
void FileInfo_free(FileInfo* self);

// Directory operations
DirectoryListing* newDirectoryListing(const char* directory_path);
int DirectoryListing_get_file_count(DirectoryListing* self);
FileInfo* DirectoryListing_get_file_at_index(DirectoryListing* self, int index);
void DirectoryListing_sort_by_name(DirectoryListing* self);
void DirectoryListing_sort_by_size(DirectoryListing* self);
void DirectoryListing_sort_by_date(DirectoryListing* self);
void DirectoryListing_free(DirectoryListing* self);

// Utility functions
void filelib_set_default_directory(const char* directory_path);
const char* filelib_get_default_directory();
void filelib_enable_verbose_logging();
void filelib_disable_verbose_logging();
const char* filelib_get_error_message(int error_code);

#endif // FILELIB_H
```

## Common Mistakes and How to Avoid Them

### ❌ Mistake 1: Inconsistent Naming Styles

```c
// BAD: Mixing different naming conventions
int getUserAge();           // camelCase
int get_user_name();        // snake_case  
int GetUserEmail();         // PascalCase
#define max_size 100        // lowercase constant
```

```c
// ✅ GOOD: Consistent naming throughout
int mylib_get_user_age();
int mylib_get_user_name();
int mylib_get_user_email();
#define MYLIB_MAX_SIZE 100
```

### ❌ Mistake 2: Missing Library Prefixes

```c
// BAD: No prefixes, potential conflicts
int connect_to_server();
#define MAX_CONNECTIONS 100
int error_code;
```

```c
// ✅ GOOD: Clear prefixes prevent conflicts
int mylib_connect_to_server();
#define MYLIB_MAX_CONNECTIONS 100
int mylib_error_code;
```

### ❌ Mistake 3: Unclear Abbreviations

```c
// BAD: Cryptic abbreviations
int calc_usr_stats();       // What does this calculate?
char* get_cfg_val();        // What configuration?
bool chk_perm();           // Check what permission?
```

```c
// ✅ GOOD: Clear, descriptive names
int mylib_calculate_user_statistics();
char* mylib_get_configuration_value();
bool mylib_check_file_permission();
```

### ❌ Mistake 4: Wrong Case for Different Element Types

```c
// BAD: Wrong cases
#define mylib_max_size 100          // Constant should be UPPER_CASE
int MYLIB_USER_COUNT;               // Variable should be lower_case
void MYLIB_PROCESS_DATA();          // Function should be lower_case
```

```c
// ✅ GOOD: Correct cases for each type
#define MYLIB_MAX_SIZE 100          // Constants: UPPER_SNAKE_CASE
int mylib_user_count;               // Variables: lower_snake_case
void mylib_process_data();          // Functions: lower_snake_case
```

### ❌ Mistake 5: Generic Names That Don't Explain Purpose

```c
// BAD: Generic, meaningless names
int data;
char* buffer;
void process();
bool flag;
int temp;
```

```c
// ✅ GOOD: Specific, meaningful names
int user_age_years;
char* file_content_buffer;
void mylib_process_payment_request();
bool is_connection_active;
int temporary_user_id;
```

## Quick Reference Chart

| Element Type | Naming Convention | Library Prefix | Examples |
|-------------|------------------|----------------|----------|
| **Constants** | `UPPER_SNAKE_CASE` | ✅ Required | `MYLIB_MAX_SIZE`, `DTW_SUCCESS` |
| **Global Variables** | `lower_snake_case` | ✅ Required | `mylib_debug_mode`, `cweb_server_port` |
| **Local Variables** | `lower_snake_case` | ❌ Not needed | `user_name`, `file_size`, `loop_counter` |
| **Functions (Regular)** | `lower_snake_case` | ✅ Required | `mylib_read_file()`, `dtw_create_dir()` |
| **Functions (OOP)** | Special rules | ✅ See OOP guide | `newCar()`, `Car_set_speed()` |
| **Structs/Types** | `PascalCase` | ⚠️ Optional | `Student`, `MylibConnection` |
| **Macros** | `UPPER_SNAKE_CASE` | ✅ Required | `MYLIB_MAX()`, `DTW_DEBUG_PRINT()` |
| **Enums** | `UPPER_SNAKE_CASE` | ✅ Required | `MYLIB_STATUS_SUCCESS` |

### 🔍 Checklist for Code Review

Before submitting your code, check:

- [ ] All global functions have library prefix
- [ ] All constants use UPPER_SNAKE_CASE with prefix
- [ ] All global variables use lower_snake_case with prefix
- [ ] All local variables use clear, descriptive names
- [ ] All macros use UPPER_SNAKE_CASE with prefix
- [ ] Struct names use PascalCase
- [ ] No abbreviations that aren't immediately clear
- [ ] No generic names like `data`, `temp`, `flag`
- [ ] OOP functions follow the OOP emulation guide
- [ ] All names are in English
- [ ] Naming style is consistent throughout the file

### 📚 Additional Resources

- [POO_emulation.md](POO_emulation.md) - Complete guide to OOP-style function naming
- [Error_handling_internal_error_strategy.md](Error_handling_internal_error_strategy.md) - Error handling conventions
- [Error_handling_union_strategy.md](Error_handling_union_strategy.md) - Union-based error handling

**Remember:** Good naming conventions make your code easier to read, maintain, and debug. Take the time to choose clear, descriptive names - your future self and your teammates will thank you! 🎉
