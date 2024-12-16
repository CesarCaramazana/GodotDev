# Jump component

## Introduction

A jump component for a `CharacterBody3D` based player with these features:

- N number of jumps
- Jump/Superjump distinction
- Cooldown
- Buffer time

## Description

TBD

## Code

```
extends Node

class_name JumpComponent

@export var num_jumps : int = 3
@export var jump_force := 10.0
@export var superjump_force := 18.0
@export var jump_direction: Vector3 = Vector3.UP : set = set_jump_direction

@export var cooldown_time := 0.3
@onready var cooldown_timer: Timer = $CooldownTimer

@export_category("Jump buffer")
@onready var buffer_timer: Timer = $BufferTimer
@export var buffer_time := 0.2

@onready var jump_sfx: AudioStreamPlayer = $JumpSFX

var can_jump: bool = true
var wants_jump: bool = false
var jump_count: int = 0
signal jump_count_changed

var character_body: CharacterBody3D

func _ready() -> void:
	reset_jumps()

func initialize(_character_body: CharacterBody3D):
	character_body = _character_body

func _physics_process(_delta: float) -> void:
	if wants_jump and can_jump and jump_count < num_jumps:
		jump()

	if character_body.is_on_floor() and can_jump:
		reset_jumps()
	
func jump():
	jump_sfx.play()
	jump_count += 1
	jump_count_changed.emit(jump_count)
	#print_debug("Jump count: " + str(jump_count) + "/" + str(num_jumps))
	var _jump_force = superjump_force if jump_count == num_jumps else jump_force

	if jump_direction != Vector3.UP:
		var jump_velocity = jump_direction * _jump_force
		character_body.velocity += jump_velocity
	else:
		character_body.velocity.y = _jump_force
	can_jump = false
	wants_jump = false
	cooldown_timer.start(cooldown_time)

func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("jump"):
		buffer_timer.stop()
		wants_jump = true
		buffer_timer.start(buffer_time)
	
func reset_jumps():
	jump_count = 0
	jump_count_changed.emit(jump_count)
	can_jump = true

func reset_jump_direction():
	jump_direction = Vector3.UP
func set_jump_direction(_direction: Vector3):
	jump_direction = _direction

# Cooldown and buffer timers
func _on_cooldown_timer_timeout() -> void:
	can_jump = true

func _on_buffer_timer_timeout() -> void:
	wants_jump = false

```