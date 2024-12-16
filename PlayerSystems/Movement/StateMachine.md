# State Machines in Godot


## Index

TBD

## StateMachine

The StateMachine class handles the transitions between States and executes the current state's updates.

Needs to keep track of the `current_state`, and, optionally, the `previous_state` or `next_state`. Initializes the set of States assuming that these are the children of the `StateMachine` root node. 

Then, defines the methods to run the State's methods `process()`, `physics_process()` and `handle_input()`.

```
#Pseudocode
class_name StateMachine extends Node

var states: Array[State]

var current_state: State
var previous_state: State

func _ready():
	for state in self.get_children():
		if state is State:
			states.append(state)
		

func _transition(new_state: State) -> void:
	#1. Call the exit() method of the current state.
	current_state.exit()

	#(Optionally, update previous state)
	previous_state = current_state

	#2. Update the current state with the new state.
	current_state = new_state

	#3. Call the enter() method of the new state.
	current_state.enter()



func _unhandled_input(event: InputEvent) -> void:
	# Run whatever the state handle_input() implements
	current_state.handle_input(event)


func _process(delta: float) -> void:
	# Run the state's process()
	current_state.process(delta)


func _physics_process(delta: float) -> void:
	# Run the state's physics_process()
	current_state.physics_process(delta)

```