> Disclaimer: многие примеры перепечатаны или внаглую скопипасчены [с этого дока](https://github.com/jagajaga/FP-Course-ITMO). Я не претендую на их оригинальность


# Lecture 1: системы сборки stack, cabal и всякая база по языку

Есть stack, есть cabal -- сборщики проектов. Интерактивная среда ghci -- позволяет ручками потыкать-повычислять значения выражений.

Из важного -- все вычисления ленивые, а функции стараются писать чистыми. Есть функции, а есть операторы. Функции пишутся в префиксном виде, а операторы -- в инфиксном, однако их тоже можно записать префиксно, если окружить скобками:
```
ghci> 1 + 2
3
ghci> (+) 1 2
3
```
Чтобы в ghci узнать тип выражения можно написать:
```
ghci> :t (&&)
Bool -> Bool -> Bool
```
Либо ебануть где-то в коде ```ghci> :set +t``` и тогда типы будут выводиться для всех выражений при выводе на консоль.

Чтобы задать свой оператор нужно укзаать ему ассоциативность и приоритет. Ассоциативности есть следующие: infixl, infixr, infix. Приоритеты -- целые числа из диапазона [0, 10). 10 не включительно, поскольку можно думать о пробеле как о операторе применения функции, у которого наивысший приоритет. А ассоциативность нужна, чтобы компилятор понимал как ему расставлять скобки, в случае выражений с повторяющимся одним и тем же опреатором (infix -- не скомпилируется, попросить ручками расставить). Пример:
```
infixr 2 ||
(||) :: Bool -> Bool -> Bool
```
Есть листы -- хранятся как пара head:tail, синтаксис для создания -- [...] или через конструктор :. Могуть быть бесконечными, могут быть аналогом ranges. Не стоит использовать листы как массивы, т.е. обращаться по индексу к элементу, но если очень хочется -- есть оператор !!. Также есть нетотальные функции над листами (т.е. функции, которые падают на некоторых значениях своей области определений) -- head, tail, init, last. Помимо них есть куча других, по типу drop n, take n, replicate n obj, zip list list, unzip listOfPairs. Базовая строка в Хаскелле -- это лист чаров, ебать какой неэффективный, но он вот есть.

В Хаскелле нет стейтментсов, есть только экспрешены (банально, в С++ if это стейтмент). Для удобной работы есть следующие приколы: let <bindings> in <expression>, пример:
```
pythagoras :: Int -> Int -> Int
pythagoras :: x y = x ^ 2 + y ^ 2

pythagoras :: Int -> Int -> Int
pythagoras x y = let x2 = x ^ 2
                     y2 = y ^ 2
                 in x2 + y2
```
Пробелы в Хаскелле важны (привет, питон). Еще есть where <clause>:
```
pythagoras :: Int -> Int -> Int
pythagoras x y = x2 + y2
    where
        square x = x ^ 2
        x2       = square x
        y2       = square y
```
Они чуть разные с let, но в чем именно -- надо сходить почитать. Мне лично кажется, что let круче.
If есть, он по сути -- теранрый оператор: if <predicate> then <expression if predicate is True> else <expression if predicate is False>. Пример:
```
factorial :: Int -> Int
factorial n = if n == 0
              then 1
              else n * factorial (n - 1)
```
Есть guards, чтобы не плодить цепочку else-if:
```
collatzSum :: Natural -> Natural
collatzSum n
    | n == 1        = 1
    | even n        = n + collatzSum(div n 2)
    | otherwise     = n + collatzSum(3 * n + 1)
```
Здесь otherwise -- синоним True.
Еще есть switch-case, он основан на паттерн-матчинге, об этом будет позже, работает так:
```
getFont :: Int -> String
getFont n = case n of
                 0 -> "plain"
                 1 -> "bold"
                 2 -> "italic"
                 _ -> "unknown"
```
В нем так же только экспрешены, никаких стейтментсов

# Lecture 2: про функции как про типы, полиморфизм и как с этим быть
В Хаскелле функции -- это объекты первого класса, в них можно передовать как числа, так и функции, им можно присваивать другие функции и константы и норм живется так. Пример:
```
inc, dec :: Int -> Int
inc n = n + 1
dec n = n - 1

changeTwiceBy (Int -> Int) -> Int -> Int
changeTwiceBy operation value = operation (operation value)
```
Есть лямбды -- синтаксис как у Штукена был:
```
> changeTwiceBy (\x -> x + 1) 2
4
```
Про полиморфизм. Есть параметрический (по типу) и ad-hoc полиморфизм (он про то, что разные объекты ведут себя по разному). Можно думать как о дженриках и интерфейсах оответственно.

Про параметрический полиморфизм -- когда не важной какой тип, его можно написать просто с маленькой буквы, и на этапе компиляции он задедюсится, вроде и так понятно как это работает. Есть map, filter, foldr1, zipWith и прикольные uncurry и curry:
```
uncurry :: (a -> b -> c) -> (a, b) -> c
```

Карри тут не просто так, все функции в Хаскелле на самом деле каррированны, т.е. представляют из себя функцию от одного аргумента, что позволяет делать частичное применение их. Самый разъеб -- функция ```flip :: (a -> b -> c) -> b -> a -> c```. С ней можно, например, маппить один и тот же список разными функциями, как бы зафиксировав его.

На подумать, я наверное пока не хочу, м.б. апдейтну потом:
```
ghci> :t flip id
???
```

## Про паттерн-матчинг
Ну типо матчится по патеррнам, что такое паттерны подронее будет потом, а пока для наглядности вот: (upd. корректнее думать о паттернах как о конструкторах)
```
fact :: Integer -> Integer
fact 0 = 1
fact n = n * fact (n - 1)

stringList :: String -> String
stringList "such" = "pattern"
stringList "much" = "amaze"
stringList _      = "wow"


sumList3 :: [Int] -> Int
sumList3 [x, y, z] = x + y + z
sumList3 _         = 0

map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (x:xs) = f x : map f xs

dropWhile :: (a -> Bool) -> [a] -> [a]
dropWhile _ [] = []
dropWhile p list@(x:xs) = if p x then dropWhile p xs else list
```
Тут же есть тема под название алиас, это name@(expresion) чтобы удобно обращаться к каким-то элементам паттерна в определении

## Про Extensions
Есть куча безобидных расширений синтаксиса языка, с которыми просто удобнее жить. Здесь будет несколько примером, а подключить их можно так:
```
ghci> :set +X<NameOfPragma>

module.hs
{-# LANGUAGE: NameOfPragma #-}
```

**-XTupleSelections**
Позволяет частично применять конструкторы пары:
```
ghci> map (\x -> (x,42)) [1..5]
[(1, 42), (2, 42), (3, 42), (4, 42), (5, 42)]

ghci> map (,42) [1..5]
[(1, 42), (2, 42), (3, 42), (4, 42), (5, 42)]
```

**-XLambaCase**
Чтобы писать паттерн матчинг без копирования длинного названия много раз
```
veryLongFunctionName :: Int -> String
veryLongFunctionName 0 = foo
veryLongFunctionName 1 = bar
veryLongFunctionName n = bax n

veryLongFunctionName :: Int -> String
veryLongFunctionName = \case
    0 -> foo
    1 -> bar
    n -> baz n
```

**-XViewPatterns**
Чтобы паттерн-матчися по результаты применения функции
```
exactTwoWords :: String -> Bool
exactTwoWords s = case words s of 
    [_, _] -> True
    _ -> False
    
exactTwoWords :: String -> Bool
exactTwoWords (words -> [_, _]) = True
exactTwoWords _                 = False
```

## Про применение функций
Из-за того что о пробеле можно думать как об операторе применения функции, получается, что бывает просто дохуище скобок в одном выражении и читать становится сложно. Чуваки придумали оператор применения функции с минимальным приоритетом, который позволяет сначала посчитать правую часть, а потом применить на нее функцию из левой части. Мем? Мем.
```
infixr 0 $
($) :: (a -> b) -> a -> b
f $ x = f x

ghci> length $ [0..5]
6
ghci> length [0..5]
6 

Самый разъеб:
foo, bar :: [Int] -> Int
foo list = length (filter odd (map (div 2) (filter even (map (div 7) list))))
bar list = length $ filter odd $ map (div 2) $ filter even $ map (div 7) list

foo ~ bar :)
```

Есть аналог джавовского применения функции к объекту -- флипнутый оператор применения функции с приоритетом 1:
```
infixl 1 &
(&) :: a -> (a -> b) -> b
x & f = f x

foo, bar, baz :: [Int] -> Int
foo list = length (filter odd (map (div 2) (filter even (map (div 7) list))))
bar list = length $ filter odd $ map (div 2) $ filter even $ map (div 7) list
baz list = list & map (div 7)
                & filter even
                & map (div 2)
                & filter odd
                & length
                
foo ~ bar ~ baz :))
```

## Про композицию функций
```
infixr 9 .
(.) :: (b -> c) -> (a -> b) -> (a -> c) 
f . g = \x -> f (g x)
```
Варианты записать композицию:
```
"outside . inside" is a composition:
1. \x -> outsude (inside x)
2. \x -> outside $ inside x
3. \x -> outside . inside $ x
4.       outside . inside
```
Можно приводить применения множества функций этта-редукцией к композиции без использования аргумета. Это называется 'бесточечный стиль', но он с точками -- как можно заметить, всем примерно похуй. Пример:
```
incNegate :: Int -> Int
incNegate x = negate (x + 1)
incNegate x = negate $ x + 1
incNegate x = (negate . (+ 1)) x
incNegate x = negate . (+ 1) $ x
incNegate = negate . (+ 1)
```

Еще пример, где можно обойтись без имени аргумента и записат функцию в терминах композиции функций:
```
foo, bar :: [Int] -> Int
foo list = length $ filter odd $ map (div 2) $ filter even $ map (div 7) list
bar      = length . folter odd . map (div 2) . filter even . map (div 7)

foo ~ bar :)))
```

## Про создание списков
```
ghci> [x | x <- [1..10], even x]
[2, 4, 6, 8, 10]

ghci> filter even [1..10]
[2, 4, 6, 8, 10]

ghci> [if even x then "!" else "?" | x <- [1..5]]
"?!?!?"

ghci> [ x * y | x <- [1, 3, 5], y <- [2, 4, 6], x * y >= 10 ] -- аналог фора за квадрат
[12, 18, 10, 20, 30]
```
Через эту штуку можно квик сорт в одну строку написать даже :)

## Про ленивые вычисления в Хаскелле
Ну там граф вычислений, такой, что каждое выражение будет вычисленно ровно один раз и тогда, когда оно впервые понадобится

Опр.: Выражение находится **нормальной форме (NF)**, если оно не имеет невычисленных подвыражений или оно является лямбдой.

Опр.: Выражение находится в **слабой головной нормальной форме (WHNF)**, если оно является либо конструктором либо лямбда. Про конструкторы подробно будет потом

Это важно потому что при паттерн-матчинге происходит вычисление до WHNF!! А все что не находится в WHNF называется **thunk**. 

Есть утверждение, что ленивая модель вычислений не медленнее строгой модели выислений. Однако с памятью там беды. Например, выражение ((((0 + 1) + 2) + 3) + 4) будет прям так невычисленным лежать в памяти и занимать дохуя места. Это называется **space leak**. Зафорсировать вычисление вражения до WHNF можно простым паттерн-матчингом. Но как-то чуть сложнее можно и до NF зафорсировать.

## TODO: прочитать про Grokking fix

## База ленивых вычислений
Решето Эратосфена:
```
primes :: [Int]
primes = filterPrime[2..]
    where
        filterPrime (p:xs) = p : filterPrime[ x | x <- xs, mod x p /= 0 ]
```
Числа Фибоначи (вообще балдеж):
```
fibs :: [Int]
fibs = 0 : 1 : zipWith (+) fibs (drop 1 fibs)
```
# Lecture 3. Про теорию типов и лямбда исчисление

