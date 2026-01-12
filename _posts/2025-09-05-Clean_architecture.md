---
title: "Clean Architecture"
category: "Architecture"
categories: ["Architecture", "Microservices"]
---

## Introduction to Clean Architecture

### What is Clean Architecture?

클린 아키텍처는 소프트웨어 시스템을 설계할 때 다음과 같은 핵심 목표를 달성하기 위한 아키텍처
패턴입니다: 기본적으로는 Layered Architecture을 베이스로 하고 있으나 거기서 Soc(Seperparation of
Concerns)와 Dependency Rule을 엄격히 적용한 아키텍처 패턴입니다.

- **테스트 가능성** (Testability)
- **유지보수성** (Maintainability)
- **적응성** (Adaptability)

이 아키텍처 접근 방식은 특정 프레임워크나 기술로부터 독립적으로 도메인 로직을 보호함으로써, 기술
변화에 따른 투자 비용을 보호합니다.

- Robert C. Martin calls a Screaming Architecture : 좋은 아키텍쳐는 그것 자신으로 목적과
  유스케이스를 “소리내어 말해야(scream)”합니다. 프레임이나 툴로써가 아니라. 아키텍쳐자체가 서류이자
  그 시스템의 코어 목적을 나타내고 기능을 전반적으로 히해할수있는 도구어이어ㅑ 한다 ( The
  architecture itself becomes a form of documentation, revealing the system’s core purpose and
  functionality at a glance) 특정 기술 프레임워크가 아니라 그 자체가 비지니스로직을 내포하고 있어야
  한다.

### Key Principle: Strategic Postponement

> "Good architecture helps you postpone decisions."

이 원칙은 단순히 결정을 미루는 것이 아니라, 다음을 의미합니다:

- 더 많은 정보가 있을 때 더 나은 결정을 내릴 수 있도록 설계
- 불확실성이 높은 상황에서 섣부른 결정을 피함
- 핵심적인 부분에 집중하면서 확장 가능성을 유지

良いアーキテクチャは、すべての要件が明確になるまで重要な技術的決定を先送りすることができます。

## Core Concepts and Terminology

아키텍처를 이해하고 논의하기 위한 핵심 용어들을 먼저 살펴보겠습니다:

| 한국어                                                | 일본어         | 발음                     |
| ----------------------------------------------------- | -------------- | ------------------------ |
| 유지보수성 (Maintainability)                          | 保守性         | ほしゅせい               |
| 변경 용이성 (Modifiability)                           | 変更容易性     | へんこうよういせい       |
| 결합도 (Coupling)                                     | 結合度         | けつごうど               |
| 응집도 (Cohesion)                                     | 凝集度         | ぎょうしゅうど           |
| 코드 품질 (Code Quality)                              | コード品質     | コードひんしつ           |
| 가독성 (Readability)                                  | 可読性         | かどくせい               |
| 경직성 / 변화에 대한 저항 (Rigidity)                  | 硬直性         | こうちょくせい           |
| 얽힌 의존 관계 (Tangled Dependencies / High Coupling) | 複雑な依存関係 | ふくざつないぞんかんけい |

## Principles of Good Architecture

### Core Objectives

잘 설계된 아키텍처는 다음과 같은 특성을 가져야 합니다:

- **높은 재사용성과 확장성**
- **낮은 결합도**
- **높은 응집도**

이러한 목표를 달성하기 위해서는 다음과 같은 객체지향 설계 원칙을 신중하게 적용해야 합니다:

- 추상화 (Abstraction)
- 캡슐화 (Encapsulation)
- 단일 책임 원칙 (Single Responsibility Principle)

## Common Architecture Problems

잘못된 아키텍처 설계는 다음과 같은 문제들을 초래할 수 있습니다:

### 1. 프로젝트 관리의 어려움

- 2주 프로젝트가 몇 달로 늘어나는 현상
- 단순한 기능 추가가 큰 작업으로 변질
- 민첩성(Agility) 저하로 인한 비즈니스 대응력 약화

### 2. 코드베이스의 극단적 패턴

