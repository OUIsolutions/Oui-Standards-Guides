# Object-Oriented Programming (OOP) Emulation in C

## Introduction

Since it's impossible to achieve true Object-Oriented Programming (OOP) in C (because C is a procedural language), we can only **emulate** it through conventions and design patterns. This means we have to be disciplined in our approach and follow certain guidelines to achieve similar results to what we would get in languages like C++, Java, or Python.

## What is OOP Emulation?

OOP emulation in C means we use:
- **Structs** to represent classes/objects
- **Function** (sometimes) to represent methods
- **Naming conventions** to distinguish between public and private functions
- **Memory management patterns** to handle object lifecycle
- **Ownership systems** to manage pointer responsibilities

## Why Emulate OOP in C?

1. **Code Organization**: Makes large C projects more manageable
2. **Encapsulation**: Groups related data and functions together
3. **Reusability**: Creates reusable "objects" that can be instantiated multiple times
4. **Maintainability**: Easier to understand and modify code
5. **Abstraction**: Hides implementation details from users

## Complete Example: Car Class in C

Here's a full example of OOP emulation using a `Car` "class":
```c 
#include <string.h>  // For string operations like strdup()
#include <stdlib.h>  // For memory allocation (malloc, free)
#include <stdio.h>   // For input/output operations (printf)
#include <stdbool.h> // For boolean type (true, false)

// This is our "class" definition - in C we use a struct
// Think of this as the blueprint for all Car objects
typedef struct {
    // The car's model name (e.g., "Ferrari", "Toyota")
    char *model;
    
    // Ownership flag: tells us if we need to free the model string
    // true = we own the memory and must free it
    // false = we're just referencing someone else's memory
    bool release_model;
    
    // Car properties (attributes in OOP terms)
    float speed;  // Current speed in km/h
    int year;     // Year of manufacture
    float price;  // Price in dollars
} Car;
// =======================================================================================
// FUNCTION DECLARATIONS (Method Signatures)
// =======================================================================================
// In OOP languages, these would be method declarations inside the class
// In C, we declare them separately and use naming conventions

// PRIVATE CONSTRUCTOR (Internal use only)
// Creates an empty Car object with all fields initialized to zero
Car *private_newCar_empty();

// PUBLIC CONSTRUCTORS (These are like class constructors in OOP)
// Creates a new Car with specified model, year, and price
Car *newCar(const char* model, int year, float price);

// Factory constructor - creates a Ferrari with predefined model
Car *newFerrari(int year, float price);

// PRIVATE HELPER METHODS (Internal use only)
// Safely frees the model string if we own it
void private_Car_free_model(Car *self);

// PUBLIC SETTER METHODS (These modify object properties)
// Three different ways to set the model, each with different ownership semantics:

// 1. COPYING: Makes a copy of the string (safest, uses more memory)
void Car_set_model_copying(Car *self, const char *model);

// 2. OWNERSHIP TRANSFER: Takes ownership of existing string (efficient, must be malloc'd)
void Car_set_model_transfering_ownership(Car *self, const char *model);

// 3. REFERENCE: Just points to existing string (fastest, but risky)
void Car_set_model_referencing(Car *self, const char *model);

// Standard setter methods for other properties
void Car_set_speed(Car *self, float speed);
void Car_set_year(Car *self, int year);
void Car_set_price(Car *self, float price);

// PUBLIC GETTER METHODS (These access object properties)
// Note: All getters are const-safe and null-safe
const char* Car_get_model(Car *self);
float Car_get_speed(Car *self);
int Car_get_year(Car *self);
float Car_get_price(Car *self);

// DESTRUCTOR (Like a destructor in C++ or __del__ in Python)
// Safely frees all memory associated with the Car object
void Car_free(Car *self);

//=======================================================================================
// FUNCTION DEFINITIONS (Method Implementations)
//=======================================================================================

// PRIVATE CONSTRUCTOR: Creates an empty Car object
// This is like a base constructor that other constructors can build upon
Car *private_newCar_empty(){
    // Allocate memory for one Car struct
    Car *car = malloc(sizeof(Car));
    
    // Initialize all fields to zero using compound literal syntax
    // This is equivalent to: car->model = NULL; car->release_model = false; etc.
    *car = (Car){0};
    
    return car;  // Return pointer to the new Car object
}

// SPECIALIZED CONSTRUCTOR: Creates a Ferrari
// This is like a factory method in OOP - it creates objects with preset values
Car *newFerrari(int year, float price) {
    // Start with an empty Car
    Car *car = private_newCar_empty();
    
    // Set Ferrari-specific properties
    // We use referencing because "Ferrari" is a string literal (always exists)
    Car_set_model_referencing(car, "Ferrari");
    Car_set_year(car, year);
    Car_set_price(car, price);
    
    return car;
}

// GENERAL CONSTRUCTOR: Creates any Car with specified properties
// This is like the main constructor in OOP languages
Car *newCar(const char* model, int year, float price) {
    // Start with an empty Car
    Car *car = private_newCar_empty();
    
    // Set properties using copying to be safe with the model string
    Car_set_model_copying(car, model);
    Car_set_year(car, year);
    Car_set_price(car, price);
    
    return car;
}
// PRIVATE HELPER METHOD: Safely frees the model string
// This implements our ownership system - only frees memory we own
void private_Car_free_model(Car *self) {
    // Safety check: don't operate on NULL pointers
    if(!self) return;
    
    // Only free if we own the memory (release_model == true)
    if(!self->release_model) return;
    
    // Only free if there's actually something to free
    if(!self->model) return;
    
    // Free the memory and reset the pointer
    free(self->model);
    self->model = NULL;
    self->release_model = false;
}

// SETTER METHOD 1: Copying strategy
// Makes a complete copy of the input string - safest but uses more memory
void Car_set_model_copying(Car *self, const char *model) {
    // First, free any existing model string we might own
    private_Car_free_model(self);
    
    // Create a copy of the input string using strdup()
    // strdup() allocates new memory and copies the string
    self->model = strdup(model);
    
    // Mark that we own this memory and must free it later
    self->release_model = true;
}

// SETTER METHOD 2: Ownership transfer strategy
// Takes over an already-allocated string - efficient but requires careful use
void Car_set_model_transfering_ownership(Car *self, const char *model) {
    // Free any existing model string we might own
    private_Car_free_model(self);
    
    // Take ownership of the provided string (cast away const)
    // WARNING: The input string MUST have been allocated with malloc()!
    self->model = (char *)model;
    
    // Mark that we now own this memory
    self->release_model = true;
}

// SETTER METHOD 3: Reference strategy
// Just points to existing memory - fastest but most dangerous
void Car_set_model_referencing(Car *self, const char *model) {
    // Free any existing model string we might own
    private_Car_free_model(self);
    
    // Just store the pointer - we don't own this memory
    self->model = (char *)model;
    
    // Mark that we DON'T own this memory (don't free it)
    self->release_model = false;
}

// SIMPLE SETTER METHODS: For primitive types (no ownership issues)
void Car_set_year(Car *self, int year) {
    self->year = year;
}

void Car_set_speed(Car *self, float speed) {
    self->speed = speed;
}

void Car_set_price(Car *self, float price) {
    self->price = price;
}

// GETTER METHODS: Safely access object properties
// All getters are null-safe - they check if 'self' is NULL before accessing

const char* Car_get_model(Car *self) {
    // Return the model if self is valid, otherwise return NULL
    return self ? self->model : NULL;
}

float Car_get_speed(Car *self) {
    // Return the speed if self is valid, otherwise return 0.0
    return self ? self->speed : 0.0f;
}

int Car_get_year(Car *self) {
    // Return the year if self is valid, otherwise return 0
    return self ? self->year : 0;
}

float Car_get_price(Car *self) {
    // Return the price if self is valid, otherwise return 0.0
    return self ? self->price : 0.0f;
}

// DESTRUCTOR: Safely destroys a Car object
// This is like a destructor in C++ - it cleans up all resources
void Car_free(Car *self) {
    // Free the model string if we own it
    private_Car_free_model(self);
    
    // Free the Car object itself
    free(self);
    
    // Note: In production code, you might want to set the pointer to NULL
    // in the calling function to avoid use-after-free bugs
}
// DEMONSTRATION: How to use our Car "class"
int main(){
    // Create a Ferrari using the specialized constructor
    // This demonstrates factory method pattern
    Car *ferrari = newFerrari(2020, 250000.0f);
    
    // Set additional properties using setter methods
    Car_set_speed(ferrari, 200.0f);
    
    // Access properties using getter methods and print them
    printf("Car model: %s\n", Car_get_model(ferrari));
    printf("Car speed: %.2f\n", Car_get_speed(ferrari));
    printf("Car year: %d\n", Car_get_year(ferrari));
    printf("Car price: %.2f\n", Car_get_price(ferrari));
    
    // IMPORTANT: Always free objects when done to prevent memory leaks
    Car_free(ferrari);
    // After this point, 'ferrari' points to freed memory - don't use it!
    
    printf("---------------------------------------\n");
    
    // Create a general car using the standard constructor
    Car *lamborghini = newCar("Lamborghini", 2021, 300000.0f);
    
    // Modify properties
    Car_set_speed(lamborghini, 210.0f);
    
    // Display information
    printf("Car model: %s\n", Car_get_model(lamborghini));
    printf("Car speed: %.2f\n", Car_get_speed(lamborghini));
    printf("Car year: %d\n", Car_get_year(lamborghini));
    printf("Car price: %.2f\n", Car_get_price(lamborghini));
    
    // Clean up memory
    Car_free(lamborghini);
    
    return 0;
}

```

