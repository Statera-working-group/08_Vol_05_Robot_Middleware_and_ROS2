**Volume 05 Robot Middleware and ROS2**

# 02. DDS Middleware

## 02.01 DDS (Data Distribution Service) Standard Overview

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

데이터 분산 서비스(Data Distribution Service, DDS)는 분산 실시간 시스템(Distributed Real-Time System)의 데이터 중심 통신(Data-Centric Communication)을 위해 객체 관리 그룹(Object Management Group, OMG)이 제정한 표준이다. DDS는 애플리케이션(Application)이 중앙 집중식 브로커(Centralized Broker)를 통해 메시지를 교환하도록 요구하는 대신, 발행자(Publisher)와 구독자(Subscriber)가 논리적으로 공유되는 데이터 공간(Data Space)을 통해 통신할 수 있도록 한다. 이러한 구조는 로보틱스(Robotics), 자율 시스템(Autonomous System), 산업 자동화(Industrial Automation), 항공우주(Aerospace)와 같이 분산된 구성요소가 예측 가능한 방식으로 정보를 교환해야 하는 환경에 특히 적합하다.

DDS는 데이터 중심 발행-구독 모델(Data-Centric Publish-Subscribe Model)을 따른다. 데이터 작성자(DataWriter)는 토픽(Topic)에 연결된 형식화된 데이터 샘플(Typed Data Sample)을 발행하고, 데이터 판독자(DataReader)는 해당 토픽이 나타내는 정보를 구독한다. 따라서 애플리케이션은 개별 네트워크 연결(Network Connection)을 직접 관리하기보다 데이터와 통신 요구사항을 중심으로 동작한다. 발행자와 구독자를 모든 통신 경로를 재설계하지 않고 추가, 제거, 재시작하거나 다른 위치로 이동할 수 있어 모듈형(Modular)이며 동적으로 변화하는 분산 시스템을 지원한다.

DDS의 기본적인 추상화 개념 중 하나는 전역 데이터 공간(Global Data Space)으로, 통신에 참여하는 구성요소들이 공유하는 논리적 정보 환경(Logical Information Environment)을 의미한다. 전역 데이터 공간은 물리적인 데이터베이스(Database)나 중앙 서버(Central Server)가 아니다. 대신 참여 컴퓨터에 분산된 DDS 미들웨어(DDS Middleware)가 협력하여 통신 관계를 유지하고 필요한 데이터 샘플을 전달한다. 이러한 분산형 모델(Decentralized Model)은 중앙 인프라에 대한 의존성을 줄이고 로봇 컴퓨터, 센서, 제어기, 엣지 프로세서(Edge Processor), 모니터링 시스템 사이의 피어 투 피어 통신(Peer-to-Peer Communication)을 지원한다.

DDS는 여러 핵심 엔터티(Entity)를 통해 통신 구조를 구성한다. 도메인 참여자(DomainParticipant)는 DDS 도메인(Domain)에 참여하는 애플리케이션 또는 프로세스를 나타낸다. 발행자(Publisher)는 데이터 작성자(DataWriter)를 포함하며, 구독자(Subscriber)는 데이터 판독자(DataReader)를 포함한다. 토픽(Topic)은 정의된 데이터 타입(Data Type) 및 토픽 이름(Topic Name)을 논리적 데이터 모델과 연결한다. 데이터 작성자와 데이터 판독자는 토픽 정의, 데이터 타입, 도메인 및 통신 정책이 서로 호환될 때 통신할 수 있으며, 미들웨어는 이러한 통신 관계를 자동으로 설정한다.

DDS 도메인(DDS Domain)은 논리적인 통신 경계(Logical Communication Boundary)를 제공한다. 동일한 도메인에 속하는 도메인 참여자(DomainParticipant)는 서로 호환되는 통신 엔드포인트(Communication Endpoint)를 발견할 수 있지만, 서로 다른 도메인에 할당된 참여자는 일반적으로 격리된다. 이러한 메커니즘은 여러 로봇, 시험 환경, 시뮬레이션 시스템 또는 운영 시스템이 동일한 인프라를 공유하는 경우에 유용하다. 따라서 도메인 분리는 통신 범위를 제어하고 독립적인 분산 시스템 사이의 의도하지 않은 상호작용을 줄이는 중요한 아키텍처 도구가 될 수 있다.

자동 탐색(Automatic Discovery)은 DDS의 또 다른 주요 기능이다. 참여자와 엔드포인트는 모든 발행자와 구독자에 대해 주소를 수동으로 설정하는 대신 서로를 동적으로 탐색할 수 있다. 호환되는 데이터 작성자(DataWriter)와 데이터 판독자(DataReader)가 발견되면 미들웨어는 양측의 통신 특성을 평가하고 적절한 관계를 설정한다. 이 기능을 통해 분산 로봇 소프트웨어(Distributed Robotic Software)는 프로세스 위치, 노드 시작 순서, 네트워크 토폴로지(Network Topology), 시스템 구성의 변화에 보다 자연스럽게 대응할 수 있다.

일반적으로 QoS로 약칭되는 서비스 품질(Quality of Service)은 DDS를 단순한 발행-구독 메시징 시스템(Publish-Subscribe Messaging System)과 구별하는 핵심 요소이다. QoS 정책(QoS Policy)은 어떤 데이터를 교환할 것인지만 정의하는 것이 아니라 정보를 어떻게 전달하고 유지할 것인지를 규정한다. 신뢰성(Reliability), 지속성(Durability), 이력(History), 마감시간(Deadline), 수명(Lifespan), 활성 상태(Liveliness), 소유권(Ownership), 자원 제한(Resource Limits) 등의 정책을 통해 애플리케이션 요구사항에 맞게 통신 동작을 조정할 수 있다. 따라서 하나의 분산 시스템에서도 서로 다른 토픽에 서로 다른 전달 의미론(Delivery Semantics)을 적용할 수 있다.

예를 들어 높은 데이터 전송률을 갖는 카메라(Camera) 또는 라이다(LiDAR) 스트림은 낮은 지연시간(Low Latency)을 우선하면서 일부 샘플 손실을 허용할 수 있지만, 설정 명령(Configuration Command)이나 중요한 상태 전환(State Transition)은 신뢰성 있는 전달(Reliable Delivery)이 필요할 수 있다. 위치 추정(Localization Estimate)은 최신 샘플만 필요할 수 있는 반면 진단 정보(Diagnostic Information)는 일정한 이력 깊이(History Depth)를 요구할 수 있다. DDS는 개발자가 전송, 버퍼링(Buffering), 재시도(Retry), 상태 관리(State Management)를 각각 구현하지 않고도 이러한 요구사항을 QoS를 통해 표현할 수 있도록 한다.

DDS는 애플리케이션 수준의 데이터 모델(Application-Level Data Model)을 네트워크 전송(Network Transport)의 많은 세부사항으로부터 분리한다. 애플리케이션은 강하게 형식화된 데이터 구조(Strongly Typed Data Structure)를 사용하고, 미들웨어는 설정된 정책에 따라 탐색(Discovery), 직렬화(Serialization), 엔드포인트 매칭(Endpoint Matching), 전송(Transmission), 전달(Delivery)을 처리한다. 이러한 분리는 기본 미들웨어 구현과 네트워크 설정이 변경되더라도 분산 소프트웨어 구성요소가 논리적 인터페이스를 유지할 수 있기 때문에 이식성(Portability)을 향상시키며, 대규모 로봇 소프트웨어 아키텍처에서 인터페이스 우선 설계(Interface-First Design)를 촉진한다.

DDS 명세(DDS Specification)는 실시간 발행-구독 프로토콜(Real-Time Publish-Subscribe Protocol, RTPS)과 구분하여 이해해야 한다. DDS는 도메인(Domain), 참여자(Participant), 토픽(Topic), 데이터 작성자(DataWriter), 데이터 판독자(DataReader), QoS 동작과 같은 상위 수준 개념을 정의하는 반면, DDSI-RTPS는 DDS 구현 간 상호운용성(Interoperability)을 위한 와이어 프로토콜(Wire Protocol)을 정의한다. 실제 DDS 시스템에서 두 계층은 밀접하게 연결되어 있지만, 애플리케이션 프로그래밍 모델(Application Programming Model)과 네트워크 수준 프로토콜(Network-Level Protocol)은 서로 다른 아키텍처 문제를 해결한다.

DDS는 ROS 2가 기반 분산 통신 기능을 제공하기 위한 미들웨어 추상화(Middleware Abstraction)를 채택하고 있기 때문에 ROS 2에서 특히 중요하다. 노드(Node), 토픽(Topic), 발행자(Publisher), 구독(Subscription), 서비스(Service), 통신 QoS와 같은 ROS 2 개념은 ROS 중심의 응용 프로그래밍 인터페이스(Application Programming Interface, API)를 통해 제공되며, 하위 미들웨어 계층은 탐색과 데이터 전송을 수행한다. ROS 미들웨어(ROS Middleware, rmw) 추상화는 ROS 2가 하나의 DDS 공급업체에 종속되지 않고 여러 미들웨어 구현을 지원할 수 있도록 한다.

이러한 관계 때문에 DDS를 이해하면 ROS 2에서 관찰되는 여러 동작의 원인을 설명할 수 있다. 탐색 지연(Discovery Delay), 신뢰성 호환성(Reliability Compatibility), 이력 깊이(History Depth), 멀티캐스트 트래픽(Multicast Traffic), 참여자 설정(Participant Configuration), 통신 지연시간(Communication Latency), 네트워크 분할(Network Segmentation)과 같은 특성은 일반적인 ROS 2 애플리케이션 계층보다 아래에 있는 미들웨어 계층에서 발생하는 경우가 많다. ROS 2 API만으로도 기능적인 애플리케이션을 구축할 수 있지만, 실제 운영 로봇을 담당하는 엔지니어는 DDS 엔터티와 정책이 전체 분산 시스템에 미치는 영향을 이해하는 것이 중요하다.

로봇 시스템에서 DDS는 센서 획득(Sensor Acquisition)과 위치 추정(Localization)부터 인지(Perception), 내비게이션(Navigation), 조작(Manipulation), 진단(Diagnostics), 플릿 인터페이스(Fleet Interface)에 이르는 다양한 작업 부하(Workload)를 연결할 수 있다. 이러한 작업은 페이로드 크기(Payload Size), 업데이트 주기(Update Frequency), 지연시간 허용 범위(Latency Tolerance), 신뢰성 요구사항(Reliability Requirement), 계산 비용(Computational Cost)이 크게 다르다. 따라서 설정 가능한 QoS를 갖춘 단일 통신 메커니즘은 로봇 전체에서 일관된 발행-구독 프로그래밍 모델을 유지하면서 각 데이터 흐름(Data Flow)에 맞는 통신 의미론을 적용할 수 있다는 아키텍처적 장점을 제공한다.

DDS는 분산형 통신(Decentralized Communication)을 통해 확장성(Scalability)을 지원하지만, 확장성이 자동으로 보장되는 것은 아니다. 대규모 시스템에는 많은 참여자, 엔드포인트, 토픽, 고대역폭 센서(High-Bandwidth Sensor), 네트워크 인터페이스가 존재할 수 있다. 탐색 트래픽(Discovery Traffic), 직렬화 오버헤드(Serialization Overhead), 멀티캐스트 설정(Multicast Configuration), QoS 선택, 페이로드 크기, 발행 주기(Publication Frequency)는 성능에 큰 영향을 미칠 수 있다. 따라서 운영 환경의 DDS 아키텍처는 미들웨어 자체가 모든 조건에서 결정적 동작(Deterministic Behavior)을 보장한다고 가정하기보다 체계적인 네트워크 엔지니어링(Network Engineering)을 통해 설계해야 한다.

DDS의 실시간 기능(Real-Time Capability) 역시 신중하게 해석해야 한다. DDS는 예측 가능하고 설정 가능한 데이터 분산을 위한 메커니즘을 제공하지만, 종단 간 타이밍(End-to-End Timing)은 운영체제 스케줄링(OS Scheduling), 미들웨어 구현, 네트워크 하드웨어, 전송 설정, 직렬화, 애플리케이션 실행, 자원 경합(Resource Contention)을 포함한 전체 시스템에 의해 결정된다. 따라서 DDS는 실시간 통신 아키텍처의 중요한 구성요소이지만, 결정적인 로봇 제어(Deterministic Robot Control)를 구현하려면 전체 소프트웨어 및 하드웨어 스택에 대한 통합적인 설계가 필요하다.

DDS는 표준화된 보안 메커니즘(Security Mechanism)을 통해 안전한 분산 통신(Secure Distributed Communication)을 위한 기반도 제공한다. 인증(Authentication)을 통해 참여자의 신원을 확인할 수 있으며, 접근 제어(Access Control)를 통해 허용된 통신을 제한하고, 암호학적 보호(Cryptographic Protection)를 통해 기밀성(Confidentiality)과 무결성(Integrity)을 유지할 수 있다. 이러한 기능은 로봇 시스템이 하나의 격리된 컴퓨터를 넘어 분산 엣지 컴퓨터(Distributed Edge Computer), 무선 네트워크(Wireless Network), 다중 로봇 플릿(Multi-Robot Fleet), 산업 인프라 및 원격 관리 자율 플랫폼으로 확장될수록 더욱 중요해진다.

소프트웨어 아키텍처(Software Architecture)의 관점에서 DDS가 제공하는 가장 중요한 가치는 분산 구성요소를 공간(Space), 시간(Time), 구현(Implementation) 측면에서 서로 분리하는 디커플링(Decoupling)이다. 발행자는 모든 구독자를 구체적으로 알 필요가 없으며, 구독자는 자신의 요구사항에 따라 정보를 소비할 수 있다. 형식화된 토픽(Typed Topic), 동적 탐색(Dynamic Discovery), 설정 가능한 QoS, 분산형 통신, 상호운용 가능한 프로토콜(Interoperable Protocol)을 결합함으로써 DDS는 모듈형이며 확장 가능한 로봇 시스템을 구축하기 위한 강력한 미들웨어 기반을 제공한다.

따라서 ROS 2의 전체 아키텍처에서 DDS는 단순한 전송 라이브러리(Transport Library)가 아니라 애플리케이션 수준의 로봇 소프트웨어 아래에서 동작하는 분산 데이터 관리 및 통신 프레임워크(Distributed Data Management and Communication Framework)로 이해해야 한다. DDS의 핵심 개념은 이후 다루게 될 RTPS, 탐색(Discovery), QoS 정책 설계(QoS Policy Design), 미들웨어 구현 비교(Middleware Implementation Comparison), 멀티캐스트와 유니캐스트 설정(Multicast and Unicast Configuration), 대역폭 최적화(Bandwidth Optimization), DDS 보안(DDS Security), 다중 도메인 아키텍처(Multi-Domain Architecture), 네트워크 진단(Network Diagnostics)의 기반을 형성하며, 이들이 결합되어 실제 운영 ROS 2 시스템의 DDS 미들웨어 계층을 구성한다.

## 02.02 RTPS (Real-Time Publish-Subscribe) Protocol Deep Dive

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

실시간 발행-구독 프로토콜(Real-Time Publish-Subscribe Protocol, RTPS)은 데이터 분산 서비스(Data Distribution Service, DDS) 구현체들이 네트워크를 통해 데이터를 교환하기 위해 일반적으로 사용하는 상호운용성 와이어 프로토콜(Interoperability Wire Protocol)이다. 객체 관리 그룹(Object Management Group, OMG)이 DDSI-RTPS로 표준화했으며, 분산 엔드포인트(Distributed Endpoint)가 서로를 발견하고 통신 엔터티(Communication Entity)를 표현하며 프로토콜 정보를 직렬화하고 사용자 데이터를 전송하는 방법을 정의한다. DDS가 데이터 중심 애플리케이션 모델(Data-Centric Application Model)을 제공한다면, RTPS는 서로 다른 구현체 간 통신을 가능하게 하는 네트워크 수준 동작(Network-Level Behavior)의 상당 부분을 규정한다.

RTPS는 DDS의 특징인 분산형 철학(Decentralized Philosophy)을 따른다. 통신은 모든 샘플을 수신한 후 다시 분배하는 중앙 메시지 브로커(Central Message Broker)에 근본적으로 의존하지 않는다. 대신 RTPS 참여자(Participant)는 UDP/IP와 같은 네트워크 전송(Network Transport)을 이용해 직접 통신한다. 이러한 피어 지향 아키텍처(Peer-Oriented Architecture)는 센서, 제어기, 인지 컴퓨터, 내비게이션 프로세스, 모니터링 시스템이 필수적인 중앙 통신 구성요소나 단일 통신 병목지점 없이 정보를 교환할 수 있기 때문에 로보틱스(Robotics)에 유용하다.

RTPS 수준에서 통신은 참여자(Participant), 작성자(Writer), 판독자(Reader)를 사용하여 모델링된다. RTPS 참여자(RTPS Participant)는 도메인 내에서 통신하는 프로세스 또는 미들웨어 인스턴스(Middleware Instance)를 나타낸다. 작성자는 데이터를 생성하고 판독자는 데이터를 소비한다. 이러한 엔터티는 개념적으로 상위 수준 DDS 엔터티와 대응하지만, RTPS는 전송 계층(Transport Layer)에 더 가까운 수준에서 동작한다. 각 엔드포인트는 프로토콜에서 정의한 식별자를 사용하므로 분산된 참여자들은 네트워크 전체에서 데이터 소스, 목적지 및 통신 관계를 구분할 수 있다.

RTPS의 기본 식별자 중 하나는 전역 고유 식별자(Globally Unique Identifier, GUID)이다. GUID는 일반적으로 참여자를 식별하는 GUID 접두부(GUID Prefix)와 해당 참여자 내부의 특정 엔터티를 식별하는 엔터티 ID(EntityId)를 결합한다. 따라서 작성자(Writer), 판독자(Reader), 내장 탐색 엔드포인트(Built-In Discovery Endpoint)는 각각 서로 다른 식별자를 갖는다. 이러한 주소 지정 방식(Addressing Scheme)은 RTPS 구현체가 IP 주소와 포트에만 의존하지 않고 통신 엔드포인트를 추적할 수 있도록 하며, 하나의 프로세스나 호스트 내부에 여러 DDS 엔터티가 존재할 수 있기 때문에 중요하다.

RTPS 통신은 메시지 헤더(Message Header)와 하나 이상의 서브메시지(Submessage)로 구성된 프로토콜 메시지를 통해 수행된다. 헤더는 RTPS 프로토콜과 송신 참여자(Originating Participant) 등의 정보를 식별하며, 서브메시지는 개별 프로토콜 동작을 기술한다. 주요 서브메시지 유형에는 샘플을 운반하는 데이터(DATA), 사용 가능한 시퀀스 범위(Sequence Range)를 알리는 하트비트(HEARTBEAT), 수신되었거나 누락된 샘플을 보고하는 확인-부정확인(ACKNACK), 작성자가 더 이상 판독자에게 전달할 필요가 없다고 판단한 시퀀스 번호를 나타내는 갭(GAP)이 있다.

데이터 서브메시지(DATA Submessage)는 직렬화된 사용자 데이터 또는 샘플과 연관된 정보를 운반하기 때문에 일반적인 정보 교환의 중심적인 역할을 한다. 신뢰성 작성자(Reliable Writer)가 생성하는 각각의 변경 데이터에는 시퀀스 번호(Sequence Number)가 연결되며, 이를 통해 판독자는 샘플이 예상된 순서대로 도착했는지 또는 누락된 샘플이 있는지를 판단할 수 있다. 이러한 시퀀스 기반 메커니즘(Sequence-Based Mechanism)은 모든 통신 흐름에 TCP와 같은 연결 지향 전송(Connection-Oriented Transport)을 사용하지 않고도 신뢰성 동작을 구현하기 위한 기반을 제공한다.

신뢰성 RTPS 통신(Reliable RTPS Communication)은 하트비트(HEARTBEAT)와 확인-부정확인(ACKNACK) 메시지 사이의 상호작용에 크게 의존한다. 작성자는 사용 가능한 데이터와 관련된 시퀀스 번호 범위를 나타내는 하트비트 정보를 주기적으로 전송한다. 판독자는 이 정보를 자신이 수신한 샘플과 비교하고 누락된 시퀀스 번호를 지정한 ACKNACK를 반환할 수 있다. 이후 작성자는 필요한 샘플을 재전송한다. 따라서 신뢰성(Reliability)은 RTPS 프로토콜 수준에서 구현되며, UDP를 사용하면서도 DDS 지향 신뢰성 의미론(DDS-Oriented Reliability Semantics)을 유지할 수 있다.

