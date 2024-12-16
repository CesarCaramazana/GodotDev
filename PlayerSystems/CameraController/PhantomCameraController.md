# PhatomCamera controller

## Introduction

A Third-Person camera controller for a `PhantomCamera3D` node from the [Phatom Camera addon](https://github.com/ramokz/phantom-camera).



## Description

WIP

## Code

```
extends Node

@onready var pcam: PhantomCamera3D

@export var sensitivity: float = 3
@export var mouse_sensitivity := 0.2

@export var min_pitch: float = -89.9
@export var max_pitch: float = 50

@export var min_yaw: float = 0
@export var max_yaw: float = 360

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	pcam = owner.get_node("%PlayerPhantomCamera3D")
	#Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)
	
func _physics_process(_delta: float) -> void:
	var input_dir = get_input_direction()
	if input_dir:
		_rotate_camera(input_dir)

func _unhandled_input(event: InputEvent) -> void:
	_set_pcam_rotation(event)

func _input(event: InputEvent) -> void:
	if event is InputEventMouseMotion:
		_set_pcam_rotation(event)

func get_input_direction() -> Vector2:
	var input_dir = Vector2.ZERO
	input_dir = Input.get_vector("camera_left", "camera_right", "camera_up", "camera_down")
	return input_dir
	
func _rotate_camera(input_dir: Vector2):
	var pcam_rotation_degrees: Vector3
	pcam_rotation_degrees = pcam.get_third_person_rotation_degrees()		
	# Change the X rotation
	pcam_rotation_degrees.x -= input_dir.y * sensitivity
	# Clamp the rotation in the X axis so it go over or under the target
	pcam_rotation_degrees.x = clampf(pcam_rotation_degrees.x, min_pitch, max_pitch)
	# Change the Y rotation value
	pcam_rotation_degrees.y -= input_dir.x * sensitivity
	# Sets the rotation to fully loop around its target, but witout going below or exceeding 0 and 360 degrees respectively
	pcam_rotation_degrees.y = wrapf(pcam_rotation_degrees.y, min_yaw, max_yaw)
	# Change the SpringArm3D node's rotation and rotate around its target
	pcam.set_third_person_rotation_degrees(pcam_rotation_degrees)

func _set_pcam_rotation(event: InputEvent) -> void:
	if event is InputEventMouseMotion:
		var pcam_rotation_degrees: Vector3
		# Assigns the current 3D rotation of the SpringArm3D node - so it starts off where it is in the editor
		pcam_rotation_degrees = pcam.get_third_person_rotation_degrees()
		# Change the X rotation
		pcam_rotation_degrees.x -= event.relative.y * mouse_sensitivity
		# Clamp the rotation in the X axis so it go over or under the target
		pcam_rotation_degrees.x = clampf(pcam_rotation_degrees.x, min_pitch, max_pitch)
		# Change the Y rotation value
		pcam_rotation_degrees.y -= event.relative.x * mouse_sensitivity
		# Sets the rotation to fully loop around its target, but witout going below or exceeding 0 and 360 degrees respectively
		pcam_rotation_degrees.y = wrapf(pcam_rotation_degrees.y, min_yaw, max_yaw)
		# Change the SpringArm3D node's rotation and rotate around its target
		pcam.set_third_person_rotation_degrees(pcam_rotation_degrees)

```