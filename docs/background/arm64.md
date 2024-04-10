---
layout: default
title: ARM64
nav_order: 4
parent: Background
last_modified_date: 2023-07-22
---

# Architecture

## 소개

- IoT, 스마트폰 등 다양한 기기에서 쓰인다.
- 대표적인 RISC 계열의 마이크로프로세서이다. (RISC 계열 : 복잡한 명령어들 없애고 사용 빈도 높은 명령어들만 남겨두기, 복잡한 처리는? 소프트웨어에게 넘기기)
- 명령어 셋의 개수는 적고, 레지스터가 굉장히 많다.

## ARM64 V8

- 물리 주소 범위가 확장되었다.
- 4GB 이상의 가상 메모리 사용가능하다.
- 하드웨어 암호화 명령어 지원으로 암호화 및 복호화 성능이 향상했다.

### ALU

32bit 연산이 가능한 ALU가 제공된다. 보통 다른 CPU에서는 shift 명령이 따로 존재하지만, ARM 에서는 명령어의 옵션으로 적용 시킬 수 있다.

# 레지스터

## Xn ( 64bit )

- X0 ~ X7 : Arguments & ret values
- X8 : Indirect result
- X9 ~ X15 : Temporary
- X16 ~ X17 : Intra procedure call temporary
- X18 : Platform define use
- X19 ~ X28 : Temporary

## Wn ( 32bit )

## Sn ( 32bit floating point )

## Dn ( 64bit floating point )

## 특수목적 레지스터 (amd64와 비교)

- X29 : frame pointer (= SFP)
- X30 : link register (= RET)
- SP : stack pointer (= RSP)
- PC : program counter (= RIP)
- XZR : zero register (64bit, 32bit는 WZR)

# ARM64 명령어

## STORE

### STP

- Store Pair of registers

`stp x29, x30, [sp, #-32]!`

prologue 상황에서 주로 발생한다.

1. sp를 sp-32 위치로 이동.
2. 해당 위치에 x29 저장
3. 해당 위치+8에 x30 저장

---

`stp x29, x30, [sp, #-32]` (느낌표가 없다.)

보통의 경우 존재하지 않는 상황일 것이다.

1. x29를 sp-32 위치에 저장
2. x30을 sp-24 위치에 저장

### STR

- Store register

`str x1, [x2], #8`

1. x2의 주소에 x1값 저장
2. x2주소 +8 증가

---

`str x1, [x2, #16]`

1. x2+16의 주소에 x1값 저장

---

STRB, STRH, STR 이라는 명령어가 존재함

- STRB : byte, 8bits
- STRH : halfword, 16bits
- STR : word, 32bits (doubleword, 64bits)

## MOVE

### MOV

- move register

`mov x29, sp`

1. sp의 값을 x29로 옮김

---

`mov x1, 0x100, lsl #16`

1. 0x100을 4비트 left shift한 값을 x1에 저장 (x1=0x100<<4)

mov 뒤의 bit shift 연산에는 lsl, ror 등이 존재한다.

### MOV[K|N|Z]

`movk R0, #0x1234` → mov with keep

1. R0 레지스터의 하위 16비트에 0x1234 값 넣기 (keep)

---

`movn R1, #0xff` → mov with not

1. R1 레지스터의 하위 8비트를 0xff 값으로 반전 (모두 반전)

---

`movz R2, #0xfff` → mov with zero

1. R2 레지스터의 하위 12비트를 0xfff값으로 설정 (비트를 모두 1로 설정)

(movz, movk는 동일한 결과를 이끌어낸다.)

## BRANCH

### B | BL | BR | BLR

- branch / jump

`b <label>`

1. label로 이동한다.

---

`bl <label>`

1. 0x30 레지스터에 pc+4주소 저장
2. label로 점프

---

`br <register>`

1. register 주소로 이동

---

`brl <register>`

1. 0x30 레지스터에 pc+4 주소 저장
2. register로 점프

## LOAD

### LDR

- load register

`ldr x1, [x2, #16]`

1. x2+16 주소에 있는 값 읽기
2. x1에 저장

---

`ldr x1, [x2], #8`

1. x2에서 값 읽기
2. x1에 저장
3. x2에 8 더하기

---

### LDP

- load pair of register

`ldp x0, x1, [sp, #8]`

1. sp+8에 있는 메모리 값을 x0에 로드
2. sp+16에 있는 메모리 값을 x1에 로드

---

`ldp x0, x1, [sp], #8`

1. sp+8의 값을 x0에 로드
2. sp+16의 값을 x1에 로드
3. sp = sp+8

## ADDRESS

### ADR | ADRP

- get address

`adr x1, loop`

1. loop 레이블의 주소를 x1에 저장

(컴파일 타임에 레이블의 주소를 가져와서 상대적으로 빠르게 주소를 얻어올 수 있다.)

---

adrp의 경우, adr과 수행하는 동작은 같지만, adrp는 4kb 정렬된 페이지의 주소를 얻어올 때 사용된다.

0x1023 

- adr : 0x1023
- adrp : 0x1000 (하위 12비트)

## RETURN

### RET

- return

`ret`

1. 0x30 레지스터로 점프

`ret x1`

1. x1 레지스터로 점프

## ARITHMETIC

### add | sub | mul | div
