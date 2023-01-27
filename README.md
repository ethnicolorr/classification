# Лабораторная 3: Классификация изображений на основе свёрточных нейросетей

## Задание

Необходимо реализовать простейшую систему классификации изображений на основе сверточных нейронных сетей. Возможно использовать любые доступные технологии, рекомендованный список такой:
• Google Colab для запуска (можно другую платформу или локальную
машину)
• PyTorch
• Torchvision
Система должна загружать изображение с диска, преобразовывать в нужный для обработки моделью нейронной сети формат (тензор), выполнять предобработку, если требуется (например, изменение размера и нормирование), затем обрабатывать с помощью нейронной сети и выводить результат (номер класса, название, его вероятность для первых 5 наиболее вероятных классов). Необходимо провести исследование по сравнению эффективности трех разных архитектур (можно использовать предобученные модели из
torchvision.models https://pytorch.org/vision/stable/models.html, например, AlexNet, VGG16, ResNet50). Нужно узнать, на каком датасете предобучены данные модели (использовать веса, полученные с одного и того же набора) и найти список классов, которые они предсказывают.
Каждую модель необходимо протестировать на вашем собственном наборе из не менее чем 50 изображений (можно использовать любые изображения, но не те, которые использовались для обучения), в качестве метрик использовать top-1 accuracy и top-5 accuracy. В данной работе не требуется проводить обучение нейронной сети (только
по желанию).

## Теоретическая база

Сверточные нейронные сети (Convolutional Neural Networks, CNN) — это тип нейронных сетей, основанный на идеях свертки и пуллинга. 
Свертка используется для извлечения признаков из изображения с использованием фильтров. Эти фильтры сдвигаются по изображению и производят свертку, что позволяет выделить особые паттерны в изображении. Затем используется пуллинг для уменьшения размерности и предотвращения переобучения. На выходе сверточного слоя получается карта признаков, которая может быть использована для классификации изображения или для дальнейшего анализа. 

Рассмотрим 3 архитектуры CNN: ResNet50, GoogLeNet, Inception V3.

### ResNet50

До ResNet наращивание количества слоев для повышения глубины нейронной сети рано или поздно вызывало ограничение или ухудшение производительности сети. Это происходило из-за проблемы исчезающих градиентов, которая возникает из-за того, что множество функций активации (таких, как сигмоида) сокращают большое входное пространство до гораздо меньшего выходного (например, от 0 до 1 для сигмоиды). Поскольку большие изменения входа преобразуются в намного меньшие изменения выхода, производная становится намного меньшей. Разработчики ResNet решили не складывать слои друг на друга для изучения отображения нужной функции напрямую, а использовать остаточные блоки, которые пытаются «подогнать» это отображение. Таким образом ResNet стала первой остаточной нейронной сетью. Другими словами ResNet "пропускает" некоторые слои. Они больше не содержат признаков и используются для нахождения остаточной функции H(x) = F(x) + x вместо того, чтобы искать H(x) напрямую.
Архитектура модели ResNet50:
![image](https://user-images.githubusercontent.com/91891296/215051986-58055fad-f61c-400d-888e-a279a3b5fa25.png)

### GoogLeNet

GoogleNet — ещё более глубокая архитектура с 22 слоями. Целью Google было разработать нейросеть с наибольшей вычислительной эффективностью. Для этого они придумали так называемый модуль Inception — вся архитектура состоит из множества таких модулей, следующих друг за другом.
Также в GoogleNet нет полносвязных слоёв, и она содержит всего 5 миллионов параметров — в 12 раз меньше, чем у AlexNet.
В составе GoogleNet есть небольшая подсеть — Stem Network. Она состоит из трёх свёрточных слоёв с двумя pooling-слоями и располагается в самом начале архитектуры.
На схеме нейросети можно увидеть небольшие промежуточные «отростки» — это вспомогательные классификационные выходы для введения дополнительного градиента на начальных слоях.
![image](https://user-images.githubusercontent.com/91891296/215068675-ecbed87d-5413-4d50-96be-670deb6a3bf6.png)
Идея основного модуля Inception заключается в том, что он сам по себе является небольшой локальной сетью. Вся его работа состоит в параллельном применении нескольких фильтров на исходное изображение. Данные фильтров объединяются, и создаётся выходной сигнал, который переходит на следующий слой.
Для уменьшения вычислительной сложности введены слои с фильтром 1х1, уменьшающие глубину изображения.
![image](https://user-images.githubusercontent.com/91891296/215069966-0ccacbeb-53cd-4f0c-b4cd-98745601fead.png)

### InceptionV3

Inception V3 — модель, основанная на нормализации откликов всех нейронных карт и поиске лучших комбинаций свойств, выделяемых из входных данных. В основные идеи заложена максимизация потока обрабатываемой информации за счёт аккуратного баланса между глубиной и шириной, а также использовании фильтров 3х3.
Архитектура Inception V3 выглядит следующим образом: 
![image](https://user-images.githubusercontent.com/91891296/215073713-54618d96-f2bb-4f03-85e9-f52b1a65d290.png)

## Описание разработанной системы

Использовались следующие модели предобученных нейронных сетей: ResNet50, GoogLeNet, Inception V3. Для тестирования был создан датасет, содержащий 50 изображений. 
![image](https://user-images.githubusercontent.com/91891296/215075481-91ed0c76-31f2-48c8-a8a4-ada00ef11589.png)
Сравнение работы моделей осуществлялось на основе времени работы, использовании вычислительных ресурсов, метрик top-1 accuracy и top-1 accuracy. 
Код для предсказания классов и оценки модели: 
```
def evaluate(model, weights):
  model = model(weights=weights)
  model.eval()
  preprocess = weights.transforms()
  t_pred = []
  n = 5
  top1_acc = 0
  top5_acc = 0
  start = time.time()
  for i in range(50):
    batch = preprocess(images[i]).unsqueeze(0)
    prediction = model(batch).squeeze().softmax(0)
    predictions = torch.topk(prediction.flatten(), n).indices
    predictionIdx = [i.item() for i in predictions]
    all_score = [prediction[i].item() for i in predictionIdx]
    names = [weights.meta["categories"][i] for i in predictionIdx]
    array_t_pred = [f"{names[i]}: {100 * all_score[i]:.1f}%" for i in range(n)]
    t_pred.append("\n".join(array_t_pred))
    top1_acc += 1 if weights.meta["categories"][predictionIdx[0]] == titles[i] else 0
    top5_acc += 1 if titles[i] in [weights.meta["categories"][i] for i in predictionIdx] else 0
  time_spent = time.time() - start
  top1_acc = top1_acc/len(images)
  top5_acc = top5_acc/len(images)
  return time_spent, memory_usage(), top1_acc, top5_acc, t_pred
```

## Результаты работы и тестирования системы

Пример предсказаний ResNet50:
![image](https://user-images.githubusercontent.com/91891296/215075851-9edcba18-4ddc-4418-abe5-928e05d9a20f.png)

Пример предсказаний GoogLeNet:
![image](https://user-images.githubusercontent.com/91891296/215075942-c9c4ea57-b96c-4d47-8c2b-200d6fdd115a.png)

Пример предсказаний Inception V3: 
![image](https://user-images.githubusercontent.com/91891296/215075999-54d48240-0d67-4ad6-b55e-359c1cdaec59.png)

Сравнение:
| Название     | Время работы, с | Потребление памяти, байт | Top-1 acc, % | Top-5 acc, % |
| ------------ | --------------- | ------------------------ | ------------ | ------------ |
| ResNet50     | 13.2            | 1022.47                  | 20           | 26           |
| GoogLeNet    | 6.93            | 1028.52                  | 20           | 26           |
| Inception V3 | 17.54           | 1103.44                  | 18           | 28           |

## Выводы по работе

В ходе выполнения лабораторной работы были рассмотрены 3 архитектуры CNN: ResNet50, GoogLeNet, Inception V3. По итогам тестирования сложно сказать, какая из моделей отработала лучше: та же Inception V3, хоть и имеет оценку точности Top-1 ниже на 2%, на те же 2% опережает в Top-5; что касается ResNet50 и GoogLeNet, то они и вовсе имеют одинаковые оценки. Однако если брать в учёт остальные параметры, победителем можно считать GoogLeNet - незначительно уступив в потреблении вычислительных ресурсов ResNet50, в скорости выполнения модель обогнала остальных в 2-2.5 раза.

## Использованные источники

1. https://habr.com/ru/post/301084/
2. https://habr.com/ru/post/302242/
3. https://iq.opengenus.org/resnet50-architecture/
