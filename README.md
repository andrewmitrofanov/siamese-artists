# Определение художника по изображению картины
(С использованием Siamese Networks и Triplet Loss)

## Задача

Научить нейронную сеть определять художника по изображению картины, которую сеть ранее "не видела".

## Используемые технологии

В основе проекта - использование "сиамских" сетей (впервые предложенных в работе [1]) и "триплет лосса" [2].


Схематично структуру используемой сети можно изобразить так:


![Diagram with siamese network with triplet loss](/images/siamese-triplet-diagram.jpg)


На вход сети подаётся три изображения, два из которых (якорь и положительный пример) принадлежат одному классу, а третье (отрицательный пример) - другому.

На основе изображений сеть формирует три вектора (embeddings), которые затем поступают в функцию потерь - triplet loss.

Основная цель triplet loss - сформировать такое векторное пространство, в котором вектора, полученные на основе изображений одного класса, располагалются близко друг к другу, а на основе на основе разных, - далеко друг от друга.

В хорошо сформированном пространстве вектора разных классов будут образовывать чётко различимые кластеры.

## Обучение сети на простом датасете

Для начала было решено проверить работоспособность выбранной конфигурации сети на простом наборе данных.

### Описание датасета

В качестве такого набора был выбран **Stanford Dogs Dataset**, содержащий порядка 19 000 изображений 120 пород собак (все они входят в ImageNet).
Все фотографии были тщательно изучены и в итоге оставлены только те, на которых присутствует ровно одно чётко различимое животное.

### Конфигурация сети

В качестве свёрточной сети была выбрана сеть с архитектурой **senet154**, предварительно обученная на ImageNet. Таким образом сеть ранее "видела" весь датасет.

Вся сеть заморожена.
Полносвязные слои сети были заменены на: 2048->128+Tanh(). Они разморожены.
Обучение: 5 эпох.

### Полученные результаты

Достигнутая точность **(normalized accuracy): 0.82**.

Для проверки свойств сформированного векторного пространства с помощью CNN были получены вектора для 12 изображений "Китайской хохлатой" породы собак, которой нет в ImageNet и которую сеть прежде "не видела".

8 из них были добавлены в пространство векторов, соответствующих всем изображениям набора данных.
Затем обучен классификатор KNN и протестировано, к какой категории он отнесёт оставшиеся 4 вектора. **Accuracy: 100%**.

Вывод - выбранная конфигурация вполне работоспособна, можно провести её испытания на более сложном для обучения датасете.

## Обучение сети на полноценном датасете
### Описание датасета

Для обучения сети был собран датасет, состоящий из произведений 30 художников:

| Художник | Стиль | Количество изображений |
| ---           | --- | --- |
| Альбрехт Дюрер | ренессанс | 139 |
| Энди Уорхол | поп-арт | 209 |
| Борис Кустодиев | реализм, импрессионизм, соцреализм и модерн | 319 |
| Камиль Писсаро | импрессионизм | 666 |
| Клод Моне | импрессионизм | 930 |
| Дмитрий Левицкий | классицизм | 93 |
| Эдгар Дега | импрессионизм | 388 |
| Эдуард Кортес | постимпрессионизм | 135 |
| Анри де Тулуз-Лотрек | постимпрессионизм | 170 |
| Анри Матисс | фовизм | 491 |
| Хиро Ямагата | поп-арт | 101 |
| Илья Репин | реализм | 362 |
| Иван Айвазовский | романтизм | 520 |
| Иван Шишкин | реализм | 317 |
| Джон Эверетт Милле | прерафаэлитизм | 89 |
| Джон Слоун | импрессионизм | 93 |
| Джон Уильям Уотерхаус | прерафаэлитизм | 102 |
| Карл Брюллов | академизм, романтизм, классицизм | 114 |
| Лев Бакст | модерн | 85 |
| Мартирос Сарьян | ? | 398 |
| Николай Богданов-Бельский | реализм, импрессионизм | 165 |
| Орест Кипренский | романтизм, академизм | 82 |
| Пабло Пикассо | кубизм, сюрреализм, постимпрессионизм | 434 |
| Пьер Огюст Ренуар | импрессионизм | 1102 |
| Сальвадор Дали | сюрреализм, дадаизм, кубизм | 206 |
| Сандро Боттичелли | реализм | 102 |
| Сергей Андрияка | классическая многослойная акварель | 85 |
| Василий Суриков | реализм, импрессионизм, академизм | 174 |
| Винсент ван Гог | постимпрессионизм | 592 |
| Зинаида Серебрякова | экспрессионизм, модерн, импрессионизм и проч. | 296 |