Есть такая парадигма -- **Матеарилизм Аристотиля**, она достаточно старая, но плотно сидит в нашем мозгу. Ее суть: всё в мире материя, которая имеет атрибуты, материя, как и атрибуты, может быть первичной и вторичной. Например: машина и ее цвет это первияная материя, машина как машина -- вторичная, причем на вторичной материи есть какой-то строгий порядок, например, машина -- инстанс VEHICLE. Аналогично с атрибутами: Primary: RED, Secondary: Color.

Эта штука позволяет как-то базово описовать реальность, что мы часто делаем, когда пишем какой-то код. Но в реальном мире сложно иерархично описать достаточно объемные явления. 

Поэтому есть **Logic Paradigm** -- естественное развитии этой теории. Каждый объект принадлежит какому-то множеству, на некоторых множествах есть отношение включения как подмножества. Так примерно и появилась теория множеств, знакомая из логики. Но там есть некоторые парадоксы, это не круто. А хочется иметь универсальный математический инструментарий, вот это круто. Поэтому в основе Хаскелля лежит что-то типо смеси System F и системы Хинди-Миллера.

## Что такое вычисление?

**Модель вычисления** -- набор базовых операций + из стоимости, примеры:
1. Машина Тьюринга
2. Стековая машина
3. Конечные автоматы
4. Лямбда-исчисление
5. Любая машина на архитектуре Фон-Неймана
6. RAM
7. Счетчиковые машины
8. Prolog
9. Transition systems (?)
10. Petri nets (?)
11. Рекурсивные функции

Двигатель лябмда-исчисления -- переписывание строчек (simple semantics) -- по сути бетта-редукция. Оно Тьюринг полное. для этого нужно выразить numbers, if then else, while => будут рекурсивные функции

## Про лямбда-исчисление

Это все про абстракные функции, стоит воспринимать их как черные ящики с которыми мы можем делать только:
1. создавать новые функции над нашей
2. делать применение функции над аргументом

Формально, ```M, N ::= x | (\x.M) | (MN)```. Это называется **preterms**, потому что **terms** -- это претермы с классами эквивалентности над ними, то есть можно переименовывать только связные переменные, условно.

**alpha-преобразование** -- корректно переименовать аргумент во что-то другое. 

**beta-редукция** -- отношение, которым связанны термы, полученные один из другого путем переписывания и раскрытия операции применения. Вообще, есть впоросы про Тьюринг-полноту того, что сейчас было введено: например, вот это -- ```(\x.x x)(\x.x x) ->b (\x.x x)(\x.x x)``` -- и дургие примеры, где выражание бесконечно может редуцироваться (в себя или в цикл). И еще вот такой пример, который можно редуцировать по внешней скобке и получить конечную редукцию: ```(\x.y)((\x.x x)(\x.x x)) ->b y```. Значит, нам важен порядок редукции. Если редукция закончилась за конечное число шагов, то ее результат называется **нормальной формой терма**.

**Слабая нормальная форма терма** -- если КАКОЙ-ТО порядок редуцирования приводит терм к нормальной форме.
**Сильная нормальная форма терма** -- если ЛЮБОЙ порядок редуцирования приводит терм к нормальной форме.

## Про стратегии редукций

1. **Аппликативная (call-by-value)**. Ну как в императивных языках -- для применения функции сначала нужно посчитать ее аргументы. По сути закапываемся в самую глубь AST, и оттуда редуцируем
2. **Нормальная (call-by-name)**. Такое уже не прокатит в императивных языках -- для применения функции к аргументам подставим вместо ее аргументов то, что нужно вычислить для рассчета значения функции, но НЕ БУДЕМ ИНПЛЭЙС ЭТО ВЫЧИСЛЯТЬ, просто подставим. Тогда получится, что условное f(g(x), g(x)) может дважды вычислить g(x) внутри себя где-то. Подход крутой, но пересчитывать одно и то же не хочется

Есть тезис, что любой терм, который может быть приведен в слабую нормальную форму, может быть привен в нее и с помощью нормального порядка редукции.

Поэтому в Хаскелле используется некоторый гибрид -- нормальные порядок вычисления с ленивыми вычисления арументов -- позволяет не пересчитывать дважды значения.

## Про чистые функции

Знаем, что чистые функции -- это функции без сайд-эффектов. А что такое сайд-эффекты? Можно думать, что это любые эффекты функции, кроме значений, которые она возвращает. Чистые функции хороши. Например, их легко тестировать и на одинаковых значениях аргументов они ведут себя одинаково. Однако, сайд-эффекты очень полезная вещь. Потому что ну а как без этого жить. ПОэтому можно думать, что функции в ЯП -- это что-то что принимает аргументы и контекст на вход и выплевывет результат и список событий.

В Хаскелле стараются писать чистые функции, вынося все сайд-эффекты в отдельные функции с маркером не-чистоты.

## Про типы

Вспомним, что до сих пор пиздец вида ```(\x.x x)(\x.x x)``` для нас ок. Но на самом деле вообще не ок. Чтобы такого избежать -- то есть ограничить области применения функций -- как раз и нужны типы. 

Важно понимать разницу между динамически-типизированными языками и статически-типизированными языками с выводом типов. По-моему все очевидно, но если что в первом случае -- переменная может иметь любой тип, а во втором его можно просто не написать, он будет выведен компилятором и будет далее тайп-чекаться из предположения, то компилятор прав.

В типизированном лямбда-исчислении функция является **функцией высшего порядка**, если она имеет некоторый тип A -> B, где A ~ C -> D (по сути думаем о функции как о переменной, не в первый раз такое).

Есть три оси движения завистимостей между типами и термами:
1. Терм зависит от типа
2. Тип зависит от типа
3. Тип зависит от терма

Какой-то чел придумал на этом лямбда-куб. Буквально там куб, благодаря нему можно увидеть, что самые развитые типовые структуры (вершина (1, 1, 1)) -- языки Coq, Agda и др. А Хаскелль в этот куб сложно засунуть, т.к. там есть расширения, которые невозможно предтавить в рамках этих осей (например, Type-classes)

## Про полиморфизм

Желание писать более общий код, засовывая в параметры какие-то около-произвольные типы, которые будут абстрактными. То есть от них важно тчобы они просто были, мы не будем напрямую оперировать ими.



# Lecture 4. Про типы данных, алгебраические типы данных, синонимы и Ad-hoc полиморфизм

Хотим иметь тип пользователя, например, это uid :: Int, login :: String, name :: String. Просто таскать за собой тапл из них -- так себе идея, понятно почему, какие есть варианты реализовать это?

## Type aliases (синонимы типов)
Похоже на юзинги в плюсах, не дают представления о том, как внутри устроен тип примеры:
```
type User = (Int, String, String)
userFullId :: User -> String
userFullId (uid, login, name) -> show uid ++ ":" ++ login
```
Можно так же параметризовывать:
```
type BunaryIntFunction = Int -> Int -> Int
type String            = [Char]
type FilePath          = String
type TripleList a      = [(a, a, a)]
type SwapPair a b      = (a, b) -> (b, a)
```

## Про алгебраические типы данных (ADT)

Конъюнкция, дизъюнкция :)

### Произведение типов
```
PT = T_1 * T_2 * ... * T_n
```
Чиать так:
> Для типа PT существует экземплярный элеменет этого класса тогда и только тогда, когда существует экземплярный элемент для класса T_1 И экземплярный элемент для класса T_2 И ... И существует экзеплярный элемент для класса T_n. 
Аналогичное определение можно дать в терменнах **насеенности ножеств**. Множество населено титт, когда оно не пусто. Следовательно, тип населен, если существует хотя бы один элемент этого типа. Тогда:

> Тип PT населен титт, когда населен тип Т_1 И населен тип Т_2 И ... И населен тип T_n.

Похоже на классы в императивных ЯП:
```
struct user {
    int uid;
    string login;
    string name;
};

=> user = int * string * string
```

### Сумма типов 
Ну тут аналогично:
```
ST = T_1 + T_2 + ... + T_n
```
> Для типа ST существует экземплярный элеменет этого класса тогда и только тогда, когда существует ЛИБО экземплярный элемент для класса T_1 ЛИБО экземплярный элемент для класса T_2 ЛИБО ... ЛИБО существует экзеплярный элемент для класса T_n. 

> Тип ST населен титт, когда ЛИБО населен тип Т_1 ЛИБО населен тип Т_2 ЛИБО ... ЛИБО населен тип T_n.

Это, очевидно, аналог std::variant<...>

### ADT
Формально теперь:
```
PrimitiveType ::= Int | Char | Double | ...
ADT ::= PrimitiveType | ADT + ADT | ADT * ADT
```

Пока этого достаточно, далее примеры:
\1. Енум
```
data TrafficLight = Red | Yellow | Green
-- здесь Red, Yellow, Green -- конструкторы типа TrafficLight
-- чуть подробнее далее, но уже знаем, что по ним можно паттерн-матчиться:

lightName :: TrafficLight -> String 
lightName Red    = "red"
lightName Yellow = "yellow"
lightName Green  = "green"
```
\2. Структурка -- полноценный тип данных
```
data User = MkUser Int String String
```
Здесь важна терминология:
    * MkUser -- конструктор данных -- в телах констант
    * Int String String -- поля типа
    * User -- конструктор типа (имя типа) -- в сигнатурах
```
getUid :: User -> Int -- селектор
getUid (MkUser uid _ _) = uid

getName :: User -> String
getName (MkUser _ _ name) = name

ghci> :t MkUser
MkUser :: Int -> String -> String -> User
```
! У конструктора всегда возращаемый тип в точности равен тому типу, в котором этот конструктор задан
\3. ADT -- можем параметризировать
```
data Point2D a = Point2D a a

point2List :: Point2D a -> [a]
point2List (Point2D x y) = [x, y]

doublePoint :: Point2D a -> Point2D (a, a)
doublePoint (Point2D x y) = Point2d (x, y) (x, y) 

maxCoord :: Point2D Int -> Int
maxCoord (Point2D x y) = max x y

distFromZero :: Point2D Double -> Double
distFromZero (Point2D x y) = sqrt (x ^ 2 + y ^ 2)
```
\4. ADT -- сумма типов
```
data IntResult = Success Int
               | Failure String
               
ghci> :t Success
Success :: Int -> IntResult -- аналогично ремарке (!) выше
ghci> :t Failure
Failure :: String -> IntResult -- аналогично ремарке (!) выше

safeDiv :: Int -> Int -> IntResult
safeDiv _ 0 = Failure "division by zero :("
safeDiv x y = Success $ div x y

-- обратно:
showResult :: IntResult -> String
showResult (Success value) = "Result: " ++ show value
showResult (Failure err) = "Error: " ++ err
```
\5. ADT -- параметризованная сумма типов
```
data Vector a = Vector2D a a | Vector3D a a a

packVector :: Vector a -> [a]
packVector (Vector2D x y)   = [x, y]
packVector (Vector3D x y z) = [x, y, z]

vecLen :: Vector Double -> Double
vecLen = sqrt . sum . map (^2) . packVector
```
\6. Maybe
```
data Maybe a = Nothing | Just a
-- здесь слово Maybe еще называется _контекстом_, в который Just оборачивает значение

maybeSecond :: [a] -> Maybe a
maybeSecond (_:x:_) = Just x
maybeSecond _       = Nothing
```
\7. Either
```
data Either a b = Left a | Right b -- порядок важен, об этом в lectures 6-7

eitherSecond :: [a] -> Either String a
eitherSecond []      = Left "list is empty"
eitherSecond [_]     = Left "only one item in list"
eitherSecond (_:x:_) = Right x
```
\8. List -- ADT recoursive
```
data List a = Nil | Cons a (List a)
-- здесь List -- рекурсивный тип, потому что один из его конструкторов, а именно Cons
-- в качестве своего поля принимает значения типа List, в котором Cons и определен

ghci> :t Nil
Nil :: List a
ghci> :t Cons
Cons :: a -> List a -> List a 
-- :)

myList :: List Int
myList = Cons 2 $ Cons 3 $ Cons 1 Nil

myMap :: (a -> b) -> List a -> List b
myMap _ Nil         = Nil
myMap f (Cons x xs) = Cons (f x) (myMap f xs)
```
На самом деле листы более удобные -- заметим, что конструкторы это просто infixr функции, а значит можно сделать их и infix путем представления их через операторы :))
```
data [] = [] | a : [a]
```
Тут ```[]``` -- спецсимволы, зарезервированные под списки, а кастомные операторы с текстом должны начинаться с ```:``` -- тут это просто единственный символ имени оператора.