#### a) 복잡한 상속 구조(Tangled Hierarchies)

- 과도한 코드 재사용 시도로 인한 깊은 상속 계층
- 예측 불가능한 부작용 발생
- 시스템 취약성 증가

#### b) 거대 클래스 증후군(Monolithic Classes)

- 추상화 부재로 인한 코드 중복
- 유지보수와 일관성 확보의 어려움
- 구조적 변경의 복잡성

### Seperparation of Concerns (관심사의 분리)

관심사의 분리는 소프트웨어 설계에서 매우 중요한 원칙으로, 시스템을 여러 개의 독립적인 부분으로
나누어 각 부분이 서로 다른 관심사를 다루도록 하는 것을 의미합니다. 이를 통해 각 부분은 자신의 역할에
집중할 수 있으며, 변경이 필요할 때 다른 부분에 미치는 영향을 최소화할 수 있습니다.

## Clean Architecture Overview

클린 아키텍처는 이를 인식하고 핵심 구성요소를 외부 기술의 변동성으로부터 격리하는 구조를 제공합니다.

이 구조는 동심원 형태의 계층으로 구성되며, 의존성은 항상 안쪽(더 추상적인 계층)을 향하도록 하여 핵심
비즈니스 규칙의 안정성을 보장합니다.

클린 아키텍처는 소프트웨어 시스템을 동심원 형태의 계층으로 구성합니다. 각 계층은 명확한 책임을
가지며, 의존성은 항상 안쪽(더 추상적인 계층)을 향해야 합니다.

![Clean Architecture Diagram](image.png)

### Core Layers

#### 1. 엔티티 (Entities)

- **정의**: 핵심 비즈니스 규칙을 캡슐화한 객체
- **특징**: 소프트웨어 없이도 존재할 수 있는 비즈니스 개념
- **예시**:
  - 전자상거래: Customer, Product, Order
  - 작업관리: User, Task, Project

#### 2. 유스케이스 (Use Cases)

- **정의**: 애플리케이션 특화 비즈니스 규칙
- **역할**: 엔티티와 데이터 흐름 조율
- **예시**:
  - 새 작업 생성
  - 작업 완료 처리
  - 작업 할당

#### 3. 인터페이스 어댑터 (Interface Adapters)

- **정의**: 내부/외부 시스템 간 데이터 변환 계층
- **구성요소**:
  - 컨트롤러 (HTTP 요청 처리)
  - 프리젠터 (데이터 표시)
  - 게이트웨이 (데이터 영속화)

#### 4. 프레임워크 & 드라이버 (Frameworks & Drivers)

- **정의**: 외부 도구 및 프레임워크 계층
- **예시**:
  - 웹 프레임워크 (Django, Flask)
  - 데이터베이스 드라이버
  - UI 프레임워크
  - 시스템 유틸리티

### 복잡한 상속 구조 (Tangled Hierarchies)

이 패턴은 코드 재사용을 극대화하기 위해 클래스들이 과도하게 깊고 복잡한 상속 관계를 맺을 때
발생합니다. 마치 엉켜버린 실타래처럼, 클래스 간의 관계가 너무 복잡해져서 의도하지 않은 부작용을
일으키기 쉽습니다.

#### 문제점

- 취약한 시스템 (Fragile System): 상속 관계의 최상위에 있는 부모 클래스에 작은 변경이라도 가해지면,
  상속을 받은 모든 하위 클래스에 예기치 않은 부작용이 발생할 수 있습니다. 이는 시스템 전체의
  안정성을 크게 해칩니다.

- 높은 결합도 (High Coupling): 부모와 자식 클래스 간의 관계가 매우 밀접해져서, 특정 클래스를
  독립적으로 이해하거나 테스트하기 어렵습니다. 이는 코드의 재사용성을 오히려 떨어뜨리고, 리팩토링을
  어렵게 만듭니다.

- 낮은 가독성 (Poor Readability): 개발자가 특정 클래스의 동작을 이해하려면 상속 계층 구조를 따라
  여러 부모 클래스를 거슬러 올라가야 합니다. 이 과정은 시간 소모적이며 코드 이해를 방해합니다.

