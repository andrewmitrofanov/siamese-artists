# Определение художника по изображению картины
(С использованием siamese networks и triplet loss)

## Задача
Научить нейронную сеть определять художника по изображению картины, которую сеть ранее "не видела".

## Используемые технологии
В основе проекта - использование "сиамских" сетей (впервые предложенных в работе Bromley, Jane, et al. "Signature Verification using a 'Siamese' Time Delay Neural Network" Advances in neural information processing systems. 1994. http://papers.nips.cc/paper/769-signature-verification-using-a-siamese-time-delay-neural-network.pdf) и "триплет лосса"
(Schroff, Florian, Dmitry Kalenichenko, and James Philbin. Facenet: A unified embedding for face recognition and clustering. CVPR 2015. https://arxiv.org/pdf/1503.03832.pdf)


Схематично структуру используемой сети можно изобразить так:


![Diagram with siamese network with triplet loss](/images/siamese-triplet-diagram.jpg)


На вход сети подаётся три изображения, два из которых (якорь и положительный пример) принадлежат одному классу, а третье (отрицательный пример) - другому.

На основе изображений сеть формирует три вектора (embeddings), которые затем поступают в функцию потерь - triplet loss.

Основная цель triplet loss - сформировать такое векторное пространство, в котором вектора, полученные на основе изображений одного класса, располагалются близко друг к другу, а на основе на основе разных, - далеко друг от друга.

В хорошо сформированном пространстве вектора разных классов будут образовывать чётко различимые кластеры.

## Обучение сети на простом датасете
Для начала было решено проверить работоспособность выбранной конфигурации сети на простом наборе данных.

В качестве такого набора был выбран **Stanford Dogs Dataset**, содержащий порядка 19 000 изображений 120 пород собак (все они входят в ImageNet).
Все фотографии были тщательно изучены и в итоге оставлены только те, на которых присутствует ровно одно чётко различимое животное.

В качестве свёрточной сети была выбрана сеть с архитектурой senet154, предварительно обученная на ImageNet. Таким образом сеть ранее "видела" весь датасет.

Вся сеть заморожена.
Полносвязные слои сети были заменены на: 2048->128+Tanh(). Они разморожены.
Обучение: 5 эпох.
Достигнутая точность (normalized accuracy): 0.82.

Для проверки свойств сформированного векторного пространства с помощью CNN были получены вектора для 12 изображений "Китайской хохлатой" породы собак, которой нет в ImageNet и которую сеть прежде "не видела".

8 из них были добавлены в пространство векторов, соответствующих всем изображениям набора данных.
Затем обучен классификатор KNN и протестировано, к какой категории он отнесёт оставшиеся 4 вектора. Accuracy: 100%.

Вывод - выбранная конфигурация вполне работоспособна, можно провести её испытания на более сложном для обучения датасете.

## Обучение сети на полноценном датасете
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