최선형 통신(Best-Effort Communication)은 이와 다르게 동작한다. 최선형 판독자(Best-Effort Reader)는 일반적으로 누락된 데이터의 재전송을 요청하지 않고 사용 가능한 샘플을 수용한다. 이는 프로토콜 트래픽을 줄이고 일부 샘플 손실을 허용할 수 있는 환경에서 지연시간을 낮출 수 있다. 이러한 방식은 새로운 샘플이 이전 샘플을 빠르게 대체하는 고주기 로봇 데이터 스트림(High-Frequency Robotic Stream)에 유용하다. 반면 명령, 설정 정보, 중요한 이벤트 또는 필수 상태 업데이트의 손실을 허용할 수 없는 경우에는 신뢰성 전달(Reliable Delivery)이 더 적합하다.

RTPS 탐색(RTPS Discovery)은 통신 관계가 동적으로 형성될 수 있도록 한다. 단순 탐색 프로토콜(Simple Discovery Protocol)은 일반적으로 단순 참여자 탐색 프로토콜(Simple Participant Discovery Protocol, SPDP)과 단순 엔드포인트 탐색 프로토콜(Simple Endpoint Discovery Protocol, SEDP)로 구분된다. SPDP는 DDS 참여자가 다른 참여자를 발견하고 기본적인 도달 가능성 정보(Reachability Information)를 교환하도록 한다. 참여자가 확인되면 SEDP가 작성자와 판독자에 관한 정보를 교환하여 토픽, 데이터 타입 및 관련 통신 정책에 따라 호환 가능한 엔드포인트를 식별하고 매칭할 수 있도록 한다.

탐색(Discovery)은 개별 주소를 미리 알지 못해도 하나의 알림을 여러 잠재적 참여자에게 전달할 수 있기 때문에 멀티캐스트(Multicast)를 자주 사용한다. 탐색이 완료된 후 사용자 데이터 트래픽(User-Data Traffic)은 미들웨어 구현, 설정, QoS, 네트워크 토폴로지 및 배포 요구사항에 따라 멀티캐스트 또는 유니캐스트(Unicast)를 사용할 수 있다. 이러한 차이는 멀티캐스트가 동적 탐색을 단순화하는 반면 잘못 구성된 스위치, Wi-Fi 네트워크, VPN, 컨테이너(Container), 분할된 산업 네트워크(Segmented Industrial Network)에서는 제대로 동작하지 않을 수 있기 때문에 로봇 네트워크에서 중요하다.

RTPS는 통신 엔터티에 도달할 수 있는 위치를 표현하기 위해 로케이터(Locator)를 사용한다. 로케이터는 IP 주소와 포트 같은 전송 관련 주소 정보(Transport-Related Addressing Information)를 나타낸다. 참여자는 탐색 과정에서 유니캐스트 및 멀티캐스트 로케이터를 알릴 수 있으며, 원격 엔드포인트는 이를 이용해 적절한 통신 경로를 결정한다. 따라서 ROS 2 수준에서 나타나는 DDS 통신 문제는 실제로 로케이터 선택, 라우팅(Routing), 방화벽 규칙(Firewall Rule), 다중 네트워크 인터페이스, 멀티캐스트 필터링 또는 잘못된 미들웨어 전송 설정에서 발생할 수 있다.

직렬화(Serialization)는 RTPS 통신의 또 다른 핵심 요소이다. DDS 데이터 구조는 서로 다른 장비와 구현체 사이에서 전송될 수 있는 표현 형식으로 변환되어야 한다. DDS 시스템은 일반적으로 공통 데이터 표현(Common Data Representation, CDR) 계열과 관련 확장형 직렬화 메커니즘(Extensible Serialization Mechanism)을 사용한다. 올바른 직렬화를 통해 형식화된 정보(Typed Information)는 데이터 모델, 바이트 순서(Byte Ordering), 필드 표현(Field Representation), 통신하는 DDS 구현체가 요구하는 상호운용성 조건을 유지하면서 프로세스 및 컴퓨터 경계를 넘어 전달될 수 있다.

이미지, 포인트 클라우드(Point Cloud), 점유 정보(Occupancy Information), 지도 또는 AI 관련 텐서(Tensor)와 같은 대규모 로봇 페이로드(Large Robotic Payload)를 전송할 때는 단편화(Fragmentation)가 중요해진다. RTPS는 데이터 단편(DATA_FRAG)과 같은 메커니즘을 제공하여 대형 직렬화 샘플을 더 작은 조각으로 나누어 전송하고 수신 측에서 다시 재구성할 수 있도록 한다. 따라서 대용량 메시지 성능은 명목상 네트워크 대역폭뿐 아니라 단편 크기(Fragment Size), 패킷 손실(Packet Loss), 재전송 동작, 소켓 버퍼(Socket Buffer), 미들웨어 설정 및 기본 네트워크 최대 전송 단위(Maximum Transmission Unit, MTU)의 영향을 받는다.

RTPS 트래픽에는 애플리케이션 페이로드(Application Payload) 외에도 프로토콜 메타데이터(Protocol Metadata)가 포함된다. 탐색 알림, 하트비트 메시지, ACKNACK 응답, 엔드포인트 정보, 재전송 및 단편화 메타데이터 역시 네트워크 용량을 사용한다. 소규모 로봇에서는 이러한 오버헤드(Overhead)가 크지 않을 수 있지만, 많은 토픽, 참여자, 카메라, 라이다 및 분산 컴퓨터를 포함하는 시스템에서는 상당한 전체 트래픽이 발생할 수 있다. 따라서 DDS 대역폭 요구사항을 계산할 때는 페이로드 크기와 발행 주기뿐 아니라 프로토콜 오버헤드도 함께 고려해야 한다.

RTPS 프로토콜 자체가 결정적인 종단 간 실시간 성능(Deterministic End-to-End Real-Time Performance)을 보장하는 것은 아니다. RTPS는 실시간 분산 통신에 적합한 메커니즘을 제공하지만 실제 지연시간(Latency)과 지터(Jitter)는 미들웨어 구현, 운영체제 스케줄링, 직렬화 비용, 소켓 동작, 네트워크 인터페이스, 스위치, 무선 통신 상태, CPU 부하 및 애플리케이션 실행에 따라 달라진다. 따라서 실시간 ROS 2 아키텍처에서는 RTPS 설정을 PREEMPT_RT, 스레드 우선순위(Thread Priority), CPU 친화성(CPU Affinity), 메모리 관리, 네트워크 엔지니어링 및 애플리케이션 수준 타이밍 요구사항과 함께 조정해야 한다.

RTPS 상호운용성(RTPS Interoperability)은 이 프로토콜의 가장 중요한 아키텍처 목적 중 하나이다. 서로 다른 DDS 구현체의 제품은 DDSI-RTPS가 공통된 네트워크 수준 표현과 동작을 정의하기 때문에 잠재적으로 서로 통신할 수 있다. 그러나 실제 상호운용을 위해서는 호환되는 데이터 타입, 토픽, QoS 정책, 탐색 설정, 전송 설정, 보안 설정 및 지원되는 명세 기능이 필요하다. 따라서 동일한 프로토콜 준수는 상호운용성의 기반을 제공하지만, 독립적으로 설정된 모든 DDS 시스템이 자동으로 정상 통신한다는 것을 보장하지는 않는다.

ROS 2에서 애플리케이션 개발자는 일반적으로 RTPS 패킷(RTPS Packet)을 직접 조작하지 않는다. ROS 2 노드는 발행자, 구독, 서비스, 액션(Action), QoS 설정을 통해 상호작용하며, ROS 미들웨어 추상화(ROS Middleware Abstraction, rmw)는 이러한 API를 Fast DDS, Cyclone DDS, Connext DDS와 같은 미들웨어 구현체에 연결한다. 이러한 계층 아래에서 RTPS는 엔드포인트 탐색과 네트워크 데이터 교환을 수행할 수 있으며, 엔지니어가 지연시간, 탐색, 신뢰성 또는 네트워크 연결 문제를 조사할 때 패킷 수준의 동작이 관찰될 수 있다.

이러한 계층적 관계(Layered Relationship)를 이해하는 것은 진단(Diagnostics)에 매우 중요하다. ROS 2 토픽은 단순한 발행자-구독자 연결처럼 보일 수 있지만, 네트워크 패킷 캡처(Network Packet Capture)를 분석하면 SPDP 탐색 알림, SEDP 엔드포인트 교환, DATA 패킷, HEARTBEAT 메시지, ACKNACK 응답, 단편화 및 재전송을 확인할 수 있다. 따라서 와이어샤크(Wireshark)와 같은 도구를 사용하면 ROS 2 API에서 숨겨진 통신 동작을 확인하고 문제가 애플리케이션, QoS 설정, DDS 미들웨어, RTPS 프로토콜 동작 또는 물리적 네트워크 중 어느 계층에서 발생하는지 판단하는 데 도움을 받을 수 있다.

실제 운영 로보틱스(Production Robotics)에서 RTPS는 추상적인 DDS 데이터 중심 모델을 실제 분산 통신으로 변환하는 네트워크 수준 메커니즘(Network-Level Mechanism)으로 이해해야 한다. 참여자와 GUID는 통신 개체의 식별을 제공하고, SPDP와 SEDP는 탐색을 수행하며, DATA와 DATA_FRAG는 샘플을 전송하고, HEARTBEAT와 ACKNACK는 신뢰성을 지원하며, 로케이터는 네트워크 도달 가능성을 설정한다. 이러한 메커니즘이 결합되어 DDS 기반 ROS 2 통신의 프로토콜 기반을 형성하며, 이후 탐색, QoS, 멀티캐스트, 대역폭 최적화, 보안 및 네트워크 진단을 심층적으로 분석하기 위한 기초를 제공한다.

## 02.03 DDS QoS Policy Full List and Robot Application Guide [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

서비스 품질(Quality of Service, QoS)은 모든 토픽(Topic)에 동일한 통신 의미론(Communication Semantics)을 적용하는 대신, DDS가 개별 데이터 흐름(Data Flow)의 동작 방식을 정의할 수 있도록 하는 메커니즘이다. 로봇 시스템에서는 카메라 영상, 라이다(LiDAR) 스캔, 위치 추정 상태(Localization State), 제어 명령(Control Command), 진단 정보(Diagnostics), 지도(Map), 설정 매개변수(Configuration Parameter)가 서로 다른 시간 및 신뢰성 요구사항을 가지므로 이러한 기능이 필수적이다. QoS 정책(QoS Policy)을 사용하면 이러한 요구사항을 통신 아키텍처의 명시적인 속성으로 정의할 수 있다.

신뢰성(RELIABILITY) 정책은 데이터 전달이 최선형(BEST_EFFORT) 또는 신뢰성형(RELIABLE) 의미론을 따를 것인지 결정한다. BEST_EFFORT는 복구 오버헤드(Recovery Overhead)를 최소화하므로 새로운 샘플이 이전 정보를 빠르게 대체하는 고주기 카메라, 라이다 또는 텔레메트리(Telemetry) 스트림에 적합하다. RELIABLE 통신은 누락된 샘플을 감지하고 재전송(Retransmission)을 지원하므로 명령, 설정 업데이트, 중요한 상태 전환(State Transition) 및 손실될 경우 시스템 동작에 영향을 줄 수 있는 정보에 더욱 적합하다.

이력(HISTORY) 정책은 엔드포인트(Endpoint)가 보관할 샘플의 수를 지정한다. 최근 유지(KEEP_LAST)는 설정된 수의 최신 샘플만 유지하는 반면, 전체 유지(KEEP_ALL)는 자원 제약(Resource Constraint)이 허용하는 범위에서 관련된 모든 샘플을 유지하려고 한다. 깊이(DEPTH)는 KEEP_LAST의 큐 깊이(Queue Depth)를 정의한다. 로봇 센서 파이프라인은 오래된 인지 데이터의 가치가 빠르게 감소하므로 일반적으로 얕은 이력을 사용하지만, 이벤트 처리, 명령 큐 또는 비동기 소비자(Asynchronous Consumer)는 일시적인 처리 지연을 견디기 위해 더 깊은 이력이 필요할 수 있다.

지속성(DURABILITY)은 데이터가 발행된 이후에 참여하는 데이터 판독자(DataReader)에게 이전에 발행된 정보가 제공될 것인지를 결정한다. 휘발성(VOLATILE)은 엔드포인트가 실제로 통신하는 동안 생성된 데이터만 제공하는 반면, 일시적 로컬(TRANSIENT_LOCAL)은 작성자(Writer)가 이후에 참여하는 판독자를 위해 샘플을 보관할 수 있도록 한다. DDS 구현과 명세에는 더욱 지속적인 지속성 모델도 존재한다. 로봇 설정, 지도, 보정 상태(Calibration State), 느리게 변화하는 시스템 정보는 보존된 데이터의 이점을 얻을 수 있지만 지속적으로 갱신되는 센서 스트림은 일반적으로 휘발성 동작을 사용한다.

마감시간(DEADLINE)은 연속적인 데이터 업데이트 사이에 예상되는 최대 시간 간격을 표현한다. 설정된 시간 내에 데이터가 생성되거나 수신되지 않으면 DDS는 마감시간 위반(Deadline Violation)을 보고할 수 있다. 따라서 DEADLINE은 주기적인 로봇 정보에 대한 통신 수준 상태 지표(Communication-Level Health Indicator)로 활용할 수 있다. 휠 오도메트리(Wheel Odometry), 위치 추정 상태, 관성 측정 장치(IMU) 업데이트, 액추에이터 피드백(Actuator Feedback), 안전 관련 상태에 마감시간 감시를 적용하면 중단된 발행자, 과부하된 프로세스, 통신 장애 또는 비정상적으로 느린 처리 파이프라인을 탐지할 수 있다.

수명(LIFESPAN)은 발행된 샘플이 유효한 상태로 유지되는 시간을 정의한다. 설정된 수명이 만료되면 오래된 데이터를 현재 데이터인 것처럼 전달하지 않고 폐기할 수 있다. 많은 관측 정보가 시간에 민감하기 때문에 이러한 기능은 로보틱스에서 특히 중요하다. 장애물 탐지(Obstacle Detection), 속도 명령(Velocity Command), 인지 결과(Perception Output), 로컬 경로 계획(Local Planning) 정보는 짧은 시간이 지나면 위험하거나 의미가 없어질 수 있지만, 정적 설정 및 의미 지도(Semantic Map) 정보는 훨씬 오랫동안 유효할 수 있다.

활성 상태(LIVELINESS)는 DDS가 발행 엔터티(Publishing Entity)가 계속 정상적으로 동작하는지를 판단하는 방법을 정의한다. 자동(AUTOMATIC) 활성 상태에서는 미들웨어가 활성 상태를 자동으로 유지하는 반면, 수동(MANUAL) 모드에서는 서로 다른 범위에서 명시적인 활성 상태 확인(Assertion)이 필요하다. 임대 기간(LEASE_DURATION)과 결합하면 일반적인 애플리케이션 데이터가 존재하지 않는 상황에서도 동작을 중단한 엔드포인트를 감지할 수 있다. 로봇 감독 시스템(Robot Supervisor)은 이를 이용해 장애가 발생한 제어기, 센서 프로세스, 미션 구성요소 또는 분산 컴퓨팅 노드를 탐지할 수 있다.

소유권(OWNERSHIP) 정책은 여러 작성자(Writer)가 호환되는 데이터 인스턴스를 발행할 때 어느 작성자가 권위 있는 데이터(Authoritative Data)를 제공할 것인지를 제어한다. 공유(SHARED) 소유권에서는 여러 작성자가 데이터를 제공할 수 있지만, 독점(EXCLUSIVE) 소유권에서는 소유권 강도(Ownership Strength) 및 관련 규칙에 따라 하나의 작성자가 실질적인 소유자가 될 수 있다. 이러한 메커니즘은 주 작성자와 백업 작성자가 존재하는 이중화 로봇 아키텍처(Redundant Robot Architecture)를 지원할 수 있지만, 장애 전환(Failover) 동작은 애플리케이션 상태, 안전 로직 및 시간 요구사항과 함께 신중하게 설계해야 한다.

목적지 순서(DESTINATION_ORDER)는 여러 작성자로부터 전달되는 샘플의 순서를 결정하며, 지원되는 설정에 따라 일반적으로 수신 순서(Reception Order) 또는 소스 타임스탬프(Source Timestamp)를 기준으로 한다. 이 정책은 여러 분산 생산자가 서로 관련된 정보를 생성하고 데이터 순서가 해석에 영향을 미치는 경우 중요하다. 그러나 DDS의 순서 정책만으로 부정확한 시계를 보정하거나 분산 시스템 전체에서 완전한 인과적 순서(Causal Ordering)를 확립할 수 없으므로 로봇 시스템에서는 시간 동기화, 센서 시계, 네트워크 지연 및 애플리케이션 수준 시퀀스 정보도 함께 고려해야 한다.

자원 제한(RESOURCE_LIMITS)은 저장 가능한 최대 샘플 수, 인스턴스 수 또는 인스턴스별 샘플 수와 같은 미들웨어 자원을 제한한다. 높은 신뢰성, 깊은 이력 또는 대형 페이로드는 상당한 메모리를 소비할 수 있기 때문에 이 정책은 중요하다. 여러 카메라와 라이다를 탑재한 로봇에서는 자원 제한이 잘못 설정되면 미들웨어 버퍼(Middleware Buffer)가 빠르게 고갈될 수 있다. 따라서 운영 시스템에서는 QoS 이력 설정을 예상 페이로드 크기, 발행 주기, 구독자 지연 및 사용 가능한 메모리와 함께 조정해야 한다.

시간 기반 필터(TIME_BASED_FILTER)는 작성자가 높은 주기로 데이터를 발행하더라도 데이터 판독자가 변경 데이터를 수신하는 빈도를 제한할 수 있도록 한다. 따라서 하나의 고주기 데이터 소스가 서로 다른 시간적 요구사항을 가진 소비자에게 데이터를 제공하면서 모든 구독자가 모든 샘플을 처리할 필요가 없도록 할 수 있다. 예를 들어 모니터링 인터페이스는 초당 몇 번의 업데이트만 필요하지만 기본 위치 추정 또는 텔레메트리 파이프라인은 훨씬 빠르게 동작할 수 있으므로, 느린 소비자의 불필요한 처리 및 통신 부하를 줄일 수 있다.

지연시간 예산(LATENCY_BUDGET)은 데이터 전달에 허용되는 목표 지연시간을 표현하고 미들웨어 구현체가 전송 최적화를 수행하기 위한 정보를 제공할 수 있다. 전송 우선순위(TRANSPORT_PRIORITY)는 지원되는 환경에서 상대적인 전송 중요도를 표현할 수 있다. 그러나 이러한 정책을 네트워크 지연시간이나 결정적 스케줄링(Deterministic Scheduling)에 대한 절대적인 보장으로 해석해서는 안 된다. 엄격한 지연시간 요구사항이 존재하는 경우 로봇 아키텍트는 운영체제 우선순위, 네트워크 설정, 미들웨어 동작, CPU 할당 및 애플리케이션 타이밍 분석을 함께 고려해야 한다.

파티션(PARTITION) 정책은 발행자(Publisher)와 구독자(Subscriber) 컨텍스트 내부에서 통신을 논리적으로 그룹화하는 기능을 제공한다. 반드시 서로 다른 DDS 도메인을 생성하지 않더라도 통신 흐름을 분리할 수 있다. 다중 로봇 또는 서브시스템 아키텍처에서는 파티션을 이용해 로봇, 기능, 시험 환경 또는 운영 그룹에 따라 트래픽을 구성할 수 있다. 그러나 파티션을 주요 격리 메커니즘으로 사용하기 전에 ROS 2 네임스페이스(Namespace) 설계, DDS 도메인 분리, 네트워크 세분화(Network Segmentation), 보안 정책 및 미들웨어별 동작도 함께 고려해야 한다.

표현(PRESENTATION)은 일관된 변경 집합(Coherent Set)이나 순서가 지정된 변경 집합을 구독 애플리케이션에 어떻게 제공할 것인지를 제어한다. 그룹 데이터(GROUP_DATA), 토픽 데이터(TOPIC_DATA), 사용자 데이터(USER_DATA)는 DDS 엔터티와 연관된 추가 메타데이터를 제공하며 미들웨어 수준 식별 또는 애플리케이션별 컨텍스트를 지원할 수 있다. 이러한 정책은 일반적인 ROS 2 프로그래밍에서 RELIABILITY, DURABILITY, HISTORY, DEADLINE보다 덜 노출되지만, 광범위한 DDS QoS 모델의 일부이며 특수한 네이티브 DDS(Native DDS) 배포 환경에서 중요할 수 있다.

DDS는 엔드포인트와 엔터티 동작에 관련된 엔터티 팩토리(ENTITY_FACTORY), 작성자 데이터 수명주기(WRITER_DATA_LIFECYCLE), 판독자 데이터 수명주기(READER_DATA_LIFECYCLE), 지속성 서비스(DURABILITY_SERVICE) 등의 정책도 정의한다. 수명주기 정책은 작성자와 판독자의 상태가 변경될 때 데이터 인스턴스가 유지되거나 제거되는 방식에 영향을 주며, 지속성 서비스 설정은 해당 지속성 모드의 과거 데이터 저장 동작을 제어한다. 이러한 기능은 데이터 생성, 폐기, 소유권 변경 및 과거 정보 자체가 의미를 가지는 상태 지향 분산 애플리케이션(State-Oriented Distributed Application)에서 중요하다.

