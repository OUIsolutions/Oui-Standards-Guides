these is the default strategy of OUI, of how to handle errors
basically we create a union, with the given type, or the possible error that 
can be generated
```c
#include <string.h>  
#include <stdlib.h> 
#include <stdio.h>  
#include <stdbool.h> 
#include <stdarg.h>
enum {
    DIVISION_BY_ZERO,
    OTHER_ERROR
};
typedef struct  {
    float value;
    const char *error;
    int code;
    bool is_error;
}ErrorOrFloat;
ErrorOrFloat newErrorOrFloatError(int code,const char *error);

ErrorOrFloat newErrorOrFloatValue(float value);

ErrorOrFloat divide(float a, float b);


ErrorOrFloat newErrorOrFloatError(int code,const char *error){
   ErrorOrFloat e = {0};
   e.is_error = true;
   e.error = error;
   e.code = code;
   return e;
}

ErrorOrFloat newErrorOrFloatValue(float value) {
   ErrorOrFloat e = {0};
   e.is_error = false;
   e.value = value;
   return e;
}
ErrorOrFloat divide(float a, float b){
    if (b == 0) {
        return newErrorOrFloatError(1,"Division by zero ");
    }
    return newErrorOrFloatValue(a / b);
}



int main(){
    ErrorOrFloat result = divide(10, 0);
    if (result.is_error) {
        printf("Error: %s (code: %d)\n", result.error, result.code);
    } else {
        printf("Result: %f\n", result.value);
    }
    ErrorOrFloat result2 = divide(10, 2);
    if (result2.is_error) {
        printf("Error: %s (code: %d)\n", result2.error, result2.code);
    } else {
        printf("Result: %f\n", result2.value);
    }

    return 0;
}
```
Adition:
if you need to store specif values into the menssage , you can make these way:
(but it will consume more memory)
```c 
#include <string.h>  
#include <stdlib.h> 
#include <stdio.h>  
#include <stdbool.h> 
#include <stdarg.h>
enum {
    DIVISION_BY_ZERO,
    OTHER_ERROR
};
typedef struct  {
    float value;
    char  error[30];
    int code;
    bool is_error;
}ErrorOrFloat;
ErrorOrFloat newErrorOrFloatError(int code,char *error,...);

ErrorOrFloat newErrorOrFloatValue(float value);

ErrorOrFloat divide(float a, float b);


ErrorOrFloat newErrorOrFloatError(int code,char *error,...) {
   ErrorOrFloat e = {0};
   e.is_error = true;
   va_list args;
   va_start(args, error);
   vsnprintf(e.error, sizeof(e.error), error, args);
   va_end(args);
   e.code = code;
   return e;
}

ErrorOrFloat newErrorOrFloatValue(float value) {
   ErrorOrFloat e = {0};
   e.is_error = false;
   e.value = value;
   return e;
}
ErrorOrFloat divide(float a, float b){
    if (b == 0) {
        return newErrorOrFloatError(1,"Division by zero %lf/0",a);
    }
    return newErrorOrFloatValue(a / b);
}



int main(){
    ErrorOrFloat result = divide(10, 0);
    if (result.is_error) {
        printf("Error: %s (code: %d)\n", result.error, result.code);
    } else {
        printf("Result: %f\n", result.value);
    }
    ErrorOrFloat result2 = divide(10, 2);
    if (result2.is_error) {
        printf("Error: %s (code: %d)\n", result2.error, result2.code);
    } else {
        printf("Result: %f\n", result2.value);
    }

    return 0;
}
```
