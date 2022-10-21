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
