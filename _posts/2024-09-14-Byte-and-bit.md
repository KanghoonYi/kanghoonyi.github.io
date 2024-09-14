---
title: 1 Byte and The number of bits(1Byte와 bit수)
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-14 19:31:00 +0900
categories: [Programming, Computer Basics]
tags: [Computer Science, programming]
pin: false
math: true
---

1bit(비트)는 0과 1의 값을 가질 수 있는 데이터의 최소 단위입니다.

이 bit를 특정갯수 만큼 묶어서, 1byte(바이트)라고 합니다.

## Byte의 의미

최초의 byte는 ‘하나의 character 데이터를 나타내는데 필요한 bit수’를 의미했습니다.

> *Byte* denotes a group of bits used to encode a character, or the number of bits transmitted in parallel to and from input-output units.  
> \- from ‘[PLANNING A COMPUTER SYSTEM](https://archive.computerhistory.org/resources/text/IBM/Stretch/pdfs/Buchholz_102636426.pdf)’

> 컴퓨터에선, character를 특정 정수(bit로 표현한 integer)로 mapping하여 사용합니다.
{: .prompt-tip }

초기 컴퓨터는 주로 text데이터를 처리했기 때문에, 자연스럽게 byte가 메모리 공간을 할당하는 최소 단위가 되었습니다(character를 저장해야하기 때문에). 이후 수많은 ‘computer architecture’에서 **메모리의 주소를 나타내는 최소 단위**로 사용되었습니다.

> 현재처럼 1byte = 8bit인 표준이 없을때는, Byte의 bit수는 Hardware에 따라 달라졌습니다.  
> 특히 초기 컴퓨터에선, 6bit character code를 사용하는 시스템이 있었습니다.
{: .prompt-tip }

### 표준이된 1 byte = 8 bits

고정된 bit수로 문자를 인코딩하는 **ASCII**(American Standard Code for Information Interchange)가 표준으로 자리잡자, 자연스럽게 ‘1 byte = 8 bits’로 자리잡게 되었습니다.  
ASCII는 원래 7bit로 128개의 문자를 표현하였습니다. 여기에는 영어와 몇가지 기호가 포함되어 있었습니다.  
이를 8bit로 확장하여, 256개의 문자를 표현할 수 있게하여 ‘1 byte = 8 bits’로 사용하게 되었습니다.  
## 숫자(Number)와 Byte

‘1 byte = 8 bits’가 표준으로 자리 잡으면서, 숫자를 저장하기 위한 공간도 byte단위로 표시하게 되었습니다.  
기본적인 단위인 byte의 bit수 8을 기준으로,  
8-bit, 16-bit, 32-bit, 64-bit, 128-bit 정수가 있습니다.  
각 bit수에 따라, 정수값 표현 범위는, 아래와 같습니다.  

$$
-(2^{n-1})\sim(2^{n-1}-1)
$$

예를 들어, 8-bit 정수의 경우, $-(2^7)\sim2^{7}-1$ 로, $-128\sim127$의 범위를 갖습니다.

여기서,  

$n-1$로 bit수에서 1을 제외하는 이유는,
: 가장 왼쪽 bit를 부호(+, - 양수인지 음수인지 나태내는것)를 표기하는데 사용하기 때문입니다.
: 양수의 범위는, `0 0000000` 에서 `0 1111111` 까지로 0에서 127까지 가리킵니다.
: 음수의 범위는, `1 0000000` 에서 `1 1111111`까지, -128에서 -1까지를 나타냅니다.

양수 범위에서 -1을 하게 되는 이유는,
: 7-bit로 표현할 수 있는 숫자의 가지수는 128개(=$2^7$)이지만, 이는 갯수를 의미하고, 0을 포함시키면 표현 가능한 최대 양수값은 127이 됩니다.

### Weakly typed언어(javascript)에서의 Number

약타입 언어(Weakly typed language)는 변수 선언시 별도의 type을 지정하지 않습니다. 때문에, 어떤 숫자를 저장하던, 이미 정해진 memory공간을 사용합니다.  
javascript를 예로 들면, number는 다른 프로그래밍 언어의 double과 같은 공간을 차지합니다.  
이 double은 소수점표현(분수값표현)이 가능합니다.  
javascript에선, number가 double로 다루어지기 때문에, 정수 32를 저장하더라도, 내부적으로 소수점을 포함한 값으로 저장합니다.  
[Number \- JavaScript \| MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number)

## References

Wikipedia에서의 Byte정보
: [Byte](https://en.wikipedia.org/wiki/Byte)

Javascript의 Number
: [Number \- JavaScript \| MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number)

Rust의 Integer Types
: [Integer Types](https://doc.rust-lang.org/beta/book/ch03-02-data-types.html#integer-types)