## Про Record Syntax

Бэк ту пример с юзером. Хотим как-то явно указывать, какое из полей произведения типов что означает. Есть вот такая штука -- рекорд (обертка):
```
data User = User
    { uid   :: Int
    , login :: String
    , name  :: String
    }
```
Это сахар над этим, а сами uid, name, login это НЕ ПОЛЯ, А так называемые СЕЛЕКТОРЫ или АКСЕССОРЫ или ГЕТТЕРЫ, но лучше называть селекторами -- И ЭТО ФУНКЦИИ:
```
data User = User Int String String 

uid :: User -> Int
uid (User x _ _) = x

login :: User -> String
login (User _ x _) -> x

name :: User -> String
name (User _ _ x) -> x
```
Как этим пользоваться:
```
ivan = User { login = "ivan"
            , name = "ivan"
            , uid = 47
            }
            
isIvan :: User -> Bool
isIvan user = login user == "ivan" -- даже не надо паттерн-матчится, т.к. есть селектор :)
```

## Record Patterns & Updates
Можно задать isIvan через паттерн-матчинг:
```
isIvan User { login == username } = username == "ivan"
-- OR --
isIvan User { login == "ivan" } = True
isIvan _                        = False

пример из будущего:
data Person
    = User { uid :: Int, login :: String }
    | Admin { aid :: Int, login :: String }

isAdmin :: Person -> Bool
isAdmin Admin{} = True
isAdmin _       = False

```

Апдейт (с копированием!) конкретных полей:
```
cloneIvan = ivan { login = "cloneIvan" } -- User{47, "cloneIvan", "ivan"}
```

## Operator record fields (мем)
```
ghci> data R = { (-->) :: Int -> Int }
ghci> let r = R { (-->) = (+1) }
ghci> r --> 47
48
```

## Records and sum types 
Нужно быть аккуратным с нетотальными селекторами (по сути функциями):
```
data Person
    = User { uid :: Int, login :: String }
    | Admin { aid :: Int, login :: String }

login :: Person -> String
login (User _ l) = l
login (Admin _ l) = l

ghci> uid $ Admin 1 "Bob"
*** Exception: No match in record selector uid
```
  
  
  
# Lecture 5. Всё о том же

Поняли что совмещать рекорд-синтаксис с суммой типов не круто, потому что могкть быть нетотальные селекторы. А что тогда делать?
```
data Man = Man { name :: String }
data Cat = Cat { name :: String }

ghci> :t name
???
```
В ванильном Хаскелле так нельзя, но можно дописать суффикс типа и проблемы не будет, либо заюзать расширение -XDuplicateRecordFields:
```
{-# LANGUAGE DuplicateRecordFields #-}

data Man = Man { name :: String }
data Cat = Cat { name :: String }

shoutOnHumanBeing :: Man -> String
shoutOnHumanBeing man = (name :: Man -> String) man ++ "!!1!"  -- though...

isGrumpy :: Cat -> Bool
isGrumpy Cat{ name = "Grumpy" } = True
isGrumpy _                      = False
```

## RecordWildCards

Помним, что 'мемберы' типа это селекторы, но с рассширением -XRecordWildCards можно чтобы помимо селекторов создались буквально поля с такими же именами, нужно указать ```..``` для этого:
```
{-# LANGUAGE RecordWildCards #-}

data User = User 
    { uid      :: Int
    , login    :: String
    , password :: String 
    } deriving (Show)

toUnsafeString :: User -> String
toUnsafeString User{ uid = 0, .. } = "ROOT: " ++ login ++ ", " ++ password
toUnsafeString User{..}            = login ++ ":" ++ password
```
Можно написать такую хуйню, она будет работать по совпадению имен и типов полей работать:
```
evilMagic :: Man -> Cat
evilMagic Man{..} = Cat{..}

ghci> evilMagic $ Man "Grumpy"
Cat {name = "Grumpy"}
```

## newtype 

Этот кейворд можно использовать тольо в контексте data, если выполнено, что тип имеет ровно один конструктор, который принимает в себя ровно один аргумент -- по сути просто обертка над типом:
```
data    Message = Message String
newtype Message = Message String
```

## Type classes -- реализация Ad-hoc полиморфизма

Эд-хок полиморфизм -- это как интерфейсы в джаве. ПО сути разница такая: при параметрическом полиморфизме функция ведет себя одинаково для экземпляра  любого типа, а здесь -- поведение определяется тем, какого конкретна класса инстанс был передан. Этот механизм реализуют **классы типов**:
```
class Printable p where  -- we don't care what 'p' stores internally
    printMe :: p -> String
```

Важно: **data** для того чтобы определить, что храним; **class** чтобы определить, что делаем с хранящимися данными.
Их можно связать через кейворд **instance**:
```
data Foo = Foo | Bar  -- don't care what we can do with 'Foo', care what it stores

instance Printable Foo where
    printMe Foo = "Foo"
    printMe Bar = "Bar (whatever)"
```
Теперь эти можно пользоваться как констрейнтами на функции:
```
helloP :: Printable p => p -> String
helloP p = "Hello, " ++ printMe p ++ "!"
```

Базовые хайповые классы типов:
\1. Eq
```
class Eq a where  
    (==) :: a -> a -> Bool  
    (/=) :: a -> a -> Bool
 
    x == y = not (x /= y)  
    x /= y = not (x == y)
    {-# MINIMAL (==) | (/=) #-}  -- minimal complete definition
```
Тут прикол в том, что иногда не хочется переопределять кучу однотипных операторов и можно задавать их уловно-рекурсивно, указав минимальную прагму -- множество операторов, которые нужно явно задать в инстансе, чтобы финальный инстанс бл полностью собран. Например, c прагмой -XInstanceSigs млжно еще и явно указать сигнатуру операторов в инстансе:
```
{-# LANGUAGE InstanceSigs #-}

data TrafficLight = Red | Yellow | Green

instance Eq TrafficLight where
    (==) :: TrafficLight -> TrafficLight -> Bool
    Red    == Red    = True  
    Green  == Green  = True  
    Yellow == Yellow = True 
         _ == _      = False
```
Или вот еще:
```
threeSame :: Eq a => a -> a -> a -> Bool
threeSame x y z = x == y && y == z

ghci> threeSame Red Red Red
True
ghci> threeSame 'a' 'b' 'b'
False
```
\2. Ord
Ну понятно что это вроде
```
data Ordering = LT | EQ | GT

-- simplified version of Ord class
class Eq a => Ord a where
   compare              :: a -> a -> Ordering
   (<), (<=), (>=), (>) :: a -> a -> Bool

   compare x y
        | x == y    =  EQ
        | x <= y    =  LT
        | otherwise =  GT

   x <= y           =  compare x y /= GT
   x <  y           =  compare x y == LT
   x >= y           =  compare x y /= LT
   x >  y           =  compare x y == GT
   
   {-# MINIMAL compare | (<=) #-}
```

\3. Num
Числа с арифметкой по сути
```
-- | Basic numeric class.
class Num a where
    {-# MINIMAL (+), (*), abs, signum, fromInteger, (negate | (-)) #-}

    (+), (-), (*)       :: a -> a -> a  -- self-explained
    negate              :: a -> a       -- unary negation
    abs                 :: a -> a       -- absolute value
    signum              :: a -> a       -- sign of number, abs x * signum x == x
    fromInteger         :: Integer -> a -- used for numeral literals polymorphism

    x - y               = x + negate y
    negate x            = 0 - x
```
Тут можно написать 0 потому что это сахар над ```fromIntager 0```
Примеры:
```
ghci> :t 5
5 :: Num p => p
ghci> :t fromInteger 5
fromInteger 5 :: Num a => a
ghci> 5 :: Int
5
ghci> 5 :: Double
5.0
```
\4. Show
```
-- simplified version; used for converting things into String
class Show a where
    show :: a -> String
```
Примеры:
```
ghci> 5
5
ghci> show 5
"5"
ghci> "5"
"5"
ghci> show "5"
"\"5\""
```
У всех Num свои show определены:
```
ghci> 5 :: Int
5
ghci> 5 :: Double
5.0
ghci> 5 :: Rational
5 % 1

```

\5. Read -- дуальный к show
```
-- simplified version; used for parsing thigs from String
class Read a where
    read :: String -> a
```
По сути -- попытка распарсить строку в конструктор данных, но нужно быть аккуратным -- нужно явно указывать тип того, что ты ждешь (пример 3), иначе будет подставлен **unit-тип** -- енум с конструктором ```()``` без аргументов (пример 2):
```
ghci> :t read
read :: Read a => String -> a
ghci> read "True"
*** Exception: Prelude.read: no parse
ghci> read "True" :: Bool
True
```

Поэтому лучше пользоваться безопасными классами типов -- readMaybe/readEither --  те же самые риды, обернутые в соответствующие контексты: 
```
ghci> :module Text.Read

ghci> :t readMaybe
readMaybe :: Read a => String -> Maybe a
ghci> :t readEither 
readEither :: Read a => String -> Either String a
ghci> readMaybe "5" :: Maybe Int
Just 5
ghci> readMaybe "5" :: Maybe Bool
Nothing
ghci> readEither "5" :: Either String Bool -- don't worry, convenient way exist
Left "Prelude.read: no parse"
```

## Больше примеров ад-хок полиморфизма

```
subtract :: Num a => a -> a -> a
subtract x y = y - x

ghci> :info Fractional
class Num a => Fractional a where
  (/) :: a -> a -> a
  recip :: a -> a
  fromRational :: Rational -> a
  {-# MINIMAL fromRational, (recip | (/)) #-}
  	-- Defined in ‘GHC.Real’
instance Fractional Float -- Defined in ‘GHC.Float’
instance Fractional Double -- Defined in ‘GHC.Float’

average :: Fractional a => a -> a -> a
average x y = (x + y) / 2
```

```
ghci> cmpSum x y = if x < y then x + y else x * y
ghci> :t cmpSum
cmpSum :: (Ord a, Num a) => a -> a -> a
```
И еще прикол, можно множественные констрейнты записывать не только туплом но и в каррированном виде: ```cmpSum :: Ord a => Num a => a -> a -> a``` :)

## про undefined

Я хз что тут писать, по сути этокласс с наиболее общим типом -- без единого контрейнта, когда вызывается выисление экземпляра этого класса вылетает error => +- логично, что не стоит его вызывать. В продакшене может использоваться как заглушка для чего-то, что еще не написано, но будеь в скором времени. Тогда можно затайпчекать сразу, а дописать потом :)

## про deriving

Автоматическое создание инстансов класса типов компилятором:
```
data TrafficLight = Red | Yellow | Green | Blue
    deriving (Eq, Ord, Enum, Bounded, Show, Read, Ix)

ghci> show Blue
"Blue"
ghci> read "Blue" :: TrafficLight 
Blue
ghci> Red == Yellow  -- (==) is from Eq  class
False
ghci> Red < Yellow   -- (<)  is from Ord class
True

ghci> :t fromEnum 
fromEnum :: Enum a => a -> Int
ghci> :t toEnum 
toEnum :: Enum a => Int -> a
ghci> fromEnum Green
2
ghci> toEnum 2 :: TrafficLight 
Green

ghci> :t maxBound 
maxBound :: Bounded a => a
ghci> maxBound :: TrafficLight  -- Bounded also has 'minBound'
Blue
ghci> [Yello .. maxBound]  -- .. is from Enum instance
[Yellow, Green, Blue] 
```

