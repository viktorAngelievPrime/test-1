# Reaper Framework Docs

If you have any further questions or need some help, be sure to hit me up on discord `Yeti#0231`.

I don't yet have a discord server for the project, but I may make one in the future.

&nbsp;


## Table of Contents
 
- [Disclaimer](#disclaimer)
- [Modules](#modules)

	- [Spark](#spark)
	- [Scoreboard](#scoreboard)
	- [Data](#data)
	- [Var](#var)
	- [Object](#object)
	- [Sleep](#sleep)
	- [Loop](#loop)
	- [Halt](#halt)
	- [Cmd](#cmd)
	- [Array](#array)
	- [File](#file)

&nbsp;


## Disclaimer

The framework only supports coding using `.bolt` files.

There is not and will never be support for `.mcfunction` files.

&nbsp;


# Modules


## Spark

Allows for quick and easy handling of `load`, `tick` and `load_player` events.

<details>
<summary>Details</summary><p>

```py
# Events:
	@Spark.load()
	@Spark.tick()
	@Spark.load_player()

# Can be prefixed with 'pre' or 'post' for ordering:
	@Spark.preload()
	@Spark.pretick()
	@Spark.postload_player()
	
# Optional 'layer' keyword argument can be used for further ordering:
# (generally don't use this unless you're making a library)
	@Spark.load(layer='LIB')
	@Spark.load(layer='SYSTEM')
```
</p></details>

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

&nbsp;


## Scoreboard

Upgraded version of `Scoreboard` from [bolt_expressions](https://github.com/rx-modules/bolt-expressions). Automatically creates objectives.

<details>
<summary>Details</summary><p>

```py
# Define scoreboards by passing the objective name as a parameter:
	my_score = Scoreboard('my_score')

# You can use custom 'criteria' using the 2nd positional argument ('dummy' by default).
	coas = Scoreboard('coas', 'minecraft.used:minecraft.carrot_on_a_stick')
	shot_bow = Scoreboard('shot_bow', 'used:bow')

# Duplicate scoreboards are prohibited (even across multiple files).
	my_score = Scoreboard('my_score')
	my_score = Scoreboard('my_score')
	
# Doing this will result in an error
```
</p></details>

### Code example:
```py
from _:reaper import Scoreboard

my_score = Scoreboard('my_scoreboard')
another_one = Scoreboard('and_another')['#one']
used_bow = Scoreboard('and_another', 'used:bow')

function ./bolt_expressions_is_awesome:
	my_score['$test'] = 666
	another_one['#one'] = my_score['$test'] * 10
```

&nbsp;

## Data

A way to import `Data` from [bolt_expressions](https://github.com/rx-modules/bolt-expressions) directly from `reaper`.





&nbsp;


## Var

A handy solution to scores/storages when you don't feel like declaring the name!

At the core they're `expressions` from [bolt_expressions](https://github.com/rx-modules/bolt-expressions).

All vars get wiped upon reloading.


<details>
<summary>Details</summary><p>

```py
# Declaring is fairly simple:
	tmp_hp = Var()

# You can also limit the variable to an 'int_var' for more optimization:
	tmp_hp = Var(int_var=True)

# Comparing scores is a bit trickier, you need to use the `.holder` & `.obj` attributes.
	tmp_hp = var(int_var=True)

	if score tmp_hp.holder tmp_hp.obj matches 1..:
		...
```
			     
</p></details>


### Code example:
```py
from _:reaper import Var, Scoreboard

tmp = Var(int_var=True)
my_score = Scoreboard('my_score')

def  very_primitive_power(val): 		# here Var() lets us skip creating a
	tmp = val 				# new scoreboard for this operation
						# and preserves the $old value,
return tmp * tmp + 10  				# since we're using a 'tmp'

function ./simple_example:
	my_score['$old'] = 10
	my_score['$new'] = very_primitive_power(my_score['$old'])
```

&nbsp;


## Object

Fully functional custom `nbt` support for entities. 

This one took ages to figure out.

&nbsp;

**IMPORTANT**: Make sure your entity death `loot table`-s contain this `json` snippet, marking them as a `death_loot_table`. Not doing so will result in severely decreased performance overtime.

```json
{
	"reaper": {
		"death_loot_table": true
	}
}
```

<details>
<summary>Death Loot Table Example</summary><p>

```json
{
	"reaper": {
		"death_loot_table": true
	}

	"type":  "minecraft:entity",
		"pools": []
}
```
</p></details>

&nbsp;

<details>
<summary>Details</summary><p>

```py
# Declaring an object (make sure it's declared AS the right context):
	as @e[type=zombie]:
		fake_hp = Object('stat.fake_hp')   # don't forget the nbt path!

# Data operations:
	Object(path).get()
	Object(path).set(value)
	Object(path).merge(value)
	Object(path).append(value)
	Object(path).prepend(value)
	Object(path).insert(index, value)
```
</p></details>


### Code example:
```py
from _:reaper import Object
from nbtlib import Byte

as @e[type=spider]:
	a_hp = Object('stat.artificial_hp')
	a_hp.set(5000)
	a_hp.set(3 * a_hp.get())
	fake_inv = Object('inventory')

	fake_inv.set('')
	fake_inv.merge('[]')
	fake_inv.append('{"id": "minecraft:bow", "Count": Byte(1)}')
	fake_inv.prepend('{"id": "minecraft:arrow", "Count": Byte(64)}')
	fake_inv.insert(1, '{"id": "minecraft:iron_sword", "Count": Byte(1)}')
	
	Object('inventory[2]').delete()
```

&nbsp;


## Sleep

Simple way of creating intervals between executions without scoreboards.

Fully supports entities.

<details>
<summary>Details</summary><p>

```py
# Sleep(interval) should be used along the 'with' statement.
	with Sleep(30):
		say Delayed hello!

# 'interval' is the time in ticks before code execution.
# In this example, 'interval' was 30 ticks.

# Running Sleep() multiple times before the previous one finished will just append it
# and each execute after it's assigned interval, as you may expect
```
</p></details>

### Code example:
```py
from _:reaper import Sleep

function ./sleepy_sleep:
	as @e[type=zombie]:
		say Hello World!

		with Sleep(40):
			say Hello World!... 2 seconds later!
			
			with Sleep(60):
				say Hello World!... 5 seconds later!
```

&nbsp;


## Loop

Provides you with the ability to create fully working loops.

Fully supports entities.

<details>
<summary>Details</summary><p>

```py
# Loop(count, interval) should be used along the 'with' statement.
	with Loop(5, 20):
		say I run every 1 second, total of 5 times!

# 'count' is the amount of times the loop will run.
# In this example, 'count' was 5.

# 'interval' is the time in ticks before the loop executes code the next time.
# In this example, 'interval' was 20 ticks.


# Recursion is supported, just set 'interval' to 0:
	with Loop(3, 0):
		say Recursive!

# Endless loops are also supported, just set 'count' to 0:
	with Loop(0, 40):
		say I run endlessly every 2 seconds!


# Running the same Loop() before it's finished
# will overwrite and restart the loop.
```
</p></details>

### Code example:
```py
from _:reaper import Loop

function ./zombie_fire_trail:
	as @e[type=zombie]:
		with Loop(32, 5):
			particle flame ~ ~ ~

function ./say_hi_5_times:
	with Loop(5, 0):
		say hi
```

&nbsp;


## Halt

Stops execution of a [Loop](#loop) and all code following it when it's called.

<details>
<summary>Details</summary><p>

```py
# All the code afterwards will simply not execute when it's called, 
# even inside the same mcfunction.

# "If it is stupid but it works, it isn't stupid" :P
```
</p></details>

### Code example:
```py
from _:reaper import Loop, Halt

function ./infinity_loop:
	with Loop(0, 40):
		say I get called on this cycle no matter what!

		if data entity @s SelectedItem:
			Halt()			# will break this loop and stop it
		
		say I wont be called on this cycle that player holds an item
```

&nbsp;


## Cmd

Pythonized version of minecraft commands. Their advantage is being passable as function arguments.

<details>
<summary>Details</summary><p>

```py
# Current commands:
	# tag:
		Cmd.tag(name)
		Cmd.untag(name)

	# time:
		Cmd.get_time(mode) 	# 'day', 'gametime', 'daytime' (DEFAULT)
		Cmd.set_time()
```
</p></details>

### Code example:
```py
from _:reaper import Cmd

function ./pythonized_commands:
	as @p:
		Cmd.tag('some_tag')
		Cmd.untag('some_tag')
	
	Cmd.set_time(666)
	Cmd.set_time(get_time() - 69)
```

&nbsp;


## Array

Useful higher level tools for array operations.

<details>
<summary>Details</summary><p>

```py
# Currently supported operations:
	Array.min(target_array)
	Array.min_index(target_array)
	Array.del_index(target_array, index)
```
</p></details>

### Code example:
```py
from _:reaper import Data, Scoreboard, Array

my_array = Data.storage('test:test').my_array
my_score = Scoreboard('test')['$my_score']

function ./array_operations:
	my_array = [666, 69, 420, 1337]
	
	# retrieves the smallest int
	my_score = Array.min(my_array)				# out: 69

	
	# retrieves the index of the smallest int
	my_score = Array.min_index(my_array)			# out: 1
	
	
	# deletes the value at a specified index
	Array.del_index(my_array, 3)				# deleted: 420

	# supports dynamic index
	Array.del_index(my_array, Array.min_index(my_array)) 	# deleted: 69

```

&nbsp;


## File

Simple interface for reading & writing files.

<details>
<summary>Details</summary><p>

```py
# File(path()
# It can be used alongside the 'with' statement similarly to python's 'open()'
# If for whatever reason you're not using 'with', don't forget to close the file lol.

# The working directory is the project directory
```
</p></details>

### Code example:
```py
from _:reaper import File

with File('random_file.txt') as f:
	f.write('hello, here go contents of the file')

with File('config/bosses/kamikaze_sheep.json') as f:
	sheep_boss_data = f.read()
```






