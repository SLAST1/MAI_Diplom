# Общие сведения

Библиотека предназначена для решения задачи прогнозирования паводков на реках Российской Федерации. Библиотека содержит комплекс математических моделей прогнозирования паводков и затопления территорий с применений технологий математического моделирования, искусственного интеллекта и машинного обучения. Библиотека реализована на языке программирования Python с использованием ряда открытых модулей и подключаемых библиотек на языке программирования С++.

Для запуска библиотеки требуется следующее стороннее программное обеспечение, модули и библиотеки, распространяемые по открытой лицензии (GPL):



* Python v.3.9,
* Blender v.3.0.0,
* GDAL 3.5.1.
* Библиотеки Python в соответствии с указанными версиями, приведенные в requirements.txt.

Полный комплект и версии подключаемого программного обеспечения содержится в файле requirements.txt в корневом каталоге библиотеки.

# Функциональное назначение

Библиотека предназначена для прогнозирования уровней воды и построения зоны затопления для рек РФ в периоды паводковых явлений, вызванных преимущественно атмосферными осадками.

Библиотека включает классы для сбора исходных данных из открытых источников по состоянию на август 2022 года. Указанные классы могут быть заменены коннекторами к данным первоисточников (Росгидромет, Роскосмос) в соответствии с описанием.

Часть для формирования предсказаний по уровням воды реализована в виде модели машинного обучения (архитектура трансформера) и может быть дополнена другими параметрами для выполнения прогноза. Текущая реализация гарантирует точность бинарного прогноза паводкового явления не менее 0.7 для региона «Средний Амур», отличающегося циклонической активностью. Основные параметры точности работы моделей и их определение приведены в программе-методике испытаний.

Для построения зоны затопления используется модель рельефа MERIT DEM (~90 м/пиксель) разрешение которой улучшено в 8 раз по данным спутниковых наблюдений. В последствии возможно использовать более точные модели рельефа и методы для построения зон затопления исходя из предсказанных уровней. Настройка параметров улучшения рельефа исследована для региона «Средний Амур», поэтому для применения на других регионах следует уточнить входные параметры библиотеки и осуществить верификацию зоны затопления по метрикам IOU (пересечение расчетных и реальных площадей затопления).

# Сборка документации

```shell
sphinx-build -b html docs/source docs/build
```

# Подготовка 

# Ubuntu/Debian

Установите библиотеки Python, содержащие Header файлы
```shell
sudo apt-get update
sudo apt-get install python3-dev
```

Установка GDAL

```shell
sudo apt-get install libgdal-dev
export CPLUS_INCLUDE_PATH=/usr/include/gdal
export C_INCLUDE_PATH=/usr/include/gdal
pip install GDAL==${gdal-`gdal-config --version`}
```

# MacOS
Во всей библиотеки используется модуль GDAL 

```shell
brew install gdal
```

# Сборка библиотеки

```bash
python setup.py install
```

# Установка библиотеки

```bash
pip install -e .
```

При установке подтянутся все необходимые библиотеки для корректной работы библиотеки

### Структура библиотеки

```bash
- examples, содержит примеры запуска кодов в формате Jupiter Notebook;
- data, папка по умолчанию для сохранения подготовленных библиотекой данных, чек-поинтов работы
моделей и временных рядов;
- docs, содержит документацию по библиотеке;
- tests, содержит тесты по наиболее важным классам библиотеки;
- flood_lib, каталог для основных кодов библиотеки на Python;
- - predictor, содержит класс модели (обучение на данных и предсказание уровней воды);
- - virtual_hydropost, содержит класс виртуального гидропоста;
- - data_source, содержит классы для получения первичных данных;
- - - satellites, содержит класс парсера данных спутниковых наблюдений;
- - - hydroposts, содержит класс парсера данных уровней воды по гидропостам;
- - - meteostations, содержит класс парсера данных по метеонаблюдениям.
```