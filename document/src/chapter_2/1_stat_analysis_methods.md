# 2.1 Анализ методов статического анализа SQL-запросов

Язык структурированных запросов SQL является одним из самых популярных языков для работы 
с реляционными базами данных. SQL-запросы используются для создания, изменения 
и извлечения данных из базы данных. SQL как язык имеет собственный набор стандартов, 
который описывает структуру запроса и перечень базовых функций, предназначенных для
работы с встроенными типами данных (целочисленные значения, числа с плавающей точкой,
строковые значения, данные формат "дата-время" и другие). Благодаря этим стандартам,
базовые конструкции запроса в разных СУБД имеют схожий синтаксис, что позволяет 
проводить статический анализ запросов на основе базовых правил и шаблонов.

Существует несколько методов статического анализа SQL-запросов, которые позволяют
провести исследование программного кода, среди которых можно выделить следующие:

- анализ кода при помощи регулярных выражений;

- анализ как последовательности разделов програмного кода (токенов);

- анализ кода как абстрактного синтаксического дерева (AST).

Анализ SQL-запросов при помощи регулярных выражений предполагает поиск в 
программном коде подстрок в соответствии с заданным шаблоном (регулярным выражением).
Корректно составленное шаблонное выражение позволяет эффективно и однозначно 
искать нужные элементы кода, необходимые для анализа. Например, при помощи
регулярных выражений можно проводить проверку программного кода на соответствие
правилам наименования объектов (Naming convention) и находить классические
ошибки при написании запросов к базам данных или другим системам, которые используют
язык SQL как инструмент для описания трансформаций над данными. Использование
регулярных выражений для статического анализа SQL-запросов также применяется для
поиска уязвимостей веб-приложений, которые используют РСУБД для работы
с данными. Основная часть уязвимостей, связанных с SQL-запросами, происходит
из возможности написания вредоносного кода на месте заполняемых 
параметров запроса, вводимых пользователем (SQL-инъекции). Таким образом,
при помощи регулярных выражений для SQL-запросов можно задавать шаблоны, которые
будут описывать наиболее уявимые для инъекций элементы кода.

<!-- TODO: Написать про последовательность токенов -->

Статический анализ SQL-запроса также можно свести к анализу абстрактного
синтаксического дерева (Abstract Syntax Tree, AST). Абстрактное синтаксическое
дерево – это древовидное представление исходного кода программы, в котором 
каждый узел представляет конструкцию, встречающуюся в исходном коде. 
AST обычно используется компиляторами и интерпретаторами для анализа 
и трансляции программы в исполняемый код. SQL также можно представить в
виде AST-дерева, где каждый элемент синтаксиса (системные выражения, названия таблиц и 
их псевдонимы, названия функций и аргументы) будет представлена отдельным узлом, при
этом вложенные элементы объекта будут являться непосредственными предками
этого узла в дереве (Ссылка на рисунок).

<!-- ! Рисунок абстрактного синтаксического дерева над SQL -->

Организация статического анализа при помощи абстрактного
синтаксического дерева позволяет проводить анализ программного кода только на
определённой глубине дерева, тем самым анализируя только необходимые элементы запроса.
Также AST позволяет проводить анализ только для конкретных видов узлов и его конкретных предков,
например, это может использоваться для анализа таблиц, которые используются для 
формирования итогового запроса. 