작성자(Writer)와 판독자(Reader) 사이의 QoS 호환성(QoS Compatibility)은 성공적인 DDS 통신을 위한 기본 조건이다. DDS는 일반적으로 작성자가 제공하는 QoS(Offered QoS)와 판독자가 요청하는 QoS(Requested QoS)를 비교한다. 호환성 검사에 참여하는 정책들이 요청된 관계를 만족하지 못하면 엔드포인트가 서로를 발견하더라도 데이터 교환을 위한 매칭이 실패할 수 있다. ROS 2에서는 토픽이 존재하는 것처럼 보이지만 메시지가 수신되지 않는 현상으로 나타날 수 있으므로 QoS 호환성은 통신 문제를 진단할 때 가장 먼저 확인해야 할 항목 중 하나이다.

센서 데이터 QoS는 일반적으로 모든 과거 샘플의 무조건적인 전달보다 데이터 최신성(Freshness)과 제한된 자원 소비(Bounded Resource Consumption)를 우선해야 한다. 카메라 프레임, 포인트 클라우드, 레이더 탐지(Radar Detection), 빠르게 변화하는 텔레메트리는 흔히 BEST_EFFORT, VOLATILE 지속성 및 얕은 KEEP_LAST 이력을 사용한다. 정확한 설정은 네트워크와 애플리케이션에 따라 달라지며 일부 센서 경로에서는 신뢰성 전달이 유용할 수 있지만, 오래된 고대역폭 샘플을 재전송하면 로봇 동작을 개선하기보다 네트워크 혼잡과 지연시간을 증가시킬 수 있다.

제어 및 상태 통신(Control and State Communication)은 이와 다른 균형을 요구한다. 중요한 명령과 불연속 상태 전환(Discrete State Transition)은 일반적으로 RELIABLE 전달이 적합하지만, 오래된 명령이 누적되지 않도록 명령 이력은 제한되어야 한다. DEADLINE, LIFESPAN, LIVELINESS는 타이밍 장애와 오래된 정보를 감지하여 애플리케이션 수준 감시 타이머(Watchdog)를 보완할 수 있다. 그러나 안전 중요 모션 제어(Safety-Critical Motion Control)는 QoS에만 의존해서는 안 되며, 명령 검증, 타임아웃 처리, 안전 상태 전환(Safe-State Transition), 결정적 로컬 제어(Deterministic Local Control)가 반드시 함께 적용되어야 한다.

지도, 보정 매개변수, 로봇 설명(Robot Description), 설정 상태처럼 정적이거나 느리게 변화하는 정보는 RELIABLE 통신과 TRANSIENT_LOCAL 같은 보존형 지속성을 결합하여 사용할 수 있다. 새롭게 시작된 구성요소는 발행자가 해당 정보를 다시 생성할 때까지 기다리지 않고 필요한 상태를 전달받을 수 있다. 이러한 패턴은 스트리밍 센서 통신(Streaming Sensor Communication)과 근본적으로 다르며, 모든 토픽에 하나의 범용 ROS 2 QoS 프로파일을 적용하는 방식이 실제 운영 로봇에서 일반적으로 적절하지 않은 이유를 보여준다.

QoS 엔지니어링(QoS Engineering)은 각 데이터 흐름의 의미론에서 시작해야 한다. 모든 샘플이 중요한지, 정보가 얼마나 빠르게 오래된 데이터가 되는지, 어느 정도의 손실을 허용할 수 있는지, 늦게 참여하는 구독자를 어떻게 처리할 것인지, 어떤 자원 제한이 존재하는지를 먼저 정의해야 한다. 이러한 요구사항을 RELIABILITY, HISTORY, DURABILITY, DEADLINE, LIFESPAN, LIVELINESS 및 관련 지원 정책으로 매핑할 수 있다. 이렇게 설계된 프로파일은 패킷 손실, CPU 과부하, 지연된 구독자, 엔드포인트 재시작 및 실제적인 네트워크 혼잡 조건에서 검증해야 한다.

실제 운영 ROS 2 로봇(Production ROS 2 Robot)에서 DDS QoS는 단순한 미들웨어 튜닝 매개변수의 집합이 아니라 시스템 아키텍처(System Architecture)의 일부로 다루어야 한다. 인지 스트림, 내비게이션 상태, 명령, 진단 정보, 설정 및 플릿 통신(Fleet Communication)은 각각의 운영 의미론에 기반한 QoS 프로파일을 가져야 한다. 적절하게 설계된 QoS는 불필요한 트래픽을 줄이고, 오래된 정보의 전파를 방지하며, 장애 탐지 능력을 향상시키고, 메모리 사용량을 제어하며, 시스템 규모가 확장될수록 분산 로봇 통신을 더욱 예측 가능하게 만든다.

## 02.04 DDS Discovery Mechanism: SPDP / SEDP

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

DDS 탐색(DDS Discovery)은 분산된 참여자(Participant), 발행자(Publisher), 구독자(Subscriber), 데이터 작성자(DataWriter), 데이터 판독자(DataReader)가 모든 네트워크 연결을 수동으로 설정하지 않고도 호환 가능한 통신 상대를 찾을 수 있도록 하는 메커니즘이다. DDSI-RTPS 시스템에서 탐색은 일반적으로 두 개의 연계된 단계로 구성된다. 단순 참여자 탐색 프로토콜(Simple Participant Discovery Protocol, SPDP)은 DDS 참여자를 탐색하고, 단순 엔드포인트 탐색 프로토콜(Simple Endpoint Discovery Protocol, SEDP)은 해당 참여자 내부에 포함된 통신 엔드포인트(Communication Endpoint)를 탐색한다.

탐색(Discovery)은 DDS의 동적 특성을 구현하는 핵심 기능이다. 로봇 컴퓨터는 인지 컴퓨터보다 먼저 위치 추정 프로세스를 시작하거나, 운용 중 센서 드라이버를 재시작하거나, 고정된 통신 경로를 다시 구성하지 않고 새로운 처리 노드를 추가할 수 있다. DDS 미들웨어는 사용 가능한 참여자와 엔드포인트에 관한 정보를 지속적으로 유지한다. 호환 가능한 엔터티(Entity)가 나타나면 도메인 소속, 토픽(Topic) 정보, 데이터 타입(Data Type), QoS 호환성 및 네트워크 도달 가능성(Network Reachability)에 따라 통신 관계를 자동으로 설정할 수 있다.

첫 번째 탐색 단계는 참여자 수준에서 동작하는 SPDP이다. SPDP의 목적은 분산 시스템의 기본적인 질문, 즉 현재 어떤 DDS 참여자가 존재하며 그 참여자에게 어떻게 접근할 수 있는지를 확인하는 것이다. 각 참여자는 자신의 식별 정보와 통신 로케이터(Communication Locator)를 설명하는 정보를 주기적으로 알린다. 이러한 알림을 수신한 다른 참여자는 발견된 원격 참여자를 나타내는 정보를 생성하거나 갱신하며, 이후 엔드포인트 수준 탐색에 필요한 기반을 구축한다.

SPDP는 일반적인 애플리케이션 토픽 대신 미리 정의된 내장 RTPS 탐색 엔터티(Built-In RTPS Discovery Entity)를 사용한다. 참여자 탐색 정보는 프로토콜에 정의된 내장 엔드포인트(Built-In Endpoint)와 표준화된 탐색 데이터를 이용해 교환된다. 따라서 서로 이전에 통신한 적이 없는 두 DDS 구현체도 탐색을 시작할 수 있다. 탐색 과정은 DDSI-RTPS가 정의한 규칙을 이용해 스스로 초기화되므로 모든 애플리케이션이 다른 모든 참여자의 주소와 식별자를 사전에 알고 있어야 할 필요가 없다.

참여자는 주로 전역 고유 식별자(Globally Unique Identifier, GUID)를 통해 식별되며, GUID의 GUID 접두부(GUID Prefix)가 참여자 컨텍스트를 식별한다. SPDP 알림은 해당 참여자에게 접근할 수 있는 방법을 나타내는 로케이터 정보(Locator Information)도 제공한다. 구현체와 설정에 따라 이러한 로케이터는 유니캐스트(Unicast) 및 멀티캐스트(Multicast) 통신 경로를 나타낼 수 있다. 따라서 수신 측 미들웨어는 전역적으로 의미 있는 참여자 식별자와 이후 RTPS 통신에 사용되는 실제 네트워크 주소를 연결할 수 있다.

멀티캐스트(Multicast)는 새롭게 시작된 참여자가 다른 참여자의 개별 주소를 알지 못하더라도 자신의 존재를 여러 기존 참여자에게 알릴 수 있기 때문에 SPDP에서 일반적으로 사용된다. 기존 참여자 역시 자신의 존재를 알림으로써 새로운 참여자가 이들을 탐색할 수 있도록 한다. 이를 통해 적절한 로컬 네트워크에서 분산형 플러그 앤 플레이(Decentralized Plug-and-Play) 동작을 구현할 수 있다. 그러나 멀티캐스트 사용 가능 여부는 스위치, 라우터, Wi-Fi 액세스 포인트, VPN, 컨테이너(Container), 방화벽 및 네트워크 세분화(Network Segmentation) 정책에 크게 영향을 받는다.

참여자 탐색(Participant Discovery)은 순간적으로 한 번만 수행되고 영구적으로 유지되는 과정이 아니다. SPDP 알림은 탐색 타이밍 규칙(Discovery Timing Rule)에 따라 반복되며, 발견된 참여자에는 임대 정보(Lease Information)가 연결된다. 알림이나 기타 예상된 신호가 충분히 오랫동안 사라지면 원격 참여자가 더 이상 활성 상태가 아닌 것으로 판단하여 해당 탐색 상태를 제거할 수 있다. 이를 통해 컴퓨터 연결이 끊기거나 프로세스가 종료되거나 네트워크 경로에 장애가 발생하거나 로봇 서브시스템이 의도적으로 재시작되는 상황에 DDS 시스템이 적응할 수 있다.

SPDP가 참여자 수준의 인식(Participant-Level Awareness)을 확립하면 SEDP가 엔드포인트 탐색(Endpoint Discovery)을 수행한다. SEDP는 발견된 참여자에 속하는 데이터 작성자(DataWriter)와 데이터 판독자(DataReader)에 관한 정보를 전달한다. 이후 미들웨어는 어떤 엔드포인트가 호환 가능한 데이터를 위한 발행자와 구독자를 나타내는지 판단할 수 있다. 이러한 2단계 구조는 네트워크 참여자의 탐색과 애플리케이션 통신 관계의 탐색을 분리함으로써 많은 토픽과 엔드포인트를 포함하는 복잡한 시스템을 DDS가 관리할 수 있도록 한다.

SEDP는 내장 탐색 작성자와 판독자(Built-In Discovery Writer and Reader)를 이용하여 엔드포인트 메타데이터(Endpoint Metadata)를 배포한다. 이 메타데이터에는 엔드포인트 식별자, 토픽 이름, 데이터 타입 정보 및 관련 QoS 속성과 같이 작성자 또는 판독자의 특성을 정의하는 데 필요한 정보가 포함된다. 다른 참여자가 이러한 정보를 수신하면 해당 미들웨어는 로컬 엔드포인트와 발견된 원격 엔드포인트가 유효한 통신 관계를 형성할 수 있는지를 평가한다. 요구사항이 서로 호환되면 엔드포인트가 매칭(Matching)될 수 있다.

따라서 토픽 및 타입 호환성(Topic and Type Compatibility)은 SEDP의 핵심 요소이다. 위치 추정 정보를 발행하는 데이터 작성자가 두 참여자가 서로 접근 가능하다는 이유만으로 전혀 관련 없는 카메라 데이터 구조를 기대하는 데이터 판독자와 자동으로 통신해서는 안 된다. 탐색 메타데이터를 통해 미들웨어는 엔드포인트를 올바른 논리적 데이터 인터페이스(Logical Data Interface)와 연결할 수 있다. 특히 독립적으로 개발된 애플리케이션이나 서로 다른 DDS 구현체가 통신하는 경우 타입 일관성(Type Consistency)과 미들웨어별 타입 표현 방식도 함께 고려해야 한다.

QoS 호환성(QoS Compatibility)은 엔드포인트 매칭 과정의 일부로 평가된다. DDS는 일반적으로 데이터 작성자가 통신 기능을 제공하고 데이터 판독자가 특정 동작을 요청하는 제공-요청 모델(Offered-versus-Requested Model)을 따른다. 호환성 검사에 참여하는 정책들은 요구되는 관계를 만족해야 한다. 따라서 SPDP와 SEDP가 참여자와 엔드포인트를 성공적으로 탐색했더라도 작성자와 판독자의 QoS 설정이 호환되지 않으면 애플리케이션 데이터가 전달되지 않을 수 있다.

이러한 구분은 ROS 2 문제 해결(Troubleshooting)에서 매우 중요하다. 네트워크에서 원격 컴퓨터가 보인다고 해서 참여자 탐색이 성공했다는 의미는 아니며, ROS 2 토픽 이름이 보인다고 해서 모든 발행자와 구독이 호환 가능한 데이터 경로를 형성했다는 의미도 아니다. 문제는 네트워크 도달 가능성, SPDP 참여자 탐색, SEDP 엔드포인트 탐색, QoS 매칭 또는 사용자 데이터 전송 단계에서 각각 독립적으로 발생할 수 있다. 따라서 효과적인 진단에서는 탐색을 하나의 단일 이벤트로 취급하지 않고 통신 스택을 단계적으로 검사해야 한다.

탐색은 네트워크 오버헤드(Network Overhead)도 발생시킨다. 각 참여자는 자신의 존재를 알리고 엔드포인트 정보를 교환하며 탐색 상태를 유지하고 엔터티가 참여하거나 이탈할 때 이에 대응해야 한다. 몇 개의 ROS 2 프로세스만 사용하는 소형 로봇에서는 탐색 트래픽이 크지 않을 수 있지만, 많은 참여자와 수백 개의 엔드포인트를 포함하는 플릿(Fleet) 또는 분산 컴퓨팅 아키텍처에서는 상당한 탐색 활동이 발생할 수 있다. 따라서 확장성(Scalability)을 평가할 때 애플리케이션 토픽의 수뿐 아니라 DDS 참여자의 수도 중요한 요소가 된다.

ROS 2 프로세스 아키텍처는 ROS 미들웨어(rmw) 계층과 선택된 DDS 구현체를 통해 ROS 2 노드 아래에 미들웨어 엔터티가 생성되기 때문에 이러한 동작에 영향을 미친다. ROS 개념과 DDS 엔터티 사이의 정확한 매핑은 미들웨어 설계와 ROS 2 구현 세부사항에 따라 달라질 수 있다. 따라서 엔지니어는 모든 ROS 2 노드가 항상 하나의 독립적인 네트워크 수준 참여자와 직접 대응한다고 가정해서는 안 된다. 패킷 캡처(Packet Capture)와 미들웨어 진단을 사용하면 실제 탐색 동작을 더욱 정확하게 확인할 수 있다.

여러 네트워크 인터페이스(Multiple Network Interfaces)는 또 다른 일반적인 탐색 문제를 발생시킨다. 로봇 컴퓨터에는 이더넷(Ethernet), Wi-Fi, 가상 도커 인터페이스(Virtual Docker Interface), VPN 어댑터 및 루프백 인터페이스(Loopback Interface)가 동시에 존재할 수 있다. 미들웨어가 원격 참여자가 실제로 접근할 수 없는 인터페이스와 연결된 로케이터를 알리거나 선택할 수 있다. 그 결과 부분적인 탐색, 비대칭 통신(Asymmetric Communication), 또는 엔드포인트가 표시되지만 사용자 데이터를 교환하지 못하는 현상이 발생할 수 있다. 실제 운영 시스템에서는 명시적인 인터페이스 선택과 전송 설정이 필요한 경우가 많다.

컨테이너화 및 가상화 배포(Containerized and Virtualized Deployment)에서도 비슷한 주의가 필요하다. 도커 브리지 네트워크(Docker Bridge Network), 호스트 네트워킹(Host Networking), 쿠버네티스 오버레이(Kubernetes Overlay), 가상 머신(Virtual Machine), 네트워크 주소 변환(Network Address Translation, NAT), 클라우드 네트워킹(Cloud Networking)은 멀티캐스트 전달과 로케이터 도달 가능성을 변화시킬 수 있다. 따라서 하나의 이더넷 LAN에서 두 네이티브 프로세스 사이에 즉시 동작하던 DDS 탐색도 동일한 애플리케이션을 컨테이너로 이동하면 실패할 수 있다. 네트워크 아키텍처는 선택된 DDS 설정에 필요한 탐색 경로와 데이터 경로를 유지하도록 설계해야 한다.

무선 네트워크(Wireless Network)는 멀티캐스트 및 브로드캐스트와 유사한 트래픽이 일반적인 유니캐스트 트래픽과 다르게 처리될 수 있기 때문에 추가적인 복잡성을 가진다. 액세스 포인트는 멀티캐스트 전송률을 낮추거나, 멀티캐스트 패킷을 필터링하거나, 무선 클라이언트를 서로 격리하거나, 효율성을 위해 멀티캐스트 동작을 변환할 수 있다. 따라서 유선 이더넷에서 정상적으로 동작하는 ROS 2 시스템도 명목상 대역폭이 충분한 Wi-Fi 환경에서는 탐색이 느려지거나 통신이 불안정해질 수 있다. 그러므로 탐색 시험은 실제 배포 네트워크 환경을 반영해야 한다.

대규모 분산 시스템(Large Distributed System)에서는 제한 없는 멀티캐스트 기반 탐색 대신 다른 방식이나 확장 기능을 사용할 수 있다. DDS 구현체는 설정된 피어 목록(Configured Peer List), 탐색 서버(Discovery Server), 정적 탐색(Static Discovery), 구현체별 탐색 최적화(Discovery Optimization) 등의 메커니즘을 제공할 수 있다. 이러한 방식은 특히 라우팅된 네트워크나 대규모 플릿에서 멀티캐스트 의존성과 탐색 트래픽을 줄일 수 있다. 그러나 이는 기본적인 참여자 및 엔드포인트 탐색 모델을 이해하는 것을 대체하는 기술이 아니라 배포 아키텍처(Deployment Architecture)의 선택 사항으로 이해해야 한다.

DDS 보안(DDS Security) 역시 탐색 과정과 상호작용할 수 있다. 참여자의 존재를 발견했다는 것이 해당 참여자와 제한 없이 통신할 권한을 부여받았다는 의미는 아니다. 보안 DDS 배포에서는 참여자를 인증(Authentication)하고 권한(Permission)을 적용하며 설정된 보안 메커니즘에 따라 탐색 또는 사용자 데이터 교환을 보호할 수 있다. 이러한 기능은 자율 로봇이 공유 산업 네트워크에서 동작할 때 특히 중요하며, 동적 탐색이 네트워크에서 보이는 모든 참여자를 신뢰하거나 모든 토픽에 대한 접근을 허용한다는 의미가 되어서는 안 된다.

와이어샤크(Wireshark)와 미들웨어 로깅(Middleware Logging)은 탐색을 분석하는 데 유용한 도구이다. 패킷 검사를 통해 SPDP 참여자 알림과 이후의 SEDP 엔드포인트 교환을 확인할 수 있으며, 이를 통해 탐색 패킷이 한 호스트에서 정상적으로 송신되어 다른 호스트에 도달하는지를 판단할 수 있다. 인터페이스 검사, 멀티캐스트 시험, 방화벽 검증, ROS 2 명령줄 도구 및 DDS 전용 로그와 패킷 분석을 함께 사용하면 네트워크 전송 장애와 엔드포인트 매칭 또는 QoS 문제를 구분할 수 있다.

따라서 실용적인 진단 절차(Practical Diagnostic Sequence)는 탐색 계층 구조(Discovery Hierarchy)를 따른다. 엔지니어는 먼저 물리적 연결과 IP 연결성을 확인한 다음 필요한 멀티캐스트 또는 설정된 유니캐스트 탐색 경로를 사용할 수 있는지 확인한다. 이후 SPDP 참여자 탐색을 검사하고, SEDP 엔드포인트 교환, 토픽 및 타입 호환성, QoS 매칭, 마지막으로 사용자 데이터 전송을 차례대로 확인한다. 이러한 계층적 방법은 애플리케이션 수준의 증상으로 인해 통신 설정 과정의 훨씬 이전 단계에서 발생한 장애가 가려지는 것을 방지한다.

