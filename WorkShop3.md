# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
- Мальцев Богдан Андреевич
- ФО-220005
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
разработать оптимальный баланс для десяти уровней игры Dragon Picker

Ход работы:

## Задание 1
### Предложите вариант изменения найденных переменных для 10 уровней в игре. Визуализируйте изменение уровня сложности в таблице.


В качестве переменных, влияющих на баланс были обнаружены следующие:

speed - скорость дракона

timeBetweenEggDrops - время между сбросом яиц

leftRightDistance - ширина рамок поля, внутри которых может перемещаться дракон

chanceDirection - шанс изменения направления дракона

Из геймплэя игры видно, что игра расчитана на скорость и координированность действий игрока,
в таком случае усложнение будем производить в этом же направлении. Нашей задачей будет воссоздать экспоненциальный рост сложности.
Методом научного эксперимента определим оптимальные начальные значения, которые будем использовать в первом уровне, а также их коэффициента приращения.
Если переменная с каждым уровнем должна увеличиваться, то коэффициент больше единицы, иначе меньше.
Так как у нас есть возможность влиять сразу на несколько переменных, то для построения зависимости сложности от значения этих переменных представим саму сложность, как функцию 4-ёх переменных:
Её график должен расти экспоненциально.

Сложность = speed * leftRightDistance * chanceDirection / timeBetweenEggDrops
Данная зависимость была построена из следующих соображений:

1. Если дракон будет двигаться быстро, в него будет сложно попасть
2. Если у дракона будут широкие границы, у него будет больше места для уклонения
3. Если дракон будет часто менять направление своего движения, то у него будет более сложный паттерн перемещения, это усложняет попадание
4. Если дракон будет часто сбрасывать яйца, то будет выше шанс, что игрок одно из них уронит.

Визуализация коэффициента сложности, а также данные для всех 10 уровней представлены в следующей таблице: https://docs.google.com/spreadsheets/d/1h0ww1CLQ7n_jWayoj7refsIzpDlBqS4Ht59HR6NNsL0/edit#gid=0

## Задание 2
### Создайте 10 сцен на Unity с изменяющимся уровнем сложности.

Подробнее с результатом выполнения этого задания вы можете ознакомиться скачав данный репозиторий

## Задание 3
### Решение в 80+ баллов должно заполнять google-таблицу данными из Python. В Python данные также должны быть визуализированы.

модель заполнения мы возьмём из WorkShop2, в качестве визуализации мы будем использовать график, который будем строить и выводить с помощью модуля "matplotlib"
Скрипт получается следующий:
```py
import enum
import time

import gspread
import matplotlib.pyplot as plt
import numpy as np


def fill_sheet(row):
    for i in range(difficulty_data.column_count):
        sh.sheet1.update(("A" + str(row)), row - 1)
        sh.sheet1.update((difficulty_data.column_names[i] + str(row)), difficulty_data.values[i])


class EnumIndex(enum.Enum):
    speed = 0
    egg_drop_time = 1
    right_left_distance = 2
    change_direction_chance = 3


class Data:
    values = [1.7, 1.75, 1.5, 0.02]
    increasing = [1.40, 0.85, 1.65, 1.15]
    column_names = ("B", "C", "D", "E")
    column_count = 4
    difficulty_array = []

    def __init__(self):
        self.add_difficult()

    def icrease_difficulty(self):
        for i in range(self.column_count):
            self.values[i] *= self.increasing[i]
        if self.values[EnumIndex.right_left_distance.value] > 10:
            self.values[EnumIndex.right_left_distance.value] = 10
        self.add_difficult()

    def add_difficult(self):
        self.difficulty_array.append(
            self.values[EnumIndex.speed.value] * self.values[EnumIndex.right_left_distance.value] /
            self.values[
                EnumIndex.egg_drop_time.value] * self.values[EnumIndex.change_direction_chance.value])


gc = gspread.service_account(filename='peveraheper-7866f5527595.json')
sh = gc.open("Difficult")

sh.sheet1.update(('A1'), "Level")
sh.sheet1.update(('B1'), "Speed")
sh.sheet1.update(('C1'), "Egg drop time")
sh.sheet1.update(('D1'), "Right-left distance")
sh.sheet1.update(('E1'), "Change Direction Chance")

difficulty_data = Data()

for i in range(1, 11):
    time.sleep(5)
    if i != 1: difficulty_data.icrease_difficulty()
    fill_sheet(i + 1)
fig, ax = plt.subplots()
x = np.arange(1, 11)
y = np.array(difficulty_data.difficulty_array)
ax.plot(x, y)
ax.grid()
plt.xticks(range(1, 11, 1), range(1, 11, 1))
plt.yticks(range(1, 50, 2), range(1, 50, 2))
plt.xlabel("Level")
plt.ylabel("Difficulty")
plt.show()
```

## Выводы

В ходе лабораторной работы была добавлена система уровней сложности в игру Dragon Picker,
которая представляет собой набор из 10 уровней, на каждом из которых происходит экспоненциальное увеличение сложности.
Также был создан дизайн-документ, в котором
была проведена визуализация динамики изменения данных средствами Python и Google Sheets


Цели лабораторной работы были достигнуты.
## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
