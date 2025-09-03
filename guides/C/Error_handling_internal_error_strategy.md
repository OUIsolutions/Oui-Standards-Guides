these its the old error handle strategy ,adopted with a lot of oui projects
use thes e pattern , only if you are working on a legacy project. 
if you are creating a new one , use the [Error_handling_union_strategy.md](/guides/C/Error_handling_union_strategy.md) instead 
 
these pattern is based on setting the error on the internal struct 
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