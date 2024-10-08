# Шутер

## Урок 1

### Создание игрока

Добавляем спрайт

![image](https://github.com/user-attachments/assets/0215290e-3c1c-4d07-9827-f1d801e7b34d)

Создание передвижения игрока (Код игрока)

![image](https://github.com/user-attachments/assets/96ff3600-d9ae-46b6-8169-f8c9bc47d393)

Для добавление стрельбы нам нужно добавить еще один спрайт - пулю 

![image](https://github.com/user-attachments/assets/bd545db3-41d5-42e6-8951-65137577b81c)

Добавить оригиналу невидимость при старте (Код пули)

![image](https://github.com/user-attachments/assets/7fb59802-f851-4c79-97b4-3ce867928e3a)

А при создании клона телепортироваться на игрока и поворачиватсья в сторону куда направлен игрок после чего делать ее видимой и задавать движение (Код пули)

![image](https://github.com/user-attachments/assets/061099d4-ee9b-4a48-909c-75bd697b104f)

Добавляем стрельбу (Код игрока)

![image](https://github.com/user-attachments/assets/2e9541e7-fb68-483b-9729-49e00cf287c4)

### Создание зомби

При старте запускаем цикл на создание клона зомби

![image](https://github.com/user-attachments/assets/f28974a8-f6ac-4561-b201-584609aba497)

Делаем рандомизацию места спавна клона

![image](https://github.com/user-attachments/assets/bce69b2c-8143-4538-9a85-0c410f61f484)

Движение зомби к игроку

![image](https://github.com/user-attachments/assets/1259ad53-81c7-469c-85f4-9be05fb31d2e)

## Урок 2

### Максимальное количество патронов и перезарядка

Добавим максимальное количество патронов в магазине. Для этого добавим переменную 

![image](https://github.com/user-attachments/assets/22cacffa-0e96-43f1-9bf0-cb07f7938d38)

Обновим код нашей стрельбы (Код игрока)

![image](https://github.com/user-attachments/assets/cbf73c2c-3a65-4b88-b0d5-04187ab9fba2)

Теперь мы делаем определенное количество выстрелов и больше не можем. Для этого добавим перезарядку 

Добавляем переменную

![image](https://github.com/user-attachments/assets/4d9a6842-ddad-4be5-85e2-f72cddb5faa9)

И сам код перезарядки (Код игрока)

![image](https://github.com/user-attachments/assets/2eebdc4b-641c-45e8-a450-7aff35f3533e)

И вновь обновим код стрельбы добавить проверку на перезарядку

![image](https://github.com/user-attachments/assets/766c88c8-5b80-47a8-904a-671d7c121f83)

Также можно добавить спрайт с надписью "перезарядка" для большей информативности

![image](https://github.com/user-attachments/assets/5c99d460-40dc-4a8d-b390-2ba2e0797f7e)

Код перезарядки 

![image](https://github.com/user-attachments/assets/1f7d9627-352f-4aa5-a3c4-b46bc2d9af46)

### Получение урона

Создаем переменную

![image](https://github.com/user-attachments/assets/3d9e448a-4874-45d6-8336-631cdab70334)

Добавляем саму логику получения урона (Код игрока)

![image](https://github.com/user-attachments/assets/1d5f1995-c262-458f-a8ee-f4e30fc514d4)

При получении урона можно добавить следы крови

![image](https://github.com/user-attachments/assets/8055c1c5-672a-4a98-bb57-cb4ef92a5f21)

Код

![image](https://github.com/user-attachments/assets/c5ed8ff6-be5c-4d8e-b2b9-d88fa32d06c2)

Также можно добавить эффект фонарика

![image](https://github.com/user-attachments/assets/faf59787-f8cb-4d60-b7b3-9a0578f02eb9)

Код

![image](https://github.com/user-attachments/assets/e68ef358-9be7-4b44-8a3b-ab96543db1ff)
