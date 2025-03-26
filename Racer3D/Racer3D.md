# Гонка 3Д
Для начала качаем модель машины и разделяем ее на корпус и колеса
![image](https://github.com/user-attachments/assets/abae8673-a980-4ee5-a448-8efe9e243f2a)
![image](https://github.com/user-attachments/assets/5014dec1-50af-426d-a8d5-89ed58764662)

Далее создаем 3ю сцену основным узлом будет `VehicleBody3D`

![image](https://github.com/user-attachments/assets/0c16f063-a91f-4111-8ef4-8063f7fa7451)

К этой сцене прикрепляем корус машины, а колеса перед тем как крепить мы добавим 4 `VehicleWheel3D` и уже к ним прикрепим сцену с колесом

![image](https://github.com/user-attachments/assets/37e57d24-2f22-4bc2-bf0e-5ae727c75722)

Теперь нужно расставить колеса по местам и задать базовые параметры

У машины основное изменение это центр массы, мы его делаем кастомным и немного мсещаем по Y

![image](https://github.com/user-attachments/assets/f64b5821-fe57-4c85-95e9-b3a6c91e6443)

У колес изменений будет намного больше
Для начала у передних мы включем оба параметра, а задних только верхний (1 отвечает за движение, другой за поворот)

![image](https://github.com/user-attachments/assets/8e366464-3f52-43a0-b811-b76e58de3581)

Далее идет настройка самих колес, основным будет радиус

![image](https://github.com/user-attachments/assets/04a0b660-aa46-4792-8b7b-81274b280e2f)

Следующим будет блок Suspension, у него основным будет Stiffness (насколько сильно прогибается подвеска)

![image](https://github.com/user-attachments/assets/105ba63c-4a6f-4184-807a-51e982bbf093)

Настройки машины готовы, добавляем коллизии и пока что базовую камеру.

## Переходим к коду машины

Переменные

```gdscript
@export var STEER_SPEED = 1.5  # Скорость поворота колес (чем выше, тем быстрее колеса поворачиваются)
@export var STEER_LIMIT = 0.6  # Максимальный угол поворота колес
var steer_target = 0  # Желаемый угол поворота колес
@export var engine_force_value = 40  # Сила ускорения двигателя
```

Создаем локальную переменную скорости. В ней мы получает текущую скорость машины и умножаем на Engine.get_frames_per_second() * delta, чтобы скорость была стабильной при разной частоте кадров.

```gdscript
var speed = linear_velocity.length() * Engine.get_frames_per_second() * delta
```

Далее добавляем функцию traction(speed), которая усиливает сцепление колес с дорогой.

```gdscript
func traction(speed):
    apply_central_force(Vector3.DOWN * speed)
```

И вызываем ее в `_physics_process`

```gdscript
func _physics_process(delta):
	var speed = linear_velocity.length()*Engine.get_frames_per_second()*delta
	traction(speed)
```

## Поворот колес

```gdscript
var fwd_mps = transform.basis.x.x
steer_target = Input.get_action_strength("ui_left") - Input.get_action_strength("ui_right")
steer_target *= STEER_LIMIT
```

Переменная fwd_mps определяет направление движения автомобиля

Если нажата `ui_left`, значение > 0, машина поворачивает влево.
Если нажата `ui_right`, значение < 0, машина поворачивает вправо.

А в `steer_target *= STEER_LIMIT` – ограничиваем угол поворота.

## Движением назад

```gdscript
if Input.is_action_pressed("ui_down"):
    if speed < 20 and speed != 0:
        engine_force = clamp(engine_force_value * 3 / speed, 0, 300)
    else:
        engine_force = engine_force_value
else:
    engine_force = 0
```

* Если нажата "ui_down" (назад), машина ускоряется.
* Если скорость низкая (speed < 20), увеличиваем силу ускорения.
* Если скорость высокая, используем стандартное значение engine_force_value.
* Если кнопку отпустить, машина перестанет ускоряться.

## Движение вперед

```gdscript
if Input.is_action_pressed("ui_up"):
    if fwd_mps >= -1:
        if speed < 30 and speed != 0:
            engine_force = -clamp(engine_force_value * 10 / speed, 0, 300)
        else:
            engine_force = -engine_force_value
    else:
        brake = 1
else:
    brake = 0.0
```

* Если нажата "ui_up" (вперед):
  * Если машина не движется назад, используем ускорение.
  * Если скорость < 30, увеличиваем начальный разгон.
  * Если машина движется назад, включаем торможение.
* Если кнопку отпустить, торможение отключается.

## Ручной тормоз

```gdscript
if Input.is_action_pressed("ui_select"):
    brake = 3
    $WheelLeftBack.wheel_friction_slip = 0.8
    $WheelRightBack.wheel_friction_slip = 0.8
else:
    $WheelLeftBack.wheel_friction_slip = 3
    $WheelRightBack.wheel_friction_slip = 3
```

* Если зажата кнопка "ui_select" (ручник):
  * Значение brake = 3 усиливает торможение.
  * wheel_friction_slip = 0.8 уменьшает сцепление колес, создавая эффект дрифта.
* Если кнопку отпустить, сцепление колес возвращается в норму.

## Плавное вращение колес

```gdscript
steering = move_toward(steering, steer_target, STEER_SPEED * delta)
```

Полный код

```gdscript
extends VehicleBody3D


@export var STEER_SPEED = 1.5
@export var STEER_LIMIT = 0.6
var steer_target = 0
@export var engine_force_value = 40


func _physics_process(delta):
	var speed = linear_velocity.length()*Engine.get_frames_per_second()*delta
	traction(speed)
	var fwd_mps = transform.basis.x.x
	steer_target = Input.get_action_strength("ui_left") - Input.get_action_strength("ui_right")
	steer_target *= STEER_LIMIT
	if Input.is_action_pressed("ui_down"):
		if speed < 20 and speed != 0:
			engine_force = clamp(engine_force_value * 3 / speed, 0, 300)
		else:
			engine_force = engine_force_value
	else:
		engine_force = 0
	if Input.is_action_pressed("ui_up"):
		# Increase engine force at low speeds to make the initial acceleration faster.
		if fwd_mps >= -1:
			if speed < 30 and speed != 0:
				engine_force = -clamp(engine_force_value * 10 / speed, 0, 300)
			else:
				engine_force = -engine_force_value
		else:
			brake = 1
	else:
		brake = 0.0
		
	if Input.is_action_pressed("ui_select"):
		brake=3
		$WheelLeftBack.wheel_friction_slip=0.8
		$WheelRightBack.wheel_friction_slip=0.8
	else:
		$WheelLeftBack.wheel_friction_slip=3
		$WheelRightBack.wheel_friction_slip=3
	steering = move_toward(steering, steer_target, STEER_SPEED * delta)

func traction(speed):
	apply_central_force(Vector3.DOWN*speed)
```

# Камера

Добавляем ноду к игроку. а уже к ней камеру, после чего рикрепляем скрипт к камере

![image](https://github.com/user-attachments/assets/4162b211-8709-4c6c-9ad8-8c341024641f)

Определим основные переменные

```gdscript
@export var target_distance = 5  # Дистанция до цели
@export var target_height = 2    # Высота камеры относительно цели
@export var speed := 20.0        # Скорость следования камеры
var follow_this = null           # Объект, за которым следит камера
var last_lookat                  # Последняя точка, куда смотрела камера
```

Определяем объект за которым камера будет сделаить

```gdscript
func _ready():
    follow_this = get_parent()
    last_lookat = follow_this.global_transform.origin
```

В _physics_process() создаем локальные переменные которые будут определять вектор смещения от камеры до объекта и целевая позиция камеры (будем ее изменять).
```gdscript
var delta_v = global_transform.origin - follow_this.global_transform.origin
var target_pos = global_transform.origin
```

Добавляем игнорирование оси y, для того чтобы камера не колебалась вверх-вниз при движении.

```gdscript
delta_v.y = 0.0
```

Добавим ограничение дистанции камеры до цели

```gdscript
if (delta_v.length() > target_distance):
    delta_v = delta_v.normalized() * target_distance
    delta_v.y = target_height
    target_pos = follow_this.global_transform.origin + delta_v
else:
    target_pos.y = follow_this.global_transform.origin.y + target_height
```

Плавное движение камеры

```gdscript
global_transform.origin = global_transform.origin.lerp(target_pos, delta * speed)
```

Плавный поворот камеры
```gdscript
last_lookat = last_lookat.lerp(follow_this.global_transform.origin, delta * speed)
look_at(last_lookat, Vector3.UP)
```

И самое главное, что нужно сделать это влючить параметр Top Level

![image](https://github.com/user-attachments/assets/7fccace3-a8fd-4d8b-a16d-9e0707c20e50)

Полный код камеры

```gdscript
extends Camera3D


@export var target_distance = 5
@export var target_height = 2
@export var speed:=20.0
var follow_this = null
var last_lookat

func _ready():
	follow_this = get_parent()
	last_lookat = follow_this.global_transform.origin

func _physics_process(delta):
	
	var delta_v = global_transform.origin - follow_this.global_transform.origin
	var target_pos = global_transform.origin
	
	# ignore y
	delta_v.y = 0.0
	
	if (delta_v.length() > target_distance):
		delta_v = delta_v.normalized() * target_distance
		delta_v.y = target_height
		target_pos = follow_this.global_transform.origin + delta_v
	else:
		target_pos.y = follow_this.global_transform.origin.y + target_height
	
	global_transform.origin = global_transform.origin.lerp(target_pos, delta * speed)
	
	last_lookat = last_lookat.lerp(follow_this.global_transform.origin, delta * speed)
	
	look_at(last_lookat, Vector3.UP)
```

Боты

1 вариант

![image](https://github.com/user-attachments/assets/0a1db881-6266-4769-b928-f212cbdbe37e)

И в Path3D просто добавляем значение progress_ratio

![image](https://github.com/user-attachments/assets/a9a1aa2e-4d57-43ef-9cf0-fbc5ef711afb)

2 вариант навигация 

Это по сути как делали ботов в шутере, только точкой до которой будет идти бот будет маркер

![image](https://github.com/user-attachments/assets/f8ea603f-190f-45cf-858c-a42b0631fe9b)

Скрипт передвижения бота

![image](https://github.com/user-attachments/assets/d09d245a-d850-44c4-83aa-e4d504fa4350)

Поворот колес

![image](https://github.com/user-attachments/assets/c048011d-5efc-4d37-9081-ebeb146f25e3)

Спидометр со стрелокой

![image](https://github.com/user-attachments/assets/0a757044-89e8-4ba0-adce-86ad9ad64e67)

Через прогрес бар

![image](https://github.com/user-attachments/assets/bd150389-790a-4e7a-af92-f20fc633878d)

Сделать страт финишь, и победу/проигрышь кто раньше приедет

![image](https://github.com/user-attachments/assets/9f1b09af-bfec-4158-afd6-f33af7f67246)

![image](https://github.com/user-attachments/assets/c6c7c704-d430-44ab-bc1a-f84bba990b09)






