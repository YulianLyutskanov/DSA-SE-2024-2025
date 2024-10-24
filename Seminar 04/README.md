# Семинар №4 - Алгоритми за търсене
**Изчислителната задача търсене** изисква да се даде отговор на въпроси от вида "дали елемент X се съдържа в дадена колекция?" или "на коя позиция се намира елемента X?". Ще разгледаме някои от най-често използваните алгоритми, които отговарят на този въпрос.

## Линейно търсене
Най-базовият и прост за работа алгоритъм за търсене е **Linear Search**. За произволна колекция преминаваме последователно през всеки един елемент и ако намерим търсеният ключ, можем да отговорим на поставеният в задачата въпрос (да дадем индикация за наличието му или да върнем позицията му).

```c++
template <class T> 
int linear_search(const std::vector<T>& data, const T& el)
{
    for (int i = 0; i < data.size(); i++)
    {
        if (data[i] == el)
            return i;
    }

    return -1;
}
```

Анализ на сложността:

* Сложност по време в най-добрият случай: **Theta(1)**
  * Когато елементът е първи в колекцията правим точно 1 проверка.
 
* Сложност по време в среден случай: **Theta(n)**
  * Средният случай се пресмята на базата на математическо очакване. On average очакваната ("средна") позиция, на която може да се намира елемента е с номер `(n + 1) / 2`. Тогава алгоритъмт би направил толкова на брой стъпки. Игнорирайки мултипликативните и адитивните константи получаваме линейна сложност.

* Сложност по време в най-лошият случай: **Theta(n)**
  * Когато елементът липсва в колекцията правим точно n + 1 стъпки.

* Сложност по памет: **Theta(1)**

Този алгоритъм работи независимо от подредбата на данните в него. Тъй като работим с несортирани данни, тривиално наблюдение е, че броят на сравненията няма как да падне под броя на елементите. На всяка стъпка обаче правим и допълнително сравнение дали не сме преминали края на масива. Възможна оптимизация, която няма да промени асимптотиката, но ще ускори работата на алгоритъма, е елиминирането на тази проверка посредством използването на **сентинел**.

```c++
template <class T> 
int linear_search_sentinel(const std::vector<T>& data, const T& el)
{
    T last = data.back();
    data.back() = el;

    int i = 0;
    while (el != data[i])
    {
        i++;
    }
    
    data.back() = last;

    if(i < data.size() - 1 || data.back() == el)
        return i;

    return -1;
}
```

## Двоично търсене
Без да имаме допълнителна информация за входната колекция, трудно бихме могли да дадем отговор на задачата "търсене" без да направим проверка за всеки един елемент (в най-лошият случай). Какво ще стане обаче, ако имаме допълнителна информация?

Ще разгледаме задачата "търсене", когато знаем, че входната колекция е сортирана.

Тогава можем да съставим алгоритми, които работят асимптотично по-бързо от линейното търсене. Ще започнем с добре познатият ни **Binary Search**. На всяка стъпка, той елиминира половината от текущият масив на база търсеният елемент дали е по-голям или по-малък от текущо изследваният. Това се дължи именно на факта, че масивът е сортиран. Ако елементът ключ е по-голям от текущия изследван, то той ще бъде по-голям и от всички преди него. Така можем да се фокусираме само над оставащите. За да махаме по възможно най-много елементи, винаги ще избираме средният елемент сред оставащите (така че независимо дали новият изследван елемент е по-голям или по-малък, винаги ще елиминираме точно половината елементи).

```c++
template <class T>
int binary_search(const std::vector<T>& data, T el)
{
    int beg = 0, end = data.size() - 1;

    while (beg <= end)
    {
        int mid = beg + (end - beg) / 2;

        if (data[mid] == el)
            return mid;

        if (el < data[mid])
            end = mid - 1;
        else
            beg = mid + 1;
    }

    return -1;
}
```

Анализ на сложността:

* Сложност по време в най-добрият случай: **Theta(1)**
  * Когато елементът е точно в средата.
 
* Сложност по време в среден случай: **Theta(log(n))**

* Сложност по време в най-лошият случай: **Theta(log(n))**
  * Когато елементът липсва в колекцията. Тогава започвайки от n елемента на всяка стъпка цепим общият брой оставащи на две. Ако представим n като 2 на степен k, броят операции ще е (приблизително) k. А от равенството `n = 2^k` следва, че `k = log(n)`.
  
* Сложност по памет: **Theta(1)**

## STL Функции за търсене
Ще разгледаме някои от функции за търсене, налични в STL:

1) `std::find` - **Linear Search**, който приема итератор към началото, итератор към края и търсен елемнет. Резултатът от изпълнението е итератор към елемента, ако е намерен, или към края, ако го няма.

```c++
template<class RandomIt, class T = typename std::iterator_traits<RandomIt>::value_type>
constexpr RandomIt find(RandomIt first, RandomIt last, const T& value)
{
    for (; first != last; ++first)
    {
        if (*first == value)
            return first;
    }
 
    return last;
}
``` 