**Для функций дерайвинг не разрешен, потому что банально не понятно, а что делать**.

Есть деррайвинг и для newtype -- нужна только прагма -XGeneralizedNewtypeDeriving -- синтаксис аналогичный, но есть прикол, что зачастую там настолько много всего нужно задеррайвить, чтобы организовать одну конуретную логику, что прям больно читать. Вроде на какой-то из следующих лекций будет рассказ про стратегии для деррайвинга

## Modules cheatsheet

Это все пишется в Extensions.hs, или типо того.
Имя модуля пишется после прагм и до испортов, в фигурных скобках указываем все, что хотим экспортировать, у конструкторов типов можно указать, какие конструкторы данных можно будет использовать (или ```..```, если все)
**Импортируется все, что экспортируется из импортируемого модуля.** Но можно через кейворд **hiding** ограничить эту область.

Можно еще алаисы указывать через ```AS```. Если после импорта указан кейворд **qualified**, то нужно писать полное квалифайд имя объекта: ```<имя модуля>.<имя объекта>```, если нет -- то как хочешь, но надо помнить про краши имен. 
```
module Lib 
       ( module Exports
       , FooB1 (..), FooB3 (FF)
       , Data.List.nub, C.isUpper
       , fooA, bazA, BAZB.isLower
       ) where

import           Foo.A
import           Foo.B     (FooB2 (MkB1), 
                            FooB3 (..))
import           Prelude   hiding (print)
import           Bar.A     (print, (<||>))
import           Bar.B     ()

import           Baz.A     as BAZA  
import qualified Data.List
import qualified Data.Char as C hiding (chr)
import qualified Baz.B     as BAZB (isLower)

import qualified Foo.X     as Exports
import qualified Foo.Y     as Exports
```
```
module Foo.A where fooA = 3
module Foo.B 
       ( FooB1, FooB2 (..), 
         FooB3 (FF, val)
       ) where

data FooB1 = MkFooB1
data FooB2 = MkB1 | MkB2
data FooB3 = FF { val :: Int }
```
Важный момент -- тут пустые скобки у экспортируемого модуля, они означают, что модуль ничего не экспортирует. **НО ИНСТАНСЫ ЭКСПОРТИРУЮТСЯ ВСЕГДА, КАК БЫ ТЫ НЕ ПЫТАЛСЯ ЭТОГО НЕ ДОПУСТИТЬ :)**. То есть тут экспортирован будет только instance Printable Int:
```
module Bar.B () where

class Printable p where 
    printMe :: p -> String

instance Printable Int where 
    printMe = show
```
```
module Baz.A (bazA) where bazA = map
```
Какой-то пример форвард-экспорта:
```
module Baz.B (C.isLower) where 

import Data.Char as C
```

  
  
  
# Lecture 6. <$> Functor, <*> Applicative. Про базовые тайп-классы

## Semigroup
Класс с замкнутой бинарной функцией, которая должна быть ассоциативной (обеспечивается программистом):
```
-- actually in 'base'
class Semigroup m where
    (<>) :: m -> m -> m
Associativity law for Semigroup: 
  1. (x <> y) <> z ≡ x <> (y <> z)

1. (++)
2. max/min
```
### Monoids
Семигруппа с нейтральным элементом относительно (<>) (оператор diamond :) )
```
-- also in 'base' but...
class Semigroup m => Monoid m where
    mempty :: m
Identity laws for Monoid: 
  2. x <> mempty ≡ x
  3. mempty <> x ≡ x
  
1. (++)
```

**Формальное определение Semigroup**:
```
class Semigroup a where
    (<>)    :: a -> a -> a
    sconcat :: NonEmpty a -> a
    stimes  :: Integral b => b -> a -> a
    {-# MINIMAL (<>) #-}

instance Semigroup [a] where
    (<>) = (++)

ghci> [1..5] <> [2,4..10]
[1,2,3,4,5,2,4,6,8,10]
```
Здесь sconcat функция над списком из элементов типа a, причем не пустым -- она буквально применяет оператор даймонд над списком, сворачивая их в один элемент типа a. NonEmpty -- фактически то же самое, что и писок, только в нем нет конструктора пустого списка. А stimes крутая, потому что похволяет сделать так:
```
ghci> concat (replicate 3 [1..5])
[1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5]

ghci> 3 `stimes` [1..5]
[1, 2, 3, 4, 5, 1, 2, 3, 4, 5, 1, 2, 3, 4, 5]
```

Ситуация: хочется полугруппу с двумя операциями даймонд, как быть --Вопользоваться оберткой :)
```
newtype Sum     a = Sum     { getSum     :: a }
newtype Product a = Product { getProduct :: a }

instance Num a => Semigroup (Sum a) where
  Sum x <> Sum y = Sum (x + y)

instance Num a => Semigroup (Product a) where
  Product x <> Product y = Product (x * y)
  
ghci> 3 <> 5 :: Sum Int
Sum { getSum = 8 }
ghci> 3 <> 5 :: Product Int
Product { getProduct = 15 }
```

Прикол в том, что дасточно определить одну операцию, чтобы реализовывать логику всего типа, еще примеры:
```
newtype Max   a = Max   { getMax   :: a      }  -- max
newtype Min   a = Min   { getMin   :: a      }  -- min
newtype Any     = Any   { getAny   :: Bool   }  -- ||
newtype All     = All   { getAll   :: Bool   }  -- &&
newtype First a = First { getFirst :: Maybe a}  -- first Just
newtype Last  a = Last  { getLast  :: Maybe a}  -- last Just

<тут надо соотвествующие инстансы создать еще>

ghci> Max 3 <> Max 10 <> Max 2
Max { getMax = 10 }
ghci> Min 3 <> Min 10 <> Min 2
Min { getMin = 2 }

ghci> Any True <> Any False <> Any True
Any { getAny = True }
ghci> All True <> All False <> All True
All { getAll = False }

ghci> First Nothing <> First (Just 10) <> First (Just 1)
First { getFirst = Just 10 }
ghci> Last (Just 2) <> Last (Just 1) <> Last Nothing
Last { getLast = Just 1 }
```

**Формальное определение Monoid**:
```
class Semigroup a => Monoid a where
    mempty  :: a
    mappend :: a -> a -> a -- обычно он равен (<>)
    mconcat :: [a] -> a
    {-# MINIMAL mempty #-}
```
Примеры:
```
instance Monoid [a] where
    mempty          = []
    l1 `mappend` l2 = l1 ++ l2
    
instance (Monoid a, Monoid b) => Monoid (a,b) where
    mempty                    = (         mempty,          mempty)
    (a1,b1) `mappend` (a2,b2) = (a1 `mappend` a2, b1 `mappend` b2)
```

> Тут была страшная картинка про отношения тайп-классов в виде графов

### Блять
Вот мы жили хорошо, умея определять тип по экземпляру, написать в ghci ```:t```. А как пел Куок: 'плохое точно сбудется', теперь мы хотим уметь узнавать 'тип типа' -- корректнее называется **'kind of type'** -- это позволить размышлять о способностях типа принимать в себя другие типы в качестве параметров:
```
ghci> :k Bool
Bool :: *
ghci> :k ()
() :: *
ghci> :k Ingteger
Ingeter :: *

ghci> :k Maybe Integer
Maybe Integer :: *
ghci> :k Maybe
Maybe :: * -> *

ghci> -- data [] a = [] | a : ([] a)
ghci> :k [Char]
[Char] :: *
ghci> :k []
[] :: * -> *

ghci> -- data Either a b
ghci> :k Either
Either :: * -> * -> *

ghci> data D m a = D (m a)
ghci> :k D
D :: (* -> *) -> * -> *

ghci> -- class Num a where
ghci> :k Num
Num :: * -> Constraint
ghci> :t (+)
(+) :: Num a => a -> a -> a
```
Про констрейнты будет позже, но вроде этот пример должен был навести на мысли о схожести с определение типа экземпляра


## Foldable

Есть функции foldr и foldl, они вот такие и вроде понятно, что делают на условном примере списка:
```
foldr :: (a -> b -> b) -> b -> [a] -> b
foldl :: (b -> a -> b) -> b -> [a] -> b 
```
Ща разъеб будет. Мы только что увидели, что кайнд списка -- это ```* -> *```, тогда можно оБоБщИтЬ свертки на вообще произвольный тип, котораый удовлетваоряет констрейнту Foldable:
```
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b 
```
Вроде очевидные примеры:
```
ghci> foldr (+) 0 [2, 1, 10] -- (2 + (1 + (10 + 0)))
13
ghci> foldr (*) 3 [2, 1, 10] -- (2 * (1 * (10 * 3)))
60
```

**Формальное определение Foldable**:
```
-- | Simplified version of Foldable
class Foldable t where
    {-# MINIMAL foldMap | foldr #-}

    fold    :: Monoid m => t m -> m
    foldMap :: Monoid m => (a -> m) -> t a -> m
    foldr   :: (a -> b -> b) -> b -> t a -> b 
```
Важно заметить, что тут везде кайнд у типа t это ```* -> *```. Вот такой мем, расписывать его не хочется, думаю понятно, что тут происходит. 

Примеры:
```
instance Foldable [] where
    foldr :: (a -> b -> b) -> b -> [a] -> b
    foldr _ z []     =  z
    foldr f z (x:xs) =  x `f` foldr f z xs
```
^ Важно, что тут [], а не [a], потому что мы должны совпадать по кайндам.
```
instance Foldable Maybe where
    foldr :: (a -> b -> b) -> b -> Maybe a -> b
    foldr _ z Nothing  = z
    foldr f z (Just x) = f x z

ghci> foldr (+) 1 (Just 3)
4
ghci> foldr (+) 0 Nothing
0
```

## Functor <$>

Теперь можно нормально дать определение слову **контекст** -- тип, функтор которого ```* -> *```.
Иногда хочется уметь оперировать над значениями, завернутыми в контекст. Напрмер, есть Maybe(2), хотим сделать +3 и получить Maybe(5).

Для таких приколов в функциональном мире есть fmap -- она по сути просто пропихивает функцию в контекст:
```
ghci> fmap (+3) (Just 2)
Just 5
ghci> fmap (+3) Nothing
Nothing
```

**Около-формальное определение Functor**:
```
class Functor f where               -- f :: * -> *
    fmap :: (a -> b) -> f a -> f b

Functor laws:
1. fmap id      ≡ id
2. fmap (f . g) ≡ fmap f . fmap g
```

Примеры:
```
instance Functor Maybe where
    fmap :: (a -> b) -> Maybe a -> Maybe b
    fmap f (Just x) = Just (f x)
    fmap f Nothing  = Nothing

```
Лист -- тоже Фанктор:
```
instance Functor [] where
    fmap :: (a -> b) -> [a] -> [b]
    fmap = map

ghci> fmap (*2) [1..3]
[2,4,6]
ghci> map (*2) [1..3]
[2,4,6]
ghci> fmap (*2) []
[]
```

### Arrow Funtor
Функция в Хаскелле это тоже тип данных, причем если посмотреть его кайнд, то увидим ```(->) :: * -> * -> *```, то есть при фиксированном типе аргумента, функция фазвращает фанктор. Вау. Тогда верно:
```
instance Functor ((->) r)  where
    fmap :: (a -> b) -> (r -> a) -> r -> b
    fmap = (.)
    
ghci> let foo = fmap (+3) (+2)
ghci> foo 10
15
```