## 📋 Essential Rules for C OOP Emulation

To maintain consistency and readability when emulating OOP in C, follow these strict naming conventions:

### 1. **Public Constructors** ⭐
- **Rule**: Must start with `new`
- **Examples**: 
  - `newCar()` - general constructor
  - `newFerrari()` - specialized constructor
  - `newStudent()` - for a Student "class"
- **Why**: Makes it immediately clear this function creates a new object

### 2. **Private Constructors** 🔒
- **Rule**: Must start with `private_new`
- **Examples**:
  - `private_newCar_empty()` - internal helper constructor
  - `private_newStudent_base()` - base constructor for inheritance
- **Why**: Indicates internal use only, not for external callers

### 3. **Methods (Functions that operate on objects)** 🛠️
- **Rule**: Must start with `StructName_methodName`
- **Examples**:
  - `Car_set_model()` - setter method
  - `Student_calculate_gpa()` - calculation method
  - `List_add_item()` - manipulation method
- **Why**: Creates a namespace and clearly shows which "class" the method belongs to

### 4. **Getter Methods** 📖
- **Rule**: Must start with `StructName_get_`
- **Examples**:
  - `Car_get_model()` - returns the model
  - `Student_get_name()` - returns student name
  - `List_get_size()` - returns list size
- **Why**: Clearly indicates read-only access to object properties

