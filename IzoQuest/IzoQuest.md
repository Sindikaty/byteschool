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
