# 2D-Шутер
Проект, направленный на отработку тем, которые ученик проходил на предыдущих уроках. В проекте затрагиваются концепции ООП.

## Урок 1

### Создание карты

Для удобства дальнейшего создания игрока, создадим карту с физическими слоями, но пока что без слоев навигации для ботов. Для этого создаем `TileMap`. В свойствах `TileMap` создаем `TileSet` после чего у нас появится сетка игрового поля. <br>
Добавляем наш набор тайлов, он должен выглядить примерно так:

![image](https://github.com/Sindikaty/byteschool/assets/158248099/ecf9a4ff-8dc5-4ae1-939a-4c10b3b9b515)

Для создания физического слоя нажимаем по `TileSet` и в его свойстве `Physics Layers` добавляем новый физический слой
![image](https://github.com/Sindikaty/byteschool/assets/158248099/2f678e59-d97c-421b-ad38-00050d068b6d)

Возвращаемя в набору тайлов, заходим в рисовать и выбираем там `Физический слов 0`
![image](https://github.com/Sindikaty/byteschool/assets/158248099/c109ea79-8b69-4f0f-94b7-b92e29e7d3b1)

Осталось лишь выделить те ячейки которые мы хотели бы сделать физическими
![image](https://github.com/Sindikaty/byteschool/assets/158248099/8e626731-e2ee-4e05-addb-c81260a2d45d)

Теперь даем волю фантазии и рисуем карту по которой будет перемещаться игрок

### Создание игрока

Для начала создадим CharacterBody2D и доабвим к нему следующие узлы:
* AnimatedSprite2D
* CollisionShape2D
* Camera2D

Добавим анимацию игроку
![image](https://github.com/Sindikaty/byteschool/assets/158248099/c6f65643-f853-440c-89d2-f967d14b4dd1)

### Переходим к работе со скриптом

Сначала зададим перемещение. Для этого создадим 2 переменные отвечающие за скорость и вектор движения

```gdscript
@export var speed : int = 100
var input_vector = Vector2.ZERO
```

После чего зададим само пермещение, а также проигрыванеи анимаций

```gdscript
func _physics_process(delta):
  input_vector.x = Input.get_action_strength("right") - Input.get_action_strength("left")
	input_vector.y = Input.get_action_strength("down") - Input.get_action_strength("up")
	
	velocity = input_vector * speed

	if(input_vector.x > 0):
		$AnimatedSprite2D.play("run")
		$AnimatedSprite2D.flip_h = false
	elif(input_vector.x < 0):
		$AnimatedSprite2D.flip_h = true
		$AnimatedSprite2D.play("run")
	move_and_slide()
```
Ниже пойдет скрипт, необходимый для осуществления поворота персонажа

```gdscript
func _physics_process(delta):
        ...
	var mouse_pos = get_global_mouse_position() # - получаем позицию мыши
	look_at(mouse_pos) # - персонаж смотрит на курсор (мышь)
	$AnimatedSprite2D.global_rotation = 0.0 # - блокирует возможность вращения текстуры персонажа (если в этом есть необходимость)
```

> [!CAUTION]
> Методы написания скрипта может отличаться в зависимости от текстур, которые вы используете. Некоторые текстуры может быть необходимо вращать, а некоторые - нет. Будьте внимательны и готовы к такому

Так как мы делаем шутер можно добавить удобный прицел. Для этого создадим еще одну переменную

```gdscript
var cursor = preload("res://textures/white_crosshair.png")
```

```gdscript
func _physics_process(delta):
  ...  
  Input.set_custom_mouse_cursor(cursor, Input.CURSOR_ARROW, Vector2(16,16))
```

>[!IMPORTANT]
> Обрати внимание на путь, где лежит текстура. У вас может немного отличаться, но лучше все разбивать на отдельные папки


#### Создание деша
Фрагмент кода ниже нужен для создания деша (ускорения)

![](https://github.com/mykweenn/byteschool/blob/main/shooter/img/tumblr_nemrsjk8hi1sulisxo1_1280.webp)

> У нас будет не так красиво выглядеть деш, но по механике будет так работать

Создадим переменные которые нам понадобятся чуть позже

```gdscript
var dash_trigger = false
var dash_time = 0
```

Теперь создадим функцию рывка
```gdscript
func dash(delta):
	transform.origin = lerp(transform.origin, transform.origin + 40 * input_vector , 0.1)
```

Этот код означает следующее:

* transform.origin - это позиция объекта в пространстве.
* lerp() - это функция, которая выполняет линейную интерполяцию между двумя значениями. В данном случае, она применяется к текущей позиции объекта и новой позиции, которая получается при добавлении вектора управления (input_vector), умноженного на 40 .
* 0.1 - это коэффициент сглаживания, который определяет, насколько быстро объект будет перемещаться к новой позиции.
Таким образом, этот код перемещает объект с текущей позиции к новой позиции, которая получается при добавлении вектора управления, умноженного на 40, с использованием линейной интерполяции для плавного перемещения.

Ниже представлен метод, в котором написаны условия для применения деша

```gdscript
func get_input_velocity(delta): # В аргумент
	if Input.is_action_just_pressed("dash"): # Если нажата кнопка деша, то...
		dash_trigger = true # срабатывает триггер деша
	if dash_trigger == true: # если триггер деша равен истине, то
		dash_time += delta # к дешу прибавляется delta (таким образом создается локальный таймер)
		dash(delta) # вызываем функцию деша, которую вы писали ранее
		if dash_time > 0.25: # Если время деша превышает 0.25 секунды, то
			dash_trigger = false # вырубаем деш
			dash_time = 0 # время деша откатывается к 0
```
Все что нам остается это добавить вызов функции в методе `_physics_process(delta)`

```gdscript
get_input_velocity(delta)
```
И последним что мы сделаем в этом уроке, добавим прицеливание. Для его создания нам понадобится `Marker2D`

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0ab3e2fc-1f94-4943-8e0f-aea472e5e4ac)

Создадим новую функцию `scope`
```gdscript
func scope():
	if Input.is_action_pressed("aim"): # При зажатии ПКМ срабатывает одно из условий ниже
		$Camera2D.position = $scope.position # Если нажат ПКМ, то камера перейдет к позиции прицела
	else:
		$Camera2D.position = Vector2(0,0) # Иначе камера сместится обратно в начальную позицию
```
И добавим ее вызов функции в методе `_physics_process(delta)`

## Урок 2

### Создание оружия
![image](https://github.com/Sindikaty/byteschool/assets/158248099/5bc84a31-b1c2-4c68-83e0-34c0203ae5f9)

Базово для создания пушки потребуются следующие узлы:

* Sprite2D - Главный узел, является узлом для спрайтов
* Marker2D - Необходим для редактирования двухмерного пространства. Задать точку спавна, вылет пули и т.д.
* AudioStreamPlayer2D - Нужен для позиционированного звукогового сопровождения

Перейдем к скрипту

Для начала сделаем поворот пушки и ее отзеркаливание при достижении определенных градусов

>[!IMPORTANT]
> Градусы при которых пушка будут зеркалиться зависит от размера пушки (спрайта)

```gdscript
func _physics_process(delta):
	var mouse_pos = get_global_mouse_position() # переменная куда сохраняется глобальная позиция мыши
	look_at(mouse_pos) # метод который поворачивает текущий узел в сторону указанного объекта
	if global_rotation_degrees > 90 or global_rotation_degrees < -90: # условие при котором если глобальные градусы поворота спрайта превышают 90 или меньше -90
		self.flip_v = true # если условие выше является истиной, то спрайт зеркалится
	else: # иначе
		self.flip_v = false # не зеркалится
```

Теперь начнем делать выстрел, однако чтобы наша пушка могла стрелять нам нужны патроны. Для этого создадим новую сцену нашего патрона

Для создания патронов потребуются следующие узлы:

* RigidBody2D
* MeshInstance2D
* CollisionShape2D
* PointLight2D

![image](https://github.com/Sindikaty/byteschool/assets/158248099/2d1e7409-b03b-4b57-b917-ec805dec6a88)

Также можно добавить элемент VisibleOnScreenNotifier2D который позволять нам удалять те пули которые находят за экраном нашей игры
![image](https://github.com/Sindikaty/byteschool/assets/158248099/e2017057-33cd-45af-b1b7-d529715ed1e4)

Для этого у него присоединим узел screen_exited()

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0945adb7-2c9a-4299-a913-9642f380af02)

И в скрипте пропишем
```gdscript
func _on_visible_on_screen_notifier_2d_screen_exited():
	queue_free()
```
А также добавим удаление патронов при касании с какими-либо физическими объектами
Для этого добавим у `RigidBody2D` переменную 
```gdscript
var motion = Vector2()
```
И пропишем следующий скрипт
```gdscript
func _physics_process(delta):
	var collide = move_and_collide(motion * delta)
	if collide:
		queue_free()
```
Теперь вернемся к нашей пушке и в ее коде создадим новую функцию которая будет отвечать за выстрел. 
Для нее нам понадобится 3 переменных 
```gdscript
var can_fire = true # проверка на возможность стрельбы
var bullet = preload("res://bullet.tscn") # подгрузка сцены с пулей
var bullet_speed = 1000 # скорость пули
```
И сама функция стрельбы
```gdscript
func shoot():
	if can_fire: # если игрок может стрелять
		var bullet_instance = bullet.instantiate() # инстанцируется пуля (создается экземпляр указанной сцены, клон проще говоря)
		$bullet_point.position = Vector2(23, -8) # позиция Marker2D (точка вылета пули) мы ставим в позицию которая находится примерно около рук персонажа
		bullet_instance.global_position = $bullet_point.global_position # глобальная позиция инстанцированной пули и точки вылета пули совпадают
		bullet_instance.apply_impulse(Vector2(bullet_speed, 0).rotated(get_parent().rotation), Vector2()) # задаем импульс для инстацнированной пули со скоростью пули и методом указываем направление родительского узла, а вторым вектором - ничего
		get_parent().get_parent().add_child(bullet_instance) # получаем родительский узел дважды и добавляем в сцену нового ребенка, который является инстанцированной пулей
		$sound_gun.play() # проигрывание звука пули
		can_fire = false # запрещаем стрелять игроку
		await get_tree().create_timer(0.5).timeout # Генератор таймера, который истечет за указанное время в скобках (проще говоря это у нас скорострельность пушки)
		can_fire = true # разрешаем пальбу
```
Все что нам осталось это добавить кнопку которая будет отвечать за стрельбу и прописать условия на ее нажатие
```gdscript
func _physics_process(delta):
...
	if Input.is_action_pressed("fire"):
		shoot()
```
### Создание тряски камеры

И последним что мы сделаем на этом уроке, это сделаем тряску камеры при стрельбе. Для нее нам понадобится глобальный скрипт.

Для его создания в меню скриптов нужно нажать `Файл` -> `Новый скрипт` и создать скрипт с названием global или каким-то подобным, после чего зайти в настройки проекта и добавить его в атозагрузку

![image](https://github.com/Sindikaty/byteschool/assets/158248099/69ce34c0-0918-4481-b92d-c2ed9d0ea30d)

```gdscript
var camera = null
```

Для создания тряски прикрепим скрипт к камеры, а также добавим ей `Timer`

![image](https://github.com/Sindikaty/byteschool/assets/158248099/9ad5bb03-c538-425d-875a-193f715db403)

Создадим 3 переменные которые нам понадобсятся для создания тряски экрана
```gdscript
var shake_amount : float = 0
@onready var timer : Timer = $Timer
@onready var tween : Tween = create_tween()
```

Устанавливаем процесс обновления, устанавливаем Global.camera в текущий узел и вызываем функцию randomize().

```gdscript
func _ready():
	set_process(true)
	Global.camera = self
	randomize()
```
Теперь генерируем случайное смещение для создания эффекта тряски.

```gdscript
func _process(_delta: float):
	offset = Vector2(randf_range(-3, 3) * shake_amount, randf_range(-3, 3) * shake_amount)
```
Теперь создадим функцию shake, которая запускает тряску камеры на заданное время с заданным количеством тряски.
```gdscript
func shake(time: float, amount: float):
	timer.wait_time = time
	shake_amount = amount
	set_process(true)
	timer.start()
```
Теперь присоединим узел `_on_timer_timeout()` таймеру. В нем мы останавливаем процесс обновления и запускаем анимацию плавного возвращения камеры в исходное положение.
```gdscript
func _on_timer_timeout() -> void:
	set_process(false)
	tween.interpolate_value(self, "offset", 1, 1, Tween.TRANS_LINEAR, Tween.EASE_IN)
```

Теперь все что нам осталось добавить вызов функции в функцию `shoot()` у оружия

```gdscript
Global.camera.shake(0.2, 1)
```

## Урок 3

### Создание слоумо

На этом уроке начнем с создания слоумо. Для него нам понадобится глобальный скрипт который мы создали на прошлом уроке.
Добавим переменную булевого (логического) типа в скрипте

```gdscript
var slowmo : bool = false
```

Создавать функцию замедления мы будем у игрока, поэтому возвращаемся в его скрипт и создаем там еще 2 переменные.

```gdscript
@export var force : float = 0.2 # Переменная ответающая за то на сколько будет замедлятся игра
var slow_dash : bool = false # Переменная которая будет проверять включен ли слоумо
```

Перейдем к созданию самой функции

```gdscript
func slowmotion():
	if Input.is_action_just_pressed("slowmo"): # Проверяем, была ли нажата кнопка "slowmo"
		Global.slowmo = not Global.slowmo # Меняем значение переменной Global.slowmo на противоположное
		if Global.slowmo == true: # Если замедленное время включено
			Engine.time_scale = force # Устанавливаем значение движка на force, чтобы внутриигровые часы медленне тикали
    			slow_dash = true # Включаем переменную slow_dash
		else: # Если замедленное время выключено:
			Engine.time_scale = 1 # Возвращаем стандартное значение внутриигровым часам
			slow_dash = false # Выключаем переменную slow_dash
```

Теперь практически все будет замедляться при использовании слоумо, однако далеко не все, например, звуки и функция рывка не будет замедлена, из-за чего даже в слоумо все звуки и рывок будут как при обычной игре. Чтобы это исрпавить начнем с изменения функции `dash`
У нее мы добавим проверку на включенное замедление
```gdscript
func dash(delta):
	if slow_dash: # если время замедлено, то это условие
		transform.origin = lerp(transform.origin, transform.origin + 2 * input_vector , 0.5)
	else: # деш не в замедле 
		transform.origin = lerp(transform.origin, transform.origin + 40 * input_vector , 0.1)
```

А также добавим функцию `slowmo` в оружие для замедления проигрывания звука стрельбы

```gdscript
func slowmo():
	if Global.slowmo == true:
		$sound_gun.pitch_scale = 0.8
	else:
		$sound_gun.pitch_scale = 1
```

### Создание врагов

Для начала создадим CharacterBody2D и доабвим к нему следующие узлы:
* AnimatedSprite2D
* CollisionShape2D
* NavigationAgent2D
* Timer (включить автостарт)

Также нам нужно сздать Навигационный слой у TileMap, иначе враг не сможет понять где он сможет двигаться

Создание Навигационного слоя 

![image](https://github.com/Sindikaty/byteschool/assets/158248099/5231c31d-beee-41a9-a186-c481139c99b4) 

Выбор тайлов под Навигационный слой

![image](https://github.com/Sindikaty/byteschool/assets/158248099/f1faa4b9-6c17-4504-a4d6-4f7d8dfa56a9)

Перейдем к скрипту

Для начала сделаем следование ботов за игроком. Для этого нам понадобятся следующие переменные

```gdscript
const speed = 100
@export var player: Node2D
@onready var nav_agent := $NavigationAgent2D as NavigationAgent2D
@onready var playa = get_node("/root/level/player")
```

```gdscript
func _physics_process(_delta: float) -> void:
	var dir = to_local(nav_agent.get_next_path_position()).normalized() # Извлекаем направление движения к следующей точке пути и нормализуем его
	velocity = dir * speed # Устанавливаем скорость движения персонажа в направлении полученного вектора dir.
	if dir.x > 0: # Проверяем направление движения по оси X и изменяем ориентацию спрайта соответственно.
		$AnimatedSprite2D.flip_h = false
	else:
		$AnimatedSprite2D.flip_h = true
	if dir != Vector2(0,0): # Анимация проигрывается если враг движется
		$AnimatedSprite2D.play("walk")
```
Для составления пути создадим следующую функцию 

```gdscript
func makepath() -> void:
	nav_agent.target_position = playa.global_position
```
Теперь нужно сделать, чтобы она где-то вызывалась. Для этого мы создали Timer который каждую секунду будет вызывать данную функцию и строить путь
```gdscript
func _on_timer_timeout():
	makepath()
```

После того как мы сделали движение врагов, можно перейти к следующей не менее важной функции. Это функция получения урона врагами. Для этого создадим функцию dmg
```gdscript
func dmg():
	hp -= 10
	if hp <= 0:
		queue_free()
```
А также нужно добавить в скрипт пули вызов данной функции. Для этого добавим в условие еще одно, которое будет проверять имеет ли столкнувшийся объект метод "dmg" и если да, то вызывать его
```gdscript
func _physics_process(delta):
	global_position += movement_vector.rotated(rotation) * speed * delta
	var collide = move_and_collide(motion * delta)
	if collide:
		if collide.get_collider().has_method("dmg"): # Проверяем, имеет ли столкнувшийся объект метод "dmg" (нанесение урона).
			collide.get_collider().dmg() # Вызываем метод "dmg" у столкнувшегося объекта для нанесения урона.
		queue_free()
```
Все что осталось сделать так это добавить чтобы враги нас тоже могли убивать. Для этого создадим функцию take_damage_e() у `врагов` который будет при помощи цикла проверять наличие столкновений, и в случае если они происходят будет вызываться функция получения урона у игрока
```gdscript
func take_damage_e():
	for i in get_slide_collision_count():
		var obj = get_slide_collision(i).get_collider()
		if obj is CharacterBody2D and obj.name == "player":
			obj.take_damage_p()
```
Данную функцию и переменную нужно создать у игрока. Она будет вызыватся при столкновении со врагами
```gdscript
@export var hp = 100.0

func take_damage_p():
	hp -= 1
	if hp == 0:
		print('dead')
```
Теперь осталось сделать чтобы функция нанесения урона врагом где-то вызывалась. Мы добавим ее в `_physics_process` врага
```gdscript
func _physics_process(_delta: float) -> void:
	...
	take_damage_e()
```

Последнее что мы на сегоджня добавим это интересная анимация источников света вокруг игрока

Для нее создадим обычную 2d сцену и добавим следующие узлы:
* PointLight2D (х3)
* AnimationPlayer

Источники света расположим примерно так. Цвет источников можно задать любой

![image](https://github.com/Sindikaty/byteschool/assets/158248099/a8a4a5ae-537f-428a-985a-14d2c08a13d7)

И теперь создадим небольшую анимацию. В ней мы будем изменять свойство `rotation` у основного узла `Node2D`. Значение rotation у анимации 0, 180 и 360, а также стоит не забыть включить автостарт

![image](https://github.com/Sindikaty/byteschool/assets/158248099/620232dd-6b6f-400e-bfa7-5f52ac6afd85)

Остается добавить анимацию на уровень или прикрепить к игроку и получится примерно следующая анимация

![Godot Engine Nvidia Profile 2024 04 05 - 13 52 07 02_Trim (1)](https://github.com/Sindikaty/byteschool/assets/158248099/92cf7ab4-e288-41dc-a745-585cd8c91929)

Можно сделать переливающийся задний фон

Для него нам нужен ColorRect и AnimationPlayer в котором мы задаем смену цветов

![image](https://github.com/Sindikaty/byteschool/assets/158248099/9f020650-6b3a-442b-8ff6-06f958277f63)

## Урок 4

Для начала добавим количество патронов нашему игроку. Для этого зайдем в скрипт оружия и добавим переменные отвечающие за количество патронов

```gdscript
var ammoInMagazine = 30
var maxAmmoInMagazine = 30
```
А также немного изменим функцию стрельбы. Нам нужно добавить в условие проверку на количество патронов, а также добавить их убавление при выстреле
```gdscript
	if can_fire and ammoInMagazine > 0:
		...
		ammoInMagazine -= 1
		...
```

Теперь после 30 выстрелов у нас пропадет возможность стрельбы, поэтому следующим шагом мы добавим зоны поднятия оружия. Для нее нам понадобится:

* Area2D
* CollisionShape2D
* Sprite2D
* Timer
* Label

По итогу у нас получится что-то подобное

![image](https://github.com/Sindikaty/byteschool/assets/158248099/a1c4231d-5fcf-4f37-b114-4554fda4becf)

Перейдем к скрипту

Для начала добавим переменные. Первые 3 отвечают за наши дочерние узлы, а оставшиеся 2 являются логическими и буду нужны для проверок на поднятие патронов

```gdscript
@onready var timer = $Timer
@onready var collision_shape_2d = $CollisionShape2D
@onready var label = $Label
@onready var ammoPickedUp = false
var canPick : bool = true
```

Присоединим узел `_on_body_entered` к нашей зоне и начнем делать подбор оружия

```gdscript
func _on_body_entered(body):
	if body.name == "player" and canPick:
		collision_shape_2d.disabled = true # убираем коллизию 
		body.call("pickupAmmo") # вызываем метод pickupAmmo
		timer.start() # включаем таймер
		ammoPickedUp = true # патроны подняты = тру
		canPick = false # можем поднять еще = фолс
```

Как мы видим мы обращаемся к функции `pickupAmmo` которой у нас еще нет, поэтому переходим в скрипт игрока и создаем там эту функцию

```gdscript
@onready var stateGun = get_node("gun") # узел нашего оружия

func pickupAmmo():
	stateGun.ammoInMagazine = stateGun.maxAmmoInMagazine 
```

Теперь мы можем поднять оружие 1 раз, для того чтобы мы могли делаеть это несколько раз с какой-то периодичностью нам нужно присоединить узел к таймеру `_on_timer_timeout`
```gdscript
func _on_timer_timeout():
	collision_shape_2d.disabled = false # включем коллизию
	timer.stop() # выключаем таймер
	label.text = ""
	ammoPickedUp = false # патроны подняты = фолс
	canPick = true # можем поднять еще = тру
```

По сути поднятие оружие готово, но для удобства можно добавить `label` в котором будет выводится время когда можно будет поднять оружие в следующий раз
```gdscript
func _process(delta):
	if ammoPickedUp:
		label.text = str(int(timer.time_left))
```

Респавн ботов

Для создания спавнера ботов на нужны следующие элементы:
* Marker2D
* Timer (автостарт)
  
Создавать мы их будет на нашем уровне

Перейдем к скрипту

```gdscript
@export var spawn_scene : PackedScene  # Экспортируем переменную spawn_scene типа PackedScene
```

Тип PackedScene в Godot Engine представляет собой упакованную сцену, которая содержит информацию о ресурсах и узлах сцены. Он используется для загрузки и создания экземпляров объектов сцены во время выполнения программы. Когда вы загружаете сцену в редакторе Godot, она сохраняется в формате .tscn, который затем может быть упакован в один файл .pck с помощью инструмента упаковки ресурсов.
Создаем функцию спавна

```gdscript
func spawn(_spawn_scene := spawn_scene) -> void:  # Определяем функцию spawn с параметром _spawn_scene по умолчанию spawn_scene
  var spawn := _spawn_scene.instantiate() as Node2D  # Создаем экземпляр объекта из PackedScene и приводим его к типу Node2D
  add_child(spawn)  # Добавляем созданный объект в качестве дочернего

  spawn.global_position = global_position  # Устанавливаем глобальную позицию созданного объекта равной глобальной позиции текущего объекта
```

Вызываем функцию создания объекта в сигнале `timeout()` у таймера

```gdscript	
func _on_timer_timeout():  # Обработчик события таймера
  spawn()  # Вызываем функцию spawn для создания объекта
```
А также нужно не забыть выбрать сцену спавна. В нашем случае это сцена с врагом

![image](https://github.com/Sindikaty/byteschool/assets/158248099/b74c47ce-fb49-4d34-aaa5-bf68bcc1c358)

Все основные механики готовы, осталось лишь сделать UI

Начнем с добавления ProgressBar нашим слаймам

![image](https://github.com/Sindikaty/byteschool/assets/158248099/7dc28a6d-89a3-4fa4-abd3-71ec45b62550)


![image](https://github.com/Sindikaty/byteschool/assets/158248099/37299c3c-09f9-4640-8183-f7a1da71556e)

И прописываем следующее в скрипте

```gdscript
func _physics_process(_delta: float) -> void:
	...
	$hp_bar.value = self.hp
	...
```

Следующим элементом можно добавить интерфейс для игрока (его хп, патроны и можно добавить фпс). Для этого создаем новой сценой CanvasLayer

Начнем с отображения патронов. Добавляем Sprite2D и Label после чего располагаем их в пределах видимости CanvasLayer, например, так

![image](https://github.com/Sindikaty/byteschool/assets/158248099/960d1de6-607f-4544-8f31-47e108a62bbd)

И прописываем следующее в скрипте Нашего Канваса

```gdscript
func _process(delta):
	$AssaultRifle/Label.text = str($"..".stateGun.ammoInMagazine)
```

Следующим добавим отображение хп игрока. Для этого добавим ProgressBar в CanvasLayer и прописываем следующее в скрипте игрока

```gdscript
func _physics_process(delta: float):
	...
	$ui_fps2/UI/ProgressBar.value = hp
 	...
```

Также можно добавить отображение фпс. Для него мы добавим Label и к нему прикрепим скрипт
```gdscript
@export var enabled := true

func _process(delta: float) -> void:
	if enabled:
		var frames : float = Engine.get_frames_per_second()
		text = "FPS: "
		text += str(frames)
```

Можно также добавить музыку на уровне. Для этого добавляем `AudioStreamPlayer2D` на наш уровень, а также не забываем о том, что у нас есть слоумо и в скрипте уровня нужно будет замедлять его если мы в слоумо

```gdscript
func _process(delta):	
	if Global.slowmo == true:
		$"Music_background".pitch_scale = 0.8
	else:
		$"Music_background".pitch_scale = 1
```

Осталосб добавить чтобы при смерти все процессы останавливались и была возможность рестарта
