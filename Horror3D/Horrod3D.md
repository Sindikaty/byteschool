# 3D-horror

## Урок 1

### Создание игрока (движение + базовая камера)

#### Создание сцены

Начнем создание игрока с создания его сцены

```gdscript
Player (CharacterBody3D)  
├── MeshInstance3D
├── CollisionShape3D  
├── PlayerController (Node3D)  
    └── head (Node3D)  
        └── Camcorder (Camera3D)  
```

#### Настройка входов

Также сразу привяжем клавишы передвижения игрока

| Название  | Клавиши |
| ------------- | ------------- |
| moveUp  | w |
| moveDown  | s |
| moveLeft  | a |
| moveRight | d |
| Jump | Space |

Выглядеть наш игрок будет примерно так

![image](https://github.com/user-attachments/assets/f02332e0-cfbd-47e2-896d-be33f2f894c1)

#### Работаем над скриптом игрока Player.gd

Прикрепляем скрипт к игроку и начнем добавлять переменные

Первыми будут переменные для движения

```gdscript
@export var SPEED = 5.0
@export var JUMP_VELOCITY = 4.5
var gravity = ProjectSettings.get_setting("physics/3d/default_gravity") 
```

Далее переменные для камеры

```gdscript
@export var mouse_sensibility = 1200
var min_camera_x = deg_to_rad(-90)
var max_camera_x = deg_to_rad(90)
```

А также ссылки на контроллер

```gdscript
@onready var controller = $PlayerController
var camera
```

В функции _ready() скрипта PlayerController мы подключаем камеру и скрываем курсор

```gdscript
func _ready():
	camera = controller.camera
	Input.mouse_mode = Input.MOUSE_MODE_CAPTURED
```

Физика и движение по стандарту делаем в _physics_process(delta) только лишь выделив его в отдельную функцию для будущего дополнения

```gdscript
func _physics_process(delta):
	walk_process(delta)

func walk_process(delta):
	# Гравитация
	if not is_on_floor():
		velocity.y -= gravity * delta

	# Прыжок
	if Input.is_action_just_pressed("Jump") and is_on_floor():
		velocity.y = JUMP_VELOCITY

	# Движение
	var input_dir = Input.get_vector("moveLeft", "moveRight", "moveUp", "moveDown")
	var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()

	if direction:
		velocity.x = direction.x * SPEED
		velocity.z = direction.z * SPEED
	else:
		velocity.x = move_toward(velocity.x, 0, SPEED)
		velocity.z = move_toward(velocity.z, 0, SPEED)

	move_and_slide()
```

Вращение камерой мы будем делать похожим образом с платформером

```gdscript
func _input(event):
	if event is InputEventMouseMotion and controller.can_move_camera:
		rotation.y -= event.relative.x / mouse_sensibility
		camera.rotation.x -= event.relative.y / mouse_sensibility
		camera.rotation.x = clamp(camera.rotation.x, min_camera_x, max_camera_x)
```

#### Камера и контроллер — PlayerController.gd

Для начала добавим переменные

```gdscript
@onready var camera = $head/Camcorder as Camera3D
var can_move_camera = true
```

Движение + базовая камера готовы

















```gdscript

```