### 5. **Pointer Ownership Control** ⚠️
- **Rule**: Any pointer should have a clear ownership mechanism
- **Three strategies**:
  1. **Copying**: `StructName_set_field_copying()` - makes a copy
  2. **Ownership Transfer**: `StructName_set_field_transfering_ownership()` - takes ownership
  3. **Reference**: `StructName_set_field_referencing()` - just references
- **Why**: Prevents memory leaks and double-free errors

## 🔧 Memory Ownership System Explained

The ownership system is crucial for preventing memory bugs. Here's how it works:

### **Copying Strategy** 📋
```c
Car_set_model_copying(car, "Toyota");
```
- **What happens**: Creates a new copy of the string
- **Memory**: Uses `strdup()` to allocate new memory
- **Ownership**: Car object owns the copy
- **When to use**: When you want safety and don't mind using extra memory
- **Pros**: Very safe, original string can be modified/freed without affecting Car
- **Cons**: Uses more memory, slower

### **Ownership Transfer Strategy** 🎯
```c
char *model = malloc(20);
strcpy(model, "BMW");
Car_set_model_transfering_ownership(car, model);
// DON'T free 'model' here - Car owns it now!
```
- **What happens**: Car takes ownership of existing string
- **Memory**: No new allocation, transfers existing memory
- **Ownership**: Car object now owns the string
- **When to use**: When you have a malloc'd string you want to give to the object
- **Pros**: Memory efficient, fast
- **Cons**: Requires careful memory management, easy to make mistakes

