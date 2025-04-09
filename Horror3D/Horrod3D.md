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

#### Настройка ввода

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

### Создание зума

Начнем с переменных

```gdscript
@export var min_fov = 1 # минимальный угол обзора (максимальный зум)
@export var max_fov = 75 # максимальный угол обзора (обычный вид)
@export var zoom_sensibiity = 1 # шаг зума при прокрутке
@export var max_zoom_rate = 2 # как сильно будет "расти" показатель зума на интерфейсе при приближении (UI эффект)

@onready var zoom_label: Label = $"../../zoom_label" # ссылка на Label для выывода игроку кратности зума
```
Сам зум будем делать в методе _input. Первым делом проверяем нажата ли какая-либо кнопка мыши

```gdscript
func _input(event):
	if event is InputEventMouseButton:
```

Если да мы создаем локальную переменную которая запоминает изначальное значение фова камеры

```gdscript
	var new_fov = fov
```

Теперь делаем приближение, но не даём опуститься ниже min_fov

```gdscript
	if event.button_index == MOUSE_BUTTON_WHEEL_UP:
		new_fov -= zoom_sensibiity
		if new_fov < min_fov:
			new_fov = min_fov
```

По той же логике уменьшение

```gdscript
	if event.button_index == MOUSE_BUTTON_WHEEL_DOWN:
		new_fov += zoom_sensibiity
		if new_fov > max_fov:
			new_fov = max_fov

```

Теперь делаем обновление зума, т.к. сейчас он изменяется лишь в этих условиях

```gdscript
	if new_fov != fov:
		fov = new_fov
```

Далее считаем расстояние между максимальным и текущим fov

```gdscript
		var max_abs_difference = abs(min_fov - max_fov)
		var current_abs_difference = abs(new_fov - max_fov)
```

Подсчёт визуального значения зума

```gdscript
		var zoom_value = (max_abs_difference + current_abs_difference * (max_zoom_rate - 1))
		zoom_value /= max_abs_difference 
```

И его вывод в label

```gdscript
		update_zoom_label(zoom_value)

func update_zoom_label(new_zoom: float):
	zoom_label.text = "%.2f X" % new_zoom
```

Весь скрипт камеры

```gdscript
extends Camera3D

@export var min_fov = 1
@export var max_fov = 75
@export var zoom_sensibiity = 1

@export var max_zoom_rate = 15 

@onready var zoom_label: Label = $"../../zoom_label"


func _input(event):
	if event is InputEventMouseButton:
		var new_fov = fov
		if event.button_index == MOUSE_BUTTON_WHEEL_UP:
			new_fov -= zoom_sensibiity
			if new_fov < min_fov:
				new_fov = min_fov
		if event.button_index == MOUSE_BUTTON_WHEEL_DOWN:
			new_fov += zoom_sensibiity
			if new_fov > max_fov:
				new_fov = max_fov
		if new_fov != fov:
			fov = new_fov
			var max_abs_difference = abs(min_fov - max_fov)
			var current_abs_difference = abs(new_fov - max_fov)
			var zoom_value = (max_abs_difference + current_abs_difference * (max_zoom_rate - 1))
			zoom_value /= max_abs_difference 
				
			update_zoom_label(zoom_value)
			
func update_zoom_label(new_zoom: float):
	zoom_label.text = "%.2f X" % new_zoom
```

Также более подробный разбор вывода зума на экран

var max_abs_difference = abs(min_fov - max_fov)

Это просто разница между минимальным и максимальным fov

var current_abs_difference = abs(new_fov - max_fov)

Это сколько сейчас мы "отдалены" от max_fov

var zoom_value = (max_abs_difference + current_abs_difference * (max_zoom_rate - 1))
zoom_value /= max_abs_difference

Сначала мы берем максимум (max_abs_difference) и добавляем усиленное приближение (current_abs_difference * (max_zoom_rate - 1)).
Потом делим обратно на max_abs_difference, чтобы нормализовать результат в привычный диапазон (от 1.0 до max_zoom_rate).

```gdscript

```

