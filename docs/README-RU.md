![IterToolsLogo Logo](https://github.com/markrogoyski/itertools-php/blob/main/docs/image/IterToolsLogo.png?raw=true)

### IterTools PHP — инструментарий для работы с итерируемыми сущностями

Спроектирован для PHP аналогично itertools в Python.

[![Coverage Status](https://coveralls.io/repos/github/markrogoyski/itertools-php/badge.svg?branch=main)](https://coveralls.io/github/markrogoyski/itertools-php?branch=main)
[![License](https://poser.pugx.org/markrogoyski/math-php/license)](https://packagist.org/packages/markrogoyski/itertools-php)

### Функционал

IterTools поддерживает два вида конструкций для итерирования:

* Итерирование в цикле
* Итерирование в потоке цепочечных вызовов

**Пример итерирования в цикле**

```php
foreach (Multi::zip(['a', 'b'], [1, 2]) as [$letter, $number]) {
    print($letter . $number);  // a1, b2
}
```

**Пример итерирования в потоке цепочечных вызовов**

```php
$result = Stream::of([1, 1, 2, 2, 3, 4, 5])
    ->distinct()                 // [1, 2, 3, 4, 5]
    ->map(fn ($x) => $x**2)      // [1, 4, 9, 16, 25]
    ->filter(fn ($x) => $x < 10) // [1, 4, 9]
    ->toSum();                   // 14
```

Методы библиотеки гарантировано работают с любыми `iterable` сущностями:
* `array`
* `Iterator`
* `Generator`
* `Traversable`

Краткий справочник
-----------

### Инструменты для итерирования в циклах

#### Итерирование нескольких коллекций
| Метод                       | Описание                                                                                    | Пример кода                                  |
|-----------------------------|---------------------------------------------------------------------------------------------|----------------------------------------------|
| [`chain`](#Chain)           | Последовательно итерирует коллекции                                                         | `Multi::chain($list1, $list2)`               |
| [`zip`](#Zip)               | Параллельно итерирует коллекции, пока не закончится самый короткий итератор                 | `Multi::zip($list1, $list2)`                 |
| [`zipEqual`](#ZipEqual)     | Параллельно итерирует коллекции одного размера, в случае разных размеров бросает исключение | `Multi::zipEqual($list1, $list2)`            |
| [`zipFilled`](#ZipFilled)   | Параллельно итерирует коллекции с подстановкой филлера для закончившихся итераторов         | `Multi::zipFilled($default, $list1, $list2)` |
| [`zipLongest`](#ZipLongest) | Параллельно итерирует коллекции, пока не закончится самый длинный итератор                  | `Multi::zipLongest($list1, $list2)`          |

#### Итерирование одной коллекции
| Метод                                          | Описание                                                                     | Пример кода                                                 |
|------------------------------------------------|------------------------------------------------------------------------------|-------------------------------------------------------------|
| [`chunkwise`](#Chunkwise)                      | Итерирует коллекцию, разбитую на чанки                                       | `Single::chunkwise($data, $chunkSize)`                      |
| [`chunkwiseOverlap`](#Chunkwise-Overlap)       | Итерирует коллекцию, разбитую на взаимонакладывающиеся чанки                 | `Single::chunkwiseOverlap($data, $chunkSize, $overlapSize)` |
| [`compress`](#Compress)                        | Возвращает элементы из коллекции, выбранные селектором                       | `Single::compress($data, $selectors)`                       |
| [`compressAssociative`](#Compress-Associative) | Возвращает элементы из коллекции по заданным ключам                          | `Single::compressAssociative($data, $selectorKeys)`         |
| [`dropWhile`](#Drop-While)                     | Пропускает элементы, пока предикат возвращает истину                         | `Single::dropWhile($data, $predicate)`                      |
| [`filter`](#Filter)                            | Возвращает только те элементы, для которых предикат возвращает истину        | `Single::filter($data)`                                     |
| [`filterTrue`](#Filter-True)                   | Возвращает только истинные элементы из коллекции                             | `Single::filterTrue($data)`                                 |
| [`filterFalse`](#Filter-False)                 | Возвращает только ложные элементы из коллекции                               | `Single::filterFalse($data, $predicate)`                    |
| [`filterKeys`](#Filter-Keys)                   | Возвращает только те элементы, для ключей которых предикат возвращает истину | `Single::filterKeys($data, $predicate)`                     |
| [`flatMap`](#Flat-Map)                         | Отображение коллекции с уплощением результата на 1 уровень вложенности       | `Single::flaMap($data, $mapper)`                            |
| [`flatten`](#Flatten)                          | Многоуровневое уплощение коллекции                                           | `Single::flatten($data, [$dimensions])`                     |
| [`groupBy`](#Group-By)                         | Группирует элементы коллекции                                                | `Single::groupBy($data, $groupKeyFunction, [$itemKeyFunc])` |
| [`limit`](#Limit)                              | Ограничивает итерирование коллекции заданным максимальным числом итераций    | `Single::limit($data, $limit)`                              |
| [`map`](#Map)                                  | Отображение коллекции с использованием callback-функции                      | `Single::map($data, $function)`                             |
| [`pairwise`](#Pairwise)                        | Итерирует коллекцию попарно (с наложением)                                   | `Single::pairwise($data)`                                   |
| [`reindex`](#Reindex)                          | Переиндексирует key-value коллекцию                                          | `Single::reindex($data, $reindexer)`                        |
| [`repeat`](#Repeat)                            | Повторяет данное значние заданное число раз                                  | `Single::repeat($item, $repetitions)`                       |
| [`reverse`](#Reverse)                          | Итерирует коллекцию в обратном порядке                                       | `Single::reverse($data)`                                    |
| [`skip`](#Skip)                                | Итерирует коллекцию, пропуская некоторое количество элементов подряд         | `Single::skip($data, $count, [$offset])`                    |
| [`slice`](#Slice)                              | Возвращает подвыборку коллекции                                              | `Single::slice($data, [$start], [$count], [$step])`         |
| [`string`](#String)                            | Итерирует строку посимвольно                                                 | `Single::string($string)`                                   |
| [`takeWhile`](#Take-While)                     | Отдает элементы, пока предикат возвращает истину                             | `Single::takeWhile($data, $predicate)`                      |

#### Бесконечное итерирование
| Метод                        | Описание                                             | Пример кода                      |
|------------------------------|------------------------------------------------------|----------------------------------|
| [`count`](#Count)            | Бесконечно перебирает последовательность целых чисел | `Infinite::count($start, $step)` |
| [`cycle`](#Cycle)            | Бесконечно зацикливает перебор коллекции             | `Infinite::cycle($collection)`   |
| [`repeat`](#Repeat-Infinite) | Бесконечно повторяет данное значение                 | `Infinite::repeat($item)`        |

#### Итерирование случайных значений
| Метод                                     | Описание                                 | Пример кода                                |
|-------------------------------------------|------------------------------------------|--------------------------------------------|
| [`choice`](#Choice)                       | Случайные выборы вариантов из списка     | `Random::choice($list, $repetitions)`      |
| [`coinFlip`](#CoinFlip)                   | Случайные броски монеты (0 или 1)        | `Random::coinFlip($repetitions)`           |
| [`number`](#Number)                       | Случайные целые числа                    | `Random::number($min, $max, $repetitions)` |
| [`percentage`](#Percentage)               | Случайные вещественные числа между 0 и 1 | `Random::percentage($repetitions)`         |
| [`rockPaperScissors`](#RockPaperScissors) | Случайный выбор "камень-ножницы-бумага"  | `Random::rockPaperScissors($repetitions)`  |

#### Математическое итерирование
| Метод                                          | Описание                            | Пример кода                                        |
|------------------------------------------------|-------------------------------------|----------------------------------------------------|
| [`frequencies`](#Frequencies)                  | Абсолютное распределение частот     | `Math::frequencies($data, [$strict])`              |
| [`relativeFrequencies`](#Relative-Frequencies) | Относительное распределение частот  | `Math::relativeFrequencies($data, [$strict])`      |
| [`runningAverage`](#Running-Average)           | Накопление среднего арифметического | `Math::runningAverage($numbers, $initialValue)`    |
| [`runningDifference`](#Running-Difference)     | Накопление разности                 | `Math::runningDifference($numbers, $initialValue)` |
| [`runningMax`](#Running-Max)                   | Поиск максимального значения        | `Math::runningMax($numbers, $initialValue)`        |
| [`runningMin`](#Running-Min)                   | Поиск минимального значения         | `Math::runningMin($numbers, $initialValue)`        |
| [`runningProduct`](#Running-Product)           | Накопление произведения             | `Math::runningProduct($numbers, $initialValue)`    |
| [`runningTotal`](#Running-Total)               | Накопление суммы                    | `Math::runningTotal($numbers, $initialValue)`      |

#### Итерирование множеств и мультимножеств
| Метод                                                           | Описание                                                              | Пример кода                                                  |
|-----------------------------------------------------------------|-----------------------------------------------------------------------|--------------------------------------------------------------|
| [`distinct`](#Distinct)                                         | Фильтрует коллекцию, сохраняя только уникальные значения              | `Set::distinct($data)`                                       |
| [`intersection`](#Intersection)                                 | Пересечение нескольких коллекций                                      | `Set::intersection(...$iterables)`                           |
| [`intersectionCoercive`](#Intersection-Coercive)                | Пересечение нескольких коллекций в режиме приведения типов            | `Set::intersectionCoercive(...$iterables)`                   |
| [`partialIntersection`](#Partial-Intersection)                  | Частичное пересечение нескольких коллекций                            | `Set::partialIntersection($minCount, ...$iterables)`         |
| [`partialIntersectionCoercive`](#Partial-Intersection-Coercive) | Частичное пересечение нескольких коллекций в режиме приведения типов  | `Set::partialIntersectionCoercive($minCount, ...$iterables)` |
| [`symmetricDifference`](#Symmetric-Difference)                  | Симметрическая разница нескольких коллекций                           | `Set::symmetricDifference(...$iterables)`                    |
| [`symmetricDifferenceCoercive`](#Symmetric-Difference-Coercive) | Симметрическая разница нескольких коллекций в режиме приведения типов | `Set::symmetricDifferenceCoercive(...$iterables)`            |
| [`union`](#Union)                                               | Объединение нескольких коллекций                                      | `Set::union(...$iterables)`                                  |
| [`unionCoercive`](#Union-Coercive)                              | Объединение нескольких коллекций в режиме приведения типов            | `Set::unionCoercive(...$iterables)`                          |

#### Итерирование с сортировкой
| Iterator          | Description                              | Code Snippet                        |
|-------------------|------------------------------------------|-------------------------------------|
| [`asort`](#ASort) | Сортирует коллекцию с сохранением ключей | `Sort::asort($data, [$comparator])` |
| [`sort`](#Sort)   | Сортирует коллекцию                      | `Sort::sort($data, [$comparator])`  |

#### Итерирование файлов
| Iterator                   | Description                                   | Code Snippet                   |
|----------------------------|-----------------------------------------------|--------------------------------|
| [`readCsv`](#Read-CSV)     | Итерирует коллекции ячеек CSV-файла построчно | `File::readCsv($fileHandle)`   |
| [`readLines`](#Read-Lines) | Итерирует содержимое файла построчно          | `File::readLines($fileHandle)` |

#### Преобразование итерируемых сущностей
| Iterator                                      | Description                                                    | Code Snippet                                                     |
|-----------------------------------------------|----------------------------------------------------------------|------------------------------------------------------------------|
| [`tee`](#Tee)                                 | Создает несколько одинаковых независимых итераторов из данного | `Transform::tee($data, $count)`                                  |
| [`toArray`](#To-Array)                        | Преобразует итерируемую коллекцию в массив                     | `Transform::toArray($data)`                                      |
| [`toAssociativeArray`](#To-Associative-Array) | Преобразует итерируемую коллекцию в ассоциативный массив       | `Transform::toAssociativeArray($data, [$keyFunc], [$valueFunc])` |
| [`toIterator`](#To-Iterator)                  | Преобразует итерируемую коллекцию в итератор                   | `Transform::toIterator($data)`                                   |

#### Саммари о коллекции
| Метод                                                   | Описание                                                                                             | Пример кода                                       |
|---------------------------------------------------------|------------------------------------------------------------------------------------------------------|---------------------------------------------------|
| [`allMatch`](#All-Match)                                | Истинно, если предикат возвращает истину для всех элементов коллекции                                | `Summary::allMatch($data, $predicate)`            |
| [`allUnique`](#All-Unique)                              | Истинно, если все элементы коллекции уникальны                                                       | `Summary::allUnique($data, [$strict])`            |
| [`anyMatch`](#Any-Match)                                | Истинно, если предикат возвращает истину хотя бы для одного элемента коллекции                       | `Summary::anyMatch($data, $predicate)`            |
| [`arePermutations`](#Are-Permutations)                  | Истинно, если коллекции являются перестановками друг друга                                           | `Summary::arePermutations(...$iterables)`         |
| [`arePermutationsCoercive`](#Are-Permutations-Coercive) | Истинно, если коллекции являются перестановками друг друга (в режиме приведения типов)               | `Summary::arePermutationsCoercive(...$iterables)` |
| [`exactlyN`](#Exactly-N)                                | Истинно, если предикат возвращает истину в точности для N элементов                                  | `Summary::exactlyN($data, $n, $predicate)`        |
| [`isPartitioned`](#Is-Partitioned)                      | Истинно, если истинные элементы находятся в коллекции перед ложными (истинность определяет предикат) | `Summary::isPartitioned($data, $predicate)`       |
| [`isEmpty`](#Is-Empty)                                  | Истинно, если коллекция пуста                                                                        | `Summary::isEmpty($data)`                         |
| [`isSorted`](#Is-Sorted)                                | Истинно, если коллекция отсортирована в прямом порядке                                               | `Summary::isSorted($data)`                        |
| [`isReversed`](#Is-Reversed)                            | Истинно, если коллекция отсортирована в обратном порядке                                             | `Summary::isReversed($data)`                      |
| [`noneMatch`](#None-Match)                              | Истинно, если предикат возвращает ложь для всех элементов коллекции                                  | `Summary::noneMatch($data, $predicate)`           |
| [`same`](#Same)                                         | Истинно, если данные коллекции одинаковы                                                             | `Summary::same(...$iterables)`                    |
| [`sameCount`](#Same-Count)                              | Истинно, если данные коллекции имеют одинаковую длину                                                | `Summary::sameCount(...$iterables)`               |

#### Редуцирование
| Метод                                  | Описание                                                                            | Пример кода                                                   |
|----------------------------------------|-------------------------------------------------------------------------------------|---------------------------------------------------------------|
| [`toAverage`](#To-Average)             | Среднее арифметическое элементов коллекции                                          | `Reduce::toAverage($numbers)`                                 |
| [`toCount`](#To-Count)                 | Длина коллекции                                                                     | `Reduce::toCount($data)`                                      |
| [`toFirst`](#To-First)                 | Первый элемент коллекции                                                            | `Reduce::toFirst()`                                           |
| [`toFirstAndLast`](#To-First-And-Last) | Первый и последни элементы коллекции                                                | `Reduce::toFirstAndLast()`                                    |
| [`toLast`](#To-Last)                   | Последний элемент коллекции                                                         | `Reduce::toLast()`                                            |
| [`toMax`](#To-Max)                     | Максимальный элемент коллекции                                                      | `Reduce::toMax($numbers, [$compareBy])`                       |
| [`toMin`](#To-Min)                     | Минимальный элемент коллекции                                                       | `Reduce::toMin($numbers, [$compareBy])`                       |
| [`toMinMax`](#To-Min-Max)              | Минимальный и максимальный элемент коллекции                                        | `Reduce::toMinMax($numbers, [$compareBy])`                    |
| [`toNth`](#To-Nth)                     | N-й элемент коллекции                                                               | `Reduce::toNth($data, $position)`                             |
| [`toProduct`](#To-Product)             | Произведение элементов коллекции                                                    | `Reduce::toProduct($numbers)`                                 |
| [`toRandomValue`](#To-Random-Value)    | Случайный элемент из коллекции                                                      | `Reduce::toRandomValue($data)`                                |
| [`toRange`](#To-Range)                 | Разница между максимальным и минимальным элементами коллекции                       | `Reduce::toRange($numbers)`                                   |
| [`toString`](#To-String)               | Преобразование коллекции в строку                                                   | `Reduce::toString($data, [$separator], [$prefix], [$suffix])` |
| [`toSum`](#To-Sum)                     | Сумма элементов коллекции                                                           | `Reduce::toSum($numbers)`                                     |
| [`toValue`](#To-Value)                 | Редуцирование коллекции до значения, вычисляемого с использованием callback-функции | `Reduce::toValue($data, $reducer, $initialValue)`             |

### Цепочечный вызов итераторов
#### Фабричные методы
| Источник                                         | Описание                                                                                             | Пример кода                                         |
|--------------------------------------------------|------------------------------------------------------------------------------------------------------|-----------------------------------------------------|
| [`of`](#Of)                                      | Создает поток для цепочечных вызовов из данной коллекции                                             | `Stream::of($iterable)`                             |
| [`ofCoinFlips`](#Of-Coin-Flips)                  | Создает поток для цепочечных вызовов из бесконечных случайных бросков монеты                         | `Stream::ofCoinFlips($repetitions)`                 |
| [`ofCsvFile`](#Of-CSV-File)                      | Создает поток для цепочечных вызовов из строк CSV-файла                                              | `Stream::ofCsvFile($fileHandle)`                    |
| [`ofFileLines`](#Of-File-Lines)                  | Создает поток для цепочечных вызовов из строк файла                                                  | `Stream::ofFileLines($fileHandle)`                  |
| [`ofEmpty`](#Of-Empty)                           | Создает поток для цепочечных вызовов из пустой коллекции                                             | `Stream::ofEmpty()`                                 |
| [`ofRandomChoice`](#Of-Random-Choice)            | Создает поток для цепочечных вызовов из бесконечных случайных выборов элемента из списка             | `Stream::ofRandomChoice($items, $repetitions)`      |
| [`ofRandomNumbers`](#Of-Random-Numbers)          | Создает поток для цепочечных вызовов из бесконечного набора случайных целых чисел                    | `Stream::ofRandomNumbers($min, $max, $repetitions)` |
| [`ofRandomPercentage`](#Of-Random-Percentage)    | Создает поток для цепочечных вызовов из бесконечного набора случайных вещественных чисел между 0 и 1 | `Stream::ofRandomPercentage($repetitions)`          |
| [`ofRange`](#Of-Range)                           | Создает поток для цепочечных вызовов из арифметической прогрессии                                    | `Stream::ofRange($start, $end, $step)`              |
| [`ofRockPaperScissors`](#Of-Rock-Paper-Scissors) | Создает поток для цепочечных вызовов из бесконечных случайных выборов "камень-ножницы-бумага"        | `Stream::ofRockPaperScissors($repetitions)`         |

#### Цепочечные операции
| Операция                                                                  | Описание                                                                                                     | Пример кода                                                                       |
|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| [`asort`](#ASort-1)                                                       | Сортирует коллекцию с сохранением ключей                                                                     | `$stream->asort([$comparator])`                                                   |
| [`chainWith`](#Chain-With)                                                | Добавляет в конец итератора другие коллекции для последовательного итерирования                              | `$stream->chainWith(...$iterables)`                                               |
| [`compress`](#Compress-1)                                                 | Отфильтровывает из коллекции элементы, которые не выбраны                                                    | `$stream->compress($selectors)`                                                   |
| [`compressAssociative`](#Compress-Associative-1)                          | Compress source by filtering out keys not selected                                                           | `$stream->compressAssociative($selectorKeys)`                                     |
| [`chunkwise`](#Chunkwise-1)                                               | Итерирует коллекцию с разбиением по чанкам                                                                   | `$stream->chunkwise($chunkSize)`                                                  |
| [`chunkwiseOverlap`](#Chunkwise-Overlap-1)                                | Итерирует коллекцию с разбиением по взаимонакладывающимся чанкам                                             | `$stream->chunkwiseOverlap($chunkSize, $overlap)`                                 |
| [`distinct`](#Distinct-1)                                                 | Фильтрует коллекцию, сохраняя только уникальные значения                                                     | `$stream->distinct($strict)`                                                      |
| [`dropWhile`](#Drop-While-1)                                              | Пропускает элементы из коллекции, пока предикат возвращает ложь                                              | `$stream->dropWhile($predicate)`                                                  |
| [`filter`](#Filter-1)                                                     | Возвращает из коллекции только те элементы, для которых предикат возвращает истину                           | `$stream->filterTrue($predicate)`                                                 |
| [`filterTrue`](#Filter-True-1)                                            | Возвращает только истинные элементы из коллекции                                                             | `$stream->filterTrue($predicate)`                                                 |
| [`filterFalse`](#Filter-False-1)                                          | Возвращает только ложные элементы из коллекции                                                               | `$stream->filterFalse($predicate)`                                                |
| [`filterKeys`](#Filter-Keys-1)                                            | Возвращает только те элементы, для ключей которых предикат возвращает истину                                 | `$stream->filterKeys($predicate)`                                                 |
| [`flatMap`](#Flat-Map-1)                                                  | Отображение коллекции с уплощением результата на 1 уровень вложенности                                       | `$stream->flatMap($function)`                                                     |
| [`flatten`](#Flatten-1)                                                   | Многоуровневое уплощение коллекции                                                                           | `$stream->flatten($dimensions)`                                                   |
| [`frequencies`](#Frequencies-1)                                           | Абсолютная частота вхождений                                                                                 | `$stream->frequencies([$strict])`                                                 |
| [`groupBy`](#Group-By-1)                                                  | Группирует элементы из коллекции по заданному правилу                                                        | `$stream->groupBy($groupKeyFunction)`                                             |
| [`infiniteCycle`](#Infinite-Cycle)                                        | Бесконечно зацикливает перебор коллекции                                                                     | `$stream->infiniteCycle()`                                                        |
| [`intersectionWith`](#Intersection-With)                                  | Возвращает пересечение хранимой коллекции с другими коллекциями                                              | `$stream->intersectionWith(...$iterables)`                                        |
| [`intersection CoerciveWith`](#Intersection-Coercive-With)                | Возвращает пересечение хранимой коллекции с другими коллекциями (в режиме приведения типов)                  | `$stream->intersectionCoerciveWith(...$iterables)`                                |
| [`limit`](#Limit-1)                                                       | Ограничивает итерирование коллекции заданным максимальным числом итераций                                    | `$stream->limit($limit)`                                                          |
| [`map`](#Map-1)                                                           | Отображение коллекции с использованием callback-функции                                                      | `$stream->map($function)`                                                         |
| [`pairwise`](#Pairwise-1)                                                 | Итерирует коллекцию попарно (с наложением)                                                                   | `$stream->pairwise()`                                                             |
| [`partialIntersectionWith`](#Partial-Intersection-With)                   | Возвращает частичное пересечение хранимой коллекции с другими коллекциями                                    | `$stream->partialIntersectionWith( $minIntersectionCount, ...$iterables)`         |
| [`partialIntersection CoerciveWith`](#Partial-Intersection-Coercive-With) | Возвращает частичное пересечение хранимой коллекции с другими коллекциями (в режиме приведения типов)        | `$stream->partialIntersectionCoerciveWith( $minIntersectionCount, ...$iterables)` |
| [`reindex`](#Reindex-1)                                                   | Переиндексирует key-value коллекцию                                                                          | `$stream->reindex($reindexer)`                                                    |
| [`relativeFrequencies`](#Relative-Frequencies-1)                          | Относительная частота вхождений                                                                              | `$stream->relativeFrequencies([$strict])`                                         |
| [`reverse`](#Reverse-1)                                                   | Итерирует коллекцию в обратном порядке                                                                       | `$stream->reverse()`                                                              |
| [`runningAverage`](#Running-Average-1)                                    | Накопление среднего арифметического элементов коллекции                                                      | `$stream->runningAverage($initialValue)`                                          |
| [`runningDifference`](#Running-Difference-1)                              | Накопление разности элементов коллекции                                                                      | `$stream->runningDifference($initialValue)`                                       |
| [`runningMax`](#Running-Max-1)                                            | Поиск максимального значения из коллекции                                                                    | `$stream->runningMax($initialValue)`                                              |
| [`runningMin`](#Running-Min-1)                                            | Поиск минимального значения из коллекции                                                                     | `$stream->runningMin($initialValue)`                                              |
| [`runningProduct`](#Running-Product-1)                                    | Накопление произведения элементов коллекции                                                                  | `$stream->runningProduct($initialValue)`                                          |
| [`runningTotal`](#Running-Total-1)                                        | Накопление суммы элементов коллекции                                                                         | `$stream->runningTotal($initialValue)`                                            |
| [`skip`](#Skip-1)                                                         | Пропускает n элементов коллекции                                                                             | `$stream->skip($count, [$offset])`                                                |
| [`slice`](#Slice-1)                                                       | Возвращает подвыборку коллекции                                                                              | `$stream->slice([start], [$count], [step])`                                       |
| [`sort`](#Sort-1)                                                         | Сортирует хранимую коллекцию                                                                                 | `$stream->sort([$comparator])`                                                    |
| [`symmetricDifferenceWith`](#Symmetric-Difference-With)                   | Возвращает симметрическую разность хранимой коллекции с другими коллекциями                                  | `$this->symmetricDifferenceWith(...$iterables)`                                   |
| [`symmetricDifference CoerciveWith`](#Symmetric-Difference-Coercive-With) | Возвращает симметрическую разность хранимой коллекции с другими коллекциями (в режиме приведения типов)      | `$this->symmetricDifferenceCoerciveWith( ...$iterables)`                          |
| [`takeWhile`](#Take-While-1)                                              | Отдает элементы из коллекции, пока предикат возвращает истину                                                | `$stream->takeWhile($predicate)`                                                  |
| [`unionWith`](#Union-With)                                                | Возвращает объединение хранимой коллекции с другими коллекциями                                              | `$stream->unionWith(...$iterables)`                                               |
| [`unionCoerciveWith`](#Union-Coercive-With)                               | Возвращает объединение хранимой коллекции с другими коллекциями (в режиме приведения типов)                  | `$stream->unionCoerciveWith(...$iterables)`                                       |
| [`zipWith`](#Zip-With)                                                    | Параллельно итерирует коллекцию вместе с другими, пока не закончится самый короткий итератор                 | `$stream->zipWith(...$iterables)`                                                 |
| [`zipEqualWith`](#Zip-Equal-With)                                         | Параллельно итерирует коллекцию вместе с другими одного размера, в случае разных размеров бросает исключение | `$stream->zipEqualWith(...$iterables)`                                            |
| [`zipFilledWith`](#Zip-Filled-With)                                       | Параллельно итерирует коллекцию вместе с другими, подставляя для закончившихся заданный филлер               | `$stream->zipFilledWith($default, ...$iterables)`                                 |
| [`zipLongestWith`](#Zip-Longest-With)                                     | Параллельно итерирует коллекцию вместе с другими, пока не закончится самый длинный итератор                  | `$stream->zipLongestWith(...$iterables)`                                          |

#### Завершающие операции
##### Саммари о коллекции
| Операция                                                         | Описание                                                                                             | Пример кода                                           |
|------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|-------------------------------------------------------|
| [`allMatch`](#All-Match-1)                                       | Истинно, если предикат возвращает истину для всех элементов коллекции                                | `$stream->allMatch($predicate)`                       |
| [`allUnique`](#All-Unique-1)                                     | Истинно, если все элементы коллеции уникальны                                                        | `$stream->allUnique([$strict]])`                      |
| [`anyMatch`](#Any-Match-1)                                       | Истинно, если предикат возвращает истину хотя бы для одного элемента коллекции                       | `$stream->anyMatch($predicate)`                       |
| [`arePermutationsWith`](#Are-Permutations-With)                  | Истинно, если коллекции являются перестановками друг друга                                           | `$stream->arePermutationsWith(...$iterables)`         |
| [`arePermutationsCoerciveWith`](#Are-Permutations-Coercive-With) | Истинно, если коллекции являются перестановками друг друга (в режиме приведения типов)               | `$stream->arePermutationsCoerciveWith(...$iterables)` |
| [`exactlyN`](#Exactly-N-1)                                       | Истинно, если предикат возвращает истину в точности для N элементов                                  | `$stream->exactlyN($n, $predicate)`                   |
| [`isEmpty`](#Is-Empty-1)                                         | Истинно, если коллекция пуста                                                                        | `$stream::isEmpty()`                                  |
| [`isPartitioned`](#Is-Partitioned-1)                             | Истинно, если истинные элементы находятся в коллекции перед ложными (истинность определяет предикат) | `$stream::isPartitioned($predicate)`                  |
| [`isSorted`](#Is-Sorted-1)                                       | Истинно, если коллекция отсортирована в прямом порядке                                               | `$stream->isSorted()`                                 |
| [`isReversed`](#Is-Reversed-1)                                   | Истинно, если коллекция отсортирована в обратном порядке                                             | `$stream->isReversed()`                               |
| [`noneMatch`](#None-Match-1)                                     | Истинно, если предикат возвращает ложь для всех элементов коллекции                                  | `$stream->noneMatch($predicate)`                      |
| [`sameWith`](#Same-With)                                         | Истинно, если данные коллекции одинаковы                                                             | `$stream->sameWith(...$iterables)`                    |
| [`sameCountWith`](#Same-Count-With)                              | Истинно, если данные коллекции имеют одинаковую длину                                                | `$stream->sameCountWith(...$iterables)`               |

##### Редуцирование
| Terminal Operation                       | Description                                                         | Code Snippet                                            |
|------------------------------------------|---------------------------------------------------------------------|---------------------------------------------------------|
| [`toAverage`](#To-Average-1)             | Среднее арфиметическое элементов коллекции                          | `$stream->toAverage()`                                  |
| [`toCount`](#To-Count-1)                 | Длина коллекции                                                     | `$stream->toCount()`                                    |
| [`toFirst`](#To-First-1)                 | Первый элемент коллекции                                            | `$stream->toFirst()`                                    |
| [`toFirstAndLast`](#To-First-And-Last-1) | Первый и последни элементы коллекции                                | `$stream->toFirstAndLast()`                             |
| [`toLast`](#To-Last-1)                   | Последний элемент коллекции                                         | `$stream->toLast()`                                     |
| [`toMax`](#To-Max-1)                     | Максимальное значение из элементов коллекции                        | `$stream->toMax([$compareBy])`                          |
| [`toMin`](#To-Min-1)                     | Минимальное значение из элементов коллекции                         | `$stream->toMin([$compareBy])`                          |
| [`toMinMax`](#To-Min-Max-1)              | Минимальное и максимальное значения из элементов коллекции          | `$stream->toMinMax([$compareBy])`                       |
| [`toNth`](#To-Nth-1)                     | N-й элемент коллекции                                               | `$stream->toNth($position)`                             |
| [`toProduct`](#To-Product-1)             | Произведение элементов коллекции                                    | `$stream->toProduct()`                                  |
| [`toString`](#To-String-1)               | Преобразование коллекции в строку                                   | `$stream->toString([$separator], [$prefix], [$suffix])` |
| [`toSum`](#To-Sum-1)                     | Сумма элементов коллекции                                           | `$stream->toSum()`                                      |
| [`toRandomValue`](#To-Random-Value-1)    | Случайный элемент из коллекции                                      | `$stream->toRandomValue()`                              |
| [`toRange`](#To-Range-1)                 | Разница между максимальным и минимальным элементами коллекции       | `$stream->toRange()`                                    |
| [`toValue`](#To-Value-1)                 | Редуцирование коллекции до значения, вычисляемого callback-функцией | `$stream->toValue($reducer, $initialValue)`             |

##### Операции трансформации
| Terminal Operation                              | Description                                                 | Code Snippet                                            |
|-------------------------------------------------|-------------------------------------------------------------|---------------------------------------------------------|
| [`toArray`](#To-Array)                          | Возвращает массив из элементов потока                       | `$stream->toArray()`                                    |
| [`toAssociativeArray`](#To-Associative-Array-1) | Возвращает ассоциативный массив из элементов потока         | `$stream->toAssociativeArray($keyFunc, $valueFunc)`     |
| [`tee`](#Tee-1)                                 | Создает несколько одинаковых независимых потоков из данного | `$stream->tee($count)`                                  |

##### Операции с побочными эффектами
| Terminal Operation              | Description                                           | Code Snippet                                         |
|---------------------------------|-------------------------------------------------------|------------------------------------------------------|
| [`callForEach`](#Call-For-Each) | Вызывает callback-функцию для каждого элемента потока | `$stream->callForEach($function)`                    |
| [`print`](#Print)               | `print` каждого элемента потока                       | `$stream->print([$separator], [$prefix], [$suffix])` |
| [`printLn`](#Print-Line)        | `print` каждого элемента потока с новой строки        | `$stream->printLn()`                                 |
| [`toCsvFile`](#To-CSV-File)     | Записывает содержимое потока в CSV файл               | `$stream->toCsvFile($fileHandle, [$headers])`        |
| [`toFile`](#To-File)            | Записывает содержимое потока в файл                   | `$stream->toFile($fileHandle)`                       |

#### Операции для дебаггинга
| Debug Operation              | Description                                                       | Code Snippet                     |
|------------------------------|-------------------------------------------------------------------|----------------------------------|
| [`peek`](#Peek)              | Просмотр каждого элемента коллекции между потоковыми операциями   | `$stream->peek($peekFunc)`       |
| [`peekStream`](#Peek-Stream) | Просмотр всей коллекции потока между операциями                   | `$stream->peekStream($peekFunc)` |
| [`peekPrint`](#Peek-Print)   | Печать в поток вывода каждого элемента коллекции между операциями | `$stream->peekPrint()`           |
| [`peekPrintR`](#Peek-PrintR) | Вызов `print_r` для каждого элемента коллекции между операциями   | `$stream->peekPrintR()`          |
| [`printR`](#Print-R)         | `print_r` каждого элемента потока                                 | `$stream->printR()`              |
| [`varDump`](#Var-Dump)       | `var_dump` каждого элемента потока                                | `$stream->varDump()`             |

Установка
-----

Добавьте библиотеку в зависимости в файле `composer.json` вашего проекта:

```json
{
  "require": {
      "markrogoyski/itertools-php": "1.*"
  }
}
```

Используйте [composer](http://getcomposer.org), чтобы установить библиотеку:

```bash
$ php composer.phar install
```

Composer установит IterTools в папку vendor вашего проекта. После этого вы можете использовать функционал библиотеки
в файлах своего проекта, используя Composer Autoloader.

```php
require_once __DIR__ . '/vendor/autoload.php';
```

Альтернативный вариант установки. Выполните данную команду в терминале, находясь в корневой директории вашего проекта:

```
$ php composer.phar require markrogoyski/itertools-php:1.*
```

#### Минимальные требования
* PHP 7.4

Использование
-----
Все функции работают с `iterable` сущностями:
* `array` (тип)
* `Generator` (тип)
* `Iterator` (интерфейс)
* `Traversable` (интерфейс)

## Итерирование нескольких коллекций
### Chain
Последовательно итерирует коллекции поэлементно.

```Multi::chain(iterable ...$iterables)```
```php
use IterTools\Multi;

$prequels  = ['Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith'];
$originals = ['A New Hope', 'Empire Strikes Back', 'Return of the Jedi'];

foreach (Multi::chain($prequels, $originals) as $movie) {
    print($movie);
}
// 'Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith', 'A New Hope', 'Empire Strikes Back', 'Return of the Jedi'
```

### Zip
Параллельно итерирует коллекции, пока не закончится самый короткий итератор.

```Multi::zip(iterable ...$iterables)```
```php
use IterTools\Multi;

$languages = ['PHP', 'Python', 'Java', 'Go'];
$mascots   = ['elephant', 'snake', 'bean', 'gopher'];

foreach (Multi::zip($languages, $mascots) as [$language, $mascot]) {
    print("The {$language} language mascot is an {$mascot}.");
}
// The PHP language mascot is an elephant.
// ...
```

Может принимать больше двух коллекций на вход.
```php
$names          = ['Ryu', 'Ken', 'Chun Li', 'Guile'];
$countries      = ['Japan', 'USA', 'China', 'USA'];
$signatureMoves = ['hadouken', 'shoryuken', 'spinning bird kick', 'sonic boom'];

foreach (Multi::zip($names, $countries, $signatureMoves) as [$name, $country, $signatureMove]) {
    $streetFighter = new StreetFighter($name, $country, $signatureMove);
}
```
Примечание: для коллекций разных длин итерирование прекратиться, когда закончится самая короткая коллекция.

### ZipFilled
Параллельно итерирует коллекции, пока не закончится самый длинный итератор.
Для законичвшихся коллекций в качестве значения подставляет заданный филлер.

```Multi::zipFilled(mixed $filler, iterable ...$iterables)```

```php
use IterTools\Multi;

$default = '?';
$letters = ['A', 'B'];
$numbers = [1, 2, 3];

foreach (Multi::zipFilled($default, $letters, $numbers) as [$letter, $number]) {
    // ['A', 1], ['B', 2], ['?', 3]
}
```

### ZipLongest
Параллельно итерирует коллекции, пока не закончится самый длинный итератор.

```Multi::zipLongest(iterable ...$iterables)```

Примечание: в случае итерирования коллекций разных длин для закончившихся коллекций в каждой следующей итерации будет подставляться значение `null`.

```php
use IterTools\Multi;

$letters = ['A', 'B', 'C'];
$numbers = [1, 2];

foreach (Multi::zipLongest($letters, $numbers) as [$letter, $number]) {
    // ['A', 1], ['B', 2], ['C', null]
}
```

### ZipEqual
Параллельно итерирует коллекции одного размера, в случае разных размеров бросает исключение.

Бросает `\LengthException`, если длины коллекций окажутся неравны, как только закончится самая короткая.

```Multi::zipEqual(iterable ...$iterables)```

```php
use IterTools\Multi;

$letters = ['A', 'B', 'C'];
$numbers = [1, 2, 3];

foreach (Multi::zipEqual($letters, $numbers) as [$letter, $number]) {
    // ['A', 1], ['B', 2], ['C', 3]
}
```

## Итерирование одной коллекции
### Chunkwise
Итерирует коллекцию, разбитую на чанки одинаковой длины.

```Single::chunkwise(iterable $data, int $chunkSize)```

Минимальный размер чанка — 1.

```php
use IterTools\Single;

$movies = [
    'Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith',
    'A New Hope', 'Empire Strikes Back', 'Return of the Jedi',
    'The Force Awakens', 'The Last Jedi', 'The Rise of Skywalker'
];

foreach (Single::chunkwise($movies, 3) as $trilogy) {
    $trilogies[] = $trilogy;
}
// [
//     ['Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith'],
//     ['A New Hope', 'Empire Strikes Back', 'Return of the Jedi'],
//     ['The Force Awakens', 'The Last Jedi', 'The Rise of Skywalker]'
// ]
```

### Chunkwise Overlap
Итерирует коллекцию, разбитую на взаимонакладывающиеся чанки.

```Single::chunkwiseOverlap(iterable $data, int $chunkSize, int $overlapSize, bool $includeIncompleteTail = true)```

* Минимальный размер чанка — 1.
* Размер наложения должен быть меньше длины чанка.

```php
use IterTools\Single;

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

foreach (Single::chunkwiseOverlap($numbers, 3, 1) as $chunk) {
    // [1, 2, 3], [3, 4, 5], [5, 6, 7], [7, 8, 9], [9, 10]
}
```

### Compress
Отфильтровывает невыбранные элементы из коллекции.

```Single::compress(string $data, $selectors)```

```php
use IterTools\Single;

$movies = [
    'Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith',
    'A New Hope', 'Empire Strikes Back', 'Return of the Jedi',
    'The Force Awakens', 'The Last Jedi', 'The Rise of Skywalker'
];
$goodMovies = [0, 0, 0, 1, 1, 1, 1, 0, 0];

foreach (Single::compress($movies, $goodMovies) as $goodMovie) {
    print($goodMovie);
}
// 'A New Hope', 'Empire Strikes Back', 'Return of the Jedi', 'The Force Awakens'
```

### Compress Associative
Возвращает элементы из коллекции по заданным ключам.

```Single::compressAssociative(string $data, array $selectorKeys)```

* Ключами могут быть только строки или целые числа (по аналогии с ключами PHP-массивов).

```php
use IterTools\Single;
$starWarsEpisodes = [
    'I'    => 'The Phantom Menace',
    'II'   => 'Attack of the Clones',
    'III'  => 'Revenge of the Sith',
    'IV'   => 'A New Hope',
    'V'    => 'The Empire Strikes Back',
    'VI'   => 'Return of the Jedi',
    'VII'  => 'The Force Awakens',
    'VIII' => 'The Last Jedi',
    'IX'   => 'The Rise of Skywalker',
];
$originalTrilogyNumbers = ['IV', 'V', 'VI'];
foreach (Single::compressAssociative($starWarsEpisodes, $originalTrilogyNumbers) as $episode => $title) {
    print("$episode: $title" . \PHP_EOL);
}
// IV: A New Hope
// V: The Empire Strikes Back
// VI: Return of the Jedi
```

### Drop While
Пропускает элементы, пока предикат возвращает истину.

После того как предикат впервые вернул `true`, все последующие элементы попадают в выборку.

```Single::dropWhile(iterable $data, callable $predicate)```

```php
use IterTools\Single;

$scores    = [50, 60, 70, 85, 65, 90];
$predicate = fn ($x) => $x < 70;

foreach (Single::dropWhile($scores, $predicate) as $score) {
    print($score);
}
// 70, 85, 65, 90
```

### Filter
Возвращает только те элементы, для которых предикат возвращает истину.

```Single::filter(iterable $data, callable $predicate)```

```php
use IterTools\Single;

$starWarsEpisodes   = [1, 2, 3, 4, 5, 6, 7, 8, 9];
$goodMoviePredicate = fn ($episode) => $episode > 3 && $episode < 8;

foreach (Single::filter($starWarsEpisodes, $goodMoviePredicate) as $goodMovie) {
    print($goodMovie);
}
// 4, 5, 6, 7
```

### Filter True
Возвращает только истинные элементы из коллекции. Истинность определяется предикатом.

Если предикат не передан, значения элементов коллекции приводятся к `bool` для оценки.

```Single::filterFalse(iterable $data, callable $predicate = null)```

```php
use IterTools\Single;

$reportCardGrades = [100, 0, 95, 85, 0, 94, 0];
foreach (Single::filterTrue($reportCardGrades) as $goodGrade) {
    print($goodGrade);
}
// 100, 95, 85, 94
```

### Filter False
Возвращает только ложные элементы из коллекции. Истинность определяется предикатом.

Если предикат не передан, значения элементов коллекции приводятся к `bool` для оценки.

```Single::filterFalse(iterable $data, callable $predicate = null)```

```php
use IterTools\Single;

$alerts = [0, 1, 1, 0, 1, 0, 0, 1, 1];
foreach (Single::filterFalse($alerts) as $noAlert) {
    print($noAlert);
}
// 0, 0, 0, 0
```

### Filter Keys
Возвращает только те элементы, для ключей которых предикат возвращает истину.

```Single::filterKeys(iterable $data, callable $predicate)```
```php
use IterTools\Single;

$olympics = [
    2000 => 'Sydney',
    2002 => 'Salt Lake City',
    2004 => 'Athens',
    2006 => 'Turin',
    2008 => 'Beijing',
    2010 => 'Vancouver',
    2012 => 'London',
    2014 => 'Sochi',
    2016 => 'Rio de Janeiro',
    2018 => 'Pyeongchang',
    2020 => 'Tokyo',
    2022 => 'Beijing',
];
$summerFilter = fn ($year) => $year % 4 === 0;
foreach (Single::filterKeys($olympics, $summerFilter) as $year => $hostCity) {
    print("$year: $hostCity" . \PHP_EOL);
}
// 2000: Sydney
// 2004: Athens
// 2008: Beijing
// 2012: London
// 2016: Rio de Janeiro
// 2020: Tokyo
```

### Flat Map
Отображение коллекции с уплощением результата на 1 уровень вложенности.

```Single::flatMap(iterable $data, callable $mapper)```

```php
use IterTools\Single;
$data   = [1, 2, 3, 4, 5];
$mapper = fn ($item) => [$item, -$item];
foreach (Single::flatMap($data, $mapper) as $number) {
    print($number . ' ');
}
// 1 -1 2 -2 3 -3 4 -4 5 -5
```

### Flatten
Многоуровневое уплощение коллекции.

```Single::flatten(iterable $data, int $dimensions = 1)```

```php
use IterTools\Single;
$multidimensional = [1, [2, 3], [4, 5]];
$flattened = [];
foreach (Single::flatten($multidimensional) as $number) {
    $flattened[] = $number;
}
// [1, 2, 3, 4, 5]
```

### Group By
Группирует элементы коллекции по заданному правилу.

```Single::groupBy(iterable $data, callable $groupKeyFunction, callable $itemKeyFunction = null)```

* Функция `$groupKeyFunction` должна возвращать общий ключ (или коллекцию ключей) для элементов группы.
* Функция `$itemKeyFunction` (опциональный аргумент) позволяет назначить кастомные индексы эелементам в группе.

```php
use IterTools\Single;

$cartoonCharacters = [
    ['Garfield', 'cat'],
    ['Tom', 'cat'],
    ['Felix', 'cat'],
    ['Heathcliff', 'cat'],
    ['Snoopy', 'dog'],
    ['Scooby-Doo', 'dog'],
    ['Odie', 'dog'],
    ['Donald', 'duck'],
    ['Daffy', 'duck'],
];

$charactersGroupedByAnimal = [];
foreach (Single::groupBy($cartoonCharacters, fn ($x) => $x[1]) as $animal => $characters) {
    $charactersGroupedByAnimal[$animal] = $characters;
}
/*
'cat' => [
    ['Garfield', 'cat'],
    ['Tom', 'cat'],
    ['Felix', 'cat'],
    ['Heathcliff', 'cat'],
],
'dog' => [
    ['Snoopy', 'dog'],
    ['Scooby-Doo', 'dog'],
    ['Odie', 'dog'],
],
'duck' => [
    ['Donald', 'duck'],
    ['Daffy', 'duck'],
*/
```

### Limit
Ограничивает итерирование коллекции заданным максимальным числом итераций.

Останавливает процесс итерирования, когда число итераций достигает `$limit`.

```Single::limit(iterable $data, int $limit)```

```php
use IterTools\Single;

$matrixMovies = ['The Matrix', 'The Matrix Reloaded', 'The Matrix Revolutions', 'The Matrix Resurrections'];
$limit        = 1;

foreach (Single::limit($matrixMovies, $limit) as $goodMovie) {
    print($goodMovie);
}
// 'The Matrix' (and nothing else)
```

### Map
Отображение коллекции с использованием callback-функции.

Результат выполнения представляет собой коллекцию результатов вызова callback-функции для каждого элемента.

```Single::map(iterable $data, callable $function)```

```php
use IterTools\Single;

$grades               = [100, 99, 95, 98, 100];
$strictParentsOpinion = fn ($g) => $g === 100 ? 'A' : 'F';

foreach (Single::map($grades, $strictParentsOpinion) as $actualGrade) {
    print($actualGrade);
}
// A, F, F, F, A
```

### Pairwise
Итерирует коллекцию попарно (с наложением).

Возвращает пустой генератор, если коллекция содержит меньше 2-х элементов.

```Single::pairwise(iterable $data)```

```php
use IterTools\Single;

$friends = ['Ross', 'Rachel', 'Chandler', 'Monica', 'Joey', 'Phoebe'];

foreach (Single::pairwise($friends) as [$leftFriend, $rightFriend]) {
    print("{$leftFriend} and {$rightFriend}");
}
// Ross and Rachel, Rachel and Chandler, Chandler and Monica, ...
```

### Repeat
Повторяет данное значение заданное число раз.

```Single::repeat(mixed $item, int $repetitions)```

```php
use IterTools\Single;

$data        = 'Beetlejuice';
$repetitions = 3;

foreach (Single::repeat($data, $repetitions) as $repeated) {
    print($repeated);
}
// 'Beetlejuice', 'Beetlejuice', 'Beetlejuice'
```

### Reindex
Переиндексирует key-value коллекцию, используя функцию-индексатор.

```Single::reindex(string $data, callable $indexer)```

```php
use IterTools\Single;
$data = [
    [
        'title'   => 'Star Wars: Episode IV – A New Hope',
        'episode' => 'IV',
        'year'    => 1977,
    ],
    [
        'title'   => 'Star Wars: Episode V – The Empire Strikes Back',
        'episode' => 'V',
        'year'    => 1980,
    ],
    [
        'title' => 'Star Wars: Episode VI – Return of the Jedi',
        'episode' => 'VI',
        'year' => 1983,
    ],
];
$reindexFunc = fn (array $swFilm) => $swFilm['episode'];
$reindexedData = [];
foreach (Single::reindex($data, $reindexFunc) as $key => $filmData) {
    $reindexedData[$key] = $filmData;
}
// [
//     'IV' => [
//         'title'   => 'Star Wars: Episode IV – A New Hope',
//         'episode' => 'IV',
//         'year'    => 1977,
//     ],
//     'V' => [
//         'title'   => 'Star Wars: Episode V – The Empire Strikes Back',
//         'episode' => 'V',
//         'year'    => 1980,
//     ],
//     'VI' => [
//         'title' => 'Star Wars: Episode VI – Return of the Jedi',
//         'episode' => 'VI',
//         'year' => 1983,
//     ],
// ]
```

### Reverse
Итерирует коллекцию в обратном порядке.

```Single::reverse(iterable $data)```

```php
use IterTools\Single;
$words = ['Alice', 'answers', 'your', 'questions', 'Bob'];
foreach (Single::reverse($words) as $word) {
    print($word . ' ');
}
// Bob questions your answers Alice
```

### Skip
Пропускает n элементов коллекции со смещением (опционально).

```Single::skip(iterable $data, int $count, int $offset = 0)```

```php
use IterTools\Single;

$movies = [
    'The Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith',
    'A New Hope', 'The Empire Strikes Back', 'Return of the Jedi',
    'The Force Awakens', 'The Last Jedi', 'The Rise of Skywalker'
];

$prequelsRemoved = [];
foreach (Single::skip($movies, 3) as $nonPrequel) {
    $prequelsRemoved[] = $nonPrequel;
} // Episodes IV - IX

$onlyTheBest = [];
foreach (Single::skip($prequelsRemoved, 3, 3) as $nonSequel) {
    $onlyTheBest[] = $nonSequel;
}
// 'A New Hope', 'The Empire Strikes Back', 'Return of the Jedi'
```

### Slice
Возвращает подвыборку коллекции.

```Single::slice(iterable $data, int $start = 0, int $count = null, int $step = 1)```

```php
use IterTools\Single;
$olympics = [1992, 1994, 1996, 1998, 2000, 2002, 2004, 2006, 2008, 2010, 2012, 2014, 2016, 2018, 2020, 2022];
$winterOlympics = [];
foreach (Single::slice($olympics, 1, 8, 2) as $winterYear) {
    $winterOlympics[] = $winterYear;
}
// [1994, 1998, 2002, 2006, 2010, 2014, 2018, 2022]
```

### String
Итерирует строку посимвольно.

```Single::string(string $string)```

```php
use IterTools\Single;

$string = 'MickeyMouse';

$listOfCharacters = [];
foreach (Single::string($string) as $character) {
    $listOfCharacters[] = $character;
}
// ['M', 'i', 'c', 'k', 'e', 'y', 'M', 'o', 'u', 's', 'e']
```

### Take While
Отдает элементы, пока предикат возвращает истину.

Останавливает процесс итерирования, как только предикат впервые вернет ложь.

```Single::takeWhile(iterable $data, callable $predicate)```
```php
use IterTools\Single;

$prices = [0, 0, 5, 10, 0, 0, 9];
$isFree = fn ($price) => $price == 0;

foreach (Single::takeWhile($prices, $isFree) as $freePrice) {
    print($freePrice);
}
// 0, 0
```

## Бесконечное итерирование
### Count
Бесконечно перебирает последовательность целых чисел.

```Infinite::count(int $start = 1, int $step = 1)```

```php
use IterTools\Infinite;

$start = 1;
$step  = 1;

foreach (Infinite::count($start, $step) as $i) {
    print($i);
}
// 1, 2, 3, 4, 5 ...
```

### Cycle
Бесконечно зацикливает перебор коллекции.

```Infinite::cycle(iterable $iterable)```

```php
use IterTools\Infinite;

$hands = ['rock', 'paper', 'scissors'];

foreach (Infinite::cycle($hands) as $hand) {
    RockPaperScissors::playHand($hand);
}
// rock, paper, scissors, rock, paper, scissors, ...
```

### Repeat (Infinite)
Бесконечно повторяет данное значение.

```Infinite::repeat(mixed $item)```

```php
use IterTools\Infinite;

$dialogue = 'Are we there yet?';

foreach (Infinite::repeat($dialogue) as $repeated) {
    print($repeated);
}
// 'Are we there yet?', 'Are we there yet?', 'Are we there yet?', ...
```

## Итерирование случайных значений
### Choice
Генерирует случайные выборы вариантов из списка.

```Random::choice(array $items, int $repetitions)```

```php
use IterTools\Random;

$cards       = ['Ace', 'King', 'Queen', 'Jack', 'Joker'];
$repetitions = 10;

foreach (Random::choice($cards, $repetitions) as $card) {
    print($card);
}
// 'King', 'Jack', 'King', 'Ace', ... [random]
```

### CoinFlip
Генерирует случайные броски монеты (0 или 1).

```Random::coinFlip(int $repetitions)```

```php
use IterTools\Random;

$repetitions = 10;

foreach (Random::coinFlip($repetitions) as $coinFlip) {
    print($coinFlip);
}
// 1, 0, 1, 1, 0, ... [random]
```

### Number
Генерирует случайные целые числа.

```Random::number(int $min, int $max, int $repetitions)```

```php
use IterTools\Random;

$min         = 1;
$max         = 4;
$repetitions = 10;

foreach (Random::number($min, $max, $repetitions) as $number) {
    print($number);
}
// 3, 2, 5, 5, 1, 2, ... [random]
```

### Percentage
Генерирует случайные вещественные числа между 0 и 1.

```Random::percentage(int $repetitions)```

```php
use IterTools\Random;

$repetitions = 10;

foreach (Random::percentage($repetitions) as $percentage) {
    print($percentage);
}
// 0.30205562629132, 0.59648594775233, ... [random]
```

### RockPaperScissors
Случайный выбор "камень-ножницы-бумага".

```Random::rockPaperScissors(int $repetitions)```

```php
use IterTools\Random;

$repetitions = 10;

foreach (Random::rockPaperScissors($repetitions) as $rpsHand) {
    print($rpsHand);
}
// 'paper', 'rock', 'rock', 'scissors', ... [random]
```

## Математическое итерирование
### Frequencies
Возвращает генератор, при обходе которого ключами оказываются элементы поданной на вход последовательности,
а значениями — количества вхождений соответствующих элементов.

```Math::frequencies(iterable $data, bool $strict = true): \Generator```

По умолчанию выполняет сравнение в [режиме строгой типизации](#Режимы-типизации). Передайте значение `false` аргумента `$strict`, чтобы работать в режиме приведения типов.

```php
use IterTools\Math;

$grades = ['A', 'A', 'B', 'B', 'B', 'C'];

foreach (Math::frequencies($grades) as $grade => $frequency) {
    print("$grade: $frequency" . \PHP_EOL);
}
// A: 2, B: 3, C: 1
```

### Relative Frequencies
Возвращает генератор, при обходе которого ключами оказываются элементы поданной на вход последовательности,
а значениями — относительные частоты вхождений соответствующих элементов.

```Math::relativeFrequencies(iterable $data, bool $strict = true): \Generator```

По умолчанию выполняет сравнение в [режиме строгой типизации](#Режимы-типизации). Передайте значение `false` аргумента `$strict`, чтобы работать в режиме приведения типов.

```php
use IterTools\Math;

$grades = ['A', 'A', 'B', 'B', 'B', 'C'];

foreach (Math::relativeFrequencies($grades) as $grade => $frequency) {
    print("$grade: $frequency" . \PHP_EOL);
}
// A: 0.33, B: 0.5, C: 0.166
```

### Running Average
Накопление среднего арифметического элементов коллекции в процессе итерирования.

```Math::runningAverage(iterable $numbers, int|float $initialValue = null)```

```php
use IterTools\Math;

$grades = [100, 80, 80, 90, 85];

foreach (Math::runningAverage($grades) as $runningAverage) {
    print($runningAverage);
}
// 100, 90, 86.667, 87.5, 87
```

### Running Difference
Накопление разности элементов коллекции в процессе итерирования.

```Math::runningDifference(iterable $numbers, int|float $initialValue = null)```

```php
use IterTools\Math;

$credits = [1, 2, 3, 4, 5];

foreach (Math::runningDifference($credits) as $runningDifference) {
    print($runningDifference);
}
// -1, -3, -6, -10, -15
```
Опционально позволяет начать вычисления с заданного значения.
```php
use IterTools\Math;

$dartsScores   = [50, 50, 25, 50];
$startingScore = 501;

foreach (Math::runningDifference($dartsScores, $startingScore) as $runningScore) {
    print($runningScore);
}
// 501, 451, 401, 376, 326
```

### Running Max
Поиск максимального значения в процессе итерирования.

```Math::runningMax(iterable $numbers, int|float $initialValue = null)```

```php
use IterTools\Math;

$numbers = [1, 2, 1, 3, 5];

foreach (Math::runningMax($numbers) as $runningMax) {
    print($runningMax);
}
// 1, 2, 2, 3, 5
```

### Running Min
Поиск минимального значения в процессе итерирования.

```Math::runningMin(iterable $numbers, int|float $initialValue = null)```

```php
use IterTools\Math;

$numbers = [3, 4, 2, 5, 1];

foreach (Math::runningMin($numbers) as $runningMin) {
    print($runningMin);
}
// 3, 3, 2, 2, 1
```

### Running Product
Накопление произведения элементов коллекции в процессе итерирования.

```Math::runningProduct(iterable $numbers, int|float $initialValue = null)```

```php
use IterTools\Math;

$numbers = [1, 2, 3, 4, 5];

foreach (Math::runningProduct($numbers) as $runningProduct) {
    print($runningProduct);
}
// 1, 2, 6, 24, 120
```

Опционально позволяет начать вычисления с заданного значения.
```php
use IterTools\Math;

$numbers      = [1, 2, 3, 4, 5];
$initialValue = 5;

foreach (Math::runningProduct($numbers, $initialValue) as $runningProduct) {
    print($runningProduct);
}
// 5, 5, 10, 30, 120, 600
```

### Running Total
Накопление суммы элементов коллекции в процессе итерирования.

```Math::runningTotal(iterable $numbers, int|float $initialValue = null)```

```php
use IterTools\Math;

$prices = [1, 2, 3, 4, 5];

foreach (Math::runningTotal($prices) as $runningTotal) {
    print($runningTotal);
}
// 1, 3, 6, 10, 15
```

Опционально позволяет начать вычисления с заданного значения.
```php
use IterTools\Math;

$prices       = [1, 2, 3, 4, 5];
$initialValue = 5;

foreach (Math::runningTotal($prices, $initialValue) as $runningTotal) {
    print($runningTotal);
}
// 5, 6, 8, 11, 15, 20
```

## Итерирование множеств и мультимножеств
### Distinct
Фильтрует коллекцию, выдавая только уникальные значения.

```Set::distinct(iterable $data, bool $strict = true)```

По умолчанию выполняет сравнение в [режиме строгой типизации](#Режимы-типизации). Передайте значение `false` аргумента `$strict`, чтобы работать в режиме приведения типов.

```php
use IterTools\Set;

$chessSet = ['rook', 'rook', 'knight', 'knight', 'bishop', 'bishop', 'king', 'queen', 'pawn', 'pawn', ... ];

foreach (Set::distinct($chessSet) as $chessPiece) {
    print($chessPiece);
}
// rook, knight, bishop, king, queen, pawn

$mixedTypes = [1, '1', 2, '2', 3];

foreach (Set::distinct($mixedTypes, false) as $datum) {
    print($datum);
}
// 1, 2, 3
```

### Intersection
Итерирует пересечение коллекций.

```Set::intersection(iterable ...$iterables)```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила пересечения [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$chessPieces = ['rook', 'knight', 'bishop', 'queen', 'king', 'pawn'];
$shogiPieces = ['rook', 'knight', 'bishop' 'king', 'pawn', 'lance', 'gold general', 'silver general'];

foreach (Set::intersection($chessPieces, $shogiPieces) as $commonPiece) {
    print($commonPiece);
}
// rook, knight, bishop, king, pawn
```

### Intersection Coercive
Итерирует пересечение коллекций в режиме [приведения типов](#Режимы-типизации).

```Set::intersectionCoercive(iterable ...$iterables)```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила пересечения [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$numbers  = [1, 2, 3, 4, 5];
$numerics = ['1', '2', 3];

foreach (Set::intersectionCoercive($numbers, $numerics) as $commonNumber) {
    print($commonNumber);
}
// 1, 2, 3
```

### Partial Intersection
Итерирует [M-частичное пересечение](https://github.com/Smoren/partial-intersection-php) коллекций.

```Set::partialIntersection(int $minIntersectionCount, iterable ...$iterables)```

* Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила пересечения [мультимножеств](https://en.wikipedia.org/wiki/Multiset).
* Если `$minIntersectionCount = 1`, работают правила объединения [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$staticallyTyped    = ['c++', 'java', 'c#', 'go', 'haskell'];
$dynamicallyTyped   = ['php', 'python', 'javascript', 'typescript'];
$supportsInterfaces = ['php', 'java', 'c#', 'typescript'];

foreach (Set::partialIntersection(2, $staticallyTyped, $dynamicallyTyped, $supportsInterfaces) as $language) {
    print($language);
}
// c++, java, c#, go, php
```

### Partial Intersection Coercive
Итерирует [M-частичное пересечение](https://github.com/Smoren/partial-intersection-php) коллекций в режиме [приведения типов](#Режимы-типизации).

```Set::partialIntersectionCoercive(int $minIntersectionCount, iterable ...$iterables)```

* Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила пересечения [мультимножеств](https://en.wikipedia.org/wiki/Multiset).
* Если `$minIntersectionCount = 1`, работают правила объединения [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$set1 = [1, 2, 3],
$set2 = ['2', '3', 4, 5],
$set3 = [1, '2'],

foreach (Set::partialIntersectionCoercive(2, $set1, $set2, $set3) as $partiallyCommonNumber) {
    print($partiallyCommonNumber);
}
// 1, 2, 3
```

### Symmetric difference
Итерирует симметрическую разность коллекций.

```Set::symmetricDifference(iterable ...$iterables)```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила получения разности [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$a = [1, 2, 3, 4, 7];
$b = ['1', 2, 3, 5, 8];
$c = [1, 2, 3, 6, 9];

foreach (Set::symmetricDifference($a, $b, $c) as $item) {
    print($item);
}
// 1, 4, 5, 6, 7, 8, 9
```

### Symmetric difference Coercive
Итерирует симметрическую разность коллекций в режиме [приведения типов](#Режимы-типизации).

```Set::symmetricDifferenceCoercive(iterable ...$iterables)```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила получения разности [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$a = [1, 2, 3, 4, 7];
$b = ['1', 2, 3, 5, 8];
$c = [1, 2, 3, 6, 9];

foreach (Set::symmetricDifferenceCoercive($a, $b, $c) as $item) {
    print($item);
}
// 4, 5, 6, 7, 8, 9
```

### Union
Итерирует объединение коллекций.

```Set::union(iterable ...$iterables)```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила получения разности [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$a = [1, 2, 3];
$b = [3, 4];
$c = [1, 2, 3, 6, 7];

foreach (Set::union($a, $b, $c) as $item) {
    print($item);
}
//1, 2, 3, 4, 6, 7
```

### Union Coercive
Итерирует объединение коллекций в режиме [приведения типов](#Режимы-типизации).

```Set::unionCoercive(iterable ...$iterables)```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила получения разности [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Set;

$a = ['1', 2, 3];
$b = [3, 4];
$c = [1, 2, 3, 6, 7];

foreach (Set::unionCoercive($a, $b, $c) as $item) {
    print($item);
}
//1, 2, 3, 4, 6, 7
```

## Итерирование с сортировкой
### ASort
Сортирует коллекцию с сохранением ключей.

```Sort::sort(iterable $data, callable $comparator = null)```

Если `$comparator` не передан, элементы коллекции должны быть сравнимы.

```php
use IterTools\Single;
$worldPopulations = [
    'China'     => 1_439_323_776,
    'India'     => 1_380_004_385,
    'Indonesia' => 273_523_615,
    'Pakistan'  => 220_892_340,
    'USA'       => 331_002_651,
];
foreach (Sort::sort($worldPopulations) as $country => $population) {
    print("$country: $population" . \PHP_EOL);
}
// Pakistan: 220,892,340
// Indonesia: 273,523,615
// USA: 331,002,651
// India: 1,380,004,385
// China: 1,439,323,776
```

### Sort
Сортирует коллекцию.

```Single::sort(iterable $data, callable $comparator = null)```

Если `$comparator` не передан, элементы коллекции должны быть сравнимы.

```php
use IterTools\Single;

$data = [3, 4, 5, 9, 8, 7, 1, 6, 2];

foreach (Single::sort($data) as $datum) {
    print($datum);
}
// 1, 2, 3, 4, 5, 6, 7, 8, 9
```

## Итерирование файлов
### Read CSV
Итерирует коллекции ячеек CSV-файла построчно.

```File::readCsv(resource $fileHandle, string $separator = ',', string $enclosure = '"', string $escape = '\\')```

```php
use IterTools\File;
$fileHandle = \fopen('path/to/file.csv', 'r');
foreach (File::readCsv($fileHandle) as $row) {
    print_r($row);
}
// Each column field is an element of the array
```

### Read Lines
Итерирует содержимое файла построчно.

```File::readLines(resource $fileHandle)```
```php
use IterTools\File;
$fileHandle = \fopen('path/to/file.txt', 'r');
foreach (File::readLines($fileHandle) as $line) {
    print($line);
}
```

## Transform
### Tee
Создает несколько одинаковых независимых итераторов из данного.

```Transform::tee(iterable $data, int $count): array```

```php
use IterTools\Transform;
$daysOfWeek = ['Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat', 'Sun'];
$count = 3;
[$week1, $week2, $week3] = Transform::tee($data, $count);
// Each $week contains iterator containing ['Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat', 'Sun']
```

### To Array
Преобразует итерируемую коллекцию в массив.

```Transform::toArray(iterable $data): array```

```php
use IterTools\Transform;
$iterator = new \ArrayIterator([1, 2, 3, 4, 5]);
$array = Transform::toArray($iterator);
```

### To Associative Array
Преобразует итерируемую коллекцию в ассоциативный массив.

```Transform::toAssociativeArray(iterable $data, callable $keyFunc = null, callable $valueFunc = null): array```

```php
use IterTools\Transform;
$messages = ['message 1', 'message 2', 'message 3'];
$keyFunc   = fn ($msg) => \md5($msg);
$valueFunc = fn ($msg) => strtoupper($msg);
$associativeArray = Transform::toAssociativeArray($messages, $keyFunc, $valueFunc);
// [
//     '1db65a6a0a818fd39655b95e33ada11d' => 'MESSAGE 1',
//     '83b2330607fe8f817ce6d24249dea373' => 'MESSAGE 2',
//     '037805d3ad7b10c5b8425427b516b5ce' => 'MESSAGE 3',
// ]
```

### To Iterator
Преобразует итерируемую коллекцию в итератор.

```Transform::toArray(iterable $data): array```
```php
use IterTools\Transform;
$array = [1, 2, 3, 4, 5];
$iterator = Transform::toIterator($array);
```

## Саммари о коллекции
### All Match
Возвращает истину, если для всех элементов коллекции предикат вернул истину.

```Summary::allMatch(iterable $data, callable $predicate): bool```

```php
use IterTools\Summary;

$finalFantasyNumbers = [4, 5, 6];
$isOnSuperNintendo   = fn ($ff) => $ff >= 4 && $ff <= 6;

$boolean = Summary::allMatch($finalFantasyNumbers, $isOnSuperNintendo);
// true

$isOnPlaystation = fn ($ff) => $ff >= 7 && $ff <= 9;

$boolean = Summary::allMatch($finalFantasyNumbers, $isOnPlaystation);
// false
```

### All Unique
Возвращает истину, все элементы коллекции уникальны.

```Summary::allUnique(iterable $data, bool $strict = true): bool```

По умолчанию работает в [режиме строгой типизации](#Режимы-типизации). Установите параметр `$strict` в `false` для работы в режиме приведения типов.

```php
use IterTools\Summary;

$items = ['fingerprints', 'snowflakes', 'eyes', 'DNA']

$boolean = Summary::allUnique($items);
// true
```

### Any Match
Возвращает истину, если хотя бы для одного элемента коллекции предикат вернул истину.

```Summary::anyMatch(iterable $data, callable $predicate): bool```

```php
use IterTools\Summary;

$answers          = ['fish', 'towel', 42, "don't panic"];
$isUltimateAnswer = fn ($a) => a == 42;

$boolean = Summary::anyMatch($answers, $isUltimateAnswer);
// true
```

### Are Permutations
Возарщает истину, если коллекции являются перестановками друг друга.

```Summary::arePermutations(iterable ...$iterables): bool```

```php
use IterTools\Summary;
$iter = ['i', 't', 'e', 'r'];
$rite = ['r', 'i', 't', 'e'];
$reit = ['r', 'e', 'i', 't'];
$tier = ['t', 'i', 'e', 'r'];
$tire = ['t', 'i', 'r', 'e'];
$trie = ['t', 'r', 'i', 'e'];
$boolean = Summary::arePermutations($iter, $rite, $reit, $tier, $tire, $trie);
// true
```

### Are Permutations Coercive
Возвращает истину, если коллекции являются перестановками друг друга
(в режиме [приведения типов](#Режимы-типизации)).

```Summary::arePermutationsCoercive(iterable ...$iterables): bool```

```php
use IterTools\Summary;
$set1 = [1, 2.0, '3'];
$set2 = [2.0, '1', 3];
$set3 = [3, 2, 1];
$boolean = Summary::arePermutationsCoercive($set1, $set2, $set3);
// true
```

### Exactly N
Истинно, если предикат возвращает истину в точности для N элементов.

- Предикат является необязательным аргументом.
- По умолчанию предикат выполняет приведение значения элемента коллекции к типу `bool`.

```Summary::exactlyN(iterable $data, int $n, callable $predicate): bool```

```php
use IterTools\Summary;

$twoTruthsAndALie = [true, true, false];
$n                = 2;

$boolean = Summary::exactlyN($twoTruthsAndALie, $n);
// true

$ages      = [18, 21, 24, 54];
$n         = 4;
$predicate = fn ($age) => $age >= 21;

$boolean = Summary::isSorted($ages, $n, $predicate);
// false
```

### Is Partitioned
Возвращает истину, если все истинные элементы находятся в коллекции перед ложными (истинность определяет предикат).

- Возвращает истину для пустой коллекции и для коллекции с одним элементом.
- Если предикат не был передан, истинность элемента получается через приведение его значения к булевому типу.

```Summary::isPartitioned(iterable $data, callable $predicate = null): bool```

```php
use IterTools\Summary;
$numbers          = [0, 2, 4, 1, 3, 5];
$evensBeforeOdds = fn ($item) => $item % 2 === 0;
$boolean = Summary::isPartitioned($numbers, $evensBeforeOdds);
```

### Is Empty
Возвращает истину, если данная коллекция пуста.

```Summary::isEmpty(iterable $data): bool```

```php
use IterTools\Summary;

$data = []

$boolean = Summary::isEmpty($data);
// true
```

### Is Sorted
Возвращает истину, если коллекция отсортирована в прямом порядке.

- Элементы должны быть сравнимы.
- Для пустой коллекции или коллекции из одного элемента всегда возвращает истину.

```Summary::isSorted(iterable $data): bool```

```php
use IterTools\Summary;

$numbers = [1, 2, 3, 4, 5];

$boolean = Summary::isSorted($numbers);
// true

$numbers = [3, 2, 3, 4, 5];

$boolean = Summary::isSorted($numbers);
// false
```

### Is Reversed
Возвращает истину, если коллекция отсортирована в обратном порядке.

- Элементы коллекции должны быть сравнимы.
- Для пустой коллекции или коллекции из одного элемента всегда возвращает истину.

```Summary::isReversed(iterable $data): bool```

```php
use IterTools\Summary;

$numbers = [5, 4, 3, 2, 1];

$boolean = Summary::isReversed($numbers);
// true

$numbers = [1, 4, 3, 2, 1];

$boolean = Summary::isReversed($numbers);
// false
```

### None Match
Возвращает истину, если для всех элементов коллекции предикат вернул ложь.

```Summary::noneMatch(iterable $data, callable $predicate): bool```

```php
use IterTools\Summary;

$grades         = [45, 50, 61, 0];
$isPassingGrade = fn ($grade) => $grade >= 70;

$boolean = Summary::noneMatch($grades, $isPassingGrade);
// true
```

### Same
Истинно, если данные коллекции одинаковы.

Если в метод передать одну коллекцию или ни одной, он вернет истину.

```Summary::same(iterable ...$iterables): bool```

```php
use IterTools\Summary;

$cocaColaIngredients = ['carbonated water', 'sugar', 'caramel color', 'phosphoric acid'];
$pepsiIngredients    = ['carbonated water', 'sugar', 'caramel color', 'phosphoric acid'];

$boolean = Summary::same($cocaColaIngredients, $pepsiIngredients);
// true

$cocaColaIngredients = ['carbonated water', 'sugar', 'caramel color', 'phosphoric acid'];
$spriteIngredients   = ['carbonated water', 'sugar', 'citric acid', 'lemon lime flavorings'];

$boolean = Summary::same($cocaColaIngredients, $spriteIngredients);
// false
```

### Same Count
Истинно, если данные коллекции имеют одинаковую длину.

Если в метод передать одну коллекцию или ни одной, он вернет истину.

```Summary::sameCount(iterable ...$iterables): bool```

```php
use IterTools\Summary;

$prequels  = ['Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith'];
$originals = ['A New Hope', 'Empire Strikes Back', 'Return of the Jedi'];
$sequels   = ['The Force Awakens', 'The Last Jedi', 'The Rise of Skywalker'];

$boolean = Summary::sameCount($prequels, $originals, $sequels);
// true

$batmanMovies = ['Batman Begins', 'The Dark Knight', 'The Dark Knight Rises'];
$matrixMovies = ['The Matrix', 'The Matrix Reloaded', 'The Matrix Revolutions', 'The Matrix Resurrections'];

$result = Summary::sameCount($batmanMovies, $matrixMovies);
// false
```

## Reduce
### To Average
Возвращает среднее арифметическое элементов коллекции.

Для пустой коллекции возвращает `null`.

```Reduce::toAverage(iterable $data): float```

```php
use IterTools\Reduce;

$grades = [100, 90, 95, 85, 94];

$finalGrade = Reduce::toAverage($numbers);
// 92.8
```

### To Count
Возвращает длину данной коллекции.

```Reduce::toCount(iterable $data): int```

```php
use IterTools\Reduce;

$someIterable = ImportantThing::getCollectionAsIterable();

$length = Reduce::toCount($someIterable);
// 3
```

### To First
Возвращает первый элемент коллекции.

```Reduce::toFirst(iterable $data): mixed```

Бросает `\LengthException` если коллекция пуста.

```php
use IterTools\Reduce;

$input = [10, 20, 30];

$result = Reduce::toFirst($input);
// 10
```

### To First And Last
Возвращает первый и последний элементы коллекции.

```Reduce::toFirstAndLast(iterable $data): array{mixed, mixed}```

Бросает `\LengthException` если хранимая в потоке коллекция пуста.

```php
use IterTools\Reduce;

$input = [10, 20, 30];

$result = Reduce::toFirstAndLast($input);
// [10, 30]
```

### To Last
Возвращает последний элемент коллекции.

```Reduce::toLast(iterable $data): mixed```

Бросает `\LengthException` если коллекция пуста.

```php
use IterTools\Reduce;

$input = [10, 20, 30];

$result = Reduce::toLast($input);
// 30
```

### To Max
Возвращает максимальный элемент коллекции.

```Reduce::toMax(iterable $data, callable $compareBy = null): mixed|null```

- Функция `$compareBy` должна возвращать сравнимое значение.
- Если аргумент `$compareBy` не передан, элементы коллекции должны быть сравнимы.
- Для пустой коллекции возвращает `null`.

```php
use IterTools\Reduce;

$numbers = [5, 3, 1, 2, 4];

$result = Reduce::toMax($numbers);
// 5

$movieRatings = [
    [
        'title' => 'Star Wars: Episode IV - A New Hope',
        'rating' => 4.6
    ],
    [
        'title' => 'Star Wars: Episode V - The Empire Strikes Back',
        'rating' => 4.8
    ],
    [
        'title' => 'Star Wars: Episode VI - Return of the Jedi',
        'rating' => 4.6
    ],
];
$compareBy = fn ($movie) => $movie['rating'];
$highestRatedMovie = Reduce::toMax($movieRatings, $compareBy);
// [
//     'title' => 'Star Wars: Episode V - The Empire Strikes Back',
//     'rating' => 4.8
// ];
```

### To Min
Возвращает минимальный элемент коллекции.

```Reduce::toMin(iterable $data, callable $compareBy = null): mixed|null```

- Функция `$compareBy` должна возвращать сравнимое значение.
- Если аргумент `$compareBy` не передан, элементы коллекции должны быть сравнимы.
- Для пустой коллекции возвращает `null`.

```php
use IterTools\Reduce;

$numbers = [5, 3, 1, 2, 4];

$result = Reduce::toMin($numbers);
// 1

$movieRatings = [
    [
        'title' => 'The Matrix',
        'rating' => 4.7
    ],
    [
        'title' => 'The Matrix Reloaded',
        'rating' => 4.3
    ],
    [
        'title' => 'The Matrix Revolutions',
        'rating' => 3.9
    ],
    [
        'title' => 'The Matrix Resurrections',
        'rating' => 2.5
    ],
];
$compareBy = fn ($movie) => $movie['rating'];
$lowestRatedMovie = Reduce::toMin($movieRatings, $compareBy);
// [
//     'title' => 'The Matrix Resurrections',
//     'rating' => 2.5
// ]
```

### To Min Max
Возвращает минимальный и максимальный элементы коллекции.

```Reduce::toMinMax(iterable $numbers, callable $compareBy = null): array```

- Функция `$compareBy` должна возвращать сравнимое значение.
- Если аргумент `$compareBy` не передан, элементы коллекции должны быть сравнимы.
- Для пустой коллекции возвращает `[null, null]`.

```php
use IterTools\Reduce;

$numbers = [1, 2, 7, -1, -2, -3];;

[$min, $max] = Reduce::toMinMax($numbers);
// [-3, 7]

$reportCard = [
    [
        'subject' => 'history',
        'grade' => 90
    ],
    [
        'subject' => 'math',
        'grade' => 98
    ],
    [
        'subject' => 'science',
        'grade' => 92
    ],
    [
        'subject' => 'english',
        'grade' => 85
    ],
    [
        'subject' => 'programming',
        'grade' => 100
    ],
];
$compareBy = fn ($class) => $class['grade'];
$bestAndWorstSubject = Reduce::toMinMax($reportCard, $compareBy);
// [
//     [
//         'subject' => 'english',
//         'grade' => 85
//     ],
//     [
//         'subject' => 'programming',
//         'grade' => 100
//     ],
// ]
```

### To Nth
Возвращает n-й элемент коллекции.

```Reduce::toNth(iterable $data, int $position): mixed```

```php
use IterTools\Reduce;

$lotrMovies = ['The Fellowship of the Ring', 'The Two Towers', 'The Return of the King'];

$rotk = Reduce::toNth($lotrMovies, 2);
// 20
```

### To Product
Возвращает произведение элементов коллекции.

Для пустой коллекции возвращает `null`.

```Reduce::toProduct(iterable $data): number|null```

```php
use IterTools\Reduce;

$primeFactors = [5, 2, 2];

$number = Reduce::toProduct($primeFactors);
// 20
```

### To Random Value
Возвращает случайный элемент из коллекции.

```Reduce::toRandomValue(iterable $data): mixed```

```php
use IterTools\Reduce;

$sfWakeupOptions = ['mid', 'low', 'overhead', 'throw', 'meaty'];

$wakeupOption = Reduce::toRandomValue($sfWakeupOptions);
// e.g., throw
```

### To Range
Возвращает разность максимального и минимального элементов коллекции.

```Reduce::toRange(iterable $numbers): int|float```

Для пустой коллекции возвращает `0`.

```php
use IterTools\Reduce;

$grades = [100, 90, 80, 85, 95];

$range = Reduce::toRange($numbers);
// 20
```

### To String
Преобразует коллекцию в строку, "склеивая" ее элементы.

* Значение необязательного аргумента `$separator` вставляется в качестве разделителя между элементами в строке.
* Значение необязательного аргумента `$prefix` вставляется в начало строки.
* Значение необязательного аргумента `$suffix` вставляется в конец строки.

```Reduce::toString(iterable $data, string $separator = '', string $prefix = '', string $suffix = ''): string```

```php
use IterTools\Reduce;

$words = ['IterTools', 'PHP', 'v1.0'];

$string = Reduce::toString($words);
// IterToolsPHPv1.0
$string = Reduce::toString($words, '-');
// IterTools-PHP-v1.0
$string = Reduce::toString($words, '-', 'Library: ');
// Library: IterTools-PHP-v1.0
$string = Reduce::toString($words, '-', 'Library: ', '!');
// Library: IterTools-PHP-v1.0!
```

### To Sum
Возвращает сумму элементов коллекции.

```Reduce::toSum(iterable $data): number```

```php
use IterTools\Reduce;

$parts = [10, 20, 30];

$sum = Reduce::toSum($parts);
// 60
```

### To Value
Редуцирует коллекцию до значения, вычисляемого с использованием callback-функции.

```Reduce::toValue(iterable $data, callable $reducer, mixed $initialValue): mixed```

```php
use IterTools\Reduce;

$input = [1, 2, 3, 4, 5];
$sum   = fn ($carry, $item) => $carry + $item;

$result = Reduce::toValue($input, $sum, 0);
// 15
```

## Цепочечный вызов итераторов

Предоставляет гибкий текучий интерфейс для преобразования массивов и других итерируемых сущностей с помощью конвейера операций.

Данный функционал содержит в себе:

1. Фабричные методы для создания объекта, предоставляющего текучий интерфейс для работы с итерируемыми сущностями.
2. Методы для преобразования текущего состояние потока в новый поток.
3. Способы завершения потока преобразований:
   * Методы, преобразующие поток в скалярное значение или в структуру данных.
   ```php
   $result = Stream::of([1, 1, 2, 2, 3, 4, 5])
      ->distinct()                      // [1, 2, 3, 4, 5]
      ->map(fn ($x) => $x**2)           // [1, 4, 9, 16, 25]
      ->filterTrue(fn ($x) => $x < 10)  // [1, 4, 9]
      ->toSum();                        // 14
   ```
   * Возможность проитерировать результат потока на любом этапе с использованием цикла `foreach`.
   ```php
   $result = Stream::of([1, 1, 2, 2, 3, 4, 5])
      ->distinct()                      // [1, 2, 3, 4, 5]
      ->map(fn ($x) => $x**2)           // [1, 4, 9, 16, 25]
      ->filterTrue(fn ($x) => $x < 10); // [1, 4, 9]
   
   foreach ($result as $item) {
       // 1, 4, 9
   }
   ```

### Фабричные методы

#### Of
Создает поток из данной коллекции.

```Stream::of(iterable $iterable): Stream```

```php
use IterTools\Stream;

$iterable = [1, 2, 3];

$result = Stream::of($iterable)
    ->chainWith([4, 5, 6], [7, 8, 9])
    ->zipEqualWith([1, 2, 3, 4, 5, 6, 7, 8, 9])
    ->toValue(fn ($carry, $item) => $carry + array_sum($item));
// 90
```

#### Of Coin Flips
Создает поток из бесконечных случайных бросков монеты.

```Stream::ofCoinFlips(int $repetitions): Stream```

```php
use IterTools\Stream;

$result = Stream::ofCoinFlips(10)
    ->filterTrue()
    ->toCount();
// 5 (random)
```

#### Of CSV File
Создает поток из строк CSV-файла.

```Stream::ofCsvFile(resource $fileHandle, string $separator = ',', string $enclosure = '"', string = $escape = '\\'): Stream```

```php
use IterTools\Stream;
$fileHandle = \fopen('path/to/file.csv', 'r');
$result = Stream::of($fileHandle)
    ->toArray();
```

#### Of Empty
Создает поток из пустой коллекции.

```Stream::ofEmpty(): Stream```

```php
use IterTools\Stream;

$result = Stream::ofEmpty()
    ->chainWith([1, 2, 3])
    ->toArray();
// 1, 2, 3
```

#### Of File Lines
Создает поток из строк файла.

```Stream::ofFileLines(resource $fileHandle): Stream```
```php
use IterTools\Stream;
$fileHandle = \fopen('path/to/file.txt', 'r');
$result = Stream::of($fileHandle)
    ->map('strtoupper');
    ->toArray();
```

#### Of Random Choice
Создает поток из бесконечных случайных выборов элемента из списка.

```Stream::ofRandomChoice(array $items, int $repetitions): Stream```

```php
use IterTools\Stream;

$languages = ['PHP', 'Go', 'Python'];

$languages = Stream::ofRandomChoice($languages, 5)
    ->toArray();
// 'Go', 'PHP', 'Python', 'PHP', 'PHP' (random)
```

#### Of Random Numbers
Создает поток из бесконечного набора случайных целых чисел.

```Stream::ofRandomNumbers(int $min, int $max, int $repetitions): Stream```

```php
use IterTools\Stream;

$min  = 1;
$max  = 3;
$reps = 7;

$result = Stream::ofRandomNumbers($min, $max, $reps)
    ->toArray();
// 1, 2, 2, 1, 3, 2, 1 (random)
```

#### Of Random Percentage
Создает поток из бесконечного набора случайных вещественных чисел между 0 и 1.

```Stream::ofRandomPercentage(int $repetitions): Stream```

```php
use IterTools\Stream;

$stream = Stream::ofRandomPercentage(3)
    ->toArray();
// 0.8012566976245, 0.81237281724151, 0.61676896329459 [random]
```

#### Of Range
Создает поток для цепочечных вызовов из арифметической прогрессии.

```Stream::ofRange(int|float $start, int|float $end, int|float $step = 1): Stream```

```php
use IterTools\Stream;
$numbers = Stream::ofRange(0, 5)
    ->toArray();
// 0, 1, 2, 3, 4, 5
```

#### Of Rock Paper Scissors
Создает поток из бесконечных случайных выборов "камень-ножницы-бумага".

```Stream::ofRockPaperScissors(int $repetitions): Stream```

```php
use IterTools\Stream;

$rps = Stream::ofRockPaperScissors(5)
    ->toArray();
// 'paper', 'rock', 'rock', 'scissors', 'paper' [random]
```

### Цепочечные операции

#### ASort
Сортирует коллекцию в потоке с сохранением ключей.

```$stream->asort(callable $comparator = null)```

Если `$comparator` не передан, элементы хранимой коллекции должны быть сравнимы.

```php
use IterTools\Stream;
$worldPopulations = [
    'China'     => 1_439_323_776,
    'India'     => 1_380_004_385,
    'Indonesia' => 273_523_615,
    'USA'       => 331_002_651,
];
$result = Stream::of($worldPopulations)
    ->filter(fn ($pop) => $pop > 300_000_000)
    ->asort()
    ->toAssociativeArray();
// USA   => 331_002_651,
// India => 1_380_004_385,
// China => 1_439_323_776,
```

#### Chain With
Добавляет в конец потокового итератора другие коллекции для последовательного итерирования.

```$stream->chainWith(iterable ...$iterables): Stream```

Создает одну длинную последовательность из последовательности в потоке и нескольких данных последовательностей.

```php
use IterTools\Stream;

$input = [1, 2, 3];

$result = Stream::of($input)
    ->chainWith([4, 5, 6])
    ->chainWith([7, 8, 9])
    ->toArray();
// 1, 2, 3, 4, 5, 6, 7, 8, 9
```

#### Compress
Отфильтровывает из потока элементы, которые не выбраны.

```$stream->compress(iterable $selectors): Stream```

Массив селекторов уточняет, какие элементы помещать в выборку (значение селектора `1`),
а какие исключать (значение селектора `0`).

```php
use IterTools\Stream;

$input = [1, 2, 3];

$result = Stream::of($input)
    ->compress([0, 1, 1])
    ->toArray();
// 2, 3
```

#### Compress Associative
Выбирает из хранимой коллекции элементы по заданным ключам.

```$stream->compressAssociative(array $keys): Stream```

* Ключами могут быть только строки или целые числа (по аналогии с ключами PHP-массивов).

```php
use IterTools\Stream;
$starWarsEpisodes = [
    'I'    => 'The Phantom Menace',
    'II'   => 'Attack of the Clones',
    'III'  => 'Revenge of the Sith',
    'IV'   => 'A New Hope',
    'V'    => 'The Empire Strikes Back',
    'VI'   => 'Return of the Jedi',
    'VII'  => 'The Force Awakens',
    'VIII' => 'The Last Jedi',
    'IX'   => 'The Rise of Skywalker',
];
$sequelTrilogyNumbers = ['VII', 'VIII', 'IX'];
$sequelTrilogy = Stream::of($starWarsEpisodes)
    ->compressAssociative($sequelTrilogyNumbers)
    ->toAssociativeArray();
// 'VII'  => 'The Force Awakens',
// 'VIII' => 'The Last Jedi',
// 'IX'   => 'The Rise of Skywalker',
```

#### Chunkwise
Итерирует элементы из потока с разбиением по чанкам.

```$stream->chunkwise(int $chunkSize): Stream```

Минимальный размер чанка — 1.

```php
use IterTools\Stream;

$friends = ['Ross', 'Rachel', 'Chandler', 'Monica', 'Joey'];

$result = Stream::of($friends)
    ->chunkwise(2)
    ->toArray();
// ['Ross', 'Rachel'], ['Chandler', 'Monica'], ['Joey']
```

#### Chunkwise Overlap
Итерирует элементы из потока с разбиением по взаимонакладывающимся чанкам.

```$stream->chunkwiseOverlap(int $chunkSize, int $overlapSize, bool $includeIncompleteTail = true): Stream```

* Минимальный размер чанка — 1.
* Размер наложения должен быть меньше длины чанка.

```php
use IterTools\Stream;

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9];

$result = Stream::of($friends)
    ->chunkwiseOverlap(3, 1)
    ->toArray()
// [1, 2, 3], [3, 4, 5], [5, 6, 7], [7, 8, 9]
```

#### Distinct
Фильтрует элементы из потока, сохраняя только уникальные значения.

```$stream->distinct(bool $strict = true): Stream```

По умолчанию выполняет сравнения в [режиме строгой типизации](#Режимы-типизации). Передайте значение `false` аргумента `$strict`, чтобы работать в режиме приведения типов.

```php
use IterTools\Stream;

$input = [1, 2, 1, 2, 3, 3, '1', '1', '2', '3'];
$stream = Stream::of($input)
    ->distinct()
    ->toArray();
// 1, 2, 3, '1', '2', '3'

$stream = Stream::of($input)
    ->distinct(false)
    ->toArray();
// 1, 2, 3
```

#### Drop While
Пропускает элементы из потока, пока предикат возвращает ложь.

```$stream->dropWhile(callable $predicate): Stream```

После того как предикат впервые вернул `true`, все последующие элементы попадают в выборку.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5]

$result = Stream::of($input)
    ->dropWhile(fn ($value) => $value < 3)
    ->toArray();
// 3, 4, 5
```

#### Filter
Возвращает из потока только те элементы, для которых предикат возвращает истину.

```$stream->filter(callable $predicate): Stream```

По умолчанию (если не передан) предикат приводит элементы коллекции к `bool`.

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($input)
    ->filter(fn ($value) => $value > 0)
    ->toArray();
// 1, 2, 3
```

#### Filter True
Возвращает из потока только истинные элементы. Истинность определяется предикатом.

```$stream->filterTrue(callable $predicate = null): Stream```

По умолчанию (если не передан) предикат приводит элементы коллекции к `bool`.

```php
use IterTools\Stream;
$input = [0, 1, 2, 3, 0, 4];
$result = Stream::of($input)
    ->filterTrue()
    ->toArray();
// 1, 2, 3, 4
```

#### Filter False
Возвращает из потока только ложные элементы. Истинность определяется предикатом.

```$stream->filterFalse(callable $predicate = null): Stream```

По умолчанию (если не передан) предикат приводит элементы коллекции к `bool`.

```php
use IterTools\Stream;

$input = [0, 1, 2, 3, 0, 4];

$result = Stream::of($input)
    ->filterFalse(fn ($value) => $value > 0)
    ->filterFalse()
    ->toArray();
// 0, 0
```

#### Filter Keys
Возвращает из потока только те элементы, для ключей которых предикат возвращает истину.

```$stream->filterKeys(callable $filter): Stream```

```php
$olympics = [
    2000 => 'Sydney',
    2002 => 'Salt Lake City',
    2004 => 'Athens',
    2006 => 'Turin',
    2008 => 'Beijing',
    2010 => 'Vancouver',
    2012 => 'London',
    2014 => 'Sochi',
    2016 => 'Rio de Janeiro',
    2018 => 'Pyeongchang',
    2020 => 'Tokyo',
    2022 => 'Beijing',
];
$winterFilter = fn ($year) => $year % 4 === 2;
$result = Stream::of($olympics)
    ->filterKeys($winterFilter)
    ->toAssociativeArray();
}
// 2002 => Salt Lake City
// 2006 => Turin
// 2010 => Vancouver
// 2014 => Sochi
// 2018 => Pyeongchang
// 2022 => Beijing
```

#### Flat Map
Отображение коллекции из потока с уплощением результата на 1 уровень вложенности.

```$stream->flatMap(callable $mapper): Stream```

```php
$data    = [1, 2, 3, 4, 5];
$mapper  fn ($item) => ($item % 2 === 0) ? [$item, $item] : $item;
$result = Stream::of($data)
    ->flatMap($mapper)
    ->toArray();
// [1, 2, 2, 3, 4, 4, 5]
```

#### Flatten
Многоуровневое уплощение коллекции из потока.

```$stream->flatten(int $dimensions = 1): Stream```

```php
$data = [1, [2, 3], [4, 5]];
$result = Stream::of($data)
    ->flatten($mapper)
    ->toArray();
// [1, 2, 3, 4, 5]
```

#### Frequencies
Абсолютная частота распредения элементов потока.

```$stream->frequencies(bool $strict = true): Stream```

```php
use IterTools\Stream;

$grades = ['A', 'A', 'B', 'B', 'B', 'C'];

$result = Stream::of($grades)
    ->frequencies()
    ->toAssociativeArray();

// ['A' => 2, 'B' => 3, 'C' => 1]
```

#### Group By
Группирует элементы из потока по заданному правилу.

```$stream->groupBy(callable $groupKeyFunction): Stream```

Функция `$groupKeyFunction` должна возвращать общий ключ для элементов группы.

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($input)
    ->groupBy(fn ($item) => $item > 0 ? 'positive' : 'negative');

foreach ($result as $group => $item) {
    // 'positive' => [1, 2, 3], 'negative' => [-1, -2, -3]
}
```

#### Infinite Cycle
Бесконечно зацикливает перебор элементов потока.

```$stream->infiniteCycle(): Stream```

```php
use IterTools\Stream;

$input = [1, 2, 3];

$result = Stream::of($input)
    ->infiniteCycle()
    ->print();
// 1, 2, 3, 1, 2, 3, ...
```

#### Intersection With
Пересечение хранимой в потоке коллекции с другими переданными коллекциями.

```$stream->intersectionWith(iterable ...$iterables): Stream```

```php
use IterTools\Stream;

$numbers    = [1, 2, 3, 4, 5, 6, 7, 8, 9];
$numerics   = ['1', '2', 3, 4, 5, 6, 7, '8', '9'];
$oddNumbers = [1, 3, 5, 7, 9, 11];

$stream = Stream::of($numbers)
    ->intersectionWith($numerics, $oddNumbers)
    ->toArray();
// 3, 5, 7
```

#### Intersection Coercive With
Пересечение хранимой в потоке коллекции с другими переданными коллекциями в режиме [приведения типов](#Режимы-типизации).

```$stream->intersectionCoerciveWith(iterable ...$iterables): Stream```

```php
use IterTools\Stream;

$languages          = ['php', 'python', 'c++', 'java', 'c#', 'javascript', 'typescript'];
$scriptLanguages    = ['php', 'python', 'javascript', 'typescript'];
$supportsInterfaces = ['php', 'java', 'c#', 'typescript'];

$stream = Stream::of($languages)
    ->intersectionCoerciveWith($scriptLanguages, $supportsInterfaces)
    ->toArray();
// 'php', 'typescript'
```

#### Limit
Ограничивает итерирование элементов из потока заданным максимальным числом итераций.

Останавливает процесс итерирования, когда число итераций достигает `$limit`.

```$stream->limit(int $limit): Stream```

```php
Use IterTools\Single;

$matrixMovies = ['The Matrix', 'The Matrix Reloaded', 'The Matrix Revolutions', 'The Matrix Resurrections'];
$limit        = 1;

$goodMovies = Stream::of($matrixMovies)
    ->limit($limit)
    ->toArray();
// 'The Matrix' (and nothing else)
```

#### Map
Отображение хранимой в потоке коллекции с использованием callback-функции.

```$stream->map(callable $function): Stream```

```php
use IterTools\Stream;

$grades = [100, 95, 98, 89, 100];

$result = Stream::of($grades)
    ->map(fn ($grade) => $grade === 100 ? 'A' : 'F')
    ->toArray();
// A, F, F, F, A
```

#### Pairwise
Итерирует элементы из потока попарно (с наложением).

```$stream->pairwise(): Stream```

Итоговый поток окажется пустым, если исходный содержит меньше 2-х элементов.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$stream = Stream::of($input)
    ->pairwise()
    ->toArray();
// [1, 2], [2, 3], [3, 4], [4, 5]
```

#### Partial Intersection With
Частичное пересечение хранимой в потоке коллекции с другими переданными коллекциями.

```$stream->partialIntersectionWith(int $minIntersectionCount, iterable ...$iterables): Stream```

```php
use IterTools\Stream;

$numbers    = [1, 2, 3, 4, 5, 6, 7, 8, 9];
$numerics   = ['1', '2', 3, 4, 5, 6, 7, '8', '9'];
$oddNumbers = [1, 3, 5, 7, 9, 11];

$stream = Stream::of($numbers)
    ->partialIntersectionWith($numerics, $oddNumbers)
    ->toArray();
// 1, 3, 4, 5, 6, 7, 9
```

#### Partial Intersection Coercive With
Частичное пересечение хранимой в потоке коллекции с другими переданными коллекциями, вычисляемое в режиме [приведения типов](#Режимы-типизации).

```$stream->partialIntersectionCoerciveWith(int $minIntersectionCount, iterable ...$iterables): Stream```

```php
use IterTools\Stream;

$languages          = ['php', 'python', 'c++', 'java', 'c#', 'javascript', 'typescript'];
$scriptLanguages    = ['php', 'python', 'javascript', 'typescript'];
$supportsInterfaces = ['php', 'java', 'c#', 'typescript'];

$stream = Stream::of($languages)
    ->partialIntersectionCoerciveWith(2, $scriptLanguages, $supportsInterfaces)
    ->toArray();
// 'php', 'python', 'java', 'typescript', 'c#', 'javascript'
```

#### Reindex
Переиндексирует key-value коллекцию из потока, используя функцию-индексатор.

```$stream->reindex(callable $indexer): Stream```

```php
use IterTools\Stream;
$data = [
    [
        'title'   => 'Star Wars: Episode IV – A New Hope',
        'episode' => 'IV',
        'year'    => 1977,
    ],
    [
        'title'   => 'Star Wars: Episode V – The Empire Strikes Back',
        'episode' => 'V',
        'year'    => 1980,
    ],
    [
        'title' => 'Star Wars: Episode VI – Return of the Jedi',
        'episode' => 'VI',
        'year' => 1983,
    ],
];
$reindexFunc = fn (array $swFilm) => $swFilm['episode'];
$reindexResult = Stream::of($data)
    ->reindex($reindexFunc)
    ->toAssociativeArray();
// [
//     'IV' => [
//         'title'   => 'Star Wars: Episode IV – A New Hope',
//         'episode' => 'IV',
//         'year'    => 1977,
//     ],
//     'V' => [
//         'title'   => 'Star Wars: Episode V – The Empire Strikes Back',
//         'episode' => 'V',
//         'year'    => 1980,
//     ],
//     'VI' => [
//         'title' => 'Star Wars: Episode VI – Return of the Jedi',
//         'episode' => 'VI',
//         'year' => 1983,
//     ],
// ]
```

#### Relative Frequencies
Относительная частота распредения элементов потока.

```$stream->relativeFrequencies(bool $strict = true): Stream```

```php
use IterTools\Stream;

$grades = ['A', 'A', 'B', 'B', 'B', 'C'];

$result = Stream::of($grades)
    ->relativeFrequencies()
    ->toAssociativeArray();

// A => 0.33, B => 0.5, C => 0.166
```

#### Reverse
Итерирует коллекцию из потока в обратном порядке.

```$stream->reverse(): Stream```

```php
use IterTools\Stream;
$words = ['are', 'you', 'as', 'bored', 'as', 'I', 'am'];
$reversed = Stream::of($words)
    ->reverse()
    ->toString(' ');
// am I as bored as you are
```

#### Running Average
Накапливает среднее арифметическое элементов из потока в процессе итерирования.

```$stream->runningAverage(int|float|null $initialValue = null): Stream```

```php
use IterTools\Stream;

$input = [1, 3, 5];

$result = Stream::of($input)
    ->runningAverage();

foreach ($result as $item) {
    // 1, 2, 3
}
```

#### Running Difference
Накапливает разность элементов из потока в процессе итерирования.

```$stream->runningDifference(int|float|null $initialValue = null): Stream```

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->runningDifference()
    ->toArray();
// -1, -3, -6, -10, -15
```

#### Running Max
Возвращает поток, ищущий максимальный элемент из исходного потока в процессе итерирования.

```$stream->runningMax(int|float|null $initialValue = null): Stream```

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($input)
    ->runningMax()
    ->toArray();
// 1, 1, 2, 2, 3, 3
```

#### Running Min
Возвращает поток, ищущий минимальный элемент из исходного потока в процессе итерирования.

```$stream->runningMin(int|float|null $initialValue = null): Stream```

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($input)
    ->runningMin()
    ->toArray();
// 1, -1, -1, -2, -2, -3
```

#### Running Product
Возвращает поток, накапливающий произведение элементов из исходного потока в процессе итерирования.

```$stream->runningProduct(int|float|null $initialValue = null): Stream```

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->runningProduct()
    ->toArray();
// 1, 2, 6, 24, 120
```

#### Running Total
Возвращает поток, накапливающий сумму элементов из исходного потока в процессе итерирования.

```$stream->runningTotal(int|float|null $initialValue = null): Stream```

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->runningTotal()
    ->toArray();
// 1, 3, 6, 10, 15
```

#### Skip
Пропускает n элементов из потока и опциональным смещением.

```$stream->skip(int $count, int $offset = 0): Stream```

```php
use IterTools\Stream;

$movies = [
    'The Phantom Menace', 'Attack of the Clones', 'Revenge of the Sith',
    'A New Hope', 'The Empire Strikes Back', 'Return of the Jedi',
    'The Force Awakens', 'The Last Jedi', 'The Rise of Skywalker'
];

$onlyTheBest = Stream::of($movies)
    ->skip(3)
    ->skip(3, 3)
    ->toArray();
// 'A New Hope', 'The Empire Strikes Back', 'Return of the Jedi'
```

#### Slice
Выделяет подвыборку коллекции из потока.

```$stream->slice(int $start = 0, int $count = null, int $step = 1)```

```php
use IterTools\Stream;
$olympics = [1992, 1994, 1996, 1998, 2000, 2002, 2004, 2006, 2008, 2010, 2012, 2014, 2016, 2018, 2020, 2022];
$summerOlympics = Stream::of($olympics)
    ->slice(0, 8, 2)
    ->toArray();
// [1992, 1996, 2000, 2004, 2008, 2012, 2016, 2020]
```

#### Sort
Сортирует хранимую в потоке коллекцию.

```$stream->sort(callable $comparator = null)```

Если `$comparator` не передан, элементы хранимой коллекции должны быть сравнимы.

```php
use IterTools\Stream;

$input = [3, 4, 5, 9, 8, 7, 1, 6, 2];

$result = Stream::of($input)
    ->sort()
    ->toArray();
// 1, 2, 3, 4, 5, 6, 7, 8, 9
```

#### Symmetric difference With
Возвращает поток, содержащий симметрическую разность исходного потока с заданным набором коллекций.

```$stream->symmetricDifferenceWith(iterable ...$iterables): Stream```

Если хотя бы в одной коллекции или в потоке встречаются повторяющиеся элементы, работают правила получения разности для [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Stream;

$a = [1, 2, 3, 4, 7];
$b = ['1', 2, 3, 5, 8];
$c = [1, 2, 3, 6, 9];

$stream = Stream::of($a)
    ->symmetricDifferenceWith($b, $c)
    ->toArray();
// '1', 4, 5, 6, 7, 8, 9
```

#### Symmetric difference Coercive With
Возвращает поток, содержащий симметрическую разность исходного потока с заданным набором коллекций, полученную в режиме [приведения типов](#Режимы-типизации).

```$stream->symmetricDifferenceCoerciveWith(iterable ...$iterables): Stream```

Если хотя бы в одной коллекции или в потоке встречаются повторяющиеся элементы, работают правила получения разности для [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Stream;

$a = [1, 2, 3, 4, 7];
$b = ['1', 2, 3, 5, 8];
$c = [1, 2, 3, 6, 9];

$stream = Stream::of($a)
    ->symmetricDifferenceCoerciveWith($b, $c)
    ->toArray();
// 4, 5, 6, 7, 8, 9
```

#### Take While
Оставляет элементы в потоке, пока предикат возвращает истину.

```$stream->takeWhile(callable $predicate): Stream```

* Останавливает процесс итерации, как только предикат впервые вернет ложь.
* По умолчанию (если не передан) предикат приводит элементы коллекции к `bool`.

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($input)
    ->takeWhile(fn ($value) => abs($value) < 3);

foreach ($result as $item) {
    // 1, -1, 2, -2
}
```

#### Union With
Возвращает поток с объединением хранимой коллекции с другими поданными на вход коллекциями.

```$stream->unionWith(iterable ...$iterables): Stream```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила получения разности [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Stream;

$input = [1, 2, 3];

$stream = Stream::of($input)
    ->unionWith([3, 4, 5, 6])
    ->toArray();
// [1, 2, 3, 4, 5, 6]
```

#### Union Coercive With
Возвращает поток с объединением хранимой коллекции с другими поданными на вход коллекциями в режиме [приведения типов](#Режимы-типизации).

```$stream->unionCoerciveWith(iterable ...$iterables): Stream```

Если хотя бы в одной коллекции встречаются повторяющиеся элементы, работают правила получения разности [мультимножеств](https://en.wikipedia.org/wiki/Multiset).

```php
use IterTools\Stream;

$input = [1, 2, 3];

$stream = Stream::of($input)
    ->unionCoerciveWith(['3', 4, 5, 6])
    ->toArray();
// [1, 2, 3, 4, 5, 6]
```

#### Zip With
Параллельно итерирует элементы из потока вместе с элементами переданных коллекций, пока не закончится самый короткий итератор.

```$stream->zipWith(iterable ...$iterables): Stream```

* Создает итератор, который агрегирует данные из нескольких итераторов.
* Работает аналогично функции `zip()` в Python.
* Для коллекций разной длины продолжает процесс итерирования до момента, пока самая короткая коллекция не закончится.

```php
use IterTools\Stream;

$input = [1, 2, 3];

$stream = Stream::of($input)
    ->zipWith([4, 5, 6])
    ->zipWith([7, 8, 9])
    ->toArray();
// [1, 4, 7], [2, 5, 8], [3, 6, 9]
```

#### Zip Filled With
Параллельно итерирует элементы из потока вместе с элементами переданных коллекций, пока не закончится самый длинный итератор.

Для закончившихся итераторов подставляет заданный филлер в кортеж значений итерации.

```$stream->zipFilledWith(mixed $default, iterable ...$iterables): Stream```

```php
use IterTools\Stream;

$input = [1, 2, 3];

$stream = Stream::of($input)
    ->zipFilledWith('?', ['A', 'B']);

foreach ($stream as $zipped) {
    // [1, A], [2, B], [3, ?]
}
```

#### Zip Longest With
Параллельно итерирует элементы из потока вместе с элементами переданных коллекций, пока не закончится самый длинный итератор.

```$stream->zipLongestWith(iterable ...$iterables): Stream```

* Создает итератор, который агрегирует данные из нескольких итераторов.
* Работает аналогично функции `zip_longest()` в Python.
* Для коллекций разной длины продолжает процесс итерирования до момента, пока самая длинная коллекция не закончится.
* Для коллекций разной длины отдает вместо элементов `null` для коллекций, которые закончились.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->zipLongestWith([4, 5, 6])
    ->zipLongestWith([7, 8, 9, 10]);

foreach ($result as $item) {
    // [1, 4, 7], [2, 5, 8], [3, 6, 9], [4, null, 10], [null, null, 5]
}
```

#### Zip Equal With
Параллельно итерирует элементы из потока вместе с элементами переданных коллекций (все коллекции должны быть одной длины).

```$stream->zipEqualWith(iterable ...$iterables): Stream```

Работает как `Multi::zip()`, но бросает `\LengthException`, когда выясняется, что длины коллекций разные
(когда закончился самая короткая коллекция).

```php
use IterTools\Stream;

$input = [1, 2, 3];

$result = Stream::of($input)
    ->zipEqualWith([4, 5, 6])
    ->zipEqualWith([7, 8, 9]);

foreach ($result as $item) {
    // [1, 4, 7], [2, 5, 8], [3, 6, 9]
}
```

### Завершающие операции

#### Саммари о потоке
##### All Match
Возвращает истину, если для всех элементов из потока предикат возвращает истину.

```$stream->allMatch(callable $predicate): bool```

```php
use IterTools\Summary;

$finalFantasyNumbers = [4, 5, 6];
$isOnSuperNintendo   = fn ($ff) => $ff >= 4 && $ff <= 6;

$boolean = Stream::of($finalFantasyNumbers)
    ->allMatch($isOnSuperNintendo);
// true
```

##### All Unique
Возвращает истину, все элементы коллекции потока уникальны.

```$stream->allUnique(bool $strict = true): bool```

По умолчанию работает в [режиме строгой типизации](#Режимы-типизации). Установите параметр `$strict` в `false` для работы в режиме приведения типов.

```php
use IterTools\Summary;

$items = ['fingerprints', 'snowflakes', 'eyes', 'DNA']

$boolean = Stream::of($items)
    ->allUnique();
// true
```

##### Any Match
Возвращает истину, если хотя бы для одного элемента из потока предикат возвращает истину.

```$stream->anyMatch(callable $predicate): bool```

```php
use IterTools\Summary;

$answers          = ['fish', 'towel', 42, "don't panic"];
$isUltimateAnswer = fn ($a) => a == 42;

$boolean = Stream::of($answers)
    ->anyMatch($answers, $isUltimateAnswer);
// true
```

##### Are Permutations With
Возарщает истину, если коллекция из потока и переданные коллекции являются перестановками друг друга.

```$stream->arePermutationsWith(...$iterables): bool```
```php
use IterTools\Summary;
$rite = ['r', 'i', 't', 'e'];
$reit = ['r', 'e', 'i', 't'];
$tier = ['t', 'i', 'e', 'r'];
$tire = ['t', 'i', 'r', 'e'];
$trie = ['t', 'r', 'i', 'e'];
$boolean = Stream::of(['i', 't', 'e', 'r'])
    ->arePermutationsWith($rite, $reit, $tier, $tire, $trie);
// true
```

##### Are Permutations Coercive With
Возарщает истину, если коллекция из потока и переданные коллекции являются перестановками друг друга
(в режиме [приведения типов](#Режимы-типизации)).

```$stream->arePermutationsCoerciveWith(...$iterables): bool```

```php
use IterTools\Summary;
$set2 = [2.0, '1', 3];
$set3 = [3, 2, 1];
$boolean = Stream::of([1, 2.0, '3'])
    ->arePermutationsCoerciveWith($set2, $set3);
// true
```

##### Exactly N
Возвращает истину, если в точности для n элементов из потока предикат возвращает истину.

- Предикат является необязательным аргументом.
- По умолчанию (если не передан) предикат приводит элементы коллекции к `bool`.

```$stream->exactlyN(int $n, callable $predicate = null): bool```

```php
use IterTools\Summary;

$twoTruthsAndALie = [true, true, false];
$n                = 2;

$boolean = Stream::of($twoTruthsAndALie)->exactlyN($n);
// true
```

##### Is Empty
Возвращает истину, если коллекция потока пуста.

```$stream->isEmpty(): bool```

```php
use IterTools\Summary;

$numbers    = [0, 1, 2, 3, 4, 5];
$filterFunc = fn ($x) => $x > 10;

$boolean = Stream::($numbers)
    ->filter($filterFunc)
    ->isEmpty();
// true
```

##### Is Partitioned
Возвращает истину, если все истинные элементы коллеции из потока находятся в коллекции перед ложными
(истинность определяет предикат).

- Возвращает истину для пустой коллекции и для коллекции с одним элементом.
- Если предикат не был передан, истинность элемента получается через приведение его значения к булевому типу.

```$stream->isPartitioned(callable $predicate = null): bool```

```php
use IterTools\Summary;
$numbers          = [0, 2, 4, 1, 3, 5];
$evensBeforeOdds = fn ($item) => $item % 2 === 0;
$boolean = Stream::($numbers)
    ->isPartitioned($evensBeforeOdds);
// true
```

##### Is Sorted
Возвращает истину, если коллекция элементов из потока отсортирована в прямом порядке, иначе — ложь.

```$stream->isSorted(): bool```

Элементы должны быть сравнимы.

Для пустой коллекции или коллекции из одного элемента всегда возвращает истину.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->isSorted();
// true

$input = [1, 2, 3, 2, 1];

$result = Stream::of($input)
    ->isSorted();
// false
```

##### Is Reversed
Возвращает истину, если коллекция элементов из потока отсортирована в обратном порядке, иначе — ложь.

```$stream->isReversed(): bool```

Элементы должны быть сравнимы.

Для пустой коллекции или коллекции из одного элемента всегда возвращает истину.

```php
use IterTools\Stream;

$input = [5, 4, 3, 2, 1];

$result = Stream::of($input)
    ->isReversed();
// true

$input = [1, 2, 3, 2, 1];

$result = Stream::of($input)
    ->isReversed();
// false
```

##### None Match
Возвращает истину, если для всех элементов из потока предикат вернул ложь.

```$stream->noneMatch(callable $predicate): bool```

```php
use IterTools\Summary;

$grades         = [45, 50, 61, 0];
$isPassingGrade = fn ($grade) => $grade >= 70;

$boolean = Stream::of($grades)->noneMatch($isPassingGrade);
// true
```

##### Same With
Возвращает истину, если коллекция элементов из потока идентична переданным в аргументах коллекциям.

```$stream->sameWith(iterable ...$iterables): bool```

Если в метод не передать ни одной коллекции, он вернет истину.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->sameWith([1, 2, 3, 4, 5]);
// true

$result = Stream::of($input)
    ->sameWith([5, 4, 3, 2, 1]);
// false
```

##### Same Count With
Возвращает истину, если и коллекция элементов из потока, и все переданные коллекции имеют одинаковую длину.

```$stream->sameCountWith(iterable ...$iterables): bool```

Если в метод не передать ни одной коллекции, он вернет истину.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($input)
    ->sameCountWith([5, 4, 3, 2, 1]);
// true

$result = Stream::of($input)
    ->sameCountWith([1, 2, 3]);
// false
```

#### Редуцирование

##### To Average
Возвращает среднее арифметическое коллекции элементов из потока.

```$stream->toAverage(): mixed```

Для пустой коллекции вернет `null`.

```php
use IterTools\Stream;

$input = [2, 4, 6, 8];

$result = Stream::of($iterable)
    ->toAverage();
// 5
```

##### To Count
Возвращает длину коллекции элементов из потока.

```$stream->toCount(): mixed```

```php
use IterTools\Stream;

$input = [10, 20, 30, 40, 50];

$result = Stream::of($iterable)
    ->toCount();
// 5
```

##### To First
Возвращает первый элемент из коллекции в потоке.

```$stream->toFirst(): mixed```

Бросает `\LengthException` если хранимая в потоке коллекция пуста.

```php
use IterTools\Stream;

$input = [10, 20, 30];

$result = Stream::of($input)
    ->toFirst();
// 10
```

##### To First And Last
Возвращает первый и последний элементы из коллекции в потоке.

```$stream->toFirstAndLast(): array{mixed, mixed}```

Бросает `\LengthException` если хранимая в потоке коллекция пуста.

```php
use IterTools\Stream;

$input = [10, 20, 30];

$result = Stream::of($input)
    ->toFirstAndLast();
// [10, 30]
```

##### To Last
Возвращает последний элемент из коллекции в потоке.

```$stream->toLast(): mixed```

Бросает `\LengthException` если хранимая в потоке коллекция пуста.

```php
use IterTools\Stream;

$input = [10, 20, 30];

$result = Stream::of($input)
    ->toLast();
// 30
```

##### To Max
Возвращает максимальный элемент коллекции из потока.

```$stream->toMax(callable $compareBy = null): mixed```

- Функция `$compareBy` должна возвращать сравнимое значение.
- Если аргумент `$compareBy` не передан, элементы коллекции должны быть сравнимы.
- Для пустой коллекции вернет `null`.

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($iterable)
    ->toMax();
// 3
```

##### To Min
Возвращает минимальный элемент коллекции из потока.

```$stream->toMin(callable $compareBy = null): mixed```

- Функция `$compareBy` должна возвращать сравнимое значение.
- Если аргумент `$compareBy` не передан, элементы коллекции должны быть сравнимы.
- Для пустой коллекции вернет `null`.

```php
use IterTools\Stream;

$input = [1, -1, 2, -2, 3, -3];

$result = Stream::of($iterable)
    ->toMin();
// -3
```

##### To Min Max
Возвращает минимальный и максимальный элементы коллекции из потока.

```$stream->toMinMax(callable $compareBy = null): array```

- Функция `$compareBy` должна возвращать сравнимое значение.
- Если аргумент `$compareBy` не передан, элементы коллекции должны быть сравнимы.
- Для пустой коллекции вернет `[null, null]`.

```php
use IterTools\Stream;

$numbers = [1, 2, 3, -1, -2, -3];

[$min, $max] = Stream::of($numbers)
    ->toMinMax();
// [-3, 3]
```

##### To Nth
Возвращает n-й элемент потока.

```$stream->toNth(int $position): mixed```

Для пустой коллекции возвращает `null`.

```php
use IterTools\Stream;

$lotrMovies = ['The Fellowship of the Ring', 'The Two Towers', 'The Return of the King'];

$result = Stream::of($lotrMovies)
    ->toNth(2);
// The Return of the King
```

##### To Product
Возвращает произведение элементов коллекции из потока.

```$stream->toProduct(): mixed```

Для пустой коллекции вернет `null`.

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($iterable)
    ->toProduct();
// 120
```

##### To Random Value
Возвращает случайный элемент из коллекции потока.

```$stream->toRandomValue(): mixed```

```php
use IterTools\Stream;

$rpsHands = ['rock', 'paper', 'scissors']

$range = Stream::of($numbers)
    ->map('strtoupper')
    ->toRandomValue();
// e.g., rock
```

##### To Range
Возвращает разницу между максимальным и минимальным элементами коллекции из потока.

```$stream->toRange(): int|float```

Для пустой коллекции вернет `0`.

```php
use IterTools\Stream;

$grades = [100, 90, 80, 85, 95];

$range = Stream::of($numbers)
    ->toRange();
// 20
```

##### To String
Преобразует коллекцию из потока в строку, "склеивая" ее элементы.

* Значение необязательного аргумента `$separator` вставляется в качестве разделителя между элементами в строке.
* Значение необязательного аргумента `$prefix` вставляется в начало строки.
* Значение необязательного аргумента `$suffix` вставляется в конец строки.

```$stream->toString(string $separator = '', string $prefix = '', string $suffix = ''): string```

```php
use IterTools\Stream;

$words = ['IterTools', 'PHP', 'v1.0'];

$string = Stream::of($words)->toString($words);
// IterToolsPHPv1.0
$string = Stream::of($words)->toString($words, '-');
// IterTools-PHP-v1.0
$string = Stream::of($words)->toString($words, '-', 'Library: ');
// Library: IterTools-PHP-v1.0
$string = Stream::of($words)->toString($words, '-', 'Library: ', '!');
// Library: IterTools-PHP-v1.0!
```

##### To Sum
Возвращает сумму элементов коллекции из потока.

```$stream->toSum(): mixed```

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($iterable)
    ->toSum();
// 15
```

##### To Value
Редуцирует коллекцию из потока до значения, вычисляемого с использованием callback-функции.

В отличие от `array_reduce()`, работает с любыми `iterable` типами.

```$stream->toValue(callable $reducer, mixed $initialValue): mixed```

```php
use IterTools\Stream;

$input = [1, 2, 3, 4, 5];

$result = Stream::of($iterable)
    ->toValue(fn ($carry, $item) => $carry + $item);
// 15
```

#### Операции конвертации

##### To Array
Возвращает массив всех элементов из потока.

```$stream->toArray(): array```

```php
use IterTools\Stream;

$array = Stream::of([1, 1, 2, 2, 3, 4, 5])
    ->distinct()
    ->map(fn ($x) => $x**2)
    ->toArray();
// [1, 4, 9, 16, 25]
```

##### To Associative Array
Возвращает ассоциативный массив всех элементов из потока.

```$stream->toAssociativeArray(callable $keyFunc, callable $valueFunc): array```

```php
use IterTools\Stream;
$keyFunc
$array = Stream::of(['message 1', 'message 2', 'message 3'])
    ->map('strtoupper')
    ->toAssociativeArray(
        fn ($s) => \md5($s),
        fn ($s) => $s
    );
// [3b3f2272b3b904d342b2d0df2bf31ed4 => MESSAGE 1, 43638d919cfb8ea31979880f1a2bb146 => MESSAGE 2, ... ]
```

##### Tee
Создает несколько одинаковых независимых потоков из данного.

```$stream->tee(int $count): array```
```php
use IterTools\Transform;
$daysOfWeek = ['Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat', 'Sun'];
$count = 3;
[$week1Stream, $week2Stream, $week3Stream] = Stream::of($daysOfWeek)
    ->tee($count);
// Each $weekStream contains ['Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat', 'Sun']
```

#### Операции с побочными эффектами

##### Call For Each
Вызывает callback-функцию для каждого элемента из потока.

```$stream->callForEach(callable $function): void```

```php
use IterTools\Stream;

$languages = ['PHP', 'Python', 'Java', 'Go'];
$mascots   = ['elephant', 'snake', 'bean', 'gopher'];

$zipPrinter = fn ($zipped) => print("{$zipped[0]}'s mascot: {$zipped[1]}");

Stream::of($languages)
    ->zipWith($mascots)
    ->callForEach($zipPrinter);
// PHP's mascot: elephant
// Python's mascot: snake
// ...
```

##### Print
Вызывает `print()` для каждого элемента из потока.

* Элементы в потоке должны иметь строковое представление.

```$stream->print(string $separator = '', string $prefix = '', string $suffix = ''): void```

```php
use IterTools\Stream;

$words = ['IterTools', 'PHP', 'v1.0'];

Stream::of($words)->print();                       // IterToolsPHPv1.0
Stream::of($words)->print('-');                    // IterTools-PHP-v1.0
Stream::of($words)->print('-', 'Library: ');       // Library: IterTools-PHP-v1.0
Stream::of($words)->print('-', 'Library: ', '!');  // Library: IterTools-PHP-v1.0!
```

##### Print Line
Печатает элементы из потока каждый с новой строки.

* Элементы в потоке должны иметь строковое представление.

```$stream->println(): void```

```php
use IterTools\Stream;

$words = ['IterTools', 'PHP', 'v1.0'];

Stream::of($words)->printLn();
// IterTools
// PHP
// v1.0
```

##### To CSV File
Записывает содержимое потока в CSV файл.

```$stream->toCsvFile(resource $fileHandle, array $header = null, string 'separator = ',', string $enclosure = '"', string $escape = '\\'): void```

```php
use IterTools\Stream;
$starWarsMovies = [
    ['Star Wars: Episode IV – A New Hope', 'IV', 1977],
    ['Star Wars: Episode V – The Empire Strikes Back', 'V', 1980],
    ['Star Wars: Episode VI – Return of the Jedi', 'VI', 1983],
];
$header = ['title', 'episode', 'year'];
Stream::of($data)
    ->toCsvFile($fh, $header);
// title,episode,year
// "Star Wars: Episode IV – A New Hope",IV,1977
// "Star Wars: Episode V – The Empire Strikes Back",V,1980
// "Star Wars: Episode VI – Return of the Jedi",VI,1983
```

##### To File
Записывает содержимое потока в файл.

```$stream->toFile(resource $fileHandle, string $newLineSeparator = \PHP_EOL, string $header = null, string $footer = null): void```

```php
use IterTools\Stream;
$data = ['item1', 'item2', 'item3'];
$header = '<ul>';
$footer = '</ul>';
Stream::of($data)
    ->map(fn ($item) => "  <li>$item</li>")
    ->toFile($fh, \PHP_EOL, $header, $footer);
// <ul>
//   <li>item1</li>
//   <li>item2</li>
//   <li>item3</li>
// </ul>
```

### Операции для дебаггинга
#### Peek
Позволяет просмотреть каждый элемент между другими потоковыми операциями, чтобы выполнить какое-либо действие без влияния на поток.

```$stream->peek(callable $callback): void```

```php
use IterTools\Stream;

$logger = new SimpleLog\Logger('/tmp/log.txt', 'iterTools');

Stream::of(['some', 'items'])
  ->map('strtoupper')
  ->peek(fn ($x) => $logger->info($x))
  ->foreach($someComplexCallable);
```

#### Peek Stream
Позволяет просмотреть коллекцию потока между другими потоковыми операциями, чтобы выполнить какое-либо действие без влияния на поток.

```$stream->peekStream(callable $callback): Stream```

```php
use IterTools\Stream;

$logger = new SimpleLog\Logger('/tmp/log.txt', 'iterTools');

Stream::of(['some', 'items'])
  ->map('strtoupper')
  ->peekStream(fn ($stream) => $logger->info($stream))
  ->foreach($someComplexCallable);
```

#### Peek Print
Распечатывает каждый элемент хранимой коллекции в поток вывода между другими потоковыми операциями.

```$stream->peekPrint(string $separator = '', string $prefix = '', string $suffix = ''): void```

```php
use IterTools\Stream;

Stream::of(['some', 'items'])
  ->map('strtoupper')
  ->peekPrint()
  ->foreach($someComplexCallable);
```

#### Peek PrintR
Вызывает `print_r()` для каждого элемента хранимой коллекции между другими потоковыми операциями.

```$stream->peekPrintR(callable $callback): Stream```

```php
use IterTools\Stream;

Stream::of(['some', 'items'])
  ->map('strtoupper')
  ->peekPrintR()
  ->foreach($someComplexCallable);
```

##### Print R
Вызывает `print_r()` для каждого элемента из потока.

```$stream->printR(): void```

```php
use IterTools\Stream;

$items = [$string, $array, $object];

Stream::of($words)->printR();
// print_r output
```

##### Var Dump
Вызывает `var_dump()` для каждого элемента из потока.

```$stream->varDump(): void```

```php
use IterTools\Stream;

$items = [$string, $array, $object];

Stream::of($words)->varDump();
// var_dump output
```

## Композиция вызовов
IterTools позволяет комбинировать вызовы методов, чтобы получать новые коллекции.

#### Zip Strings
```php
use IterTools\Multi;
use IterTools\Single;

$letters = 'ABCDEFGHI';
$numbers = '123456789';

foreach (Multi::zip(Single::string($letters), Single::string($numbers)) as [$letter, $number]) {
     $battleshipMove = new BattleshipMove($letter, $number)
}
// A1, B2, C3
```

#### Chain Strings
```php
use IterTools\Multi;
use IterTools\Single;

$letters = 'abc';
$numbers = '123';

foreach (Multi::chain(Single::string($letters), Single::string($numbers)) as $character) {
    print($character);
}
// a, b, c, 1, 2, 3
```

## Режимы типизации

Для методов, которые используют сравнение элементов коллекций для получения результата,
по умолчанию сравнения выполняются строго без приведения типов:

* scalars: сравнивает строго по типу;
* objects: всегда считает разные экземпляры неравными;
* arrays: сравнивает сериализованными.

В случае, если метод опционально поддерживает режим приведения типов (имеет аргумент `$strict`) при `$strict` установленном в `false`
либо если метод имеет в названии слово `Coercive`, он будет работать в нестрогом режиме сравнения:

* scalars: сравнивает нестрого по значению;
* objects: сравнивает сериализованными;
* arrays: сравнивает сериализованными.

Стандарты
---------

IterTools PHP соответствует следующим стандартам:

* PSR-1  - Basic coding standard (http://www.php-fig.org/psr/psr-1/)
* PSR-4  - Autoloader (http://www.php-fig.org/psr/psr-4/)
* PSR-12 - Extended coding style guide (http://www.php-fig.org/psr/psr-12/)

Лицензия
-------

IterTools PHP распространяется по лицензии MIT License.