## 거대하고 추상화가 부족한 클래스 (Monolithic Classes)

이 패턴은 하나의 클래스가 너무 많은 책임을 떠맡아 '만능 클래스(Do-everything Class)'가 될 때
나타납니다. 객체지향의 핵심 원칙인 **단일 책임 원칙(Single Responsibility Principle)**을 위반하며,
응집도가 낮고 결합도가 높은 결과를 초래합니다.

### 문제점

- 낮은 유지보수성 (Low Maintainability): 한 클래스에 너무 많은 기능이 집중되어 있기 때문에, 버그를
  수정하거나 새로운 기능을 추가하기가 매우 어렵습니다. 하나의 기능을 변경하려다 다른 기능에 영향을
  미치는 부작용이 흔하게 발생합니다.

- 높은 코드 중복 (High Code Duplication): 특정 기능을 필요로 하는 여러 곳에서 거대 클래스의 일부
  코드를 복사해서 사용하는 경우가 많아집니다. 이는 코드를 비효율적으로 만들고, 기능의 일관성을
  유지하기 어렵게 합니다.

-테스트의 어려움 (Difficulty in Testing): 클래스가 너무 많은 기능을 포함하고 있어 단위 테스트를
작성하기가 복잡하고, 테스트 자체의 신뢰성도 떨어집니다. 특정 기능만 테스트하기 위해 불필요한 다른
기능들을 모두 준비해야 하기 때문입니다.

## Object-Oriented Principles in Clean Architecture

### Core Object-Oriented Principles

#### 1. 抽象化 (Abstraction)

- 복잡한 세부사항을 숨기고 고수준 인터페이스만 노출
- 시스템 이해도 향상 및 구현 변경의 영향 최소화
- **주의점**: 과도한 상속은 높은 결합도를 초래할 수 있음

#### 2. カプセル化 (Encapsulation)

- 데이터와 이를 조작하는 메서드를 하나의 객체로 통합
- 객체의 내부 상태를 외부 변경으로부터 보호
- 결합도 감소에 기여

#### 3. 単一責任の原則 (Single Responsibility Principle)

- 각 클래스는 단 하나의 책임만을 가져야 함
- 높은 응집도 유지
- 예기치 않은 부작용 방지

### The Dependency Rule

Clean Architecture의 핵심 원칙인 의존성 규칙:

```txt
[ Entities ] ← [ Use Cases ] ← [ Interface Adapters ] ← [ Frameworks & Drivers ]
```

의존성은 항상 안쪽(더 추상적인 계층)을 향해야 합니다.

## Continuous Integration and Deployment

### CI (Continuous Integration)

#### 정의

개발자의 코드 변경사항을 정기적으로 공유 저장소에 통합하는 자동화 프로세스

#### 주요 구성요소

1. 자동 빌드
2. 자동화된 테스트 실행
3. 결과 보고 시스템

#### 목표

- 조기 버그 발견
- 코드 품질 향상
- 통합 문제 최소화

### CD (Continuous Delivery/Deployment)

#### 1. Continuous Delivery

- CI 이후 단계
- 배포 가능한 상태로 코드 준비
- 수동 승인 후 최종 배포

#### 2. Continuous Deployment

- Delivery의 확장된 형태
- 완전 자동화된 배포 파이프라인
- 인적 개입 없는 프로덕션 배포

## 예시