실제 운영 로보틱스(Production Robotics)에서 SPDP와 SEDP는 변화하는 컴퓨터 네트워크를 동적으로 연결된 DDS 데이터 공간(DDS Data Space)으로 변환하는 핵심 메커니즘으로 이해해야 한다. SPDP는 누가 존재하며 참여자에게 어디를 통해 접근할 수 있는지를 결정하고, SEDP는 해당 참여자가 어떤 작성자와 판독자를 포함하고 있으며 이들 엔드포인트가 서로 통신할 수 있는지를 결정한다. GUID, 로케이터, QoS 매칭, 임대 관리(Lease Management), RTPS 전송과 결합된 SPDP와 SEDP는 확장 가능한 ROS 2 통신을 위한 탐색 기반을 제공한다.

## 02.05 rmw Layer Comparison: FastDDS, CycloneDDS, Connext

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 미들웨어(ROS Middleware, rmw) 계층은 ROS 2 애플리케이션 프로그래밍 인터페이스(Application Programming Interface, API)와 하위 통신 미들웨어(Communication Middleware)를 분리하는 추상화 경계(Abstraction Boundary)이다. ROS 2 노드는 공통 ROS 인터페이스를 통해 발행자(Publisher), 구독(Subscription), 서비스(Service), 클라이언트(Client), QoS를 사용하며, rmw는 이러한 동작을 미들웨어별 메커니즘으로 변환한다. 이러한 아키텍처를 통해 ROS 2 소프트웨어는 애플리케이션 코드가 특정 공급업체 API에 직접 종속되지 않고도 서로 다른 DDS 구현체를 사용할 수 있다.

rmw 아키텍처는 일반적으로 ROS 클라이언트 라이브러리(ROS Client Library) 인터페이스 아래와 선택된 DDS 또는 미들웨어 구현체 위에 위치한다. rclcpp 또는 rclpy로 작성된 애플리케이션 코드는 최종적으로 ROS 클라이언트 라이브러리 스택을 통해 rmw 인터페이스의 기능을 호출한다. rmw_fastrtps_cpp, rmw_cyclonedds_cpp, rmw_connextdds와 같은 구현체는 이러한 공통 인터페이스를 각각 Fast DDS, Cyclone DDS, RTI Connext DDS에 연결한다.

이러한 추상화가 중요한 이유는 DDS 구현체들이 표준화된 개념을 공유하면서도 구현 아키텍처, 설정 메커니즘, 탐색(Discovery) 동작, 메모리 관리, 전송 기능, 진단 도구 및 성능 특성에서 차이를 가지기 때문이다. rmw 계층을 통해 ROS 2는 비교적 일관된 프로그래밍 모델을 제공하면서 배포 요구사항에 따라 미들웨어를 선택할 수 있는 능력을 유지한다. 따라서 미들웨어 선택은 애플리케이션 자체를 다시 작성하는 문제가 아니라 시스템 아키텍처의 선택 사항이 된다.

이프로시마(eProsima)가 개발한 Fast DDS는 ROS 2 생태계에서 널리 사용되는 오픈소스 DDS 구현체이다. DDS 및 DDSI-RTPS 통신 메커니즘을 구현하며 탐색, 전송(Transport), QoS, 메모리 동작 및 참여자 관리(Participant Management)를 위한 폭넓은 설정 기능을 제공한다. ROS 2 애플리케이션은 rmw_fastrtps_cpp를 통해 Fast DDS를 사용하면서도 표준 ROS 2 발행자, 구독, 서비스, 액션(Action), QoS 추상화를 계속 사용할 수 있다.

Fast DDS의 중요한 기능 중 하나는 폭넓은 배포 설정 메커니즘(Deployment Configuration Mechanism)이다. XML 프로파일(XML Profile)을 사용하여 참여자, 전송 방식, 탐색 설정, QoS 및 기타 미들웨어 동작을 정의할 수 있으므로 모든 매개변수를 애플리케이션 코드에 직접 포함할 필요가 없다. Fast DDS는 제한 없는 피어 투 피어 탐색(Peer-to-Peer Discovery) 트래픽을 줄일 수 있는 탐색 서버(Discovery Server) 아키텍처와 같은 메커니즘도 제공한다. 이러한 기능은 분산 로봇, 대규모 ROS 2 네트워크 및 멀티캐스트 탐색을 사용하기 어려운 네트워크 환경에서 유용할 수 있다.

Fast DDS는 설정 및 플랫폼 기능에 따라 공유 메모리(Shared Memory)와 네트워크 기반 전송 메커니즘을 지원한다. 카메라 영상이나 포인트 클라우드(Point Cloud)와 같은 대용량 데이터가 동일한 컴퓨터의 여러 프로세스 사이에서 이동하는 시스템에서는 불필요한 네트워크 스택 처리와 데이터 복사를 줄이는 것이 성능에 큰 영향을 미칠 수 있다. 그러나 실제 동작은 ROS 2 배포판, 미들웨어 버전, 메시지 경로, 설정 및 고급 데이터 공유(Data Sharing) 또는 대여 메시지(Loaned Message) 기능이 전체 소프트웨어 스택에서 지원되는지 여부에 따라 달라진다.

Cyclone DDS는 표준 준수(Standard Compliance), 상호운용성(Interoperability), 예측 가능한 동작 및 효율적인 구현을 중요하게 고려하여 개발된 오픈소스 DDS 구현체이다. ROS 2는 rmw_cyclonedds_cpp를 통해 Cyclone DDS를 사용한다. Cyclone DDS는 표준 ROS 2 애플리케이션 프로그래밍 모델을 유지하면서 네트워크 인터페이스 선택, 탐색 동작, 멀티캐스트 동작 및 전송 매개변수를 설정할 수 있는 비교적 간결한 DDS 구현체를 필요로 하는 로봇 시스템에서 활용될 수 있다.

Cyclone DDS 설정은 일반적으로 XML 기반 설정과 환경 변수 기반 설정(Environment-Controlled Configuration)을 통해 표현된다. 엔지니어는 네트워크 인터페이스, 멀티캐스트 동작, 참여자 탐색 매개변수, 주소 선택 및 기타 통신 특성을 지정할 수 있다. 명시적인 인터페이스 설정은 이더넷, Wi-Fi, 도커 브리지(Docker Bridge), VPN 어댑터 및 여러 물리적 인터페이스를 동시에 사용하는 로봇 컴퓨터에서 특히 중요하다. 자동 인터페이스 선택을 사용할 경우 원격 컴퓨터에서 실제로 접근할 수 없는 통신 경로가 광고될 수 있기 때문이다.

Cyclone DDS 역시 ROS 2 추상화 수준에서 설명되는 신뢰성(Reliability), 지속성(Durability), 이력(History), 엔드포인트 탐색(Endpoint Discovery), 형식화된 데이터 교환(Typed Data Exchange)과 같은 DDS QoS 및 RTPS 개념을 사용한다. 그러나 내부 알고리즘과 설정 문법은 Fast DDS와 동일하지 않다. 따라서 rmw 구현체를 변경하는 것을 단순히 라이브러리 이름을 바꾸는 작업으로 간주해서는 안 된다. 탐색 시간, 메모리 사용량, 네트워크 트래픽, 지연시간 분포 및 패킷 손실 상황에서의 동작은 구현체에 따라 달라질 수 있다.

RTI Connext DDS는 분산 실시간 시스템(Distributed Real-Time System)과 임무 중심 시스템(Mission-Oriented System) 분야에서 오랜 사용 역사를 가진 상용 DDS 구현체이다. ROS 2 환경과 설치된 Connext 구성요소에서 지원되는 경우 rmw_connextdds를 통해 ROS 2와 통합된다. Connext는 광범위한 DDS 기능, 설정 기능, 모니터링 기능, 보안 옵션 및 상용 기술 지원(Commercial Support)을 제공하므로 로봇 배포 환경에서 공급업체가 지원하는 미들웨어와 광범위한 산업용 DDS 인프라와의 통합이 필요한 경우 활용할 수 있다.

Connext는 ROS 2 시스템이 비(非) ROS DDS 애플리케이션(Non-ROS DDS Application)과 함께 동작해야 하는 경우 특히 중요할 수 있다. DDS는 ROS 2와 독립적으로 데이터 중심 통신 아키텍처를 정의하므로 산업용 제어기, 시뮬레이션 시스템, 항공우주 애플리케이션 또는 분산 모니터링 소프트웨어가 네이티브 DDS(Native DDS)를 직접 사용할 수 있다. 따라서 Connext 기반 환경은 ROS 2가 더 큰 DDS 시스템의 일부만을 구성하는 아키텍처에 참여할 수 있지만, 데이터 타입 매핑, QoS, 명명 규칙(Naming), 상호운용성 설정을 신중하게 정렬해야 한다.

세 가지 미들웨어 선택지를 하나의 보편적인 순위로 단순화해서는 안 된다. Fast DDS는 ROS 2에서 폭넓게 사용되며 설정 가능한 탐색 및 전송 기능을 제공한다. Cyclone DDS는 네트워크 설정 능력을 갖춘 간결하고 표준 지향적인 구현체를 제공한다. Connext DDS는 성숙한 도구와 광범위한 산업 기능을 갖춘 상용 지원 DDS 플랫폼을 제공한다. 어떤 특성이 가장 중요한지는 로봇 아키텍처, 네트워크 토폴로지(Network Topology), 인증 요구사항, 기술 지원 요구사항 및 실제 운영 환경에 따라 달라진다.

탐색 동작(Discovery Behavior)은 실험적으로 비교해야 하는 가장 중요한 영역 중 하나이다. Fast DDS와 Cyclone DDS는 모두 DDS/RTPS 참여자 및 엔드포인트 탐색을 지원하지만 설정 옵션과 최적화 메커니즘에는 차이가 있다. Connext 역시 자체적인 구현별 설정 생태계를 통해 DDS 탐색 기능을 제공한다. 따라서 수십 대의 컴퓨터 또는 수백 개의 엔드포인트를 포함하는 시스템에서는 탐색 수렴 시간(Discovery Convergence Time), 멀티캐스트 의존성, 재시작 동작, 트래픽 양, 라우팅되거나 분할된 네트워크에서의 동작을 평가해야 한다.

지연시간(Latency)과 처리량(Throughput)을 비교할 때는 단순히 미들웨어 이름에 기반하여 판단하기보다 통제된 벤치마킹(Controlled Benchmarking)을 수행해야 한다. 작은 제어 메시지, 고주기 상태 데이터, 대용량 카메라 프레임 및 수 메가바이트 규모의 포인트 클라우드는 서로 다른 통신 경로 특성을 사용한다. 측정에는 중앙값 및 꼬리 지연시간(Tail Latency), 지터(Jitter), 처리량, CPU 사용률, 메모리 사용량, 패킷 손실 동작 및 혼잡 이후의 복구 특성이 포함되어야 한다. 동일한 미들웨어도 QoS, 페이로드 크기, 구독자 수, 전송 방식 또는 호스트 토폴로지가 변경되면 서로 다른 성능을 나타낼 수 있다.

프로세스 내 통신(Intra-Process Communication)과 프로세스 간 통신(Inter-Process Communication)도 구분해야 한다. ROS 2는 동일한 프로세스 내부의 구성요소 사이에서 DDS 네트워크 경로의 상위 또는 병렬 계층에 존재하는 메커니즘을 이용하여 통신을 최적화할 수 있으며, 서로 다른 프로세스 사이의 통신에서는 미들웨어 직렬화, 공유 메모리 또는 네트워크 전송이 사용될 수 있다. 따라서 Fast DDS, Cyclone DDS, Connext를 벤치마킹할 때는 엔드포인트가 동일한 프로세스에 있는지, 동일한 호스트의 서로 다른 프로세스에 있는지, 또는 물리적 네트워크로 연결된 서로 다른 컴퓨터에 있는지를 명확하게 구분해야 한다.

대용량 메시지 통신(Large-Message Communication)은 피지컬 AI(Physical AI)와 자율 로봇에서 특히 중요하다. 카메라, 깊이 영상(Depth Image), 라이다 포인트 클라우드, 지도, 점유 격자(Occupancy Grid), 학습 기반 인지 결과는 네트워크 트래픽의 대부분을 차지할 수 있다. 단편화(Fragmentation), 소켓 버퍼, 공유 메모리 기능, 직렬화 오버헤드, 신뢰성 재전송 및 네트워크 최대 전송 단위(Maximum Transmission Unit, MTU)가 모두 성능에 영향을 미친다. 따라서 미들웨어 비교에는 실제 센서 페이로드를 포함해야 하며 운영 환경을 대표하지 못하는 작은 합성 핑퐁 메시지(Synthetic Ping-Pong Message)에만 의존해서는 안 된다.

QoS 의미론(QoS Semantics)은 rmw 구현체 전반에서 개념적으로 일관성을 유지하지만 지원되는 세부 기능과 런타임 동작에는 차이가 있을 수 있다. RELIABLE과 BEST_EFFORT, VOLATILE과 TRANSIENT_LOCAL, HISTORY 깊이, DEADLINE, LIVELINESS 및 기타 정책은 선택한 ROS 2 배포판과 미들웨어 버전을 기준으로 검증해야 한다. 하나의 구현체에서 정상적으로 동작한 설정도 미들웨어를 변경한 후에는 다시 시험해야 하며, 특히 고급 또는 비교적 사용 빈도가 낮은 DDS 정책을 사용하는 경우 이러한 검증이 중요하다.

여러 호환 가능한 구현체가 설치되어 있는 경우 ROS 2 미들웨어 구현체 전환은 일반적으로 RMW_IMPLEMENTATION 환경 변수(Environment Variable)를 통해 제어한다. 이를 통해 대부분의 애플리케이션 코드를 변경하지 않고 하위 미들웨어만 교체할 수 있으므로 비교 시험을 수행하기에 편리하다. 그러나 설정 파일, 환경 변수, 탐색 서버, 전송 설정, 보안 관련 파일 및 미들웨어별 튜닝도 변경해야 할 수 있으므로 실제 운영 환경의 마이그레이션(Migration)은 단일 환경 변수만 변경하는 것보다 더 많은 작업을 요구한다.

서로 다른 DDS 구현체 사이의 상호운용성은 DDSI-RTPS를 통해 가능하지만, 이를 당연히 보장되는 것으로 가정하지 말고 실제로 검증해야 한다. 호환 가능한 RTPS 통신은 상호운용성의 기반을 제공하지만 ROS 2 시스템에서는 여전히 도메인 설정, 토픽 및 타입 표현, QoS, 탐색, 보안 및 지원되는 프로토콜 기능을 서로 일치시켜야 한다. 특히 소프트웨어 버전이 다르거나 미들웨어별 최적화 기능이 일반적인 통신 경로를 변경하는 경우 혼합 공급업체 배포(Mixed-Vendor Deployment)에 대한 명시적인 통합 시험이 필요하다.

운영 도구(Operational Tooling)는 순수한 성능만큼 중요할 수 있다. 실험실에서 높은 성능을 제공하는 미들웨어라도 대규모 현장 배포에서 충분한 관찰 가능성(Observability)을 제공하지 못하면 유지보수의 어려움이 증가할 수 있다. 엔지니어는 로깅, 탐색 진단, 네트워크 추적(Network Tracing), 통계, 설정 관리, 보안 관리 및 공급업체 또는 커뮤니티 지원을 평가해야 한다. 와이어샤크(Wireshark)는 RTPS 수준 분석에 유용하며, 구현체별 도구는 패킷만으로 파악하기 어려운 정보를 제공할 수 있다.

실제 운영 로봇의 미들웨어를 선택할 때는 전체 배포 아키텍처를 기준으로 rmw 구현체를 평가해야 한다. 단일 컴퓨터 기반 연구용 로봇, 여러 엣지 컴퓨터를 사용하는 실외 자율 이동 로봇(Autonomous Mobile Robot, AMR), Wi-Fi 기반 로봇 플릿, 산업용 검사 시스템은 서로 완전히 다른 미들웨어 우선순위를 가질 수 있다. 네트워크 토폴로지, 데이터 양, 장애 복구, 보안, 결정적 동작(Deterministic Behavior), 유지보수성, 라이선스 및 통합 요구사항을 기준으로 비교 조건을 정의해야 한다.

rmw의 아키텍처적 가치는 궁극적으로 ROS 2 애플리케이션 로직을 이러한 미들웨어 결정으로부터 상당 부분 독립시킬 수 있다는 데 있다. Fast DDS, Cyclone DDS, Connext DDS는 공통 ROS 지향 추상화 아래에서 서로 다른 구현을 제공하며, 그 아래에서는 DDS와 RTPS가 표준화된 통신 개념을 제공한다. 이러한 계층 분리를 이해하면 로보틱스 엔지니어는 인지(Perception), 내비게이션(Navigation), 제어(Control), 미션 소프트웨어(Mission Software)를 하나의 미들웨어 제품에 불필요하게 결합하지 않으면서 통신 계층을 체계적으로 벤치마킹하고 튜닝할 수 있다.

## 02.06 DDS Multicast vs Unicast Configuration [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

DDS 통신은 멀티캐스트(Multicast)와 유니캐스트(Unicast) 네트워킹을 모두 사용할 수 있으며, 여러 컴퓨터, 로봇, 컨테이너(Container) 또는 네트워크 세그먼트(Network Segment)에 걸쳐 구성되는 ROS 2 시스템을 설계할 때 두 방식의 차이를 이해하는 것이 중요하다. 멀티캐스트는 하나의 패킷을 멀티캐스트 그룹(Multicast Group)에 참여하는 여러 수신자에게 전달하는 반면, 유니캐스트는 특정 네트워크 주소 사이에서 트래픽을 전달한다. DDS 구현체는 탐색(Discovery)과 애플리케이션 데이터에 서로 다른 방식을 적용하면서 두 메커니즘을 동시에 사용할 수 있다.

멀티캐스트는 새롭게 시작된 DDS 참여자(Participant)가 일반적으로 기존 참여자의 주소를 알지 못하기 때문에 자동 탐색(Automatic Discovery)에 특히 유용하다. 참여자는 미리 정의된 멀티캐스트 그룹으로 탐색 정보를 전송함으로써 여러 피어(Peer)에게 자신의 존재를 한 번에 알릴 수 있다. 동일한 그룹을 수신하는 다른 참여자는 이 알림을 받아 응답하거나 추가적인 탐색 정보를 교환할 수 있으며, 모든 엔드포인트(Endpoint)를 관리하는 중앙 레지스트리(Central Registry) 없이 분산 통신을 구성할 수 있다.

DDSI-RTPS에서 SPDP를 통한 참여자 탐색(Participant Discovery)은 일반적인 근거리 통신망(Local Area Network, LAN) 설정에서 멀티캐스트를 사용하는 경우가 많다. 참여자들이 서로를 발견하면 SEDP를 통해 데이터 작성자(DataWriter), 데이터 판독자(DataReader), 토픽(Topic), 타입(Type), QoS 정책 및 로케이터(Locator)를 설명하는 정보를 교환한다. 이후의 탐색 정보 교환과 사용자 데이터 통신은 미들웨어 및 설정에 따라 점차 유니캐스트 경로를 사용할 수 있다. 따라서 멀티캐스트 탐색과 유니캐스트 데이터 전송은 하나의 DDS 배포 환경에서 함께 사용되는 경우가 많다.

유니캐스트 통신(Unicast Communication)은 패킷을 그룹이 아니라 특정 목적지로 전송한다. DDS 참여자가 다른 참여자 또는 엔드포인트의 로케이터를 알고 있으면 유니캐스트를 통해 이들 사이에 직접적인 통신 경로를 구성할 수 있다. 이러한 방식은 라우팅(Routing), 방화벽 규칙(Firewall Rule), 대역폭 계산 및 패킷 추적이 동적으로 참여하는 멀티캐스트 그룹이 아니라 명시적인 출발지와 목적지 주소에 대응하기 때문에 통제된 네트워크 인프라에서 비교적 이해하고 관리하기 쉽다.

일대다(One-to-Many) 데이터 분산에서 멀티캐스트는 이론적으로 상당한 대역폭 효율성을 제공할 수 있다. 하나의 작성자(Writer)가 동일한 대용량 샘플을 10개의 판독자(Reader)에게 발행하는 경우 10개의 독립적인 유니캐스트 복사본 대신 하나의 멀티캐스트 스트림을 전송할 수 있다. 그러나 실제 효과는 미들웨어 지원, QoS, 전송 설정, 스위치 동작, 수신자 기능 및 네트워크 토폴로지(Network Topology)에 따라 달라진다. 따라서 많은 DDS 배포에서는 멀티캐스트를 주로 탐색에 사용하고 애플리케이션 데이터는 유니캐스트로 전달한다.

