# CosmoShooter

## Урок 1

### Создание игрока (передвижения)

Начнем разработку нашей игры с создания корабля, на котором мы будем разрушать астеройды

Для его создания нам нужен узел CharacterBody2D и к нему добавить следующие узлы:
* AnimatedSprite2D
* CollisionShape2D (Можно использовать CollisionPolygon2D если у формы очень много углов)
* Node2D 2 узла (Это место вылета снарядов)

### Переходим к работе со скриптом

Сначала зададим перемещение. Для этого создадим 3 переменные

```gdscript
@export var acceleration := 10.0
@export var max_speed := 350.0
@export var rotation_speed := 250.0
```

Задидаим перемещение вперед и назад, а также ограничим скорость перемещения

```gdscript
func _physics_process(delta):
	var input_vector := Vector2(0,Input.get_axis("up","down"))
	velocity += input_vector.rotated(rotation) * acceleration #перемещение в пространстве
	velocity = velocity.limit_length(max_speed) #макс скорость
	...
```

Следующим шагом добавим поворот нашего корабля

```gdscript
func _physics_process(delta):
	...
	if Input.is_action_pressed("right"): #поворот вправо
		rotate(deg_to_rad(rotation_speed*delta))
	if Input.is_action_pressed("left"): #поворот влево
		rotate(deg_to_rad(-rotation_speed*delta))
	...
```

Теперь наш корабль может двигаться, однако он не перестает это делать в направлении вектора даже если мы отпустили все кнопки. Чтобы это исправить мы добавим еще одно условие 

```gdscript
func _physics_process(delta):
	...
if input_vector.y == 0: #останавка игрока при отпущенной клавише
		velocity = velocity.move_toward(Vector2.ZERO,3)
	...
```

Так как мы будет ограничены в рамках изначальных размеров поля и у нас не должно быть возможности вылета за экран добавим условия на проверку этого
```gdscript
func _physics_process(delta):
	...
	var screen_size = get_viewport_rect().size #размер экрана
	if global_position.y <0: #п
		global_position.y = screen_size.y #п
	elif global_position.y > screen_size.y: #п
		global_position.y = 0 #п
	if global_position.x <0: #п
		global_position.x = screen_size.x #п
	elif global_position.x > screen_size.x: #п
		global_position.x = 0 #п
	...
```

### Создание игрока (выстрел)

Начнем создание выстрела с создания самого лазера. Для его создания нам нужен узел Area2D и к нему добавить следующие узлы:

* Sprite2D
* CollisionShape2D
* VisibleOnScreenNotifier2D (Будет удалять снаряды за экраном)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/20430f48-72ca-4f13-849b-d5af0ae41c8d)

### Переходим к работе со скриптом

Сначала зададим перемещение снаряда. Для этого создадим 2 переменные

```gdscript
var movement_vector := Vector2(0,-1)
var speed := 500.0
```

После чего будем задавать глобальную позициию лазера

```gdscript
func _physics_process(delta):
	global_position += movement_vector.rotated(rotation) * speed * delta #поворот лазера в ту сторону куда повернут игрок
	pass
```

Также сразу можно присоединить сигнал screen_exited() для удаления лазера

