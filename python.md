Python을 배우면서 틈틈히 정리한 내용을 공유합니다. 자주 쓰는 언어가 아니라 그런지 정말 잘 까먹게 되더군요.
그래서 이 문서, 저 코드를 보면서 간략하게 정리했습니다.

1989년 12월 크리스마스를 심심하지 않게 보내려고, 네덜란드 암스테르담에 사는 귀도 반 로섬이 혼자 집에서 취미 삼아 재미로 개발한 프로젝트임. 연구실은 닫히고 집에서 특별히 할일이 없어서 개발했다고 함. 이름은 영국 드라마(Monty Python's Flying Circus)를 참고

http://en.wikipedia.org/wiki/Guido_van_Rossum
Hello World 예제


#!/usr/bin/python
# hello.py
print "Hello World\n"


chmod 755 hello.py

Commnet


# 이것은 주석입니다.

import sys   # 이것도 주석입니다.


"""

이것도 주석입니다.

여러줄을 주석처리할 때 사용합니다...

"""


Datatype




객체 자료형	상수와 사용법의 예
수치형(Numbers)	정수(2134), 실수(2.71), 롱형 정수(345L), 복소수(4+5j)
문자열(Strings)	‘spam’,“ eggs”
리스트(Lists)	[1,‘ two’, 3, [5,‘ five’]]
사전(Dictionaries)	{ 'gslee':5284,'kslee':5582,'mylee':5382}
터플(Tuples)	(1,3,’five’)
파일(Files)	f = open( ‘toasts’)

정수형 상수

>>> a = 23 # 10진 상수
>>> b = 023 # 8진 상수, 0으로 시작하면 8진수이다.
>>> c = 0x23 # 16진 상수, 0x 혹은 0X로 시작하면 16진수다.
>>> print a, b, c # 10진수로 출력
23 19 35

실수형 상수

이렇게 한 줄에 여러 변수 값을 할당할 수 있다.

>>> e, f, g = 3.14, 2.16e-9, 3E220
>>> print e, f, g
3.14 2.16e-009 3e+220

Long 형 상수

>>> h = 123456789012345678901234567890L
>>> print h * h
15241578753238836750495351562536198787501905199875019052100L

유효 자리수는 메모리가 허용하는 만큼 가능하다.
복소수형 상수

>>> c = 4+5j
>>> d = 7-2J
>>> print c * d
(38+27j)

문자열

문자를 표현하기 위한 자료형

print 'Hello World!'

print "hello World!"


따옴표(Single Quotation mark), 큰 따옴표(Double Quotation Mark) 모두 사용된다.
리스트


L = [1, 4, 3, 2, 5, 'aa']
print len(L)
print L[1:3]
print L[5]
print L+L
print L*3

L.append(6)
print L
L.reverse()
print L
L.sort()
print L


list.py
튜플

튜플과 리스트의 차이는 튜플은 값 변경이 안되고, 리스트는 가능하다.

t = (1, 4, 3, 2, 5)

print len(t)

print t[1:3]

print t[1]

print t+t

print t*3


tuple.py
사전 만들기

아이템들은‘키’와‘값(value)’으로‘key:value’형식으로 구성된다.
>>> phone = {} # 공 사전 생성
>>> phone = {'jack':9465215, 'jim':6851325, 'Joseph':6584321} # 초기값 부여
>>> len(phone) # 사전의 크기(아이템의 개수)
3


phone = {}

phone = {'joone':12777347, 'minsoo':23443333, 'Soony':23454458}

print len(phone)

print 'the value of joone is %s' %  phone['joone']


# 검색 추가 변경

phone['jack'] = 33455634

print phone

phone['joone']= 1111111# 삭제

del phone['jack']

print phone# 키, 값, 아이템 얻기

print phone.keys();

print phone.values();

print phone.items();

# 키보드로부터 키 스트링(이름) 입력을 받는다. ‘name?’은 프롬프트다.

name = raw_input('name?')

# 키를 가지고 있는지 검사

if phone.has_key(name):

print phone[name] # 있으면 전화번호 출력

else:

print name,'not found' #없으면


dic.py
관계연산자(relational operator)

> 크다
< 작다
>= 크거나 같다
<= 작거나 같다
== 같다
!= 같지 않다
<> 같지 않다
포맷 스트링


s3 = 'Hi %s! How are you doing?'

print s3 % "joone"

s4 = '%s * %s = %s'

print s4 % (12, 23, 12*23)

print '%5d %o %x %5.2f' % (23, 23, 23, 43.123)


제어문

파이썬은 단순하게 if, for, while이라는 3개의 제어문을 갖고 있다.
if 문


n = int(raw_input('value?'))

if n > 0:

print 'positive'

elif n < 0:

print 'negative'

else:

print 'zero'


한 가지 주의 깊게 봐야 할 점이 있는데 들여쓰기(indentation)이다. 즉, if - elif - else는 같은 열에 있어야 하고, 세 개의 print 문은 탭
혹은 같은 수의 스페이스를 이용해 들여쓰기를 한다. 파이썬에서는 이와 같은 들여쓰기 필수다.
for 문


a = [0,1,2,3]
for x in a:      # a에 있는 모든 x에 대해
print x,    # x값 출력



for x in range(100):

print x


순차적으로 숫자를 증가시켜가면서 반복할 경우에는 range함수를 이용한다
while문


a, b = 0, 1
while b < 100:
print b,
a, b = b, a+b


함수 정의와 호출

def 함수명 (인수들..):
문(statements)
return <값>


def add(a, b):

return a+b

print add(1,2)



add(int, int);
add(int, double);
add(double, int);
add(double, double);


파이썬은 자료형을 동적으로 결정하므로 다른 언어에서 해야 하는 많은 중복(overloading) 작업을 피하게 해준다.
파일 입출력


파이썬의 파일 입출력(File I/O) 기능은 C에서 사용하는 함수를 확장한 것

C의 래퍼(wrapper)로서 대부분의 C 파일 입출력 기능과 추가적인 편의 함수를 제공

파일을 사용하기에 앞서 우선 파일을 오픈해야 하는데,

이때 파일명과 처리 모드를 인수로 넘겨줘야 한다. 처리 모드는‘w:쓰기, r:읽기, a:추가하기’이다.



f = open(‘testfile’, ‘w’)                  # 쓰기 모드로 testfile을 오픈한다.
f.write(‘파이썬 파일 쓰기 테스트 중이예요!\n’)    # 문자열을 출력한다
f.close()                                  # 파일을 닫는다. 생략 가능


읽기 쓰기 메쏘드
파일 객체의 . 더 다양한 메쏘드에 대해 배워보자. 먼저 읽기를 위한 메쏘드는

read()     파일 전체 내용을 한꺼번에 문자열로 읽어들인다

read(N)   N의 양의 정수만큼 읽기

readline()  한줄씩 읽기

readlines() 파일 전체를 라인단위로 끊어서 리스트에 저장한다
write(S), writelines(L)
파일 목록 읽기


#!/usr/bin/python
# file : readfiles.py

import sys
import os

def main(argv):
fileNames = os.listdir(argv[0])
fileNames.sort()

for fileName in fileNames:
print '%s/%s' % (argv[0], fileName)

if __name__=="__main__":
main(sys.argv[1:])


명령행 인수 처리


import sys # for handling argv
import re   # import regular expression module for using subn

def replace(fname, srcstr, deststr):
f = open(fname)
txt = f.read()
txt = re.subn(srcstr, deststr, txt)[0]
return txt

if __name__ == '__main__':
if len(sys.argv) != 4:
print "Usage : replace filename srcstr deststr"
sys.exit()
print replace(sys.argv[1], sys.argv[2], sys.argv[3])


command.py

import sys
import os

def main(argv):
for arg in argv:
print arg
if __name__=="__main__":
main(sys.argv[1:])


OOP


class MyClass:

def setValue(self, val):

self.value = val;

def getValue(self):

print self.value

c = MyClass()

c.setValue('Hello')

c.getValue()


oop.py
Debugging

python -m pdb oop.py
참고문헌


이강성, 파이썬의 기본 구문과 기초자료형 이해, 마이크로소프트웨어 2000년 7월호