### **Reference Strategy** 🔗
```c
Car_set_model_referencing(car, "Honda"); // String literal
```
- **What happens**: Car just stores a pointer to existing memory
- **Memory**: No allocation, just stores pointer
- **Ownership**: Car does NOT own the memory
- **When to use**: With string literals or long-lived strings
- **Pros**: Very fast, no memory overhead
- **Cons**: Original string must outlive the Car object

## 🏎️ Equivalent Implementations in Other Languages

### C++ Version (True OOP)

```cpp
#include <iostream>
#include <string>
#include <memory>

class Car {
private:
    std::string model;  // C++ strings manage their own memory
    float speed;
    int year;
    float price;

public:
    // Constructors
    Car(const std::string& model, int year, float price) 
        : model(model), year(year), price(price), speed(0.0f) {}
    
    // Static factory method (like our newFerrari)
    static std::unique_ptr<Car> createFerrari(int year, float price) {
        return std::make_unique<Car>("Ferrari", year, price);
    }
    
    // Setters
    void setModel(const std::string& model) { this->model = model; }
    void setSpeed(float speed) { this->speed = speed; }
    void setYear(int year) { this->year = year; }
    void setPrice(float price) { this->price = price; }
    
    // Getters (const methods - can't modify object)
    const std::string& getModel() const { return model; }
    float getSpeed() const { return speed; }
    int getYear() const { return year; }
    float getPrice() const { return price; }
    
    // Destructor (automatically called when object is destroyed)
    ~Car() {
        // std::string automatically cleans up its memory
        std::cout << "Car destroyed: " << model << std::endl;
    }
};

// Usage example
int main() {
    // Create objects on the stack (automatic memory management)
    Car ferrari("Ferrari", 2020, 250000.0f);
    ferrari.setSpeed(200.0f);
    
    std::cout << "Model: " << ferrari.getModel() << std::endl;
    std::cout << "Speed: " << ferrari.getSpeed() << std::endl;
    
    // Create with smart pointer (modern C++)
    auto lamborghini = std::make_unique<Car>("Lamborghini", 2021, 300000.0f);
    lamborghini->setSpeed(210.0f);
    
    std::cout << "Model: " << lamborghini->getModel() << std::endl;
    std::cout << "Speed: " << lamborghini->getSpeed() << std::endl;
    
    // Objects are automatically destroyed when going out of scope
    return 0;
}
```

### JavaScript Version (Prototype-based OOP)

```javascript
class Car {
    // Constructor
    constructor(model, year, price) {
        this.model = model;   // JavaScript handles string memory automatically
        this.year = year;
        this.price = price;
        this.speed = 0.0;
    }
    
    // Static factory method (like our newFerrari)
    static createFerrari(year, price) {
        return new Car("Ferrari", year, price);
    }
    
    // Setters
    setModel(model) { this.model = model; }
    setSpeed(speed) { this.speed = speed; }
    setYear(year) { this.year = year; }
    setPrice(price) { this.price = price; }
    
    // Getters
    getModel() { return this.model; }
    getSpeed() { return this.speed; }
    getYear() { return this.year; }
    getPrice() { return this.price; }
    
    // Method to display car info
    displayInfo() {
        console.log(`Model: ${this.model}`);
        console.log(`Speed: ${this.speed}`);
        console.log(`Year: ${this.year}`);
        console.log(`Price: ${this.price}`);
    }
}

// Usage example
function main() {
    // Create Ferrari using factory method
    const ferrari = Car.createFerrari(2020, 250000.0);
    ferrari.setSpeed(200.0);
    
    console.log("=== Ferrari ===");
    ferrari.displayInfo();
    
    console.log("---------------------------------------");
    
    // Create Lamborghini using regular constructor
    const lamborghini = new Car("Lamborghini", 2021, 300000.0);
    lamborghini.setSpeed(210.0);
    
    console.log("=== Lamborghini ===");
    lamborghini.displayInfo();
    
    // JavaScript garbage collector automatically cleans up memory
}

main();
```