2) `std::find_if` - **Linear Search**, който приема итератор към началото, итератор към края и унарен предикат. В този си вид търсим елемент, който отговаря на подадения предикат. Резултатът от изпълнението е итератор към елемента, ако е намерен, или към края, ако го няма.

```c++
template<class RandomIt, class UnaryPred>
const RandomIt find_if(RandomIt first, RandomIt last, UnaryPred p)
{
    for (; first != last; ++first)
        if (p(*first))
            return first;
 
    return last;
}
``` 

3) `std::find_if_not` - **Linear Search**, който приема итератор към началото, итератор към края и унарен предикат. В този си вид търсим елемент, който НЕ отговаря на подадения предикат. Резултатът от изпълнението е итератор към елемента, ако е намерен, или към края, ако го няма.

```c++
template<class RandomIt, class UnaryPred>
const RandomIt find_if_not(RandomIt first, RandomIt last, UnaryPred p)
{
    for (; first != last; ++first)
        if (!p(*first))
            return first;
 
    return last;
}
```

4) `std::lower_bound` - **Binary Search**, който приема итератор към началото, итератор към края и търсен елемент. Резултатът е итератор към първият елемент, който е **по-голям или равен** на търсения.

```c++
template <class RandomIt, class T = typename std::iterator_traits<RandomIt>::value_type>
RandomIt lower_bound(RandomIt beg, RandomIt end, const T& value) {
    while (beg < end) {
        RandomIt mid = beg + (end - beg) / 2;

        if (*mid < value) 
            beg = std::next(mid);
        else
            end = mid;
    }
    return beg;
}
```

5) `std::upper_bound` - **Binary Search**, който приема итератор към началото, итератор към края и търсен елемент. Резултатът е итератор към първият елемент, който е **по-голям** от търсения.

```c++
template <class RandomIt, class T = typename std::iterator_traits<RandomIt>::value_type>
RandomIt upper_bound(RandomIt beg, RandomIt end, const T& value) {
    while (beg < end) {
        RandomIt mid = beg + (end - beg) / 2;

        if (*mid <= value) 
            beg = std::next(mid);
        else
            end = mid;
    }
    return beg;
}
```

## Техника - Binary Search по отговор
**Binary Search по отговор** е техника за решаване на оптимизационни изчислителни задачи. Използва се, когато:

1) Можем да проверим дали дадена стойност е решение на задачата
2) Възможните стойности са ограничени отгоре и отдолу

Тогава вемсто да търсим при каква конфигурация се получава оптималното решение, проверяваме дали дадена стойност е решение, като това се случва посредством двоично търсене измежду ограниченото множество от кандидат-решенията.

## Троично търсене
**Ternary Search** е друг алгоритъм за търсене, много сходен с работата на Binary Search. Работата му е аналогична, но вместо да избираме всеки път средата на колекция, той разбива колекцията на 3 равни по големина парчета, като прави 2 сравнения. На база тези две сравнения той може да елиминира 2/3 от колекцията. По аналогични съображения като за Binary Search сложността му в най-лошият случай е логаритмична (тъй като основата на логаритъма не ни интересува в асимптотичния анализ), но работата му върху големи колекции е по-бърза от тази на обикновеното двоично търсене, тъй като отхърля повече данни за една стъпка.

```c++
int ternarySearch(const int* arr, int len, int x) {
  int left = 0;
  int right = len - 1;
  while (right >= left) {
        int mid1 = left + (right - left) / 3; 
        int mid2 = right - (right - left) / 3;
 
        if (arr[mid1] == x) return mid1; 
        if (arr[mid2] == x) return mid2; 

        if (x < arr[mid1]) 
            right = mid1 - 1;
        else if (x > arr[mid2]) 
            left = mid2 + 1;
        else {
            left = mid1 + 1;
            right = mid2; 
        }
    }
    return -1;
}
```

Полезно свойство, което **Ternary Search** има е, че може да се използва и за колекции, които са подредени по следният начин:

* Вариант 1: от 1 до i-ти елемент са растящи, а след i-ти са намаляващи
* Вариант 2: от 1 до i-ти елемент са намаляващи, а след i-ти са растящи

(За разлика от **Binary Search**, който работи единствено с монотонно подредени колекции)

## Jump Search
Notes: Когато връщането назад е бавна операция

## Exponential Search
Notes: Когато търсеният елемент е близо до началото на колекцията или когато не знаем размера на входа (пример - поток от данни).

## Решаване на задачи за търсене

[01. Insert -1](./Solutions/Insert%20-1/Task.md)

[02. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)

Пробвайте да решите вкъщи:
* [Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/description/)
* [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array/description/)

[03. Perfect Printer](https://www.hackerrank.com/contests/sda-homework-3/challenges/challenge-2674) - Binary Search по отговор

[04. Search Insert Position](https://leetcode.com/problems/search-insert-position/description/) - Използване на bound функции от STL

[05. Coast Lifeguard](./Solutions/Coast%20Lifeguard/description.pdf)

[06. Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/description/)
