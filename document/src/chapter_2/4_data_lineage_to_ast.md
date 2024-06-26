# 2.4 Линия данных как абстрактная модель статического анализа SQL-запросов

Для формирования линии данных как абстрактной модели представления
статического анализа SQL-запроса необходимо определить условия формирования
линии данных (в каком формате должен быть представлен код запроса), определить
структуру абстрактной модели данных и правила обработки элементов исходного
кода. Сформированная модель должна покрывать все задачи по аналитике запросов, поэтому
набор должен быть полным для выполнения разных задач по аналитике данных.

Входной формат данных (то, как будет представлен исходный код программы) должен представлять
из себя целостную последовательность запросов, необходимую для формирования конечной
таблицы отчётности данных. Таким образом, последовательность выполнения запросов и
зависимости между ними будут определять их порядком в исходном коде программы.
Это может быть как написанный на SQL файл-скрипт с выполняемыми запросами
трансформаций данных, так и хранимая процедура в СУБД. Требование к такому 
формату выходных данных предъявлен для того, чтобы избежать в рамках статического анализа 
отслеживание зависимостей от других вызываемых процедур или других скриптов трансформации
над данными.

Линия формирования данных (Data lineage) представляет собой направленный ациклический
граф, в котором узлы представлены в виде трансформаций над данными, крайние узлы являются
источниками данных, а конечная точка - целевой объект формирования данных. Представление
линии данных в виде направленного графа в абстрактной модели позволит выполнять анализ
всех узлов (трансформаций) при помощи стандартных алгоритмов обхода дерева, например, 
алгоритма поиска в глубину или алгоритма поиска в ширину. Оба алгоритма будут полезны
для анализа линии данных, подскольку предоставляют реализовывать разные подходы к
расчёту показателей эффективности алгоритма: как с акцентом на глубину дерева, 
так и рассчитывать накопительные метрики с учётом всех узлов графа. Таким образом,
линия формирования данных представляется в виде составной модели: множества узлов
(трансформаций над данными) и множества связей между ними, описывающих последовательность
выполнения этих трансформаций относительно друг друга.

Каждый узел представляет собой логически атомарную трансформацию над данными. Она
может включать в себя объединение, соединение таблиц, а также агрегацию и фильтрацию 
над данными. Набор перечисленных элементов в трансформации так или иначе влияет 
на производительность запроса, при этом некоторые трансформации могут иметь накопительный
эффект между операциями, особенно без использования промежуточной материализации
шагов вычисления. Таким образом, модель трансформации данных должна покрывать как
вычисление локальных метрик эффективности запроса, так и иметь характеристики, 
которые могут быть использованы для расчёта накопительных показателей эффективности
запроса. Модель узла трансформации данных должна включать следующие атрибуты:

- наименование узла трансформации;

- флаг материализации трансформации;

- флаг источника данных;

- перечень используемых трансформаций для трансформации в порядке их использования;

- условия соединения таблиц в трансформации;

- перечень результирующих полей трансформации, используемые для их формирования поля из источников (маппинг).

Описанный набор атрибутов абстрактной модели представления обеспечивает
целостно информацию о сформированной при помощи скрипта на языке SQL линии
формирования данных. Данную модель можно интерпретировать в графическое представление,
поскольку все блоки трансформаций определяются однозначно в последовательности их описания
в написанном запросе. Также данная модель позволяет формировать и рассчитывать
разные аналитические показатели эффективности запроса: как применимые к конкретному
узлу трансформации, так и для всей линии формирования данных.

С точки зрения статического анализа кода для задачи формирования линии данных
наиболее оптимальным методом является анализ абстрактного синтаксического дерева,
поскольку представление вложенных конструкций языка в них интерпретируются
в поддерево, которое может быть проанализировано изолировано от остальных 
синтаксических элементов программного кода. Таким образом, при помощи 
абстрактного статического дерева можно определить перечень используемых
в трансформации таблиц, условия их соединения между собой, а также
итоговые поля трансформации, который могут быть как 1 к 1 из одного из 
используемого источника данных, так и быть вычисляемым значением на основе
нескольких полей. Определение анализируемых конструкций SQL-запроса основано на перечне атрибутов
абстрактной модели представления линии формирования данных. Таким образом, 
необходимо определить, какие синтаксические конструкции языка 
структурированных запросов будут отвечать каждому из атрибутов.

Атомарная трансформация над данными должна определяться по созданию временного
объекта в базе данных, при этом это может быть как представление (VIEW) так и 
материализованная таблица (TABLE). Сооветственно, при определении трансфорации
необходимо искать определение данных объеков в написанном скрипте. При создании
объекта трансформации должно быть описано, на основе чего должен создаваться данный
объект, в данном случае основанием создания объекта должен являться SELECT-запрос.
При этом стоит также учитывать, что перед самим SELECT-запросом могут быть использованы
CTE-выражения (Common Table Expressions), которые также из себя представляют 
отдельные шаги трансформаций над данными. На данном этапе для каждой трансформации
можно определить её уникальное название и определить признак метриализации (трансформация
через создание таблицы относится к материализованному шагу).

Перечень используемых трансформаций и источников данных можно определить для каждого
SELECT-выражения в разделе FROM, где как раз и описываются таблицы, которые используются для
формирования данных запроса. При этом используемые трансформации и таблицы могут как 
просто перечисляться через запятую, так и соединяться друг с другом при помощи
конструкций соединения и объединения данных. Обозначения таблиц в запросах могут быть не на
прямую, а через псевдонимы (aliases), что также надо учитывать на этапе статического 
анализа полей запроса.

Перечень полей запроса указывается после SELECT через запятую, при этом перечисляемые значения
могут быть как колонками одной из таблиц-источника, так и рассчитываемыми значениями с использованием
встроенных в РСУБД функций и условных выражений. На данном этапе необходимо для каждого итогового
поля определить перечень используемых в её вычислении полей и определить правило наименования 
поля, если не был явно указан псевдоним.

<!-- TODO: Привести пример перехода от запроса к абстрактной модели -->

Таким образом, было представлено определение структуры абстрактно модели представления
линии данных. Она представляет из себя ориентированных ациклический граф, каждый узел
которого представляет собой атомарную трансформацию над данными. Эта трансформация 
имеет свой атрибутный состав, который позволяет проводить анализ эффективности
линии данных как в разрезе каждой трансформации отдельно, так и в совокупности для
всей графовой модели. Каждый атрибут трансформации имеет свои правила интерпретации 
в исходном коде SQL-запроса.