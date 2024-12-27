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