이더넷 스위치(Ethernet Switch)는 멀티캐스트 동작에 큰 영향을 미친다. 적절한 멀티캐스트 관리가 없으면 멀티캐스트 패킷이 여러 스위치 포트로 플러딩(Flooding)되어 불필요한 트래픽을 발생시킬 수 있다. 일반적으로 IGMP 스누핑(IGMP Snooping)이라고 하는 인터넷 그룹 관리 프로토콜 스누핑(Internet Group Management Protocol Snooping)은 호환되는 스위치가 어떤 포트에 멀티캐스트 그룹 구성원이 존재하는지 파악하여 트래픽을 보다 선택적으로 전달할 수 있도록 한다. 따라서 대규모 로봇 네트워크에서는 올바른 스위치 설정이 탐색 효율성과 전체 네트워크 안정성에 큰 영향을 미칠 수 있다.

라우터(Router)는 또 다른 중요한 경계를 형성한다. 멀티캐스트 트래픽은 일반적인 유니캐스트 트래픽과 동일한 방식으로 IP 서브넷(IP Subnet) 사이에 자동 전달되지 않는다. 모든 컴퓨터가 하나의 계층 2 네트워크(Layer-2 Network)에 존재할 때 정상적으로 통신하던 ROS 2 시스템도 장치들이 서로 다른 가상 근거리 통신망(Virtual LAN, VLAN)이나 라우팅된 서브넷(Routed Subnet)으로 분리되면 자동 탐색이 중단될 수 있다. 따라서 서브넷을 넘어 DDS 통신을 수행하려면 의도적으로 설계된 탐색 아키텍처, 라우팅 지원, 설정된 피어(Configured Peer), 탐색 서비스 또는 미들웨어별 메커니즘이 필요하다.

Wi-Fi는 멀티캐스트 트래픽이 유니캐스트 트래픽과 상당히 다르게 동작할 수 있기 때문에 특별한 고려가 필요하다. 무선 액세스 포인트(Wireless Access Point)는 멀티캐스트를 보수적인 데이터 전송률로 전송하거나, 다른 방식으로 버퍼링하거나, 전송을 억제하거나, 멀티캐스트-유니캐스트 변환(Multicast-to-Unicast Conversion)을 적용할 수 있다. 클라이언트 격리(Client Isolation)가 활성화되어 있으면 동일한 액세스 포인트를 사용하더라도 장치 간 통신이 차단될 수 있다. 따라서 유선 DDS 시험이 성공했다고 해서 무선 네트워크에서도 동일한 탐색 지연시간, 패킷 손실 및 안정성이 보장되는 것은 아니다.

멀티캐스트 설정은 여러 네트워크 인터페이스(Multiple Network Interfaces)를 사용하는 컴퓨터와도 밀접하게 관련된다. 로봇은 이더넷, Wi-Fi, VPN, 도커(Docker), 가상 머신(Virtual Machine), 루프백(Loopback) 인터페이스를 동시에 사용할 수 있다. DDS가 부적절한 인터페이스를 자동으로 선택하면 멀티캐스트 알림이 잘못된 네트워크를 통해 전송되거나 피어가 접근할 수 없는 로케이터가 광고될 수 있다. 실제 운영 설정에서는 탐색 및 사용자 트래픽이 의도한 물리적 로봇 네트워크를 사용하도록 허용되는 인터페이스를 명시적으로 제한하는 경우가 많다.

유니캐스트 기반 탐색(Unicast-Based Discovery)은 알려진 피어 또는 참여자들을 서로 연결해 주는 인프라를 설정함으로써 멀티캐스트 의존성을 줄일 수 있다. 이러한 방식은 멀티캐스트가 차단되거나 비효율적이거나 운영 측면에서 적합하지 않은 환경에서 유용하다. 그러나 네트워크 토폴로지에 관한 일부 정보를 설정이나 탐색 서비스를 통해 제공해야 한다는 절충점이 존재한다. 시스템은 제한 없는 멀티캐스트에 대한 의존성은 줄어들지만 정확한 주소 지정, 서비스 가용성 및 설정 관리(Configuration Management)에 대한 의존성은 증가한다.

Fast DDS는 인터페이스, 전송 방식, 탐색 동작, 초기 피어(Initial Peer), 탐색 서버(Discovery Server)와 같은 아키텍처를 제어하기 위한 설정 메커니즘을 제공한다. 탐색 서버는 참여자가 지정된 서버를 통해 탐색 정보를 획득하도록 하여 모든 참여자가 서로 직접 탐색하는 전대전(All-to-All) 탐색 패턴을 줄일 수 있다. 이는 제한 없는 멀티캐스트 탐색이 과도한 트래픽을 발생시키거나 기반 네트워크 인프라가 필요한 모든 통신 경계에서 멀티캐스트를 원활하게 지원하지 않는 대규모 ROS 2 시스템에서 유용할 수 있다.

Cyclone DDS 역시 인터페이스 선택, 멀티캐스트 사용, 피어 탐색(Peer Discovery), 주소 지정에 영향을 줄 수 있는 네트워크 설정 기능을 제공한다. 배포 네트워크에서 멀티캐스트가 전달될 수 없는 경우 명시적인 피어 설정(Explicit Peer Configuration)이 유용할 수 있다. 설정 문법과 세부 기능은 미들웨어 버전에 따라 달라질 수 있으므로 엔지니어는 Fast DDS의 설정 개념이나 매개변수가 Cyclone DDS에 직접 대응한다고 가정하지 말고 선택한 ROS 2 및 Cyclone DDS 버전을 기준으로 설정을 검증해야 한다.

RTI Connext DDS 역시 로컬 네트워크부터 대규모 분산 배포 환경까지 사용할 수 있는 설정 가능한 탐색 및 전송 아키텍처를 지원한다. Connext의 설정 생태계는 참여자 탐색, 인터페이스, 전송 방식 및 인프라 지원 통신(Infrastructure-Assisted Communication)을 제어하기 위한 메커니즘을 제공한다. 다른 DDS 구현체와 마찬가지로 멀티캐스트나 유니캐스트 중 어느 하나가 모든 로봇 네트워크에서 보편적으로 우수하다고 가정하기보다 실제 배포 요구사항을 기준으로 아키텍처를 결정해야 한다.

QoS는 선택된 전송 방식의 결과에도 영향을 미친다. 최선형(BEST_EFFORT) 센서 트래픽은 재전송 없이 패킷 손실을 허용할 수 있지만, 신뢰성(RELIABLE) 통신에서는 확인 응답, 부정 확인 응답, 하트비트(Heartbeat), 재전송 트래픽이 발생할 수 있다. 많은 판독자가 고대역폭의 신뢰성 데이터를 수신하면 복구 동작(Recovery Behavior)이 중요한 요소가 될 수 있다. 따라서 멀티캐스트와 유니캐스트의 선택은 신뢰성(RELIABILITY), 이력(HISTORY), 페이로드 크기, 발행 주기 및 구독자 수와 함께 평가해야 한다.

대용량 로봇 데이터 스트림(Large Robotic Data Stream)은 이러한 분석을 특히 중요하게 만든다. 여러 카메라, 깊이 센서(Depth Sensor), 라이다, 점유 격자(Occupancy Grid), 지도는 프로토콜 오버헤드와 재전송을 고려하기 전에도 초당 수백 메가비트의 대역폭을 사용할 수 있다. 동일한 스트림을 여러 소비자에게 유니캐스트로 각각 전송하면 대역폭 사용량이 배가될 수 있으며, 제대로 제어되지 않은 멀티캐스트는 관련 없는 장치에도 부하를 줄 수 있다. 따라서 네트워크 설계에서는 주요 토픽별로 예상 생산자, 소비자, 데이터 전송률, 페이로드, QoS 및 전송 경로를 매핑해야 한다.

보안 정책(Security Policy) 역시 멀티캐스트와 유니캐스트 아키텍처를 제한할 수 있다. 공유 산업 네트워크에서는 탐색 트래픽을 수신할 수 있는 모든 호스트가 신뢰할 수 있다고 가정해서는 안 된다. DDS 보안(DDS Security)은 참여자 인증(Authentication), 접근 제어(Access Control), 암호학적 보호(Cryptographic Protection)를 제공할 수 있으며, 네트워크 세분화와 방화벽 규칙은 하위 계층에서 통신 경로를 제한할 수 있다. 따라서 로봇이 생산 장비 또는 외부 시스템과 동일한 네트워크에서 동작할 경우 탐색의 편의성과 명시적인 신뢰 경계(Trust Boundary)를 균형 있게 설계해야 한다.

컨테이너(Container)는 두 통신 방식 모두를 복잡하게 만들 수 있다. 도커 브리지 네트워크와 오케스트레이션 오버레이(Orchestration Overlay)는 호스트 LAN과 동일한 방식으로 멀티캐스트를 전달하지 않을 수 있으며, 네트워크 주소 변환(Network Address Translation, NAT)은 광고된 유니캐스트 로케이터에 접근하지 못하게 만들 수 있다. 호스트 네트워킹(Host Networking)은 일부 DDS 배포를 단순화할 수 있지만 격리에 대한 기존 가정을 변화시킨다. 엔지니어는 컨테이너 내부의 ROS 2 프로세스가 네이티브 호스트 프로세스와 동일한 네트워크 토폴로지를 경험한다고 가정하지 말고 실제 인터페이스와 주소를 확인해야 한다.

진단(Diagnostics)에서는 멀티캐스트 탐색 문제와 유니캐스트 사용자 데이터 문제를 구분해야 한다. 엔지니어는 먼저 IP 연결성과 인터페이스 설정을 확인한 다음 멀티캐스트 패킷이 예상된 호스트까지 도달하는지 시험할 수 있다. 와이어샤크(Wireshark)를 사용하면 SPDP 알림, 목적지 멀티캐스트 주소, SEDP 교환, 광고된 로케이터 및 이후의 RTPS 데이터 패킷을 확인할 수 있다. 탐색은 성공하지만 사용자 데이터 전달이 실패한다면 엔드포인트 매칭, QoS, 라우팅, 방화벽 규칙 또는 광고된 유니캐스트 경로를 중점적으로 검사할 수 있다.

단일 로봇의 네트워크 설계는 일반적으로 플릿(Fleet)의 네트워크 설계보다 단순하다. 하나의 관리형 이더넷 스위치(Managed Ethernet Switch)에 연결된 여러 컴퓨터는 일반적인 멀티캐스트 탐색과 유니캐스트 애플리케이션 트래픽을 사용하여 효과적으로 동작할 수 있다. 그러나 아키텍처가 수십 대의 로봇, 무선 링크, VLAN, 엣지 서버(Edge Server), 플릿 관리 시스템으로 확장되면 제한 없는 탐색을 제어하기가 점점 어려워진다. 이 경우 계층적 세분화(Hierarchical Segmentation)와 인프라 지원 탐색(Infrastructure-Assisted Discovery)을 통해 더욱 예측 가능한 확장성을 확보할 수 있다.

실제 운영 아키텍처에서는 모든 DDS 참여자가 다른 모든 참여자를 탐색하도록 허용하는 대신 통신 범위(Communication Scope)를 분리할 수 있다. 로컬 센서와 제어기는 로봇 내부 네트워크에서 통신하고, 선택된 내비게이션, 미션, 진단 및 플릿 인터페이스만 정의된 네트워크 경계를 통과하도록 구성할 수 있다. DDS 도메인, 설정된 탐색 메커니즘, VLAN, 라우팅 정책 및 보안 제어를 함께 사용하면 이러한 아키텍처를 적용하면서 불필요한 탐색 및 데이터 트래픽을 줄일 수 있다.

따라서 멀티캐스트와 유니캐스트는 서로 경쟁하는 보편적 해결책이 아니라 상호보완적인 네트워킹 메커니즘(Complementary Networking Mechanism)으로 이해해야 한다. 멀티캐스트는 분산형 탐색에 매우 효과적이며 네트워크가 적절하게 설계된 경우 효율적인 일대다 데이터 분산을 지원할 수 있다. 유니캐스트는 명시적인 피어 투 피어(Peer-to-Peer) 경로를 제공하여 라우팅, 필터링, 진단 및 제어가 상대적으로 용이하다. DDS 미들웨어는 탐색 전략, 전송 설정, QoS 및 배포 토폴로지에 따라 이러한 메커니즘을 조합한다.

실제 운영 ROS 2 로보틱스(Production ROS 2 Robotics)에서 올바른 설정은 물리적 네트워크와 통신 그래프(Communication Graph)에 의해 결정된다. 엔지니어는 실제 부하 조건에서 유선 및 무선 동작, 멀티캐스트 지원, 인터페이스 선택, 라우팅 경계, 방화벽 정책, 구독자 수, 데이터 전송률 및 장애 복구를 검증해야 한다. 멀티캐스트 탐색, 통제된 유니캐스트 통신 및 확장 가능한 탐색 인프라를 의도적으로 결합하면 단일 자율 로봇부터 분산형 다중 로봇 플릿까지 예측 가능한 DDS 네트워킹을 구축할 수 있다.

## 02.07 DDS Bandwidth Optimization: Payload Size / Pub Rate [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

DDS 대역폭 최적화(DDS Bandwidth Optimization)는 페이로드 크기(Payload Size), 발행 주기(Publication Rate), 수신 엔드포인트(Receiving Endpoint) 수 사이의 단순한 관계에서 시작한다. S바이트의 페이로드를 F헤르츠(Hz)의 발행 주기로 전송하는 토픽(Topic)은 하나의 스트림에서 대략 S × F 바이트/초의 애플리케이션 수준 데이터 전송률을 생성한다. 실제 네트워크 부하는 직렬화(Serialization), RTPS 헤더, UDP/IP 헤더, 단편화(Fragmentation), 탐색 트래픽(Discovery Traffic), 신뢰성 제어 메시지 및 재전송으로 인해 이보다 더 높다.

페이로드 크기는 인지 중심 로봇 통신(Perception-Oriented Robot Communication)에서 가장 지배적인 요소가 되는 경우가 많다. 자세(Pose), 관절 상태(Joint State), 온도 또는 모드 표시와 같은 작은 메시지는 비교적 높은 발행 주기에서도 대역폭을 거의 사용하지 않을 수 있다. 반면 RGB 영상, 깊이 영상(Depth Image), 포인트 클라우드(Point Cloud), 점유 격자(Occupancy Grid), 학습 특징 텐서(Learned Feature Tensor)는 샘플당 수백 킬로바이트에서 수 메가바이트에 이를 수 있으므로 중간 수준의 주기에서도 하나의 토픽이 상당한 네트워크 용량을 소비할 수 있다.

발행 주기(Publication Rate) 역시 중요하다. 페이로드 크기가 일정하게 유지되면 대역폭은 메시지 주파수에 거의 선형적으로 증가하기 때문이다. 1MB 메시지를 10Hz로 발행하면 약 10MB/s의 애플리케이션 페이로드가 발생하지만, 동일한 메시지를 30Hz로 발행하면 프로토콜 오버헤드를 고려하기 전에도 약 30MB/s가 된다. 따라서 센서 또는 처리 파이프라인의 주기를 10Hz에서 30Hz로 증가시키면 메시지 표현 방식이 변하지 않더라도 직접적인 통신 비용이 증가한다.

독립적인 유니캐스트(Unicast) 방식으로 N개의 구독자에게 데이터를 전달하는 경우 1차적인 대역폭 모델은 Bpayload = S × F × N으로 표현할 수 있다. 이 근사식은 의도적으로 하위 계층 오버헤드를 제외하지만 아키텍처의 확장 문제를 즉시 파악할 수 있게 한다. 예를 들어 2MB 포인트 클라우드를 10Hz로 네 개의 원격 소비자에게 개별 유니캐스트 스트림으로 발행하면 RTPS와 네트워크 오버헤드를 고려하기 전에도 발행 측의 이론적인 페이로드 트래픽은 약 80MB/s에 도달할 수 있다.

따라서 실제 와이어 대역폭(Wire Bandwidth)은 ROS 메시지 정의만으로 추정하기보다 직접 측정해야 한다. DDS는 애플리케이션 데이터를 전송에 적합한 표현으로 직렬화하고, RTPS는 직렬화된 정보에 프로토콜 구조를 추가한다. UDP, IP, 이더넷(Ethernet)은 여기에 추가적인 헤더를 더한다. 대용량 메시지는 여러 RTPS 또는 네트워크 패킷으로 단편화될 수도 있다. 결과적으로 페이로드가 커지고 단편화가 증가할수록 패킷 수와 프로토콜 오버헤드도 증가한다.

단편화(Fragmentation)는 카메라 영상, 포인트 클라우드, 지도 및 기타 대용량 로봇 메시지에서 특히 중요해진다. 점보 프레임(Jumbo Frame)을 의도적으로 설정하지 않는 한 이더넷은 일반적으로 약 1500바이트의 최대 전송 단위(Maximum Transmission Unit, MTU)를 사용한다. DDS/RTPS 구현체는 대규모 직렬화 샘플을 여러 전송 단위로 분할할 수 있다. 특히 신뢰성(RELIABLE) QoS에서는 하나의 단편 손실도 더 큰 샘플의 복구에 영향을 줄 수 있으므로 대용량 메시지 성능은 전체 대역폭뿐 아니라 패킷 손실과 재전송 동작에도 좌우된다.

발행 주기는 단순히 센서의 최대 출력 주기에 맞추기보다 하위 알고리즘(Downstream Algorithm)이 실제로 요구하는 정보량을 기준으로 결정해야 한다. 초당 60프레임을 출력할 수 있는 카메라라고 해서 모든 소비자가 반드시 60FPS를 수신해야 하는 것은 아니다. 매핑(Mapping), 객체 탐지(Object Detection), 시각화(Visualization), 로깅(Logging)은 각각 서로 다른 주기를 요구할 수 있다. 센서 획득 주기(Acquisition Frequency)와 데이터 배포 주기(Distribution Frequency)를 분리하면 센서 자체의 기능을 제한하지 않으면서 불필요한 DDS 트래픽을 크게 줄일 수 있다.

동일한 원칙은 고주기 상태 정보(High-Rate State Information)에도 적용된다. 관성 측정 장치(Inertial Measurement Unit, IMU)는 로컬 추정기(Local Estimator)가 필요로 하기 때문에 수백 헤르츠의 측정값을 생성할 수 있지만, 플릿 모니터(Fleet Monitor)는 초당 몇 번의 업데이트만 필요할 수 있다. 전체 고주기 스트림을 모든 네트워크 경계를 넘어 전송하면 대역폭과 처리 자원을 낭비하게 된다. 로컬 처리 노드는 전체 주기의 신호를 사용하고 원격 시스템에는 더 낮은 주기의 파생 상태(Derived State), 진단 정보 또는 요약 정보를 발행할 수 있다.

따라서 ROS 2 및 DDS 아키텍처는 로컬 실시간 데이터(Local Real-Time Data)와 분산 운영 데이터(Distributed Operational Data)를 구분해야 한다. 고주기 원시 센서 정보(Raw Sensor Information)는 로봇 내부 또는 하나의 컴퓨팅 호스트 내부에 유지하고, 위치 추정, 객체 트랙(Object Track), 상태 정보, 미션 상태(Mission Status), 선택된 인지 결과만 외부 네트워크 링크를 통과하도록 구성할 수 있다. 이러한 계층적 데이터 흐름 설계(Hierarchical Data-Flow Design)는 제한 없는 통신 그래프를 먼저 만든 후 개별 토픽을 최적화하는 것보다 효과적인 경우가 많다.

QoS 선택은 대역폭 소비에 직접적인 영향을 미친다. 최선형(BEST_EFFORT) 통신은 신뢰성 복구 트래픽을 발생시키지 않으므로 일부 샘플을 손실하는 것이 오래된 데이터를 재전송하는 것보다 나은 빠르게 갱신되는 센서 스트림에 적합한 경우가 많다. 신뢰성(RELIABLE) 통신에서는 하트비트(Heartbeat), 확인 응답(Acknowledgment), 부정 확인 응답(Negative Acknowledgment), 재전송 메커니즘이 추가된다. 이러한 비용은 안정적인 네트워크에서는 작을 수 있지만 패킷 손실, 혼잡 또는 많은 구독자가 존재하면 크게 증가할 수 있다.

이력 깊이(History Depth)는 네트워크와 메모리 동작에 간접적으로 영향을 미칠 수 있다. 깊은 최근 유지(KEEP_LAST) 큐는 구독자의 처리가 지연되는 동안 더 많은 샘플을 보관할 수 있지만, 오래된 고대역폭 센서 데이터는 운영상 가치가 거의 없을 수 있다. 깊은 이력과 RELIABLE 통신을 함께 사용하면 메모리 압박과 복구 트래픽이 증가할 수 있다. 실시간 인지 스트림에서는 얕은 큐가 최신성(Freshness)을 유지하는 데 더 효과적인 경우가 많지만, 명령, 이벤트 또는 상태 전환에는 다른 이력 및 신뢰성 설정이 적합할 수 있다.

