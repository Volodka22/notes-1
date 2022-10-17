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