# Glide component

## Introduction

A glide component that modifies the gravity value of a `GravityComponent` when the jump button is held.

- Glide gravity

## Description


## Code

```
extends Node

@export var glide_gravity := 3.0
@export var gravity_component: GravityComponent

@export_group("Input")
@export var hold_time_threshold := 0.5

@export_group("SFX")
@onready var glide_sfx: AudioStreamPlayer = $GlideSFX

var button_pressed := false
var pressed_time := 0.0
var is_gliding := false


func _physics_process(delta: float) -> void:
	if button_pressed:
		pressed_time += delta
		if (pressed_time > hold_time_threshold) and not is_gliding:
			#print_debug("GLIDE!")
			is_gliding = true
			gravity_component.set_gravity(glide_gravity)
			glide_sfx.play()
			

func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("jump"):
		button_pressed = true
	if event.is_action_released("jump"):
		button_pressed = false
		pressed_time = 0.0
		if is_gliding:
			glide_sfx.stop()
			#print("STOP GLIDING")
			is_gliding = false
			gravity_component.reset_gravity()

```