수명(LIFESPAN)은 오래된 정보가 자원을 소비하는 것을 방지하기 위한 또 다른 메커니즘을 제공한다. 장애물 탐지, 속도 명령 또는 로컬 인지 결과가 제한된 시간 동안만 유효하다면 적절한 수명을 설정하여 만료된 샘플이 통신에서 계속 유효한 정보로 취급되지 않도록 할 수 있다. 애플리케이션 수준 타임스탬프(Application-Level Timestamp)와 최신성 검사는 계속 유지해야 하지만, DDS 수명 의미론은 분산 데이터의 시간적 유효성(Temporal Validity)을 명시적으로 표현하여 이를 보완할 수 있다.

네트워크 확장성을 추정할 때는 구독자 수(Subscriber Count)를 반드시 고려해야 한다. 유니캐스트 전송에서는 유사한 샘플이 여러 목적지에 개별적으로 전달되기 때문에 판독자(Reader)가 추가될수록 송신 트래픽이 증가할 수 있다. 하나의 구독자에서는 부담이 적은 토픽도 10개 또는 20개의 구독자가 연결되면 상당한 트래픽을 발생시킬 수 있다. 적절한 일대다 패턴에서는 멀티캐스트(Multicast) 기반 데이터 분산이 중복 전송을 줄일 수 있지만, 실제 효율은 DDS 구현체, 네트워크 인프라, QoS 및 수신자 동작에 따라 달라진다.

영상 압축(Image Compression)은 추가적인 계산 비용과 지연시간을 허용할 수 있다면 대역폭을 크게 줄일 수 있다. 원시 RGB 영상은 각 프레임마다 가로 해상도 × 세로 해상도 × 픽셀당 바이트의 저장 공간을 요구하므로 해상도와 프레임률이 증가하면 트래픽도 빠르게 증가한다. JPEG, H.264, H.265 또는 애플리케이션별 압축 표현을 사용하면 전송 크기를 크게 줄일 수 있지만 압축 과정에서 처리 지연, 계산 부하, 품질 손실 및 추가적인 장애 가능성이 발생할 수 있으므로 함께 고려해야 한다.

포인트 클라우드 최적화(Point-Cloud Optimization)에서도 유사한 아키텍처 결정이 필요하다. 수십만 또는 수백만 개의 포인트를 반복적으로 전송하면 원시 XYZ 또는 XYZRGB 표현의 크기가 매우 커질 수 있다. 복셀 다운샘플링(Voxel Downsampling), 관심 영역 필터링(Region-of-Interest Filtering), 거리 필터링(Range Filtering), 정밀도 축소(Reduced Precision), 불필요한 필드 제거 또는 상위 수준 기하학적 표현(Geometric Representation)으로의 변환을 통해 페이로드 크기를 줄일 수 있다. 적절한 전략은 하위 소비자가 원시 측정값을 필요로 하는지 또는 측정값에서 추출된 정보만 필요로 하는지에 따라 결정된다.

메시지 설계(Message Design) 자체가 불필요한 트래픽을 만들 수도 있다. 데이터 구조의 작은 일부만 변경되는데도 전체 대형 구조를 반복해서 발행하면 대역폭이 낭비된다. 정적 설정, 보정 정보(Calibration), 지도 메타데이터 또는 로봇 설명(Robot Description)은 다른 메시지에 포함하기 편리하다는 이유만으로 높은 주기로 전송해서는 안 된다. 정적 정보, 느리게 변경되는 정보, 고주기 동적 정보를 적절한 토픽으로 분리하면 각 데이터 유형에 실제 의미론에 맞는 발행 주기와 QoS 프로파일을 적용할 수 있다.

자주 전송되는 데이터 구조에서는 직렬화 효율(Serialization Efficiency)도 고려해야 한다. 필드 정렬(Field Alignment), 가변 길이 배열(Variable-Length Array), 문자열(String), 중첩 메시지(Nested Message), 대규모 컬렉션(Collection)은 직렬화된 표현과 처리 비용에 영향을 미친다. 그러나 작은 제어 메시지에서 몇 바이트를 줄이는 것은 일반적으로 영상 해상도나 포인트 클라우드 밀도를 줄이는 것보다 효과가 작다. 따라서 최적화 작업은 먼저 페이로드 크기와 전송 주기의 곱이 가장 큰 토픽에 집중해야 한다.

공유 메모리 전송(Shared-Memory Transport)은 동일한 컴퓨터의 여러 프로세스 사이에서 대용량 메시지를 교환할 때 네트워크 스택 오버헤드를 크게 줄일 수 있다. DDS 구현체, ROS 2 배포판 및 설정에 따라 데이터 공유(Data Sharing) 또는 공유 메모리 메커니즘은 일부 직렬화, 데이터 복사 또는 소켓 동작을 줄일 수 있다. 호환 가능한 구성요소가 하나의 프로세스에서 실행되는 경우 프로세스 내 통신(Intra-Process Communication)을 통해 오버헤드를 더욱 줄일 수 있다. 정확한 데이터 경로는 소프트웨어 스택에 따라 달라지므로 이러한 최적화는 실제 벤치마크를 통해 검증해야 한다.

네트워크 용량은 명목상의 링크 속도만을 기준으로 계획해서는 안 된다. 1Gbit/s 이더넷 링크가 존재한다고 해서 로봇 애플리케이션이 지속적으로 정확히 1Gbit/s의 DDS 트래픽을 사용하도록 설계해도 된다는 의미는 아니다. 프로토콜 오버헤드, 순간적인 트래픽 버스트(Traffic Burst), 신뢰성 복구, 운영체제 스케줄링, 스위치 버퍼링, 다른 서비스의 트래픽 및 향후 확장을 위한 엔지니어링 여유가 필요하다. 물리적 용량에 가까운 지속적인 사용률은 큐 누적, 패킷 손실, 지연시간 급증 및 불안정한 복구 동작을 발생시킬 수 있다.

따라서 대역폭 측정에는 평균 동작과 최대 부하 동작(Peak Behavior)을 모두 포함해야 한다. 평균 처리량은 적절해 보이더라도 동기화된 카메라 또는 주기적인 지도 업데이트가 짧은 트래픽 버스트를 생성하여 버퍼를 초과할 수 있다. 엔지니어는 초당 바이트(Bytes per Second), 초당 패킷(Packets per Second), 지연시간, 지터(Jitter), 손실 패킷, 재전송, CPU 사용률 및 큐 동작을 측정해야 한다. 와이어샤크(Wireshark), 운영체제 네트워크 통계, DDS 진단 및 ROS 2 토픽 측정은 통신 경로를 서로 다른 관점에서 분석할 수 있게 한다.

최적화는 가장 큰 트래픽 기여 요소부터 순차적으로 진행해야 한다. 엔지니어는 각 토픽을 목록화하고 직렬화된 페이로드 크기를 추정하며 발행 주기를 측정하고 구독자 수, QoS 및 네트워크 경계를 기록할 수 있다. 페이로드 크기와 전송 주기를 곱하면 높은 데이터량을 발생시키는 스트림을 즉시 식별할 수 있다. 이후 이러한 토픽을 대상으로 해상도 감소, 주기 감소, 필터링, 압축, 로컬 처리, 공유 메모리, 멀티캐스트 분산 또는 아키텍처 분리를 검토할 수 있다.

다중 로봇 플릿(Multi-Robot Fleet)에서는 대역폭 최적화를 개별 토픽 수준에서 통신 범위(Communication Scope) 수준으로 확장해야 한다. 애플리케이션이 명시적으로 요구하지 않는 한 모든 로봇의 원시 카메라 및 라이다 스트림을 모든 플릿 컴퓨터로 배포하는 것은 일반적으로 적절하지 않다. 로봇은 대용량 데이터를 로컬에서 처리하고 압축된 의미 정보(Semantic Result), 궤적(Trajectory), 상태 정보 및 미션 상태를 발행할 수 있다. 디버깅, 원격 조작(Teleoperation) 또는 사고 분석이 필요한 경우에만 선택적으로 원시 데이터를 전송할 수 있다.

실제 운영 DDS 대역폭 예산(Bandwidth Budget)은 정상 동작뿐 아니라 네트워크 성능 저하 및 장애 복구 상황까지 포함해야 한다. 패킷 손실로 재전송이 발생하거나, 구독자가 일시적으로 정지하거나, 여러 구성요소가 재시작되거나, 다수의 고용량 토픽이 동시에 활성화되는 상황에서도 시스템은 안정성을 유지해야 한다. 따라서 페이로드 크기와 발행 주기는 단순한 성능 매개변수가 아니라 QoS, 구독자 토폴로지, 전송 방식 및 처리 위치와 함께 전체 통신 아키텍처를 결정하는 요소이다.

효과적인 DDS 최적화는 궁극적으로 의미론적 원칙(Semantic Principle)을 따른다. 필요한 정보를, 필요한 해상도와 주기로, 실제로 필요로 하는 소비자에게만 전송해야 한다. 페이로드 감소, 주기 제어, QoS 튜닝, 로컬 처리, 압축, 공유 메모리 및 네트워크 세분화는 서로 보완적인 기법이다. 이러한 방법을 체계적으로 적용하면 ROS 2 통신을 통제되지 않은 토픽 집합에서 공학적으로 설계된 데이터 흐름 시스템(Engineered Data-Flow System)으로 전환하여 단일 로봇 컴퓨터부터 분산 피지컬 AI 플릿(Distributed Physical AI Fleet)까지 확장할 수 있다.

## 02.08 DDS Security Plugin: Auth, Encryption, Access Control [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

DDS 보안(DDS Security)은 분산 참여자(Distributed Participant), 탐색 정보(Discovery Information), 애플리케이션 데이터(Application Data)를 보호하기 위한 표준화된 메커니즘을 데이터 분산 서비스(Data Distribution Service, DDS) 통신 모델에 확장하여 제공한다. 보안을 외부 네트워크 기능으로만 취급하는 대신 DDS는 보안 제어(Security Control)를 미들웨어 아키텍처에 통합한다. 주요 보안 기능은 인증(Authentication), 접근 제어(Access Control), 암호학적 보호(Cryptographic Protection)이며, 이를 통해 분산 로봇 시스템은 참여자를 검증하고 통신 권한을 제한하며 교환되는 정보를 보호할 수 있다.

DDS 보안 아키텍처(DDS Security Architecture)는 플러그인 지향 모델(Plugin-Oriented Model)을 기반으로 한다. 보안 기능은 표준화된 플러그인 인터페이스(Plugin Interface)로 분리되므로 구현체는 공통 DDS 보안 의미론을 유지하면서 인증, 접근 제어, 암호화 연산(Cryptographic Operation), 로깅(Logging) 및 관련 기능을 제공할 수 있다. 이러한 모듈형 설계는 애플리케이션 통신 로직과 보안 구현 세부사항을 분리하며, ROS 2 애플리케이션이 자체 보호 프로토콜을 구현하지 않고도 미들웨어 공급업체가 보안 메커니즘을 통합할 수 있도록 한다.

인증(Authentication)은 원격 DDS 참여자가 허용 가능한 신원(Identity)을 가지고 있는지를 확인하는 기본적인 문제를 다룬다. 보호된 통신을 설정하기 전에 참여자는 인증 정보를 교환하고 설정된 자격 증명(Credential)을 이용하여 핸드셰이크(Handshake)를 수행할 수 있다. 일반적으로 인증서 기반 메커니즘(Certificate-Based Mechanism)이 사용되며, 신원은 디지털 인증서(Digital Certificate)와 이에 대응하는 개인 키(Private Key)를 통해 표현된다. 인증이 성공하면 통신 참여자가 승인된 신원과 연관된 자격 증명을 실제로 보유하고 있다는 신뢰를 형성할 수 있다.

인증서 기반 배포(Certificate-Based Deployment)는 일반적으로 신원 인증서(Identity Certificate), 개인 키, 신뢰할 수 있는 인증 기관(Certificate Authority, CA)을 포함하는 신뢰 모델(Trust Model)을 사용한다. 각 DDS 참여자에는 자신의 신원을 나타내는 자격 증명을 제공할 수 있으며, 신뢰할 수 있는 CA 인증서는 원격 인증서를 검증하기 위한 기반을 제공한다. 개인 키는 반드시 보호되어야 한다. 참여자 인증서만 소유하고 해당 개인 키를 가지고 있지 않은 경우에는 암호학적 인증 과정에서 해당 참여자를 사칭할 수 없어야 하기 때문이다.

인증만으로는 인증된 참여자가 무엇을 수행할 수 있는지를 결정하지 않는다. 접근 제어(Access Control)는 참여자가 사용할 수 있는 DDS 자원과 동작을 정의하여 인가(Authorization)를 제공한다. 권한(Permission)을 통해 특정 도메인(Domain)에 대한 참여를 제한하고 인증된 엔터티가 특정 토픽(Topic)을 발행하거나 구독할 수 있는지를 제어할 수 있다. 이는 기본적인 보안 원칙을 따른다. 즉, 인증은 참여자가 누구인지를 확인하고, 인가는 해당 신원이 어떤 통신 활동을 수행하도록 허용되는지를 결정한다.

DDS 보안은 일반적으로 거버넌스(Governance)와 권한(Permissions) 정보를 사용하여 보안 정책(Security Policy)을 표현한다. 거버넌스 규칙(Governance Rule)은 도메인, 토픽, 탐색 보호(Discovery Protection), 메타데이터(Metadata), 데이터 보호와 관련된 보안 요구사항을 정의한다. 권한 규칙(Permissions Rule)은 특정 인증 신원이 어떤 데이터를 발행하거나 구독하고 어떤 자원에 접근할 수 있는지를 정의한다. 이러한 정책 구성요소를 결합하면 동적으로 탐색되는 DDS 네트워크를 암묵적으로 개방된 통신 환경에서 명시적인 신뢰 및 인가 규칙이 적용되는 환경으로 전환할 수 있다.

토픽 수준 접근 제어(Topic-Level Access Control)는 모든 컴퓨터가 모든 데이터 스트림에 접근할 필요가 없는 로보틱스에서 특히 유용하다. 시각화 워크스테이션(Visualization Workstation)은 센서 및 진단 토픽을 필요로 할 수 있지만 액추에이터 명령(Actuator Command)을 발행할 필요는 없을 수 있다. 플릿 서버(Fleet Server)는 모든 원시 카메라 스트림을 수신하지 않고도 미션 상태와 시스템 상태 정보를 필요로 할 수 있다. 따라서 안전 관련 명령 토픽에는 일반 모니터링 또는 비중요 텔레메트리(Noncritical Telemetry)보다 엄격한 권한을 적용할 수 있다.

암호학적 보호(Cryptographic Protection)는 DDS 통신의 기밀성(Confidentiality)과 무결성(Integrity)을 제공한다. 기밀성은 권한이 없는 관찰자가 보호된 정보를 읽는 것을 방지하고, 무결성 메커니즘은 수신자가 승인되지 않은 데이터 변경을 탐지할 수 있도록 한다. 설정된 DDS 보안 정책에 따라 애플리케이션 페이로드와 선택된 프로토콜 메타데이터를 포함하여 통신의 서로 다른 부분에 보호를 적용할 수 있다. 정확한 보호 수준은 위협 모델(Threat Model), 성능 요구사항 및 상호운용성 제약을 기준으로 선택해야 한다.

암호화(Encryption)는 실시간 로봇 시스템에서 고려해야 하는 계산 및 통신 비용을 발생시킨다. 암호학적 처리는 CPU 자원을 사용하고 추가적인 지연시간(Latency)을 발생시킬 수 있으며, 보안 메타데이터가 추가되면서 패킷 크기도 증가한다. 낮은 주기의 명령 메시지에서는 이러한 영향이 작을 수 있지만 고대역폭 카메라 또는 라이다(LiDAR) 스트림에서는 더욱 중요해질 수 있다. 따라서 보안 성능은 실제 페이로드 크기, 발행 주기, 구독자 수 및 목표 컴퓨팅 하드웨어를 사용하여 측정해야 한다.

DDS는 일반적으로 참여자와 엔드포인트를 동적으로 탐색하기 때문에 탐색(Discovery)에 특별한 주의가 필요하다. 보안되지 않은 탐색 메커니즘은 애플리케이션 페이로드가 별도로 보호되어 있더라도 사용 가능한 참여자, 토픽, 주소 및 통신 관계에 관한 정보를 노출할 수 있다. DDS 보안은 필요한 탐색 및 메타데이터 교환을 보호하기 위한 메커니즘을 제공하지만, 어떤 정보를 보호하고 보안 기능이 탐색 과정과 어떻게 상호작용할지는 선택된 거버넌스 설정에 의해 결정된다.

보안 참여자 탐색(Secure Participant Discovery)은 일반적인 데이터 통신이 가능해지기 전에 필요한 처리 순서도 변화시킨다. 참여자들은 먼저 보안 처리를 시작하기에 충분한 정보를 탐색하고, 서로를 인증하며, 권한을 평가하고, 필요한 경우 암호학적 자료(Cryptographic Material)를 설정한 후 승인된 통신 관계를 형성해야 한다. 따라서 보안이 적용된 시작 과정은 보안이 적용되지 않은 DDS 배포보다 추가적인 메시지 교환과 처리를 요구할 수 있으며, 탐색 수렴 시간(Discovery Convergence)과 재시작 동작을 측정할 때 이를 고려해야 한다.

접근 제어 정책은 최소 권한 원칙(Principle of Least Privilege)을 따라야 한다. 각 로봇 구성요소에는 운영 역할에 필요한 권한만 제공해야 한다. 카메라 프로세스가 모션 제어 기능과 동일한 컴퓨터에서 실행된다는 이유만으로 모션 명령을 발행할 권한까지 자동으로 가져서는 안 된다. 마찬가지로 진단 도구(Diagnostic Tool)가 미션 또는 제어 토픽에 제한 없는 쓰기 권한을 가져서는 안 된다. 세분화된 권한(Fine-Grained Permission)은 구성요소가 침해되거나 잘못 설정되었을 때 발생할 수 있는 영향을 줄인다.

보안 아키텍처에서는 DDS 도메인(DDS Domain)도 고려해야 한다. 도메인 분리는 시스템 사이의 의도하지 않은 통신을 줄일 수 있지만 도메인 식별자(Domain Identifier) 자체가 강력한 보안 경계(Security Boundary)는 아니다. 네트워크에 접근할 수 있는 권한 없는 참여자가 올바른 도메인 설정을 알고 있다는 이유만으로 신뢰받아서는 안 된다. 의미 있는 보안 격리가 필요한 경우 인증, 접근 제어, 네트워크 세분화(Network Segmentation), 방화벽 정책 및 자격 증명 관리를 도메인 구성과 함께 적용해야 한다.

네트워크 암호화(Network Encryption)와 DDS 수준 보안(DDS-Level Security)은 서로 다른 문제를 해결한다. VPN과 같은 기술은 네트워크 링크를 통과하는 트래픽을 보호할 수 있지만 보호된 네트워크 내부의 개별 DDS 참여자 사이에서 토픽 수준 인가(Topic-Level Authorization)를 반드시 제공하는 것은 아니다. DDS 보안은 DDS 신원, 도메인, 토픽 및 통신 역할을 이해한 상태에서 동작한다. 따라서 실제 운영 아키텍처에서는 네트워크 계층 보호와 미들웨어 수준 인증 및 접근 제어를 서로 대체 가능한 기능으로 보지 않고 함께 사용할 수 있다.

자격 증명 수명주기 관리(Credential Lifecycle Management)는 초기 보안 설정만큼 중요하다. 인증서와 개인 키는 안전하게 프로비저닝(Provisioning)되고 적절하게 저장되어야 하며, 만료 전에 갱신되고 장치가 폐기되거나 침해된 경우 취소 또는 교체되어야 한다. 많은 로봇으로 구성된 플릿에서는 수동 자격 증명 관리가 빠르게 운영상의 문제가 될 수 있다. 따라서 자동화된 프로비저닝, 안전한 장치 신원(Device Identity), 통제된 인증서 교체(Certificate Rotation), 감사 가능성(Auditability), 복구 절차를 초기 설계 단계부터 고려해야 한다.

모바일 로봇은 배포된 하드웨어에 물리적으로 접근할 가능성이 있기 때문에 개인 키 보호(Private-Key Protection)에 특별한 주의가 필요하다. 장기간 사용하는 개인 키를 제한 없는 평문 파일(Plaintext File)로 저장하면 DDS 보안이 올바르게 설정되어 있더라도 전체 보안 아키텍처가 약화된다. 플랫폼과 위협 모델에 따라 보호된 키 저장소(Protected Key Store), 하드웨어 보안 모듈(Hardware Security Module, HSM), 신뢰 플랫폼 모듈(Trusted Platform Module, TPM), 제한된 파일 권한, 보안 부팅(Secure Boot), 디스크 보호 등을 사용하여 DDS 참여자를 둘러싼 자격 증명 경계를 강화할 수 있다.