## Applicative Functors
Мотивация:
```
ghci> :t fmap (++) (Just "hey")
fmap (++) (Just "hey") :: Maybe ([Char] -> [Char])

ghci> :t fmap compare (Just 'a')
fmap compare (Just 'a') :: Maybe (Char -> Ordering)

ghci> let a = fmap (*) [1,2,3,4]
ghci> :t a
a :: [Integer -> Integer]
ghci> fmap (\f -> f 9) a
[9,18,27,36]
```
**Формальное определение Applicative Functor**:
```
class Functor f => Applicative f where  -- f :: * -> *
    pure  :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
    liftA2 :: (a -> b -> c) -> f a -> f b -> f c  -- since GHC 8.2.1
    {-# MINIMAL pure, ((<*>) | liftA2) #-}
    
    
Applicative laws:
1. identity
   pure id <*> v ≡ v

2. composition
   pure (.) <*> u <*> v <*> w ≡ u <*> (v <*> w)

3. homomorphism
   pure f <*> pure x ≡ pure (f x)

4. interchange
   u <*> pure y ≡ pure ($ y) <*> u
```
Примеры:
\1. Maybe Applicative
```
instance Applicative Maybe where
    pure :: a -> Maybe a
    pure = Just
    
    (<*>) :: Maybe (a -> b) -> Maybe a -> Maybe b
    Nothing <*> _         = Nothing
    Just f  <*> something = fmap f something
    
ghci> Just (+3) <*> Just 9
Just 12
ghci> pure (+3) <*> Just 10
Just 13
ghci> Just (++"hahah") <*> Nothing
Nothing
```
\2. List Applicative
Какие-то заумные штуки начались:
```
instance Applicative [] where
    pure :: a -> [a]
    pure x    = [x]

    (<*>) :: [a -> b] -> [a] -> [b]
    fs <*> xs = [f x | f <- fs, x <- xs] -- тут важен порядок, т.к. более дальнее упоминание = более частое итеррирование

    
ghci> [(*2), (+3)] <*> [1, 2, 3]
[2, 4, 6, 4, 5, 6]

ghci> [ x*y | x <- [2,5,10], y <- [8,10,11]]
[16,20,22,40,50,55,80,100,110]

ghci> (*) <$> [2,5,10] <*> [8,10,11]
[16,20,22,40,50,55,80,100,110]
```

\3. Arrow Applicative
Еще более умные штуки:
```
instance Applicative ((->) r) where
    pure :: a -> r -> a
    pure x  = \_ -> x

    (<*>) :: (r -> a -> b) -> (r -> a) -> r -> b
    f <*> g = \x -> f x (g x)

ghci> :t (+) <$> (+3) <*> (*100)
(+) <$> (+3) <*> (*100) :: (Num a) => a -> a

ghci> (+) <$> (+3) <*> (*100) $ 5
508

ghci> (\x y z -> [x,y,z]) <$> (+3) <*> (*2) <*> (/2) $ 5
[8.0,10.0,2.5]
```

