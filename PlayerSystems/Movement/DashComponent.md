# Dash component

## Introduction

A dash component for a `CharacterBody3D` based player with these features:

- Dash as horizontal jump
- N number of dashes (two cooldowns)
- Dash/Superdash distinction
- Buffer time
- Dash in input or forward direction

## Description

Two cooldowns to avoid spamming dash:
- A short cooldown in between consecutive dashes, while dash_count > num_dashes.
- A long cooldown to reset the dash_count.

## Code

```
extends Node

class_name DashComponent

@export var dash_speed := 8.0
@export var superdash_speed := 25.0
@export var dash_duration := 0.15
@export var num_dashes: int = 3

@export_group("Cooldowns")
@export var dash_cooldown_time := 0.4
@export var reset_cooldown_time := 1.0
@export var input_threshold_time := 0.5

@export_group("Input buffer")
@export var buffer_time := 0.3

@onready var dash_timer: Timer = $DashTimer
@onready var cooldown_timer: Timer = $CooldownTimer
@onready var reset_timer: Timer = $ResetTimer
@onready var buffer_timer: Timer = $BufferTimer

@onready var dash_sfx: AudioStreamPlayer = $DashSFX

# Flags
var is_dashing := false
var can_dash := true
var button_pressed := false
var pressed_time := 0.0
var wants_dash := false

# Signal dash count
signal dash_count_changed

var character_body: CharacterBody3D
var initial_velocity: Vector3
@onready var cam: Camera3D

var dash_direction := Vector3.ZERO
var dash_count := 0

func initialize(_character_body: CharacterBody3D):
	character_body = _character_body
	
func _ready() -> void:
	cam = get_viewport().get_camera_3d()
	reset_dashes()

func _physics_process(delta: float) -> void:
	if button_pressed:
		pressed_time += delta
		
	if wants_dash and can_dash and dash_count < num_dashes:
		dash()
	if is_dashing:
		var speed = superdash_speed if dash_count == num_dashes else dash_speed
		character_body.velocity.x = initial_velocity.x + dash_direction.x * speed
		character_body.velocity.z = initial_velocity.z + dash_direction.z * speed
		

func dash():
	"""Updates the dash count, enables VFX and SFX and gets the dash direction
	as either the input direction or the forward direction of the character_body."""
	# Update dash count
	dash_count += 1
	dash_count_changed.emit(dash_count)
	dash_sfx.play()
	# Get dash direction
	var input_dir = get_input_direction()
	dash_direction = input_dir if input_dir else -character_body.global_transform.basis.z
	initial_velocity = Vector3(character_body.velocity.x, 0.0, character_body.velocity.z)

	# Start timers
	cooldown_timer.start(dash_cooldown_time)
	reset_timer.start(reset_cooldown_time)
	dash_timer.start(dash_duration)
	
	# Set flags
	can_dash = false
	wants_dash = false
	is_dashing = true
	
func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("dash_run"):
		button_pressed = true

	if event.is_action_released("dash_run"):
		button_pressed = false
		#Dash
		if pressed_time < input_threshold_time:
			wants_dash = true
			buffer_timer.start(buffer_time)

		pressed_time = 0.0

func get_input_direction() -> Vector3:
	var input_dir = Input.get_vector("move_left", "move_right", "move_forward", "move_back")
	
	var direction := Vector3.ZERO
	direction = Vector3(
		input_dir.x,
		0.0,
		input_dir.y
	).rotated(Vector3.UP, cam.rotation.y) #Relative to cam
	
	return direction

func reset_dashes():
	dash_count = 0
	dash_count_changed.emit(dash_count)

func _on_cooldown_timer_timeout() -> void:
	can_dash = true

func _on_reset_timer_timeout() -> void:
	reset_dashes()

func _on_dash_timer_timeout() -> void:
	is_dashing = false

func _on_buffer_timer_timeout() -> void:
	wants_dash = false

```