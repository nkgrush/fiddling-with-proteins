# fiddling-with-proteins
Fiddling with natural language processing on protein sequences.

## Графики обучения на WandB
https://wandb.ai/ctcf/fiddling-with-proteins

![image](https://user-images.githubusercontent.com/25302233/179421506-2a9fde17-e3bb-4506-9611-0171db9495f7.png)

## Google colab
https://colab.research.google.com/drive/1tWy1hdjkBp_WK3Tdb0IBD7-L7tZ7oTKS?usp=sharing

# Результаты
## Данные
Для начала я скачал данные с белковыми последовательностями и их расположениями в клетке. Я выбрал бактериальную клетку, так как в ней есть 3 основных расположения: цитоплазма, мембрана, клеточная стенка и периплазм (пространство внутри клеточной стенки). Были выбраны из базы Uniprot, так как это курируемая база данных. Были скачаны только те последовательности белков, которые были отобраны людьми (reviewed), для которых известна их полная последовательность, известна функция. Всего было скачано 150000+ из 200000 известных бактерикальных белков, аннотированных людьми.

Был написан класс Dataset, для загрузки последовательностей. Визуализировано распределение белков по классам и по длине. Для борьбы с дисбалансом по классам использовались веса по классам.  
EmbeddingBagBaseline и LSTMBaseline могли учиться на белках полной длины (до 10000 аминокислот), для DistilBERT пришлось обрезать белок до первых 360 аминокислот, так как иначе модель и батчи не помещалась на видеокарте.

## Сравнение моделей
Были обучены 3 модели разной сложности для классификации белков по их расположению в клетке:

- EmbeddingBagBaseline - линейный классификатор, обученный на усреднении всех эмбеддингов. Эта простая модель использовалась в качестве бейслайна и соответствует классическим подходам, до использования RNN. 
    - Точность на тесте **0.689**. (0.615 - ожидаемая точность при предсказании самого часто встречающегося класса).
    - Для обучения пришлось начать с большого lr и затем уменьшать его.
- LSTMBaseline - bidirectional LSTM, два слоя + с линейный классификатор. Мне были интересны LSTM модели, так как они активно используются в биологических лабораторих. 
    - Точность на тесте **0.805**. Несмотря на большой объём данных, точность была не намного выше чем у EmbeddingBag. 
    - Использование n-граммов с маленькими n давало похожую точность, на больших n точность падала.
- DistilProtBERT, предобученный на последовательностях белков (masking). Transformers являются самыми продвинутыми языковыми моделями. 
    - ProtBERT адаптирован для работы с белковыми последовательностями, но его оказалось невозможно эффективно обучать на колабе из-за большого размера. 
    - DistilProtBERT быстро достиг 95% точности и вышел на плато.
    - Во время обучения использовались веса по классам, AdamW, и warmup lr scheduler для того чтобы не повредить предобученную модель.
    - без `clip_grad_norm_` модель деградировала. Обрезать взрывающиеся градиенты необходимо! 
    - Точность на тесте **0.978**. Я считаю проект законченным, так как с ресурсами колаба точность больше 98% получить затруднительно.

## Вывод
- Для работы с биологическими последовательностями трансформеры являются лучшей архитектурой на данный момент. Они доступны на huggingface и по ним уже написано много статей в биологических журналах, например dnaBERT, rnaBERT итд.
- Для отдельного исследователя сейчас наиболее актуально дообучать большие языковые модели для своих данных или искать закономерности/аномалии в открытых данных, например предсказывать вероятные антибиотики, токсины или функции ферментов.
