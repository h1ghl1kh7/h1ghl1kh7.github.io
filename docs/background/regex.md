---
layout: default
title: Regex (with Python3)
nav_order: 3
parent: Background
last_modified_date: 2024-02-01
---
---


## 메타 문자

`$()*+.?[]\^{}|` 등이 존재하고, 특정 문자 혹은 문자 계열을 대신하여 표현하는 문자이다.

이를 이용하면 특정 규칙을 가진 여러 단어들을 하나의 패턴으로 함축할 수 있다.

(메타 문자에 해당되는 문자 자체를 검색하고 싶은 경우, 백슬래쉬(`\`)를 앞에 붙이면 된다.)

### ^ 문자

[^a] → a 가 아닌 것과 매치를 뜻한다.

그냥 사용할 경우, 문자열의 시작을 표현한다. 

### $ 문자

문자열의 끝을 표현한다.

### **[ ] 문자 - 문자 클래스**

[ ] 사이에 들어간 문자들과의 매치를 뜻한다.

하이폰 (-)를 사용하여 문자 사이의 범위를 패턴으로 하여 매치할 수 있다.

**자주 사용하는 문자 클래스**

`[0-9]` 또는 `[a-zA-Z]` 등은 무척 자주 사용하는 정규 표현식이다. 이렇게 자주 사용하는 정규식은 별도의 표기법으로 표현할 수 있다. 다음을 기억해 두자.

- `\d` - 숫자와 매치된다. `[0-9]`와 동일한 표현식이다.
- `\D` - 숫자가 아닌 것과 매치된다. `[^0-9]`와 동일한 표현식이다.
- `\s` - 화이트스페이스(whitespace) 문자와 매치된다. `[ \t\n\r\f\v]`와 동일한 표현식이다. 맨 앞의 빈칸은 공백 문자(space)를 의미한다.
- `\S` - 화이트스페이스 문자가 아닌 것과 매치된다. `[^ \t\n\r\f\v]`와 동일한 표현식이다.
- `\w` - 문자+숫자(alphanumeric)와 매치된다. `[a-zA-Z0-9_]`와 동일한 표현식이다.
- `\W` - 문자+숫자(alphanumeric)가 아닌 문자와 매치된다. `[^a-zA-Z0-9_]`와 동일한 표현식이다.

대문자로 사용된 것은 소문자의 반대임을 추측할 수 있다.

### .(dot) 문자 - `\n`를 제외한 모든 문자

```
a.b - "a'+모든 문자+"b"
a[.]b - "a.b"
```

위와 같이 사용하고, 혼동하지 않도록 주의해야 한다.

### * 문자

* 문자 앞에 나온 문자를 0번 이상 반복되면 매치된다.

```
a*b - "ab", "b", "aaaab" 등과 매치
```

### + 문자

위의 * 문자와 거의 똑같지만, 앞에 나온 문자가 1번 이상 반복되면 매치된다.

### {} 문자와 ? 문자

앞에 나온 문자가 중괄호 안에 들어간 카운트 수 만큼 매치된다.

```python
a{2} # a 2개
a{2,5} # a 2개 ~ 5개
```

? 는 {0,1}과 같다.

## 파이썬

### 함수

- match
    
    문자열의 시작부터 패턴과 일치하는지 확인한다.
    
    ```python
    # re.match(pattern,string,flag)
    re.match('a','ab') # O
    re.match('b','ab') # X
    ```
    
- search
    
    문자열에 패턴과 일치하는 결과가 있는지 확인한다.
    
    ```python
    # re.search(pattern,string,flag)
    re.search('a','ab') # O
    re.search('b','ab') # O
    ```
    
- findall
    
    문자열에 패턴과 일치하는 결과가 있는지 확인하고, 여러 개일 경우, 모든 결과를 리스트로 반환한다.
    
    ```python
    # re.findall(pattern,string,flag)
    re.findall('a','a') # ['a']
    re.findall('a','aaaaaaa') # ['a', 'a', 'a', 'a', 'a', 'a', 'a']
    re.findall('aaa','aaaaaaaaa') # ['aaa', 'aaa', 'aaa']
    ```
    
- finditer
    
    findall과 비슷하지만, 리스트가 아닌 iterator 형식으로 반환한다.
    
    ```python
    # re.finditer(pattern,string,flag)
    re.finditer('a','a') # iterator 객체
    re.finditer('a','aaaaaa') # iterator 객체
    ```
    
- fullmatch
    
    문자열과 패턴이 완벽하게 일치하는지 확인한다.
    
    ```python
    # re.fullmatch(pattern,string,flag)
    re.fullmatch('a','a') # O
    re.fullmatch('a','aa') # X
    re.fullmatch('a','ab') # X
    ```
    
- split
    
    문자열에서 패턴이 맞으면 이를 기점으로 쪼갠다. (최대 split 수를 설정하면 지정한 수 만큼 쪼개고, 나머지 뒤 문자열을 리스트의 마지막에 추가한다)
    
    ```python
    # re.split(pattern,string,max split,flag)
    re.split('a','a') # ['', '']
    re.split('a','bab') # ['b', 'b']
    ```
    
- sub
    
    문자열에 맞는 패턴을 교체할 문자열로 교체한다. (최대 sub 수를 설정하면, 지정한 수 만큼 교체하고, 나머지 뒤 문자열은 그대로 둔다)
    
    ```python
    # re.sub(pattern,string,max sub,flag)
    re.sub('a','z','a') # z
    re.sub('a','zz','ab') # zzb
    ```
    
- subn
    
    sub와 동작은 비슷하지만, (문자열, 매칭 횟수) 형태로 반환한다.
    
    ```python
    # re.subn(pattern,string,max sub,flag)
    re.subn('a','z','a') # ('z',1)
    re.subn('a','zz','aaab') # ('zzzzzzb', 3)
    ```
    
- compile
    
    패턴과 플래그가 동일한 정규식을 여러번 사용할 때 쓴다.
    
    ```python
    # re.compile(pattern,flag)
    a = re.compile('a')
    a.search('abab') # O
    ```
    
- purge
    
    compile로 만든 객체는 보통 100개까지 캐시에 저장하고, 그 수를 넘어가면 초기화 된다.
    
    purge 함수는 100개가 넘어가지 않아도 캐시를 초기화 할 수 있다.
    
- escape
    
    패턴을 입력 받으면, 특수문자들에 이스케이프(백슬래쉬) 처리를 한 후, 반환한다.
    
    ```python
    # re.escape(pattern)
    re.escape("a") # 'a'
    re.escape("a{2,3}b") # 'a\\{2,3\\}b'
    ```
    

### 오브젝트

함수의 반환이 match object로 반환되는 경우, group(), start(), end() 등의 함수를 사용할 수 있다.

- groups
    
    일치하는 모든 서브 그룹을 포함하는 튜플을 반환한다.
    
    기본 값은 None이다.
    
- group
    
    group(0), group(1) 과 깉이 사용할 수 있고, groups 튜플의 index 접근과 비슷하다.
    
- groupdict
    
    `(?P<group_name>PATTERN)` 와 같이 정규표현식을 그룹화하고, 네이밍한 결과를 딕셔너리 형태로 반환 받을 수 있다.
    
    ```python
    a=re.match("(?P<Test>a*)","aaa")
    a.groupdict() # {'Test': 'aaa'}
    ```
    

### 플래그

- DOTALL(S) - `.`(dot)이 줄바꿈 문자를 포함해 모든 문자와 매치될 수 있게 한다.
- IGNORECASE(I) - 대소문자에 관계없이 매치될 수 있게 한다.
- MULTILINE(M) - 여러 줄과 매치될 수 있게 한다. `^`, `$` 메타 문자 사용과 관계 있는 옵션이다.
- VERBOSE(X) - verbose 모드를 사용할 수 있게 한다. 정규식을 보기 편하게 만들 수 있고 주석 등을 사용할 수 있게 된다.