```python
# ============================================
# 1️⃣ 'Notifier'라는 설계도(인터페이스)를 만들어요
# ============================================
# ABC는 "추상 클래스"라는 뜻이에요.
# Notifier는 알림을 보내는 역할을 가진 친구들이 따라야 하는 규칙을 정의해요.
# send_notification이라는 기능을 꼭 만들어야 한다고 약속만 시킵니다.
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def send_notification(self, message: str) -> None:
        pass
    # 'pass'는 아직 아무것도 안 한다는 뜻
    # 하지만 이 함수는 나중에 무조건 구현해야 해요.

# ============================================
# 2️⃣ 이메일 알림을 보내는 친구 만들기
# ============================================
# Notifier 규칙을 그대로 따라가는 EmailNotifier 만들기
# send_notification 기능을 실제로 구현
class EmailNotifier(Notifier):
    def send_notification(self, message: str) -> None:
        print(f"Sending email: {message}")
        # 이메일을 보내는 흉내만 냈어요, 실제로 보내는 건 아님

# ============================================
# 3️⃣ SMS 알림을 보내는 친구 만들기
# ============================================
# Notifier 규칙을 그대로 따라가는 SMSNotifier 만들기
class SMSNotifier(Notifier):
    def send_notification(self, message: str) -> None:
        print(f"Sending SMS: {message}")
        # SMS 보내는 흉내만 냈어요

# ============================================
# 4️⃣ 실제로 알림을 관리해주는 서비스 만들기
# ============================================
# NotificationService는 누가 알림을 보낼지(notifier)를 알고 있음
# "이거 보내!"라고 요청하면 notifier에게 일을 맡김
class NotificationService:
    def __init__(self, notifier: Notifier):
        self.notifier = notifier
        # notifier는 EmailNotifier나 SMSNotifier 중 하나

    def notify(self, message: str) -> None:
        # 실제로 알림을 보내는 건 notifier가 담당
        self.notifier.send_notification(message)

# ============================================
# 5️⃣ 사용 예시
# ============================================
# 이메일 알림을 보낼 준비
email_notifier = EmailNotifier()
# NotificationService에 이메일 알림 담당을 맡김
email_service = NotificationService(email_notifier)
# 실제 알림 보내기
email_service.notify("Hello via email")
# 결과: "Sending email: Hello via email" 출력
```

위의 예에서, 의존성 주입으로 `NotificationService`는 `Notifier` 인터페이스에만 의존합니다. 이를 통해
`EmailNotifier`나 `SMSNotifier` 같은 구체적인 구현체에 대한 의존성을 피할 수 있습니다. 새로운 알림
방법을 추가할 때도 `Notifier` 인터페이스만 구현하면 되므로, 기존 코드를 수정할 필요가 없습니다. 이는
클린 아키텍처의 핵심 원칙인 의존성 규칙을 잘 따르는 예시입니다.

Python에서는 클래스 계층 구조를 사용하지 않고도 같은 클린 아키텍처 원칙을 구현할 수 있으며, 대신
Python의 **덕 타이핑(duck typing)**을 활용할 수 있습니다.

즉, 객체가 어떤 클래스에서 왔는지보다, 필요한 기능(메서드)이 있는지가 중요하다는 거예요.

- If it walks like a duck and quacks like a duck, it’s a duck.

```python
class EmailNotifier:
    def send_notification(self, message: str) -> None:
        print(f"Sending email: {message}")

class SMSNotifier:
    def send_notification(self, message: str) -> None:
        print(f"Sending SMS: {message}")

def notify_user(notifier: Notifier, message: str) -> None:
    notifier.send_notification(message)

email_notifier = EmailNotifier()
sms_notifier = SMSNotifier()

notify_user(email_notifier, "Hello via email")
notify_user(sms_notifier, "Hello via SMS")
```

파이썬은 동적 타이핑 언어이기 때문에, 인터페이스를 명시적으로 정의하지 않아도 됩니다. 대신,
`send_notification` 메서드가 있는 객체라면 어떤 객체든 `notify_user` 함수에 전달할 수 있습니다. 이는
파이썬의 유연성을 활용한 클린 아키텍처 구현 방법입니다.

```python
from typing import Protocol

class Notifier(Protocol):
    def send_notification(self, message: str) -> None:
        ...

class EmailNotifier:
    def send_notification(self, message: str) -> None:
        print(f"Email: {message}")

class SMSNotifier:
    def send_notification(self, message: str) -> None:
        print(f"SMS: {message}")

def notify_user(notifier: Notifier, message: str):
    notifier.send_notification(message)

# ✅ 타입 검사기(MyPy 등)는 이걸 문제없이 통과시킵니다.
notify_user(EmailNotifier(), "Hello!")
notify_user(SMSNotifier(), "Hi!")
```