![image](https://github.com/Sindikaty/byteschool/assets/158248099/fb84b526-6111-4f15-a52b-bea6dde301d8)

И прописать его удаление

![image](https://github.com/Sindikaty/byteschool/assets/158248099/de60acc7-4da4-467e-8af2-1a4e9e325b33)

Теперь можно вернуться к игроку и создать у него функцию shoot_laser(). Для нее нам понадобится 2 переменные и сигнал

```gdscript
signal laser_shot(laser) # Сигнал и какой параметр будет передаваться
@onready var muzzle = $Muzzle
var laser_scene = preload("res://scenes/laser.tscn")
```

В самой же функции определяем напрявление выстрела и то место откуда он будет произведен, и передаем это значение в сигнал

```gdscript
func shoot_laser():
	var l = laser_scene.instantiate()
	l.global_position = muzzle.global_position # откуда стреляем
	l.rotation = rotation # напрявление куда стреляем
	emit_signal("laser_shot", l)
```

Далее нам понадобится 2 переменные

```gdscript
var shoot_cd = false
var rate_of_fire = 0.35
```

В _process(delta) прописываем выстрел на нажатие кнопки с созданием таймера для создания времени между выстрелами.

```gdscript
func _process(delta):
	if Input.is_action_pressed("shoot"): #стрельба на зажатие клавишы
		if !shoot_cd:
			shoot_cd = true
			shoot_laser()
			await get_tree().create_timer(rate_of_fire).timeout #время перезарядки
			shoot_cd = false
```

Последним что нам осталось для произведение выстрела это вернуться на наш уровень. В нем мы создадим Node2D отвечающий за место выстрела и звук выстрела

![image](https://github.com/Sindikaty/byteschool/assets/158248099/165191bd-95a5-407c-b9eb-47bff21fe170)

Далее создадим скрипт у нашего уровня. В нем создадим 2 переменные

```gdscript
@onready var lasers = $Lasers
@onready var player = $player
```

Создадим новую функцию в которой мы будем включать звук выстрела и добавлять клон нашего лазера

```gdscript
func _on_player_laser_shot(laser):
	$Audio/shoot.play()
	lasers.add_child(laser)
```

И наконец нам осталось подключить сигнал в котором мы хранили координаты и направление куда полетит выстрел к новой функции создания клона нашего лазера

```gdscript
func _ready():
	player.connect("laser_shot", _on_player_laser_shot)
```

Стрельба готова

### Создание астероидов

Для астероидов нам понадобится узел Area2D и к нему добавить следующие узлы:

* Sprite2D
* CollisionShape2D

![image](https://github.com/Sindikaty/byteschool/assets/158248099/b3fa5450-0f01-40b2-9fea-81e6473770a7)

Однако астероидов у нас 3 вида, а создаем мы только 1, так что же нам делать? Со спрайтами все просто, они у нас уже есть, а с коллизиями мы поступим следующим образом. Начнем создание астеройдов с самого маленького и после создания сохраняем коллизию в отдельный файл, и переходим к следующему, а когда доходим до большого также сохраняем и оставляем его

![image](https://github.com/Sindikaty/byteschool/assets/158248099/304e41d4-5ae7-4566-b7e9-47828a2d53d3)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/8216241d-9707-4c85-b5d7-896efacf40c0)

После создания астероидов начнем создавать переменные которые нам будут нужны в дальнейшем, например, переменную определяющую размер астероида, размер очков за его уничтожение и т.д.

```gdscript
@onready var sprite = $Sprite2D
@onready var cshape = $CollisionShape2D

enum AsteroidSize{LARGE,MEDIUM,SMALL} # перечисления (enum)
@export var size := AsteroidSize.LARGE

var movement_vector := Vector2(0,-1)
var speed := 50

var points: int:
	get:
		match size:
			AsteroidSize.LARGE:
				return 100
			AsteroidSize.MEDIUM:
				return 50
			AsteroidSize.SMALL:
				return 25
			_:
				return 0
```

Добавим перемещение нашим астероидам. Начнем с функции _ready(), чтобы задать изначальные параметры движения (чему равна скорости всех астероидов, задать их текстуры и коллизии)
```gdscript
func _ready():
    rotation = randf_range(0, 2*PI) # Устанавливаем случайный угол поворота для астероида
    match size: # Используем конструкцию match для установки параметров в зависимости от размера астероида
        AsteroidSize.LARGE:
            speed = randf_range(50,100) # Устанавливаем случайную скорость для большого астероида
            sprite.texture = preload("res://textures/B_meteor.png") # Устанавливаем текстуру для большого астероида
            cshape.set_deferred("shape", preload("res://resourse/asteroid_b.tres")) # Устанавливаем форму столкновений для большого астероида
        AsteroidSize.MEDIUM:
            speed = randf_range(100,150) # Устанавливаем случайную скорость для среднего астероида
            sprite.texture = preload("res://textures/M_meteor.png") # Устанавливаем текстуру для среднего астероида
            cshape.set_deferred("shape", preload("res://resourse/asteroid_m.tres")) # Устанавливаем форму столкновений для среднего астероида
        AsteroidSize.SMALL:
            speed = randf_range(100,200)  # Устанавливаем случайную скорость для маленького астероида
            sprite.texture = preload("res://textures/S_meteor.png") # Устанавливаем текстуру для маленького астероида
            cshape.set_deferred("shape", preload("res://resourse/asteroid_s.tres")) # Устанавливаем форму столкновений для маленького астероида
```

Зададим передвижение астеройдам, а также добавим ограничение на вылет за экран и им 

```gdscript
func _physics_process(delta):
	global_position += movement_vector.rotated(rotation) * speed * delta
	
	var radius = cshape.shape.radius
	
	var screen_size = get_viewport_rect().size #перемещение в начало/конец астероида
	if (global_position.y + radius) < 0: #п
		global_position.y = (screen_size.y + radius) #п
	elif (global_position.y - radius) > screen_size.y: #п
		global_position.y = -radius #п
	if (global_position.x + radius) < 0: #п
		global_position.x = (screen_size.x + radius) #п
	elif (global_position.x - radius) > screen_size.x: #п
		global_position.x =  -radius #п
```

Также в дальнейшем для удобства обращение к переменным у астероида добавим имя класса у нашей Area2D

```gdscript
class_name Asteroid extends Area2D
```

Теперь наши астеройды двигаются, но мы никак не можем с ними взаимодейтвовать.

## Урок 2

Начнем создавать уничтожение астероидов. Для этого нам понадобится сигнал который будет передавать уже 3 параметра

```gdscript
signal exploded(pos, size, points)
```

И создаем функцию в которой этот сигнал получает информацию об астероиде

```gdscript
func explode():
	emit_signal("exploded", global_position, size, points)
	queue_free()
```

И теперь нам нужно присоединить сигнал area_entered() у `лазера`

```gdscript
func _on_area_entered(area):
	if area is Asteroid:
		var asteroid = area
		asteroid.explode()
		queue_free()
```

Теперь наши астероиды уничтожаются, однако уничтожается только большие и ничего больше не происходит. Сделаем чтобы наш большой распадался на 2 средний, а средний на 2 маленьких астероида. Для этого вернемся в скрипт основного уровня и в ней создадим переменную подгружающую нашу сцену с астероидом

```gdscript
var asteroid_scene = preload("res://scenes/asteroid.tscn")
@onready var asteroids = $Asteroids # это узел в котором лежат наши астероиды
```

Теперь нам нужно создать еще 2 функции. Первая функция будет отвечать за спавн астероидов
```gdscript
func spawn_asteroid(pos, size):
    var a = asteroid_scene.instantiate()  # Создаем экземпляр астероида из сцен
    a.global_position = pos # Устанавливаем глобальную позицию для астероида
    a.size = size # Устанавливаем размер для астероида
    a.connect("exploded", _on_asteroid_exploded) # Подключаем сигнал "exploded" к методу _on_asteroid_exploded
    asteroids.call_deferred("add_child", a)  # Добавляем созданный астероид в группу asteroids через вызов метода add_child
```
А второй будет функция отвечающая за то, какой астероид и сколько будет появлятся при уничтожении 
```gdscript
func _on_asteroid_exploded(pos, size, points):
    $AsteroidHitSound.play() # Воспроизводим звук столкновения с астероидом
    for i in range(2): # Создаем два новых астероида после разрушения текущего
        match size:  # Используем конструкцию match для определения размера астероида и создания новых астероидов
            Asteroid.AsteroidSize.LARGE:
                spawn_asteroid(pos, Asteroid.AsteroidSize.MEDIUM) # Если астероид был большим, создаем два средних астероида
            Asteroid.AsteroidSize.MEDIUM:
                spawn_asteroid(pos, Asteroid.AsteroidSize.SMALL)   # Если астероид был средним, создаем два маленьких астероида
            Asteroid.AsteroidSize.SMALL:
```

И последним что осталось это подключить сигнал в _ready()

```gdscript
func _ready():
	...
	for asteroid in asteroids.get_children(): # asteroids это узел в котором лежат наши астероиды
		asteroid.connect("exploded", _on_asteroid_exploded)
```

Начнем делать смерть игрока от астероидов. Для этого добавим у игрока 3 переменные и 1 сигнал

* var alive = true
* @onready var sprite = $Sprite2D
* @onready var cshape = $CollisionPolygon2D
* signal died()

Добавим функцию смерти игрока в которой мы выключаем коллизию и спрайт

```gdscript
func die():
	if alive == true:
		alive = false
		sprite.visible = false
		cshape.set_deferred("disabled", true)
		emit_signal("died")
```

Теперь добавим вызов этой функции в скрипте астероида. Для этого у Area2D астероида присоединим сигнал `body_entered` и добавим проверку на то игрок ли это, а для того чтобы условия работало корректно добавим имя класса игроку `Player`

```gdscript
class_name Player extends CharacterBody2D
```

```gdscript
func _on_body_entered(body):
	if body is Player:
		var player = body
		player.die()
```

После этого как и с выстрел возвращаемся в сцену игры и создаем новую функцию
```gdscript
func _on_player_died():
	$PlayDieSound.play()
	print("you are died")
```

И коннектим наш сигнал с функцией

```gdscript
func _ready():
	...
	player.connect("died", _on_player_died)
	...
```

Теперь наш игрок погибает при столкновении с астероидом. Добавим нашему игроку возраждение, для этого создадим у игрока функцию респавна

```gdscript
func respawn(pos):
	if alive == false:
		alive = true
		global_position = pos
		velocity = Vector2.ZERO
		sprite.visible = true
		cshape.set_deferred("disabled", false)
```

Теперь нам нужно определить место спавна. Мы его создадим из узла Node2D, который мы созаддим в основном узле и поставим в месте где хотим, чтобы наш игрок спавнился

![image](https://github.com/Sindikaty/byteschool/assets/158248099/41618f3a-1f64-454c-bdac-10e8ce14dcf0)

Добавим переменную которая будет отвечать заэ ту точку

```gdscript
@onready var player_spawn_pos = $PlayerSpawnPos
```

А теперь добавим в функцию смерти игрока в основной сцене 2 строчки. 1 отвечаемт за место спавна, вторая вызывает функцию респавна

```gdscript
func _on_player_died():
	$PlayDieSound.play()
	print("you are died")
	player.global_position = player_spawn_pos.global_position
	player.respawn(player_spawn_pos.global_position)
```

Спавн работает, однако мы можем делать это бесконечно. Давайте теперь добавим хп и счет за разрушение астероидов

Для создании HUD нам понадобится 2 сцены, в первой мы добавим текстурки hp, а втора сам HUD. Начнем с создания первой сцены, она будет состоять из одного элемента

![image](https://github.com/Sindikaty/byteschool/assets/158248099/a8c60424-2254-4790-af0a-a631dd1763cc)

И ставим его в то место где у анс будут находится hp

![image](https://github.com/Sindikaty/byteschool/assets/158248099/82f2278a-3c2d-4e22-953e-ebeabf709617)

Сохраняем и переходим ко второй сцене. Она состоит из следующих элементов:
* Control (основнрй узел)
* Label (счет)
* BoxContainer (хп)

У `Label` можно задать изначальный текст для проверки того как будет выглядить и добавить шрифт и его размер

![image](https://github.com/Sindikaty/byteschool/assets/158248099/c76bade0-478a-411c-8de4-0ccffef8adc7)

Располагаем элементы как хотит и переходим к работе со скриптом. Создаем его у элемента `Control`. Нам понадобится 3 переменные

```gdscript
@onready var score = $Score:
	set(value):
		score.text = "SCORE: " + str(value)

var uilife_scene = preload("res://scenes/ui_life.tscn")

@onready var lives = $Lives
```

И создаем функцию которая отвечает за отображение количества HP в виде наших кораблей

```gdscript
func init_lives(amount):
	for ul in lives.get_children():
		ul.queue_free()
	for i in amount:
		var ul = uilife_scene.instantiate()
		lives.add_child(ul)
```

Добавляем нащ худ на сцену с уровнем (можно создать CanvarLayer и к нему прикрепить HUD). Переходим в скрипт нашего уровня и также создаем 3 переменные. 

```gdscript
@onready var hud = $UI/HUD

var score := 0:
	set(value):
		score = value
		hud.score = score

var lives: int:
	set(value):
		lives = value
		hud.init_lives(lives)
```

Что такое set() и для чего оно нам нужно. Сеттер — это метод, который автоматически вызывается каждый раз, когда значение связанной с ним переменной изменяется. В нашем случае set(value) определён для переменной lives. Это значит, что каждый раз, когда мы пытаетесь изменить значение lives в коде (например, lives = 5), вместо того чтобы просто обновить значение переменной, Godot автоматически вызывает метод set(value) с новым значением в качестве аргумента value.

В `_ready` добавляем размер изначальных очков и HP

```gdscript
func _ready():
	score = 0
	lives = 3
```

Для отображения хп изменяем функцию `_on_player_died()`

```gdscript
func _on_player_died():
	$Audio/hit.play()
	lives -= 1
	player.global_position = player_spawn_pos.global_position
	if lives <= 0:
		await get_tree().create_timer(2).timeout
		print("you are dead)
	else:
		await get_tree().create_timer(1).timeout
		player.respawn(player_spawn_pos.global_position)
```

А для подсчета очков добавим в функцию `_on_asteroid_exploded`

```gdscript
func _on_asteroid_exploded(pos, size, points):
	...
	score += points
	...
```

Раз мы начали делать UI можно добавить также сцену отвечающую за рестарт игры при смерти. Для этого создадим новую сцену состоящую из следующих элементов:
* Control (Основной узел)
* Label (Текст о рестарке)
* Button (Кнопка обновления игры)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/39a5ccff-303e-4b15-96d2-5164f7b83cdb)
![image](https://github.com/Sindikaty/byteschool/assets/158248099/d38a4e3d-b23b-4937-a424-5252e58ff580)

У кнопки добавляем сигнал который считывает нажатие и прописываем в нем обновление сцены

```gdscript
func _on_restart_button_pressed():
	get_tree().reload_current_scene()
```

Теперь нужно добавить ее на наш уровень в CanvasLayer и делаем изначально невидымым после чего переходим в код 

Создадим переменную отвечающую за нас экран смерти. Также можем добавить в _ready что изначальное отображение отсутствует

```gdscript
@onready var game_over_screen = $UI/GameOverScreen

func _ready():
	game_over_screen.visible = false
 	...
```

И в конце добавим чтобы она включалась при HP <= 0 в функции смерти игрока
```gdscript
if lives <= 0:
	await get_tree().create_timer(2).timeout
	game_over_screen.visible = true
```

Теперь у нас есть HP однако появляется еще одна проблема. Если мы умираем и в точке спавна есть астероид мы при появление сразу же умираем, поэтому мы создадим еще одну сцену которая будет проверять наличие чего-либо в ней.
Состоять сцена будет из следующих элементов:
* Area2D (Основной узел)
* CollisionShape2D (Зона в которой будет проверка)

Создаем скрипт у Area2D и в нем создаем единственную переменную

```gdscript
var is_empty: bool:
	get:
		return (!has_overlapping_areas() && !has_overlapping_bodies())
```

Геттер проверяет наличие перекрывающихся зон и тел, используя две функции: has_overlapping_areas() и has_overlapping_bodies(). Эти функции обычно применяются в контексте физических узлов (например, Area2D или PhysicsBody2D), чтобы проверить, перекрываются ли они с другими физическими объектами в пространстве.

Сохраняем сцену и присоединяем ее в нашем уровне к точке спавна игрока

![image](https://github.com/Sindikaty/byteschool/assets/158248099/ded66e76-cd76-4556-9c70-79d07fbf5ab3)

Всё что нам осталось это в функцию смерти игрока в сцене основного уровня добавить цикл проверяющий это
```gdscript
	else:
		await get_tree().create_timer(1).timeout
		while !player_spawn_area.is_empty:
			await get_tree().create_timer(0.1).timeout
		player.respawn(player_spawn_pos.global_position)
```

Осталось добавить проверку на то жив ли игрок в `_physics_process(delta)` и `_process(delta)` в самом начале

```gdscript
func _physics_process(delta):
	if !alive: return
```

Добавим эффект параллакса, для него создадим отдельный Node2D на нашем уровне. К нему присоединим 3 `GPUParticles2D`.

![image](https://github.com/Sindikaty/byteschool/assets/158248099/d71ed235-262e-45b2-8fe8-06fd9ed47f9e)

Начнем со звезд которые дальше всего от нас и с следовательно их будет больше и двигаться они будут медленнее. Ставим их справа за видимым краем примерно по середине экрана
Свойста мелких

![image](https://github.com/Sindikaty/byteschool/assets/158248099/d6cb9eff-1236-4ba4-a7bc-6772a43e8f74)
![image](https://github.com/Sindikaty/byteschool/assets/158248099/4407acfc-fa43-4e45-8e57-0e051a4c9c62)
![image](https://github.com/Sindikaty/byteschool/assets/158248099/ae2f8557-d45e-4d65-aa31-03cf72071837)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/8e2d5ce1-d380-4275-ad70-9494911cf9db)
![image](https://github.com/Sindikaty/byteschool/assets/158248099/36b13e85-741b-4939-b612-034a8eda1d30)

У средних менять кол-во и размер (кол-во 250 скэйл 3)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/65ac4327-7d91-4476-a1c3-c10fc7be3fa2)


Большие (кол-во 30 скэйл 5)

## Создание спавна астероидов	

Для спавна астероидов создадим Node на нашем основном уровне в котором будут хранится точки спавна и Timer
В нашу Node добавляем Marker2D и переименовываем его, а также Timer, после чего создаем функцию спавна большого астероида

```gdscript
func spawn_large_asteroid(pos):
	spawn_asteroid(pos, Asteroid.AsteroidSize.LARGE)
```

Далее подключаем сигнал timeout таймера к функции, которая будет спавнить астероид.

```gdscript
func _on_timer_timeout():
	spawn_large_asteroid($Spawners/Spawner.global_position)
```

Теперь сделаем, чтобы наш астероид мог спавнится случайно в разных местах. Для этого добавим еще 3 Marker2D и переименуем их. 
Вернемся к коду и создадим массим хранящий точки спавна

```gdscript
var spawn_points = []  # Массив для хранения точек спавна
```

В методе _ready() добавляем наши точки в сам массив

```gdscript
func _ready():
	...	
	spawn_points.append($SpawnPoint1)
	spawn_points.append($SpawnPoint2)
	spawn_points.append($SpawnPoint3)
	spawn_points.append($SpawnPoint4)
```

Комментируем функцию spawn_large_asteroid(pos), она на данный момент нам не нужна, и изменяем функцию _on_timer_timeout(). 

```gdscript
func _on_timer_timeout():
	var random_spawn_point = spawn_points[randi() % spawn_points.size()].global_position
	spawn_asteroid(random_spawn_point, Asteroid.AsteroidSize.LARGE)
```

Поясним что происходит в функции _on_timer_timeout()
randi() — генерирует случайное целое число. spawn_points.size() возвращает количество элементов в списке spawn_points. % используется для того, чтобы случайное число от randi() было ограничено длиной списка spawn_points. 

Теперь добавим, чтобы спавнилось случайно количество астеройдов по этим точка
```gdscript
	var num_asteroids = randi_range(1, 4)  # Случайное количество астероидов от 1 до 4
	for i in range(num_asteroids):
		var random_spawn_point = spawn_points[randi() % spawn_points.size()].global_position
		spawn_asteroid(random_spawn_point, Asteroid.AsteroidSize.LARGE)
```
Однако в таком случае астероиды могут спавнится друг в друге, давайте изменим функцию чтобы такой пробелмы у нас не возникало

```gdscript
func _on_timer_timeout():
	var num_asteroids = randi_range(1, 4)  # Случайное количество астероидов от 1 до 4
	var num_spawn_points = spawn_points.size()
	
	# Спавним астероиды
	for i in range(num_asteroids):
		var spawn_index = randi() % num_spawn_points
		var random_spawn_point = spawn_points[spawn_index].global_position
		spawn_asteroid(random_spawn_point, Asteroid.AsteroidSize.LARGE)
```

В этом коде мы просто генерируем случайный индекс spawn_index в диапазоне от 0 до num_spawn_points - 1, где num_spawn_points - это количество точек спавна. Этот индекс затем используется для выбора случайной точки спавна из массива spawn_points. Таким образом, мы гарантируем, что каждый астероид спавнится в уникальной точке.

Ну и последним что мы сделаем, это не фиксированное время на справн астероидов, а рандомное, для этого нам нужно 
Для этого мы отключим автостарт у таймера и в _ready добавим его изначальный старт

```gdscript
$Spawners/Timer.start()
```
И изменим функцию `_on_timer_timeout()`. В ней бы добавляем минимальное и максимальное значения задержки между запуском таймера, рандомим его и задаем это значение параметру `wait_time`

```gdscript
func _on_timer_timeout():
	var min_delay = 1.0
	var max_delay = 5.0
	var random_delay = randf_range(min_delay, max_delay)
	
	$Spawners/Timer.wait_time = random_delay
	$Spawners/Timer.start()
	
	print("Новое значение wait_time:", $Spawners/Timer.wait_time)
	
	var num_asteroids = randi_range(1, 4)  # Случайное количество астероидов от 1 до 4
	var num_spawn_points = spawn_points.size()
		# Спавним астероиды
	for i in range(num_asteroids):
		var spawn_index = randi() % num_spawn_points
		var random_spawn_point = spawn_points[spawn_index].global_position
		spawn_asteroid(random_spawn_point, Asteroid.AsteroidSize.LARGE)
```

НЕ ФАКТ ЧТО РАБОТАЕТ
```gdscript
    # Проверяем, находится ли астероид за пределами видимого экрана
    var viewport = get_viewport()
    var screen_rect = Rect2(viewport.get_visible_rect().position, viewport.get_visible_rect().size)
    if not screen_rect.has_point(pos):
        # Задаем импульс движения к центру экрана
        var center = screen_rect.position + screen_rect.size / 2
        var direction = (center - pos).normalized()
        var impulse = direction * 100  # Настройте значение импульса по необходимости
        a.apply_impulse(Vector2(), impulse)
```
