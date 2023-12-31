---
title: "Design"
description: "소프트웨어 설계 원칙"
time: 2023-11-29 18:43:38
tags:
  - software-engineering
---

## 좋은 소프트웨어 설계

Udemy에서 Software Engineering을 공부하고 있다. 디자인이 잘 된 소프트웨어란 **변경이 쉬운 소프트웨어** 라고 정의한다. 변경이 쉬운 소프트웨어는 유지보수가 쉬우며 확장하기가 편하다고 한다. 변경이 쉬운 소프트웨어는 각 구성요소의 의존성이 낮다고 한다. 문제를 해결하기 위해 필요한 모듈 들이 다같이 서로 의존하고 있다면 변경하기가 어려울 것이고, 어디서 부터 변경해야 하는지 감이 잡히지 않는다고 한다.

맞다. backtest도구도 전략, 데이터, 백테스팅 방식이 서로 다르도록 분리하고 있지만, 백테스팅의 세부 부분(수익율 계산, 리벨런싱, 구입 조건, 판매 조건 등등)을 변경하고자 했을 때 무지 어려웠었다. 

## 이미 있는 소프트웨어 변경이 어렵다

다른 한 편으로는 변경이 쉬운 소프트웨어를 개발 한다는 것은 정말 어려운 일인 것 같다. 특히 **이미 개발되어진 소프트웨어**를 변경하고자 한다면 처음부터 다시 개발하거나, 그 많은 덩굴을 해집고 정리하는 수 밖에 없다.

## 소프트웨어 설계는 방정리와 같다

소프트웨어 개발은 방정리와 같다고 생각한다. 어떤 물건이 어느 위치에 있을지 미리 정한다면 방정리가 쉽겠지만, 그런 규칙이 없을 경우에는 규칙을 만들면서, 다시 재배치하고, 다시 유지보수하고 등등 수많은 수작업이 기다리고 있다. 이를 바로 해결할 순 없을까? 나는 있다고 생각한다.
**이미 검증된 구조를 익혀 유사한 문제가 발생했을 때 이를 적용하면 될 것이다.** <<디자인 패턴>>, <<클린 코드>>, <<클린 아키텍처>> 등등.. 잘 짜여진 구조와 알고리즘이 이미 존재한다.

## 간접 경험이라도 해보자

책을 읽어 기록을 꾸준히 하고 손에 익도록 반복 숙달을 한다면 유지보수가 가능한 코드를 설계할 수 있을 것이다. 소프트웨어 개발은 특이한게, 오픈소스 커뮤니티가 활성화 되어 있고, 다른 사람이 디자인한 코드를 분석할 수 있으며, 자신이 직접 개발도 가능하기 때문에 간접 경험이 아닌 직접 경험도 할 수 있다. 미래의 자신을 위하여 그리고 동료들을 위하여 유지보수가 쉽고 읽기 쉬운 코드를 작성하자.

## 그렇다면 내가 생각하는 코드

적은 줄의 코드, 가독성, 각 구성요소들 간의 의존성이 적은 코드가 최고가 아닐까? 너무 이상적인가 


