---
title: Lua 정리
date: 2019-01-30 00:00:00 +0300
category: 'lua'
draft: false
---
![](./images/lua.PNG)
<div style="text-align: center;">
    <span style="font-size:11px; color:grey">
        이 정적 페이지는 PC 버전에 최적화되어 있습니다.  
    </span>
</div>


---
게임 스크립트로 사용되는 [루아(Lua)](http://www.lua.org/manual/5.3/) 프로그래밍 언어에 대해 알아보겠습니다.

TODO
루아 CAPI에 대해 작성한다.

## 특징

루아는 다른 언어에 embedded하기 위해 설계된 언어입니다.  
따라서 확장에 용이하고 glue language라고도 불립니다.  
현재 5.3 버전까지 release 되었고 무료 오픈소스입니다.  
인터프리터로 동작하는 동적 스크립트 언어이기 때문에 변경 사항이 생겨도 재컴파일이 필요 없습니다.  
블리자드의 w.o.w의 개발 스펙에 포함되어 유명세를 탔다고 합니다.  
게임 설계를 코어(C++, 렌더링, 네트워크 등) + 스크립트(로직, UI)의 구조로 디자인합니다.  
이 덕분에 게임 바이너리 패치나 빌드 등을 매번 할 필요가 없이 수정 사항이 발생하면
루아 파일을 패킹만 해주면 되기 때문에 개발 생산성이 높아집니다.  
Lua Tinker, Lua Binder 등의 라이브러리를 통해 메인 프로그램과 연동하여 데이터 교환 및 상호 호출이 가능합니다.  
GC도 가지고 있어 메모리로부터 좀 더 자유롭습니다.  
다중 배정문이 가능하고 세미콜론 따위는 안 써도 되며, goto를 권장하는 재밌는 언어입니다.  

## 개발 환경

[링크](https://www.lua.org/demo.html)를 통해 개발 환경의 설치 없이 실행해 볼 수 있습니다.  
로컬에 개발 환경을 구성하기 위해서는 루아 인터프리터의 설치가 필요합니다.  
### MAC
---
```terminal
curl -R -O http://www.lua.org/ftp/lua-5.3.0.tar.gz
tar zxf lua-5.3.0.tar.gz
cd lua-5.3.0
make macosx test
make install
```
### WIN
---
[코드](https://code.google.com/archive/p/luaforwindows/)를 다운 받아 윈도우용 콘솔인 lua.exe를 실행합니다.  
실행하면 cli를 통해 루아 스크립트를 작성 후 테스트해볼 수 있습니다.  

EOF 문자를 입력하거나 os.eixt()을 호출하여 종료합니다.  
## 주석

```lua
--이것은 한줄 주석
--[[이것은 
다중 주석]]
```
## Swap

루아의 swap은 C와 다릅니다.  
아래의 처리만 해주면 끝입니다.
```lua
local a, b = 1, 2
print(a, b) --1, 2
a, b = b, a
print(a, b) --2, 1
```

## 자료형

루아에는 기본적으로 8가지의 자료형이 존재합니다.  
또한 동적 타입이므로 변수에는 값을 할당하기 전에는 타입을 가지지 않습니다.  
따라서 모든 값은 아래와 같은 타입 중 하나를 가집니다.  
(nil, Boolean, String, Number, table, Function, UserData, Thread)  
또한 일급 값이기 때문에 모든 값을 변수로 저장하거나 인자로 전달하거나 결과로 받을 수 있습니다.  
변수는 기본적으로 별도의 명시가 없을 경우 global입니다.  
타입을 반환하는 type()을 제공합니다.  
변수는 대소문자를 구분하며 local을 명시할 경우 스코프는 선언된 block 단위입니다.  

## nil

루아만의 독특한 자료형으로 변수가 초기화되지 않은 경우를 지칭합니다.  
전역 변수에서 nil을 할당하면 변수가 삭제됩니다.  
루아의 GC는 점진적으로 진행되기 때문에 메모리를 즉시 회수하지 않을 가능성이 있습니다.  
따라서 update 등의 함수에서 프로그래머가 직접 nil을 할당하여 바로 회수할 수 있습니다.  

## Boolean

true, false으로 제공됩니다.  
false, nil 등 명시된 의미만이 거짓을 의미합니다.  
특이하게도 0 혹은 ''은 true입니다.

## Number

int형이 별도로 존재하지 않고 부동소수점을 표현하기 위한 8바이트의 double형으로 통일되어 있습니다.  
따라서 루아는 정수 21억까지 커버가 가능하고 유효자리 53(소수부 - 15)의 안전을 보장합니다.

## String

문자열 개행이 필요한 경우 \n 이스케이프 문자 혹은 ```[[]]```로 문자열을 감쌉니다.  
자동형변환이 제공해 문자열 "10" + 숫자 1의 연산이 가능합니다.  
length()가 아닌 '#' 연산자를 통하여 문자열의 길이를 구합니다.

## Function

JS와 비슷하게 일급 객체로써, 함수 선언식과 표현식 모두 가능합니다.  
또한 클로저 구현도 가능하며 함수 리턴이나 콜백 구현이 가능합니다.  
인자가 한 개일 경우에는 '()' 생략이 가능합니다.  
가변인자를 지원하고 다중 리턴이 가능합니다.  
```lua
x, y = 1, 2
local sum = function(x, y, z)
    z = 4
    return x+y, x, y, z
end
res, x, y, z = sum(x, y)
print(res, x, y, z) --print 3 1 2 4
```

## 클로저

루아의 함수는 렉스컬 스코프를 가지고 일급 객체이기 때문에 JS와 마찬가지로 클로저 사용이 가능합니다.  
```lua
names = {"ronaldo", "messi", "bale"}
grades = {ronaldo = 7, messi = 10, bale = 11}
-- grade로 정렬 
function sortbygrade (names, grades)
    table.sort(names, function (key1, key2)
        return grades[key1] > grades[key2]
    end)
end
```
위의 코드에서 sort 함수의 두번째 인자인 함수 호출 시에 grades를 참조합니다.  
전역 변수도, 지역 변수도 아님에도 불구하고 업밸류 혹은 호이스팅으로 불리는 동작이 수행되어 grades를 참조 가능합니다.  
따라서 클로저는 자신을 감싼 환경의 지역 변수에 접근할 수 있는 함수라고 볼 수 있습니다.  

## Table

루아만의 독특한 자료형으로 연관 배열을 구현한 기능입니다.  
nil 타입과 NAN 값을 제외한 모든 타입, 값을 할당할 수 있습니다.  
인덱스는 0이 아닌 1부터 시작하는 것을 권장합니다.  
대부분의 루아 라이브러리가 1부터 읽기 때문이라고 합니다.  
만약, 0부터 시작하게 표현하고 싶다면 아래와 같이 구현하면 됩니다.
```Lua
season = {[0]="spring", "summer", "fall", "winter"}
```
인덱싱을 필드 이름으로도 사용 가능해 유연하고 편리합니다.  
또한 a.name과 같은 레코드 표현법도 제공합니다.  
```a.name = a["name"]```
프로퍼티로는 기본적인 숫자, 문자열 뿐만 아니라 함수도 가질 수 있습니다.
```lua
-- x는 "x" y는 "y"이다.
table = { x = 3, y = 4 } 
    print(table.x)
--print(table."x") expect nil
    print(table["x"])
--print(talbe[x]) expect nil
```
테이블에 대해서 좀 더 알고 싶다면 [링크](http://codeshared.com/archives/36)를 참고하세요.  

## Userdata

루아의 변수에 C/C++ 등의 데이터를 저장하기 위한 자료형입니다.  
따라서 userdata의 값은 메모리 블록을 그대로 나타냅니다.  
크게 full userdata와 light userdate가 존재합니다.  
전자는 루아에서 메모리 블록을 관리하고 후자는 C 포인터 값입니다.  
userdata에는 할당과 동일성 검사를 제외하고 어떤 연산도 미리 정의되어 있지 않습니다.  
또한 루아 내에서는 userdata 값을 생성하거나 변경할 수 없으며 C API를 통해서만 가능합니다.  

## 연산자

~= 연산자는 != 연산자를 대체합니다.  
.. 연산자는 문자열 결합 연산자는 .(php)를 대체합니다.  
삼항 연산자는 value = a and true or false (a?b:c)로 표현합니다.  
= 연산자는 열거형으로도 가능합니다 x, y= 1 , 2, 3 (3는 버려짐) x, y = 1 (y는 nil)  
특이하게 a++, a += 2 등 단축 연산을 제공하지 않습니다.  

## 반복문과 조건문

일반 for문은 아래와 같이 작성합니다.
```lua
for i = 1, 4 do -- default add iter = 1, 일반적인 for문
    print(i)
end
```
특이하게 제네릭 for문(반복자 함수형)을 지원합니다.  
- pairs()  
- ipairs()  
- next()  
- io.lines()  
- file:lines()  
- 사용자 정의 함수

pairs와 ipairs는 아래와 같이 동작합니다.  
```lua
testTable = {
    "1",
    "2",
    c = 4,
    d = function() return 4
}
for key, value in pairs(testTable) do
    print(key, value) 
end
for index, value in ipairs(testTable) do
    print(key, value)
end
--[[ 
실행 후
1	1
2	2
d	function: 0xbd1740
c	4
nil	1
nil	2
]]
```
이를 통해 pairs는 key로 ipairs는 index로 동작하는 것을 알 수 있습니다.  
if문은 특별히 어렵지 않습니다.  
```lua
local isTrue = true
if isTrue == true then
    print("true")
else
    print("false")
end --return true
```
while 또한 존재하며 repeat도 존재합니다.
```lua
local i = 0
local sum = 0
repeat
    i = i + 1
    sum = sum + i
until i == 10
print(sum) --55
sum = 0
while i ~=0 do
    sum = sum + i
    i = i - 1
end
print(sum) --55
```
기본 골조는 비슷하지만 while은 비교조건이 참이면 계속 반복 수행하고 repeat은 참이 될 때까지 수행한다는 차이가 있습니다.  
## Curly braket

테이블을 생성할 때만 사용합니다.  
일반적으로 자주 쓰이는 "{"은 do, then 등으로 "}"은 end로 매칭 되어 조건식, 함수 등을 나타내는 데 사용합니다.  
## 전역 변수

루아의 모든 전역 변수는 환경(envirnment)라고 불리는 정규 테이블 안에 보관됩니다.
아래의 코드는 현재 환경에 정의된 모든 전역 변수의 이름을 출력합니다.
```lua
for n in pairs(_G) do
    print(n)
end
```
## 모듈과 패키지

루아에서는 require이라는 함수를 통해 다른 모듈을 호출 할 수 있습니다.

## Scope

블록 단위의 스코프를 가집니다.  
이는 위에서 말했듯이 스코프 생성은 '{}'가 아닌 do ~ end 혹은 then end 등을 사용합니다.  
자바스크립트의 호이스팅은 발생하지 않습니다.  
```lua
x = 10                -- 전역 변수
do                    -- 새 블록
    local x = x         -- 값 10으로 새 'x'
    print(x)            --> 10
    x = x + 1
    do                  -- 또 다른 블록
        local x = x+1     -- 또 다른 'x'
        print(x)          --> 12
    end
    print(x)            --> 11
end
print(x)              --> 10  (전역 변수)
```
## Self

C++의 this나 js의 self와 유사한 의미로 사용됩니다.  
호출된 자신을 지칭합니다.  
일반적인 '.' 연산자를 통한 테이블내의 함수를 구현하면 테이블 자신(self)로 접근할 수 없습니다.  
따라서 테이블 내의 함수 호출 시 객체를 직접 넘겨주어야 합니다.  
```lua
function Test()
    local tbl = {}
    tbl.values = {}
    function tbl.set(self, key, value)
        self.values[key] = value
    end
    function tbl.get(self, key)
        return self.values[key]
    end
    return tbl
end
temp = Test()
temp.set(temp, "test", 111)
print(temp.get(temp, "test"))
```
위의 방식은 temp 테이블 자신의 메소드를 호출함에도 불구하고 자신을 넘겨주어야 합니다.  
이러한 번거로움을 없애기 위해 :(colon)을 제공하고 있습니다.  
좀 더 간편해진 코드는 아래와 같습니다.  
```lua
function Test()
    local tbl = {}
    tbl.values = {}
    function tbl:set(key, value)
        self.values[key] = value
    end
    function tbl:get(key)
        return self.values[key]
    end
    return tbl
end
temp = Test()
temp:set("test", 111)
print(temp:get("test"))
```
이를 통해 루아의 테이블의 메소드 내부에서 self를 사용하기 위해선  
':'로 메소드를 구현하고 ':'로 호출하거나 직접 인자로 객체를 넘겨주어야 한다는 것을 알 수 있습니다.  
## 주소를 가지는 타입

루아에서 테이블, 함수, 쓰레드, userdata 등 타입의 값은 모두 주소입니다.  
실제로 값을 담지 않기 때문에 할당, 매개 변수 전달, 함수 반환 등의 작업은 값의 참조를 직접 조작하는 것입니다.  
## 테이블의 복사

위에서 말했듯이 테이블 타입은 주소 값을 가지고 있습니다.  
따라서 테이블을 '=' 연산자를 통해 복사한다면 copy by value가 아닌 copy by reference가 발생합니다.
다시말해, 참조하고 있는 주소를 복사하기 때문에 복사 된 테이블에서 값을 변경하면 원본 테이블의 값에도
영향을 끼칩니다.  
```lua
Tbl = {
    person = {
        age = 23,
        name = wonhee
    }
}
test1 = Tbl
test2 = Tbl
test2.person.age = 25
print("Test1 age" .. test1.person.age)
print("Test2 age" .. test2.person.age)
```
주소의 복사가 일어났기 때문에 위의 코드의 결과 값은 모두 25입니다.    
따라서 프로젝트 내에서 값만 복사하는 로직 혹은 함수를 구현할 필요가 있습니다.  
아래와 같은 DeepCopy 함수를 구현해 copy by referece으로 인한 문제를 방지할 수 있습니다.  
```lua
Tbl = {
    person = {https://bengi.kr/1349?fbclid=IwAR261ONxQIevN9r8vhNJDF_W7_DIrD5hD27IonjxI3UrpAhyXArSjWTKBeo
                 age = 23,
                 name = wonhee
    }
}
function DeepCopy(t)
    local t2 = {}
    for k, v in pairs(t) do
        if type(v) == "table" then
                   t2[k] = DeepCopy(v)
        else t2[k] = v
        end
     end
     return t2
end
test1 = DeepCopy(Tbl)
test2 = DeepCopy(Tbl)
print("Origin age: " .. Tbl.person.age)
test1.person.age = 20
print("Test1 age:" .. test1.person.age)
print("Test2 age:" .. test2.person.age)  
```
## Metatable, Metamethod

루아에서 테이블은 거의 만능의 역할을 하지만 테이블 간 비교와 연산 불가능 같은 한계가 존재합니다.  
이를 극복하기 위해 메타 테이블을 제공합니다.  
메타 테이블도 테이블이지만 특수한 방식으로 동작하는 일반 테이블이라고 생각하면 됩니다.  
setmetatable 함수를 통해 메타 테이블을 생성할 수 있고 getmetatable을 통해 검증할 수 있습니다.  
기본적으로 테이블을 비롯한 모든 타입은 메타 테이블을 적용하지 않지만 문자열은 전용 메타 테이블을 설정합니다.  
```lua
Tbl = {}
mtTbl = {}
setmetatable(Tbl, mtTbl) -- Tbl 테이블의 메타 테이블을 설정
if getmetatable(Tbl) == mtTbl then
    print('Tbl is metatable')
end
```
이제 mtTbl은 Tbl의 메타 테이블로 설정되었기 때문에 메타 메소드들을 사용 및 재정의가 가능합니다.  
각 이벤트에 대한 키는 이벤트 이름 앞에 __가 붙은 문자열입니다.
이러한 키에 대응하는 값을 메타 메소드라고 합니다.
__add라는 키가 존재할 때 메타 메소드는 덧셈 수행 함수라고 볼 수 있습니다.  
key 값을 기준으로 union의 기능을 하는 함수를 __add에 재정의하여 구현해보았습니다.  
```lua
function Union(left, right)
    local result = {}
    for k, v in pairs(left) do
        result[k] = v
    end
    for k, v in pairs(right) do
        local temp = rawget(left, v) --raw
        if temp == nil then
            result[k] = v
        end
    end
    return result
end

function PrintTbl(table)
    for key, value in pairs(table) do
        print("{", key, ",", value, "}")
    end
end
samsung = {phone = "galaxy", car = "sm3"}
apple = {pc = "mac", phone = "iphone"}
mtTbl = {}
setmetatable(samsung, mtTbl)
setmetatable(apple, mtTbl)
-- mtTbl의 __add 메소드는 위의 union으로 재정의 됩니다.  
mtTbl.__add = Union
mergeCompany = samsung + apple
PrintTbl(mergeCompany)
-- result
--{	pc	,	mac	}
--{	price	,	10000	}
--{	phone	,	iphone	}
```
key를 기준으로 하였기 때문에 phone = "galaxy"이 phone = "iphone"으로 바뀌었습니다.  
'+' 연산자가 메타 메소드의 __add로 대체되었고 __add는 Union으로 재정의 되었습니다.  
함수 포인터와 오버라이딩의 모습과 비슷하다고 볼 수 있습니다.  
또 다른 예제입니다.
```lua
set = {}

function set.new(l) --set.new 함수로 생성된 모든 집합은 동일한 테이블을 자신의 메타테이블로 갖는다.
    local set = {}
    setmetatable(set, mt)
    for _, value in ipairs(l) do
        set[value] = true
    end
    return set
end

function set.union(a, b) -- 합집합
    local res = set.new{}
    for key in pairs(a) do
        res[key] = true
    end
    for key in pairs(b) do
        res[key] = true
    end
    return res
end

function set.intersection(a, b) -- 교집합
    local res = set.new{}
    for key in pairs(a) do
        res[key] = b[key]
    end
    return res
end

function set.tostring (set)
    local l = {}
    for e in pairs(set) do
        l[#l + 1] = e
    end
    return "{" .. table.concat(l, ", " ) .. "}"
end

function set.print (s)
    print(set.tostring(s))
end

local mt = {} -- 집합에서 사용할 메타 테이블

mt.__add = set.union
mt.__mul = set.intersection
s1 = set.new{1, 2, 4, 6, 8} -- 8의 약수
s2 = set.new{1, 3, 9} -- 9의 약수
print(getmetatable(s1))
print(getmetatable(s2))
set.print(s1)
set.print(s2)
s3 = s1 + s2
s4 = s1 * s2
set.print(s3)
set.print(s4)
```

테이블에 존재하지 않는 필드에 접근할 때 nil으로 처리합니다.  
__index 메타메서드는 이와 같은 처리를 위해 존재하는 메타 메서드입니다.  
```lua
Tbl = {"won"}
mtTbl = {}
fucntion testIndex()
    return "hee"
end
setmetatable(Tbl, mtTbl)
mtTbl.__index = check_idx
print(Tbl[1], Tbl[2]) --wonhee
```
위의 코드에서 Tbl[2]처럼 없는 key를 호출한다면 testIndex를 호출합니다.  
때문에 __index를 이용해 객체지향의 constructor를 구현하곤 합니다.  
또한, __newindex 메타 메서드는 
## Coroutine

코루틴 관련 함수들은 coroutine 테이블에 존재합니다.  
코루틴은 상태는 크게 suspended(일시정지), running(실행), dead(죽음), normal(정상)로 이루어집니다.  
resume()의 호출을 통해 yield를 만날때까지 지속하비다. 
코루틴은 프로그래머가 임의로 루아 내부의 라이브러리 함수(coroutine.yield)를 호출해야 컨텍스트 스위칭이 이루어집니다.  
이는 쓰레드와 달리 CPU Time으로 사용할 수 있는 코루틴이 단일하다는 것을 의미한다.  
고유의 스택과 지역 변수 등을 가지면서도 다른 코루틴들과는 전역 변수를 공유하기도 합니다.  
또한 서브루틴과는 달리 로직 실행 완료 후 호출자에게 제어권이 돌아가지 않습니다.  
```lua
function test (a)
    print("arguments", a)
    return coroutine.yield(2 * a)
end

co = coroutine.create(function (a, b)
     print("co-body1", a, b)
     local r = test(a + 1)
     print("co-body2", r)
     local r, s = coroutine.yield(a + b, a - b)
     print("co-body3", r, s)
     return b, "end"
end)

print("test1", coroutine.resume(co, 1, 10))
print("test2", coroutine.resume(co, "r"))
print("test3", coroutine.resume(co, "x", "y"))
print("test4", coroutine.resume(co, "x", "y"))  
-- result
-- co-body1	1	10
-- arguments	2
-- test1	true	4
-- co-body2	r
-- test2	true	11	-9
-- co-body3	x	y
-- test3	true	10	end
-- test4	false	cannot resume dead coroutine
```

## Module

보통 루아 파일은 require 혹은 include를 통해 다른 모듈에서 사용할 수 있습니다.  
require의 반환값은 캐시되어 여러번 실행되어도 한번만 메모리에 올라 갑니다.  
dofile은 require과 유사하나 캐시하지 않습니다.  
loadfile은 파일을 읽지만 호출하지 않으면 실행하지 않습니다.  
loadstring은 문자열에 대한 loadfile입니다. 역시 호출하지 않으면 메모리에 올라가지 않습니다.  
## Garbage Collector

 
루아에서는 점진적 mark-and-sweep 방식으로 메모리를 회수합니다.  
C에서 lua_gc를 호출하거나 루아에서 collectgarbage를 호출해서 주기를 바꿀 수 있습니다.  
루아 스크립트 자체가 가볍기 때문에 프로그래머가 gc 부분을 직접 건드리기 보다는 nil 처리나 최적화된 반복 로직을 구현하는게 좋다고 생각합니다.

## Debugging

루아는 기본적으로 C/C++의 공통 부분을 확장한 언어입니다.  
호스트 프로그램(C/C++)이 루아의 함수 등을 호출하는 구조입니다.  
루아 파일을 실행 중에 오류가 발생하면 제어가 호스트로 되돌아가기 때문에  
호스트에서도 처리가 가능하나 루아에서 제공하는 error, pcall 등을 사용하는 것이 바람직합니다.  

## Lua Tinker

C++와의 binding을 목적으로 존재합니다.  
LuaBind에서 불필요한 것을 덜어내어 만들었다고 합니다.  
따라서 훨씬 가볍고 컴파일 시간을 단축하였다고 합니다.  
위와 같은 방식으로 lib 파일과 header파일을 include하면  
C++에서 lua를 호출할 수 도 lua에서 c++ 함수를 호출할 수 있습니다.  
하지만 서로 가급적이면 호출하지 않는게 독립성에 좋다고 생각합니다.

## 참고자료

[프로그래밍 루아](https://books.google.co.kr/books/about/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EB%A3%A8%EC%95%84.html?id=mWAvMwAACAAJ&source=kp_cover&redir_esc=y)
[LuaTinker](https://www.gpgstudy.com/gpgiki/LuaTinker_Manual)  
[LuaTinker-Mannual](https://www.gpgstudy.com/gpgiki/LuaTinker)  
[LuaReference](http://www.lua.org/manual/5.3/)  
[한글 레퍼런스](https://wariua.github.io/lua-manual/5.3/manual.html#2.4)
