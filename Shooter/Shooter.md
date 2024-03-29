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
		Global.camera.shake(0.2, 1) # глобальные настройки тряски камеры
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