## 🔍 Comparison of Approaches

| Feature | C (Emulated) | C++ (True OOP) | JavaScript (Prototype OOP) |
|---------|-------------|----------------|---------------------------|
| **Memory Management** | Manual (malloc/free) | RAII + Smart Pointers | Automatic (Garbage Collection) |
| **Encapsulation** | Convention-based | Language-enforced | Convention-based |
| **Inheritance** | Complex emulation | Native support | Prototype chain |
| **Polymorphism** | Function pointers | Virtual functions | Dynamic dispatch |
| **Performance** | Very fast | Fast | Moderate |
| **Safety** | Manual ownership | RAII safety | Memory safe |
| **Learning Curve** | Steep | Moderate | Easy |

## 🛡️ Best Practices for C OOP Emulation

### 1. **Always Initialize Pointers**
```c
// Good
Car *car = private_newCar_empty();  // Initializes all fields to 0/NULL

// Bad
Car *car = malloc(sizeof(Car));  // Uninitialized memory
```

### 2. **Check for NULL Before Using**
```c
// Good
const char* Car_get_model(Car *self) {
    return self ? self->model : NULL;  // NULL-safe
}

// Bad
const char* Car_get_model(Car *self) {
    return self->model;  // Crashes if self is NULL
}
```

### 3. **Always Free What You Allocate**
```c
// Good
Car *car = newCar("Toyota", 2020, 25000.0f);
// ... use car ...
Car_free(car);  // Always free!

// Bad
Car *car = newCar("Toyota", 2020, 25000.0f);
// ... use car ...
// Forgot to free - memory leak!
```

### 4. **Use Consistent Naming**
```c
// Good
Car *newCar(...);           // Public constructor
void Car_set_model(...);    // Public method
float Car_get_speed(...);   // Public getter

// Bad
Car *create_car(...);       // Inconsistent naming
void set_car_model(...);    // Wrong order
float speed(...);           // Missing namespace
```

### 5. **Document Ownership**
```c
// Good - clearly documented
// Takes ownership of the provided string
void Car_set_model_transfering_ownership(Car *self, const char *model);

// Makes a copy of the provided string
void Car_set_model_copying(Car *self, const char *model);
```

## 🚨 Common Pitfalls and How to Avoid Them

### 1. **Memory Leaks**
```c
// BAD: Memory leak
Car *car = newCar("BMW", 2020, 50000.0f);
// Forgot to call Car_free(car);

// GOOD: Proper cleanup
Car *car = newCar("BMW", 2020, 50000.0f);
// ... use car ...
Car_free(car);
car = NULL;  // Prevent accidental reuse
```

### 2. **Use After Free**
```c
// BAD: Use after free
Car *car = newCar("BMW", 2020, 50000.0f);
Car_free(car);
printf("%s\n", Car_get_model(car));  // CRASH!

// GOOD: Set to NULL after freeing
Car *car = newCar("BMW", 2020, 50000.0f);
Car_free(car);
car = NULL;
```

### 3. **Double Free**
```c
// BAD: Double free
char *model = strdup("Toyota");
Car_set_model_transfering_ownership(car, model);
free(model);  // DON'T DO THIS! Car owns it now

// GOOD: Transfer ownership properly
char *model = strdup("Toyota");
Car_set_model_transfering_ownership(car, model);
// Don't free 'model' - Car will free it
```

### 4. **Dangling Pointers with References**
```c
// BAD: Dangling pointer
{
    char temp_model[20];
    strcpy(temp_model, "BMW");
    Car_set_model_referencing(car, temp_model);
}  // temp_model goes out of scope here!
printf("%s\n", Car_get_model(car));  // Undefined behavior!

// GOOD: Use with long-lived strings
Car_set_model_referencing(car, "BMW");  // String literal lives forever
```

