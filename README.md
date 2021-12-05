# Определение зоны механического разрушения ультразвуковым пучком в тканях свиной печени с помощью нейросети Resnet-18

____

## Оглавление

0. [Введение](#Введение)
1. [Разметка данных. Формирование датасета](#Разметкаданных.Формированиедатасета)
2. [Обучение нейросети](#Обучение-нейросети)
3. [Фильтрация с помощью дополнительной нейросети](#Фильтрация-с-помощью-дополнительной-нейросети)
4. [Результаты обработки экспериментальных данных](#Результаты-обработки-экспериментальных-данных)
5. [Итоги](#Итоги)
6. [Ссылки](#Ссылки)

 ____
 
 ## Введение
 
Проект направлен на развитие методов компьютерного зрения для определения области разрушения биологической ткани в результате воздействия фокусированного ультразвукового пучка высокой интенсивности - гистотрипсии, которое приводит к локальному разрушению мягких тканей на субклеточные фрагменты. Высокоинтенсивные, но короткие ударно-волновые ультразвуковые импульсы фокусируются внутрь биоткани, в результате чего в фокусе происходит локальное вскипание ткани, образуется паровая полость миллиметрового размера. Взаимодействие ультразвука с этой полостью приводит к разрушению вокруг нее на субклеточные компоненты размером не более ядра клетки (рис.1) [1,2]. 

Исследование объема и однородности разрушения ткани, наличия уцелевших фрагментов внутри зоны разрушения в зависимости от тепловой дозы, мощности и конфигурации ультразвукового пучка необходимо для развития новых возможностей использования высокоинтенсивного фокусированного ультразвука в клинической и экспериментальной медицине в направлениях неинвазивного разрушения опухолей различных органов либо адресной доставки лекарств. В настоящее время накоплено большое количество экспериментальных результатов – гистологических изображений срезов тканей в области воздействия ультразвукового пучка, однако их количественная оценка посредством разметки вручную осложнена в связи с обширной и неоднородной области разрушения ткани, необходимости высокой детализации клеточных фрагментов и огромного количества экспериментальных изображений.

В связи с этим возникла задача оценить площадь разрушенной ткани и поиск неразрушенных фрагментов внутри зоны воздействия ультразвукового пучка на гистологическом срезе с помощью нейронной сети. Работы по сегментации гистологий проводились и ранее, однако либо использовали методы машинного обучения [3], либо были направлены на детектирование и классификацию опухолей [4-6], и не подразумевали обширный размер и сложную геометрию сегментируемой области. Нами разрабатывается следующий подход: исходное гистологическое изображение большого размера разбивается на небольшие фрагменты приемлемого для анализа разрешения, и затем эти фрагменты классифицируются на разрушенные или здоровые участки ткани с помощью нейронной сети. Предварительный анализ показывает высокую точность (более 97 %) предсказания нейросетью областей разрушений ткани, что делает предложенный подход перспективным инструментом в исследовании воздействия высокоинтенсивного ультразвука на различные типы тканей (печени, сердца, мышечной ткани). Проект доступен на GitHub [7](https://github.com/akrokhmal/histology_segmentation).

![Рис. 1](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig1.png)
*Рис.1. Схема эксперимента по гистотрипсии – разжижения целевого объема в участке ткани [1,2]*
 
 ## Разметка данных. Формирование датасета
 
 В ходе эксперимента методом кипящей гистотрипсии были разрушены небольшие объемы ткани свиной печени при разных режимах облучения. Импульсный пучок характеризуется двумя параметрами – длительностью пучка и количеством импульсов в пучке. Было проведено 10 экспериментов по разрушению целевого объема в ткани свиной печени для четырех различных длительностей импульсного пучка от 1 до 10 мкс и для трех различных количеств импульсов в пучке от 5 до 15, что соответствовало различным дозам облучения. После эксперимента ткань фиксировалась с помощью формалина, затем заливалась парафином, и разрезалась для получения тонких гистологических срезов с шагом 1 мм (рис. 2). Полученные с помощью микроскопа гистологические изображения в дальнейшем использовались как база для датасета. Таким образом, было получено в общей сложности 89 гистологических изображений разрушений в свиной печени с разным соотношением здоровой к пораженной ткани (от 50% площади изображения до 0%). Так как разрешение изображения достаточно велико и должно позволять наблюдать единичные клетки, вес каждого составлял порядка 400 Мб. 
![Рис. 2](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig2.png)

*Рис.2. Пример целевого объемного разрушения*
 
Для того, чтобы обучить и валидировать нейросеть, потребовалась тщательная ручная разметка наиболее характерных изображений. Вручную с помощью сервиса CVAT были размечены изображения, соответствующие самой большой и самой маленькой дозе облучения. Всего вручную было размечено 15 изображений из 89 (изображения гистологий и их ручная сегментация доступны по ссылкам [8](https://drive.google.com/drive/folders/1K7BbjFaPSDk4xhtRUau5RZGhMwslTkOg?usp=sharing), [9](https://drive.google.com/drive/folders/176wQ9hVBx23etjerM36CNIqn7zW_7oYs?usp=sharing). К изображению создавалась маска, в которой красным цветом размечались те зоны, где ткань полностью разрушена – разжижена, зеленым цветом размечались те зоны, где оставались клеточные элементы внутри общей зоны разрушения. Черным цветом был размечен фон и здоровая ткань (рис. 3).
 ![Рис. 3](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig3.png)
*Рис.3. Разметка базы данных*

Из-за значительного объема изображений обрабатывать каждое целиком не представляется возможным. Поэтому было принято решение разбить изображение на небольшие фрагменты и обучать нейросеть на этих фрагментах. В качестве эталонного изображения, на котором обучалась нейросеть, была выбрана гистология с наибольшей площадью разрушенной ткани и минимальным количеством сосудов (PL1A0) [8](https://drive.google.com/drive/folders/1K7BbjFaPSDk4xhtRUau5RZGhMwslTkOg?usp=sharing). Как показали дальнейшие исследования, если создавать датасет на базе нескольких размеченных изображений, то результаты обучения нейросети и дальнейшего распознавания неразмеченных изображений оказывался хуже. Скорее всего, это связано с тем, что на разных изображениях ручная разметка была немного отличающейся – не всегда представляется возможным однозначно сказать, к зеленой, красной или черной зоне отнести конкретный участок гистологии. Неоднозначности вносят и кровеносные сосуды, так как содержат внутри себя жидкость, которая визуально не отличается от разрушенной разжиженной ткани, однако относится к области здоровой ткани. Затем было выбрано разрешение фрагментов, на которые разбивалось исходное изображение, которое удовлетворяет требованиям дальнейшего анализа – было принято решение разбивать гистологию на фрагменты по 200×200 микрон – 80×80 пикселей. Каждому фрагменты присваивалась метка в зависимости от площади разрушенной ткани – «0», если площадь красной области на фрагменте составляет менее 55% от общей площади фрагмента, и «1» - если более (рис.3).

Для более быстрого обучения и сбалансированности классов из общего числа фрагментов было отобрано случайным образом по 5000 фрагментов с метками «0» и «1», которые вошли в итоговый датасет. Для увеличения скорости обучения датасет был отдельно конвертирован в виде тензора в файл формата .npz (библиотеки numpy) [10](https://drive.google.com/file/d/1CX7_9CCkOu7EKSAyPTPB5HDrDxa-BU05/view?usp=sharing).

## Обучение нейросети

Для классификации фрагментов гистологии использовалась сверточная нейросеть ResNet18. Выбор данной нейросети был обусловлен высокой скоростью обучения и приемлемой точностью результатов, а также готовой реализацией в библиотеке PyTorch, с помощью которой был реализован код. Для обучения использовались следующие параметры:
- Размер пакета (batch size) варьировался от 100 до 1000 фрагментов 
- Скорость обучения (learning rate) варьировалась от 0.1 до 0.001
- Использовался оптимизатор Adam
- В качестве функции потерь была выбрана CrossEntropyLoss
- Весь датасет разбивался на тренировочную и валидационную выборку случайным образом в пропорциях 80% : 20% от общего размера датасета
- Функция точности определялась как отношение правильно размеченных элементов датасета к общему числу элементов датасета
- Выполнение кода и обучение нейросети происходила с помощью графического ускорителя на базе сервиса Google Colab, также возможно обучение нейросети на компьютере с оперативной памятью от 32 Гб без графического ускорителя
- Обучение производилось на более чем 80 эпохах

Для оптимизации результатов обучения была найдена зависимость максимальной точности от таких гиперпараметров, как размер пакета и скорость обучения. Оказалось, что наибольшей точности (0.974) результаты предсказаний нейросети достигаются в случае batch size = 100 и learning rate = 0.001 (рис.4). Найденные веса доступны по ссылке [11](https://drive.google.com/file/d/1INQezpgeoHZ1aPJ4WQ1wFkIH34sI7Z_V/view?usp=sharing).

![Рис. 4](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig4.png)

*Рис.4. Функции потерь и точности для разных размеров пакетов данных*
  
## Фильтрация с помощью дополнительной нейросети

Анализ зон, выделяемых нейросетью как разрушение, показал, что есть характерные области, в которых нейросеть может ошибиться – это достаточно крупные кровеносные сосуды с жидким содержимым, неотличимым от разжиженного разрушения, внутри площади которых может поместиться несколько фрагментов 80×80 пикселей. Также возможны случайные точечные ошибки, когда внутри зоны одного класса возникает единичный элемент другого класса. Если второй источник ошибок решается через простую фильтрацию массива и логическое условие на принадлежность к классу исходя из класса окружающих элементов, то различить разрушенную ткань от жидкости внутри сосуда для данной нейросети не представляется возможным.	
Для правильного распознавания зон с сосудами было принято решение сделать наложение предсказаний двух нейросетей, обученных на фрагментах разного масштаба. В дополнение к уже упомянутой, аналогичным образом resnet18 была обучена на фрагментах размера 256×256 пикселей, причем классификация фрагментов была изменена: фрагменту присваивалась метка «0», если доля красной области составляла менее 1%, и «1» - в остальных случаях (рис. 5). Такое строгое условие было необходимо для определения фрагментов, гарантированно не содержащих в себе разрушений. В качестве изображений для обучения использовались гистологии самых больших разрушений для самой большой и самой маленькой тепловой дозы – изображения PL1A0 и PL10A0 [8](https://drive.google.com/drive/folders/1K7BbjFaPSDk4xhtRUau5RZGhMwslTkOg?usp=sharing). Датасет также был конвертирован в формат .npz для более быстрой работы кода [12](https://drive.google.com/file/d/1Xq3ztV7wAXx3DPzJ8sAWPX2aCAMIIbP6/view?usp=sharing ). Также для обучения были использованы следующие параметры:
-	Размер пакета (batch size) составлял 50 фрагментов (для оптимальной скорости обучения на ПК)
-	Скорость обучения (learning rate) была задана как 0.001
-	Использовался оптимизатор Adam
-	Функция потерь CrossEntropyLoss
-	Весь датасет разбивался на тренировочную и валидационную выборку случайным образом в пропорциях 80% : 20% от общего размера датасета
-	Функция точности определялась как отношение правильно размеченных элементов датасета к общему числу элементов датасета
-	Выполнение кода и обучение нейросети происходила с помощью графического ускорителя на базе сервиса Google Colab, также возможно обучение нейросети на компьютере с оперативной памятью от 32 Гб без графического ускорителя
-	Обучение производилось на 100 эпохах
Найденные веса доступны по ссылке [13](https://drive.google.com/file/d/107KdhNkixkQjARVnXvy6aN_PE5erIWhq/view?usp=sharing ). Результат работы такой нейросети показан на рис. 5, где области без разрушения отмечены зеленым цветом. Максимальная точность достигла 94%

![Рис. 5](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig5.png)
*Рис. 5. Слева – пример фрагментов разбиения для дополнительной фильтрующей нейросети, справа – результат обучения такой нейросети*

Наложение предсказаний двух нейросетей (отбрасывание всех красных зон, попавших внутрь зеленых) и фильтрация от единичных ошибочных элементов составила итоговый результат разметки гистологии (рис. 6). Результаты предсказания нейросети на 80-пиксельных фрагментах доступны по ссылке [14](https://drive.google.com/drive/folders/1puh7qqkoDURz8eJX-bKG5fzrblbrfpU4?usp=sharing ), на 256-пиксельных фрагментах – [15](https://drive.google.com/drive/folders/12cPh2CPOSM9Z1R_WKHNfGY0cKMDsOF1H?usp=sharing ), наложение результатов предсказаний двух нейросетей – [16](https://drive.google.com/drive/folders/1MIYUHEpWnIliv6kjigXXY6L5nclYXL0Z?usp=sharing).

![Рис. 6](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig6.png)
*Рис. 6. Пример наложения предсказания двух нейросетей разных масштабов – благодаря такому подходу ошибочно размеченный крупный сосуд (обведен красной линий) оказался исключен из итоговой сегментации разрушения*

## Результаты обработки экспериментальных данных

Результат предсказания нейросетью показан на рис. 7.[16] Как видно по изображению предсказанной маски, довольно точно определяется геометрия и площадь области ткани, подвергшейся воздействию ударного ультразвукового пучка. Основные ошибки связаны с такими большими кровеносными сосудами на изображении, размер которых превышал масштаб фильтрующих фрагментов 256×256 пикселей (таких изображений было всего 3 из 89). Также была проведена оценка площади зон с жизнеспособными тканями внутри области разрушения с помощью blob-анализа. 
![Рис. 7](https://github.com/akrokhmal/histology_segmentation/blob/main/images/fig7.png)
*Рис.7. Сопоставление предсказания нейросети и ручной разметки*

Эти результаты были сравнены с ручной разметкой. Все предсказания попали в доверительный интервал 3% от общей площади изображения.

Также были реконструированы объемные разрушения для всех 10 протоколов облучения. Анализ оставшихся неразмеченными вручную изображений гистологий показал, что в вопросе влияния протокола пульсации на эффективность лечения весь объем поражения сильно зависел от протокола пульсации, тогда как степень разрушения, которая оценивалась по проценту оставшихся фрагментов ткани внутри поражения, не показала значительных различий между тестируемыми режимами. Скорость разжижения, однако, была значительно выше для импульсного протокола с импульсами длительностью 1 мс, подаваемыми 5 раз в каждую точку обработки ультразвуком, по сравнению с другими испытанными режимами.

## Итоги

Произведена качественная и детальная ручная разметка нескольких разнообразных гистологий свиной печени для последующего обучения нейросети. Создан алгоритм на базе нейросети resnet-18, способный с высокой точностью (более 97%) определять зоны, подвергшиеся механическому разрушению в результате воздействия высокоинтенсивного фокусированного ультразвукового пучка, предсказать площадь области разрушения и правильно восстановить геометрию этой области. Алгоритм способен определять области разрушения с высокой детализацией, что делает его эффективным инструментом анализа гистологических изображений. Результаты исследования будут использованы для определения того, как режим облучения влияет на степень и характер механического разрушения тканей. Это исследование позволит определить восприимчивость различных типов тканей к такому воздействию и подобрать оптимальные параметры для неинвазивного разрушения опухолей, а определение связи между характером механического разрушения тканей и параметрами импульсного ультразвукового облучения позволит контролировать безопасность и эффективность методов гистотрипсии. 

## Ссылки

1.	T.D. Khokhlova, et al. JASA, 130(5):3498–3510, 2011
2.	J.C. Simon, et al. Phys Med Biol 57 8061–8078, 2021
3.	Homeyer A., Schenk A., Arlt J., Dahmen U., Dirsch O., Hahn H.K. (2013). Practical quantification of necrosis in histological whole-slide images, Computerized Medical Imaging and Graphics, 37(4), 313-322.
4.	Brady L., Wang Y. N., Rombokas E., Ledoux W. R. (2021). Comparison of texture-based classification and deep learning for plantar soft tissue histology segmentation. Computers in Biology and Medicine, 104491.
5.	Wei J. W., Tafe L. J., Linnik Y. A., Vaickus L. J., Tomita N.,  Hassanpour S. (2019). Pathologist-level classification of histologic patterns on resected lung adenocarcinoma slides with deep neural networks. Scientific reports, 9(1), 1-8.
6.	Iizuka O., Kanavati F., Kato K., Rambeau M., Arihiro K., Tsuneki M. (2020). Deep learning models for histopathological classification of gastric and colonic epithelial tumours. Scientific reports, 10(1), 1-11.
7.	https://github.com/akrokhmal/histology_segmentation 
8.	Гистологические изображения https://drive.google.com/drive/folders/1K7BbjFaPSDk4xhtRUau5RZGhMwslTkOg?usp=sharing
9.	Сегментация гистологических изображений https://drive.google.com/drive/folders/176wQ9hVBx23etjerM36CNIqn7zW_7oYs?usp=sharing 
10.	Датасет из 80-пиксельных фрагментов гистологий https://drive.google.com/file/d/1CX7_9CCkOu7EKSAyPTPB5HDrDxa-BU05/view?usp=sharing 
11.	Веса модели resnet-18 для датасета из 80-пиксельных фрагментов https://drive.google.com/file/d/1INQezpgeoHZ1aPJ4WQ1wFkIH34sI7Z_V/view?usp=sharing 
12.	Датасет из 256-пиксельных фрагментов гистологий https://drive.google.com/file/d/1Xq3ztV7wAXx3DPzJ8sAWPX2aCAMIIbP6/view?usp=sharing 
13.	Веса модели resnet-18 для датасета из 256-пиксельных фрагментов https://drive.google.com/file/d/107KdhNkixkQjARVnXvy6aN_PE5erIWhq/view?usp=sharing 
14.	Предсказания нейросети при разбиении изображения на 80-пиксельные фрагменты https://drive.google.com/drive/folders/1puh7qqkoDURz8eJX-bKG5fzrblbrfpU4?usp=sharing 
15.	Предсказания нейросети при разбиении изображения на 256-пиксельные фрагменты https://drive.google.com/drive/folders/12cPh2CPOSM9Z1R_WKHNfGY0cKMDsOF1H?usp=sharing 
16.	Результат сегментации при комбинации результатов предсказаний двух нейросетей https://drive.google.com/drive/folders/1MIYUHEpWnIliv6kjigXXY6L5nclYXL0Z?usp=sharing 

