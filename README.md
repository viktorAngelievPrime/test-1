
## Table of Contents
 
1. [Intro](#intro)
2. [Installation](#installation)
3. [Disclaimer](#disclaimer)
4. [Modules](#modules)
  
  
  
  
  
  
# Intro
So what is this? It's basically a framework for `bolt`.

Featuring modules that allow you to accelerate your development process.



This is so far a solo project. It has been in development for months, with hundreds of hours in.
I would genuinely be the happiest person ever to find out it helped someone out there.

Feedback is greatly appreciated!


# Installation

1. Download the [latest release](https://github.com/reapermc/reaper_framework/releases/latest).
2. Extract the `zip` file inside of your project's `src/data`.
3. Create a starting point of your project with a `main.bolt` file inside of `<your_namespace>/modules/`

The finished product should look as such:

```
src
└─── data
	 ├─── _                          
     │    ├─── functions   
     │    │    └─── reaper_framework
     │    └─── modules
     │         └─── reaper_framework
     │
     │
	 └─── demo           <------ your project namespace
          └─── modules
               └─── main.bolt
```

# Disclaimer

The framework only supports coding using `.bolt` files.
There is not and will never be support for `.mcfunction` files.


# Modules

## Spark

Allows for quick and easy handling of `load`, `tick` and `load_player` events.

Events:
```
	Spark.load()
	Spark.tick()
	Spark.load_player()
```

Can be prefixed with `pre` or `post` for ordering:
```
	Spark.preload()
	Spark.pretick()
	Spark.postload_player()
```
Optional `layer` keyword argument can be used for further ordering:
(don't overuse unless necessary)
```
	Spark.load(layer='LIB')
	Spark.load(layer='SYSTEM')
```

### Code example:
```py
from _:reaper import Spark

@Spark.load()
def server_load():
	say Hello World!

@Spark.load_player()
def greeter():
	say Hello @s, glad you're here!

```







