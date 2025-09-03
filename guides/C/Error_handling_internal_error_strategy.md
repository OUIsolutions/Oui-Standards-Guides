# OUI Internal Error Strategy (Legacy)

## Table of Contents
1. [Introduction](#introduction)
2. [⚠️ Important Notice](#️-important-notice)
3. [Why Was This Strategy Used?](#why-was-this-strategy-used)
4. [Basic Concept](#basic-concept)
5. [Core Components](#core-components)
6. [Complete Example Implementation](#complete-example-implementation)
7. [How to Use This Pattern](#how-to-use-this-pattern)
8. [Error Management Functions](#error-management-functions)
9. [Best Practices for Legacy Code](#best-practices-for-legacy-code)
10. [Problems with This Approach](#problems-with-this-approach)
11. [Migration Guide](#migration-guide)
12. [When You Must Use This Pattern](#when-you-must-use-this-pattern)

## Introduction

This document describes the **Internal Error Strategy** that was used in many older OUI projects. This pattern stores error information directly inside the data structure, allowing each object to maintain its own error state.

**Key Principle:** Each object carries its own error flag and error message, and operations on the object automatically fail if it's already in an error state.

## ⚠️ Important Notice

**🚨 THIS IS A LEGACY PATTERN 🚨**

**For NEW projects:** Use the [Error Handling Union Strategy](Error_handling_union_strategy.md) instead!

**Only use this pattern if:**
- ✅ You're maintaining existing legacy code
- ✅ You're working with an established codebase that already uses this pattern
- ✅ You need to extend functionality in a legacy system

**DO NOT use for new projects because:**
- ❌ The [Union Strategy](Error_handling_union_strategy.md) is safer and more modern
- ❌ This pattern has several inherent problems (explained below)
- ❌ It's harder to compose operations safely

## Why Was This Strategy Used?

### Historical Context

Before the Union Strategy was developed, OUI projects needed a way to handle errors in C that was:
- **Object-oriented**: Each data structure manages its own state
- **Persistent**: Error state survives across multiple function calls
- **Self-contained**: No global error variables needed

### The Problem It Solved

```c
// Traditional C - global error state is problematic
int global_errno = 0;

void risky_operation1() {
    global_errno = 42; // Set error
}

void risky_operation2() {
    global_errno = 0;  // Accidentally clears previous error!
}

// You lost the first error!
```

### How Internal Error Strategy Helped

```c
// Internal Error Strategy - each object has its own error state
StringArray *array = newStringArray();
StringArray_append(array, "hello");      // Success
StringArray_set_value(array, 999, "x");  // Error - sets internal error flag
StringArray_append(array, "world");      // Automatically fails due to previous error

// Error is preserved until explicitly cleared
if (StringArray_is_error(array)) {
    printf("Error: %s
", array->error); // Shows the index out of bounds error
}
```

## Basic Concept

The Internal Error Strategy embeds error handling directly into your data structures:

```c
typedef struct {
    // Error handling fields (always include these)
    char error[100];     // Human-readable error message
    bool is_error;      // Flag: true = object is in error state
    
    // Your actual data fields
    int size;           // Example: number of elements
    char **strings;     // Example: array of strings
} StringArray;
```

**How it works:**
1. **Normal State**: `is_error = false`, operations succeed
2. **Error State**: `is_error = true`, all operations automatically fail
3. **Error Recovery**: Call error clearing function to reset state

## Core Components

### 1. Error State Fields

Every structure using this pattern must include:

```c
typedef struct {
    char error[100];    // Fixed-size buffer for error messages
    bool is_error;     // Boolean flag indicating error state
    
    // ... your data fields here ...
} YourStructName;
```

**Field Details:**
- `error[100]`: Stores human-readable error description
- `is_error`: Boolean flag - `true` means object is in error state

### 2. Error State Checking

Before every operation, check if object is already in error state:

```c
void YourStruct_operation(YourStructName *self) {
    // 1. Validate parameters
    if (!self) {
        return; // Can't set error on NULL pointer
    }
    
    // 2. Check if already in error state
    if (self->is_error) {
        return; // Fail silently - already in error state
    }
    
    // 3. Perform your operation...
    // 4. Set error state if something goes wrong
}
```

### 3. Error Setting Pattern

When an error occurs, set both the flag and message:

```c
void YourStruct_risky_operation(YourStructName *self, int index) {
    if (!self) return;
    if (self->is_error) return; // Already in error state
    
    if (index < 0 || index >= self->size) {
        // Set error state
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Index %d out of bounds (size: %d)", index, self->size);
        return;
    }
    
    // Continue with normal operation...
}
```

### 4. Error Management Functions

Every structure needs these utility functions:

```c
// Check if object is in error state
bool YourStruct_is_error(YourStructName *self) {
    if (!self) return true;  // NULL pointer is considered an error
    return self->is_error;
}

// Clear error state and reset object to normal operation
void YourStruct_clear_error(YourStructName *self) {
    if (!self) return;
    self->is_error = false;
    self->error[0] = '\0';  // Clear error message
}

// Get current error message
const char* YourStruct_get_error(YourStructName *self) {
    if (!self) return "Object is NULL";
    if (!self->is_error) return "No error";
    return self->error;
}
```

## Complete Example Implementation

Here's a complete, heavily documented implementation of the StringArray using the Internal Error Strategy:

```c
#include <stdlib.h>
#include <stdbool.h>
#include <stdio.h>
#include <string.h>

// ================================
// STRUCTURE DEFINITION
// ================================

typedef struct {
    // Error handling fields (ALWAYS INCLUDE THESE)
    char error[100];    // Human-readable error message
    bool is_error;     // Flag: true = error state, false = normal state
    
    // Actual data fields
    int size;          // Number of strings currently stored
    char **strings;    // Dynamic array of string pointers
} StringArray;

// ================================
// FUNCTION DECLARATIONS
// ================================

// Constructor and destructor
StringArray* newStringArray();
void StringArray_free(StringArray *self);

// Core operations
void StringArray_append(StringArray *self, const char *string);
void StringArray_set_value(StringArray *self, int index, const char *value);
void StringArray_pop(StringArray *self, int position);
int StringArray_find_position(StringArray *self, const char *string);

// Advanced operations
void StringArray_merge(StringArray *self, StringArray *other);
StringArray* StringArray_clone(StringArray *self);
void StringArray_append_getting_ownership(StringArray *self, char *string);

// Utility operations
void StringArray_represent(StringArray *self);

// Error management (REQUIRED FOR THIS PATTERN)
bool StringArray_is_error(StringArray *self);
void StringArray_clear_error(StringArray *self);
const char* StringArray_get_error(StringArray *self);

// ================================
// IMPLEMENTATION
// ================================

/**
 * Constructor: Creates a new StringArray
 * Returns: Pointer to new StringArray, or NULL if memory allocation fails
 */
StringArray* newStringArray() {
    // Allocate memory for the structure
    StringArray *self = (StringArray*)malloc(sizeof(StringArray));
    if (!self) {
        return NULL; // Memory allocation failed
    }
    
    // Initialize error state to "no error"
    self->size = 0;
    self->is_error = false;
    self->error[0] = '\0'; // Empty error message
    
    // Allocate initial memory for strings array
    self->strings = (char**)malloc(sizeof(char*));
    if (!self->strings) {
        free(self); // Clean up partial allocation
        return NULL;
    }
    
    return self;
}

/**
 * Find the position of a string in the array
 * Returns: Index if found, -1 if not found or error
 */
int StringArray_find_position(StringArray *self, const char *string) {
    // Parameter validation
    if (!self || !string) {
        return -1;
    }
    
    // Check if object is in error state
    if (self->is_error) {
        return -1; // Cannot operate on object in error state
    }
    
    // Search for the string
    for (int i = 0; i < self->size; i++) {
        if (strcmp(self->strings[i], string) == 0) {
            return i; // Found at index i
        }
    }
    
    return -1; // Not found
}

/**
 * Set the value at a specific index
 * This demonstrates the error setting pattern
 */
void StringArray_set_value(StringArray *self, int index, const char *value) {
    // 1. Parameter validation
    if (!self || !value) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters: self or value is NULL");
        }
        return;
    }
    
    // 2. Check if already in error state
    if (self->is_error) {
        return; // Fail silently - object already in error state
    }
    
    // 3. Validate index bounds
    if (index >= self->size || index < 0) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Index %d out of bounds (valid range: 0-%d)", index, self->size - 1);
        return;
    }
    
    // 4. Attempt to resize memory for new string
    int new_size = strlen(value);
    char *new_string = (char*)realloc(self->strings[index], new_size + 1);
    if (!new_string) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Failed to allocate %d bytes for string at index %d", new_size + 1, index);
        return;
    }
    
    // 5. Success - update the string
    self->strings[index] = new_string;
    strcpy(self->strings[index], value);
}

/**
 * Append a string to the array (makes a copy)
 * This is the most commonly used function
 */
void StringArray_append(StringArray *self, const char *string) {
    // 1. Parameter validation
    if (!self || !string) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters: self or string is NULL");
        }
        return;
    }
    
    // 2. Check error state
    if (self->is_error) {
        return; // Already in error state
    }
    
    // 3. Expand the strings array
    char **new_strings = (char**)realloc(self->strings, (self->size + 1) * sizeof(char*));
    if (!new_strings) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Failed to allocate memory for %d string pointers", self->size + 1);
        return;
    }
    self->strings = new_strings;
    
    // 4. Create a copy of the string
    self->strings[self->size] = strdup(string);
    if (!self->strings[self->size]) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Failed to allocate memory for string copy: '%s'", string);
        return;
    }
    
    // 5. Success - increment size
    self->size += 1;
}

/**
 * Append a string by taking ownership (no copy made)
 * Caller gives up ownership of the string pointer
 */
void StringArray_append_getting_ownership(StringArray *self, char *string) {
    // 1. Parameter validation
    if (!self || !string) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters: self or string is NULL");
        }
        return;
    }
    
    // 2. Check error state
    if (self->is_error) {
        return;
    }
    
    // 3. Expand the strings array
    char **new_strings = (char**)realloc(self->strings, (self->size + 1) * sizeof(char*));
    if (!new_strings) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Failed to allocate memory for %d string pointers", self->size + 1);
        return;
    }
    self->strings = new_strings;
    
    // 4. Take ownership of the string (no copy)
    self->strings[self->size] = string;
    self->size += 1;
}

/**
 * Remove an element at the specified position
 */
void StringArray_pop(StringArray *self, int position) {
    // 1. Parameter validation
    if (!self) {
        return;
    }
    
    // 2. Check error state
    if (self->is_error) {
        return;
    }
    
    // 3. Validate position
    if (position < 0 || position >= self->size) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Position %d out of bounds (valid range: 0-%d)", position, self->size - 1);
        return;
    }
    
    // 4. Free the string at this position
    free(self->strings[position]);
    
    // 5. Shift remaining elements left
    for (int i = position; i < self->size - 1; i++) {
        self->strings[i] = self->strings[i + 1];
    }
    
    // 6. Decrease size
    self->size -= 1;
}

/**
 * Merge another StringArray into this one
 * Demonstrates error propagation between objects
 */
void StringArray_merge(StringArray *self, StringArray *other) {
    // 1. Parameter validation
    if (!self || !other) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters: self or other is NULL");
        }
        return;
    }
    
    // 2. Check error state of both objects
    if (self->is_error) {
        return; // Already in error state
    }
    
    if (other->is_error) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), 
                "Source array has error: %s", other->error);
        return;
    }
    
    // 3. Append each string from other array
    for (int i = 0; i < other->size; i++) {
        StringArray_append(self, other->strings[i]);
        if (self->is_error) {
            return; // Error occurred during append
        }
    }
}

/**
 * Display the contents of the array
 * Shows how to handle display when in error state
 */
void StringArray_represent(StringArray *self) {
    // Handle NULL pointer
    if (!self) {
        printf("Error: StringArray is NULL
");
        return;
    }
    
    // Handle error state
    if (self->is_error) {
        printf("Error: %s
", self->error);
        return;
    }
    
    // Display normal contents
    printf("StringArray contents (%d items):
", self->size);
    for (int i = 0; i < self->size; i++) {
        printf("  [%d]: %s
", i, self->strings[i]);
    }
}

/**
 * Create a deep copy of the StringArray
 * Returns NULL if cloning fails
 */
StringArray* StringArray_clone(StringArray *self) {
    // Parameter validation
    if (!self) {
        return NULL;
    }
    
    // Cannot clone object in error state
    if (self->is_error) {
        return NULL;
    }
    
    // Create new array
    StringArray *clone = newStringArray();
    if (!clone) {
        return NULL; // Memory allocation failed
    }
    
    // Copy all strings
    for (int i = 0; i < self->size; i++) {
        StringArray_append(clone, self->strings[i]);
        if (clone->is_error) {
            StringArray_free(clone);
            return NULL; // Error during copying
        }
    }
    
    return clone;
}

// ================================
// ERROR MANAGEMENT FUNCTIONS
// ================================

/**
 * Check if the object is in error state
 * Returns: true if error, false if normal
 */
bool StringArray_is_error(StringArray *self) {
    if (!self) {
        return true; // NULL pointer is considered an error
    }
    return self->is_error;
}

/**
 * Clear the error state and reset object to normal operation
 */
void StringArray_clear_error(StringArray *self) {
    if (!self) {
        return;
    }
    self->is_error = false;
    self->error[0] = '\0'; // Clear error message
}

/**
 * Get the current error message
 * Returns: Error message string
 */
const char* StringArray_get_error(StringArray *self) {
    if (!self) {
        return "Object is NULL";
    }
    if (!self->is_error) {
        return "No error";
    }
    return self->error;
}

/**
 * Cleanup function - free all memory
 */
void StringArray_free(StringArray *self) {
    if (!self) {
        return;
    }
    
    // Free all individual strings
    if (self->strings) {
        for (int i = 0; i < self->size; i++) {
            if (self->strings[i]) {
                free(self->strings[i]);
            }
        }
        free(self->strings);
    }
    
    // Free the structure itself
    free(self);
}

// ================================
// DEMONSTRATION
// ================================

int main() {
    printf("=== OUI Internal Error Strategy Demo ===

");
    
    // Create a new StringArray
    StringArray *array = newStringArray();
    if (!array) {
        printf("Failed to create StringArray
");
        return 1;
    }
    
    printf("1. Adding strings to array:
");
    StringArray_append(array, "Hello");
    StringArray_append(array, "World");
    StringArray_append(array, "from");
    StringArray_append(array, "OUI");
    StringArray_represent(array);
    
    printf("
2. Attempting to set value at invalid index:
");
    StringArray_set_value(array, 50, "New Value");
    
    if (StringArray_is_error(array)) {
        printf("Error occurred: %s
", StringArray_get_error(array));
        printf("Array is now in error state - no operations will work
");
    }
    
    printf("
3. Trying to add more strings (should fail silently):
");
    StringArray_append(array, "This won't work");
    StringArray_represent(array); // Will show the error
    
    printf("
4. Clearing error state:
");
    StringArray_clear_error(array);
    printf("Error cleared. Array is now functional again.
");
    
    printf("
5. Adding strings after clearing error:
");
    StringArray_append(array, "I'm back!");
    StringArray_represent(array);
    
    printf("
6. Testing pop operation:
");
    StringArray_pop(array, 2); // Remove "from"
    StringArray_represent(array);
    
    printf("
7. Testing find operation:
");
    int pos = StringArray_find_position(array, "World");
    if (pos >= 0) {
        printf("Found 'World' at index %d
", pos);
    } else {
        printf("'World' not found
");
    }
    
    // Cleanup
    StringArray_free(array);
    printf("
Demo completed successfully!
");
    
    return 0;
}
```
```c
#include <stdlib.h>
#include <stdbool.h>
#include <stdio.h>
#include <string.h>

typedef struct  {
    char error[100];
    bool is_error;
    int size;
    char **strings;
} StringArray;

StringArray * newStringArray();
int StringArray_find_position( StringArray *self, const char *string);


void StringArray_set_value( StringArray *self, int index, const char *value);


void StringArray_append_getting_ownership( StringArray *self, char *string);


// Function prototypes
void StringArray_append( StringArray *self, const  char *string);


void StringArray_pop( StringArray *self, int position);


void StringArray_merge( StringArray *self,  StringArray *other);


void StringArray_represent( StringArray *self);


 StringArray * StringArray_clone(StringArray *self);

char * privateStringArray_append_if_not_included(StringArray *self,char *value);

void StringArray_free(StringArray *self);

bool StringArray_is_error(StringArray *self);

void StringArray_clear_error(StringArray *self);

//========================================Definitions ===================

StringArray * newStringArray(){
    StringArray *self = (StringArray*)malloc(sizeof(StringArray));
    if (!self) {
        return NULL;
    }
    
    self->size = 0;
    self->is_error = false;
    self->error[0] = '\0';

    self->strings = (char**)malloc(sizeof(char*));
    if (!self->strings) {
        free(self);
        return NULL;
    }

    return self;
}

int StringArray_find_position( StringArray *self, const char *string){
    if (!self || !string) {
        return -1;
    }
    if (self->is_error) {
        return -1;
    }
    
    for(int i = 0; i < self->size; i++){
        if(strcmp(self->strings[i], string) == 0){
            return i;
        }
    }
    return -1;
}


void StringArray_set_value( StringArray *self, int index, const char *value){
    if (!self || !value) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters");
        }
        return;
    }
    if (self->is_error) return;
    
    if(index >= self->size || index < 0){
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Index out of bounds");
        return;
    }

    int size = strlen(value);
    char *new_string = (char*)realloc(self->strings[index], size + 1);
    if (!new_string) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Failed to allocate memory");
        return;
    }
    self->strings[index] = new_string;
    strcpy(self->strings[index], value);
}
void StringArray_append_getting_ownership( StringArray *self, char *string){
    if (!self || !string) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters");
        }
        return;
    }
    if (self->is_error) return;

    char **new_strings = (char**)realloc(self->strings, (self->size + 1) * sizeof(char*));
    if(!new_strings){
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Failed to allocate memory");
        return;
    }
    self->strings = new_strings;
    self->strings[self->size] = string;
    self->size+=1;
}

// Function prototypes
void StringArray_append( StringArray *self, const  char *string){
    if (!self || !string) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters");
        }
        return;
    }
    if (self->is_error) return;

    char **new_strings = (char**)realloc(self->strings, (self->size + 1) * sizeof(char*));
    if (!new_strings) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Failed to allocate memory");
        return;
    }
    self->strings = new_strings;
    
    self->strings[self->size] = strdup(string);
    if (!self->strings[self->size]) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Failed to allocate memory for string copy");
        return;
    }
    self->size+=1;
}

void StringArray_pop( StringArray *self, int position){
    if (!self) {
        return;
    }
    if (self->is_error) return;

    if(position < 0 || position >= self->size){
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Index out of bounds");
        return;
    }

    free(self->strings[position]);
    for(int i = position; i < self->size -1; i++){
        self->strings[i] = self->strings[i+1];
    }
    self->size-=1;
}

void StringArray_merge( StringArray *self,  StringArray *other){
    if (!self || !other) {
        if (self) {
            self->is_error = true;
            snprintf(self->error, sizeof(self->error), "Invalid parameters");
        }
        return;
    }
    if (self->is_error) return;
    if (other->is_error) {
        self->is_error = true;
        snprintf(self->error, sizeof(self->error), "Source array has error");
        return;
    }
    
    for(int i = 0; i < other->size; i++){
        StringArray_append(self, other->strings[i]);
        if (self->is_error) {
            return;
        }
    }
}


void StringArray_represent( StringArray *self){
    if (!self) {
        printf("Error: StringArray is NULL\n");
        return;
    }
    if (self->is_error) {
        printf("Error: %s\n", self->error);
        return;
    }
    
    for(int i = 0; i < self->size; i++){
        printf("%s\n", self->strings[i]);
    }
}


 StringArray * StringArray_clone(StringArray *self){
    if (!self) {
        return NULL;
    }
    if (self->is_error) {
        return NULL;
    }
    
    StringArray  *clone = newStringArray();
    if (!clone) {
        return NULL;
    }
    
    for(int i = 0; i< self->size; i++){
        StringArray_append(clone,self->strings[i]);
        if (clone->is_error) {
            StringArray_free(clone);
            return NULL;
        }
    }
    return clone;
}

char * privateStringArray_append_if_not_included(StringArray *self,char *value){
    if (!self || !value) {
        if (value) free(value);
        return NULL;
    }
    if (self->is_error) {
        free(value);
        return NULL;
    }
    
    long position=StringArray_find_position(self,value);
    if(position != -1){
        free(value);
        return self->strings[position];
    }
    StringArray_append_getting_ownership(self,value);
    if (self->is_error) {
        return NULL;
    }
    return value;
}
void StringArray_free(StringArray *self){
    if (!self) {
        return;
    }
    
    if (self->strings) {
        for(int i = 0; i < self->size; i++){
            if (self->strings[i]) {
                free(self->strings[i]);
            }
        }
        free(self->strings);
    }
    free(self);
}

bool StringArray_is_error(StringArray *self){
    if (!self) {
        return true;
    }
    return self->is_error;
}

void StringArray_clear_error(StringArray *self){
    if (!self) {
        return;
    }
    self->is_error = false;
    self->error[0] = '\0';
}

int main(){
    StringArray *array = newStringArray();
    StringArray_append(array, "Hello");
    StringArray_append(array, "World");
    StringArray_represent(array);

    StringArray_set_value(array,50,"New Value");
    if(StringArray_is_error(array)){
        printf("Error occurred: %s\n", array->error);
        StringArray_clear_error(array);
    }
    
    StringArray_free(array);
    return 0;
}

```