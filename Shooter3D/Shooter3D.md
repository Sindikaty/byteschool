# Космошутер

# Создание глобального освещения

Создаем узел `WorldEnvironment`. Настройка самого узла показана ниже

![image](https://github.com/user-attachments/assets/4db70300-fbe8-4633-b01f-4718e407b860)

# Создание базовой платформы для персонажа

Создадим мини платформу для изначальных тестов камеры персонажа, состоять она будет из следующий узлов<br>
* MeshInstance3D
  * StaticBody3D
    * CollisionShape3D

# Персонаж

Создаем новую сцену, основным узлом будет CharacterBody3D. Игра от первого лица, поэтому модельку нам добавлять не нужно, в будущем мы добавим руки персонажу. но пока, что остановимяся на камере и коллизии

![image](https://github.com/user-attachments/assets/8e8cc256-8011-4e6f-bd8b-54414fbb3d92)

Добавляем игрока на сцену и запускаем, будет выглядеть примерно так

![image](https://github.com/user-attachments/assets/e609e7c9-084d-4994-96e6-c72bcfd56ffb)

Начинаем программировать камеру и движиение игрока. Для начала поместим камеру в отдельный узел Node3D в игроке

![image](https://github.com/user-attachments/assets/75ea3875-8d14-4b74-9810-c820f5ea8558)

Теперь сбрасываем координаты у камеры и перемещаем Node3D, вместе с ним будет перемещатся камера

Для начала уберем видимость курсор мыши

```gdscript
func _ready() -> void:
	Input.mouse_mode = Input.MOUSE_MODE_CAPTURED
```

Создаем переменные которые пригодятся нам для создания камеры

```gdscript
var mouse_input : bool = false # включение/отключение ввода мыши
var tilt_input : float # для ограничения камеры вверх/вниз
var mouse_rotation : Vector3 # изменение вращения мыши в пространстве
var camera_rotation : Vector3 # изменение вращения в пространстве

@export var _camera_contoller : Camera3D
```







