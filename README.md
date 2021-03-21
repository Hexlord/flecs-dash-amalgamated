# flecs-dash-amalgamated

Amalgamated autonomous version of https://github.com/flecs-hub/flecs-dash

## Usage
0. Copy flecs-dash to the build directory (where the application is executed)
1. Include the CMakeLists
2. Link against flecs-dash-amalgamated_static
3. After setting up os api execute the initialization code:
```c
world.import<flecs::dash>();
world.import<flecs::systems::civetweb>();

ecs.entity().set<flecs::dash::Server>({9090});
```

## Details
I was not able to link against shared library (flecs-dash-amalgamated) for some reason.