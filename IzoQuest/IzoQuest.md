# QuestVille
Проект, направлен на отработку тем, изученных на предыдущих уроках. В ходе создания игры ученики закрепят концепции объектно-ориентированного программирования (ООП) и научатся применять их на практике.

## Урок 1

### Создание базового игрового поля

Создадим базовое поле, на котором уже будет добавлять все остальное, в нашем случае это будет зеленое поле. Для этого создаем `TileMap`. В свойствах `TileMap` создаем `TileSet` после чего у нас появится сетка игрового поля. Т.к. мы создаем изометрический проект, `TileSet` нам нужно настроить под изометрию. Для этого изменим следующие параметры `TileSet`. 

![image](https://github.com/Sindikaty/byteschool/assets/158248099/c7d6a758-b1da-43e8-999c-91fa5c19e291)

Добавим сам тайл травы и зададим его размер в максимально допустимое значение, в нашем случае размер будет равен 128х128

![image](https://github.com/Sindikaty/byteschool/assets/158248099/384b0a1c-0d46-4519-af84-44b375222761)

Закрашиваем область в который мы будет дальше работать травой и переходим к созданию зданий. Сделать это быстро можно зажав Ctrl_Shift и левой кнопкой выделить обрасть заполнение. ВЫглядить будет примерно так

![image](https://github.com/Sindikaty/byteschool/assets/158248099/5e0c0e16-8689-4f96-9c79-36117279b811)

### Создание зданий

Прежде чем создавать здания поменяем рендеринг текстур, нужно нам это для того, чтобы картинки были не мыльными, а пиксельными

![image](https://github.com/Sindikaty/byteschool/assets/158248099/47098377-cd91-4480-b12b-6fe47de1ff29)

Теперь для каждого здания нужно создать отдельную сцену родительским узлом которой будет `StaticBody2D`, а дочерними узлами будут `Sprite2D` и `CollisionPolygon2D`

Для `Sprite2D` используем текстурки зданий, а у `CollisionPolygon2D` задаем зону по которой не сможет ходить игрок, условно зону находящуюся под окнами первого этажа. Это нам позволит в будущем сделать эффект того, что игрок заходит за здание

Примеры зданий:

| Название  | Скриншот |
| ------------- | ------------- |
| Обычный дом   | ![image](https://github.com/Sindikaty/byteschool/assets/158248099/e6ecf0cc-9317-43b7-aeaf-5356ff3451e5)  |
| Замок  | ![image](https://github.com/Sindikaty/byteschool/assets/158248099/1925b995-caf5-429d-b3e1-38b4cd8fb6b5) |
| Вилла  | ![image](https://github.com/Sindikaty/byteschool/assets/158248099/07038bb7-b178-4953-8706-c610dcb1a53e) |
| Торговцы | ![image](https://github.com/Sindikaty/byteschool/assets/158248099/7177fadf-39b3-40c3-b043-692a5636f88c) |
| Таверна | ![image](https://github.com/Sindikaty/byteschool/assets/158248099/6ac6a5a1-9eeb-4d4d-b4ae-70f431823551) |

Теперь можно расставить эти здания по нашей карте, как делать это зависит от фантазии ученика, пример того как можно расставить ниже (для всех зданий нужно создать узел Node2D в котором будут хранится все здания и в будущем игрок и NPC)
![image](https://github.com/Sindikaty/byteschool/assets/158248099/747376c6-6f29-4a61-a099-5c2c53ff2c36)

### Создание дорожек

Для дорожек нам нужен еще один `TileMap` со следующими параметрами

![image](https://github.com/Sindikaty/byteschool/assets/158248099/22f8813e-a7be-4423-8157-4ec4c1913f28)


Добавим тайл дорожки и зададим его размер примерно в такие значение, их можно подобрать по желанию ученика

![image](https://github.com/Sindikaty/byteschool/assets/158248099/f425f890-51a8-4de3-9afb-56f9bf78cfca)

В результате мы получим примерно следующее:

![image](https://github.com/Sindikaty/byteschool/assets/158248099/cc79d761-d492-4ad1-af33-1af0ce036e41)

### Создание игрока (передвижение)

Начнем создание игрока. Основным узлом будет CharBody и к нему мы присоединяем AnimSprite, Camera и CollisionPolygon

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0e7a0632-0e6e-42e6-80d9-52f0f03e4bdd)

У спрайта создаем 4 анимации:
* Стоим и повернуты вверх
* Стоим и повернуты вниз
* Идем вверх
* Идем вниз

Коллизию задаем у нижней части игрока

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0f4a2f0a-33d7-4d67-a17f-de9fa8413c0e)

Переходим к скрипту, изначально создаем 2 переменные.

```gdscript
var motion = Vector2()
@export var speed = 200
```

И теперь задаем само передвижение + анимации
```gdscript
func _physics_process(delta):
	motion = Vector2()
	if Input.is_action_pressed("up"): # нажимаем на W
		motion.y = -speed # уменьшаем Y с значением 200
		$AnimatedSprite2D.play("walk_up") # Анимация вверх
	if Input.is_action_pressed("right"): 
		motion.x = speed
		$AnimatedSprite2D.flip_h = false
		if $AnimatedSprite2D.animation == "idle_down":
			$AnimatedSprite2D.play("walk_down")

	if Input.is_action_pressed("left"):
		motion.x = -speed
		$AnimatedSprite2D.flip_h = true
		if $AnimatedSprite2D.animation == "idle_down":
			$AnimatedSprite2D.play("walk_down")

	if Input.is_action_pressed("down"):
		motion.y = speed
		$AnimatedSprite2D.play("walk_down")
	
	if motion == Vector2():
		$AnimatedSprite2D.play("idle_down")

  	set_velocity(motion.normalized() * speed) # Это нужно для того чтобы скорость не складывалась если мы идем по диагонали
	move_and_slide()
```

Также можно добавить небольшое ускорение, можно такое как показано ниже
```gdscript
if Input.is_action_pressed("shift"):
		speed = 300
	else:
		speed = 200
```

А можно такой вариант с ограниченным временем действия

```gdscript
	if Input.is_action_pressed("run") and stamina > 5:
		speed = 80
		stamina -= 1
	else:
		speed = 50
		stamina += 1
		
	if stamina <= 6:
		speed = 50
```

Однако в таком случае нужно задать максимальное значение стамины и задать следующее условие

```gdscript
	if stamina > max_stamina:
		stamina = max_stamina
```

Можно сделать управление мышкой
```gdscript
var speed = 100
var stop = Vector2()

func _physics_process(delta):
	var agent = $NavigationAgent2D2
	var next = agent.get_next_path_position()
	agent.target_position = stop
	velocity = position.direction_to(next)*speed
	
	if not agent.is_navigation_finished():
		#update_animation(velocity)
		move_and_slide()
	#else:
		#animated_sprite.play("idle_down")
		
func _input(event):
	if InputEventMouseButton and event.is_pressed():
		stop = get_global_mouse_position()
```
Анимашки

```gdscript
@onready var animated_sprite = $AnimatedSprite2D

#func update_animation(velocity):
	#if velocity.x > 0:
		#animated_sprite.flip_h = false
		#animated_sprite.play("walk_down")
	#elif velocity.x < 0:
		#animated_sprite.flip_h = true
		#animated_sprite.play("walk_down")
	#elif velocity.y < 0:
		#animated_sprite.play("walk_down")
	#elif velocity.y > 0:
		#animated_sprite.play("walk_down")
 ```

Теперь для корректного отображения игрока проходящего рядом со зданиями включим следующий параметр у Node2D в котором всё хранится

![image](https://github.com/Sindikaty/byteschool/assets/158248099/1346e507-98ca-4b85-8c27-47a9aea0e53e)

Это позволит создать эффект того, что игрок заходит за здание

![image](https://github.com/Sindikaty/byteschool/assets/158248099/e3ab41fb-24fa-47c1-a5d0-df79d7e43d69)

И последним, что мы создадим на этом уроке будет самый обычный NPC не дающий заданий. Основным узлом будет CharBody и к нему мы присоединяем AnimSprite и CollisionPolygon. Анимации и коллизию делаем как у игрока полсе чего переходим к скрипту.

![image](https://github.com/Sindikaty/byteschool/assets/158248099/929a2d10-27ce-45d5-a9e5-fd5bdeea1175)

Для реализации нам понадобятится 4 переменные

```gdscript
var move_dir = Vector2.ZERO
var move_speed = 50
var time_to_change_dir = 2
var timer = 0
```

После чего в process прописываем случайное перемещение бота каждые 2 секунды и я

```gdscript
func _process(delta):
	timer += delta;
	if timer >= time_to_change_dir:
		timer = 0
		move_dir = Vector2(randf_range(-1, 1), randf_range(-1, 1)).normalized()

	if move_dir == Vector2(0,0):
		$AnimatedSprite2D.play("idle")
	else:
		move_anim()
	
	set_velocity(move_dir * move_speed)
	move_and_slide()

func move_anim():
	if move_dir.x > 0:
		$AnimatedSprite2D.flip_h = false
	else:
		$AnimatedSprite2D.flip_h = true
		
	if move_dir.y > 0:
		$AnimatedSprite2D.play("walk_dwn")
	else:
		$AnimatedSprite2D.play("walk_up")
```

На уровне добавляем отдельный узел Node2D где будут хранится все NPC, после чего присоединяем туда наших ботов.

## Урок 2

Начнем урок с создания торговца у которого мы можем купить питомца. Для того чтобы мы покупали питомца не у пустоты, а у NPC добавим его спрайт к нашему месту продажи, например так

![image](https://github.com/Sindikaty/byteschool/assets/158248099/7c821aa8-5737-4f83-8fc5-34ca97c27841)

Теперь нам нужно создать сам диалог с продавцом, для этого создадим `CanvasLayer` и в нем узел `Control`. ТАкже нам понадобятся следующие элементы:
* Panel x2 (рамка диалогового окна и рамка текста)
* RichTextLabel (Текст самого NPC)
* Button x2 (ВЫбор питомца)
* AnimatedSprite2D (Скин NPC)

И при водим примерно к этому виду

![image](https://github.com/Sindikaty/byteschool/assets/158248099/dfdd6df8-3acb-45c2-ba87-2030831d5ad6)

Добавим появление этого диалога с торговцем. Для этого создадим зону с коллизией

![image](https://github.com/Sindikaty/byteschool/assets/158248099/13fbe0d8-4b58-4a79-8d17-bc9ee9f960a9)

Присоединяем сигналы на вход и выход из зоны торговца и в них прописываем включение/выключение диалога соответственно

```gdscript
var in_area = false

func _on_torgovec_body_entered(body):
	if body.name == "Player":
		in_area = true
		if in_area == true:
			$CanvasLayer/torgovec_dialog.visible = true


func _on_torgovec_body_exited(body):
	if body.name == "Player":
		in_area = false
		if in_area == false:
			$CanvasLayer/torgovec_dialog.visible = false
```

Теперь неужно добавить самих питомцев которых будет продавать NPC. Для каждого питомца создаем отдельные сцены состаящие из следующих узлов

![image](https://github.com/Sindikaty/byteschool/assets/158248099/2050358a-7455-48a0-b58f-5dd04af9afc2)

Для создания передвижения питомца нам понадобится 2 переменные 

```gdscript
@export var speed = 100
var player_position
```

В методе `_ready` мы определяем позицию игрока, после чего в `_physics_process` мы также определяем позицию игрока и создаем локальную переменную которая определяет расстояние от питомца до игрока, после чего создаем проверку на это расстояние, если оно меньше 50 питомец двигается к нам. 

```gdscript
func _ready():
	player_position = $"../Player".position

func _physics_process(delta):
	player_position = $"../Player".position
	var distance = position.distance_to(player_position)
	 
	if distance > 50:
		var direction = (player_position - position).normalized()
		set_velocity(direction * speed)
		move_and_slide()

		if direction.x > 0 and speed > 0:
			$AnimatedSprite2D.play("walk")
			$AnimatedSprite2D.flip_h = false
		elif direction.x < 0 and speed > 0:
			$AnimatedSprite2D.play("walk")
			$AnimatedSprite2D.flip_h = true
	else:
		$AnimatedSprite2D.play("idle")
```

Все что нам осталось это добавить спавн питомцев при нажатии на кнопку. Для этого в основном уровне создаем 2 переменные в которые мы предварительно загружаем сцены.

```gdscript
var pet_wolf = preload("res://pet.tscn")
var pet_bear = preload("res://bear.tscn")
```

Присоединяем 2 сигнала на кнопки и создаем в них клон наших питомцев.

```gdscript
func _on_option_1_pressed():
		var pet = pet_wolf.instantiate()
		pet.position = $".".position
		get_parent().add_child(pet)
		$"../../CanvasLayer/torgovec_dialog/RichTextLabel2".text = "Отлинчый выбор! Он в цирке не выступает"

func _on_option_2_pressed():
		var pet = pet_bear.instantiate()
		pet.position = $".".position
		get_parent().add_child(pet)
		$"../../CanvasLayer/torgovec_dialog/RichTextLabel2".text = "Ну слушай... Зато к тебе и близко никто не подойдет"
```

Можно также добавить проверку на количество питомцев, для этого добавим переменную и проверку

```gdscript
pet_count = 0

func _on_option_1_pressed():
	if pet_count < 1:
		var pet = pet_wolf.instantiate()
		pet_count += 1
		pet.position = $".".position
		get_parent().add_child(pet)
		$"../../CanvasLayer/torgovec_dialog/RichTextLabel2".text = "Отлинчый выбор! Он в цирке не выступает"
	else:
		$"../../CanvasLayer/torgovec_dialog/RichTextLabel2".text = "У тебя уже есть питомец"
		
func _on_option_2_pressed():
	if pet_count < 1:
		var pet = pet_bear.instantiate()
		pet_count += 1
		pet.position = $".".position
		get_parent().add_child(pet)
		$"../../CanvasLayer/torgovec_dialog/RichTextLabel2".text = "Ну слушай... Зато к тебе и близко никто не подойдет"
	else:
		$"../../CanvasLayer/torgovec_dialog/RichTextLabel2".text = "У тебя уже есть питомец"
```

Можно добавить проверку на диалог, чтобы маршрут не строился в случае диалога. Для этого создаем глобальный скрипт и добавляем изменение в зависимости от ситуации.

Если мы зашли в зону диалога делаем переменную true

```gdscript
func _on_area_2d_body_entered(body):
	if body.name == "player":
		in_area = true
		if in_area == true:
			GlobalScript.dialog = true
			$"../../CanvasLayer/Pet_tailor".visible = true
```

А если мы нажали по выбору питомца или он уже у нас есть мы делаем ее false

```gdscript
func _on_wolf_2_pressed():
	if pet_count < 1:
		var pet = pet_bear.instantiate()
		pet_count += 1
		pet.position = $".".position
		get_parent().add_child(pet)
		$"../../CanvasLayer/Pet_tailor/RichTextLabel".text = "Ну слушай... Зато к тебе и близко никто не подойдет"
		GlobalScript.dialog = false
	else:
		$"../../CanvasLayer/Pet_tailor/RichTextLabel".text = "У тебя уже есть питомец"
		GlobalScript.dialog = false
```

Следующего NPC которого мы сделаем будет стражник который рассказывает что где находится. Нам понадобятся следующие узлы:
* Area2D
* AnimatedSprite2D
* CollisionShape2D
* Label

MPC будет выглядить примерно так

![image](https://github.com/Sindikaty/byteschool/assets/158248099/c207a3d5-15f4-4dde-8834-0ef7d2b5b3c3)

Теперь нам нужно сздать сам диалог со стражником. Делать это мы будем в ранее созданном `CanvasLayer`. Основным узлом бьудет `Control` и к нему присоединяем следующие узлы:
* Panel х2
* RichTextLabel
* Button x4

Расставляем все и создаем скрипт у Control после чего присоединяем сигналы к кнопкам.

![image](https://github.com/Sindikaty/byteschool/assets/158248099/d27b2410-92b7-4c29-bbec-002ef9d1d688)

```gdscript
var variation_fact = 0


func _on_answer1_pressed():
	$RichTextLabel.text = "Просто иди дальше по дороге. Не ошибешься. Они полказны угрохали на этот замок."


func _on_answer2_pressed():
	$RichTextLabel.text = "Парень, она слева от тебя."


func _on_answer3_pressed():
	randomize()
	variation_fact = int(randf_range(0, 10))
	if variation_fact == 0:
		$RichTextLabel.text = "Что-то интересное? Я тебе что, библиотекарь что ли?"
	if variation_fact == 1:
		$RichTextLabel.text = "Говорят в городе завелся некий Давахин... Ничего не слышал об этом?"
	if variation_fact == 2:
		$RichTextLabel.text = "Хочешь что-то интересное - дуй в таверну. Там наслушаешься"
	if variation_fact == 3:
		$RichTextLabel.text = "Ты никому не говори, но я на самом деле хотел быть в разведотряде."
	if variation_fact == 4:
		$RichTextLabel.text = "А что если я тоже избранный? Я вон, в детстве в чан с облепиховым морсом упал."
	if variation_fact == 5:
		$RichTextLabel.text = "Если ты слышал местные слухи о слуге, который убил своего хозяина и стал стражником, то это не я"

func _on_answer4_pressed():
	$RichTextLabel.text = "Ага. Всего хорошего."
```

Теперь нужно добавить собственно появление этого диалога. Для этого создаем скрипт у Area2D нашего NPC

```gdscript
func _on_guard_body_entered(body):
	if body.name == "Player":
		$Label.visible = true
		$Label.text = "Чего тебе, приключенец?"
		$"../../CanvasLayer/dialog".visible = true
		
	if body.is_in_group("NPC"):
		$Label.visible = true
		$Label.text = "Не мешайте службе, житель"

func _on_guard_body_exited(body):
	if body.name == "Player":
		$Label.visible = false
		$"../../CanvasLayer/dialog".visible = false
		
	if body.is_in_group("NPC"):
		$Label.visible = false
```

## Урок 3

Этот урок мы начнем с создания NPC который будет выдавать нам задание найти объект. Структура NPC следующая

![image](https://github.com/Sindikaty/byteschool/assets/158248099/7e395b2a-b1c2-4bf8-902d-d73bc5410e1d)

Создаем скрипт у Area2D и создаем следующие переменные

```gdscript
var task = 0;
var task_accept = false
```

Сначала добавим появлени и исчезновение диалога с NPC при входе в зону

```gdscript
func _on_guild_of_heroes_body_entered(body):
	if body.name == "Player":
		$Label_quest.visible = true
		$Button.visible = true

func _on_guild_of_heroes_body_exited(body):
	if body.name == "Player":
		$Label_quest.visible = false
		$Button.visible = false
```

В ранее созданном глобальном скрипте доьбавим список в который мы будем добавлять и удалять задания


```gdscript
var tasks = []

func add_task(task):
	tasks.append(task)

func remove_task(task):
	tasks.erase(task)
```

Также нам нужно создать само яблоко, оно состоит из следующих узлов

![image](https://github.com/Sindikaty/byteschool/assets/158248099/82c99238-e46e-4a22-ad08-9c4d251496eb)

Еще можно создать список текущих квестов, по сути это просто Label в который мы будем вносить текст

![image](https://github.com/Sindikaty/byteschool/assets/158248099/6a16229d-fb1c-433c-9626-17b97de5429b)

Возвращаемся к скрипту зоны. Присоединяем сигнал к кнопке 

```gdscript
func _on_Button_pressed():
	task_accept = true
	$Label_quest.visible = false
	$Button.visible = false
	$"../../CanvasLayer/Quest_list/RichTextLabel".text += "Волшебное яблоко"
	Quest.add_task(task)
	#print("Current quest is: ", task)
	task = 1
	$"../Apple_point".visible = true
```

Теперь у нас на карте появляется яблоко которое нам нужно найти. Добавим у него сигнал при входе в него

```gdscript
func _on_Apple_point_body_entered(body):
	if body.name == "Player":
		if task == 1:
			$"../Apple_point".visible = false
			task = 2
			$"../../CanvasLayer/Quest_list/RichTextLabel".text = "Текущие квесты: \n"
			task_accept = true
			$"../../CanvasLayer/Quest_list/RichTextLabel".text += "Вернись к искателям приключений"
```

Все что осталось дополнить вход в зону

```gdscript
func _on_guild_of_heroes_body_entered(body):
	if body.name == "Player":
		$Label_quest.visible = true
		$Button.visible = true
		if task == 2:
			$Button.visible = false
			Quest.remove_task(task)
			$Label_quest.text = "Отлично! Мы подумаем над твоим запросом"
			$Label_quest.visible = true
```

Также создадим стены нашей деревни. Для этого создадим `tilemap` со следующими параметрами `tileset`

![image](https://github.com/Sindikaty/byteschool/assets/158248099/2c59d124-1003-4d29-9485-e6fd1ccf2168)

А также нужно включить автоматическую сортировку слоев у `tilemap`

![image](https://github.com/Sindikaty/byteschool/assets/158248099/74ef6760-a48b-435d-90fe-c0299212ff38)

Получится примерно следующее

![image](https://github.com/Sindikaty/byteschool/assets/158248099/74f1cd9b-fe2a-470f-905e-71f2a63d9265)

Также можно сделать таверну и попробовать добавить там квест на поиск предмета

Таверна состоит из следующий узлов

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0fad591c-bd92-499d-b657-bb2c815a13d2)

Для входа создаем зону 

![image](https://github.com/Sindikaty/byteschool/assets/158248099/b3ca2b6b-f0b6-481d-8309-4d7ee6111308)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/86d94132-3cf8-44f4-8622-bfc04daba635)

Для указание точки спавна при выходе можно добавить следующее в код открывания сцены

```gdscript
$player.global_position = Vector2(511,659)
```

Скрипт бота

![image](https://github.com/Sindikaty/byteschool/assets/158248099/df573500-e046-4641-b97f-0c9ebf887ae9)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/f79d3ba1-9b8c-44e3-a57b-45bf086310c3)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0ebb27bd-ed23-4d7e-a90c-44cc8063e210)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/ce99219c-d5b5-48f6-959c-cbcc1b738a76)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/f1fb5c1a-4a43-48a2-94f1-362fe7beb981)

У таверны добавяем физические слои, чтобы нельзя было проходить сквозь стены

Сделаем меню игры, для этого солздаем отдельную сцену и делаем ее для запуска первой

![image](https://github.com/Sindikaty/byteschool/assets/158248099/ce93da98-e596-4e04-a501-3297707d4c14)

Состоит из следующий элементов

![image](https://github.com/Sindikaty/byteschool/assets/158248099/97cfcfd0-1fa2-4588-9f2b-f8f2390e5ff4)

И выглядит примерно так

![image](https://github.com/Sindikaty/byteschool/assets/158248099/14496949-6822-48e2-a574-7312fd0ee278)

Создадим сундук с которого будут выпадать монеты для покупки питомцев. Состоит из следующих узлов

![image](https://github.com/Sindikaty/byteschool/assets/158248099/f1e4d9d8-7b4f-41ce-a163-3e031bd55521)

Создаем сцену с монеткой

![image](https://github.com/Sindikaty/byteschool/assets/158248099/027c17eb-c4a9-43f9-98a0-5c29342375b0)

Код сундука

```gdscript
extends Area2D

@export var coin_scene: PackedScene
@export var min_coins = 1
@export var max_coins = 5
var player_in_range = false
var use = false

func _on_body_entered(body):
	if body.name == "player":
		$Label.visible = true
		player_in_range = true
		$Label.text = "Нажмите Е чтобы открыть сундук"

func _process(delta):
	if player_in_range and Input.is_action_just_pressed("use1") and use == false:
		open_chest()
		use = true

func open_chest():
	var coin_count = randi() % (max_coins - min_coins + 1) + min_coins
	for i in range(coin_count):
		var coin_instance = coin_scene.instantiate()
		var chest_position = global_position
		var offset = Vector2(randf() * 64 - 32, randf() * 64 - 32)  # Случайное смещение вокруг сундука
		coin_instance.position = chest_position + offset
		get_parent().add_child(coin_instance)
	$Label.visible = false  # Скрыть метку после открытия сундука
	player_in_range = false  # Игрок больше не в зоне сундука
```

Код монетки

```gdscript
extends Area2D

func _on_body_entered(body):
	if body.name == "player":
		GlobalScript.Coins += 1
		$"../../CanvasLayer/QuestList/Label".text = "Количество монет - " + str(GlobalScript.Coins)
		queue_free()
```

И можно изменить торговца и добавить ему продажу петомцев за монетки

```gdscript
	if pet_count < 1 and GlobalScript.Coins >= 5:
		var pet = pet_snake.instantiate()
		pet_count += 1
		pet.position = $".".position
		get_parent().add_child(pet)
		$"../../CanvasLayer/Pet_tailor/RichTextLabel".text = "Отлинчый выбор!"
		GlobalScript.dialog = false
	elif pet_count > 1 and GlobalScript.Coins >= 5:
		$"../../CanvasLayer/Pet_tailor/RichTextLabel".text = "У тебя уже есть питомец"
		GlobalScript.dialog = false
	elif pet_count < 1 and GlobalScript.Coins <= 5:
		$"../../CanvasLayer/Pet_tailor/RichTextLabel".text = "У тебя не хватает денег, сходи заработай"
		GlobalScript.dialog = false
```

Пожилые анимации монеток

![image](https://github.com/Sindikaty/byteschool/assets/158248099/4355c992-c96b-40f7-b88b-e3325d12ed9e)

а
