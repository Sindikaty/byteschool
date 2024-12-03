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