### Applicative vs. Functor
:(
```
ghci> (*) <$> Just 5 <*> Just 3
Just 15
ghci> liftA2 (*) (Just 5) (Just 3)
Just 15

ghci> :t liftA3
liftA3 :: Applicative f => (a -> b -> c -> d) -> f a -> f b -> f c -> f d

ghci> import Data.Char (isUpper, isDigit)
ghci> import Control.Applicative (liftA2)
ghci> let isUpperOrDigit = liftA2 (||) isUpper isDigit
ghci> :t isUpperOrDigit 
isUpperOrDigit :: Char -> Bool
ghci> isUpperOrDigit 'A'
True
ghci> isUpperOrDigit '3'
True
ghci> isUpperOrDigit 'a'
False
```
  
# Lecture 7: Applicative style programming, Phantom types

## Пример использования liftA3
  
```
data User = User
    { userFirstName :: String
    , userLastName  :: String
    , userEmail     :: String
    }
  
type Profile = [(String, String)]

profileExample = 
    [ ("first_name", "Pat"            )
    , ("last_name" , "Brisbin"        )
    , ("email"     , "me@pbrisbin.com")
    ]
  
lookup "first_name" p :: Maybe String

buildUser :: Profile -> Maybe User
buildUser p = User
    <$> lookup "first_name" p
    <*> lookup "last_name"  p
    <*> lookup "email"      p
-- using liftA* common pattern
buildUser :: Profile -> Maybe User
buildUser p = liftA3 User
                     (lookup "first_name" p)
                     (lookup "last_name"  p)
                     (lookup "email"      p)
```
      
## Alternative
Это моноид апплекативных функторов: <|> - возвращает первое не "ошибочное" значение, empty - "ошибочное" значение
```
class Applicative f => Alternative f where
    empty :: f a
    (<|>) :: f a -> f a -> f a
COPY
instance Alternative Maybe where
    empty :: Maybe a
    empty = Nothing

    (<|>) :: Maybe a -> Maybe a -> Maybe a
    Nothing <|> r = r
    l       <|> _ = l
ghci> Nothing <|> Just 3 <|> empty <|> Just 5
Just 3
instance Alternative [] where
    empty :: [a]
    empty = []
    
    (<|>) :: [a] -> [a] -> [a]
    (<|>) = (++)
ghci> [] <|> [1,2,3] <|> [4]
[1,2,3,4]   
```
      
## Traversable
Принимает контейнер наследованный от Functor и Foldable. Traverse - это как foldr, но не свертывает контейнер: "-> f (t b)"
      
```
class (Functor t, Foldable t) => Traversable t where
    traverse  :: Applicative f => (a -> f b) -> t a -> f (t b)
    sequenceA :: Applicative f => t (f a) -> f (t a)
ghci> let half x = if even x then Just (x `div` 2) else Nothing
ghci> traverse half [2,4..10]
Just [1,2,3,4,5]
ghci> traverse half [1..10]
Nothing
instance Traversable Maybe where
    traverse :: Applicative f => (a -> f b) -> Maybe a -> f (Maybe b)
    traverse _ Nothing  =
    traverse f (Just x) =
instance Traversable Maybe where
    traverse :: Applicative f => (a -> f b) -> Maybe a -> f (Maybe b)
    traverse _ Nothing  = pure Nothing
    traverse f (Just x) = Just <$> f x
instance Traversable [] where
    traverse :: Applicative f => (a -> f b) -> [a] -> f [b]
    traverse f = foldr consF (pure [])
      where 
        consF x ys = (:) <$> f x <*> ys
```
## Phantom types
Проблема: 
```
newtype Hash = MkHash String
class Hashable a where
hash :: a -> Hash
hashPair :: Int -> Hash
hashPair n = hash (n, n)
hashPair :: Int -> Hash
hashPair n = hash n  -- oops, this is valid definition!
```
Хотим уточнить тип функции
```
newtype Hash a = MkHash String -- `a` is a phantom type, not present in constructor
class Hashable a where
hash :: a -> Hash a
hashPair :: Int -> Hash (Int, Int)
hashPair n = hash (n, n)
hashPair :: Int -> Hash (Int, Int)
hashPair n = hash n  -- This is no longer a valid definition!
```
## Type @pplication
Не компилится:
```
-- v2.0.0: compiles or not?
-- v3.0.0: doesn't compile!
prepend2 :: a -> [a] -> [a]
prepend2 x xs = pair ++ xs 
where pair :: [a]
      pair = [x, x] 
```
Потому что компилятор считает, что это две различные связки forall. 
```
prepend2 :: forall a . a -> [a] -> [a]
prepend2 x xs = pair ++ xs 
  where 
    pair :: forall a . [a]
    pair = [x, x] 
```
Решается добавлением экстеншина {-# LANGUAGE ScopedTypeVariables #-}
## XTypeApplications
```
ghci> read "3"   :: Int   -- Было
ghci> rread @Int "3" -- Стало 
```
Пиши как удобней!

## -XAllowAmbiguousTypes
Не компилится :(
```
class Size a where
    size :: Int
    • Could not deduce (Size a)
      from the context: Size a
        bound by the type signature for:
                   size :: forall a. Size a => Int
```
Решение: 
```
{-# LANGUAGE AllowAmbiguousTypes #-}

class Size a where size :: Int
instance Size Int    where size = 8
instance Size Double where size = 16
ghci> size @Double  -- use -XTypeApplications
16
```
# Lectures 8-9 Monads
```
class Monad m where   -- m :: * -> *
    return :: a -> m a                  -- return
    (>>=)  :: m a -> (a -> m b) -> m b  -- bind
```
## Maybe Monad
Пример реализации и использования:
```
instance Monad Maybe where
    return :: a -> Maybe a
    return = Just
  
    (>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
    Nothing >>= _ = Nothing
    Just a  >>= f = f a

monadPair :: Monad m => m a -> m b -> m (a, b)    -- polymorphic implementation
monadPair ma mb = ma >>= \a -> mb >>= \b -> return (a, b)      
      
mkUser :: String -> Maybe Username  -- FP programming pattern: smart constructor
mkUser name = stripUsername name >>= validateLength 15 >>= return . Username

```
Вместо:
```
maybePair :: Maybe a -> Maybe b -> Maybe (a, b)    -- naive implementation
maybePair Nothing  _        = Nothing
maybePair _        Nothing  = Nothing
maybePair (Just a) (Just b) = Just (a, b)
      
mkUser :: String -> Maybe Username  -- FP programming pattern: smart constructor
mkUser name = case stripUsername name of
    Nothing    -> Nothing
    Just name' -> case validateLength 15 name' of
        Nothing     -> Nothing
        Just name'' -> Just $ Username name''
```
      
## Either monad
Так как монады можно применять только к * -> *, придется зафиксировать Eather e и работать со вторым типом
```
instance Monad (Either e) where ...  -- Either a :: * -> *
    return :: a -> Either e a
    return = Right

    (>>=) :: Either e a -> (a -> Either e b) -> Either e b
    Left e  >>= _ = Left e
    Right a >>= f = f a
```
      
 ## List Monad
```
instance Monad [] where
    return :: a -> [a]
    return x = [x]  -- using bender-operator: return = (:[])
    
    (>>=) :: [a] -> (a -> [b]) -> [b]
    l >>= f  = concat (map f l)  -- or using concatMap
     
ghci> [10, 5, 7] >>= replicate 3
[10, 10, 10, 5, 5, 5, 7, 7, 7]
```
      
## Рыбы
Как композиция, только с монадами. Их нет в дефолт мондах, нужно писать ручками
```
(<=<) :: Monad m => (b -> m c) -> (a -> m b) -> a -> m c
(>=>) :: Monad m => (a -> m b) -> (b -> m c) -> a -> m c
m >>= (f >=> g) ≡ m >>= f >>= g
m >>= (f <=< g) ≡ m >>= g >>= f
(f >=> g) >=> h ≡ f >=> (g >=> h)  
```
      
## Monad zen
```
(>>) :: Monad m => m a -> m b -> m b  -- then
m >> k = m >>= \_ -> k
      
ghci> Nothing >> Just 5
Nothing
```
      
## Join
```
join :: Monad m => m (m a) -> m a
      
ghci> join [[3, 4], [7, 10]]
[3, 4, 7, 10]
```
      
## Control.Monad
Тоже самое что и для аплекативов (даже реализации наследуются), но так исторически сложилось - монады появились раньше
```
liftM :: Monad m => (a -> b) -> m a -> m b

ghci> liftM (+1) (Just 3)
Just 4
ghci> liftM (+1) Nothing
Nothing
      
liftM2 :: Monad m => (a -> b -> c) -> m a -> m b -> m c
         
ghci> liftM2 (+) (Just 1) (Just 2)
Just 3
```

## Monadic lows
```
1. return a >>= f  ≡ f a                      -- left identity
2. m >>= return    ≡ m                        -- right identity
3. (m >>= f) >>= g ≡ m >>= (\x -> f x >>= g)  -- associativity
```
## Logging evaluation
      
Для чистоты функции можем таскать логи с собой, а потом уже выводить!
```
newtype Writer w a = Writer { runWriter :: (a, w) } -- a is value, w is log
      
instance Monoid w => Monad (Writer w) where
    return :: a -> Writer w a
    return a = Writer (a, mempty)
    
    (>>=) :: Writer w a -> (a -> Writer w b) -> Writer w b
    Writer (a, oldLog) >>= f = let Writer (b, newLog) = f a 
                               in Writer (b, oldLog <> newLog)
tell       :: w -> Writer w ()
execWriter :: Writer w a -> w
writer     :: (a, w) -> Writer w a
      
binPow :: Int -> Int -> Writer String Int
binPow 0 _      = return 1
binPow n a
    | even n    = binPow (n `div` 2) a >>= \b -> 
                  tell ("Square " ++ show b ++ "\n") >>
                  return (b * b)
    | otherwise = binPow (n - 1) a >>= \b -> 
                  tell ("Mul " ++ show a ++ " and " ++ show b ++ "\n") >>
                  return (a * b)      
```
На самом деле никакого конструктора Writer не существует (там WriterT), поэтому сделали writer.
Лучше никогда его не использовать, потому что из-за ленивого вычисления копится невычесленный лог (что очень плохо)
      
## Reader
Просто удобная обертка над функцией. 
```
newtype Reader e a = Reader { runReader :: e -> a }
      
instance Monad (Reader e) where
    return :: a -> Reader e a
    return a = Reader $ \_ -> a

    (>>=) :: Reader e a -> (a -> Reader e b) -> Reader e b
    m >>= f = Reader $ \r -> runReader (f $ runReader m r) r
      
ask   :: Reader e e                            -- get whole env
asks  :: (e -> a) -> Reader e a                -- get part of env
local :: (e -> b) -> Reader b a -> Reader e a  -- change env locally
```
## State monad
Позволяет добавить императивный синтаксис в мир фп
```
newtype State s a = State { runState :: s -> (a, s) }

instance Monad (State s) where
    return :: a -> State s a
    return a = State $ \s -> (a, s)

    (>>=) :: State s a -> (a -> State s b) -> State s b
    oldState >>= f = ...   
      
      
type Stack = [Int]

pop :: State Stack Int
pop = state $ \(x:xs) -> (x, xs)

push :: Int -> State Stack ()
push x = state $ \xs -> ((), x:xs)
stackOps :: State Stack Int
stackOps = pop >>= \x -> push 5 >> push 10 >> return x
```
### Useful function
      
```
get       :: State s s
put       :: s -> State s ()
modify    :: (s -> s) -> State s ()
gets      :: (s -> a) -> State s a
withState :: (s -> s) -> State s a -> State s a
evalState :: State s a -> s -> a
execState :: State s a -> s -> s      
```


      
      
      

# Lecture 10. Про реальный мир

Хочется уметь взаимодействовать с реальным миром, как в императивных языках, например, считывать что-то. Предположим, что мы как-то реализовали функцию getchar:
```
getchar :: Char
get2chars = [getchar, getchar]
```
Уже есть проблемы:
1. Это по сути константа а не функция + она не чистая
2. Поскольку Хаскелль думает, что все -- чистые функции, он имеет право переупорядочивать вычисления, тем самым get2chars получается не валидной
3. Вообще в get2chars будет один и тот же символ, т.к. вычисления ленивые

```
getchar   :: Int -> Char
get2chars :: Int -> String
get2chars _ = [getchar 1, getchar 2]

getchar :: Int -> (Char, Int)

get2chars i = [a,b] where (a,i1) = getchar i
                          (b,i2) = getchar i1

get4chars = [get2chars 1, get2chars 2]
```
Добавили некоторый счетчик, чтобы задать номер считывания -- уже лучше: решили 1, решили 3.
```
get2chars :: Int -> (String, Int)
get2chars i0 = ([a,b], i2)  where (a,i1) = getchar i0
                                  (b,i2) = getchar i1
get4chars :: Int -> String
get4chars i0 = (a++b) where (a,i1) = get2chars i0
                            (b,i2) = get2chars i1
```
Добавили связь через ретерн вэлью -- решили 2.
Мы изобрели I/O Monad :)

### Фейк-реализация монады ИО:
```
type IO a  =  RealWorld -> (a, RealWorld)

main :: RealWorld -> ((), RealWorld)
main :: IO ()

getChar :: RealWorld -> (Char, RealWorld)

--- функция main, считывающая два символа из консоли:
main :: RealWorld -> ((), RealWorld)
main world0 = 
    let (a, world1) = getChar world0
        (b, world2) = getChar world1
    in ((), world2)
```
Тут решены все проблемы кроме того, что мы можем внутри сделать двойное вычисление от того же токена. Зачем? Непонятно. Но можем. 

Поэтому тут нужны свойства монады, чтобы делигировать на нее контроль этого

### I/O Monad
```
newtype IO a = IO {unIO :: State# RealWorld -> (State# RealWorld, a)}

{-# LANGUAGE MagicHash #-}  -- allows using # in names
data Mystery# a = Magic# a deriving (Show)
ghci> Magic# 3
Magic# 3
ghci> :t Magic# 3
Magic# 3 :: Num a => Mystery# a
```
Здесь # -- позволяет объявлять функции, реализация которых придет, например, из ассемблера (аналог extern "C", как будто)
Сама монада:

```
newtype IO a = IO {unIO :: State# RealWorld -> (State# RealWorld, a)}
data State# s
data RealWorld

instance Monad IO where
  IO m >>= k = IO $ \ s -> 
     case m s of 
       (new_s, a) -> unIO (k a) new_s
  
  return x = IO (\s -> (s, x))
```
Здесь RealWorld -- какой-то глобальный контекст, а State# (не монада State!!) -- некоторый дескриптор, позволяющий опознать поток исполнения.

### do-notation (do, <-)

Позволяет дропать zen (>>):
```
(>>) :: IO a -> IO b -> IO b

(action1 >> action2) world0 =
    let (_, world1) = action1 world0
       (b, world2) = action2 world1
    in (b, world2)

--- Тогда можно писать так:
putStrLn :: String -> IO ()

main = do putStrLn "Hello!"
main = putStrLn "Hello!"

--- выглядит неочень полезно для одного действия, но для пачки - круто:
main = do putStrLn "What is your name?"
          putStrLn "How old are you?"
          putStrLn "Nice day!"

main = putStrLn "What is your name?" >>
       putStrLn "How old are you?"   >>
       putStrLn "Nice day!"
```

Пачку действий можно записать и так:
```
ioActions :: [IO ()]
ioActions = [ print "Hello!"
            , putStr "just kidding"
            , getChar >> return ()
            ]

main = do head ioActions
          ioActions !! 1
          last ioActions

--- выглядит внешне легко, но стремно так делать, есть sequence_ 
--- (_ означает, что вернется юнит -- т.е. нам не важен результат)
--- аналог из монады traversible:

sequence_ :: [IO a] -> IO ()

main = sequence_ ioActions
sequence_ :: [IO a] -> IO ()
sequence_ []     = return ()
sequence_ (x:xs) = do x
                      sequence_ xs

--- пишется вообще чиллово :)
```

Также в ду-нотацию можно запихнуть и бинд:
```
(>>=) :: IO a -> (a -> RealWorld -> (b, RealWorld)) -> IO b

(action1 >>= action2) world0 =
    let (a, world1) = action1 world0
       (b, world2) = action2 a world1
    in (b, world2)
    

getLine :: IO String
--- все это эквивалентно:
main = do s <- getLine  --- похоже на R :) - засовываемм в переменную результат функции
          putStrLn s
          
main = getLine >>= \s ->
       putStrLn s

main = getLine >>= putStrLn
```

Пример:
```
main = do putStr "What is your name?"
          a <- readLn
          putStr "How old are you?"
          b <- readLn
          print (a,b)
          
---  тоже самое, что и:
main =    putStr "What is your name?" >>
          readLn >>= \a -> 
          putStr "How old are you?" >>
          readLn >>= \b -> 
          print (a,b)
```

#### про return в do-notation
```
return :: a -> IO a
return a world0 = (a, world0)
getReversedLine :: IO String
getReversedLine = do
    s <- getLine
    return $ reverse s

main :: IO ()
main = do
    rs <- getReversedLine
    putStrLn rs
```

Если хотим, чтобы функция возвращала что-то, завернутое в монадный контекст, можно писать ретерн, здесь (и только здесь) он похоже на плюсовый. НО:
```
main = do a <- readLn
          if a >= 0 then 
              return ()
          else do
              putStrLn "a is negative"

          putStrLn "a is positive"  -- is this executed?
```
Поскольку if это экспрешен, последняя строка будет выводиться всегда

#### про let в do-notation

Можно делать let, просто не написав in:
```
main :: IO ()
main = do
    s <- getLine
    let rs = reverse s
    putStrLn $ "Reversed input : " ++ rs

--- desugars into:
main :: IO ()
main =     getLine >>= \s -> 
           let rs = reverse s in
           putStrLn $ "Reversed input : " ++ rs
```

#### Микровывод
Типичные ошибки:
```
let s = getLine  -- !!! Doesn't read from console to `s`
rs <- reverse s  -- !!! `reverse s` is not a monadic action inside IO
```
Лет -- для результатов чистых функций
```<-``` -- для результатов монаидичных функций

Также do можно писать в любой монаде (прикол: в монаде листа второй пример ошибки -- корректно отработает)
Также do можно писать вне монад (опуская in по сути), но непонятно зачем
А ghci это по своей природе бесконечный do-блок внутри монады IO :)

\+ прикол:
можно так:
```
foo :: Int -> Int
foo = do
    a <- (+1)
    return (a * 2)
    
ghci> foo 3
8
```
Поскольку стрелка -- это монада


### Приколы

### Lazy IO
```
main = do
  fileContent <- readFile "foo.txt"
  writeFile "bar.txt" ('a':fileContent)
  readFile  "bar.txt" >>= putStrLn
  
ghci> :run main
afoo
bar
--- все ок
```

```
main = do
  fileContent <- readFile "foo.txt"
  writeFile "foo.txt" ('a':fileContent)
  readFile  "foo.txt" >>= putStrLn

ghci> :run main
*** Exception: foo.txt: openFile: resource busy (file is locked)

--- читаем и пишем в один файл -- не ок
--- потому что язык ленивый, и считывание будет при записи только выполнено
```
Может быть пофикшено так:
```
main = do
  fileContent <- readFile "foo.txt"
  putStrLn fileContent
  writeFile "foo.txt" ('a':fileContent)
--- вывели на консоль все содержимое файла -- зафорсили его чтение
--- но очевидно, это стремно
```
Какие решения?
* можно форсировать вычисления, об этом позже, -- тем самым сжигая оперативку
* можно использовать библиотеки conduit, pipes, streaming -- там пиздец, но делает работу похожей на работу со стримами


### FFI -- интеграция с другими языками
Ну, экстерн Си:
```
/* clang -c simple.c -o simple.o */

int example(int a, int b)
{
  return a + b;
}
```

```
-- ghc simple.o simple_ffi.hs -o simple_ffi

{-# LANGUAGE ForeignFunctionInterface #-}

import Foreign.C.Types

foreign import ccall safe "example" 
    example :: CInt -> CInt -> CInt

main = print (example 42 27)
```

А что если там будет какой-то принт например? [Есть статейка](https://en.wikibooks.org/wiki/Haskell/FFI), но в рамках курса похер.

### Мутабельные переменные
Представим, что мы реализовали их так:
```
main = do let a0 = readVariable  varA
          let _  = writeVariable varA 1
          let a1 = readVariable  varA
          print (a0, a1)
```
Проблемы? Ну все такие же как с гетчаром (буквально). Корректно делать с параметризацией токеном через IORef:
```
import Data.IORef (newIORef, readIORef, writeIORef)

foo :: IO ()
foo = do 
    varA <- newIORef 0 
    a0   <- readIORef varA
    writeIORef varA 1
    a1   <- readIORef varA
    print (a0, a1)
    
ghci> foo
(0,1)
```

### Мутабельные массивы
Они просто есть, стремные, есть крутой [vector](https://hackage.haskell.org/package/vector):
```
import Data.Array.IO (IOArray, newArray, readArray, writeArray)

bar :: IO ()
bar = do 
    arr <- newArray (1,10) 37 :: IO (IOArray Int Int)
    a   <- readArray arr 1
    writeArray arr 1 64
    b   <- readArray arr 1
    print (a, b)

ghci> bar
(37,64)
```

Здесь параметризировано двумя типами потому что индексация мб не только интом

Вектор, кстати, саморасширяющийся и мб мутабельным или иммутабельным

### IO Exceptions

Есть функция throwIO -- бросает эксепшн и прерывает исполнение программы. Пример безопастного деления:
```
throwIO :: Exception e => e -> IO a
import           Control.Exception (ArithException (..), catch, throwIO)
import           Control.Monad     (when)

readAndDivide :: IO Int
readAndDivide = do
    x <- readLn
    y <- readLn
    when (y == 0) $ throwIO DivideByZero
    return $ x `div` y
ghci> readAndDivide 
7
3
2
ghci> readAndDivide 
3
0
*** Exception: divide by zero
```
Здесь функция when -- по сути иф с одной веткой (else у него -- return ()) -- принимает кондишн и монадическое вычисление.

Можно поймать эксепшн:
```
catch :: Exception e => IO a -> (e -> IO a) -> IO a
safeReadAndDivide :: IO Int
safeReadAndDivide = readAndDivide `catch` \DivideByZero -> return (-1)
ghci> safeReadAndDivide 
7
3
2
ghci> safeReadAndDivide 
3
0
-1
```

Важно, что троу бросает ЛЮБОЙ эксепшн, а кэтч ловит КОНКРЕТНЫЙ. Это можно пофиксить, но нужно еще теории:

Существует экзистенциальный тип SomeException, который хранит в себе что-то, что является эксепшеном. Typeable -- позволяет в рантайме получить информацию о типе (по дефолту она затирается). На этом все и построено:
```
data SomeException = forall e . Exception e => SomeException

class (Typeable e, Show e) => Exception e where
    displayException :: e -> String
    --- displayException = show
    
    fromException :: SomeException -> Maybe e  --- если тип = нашему, то Just
    toException :: e -> SomeException  --- просто обернет в экзистенциальный тип


tryReadAndDivide :: IO (Either String Int)
tryReadAndDivide = fmap Right $ readAndDivide `catch` \(e :: SomeException) -> 
    case fromException e of
        Just (dbze :: DivideByZero) -> return $ Left $ displayException dbze
        _ -> return $ Left $ "Smth else happened"
```


Как сделать свое исключение?
```
{-# LANGUAGE DeriveAnyClass     #-}
{-# LANGUAGE DeriveDataTypeable #-}

import           Control.Exception (Exception)
import           Data.Typeable     (Typeable)

data MyException = DummyException
    deriving (Show, Typeable, Exception)
ghci> throwIO DummyException 
*** Exception: DummyException

ghci> :{
ghci| throwIO DummyException `catch` \DummyException ->
ghci|     putStrLn "Dummy exception is thrown"
ghci| :}
Dummy exception is thrown
```

Так же есть привычные плюсовые конструкции, связанные с исключениями, но ЗДЕСЬ ЭТО ФУНКЦИИ А НЕ КОНСТРУКЦИИ!!!

```
try     :: Exception e => IO a -> IO (Either e a)
tryJust :: Exception e => (e -> Maybe b) -> IO a -> IO (Either b a)

finally :: IO a	 -- computation to run first
        -> IO b	 -- computation to run afterward (even if an exception was raised)
        -> IO a

-- | Like 'finally', but only performs the final action 
-- if there was an exception raised by the computation.
onException :: IO a -> IO b -> IO a
```

Штука типа юник-поинтера, позволяет указать как открывать и откпускать ресурс (попытка в raii):
```
bracket :: IO a         -- ^ computation to run first (\"acquire resource\")
        -> (a -> IO b)  -- ^ computation to run last (\"release resource\")
        -> (a -> IO c)  -- ^ computation to run in-between
        -> IO c         -- returns the value from the in-between computation
```

### Напоминание про гарды и (<-)

В гардах можно паттерн-матчиться по результатам функций через `<-`:
```
lookup :: FiniteMap -> Int -> Maybe Int

addLookup :: FiniteMap -> Int -> Int -> Maybe Int
addLookup env var1 var2
 | Just val1 <- lookup env var1
 , Just val2 <- lookup env var2
 = val1 + val2
{-...other equations...-}


strangeOperation :: [Int] -> Ordering
strangeOperation xs 
   | 7  <- sum xs
   , n  <- length xs
   , n  >= 5
   , n  <= 20 
   = EQ
   | 1  <- sum xs
   , 18 <- length xs
   , r  <- nub xs `compare` [1,2,3]
   , r /= EQ
   = r
   | otherwise
   = [3,1,2] `compare` xs

main = print $ strangeOperation ([5,7..21] ++ [20,19..4])
```

По сути накладываем кучу условий, и если все из них выполнены, то мы дойдем до основного действия ветки гарда. Прикольно.

### UnsafePerformIO

Можно доставать из ИО Монады значение результата. Это, очевидно, создает ряд проблем:
```
import System.IO.Unsafe

foo :: ()
foo = unsafePerformIO $ putStrLn "foo"

bar :: String
bar = unsafePerformIO $ do
          putStrLn "bar"
          return "baz"

main = do let f = foo
          putStrLn bar

--- здесь будет выведено:
--- bar
--- baz
--- поскольку компилятор считает, что foo -- чистая и мы никак не используем ее результат

helper i = print i >> return i

main = do
    one <- helper 1
    two <- helper 2
    print $ one + two

--- здесь может быть выведено:
--- 1
--- 2
--- 3
--- либо:
--- 2
--- 1
--- 3
```
Наводит на мысли, что мы теряем порядок вычислений, если делаем такие приколы, опять таки в виду ленивости языка

А если второй пример переписать наполовину небезопасно?
```
import System.IO.Unsafe

helper i = print i >> return i

main = do
    one <- helper 1
    let two = unsafePerformIO $ helper 2
    print $ one + two
```
Все равно плохо, компилятор может переупорядочить монадоичное вычисление и вычисление чистой функции внутри :(

Вспомним, что внутри ИО монады есть стейт токены. Так вот performUnsafe как бы игнорирует их и подсовывает фейковый токен в вычисление :))
```
type S# = State# RealWorld  -- let's use this short alias

print 1 :: S# -> (S#, ())

print 1 >> print 2 = 
    \s0 -> case print 1 s0 of
               (s1, _ignored) -> print 2 s1

--- то есть:
unsafePerformIO (IO f) =
    case f fakeStateToken of
        (_ignoredStateToken, result) -> result
```
Пример рассахаривания (про переупорядочивание полубезопасных вычислений):
```
import System.IO.Unsafe

helper i = print i >> return i

main = do
    one <- helper 1
    let two = unsafePerformIO $ helper 2
    print $ one + two
    
main s0 =
    case helper 1 s0 of
        (s1, one) ->
            case helper 2 fakeStateToken of
                (_ignored, two) ->
                    print (one + two) s1

main s0 =
    case helper 2 fakeStateToken of
        (_ignored, two) ->
            case helper 1 s0 of
                (s1, one) ->
                    print (one + two) s1
```

Что тогда делать?
* Везде где можно избежать ансейв вычислений -- лучше их избежать
* Если мы не в ИО монаде, или можем гарантировать порядок логикой программы (что бывает очев сложно), то использовать unsafePerformIO

##### А когда правда нужно использовать?

\1. Для дебага чистых функций:
```
import Debug.Trace

trace     :: String -> a -> a  --- помимо вычисления принтит строку
traceShow :: Show a => a -> b -> b
traceM    :: Applicative f => String -> f () 

trace :: String -> a -> a
trace string expr = unsafePerformIO $ do  --- экстактит результат из монадоическиго вычисления
    traceIO string  -- slightly clever version of `putStrLn`
    return expr


fib :: Int -> Int
fib 0 = 0
fib 1 = 1
fib n = trace ("n: " ++ show n) $ fib (n - 1) + fib (n - 2)

ghci> putStrLn $ "fib 4:\n" ++ show (fib 4)
fib 4:
n: 4
n: 3
n: 2
n: 2
3
```

\2. В строках.
Сейчас будет немного страшно, но главное помнить, что кроме Text и ByteString ничего толком то и не нужно.

Cами строки долгие, т.к. они = лист чаров, можно перегрузить их тип как что-то, что IsString:

```
type String = [Char]
{-# LANGUAGE OverloadedStrings #-}

class IsString a where
    fromString :: String -> a
ghci> :t "foo"
"foo" :: [Char]

ghci> :set -XOverloadedStrings

ghci> :t "foo"
"foo" :: IsString a => a
```

Собственно Текст и БайтСтринг:
```
{-# LANGUAGE OverloadedStrings #-}

import qualified Data.Text as T

-- From pack
myTStr1 :: T.Text
myTStr1 = T.pack ("foo" :: String)

-- From overloaded string literal.
myTStr2 :: T.Text
myTStr2 = "bar"
```

```
{-# LANGUAGE OverloadedStrings #-}

import qualified Data.ByteString       as S
import qualified Data.ByteString.Char8 as S8

-- From pack
bstr1 :: S.ByteString
bstr1 = S.pack ("foo" :: String)

-- From overloaded string literal.
bstr2 :: S8.ByteString
bstr2 = "bar"
```

Причем тут unsafePerformIO ?
-- Хочется строки не только в ИО монаде использовать, поэтому там под капотом есть эта функция, вот (похуй как оно устронно в точности):
```
data ByteString = PS (ForeignPtr Word8) Int Int

unsafeCreate :: Int -> (Ptr Word8 -> IO ()) -> ByteString
unsafeCreate l f = unsafePerformIO (create l f)

create :: Int -> (Ptr Word8 -> IO ()) -> IO ByteString
create l f = do
    fp <- mallocByteString l
    withForeignPtr fp $ \p -> f p
    return $ PS fp 0 l
    
-- | /O(1)/ 'splitAt' @n xs@ is equivalent to @('take' n xs, 'drop' n xs)@.
splitAt :: Int -> ByteString -> (ByteString, ByteString)

-- | /O(1)/ Extract the last element of a ByteString.
last :: ByteString -> Word8
last ps@(PS x s l)
    | null ps   = errorEmptyList "last"
    | otherwise = unsafePerformIO $
                    withForeignPtr x $ \p -> peekByteOff p (s+l-1)
```

What to use?
I. Binary:
    * Packed:
        - Lazy: Data.ByteString.Lazy
        - Strict: Data.ByteString
II. Text:
    1. ASCII or 8-bit:
        * Packed and lazy: Data.ByteString.Lazy.Char8
        * Packed and strict: 
            - Data.ByteString.Char8,               
            - Data.CompactString.ASCII
            - Data.CompactString with Latin1
    2. Unicode:
        2.1 UTF-32: 
            * Unpacked and lazy:
                - [Char]
        2.2 UTF-16:
            * Packed and lazy: 
                - Data.Text.Lazy
            * Packed and strict: 
                - Data.Text
                - Data.CompactString.UTF16
        2.3 UTF-8:
            * Unpacked and lazy: 
                - Codec.Binary.UTF8.Generic contains generic operations that can be used to process [Word8]
            * Packed and lazy: 
                - Data.ByteString.Lazy.UTF8
            * Packed and strict: 
                - Data.CompactString.UTF8 
                - Data.ByteString.UTF8
                
После этого дерева хочется сделать напоминание:
> Сейчас будет немного страшно, но главное помнить, что кроме Text и ByteString ничего толком то и не нужно.




# Lecture 11: КоМбИнАтОрНыЕ пАрСеРы и тестирование

Идея понятна, сразу к сути:

### Что является результатом парсинга?
```
parseInteger :: String -> Bool
--- нет, тк хочется интеджер

parseInteger :: String -> Integer
--- нет, тк не каждая строка = интеджер

parseInteger :: String -> Maybe Integer -- Either for more descriptive error
--- нет, а что если помимо числа в строке есть что-то еще

parseInteger :: String -> Maybe (Integer, String)
--- в целом, да
```

Теперь сделаем это красиво:
```
                                      ┌── input stream
                                      │
               ┌─ result type         │                 ┌─ remain stream
               │                      │                 │
newtype Parser a = Parser { runP :: String -> Maybe (a, String) }
                             │                       │
                             └─ unwrapper            └─ parsing result
```

Немного примеров:
```
--- parser combinator type
newtype Parser a = Parser { runP :: String -> Maybe (a, String) }

--- parsing function
parseInteger :: Parser Integer  -- String -> Maybe (Integer, String)

--- apply parsing function
runP :: Parser a -> String -> Maybe (a, String)


--- examples:
ghci> runP parseInteger "5"
Just (5, "") :: Maybe (Integer, String)

ghci> runP parseInteger "42x7"
Just (42, "x7") :: Maybe (Integer, String)

ghci> runP parseInteger "abc"
Nothing :: Maybe (Integer, String)
```

### Примитивные парсеры
```
newtype Parser a = Parser { runP :: String -> Maybe (a, String) }

-- always succeeds without consuming any input
ok :: Parser ()
ok = Parser $ \s -> Just ((), s)

-- fails w/o consuming any input if given parser succeeds,
-- and succeeds if given parser fails
isnot :: Parser a -> Parser ()
isnot parser = Parser $ \s -> case runP parser s of
    Just _  -> Nothing
    Nothing -> Just ((), s)
    
-- succeeds only at the end of input stream
eof :: Parser ()
eof = Parser $ \s -> case s of
    [] -> Just ((), "")
    _  -> Nothing
    
-- consumes only single character and returns it if predicate is true
satisfy :: (Char -> Bool) -> Parser Char
satisfy p = Parser $ \s -> case s of
    []     -> Nothing
    (x:xs) -> if p x then Just (x, xs) else Nothing
```

Теперь можно уже их комбинировать:
```
-- always fails without consuming any input
notok :: Parser ()
notok = isnot ok

-- consumes given character and returns it
char :: Char -> Parser Char
char c = satisfy (== c)

-- consumes any character or any digit only
anyChar, digit :: Parser Char
anyChar = satisfy (const True)
digit   = satisfy isDigit

--- examples
ghci> runP eof ""
Just ((),"")
ghci> runP eof "aba"
Nothing
ghci> runP (char 'a') "aba"
Just ('a',"ba")
ghci> runP (char 'x') "aba"
Nothing
```


### Полезные штуки -- инстансы
```
instance Functor     Parser  -- replace parser value
instance Applicative Parser  -- run parsers sequentially one after another
instance Monad       Parser  -- same as above but with monadic capabilities
instance Alternative Parser  -- allows to choose parser
```

Functor -- применяет функцию к значению в контексте и при этом не меняет сам контекст:
```
instance Functor Parser where
    fmap :: (a -> b) -> Parser a -> Parser b
    fmap f (Parser parser) = Parser (fmap (first f) . parser)
```

Аппликативный функтор -- оператор апп должен распарсить (1) функцию, после распарсить (2) значение и применить функцию к значению:
```
instance Applicative Parser where
    pure :: a -> Parser a
    pure a = Parser $ \s -> Just (a, s)

    (<*>) :: Parser (a -> b) -> Parser a -> Parser b
    Parser pf <*> Parser pa = Parser $ \s -> case pf s of
        Nothing     -> Nothing
        Just (f, t) -> case pa t of
            Nothing     -> Nothing
            Just (a, r) -> Just (f a, r)

    -- can be written shorter using Maybe as Monad
```

:)
```
instance Monad Parser -- exercise
```

Альтернатива -- выбор из двух парсеров:
```
instance Alternative Parser where
    empty :: Parser a  -- always fails
    (<|>) :: Parser a -> Parser a -> Parser a  -- run first, if fails — run second
```

### Что уже имеем?
```
-- type
newtype Parser a = Parser { runP :: String -> Maybe (a, String) }

-- parsers
eof, ok :: Parser ()
satisfy :: (Char -> Bool) -> Parser Char
empty   :: Parser a

-- combinators
pure  :: a -> Parser a
(<|>) :: Parser a -> Parser a -> Parser a         -- orElse
(>>=) :: Parser a -> (a -> Parser b) -> Parser b  -- andThen
```

### Что будет полезно еще?
Все можно вывести из уже имеющихся функций
```
-- type
newtype Parser a = Parser { runP :: String -> Maybe (a, String) }

-- primitive parsers
eof, ok :: Parser ()
satisfy :: (Char -> Bool) -> Parser Char

-- combinators
-- * Functor
fmap  :: (a -> b) -> Parser a -> Parser b
(<$)  :: a -> Parser b -> Parser a

-- * Applicative
pure  :: a -> Parser a
(<*>) :: Parser (a -> b) -> Parser a -> Parser b
(<*)  :: Parser a -> Parser b -> Parser a -- run both in sequence, result of first
(*>)  :: Parser a -> Parser b -> Parser b -- similar to above

-- * Alternative
empty :: Parser a
(<|>) :: Parser a -> Parser a -> Parser a -- orElse
many  :: Parser a -> Parser [a] -- zero or more
some  :: Parser a -> Parser [a] -- one or more (should be NonEmpty) 

-- * Monadic
(>>=) :: Parser a -> (a -> Parser b) -> Parser b  -- andThen
```

UPD: еще полезно иметь:
```
string :: String -> Parser String  -- like 'char' but for string
oneOf  :: [String] -> Parser String  -- parse first matched string from list
```

Примеры:
```
ghci> runP (ord <$> char 'A') "A"
Just (65,"")


-- аппликатив стайл читать так:  f <$> c1 <*> c2
-- возьми функцию f, примени к значению в контексте c1 -- получи функцию в контексте,
-- и примени ее к значению в контексте c2
ghci> runP ((\x y -> [x, y]) <$> char 'a' <*> char 'b') "abc"
Just ("ab","c")
ghci> runP ((\x y -> [x, y]) <$> char 'a' <*> char 'b') "xxx"
Nothing


ghci> runP (char 'a' <* eof) "a"
Just ('a',"")
ghci> runP (char 'a' <* eof) "ab"
Nothing


ghci> runP (many $ char 'a') "aaabcd"
Just ("aaa","bcd")
ghci> runP (many $ char 'a') "xxx"
Just ("","xxx")
ghci> runP (some $ char 'a') "xxx"
Nothing


ghci> runP (char 'a' <|> char 'b') "abc"
Just ('a',"bc")
ghci> runP (char 'a' <|> char 'b') "bca"
Just ('b',"ca")
ghci> runP (char 'a' <|> char 'b') "cab"
Nothing
```


### Более умный пример
Распарсить ответ пользователя: [y/n]
```
data Answer = Yes | No

yesP :: Parser Answer
yesP = Yes <$ oneOf ["y", "Y", "yes", "Yes", "ys"]
-- тут проблема, тк oneOf идет до первого удачного парсинга => 
-- надо писать сначала длинные строки, а потом короткие :)

noP :: Parser Answer
noP = No <$ oneOf ["n", "N", "no", "No"]

answerP :: Parser Answer
answerP = yesP <|> noP
```

### Библиотеки
Это все круто и понятно, что мы будем писать все это руками, но есть уже готовые либы:
* parsec -- old as fuck
* attopaarsec -- fast, but poor error msgs, backtracks by default
  Бэктрекинг -- это про альтернативу -- если левый парсер падает, то есть два варианта:
    - запустить правый парсер на изначальной строке -- backtracking
    - запустить правый парсер на пожованном остатке строке -- no backtracking
* megaparsec -- hype


## Тестирование

### Библиотеки
* hspec — declarative unit testing
* hedgehog — property-based testing
* tasty — testing framework for combining different approaches
  - tasty-hspec — tasty provider for hspec
  - tasty-hedgehog — tasty provider for hedgehog

Тэйсти позволяет связывать тестовые типы из различных библиотек в свой тип TestTree. После берет нужный провайдер, специфицирует тип, запускает тесты

### Unit тесты

```
module Test.Unit
       ( hspecTestTree
       ) where

import Data.Maybe (isJust, isNothing)

import Test.Tasty (TestTree)
import Test.Tasty.Hspec (Spec, describe, it, shouldBe, shouldSatisfy, testSpec)

import Parser (char, eof, runP)

hspecTestTree :: IO TestTree
hspecTestTree = testSpec "Simple parser" spec_Parser

-- принято писать строго так:
spec_Parser :: Spec
spec_Parser = do
  describe "eof works" $ do
    it "eof on empty input" $
      runP eof "" `shouldSatisfy` isJust
    it "eof on non-empty input" $
      runP eof "x" `shouldSatisfy` isNothing
  describe "char works" $ do
    it "char parses character" $
      runP (char 'a') "abc" `shouldBe` Just ('x', "bc") -- it fails
```
Запускать так:
```
module Main where

import Test.Tasty (defaultMain, testGroup)

import Test.Unit (hspecTestTree)

main :: IO ()
main = hspecTestTree >>= \unitTests ->
       let allTests = testGroup "Parser" [unitTests]
       in defaultMain allTests
```

### Property-based тесты

Юнит тесты норм, но по сути мы хардкодим все результаты -- как следствие, можем спокойно что-то упустить. Вариант круче -- тестировать свойства функций. Библиотека hedgehog позволяет под описанное свойство как-то нагенерить выборок и проверять их. Понятно, что это намного мощнее, чем юнит тесты, но и намного сложнее, очеидно, тоже

Пример свойства:
```
forall xs : reverse(reverse xs) == xs
```
Пишем генератор и проперти:
```
import Hedgehog

import qualified Data.List as List
import qualified Hedgehog.Gen as Gen
import qualified Hedgehog.Range as Range

genIntList :: Gen [Int]
genIntList =
  let listLength = Range.linear 0 100000
  in  Gen.list listLength Gen.enumBounded

prop_reverse :: Property
prop_reverse = property $
  forAll genIntList >>= \xs ->
  List.reverse (List.reverse xs) === xs
```

### Shrinking
Прикольная штука, которая позволяет при падении проперти теста найти наименьший аналог для него, чтобы тест выглядел человеко-читаемым для ручного дебага далее

### Юз-кейсы для проперти-бэйсд тестированиы:
1. Round-trip properties:
    * read        . show      ≡ id
    * decode      . encode    ≡ id
    * deserialize . serialize ≡ id
2. Type classes laws:
    * (a <> b) <> c ≡ a <> (b <> c)
    * a <> mempty ≡ a
    * mempty <> a ≡ a

Но вот такое ```(m >>= f) >>= g ≡ m >>= (\x -> f x >>= g)``` кажется уже сложным закодить

Пример:
```
module Test.Property (okTestTree) where

import Hedgehog (Gen, Property, forAll, property, (===))
import Test.Tasty (TestTree)
import Test.Tasty.Hedgehog (testProperty)

import Parser (ok, runP)

import qualified Hedgehog.Gen as Gen
import qualified Hedgehog.Range as Range

okTestTree :: TestTree
okTestTree = testProperty "ok always succeeds" prop_Ok

genString :: Gen String
genString =
  let listLength = Range.linear 0 100
  in  Gen.list listLength Gen.alpha

prop_Ok :: Property
prop_Ok = property $
  forAll genString >>= \s ->
  runP ok s === Just ((), s)
```
