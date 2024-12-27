```gdscript
func _on_spot_area_body_entered(body):
	if body.name == "player":
		is_player_in_area = true
		$LeftSide/Timer1.start()
		%Timer2.start()

func _on_spot_area_body_exited(body):
	if body.name == "player":
		is_player_in_area = false
		$RightSide/RayCast3D.enabled = false
		$RightSide/RayCast3D2.enabled = false
		$RightSide/RayCast3D3.enabled = false
		$LeftSide/RayCast3D.enabled = false
		$LeftSide/RayCast3D2.enabled = false
		$LeftSide/RayCast3D3.enabled = false

func _on_timer_2_timeout() -> void:
	%ExplosionScene4/Explosion1.emitting = false
	%ExplosionScene4/Explosion2.emitting = false
	
	%ExplosionScene5/Explosion1.emitting = false
	%ExplosionScene5/Explosion2.emitting = false
		
	%ExplosionScene6/Explosion1.emitting = false
	%ExplosionScene6/Explosion2.emitting = false
	
	$RightSide/RayCast3D.enabled = true
	$RightSide/RayCast3D2.enabled = true
	$RightSide/RayCast3D3.enabled = true

func _on_timer_1_timeout() -> void:
	%ExplosionScene/Explosion1.emitting = false
	%ExplosionScene/Explosion2.emitting = false
	
	%ExplosionScene2/Explosion1.emitting = false
	%ExplosionScene2/Explosion2.emitting = false
		
	%ExplosionScene3/Explosion1.emitting = false
	%ExplosionScene3/Explosion2.emitting = false
	
	$LeftSide/RayCast3D.enabled = true
	$LeftSide/RayCast3D2.enabled = true
	$LeftSide/RayCast3D3.enabled = true
```
```gdscript
func _process(delta):
	move_in_circle(delta)
	if bot_death == false:
		deal_damage()


func deal_damage():
# левая сторона
	if $LeftSide/RayCast3D.get_collider():#
		print($LeftSide/RayCast3D.get_collider())
		print('$LeftSide/RayCast3D')
		var collider = $LeftSide/RayCast3D.get_collider()
		if collider.is_in_group("Player"):
			collider.get_damage_player()
	
	
	if $LeftSide/RayCast3D2.get_collider():#
		print('$LeftSide/RayCast3D2')
		var collider = $LeftSide/RayCast3D2.get_collider()
		if collider.is_in_group("Player"):
			collider.get_damage_player()
			
	if $LeftSide/RayCast3D3.get_collider():#
		print('$LeftSide/RayCast3D3')
		var collider = $LeftSide/RayCast3D3.get_collider()
		if collider.is_in_group("Player"):
			collider.get_damage_player()
# ------------------------------------

# правая сторона
	if $RightSide/RayCast3D.get_collider():
		#$RightSide/RayCast3D.get_collider(Object.has_method("death"))
		print("")
		var collider = $RightSide/RayCast3D.get_collider()
		if collider.is_in_group("Player"):
			collider.get_damage_player()
		
	
	if $RightSide/RayCast3D2.get_collider(): #
		print('$RightSide/RayCast3D2')
		var collider = $RightSide/RayCast3D2.get_collider()
		if collider.is_in_group("Player"):
			collider.get_damage_player()
	
	if $RightSide/RayCast3D3.get_collider(): #
		print('$RightSide/RayCast3D3')
		var collider = $RightSide/RayCast3D3.get_collider()
		if collider.is_in_group("Player"):
			collider.get_damage_player()
```