Были отобраны только полноценные произведения (всего около 9000 картин). В датасете нет эскизов, набросков, незавершённых работ и т.п.

Разделение на train/val: 80/20.

### Краткая описание стилей живописи:

![Art styles](/images/art-styles.jpg)

Живописные стили довольно сильно отличаются. Ожидаем, что сеть выучит эти различия и будет легко отличать художников, творивших в разных стилях.

### Конфигурация сети

Сеть, показавшая наилучшие результаты, имеет следующую конфигурацию:
* Свёрточная сеть - **resnet101**. Заморожена, кроме layer4.
* Полносвязные слои сети были заменены на: 2048->128+Tanh(). Они разморожены.
* Adam. Lr = 0.0005. lr_scheduler.StepLR(optimizer, 10, gamma=0.2)
* 30 эпох.
* Стратегия формирования батча - сбалансированный батч, состоящий из 8 классов, по 16 примеров в каждом.
* Лосс считается между всеми возможными триплетами. (Про стратегии формирования батча и методы выбора троек для подсчёта лосса упоминается в [2] и [3].)

### Полученные результаты

Достигнутая точность **(normalized accuracy): 0.73**.
![Confusion matrix](/images/confusion-matrix.png)

По confusion matrix видно, что сеть часто путает художников, чьи произведения относятся к одному стилю.

Для проверки свойств сформированного векторного пространства с помощью CNN были получены вектора для 27 изображений Василия Ложкина, чьи картины сеть прежде "не видела".

17 из них были добавлены в пространство векторов, соответствующих всем изображениям набора данных.
Затем обучен классификатор KNN и протестировано, к какой категории он отнесёт оставшиеся 10 векторов. **Accuracy: 50%**.

Код и результаты его выполнения находится в файле: [siamese-triplet-loss-artists-dataset.ipynb](siamese-triplet-loss-artists-dataset.ipynb)

### Обсуждение результатов и дальнейшие исследования
Достигнутую точность нельзя назвать идеальной.
Однако и Triplet Loss нельзя назвать оптимальной функцией потерь при работе с сиамскими сетями (см, например, статью [4]).

В дальнейших исследованиях можно оценить влияние различных loss-функций на точность классификации: см. [5] и [6].

Кроме того, имеет смысл изучить применение метода, при котором сначала на основе сети обучается классификатор, а затем, сеть преобразуется в сиамскую и дообучается с использованием одной из функций потерь.

# Ссылки

[1] Bromley, Jane, et al. ["Signature Verification using a 'Siamese' Time Delay Neural Network" Advances in neural information processing systems. 1994](http://papers.nips.cc/paper/769-signature-verification-using-a-siamese-time-delay-neural-network.pdf)

[2] Schroff, Florian, Dmitry Kalenichenko, and James Philbin. [Facenet: A unified embedding for face recognition and clustering. CVPR 2015.](https://arxiv.org/pdf/1503.03832.pdf)

[3] Alexander Hermans, Lucas Beyer, Bastian Leibe, 2017. [In Defense of the Triplet Loss for Person Re-Identification](https://arxiv.org/pdf/1703.07737)

[4] Rippel et al. [Metric Learning With Adaptive Density Discrimination](https://arxiv.org/pdf/1511.05939.pdf)

[5] Jiankang Deng et al. [ArcFace: Additive Angular Margin Loss for Deep Face Recognition. 2018](https://arxiv.org/pdf/1801.07698v1.pdf)

[6] Mei Wang, Weihong Deng [Deep Face Recognition: A Survey. 2019](https://arxiv.org/pdf/1804.06655.pdf)
