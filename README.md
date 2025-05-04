## Ответ
PPPSD{haskell4life_p34n0_numbers_as_lists4w1n_e160d0248b93cdb092_1!1}


## Решение:

Сначала надо понять, что делает оригинальная программа:

```
main :: IO ()
main =
  putStrLn answer
  where
    answer = zipWith foo bar numbers
    foo = curry $ chr . uncurry (-) . first ((`mod` 1021) . e)
    bar = c1 : c2 : zipWith f bar (tail bar)
```

`putStrLn answer` - выводит answer, т.е. результат работы программы

`answer = zipWith foo bar numbers` - answer — результат рыботы zipWith, т.е. поэлементного сочетания списков одинаковой длины bar и numbers при помощи функции foo.

`foo` - некоторая функция, как работает определим чуть [позже](#foo)

### Функции для вычисления:

```haskell
--создает пустой список z = 0
z :: [a]
z = []

--Удлинняет список на 1
s :: [()] -> [()]
s x = () : x

--Определяет длинну списка рекурсивно вызывая функцию e и прибавляя единицу или выводя ноль если x == z (пустой список)
e :: (Eq a, Num p) => [a] -> p
e x | x == z = 0
e (_ : xs) = (1 +) $ e xs
```

Т.е. в Python можно использовать:

```Python
def e_len(x):
  return len(x)
```
Дальше почитав код можно понять, что на самом деле программа работает не с самими списками и их элементами а лишь с их длинами:

```haskell
-- функция складывает списки
a :: [a] -> [a] -> [a]
a x y
  | x == z    = y
  | otherwise = a (tail x) (head x : y)

-- функция вычитает списки
sb :: [b] -> [a] -> [b]
sb x y
  | y == z    = x
  | otherwise = sb (tail x) (tail y)

-- умножение: повторяем сложение x столько раз, сколько единиц в y
m :: [b] -> [a] -> [b]
m x y
  | y == z    = z
  | otherwise = a x (m x (tail y))

-- сравнение: True, если length x >= length y
c :: [b] -> [a] -> Bool
c _ y | y == z    = True
c x _ | x == z    = False
c (_:xs) (_:ys)   = c xs ys
```

Тогда эти функции для ускорения можно перевсти без рекурсии на Python как:
```Python
def a_len(x, y):    return x + y       # сложение int
def sb_len(x, y):   return x - y       # вычитание int
def m_len(x, y):    return x * y       # умножение int
def c_len(x, y):    return x >= y      # сравнение int
```

### foo

```
foo = curry $ chr . uncurry (-) . first ((`mod` 1021) . e)
```
К списку сначала применяется функция `e` — получение длины списка (обфускация работы с числами на основе длин списков)

Затем от этой длины берётся модуль 1021 
```haskell
(`mod` 1021)
```

Далее из этого числа вычитается число из списка numbers:

```
uncurry (-) . first
```

При помощи `chr .` число переводится в `Unicode`

На Python этот фрагмент можно перевести как-то так:

```Python
message = ''.join(
    chr((bar[i] % 1021) - numbers[i])
    for i in range(len(numbers))
)
print(message)
```
### bar
```haskell
-- применяя функцию f рекурентно строит элементы используя предыдущий и начиная с заранее заданных значений c1 и c2, где c1 = 233, c2 = 666 (после обработки функцией e). Вызов функции f получается на подобии: f(bar[0], bar[1]) → bar[2], f(bar[1], bar[2]) → bar[3], ..., bar[i] = f(bar[i-2], bar[i-1]), ...
bar = c1 : c2 : zipWith f bar (tail bar) 


--If ветвление:
--x+y <=d1 => bar = x-y
--x+y >= d2 => bar = x+y
--else: рекурсивное вычисление x*y пока не заработает какая-то ветка.
f x y
  | d1 `c` a x y = sb x y
  | a x y `c` d2 = a x y
  | otherwise   = f x (m x y)
```

На Python это будет выглядеть как-то так:
```Python
def f_len(x, y):
    if c_len(d1_len, a_len(x, y)):       # если x+y <= d1
        return sb_len(x, y)              # — вычесть
    elif c_len(a_len(x, y), d2_len):     # иначе если x+y >= d2
        return a_len(x, y)               # — сложить
    else:
        return f_len(x, m_len(x, y))     # иначе — умножить и повторить

# строим bar:
bar = [c1_len, c2_len]
for i in range(2, N):
    bar.append(f_len(bar[i-2], bar[i-1]))
```

Тогда чтобы ускорить вычисление флага достаточно избавиться от рекурсий и операций со списками:

```Python
haskell_code = '''
import Control.Arrow (Arrow (first))
import Data.Char (chr)

main :: IO ()
main =
  putStrLn answer
  where
    answer = zipWith foo bar numbers
    foo = curry $ chr . uncurry (-) . first ((`mod` 1021) . e)
    bar = c1 : c2 : zipWith f bar (tail bar)

f :: Eq a => [a] -> [a] -> [a]
f x y
  | d1 `c` a x y = sb x y
  | a x y `c` d2 = a x y
  | otherwise = f x $ m x y

z :: [a]
z = []

s :: [()] -> [()]
s x = () : x

a :: Eq a => [a] -> [a] -> [a]
a x y | x == z = y
a (x : xs) y = a xs (x : y)

sb :: Eq a => [b] -> [a] -> [b]
sb x y | y == z = x
sb (_ : xs) (_ : ys) = sb xs ys

m :: (Eq a, Eq b) => [b] -> [a] -> [b]
m x y | y == z = z
m x (y : ys) = a x $ m x ys

c :: (Eq a, Eq b) => [b] -> [a] -> Bool
c _ y | y == z = True
c x _ | x == z = False
c (_ : xs) (y : ys) = c xs ys

e :: (Eq a, Num p) => [a] -> p
e x | x == z = 0
e (_ : xs) = (1 +) $ e xs

numbers = [153, 586, 819, 461, 354, 843, 263, 215, 564, 884, 548, 511, 139, 814, -16, 853, -73, 886, 921, 870, 926, 886, 784, 763, 589, 364, 20, 502, 650, 237, -49, 288, 373, 774, 203, 73, 378, 549, 4, 657, 777, 592, 396, 89, 543, 696, 322, 144, 562, 761, 304, 144, 546, 736, 307, 32, 436, 572, -4, 618, 715, 462, 245, 762, -2, 856, 965, 833, 734]

c1 = s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s z

c2 = s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s z

d1 = s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s z

d2 = s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s $ s z

'''


def count_s(var_name):
    lines = haskell_code.splitlines()
    total = 0
    recording = False
    for line in lines:
        stripped = line.strip()
        if stripped.startswith(f"{var_name} ="):
            recording = True
        if recording:
            total += line.count('s')
            # считаем, что определение заканчивается, когда строчка оканчивается на "z"
            if stripped.endswith("z"):
                break
    if not recording:
        raise ValueError(f"Не нашёл определение {var_name}")
    return total

c1_len = count_s('c1')
c2_len = count_s('c2')
d1_len = count_s('d1')
d2_len = count_s('d2')

print("c1_len =", c1_len)
print("c2_len =", c2_len)
print("d1_len =", d1_len)
print("d2_len =", d2_len)


def a_len(x, y):   return x + y
def sb_len(x, y):  return x - y
def m_len(x, y):   return x * y
def c_len(x, y):   return x >= y

def f_len(x, y):
    if c_len(d1_len, a_len(x, y)):
        return sb_len(x, y)
    elif c_len(a_len(x, y), d2_len):
        return a_len(x, y)
    else:
        return f_len(x, m_len(x, y))

numbers = [153, 586, 819, 461, 354, 843, 263, 215, 564, 884, 548, 511, 139, 814, -16, 853, -73, 886, 921, 870, 926, 886, 784, 763, 589, 364, 20, 502, 650, 237, -49, 288, 373, 774, 203, 73, 378, 549, 4, 657, 777, 592, 396, 89, 543, 696, 322, 144, 562, 761, 304, 144, 546, 736, 307, 32, 436, 572, -4, 618, 715, 462, 245, 762, -2, 856, 965, 833, 734]


bar = [c1_len, c2_len]
for i in range(2, len(numbers)):
    bar.append(f_len(bar[i-2], bar[i-1]))


message = ''.join(
    chr((bar[i] % 1021) - numbers[i])
    for i in range(len(numbers))
)

print("\nПолученный текст сообщения:\n")
print(message)
```
