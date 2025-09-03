since its impossible to make true OOP in C, we can only emulate it through conventions and design patterns. This means we have to be disciplined in our approach and follow certain guidelines to achieve similar results.

Here , its a full example of POO emulation, using C
```c 
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <stdbool.h>

typedef struct {
 char *model;
 bool release_model;
 float speed;
 int year;
 float price;
}Car;
// ==================================+Declaration========================================

Car *private_newCar_empty();

Car *newCar(const char* model, int year, float price);

Car *newFerrari(int year, float price);

void private_Car_free_model(Car *self);

void Car_set_model_copying(Car *self, const char *model);

void Car_set_model_getting_ownership(Car *self, const char *model);

void Car_set_model_referencing(Car *self, const char *model);


void Car_set_speed(Car *self, float speed);

void Car_set_year(Car *self, int year);

void Car_set_price(Car *self, float price);

const char* Car_get_model(Car *self);

float Car_get_speed(Car *self);

int Car_get_year(Car *self);

float Car_get_price(Car *self);

void Car_free(Car *self);

//===================================Definitions============================================

// internal constructors
Car *private_newCar_empty(){
    Car *car = malloc(sizeof(Car));
    *car = (Car){0};
    return car;
}
// custom Constructor
Car *newFerrari(int year, float price) {
     Car *car = private_newCar_empty();
    Car_set_model_referencing(car, "Ferrari");
    Car_set_year(car, year);
    Car_set_price(car, price);
    return car;
}
// default constructor
Car *newCar(const char* model, int year, float price) {
    Car *car = private_newCar_empty();
    Car_set_model_copying(car, model);
    Car_set_year(car, year);
    Car_set_price(car, price);
    return car;
}
void private_Car_free_model(Car *self) {
    if(!self)return;
    if(!self->release_model) return;
    if(!self->model) return;
    free(self->model);
    self->model = NULL;
    self->release_model = false;
}
void Car_set_model_copying(Car *self, const char *model) {
    private_Car_free_model(self);
    self->model = strdup(model);
    self->release_model = true;
}

void Car_set_model_getting_ownership(Car *self, const char *model) {
    private_Car_free_model(self);
    self->model = (char *)model;
    self->release_model = true;
}

void Car_set_model_referencing(Car *self, const char *model) {
    private_Car_free_model(self);
    self->model = (char *)model;
    self->release_model = false;
}



void Car_set_year(Car *self, int year) {
    self->year = year;
}
void Car_set_speed(Car *self, float speed) {
    self->speed = speed;
}
void Car_set_price(Car *self, float price) {
    self->price = price;
}

const char* Car_get_model(Car *self) {
    return self ? self->model : NULL;
}

float Car_get_speed(Car *self) {
    return self ? self->speed : 0.0f;
}

int Car_get_year(Car *self) {
    return self ? self->year : 0;
}

float Car_get_price(Car *self) {
    return self ? self->price : 0.0f;
}

void Car_free(Car *self) {
    private_Car_free_model(self);
    free(self);
}
int main(){
    Car *ferrari = newFerrari(2020, 250000.0f);
    Car_set_speed(ferrari, 200.0f);
    printf("Car model: %s\n", Car_get_model(ferrari));
    printf("Car speed: %.2f\n", Car_get_speed(ferrari));
    printf("Car year: %d\n", Car_get_year(ferrari));
    printf("Car price: %.2f\n", Car_get_price(ferrari));
    Car_free(ferrari);
    printf("---------------------------------------\n");
    Car *Lamborghini = newCar("Lamborghini", 2021, 300000.0f);
    Car_set_speed(Lamborghini, 210.0f);
    printf("Car model: %s\n", Car_get_model(Lamborghini));
    printf("Car speed: %.2f\n", Car_get_speed(Lamborghini));
    printf("Car year: %d\n", Car_get_year(Lamborghini));
    printf("Car price: %.2f\n", Car_get_price(Lamborghini));
    Car_free(Lamborghini);
    return 0;

}

```
note , that you must follow these rules:

1. All the public constructors, must starts with **new**
2. All the private constructors must start with **private_new**
3. All the methods must start with the struct name followed by an underscore, like **Car_set_model**
4. All the getters must start with the struct name followed by an underscore, like **Car_get_model**
5. Any Pointer should have a ownership control mechanism (unless if its just one default behavior)

