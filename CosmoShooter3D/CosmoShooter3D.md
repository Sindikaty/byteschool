# CosmoShooter3D

## Урок 1

### Создание космоса

Переименовываем корневой узел в World и добавляем к нему ещё один Node3D. Добавляем к Node3D узел WorldEnvironment и начинаем его настраивать.

Создаем `Новый Environment`

![image](https://github.com/user-attachments/assets/70497967-373a-4c46-96b0-34c052d21174)

В разделе Backgroudn выбираем `Sky`

![image](https://github.com/user-attachments/assets/bb5bbe02-dbf7-4650-b1b7-d3a80840ceac)

В разделе Sky `Создать Sky`

![image](https://github.com/user-attachments/assets/98181a7c-2e83-4a48-acd5-3dda31a0582a)

А уже в Sky `Создать PanoramaSkyMaterial`

![image](https://github.com/user-attachments/assets/3ec7c941-483f-4cca-af8d-644ece8b983a)

Остальные настройки ученики могут поизменять сами. Основные свойства для настройки - `Custom FOV`, `Ambient Light`, `Tonemap`, `SDFGI`, `Glow`, `Adjustments`

Пример как можно настроить:

![image](https://github.com/user-attachments/assets/18db1442-fcc1-4bb9-a852-a5867a199ee9)__________________________
![image](https://github.com/user-attachments/assets/e1fe6a6f-dddf-4c5a-927d-5054beab9fb2)__________________________
![image](https://github.com/user-attachments/assets/f21e9c54-47a8-49c6-8679-dd14c80385ea)

### Создание игрока

Создаем переменные для того чтобы задавать скорость космолету

```gdscript
var forward_speed = 0.0
@export var acceleration = 0.6
@export var max_speed = 50
@export var max_speed_n = -20
```

Далее задаём клавиши для ускорения, обратного ускорения и сброса скорости

![image](https://github.com/user-attachments/assets/db609b4c-3399-4275-a291-a01247a78c47)

Создаём функцию get_input(delta). Параметр delta, будет необходим чтобы принимать в качестве аргумента delta из _physics_process(delta). 

```gdscript
func get_input(delta):
	if Input.is_action_pressed("throttle_up"):
		forward_speed = lerp(forward_speed, max_speed, acceleration * delta)
		print(forward_speed)
	if Input.is_action_pressed("throttel_negative"):
		forward_speed = lerp(forward_speed, max_speed_n, acceleration * delta)
#		print(forward_speed)
	if Input.is_action_pressed("throttel_down"):
		forward_speed = lerp(forward_speed, 0.0, acceleration * delta * 4)
#		print(forward_speed)
```

Создаём функцию _physics_process(delta) и вызываем в ней get_input(delta)

```gdscript
func _physics_process(delta):
	get_input(delta)
```

А также move_and_collide для движения космолета

```gdscript
func _physics_process(delta):
	...
	velocity = -transform.basis.z * forward_speed
	var coll = move_and_collide(velocity * delta)
```

Добавляем камеру

Запрограммируем кнопки на рысканье, тангаж и вращение. Для это создадим ещё несколько экспортных и обычных переменных:

```gdscript
@export var pitch_speed = 1.5
@export var roll_speed = 1.9
@export var yaw_speed = 1.25
@export var input_response = 8.0
var pitch_input = 0.0
var roll_input = 0.0
var yaw_input = 0.0
```

> [!Tip]
> * pitch - тангаж (наклон корабля вверх/вниз)
> * roll - вращение
> * yaw - рыскание (вправо/влево).<br>

```gdscript
func get_input(delta):
	...
	pitch_input = lerp(pitch_input, Input.get_axis("pitch_down", "pitch_up"),
		input_response * delta)
	roll_input = lerp(roll_input, Input.get_axis("roll_right", "roll_left"), 
		input_response * delta)
	yaw_input = lerp(yaw_input, Input.get_axis("yaw_right", "yaw_left"),
		input_response * delta)
```

Мы добавили изменение значений, теперь нужно изменить при помощи них положение объекта в пространстве. Для этого мы будем использовать basis 
basis - Матрица 3×3, используемая для представления трехмерного вращения и масштабирования. Обычно используется в качестве ортогонального базиса для Transform3D. Простыми словами, это параметр при помощи которого можно изменять вращение объекта в 3-х мерном пространстве.

```gdscript
func _physics_process(delta):
	...
	transform.basis = transform.basis.rotated(transform.basis.x, pitch_input * pitch_speed * delta)
	transform.basis = transform.basis.rotated(transform.basis.z, roll_input * roll_speed * delta)
	transform.basis = transform.basis.rotated(transform.basis.y, yaw_input * yaw_speed * delta)
	transform.basis = transform.basis.orthonormalized()
	...
```

## Урок 2

### Камера

Создаем камеру у уровня и прикрепляем к нему скрипт

![image](https://github.com/user-attachments/assets/f439a2b1-7437-4b01-9d32-349045864fbd)

В нем создаем 3 экспортные переменные и 1 пустую переменную с целью для камеры

```gdscript
@export var lerp_speed = 3.0
@export var target_path : NodePath
@export var offset = Vector3.ZERO
var target = null
```

При запуске камера должна сразу понимать цель и фокусироваться на ней. Для этого создадим метод _ready()

```gdscript
func _ready() -> void:
	if target_path:
		target = get_node(target_path)
```

Далее добавляем метод _physics_process(delta) и добавляем в неё условие. В случае если цель исчезнет вернется пустой таргет

```gdscript
func _physics_process(delta: float) -> void:
	if !target:
		return
```

Теперь создадим локальную переменную в которую сохраним копию преобразованной глобальной позиции таргета, относительно того насколько от него должна сместиться камера. Сделать это можно при помощи метода translated_local(). Для плавного перемещения камеры также применим интерполяцию для изменения её глобальной позиции. А также чтобы камера смотрела на корабль, а не в абстрактную точку, используем метод look_at().

```gdscript
func _physics_process(delta: float) -> void:
	...
	var target_xform = target.global_transform.translated_local(offset)
	transform = global_transform.interpolate_with(target_xform,lerp_speed*delta)
	look_at(target.global_transform.origin, target.transform.basis.y)
```

Не забываем выбрать target_path у камеры

![image](https://github.com/user-attachments/assets/56451ea1-959c-4763-b29f-afb3c4d2f7e5)

Можно сделать смену камеры у игрока, от первого лица и от 3го лица, для этого создадим скрипт у уровня. Создадим переменную которая будет хранить "номер" камеры

```gdscript
var curCamera = 1
```

Добавим условия на проверу нажатия кнопки, увеличение переменной будет означать изменение камеры

```gdscript
func _process(delta: float) -> void:
	if Input.is_action_just_pressed("nextCamera"):
		curCamera += 1
```

Логика смены камера следующая

```gdscript
	match curCamera:
		0:
			$Player/Camera3D2.current = true
		1:
			$Camera3D.current = true
```

А также нужно не забыть добавить сброс переменной, иначе мы не сможем больше менять камеру
```gdscript
func _process(delta: float) -> void:
	...
		if curCamera == 2:
			curCamera = 0
```
