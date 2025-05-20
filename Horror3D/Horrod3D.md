![image](https://github.com/user-attachments/assets/070db9a6-5e3e-4065-bc33-3a821237adee)# 3D-horror

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


### Создание окружения

Начнем с WorldEnvironment. Большинство подобный игр у нас происходят в полумраке или вовсе во тьме, поэтому сейчас мы настроим наш узел так, чтобы вокруг игрока было достаточно темно. Настройки WorldEnvironment


Настройки неба

![image](https://github.com/user-attachments/assets/e2f5d869-6bc1-46f1-a0bf-8ff52c8e6e79)

Настройки яркости

![image](https://github.com/user-attachments/assets/57bf37fe-1c81-479f-9ca2-ab70309f341f)

Туман

![image](https://github.com/user-attachments/assets/b5613d5c-cdd1-4438-95c1-f46d542c2a2c)

Доп нстройки тумана

![image](https://github.com/user-attachments/assets/7cd6a8e0-3734-4778-aa88-494506f7d53e)

Доп настройки в целом

![image](https://github.com/user-attachments/assets/03912a5f-44b2-484c-8367-37a471eff71a)

Также в таких играх обычнет идет снег или дождь

![image](https://github.com/user-attachments/assets/7c0c1113-895e-41b9-b0dd-d4343277b7c3)

Раздел спавна партиклов

![image](https://github.com/user-attachments/assets/982887d8-8b90-4d9c-ac99-a7f3ecaf2f61)

![image](https://github.com/user-attachments/assets/467b7a7b-c922-4619-be93-4a7706932c6c)

![image](https://github.com/user-attachments/assets/fd461b7a-2b49-4c8e-9771-a635c7e1a87b)

Угловая быстрота

![image](https://github.com/user-attachments/assets/ca14253f-d1a5-4071-9484-d984d01625f6)

Размер на экране

![image](https://github.com/user-attachments/assets/76524f9d-b18e-46ab-b2fe-ec209436b048)

Турбулентность партиклов

![image](https://github.com/user-attachments/assets/3f30f462-2289-4eda-a84c-c3371b413df5)

### Добавление лестницы

Для начала создадим базовый для всех будущих объектов скрипт

```gdscript
extends StaticBody3D
class_name InteractionBase

func interact(_parameters=null):
	pass
```

Пока что в нем будет только 1 функция

Следующий скрипт это скрипт самой лестницы, которую перед этим нам нужн создать (Это Static body c мэшэм и коллизией)

```gdscript
extends InteractionBase

class_name Ladder

@onready var mesh: MeshInstance3D = $MeshInstance3D
@onready var position_reference: Node3D = $PlayerPositionReference
var height

func _ready():
	var aabb = mesh.get_aabb()
	height = mesh.global_position.y + aabb.size.y
	
func interact(player=null):
	#if not is_player_in_front(player,self):
	player.get_parent().set_on_ladder(true,height)
```

У игрока нужно добавить RayCast который будет проверять навелся ли игрока на объект с который можно взаимодействовать. Теперь в скрипте котнроллера игрока мы проверяем нажатие кнопки и соприкосновение рэй каста и если он подходит под наши условия вызываем функцию у лестницы

```gdscript
@onready var ray_cast: RayCast3D = $head/RayCast3D

func _input(event):
	if event.is_action_pressed("Interact") and ray_cast.is_colliding():
		var object = ray_cast.get_collider()
		if object is InteractionBase:
			object.interact(self)
```

Последним что нам нужно доабвить это все движение в скрипте игрока. Начнем с переменных

```gdscript=
enum PLAYER_MODES {
	WALK,
	LADDER
}
var current_mode := PLAYER_MODES.WALK
var ladder_height = 0
@export var ladder_height_subtract = 1
```

enum — это способ создать набор именованных констант для удобной работы с числами или режимами.
Далее добавляем смену режима движения в скрипте игрока

```gdscript
func _physics_process(delta):
	match current_mode:
		PLAYER_MODES.WALK:
			walk_process(delta)
		PLAYER_MODES.LADDER:
			ladder_process(delta)
```

Сам режим переджвижения по лестницам (скрипт игрока)

```gdscript
func ladder_process(_delta):
	
	if Input.is_action_just_pressed("Jump"):
		velocity.y = JUMP_VELOCITY
		set_player_mode(PLAYER_MODES.WALK)
		return
	
	var input_dir = Input.get_vector("moveLeft", "moveRight", "moveDown", "moveUp")
	var direction = (transform.basis * Vector3(0, input_dir.y,0)).normalized()
	if direction:
		velocity.y = direction.y * SPEED	
	else:
		velocity.y = move_toward(velocity.x, 0, SPEED)
	if position.y >= ladder_height - ladder_height_subtract and velocity.y > 0:
		velocity.y = 0
		
	move_and_slide()
```

И функции по сменам режимов (скрипт игрока)

```gdscript
func set_player_mode(mode: PLAYER_MODES):
	current_mode = mode


func set_on_ladder(on_ladder, height):
	if on_ladder:
		set_player_mode(PLAYER_MODES.LADDER)
		velocity = Vector3(0,0,0)
	else:
		set_player_mode(PLAYER_MODES.WALK)
	ladder_height = height
```


Добавление фонарика

Узел для света можно выбрать по желанию, я взял SpotLight (На скрине пример настроек, ученик может настроить как он хочет)

![image](https://github.com/user-attachments/assets/e9f197d6-2963-4d25-89b1-22ecc2be8742)

Скрипт включения выключения

```gdscript
var lightOn = false
func _physics_process(delta):
	if Input.is_action_just_pressed("lightOn"):
		lightOn = !lightOn
	$SpotLight3D.visible = lightOn
```

скрипт лестницы

```gdscript
func grow_visible():
	mesh.material_override.next_pass.grow = true

func grow_invisible():
	mesh.material_override.next_pass.grow = false
```

скрипт контроллера камеры
```gdscript
	if ray_cast.is_colliding() and ray_cast.get_collider() is InteractionBase:
		obj = ray_cast.get_collider()
		grow_visible = !grow_visible
		obj.grow_visible()
	elif obj != null:
		obj.grow_invisible()
```



Бот

Узлы

![image](https://github.com/user-attachments/assets/64da68ea-1831-45e2-9f26-a6afb4f74a8e)

Скрипт

```gdscript
extends CharacterBody3D

@export var base_speed: float = 3.0

@export var nav_agent: NavigationAgent3D
@export var target: Node3D

var bot_chasing = false
@onready var ray_cast_3d: RayCast3D = $RayCast3D



var current_patrol_index = 0
var is_aggro = false

var mav_map : RID


func _ready() -> void:
	mav_map = self.get_world_3d().get_navigation_map()

func _physics_process(delta):
	var player_distance = global_position.distance_to(target.global_position)
		
	var speed = base_speed
		
	AgroCollide(delta)
	if bot_chasing == true:
		# Преследование
		nav_agent.target_position = target.global_position
	else:
		# Патрулирование
		if nav_agent.is_navigation_finished():
			var next_pos = NavigationServer3D.map_get_random_point(mav_map,1,true)
			nav_agent.target_position = next_pos
		
	# Движение по пути
	var direction = nav_agent.get_next_path_position() - global_position
	direction = direction.normalized()

	velocity = velocity.lerp(direction * speed, 10 * delta)
	look_at(nav_agent.get_next_path_position(), Vector3.UP)
	move_and_slide()


func _on_area_3d_body_entered(body: Node3D) -> void:
	if "player" in body.name:
		is_aggro = true

func _on_area_3d_body_exited(body: Node3D) -> void:
	if "player" in body.name:
		is_aggro = false

func AgroCollide(delta):
	if is_aggro == true:
		ray_cast_3d.look_at(target.global_position,Vector3.UP)
		if ray_cast_3d.get_collider() == target:
			bot_chasing = true
	elif is_aggro == false:
		bot_chasing = false
```










1. InteractionBase (базовый класс для всех интерактивных объектов)
Что делает:
Это базовый класс для всех объектов, с которыми может взаимодействовать игрок (двери, лестницы, предметы и т. д.).

Определяет метод interact(), который должен быть переопределён в дочерних классах.

Содержит функцию is_player_in_front(), которая проверяет, находится ли игрок перед объектом.

Как работает is_player_in_front():
Берет позицию игрока (player.position).

Получает вектор "вперёд" игрока (ось X его глобального basis).

Вычисляет направление от игрока к объекту (object.position - player.position).

Нормализует этот вектор.

Считает скалярное произведение между:

Нормализованным вектором "игрок → объект".

Вектором "вперёд" игрока.

Если результат > 0, значит, объект находится перед игроком.

Зачем нужно:
Чтобы игрок не мог взаимодействовать с объектами, стоя к ним спиной.

Упрощает создание новых интерактивных объектов (достаточно унаследоваться от InteractionBase).

2. Ladder (скрипт лестницы)
Что делает:
Наследуется от InteractionBase и реализует логику взаимодействия с лестницей.

В _ready() вычисляет высоту лестницы через AABB (границы меша).

При взаимодействии (interact()) проверяет, стоит ли игрок перед лестницей, и если да — переводит его в режим лазания.

Ключевые элементы:
mesh: MeshInstance3D — визуальная модель лестницы (нужна для расчёта высоты).

position_reference: Node3D (не используется в текущей версии, но мог бы задавать точку входа на лестницу).

height — высота лестницы, вычисляется из размера меша.

Как вызывается взаимодействие:
Игрок нажимает кнопку взаимодействия (Interact).

RayCast проверяет, во что он смотрит.

Если это InteractionBase (например, лестница), вызывается interact().

Лестница проверяет, стоит ли игрок перед ней (is_player_in_front).

Если да — вызывает player.get_parent().set_on_ladder(true, height).

Зачем нужно:
Даёт лестнице возможность "активироваться" только при правильном угле взаимодействия.

Передаёт управление игроку, сообщая ему высоту лестницы.

3. Скрипт игрока (добавленные части)
Что добавилось:
Режимы передвижения (PLAYER_MODES):

WALK — обычное перемещение.

LADDER — режим лазания по лестнице.

ladder_process(delta) — управление на лестнице:

Движение вверх/вниз по input_dir.y (клавиши W/S или Up/Down).

Прыжок (Jump) сбрасывает режим лестницы.

Автоматическая остановка при достижении верха (ladder_height - ladder_height_subtract).

set_on_ladder(on_ladder, height):

Включает/выключает режим лестницы.

Сбрасывает скорость (velocity = Vector3.ZERO), чтобы игрок не падал.

Запоминает высоту лестницы для ограничения движения.

Зачем нужно:
Разделяет логику передвижения на земле и на лестнице.

Обеспечивает плавное перемещение вверх/вниз.

Не даёт игроку "перелететь" верх лестницы.

4. Контроллер игрока (взаимодействие через RayCast)
Как работает:
При нажатии Interact (E или другая клавиша) проверяется ray_cast.is_colliding().

Если луч попал в объект, проверяется, является ли он InteractionBase.

Если да — вызывается interact(self) (игрок передаёт себя в качестве параметра).

Зачем нужно:
Универсальный способ взаимодействия с объектами.

Можно легко добавлять новые интерактивные объекты без изменения кода игрока.

Б
