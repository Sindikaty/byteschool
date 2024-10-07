## Сапер
В данном проекте ученики начнут работать со списками и массивами, а также освоят работу с циклами. Ученики будут использовать списки или массивы для хранения информации о состоянии каждой клетки (наличие в клетке бомбы, алмаза или же она пустая).

## 1 урок

### Создание игровой доски

В качестве корневого узла мы выбираем 2D ноду

<img src="https://github.com/Sindikaty/byteschool/blob/patch-1/Miner/img/2D.png?raw=true">

Для создания игровой доски будет использоватся `TileMap` <br>
В свойствах `TileMap` создаем `TileSet` после чего у нас появится сетка которая будет играть роль нашей доски в будущем
> [!Tip]
>  Для удобства дальнейшего добавления тайлов можно сразу поставить в свойствах `TileSet` размер тайла 48x48px .<br>

Добавляем наш набор тайлов, он должен выглядить примерно так:
![image](https://github.com/Sindikaty/byteschool/assets/158248099/bcd51ef5-c1ba-401f-af4c-4871413c440b)

Также для удобства дальнейшей работы можно добавить камеру 

![image](https://github.com/Sindikaty/byteschool/assets/158248099/bc40bbec-0d2a-4019-a396-2eadf3470cca)

### Перейдем к работе со скриптом

Создаем скрипт у `TileMap`. Для начала сделаем появление нашей доски. Тут нам нужно создать 5 переменных. Первая переменная является переменной `список`, 2 переменные `@export` и 2 переменные `const`
> [!IMPORTANT]
> * `Список` это вид переменной у которой сразу может быть несколько значений. Значения могут быть разных типов, например в одном списке может быть как и int (целое число) так и float (число с плавающей точкой). <br>
> * Переменная `@export` позволяет менять значения в свойствах объекта <br>
> * Переменные const используются для определения значений, которые не могут быть изменены во время выполнения программы. <br>
```gdscript
const cells = {
	"wall": Vector2i(0, 2),
}

const defaultLayer = 0
const tileSetId = 0

@export var columns : int = 10
@export var rows : int = 10

```
Для определения координат точек в списке нужно навестись на нужный нам тайл в наборе тайлов и там мы увидим координаты алтаса. Это и есть нужные нам координаты для списка

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0f3a3abb-1f09-4b49-9f2d-88921ed1b1f8)

После объявления переменных создаем метод `ready`. Доску нам нужно создавать при запуске поэтому данный метод нам подходит идеально т.к он вызывается один раз во время инициализации объекта и используется для настройки начальных параметров и компонентов объекта.

```gdscript
func _ready():
	var cell_coord = Vector2(1,1)
	set_tile_set(cell_coord, "wall")
	
func set_tile_set(cell_coord, cell_type):
	set_cell(defaultLayer, cell_coord, tileSetId, cells[cell_type])
```
Добавим 1 ячейку и посмотрим как оно работает после чего перейдем к созданию полноценной доски. Теперь, допустим, мы хотим расставить 10 таких тайлов. Писать 10 раз строчку и менять цифры долго. Поэтому мы будем использовать оператор цикла for  - он позволяет повторять действия несколько раз. 

```gdscript
func _ready():
	for i in rows:
		for j in columns:
			var cell_coord = Vector2(i - rows / 2, j - columns / 2)
			set_tile_set(cell_coord, "wall") # Устанавливаем ячейку с типом "wall"

func set_tile_set(cell_coord, cell_type):
	set_cell(defaultLayer, cell_coord, tileSetId, cells[cell_type])  # Устанавливаем ячейку 
```

Тут мы используем вложенный цикл, что позволит не писать 10 раз строчку и менять цифры вручную. Ниже показано в каком порядке будет заполнятся данный цикл
```gdscript
1. i = rows[0]
   - j = columns[0]
   - j = columns[1]
   - ...
   - j = columns[n]
2. i = rows[1]
   - j = columns[0]
   - j = columns[1]
   - ...
   - j = columns[n]
...
n. i = rows[m]
   - j = columns[0]
   - j = columns[1]
   - ...
   - j = columns[n]
```
Теперь создадим действие, на которое будем открывать ячейки. Для этого заходим в `Проект` - `Настройки проект` - `Список действий` в поле Действие пишем название и нажимаем Добавить. Внизу списка находим действие click, нажимаем "+" - кнопка мыши - ЛКМ. <br>
Создаем метод `_input` который является частью класса Node и вызывается каждый кадр, когда происходит ввод от пользователя. 
> [!IMPORTANT]
>  Параметр "event: InputEvent" в функции "_input" указывает на то, что функция принимает в качестве аргумента объект типа "InputEvent". Это позволяет функции обрабатывать события ввода, такие как нажатие клавиши, клик мыши или события сенсорного экрана. <br>

```gdscript
func _input(event: InputEvent):
	if (Input.is_action_just_pressed("LMB")): 
		var tile : Vector2 = local_to_map(get_global_mouse_position())
		erase_cell(0, tile)
```
Для определения какая ячейка будет удаляться при нажатии нужно получить позицию курсора мыши. При помощи `get_global_mouse_position` мы получаем координаты в пикселях после чего сразу их преобразовываем в координаты доски с помощью `local_to_map`. Функция `erase_cell` используется для удаления ячейки (тайла) на карте. Первый аргумент указывает на слой, на котором находится ячейка, которую нужно удалить. Второй аргумент указывает на позицию ячейки, которую нужно удалить

## Расстановка бомб и алмазов

Добавляем к `Node2D` дочерний узел `TileMap` и переименовываем (например BombsAndDiamonds). Далее добавляем два набора тайлов: <br> 1. Для алмазов <br> 2. Для бомб <br>

### Работа со скриптом

Создаем скрипт у `TileMap`. Тут нам также понадобятся переменная хранящая координаты бомб и алмазов, 2 переменные `@export` и 2 переменные `const`, а также добавляются еще 3 переменные. Две из них хранят количество бомб и алмазов (number_of_mines, numberOfDiamonds), а также переменная хранящая координаты ячеек где уже есть бомба или алмаз (массив). 

> [!IMPORTANT]
> * `Массив` переменная которая может хранить множество значений одного типа, а элементы массива доступны по индексу, начиная с 0<br>

```gdscript
const cells = {
	"bomb": Vector2i(2, 1),
	"diamond": Vector2i(5, 0)
}

@export var columns : int = 10
@export var rows : int = 10
@export var number_of_mines : int = 10
@export var numberOfDiamonds : int = 10

const defaultLayer = 0
const tileSetId = 0

var spawned_coords = []
```
После объявления переменных создаем метод `ready`. Расставить объекты нам нужно также как и доску при запуске игры.

```gdscript
func _ready():
	randomize()
	for i in number_of_mines: 
		var cell_coord_bomb = Vector2(randi_range(- rows/2, rows/2 -1), randi_range(-columns/2, columns /2 -1))
		while spawned_coords.find(cell_coord_bomb) != -1:
			cell_coord_bomb = Vector2(randi_range(- rows/2, rows/2 -1), randi_range(-columns/2, columns /2 -1))

		spawned_coords.append(cell_coord_bomb)

		set_tile_set(cell_coord_bomb, "bomb", 1)
```
Данный блок кода ответает за спавн бомб. Для начала мы запускаем алгоритм по генерации последовательности чисел randomize(). Далее мы создаем цикл который будет выполняться в количестве number_of_mines раз. Внутри цикла создается переменная которая генерирует случайные координаты для бомбы. Далее создается цикл `while` в котором мы проверяем наличие бомбы в ту координату которая выпала в генераторе и если она уже занята цикл начинается заново, после чего координаты бомбы добавляются в массив и ставится на доске.
> [!IMPORTANT]
> * randomize() - это метод, который создает последовательность чисел (Seed) <br>
> * Цикл `while` это цикл, который выполняется до тех пор, пока заданное условие истинно.

Блок с алмазами работает по тому же принципу, что и с бомбами
```gdscript
	for i in numberOfDiamonds:
		var cell_coord_diamonds = Vector2(randi_range(- rows/2, rows/2 -1), randi_range(-columns/2, columns /2 -1))
		while spawned_coords.find(cell_coord_diamonds) != -1:
			cell_coord_diamonds = Vector2(randi_range(- rows/2, rows/2 -1), randi_range(-columns/2, columns /2 -1))
	
		spawned_coords.append(cell_coord_diamonds)
	
		set_tile_set(cell_coord_diamonds, "diamond", 0)

func set_tile_set(cell_coord, cell_type, tileSetId):
	set_cell(defaultLayer, cell_coord, tileSetId, cells[cell_type])
```
Теперь нам нужно определять, нашли ли мы что-нибудь, и что именно: сокровище или бомбу и посчитать их количество. Для этого создаем 2 переменные:

```gdscript
var bomb = 0
var diamond = 0
```
Возвращаемся в конец кода и создаем метод `_input` в котором создаем переменную `tile` для определение координат нажатия, а также `index` в котором мы получаем ID лайла в котором находится данная клетка.
```gdscript
func _input(event: InputEvent):
	if (Input.is_action_just_pressed("LMB")):
		var tile : Vector2 = local_to_map(get_global_mouse_position())
		var index = self.get_cell_source_id(0, tile)
		if index == 1:
			bomb += 1
		if index == 0:
			diamond += 1
	print("Bombs: ", bomb, " Diamonds: ", diamond)
```
Теперь добавим победу либо проигрышь. Для этого создадим скрипт у `основного узла` и в процесс пропишем условия
```gdscript
func _process(delta):
	if $BombsAndDiamonds.bomb >= 3:
		print("lose")
	if $BombsAndDiamonds.diamond >= 3:
		print("win")
```
Однако в таком случае нет никаких ограничений на нажатия и помедить можно найдя 1 алмаз либо же вовсе сразу и победить и проиграть нажав на бомбы и алмазы. Чтобы такого не происходило добавим массив который будет в себе хранить координаты ячеек по которым игрок уже нажал.
```gdscript
var clickedTile = []
```
А также добавить его в условия прибавки значений бомб и алмазов
```gdscript
	if index == 1 and !clickedTile.has(tile):
		clickedTile.append(tile)" 
		bomb += 1
	if index == 0 and !clickedTile.has(tile):
		clickedTile.append(tile)
		diamond += 1
```
Теперь при нажатии на один и тот же алмаз или же бомбу ничего не будет происходить, однако все еще можно сразу и победить и проиграть. Чтобы исправить это для начала добавим 2 переменные в основной узел
```gdscript
var isWin : bool
var isLose : bool
```
И добавим их в наши условия
```gdscript
func _process(delta):
	if $BombsAndDiamonds.bomb >= 3 and !isLose:
		isLose = true
		print("lose")
	if $BombsAndDiamonds.diamond >= 3 and !isWin:
		isWin = true
		print("win")
```
После чего в скриптах обоих `TileMap` добавим еще одну логическую переменню изначально равного true
```gdscript
var canClick : bool = true
```
Добавим переменную в условия связанные с нажатием клавишь (В метотах `_input` у обоих `TileMap`)
```gdscript
if (Input.is_action_just_pressed("LMB")) and canClick:
```
И все что нам осталось это добавить условие в скриптах `TileMap`
```gdscript
func _process(delta):
	if $"..".isLose or $"..".isWin:
		canClick = false
```
Дополнительно можно добавить звук при попадании на алмаз и бомбу, а также звук победы и поражения <br>
Для этого добавляем дочерний узел ![image](https://github.com/Sindikaty/byteschool/assets/158248099/1732f0f8-571d-4069-b2b6-8b9115a6f61a) 
И переноссим наш аудиофайл в свойство Stream
Создав все элементы остается только расставить код в нужных местах
![image](https://github.com/Sindikaty/byteschool/assets/158248099/a2727e61-08bd-4a2b-9269-bc5c48be5496)
![image](https://github.com/Sindikaty/byteschool/assets/158248099/ed302874-c287-41a4-a9c3-74ff684da166) <br>

