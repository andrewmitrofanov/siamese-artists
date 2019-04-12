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

На основе изображений сеть формирует три вектора (обычно их называют embeddings), которые затем поступают в функцию потерь.
