# Wall detector

## Introduction

A generic wall detector component using children `Raycast3D` nodes. The `Raycast3D` nodes must be set manually in the desired directions. In the component, as `@export` variables, you can set:

- Raycast length
- Collision mask

## Description


## Code

```
extends Node3D

class_name WallDetector

var raycasts: Array[RayCast3D]

@export var raycast_length := 1.0
@export_flags_3d_physics var collision_mask

var _is_on_wall: bool = false : get = is_on_wall
var normal: Vector3 : get = get_normal
var point: Vector3 : get = get_point
var collider: Node3D : get = get_collider

func _ready() -> void:
	for child in get_children():
		if child is RayCast3D:
			raycasts.append(child)

	_initialize_raycasts()

func _initialize_raycasts():
	for raycast in raycasts:
		raycast.target_position *= raycast_length
		raycast.set_collision_mask(collision_mask)

func _physics_process(delta):
	for raycast in raycasts:
		if raycast.is_colliding():
			_is_on_wall = true
			_set_result(raycast)
			break
		_is_on_wall = false

func _set_result(raycast: RayCast3D):
	""" Sets the collider, collision point and collision normal
	from the raycast query result."""
	collider = raycast.get_collider()
	point = raycast.get_collision_point()
	normal = raycast.get_collision_normal()

func get_point():
	return point
func get_normal():
	return normal
func get_collider():
	return collider
func is_on_wall():
	return _is_on_wall

```