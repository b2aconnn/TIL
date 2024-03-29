# 정의

마틴 파울러의 [NoSQL : 빅데이터 세상으로 떠나는 간결한 안내서] 에서는 NoSQL이 아래의 조건을 만족하는 데이터 저장소라고 기술되어 있다.

- 대용량 웹 서비스를 위하여 만들어진 데이터 저장소
- 관계형 데이터 모델을 지양하며 대량의 분산된 데이터를 저장하고 조회하는데 특화된 저장소
- 스키마 없이 사용 가능하거나 느슨한 스키마를 제공하는 저장소

💁‍♂️ **여기서는 'NoSQL이란 빅데이터를 처리하기 위한 분산 데이터 저장소의 통칭' 이라고 정의한다.**

<br>

**NoSQL의 일반적인 특징**

- NoSQL은 저마다 쓰기/읽기 성능 특화, 2차 인덱스 지원, 자동 샤딩 지원과 같은 고유한 특징을 가진다.

- 일반적으로 NoSQL은 관계형 데이터 베이스보다 쓰기와 읽기 성능이 월등히 빠르다.

<br>

# 탄생 배경

**글로벌 웹 서비스가 2000년대 중반부터 다수 등장하고, 글로벌 웹 서비스들은 기존의 관계형 데이터 베이스로 처리할 수 없을 정도로 데이터가 방대해졌다.**

- 대부분의 관계형 데이터 베이스에서 채택하고 있는 인덱스 처리 방법인 B-트리의 한계로 인해 데이터가 방대해질수록 읽기/쓰기 성능이 저하된다.
- 관계형 데이터 베이스는 중앙 집중식 데이터 관리 패턴에 해당하기 때문에 단일 하드웨어 성능에 따라 전체 시스템의 성능이 결정된다.

<br>

**구글, 아마존 등에서 논문과 자료를 통해 데이터를 실시간으로 분산 처리하는 개념을 공개하므로써, 이 개념을 토대로 실시간 분산 처리를 위한 오픈 소스 솔루션들이 개발이 되었고 이와 같은 노력의 결과물이 NoSQL이다.**

<br>

👉 **NoSQL 분산 환경에서 대량의 데이터를 빠르게 처리하기 위해 한두 가지 단점을 가진채로 개발이 되었다.**

- 관계형 데이터 베이스가 제공하는 쿼리와 트랜잭션 같은 편의 기능을 제공하지 않으며, 제공하는 데이터의 일관성 레벨도 NoSQL마다 다르다.
- 솔루션을 도입하기 위해선 해당 NoSQL의 설계 사상과 내부 구조를 파악해야 한다.

<br>

👉 **NoSQL 탄생과 더불어 일각에서는 전통적인 데이터 베이스 중심의 어플리케이션 개발에서 서비스 중심의 어플리케이션 개발이라는 패러다임의 전환이 일어났다.**

- 이러한 패러다임 전환은 데이터 베이스를 서비스 제공의 도구로 바라보게 되는 계기가 되었다.

<br>

👉 **NoSQL은 대량의 데이터를 빠르게 처리하기 위해서 다양한 방법을 사용한다.**

- 메모리에 데이터를 임시 저장하고 응답
- 동적인 스케일 아웃 지원
- 가용성을 위한 데이터 복제
- 스키마 없이 데이터 저장/조회가 가능해 실시간으로 시스템의 확장과 축소를 지원
- 시스템 정지 없이 저장소 소프트웨어 업그레이드 가능

<br>

# CAP 정리

**CAP 정리란, 이론 컴퓨터 과학 분야에서 분산 컴퓨터 시스템을 설명하는데 사용되는 이론이다.**

- '일관성, 가용성, 분할 허용성 모두를 동시에 지원하는 분산 컴퓨터 시스템은 없다'라고 정의되어 있다.
- NoSQL은 일관성, 가용성, 분할 허용성 가운데 두 가지 속성만을 지원하며 나머지 한 속성은 특정 조건에서만 만족한다.

CAP 정리를 이해하기 위해선 분산 시스템의 용어에 대한 이해가 우선되어야 한다.

<br>

## 분산 시스템

### **분산 시스템을 구성하는 개별 요소**

- 노드 : 분산 시스템을 구성하는 각각의 하드웨어 또는 소프트웨어
- 클러스터 : 동일한 기능을 수행하는 노드들의 모임

<img width="450" src="https://github.com/b2aconnn/TIL/assets/101120568/dc198a3a-bb42-44e6-a2df-7a8ec145e072"/>

<br>

### **일관성**

동시성 또는 동일성이라고도 하며 '다중 클라이언트에서 같은 시간에 조회하는 데이터는 항상 동일한 데이터임을 보증하는 것'을 의미

- 관계형 데이터 베이스에서는 기본적으로 일관성을 지원

