# Шутер

# Создание глобального освещения

Создаем узел `WorldEnvironment`. Настройка самого узла показана ниже

![image](https://github.com/user-attachments/assets/4db70300-fbe8-4633-b01f-4718e407b860)

# Создание базовой платформы для персонажа

Создадим мини платформу для изначальных тестов камеры персонажа, состоять она будет из следующий узлов<br>
* MeshInstance3D
  * StaticBody3D
    * CollisionShape3D

# Персонаж

Создаем новую сцену, основным узлом будет CharacterBody3D. Игра от первого лица, поэтому модельку нам добавлять не нужно, в будущем мы добавим руки персонажу. но пока, что остановимяся на камере и коллизии

![image](https://github.com/user-attachments/assets/8e8cc256-8011-4e6f-bd8b-54414fbb3d92)

Добавляем игрока на сцену и запускаем, будет выглядеть примерно так

![image](https://github.com/user-attachments/assets/e609e7c9-084d-4994-96e6-c72bcfd56ffb)

Начинаем программировать камеру и движиение игрока. Для начала поместим камеру в отдельный узел Node3D в игроке

![image](https://github.com/user-attachments/assets/75ea3875-8d14-4b74-9810-c820f5ea8558)

Теперь сбрасываем координаты у камеры и перемещаем Node3D, вместе с ним будет перемещатся камера

Для начала уберем видимость курсор мыши

```gdscript
func _ready() -> void:
	Input.mouse_mode = Input.MOUSE_MODE_CAPTURED
```

Создаем переменные которые пригодятся нам для создания камеры

```gdscript
var mouse_input : bool = false # включение/отключение ввода мыши
var tilt_input : float # для ограничения камеры вверх/вниз
var mouse_rotation : Vector3 # изменение вращения мыши в пространстве
var camera_rotation : Vector3 # изменение вращения в пространстве

@export var _camera_contoller : Camera3D
```

Далее для камеры мы будет использовать встроенный метод `_unhandled_input`, давайте разберемся в чем разница между `_unhandled_input` и `_input`. Для этого создадим тестовую сцену состоящую из следующих узлов

![image](https://github.com/user-attachments/assets/7a8a73d5-760f-4f5e-8c72-6a70ce642006)

А в коде поочереднем посмотирм разницу в инпутах

![image](https://github.com/user-attachments/assets/7e49df75-7049-4135-a79f-f33de1cab6fa)

Как можно увидеть _input работает всегда, а _unhandled_input при вводе текста перестает считывать движение мышью внутри LineEdit

Создаем метод `_unhandled_input` и делаем проверку на то, что если mouse_input == true, тогда мы изменяем значение y в зависимости от того куда ведем мышь

```gdscript
func _unhandled_input(event: InputEvent) -> void:
	mouse_input = event is InputEventMouseMotion and Input.mouse_mode == Input.MOUSE_MODE_CAPTURED
	if mouse_input:
		tilt_input = -event.relative.y * 0.02
	print(Vector2(0,tilt_input))
```

В данной функции мы увеличиваем mouse_rotation на tilt_input, после чего передаем mouse_rotation в camera_rotation. Далее изменяем basis у камеры, однако он отвечает за 3 параметра - position, rotation, scale. Нам же нужно изменять лишь rotation. Для этого создаем новй экземпляр Basis, а у него метод from_euler который позволить сделать лишь изменение вращения

```gdscript
func update_camera(delta):
	mouse_rotation.x += tilt_input * delta 
	camera_rotation = Vector3(mouse_rotation.x, 0.0, 0.0)
	_camera_contoller.transform.basis = Basis.from_euler(camera_rotation)
```

Нам остается лишь добавить вызов нашего метода в `_physics_process`

```gdscript
func _physics_process(delta: float) -> void:
	update_camera(delta)
```

Теперь нам нужно сбрасывать tilt_input когда не происходить инпута мышью

```gdscript
func update_camera(delta):
	...
	tilt_input = 0.0
```

Теперь сделаем движение камеров влево вправо, для этого создадим новую переменную

```gdscript
var rotation_input : float
```

Добавляем rotation_input в _unhandled_input

```gdscript
func _unhandled_input(event: InputEvent) -> void:
	...
	if mouse_input:
		rotation_input = -event.relative.x * 0.2
```

Для движение камерыф влево/вправо нам осталось изменить метод update_camera(), добавив изменение ротации по y

```gdscript
func update_camera(delta):
	...
	mouse_rotation.y += rotation_input * delta 
	...
```

Также необходимо, чтобы при движении камерой игрок также поворачивался вместе с ней, для этого создадим еще одну переменную

```gdscript
var player_rotation : Vector3
```

И изменим метод update_camera()

```gdscript
func update_camera(delta)
	...
	player_rotation = Vector3(0.0,mouse_rotation.y,0.0)
	...
	global_transform.basis = Basis.from_euler(player_rotation)
	rotation_input = 0.0
```

Теперь ограничим вращение камеры, чтобы она не вращалась у нас на все 360 градусов. Для этого создадим новые экспортные переменные

```gdscript
@export var _tilt_down_limit := deg_to_rad(-90.0)
@export var _tilt_up_limit := deg_to_rad(90.0)
@export var mouse_sensitivity : float = 4.5
```

Для начала изменим _unhandled_input, добавив вместо числа переменную mouse_sensitivity

```gdscript
func _unhandled_input(event: InputEvent) -> void:
	...
	if mouse_input:
		rotation_input = -event.relative.x * mouse_sensitivity
		tilt_input = -event.relative.y * mouse_sensitivity
```

Теперь делаем ограничение поворота камеры

```gdscript
func update_camera(delta):
	...
	mouse_rotation.x = clamp(mouse_rotation.x, _tilt_down_limit, _tilt_up_limit)
```

Также ограничим вращение камеры по оси z, во избежании каких-либо багов
```gdscript
func update_camera(delta):
	...
	_camera_contoller.rotation.z = 0.0
```

Тестовый уровень можно накидывать из CSGBox3D, можно использовать аддон 
![image](https://github.com/user-attachments/assets/c7f7391a-9d45-48b6-854e-d2e374ae6933)

# Создание основной карты

Создание основной карты можно сделать при помощи GridMap (как делали в платформере)

Можно скачать готовую карту с сайтов (например ScetchFab)

# Создание оружия персонажа и их анимаций

Для начала качаем любую понравившуюся модельку 

![image](https://github.com/user-attachments/assets/5e4eca95-3354-488e-9d36-718353ae1524)


Создаем отдельную папку для оружия и перекидываем его в наш проект

![image](https://github.com/user-attachments/assets/e6aaeb73-76e9-4680-bd4d-062a958ef5aa)

Открываем скаченный файл как новую унаследованную сцену

![image](https://github.com/user-attachments/assets/d1d71173-04cf-40de-85ec-a99a9cf1a212)

Можн сразу посмотреть анимации и если все устраивает нажимаем открыть в редакторе

![image](https://github.com/user-attachments/assets/a67ea330-5c65-4b5e-b759-2ff3c2d6f624)

После чео сохраняем нашу модельку как сценуэ. Теперь при необохимости мы можем редактировать анимации по времени, скорости и т.п.

Добавляем модельку к камере

![image](https://github.com/user-attachments/assets/5ade23ce-c821-499c-b39f-bb671ba293b1)

И настраиваем то как игрок будет видеть оружие. Для удобства можно выбрать 2 окна и в одном открыть предпросмотр камеры, а в другом настраивать само положение

![image](https://github.com/user-attachments/assets/63d3a739-b5a7-488b-bb4c-2c3681b2dc1c)

![image](https://github.com/user-attachments/assets/27cf652a-8565-4a89-9e58-3fdcc18ef43c)

Если встроенное движение удаляли нужно будет его сделать, если оставляли можно запустить игру и посмотреть как сейчас это выглядит

![image](https://github.com/user-attachments/assets/34e511c7-203b-4f1d-8ffa-29e15af8c91a)


## Создание выстрела и анимаций

Для выстрела мы будем использовать RayCast, добавляем его к камере

![image](https://github.com/user-attachments/assets/0b3290ca-eff1-49d3-99ed-4cfbf2c90989)

Для проверки работоспособности raycast можно включить отладку

![image](https://github.com/user-attachments/assets/aa9f9306-0e9b-4d47-b508-4c474bcf30c6)

raycast при наведении на объекты должен становится красным

Переходим к созданию выстрела

```gdscript
	# Анимация выстрела
	if Input.is_action_just_pressed("shoot") and ray_cast.is_colliding():
		animations.play("Rig|AK_Shot")
	# Анимация хотьбы
	if velocity and animations.current_animation != "Rig|AK_Shot":
		animations.play("Rig|AK_Walk")
	elif velocity == Vector3.ZERO and animations.current_animation != "Rig|AK_Shot" or animations.current_animation != "AK_shoot_auto":
		animations.play("RESET")
```

Начнем создания выстрела от пуль, расмер пусть ученик выберет сам

Узлы

![image](https://github.com/user-attachments/assets/07ee05ce-16eb-4e23-977e-74f78d1cfe27)

Скрипт

![image](https://github.com/user-attachments/assets/50296673-1173-4381-ae50-17a19fcef517)

Анимация

![image](https://github.com/user-attachments/assets/bf547459-7901-456b-8769-205fe89be800)

Переходим в скрипт игрока, сделась создаем переменную в которой будет хранится сцена от следа пуль и создаем функцию по созданию клонов

```gdscript
@onready var decal = preload("res://guns/decal.tscn")

func decal_spawn():
	var col_point = ray_cast.get_collision_point()
	var b = decal.instantiate()
	ray_cast.get_collider().add_child(b)
	b.global_transform.origin = col_point
```

А также не забываем добавить ее в функцию стрельбы

```gdscript
	if Input.is_action_just_pressed("shoot") and ray_cast.is_colliding():
		animations.play("Rig|AK_Shot")
		decal_spawn()
```

Последнее на чем мы поработаем это освещение

Можно добавить фонарик, предварительно на уровне уменьшив энергию у неба. У игрока добавляем SpotLight3D

![image](https://github.com/user-attachments/assets/7169688d-4917-4943-8f38-b2042837fbea)

Примерные настройки узла

![image](https://github.com/user-attachments/assets/79140dfc-a57e-41ce-89df-b366d311555c)

Можно подобовлять лампы освещение в коридорах (можно также моргающие)

Сейчас декаль правильно ставиться лишь при стрельбе в пол, для того чтобы это исправить доработаем нашу функцию ее спавна + нужно в инспекторе повернуть декаль по х на -90

```gdscript
func decal_spawn():
	if fire_delay == false:
		var col_nor = ray_cast.get_collision_normal()
		print(col_nor)
		var col_point = ray_cast.get_collision_point()
		var b = decal.instantiate()
		ray_cast.get_collider().add_child(b)
		b.global_transform.origin = col_point
		if col_nor == Vector3.UP:
			b.rotation_degrees.x = 90
		elif col_nor == Vector3.DOWN:
			b.rotation_degrees.x = -90
		elif col_nor != Vector3.UP:
			b.look_at(col_point - col_nor, Vector3(0,1,0))
		fire_delay = true
		await animations.animation_finished
		fire_delay = false
```

Перейдем к созданию врагов

Для начала с сайта mixamo нам нужно выбрать модельку и подобрать для нее анимации (idle, walk, attack, death) в формате fbx.

Сейчас у нас 4 и более моделек, а нам нужна 1 со всеми анимациями. Для этого нам нужно будет сохранить все анимации как анимации

![image](https://github.com/user-attachments/assets/7e5c8e69-16ee-46b1-afd8-828d25489a6d)

![image](https://github.com/user-attachments/assets/cdea4470-e261-47e6-a32f-a6f6e07bec08)

![image](https://github.com/user-attachments/assets/244f8905-cf12-46eb-b5d3-c6b4b6a71cde)

Чтобы доабвить сохраненные анимации в том же меню есть следующая кнопка

![image](https://github.com/user-attachments/assets/ec2a18df-b435-4731-9c03-312af20c64a8)

Коллизию добавляем по стандарту

# Движение бота

Для начала создадим NavigationRegion3D и создаем в нем NavigationMesh

Далее добавляем NavigationAgent3D к врагу

![image](https://github.com/user-attachments/assets/122ae75e-6f9c-48fc-958b-411c6ac124b2)

Добавляем скрипт боту и удаляем все кроме гравитации, и переменную которая будет хранить NavigationAgent3D

```gdscript
@onready var nav: NavigationAgent3D = $NavigationAgent3D
```

А также внутри _physics_process создадим переменную хранящую направление движения

```gdscript
	var direction = Vector3()
```

И последней переменной будет экспортная переменная хранящая цель к которой мы будем двигать врага

```gdscript
@export var target : CharacterBody3D
```

Перейдем к созданию движения

```gdscript
	nav.target_position = target.global_position
	
	direction = nav.get_next_path_position() - global_position
	direction = direction.normalized()
	
	velocity = velocity.lerp(direction*SPEED, 10* delta)
```

Теперь бот ходить за нами однако делает это не всегда смотря на нас, для этого добавим метод look_at()

```gdscript
	look_at(target.global_position, Vector3.UP)
```

Анимация ходьбы и стояния на месте

```gdscript
	if velocity.z or velocity.x:
		$AnimatedSprite3D.play("walk")
	else:
		$AnimatedSprite3D.play("idle")
```

Включении анимации при атаке

Для начала добавить raycast который будет учитывать максиальную длину атаки и в случае коллайда будет срабатывать анимация

![image](https://github.com/user-attachments/assets/23c22221-f352-4ff7-8013-fa1d248e7c74)

```gdscript
	if not $AttackSensor.is_colliding():
		if velocity.z or velocity.x:
			$AnimatedSprite3D.play("walk")
		else:
			$AnimatedSprite3D.play("idle")
	else:
		$AnimatedSprite3D.play("attack")
```

Также сделаем остановку бота при ударе

```gdscript
	if not $AttackSensor.is_colliding():
		velocity = velocity.lerp(direction*SPEED, 10 * delta)
	else:
		velocity = Vector3()
```

Сейчас у нас появляется 1 проблема. бот атакует всё, с чем сталкивается, а нам нужно лишь с игроком, для этого переработаем условие
Само движение
```gdscript
	if not $AttackSensor.is_colliding():
		velocity = velocity.lerp(direction*SPEED, 10 * delta)
	elif $AttackSensor.get_collider().name == "player":
		velocity = Vector3()
```

И анимации
```gdscript
	if not $AttackSensor.is_colliding():
		if velocity.z or velocity.x:
			$AnimatedSprite3D.play("walk")
		else:
			$AnimatedSprite3D.play("idle")
	elif $AttackSensor.get_collider().name == "player":
		$AnimatedSprite3D.play("attack")
```

Добавим хп

```gdscript
func damage():
	randomize()
	hp -= randi_range(5,15)
	print(hp)
```

И нужно ее вызвать у игрока. Добавим ее в функцию decal_spawn()

```gdscript
	if fire_delay == false:
		if ray_cast.get_collider().name == "enemy":
			ray_cast.get_collider().damage()
```

Также можно поменять то, что сейчас выстрелить в воздух нельзя, лишь когда есть объект выстрела
```gdscript
	if Input.is_action_just_pressed("shoot"):
		animations.play("Rig|AK_Shot")
		if ray_cast.is_colliding():
			decal_spawn()
```

Добавим смерть боту, сейчас у него хп просто уходят в минус

```gdscript
func damage():
	randomize()
	hp -= randi_range(5,15)
	print(hp)
	if hp <= 0:
		$AnimatedSprite3D.play("death")
		set_physics_process(false)

```

А также сделаем так, что все предыдущие анимации работают лишь тогда, когда текущая анимация не смерть

```gdscript
	if $AttackSensor.current_animation != "death":
		if not $AttackSensor.is_colliding():
			if velocity.z or velocity.x:[](url)
				$AnimatedSprite3D.play("walk")
			else:
				$AnimatedSprite3D.play("idle")
		elif $AttackSensor.get_collider().name == "player":
			$AnimatedSprite3D.play("attack")
```

Добавим отключение коллизии и рэйкаста при смерти
```gdscript
	if hp <= 0:
		$AnimatedSprite3D.play("death")
		set_physics_process(false)
		$CollisionShape3D.disabled = true
		$AttackSensor.enabled = false
```

Создадим хп у игрока

```gdscript
var hp = 100

func damage():
	randomize()
	hp -= randi_range(5,15)
	print(hp)
	if hp <= 0:
		get_tree().paused = true
```

Теперь где-то нужно добавить вызов этой функции, делать мы это будем в скрипте бота, когда он останавливается для атаки
```gdscript
	elif $AttackSensor.get_collider().name == "player":
		velocity = Vector3()
		target.damage()
```

Сейчас он нас дамажит, но убивает мгновенно. Для того чтобы это пофиксить нам нужно переработать само получение урона, добавим дилей для дамага, и условие его проверки
```gdscript
var damage_delay = 1.3
	...
	elif $AttackSensor.get_collider().name == "player":
		velocity = Vector3()
		damage_delay -= delta
		if damage_delay <= 0:
			target.damage()
			damage_delay = 1.3 * 2
```

Сейчас он стал работать лучше, однако проблемы остались. Если подходить и отходить от бота урон будет производится не в нужный тайминг, из-за того, что наш таймер убавляется всегда, а в изначальное значение не возвращается в случае если игрок успел отойти. Вновь доработаем функцию
```gdscript
	if not $AttackSensor.is_colliding():
		velocity = velocity.lerp(direction*SPEED, 10 * delta)
		damage_delay = 1.3
```

Функция сохранения в скрипте уровня
```gdscript
func saving():
	var file = FileAccess.open("res://save_data.txt", FileAccess.WRITE)
	var save_data = {}
	for i in get_tree().get_nodes_in_group("save"):
		save_data[i.name] = i.save()
	file.store_var(save_data)
```
Функция save у игрока и бота
```gdscript
func save():
	var data = {
		"hp": hp,
		"position" : position
	}
	return data
```

Функция загрузки
```gdscript
func loading():
	var file = FileAccess.open("res://save_data.txt", FileAccess.READ)
	if !FileAccess.file_exists("res://save_data.txt"):
		return
	var load_data = file.get_var()
	for i in get_tree().get_nodes_in_group("save"):
		i.Load(load_data[i.name])
```

Функции Load у игрока и бота
```gdscript
func Load(data):
	hp = data['hp']
	position = data['position']
```

Добавить магазин (скрипт игрока). 1 из вариантов, переработаем структуру функции стрельбы, чтобы все работало корректно и красиво

```gdscript
	if max_ammo >= ammo and ammo > 0:
		if Input.is_action_just_pressed("shoot"):
			shoot()
```

```gdscript
func shoot():
	animations.play("Rig|AK_Shot")
	if fire_delay == false:
		if ray_cast.is_colliding():
			decal_spawn()
		fire_delay = true
		await animations.animation_finished
		ammo -= 1
		fire_delay = false
```

```gdscript
func decal_spawn():
	if ray_cast.get_collider().name == "enemy":
		ray_cast.get_collider().damage()
	var col_nor = ray_cast.get_collision_normal()
	var col_point = ray_cast.get_collision_point()
	var b = decal.instantiate()
	ray_cast.get_collider().add_child(b)
	b.global_transform.origin = col_point
	if col_nor == Vector3.UP:
		b.rotation_degrees.x = 90
	elif col_nor == Vector3.DOWN:
		b.rotation_degrees.x = -90
	elif col_nor != Vector3.UP:
		b.look_at(col_point - col_nor, Vector3(0,1,0))
```

UI игрока

![image](https://github.com/user-attachments/assets/466c041d-5411-4788-86ae-e72abc706c93)

По скрипту все просто

![image](https://github.com/user-attachments/assets/d7d393ee-4c5a-4bf2-acb5-7b7a26816cb7)

Перейдем к перезарядке

```gdscript
	if Input.is_action_just_pressed("reload"):
		last_ammo = ammo
		reload()
```

```gdscript
func reload():
	if is_reloading or ammo == max_ammo:
		return  # Если уже идет перезарядка или обойма полная, ничего не делаем

	is_reloading = true
	last_ammo = ammo  # Сохраняем текущее количество патронов перед перезарядкой
	var last_animation = "Rig|AK_Reload"  # Запоминаем название анимации
	animations.play(last_animation)  # Запускаем перезарядку
	# Ждем завершения анимации, но при этом следим, не поменялась ли она раньше времени
	while animations.is_playing():
		if animations.current_animation != last_animation:
			ammo = last_ammo - 1  # Если анимация сменилась принудительно, перезарядка отменяется
			is_reloading = false
			return
		await get_tree().process_frame  # Проверяем каждую кадр
	# Если мы вышли из цикла без прерывания, значит анимация завершилась нормально
	ammo = max_ammo
	is_reloading = false  # Разрешаем следующую перезарядку
```

## Доп по ботам

Сейчас все боты которых мы добавляем постоянно идут за нами, что не совсем логично, сделаем так, что боты идут к нам если мы у них в прямой видимости или находимся где-то рядом, а также добавим, что если мы будем в месте где боты нас не могут достать, они уходили на свою стартовую позицию

Для добавление преследования ботом игрока только в случае когда игрок в зоне агра и не застеной нужно добавить следующие узлы

![image](https://github.com/user-attachments/assets/693a9b74-a5be-4446-b727-a60cdfe3d7c9)

Переменные которые нам понадобятся

```gdscript
var bot_chasing = false
var in_agro_area = false
@export var detection_area: Area3D  # Зона агро
@export var agro_ray_cast: RayCast3D  # RayCast для проверки видимости
```

И переработанный код, в котором будет удобно что-либо нажодить
```gdscript
func _physics_process(delta: float) -> void:
	print(bot_chasing)
	print(in_agro_area)
	if not is_on_floor():
		velocity.y -= gravity * delta
	AgroCollide(delta)
	if bot_chasing == true:
		update_path_to_target(delta)
	elif bot_chasing == false:
		velocity.x = 0
		velocity.z = 0
		agro_ray_cast.target_position = Vector3(0,0,-4.5)
	move_and_slide()
```

Для начала добавим переработаем функцию _physics_process, выделив из нее функцию преследования

```gdscript
func update_path_to_target(delta):
	var direction = Vector3()
	nav.target_position = target.global_position
	direction = nav.get_next_path_position() - global_position
	direction = direction.normalized()
	if not $AttackSensor.is_colliding():
		velocity = velocity.lerp(direction*SPEED, 10 * delta)
		damage_delay = 1.3
	elif $AttackSensor.get_collider().name == "player":
		velocity = Vector3()
		damage_delay -= delta
		if damage_delay <= 0:
			target.damage()
			damage_delay = 1.3 * 2
	var look_at_pos = target.global_position
	look_at_pos.y = global_position.y
	look_at(look_at_pos, Vector3.UP)
```

Далее создадим сигналы у зоны на проверку входа игрока

```gdscript
func _on_area_chasing_body_entered(body: Node3D) -> void:
	if "player" in body.name:
		in_agro_area = true

func _on_area_chasing_body_exited(body: Node3D) -> void:
	if "player" in body.name:
		in_agro_area = false

```

И поворот рэй каста для проверки находится ли игрок за стеной

```gdscript
func AgroCollide(delta):
	if in_agro_area == true:
		agro_ray_cast.look_at(target.global_position,Vector3.UP)
		if agro_ray_cast.get_collider() == target:
			bot_chasing = true
	elif in_agro_area == false:
		bot_chasing = false
```


```gdscript
func move_towards_home(delta):
	var direction = home_pos - global_position
	if direction.length() > 0.1:  # Когда расстояние до home_pos больше порога, двигаемся
		direction = direction.normalized()
		velocity = direction * SPEED
		look_at(home_pos, Vector3.UP)  # Поворачиваем в сторону home_pos
	else:
		velocity = Vector3.ZERO  # Останавливаемся, когда достигли home_pos
		print("Bot reached home position!")
```

Еще одна вариация
```gdscript
func move_towards_home(delta):
	if global_position.distance_to(home_pos) > 0.5 and in_agro_area == false:  # Если бот ещё не достиг точки
		print("ne aboba")
		nav.target_position = home_pos  # Устанавливаем цель
		var next_position = nav.get_next_path_position()  # Получаем следующую точку пути
		
		if next_position:  # Проверяем, есть ли следующая точка пути
			var direction = (next_position - global_position).normalized()  # Направление к цели
			velocity = velocity.lerp(direction * SPEED, 10 * delta)  # Плавное движение
			var look_at_pos = next_position
			look_at_pos.y = global_position.y  # Зафиксировать Y, чтобы не смотрел в пол
			look_at(look_at_pos, Vector3.UP)
		else:
			velocity = Vector3.ZERO  # Останавливаемся, если пути нет
	else:
		velocity = Vector3.ZERO  # Останавливаемся, если дошли до home_pos
```

```gdscript
func _process(delta: float) -> void:
	if ray.is_colliding():
		move_weapon(Vector3(0, -0.15, -0.15), Vector3(-1.2, 0, 0), 0.12)
	else:
		move_weapon(Vector3(0, 0, 0), Vector3(0, 0, 0), 0.1)

func move_weapon(target_position: Vector3, target_rotation: Vector3, duration: float):
	var tween = create_tween()
	tween.tween_property(weapon, "position", target_position, duration)
	tween.tween_property(weapon, "rotation", target_rotation, duration)
```
Можно добавить звуки, остальное по желанию