ROS 2는 일반적인 노드가 직접 암호화 연산을 수행하도록 요구하는 대신 미들웨어 아키텍처를 통해 DDS 보안 기능을 제공한다. 보안 동작은 선택된 ROS 미들웨어(ROS Middleware, rmw) 구현체, DDS 구현체, ROS 2 배포판 및 보안 설정에 따라 달라진다. Fast DDS, Cyclone DDS, Connext DDS는 지원 기능, 설정 절차, 도구 및 운영 세부사항에서 차이가 있을 수 있으므로 실제 보안 배포에서는 사용되는 정확한 미들웨어 스택을 기준으로 검증해야 한다.

ROS 2 보안 설정은 일반적으로 보안 엔클레이브(Secure Enclave)와 통신 컨텍스트를 식별하고 자격 증명 및 정책을 제공하는 보안 구성요소(Security Artifact)와 연관된다. 보호된 환경에서 동작하는 노드는 통신에 성공하기 전에 적절한 구성요소와 권한을 보유해야 한다. 이를 통해 하나의 로봇에서 실행된다는 이유만으로 모든 프로세스가 전체 분산 통신 그래프에서 동일한 권한을 가진다고 가정하지 않고 기능적 역할(Functional Role)에 따라 ROS 2 보안을 구성할 수 있다.

보안이 활성화되면 상호운용성(Interoperability)은 더욱 까다로워진다. 두 DDS 구현체가 일반적인 RTPS 트래픽을 정상적으로 교환하더라도 인증 메커니즘, 인증서, 거버넌스 규칙, 권한, 암호화 알고리즘 또는 지원되는 보안 기능이 호환되지 않으면 보안 통신을 설정하지 못할 수 있다. 따라서 혼합 공급업체 보안 배포(Mixed-Vendor Secure Deployment)는 보안되지 않은 DDSI-RTPS 상호운용성이 보안 상호운용성까지 자동으로 보장한다고 가정하지 말고 명시적으로 시험해야 한다.

보안 장애(Security Failure)는 일반적인 통신 장애와 유사하게 나타날 수 있다. 발행자와 구독자가 ROS 2 수준에서 올바르게 설정된 것처럼 보여도 인증이 거부되거나 권한 정책이 해당 토픽을 차단하면 데이터를 교환하지 못할 수 있다. 엔지니어는 진단 과정에서 네트워크 도달 가능성, DDS 탐색, 신원 검증(Identity Validation), 인가, 엔드포인트 매칭(Endpoint Matching), 보호된 데이터 전송을 구분해야 한다. 특히 미들웨어 보안 로그(Security Log)는 패킷 추적만으로는 인증 관계가 거부된 원인을 확인하기 어려울 수 있기 때문에 중요하다.

로깅 및 감사 정보(Logging and Audit Information)는 보안 DDS 시스템의 중요한 운영 구성요소이다. 인증 실패, 거부된 발행 시도, 차단된 구독, 만료된 자격 증명 및 정책 위반을 민감한 키 자료(Key Material)를 노출하지 않으면서 관찰할 수 있어야 한다. 중앙 집중식 모니터링(Centralized Monitoring)은 플릿 운영자가 반복적인 비인가 접근 시도나 설정 오류를 탐지하는 데 도움을 줄 수 있다. 보안 로깅 자체도 진단 정보가 자격 증명이나 보호된 시스템 아키텍처에 관한 과도한 정보를 의도치 않게 노출하지 않도록 신중하게 설계해야 한다.

성능 시험(Performance Testing)에는 정상 조건뿐 아니라 불리한 조건도 포함해야 한다. 엔지니어는 암호화와 인증이 활성화된 상태에서 보안 탐색 시간, 메시지 지연시간, 처리량(Throughput), CPU 사용률, 메모리 사용량 및 재시작 복구를 측정해야 한다. 대용량 암호화 센서 스트림, 많은 보안 참여자, 동시 재연결 및 인증서 검증은 소규모 실험실 시험에서는 나타나지 않는 병목현상을 드러낼 수 있다. 따라서 보안은 전체 통신 아키텍처의 일부로 벤치마킹해야 한다.

다중 로봇 플릿(Multi-Robot Fleet)에서는 보안 정책이 통신 범위(Communication Scope)를 반영해야 한다. 로봇 내부 제어기는 엄격하게 제한된 명령 경로를 필요로 할 수 있으며, 플릿 서비스는 미션, 위치 추정, 진단 또는 선택된 인지 정보에만 접근하도록 구성할 수 있다. 자격 증명을 통해 개별 로봇 또는 기능적 역할을 식별하고 권한을 통해 로봇 간 통신을 제한할 수 있다. VLAN, 방화벽, DDS 도메인 및 통제된 탐색 인프라와 결합하면 여러 개의 상호보완적인 보호 계층을 구성할 수 있다.

DDS 보안은 궁극적으로 배포 직전에 추가하는 단순한 암호화 스위치가 아니라 수명주기 아키텍처(Lifecycle Architecture)로 다루어야 한다. 인증은 신뢰할 수 있는 참여자 신원을 확립하고, 접근 제어는 허용된 DDS 동작을 제한하며, 암호학적 보호는 기밀성과 무결성을 유지한다. 여기에 자격 증명 관리, 최소 권한 정책, 안전한 저장, 모니터링, 상호운용성 시험 및 성능 검증을 결합해야 단일 로봇부터 분산 피지컬 AI 플릿(Distributed Physical AI Fleet)에 이르는 안전한 ROS 2 통신 아키텍처를 완성할 수 있다.

## 02.09 Multi-Domain DDS Architecture Design

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

다중 도메인 DDS 아키텍처(Multi-Domain DDS Architecture)는 모든 참여자(Participant), 토픽(Topic), 데이터 스트림을 하나의 DDS 도메인(Domain)에 배치하는 대신 분산 로봇 시스템을 여러 개의 논리적 통신 공간(Logical Communication Space)으로 분리한다. DDS 도메인은 도메인 ID(Domain ID)로 식별되는 격리된 데이터 공유 환경을 의미한다. 서로 다른 도메인에서 동작하는 참여자는 일반적으로 서로를 직접 탐색하거나 통신하지 않으므로 시스템 설계자는 통신 범위, 탐색 트래픽(Discovery Traffic), 장애 전파(Fault Propagation), 조직적 경계를 제어할 수 있다.

하나의 DDS 도메인은 몇 대의 컴퓨터와 ROS 2 프로세스만 포함하는 소규모 로봇에서는 충분한 경우가 많다. 그러나 시스템이 확장되면 수백 개의 토픽, 많은 참여자, 다수의 센서, 엣지 컴퓨터(Edge Computer), 플릿 서버(Fleet Server), 시뮬레이션 시스템 및 엔지니어링 도구가 동일한 탐색 공간을 공유할 수 있다. 이로 인해 탐색 복잡성이 증가하고 서로 관련 없는 구성요소가 서로를 인식하기 쉬워진다. 다중 도메인 설계는 통신 그래프가 불필요하게 커지기 전에 명시적인 경계를 도입한다.

도메인 분리(Domain Separation)는 임의의 숫자 그룹이 아니라 시스템 아키텍처를 기준으로 구성해야 한다. 로봇 로컬 도메인(Robot-Local Domain)은 높은 주기로 통신해야 하는 인지(Perception), 위치 추정(Localization), 내비게이션(Navigation), 제어(Control), 하드웨어 인터페이스를 포함할 수 있다. 플릿 도메인(Fleet Domain)은 미션 상태, 로봇 상태, 작업 할당(Task Assignment), 선택된 위치 정보를 포함할 수 있다. 엔지니어링 또는 시뮬레이션 도메인을 추가로 분리하여 개발 도구가 운영 로봇 통신에 자동으로 참여하지 않도록 할 수도 있다.

도메인 ID(Domain ID)는 DDS 참여자를 특정 통신 도메인과 연결하는 데 사용되는 논리적 식별자를 제공한다. 서로 다른 도메인 ID로 설정된 참여자는 일반적으로 별도의 탐색 공간에 속한다. ROS 2에서는 ROS 2 통신이 수행되는 도메인을 선택하기 위해 ROS_DOMAIN_ID가 일반적으로 사용된다. 이를 통해 모든 토픽 이름이나 애플리케이션 인터페이스를 다시 설계하지 않고도 ROS 2 프로세스 그룹을 편리하게 분리하여 배포할 수 있다.

도메인 분리는 참여자가 전체 분산 시스템에 존재하는 모든 엔드포인트(Endpoint)를 탐색할 필요가 없도록 하여 탐색 범위(Discovery Scope)를 줄인다. 30대의 로봇 각각에 많은 로컬 DDS 엔드포인트가 존재할 때 모든 로봇을 하나의 제한 없는 도메인에 배치하면 필요 이상으로 큰 탐색 그래프가 생성될 수 있다. 통신 범위를 의도적으로 할당하면 고주기 로봇 내부 엔터티를 로컬에 유지하면서 실제로 로봇 간 가시성이 필요한 인프라에만 선택된 정보를 노출할 수 있다.

따라서 다중 도메인 아키텍처는 로봇 플릿(Robot Fleet)에서 특히 중요하다. 각 로봇은 센서와 제어를 위한 로컬 통신 환경을 유지하고, 별도의 플릿 수준 통신 공간(Fleet-Level Communication Space)을 통해 상대적으로 낮은 대역폭의 운영 정보를 전달할 수 있다. 원시 카메라, 라이다(LiDAR) 포인트 클라우드, 관성 측정 장치(IMU) 스트림, 액추에이터 피드백은 일반적으로 플릿 전체에서 공유할 필요가 없다. 미션 상태, 위치, 시스템 상태, 경보 및 작업 상태는 도메인 간 분산(Cross-Domain Distribution)에 더 적합한 정보이다.

도메인은 네트워크 서브넷(Network Subnet)이나 가상 근거리 통신망(Virtual LAN, VLAN)과 혼동해서는 안 된다. DDS 도메인은 미들웨어 수준 통신 범위(Middleware-Level Communication Scope)인 반면 VLAN과 IP 서브넷은 더 낮은 네트워크 계층에서 통신을 분리한다. 여러 DDS 도메인이 동일한 물리적 이더넷 네트워크에서 동작할 수 있으며, 탐색 및 라우팅 아키텍처가 허용하는 경우 하나의 DDS 도메인이 여러 네트워크 세그먼트에 걸쳐 구성될 수도 있다. 실제 운영 시스템에서는 DDS 수준과 네트워크 수준 세분화를 함께 사용하는 경우가 많다.

도메인 ID는 보안 자격 증명(Security Credential)도 아니다. 도메인에 따라 참여자를 분리하면 의도하지 않은 탐색을 방지하고 불필요한 통신을 줄일 수 있지만, 네트워크에 접근하여 동일한 도메인 ID를 선택할 수 있는 엔터티를 자동으로 신뢰해서는 안 된다. 도메인 경계를 실질적인 보안 또는 신뢰 경계(Trust Boundary)로 사용해야 한다면 인증(Authentication), DDS 보안 권한(DDS Security Permission), 방화벽, VLAN 정책 및 안전한 자격 증명 관리가 필요하다.

서로 다른 도메인의 참여자는 일반적으로 애플리케이션 데이터를 직접 교환하지 않으므로 도메인 간 통신(Cross-Domain Communication)은 의도적으로 도입해야 한다. 브리지(Bridge), 라우터(Router), 게이트웨이(Gateway) 또는 애플리케이션 수준 릴레이(Application-Level Relay)가 한 도메인의 선택된 데이터를 구독하고 다른 도메인으로 재발행하거나 라우팅할 수 있다. 이를 통해 어떤 토픽이 경계를 통과하는지, 어떤 주기와 QoS를 사용하는지, 어떤 보안 정책을 적용하는지를 결정할 수 있는 아키텍처 제어 지점이 형성된다.

DDS 라우터(DDS Router) 또는 유사한 라우팅 구성요소는 격리된 DDS 통신 공간을 연결하면서도 통제된 데이터 교환을 유지할 수 있다. 모든 참여자를 하나의 탐색 도메인으로 통합하는 대신 라우터가 선택된 정보만 도메인 사이에 노출한다. 구현 방식에 따라 라우팅 규칙은 토픽을 필터링하고 통신 경로를 변경하며 서로 다른 네트워크 또는 원격 사이트를 연결할 수 있다. 이를 통해 통신 경계를 명확하게 유지하면서 시스템 수준의 통합을 지원할 수 있다.

ROS 2 애플리케이션은 단순한 투명 라우팅(Transparent Routing)보다 높은 수준의 애플리케이션 의미론이 필요한 경우 게이트웨이 노드(Gateway Node)를 구현할 수도 있다. 게이트웨이는 고주기 로봇 로컬 데이터를 구독하고 필터링 또는 집계(Aggregation)를 수행한 후 압축된 표현을 다른 도메인에 발행할 수 있다. 예를 들어 로컬 인지 시스템이 카메라와 라이다 스트림을 내부에서 처리하고 추적 객체(Tracked Object), 주행 가능 공간(Free-Space Information), 경보만 플릿 도메인으로 전달하면 대역폭과 아키텍처 결합도(Coupling)를 모두 줄일 수 있다.

데이터가 도메인 경계를 통과할 때는 서비스 품질(Quality of Service, QoS)을 고려해야 한다. 로컬 최선형(BEST_EFFORT) 센서 스트림이 게이트웨이에 의해 재발행된다는 이유만으로 자동으로 플릿 전체의 고주기 신뢰성(RELIABLE) 스트림으로 변환되어서는 안 된다. 게이트웨이는 목적지 통신 범위에 적합한 QoS를 의도적으로 지정할 수 있다. 신뢰성(Reliability), 지속성(Durability), 이력 깊이(History Depth), 수명(Lifespan), 데드라인(Deadline), 발행 주기는 경계 양쪽의 데이터 의미와 네트워크 특성을 반영해야 한다.

대역폭 제어(Bandwidth Control)는 도메인 게이트웨이를 사용하는 가장 강력한 이유 중 하나이다. 로봇 내부 이더넷은 고주기 인지 트래픽을 처리할 수 있지만 로봇과 플릿 인프라 사이에서 공유되는 Wi-Fi 또는 5G 연결은 용량이 훨씬 제한적이고 변동성이 클 수 있다. 게이트웨이는 이러한 링크를 통과하기 전에 데이터를 다운샘플링(Downsampling), 압축(Compression), 필터링(Filtering), 집계 또는 이벤트 기반(Event-Triggered) 형태로 변환할 수 있다. 따라서 다중 도메인 아키텍처는 데이터 흐름 경계를 명확하게 만들어 DDS 대역폭 최적화를 보완한다.

장애 격리(Fault Containment)는 또 다른 아키텍처상의 이점이다. 개발 도구, 실험적 알고리즘 또는 시뮬레이션 참여자가 별도의 도메인에서 동작하면 예상하지 못한 엔드포인트 생성이나 탐색 폭주(Discovery Storm)가 운영 통신에 직접 영향을 미칠 가능성이 줄어든다. 마찬가지로 하나의 통신 범위에서 많은 참여자가 재시작되더라도 격리된 다른 도메인의 모든 참여자가 동일한 탐색 변화를 처리할 필요는 없다. 따라서 도메인 경계는 운영 예측 가능성(Operational Predictability)을 향상시킬 수 있다.

다중 도메인 아키텍처는 개발(Development), 통합(Integration), 시험(Test), 운영(Production) 환경 사이의 수명주기 분리(Lifecycle Separation)도 지원한다. 시뮬레이션 시스템은 전용 도메인을 사용하고 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 시스템은 별도의 통신 범위를 사용할 수 있다. 엔지니어는 진단 또는 실험용 ROS 2 그래프를 운영 로봇에 자동으로 연결하지 않고 실행할 수 있다. 서로 분리되어야 하는 환경에 팀이 실수로 중복된 ID를 할당하지 않도록 도메인 번호 규칙(Domain Numbering Convention)을 중앙에서 문서화해야 한다.

대형 로봇은 플릿 통합 이전 단계에서도 여러 도메인을 사용하는 것이 유용할 수 있다. 복잡한 자율 플랫폼에는 안전 관련 제어, 고대역폭 인지, 미션 컴퓨팅(Mission Computing), 페이로드 시스템 및 외부 인터페이스가 포함될 수 있다. 이러한 서브시스템은 서로 다른 통신 요구사항과 신뢰 관계를 가진다. 이를 아키텍처 통신 영역(Communication Zone)으로 분리하면 불필요한 결합을 줄일 수 있지만, 과도한 도메인 분할은 게이트웨이 복잡성을 증가시키므로 명확한 시스템 경계에 의해 정당화되어야 한다.

도메인 설계에는 통신 매트릭스(Communication Matrix)가 함께 구성되어야 한다. 각 도메인에 대해 설계자는 참여자, 주요 토픽, 예상 페이로드 크기, 발행 주기, QoS, 네트워크 인터페이스 및 허용된 도메인 간 데이터 흐름을 정의해야 한다. 또한 각 경계를 담당하는 게이트웨이가 무엇인지와 정보가 투명하게 전달되는지, 필터링되는지, 집계되는지, 변환되는지 또는 차단되는지를 명시해야 한다. 이를 통해 도메인 할당을 비공식적인 설정 선택이 아니라 통제된 시스템 아키텍처로 전환할 수 있다.

도메인이 격리를 제공하더라도 명명 규칙(Naming Convention)은 여전히 중요하다. 두 도메인은 동일한 이름의 토픽을 포함하면서 서로 직접 통신하지 않을 수 있으며, 이는 반복되는 로봇 로컬 아키텍처에서 유용하다. 예를 들어 각 로봇은 내부적으로 동일한 센서 및 제어 토픽 이름을 사용할 수 있다. 이후 게이트웨이가 선택된 로봇 정보를 플릿 수준 네임스페이스(Namespace) 또는 식별자로 매핑하여 플릿 시스템이 Robot A, Robot B 및 다른 플랫폼에서 발생한 데이터를 구분하도록 할 수 있다.

다중 로봇 시스템에서는 각 로봇에 고유한 도메인을 할당할지 또는 여러 로봇이 도메인을 공유하도록 할지를 신중하게 결정해야 한다. 로봇별 고유 도메인은 강력한 통신 격리를 제공하지만 라우팅 및 설정 요구사항을 증가시킨다. 공유 로봇 도메인은 직접적인 피어 통신(Peer Communication)을 단순화하지만 탐색 범위를 확대한다. 적절한 모델은 플릿 규모, 로봇 간 통신 요구사항, 네트워크 토폴로지(Network Topology), 탐색 인프라 및 실제로 공유되어야 하는 데이터의 양에 따라 결정된다.

따라서 탐색 아키텍처(Discovery Architecture)와 도메인 아키텍처는 함께 설계해야 한다. 시스템을 여러 도메인으로 분할하면 탐색 범위가 감소하며, 탐색 서버(Discovery Server), 설정된 피어(Configured Peer) 또는 기타 미들웨어별 메커니즘을 사용하면 각 도메인 내부에서 참여자가 서로를 찾는 방식을 추가로 제어할 수 있다. 도메인은 누가 동일한 논리적 데이터 공간에 속하는지를 결정하고, 탐색 설정은 해당 참여자들이 물리적 네트워크를 통해 어떻게 서로의 존재를 인식하는지를 결정한다.

다중 도메인 시스템에서는 하나의 참여자가 전체 DDS 그래프를 자동으로 볼 수 없기 때문에 모니터링(Monitoring)이 더욱 복잡해진다. 따라서 운영 관찰 가능성(Operational Observability)을 명시적으로 설계해야 한다. 모니터링 서비스는 통제된 게이트웨이를 통해 연결하거나 도메인별 에이전트(Domain-Specific Agent)를 배치하거나 일반 애플리케이션 데이터 경로 외부에서 메트릭을 수집할 수 있다. 통신 장애를 올바른 아키텍처 경계까지 추적할 수 있도록 로그에는 도메인 ID, 참여자 신원, 게이트웨이 경로 및 네트워크 인터페이스를 식별할 수 있는 정보가 포함되어야 한다.

문제 해결(Troubleshooting) 역시 동일한 경계를 따라 수행해야 한다. 엔지니어는 먼저 통신하려는 엔드포인트가 의도적으로 동일한 도메인에 위치하는지 또는 승인된 게이트웨이를 통해 연결되는지를 확인한다. 이후 탐색, 토픽 및 타입 호환성(Type Compatibility), QoS, 라우팅 규칙, 네트워크 도달 가능성 및 보안 정책을 검사한다. 다중 도메인 아키텍처에서 특정 토픽이 보이지 않는 것은 해당 토픽이 의도적으로 도메인 경계를 통과하지 못하도록 설계되었다면 장애가 아니라 정상 동작일 수 있다.

보안 정책(Security Policy)은 도메인 자체에만 의존하지 않으면서 도메인 아키텍처와 정렬되어야 한다. 로봇 제어 도메인은 제한적인 DDS 권한을 적용할 수 있으며, 플릿 도메인은 지정된 서비스만 미션 명령을 발행하도록 허용할 수 있다. 엔지니어링 도메인은 게이트웨이를 통해 읽기 전용 텔레메트리(Read-Only Telemetry)를 수신하면서 제어 경로에는 접근하지 못하도록 할 수 있다. 도메인 격리, 최소 권한 DDS 보안(Least-Privilege DDS Security), 네트워크 세분화 및 명시적인 라우팅을 결합하면 분산 로봇 시스템에 심층 방어(Defense in Depth)를 구성할 수 있다.