- NoSQL에서는 빠른 분산 처리를 위해 일관성을 희생하기도 한다.

<br>

🙂 **최종 일관성(궁긍적인 일관성)**

- 시간이 지남에 따라서 최종적으로 일관성이 유지되는 일관성

<br>

**가용성과 분할 허용성을 지원하는 NoSQL의 느슨한 일관성 예시**

<img width="500" src="https://github.com/b2aconnn/TIL/assets/101120568/0dd429c1-d9df-43c9-b518-33febe272644"/>

1. 클라이언트1이 노드 A에 접속하여 key1에 300이라는 값을 저장

2. 노드 A에서 '데이터 변경 전파' 이벤트가 발생
3. 데이터 변경 전파 이벤트에 의해 노드 B에 key1과 값 300이 복제
4. 클라이언트1이 노드 A에 저장된 key1의 값을 500으로 변경
5. 클라이언트2가 노드 B에 접속해 key1의 값을 조회
   - 노드 B에는 아직 key1:500 값이 변경되지 않았기 때문에 결과가 300으로 조회

> '데이터 변경 전파 이벤트' 는 NoSQL의 용어가 아니라 임의로 사용한 용어이다.

<br>

**각 NoSQL은 분산 노드 간의 데이터 동기화를 위해 두 가지 방법을 사용**

- 데이터의 저장 결과를 클라이언트로 응답 하기 전 모든 노드에 데이터를 저장하는 동기식 방법
  - 장점 : 강한 데이터의 정합을 보장
  - 단점 : 느린 응답 시간
- 메모리나 임시 파일에 기록하고 클라이언트에 먼저 응답한 다음, 특정 이벤트 또는 프로세스를 사용하여 노드로 데이터를 동기화하는 비동기식 방법
  - 장점 : 클라이언트에게 빠른 응답
  - 단점 : 쓰기 노드에 장애가 발생했을 경우 데이터를 잃어버릴 가능성

<br>

🙂 **강한 일관성(엄밀한 일관성)**

- NoSQL에서 강한 일관성은 응답 시간(=성능)을 희생하여 지원

<br>

**카산드라의 일관성 레벨 설정**

- One : 하나의 노드로부터 읽기 또는 쓰기의 성공 응답을 받으면 클라이언트로 응답
- Quorum : '설정된 복제계수 / 2 + 1'개의 노드로부터 읽기 또는 쓰기 성공 응답을 받으면 클라이언트로 응답
- All : '설정된 복제계수'의 노드로부터 읽기 또는 쓰기의 성공 응답을 받으면 클라이언트로 응답

All 설정이 가장 강한 일관성을 유지한다.

<br>

💁‍♂️ 많은 NoSQL 솔루션은 읽기와 쓰기의 성능 향상을 위해 데이터를 메모리에 임시 저장한 다음 클라이언트로 응답하고, 백그라운드 스레드로 해당 데이터를 디스크에 저장한다.

- 장점
  - 빠른 응답 속도
  - 데이터 변경에 따른 적은 수정 비용
- 단점
  - 하드웨어 장애 발생 시 데이터 유실 발생 가능성

💡 데이터 유실 방지 방법

- 카산드라와 HBase에서는 메모리에 저장하기 전에 커밋로그 및 WAL 파일에 먼저 정보를 기록
- 레디스는 위와 유사한 기능인 AOF를 사용

<br>

### 가용성(내고장성)

모든 클라이언트의 읽기와 쓰기 요청에 대하여 항상 응답이 가능해야 함을 보증하는 것

- 클러스터 내에서 일부 노드가 망가지더라도 정상적인 서비스가 가능

<br>

**가용성을 위한 데이터 중복 저장 방법**

👉 마스터-슬레이브 복제 방법

- 관계형 데이터베이스 시스템에서 고가용성을 지원하기 위한 솔루션으로 사용되기도 함

👉 피어-투-피어 복제 방법

- 동일한 데이터를 다중 노드에 중복 저장하여 일부 노드가 고장 나도 데이터가 유실되지 않도록 하는 방법

<img width="250" src="https://github.com/b2aconnn/TIL/assets/101120568/f3653993-5953-40c4-8241-41d9800526f6"/>

노드 2와 노드4가 장애가 발생하더라도 데이터(2)를 노드 3에서 읽을 수 있기 때문에 데이터의 유실이 밝생하지 않는다.

<br>

**단일 고장점(SPOF)**

- 시스템을 구성하는 개별 요소 중에서 하나의 요소가 망가졌을 때 시스템 전체를 멈추게 만드는 요소
- 단일 고장점을 가진 NoSQL(HBase 등)은 자체적으로 가용성을 지원하지 못하기 때문에 이를 지원하기 위해 별도의 솔루션을 함께 사용할 수도 있다.

<br>

