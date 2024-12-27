# Космошутер

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




































