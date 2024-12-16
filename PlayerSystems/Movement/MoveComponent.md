# Player horizontal movement

## Introduction

A movement component for player horizontal movement with features:

- Walk/Sprint speed
- Acceleration model
- Ground/Air distinction
- Camera-relative 3D movement
- Rotation towards forward direction


## Description

This component must be attached as a child node of a `CharacterBody3D`. In the `CharacterBody3D` script, the `initialize()` method must be called, passing a reference to `self`.

## Code

```
extends Node

class_name MoveComponent

@export var speed := 10.0
@export var rotation_speed := 15.0

@export_group("Ground movement")
@export var ground_acceleration := 15.0
@export var ground_friction := 25.0

@export_group("Air movement")
@export var air_acceleration := 5.0
@export var air_friction := 3.0

@export_group("Sprint")
@export var sprint_multiplier := 2.0
@export var input_threshold_time := 0.5

@onready var cam: Camera3D
var character_body: CharacterBody3D

var last_direction := Vector3.FORWARD
var is_sprinting : bool = false
var button_pressed: bool = false
var pressed_time := 0.0

func _ready() -> void:
	cam = get_viewport().get_camera_3d()

func initialize(_character_body: CharacterBody3D):
	character_body = _character_body
	
	
func _physics_process(delta: float) -> void:	
	var direction := get_input_direction()
	var target_speed := 0.0	
	var change_speed := ground_friction if character_body.is_on_floor() else air_friction
	
	if direction:
		# Sprint on button held
		if button_pressed:
			pressed_time += delta
			is_sprinting = pressed_time > input_threshold_time
		# Calculate target speed
		var speed_multiplier = sprint_multiplier if is_sprinting else 1.0
		target_speed = speed * speed_multiplier
		change_speed = ground_acceleration if character_body.is_on_floor() else air_acceleration
		last_direction = direction
		
		# Face moving direction
		character_body.rotation.y = lerp_angle(
			character_body.rotation.y,
			atan2(-last_direction.x, -last_direction.z),
			rotation_speed * delta
		)

	# Apply velocity change
	character_body.velocity.x = move_toward(
		character_body.velocity.x,
		direction.x * target_speed,
		change_speed * delta
	)
	character_body.velocity.z = move_toward(
		character_body.velocity.z,
		direction.z * target_speed,
		change_speed * delta
	)

	

func get_input_direction() -> Vector3:
	var input_dir = Vector2.ZERO
	input_dir = Input.get_vector("move_left", "move_right", "move_forward", "move_back")
	
	var direction := Vector3.ZERO
	direction = Vector3(
		input_dir.x,
		0.0,
		input_dir.y
	).rotated(Vector3.UP, cam.rotation.y) #Relative to cam
	
	return direction

func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("dash_run"):
		button_pressed = true
	if event.is_action_released("dash_run"):
		button_pressed = false
		pressed_time = 0.0
		is_sprinting = false

```