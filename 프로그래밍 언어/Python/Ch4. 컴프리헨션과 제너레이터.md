# Ch4. 컴프리헨션과 제너레이터

 파이썬에서는 데이터를 쉽게 담아낼 수 있는 `컴프리헨션`이라는 특별한 문법이 존재한다. `컴프리헨션`은 리스트나 딕셔너리, 집합 등의 파이썬 자료구조에 대해 반복문과 조건문을 결합하여 하나의 구문을 만들어 냄으로써, 이를 사용하면 간결하게 이터레이션을 할 수 있고 코드의 가독성도 높일 수 있다.

 `컴프리헨션` 스타일은 `제너레이터`를 사용하는 함수로 확장할 수 있다. `제너레이터`는 파이썬에서 고성능이면서 메모리를 절약하고 가독성을 높이는 형태의 반복을 위한 방법으로, 이를 사용하면 함수가 점진적으로 반환하는 값으로 이뤄지는 스트림을 만들어준다. `제너레이터 함수` (yield를 통해 반환함)를 호출하면 `제너레이터 객체`가 반환되는데, 이 객체는 `이터레이터 프로토콜`을 따르므로 `이터레이터`처럼 사용할 수 있다.

**목차**
- [Ch4. 컴프리헨션과 제너레이터](#ch4-컴프리헨션과-제너레이터)
- [27. map과 filter 대신 컴프리헨션을 사용하라](#27-map과-filter-대신-컴프리헨션을-사용하라)
- [28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라](#28-컴프리헨션-내부에-제어-하위-식을-세-개-이상-사용하지-말라)
- [29. 대입식(왈러스 연산자)을 사용해 컴프리헨션 안에서 반복 작업을 피하라](#29-대입식왈러스-연산자을-사용해-컴프리헨션-안에서-반복-작업을-피하라)
- [30. 리스트를 반환하기보다는 제너레이터를 사용하라](#30-리스트를-반환하기보다는-제너레이터를-사용하라)
- [31. 인자에 대해 이터레이션할 때는 방어적이 돼라](#31-인자에-대해-이터레이션할-때는-방어적이-돼라)
- [32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라](#32-긴-리스트-컴프리헨션보다는-제너레이터-식을-사용하라)
- [33. yield from을 사용해 여러 제너레이터를 합성하라](#33-yield-from을-사용해-여러-제너레이터를-합성하라)
- [34. send로 제너레이터에 데이터를 주입하지 마라](#34-send로-제너레이터에-데이터를-주입하지-마라)
- [35. 제너레이터 안에서 throw로 상태를 변화시키지 마라](#35-제너레이터-안에서-throw로-상태를-변화시키지-마라)
- [36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라](#36-이터레이터나-제너레이터를-다룰-때는-itertools를-사용하라)
    - [여러 이터레이터 연결하기](#여러-이터레이터-연결하기)
    - [이터레이터에서 원소 거르기](#이터레이터에서-원소-거르기)
    - [이터레이터에서 원소의 조합 만들어내기](#이터레이터에서-원소의-조합-만들어내기)

# 27. map과 filter 대신 컴프리헨션을 사용하라

 파이썬에서는 다른 시퀸스나 이터러블에서 새 리스트를 만들어내는 간결한 구문을 제공하는데, 이를 `리스트 컴프리헨션`이라고 한다. 예제로, 리스트에 있는 모든 원소의 제곱을 출력한다고 해보자.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
squares = list()
for x in a:
    squares.append(x**2)
print(squares)
```

```python
# 실행결과
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

이를 `리스트 컴프리헨션`을 사용해, 루프에 처리 대상인 입력 시퀸스 원소에 적용할 반환식을 지정하여 동일한 결과를 얻어낼 수 있다.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
squares = [x**2 for x in a]
print(squares)
```

```python
# 실행결과
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

간단한 경우에도 `map` 함수보다는 `리스트 컴프리헨션`이 더 명확하다. `lamdba` 함수를 정의하는 것이 가독성이 떨어지기 때문이다.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
squares = list(map(lambda x: x**2, a))
print(squares)
```

 `리스트 컴프리헨션`은 루프를 단순화하는 것을 넘어서, 입력 리스트에서 쉽게 조건 처리하여 원하는 원소를 제거할 수도 있다. 예를 들어 리스트 내에서 짝수의 제곱만 출력하고 싶다고 하자.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
even_squares = [x**2 for x in a if x%2 == 0]
print(even_squares)
```

```python
# 실행 결과
[4, 16, 36, 64, 100]
```

이와 동일한 기능을 map으로 구현하면, 아래처럼 가독성이 떨어지는 코드를 작성하게 된다.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
alt = list(map(lambda x: x**2, filter(lambda x: x%2 == 0, a)))
print(alt)
```

 리스트 컴프리헨션 이외에도 딕셔너리 컴프리헨션과 집합 컴프리헨션이 존재한다. 위와 비슷한 예제를 출력하는 것으로 확인해보자.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
even_squares_list = [x**2 for x in a if x%2 == 0]
even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
threes_cubed_set = {x**3 for x in a if x % 3 == 0}

print(even_squares_list)
print(even_squares_dict)
print(threes_cubed_set)
```

```python
#실행 결과
[4, 16, 36, 64, 100]
{2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
{216, 729, 27}
```

한편, 동일한 기능을 map과 filter를 사용하면 아래처럼 가독성이 떨어지게 작성해야만 한다.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9,10]
even_squares_list = list(map(lambda x: x**2, filter(lambda x: x%2 == 0, a)))
even_squares_dict = dict(map(lambda x: (x, x**2), filter(lambda x: x%2 == 0, a)))
threes_cubed_set = set(map(lambda x : x**3, filter(lambda x : x%3 == 0, a)))

print(even_squares_list)
print(even_squares_dict)
print(threes_cubed_set)
```

그러므로 map과 filter를 활용하기보다는, 이것의 상위호환 격의 `컴프리헨션`을 적극 활용하자.

# 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라

컴프리헨션으로 다중 루프도 구현할 수 있다. 컴프리헨션에 들어간 순서대로 왼쪽에서 오른쪽으로 하위 식이 실행된다. 아래는 이중 리스트를 일차원 리스트로 변환하는 예제 코드이다.

```python
matrix = [[1,2,3],[4,5,6],[7,8,9]]
flat = [x for row in matrix for x in row]
print(flat)
```

```python
# 실행결과
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

다중 컴프리헨션을 사용하여 아래와 같이 다중 루프를 사용하여 2차원의 입력 list를 만들어낼 수도 있다.  아래는 이중 리스트의 각 원소를 제곱하는 예제 코드이다. 

```python
matrix = [[1,2,3],[4,5,6],[7,8,9]]
squared = [[x**2 for x in row] for row in matrix]
print(squared)
```

```python
# 실행결과
[[1, 4, 9], [16, 25, 36], [49, 64, 81]]
```

느낌 상 외부 컨프리헨션이 먼저 실행되고 그 다음에 내부 컴프리헨션이 실행되는 듯하다. 그래야 로컬변수 row의 동작이 말이 되지.

하지만 만약 3개 이상의 단일 루프가 이어진 꼴이라면, 컴프리헨션은 가독성을 떨어뜨리는 선택지이다.

```python
matrix = [[[1,2,3],[4,5,6]],
          [[7,8,9],[10,11,12]]]
myList = [x for sublist1 in matrix 
           for sublist2 in sublist1 
           for x in sublist2]
print(myList)
```

```python
# 실행결과
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
```

이럴 바에는 차라리 일반 루프문을 쓰는게 더 가독성이 좋다.

```python
matrix = [[[1,2,3],[4,5,6]],
          [[7,8,9],[10,11,12]]]
myList = list()
for sublist1 in matrix:
    for sublist2 in sublist1:
        myList.extend(sublist2)
print(myList)
```

```python
# 참고: 리스트의 extend 함수는 iterable한 요소를 리스트에 추가하는 함수.
my_list = [1, 2, 3]
another_list = [4, 5, 6]

my_list.extend(another_list)
print(my_list)  # 출력: [1, 2, 3, 4, 5, 6]
```

그리고 컴프리헨션에서는 여러 개의  if를 사용할 수 있다. 같은 수준의 루프에 여러 조건을 사용하게 되면 암시적으로 and와 동일한 기능을 하게 된다.

```python
a = [1,2,3,4,5,6,7,8,9,10]
b = [x for x in a if x > 4 if x % 2 == 0]
print(b)
```

```python
# 실행결과
[6, 8, 10]
```

또한 각 수준의 for 하위 식에 if를 추가해주어 수준 별로 조건을 제시할 수 있다.

```python
matrix =  [[1,2,3],[4,5,6],[7,8,9]]
filtered = [[x for x in row if x % 3 == 0] for row in matrix if sum(row)>= 10]
print(filtered)
```

```python
# 실행결과
[[6], [9]]
```

하지만 위 코드가 가독성이 좋다고는 말할 수 없다. 우리는 컴프리헨션을 가독성이 좋게 쓰는 것이 목표이다. 그러기 위해서는 아래의 원칙을 지켜라.

- **하나의 컴프리헨션 안에는 루프와 조건문을 포함해서 총 2개만 사용할 것.** (다중 컴프리헨션의 하위 식까지 포함해서)
    - 2 루프
    - 1 루프 & 1 조건문
    - 2 조건문

만약 이 조건을 만족시킬 수 없다면 컴프리헨션이 아닌 일반 루프와 조건문을 사용하고, 대신 `도우미 함수`를 사용하는 것이 가독성을 보장한다. ([30. 리스트를 반환하기보다는 제너레이터를 사용하라](https://www.notion.so/30-61464714af344715b0906b8742e2b316?pvs=21))

# 29. 대입식(왈러스 연산자)을 사용해 컴프리헨션 안에서 반복 작업을 피하라

아래는 주문 관리 프로그램이다. 고객의 요청이 재고 수량을 넘지 않으며, 배송에 필요한 최소 수량을 만족하는지 확인해야 한다. (최소 수량은 ‘8’로 지정. 8개가 배송 가능한 1세트) 출력값은 물건의 이름과, 해당 물건의 제공 가능한 세트의 개수이다.

```python
stock = {
    '못' : 125,
    '나사못' : 35,
    '나비너트' : 8,
    '와셔' : 24,
}

order = ['나사못', '나비너트', '클립']

def get_batches(count, size):
    return count // size

result = dict()
for name in order:
    count = stock.get(name, 0)
    batches = get_batches(count, 8)
    if batches:
        result[name] = batches

print(result)
```

```python
# 실행결과
{'나사못': 4, '나비너트': 1}
```

위 코드는 딕셔너리 컴프리헨션을 통해 간결하게 표현이 가능하다.

```python
stock = {
    '못' : 125,
    '나사못' : 35,
    '나비너트' : 8,
    '와셔' : 24,
}

order = ['나사못', '나비너트', '클립']

def get_batches(count, size):
    return count // size

result = {name : get_batches(stock.get(name, 0), 8)
         for name in order
         if get_batches(stock.get(name, 0), 8)}
print(result)
```

result 대입문에서 get_batches가 불필요하게 중복이 되기 때문에, `왈러스 연산자(:=)`를 활용하여 다시 정리해보자.

```python
result = {name : batches for name in order
          if (batches := get_batches(stock.get(name,0), 8))}
```

왈러스 연산자를 사용하니 컴프리헨션 구문이 깔끔하게 정리되는 모습이다.

하지만 **컴프리헨션에 조건문 외의 영역에 왈러스 연산자를 사용하면, 컴프리헨션의 루프 밖 영역으로 루프 변수가 누출되는 문제가 생긴다. 변수가 누출되면 버그가 터질 수 있으므로 나쁜 상황이다.** (Better way 21 참고)

```python
stock = {
    '못' : 125,
    '나사못' : 35,
    '나비너트' : 8,
    '와셔' : 24,
}

order = ['나사못', '나비너트', '클립']

half = [(last := count // 2) for count in stock.values()]
print(f'{half}의 마지막 원소는 {last}')
```

```python
# 실행결과
[62, 17, 4, 12]의 마지막 원소는 12
```

**그러나 조건문에 왈러스 연산자를 사용할 때는 변수가 누출되지 않는다.**

```python
stock = {
    '못' : 125,
    '나사못' : 35,
    '나비너트' : 8,
    '와셔' : 24,
}

order = ['나사못', '나비너트', '클립']

half = [last for count in stock.values() if (last := count // 2)]
print(half)
print(count)
```

```python
# 실행결과
[62, 17, 4, 12]
NameError: name 'count' is not defined.
```

마찬가지로 기존의 컴프리헨션의 루프 변수에 대해서는 변수가 누출되지 않는다. 

```python
stock = {
    '못' : 125,
    '나사못' : 35,
    '나비너트' : 8,
    '와셔' : 24,
}

order = ['나사못', '나비너트', '클립']

half = [count // 2 for count in stock.values()]
print(count)
```

```python
# 실행결과
NameError: name 'count' is not defined.
```

**그러므로, 왈러스 연산자는 조건문에 대해서만 사용하도록 한다.**

# 30. 리스트를 반환하기보다는 제너레이터를 사용하라

아래는 문자열에서 찾은 단어의 인덱스를 반환하는 예제 코드이다. 공백을 활용하여 문자 인덱스의 위치를 추적한다. append 메서드를 사용하여 리스트에 결과값을 추가하고, 함수 마지막에서는 이 리스트를 반환한다.

```python
def index_words(text):
    result = list()
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index+1)
    return result

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어: 전산기)는 진공관'
result = index_words(address)
print(result[:10])
```

```python
# 실행결과
[0, 8, 18, 23, 28, 33, 39]
```

```python
# 참고: enumerate 함수는 이터러블 객체를 (인덱스, 값) 형태의 튜플들로 생성하여 반환한다.
fruits = ['apple', 'banana', 'orange']

for index, value in enumerate(fruits):
    print(index, value)

# 출력:
# 0 apple
# 1 banana
# 2 orange

# 위 예제처럼 문자열의 경우에는 한 글자씩 튜플을 생성하여 반환한다.
```

하지만 리스트로 구현된 이 코드는 2개의 문제를 갖는다.

1. 가독성이 현저히 떨어진다.
2. 불필요하게 리스트를 만들어야 한다. 그래서 입력이 매우 크면 메모리 고갈로 중단될 수 있다.

위 문제들은 `제너레이터` 를 사용하면 해결할 수 있다. 제너레이터는 `yield` 식을 사용하는 함수에 의해 만들어진다.

```python
# 제너레이터는 `stream`으로 값을 여러 번에 걸쳐서 즉시 생성하므로, 데이터를 메모리에 로드하지 않는다.
def number_generator(start, end):
    current = start
    while current <= end:
        yield current
        current += 1

numbers = number_generator(1, 5)

for num in numbers:
    print(num)

# 출력:
# 1
# 2
# 3
# 4
# 5
```

```python
def index_words(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어: 전산기)는 진공관'
result = index_words(address)
print(next(result))
print(next(result))
print(next(result))
```

```python
# 실행결과
0
8
18
```

제너레이터가 반환하는 이터레이터를 리스트에 담아내면, 제너레이터 반환값을 리스트로 담아낼 수 있다. ([32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라](https://www.notion.so/32-6860d00ac7e74148ae4128326eeb649f?pvs=21) )

```python
... 

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어: 전산기)는 진공관'
result = list(index_words(address))
print(result)
```

```python
# 실행결과
[0, 8, 18, 23, 28, 33, 39]
```

그리고 **제너레이터 버전으로 만들면 메모리를 절약할 수 있으므로, 입력 길이가 아무리 길어도 손쉽게 처리할 수 있다.** 다음 예제는 파일에서 한 번에 한 줄씩 읽어서 한 단어씩 출력하는 제너레이터를 정의한 코드이다. ([36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라](https://www.notion.so/36-itertools-1a8084ced7824c0c9b469bbdbfd32f37?pvs=21) )

```python
import itertools

def index_file(handle):
    offset = 0
    for line in handle:
        if line:
            yield offset
        for letter in line:
            offset += 1
            if letter == ' ':
                yield offset

# 파일은 동일 디렉토리에 생성해준다. 파일 내용은 위 예제와 동일한 문장을 넣어주자.
with open('address.txt', 'r', encoding='utf-8') as f:
    it = index_file(f)
    results = itertools.islice(it, 0, 10)
    print((list(results)))
```

```
# address.txt
컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어: 전산기)는 진공관
```

```
# 실행결과
[0, 8, 18, 23, 28, 33, 39]
```

```python
# itertools의 islice 함수는 데이터를 슬라이싱하거나 필터링하는 용도로 사용한다.
from itertools import islice

numbers = range(10)  # 0부터 9까지의 숫자

sliced_numbers = islice(numbers, 2, 7, 2)  # 2 ~ 6까지의 숫자들을 간격 2로 슬라이싱

print(list(sliced_numbers))  # [2, 4, 6]
```

**단, 제너레이터가 반환하는 이터레이터에는 상태가 있기 때문에 호출자 측에서 재사용이 불가능하다는 점에 주의하도록 하자.** ([31. 인자에 대해 이터레이션할 때는 방어적이 돼라](https://www.notion.so/31-7cdf3f11750341a4b21da96859d9656b?pvs=21) )

# 31. 인자에 대해 이터레이션할 때는 방어적이 돼라

미국 텍사수 주의 여행자 수를 분석하고 싶다고 하자. 데이터 집합이 도시별 방문자 수(단위: 100만 명/년)으로 가정했을 때, 각 도시가 전체 여행자 수 중에서 차지하는 비율을 계산하는 예제 코드는 아래와 같다.

```python
def normalize(numbers):
    total = sum(numbers)
    result = list()
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = [15, 35, 80]
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
```

```python
# 실행결과
[11.538461538461538, 26.923076923076923, 61.53846153846154]
```

위 코드는 세 도시에 대해서만 여행자 분석을 진행했지만, 전 세계를 대상으로 여행자 분석을 해야 하는 경우에는 대량의 메모리가 소모될 것이다. 그러므로, 이 경우에는 리스트가 아닌 제너레이터로 정의하는 것이 좋다. ([30. 리스트를 반환하기보다는 제너레이터를 사용하라](https://www.notion.so/30-61464714af344715b0906b8742e2b316?pvs=21) )

```python
def normalize(numbers):
    total = sum(numbers)
    result = list()
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('my_numbers.txt')
percentages = normalize(it)
print(percentages)
```

```
# my_numbers.txt
15
35
80
```

```python
# 실행결과
[]
```

**하지만 이터레이터는 결과를 단 한 번만 만들어내기 때문에, 이미 StopIteration 예외가 발생한 이터레이터나 제너레이터를 다시 이터레이션해보았자 아무런 값도 얻어낼 수 없다.** 아무런 출력이 없는 이터레이터와 이미 소진된 이터레이터를 구분하지 못하기 때문에, 이미 소진된 이터레이터에 대해서 이터레이션을 수행해도 오류가 발생하지 않는다. 

```python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('my_numbers.txt')
print(list(it))
print(list(it))  # 아무런 값도 얻어낼 수 없다.
```

```python
# 실행결과
[15, 35, 80]
[]
```

**이 문제를 해결하기 가장 적합한 방법은 `이터레이터 프로토콜`을 구현한 새로운 컨테이너 클래스를 제공하는 것이다.** `이터레이터 프로토콜` 은 파이썬의 반복문에 관련된 식들이 컨테이너 타입의 내용을 조회할 때 사용하는 절차이다. iter 내장 함수는 ‘foo.__iter__’ 라는 특별 메소드를 호출하는데, ‘__iter__’ 메소드는 반드시 이터레이터 객체를 반환해야만 한다. 참고로 for 루프문은 반환받은 이터레이터 객체가 데이터를 소진할 때까지 반복적으로 이터레이터 객체에 next 내장 함수를 호출하면서 동작한다.

다음은 여행 데이터가 들어 있는 파일을 읽는 이터러블 컨테이너 클래스를 정의하는 코드이다.

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

def normalize(numbers):
    total = sum(numbers)
    result = list()
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = ReadVisits('my_numbers.txt')
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
```

```python
# 실행결과
[11.538461538461538, 26.923076923076923, 61.53846153846154]
```

**normalize 함수 안의 sum 메소드가 ReadVisits.__iter__를 호출하여 새로운 이터레이터 객체를 할당하기 때문에, 데이터가 소진되지 않으므로 정상적으로 출력되는 모습이다.**  ReadVisits.__iter__ 내의 for 문과 normalize 내의 for 문은 둘 다 이터레이터를 소비하지만 서로 독립적으로 진행되고 소진되므로, 이로 인해 두 이터레이터는 모든 입력 데이터 값을 볼 수 있는 별도의 이터레이션을 만들어낼 수 있다. 하지만 이 방식은 입력 데이터를 여러 번 읽는다는 단점을 갖는다.

이터레이터가 iter 내장 함수에 전달되는 경우에는 잔달받은 이터레이터가 그대로 반환된다. 반면 컨테이너 타입이 iter에 전달되면 매번 새로운 이터레이터 객체가 반환된다. 따라서 입력값을 검사하여 반복적으로 이터레이션할 수 없는 인자의 경우 TypeError를 발생시켜 인자를 거부할 수 있다.

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

def normalize(numbers):
    if iter(numbers) is numbers:
        raise TypeError('컨테이너만 허용됩니다.')
    total = sum(numbers)
    result = list()
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = ReadVisits('my_numbers.txt')
percentages = normalize(visits)
print(percentages)
visits = read_visits('my_numbers.txt')
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
```

```python
# 실행결과
[11.538461538461538, 26.923076923076923, 61.53846153846154]
TypeError: 컨테이너만 허용됩니다.
```

# 32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라

파일을 읽어서 각 줄에 들어 있는 문자 수를 반환하고 싶다고 하자. 다음 코드는 리스트 컴프리헨션으로 파일 각 줄의 길이를 메모리에 저장한다.

```python
value = [len(x) for x in open('my_file.txt', encoding='utf-8')]
print(value)
```

```python
# my_file.txt
이제 괜찮니 너무 힘들었잖아
우리 그 마무리가 고작 이별뿐인 건데
우린 참 어려웠어
잘 지낸다고 전해 들었어 가끔
벌써 참 좋은 사람
만나 잘 지내고 있어
굳이 내게 전하더라
```

```python
# 실행결과
[16, 21, 10, 17, 11, 12, 10]
```

하지만 이런 방식은 파일이 아주 크거나 절대로 끝나지 않는 네트워크 소켓이라면 사용하기 까다롭다.

이 문제를 해결하기 위해서 `제너레이터 식` 을 사용한다. **`제너레이터 식`은 리스트 컴프리헨션과 제너레이터를 일반화한 것으로, 제너레이터 식을 실행해도 출력 시퀸스 전체가 실체화되지 않는다.** 대신 제너레이터 식에 들어 있는 식으로부터 원소를 하나씩 만들어내는 이터레이터가 생성된다.

다음 코드는 제너레이터 식을 적용한 코드이다.

```python
# `제너레이터 식`은 필요한 데이터를 실제로 필요한 시점마다 하나씩 생성한다.
# `제너레이터 식`은 리스트 컴프리헨션과 유사한 구조를 갖지만, [] 대신 ()를 사용한다.
numbers = [1, 2, 3, 4, 5]

# 제곱값을 구하는 제너레이터 식
squares = (x**2 for x in numbers)

for square in squares:
    print(square)
#  출력결과
#  1
#  4
#  9
#  16
#  25
```

```python
value = (len(x) for x in open('my_file.txt', encoding='utf-8'))
print(value)
```

```python
# 실행결과
<generator object <genexpr> at 0x00000215F98FDA80>
```

제너레이터 식은 이터레이터로 즉시 평가되기 때문에 더 이상 시퀸스 원소 계산이 진행되지 않는다.

그 대신 **변환된 제너레이터에서 next 내장 함수를 사용하여 메모리를 모두 소모하는 것을 염려할 필요 없이 값을 원하는 대로 가져와 소비할 수 있다.**

```python
value = (len(x) for x in open('my_file.txt', encoding='utf-8'))
print(next(value))
print(next(value))
```

```python
# 실행결과
16
21
```

아래와 같이, 두 제너레이터 식을 합성할 수도 있다. 이 경우, 하나의 제너레이터를 공유하기 때문에 인덱스 포인터(?)를 공유하게 된다.

```python
value = (len(x) for x in open('my_file.txt', encoding='utf-8'))
roots = ((x, x**0.5) for x in value)
print(next(value))
print(next(roots))
```

```python
# 실행결과
16
(21, 4.58257569495584)
```

**이처럼 아주 큰 입력 스트림에 대해 여러 기능을 합성해 적용해야 한다면 `제너레이터 식`이 최고의 선택이 될 것이다.** 다만, 제너레이터가 반환하는 이터레이터는 상태를 가지므로 이터레이터를 한 번만 사용해야 한다. ([31. 인자에 대해 이터레이션할 때는 방어적이 돼라](https://www.notion.so/31-7cdf3f11750341a4b21da96859d9656b?pvs=21) )

# 33. yield from을 사용해 여러 제너레이터를 합성하라

제너레이터를 사용해 화면의 이미지를 움직이게 하는 그래픽 프로그램을 만든다고 하자. 원하는 시각적인 효과를 얻으려면 처음에는 이미지가 빠르게 이동하고 잠시 멈춘 다음, 다시 이미지가 느리게 이동해야 한다. 그래서 이 애니메이션의 각 부분에서 필요한 화면 상 이동변위(delta)를 만들어낼 때 사용할 두 가지 제너레이터를 정의한 함수 move와 pause를 정의한다. 그리고 최종 에니메이션을 만드려면 move와 pause를 합성하여 변위 시퀸스를 하나만 만들어야 한다. 그러므로 애니메이션의 각 단계마다 제너레이터를 호출하여 차례로 이터레이션하고, 각 이터레이션에서 나오는 변위를 순서대로 내보내는 방식으로 시퀸스를 만든다. 마지막으로 이렇게 만든 화면 상의 변위를 단일 애니메이션 제너레이터에서 만들어진 것처럼 화면에 표시한다.

```python
def move(period, speed):
    for _ in range(period):
        yield speed

def pause(delay):
    for _ in range(delay):
        yield 0

def animate():
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta

def render(delta):
    print(f'Delta: {delta:.1f}')

def run(func):
    for delta in func():
        render(delta)

run(animate)
```

```python
# 실행결과
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 0.0
Delta: 0.0
Delta: 0.0
Delta: 3.0
Delta: 3.0
```

delta는 run 함수에서 인자로 받은 func 를 통해 전달되는 값이다. animate 함수의 yield 문을 통해 반환된 값들이 run 함수의 for 루프에서 delta 변수에 의해 순차적으로 할당된다. 

하지만 이 코드의 문제는 animate가 너무 반복적이라는 것이다. for문과 yield 식이 반복되면서 가독성이 점점 떨어진다. 이 문제를 해결하기 위해서 yield from 식을 활용할 수 있다.  `yield from`은 고급 제너레이터 기능으로, 제어를 부모 제너레이터에게 전달하기 전에 내포된 제너레이터가 모든 값을 내보낸다. **쉽게 말해, `yield from`은 제너레이터 함수 내의 모든 제너레이터와 이터레이터를 순차적으로 반환한다는 뜻이다.**

```python
# yield from의 동작 이해
def numbers():
    yield 1
    yield 2
    yield 3

def letters():
    yield 'a'
    yield 'b'
    yield 'c'

def my_generator():
    yield from numbers()
    yield from letters()

gen = my_generator()
print(next(gen))  # 출력: 1
print(next(gen))  # 출력: 2
print(next(gen))  # 출력: 3
print(next(gen))  # 출력: 'a'
print(next(gen))  # 출력: 'b'
print(next(gen))  # 출력: 'c'
```

 ****yield from은 ****파이썬 인터프리터가 개발자 대신 for 루프를 내포시키고 yield 식을 처리하도록 만들어 성능을 더 좋게 만들고, 가독성도 높여준다. ****다음 코드는 yield from을 적용한 코드이다.

```python
def move(period, speed):
    for _ in range(period):
        yield speed

def pause(delay):
    for _ in range(delay):
        yield 0

def animate():
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)

def render(delta):
    print(f'Delta: {delta:.1f}')

def run(func):
    for delta in func():
        render(delta)

run(animate)
```

**이전과 결과는 같으면서 가독성이 보장되며, 더 좋은 성능을 기대할 수도 있다. 따라서 제너레이터를 합성해야 하는 경우에는 yield from을 사용하도록 하자.**

# 34. send로 제너레이터에 데이터를 주입하지 마라

[30. 리스트를 반환하기보다는 제너레이터를 사용하라](https://www.notion.so/30-61464714af344715b0906b8742e2b316?pvs=21) 

앞의 내용처럼, yield 식을 사용하면 제너레이터 함수를 만들어서 간단하게 이터레이션이 가능한 출력 값을 만들어내도록 할 수 있다. 하지만 이 방식은 단방향 데이터 채널로써, 제너레이터가 데이터를 내보내는 도중 다른 데이터를 받아들일 때 직접 쓸 수 있는 방법이 없다. 

이번에는 양방향 통신을 구현해보자. 다음 코드는 주어진 간격과 진폭에 따른 사인파 값을 생성하여, wave 제너레이터를 이터레이션하면서 진폭이 고정된 파형 신호를 송신하는 코드이다.

```python
import math

def wave(amplitude, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output =  amplitude * fraction
        yield output

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:>5.1f}')

def run(it):
    for output in it:
        transmit(output)

run(wave(3.0, 8))
```

```python
# 실행결과
출력:   0.0
출력:   2.1
출력:   3.0
출력:   2.1
출력:   0.0
출력:  -2.1
출력:  -3.0
출력:  -2.1
```

하지만 진폭을 지속적으로 변경해야 하는 경우에는 쓸 수가 없다. 이 방법 말고, 제너레이터를 이터레이션할 때마다 진폭을 변조할 수 있는 방법이 필요하다.

파이썬 제너레이터의 send 메소드는 yield 식을 양방향 채널로 격상시켜준다. (참고: 우리의 주제는 send를 쓰지 말자는 내용이다!) 제너레이터를 일시 중단시키고 다시 활성화할 수 있도록 만들어서, 입력을 스트리밍하는 동시에 출력을 내보낼 수 있도록 한다.

```python
def my_generator():
    num = yield
    print('Received:', num)

gen = my_generator()
next(gen)         # 제너레이터 시작
gen.send(10)      # "Received: 10" 출력
```

초기에 `send()` 함수를 호출하기 전에는 `next()` 함수를 사용해야 한다. 그래야 제너레이터를 실행하여 `yield` 문이 실행되도록 할 수 있기 때문이다. `yield` 문은 제너레이터 값을 반환하고 다음 호출까지 현재 제너레이터 포인터 위치를 대기시키는 스탑 포인트 역할을 한다. 이후 `send()` 함수를 사용하여 값을 보낼 수 있다. 

```python
def my_generator():
    received = yield 1
    print(f'받은 값 = {received}')

it = iter(my_generator())
output = next(it)
print(f'출력값 = {output}')

try:
    next(it)
except StopIteration:
    pass
```

```python
# 실행결과
출력값 = 1
받은 값 = None
```

일반적으로 제너레이터를 이터레이션할 때 처음에 yield 식이 반환하는 값은 None이다. 왜냐하면 방금 생성된 제너레이터는 yield 식에 도달하지 못하기 때문이다.

그래서 최초로 send를 호출할 때 인자로 전달할 수 있는 값은 None 뿐이다.(None 외에 다른 값을 전달하면 에러남) 만약 send를 쓰고 싶다면 아래처럼 제너레이터에 None을 먼저 보낸 후에 데이터를 전송해야 한다.

```python
def my_generator():
    received = yield 1
    print(f'받은 값 = {received}')

it = iter(my_generator())
output = it.send(None)
print(f'출력값 = {output}')

try:
    it.send('안녕!')
except StopIteration:
    pass
```

```python
# 실행결과
출력값 = 1
받은 값 = 안녕!
```

send를 활용하면 입력 시그널을 바탕으로 사인 파의 진폭을 변조할 수 있다.

```python
import math

def wave(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield   # send() 사용을 위해 초기 진폭을 받는다.
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output =  amplitude * fraction
        amplitude = yield output    # 다음 진폭을 받을 준비

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:>5.1f}')

def run(it):
    amplitudes = [None,7,7,7,2,2,2,2,10,10,10,10,10]
    for amplitude in amplitudes:
        output = it.send(amplitude)
        transmit(output)

run(wave(12))
```

```python
# 실행결과
출력: None
출력:   0.0
출력:   3.5
출력:   6.1
출력:   2.0
출력:   1.7
출력:   1.0
출력:   0.0
출력:  -5.0
출력:  -8.7
출력: -10.0
출력:  -8.7
출력:  -5.0
```

하지만 위 코드는 가독성이 너무 낮다.

이번에는 요구 사항이 단순 사인파를 반송파로 사용하는 대신, 여러 신호의 시퀸스로 이뤄진 복잡한 파형을 사용해야 한다고 하자. `yield form` 식을 활용하면 여러 제너레이터를 합성하여 사용할 수 있다.  [33. yield from을 사용해 여러 제너레이터를 합성하라](https://www.notion.so/33-yield-from-542840e6b3e24fe7b12c3768f0da5cea?pvs=21) 

1. 기존 프로그램에 yield from을 적용

```python
import math

def wave(amplitude, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output =  amplitude * fraction
        yield output

def complex_wave():
    yield from wave(7.0, 3)
    yield from wave(2.0,4)
    yield from wave(10.0,5)

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:>5.1f}')

def run(it):
    for output in it:
        transmit(output)

run(complex_wave())
```

```python
# 실행결과
출력:   0.0
출력:   6.1
출력:  -6.1
출력:   0.0
출력:   2.0
출력:   0.0
출력:  -2.0
출력:   0.0
출력:   9.5
출력:   5.9
출력:  -5.9
출력:  -9.5
```

1. send()를 추가한 프로그램에 yield from을 적용

```python
import math

def wave(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield   # send() 사용을 위해 초기 진폭을 받는다.
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output =  amplitude * fraction
        amplitude = yield output    # 다음 진폭을 받을 준비

def complex_wave():
    yield from wave(3)
    yield from wave(4)
    yield from wave(5)

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:>5.1f}')

def run(it):
    amplitudes = [None,7,7,7,2,2,2,2,10,10,10,10,10]
    for amplitude in amplitudes:
        output = it.send(amplitude)
        transmit(output)

run(complex_wave())
```

```python
# 실행결과
출력: None
출력:   0.0
출력:   6.1
출력:  -6.1
출력: None
출력:   0.0
출력:   2.0
출력:   0.0
출력: -10.0
출력: None
출력:   0.0
출력:   9.5
출력:   5.9
```

기존 프로그램에서는 성공적으로 코드 수행이 됐는데, send를 추가한 프로그램에서는 `yield from` 식이 끝날 때마다 None이 출력되는 모습이 보인다. 각각 내포된 제너레이터는 send 메소드 호출로부터 값을 받기 위해 초기에는 단순 yield 식으로 시작하게 되어 None을 받기 때문이다.  None문제를 우회할 코드를 작성할 수는 있으나, 이는 run() 함수의 복잡도를 증가시키며 yield from 구조에 대해 추가적인 학습이 요구된다.

그래서 send 를 쓰지 말자는 거다. 대신 제시할만한 해결책은 다음과 같다.

- 함수 파라미터로 이터레이터 객체를 전달한다.

wave 함수에 이터레이터 객체를 전달해보자. 이터레이터는 자신에 대해 next 내장 함수가 호출될 때마다 입력으로 받은 진폭을 하나씩 돌려준다. 

```python
# 참고: 리스트와 이터레이터의 차이점

# 리스트는 iterable한 컬렉션 자체를 나타낸다. 리스트 자체는 이터레이터 객체가 아니다!
my_list = [1, 2, 3]
for item in my_list:
    print(item)

# 이터레이터는 iterable한 컬렉션의 요소를 하나씩 순차적으로 반환한다.
my_iterator = iter([1, 2, 3])
print(next(my_iterator))
print(next(my_iterator))
print(next(my_iterator))
```

이런 식으로 이전 제너레이터를 다음 제너레이터의 입력으로 연쇄시켜 연결하면 입력과 출력이 차례로 처리되게 만들 수 있다.  [32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라](https://www.notion.so/32-6860d00ac7e74148ae4128326eeb649f?pvs=21) 

```python
import math

def wave(amplitude_it, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        amplitude = next(amplitude_it)  # 다음 입력 받기
        output =  amplitude * fraction
        yield output   

def complex_wave(amplitude_it):
    yield from wave(amplitude_it,3)
    yield from wave(amplitude_it,4)
    yield from wave(amplitude_it,5)

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:>5.1f}')

def run():
    amplitudes = [7,7,7,2,2,2,2,10,10,10,10,10]
    it = complex_wave(iter(amplitudes))  # 이터레이터를 넣고 제너레이터를 반환
    for _ in amplitudes:
        output = next(it)
        transmit(output)

run()
```

```python
# 실행결과
출력:   0.0
출력:   6.1
출력:  -6.1
출력:   0.0
출력:   2.0
출력:   0.0
출력:  -2.0
출력:   0.0
출력:   9.5
출력:   5.9
출력:  -5.9
출력:  -9.5
```

이 방법은 다음의 장점을 갖는다.

1. 아무데서나 이터레이터를 가져올 수 있다.
2. 이터레이터가 동적인 경우(위처럼 제너레이터 함수를 사용해서 이터레이터를 만든 경우)에도 잘 동작한다.

하지만 이 코드는 입력 제너레이터가 완전히 스레드로부터 안전하다고 볼 수는 없다. 만약 스레드 경계를 넘나들면서 제너레이터를 사용하게 되면 순서가 뒤죽박죽 될 수 있기 때문에, async 함수를 쓰는 것이 더 나을 수도 있다. (Beter way 62 참고)

# 35. 제너레이터 안에서 throw로 상태를 변화시키지 마라

핵심은 예외처리를 해야 한다면 throw를 사용하지 말고 iterable 클래스를 사용하자는 것이다.

제너레이터의 고급 기능으로 이미 몇 개 살펴본 적이 있다.

1. yield from  [33. yield from을 사용해 여러 제너레이터를 합성하라](https://www.notion.so/33-yield-from-542840e6b3e24fe7b12c3768f0da5cea?pvs=21) 
2. send  [34. send로 제너레이터에 데이터를 주입하지 마라](https://www.notion.so/34-send-cb023b6557bb4952b05e9f30c572c6d0?pvs=21) 

그리고 `throw` 도 제너레이터의 고급 기능 중 하나이다. `throw` 는 제너레이터 안에서 Exception을 다시 던질 수 있는 메소드이다. 어떤 제너레이터에 대해 throw가 호출되면 이 제너레이터는 값을 내놓고 일시적으로 정지되어 있는 yield로부터 기존의 제너레이터 실행을 하지 않고, throw가 제공한 Exception을 던진다. (Better Way 65 참고)

```python
# throw 메소드 예시: 제너레이터 함수에 try-except 블록이 없는 경우
class MyError(Exception):
    pass

def myGenerator():
    yield 1
    yield 2
    yield 3

it = myGenerator()  # 제너레이터 객체 생성
print(next(it))
print(next(it))
print(it.throw(MyError('test Error')))
print(next(it))  # 실행되지 않는다.
```

```python
# 실행결과
1
2
Traceback ...
MyError: test Error
```

위 코드에서는 제너레이터 함수 내에서 에러 처리를 할 try-except 블록이 없으므로 곧바로 예외가 발생한다.

```python
# throw 메소드 예시: 제너레이터 함수에 try-except 블록이 있는 경우
def my_generator():
    try:
        yield 1
        yield 2
        yield 3
    except ValueError as e:
        yield 'Caught ValueError: {}'.format(str(e))
    except Exception as e:
        yield 'Caught Exception: {}'.format(str(e))

gen = my_generator()  # 제너레이터 객체 생성
print(next(gen))  # 출력: 1
print(next(gen))  # 출력: 2
gen.throw(ValueError('Custom Exception'))  # 발생한 예외는 제너레이터 내에서 처리됩니다.
print(next(gen))  # 출력: 'Caught ValueError: Custom Exception'
print(next(gen))  # 출력:  StopIteration
```

```python
class MyError(Exception):
    pass

def myGenerator():
    yield 1

    try:
        yield 2
    except MyError:
        print("MyError")
    else:
        yield 3
    
    yield 4

it = myGenerator()  # 제너레이터 객체 생성
print(next(it))  # 출력: 1
print(next(it))  # 출력: 2
print(it.throw(MyError('test Error')))  # 출력: MyError
																				#				4
print(next(it))  # 출력:  StopIteration
```

반면 위 코드에서는 제너레이터 함수 내에서 예외를 처리할 try-except 블록이 있기 때문에 다음 yield에 해당 error을 출력하며 try-except 블록을 종료한다. 그렇다고 제너레이터 함수가 종료되는 것은 아니므로, try-except 블록 밖의 나머지 yield에 대해서는 그대로 실행되는 모습이다.

위의 try-except 블록에 대한 예외 처리 방식은 제너레이터와 제너레이터 호출자 사이에 양방향 통신 수단을 제공한다. (다른 양방향 통신 수단은: [34. send로 제너레이터에 데이터를 주입하지 마라](https://www.notion.so/34-send-cb023b6557bb4952b05e9f30c572c6d0?pvs=21))

아래 코드는 throw 메소드에 의존하는 제너레이터를 통해 타이머를 구현한 코드이다.

```python
class Reset(Exception):
    pass

def timer(period):
    current = period
    while current:
        current -= 1
        try:
            yield current
        except Reset:
            current = period

def checkForReset():
    # 외부 이벤트 polling
    user_input = input("Enter 'reset' to reset the timer: ")
    if user_input.lower() == "reset":
        return True
    return False

def announce(remaining):
    print(f'{remaining} 틱 남음')

def run():
    it = timer(4)
    while True:
        try:
            if checkForReset():
                current = it.throw(Reset())
            else:
                current = next(it)
        except StopIteration:
            break
        else:
            announce(current)

run()
```

```python
# 실행결과
Enter 'reset' to reset the timer: 
3 틱 남음
Enter 'reset' to reset the timer:
2 틱 남음
Enter 'reset' to reset the timer: reset
3 틱 남음
Enter 'reset' to reset the timer:
2 틱 남음
Enter 'reset' to reset the timer: reset
3 틱 남음
Enter 'reset' to reset the timer:
2 틱 남음
Enter 'reset' to reset the timer:
1 틱 남음
Enter 'reset' to reset the timer:
0 틱 남음
Enter 'reset' to reset the timer:
```

동작은 잘 되지만 가독성이 떨어진다.

위 기능을 더 단순하게 해결하는 방법은 iterable 컨테이너 객체를 사용하여 상태가 있는 클로저를 정의하는 것이다. ([31. 인자에 대해 이터레이션할 때는 방어적이 돼라](https://www.notion.so/31-7cdf3f11750341a4b21da96859d9656b?pvs=21), Better Way 38 참고)

```python
class Reset(Exception):
    pass

class Timer:
    def __init__(self, period) -> None:
        self.current = period
        self.period = period

    def reset(self):
        self.current = self.period

    def __iter__(self):
        while self.current:
            self.current -= 1
            yield self.current

def checkForReset():
    # 외부 이벤트 polling
    user_input = input("Enter 'reset' to reset the timer: ")
    if user_input.lower() == "reset":
        return True
    return False

def announce(remaining):
    print(f'{remaining} 틱 남음')

def run():
    timer = Timer(4)
    for current in timer:
        if checkForReset():
            timer.reset()
        else: announce(current)

run()
```

```python
# 실행결과
Enter 'reset' to reset the timer: 
3 틱 남음
Enter 'reset' to reset the timer: 
2 틱 남음
Enter 'reset' to reset the timer: reset
Enter 'reset' to reset the timer: 
3 틱 남음
Enter 'reset' to reset the timer: 
2 틱 남음
Enter 'reset' to reset the timer:
1 틱 남음
Enter 'reset' to reset the timer: reset
Enter 'reset' to reset the timer:
3 틱 남음
Enter 'reset' to reset the timer:
2 틱 남음
Enter 'reset' to reset the timer:
1 틱 남음
Enter 'reset' to reset the timer:
0 틱 남음
```

iterable 클래스를 사용함으로써, throw가 포함된 try-except 블록이 단순 for문으로 개선되어 가독성이 높아진 모습이다. 만약 제너레이터와 예외를 섞어서 만들어야 하는 작업이 있다면 비동기 기능을 사용하면 더 좋다. (Better way 60 참고) 

```python
# 비동기 기능 참고
class MyError(Exception):
    pass

async def myGenerator():
    yield 1

    try:
        yield 2
    except MyError:
        print("MyError")
    else:
        yield 3
    
    yield 4

async def main():
    async for value in myGenerator():
        print(value)

try:
    import asyncio
    asyncio.run(main())
except ImportError:
    pass  # Python 3.6 이하 버전에서는 asyncio.run()을 사용할 수 없습니다.
```

```python
# 실행결과
1
2
MyError
3
4
```

# 36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라

`itertools` 는 이터레이터 객체(

[30. 리스트를 반환하기보다는 제너레이터를 사용하라](https://www.notion.so/30-61464714af344715b0906b8742e2b316?pvs=21) 

[31. 인자에 대해 이터레이션할 때는 방어적이 돼라](https://www.notion.so/31-7cdf3f11750341a4b21da96859d9656b?pvs=21) 

)를 처리할 때 매우 유용하다. 특히 코딩테스트에서도 반복되는 데이터를 처리할 때 매우 용이하게 쓸 수 있으므로 반드시 알아두자.

이터레이터를 다루고 있는데 코드가 복잡해보인다? 그러면 itertools 라이브러리 문서를 꼭 확인해보자. 아래는 itertools에서 주요하게 쓸만한 기능들을 살펴본다.

### 여러 이터레이터 연결하기

itertools 내장 모듈에는 여러 이터레이터를 하나로 합치는 함수들이 더러 있다.

- **chain**

여러 이터레이터를 하나의 순차적인 이터레이터로 합친다.

```python
import itertools

it = itertools.chain([1,2,3], [4,5,6])
print(list(it))  # [1, 2, 3, 4, 5, 6]
```

- **repeat**

한 값을 계속 반복하여 출력하고 싶을 때 사용한다.

```python
import itertools

it = itertools.repeat('안녕', 3)
print(list(it))  # ['안녕', '안녕', '안녕']
```

- **cycle**

어떤 이터레이터가 내놓은 원소들을 계속 반복하고 싶을 때 사용한다.

```python
import itertools

it = itertools.cycle([1,2])
result = [next(it) for _ in range(10)]
print(result)  # [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]
```

- **tee**

동일한 이터레이터를 병렬적으로 지정된 개수만큼 만들고 싶을 때 사용한다.

```python
import itertools

it1, it2, it3 = itertools.tee(['하나', '둘'], 3)
print(list(it1))  # ['하나', '둘']
print(list(it2))  # ['하나', '둘']
print(list(it3))  # ['하나', '둘']
```

- **zip_longest**

zip 내장함수(Better Way 8 참고)의 변종으로, 여러 이터레이터 중 짧은 쪽의 이터레이터가 원소를 다 사용한 경우, fillvalue로 지정한 값을 채워 넣는다. 아무것도 지정하지 않으면 None을 넣는다.

```python
import itertools

keys = ['하나', '둘', '셋']
values = [1, 2]

normal = list(zip(keys, values))
print('zip:', normal)  # zip: [('하나', 1), ('둘', 2)]

it = itertools.zip_longest(keys, values, fillvalue = '없음')
longest = list(it)
print('zip_longest:', longest)  # zip_longest: [('하나', 1), ('둘', 2), ('셋', '없음')]
```

### 이터레이터에서 원소 거르기

itertools 내장 모듈에는 이터레이터에서 원소를 필터링하는 함수들이 들어있다.

- **islice**

이터레이터를 복사하기는 싫은데, 원소 인덱스를 이용하여 슬라이싱하고 싶을 때 사용한다. (Better Way 11, 12 참고)

```python
import itertools

values = [1,2,3,4,5,6,7,8,9,10]

first_five =itertools.islice(values, 5)
print('앞에서 다섯:', list(first_five))  # 앞에서 다섯: [1, 2, 3, 4, 5]

middle_odds = itertools.islice(values, 2,8,2)
print('홀수들:', list(middle_odds))  # 홀수들: [3, 5, 7]
```

- **takewhile**

이터레이터에서 주어진 술어가 False를 반환하는 첫 원소가 나타날 때까지 원소를 돌려준다. 반대로 말하면, 술어가 계속 True를 반환한다면 계속 원소를 돌려준다.

```python
import itertools

values = [1,2,3,4,5,6,7,8,9,10]

less_than_seven = lambda x : x<7
it = itertools.takewhile(less_than_seven, values)
print(list(it))  # [1, 2, 3, 4, 5, 6]
```

- **dropwhile**

takewhile과 정반대의 기능을 한다.

```python
import itertools

values = [1,2,3,4,5,6,7,8,9,10]

less_than_seven = lambda x : x<7
it = itertools.dropwhile(less_than_seven, values)
print(list(it))  # [7, 8, 9, 10]
```

- fillterfalse

주어진 이터레이터에서 술어가 False를 반환하는 모든 원소를 돌려준다. filter 내장 함수의 반대 행동을 한다.

```python
import itertools

values = [1,2,3,4,5,6,7,8,9,10]
evens = lambda x : x%2==0

filter_result = filter(evens, values)
print('Filter:', list(filter_result))  # Filter: [2, 4, 6, 8, 10]

filter_false_result = itertools.filterfalse(evens, values)
print('FilterFalse', list(filter_false_result))  # FilterFalse [1, 3, 5, 7, 9]
```

### 이터레이터에서 원소의 조합 만들어내기

itertools 내장 모듈에는 이터레이터가 돌려주는 원소들의 조합을 만들어내는 함수들이 있다.

- **accumulate**

파라미터를 두 개 받는 함수를 반복 적용하면서 이터레이터 원소 값을 하나로 줄여준다. 한 번에 한 단계씩 결과를 내놓는다는 점을 제외하면, 기본적으로 functools 내장 모듈에 있는 reduce 함수와 같은 결과를 반환한다. 단일 파라미터만 넣으면 첫 번째 파라미터에 대해서만 기능을 수행한다.

```python
import itertools

values = [1,2,3,4,5,6,7,8,9,10]
sum_reduce = itertools.accumulate(values)
print('합계:', list(sum_reduce))  # 합계: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]

def sum_modulo_20(first, second):
    output = first + second
    return output % 20

modulo_reduce = itertools.accumulate(values, sum_modulo_20)
print('20으로 나눈 나머지의 합:', list(modulo_reduce))  # 20으로 나눈 나머지의 합: [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]
```

- **product**

하나 이상의 이터레이터에 들어 있는 아이템들의 테카르트 곱을 반환한다. 리스트 컴프리헨션을 깊이 내포시키는 대신 편리하다는 장점이 있다. ([28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라](https://www.notion.so/28-e1aed2d0d5fa47759dfe32f8fc7325a2?pvs=21) )

```python
import itertools

single = itertools.product([1,2], repeat=2)
print('리스트 한 개:', list(single))  # 리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]

multipie = itertools.product([1,2], ['a', 'b'])
print('리스트 두 개:', list(multipie))  # 리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
```

- **permutations**

이터레이터가 내놓은 원소들로부터 만들어낸 길이 N인 순열을 반환한다. 원소 집합은 첫 번째 파라미터에, N은 두 번째 파라미터에 들어간다.

```python
import itertools

it = itertools.permutations([1,2,3,4], 2)
print(list(it))  # [(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]
```

- **combinations**

이터레이터가 내놓은 원소들로부터 만들어낸 길이 N인 조합을 반환한다. 원소 집합은 첫 번째 파라미터에, N은 두 번째 파라미터에 들어간다.

```python
import itertools

it = itertools.combinations([1,2,3,4], 2)
print(list(it))  # [(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]
```

- combinations_with_replacement

combinations에서 반복을 허용한다. (중복 조합을 포함하여 반환한다.)

```python
import itertools

it = itertools.combinations_with_replacement([1,2,3,4], 2)
print(list(it))  # [(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]
```