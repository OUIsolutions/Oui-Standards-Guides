### Library Namespace Collision
All Constants, Function Names, global variables, macros (basically everything that is global) MUST have the library initial in the name:
for example:
```c
dtw_list_files_recursively(const char *path); 
```
where the **dtw** its the initial of [DoTheWorld](https://github.com/OUIsolutions/DoTheWorld) or:
```c
CwebHttpRequest_get_param(CwebHttpRequest *request ,const char *name);
```
wich its part of the [CWebStudio](https://github.com/OUIsolutions/CWebStudio) project
even on constans or enuns, you should use it, as example:
```c 
#define CWEB_BAD_REQUEST 400
```

### Constants:
all Constants should be UPPER_SNAKE_CASE
### Global Variables
all Global Variables should be lower_snake_case
### Variables
all Variables should be lower_snake_case

### Functions
if the function is part of [POO_emulation.md](/guides/C/POO_emulation.md) follow the POO Emulation naming conventions
if its not, follow the lower_snake_case convention