피지컬 AI(Physical AI) 아키텍처에서는 도메인 경계를 컴퓨팅 계층(Computing Hierarchy)에 대응시킬 수도 있다. 엣지 컴퓨터는 로봇 로컬 도메인에서 고주기 센서 및 제어 데이터를 교환하고, 온프레미스 시스템(On-Premise System)은 선택된 의미 정보와 운영 정보를 수신하며, 클라우드 시스템(Cloud System)은 분석, 모델 관리 또는 장기 학습을 위해 더욱 집계된 데이터를 수신할 수 있다. 이러한 계층 사이의 게이트웨이는 대역폭, 보안, 데이터 소유권(Data Ownership), 운영 권한(Operational Authority)을 관리하는 정책 집행 지점(Policy Enforcement Point)이 된다.

확장 가능한 설계는 조직 전체를 하나의 제한 없는 도메인에 배치하는 방식과 복잡한 라우팅 의존성을 가진 지나치게 많은 도메인으로 세분화하는 방식이라는 두 극단을 모두 피해야 한다. 목표는 물리적 시스템, 운영 책임, 네트워크 제약 및 신뢰 경계에 대응하는 소수의 의미 있는 통신 영역을 구성하는 것이다. 추가되는 각 도메인은 확장성, 격리, 대역폭 제어, 수명주기 관리 또는 보안 아키텍처 측면에서 명확한 이점을 제공해야 한다.

다중 도메인 DDS 아키텍처는 궁극적으로 도메인 설정을 단순한 숫자 설정에서 시스템 수준 설계 메커니즘(System-Level Design Mechanism)으로 전환한다. 도메인은 통신 범위를 정의하고, 게이트웨이는 통제된 정보 교환을 정의하며, QoS는 전달 동작을 정의하고, 보안은 승인된 참여를 정의한다. 이러한 요소를 함께 설계하면 ROS 2와 DDS를 단일 자율 로봇에서 분산 플릿, 엣지 인프라, 시뮬레이션 환경 및 대규모 피지컬 AI 시스템까지 확장하면서 모든 참여자에게 모든 데이터 스트림을 노출하지 않는 구조를 구현할 수 있다.

## 02.10 DDS Network Diagnostics: Wireshark DDS Analysis [w/Code]

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

DDS 네트워크 진단(DDS Network Diagnostics)은 애플리케이션 증상과 미들웨어 및 네트워크 동작을 구분하는 것에서 시작해야 한다. ROS 2 노드는 데이터 누락, 메시지 지연, 불안정한 탐색(Discovery), 간헐적인 통신과 같은 문제를 보고할 수 있지만 이러한 증상만으로는 어느 계층에서 장애가 발생했는지 즉시 판단할 수 없다. 효과적인 진단은 ROS 2 애플리케이션 동작에서 시작하여 rmw, DDS, RTPS, UDP/IP, 이더넷(Ethernet) 또는 무선 전송 계층을 거쳐 최종 수신 엔드포인트(Receiving Endpoint)까지 전체 통신 경로를 따라 수행해야 한다.

와이어샤크(Wireshark)는 DDS 구현체가 일반적으로 UDP 기반의 DDSI-RTPS 와이어 프로토콜(Wire Protocol)을 사용하여 데이터를 교환하기 때문에 특히 유용하다. 패킷 캡처(Packet Capture)를 사용하면 ROS 2 애플리케이션과 독립적으로 탐색 메시지, 사용자 데이터 트래픽, 신뢰성 제어 메시지, 단편화(Fragmentation), 재전송(Retransmission), 멀티캐스트(Multicast) 동작 및 네트워크 주소를 관찰할 수 있다. 따라서 소프트웨어 구성요소가 전송했다고 가정하는 정보가 아니라 실제 네트워크에서 전송되는 정보를 직접 확인할 수 있다.

패킷을 캡처하기 전에 엔지니어는 관련 네트워크 인터페이스(Network Interface)와 통신 토폴로지(Communication Topology)를 파악해야 한다. 하나의 로봇 컴퓨터에는 이더넷, Wi-Fi, VPN, 컨테이너 브리지(Container Bridge), 가상 인터페이스 및 여러 개의 물리적 네트워크 인터페이스 카드(Network Interface Card, NIC)가 존재할 수 있다. 잘못된 인터페이스를 캡처하면 DDS 트래픽이 존재하지 않는 것처럼 잘못 판단할 수 있다. 따라서 상세한 패킷 분석을 시작하기 전에 인터페이스 주소, 라우팅 테이블, DDS 전송 설정, 도메인 ID(Domain ID), 예상 원격 엔드포인트를 기록해야 한다.

RTPS 패킷은 DDS 분석에 중요한 구조적 정보를 제공한다. RTPS 메시지는 프로토콜 헤더(Protocol Header)와 그 뒤에 이어지는 하나 이상의 서브메시지(Submessage)로 구성된다. 와이어샤크는 많은 표준 RTPS 구조를 해석하여 프로토콜 버전, 공급업체 정보(Vendor Information), GUID 접두부(GUID Prefix), 엔터티 ID(EntityId), 시퀀스 번호(Sequence Number), 로케이터(Locator), 서브메시지 유형 등의 필드를 표시할 수 있다. 이러한 필드는 어떤 DDS 참여자(Participant)가 트래픽을 생성했는지와 특정 패킷이 어떤 역할을 수행하는지를 파악하는 데 도움을 준다.

탐색 분석(Discovery Analysis)은 일반적으로 SPDP 트래픽에서 시작한다. 단순 참여자 탐색 프로토콜(Simple Participant Discovery Protocol, SPDP)은 DDS 참여자가 자신의 존재를 알리고 GUID와 네트워크 로케이터 같은 참여자 수준 정보를 교환할 수 있도록 한다. 두 ROS 2 시스템이 서로를 탐색하지 못한다면 먼저 SPDP 알림이 실제로 전송되는지, 해당 패킷이 원격 네트워크에 도달하는지, 반대 방향의 탐색 트래픽이 관찰되는지를 확인해야 한다.

참여자 탐색 이후에는 SEDP 분석을 수행하며, 이는 엔드포인트 정보에 초점을 맞춘다. 단순 엔드포인트 탐색 프로토콜(Simple Endpoint Discovery Protocol, SEDP)은 토픽(Topic), 타입(Type), QoS 특성 및 엔드포인트 식별자를 포함한 데이터 작성자(DataWriter)와 데이터 판독자(DataReader)의 정보를 전달한다. 따라서 SPDP는 성공하지만 애플리케이션 통신이 이루어지지 않는다면 참여자 탐색은 정상적으로 작동하지만 엔드포인트 탐색 또는 매칭(Endpoint Matching)에 문제가 있을 수 있다. SEDP 트래픽을 검사하면 장애가 발생한 단계를 보다 정확하게 좁힐 수 있다.

DDS 탐색은 기본 설정에서 멀티캐스트에 의존하는 경우가 많기 때문에 멀티캐스트 동작(Multicast Behavior)은 중요한 진단 대상이다. 와이어샤크를 사용하면 멀티캐스트 패킷이 송신 측에서 실제로 전송되는지, 어떤 멀티캐스트 주소와 UDP 포트를 사용하는지, 원격 호스트가 이를 수신하는지를 확인할 수 있다. 멀티캐스트 트래픽이 보이지 않는다면 ROS 2 노드 자체의 장애보다는 스위치 설정, IGMP 스누핑(IGMP Snooping), VLAN 경계, Wi-Fi 동작, 방화벽 규칙, DDS 설정 또는 잘못된 네트워크 인터페이스 선택이 원인일 수 있다.

탐색을 통해 참여자 로케이터가 설정된 이후에는 유니캐스트(Unicast) 트래픽을 검사해야 한다. DDS 구현체는 엔드포인트 통신과 신뢰성 메시지 교환에 유니캐스트를 일반적으로 사용한다. 엔지니어는 출발지와 목적지 IP 주소, UDP 포트, 패킷 방향 및 인터페이스 선택을 확인해야 한다. 여러 NIC, 컨테이너, VPN 인터페이스 또는 잘못된 라우팅 때문에 참여자가 도달할 수 없는 로케이터를 광고하면 탐색은 성공한 것처럼 보이지만 애플리케이션 데이터는 실제 수신자에게 도달하지 않을 수 있다.

RTPS DATA 서브메시지는 애플리케이션 데이터 분석에서 중요한 부분을 차지한다. 엔지니어는 작성자(Writer) 및 판독자(Reader) 식별자, 시퀀스 번호, 페이로드 크기(Payload Size), 사용 가능한 경우 타임스탬프(Timestamp), 패킷 주기를 검사할 수 있다. 관찰된 패킷 주기와 예상 ROS 2 발행 주기(Publication Rate)를 비교하면 데이터가 일관되게 생성되는지를 판단할 수 있다. 누락된 시퀀스 번호 또는 불규칙한 패킷 간격은 손실, 스케줄링 지연, 혼잡 또는 추가 조사가 필요한 미들웨어 수준 동작을 나타낼 수 있다.

신뢰성 DDS 통신(Reliable DDS Communication)은 추가적인 진단 신호를 제공한다. 하트비트(HEARTBEAT) 메시지는 판독자에게 작성자의 시퀀스 번호 상태를 알려주며, 확인 및 부정 확인 응답(ACKNACK) 메시지는 판독자가 수신 상태를 알리거나 누락된 샘플을 요청할 수 있도록 한다. 반복적인 ACKNACK 동작, 빈번한 부정 확인 응답 또는 재전송되는 DATA 패킷은 패킷 손실이나 수신자 처리 압력(Receiver Pressure)을 나타낼 수 있다. 따라서 신뢰성 트래픽은 네트워크가 정상적으로 동작하는 것처럼 보이면서도 지연시간이나 처리량이 불안정한 문제를 진단하는 중요한 증거가 된다.

최선형(BEST_EFFORT) 통신은 손실된 샘플이 일반적으로 신뢰성 재전송을 통해 복구되지 않기 때문에 다른 방식으로 동작한다. 패킷 캡처에서는 시퀀스 번호의 공백이 관찰되더라도 이후 복구 트래픽이 나타나지 않을 수 있다. 빠르게 갱신되는 센서 스트림에서는 이것이 프로토콜 장애가 아니라 정상적인 동작일 수 있다. 따라서 엔지니어는 누락된 패킷, 재전송 또는 확인 응답 트래픽의 부재를 해석하기 전에 분석 대상 토픽의 QoS 프로파일을 파악해야 한다.

대용량 ROS 2 메시지는 DDS/RTPS가 직렬화된 샘플을 여러 부분으로 단편화할 수 있기 때문에 특별한 주의가 필요하다. 카메라 영상, 포인트 클라우드(Point Cloud), 점유 지도(Occupancy Map) 및 기타 대용량 페이로드는 하나의 논리적 샘플을 구성하는 여러 단편이 포함된 DATA_FRAG 트래픽을 생성할 수 있다. 와이어샤크를 이용하면 단편 크기, 단편 수, 패킷 손실 및 반복되는 단편 전송을 확인할 수 있다. 대용량 토픽에서만 발생하는 문제는 탐색 장애보다 단편화, MTU, 버퍼링, 대역폭 또는 신뢰성 문제를 나타내는 경우가 많다.

패킷 크기는 네트워크 최대 전송 단위(Maximum Transmission Unit, MTU) 및 DDS 단편화 설정과 비교해야 한다. 이더넷은 일반적으로 약 1500바이트의 MTU를 사용하지만 통제된 네트워크에서는 더 큰 점보 프레임(Jumbo Frame)을 설정할 수 있다. IP 단편화(IP Fragmentation)와 RTPS 수준 단편화(RTPS-Level Fragmentation)는 서로 다른 메커니즘이므로 혼동해서는 안 된다. 컴퓨터, 스위치, 가상 네트워크 및 미들웨어 전송 계층에서 MTU 설정을 일관되게 유지하면 진단하기 어려운 대용량 메시지 통신 장애를 방지하는 데 도움이 된다.

대역폭 분석(Bandwidth Analysis)에서는 초당 바이트(Bytes per Second)와 초당 패킷(Packets per Second)을 모두 고려해야 한다. 네트워크가 충분한 명목 비트 전송률을 가지고 있더라도 많은 수의 작은 패킷이 생성되면 과도한 패킷 처리 부하가 발생할 수 있다. 반대로 고해상도 센서 스트림은 매우 큰 전체 페이로드로 링크 용량을 소비할 수 있다. 와이어샤크 통계와 운영체제 카운터를 이용하면 주요 통신 흐름, 트래픽 버스트(Traffic Burst), 네트워크 용량 한계에 접근하는 구간을 식별할 수 있다.

통신 자체는 이루어지지만 지연시간이 허용 범위를 벗어나는 경우에는 타이밍 분석(Timing Analysis)이 필요하다. 캡처 타임스탬프를 사용하면 패킷 간격, 버스트, 재전송 지연 및 통신 중단 구간을 분석할 수 있다. 그러나 참여 컴퓨터의 시계가 충분히 동기화되지 않았다면 서로 독립적인 패킷 캡처만으로 단방향 지연시간(One-Way Latency)을 정확하게 계산할 수 없다. 따라서 엄밀한 분산 지연시간 측정에는 정밀 시간 프로토콜(Precision Time Protocol, PTP), 네트워크 시간 프로토콜(Network Time Protocol, NTP), 하드웨어 타임스탬핑(Hardware Timestamping) 또는 검증된 다른 동기화 방법이 중요하다.

지터(Jitter)는 평균 지연시간과 별도로 평가해야 한다. 제어 또는 인지 파이프라인은 대부분 3ms에 전달되지만 가끔 100ms가 소요되는 통신 경로보다 안정적으로 10ms의 통신 지연이 발생하는 경로를 더 잘 수용할 수 있다. 패킷 캡처를 통해 이러한 이상값과 연관된 트래픽 버스트, 지연된 확인 응답, 재전송 구간 또는 스케줄링 공백을 확인할 수 있다. 따라서 단일 평균값보다 백분위 지연시간(Percentile Latency)과 관찰된 최대 지연시간이 더 유용한 경우가 많다.

패킷 손실(Packet Loss)은 여러 위치에서 발생할 수 있으므로 패킷 캡처 결과를 신중하게 해석해야 한다. 캡처에서 특정 패킷이 누락되었다고 해서 물리적 네트워크에서 해당 패킷이 손실되었다고 단정할 수는 없다. 캡처 버퍼, 운영체제 스케줄링 또는 과부하된 모니터링 시스템 자체에서도 패킷이 누락될 수 있기 때문이다. 송신자와 수신자 양쪽에서 캡처한 결과를 비교하는 것이 더 강력한 방법이며, 송신 측에서는 확인되지만 수신 측에서 지속적으로 보이지 않는 패킷은 네트워크 경로에서 발생한 손실에 대한 더 명확한 증거가 된다.

DDS 포트 동작(DDS Port Behavior)도 문제 해결에 활용할 수 있다. RTPS는 일반적으로 도메인 ID와 참여자 관련 값 등의 매개변수에 영향을 받는 표준화된 포트 매핑 규칙(Port-Mapping Rule)을 기반으로 UDP 포트를 결정하지만 구현체 설정에 따라 실제 동작은 달라질 수 있다. 특정 UDP 범위만 허용하는 방화벽은 예상하지 못하게 탐색 또는 사용자 트래픽을 차단할 수 있다. 패킷 캡처는 실제 사용되는 포트를 보여주므로 기본 공식만을 근거로 추정하는 것보다 실제 캡처 결과를 우선해야 한다.

ROS 2 명령줄 진단(Command-Line Diagnostics)은 와이어샤크를 대체하는 것이 아니라 보완한다. 노드, 토픽, 발행자, 구독자, 전송 주기 및 대역폭을 검사하는 명령은 ROS 그래프의 애플리케이션 수준 관점을 제공한다. DDS 로그와 미들웨어별 진단 도구는 또 다른 계층의 정보를 제공하며, 와이어샤크는 실제 와이어 동작(Wire Behavior)을 관찰한다. 이러한 관점을 비교하면 장애가 DDS 상위 계층에서 발생했는지, DDS 탐색이나 QoS 매칭 내부에서 발생했는지, 또는 미들웨어 아래의 네트워크에서 발생했는지를 판단할 수 있다.

QoS 비호환성(QoS Incompatibility)은 네트워크 패킷이 존재하는데도 ROS 2 데이터가 전달되지 않는 대표적인 경우이다. 참여자들은 서로를 탐색할 수 있지만 제공 QoS(Offered QoS)와 요청 QoS(Requested QoS)가 호환되지 않으면 엔드포인트가 정상적으로 매칭되지 않을 수 있다. 따라서 신뢰성(Reliability), 지속성(Durability), 데드라인(Deadline) 및 기타 정책을 패킷 추적 결과와 함께 확인해야 한다. 탐색 패킷은 확인되지만 예상되는 데이터가 존재하지 않는다면 기본 네트워크 연결보다 엔드포인트 호환성에 우선적으로 주의를 기울일 수 있다.

DDS 보안(DDS Security)이 적용되면 보호된 통신에서 애플리케이션 페이로드 또는 선택된 메타데이터가 수동적인 패킷 분석에서 보이지 않을 수 있으므로 분석 방식도 달라진다. 와이어샤크는 여전히 주소, 패킷 타이밍, 크기 및 일부 프로토콜 교환에 관한 유용한 정보를 제공할 수 있지만 암호화된 필드는 적절한 접근 권한과 도구 없이는 해석할 수 없다. 따라서 인증(Authentication) 또는 접근 제어(Access Control) 장애는 패킷 내용만으로 진단하기보다 DDS 보안 로그와 연계하여 분석해야 한다.

다중 도메인 DDS 시스템(Multi-Domain DDS System)의 진단에서는 아키텍처 경계를 고려해야 한다. 서로 다른 도메인 ID에 속한 참여자는 라우터, 브리지 또는 게이트웨이가 의도적으로 통신 공간을 연결하지 않는 한 서로를 직접 탐색할 것으로 기대해서는 안 된다. 탐색 트래픽이 보이지 않는 것을 장애로 판단하기 전에 패킷 분석을 통해 도메인, 인터페이스, 게이트웨이 및 예상 경로를 확인해야 한다. 도메인 간 전달 문제를 진단할 때는 게이트웨이 양쪽에서 트래픽을 캡처해야 한다.

반복 가능한 진단 절차(Repeatable Diagnostic Workflow)는 설정을 변경하기 전에 증거를 수집해야 한다. 엔지니어는 ROS_DOMAIN_ID, rmw 구현체, DDS 공급업체, QoS 프로파일, 네트워크 인터페이스, IP 주소, MTU, 라우팅 정보, 방화벽 상태 및 패킷 캡처 결과를 기록해야 한다. 여러 매개변수를 동시에 변경하면 통신이 일시적으로 복구될 수 있지만 원래 장애에 관한 증거를 잃게 된다. 통제된 실험(Controlled Experiment)을 사용하면 DDS 네트워크 문제를 재현할 수 있고 장애 원인을 훨씬 쉽게 분리할 수 있다.

운영 모니터링(Production Monitoring)은 일시적인 와이어샤크 캡처를 넘어 지속적으로 수행되어야 한다. DDS 기반 로봇은 탐색 상태, 메시지 전송 주기, 대역폭, 패킷 손실, 재전송, 지연시간, 지터, CPU 부하 및 네트워크 인터페이스 오류를 지속적으로 측정함으로써 이점을 얻을 수 있다. 정상 운영 상태에서 기록한 기준값(Baseline Value)은 성능 저하를 탐지하기 위한 기준을 제공한다. 이후 모니터링에서 비정상적인 동작이 감지될 때 선택적으로 패킷 캡처를 활성화하면 대량의 정상 트래픽을 항상 분석할 필요가 줄어든다.

효과적인 DDS 네트워크 진단은 궁극적으로 여러 계층의 정보를 상호 연관시키는 과정이 필요하다. ROS 2 그래프 정보는 의도된 통신 구조를 설명하고, DDS와 RTPS는 탐색 및 데이터 전달 메커니즘을 보여주며, UDP/IP와 이더넷은 전송 동작을 나타내고, 와이어샤크는 이러한 계층을 연결하는 패킷 수준 증거(Packet-Level Evidence)를 제공한다. 탐색부터 시작하여 엔드포인트 매칭, 데이터 흐름, 신뢰성, 단편화, 타이밍 및 네트워크 상태를 순차적으로 분석하면 시행착오식 설정 변경에 의존하지 않고 통신 장애를 체계적으로 분리하고 진단할 수 있다.
