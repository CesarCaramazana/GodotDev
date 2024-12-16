# Gravity component

## Introduction

A gravity component for `CharacterBody3D` nodes with the following features:

- Base gravity value
- Terminal velocity
- Coyote time
- Gravity direction

## Description

TBD

## Code

```
extends Node

class_name GravityComponent

@export var base_gravity := 12.0
@export var terminal_velocity := 200
@export var gravity_direction := Vector3.DOWN

@export var coyote_time := 0.1
@onready var coyote_timer: Timer = $CoyoteTimer

var character_body: CharacterBody3D

var can_fall: bool = false
var gravity = base_gravity : set = set_gravity, get = get_gravity

func initialize(_character_body: CharacterBody3D):
	character_body = _character_body


func _physics_process(delta: float) -> void:
	if character_body.is_on_floor():
		can_fall = false
		coyote_timer.stop()
	else:
		if coyote_timer.is_stopped():
			coyote_timer.start(coyote_time)

	if can_fall and gravity > 0:
		character_body.velocity += gravity_direction * gravity * delta
		clamp(character_body.velocity.y, 0, terminal_velocity)

func reset_gravity():
	set_gravity(base_gravity)

func set_gravity(_gravity):
	gravity = _gravity

func get_gravity():
	return gravity

func _on_coyote_timer_timeout() -> void:
	can_fall = true

```