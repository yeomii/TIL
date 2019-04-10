# Euclidean Algorithm (유클리드 호제법)

* 유클리드 호제법은 2개의 자연수의 최대 공약수를 구하는 알고리즘의 하나이다.

* 2개의 자연수 a, b (단, a> b) 에 대해서 `a % b` 의 값을 r 이라고 하면 a 와 b 의 최대 공약수는 b 와 r 의 최대 공약수와 같다.

* `gcd(a, b)` 를 a 와 b 의 최대 공약수라 하면 위 성질을 이용해 함수의 인자 값 중에 하나가 0 이 될 때 까지 값을 줄일 수 있다.
```
gcd(a, b) = gcd(b, r) = gcd(r, r') = ... = gcd(x, 0) = x
```

* 구현은 다음과 같이 할 수 있다.
```python3
def gcd(a, b):
    if a < b:
        a, b = b, a
    while b > 0:
        a, b = b, a % b
    return a
```
