# Wall jump component

## Introduction

A wall jump component for a `CharacterBody3D` that uses a `WallDetector` and a `JumpComponent` to bounce off walls. The features are:

- Boolean to reset the number of remaining jumps on the `JumpComponent`.
- Wall jump direction as weighted sum of `Vector3.UP` and `collision.normal`.

## Description

TBD

## Code

```
extends Node

@export var reset_jumps := true
@export_range(0,1) var jump_angle := 0.5

@export var wall_detector: WallDetector
@export var jump_component: JumpComponent

var character_body: CharacterBody3D

func _ready() -> void:
	pass # Replace with function body.

func initialize(_character_body: CharacterBody3D):
	character_body = _character_body

func _physics_process(delta: float) -> void:
	if wall_detector.is_on_wall() and not character_body.is_on_floor():
		# Reset jumps on wall
		if reset_jumps: 
			jump_component.reset_jumps()

		# Wall jump direction
		var wall_jump_direction: Vector3
		var normal = wall_detector.get_normal()
		# Jump in angle
		wall_jump_direction = (jump_angle*Vector3.UP + (1-jump_angle)*normal).normalized()
		jump_component.set_jump_direction(wall_jump_direction)
		
	else:
		jump_component.reset_jump_direction()

```