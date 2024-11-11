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

Можно добавить солнце, состоит оно из следующий узлов
  
  📦Sun (Area3D) `Зона уничтожения игрока`<br>
    ┣- 📂CollisionShape3D `Коллизия`<br>
    ┣- 📂StaticBody3D`Тело солнца`<br>
    ┣--- 📂CollisionShape3D `Коллизия`<br>
    ┣--- 📂Node3D `Моделька солнца`<br>

Добавим переменные

```gdscript
var timerStart = false
var timer = 0
```

Код проверки на влете в солнце

```gdscript
func _process(delta: float) -> void:
	...
	$Sun/StaticBody3D.rotation.y += 0.001
	if timerStart == true:
		timer += delta
		$"Player/OmniLight3D".visible = true
	if timer >= 10:
		print("tobi pizda")
		$Player.queue_free()
```

Код на вход/выход из зоны

```gdscript
func _on_sun_body_entered(body: Node3D) -> void:
	if body.name == "Player":
		timerStart = true
		print("aboba")

func _on_sun_body_exited(body: Node3D) -> void:
	if body.name == "Player":
		timerStart = false
		timer = 0
		$"Player/OmniLight3D".visible = false
		print("aboba1")
```

Анимация тревоги

![image](https://github.com/user-attachments/assets/29f70f53-1788-4a9e-803c-3a5921540d05)

![image](https://github.com/user-attachments/assets/c9901df7-003e-4413-8293-4a7386b32470)


## Урок 3

Создание стрельбы

Пример пули и ее узлы

![image](https://github.com/user-attachments/assets/0675bcd1-11f5-45c7-aab5-92f77c45459c)

Отличие CharacterBody3D от AnimatableBody3D в том что последнего можно переместить только из кода, и на него не действуют внешние силы. 

Далее создаем 3 переменные 

```gdscript
var dir
var timer = 0
var speed = 50
```

У пули должно быть задано правильное направление сразу с момента как она появится на главной сцене поэтому добавляем функцию ready()

```gdscript
func _ready() -> void:
	if $"../Player".forward_speed > 1:
		dir = (global_transform.basis.z.normalized() * - speed) * ($"../Player".forward_speed / 3)
	else:
		dir = global_transform.basis.z.normalized() * - speed
```

Далее создаём функцию _physics_process(delta). Добавляем в неё время жизни пули и move_and_collide()

```gdscript
func _physics_process(delta: float) -> void:
	timer += delta
	if timer > 5:
		queue_free()
	
	var object = move_and_collide(dir*delta)
```

Код пули готов, теперь идём прописывать и обдумывать логику выстрела. Идём на сцену с персонажем и добавляем ему столько узлов Marker3D, сколько будет пушек. 

![image](https://github.com/user-attachments/assets/7f7ba031-bdbc-4154-b6e3-7a0ade9c154a)

Прикрепляем скрипт в маркерам и прописываем логику появления лазера

```gdscript
const bulletScene = preload("res://laser.tscn")


func fire():
	var bullet = bulletScene.instantiate()
	bullet.global_position = global_position
	bullet.rotation = global_rotation
	Global.add(bullet)
```

Как можно заметить в последней строке мы используем глобальный скрипт, нам его для начала нужно создать.

![image](https://github.com/user-attachments/assets/97e089a9-3494-4765-8958-578530cdd990)

В глобальном скрипте прописываем переменную в которой будет храниться основная сцена и также создаём функцию которая будет спавнить пулю на этой сцене.

```gdscript
var lvl = null

func add(obj):
	lvl.add_child(obj)
```

Переходим на сцену уровня и в методе _ready() и обращаясь к глобальной переменной lvl определяем, что в ней хранится основная сцена при помощи self.

```gdscript
func _ready() -> void:
	Global.lvl = self
```

Остается лишь задать клавишы для выстрела и сам выстрел (скрипт игрока)

![image](https://github.com/user-attachments/assets/80c87fb7-522e-480e-80e1-fbd484d1abcd)

```gdscript
func _physics_process(delta):
	if Input.is_action_just_pressed("Fire"):
		$Marker3D.fire()
		$Marker3D2.fire()
```

## Урок 4

Создание врага.

Создание модельки повторяет создание модельки игрока, поэтому ученики ее делают сами

В скрипте создаем 3 переменные

```gdscript
var speed = 30
var time = 0
var hp = 100
```

Движение врага

```gdscript
func _physics_process(delta: float) -> void:
	var dir = $"../Player".position - position
	if dir.length() > 30:
		velocity = -transform.basis.z * speed
		move_and_collide(velocity * delta)
	look_at($"../Player".position, Vector3.UP)
```

Стрельба
```gdscript
	time += delta
	if time > 2:
		$Marker3D.fire()
		$Marker3D2.fire()
		time = 0
```


