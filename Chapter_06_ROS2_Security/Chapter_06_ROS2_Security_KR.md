**Volume 05 Robot Middleware and ROS2**

# 06. ROS2 Security

## 06.01 SROS2 Security Architecture Overview

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

SROS2는 ROS 2 통신 아키텍처의 보안 기능을 제공하고 관리하기 위해 사용되는 보안 프레임워크(Security Framework)이다. SROS2는 독립적인 보안 전송 계층(Security Transport Layer)을 새로 도입하는 대신 ROS 2 하위의 DDS 보안(DDS Security) 표준을 기반으로 하며, 신원 생성(Identity Creation), 보안 구성(Security Configuration), 접근 제어 정책(Access-Control Policy) 생성 및 보안 산출물(Security Artifacts) 배포를 위한 ROS 지향 도구를 제공한다. 목적은 일반적인 ROS 2 개발 및 운영 환경에서 안전한 분산 로봇 통신(Secure Distributed Robot Communication)을 실용적으로 구현하는 것이다.

SROS2의 아키텍처적 위치는 ROS 2의 계층 구조(Layered Structure)와 밀접하게 연관된다. 애플리케이션 노드(Application Node)는 rclcpp 또는 rclpy와 같은 클라이언트 라이브러리(Client Library)를 통해 통신하며, 이들은 ROS 미들웨어 인터페이스(ROS Middleware Interface)와 하위 DDS 구현체(DDS Implementation)를 사용한다. 실제 보안 집행(Security Enforcement)은 DDS 보안 메커니즘(DDS Security Mechanism)에서 수행되며, SROS2는 ROS 2 개발자가 노드(Node), 토픽(Topic), 서비스(Service), 액션(Action)과 같은 ROS 개념을 사용하여 이러한 메커니즘을 구성하도록 지원한다.

이 아키텍처의 기본 원칙은 참여하는 모든 ROS 2 프로세스(Process)가 검증 가능한 디지털 신원(Digital Identity)을 보유해야 한다는 것이다. 신원은 일반적으로 신뢰할 수 있는 공개 키 기반 구조(Public Key Infrastructure)에서 발급된 인증서(Certificate)와 개인 키(Private Key)를 통해 표현된다. 참여자(Participant)가 서로를 발견하면 인증 메커니즘(Authentication Mechanism)이 보호된 통신을 설정하기 전에 해당 참여자가 예상된 보안 환경(Security Environment)에 속하는지 검증할 수 있다. 이를 통해 ROS 2 디스커버리(Discovery)는 단순한 참여자 탐색에서 인증된 분산 시스템 멤버십(Authenticated Distributed-System Membership)으로 확장된다.

신뢰 기반(Trust Foundation)은 일반적으로 보안 엔클레이브(Security Enclave) 구조와 인증 기관(Certificate Authority)을 중심으로 구성된다. 인증 기관(CA)은 참여자 신원과 권한을 파생할 수 있는 신뢰 루트(Root of Trust)를 형성한다. 보안 산출물(Security Artifacts)은 키스토어(Keystore)에 저장되고 승인된 로봇 컴퓨터 또는 프로세스에 배포된다. 유효한 자격 증명(Credential)을 소유한 소프트웨어는 보호된 ROS 2 통신 도메인(Communication Domain)에 참여할 수 있으므로 이러한 보안 자료를 보호하는 것이 매우 중요하다.

인증(Authentication)만으로는 인증된 참여자가 무엇을 수행할 수 있는지 결정할 수 없다. 따라서 SROS2는 DDS 보안 접근 제어(DDS Security Access Control)를 통해 신원(Identity)과 권한 부여(Authorization)를 결합한다. 권한(Permission)은 도메인(Domain), 토픽(Topic), 파티션(Partition) 및 기타 DDS 엔티티(Entity)에 따라 통신을 제한할 수 있다. ROS 2 수준에서는 이러한 제어를 애플리케이션 통신 요구사항에 대응시켜 인지 노드(Perception Node), 내비게이션 구성요소(Navigation Component), 액추에이터 인터페이스(Actuator Interface), 진단 프로세스(Diagnostic Process)가 자신의 기능에 필요한 권한만 갖도록 설계할 수 있다.

이러한 아키텍처는 최소 권한 원칙(Principle of Least Privilege)을 지원한다. 예를 들어 카메라 처리 구성요소(Camera-Processing Component)는 영상 데이터를 수신하고 인지 결과를 발행할 권한은 필요하지만 속도 명령(Velocity Command)을 발행할 권한까지 자동으로 가져서는 안 된다. 마찬가지로 모니터링 애플리케이션(Monitoring Application)은 진단 정보에 대한 읽기 권한은 필요할 수 있지만 제어 파라미터(Control Parameter)를 변경할 권한은 필요하지 않을 수 있다. 따라서 보안 정책(Security Policy)은 단순한 네트워크 수준의 보호 장치가 아니라 소프트웨어 아키텍처(Software Architecture)의 일부가 된다.

기밀성(Confidentiality)과 무결성 보호(Integrity Protection)는 또 다른 주요 아키텍처 계층을 구성한다. DDS 보안 암호화 플러그인(DDS Security Cryptographic Plugin)은 메시지 교환을 보호하여 승인되지 않은 관찰자가 보호된 페이로드(Payload)를 쉽게 확인하지 못하게 하고 악의적인 변경을 탐지할 수 있도록 한다. 구성에 따라 통신 데이터와 일부 프로토콜 요소(Protocol Element)에 보호 기능을 적용할 수 있다. 따라서 모든 통신 경로를 동일하게 처리하기보다 어떤 정보에 암호화(Encryption), 인증(Authentication), 무결성 보호(Integrity Protection) 또는 이들의 조합이 필요한지를 결정해야 한다.

SROS2 보안이 특히 중요한 이유는 ROS 2 디스커버리(Discovery)가 동적인 분산 그래프(Dynamic Distributed Graph)를 생성하기 때문이다. 노드는 시작되거나 종료될 수 있고, 서로 다른 컴퓨터로 이동하거나 여러 네트워크 인터페이스(Network Interface)를 통해 통신할 수 있으며, DDS는 사용 가능한 참여자와 엔드포인트(Endpoint)를 자동으로 발견한다. 보안 제어가 없다면 이러한 유연성이 공격 표면(Attack Surface)을 확대할 수 있다. 반면 인증된 참여자와 명시적인 권한을 사용하면 디스커버리의 동적 특성을 유지하면서도 배포된 신뢰 모델(Trust Model)에 따라 멤버십과 통신 권한을 제한할 수 있다.

따라서 실제 SROS2 배포(Deployment)는 단일 암호화 기능이 아니라 여러 보안 기능이 협력하는 구조로 이해해야 한다. 신원(Identity)은 참여자가 누구인지를 정의하고, 인증(Authentication)은 해당 신원을 검증하며, 접근 제어(Access Control)는 허용되는 통신을 결정한다. 암호화 보호(Cryptographic Protection)는 선택된 데이터 교환을 보호하고, 거버넌스 규칙(Governance Rules)은 보다 광범위한 보안 동작을 정의한다. 키스토어(Keystore), 인증서(Certificate), 권한(Permission), 거버넌스 파일(Governance File), 런타임 환경 구성(Runtime Environment Configuration)이 결합되어 이러한 기능을 실제 ROS 2 시스템에 적용한다.

보안 엔클레이브(Security Enclave)는 애플리케이션 아키텍처(Application Architecture)와 자격 증명 관리(Credential Management) 사이에서 중요한 추상화(Abstraction)를 제공한다. 모든 ROS 2 노드가 항상 완전히 독립적인 보안 구성을 가져야 한다고 가정하는 대신 실행 모델(Execution Model)에 적합한 보안 경계(Security Boundary)에 따라 프로세스를 구성할 수 있다. 엔클레이브 개념은 노드가 공통 프로세스로 컴포지션(Composition)되거나 컨테이너(Container)에 배포되거나 서브시스템(Subsystem)별로 그룹화되는 경우 실제 운영상의 신뢰 경계를 반영할 수 있다는 점에서 특히 유용하다.

이는 노드 아키텍처(Node Architecture)와 프로세스 아키텍처(Process Architecture)를 함께 고려해야 한다는 의미이기도 하다. ROS 2 컴포지션(Composition)은 통신 오버헤드(Communication Overhead)를 줄이기 위해 여러 구성요소를 하나의 프로세스에 배치할 수 있지만, 동일 프로세스를 공유하는 구성요소는 보안 컨텍스트(Security Context)도 공유할 수 있다. 따라서 논리적인 노드 분리가 자동으로 동일 수준의 보안 경계를 제공한다고 가정해서는 안 된다. 성능 중심의 컴포지션 결정과 보안 중심의 격리(Isolation) 결정은 시스템 아키텍처 설계 과정에서 함께 평가해야 한다.

또한 SROS2는 완전한 로봇 사이버보안(Robot Cybersecurity)과 구분해야 한다. SROS2는 중요한 ROS 2 및 DDS 통신 경로를 보호하지만 운영체제(Operating System), 부트 체인(Boot Chain), 컨테이너 런타임(Container Runtime), 이더넷 인프라(Ethernet Infrastructure), 무선 네트워크(Wireless Network), 클라우드 API(Cloud API), 펌웨어(Firmware), 물리 인터페이스(Physical Interface), 애플리케이션 로직(Application Logic)을 독립적으로 보호하지는 않는다. 따라서 생산용 로봇(Production Robot)은 SROS2와 호스트 강화(Host Hardening), 네트워크 분할(Network Segmentation), 자격 증명 보호(Credential Protection), 안전한 업데이트(Secure Update), 모니터링(Monitoring), 애플리케이션 수준 검증(Application-Level Validation)을 결합한 심층 방어(Defense in Depth)가 필요하다.

DDS 보안(DDS Security)이 활성화되어 있더라도 네트워크 분할(Network Segmentation)은 여전히 중요하다. 안전 제어기(Safety Controller), 인지 컴퓨터(Perception Computer), 운영자 인터페이스(Operator Interface), 플릿 게이트웨이(Fleet Gateway), 개발 컴퓨터(Development Machine), 외부 서비스(External Service)는 서로 다른 신뢰 요구사항을 가질 수 있다. VLAN, 방화벽(Firewall), DDS 도메인 분리(Domain Separation), 라우팅 제어(Routing Control), SROS2 권한을 상호 보완적으로 적용할 수 있다. 네트워크 제어는 접근 가능한 공격 표면을 줄이고 SROS2는 ROS 2 통신 객체에 보다 가까운 신원 기반 보호(Identity-Aware Protection)를 제공한다.

운영 보안(Operational Security)에는 자격 증명과 정책의 수명주기 관리(Lifecycle Management)도 필요하다. 인증서는 결국 갱신 또는 교체가 필요하고, 손상된 자격 증명은 폐기해야 할 수 있으며, 소프트웨어 업데이트는 새로운 토픽이나 서비스를 도입하여 권한 정책 변경을 요구할 수 있다. 따라서 SROS2 구성은 소프트웨어 릴리스(Software Release)와 연계된 버전 관리형 배포 자료(Version-Controlled Deployment Material)로 관리하는 것이 바람직하다. 개별 로봇에서 수동으로 관리되는 보안 정책은 대규모 플릿(Fleet) 환경에서 감사(Audit)와 일관된 운영을 어렵게 만든다.

보안 구성(Security Configuration)은 필연적으로 성능(Performance)과 상호작용한다. 인증은 시작 및 디스커버리 과정에 추가 작업을 발생시키며, 암호화, 무결성 검사(Integrity Checking), 키 관리(Key Management), 보호된 통신은 계산 및 대역폭 오버헤드(Computational and Bandwidth Overhead)를 추가한다. 실제 영향은 하드웨어, DDS 구현체, 페이로드 크기, 메시지 주기(Message Frequency), 네트워크 토폴로지(Network Topology), 보호 구성에 따라 달라진다. 따라서 고속 카메라, LiDAR 스트림, 제어 메시지, 저대역폭 진단 데이터가 동일한 보안 비용을 가진다고 가정해서는 안 된다.

실시간 로봇 시스템(Real-Time Robotic System)에서는 이러한 성능 관계가 더욱 중요하다. SROS2가 ROS 2를 결정론적 안전 네트워크(Deterministic Safety Network)로 변환하는 것은 아니며, 암호화 보호는 통신 경로에 추가적인 지연 변동(Latency Variation)을 발생시킬 수 있다. 따라서 시간 임계 제어 루프(Time-Critical Control Loop)는 보안이 해제된 개발 환경이 아니라 실제 사용할 보안 구성을 활성화한 상태에서 벤치마크(Benchmark)해야 한다. 보안 요구사항, 지연 예산(Latency Budget), CPU 사용률, 메시지 주기, 장애 동작(Failure Behavior)을 하나의 통합 시스템 문제로 평가해야 한다.

장애 처리(Failure Handling) 역시 중요한 아키텍처 고려사항이다. 자격 증명이 누락되거나 만료되거나 손상되었거나 배포된 신뢰 체인(Trust Chain)과 일치하지 않으면 참여자의 인증이 실패할 수 있다. 또한 권한 정책이 애플리케이션 개발자가 존재할 것으로 예상한 엔드포인트를 거부하여 통신이 실패할 수도 있다. 안전한 ROS 2 시스템은 이러한 장애를 일반적인 네트워크 장애와 구분하고, 운영자가 통신 단절의 원인이 연결성(Connectivity), 디스커버리, 권한 부여 또는 자격 증명 문제인지 판단할 수 있도록 충분한 진단 정보(Diagnostic Information)를 제공해야 한다.

다중 로봇 시스템(Multi-Robot System)에서 SROS2는 개별 로봇, 플릿 인프라(Fleet Infrastructure), 운영 네트워크(Operational Network)를 연결하는 더 광범위한 신뢰 아키텍처(Trust Architecture)의 일부가 된다. 자격 증명과 권한은 하나의 손상되거나 잘못 구성된 참여자가 모든 로봇에 무제한으로 접근하는 것을 방지하도록 설계되어야 한다. 네임스페이스 규칙(Namespace Convention)만으로는 보안 경계가 형성되지 않는다. ROS 2 통신이 단일 컴퓨터를 넘어 확장되면 신원, 권한 부여, 네트워크 분할, 도메인 아키텍처(Domain Architecture), 플릿 수준 자격 증명 관리(Fleet-Level Credential Management)가 함께 작동해야 한다.

이 장의 구조에서는 이 개요 이후에 인증 기관 설정(Certificate-Authority Setup), 권한 XML(Permission XML), DDS 보안 구성(DDS Security Configuration), 암호화 성능(Encryption Performance), 취약점 분석(Vulnerability Analysis), 네트워크 분할, 감사 로그(Audit Logging), 자동화된 보안 테스트(Automated Security Testing), IEC 62443 전략을 상세히 다룬다. 따라서 SROS2 아키텍처는 개별적인 구성 절차라기보다 이후의 여러 보안 메커니즘을 하나의 보안 모델(Security Model)로 연결하는 기반으로 이해해야 한다.

궁극적으로 SROS2는 보안의 핵심 질문을 단순히 특정 컴퓨터가 ROS 2 네트워크에 접근할 수 있는가에서, 암호학적으로 식별된 참여자(Cryptographically Identified Participant)가 신뢰할 수 있으며 특정 통신 작업을 수행하도록 명시적으로 승인되었는가로 전환한다. 이러한 신원 중심 모델(Identity-Centered Model)은 현대 ROS 2 시스템이 여러 프로세서, 컨테이너, 네트워크, 엣지 컴퓨터(Edge Computer), 플릿 서비스(Fleet Service)에 걸쳐 분산되기 때문에 생산용 로봇에서 특히 중요하다. 안전한 아키텍처는 ROS 2의 유연성을 유지하면서 신뢰 범위와 통신 권한을 의도적으로 제한하도록 설계되어야 한다.

## 06.02 CA Setup and Certificate Generation [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

SROS2에서 인증 기관(Certificate Authority, CA) 설정은 ROS 2 참여자(Participant)를 인증하고 DDS 기반 통신(DDS-Based Communication)을 보호하는 데 사용되는 신뢰 루트(Root of Trust)를 구축하는 과정이다. CA는 노드(Node) 또는 보안 엔클레이브(Security Enclave)가 신뢰된 로봇 시스템의 구성원임을 증명할 수 있도록 암호학적 신원(Cryptographic Identity)을 발급한다. ROS 2 보안은 궁극적으로 DDS 보안(DDS Security)에 의존하므로 인증서 생성(Certificate Generation)은 단순한 관리 작업이 아니라 인증(Authentication), 권한 부여(Authorization), 암호화(Encryption), 안전한 디스커버리(Secure Discovery)의 기반을 정의하는 과정이다.

SROS2 보안 환경(Security Environment)은 일반적으로 보호된 ROS 2 프로세스가 필요로 하는 암호학적 자료(Cryptographic Material)를 포함하는 키스토어(Keystore)를 중심으로 구성된다. 키스토어는 인증서(Certificate), 개인 키(Private Key), 거버넌스 정보(Governance Information), 권한(Permission), 엔클레이브별 보안 산출물(Security Artifacts)을 구조적으로 저장한다. 개발 환경에서는 하나의 워크스테이션에 존재할 수 있지만, 운영 환경에서는 민감한 개인 키와 서명 자격 증명(Signing Credential)이 불필요하게 노출되지 않도록 통제된 생성 및 배포 절차가 필요하다.

키스토어(Keystore)를 생성하는 것은 ROS 2 배포 환경을 위한 공개 키 기반 구조(Public Key Infrastructure, PKI)의 수명주기(Lifecycle)를 시작하는 단계이다. 보안 관리자는 다수의 참여자 신원을 생성하기 전에 키스토어의 저장 위치를 정의하고 신뢰 구조(Trust Structure)를 초기화한다. 이 구조는 일반적인 애플리케이션 데이터가 아니라 보안 핵심 구성(Security-Critical Configuration)으로 취급해야 한다. 따라서 대규모 로봇 신원을 생성하기 전에 접근 권한, 백업 절차, 저장소 보호 및 키스토어 소유권을 명확히 설정해야 한다.

이 신뢰 구조의 중심에는 인증 기관(Certificate Authority, CA)이 있다. CA는 보안 엔클레이브(Security Enclave)가 사용하는 인증서를 발급하거나 검증할 수 있는 서명 자격 증명(Signing Credential)을 보유한다. 신뢰된 인증 기관이 서명한 인증서는 특정 신원과 해당 신원이 동작하는 보안 도메인(Security Domain) 사이에 암호학적 신뢰 관계를 형성한다. CA 개인 키(CA Private Key)가 침해되면 공격자가 정상적인 신원처럼 보이는 인증서를 생성할 가능성이 있으므로 CA 서명 자료를 보호하는 것은 가장 중요한 운영 보안 요구사항 중 하나이다.

SROS2 도구(SROS2 Tooling)는 키스토어와 엔클레이브 관리(Enclave Management)를 위한 ROS 지향 명령을 제공하여 이러한 보안 인프라 생성을 단순화한다. 개발자가 모든 DDS 보안 산출물(DDS Security Artifact)을 수동으로 구성하는 대신 필요한 디렉터리 구조와 암호학적 파일을 도구가 생성한다. 이러한 추상화(Abstraction)는 기본 DDS 보안 신뢰 모델(Trust Model)을 유지하면서 구성 복잡성을 줄여준다. 그러나 자동화 도구를 사용하더라도 안전한 키 관리(Secure Key Handling)의 필요성이 사라지는 것은 아니므로 생성되는 자료의 의미를 이해해야 한다.

보안 엔클레이브(Security Enclave)는 SROS2 아키텍처에서 실행 신원(Execution Identity)을 나타낸다. 시스템 설계에 따라 하나의 엔클레이브는 특정 노드(Node), 프로세스(Process), 컨테이너(Container), 서브시스템(Subsystem) 또는 다른 운영 보안 경계(Operational Security Boundary)에 대응할 수 있다. 엔클레이브를 생성하면 SROS2는 해당 신원이 보호된 통신에 참여하는 데 필요한 보안 자료를 생성한다. 따라서 엔클레이브 이름은 배포 아키텍처의 중요한 요소가 되며 안정적이고 예측 가능한 명명 규칙(Naming Convention)을 따라야 한다.

인증서 생성(Certificate Generation)은 일반적으로 신원 인증서(Identity Certificate)와 이에 대응하는 개인 키(Private Key)를 생성한다. 인증서는 다른 DDS 참여자에게 제시할 수 있는 정보를 포함하지만 개인 키는 해당 신원의 소유자만 접근할 수 있도록 기밀로 유지해야 한다. 인증 과정에서 암호학적 연산(Cryptographic Operation)을 통해 네트워크로 개인 키 자체를 전송하지 않고도 해당 키를 보유하고 있음을 증명한다. 이를 통해 애플리케이션 수준의 ROS 2 통신이 허용되기 전에 참여자 간 신뢰를 확립할 수 있다.

신원 인증서(Identity Certificate)는 권한 부여(Authorization Permission)와 혼동해서는 안 된다. 유효한 인증서는 참여자가 보안 인프라가 신뢰하는 신원을 보유하고 있는지를 확인하지만, 이것만으로 모든 통신 권한이 자동으로 부여되는 것은 아니다. 권한(Permission)은 인증된 참여자가 무엇을 발행(Publish), 구독(Subscribe), 요청(Request)하거나 접근할 수 있는지를 결정한다. 따라서 안전한 배포를 위해서는 올바르게 생성된 신원과 적절하게 제한된 권한 정책(Authorization Policy)이 모두 필요하다.

DDS 보안(DDS Security)은 신원 자격 증명(Identity Credential)과 함께 거버넌스(Governance) 및 권한 정보(Permissions Information)를 사용한다. 거버넌스 구성(Governance Configuration)은 DDS 도메인과 통신 범주의 보안 동작을 설정하고, 권한 문서(Permissions Document)는 특정 신원이 수행할 수 있는 작업을 정의한다. 이러한 산출물은 승인되지 않은 변경을 탐지할 수 있도록 암호학적으로 서명된다. 결과적으로 신뢰 기관에서 서명된 보안 문서를 거쳐 런타임 참여자까지 이어지는 신뢰 체인(Trust Chain)이 형성된다.

따라서 인증서 생성은 정책 생성(Policy Generation)과 통합되어야 하며 독립적인 작업으로 취급해서는 안 된다. 새로 생성된 엔클레이브는 유효한 암호학적 자격 증명을 보유하더라도 해당 ROS 2 인터페이스에 적합한 권한이 추가로 필요할 수 있다. 예를 들어 위치 추정 구성요소(Localization Component)는 센서 데이터를 구독하고 위치 정보를 발행해야 하지만 액추에이터 제어기(Actuator Controller)는 전혀 다른 정책이 필요하다. 따라서 신원 명명(Identity Naming)과 통신 정책은 함께 설계해야 한다.

런타임 구성(Runtime Configuration)은 생성된 보안 산출물을 ROS 2 실행 환경과 연결한다. 시스템은 보안 집행(Security Enforcement)이 활성화되어 있는지, 그리고 프로세스가 어떤 엔클레이브 또는 키스토어 컨텍스트(Keystore Context)를 사용해야 하는지 알아야 한다. 경로(Path), 엔클레이브 신원, 환경 설정(Environment Setting), 보안 파일이 서로 일치하지 않으면 일반적인 비보안 ROS 2 환경에서는 통신할 수 있더라도 노드 간 인증 또는 디스커버리가 실패할 수 있다. 따라서 전체 로봇 스택을 시작하기 전에 배포 자동화(Deployment Automation)를 통해 보안 구성을 검증하는 것이 중요하다.

파일 시스템 보호(File-System Protection)는 특히 개인 키(Private Key)에 중요하다. 인증서는 인증 과정에서 공유되도록 설계되지만 개인 키는 비밀 자격 증명(Secret Credential)이므로 승인된 프로세스와 관리자만 접근할 수 있어야 한다. 운영 로봇에서는 운영체제 권한(Operating-System Permission), 보호 저장소(Protected Storage), 컨테이너 격리(Container Isolation), 보안 부트(Secure Boot), 하드웨어 기반 키 저장소(Hardware-Backed Key Storage) 등을 SROS2와 함께 사용할 수 있다. 개발용 키스토어 전체를 여러 로봇에 무분별하게 복사하면 강력한 암호화 보호 자체가 약화될 수 있다.

개발용 인증 기관(Development CA)과 운영용 인증 기관(Production CA)은 서로 다른 운영 자산(Operational Asset)으로 취급하는 것이 바람직하다. 개발 환경에서는 신원을 반복적으로 생성하고 제거해야 하는 경우가 많지만 실제 배포된 로봇에서는 통제된 발급과 추적성(Traceability)이 필요하다. 두 환경을 분리하면 실험용 자격 증명이 의도하지 않게 운영 시스템에 접근할 가능성을 줄일 수 있다. 또한 운영용 서명 키(Production Signing Key)는 모든 엔지니어링 워크스테이션이나 로봇에 상시 저장하기보다 더 강력하게 보호된 시스템에서 관리할 수 있다.

대규모 로봇 플릿(Robot Fleet)에서는 체계적인 인증서 명명 및 소유권 규칙이 필요하다. 신원은 개별 로봇, 컴퓨팅 모듈(Computing Module), 컨테이너, 플릿 서비스(Fleet Service), 운영자 스테이션(Operator Station), 기능 서브시스템(Functional Subsystem)을 나타낼 수 있다. 일관된 규칙이 없으면 어떤 인증서가 어떤 운영 구성요소에 속하는지 파악하기 어려워진다. 구조화된 신원 모델(Structured Identity Model)은 관리자가 관찰된 DDS 참여자를 예상된 소프트웨어 기능 및 배포 위치와 연결할 수 있게 하므로 감사(Audit)에도 도움이 된다.

인증서 수명주기 관리(Certificate Lifecycle Management)는 초기 생성 이후에도 계속된다. 자격 증명은 만료될 수 있고 시스템이 교체될 수 있으며 개인 키가 침해되었다고 의심될 수도 있고 로봇 소프트웨어의 발전에 따라 권한 요구사항이 변경될 수도 있다. 따라서 조직은 인증서 갱신(Renewal), 교체(Replacement), 폐기(Revocation), 재배포(Redistribution) 절차를 마련해야 한다. 지나치게 긴 유효 기간은 키가 분실되거나 침해되었을 때 노출 위험을 증가시키므로 인증서 유효 기간은 단순히 유지보수를 피하기 위한 목적으로 결정해서는 안 된다.

키 순환(Key Rotation)은 장기간의 자격 증명 노출을 제한하기 위한 또 다른 메커니즘이다. 새로운 키와 인증서를 주기적으로 또는 보안 관련 이벤트 이후에 발급하고 이전 자격 증명을 활성 배포 환경에서 제거할 수 있다. 그러나 분산 ROS 2 시스템에서 자격 증명이 서로 일치하지 않으면 인증된 디스커버리가 실패할 수 있으므로 순환 절차를 신중하게 검증해야 한다. 따라서 플릿 규모 시스템에서는 각 로봇에 설치된 보안 산출물을 추적할 수 있는 자동화된 인벤토리(Inventory) 및 배포 메커니즘이 유용하다.

CA와 키스토어 자료의 백업(Backup)은 일반적인 소스 코드 백업과 다른 방식으로 접근해야 한다. 중요한 서명 자료를 잃으면 향후 인증서 관리가 어려워질 수 있지만 개인 키를 무분별하게 복제하면 추가적인 침해 가능성이 발생한다. 따라서 백업은 암호화하고 접근을 통제하며 포함된 키의 중요성에 따라 안전하게 저장해야 한다. 또한 보안 인프라 장애로 인해 배포된 로봇이 새로운 유효 자격 증명을 영구적으로 받을 수 없는 상황을 방지하도록 복구 절차(Recovery Procedure)를 사전에 검증해야 한다.

인증서 생성 과정은 절차 수준에서는 재현 가능(Reproducible)해야 하지만 비밀 키 자체를 재현 가능하게 만들어서는 안 된다. 구성, 명명 규칙, 정책 템플릿(Policy Template), 스크립트 및 배포 절차는 버전 관리(Version Control)할 수 있지만 생성된 개인 키는 일반적인 저장소에서 분리해야 한다. 이를 통해 엔지니어는 Git 기록, 빌드 산출물(Build Artifact), 컨테이너 이미지(Container Image), 공유 개발 디렉터리에 민감한 암호학적 자료를 노출하지 않고도 통제된 절차에 따라 보안 환경을 다시 구축할 수 있다.

컨테이너 기반 ROS 2 배포(Containerized ROS 2 Deployment)는 추가적인 자격 증명 관리 고려사항을 요구한다. 장기간 사용하는 개인 키를 재사용 가능한 컨테이너 이미지에 직접 포함하는 방식은 모든 이미지 복사본이 동일한 비밀 정보를 보유할 수 있기 때문에 바람직하지 않다. 더 안전한 방식은 보호된 마운트(Protected Mount) 또는 비밀 관리 메커니즘(Secret-Management Mechanism)을 통해 배포 시점에 필요한 보안 산출물을 컨테이너에 제공하는 것이다. 이를 통해 컨테이너의 재현성을 유지하면서 특정 로봇이나 서브시스템에 할당된 신원을 독립적으로 제어할 수 있다.

다중 로봇 시스템(Multi-Robot System)은 잘못된 인증서 아키텍처의 영향을 더욱 확대한다. 모든 로봇에 동일한 자격 증명을 제공하면 배포는 단순해질 수 있지만 특정 침해 로봇을 식별하거나 격리하고 개별적으로 인증을 폐기하는 능력은 감소한다. 로봇별 또는 적절한 범위의 엔클레이브 신원을 사용하면 보다 세밀한 제어가 가능하다. 플릿 관리자(Fleet Manager), 로봇 제어기(Robot Controller), 인지 서브시스템(Perception Subsystem), 유지보수 워크스테이션(Maintenance Workstation)은 동일한 전체 신뢰 인프라에 속하면서도 서로 구별되는 신원과 권한을 가질 수 있다.

테스트(Testing)는 안전한 통신이 성공하는지만 확인해서는 안 된다. 엔지니어는 유효한 참여자가 정상적으로 인증되는지, 유효하지 않거나 알 수 없는 인증서가 거부되는지, 승인되지 않은 작업이 차단되는지, 손상된 보안 산출물이 예측 가능한 장애를 발생시키는지, 자격 증명이 누락되었을 때 보안 기능이 조용히 비활성화되지 않는지를 확인해야 한다. 유효한 인증서로 시스템이 동작한다는 사실만으로는 신뢰 경계(Trust Boundary)가 유효하지 않은 참여자를 실제로 차단한다는 것이 검증되지 않으므로 부정 보안 테스트(Negative Security Test)가 필수적이다.

운영 진단(Operational Diagnostics)은 비밀 자료를 노출하지 않으면서 인증서 관련 장애를 관찰할 수 있도록 해야 한다. 로그(Log)는 인증 실패, 유효하지 않은 권한, 인증서 유효성 문제, 엔클레이브 구성 오류, 접근할 수 없는 키스토어 파일 등을 식별할 수 있어야 하지만 개인 키나 민감한 자격 증명 내용은 진단 출력에 기록해서는 안 된다. 명확한 오류 분류(Error Classification)는 외형상 ROS 2 디스커버리 실패로 보이는 문제가 실제로는 인증서 또는 정책 불일치에서 발생하는 플릿 환경에서 특히 중요하다.

궁극적으로 CA 및 인증서 인프라(Certificate Infrastructure)는 보호된 ROS 2 환경 내부에서 누가 신뢰할 수 있는 신원(Trusted Identity)을 제시할 수 있는지를 결정한다. 이후의 SROS2 메커니즘은 이 기반 위에서 해당 신원이 무엇에 접근할 수 있는지 정의하고 통신을 암호학적으로 보호한다. 따라서 견고한 구현은 보호된 CA 운영, 통제된 인증서 생성, 적절한 범위의 엔클레이브 신원, 안전한 키 배포, 수명주기 관리 및 체계적인 테스트를 하나의 연속적인 보안 프로세스(Continuous Security Process)로 통합해야 한다.

## 06.03 Node Access Control Policy: Permission XML [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

SROS2의 노드 수준 접근 제어(Node-Level Access Control)는 인증된 ROS 2 참여자(Participant)가 어떤 통신 작업을 수행할 권한이 있는지를 결정한다. 인증(Authentication)은 신원을 확인하고, 접근 제어(Access Control)는 해당 신원에 연결된 권한을 제한한다. 따라서 권한 정책(Permission Policy)은 보안 엔클레이브(Security Enclave)가 토픽(Topic)을 발행 또는 구독하거나, 서비스(Service)를 호출 또는 제공하거나, 액션(Action)을 사용하거나, 지정된 DDS 도메인(Domain)에 참여할 수 있는지를 정의한다. 신원과 권한을 분리하는 것은 ROS 2 최소 권한 보안(Least-Privilege Security)의 기본 원칙이다.

SROS2 접근 제어는 DDS 보안 접근 제어 플러그인(DDS Security Access Control Plugin)을 기반으로 하며, 서명된 권한 문서(Signed Permissions Document)를 통해 권한 부여 규칙(Authorization Rule)을 표현한다. 런타임(Runtime)에서 DDS 구현체는 보호된 통신 엔드포인트(Communication Endpoint)의 상호작용을 허용하기 전에 이러한 규칙을 평가한다. SROS2는 개발자가 익숙한 ROS 개념을 이용하여 필요한 통신을 정의하면서도 최종적으로 하위 DDS 보안 아키텍처와 호환되는 보안 산출물을 생성할 수 있도록 ROS 지향 도구와 정책 추상화(Policy Abstraction)를 제공한다.

권한 XML 문서(Permissions XML Document)는 이러한 메커니즘의 핵심 산출물이다. 이 문서는 참여자 신원(Participant Identity)을 하나 이상의 권한 부여(Authorization Grant)와 연결하고 통신이 허용되거나 거부되는 조건을 정의한다. 하나의 권한 부여는 일반적으로 적용 대상 인증서의 주체(Subject)를 식별하고, 유효 기간(Validity Interval)을 설정하며, 권한 규칙을 포함하고, 명시적으로 일치하지 않는 작업에 대한 기본 동작(Default Behavior)을 지정한다. 결과적으로 이 문서는 참여자에게 적용되는 기계 집행 가능한 보안 계약(Machine-Enforceable Security Contract)을 표현한다.

권한 부여에 포함된 주체 신원(Subject Identity)은 참여자 인증서(Participant Certificate)가 나타내는 신원과 일치해야 한다. 이러한 결합은 특정 신뢰 참여자를 위해 작성된 권한 문서가 다른 신원에 임의로 재사용되는 것을 방지한다. 따라서 인증서 명명(Certificate Naming)과 권한 정책 설계는 보안 아키텍처 초기 단계부터 함께 조정해야 한다. 신원 명명 규칙이 일관되지 않으면 인증서는 정상적으로 인증되지만 적절한 권한 부여 규칙과 일치하지 않아 배포 과정에서 해결하기 어려운 장애가 발생할 수 있다.

권한 규칙(Permission Rule)은 DDS 도메인(DDS Domain)에 따라 접근을 제한할 수 있다. 이는 여러 ROS 2 시스템, 개발 환경, 로봇 또는 운영 구역(Operational Zone)이 서로 다른 도메인 식별자(Domain Identifier)를 사용하는 경우 유용하다. 참여자에게 전체 DDS 환경에 대한 접근 권한을 제공하는 대신 지정된 도메인에서만 동작하도록 허용할 수 있다. 도메인 제한(Domain Restriction)은 미들웨어 보안 계층에서 암호학적으로 집행되는 권한 부여를 추가함으로써 네트워크 분할(Network Segmentation)과 ROS 도메인 구성을 보완한다.

발행(Publish) 및 구독(Subscribe) 권한은 ROS 2 토픽 통신(Topic Communication)에 가장 직접적으로 대응한다. 인지 엔클레이브(Perception Enclave)는 카메라 데이터를 구독하고 감지된 객체 정보를 발행할 수 있지만 액추에이터 명령을 발행하는 권한은 제한할 수 있다. 반대로 모션 제어기(Motion Controller)는 승인된 궤적 정보를 구독하고 제어 상태를 발행하도록 구성할 수 있다. 실제 데이터 흐름(Data Flow)을 기준으로 이러한 규칙을 설계하면 인증된 모든 구성요소가 ROS 2 그래프에서 무제한 발행자 또는 구독자가 되는 것을 방지할 수 있다.

서비스(Service)는 ROS 추상화 계층 아래에서 DDS 엔티티(DDS Entity)를 사용하여 구현되므로 추가적인 고려가 필요하다. 마찬가지로 액션(Action)은 목표(Goal), 결과(Result), 피드백(Feedback), 상태(Status), 취소(Cancellation) 동작을 위해 여러 통신 채널을 사용한다. 따라서 정책 작성자는 하나의 가시적인 ROS 인터페이스가 항상 하나의 DDS 토픽과 대응한다고 가정해서는 안 된다. SROS2 도구는 ROS 지향 정책을 토픽, 서비스, 액션에 필요한 하위 통신 규칙으로 변환하는 과정을 지원한다.

접근 제어 정책(Access-Control Policy)은 제한이 없는 ROS 그래프에서 출발하기보다 의도된 노드 인터페이스(Intended Node Interface)를 기준으로 설계해야 한다. 엔지니어는 구성요소가 어떤 정보를 소비하고, 어떤 정보를 생성하며, 어떤 서비스를 호출하거나 제공하고, 어떤 액션에 참여하는지를 식별해야 한다. 이러한 통신 요구사항을 명시적인 허용 목록(Allowlist)으로 변환할 수 있다. 광범위한 권한을 먼저 부여하고 이후 불필요한 접근을 제거하는 방식은 소프트웨어에 실제로 필요하지 않은 권한을 남길 가능성이 높다.

이러한 접근 방식은 최소 권한 원칙(Principle of Least Privilege)을 구현한다. 위치 추정 노드(Localization Node)는 센서 관측값을 필요로 하고 위치 추정 결과를 발행할 수 있지만 일반적으로 펌웨어 업데이트 명령을 실행할 이유는 없다. 진단 프로세스(Diagnostic Process)는 시스템 상태를 읽을 수 있지만 모션 명령에 대한 권한은 필요하지 않을 수 있다. AI 추론 노드(AI Inference Node)는 영상을 입력받아 추론 결과를 발행하면서 내비게이션 파라미터를 변경할 권한은 갖지 않도록 구성할 수 있다. 권한 XML은 이러한 아키텍처적 차이를 미들웨어에서 집행 가능한 규칙으로 변환한다.

허용 규칙(Allow Rule)과 거부 규칙(Deny Rule)은 정책 경계를 표현하는 메커니즘을 제공한다. 허용 규칙은 참여자가 수행할 수 있는 통신을 정의하고, 거부 규칙은 일치하는 작업의 수행을 명시적으로 차단한다. 어떤 규칙과도 일치하지 않는 경우의 기본 동작(Default Action) 역시 정책 동작에 영향을 준다. 보안이 중요한 시스템에서는 필요한 통신만 명시적으로 허용하는 거부 중심 정책(Deny-Oriented Policy)이 유리하다. 이렇게 하면 새롭게 추가되거나 예상하지 못한 인터페이스에 권한이 자동으로 부여되는 것을 방지할 수 있다.

와일드카드 표현식(Wildcard Expression)은 정책 정의를 단순화할 수 있지만 신중하게 사용해야 한다. 와일드카드를 사용하면 관련 토픽 또는 네임스페이스(Namespace) 집합을 편리하게 허용하여 대규모 시스템의 반복적인 구성을 줄일 수 있다. 그러나 지나치게 광범위한 패턴은 향후 추가되는 토픽이 동일한 표현식과 일치하면서 의도하지 않은 접근 권한을 획득하게 만들 수 있다. 따라서 와일드카드는 단순히 상세한 인터페이스 분석을 피하기 위한 수단이 아니라 의도적으로 설계된 보안 그룹(Security Group)을 표현하는 용도로 사용해야 한다.

네임스페이스(Namespace)는 다중 로봇 ROS 2 그래프(Multi-Robot ROS 2 Graph)를 구성하는 데 유용하지만 네임스페이스 분리 자체가 권한 부여 메커니즘은 아니다. \`/robot1\`과 \`/robot2\`와 같은 로봇은 논리적으로 분리된 인터페이스를 가질 수 있지만 동일한 네트워크와 DDS 인프라를 공유할 수 있다. 권한 정책은 로봇별 엔클레이브가 의도된 인터페이스와 관련된 통신만 수행하도록 제한하여 이러한 아키텍처를 강화할 수 있다. 이는 다수의 로봇이 공유 플릿 인프라(Fleet Infrastructure)에서 운영될 때 특히 중요하다.

보안 엔클레이브(Security Enclave)는 이러한 권한이 집행되는 신원 컨텍스트(Identity Context)를 제공한다. 여러 노드가 의도적으로 동일한 신뢰 경계(Trust Boundary)에 속한다면 하나의 엔클레이브를 공유할 수 있으며, 보안에 민감한 구성요소에는 더 제한적인 권한을 가진 별도의 엔클레이브를 할당할 수 있다. 적절한 세분성(Granularity)은 프로세스 컴포지션(Process Composition), 배포 아키텍처, 성능 및 위협 가정(Threat Assumption)에 따라 달라진다. 서로 관련 없는 구성요소를 하나의 광범위한 권한 엔클레이브로 결합하면 노드 수준 정책이 제공하려는 격리 효과를 약화시킬 수 있다.

권한 문서(Permissions Document)는 승인되지 않은 변경으로부터 보호되어야 한다. 따라서 DDS 보안은 암호학적 서명(Cryptographic Signature)을 사용하여 참여자가 권한 정보가 신뢰할 수 있는 기관에서 생성되었으며 변경되지 않았음을 검증할 수 있도록 한다. XML 파일을 서명한 이후 직접 수정하는 것은 유효한 정책 업데이트가 아니며, 수정된 산출물은 적절한 서명 절차를 다시 거쳐야 한다. 따라서 정책 생성(Policy Generation)과 서명(Signing)은 통제된 배포 워크플로(Controlled Deployment Workflow)로 관리해야 한다.

거버넌스(Governance)와 권한(Permissions)은 서로 다르지만 상호 보완적인 역할을 수행한다. 거버넌스 구성(Governance Configuration)은 인증 또는 암호화 보호가 필요한지와 같은 DDS 도메인 및 통신 범주의 전반적인 보안 요구사항을 정의한다. 권한은 이러한 제약 조건 내에서 특정 인증 신원이 수행할 수 있는 작업을 정의한다. 따라서 하나의 배포 환경에서 강력한 도메인 수준 보안 요구사항을 적용하면서 인지, 내비게이션, 제어, 진단, 플릿 서비스, 운영자 인터페이스에 서로 다른 통신 권한을 할당할 수 있다.

ROS 2 소프트웨어가 발전함에 따라 정책 버전 관리(Policy Versioning)가 중요해진다. 새로운 토픽 추가, 네임스페이스 변경, 서비스 도입 또는 액션 인터페이스 재구성으로 인해 기존 권한 정책이 이전에는 정상적으로 동작하던 통신을 거부할 수 있다. 따라서 보안 정책은 소프트웨어 인터페이스 계약(Software Interface Contract)의 일부로 취급하고 애플리케이션 변경과 함께 관리해야 한다. 버전 관리된 정책 소스는 언제 특정 권한이 추가, 변경 또는 제거되었는지를 검토할 수 있게 한다.

자동화된 정책 생성(Automated Policy Generation)은 특히 많은 노드와 인터페이스를 포함하는 시스템에서 수동 오류를 줄일 수 있다. 그러나 제한 없는 권한으로 실행되는 애플리케이션을 관찰하여 정책을 자동 생성하면 불필요하거나 임시적인 통신까지 포함될 수 있다. 따라서 생성된 정책은 자동으로 승인하기보다 의도된 아키텍처와 비교하여 검토해야 한다. 목표는 테스트 중 관찰된 모든 통신을 그대로 재현하는 것이 아니라 정상 운영에 실제로 필요한 통신만 허용하는 것이다.

테스트(Testing)는 허용된 동작과 금지된 동작을 모두 검증해야 한다. 긍정 테스트(Positive Test)는 SROS2 보안 집행이 활성화된 상태에서 승인된 발행자, 구독자, 서비스 및 액션이 정상적으로 동작하는지 확인한다. 부정 테스트(Negative Test)는 참여자가 수행해서는 안 되는 작업을 의도적으로 시도하고 미들웨어가 이를 거부하는지 확인한다. 부정 테스트가 없다면 정책이 운영상 정상적으로 보이면서도 실제로는 보안 경계를 크게 약화시키는 과도한 권한을 포함하고 있을 수 있다.

권한 오류(Permission Failure)는 일반적인 ROS 2 통신 문제처럼 보일 수 있다. 발행자가 존재하지만 구독자와 통신하지 못하거나, 서비스 클라이언트가 사용 가능한 것처럼 보이는 서버에 접근하지 못하는 이유가 보안 정책에서 필요한 DDS 엔드포인트를 거부했기 때문일 수 있다. 따라서 진단 기능은 디스커버리(Discovery), 네트워크(Network), 인증(Authentication), 권한 부여(Authorization) 실패를 구분할 수 있어야 한다. 명확한 보안 로깅(Security Logging)은 여러 컴퓨터 또는 로봇에 복잡한 권한 집합을 배포할 때 문제 해결 시간을 크게 줄인다.

접근 제어는 배포 자동화(Deployment Automation)에도 영향을 준다. 각 로봇 또는 서브시스템은 올바른 인증서, 개인 키, 권한, 거버넌스 정보 및 엔클레이브 구성을 하나의 일관된 보안 패키지(Security Package)로 받아야 한다. 하나의 신원에 속한 자격 증명을 다른 신원을 위한 권한과 혼합하면 인증 또는 권한 부여가 실패할 수 있다. 따라서 플릿 배포 시스템(Fleet Deployment System)은 보안 산출물을 신원별 구성(Identity-Specific Configuration)으로 취급하고 소프트웨어 릴리스를 활성화하기 전에 이들의 일관성을 검증해야 한다.

다중 로봇 환경(Multi-Robot Environment)에서는 세밀하게 제한된 정책이 침해된 구성요소의 영향을 억제하는 데 도움이 된다. 한 로봇의 인지 프로세스가 침해되더라도 해당 프로세스의 권한이 모든 로봇을 제어하거나 관련 없는 플릿 서비스에 접근하도록 허용해서는 안 된다. 로봇별, 서브시스템별 또는 기능별 신원과 제한적인 정책을 결합하면 잠재적인 피해 범위(Blast Radius)를 줄일 수 있다. 네트워크 분할과 DDS 도메인 설계는 SROS2 권한 부여 경계 주위에 추가적인 보호 계층을 제공한다.

궁극적으로 권한 XML(Permission XML)은 로봇 통신 아키텍처(Robot Communication Architecture)를 실행 가능한 형태로 표현한 것으로 이해해야 한다. 예를 들어 "인지 시스템은 감지 결과를 발행할 수 있지만 모션을 명령할 수 없다"라는 아키텍처 요구사항을 미들웨어가 실제로 집행하는 규칙으로 변환한다. 따라서 효과적인 SROS2 접근 제어는 올바른 XML 문법만으로 완성되지 않으며, 정확한 인터페이스 모델링(Interface Modeling), 안정적인 신원 명명 규칙, 최소 권한 설계, 서명된 정책 산출물, 통제된 배포, 버전 관리 및 지속적인 긍정·부정 보안 검증을 함께 요구한다.

## 06.04 DDS Security Plugin Configuration [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

DDS 보안(DDS Security)은 SROS2가 ROS 2 분산 통신(Distributed Communication)을 보호하기 위해 사용하는 미들웨어 수준 보안 기반(Middleware-Level Security Foundation)을 제공한다. ROS 2 애플리케이션 노드 내부에서 인증(Authentication), 권한 부여(Authorization), 암호화(Encryption)를 직접 구현하는 대신, DDS 보안은 ROS 미들웨어 계층 아래에서 동작하는 표준화된 플러그인 인터페이스(Standardized Plugin Interface)를 정의한다. 이러한 플러그인의 구성에 따라 참여자가 신뢰를 형성하고, 권한을 집행하며, 메시지를 보호하고, DDS 도메인 내에서 서로를 안전하게 발견하는 방법이 결정된다.

DDS 보안 아키텍처(DDS Security Architecture)는 보안 책임을 서로 협력하는 여러 플러그인으로 분리한다. 인증 플러그인(Authentication Plugin)은 참여자 신원을 검증하고, 접근 제어 플러그인(Access Control Plugin)은 해당 신원이 수행할 수 있는 DDS 작업을 결정하며, 암호화 플러그인(Cryptographic Plugin)은 암호화와 무결성 메커니즘을 통해 통신을 보호한다. 추가적인 로깅 관련 기능은 보안 감사(Security Auditing)를 지원할 수 있다. SROS2는 이러한 메커니즘에 필요한 구성 산출물을 조정하여 ROS 2 애플리케이션이 암호화 프로토콜을 직접 구현하지 않고도 사용할 수 있도록 한다.

인증 플러그인(Authentication Plugin)은 발견된 DDS 참여자(Participant)가 신뢰할 수 있는 신원을 나타내는지를 판단한다. 참여자는 신뢰할 수 있는 인증 기관(Certificate Authority)이 발급한 신원 인증서(Identity Certificate)를 제시하고 이에 대응하는 개인 키(Private Key)를 보유하고 있음을 증명한다. 인증 과정은 보호된 통신이 시작되기 전에 인증서 체인(Certificate Chain)과 암호학적 증거(Cryptographic Evidence)를 검증한다. 이를 통해 단순히 네트워크에 연결되거나 DDS 디스커버리(Discovery)에 참여했다는 이유만으로 보호된 ROS 2 시스템의 신뢰 구성원이 되는 것을 방지한다.

따라서 인증 구성(Authentication Configuration)은 올바르게 배포된 신원 산출물(Identity Artifact)에 의존한다. 각 보안 엔클레이브(Security Enclave)는 해당 보안 환경에 적합한 신원 인증서, 개인 키 및 신뢰할 수 있는 CA 인증서를 필요로 한다. 이러한 파일들은 서로 일치해야 하며 런타임에서 DDS 구현체가 접근할 수 있어야 한다. 잘못된 경로, 일치하지 않는 인증서와 키, 유효하지 않은 인증서 체인 또는 만료된 자격 증명은 기본 ROS 2 네트워크와 DDS 디스커버리가 정상적으로 동작하더라도 인증 실패를 발생시킬 수 있다.

접근 제어 플러그인(Access Control Plugin)은 신원이 확인된 이후 동작하며, 인증된 참여자가 특정 DDS 작업을 수행하도록 허가되었는지를 평가한다. 이 동작은 주로 서명된 거버넌스 문서(Signed Governance Document)와 권한 문서(Permissions Document)에 의해 제어된다. 거버넌스(Governance)는 도메인과 통신 범주에 광범위하게 적용되는 보안 요구사항을 정의하고, 권한(Permissions)은 참여자 신원과 허용 또는 거부되는 작업을 연결한다. 두 요소가 결합되어 인증된 시스템 참여를 통제된 통신 권한(Controlled Communication Authority)으로 변환한다.

거버넌스 구성(Governance Configuration)은 중요한 도메인 수준 보안 동작(Domain-Level Security Behavior)을 결정한다. 참여자 인증이 필요한지 여부와 디스커버리 정보 또는 사용자 데이터에 어떤 보호를 적용할지를 지정할 수 있다. 따라서 운영 요구사항에 따라 서로 다른 통신 범주에 다른 수준의 보안을 적용할 수 있다. 참여자 간 보안 요구사항이 일관되지 않으면 통신이 실패하거나 분산 시스템 내부의 보호 수준에 의도하지 않은 차이가 발생할 수 있으므로 거버넌스는 시스템 전체 보안 정책(System-Wide Security Policy)으로 설계해야 한다.

권한 구성(Permissions Configuration)은 개별 신원 또는 엔클레이브에 대해 더욱 세밀한 권한 부여를 제공한다. 규칙은 발행(Publication), 구독(Subscription), 도메인, 파티션(Partition), ROS 2 인터페이스와 관련된 통신을 제한할 수 있다. SROS2는 ROS 지향 정책 정의를 DDS 보안이 이해할 수 있는 산출물로 변환한다. 따라서 제어 엔클레이브(Control Enclave)에는 모션 인터페이스에 필요한 권한을 부여하고 진단 엔클레이브(Diagnostic Enclave)에는 모니터링과 관련된 권한만 부여하여 미들웨어 통신 경계에서 최소 권한(Least Privilege)을 구현할 수 있다.

암호화 플러그인(Cryptographic Plugin)은 구성된 보안 정책에 따라 DDS 통신에 기밀성(Confidentiality)과 무결성 보호(Integrity Protection)를 제공한다. 암호화는 승인되지 않은 관찰자가 보호된 페이로드를 해석하는 것을 방지하고, 메시지 인증(Message Authentication)과 무결성 메커니즘은 승인되지 않은 변경을 탐지하는 데 도움을 준다. 암호학적 처리는 개별 ROS 2 노드가 자체 암호화 방식을 구현하는 대신 DDS 내부에서 수행되므로 호환되는 미들웨어 참여자 전체에 일관된 보안 동작을 적용할 수 있다.

암호화 구성(Cryptographic Configuration)은 로봇 데이터의 민감도와 시간 특성(Timing Characteristics)을 반영해야 한다. 제어 명령, 운영 자격 증명, 지도, 인지 정보, 진단 데이터 및 고대역폭 센서 스트림(High-Bandwidth Sensor Stream)은 서로 다른 기밀성과 무결성 요구사항을 가질 수 있다. 모든 데이터에 무조건 최대 수준의 보호를 적용하면 불필요한 계산 및 대역폭 비용이 발생할 수 있고, 보호가 부족하면 중요한 정보가 노출될 수 있다. 따라서 데이터 분류(Data Classification), 위협 분석(Threat Analysis), 성능 요구사항을 DDS 보호 설정과 연결해야 한다.

DDS 디스커버리(DDS Discovery) 역시 일반적인 애플리케이션 트래픽이 시작되기 전에 참여자와 통신 엔드포인트 정보를 교환하기 때문에 신중한 보안 처리가 필요하다. 보안 디스커버리(Secured Discovery)는 참여자들이 서로 인증하고 동적으로 보호된 통신 관계를 형성할 수 있게 한다. 그러나 인증 핸드셰이크(Authentication Handshake)와 암호학적 연산은 참여자 시작 과정에 추가 작업을 발생시킨다. 따라서 다수의 노드 또는 로봇을 포함하는 시스템에서는 비보안 ROS 2 테스트 결과만으로 판단하지 말고 실제 보안 배포 조건에서 디스커버리 동작을 평가해야 한다.

SROS2 런타임 구성(Runtime Configuration)은 ROS 2 프로세스를 DDS 보안 플러그인 및 관련 산출물과 연결한다. 런타임 환경은 ROS 보안을 활성화하고 적절한 키스토어(Keystore) 또는 엔클레이브를 지정하며 보안 실패를 처리하는 방법을 설정해야 한다. 보안 집행(Security Enforcement)이 활성화된 경우 누락되거나 유효하지 않은 산출물은 통신 실패를 발생시켜야 하며, 보안이 해제된 상태로 조용히 전환되어서는 안 된다. 이러한 안전 실패 동작(Fail-Secure Behavior)은 운영 로봇이 예상된 보호 없이 통신하는 상황을 방지하는 데 중요하다.

SROS2와 ROS 미들웨어 구현체(ROS Middleware Implementation)의 관계도 고려해야 한다. ROS 2는 서로 다른 DDS 기반 RMW 구현체를 통해 동작할 수 있으며 실제 구성 세부사항과 지원되는 보안 동작은 미들웨어 제품 및 버전에 따라 달라질 수 있다. 따라서 보안 검증(Security Validation)은 실제 배포에 사용할 정확한 RMW 구현체를 사용하여 수행해야 한다. 하나의 DDS 구현체에서 검증된 구성이 다른 구현체에서도 동일하게 동작한다고 자동으로 가정해서는 안 된다.

보안 플러그인 구성(Security Plugin Configuration)은 ROS 2 실행(Launch) 및 배포 아키텍처(Deployment Architecture)와 통합되어야 한다. 별도의 프로세스로 실행되는 노드는 서로 다른 보안 엔클레이브를 사용할 수 있으며, 컴포지션된 노드(Composed Node)는 배포 설계에 따라 프로세스 수준의 보안 컨텍스트(Security Context)를 공유할 수 있다. 컨테이너(Container)는 자격 증명과 보안 산출물을 런타임에 안전하게 제공해야 하므로 또 다른 경계를 형성한다. 따라서 엔클레이브 구조는 실제 프로세스 격리, 애플리케이션 책임 및 원하는 최소 권한 경계를 반영해야 한다.

DDS 보안이 강력한 통신 보호를 제공하더라도 파일 및 디렉터리 보호(File and Directory Protection)는 여전히 필수적이다. 신원 개인 키, CA 자료, 권한 파일 및 기타 보안 산출물은 승인된 사용자와 프로세스만 읽을 수 있어야 한다. 개인 자격 증명(Private Credential)을 소스 저장소, 범용 컨테이너 이미지 또는 공유 파일 시스템 위치에 부주의하게 포함해서는 안 된다. 공격자가 신뢰 신원을 구축하는 데 사용되는 개인 키를 획득한 경우 통신 암호화만으로 이를 보완할 수 없다.

분산 로봇 시스템(Distributed Robot System)에서는 구성 일관성(Configuration Consistency)이 특히 중요하다. 인증서, 거버넌스 규칙, 권한, 엔클레이브 이름, 도메인 식별자 및 런타임 설정이 통신 참여자 전체에서 일관된 보안 구성을 형성해야 한다. 작은 불일치도 디스커버리 실패, 엔드포인트 거부 또는 ROS 2 토픽과 서비스가 사라진 것처럼 보이는 현상을 발생시킬 수 있다. 따라서 자동화된 배포는 보안 애플리케이션을 시작하기 전에 산출물 신원, 서명, 경로, 유효 기간 및 정책 버전을 검증해야 한다.

보안 로깅(Security Logging)은 이러한 장애를 진단하는 데 필요한 관찰 가능성(Observability)을 제공한다. 인증 거부, 권한 거부, 유효하지 않은 서명, 인증서 만료, 암호화 협상 문제 및 구성 오류를 일반적인 패킷 손실이나 네트워크 연결 장애와 구분할 수 있어야 한다. 로그는 어떤 보안 단계에서 장애가 발생했는지를 식별할 수 있을 만큼 충분한 정보를 포함해야 하지만 개인 키 또는 기타 민감한 자격 증명 자료를 노출해서는 안 된다. 중앙 집중식 로깅(Centralized Logging)은 다중 로봇 배포에서 특히 유용하다.

성능 평가(Performance Evaluation)는 전체 DDS 보안 구성이 활성화된 상태에서 수행해야 한다. 인증은 참여자 시작과 디스커버리에 영향을 주고, 암호화는 CPU 자원을 소비하며, 무결성 보호는 추가적인 처리를 요구하고, 보안 메타데이터(Security Metadata)는 네트워크 트래픽을 증가시킬 수 있다. 실제 영향은 메시지 크기, 발행 주기, 프로세서 성능, 미들웨어 구현체 및 네트워크 토폴로지에 따라 달라진다. 카메라 및 LiDAR 스트림은 저주기 제어 또는 진단 메시지와 서로 다른 오버헤드 특성을 나타낼 수 있다.

실시간 ROS 2 시스템(Real-Time ROS 2 System)은 평균 지연 시간만으로 보안 영향을 설명할 수 없으므로 추가적인 주의가 필요하다. 엔지니어는 지연 시간 분포(Latency Distribution), 지터(Jitter), CPU 사용률, 스케줄링 간섭(Scheduling Interference), 디스커버리 시간 및 부하 상태의 통신 동작을 측정해야 한다. PREEMPT_RT, 실행기 구성(Executor Configuration), 메모리 관리 및 DDS QoS 정책이 운영 아키텍처에 포함된다면 보안 동작도 이들과 함께 테스트해야 한다. 보안과 실시간 성능은 독립적으로 검증할 수 없다.

장애 모드 테스트(Failure-Mode Testing)는 의도적으로 잘못된 구성을 적용하여 수행해야 한다. 유효하지 않은 인증서, 일치하지 않는 키, 만료된 자격 증명, 수정된 거버넌스 파일, 승인되지 않은 발행자, 잘못된 엔클레이브 할당 또는 호환되지 않는 보안 설정을 사용하고 DDS 보안이 이를 예측 가능한 방식으로 거부하는지 확인할 수 있다. 이러한 부정 테스트(Negative Test)는 올바르게 구성된 참여자들이 통신할 수 있다는 사실만 확인하는 것이 아니라 플러그인 집행이 실제로 통신 경계를 보호하고 있음을 검증한다.

다중 로봇 배포(Multi-Robot Deployment)는 구성 규모와 보안상의 영향을 모두 증가시킨다. 각 로봇은 여러 엔클레이브를 포함하면서 동시에 플릿 관리자(Fleet Manager), 운영자 스테이션(Operator Station), 엣지 컴퓨터(Edge Computer), 외부 게이트웨이(External Gateway)와 통신할 수 있다. 전체 플릿에서 제한 없는 신원을 재사용하면 구성은 단순해지지만 침해 발생 시 영향 범위가 커진다. 로봇별 및 기능별 신원, 제한적인 권한, 네트워크 분할, 적절한 DDS 도메인 경계를 결합하면 접근 가능한 공격 표면(Attack Surface)을 줄일 수 있다.

따라서 보안 구성(Security Configuration)은 각 소프트웨어 릴리스와 연계된 버전 관리형 인프라(Version-Controlled Infrastructure)로 유지해야 한다. 정책 소스, 거버넌스 정의, 엔클레이브 매핑(Enclave Mapping), 배포 스크립트 및 검증 절차는 검토하고 재현할 수 있도록 관리하면서 개인 키는 일반적인 소스 관리 시스템 외부에 유지해야 한다. ROS 인터페이스가 변경되면 새롭게 추가된 토픽, 서비스 또는 액션이 보안 시스템에서 동작하기 위해 명시적인 권한을 요구할 수 있으므로 이에 대응하는 보안 정책 검토도 수행해야 한다.

궁극적으로 DDS 보안 플러그인 구성(DDS Security Plugin Configuration)은 SROS2 정책과 공개 키 기반 구조(Public Key Infrastructure, PKI) 산출물을 실제 런타임 보호(Runtime Protection)로 변환한다. 인증(Authentication)은 누가 신뢰된 참여를 시작할 수 있는지를 결정하고, 접근 제어(Access Control)는 해당 신원이 무엇을 통신할 수 있는지를 결정하며, 암호화 메커니즘(Cryptographic Mechanism)은 보호된 정보가 어떻게 교환되는지를 결정한다. 신뢰할 수 있는 ROS 2 보안은 이러한 플러그인을 하나의 통합 시스템으로 구성하고 보호된 자격 증명, 통제된 배포, 진단, 성능 검증 및 지속적인 보안 테스트와 결합할 때 구현된다.

## 06.05 ROS2 Communication Encryption Performance Impact

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 통신에서 암호화(Encryption)를 적용하면 일반적인 DDS 데이터 경로(Data Path)에 보안 처리가 추가되므로 측정 가능한 성능 비용이 발생한다. SROS2와 DDS 보안(DDS Security)이 활성화되면 참여자(Participant)는 상대방을 인증하고, 암호학적 변환(Cryptographic Transformation)을 적용하며, 무결성 정보(Integrity Information)를 계산하고, 추가적인 보안 메타데이터(Security Metadata)를 처리할 수 있다. 이러한 오버헤드는 지연 시간(Latency), 지터(Jitter), CPU 사용률, 메모리 동작, 네트워크 대역폭 및 디스커버리 시간(Discovery Time)에 영향을 주므로 실시간 로봇 시스템에서 보안 성능은 중요한 아키텍처 고려사항이 된다.

암호화 오버헤드(Encryption Overhead)의 크기는 하나의 보편적인 백분율로 표현할 수 없다. 프로세서 아키텍처(Processor Architecture), DDS 구현체, 암호화 알고리즘(Cryptographic Algorithm), 페이로드 크기(Payload Size), 발행 주기(Publication Frequency), 구독자 수, 네트워크 토폴로지(Network Topology), 활성화된 보호 범위(Protection Scope)에 따라 달라진다. 작은 메시지를 사용하는 워크스테이션에서 측정한 벤치마크를 카메라나 LiDAR 스트림을 전송하는 임베디드 제어기에 직접 적용할 수 없다. 따라서 보안 성능은 실제 목표 시스템 구성에서 평가해야 한다.

메시지 크기(Message Size)는 관찰되는 성능 비용에 큰 영향을 준다. 작은 제어 메시지의 경우 절대적인 처리 시간은 작더라도 고정적인 암호화 및 미들웨어 처리 비용이 전체 전송 지연에서 상당한 비율을 차지할 수 있다. 대용량 센서 메시지는 훨씬 많은 데이터를 암호화해야 하므로 CPU와 메모리 대역폭 소비가 증가한다. 따라서 명령 트래픽(Command Traffic), 텔레메트리(Telemetry), 영상, 포인트 클라우드(Point Cloud), 지도(Map)는 동일한 DDS 보안 설정에서도 서로 다른 성능 특성을 나타낼 수 있다.

메시지 주기(Message Frequency) 역시 중요하다. 발행 속도가 증가하면 암호학적 연산이 누적되기 때문이다. 저주기 진단 토픽(Diagnostic Topic)은 시스템 부하를 거의 증가시키지 않을 수 있지만 초당 수백 또는 수천 개의 보안 메시지를 처리하면 상당한 프로세서 부하가 발생할 수 있다. 고주기 제어 루프(High-Frequency Control Loop)는 추가 처리 시간이 사용 가능한 타이밍 예산(Timing Budget)의 일부를 소비할 수 있어 특히 민감하다. 따라서 벤치마크에서는 실제 메시지 크기뿐 아니라 실제 발행 주기도 재현해야 한다.

종단 간 지연 시간(End-to-End Latency)은 가능한 경우 애플리케이션 관점에서 측정해야 한다. 중요한 값은 단순히 암호화 연산 자체의 실행 시간이 아니라 발행(Publication), 직렬화(Serialization), DDS 처리, 보안 변환, 전송, 수신 측 검증, 역직렬화(Deserialization), 콜백 실행(Callback Execution)을 모두 포함하는 전체 지연이다. 동일한 조건에서 보안 구성과 비보안 구성(Unsecured Configuration)을 비교하면 전체 ROS 2 통신 경로를 유지하면서 보안 기능이 추가하는 순수한 성능 비용을 확인할 수 있다.

평균 지연 시간(Average Latency)만으로는 실시간 분석에 충분하지 않다. 로봇 제어 시스템은 평균값의 작은 변화보다 지연 변동(Latency Variation)과 최악 조건의 동작에 더 민감한 경우가 많다. 암호화와 무결성 연산은 스레드 스케줄링(Thread Scheduling), 캐시 동작(Cache Activity), 메모리 할당, 미들웨어 큐(Middleware Queue), 경쟁 워크로드와 상호작용할 수 있다. 따라서 보안으로 인한 타이밍 변동이 평균값에 가려지지 않도록 지연 분포, 백분위수(Percentile), 최대 관찰 지연 및 지터를 함께 측정해야 한다.

CPU 사용률(CPU Utilization)은 암호화 영향을 평가하는 또 다른 중요한 지표이다. 암호학적 처리는 송신 시스템과 수신 시스템 모두에서 실행 자원을 소비하며, 트래픽 양과 보호되는 엔드포인트 수가 증가할수록 그 영향도 커진다. 현대 프로세서는 일반적인 암호 연산을 위한 하드웨어 가속(Hardware Acceleration)을 제공하여 오버헤드를 크게 줄일 수 있다. 반면 임베디드 프로세서나 이미 높은 부하로 동작하는 엣지 컴퓨터(Edge Computer)는 여유 자원이 적기 때문에 동일한 보안 구성이 애플리케이션 스케줄링에 더 큰 영향을 줄 수 있다.

메모리 동작(Memory Behavior)도 보안 통신 성능에 영향을 줄 수 있다. DDS 처리는 기본적으로 직렬화 버퍼(Serialization Buffer), 미들웨어 큐, 전송 버퍼를 필요로 하며, 보안 메커니즘은 추가적인 임시 데이터 또는 데이터 변환을 발생시킬 수 있다. 추가 복사와 메모리 할당은 메모리 대역폭 요구량을 증가시키고 타이밍 변동을 발생시킬 수 있다. 따라서 결정론적 실행(Deterministic Execution)을 목표로 하는 시스템에서는 암호화 비용을 단순한 CPU 산술 연산 문제로 보지 말고 메모리 할당과 데이터 이동도 함께 분석해야 한다.

보호된 DDS 통신은 애플리케이션 페이로드 외에도 보안 관련 메타데이터를 전달하므로 네트워크 대역폭(Network Bandwidth)이 증가할 수 있다. 대용량 메시지에서는 이러한 오버헤드가 페이로드에 비해 상대적으로 작을 수 있지만 작고 빈번한 메시지에서는 비율상 더 크게 나타날 수 있다. 여러 구독자가 보안 데이터를 수신하면 네트워크 부하가 더욱 증가할 수 있다. 따라서 대역폭 측정에서는 ROS 메시지의 명목상 페이로드 크기만이 아니라 실제 전체 패킷 트래픽(Packet Traffic)을 고려해야 한다.

디스커버리 및 시작 성능(Discovery and Startup Performance)은 정상 상태 데이터 전송(Steady-State Data Transfer)과 별도로 평가해야 한다. 보안 DDS 참여자는 일반 통신을 시작하기 전에 신원을 확인하고 보호된 관계를 설정해야 한다. 인증 교환(Authentication Exchange), 인증서 검증(Certificate Verification), 보안 핸드셰이크(Security Handshake)는 참여자 디스커버리 시간을 증가시킬 수 있다. 지속적으로 실행되는 시스템에서는 영향이 작을 수 있지만 노드가 자주 재시작되거나 로봇이 플릿(Fleet)에 동적으로 합류하거나 장애 후 빠른 재연결이 요구되는 시스템에서는 중요한 요소가 된다.

보호 범위(Protection Scope)는 성능과 직접적인 관계를 갖는다. DDS 보안은 거버넌스 구성(Governance Configuration)에 따라 참여자 상호작용, 디스커버리 정보, 메타데이터 및 사용자 데이터에 보호를 적용할 수 있다. 더 강력한 보호는 일반적으로 더 많은 보안 처리를 요구하지만 실제 비용은 구현체에 따라 달라진다. 목표는 단순히 성능을 위해 보호 기능을 해제하는 것이 아니라 어떤 정보에 기밀성, 무결성 또는 인증이 필요한지 판단하고 시스템 위협 모델(Threat Model)에 맞는 제어를 적용하는 것이다.

서로 다른 DDS 및 RMW 구현체(RMW Implementation)는 유사한 보안 개념을 지원하더라도 서로 다른 성능 특성을 나타낼 수 있다. 내부 버퍼링(Buffering), 스레딩 모델(Threading Model), 직렬화 경로, 전송 구성 및 암호화 통합 방식이 측정된 오버헤드에 영향을 준다. 따라서 결과의 재현성과 의미를 확보하려면 보안 벤치마크에 정확한 ROS 2 배포판(Distribution), RMW 구현체, DDS 공급자와 버전, 운영체제, 커널 구성, 하드웨어 플랫폼 및 보안 설정을 명시해야 한다.

DDS 서비스 품질(Quality of Service, QoS) 설정도 암호화 통신과 상호작용한다. 신뢰성(Reliability), 히스토리 깊이(History Depth), 내구성(Durability), 데드라인(Deadline) 등의 QoS 동작은 큐잉(Queueing), 재전송(Retransmission), 메모리 소비 및 트래픽 패턴을 변경할 수 있다. 최선형(Best-Effort) 통신에서 측정된 암호화 오버헤드는 패킷 손실이나 혼잡이 발생한 신뢰형(Reliable) 통신과 다를 수 있다. 따라서 단순화된 벤치마크 기본값이 아니라 실제 운영 로봇에서 사용할 QoS 프로파일로 보안 성능을 평가해야 한다.

실시간 운영체제 구성(Real-Time Operating-System Configuration) 역시 결과 해석에 영향을 준다. PREEMPT_RT, CPU 친화도(CPU Affinity), 스레드 우선순위(Thread Priority), 인터럽트 처리(Interrupt Handling), 실행기 스케줄링(Executor Scheduling)은 추가적인 보안 처리가 허용할 수 없는 간섭을 발생시키는지를 결정할 수 있다. 평균적으로 비용이 작은 암호화 연산도 중요 콜백과 동일한 CPU 코어에서 실행되면 해당 콜백을 지연시킬 수 있다. 따라서 보안 벤치마크는 전체 실시간 스케줄링 아키텍처와 통합하여 수행해야 한다.

유용한 벤치마크 방법론(Benchmark Methodology)은 다른 모든 변수를 동일하게 유지하면서 비보안 기준선(Unsecured Baseline)과 하나 이상의 보안 구성을 비교하는 것이다. 동일한 페이로드, 메시지 주기, QoS 프로파일, 네트워크 조건, 실행기 및 하드웨어를 사용해야 한다. 이를 통해 지연, 지터, 처리량(Throughput), CPU 사용률, 메모리 사용량, 네트워크 트래픽 및 디스커버리 시간의 변화를 정량화할 수 있다. 지속적인 보안 오버헤드와 일반적인 시스템 잡음(System Noise)을 구분하려면 반복 시험과 충분한 측정 시간이 필요하다.

벤치마크 워크로드(Benchmark Workload)는 하나의 합성 토픽(Synthetic Topic)이 아니라 여러 통신 클래스를 표현해야 한다. 작고 고주기인 제어 메시지, 중간 크기의 텔레메트리 데이터, 서비스 상호작용, 대용량 카메라 또는 포인트 클라우드 페이로드는 통신 스택의 서로 다른 부분에 부하를 준다. 다중 발행자(Multi-Publisher) 및 다중 구독자(Multi-Subscriber) 테스트는 단순한 일대일 시험에서 발견하기 어려운 경합(Contention)을 보여줄 수 있다. 대표성 있는 워크로드가 하나의 "ROS 2 암호화 오버헤드" 수치보다 더 유용한 엔지니어링 근거를 제공한다.

운영 로봇은 통신 벤치마크만 단독으로 실행하는 경우가 거의 없으므로 시스템 부하(System Load)도 고려해야 한다. 인지 추론(Perception Inference), 위치 추정(Localization), 경로 계획(Planning), 로깅, 시각화 및 장치 드라이버(Device Driver)가 CPU, 메모리, 네트워크 자원을 경쟁적으로 사용한다. 따라서 암호화 테스트에는 유휴 시스템(Idle System) 측정과 실제 동시 워크로드(Concurrent Workload) 측정을 모두 포함해야 한다. 두 결과의 차이를 통해 로봇이 예상 운영 자원 사용률에 접근했을 때 보안 처리가 여전히 예측 가능한지를 판단할 수 있다.

고대역폭 인지 파이프라인(High-Bandwidth Perception Pipeline)에서는 필요한 보안 경계를 약화시키지 않으면서 불필요한 보안 처리를 줄일 수 있도록 아키텍처를 설계할 수 있다. 신뢰된 로컬 컴퓨팅 경계 내부에 유지되는 대용량 센서 스트림은 컴퓨터 간 또는 외부 네트워크를 통해 전송되는 명령과 서로 다른 보호 요구사항을 가질 수 있다. 이러한 결정은 성능상의 편의가 아니라 위협 분석과 배포 가정(Deployment Assumption)에 근거해야 한다. 네트워크 분할과 호스트 격리(Host Isolation)는 통신 수준의 암호학적 보호를 보완할 수 있다.

다중 로봇 시스템(Multi-Robot System)은 보안 관계와 통신 흐름의 수가 크게 증가할 수 있으므로 암호화 비용을 확대한다. 플릿 관리자(Fleet Manager), 엣지 서버(Edge Server), 운영자 스테이션(Operator Station), 로봇은 상태, 임무, 지도, 텔레메트리 및 협업 정보를 동시에 교환할 수 있다. 따라서 하나의 발행자와 하나의 구독자만 시험하면 실제 운영 동작을 과소평가할 수 있다. 플릿 규모 벤치마크에서는 참여자 수와 동시 트래픽을 증가시키면서 디스커버리 확장성, 프로세서 부하, 대역폭 및 지연 안정성을 관찰해야 한다.

성능 최적화(Performance Optimization)는 측정을 통해 실제 병목 지점(Bottleneck)을 확인한 이후 시작해야 한다. 가능한 개선 방법에는 하드웨어 암호화 가속, CPU 친화도, 미들웨어 튜닝(Middleware Tuning), QoS 조정, 불필요한 메시지 복사 감소, 고대역폭 트래픽 분리 또는 위협 모델이 허용하는 범위에서 보호 범위 조정 등이 있다. 하나의 벤치마크에서 지연 증가가 관찰되었다는 이유만으로 보안을 전체적으로 비활성화하는 것은 적절하지 않다. 최적화는 필요한 보안 속성을 유지하면서 타이밍 여유(Timing Margin)를 확보하는 방향으로 수행해야 한다.

보안 성능(Security Performance)은 용량 계획(Capacity Planning)에도 포함해야 한다. 실제 운영 시스템이 DDS 보안을 활성화할 예정이라면 CPU와 네트워크 자원을 비보안 ROS 2 동작만을 기준으로 산정해서는 안 된다. 암호화, 인증, 일시적인 트래픽 증가, 소프트웨어 업데이트 및 향후 워크로드 증가를 처리할 수 있는 충분한 여유를 확보해야 한다. 이는 인지 및 AI 추론이 이미 사용 가능한 계산 자원의 상당 부분을 소비할 수 있는 엣지 플랫폼에서 특히 중요하다.

미들웨어 업그레이드, ROS 2 배포판 변경, 보안 정책 수정, 운영체제 업데이트 또는 애플리케이션 구조 변경 이후에는 암호화 성능이 달라질 수 있으므로 회귀 테스트(Regression Testing)가 필요하다. 지속적인 테스트(Continuous Testing)에 통합된 벤치마크 모음은 지연, 지터, CPU 부하 또는 디스커버리 시간의 예상하지 못한 증가를 탐지할 수 있다. 따라서 보안 구성은 시스템 최적화가 완료된 이후에만 활성화하는 부가 기능이 아니라 성능 기준선(Performance Baseline)의 일부로 취급해야 한다.

궁극적으로 ROS 2 통신 암호화의 성능 영향(Performance Impact)은 필요한 보호 수준과 사용 가능한 타이밍 및 계산 자원 사이의 시스템 수준 절충(System-Level Tradeoff)이다. 올바른 엔지니어링 목표는 보안 오버헤드를 완전히 제거하는 것이 아니라 정의된 성능 예산(Performance Budget) 안에서 예측 가능한 수준으로 유지하는 것이다. 실제 보안 워크로드를 측정하고 처리량뿐 아니라 지연과 지터를 분석하며 전체 목표 플랫폼을 검증함으로써 로봇 시스템의 운영 요구사항을 희생하지 않고 SROS2 보안을 통합할 수 있다.

## 06.06 ROS2 Vulnerability Analysis: Known CVEs and Patches

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 취약점 분석(Vulnerability Analysis)은 로봇 시스템을 구성하는 소프트웨어, 미들웨어(Middleware), 운영체제(Operating System), 네트워크 및 배포 구성요소 전반의 약점을 체계적으로 식별하는 과정이다. 취약점(Vulnerability)은 ROS 2 패키지, DDS/RMW 구현체, 서드파티 라이브러리(Third-Party Library), 컨테이너(Container), 드라이버(Driver), 시스템 서비스(System Service) 등에 존재할 수 있다. 따라서 효과적인 분석에서는 ROS 2 애플리케이션 코드만 독립적으로 조사하는 것이 아니라 전체 소프트웨어 공급망(Software Supply Chain)을 보안 경계(Security Boundary)로 다루어야 한다.

공통 취약점 및 노출(Common Vulnerabilities and Exposures, CVE) 식별자는 공개적으로 보고된 보안 취약점에 대한 표준화된 참조 정보를 제공한다. CVE 기록은 엔지니어링 팀이 영향을 받는 소프트웨어 버전을 공급업체 보안 권고(Vendor Advisory), 심각도 정보(Severity Information), 패치(Patch), 완화 지침(Mitigation Guidance)과 연결하는 데 도움을 준다. 그러나 CVE 식별자 자체만으로 실제 로봇의 위험 수준이 결정되는 것은 아니다. 취약한 구성요소의 설치 여부, 활성화 상태, 접근 가능성, 영향받는 구성의 사용 여부 및 현실적인 공격자에게 노출되는지에 따라 실제 악용 가능성(Exploitability)이 달라진다.

취약점 관리(Vulnerability Management)는 정확한 소프트웨어 인벤토리(Software Inventory)에서 시작해야 한다. 인벤토리에는 ROS 2 배포판(Distribution), 패키지 버전, RMW 구현체, DDS 공급업체와 버전, 운영체제 패키지, 커널(Kernel), 장치 드라이버, 암호화 라이브러리(Cryptographic Library), 컨테이너 이미지, 개발 도구 및 외부에 노출된 서비스를 포함해야 한다. 이러한 기준 정보가 없으면 새롭게 공개된 CVE가 실제 배포된 로봇에 영향을 미치는지 또는 운영 구성에 존재하지 않는 소프트웨어에만 해당하는지를 신뢰성 있게 판단하기 어렵다.

소프트웨어 자재 명세서(Software Bill of Materials, SBOM)는 소프트웨어 구성요소와 의존성(Dependency)을 기계 판독 가능한 형태로 기록함으로써 이러한 과정을 강화한다. ROS 2 애플리케이션은 일반적으로 다수의 패키지에 의존하며, 해당 패키지들도 다시 운영체제 및 서드파티 라이브러리에 의존한다. 따라서 취약점은 직접 관리하는 ROS 패키지가 아니라 전이적 의존성(Transitive Dependency)을 통해 로봇에 유입될 수도 있다. SBOM을 유지하면 보안 팀이 취약점 공개 정보를 각 소프트웨어 릴리스의 실제 구성과 연결할 수 있다.

ROS 2 자체는 하위 통신 미들웨어와 함께 평가해야 한다. ROS 2는 일반적으로 ROS 미들웨어 인터페이스(ROS Middleware Interface)를 통해 DDS를 사용하므로 DDS 디스커버리(Discovery), 직렬화(Serialization), 전송(Transport), 보안 플러그인(Security Plugin), 자원 처리(Resource Handling)의 취약점이 애플리케이션 노드에 명백한 결함이 없더라도 ROS 2 통신에 영향을 줄 수 있다. 따라서 보안 평가에서는 선택한 ROS 2 배포판뿐 아니라 운영 환경에서 사용하는 특정 RMW 및 DDS 구현체의 보안 권고도 함께 추적해야 한다.

서드파티 의존성(Third-Party Dependency) 역시 동일한 수준의 주의가 필요하다. 영상 처리, 네트워킹, 압축(Compression), XML 파싱(Parsing), 암호화, 로깅, 데이터베이스 및 장치 통신을 위한 라이브러리는 ROS 2와 독립적으로 취약점을 포함할 수 있다. 로봇에 AI 프레임워크(AI Framework), 웹 대시보드(Web Dashboard), 원격 관리 도구 또는 클라우드 클라이언트(Cloud Client)를 통합하면 이러한 의존성 표면(Dependency Surface)은 더욱 확대된다. 따라서 취약점 분석은 기밀성(Confidentiality), 무결성(Integrity), 가용성(Availability) 또는 제어 동작에 영향을 줄 수 있는 모든 런타임 구성요소를 포함해야 한다.

알려진 CVE(Known CVE)는 심각도 점수만이 아니라 실제 배포 환경을 기준으로 우선순위를 결정해야 한다. 사용하지 않는 라이브러리의 높은 심각도 취약점보다 로봇 제어 시스템과 연결되어 원격 접근이 가능한 서비스의 중간 수준 취약점이 더 긴급한 운영 위험을 가질 수 있다. 우선순위 결정에서는 공격 벡터(Attack Vector), 필요한 권한, 사용자 상호작용, 익스플로잇(Exploit) 존재 여부, 네트워크 노출, 영향받는 기능, 안전상의 결과 및 기존 보완 통제(Compensating Control)가 취약 구성요소에 대한 접근을 제한하고 있는지를 고려해야 한다.

로봇 시스템에서는 네트워크 접근 가능성(Network Reachability)이 특히 중요하다. 로컬호스트(Localhost)에서만 접근 가능한 취약점은 Wi-Fi, 이더넷(Ethernet), VPN, 플릿 인프라(Fleet Infrastructure) 또는 인터넷에 연결된 게이트웨이(Internet-Facing Gateway)를 통해 접근 가능한 취약점과 서로 다른 위협 특성을 갖는다. 엔지니어는 외부 인터페이스와 신뢰 경계(Trust Boundary)를 매핑하여 공격자가 취약한 소프트웨어에 어떻게 접근할 수 있는지를 판단해야 한다. 네트워크 분할(Network Segmentation), 방화벽(Firewall), 제한된 포트, VPN 및 SROS2 권한 부여(Authorization)는 영구적인 수정이 준비되는 동안 노출을 줄일 수 있다.

패치 관리(Patch Management)는 취약점 정보를 실제 운영 위험 감소로 전환한다. 관련 보안 업데이트(Security Update)가 제공되면 영향을 받는 패키지를 식별하고 신뢰할 수 있는 출처에서 업데이트를 확보하며 호환성 요구사항을 검토하고 패치된 시스템을 실제 배포 전에 검증해야 한다. 로봇 소프트웨어는 실시간 제어(Real-Time Control), 미들웨어, 하드웨어 드라이버 및 AI 워크로드를 결합하는 경우가 많으므로 보안 문제를 올바르게 수정하는 패치라 하더라도 검증 없이 적용하면 기능 또는 타이밍 회귀(Timing Regression)가 발생할 수 있다.

따라서 패치 테스트(Patch Testing)는 사이버보안과 로봇 동작을 모두 포함해야 한다. 엔지니어는 취약점이 제거되거나 완화되었는지 확인하면서 ROS 2 디스커버리, 토픽(Topic), 서비스(Service), 액션(Action), QoS 동작, SROS2 인증(Authentication), 장치 통신, 제어 루프(Control Loop), 애플리케이션 시작을 함께 테스트해야 한다. 실시간 시스템에서는 보안 관련 업데이트 이후 지연 시간(Latency)과 지터(Jitter)도 추가로 측정해야 한다. 미들웨어 또는 커널 동작을 변경하는 패치는 애플리케이션 인터페이스가 그대로 유지되더라도 타이밍에 영향을 줄 수 있다.

모든 취약점을 즉시 패치할 수 있는 것은 아니다. 공급업체 업데이트가 아직 존재하지 않거나, 새로운 버전이 필요한 하드웨어와 호환되지 않거나, 운영 제약으로 인해 로봇을 즉시 중단할 수 없는 경우가 있다. 이러한 상황에서는 서비스 비활성화, 네트워크 격리(Network Isolation), 방화벽 규칙, 권한 제한, 기능 비활성화, 컨테이너 격리(Container Isolation), 강화된 모니터링 등을 통해 임시로 노출을 줄일 수 있다. 이러한 완화책은 임시 통제(Temporary Control)로 문서화하고 영구적인 수정이 제공되면 다시 검토해야 한다.

로봇이 배포된 이후에도 취약점 상태는 계속 변화하므로 보안 권고(Security Advisory)를 지속적으로 모니터링해야 한다. 릴리스 시점에 안전하다고 판단된 구성요소에서도 이후 새로운 CVE가 발견되거나 심각도 평가가 변경되거나 새로운 익스플로잇 정보가 공개될 수 있다. ROS 2 배포판과 운영체제 릴리스에도 정의된 지원 수명주기(Support Lifecycle)가 존재한다. 따라서 운영 시스템에서는 가능한 한 지원되는 소프트웨어를 사용하고 ROS, 미들웨어, 운영체제 및 의존성 관련 보안 정보를 지속적으로 확인하는 절차를 구축해야 한다.

취약점 스캐너(Vulnerability Scanner)는 설치된 패키지, 컨테이너 계층(Container Layer), SBOM 항목을 취약점 데이터베이스와 비교하여 이러한 작업의 일부를 자동화할 수 있다. 그러나 스캐너 결과를 실제 악용 가능성에 대한 직접적인 증거로 간주해서는 안 된다. 버전 정보가 불명확하거나 배포판이 예상되는 업스트림 버전 번호를 변경하지 않은 채 보안 수정 사항을 백포트(Backport)하는 경우 오탐(False Positive)이 발생할 수 있다. 따라서 발견된 항목은 공급업체 패키지 정보와 실제 배포 구성을 기준으로 검증해야 한다.

컨테이너 기반 ROS 2 시스템(Containerized ROS 2 System)은 취약점 관리 측면에서 장점과 추가적인 책임을 동시에 제공한다. 컨테이너 이미지는 재현 가능한 소프트웨어 기준선을 제공하여 취약점 스캔과 통제된 업데이트를 단순화할 수 있지만 오래된 베이스 이미지(Base Image)는 알려진 취약 패키지를 장기간 유지할 수 있다. 이미지는 유지보수되는 베이스를 기반으로 정기적으로 다시 빌드하고 릴리스 전에 스캔하며 변경 불가능한 버전 또는 다이제스트(Immutable Version or Digest)로 식별해야 한다. 운영 시스템이 로봇 시작 과정에서 통제되지 않은 이미지 변경을 자동으로 가져오는 방식은 피해야 한다.

취약점 분석에서는 취약점(Vulnerability), 노출(Exposure), 악용 가능성(Exploitability), 영향(Impact)을 구분해야 한다. CVE는 문서화된 약점을 나타내고, 노출은 영향받는 구성요소에 접근할 수 있는지를 설명하며, 악용 가능성은 현실적인 조건에서 해당 약점을 악용할 수 있는지를 나타낸다. 영향은 실제 악용에 성공했을 때 발생하는 결과를 의미한다. 이러한 개념을 구분하면 취약점 관리를 단순한 CVE 개수 집계 작업으로 만드는 것을 방지하고 기술적 발견 사항을 실제 로봇 시스템 위험과 연결할 수 있다.

로봇 시스템에서는 가용성(Availability)과 물리적 결과(Physical Consequence)에 특별한 주의를 기울여야 한다. 일반적인 정보 시스템의 취약점은 데이터를 노출하거나 서비스를 중단시킬 수 있지만 로봇 구성요소의 침해는 잠재적으로 내비게이션(Navigation), 모션(Motion), 인지(Perception), 액추에이터 동작에 영향을 줄 수 있다. 따라서 보안 우선순위 평가에서는 악용이 정보기술 영역에서 운영 또는 물리적 동작으로 확장될 수 있는지를 고려해야 한다. 명령 경로(Command Path)에 연결된 구성요소는 네트워크 노출이 제한적으로 보이더라도 특히 신중하게 검토해야 한다.

SROS2는 일부 통신 위험을 줄이지만 소프트웨어 취약점 자체를 제거하지는 않는다. 인증과 접근 제어는 승인되지 않은 DDS 참여자가 허용된 작업을 수행하는 것을 제한하고, 암호화는 통신의 기밀성과 무결성을 보호한다. 그러나 이러한 메커니즘이 메모리 안전성 결함(Memory-Safety Defect), 취약한 파서(Parser), 안전하지 않은 웹 서비스, 침해된 의존성 또는 운영체제 결함을 수정하지는 않는다. 따라서 취약점 관리와 통신 보안은 심층 방어(Defense in Depth) 아키텍처에서 서로 보완하는 계층으로 동작해야 한다.

패치 배포 이후에는 회귀 테스트(Regression Testing)가 필수적이다. 자동화된 테스트는 애플리케이션 기능, ROS 그래프(Graph) 동작, 보안 정책, 네트워크 인터페이스, 자원 소비 및 타이밍에 민감한 작업을 검증해야 한다. 부정 보안 테스트(Negative Security Test)를 통해 기존에 차단되었던 통신이 계속 차단되는지, 구성 변경으로 인해 보안 집행(Security Enforcement)이 비활성화되지 않았는지도 확인할 수 있다. 반복 가능한 회귀 테스트 모음을 유지하면 운영 로봇 시스템에서 빈번한 보안 유지보수를 훨씬 안전하게 수행할 수 있다.

로봇 플릿(Robot Fleet) 전체에 패치를 배포할 때는 통제된 롤아웃(Controlled Rollout)이 필요하다. 모든 로봇을 동시에 업데이트하면 예상하지 못한 호환성 문제가 발생했을 때 그 영향이 전체 플릿으로 확대될 수 있다. 단계적 접근 방식(Staged Approach)을 사용하면 개발 시스템, 하드웨어 인더루프(Hardware-in-the-Loop) 환경, 일부 파일럿 로봇(Pilot Robot), 점차 확대되는 배포 그룹에서 업데이트를 순차적으로 검증할 수 있다. 패치된 소프트웨어에서 허용할 수 없는 문제가 발생할 경우 운영 서비스를 복구할 수 있도록 롤백 패키지(Rollback Package)와 구성 백업도 배포 전에 준비해야 한다.

추적성(Traceability)은 각각의 취약점 관련 의사결정을 근거 자료와 연결해야 한다. 취약점 기록에는 영향받는 구성요소, CVE 식별자, 설치된 버전, 노출 평가(Exposure Assessment), 완화 조치, 패치 버전, 테스트 결과, 배포 날짜 및 책임자(Owner)를 기록할 수 있다. 이를 통해 특정 취약점이 패치되었는지, 임시로 완화되었는지, 위험이 수용되었는지 또는 시스템에 영향을 미치지 않는 것으로 판단되었는지를 설명하는 감사 가능한 이력(Auditable History)을 구축할 수 있다. 이러한 기록은 로봇 플릿과 소프트웨어 의존성이 증가할수록 더욱 중요해진다.

지속적 통합 및 배포 파이프라인(Continuous Integration and Deployment Pipeline)은 릴리스가 실제 로봇에 도달하기 전에 취약점 스캔, 의존성 검사, SBOM 생성, 정책 검증 및 회귀 테스트를 포함할 수 있다. 보안 게이트(Security Gate)는 허용할 수 없는 알려진 취약점을 포함하는 소프트웨어가 운영 환경으로 자동 진행되는 것을 차단할 수 있다. 그러나 취약점의 실제 관련성은 구성과 노출 상태에 따라 크게 달라지므로 자동화된 게이트는 모든 CVE를 무조건 거부하기보다 정의된 조직 기준과 검증된 근거를 사용해야 한다.

수명 종료 소프트웨어(End-of-Life Software)는 보안 패치가 더 이상 제공되지 않을 수 있기 때문에 구조적인 취약점 관리 문제를 발생시킨다. 지원이 종료된 ROS 배포판, 운영체제, 미들웨어 버전 또는 라이브러리를 계속 사용하면 해결되지 않은 취약점이 점차 누적될 수 있다. 따라서 수명주기 계획(Lifecycle Planning)에는 지원 종료 이전의 마이그레이션 전략(Migration Strategy)을 포함해야 한다. 기계적인 사용 수명이 원래 소프트웨어 플랫폼의 지원 기간보다 훨씬 길 수 있는 장기 운영 산업용 로봇(Long-Lived Industrial Robot)에서는 이러한 문제가 특히 중요하다.

궁극적으로 ROS 2 취약점 분석(ROS 2 Vulnerability Analysis)은 자산 인벤토리(Asset Inventory), CVE 모니터링, 노출 평가, 위험 우선순위 결정, 패치 검증, 통제된 배포 및 회귀 테스트를 연결하는 지속적인 엔지니어링 프로세스이다. 목표는 단순히 취약점 목록의 숫자를 줄이는 것이 아니라 로봇의 전체 수명주기 동안 방어 가능한 보안 상태(Defensible Security State)를 유지하는 것이다. SROS2, 네트워크 분할, 지원되는 소프트웨어, 적시 패치(Timely Patch), SBOM 기반 추적 및 검증된 운영 테스트를 결합하면 복원력 있는 ROS 2 배포(Resilient ROS 2 Deployment)를 위한 실용적인 기반을 구축할 수 있다.

## 06.07 Robot Network Segmentation and ROS2 Domain Isolation

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 네트워크 분할(Robot Network Segmentation)은 모든 장치가 하나의 제한 없는 네트워크를 통해 통신하도록 허용하는 대신, 로봇 통신 환경을 통제된 네트워크 영역(Network Zone)으로 분리하는 아키텍처 기법이다. ROS 2 시스템에서는 로봇, 센서, 제어 컴퓨터, 운영자 스테이션(Operator Station), 플릿 서버(Fleet Server), 개발 시스템 및 외부 서비스를 서로 분리할 수 있다. 이를 통해 불필요한 연결성을 줄이고 장애, 잘못된 구성 또는 악의적인 활동이 전파될 수 있는 경로를 제한할 수 있다.

ROS 2 통신은 일반적으로 분산 참여자(Distributed Participant) 사이의 DDS 디스커버리(DDS Discovery)와 데이터 교환에 의존한다. 의도적인 네트워크 경계가 없으면 서로 접근 가능한 인프라를 공유하는 참여자들이 원래 의도된 운영 범위를 넘어선 통신 엔드포인트(Communication Endpoint)를 발견할 수 있다. 네트워크 분할은 이러한 통신 관계 주변에 인프라 수준 경계(Infrastructure-Level Boundary)를 형성한다. ROS 2 도메인 격리(Domain Isolation)는 논리적 미들웨어 경계(Logical Middleware Boundary)를 추가하여 시스템 설계자가 로봇, 서브시스템, 임무, 환경 또는 보안 기능에 따라 참여자를 통신 그룹으로 구성할 수 있도록 한다.

ROS_DOMAIN_ID는 ROS 2 참여자가 사용하는 DDS 도메인(DDS Domain)을 식별한다. 서로 다른 도메인 식별자(Domain Identifier)를 사용하도록 구성된 노드는 일반적으로 서로 다른 DDS 디스커버리 공간(Discovery Space)에 참여하므로 일반적인 DDS 통신을 통해 서로를 직접 발견하지 않는다. 이는 개발 시스템, 로봇 그룹, 테스트 환경 또는 독립적인 애플리케이션을 분리하기 위한 편리한 메커니즘을 제공한다. 따라서 도메인 ID는 개별 개발자가 임의로 선택하기보다 의도적으로 설계된 시스템 아키텍처에 따라 할당해야 한다.

ROS 2 도메인 격리를 암호학적 보안 경계(Cryptographic Security Boundary)와 동일한 것으로 간주해서는 안 된다. 도메인 식별자는 DDS 통신을 구성하지만 그 자체로 참여자를 인증하거나, 충분한 권한을 가진 호스트가 설정을 변경하여 다른 도메인에 참여하는 것을 방지하지는 않는다. 더 강력한 격리를 위해서는 네트워크 분할, 방화벽 정책(Firewall Policy), SROS2 인증(Authentication), 접근 제어(Access Control), 보호된 자격 증명(Protected Credential), 보안 라우팅(Secure Routing)과 같은 보완 메커니즘이 필요하다. 따라서 도메인 분리는 심층 방어(Defense in Depth) 아키텍처의 한 계층이다.

물리적 및 논리적 네트워크 분할(Physical and Logical Network Segmentation)은 별도의 네트워크 인터페이스, 스위치(Switch), VLAN, 라우팅된 서브넷(Routed Subnet), 방화벽, 무선 네트워크 또는 소프트웨어 정의 네트워킹(Software-Defined Networking) 메커니즘을 사용하여 구현할 수 있다. 적절한 기술은 배포 규모와 운영 요구사항에 따라 달라진다. 소형 이동 로봇은 내부 센서 트래픽과 외부 플릿 통신을 분리할 수 있으며, 산업용 로봇 플릿은 여러 VLAN과 라우팅된 보안 영역을 사용하여 제어, 인지, 관리 및 기업 네트워크 연결을 격리할 수 있다.

유용한 분할 모델(Segmentation Model)은 안전 또는 제어에 민감한 트래픽을 상대적으로 신뢰도가 낮은 통신으로부터 분리하는 것이다. 모터 제어기(Motor Controller), 실시간 제어 컴퓨터, 위치 추정 시스템(Localization System), 중요 센서는 제한된 로봇 제어 영역(Robot-Control Zone)에 배치하고, 운영자 인터페이스, 모니터링 도구, 소프트웨어 저장소 또는 클라우드 게이트웨이(Cloud Gateway)는 다른 영역에 배치할 수 있다. 영역 간 통신은 명시적으로 통제된 인터페이스를 통과하도록 하여 범용 서비스가 모션 중요 구성요소(Motion-Critical Component)에 자동으로 무제한 네트워크 접근 권한을 갖지 못하도록 한다.

고대역폭 인지 트래픽(High-Bandwidth Perception Traffic) 역시 아키텍처적 분리를 통해 이점을 얻을 수 있다. 카메라, LiDAR, 깊이 센서(Depth Sensor), 인지 컴퓨터(Perception Computer)는 로컬 인지 네트워크 외부에서는 필요하지 않은 대량의 연속 트래픽을 생성할 수 있다. 이러한 스트림을 적절한 네트워크 세그먼트 내부에 유지하면 제어 및 플릿 네트워크의 혼잡을 줄이는 동시에 데이터 노출도 제한할 수 있다. 내비게이션, 모니터링 또는 상위 수준 지능(Higher-Level Intelligence)에 필요한 경우 모든 원시 센서 데이터 대신 선택된 인지 결과만 경계를 통과하도록 구성할 수 있다.

다중 로봇 시스템(Multi-Robot System)은 로봇 수가 증가함에 따라 디스커버리 트래픽과 잠재적인 통신 관계의 수가 모두 증가하므로 특히 신중한 도메인 계획(Domain Planning)이 필요하다. 직접적인 상호 디스커버리가 필요하지 않은 경우 로봇별 또는 로봇 그룹별로 독립적인 ROS 2 도메인을 할당할 수 있다. 반대로 애플리케이션이 기본 DDS 상호작용을 필요로 하는 경우 동일한 도메인을 공유하면서 네임스페이스(Namespace)와 SROS2 권한(Permission)을 이용해 보다 세밀하게 제어할 수 있다. 적절한 선택은 단순한 로봇 수가 아니라 필요한 통신 패턴(Communication Pattern)에 따라 결정해야 한다.

네임스페이스(Namespace)와 도메인 ID(Domain ID)는 서로 다른 아키텍처 문제를 해결한다. \`/robot1\` 또는 \`/robot2\`와 같은 네임스페이스는 ROS 그래프(Graph)의 이름을 구성하고 애플리케이션이 서로 다른 로봇에 속하는 인터페이스를 구별할 수 있도록 한다. 도메인 ID는 더 낮은 계층에서 DDS 디스커버리 공간을 분리한다. 두 로봇이 서로 다른 네임스페이스를 사용하면서 동일한 DDS 도메인에 존재할 수도 있고, 유사한 인터페이스 이름을 사용하면서 서로 다른 도메인에서 동작할 수도 있다. 따라서 두 메커니즘은 독립적으로 설계한 후 의도적으로 결합해야 한다.

SROS2는 인증되고 권한이 부여된 통신 경계(Authenticated and Authorized Communication Boundary)를 구축하여 네트워크 및 도메인 격리를 보완한다. 네트워크 분할은 어떤 시스템이 서로 접근할 수 있는지를 제어하고, 도메인 격리는 어떤 DDS 참여자들이 일반적으로 동일한 디스커버리 공간을 공유하는지를 제어하며, SROS2는 인증된 신원이 특정 통신 작업을 수행할 권한이 있는지를 결정한다. 이러한 계층을 결합하면 하나의 메커니즘에 대한 의존성을 크게 줄이고 승인되지 않은 참여 또는 횡적 이동(Lateral Movement)에 대해 더 강력한 보호를 제공할 수 있다.

방화벽(Firewall)은 로봇 네트워크 영역 사이에서 명시적인 트래픽 정책(Traffic Policy)을 집행할 수 있다. 서브넷 사이에 임의의 통신을 허용하는 대신 아키텍처에서 요구되는 프로토콜, 주소, 통신 방향 및 서비스만 허용하도록 규칙을 설정할 수 있다. ROS 2의 방화벽 설계에서는 디스커버리 및 전송 구성을 포함하여 선택된 DDS 구현체의 동작을 고려해야 한다. 지나치게 제한적인 필터링은 원인을 파악하기 어려운 ROS 디스커버리 또는 통신 장애처럼 나타날 수 있으므로 실제 운영 미들웨어(Production Middleware)를 사용하여 규칙을 검증해야 한다.

DDS 디스커버리 동작(DDS Discovery Behavior)은 트래픽이 분할된 네트워크를 통과할 때 중요한 고려사항이다. 멀티캐스트 디스커버리(Multicast Discovery)는 로컬 네트워크 내부에서는 자연스럽게 동작할 수 있지만 특정 인프라 또는 미들웨어 설정이 없으면 라우팅 경계를 통과하지 못할 수 있다. 대규모 배포에서는 필요한 경우 통제된 디스커버리 메커니즘, 디스커버리 서버(Discovery Server), 구성된 피어(Configured Peer) 또는 공급업체가 지원하는 다른 방식을 사용할 수 있다. 목표는 모든 네트워크 세그먼트에 제한 없는 멀티캐스트를 확장하는 것이 아니라 의도된 통신 토폴로지(Communication Topology)에 따라 디스커버리가 이루어지도록 하는 것이다.

세그먼트 간 라우팅(Routing)은 명시적인 신뢰 전환(Trust Transition)으로 취급해야 한다. 로봇 제어 네트워크와 플릿 네트워크를 연결하는 게이트웨이(Gateway)는 해당 경계를 넘어 반드시 필요한 정보와 서비스만 노출해야 한다. 게이트웨이는 라우팅, 필터링(Filtering), 프로토콜 중재(Protocol Mediation), 모니터링 또는 애플리케이션 수준 포워딩(Application-Level Forwarding)을 수행할 수 있다. 영역 간 통신을 정의된 경로에 집중시키면 관찰 가능성(Observability)이 향상되고 어떤 구성요소가 안전 중요 또는 운영 기능에 영향을 줄 수 있는지 판단하기 쉬워진다.

가능한 경우 관리 트래픽(Management Traffic)은 일반적인 로봇 애플리케이션 통신과 분리해야 한다. 소프트웨어 업데이트, 원격 관리(Remote Administration), SSH 접근, 진단, 로깅 및 구성 관리는 강력한 기능을 제공하지만 모든 운영 인터페이스에 지속적으로 접근할 필요는 없을 수 있다. 전용 관리 영역(Management Zone) 또는 통제된 관리 경로를 사용하면 유지보수 기능을 유지하면서 노출을 줄일 수 있다. 관리 서비스 역시 강력한 인증을 사용해야 하며 단순히 네트워크 위치만을 신뢰의 증거로 사용해서는 안 된다.

개발 및 테스트 환경(Development and Test Environment)은 일반적으로 운영 로봇 네트워크(Production Robot Network)와 분리해야 한다. 개발자 노트북에는 실험용 패키지, 디버깅 도구, 임시 서비스 및 지속적으로 변경되는 구성이 포함되는 경우가 많아 실제 배포 시스템과 다른 위험 특성을 갖는다. ROS 2 디스커버리는 테스트 노드가 운영 토픽 이름을 사용할 경우 의도하지 않은 상호작용을 특히 위험하게 만들 수 있다. 별도의 네트워크 세그먼트와 ROS 도메인은 실험용 발행자 또는 서비스가 운영 로봇에 의도하지 않은 영향을 미치는 것을 방지하는 데 도움이 된다.

외부 연결(External Connectivity)은 또 하나의 명확한 경계를 필요로 한다. 로봇은 클라우드 플랫폼, 기업 시스템, 원격 운영자 또는 서드파티 서비스와 통신할 수 있지만 이러한 외부 연결이 내부 ROS 2 네트워크에 제한 없는 접근 경로를 제공해서는 안 된다. 게이트웨이, VPN, 애플리케이션 프록시(Application Proxy), 방화벽 규칙 및 인증된 인터페이스를 이용하여 외부 접근을 제한할 수 있다. 로봇으로 들어오거나 외부로 나가는 데이터는 기본적으로 투명하게 전달하기보다 운영상의 필요성과 보안 정책에 따라 명확하게 식별해야 한다.

무선 통신(Wireless Communication)은 무선 연결이 물리적 케이블 경계를 넘어 확장되므로 추가적인 분할 요구사항을 갖는다. 플릿 통신, 유지보수, 게스트 접근 또는 개발에 사용되는 Wi-Fi 네트워크가 자동으로 동일한 신뢰 수준을 공유해서는 안 된다. 적절한 네트워크 분리, 인증, 암호화 및 라우팅 제어를 통해 하나의 무선 서비스에 대한 접근이 로봇 제어 시스템으로의 직접적인 접근으로 이어지는 것을 방지할 수 있다. 무선 장애와 로밍(Roaming) 동작 역시 실제 운영 조건에서 테스트해야 한다.

분할 아키텍처(Segmentation Architecture)는 사이버보안뿐 아니라 장애 격리(Failure Containment)도 고려해야 한다. 과도한 브로드캐스트(Broadcast) 또는 멀티캐스트 트래픽, 오동작 장치, 잘못된 DDS 구성, 소프트웨어 루프(Software Loop)는 공격자가 존재하지 않더라도 공유 네트워크의 성능을 저하시킬 수 있다. 트래픽 도메인을 분리하면 이러한 장애의 전파를 제한할 수 있다. 비정상적인 센서 트래픽이 발생한 인지 네트워크가 모션 제어, 안전 통신 또는 플릿 협업에 필요한 대역폭을 반드시 소모하도록 해서는 안 된다.

따라서 분할 설계에는 성능 요구사항(Performance Requirement)도 포함해야 한다. 추가적인 라우팅, 방화벽 검사(Firewall Inspection), VPN 처리 또는 애플리케이션 게이트웨이는 지연 시간(Latency)과 지터(Jitter)를 증가시킬 수 있다. 중요한 제어 경로는 불필요한 네트워크 경계 통과를 피해야 하며, 영역 간 통신에는 측정 가능한 성능 예산(Performance Budget)을 설정해야 한다. 엔지니어는 실제 배포에 사용할 것과 동일한 분할 및 보안 메커니즘을 적용하여 처리량(Throughput), 지연 분포, 패킷 손실, 디스커버리 시간 및 복구 동작을 평가해야 한다.

시간 동기화 트래픽(Time Synchronization Traffic) 역시 명시적으로 고려해야 한다. 분산 로봇은 센서 융합(Sensor Fusion)과 협조 제어(Coordinated Control)를 위해 PTP, NTP, GNSS 기반 시간(GNSS-Derived Time) 또는 하드웨어 타임스탬핑(Hardware Timestamping)에 의존할 수 있다. 네트워크 분할은 시간 정보가 장치와 영역 사이에서 전달되는 방식에 영향을 줄 수 있다. 따라서 스위치, 라우터, VLAN 구성 및 경계 클록(Boundary Clock)을 시간 아키텍처에 포함하여 보안 분할로 인해 인지 또는 제어에 필요한 동기화 정확도가 의도하지 않게 저하되지 않도록 해야 한다.

모니터링(Monitoring)은 네트워크 경계를 제거하지 않으면서도 경계 전반에 대한 가시성(Visibility)을 제공해야 한다. 로그, 네트워크 텔레메트리(Network Telemetry), DDS 진단, 보안 이벤트 및 장치 상태 정보를 통제된 경로를 통해 중앙 모니터링 시스템으로 전달할 수 있다. 운영자는 연결 장애, 방화벽 거부, DDS 디스커버리 문제, 도메인 불일치(Domain Mismatch), SROS2 권한 부여 실패를 서로 구분할 수 있어야 한다. 이러한 관찰 가능성은 네트워크 분할의 복잡성이 증가할수록 더욱 중요해진다.

도메인 ID, VLAN 할당, IP 주소 지정, 방화벽 규칙, DDS 디스커버리 설정, 네임스페이스 및 SROS2 정책은 서로 일관성을 유지해야 하므로 구성 관리(Configuration Management)가 필수적이다. 대규모 플릿에서 이러한 설정을 수동으로 관리하면 의도하지 않은 연결이나 격리가 쉽게 발생할 수 있다. 버전 관리된 네트워크 정의(Version-Controlled Network Definition)와 자동화된 배포 절차를 사용하면 분할 구성을 재현 가능하게 만들 수 있다. 구성 변경은 개별적인 네트워크 설정 변경이 아니라 아키텍처 변경으로 검토해야 한다.

테스트(Testing)는 의도된 연결성과 의도된 격리를 모두 검증해야 한다. 긍정 테스트(Positive Test)는 필요한 ROS 2 토픽, 서비스, 액션, 디스커버리 경로, 관리 기능 및 플릿 통신이 승인된 경계를 통해 정상적으로 동작하는지 확인한다. 부정 테스트(Negative Test)는 격리되어야 하는 영역, 도메인 또는 신원 사이에서 통신을 시도한다. 예상된 통신이 정상적으로 동작한다는 사실만으로 분할 설계가 완전히 검증된 것은 아니며, 금지된 통신 경로가 실제로 사용할 수 없는 상태인지도 입증해야 한다.

궁극적으로 로봇 네트워크 분할(Robot Network Segmentation)과 ROS 2 도메인 격리(ROS 2 Domain Isolation)는 분산 로봇 지능(Distributed Robotic Intelligence) 주변에 구조화된 통신 경계(Structured Communication Boundary)를 형성한다. 네트워크 영역은 접근 가능성(Reachability)을 제어하고, ROS 도메인은 DDS 디스커버리를 구성하며, 네임스페이스는 애플리케이션 인터페이스를 구조화하고, SROS2는 신원 기반 인증과 권한 부여(Identity-Based Authentication and Authorization)를 제공한다. 이러한 메커니즘을 방화벽, 게이트웨이, 모니터링, 시간 동기화 및 배포 자동화와 통합하면 ROS 2 시스템에서 더 강력한 장애 격리, 감소된 공격 표면(Attack Surface), 예측 가능한 통신 및 확장 가능한 다중 로봇 격리(Scalable Multi-Robot Isolation)를 구현할 수 있다.

## 06.08 ROS2 Security Audit Log Collection [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2의 보안 감사 로깅(Security Audit Logging)은 로봇 시스템에서 누가 또는 무엇이 상호작용했는지, 해당 상호작용이 언제 발생했는지, 그리고 보안 통제(Security Control)가 이를 허용했는지 또는 거부했는지를 설명할 수 있는 구조화된 이벤트 기록을 제공한다. 일반적인 애플리케이션 로깅과 달리 보안 감사 로그(Security Audit Log)는 인증(Authentication), 권한 부여(Authorization), 구성 변경, 접근 시도, 통신 이상, 관리 작업 및 보안 관련 장애에 중점을 둔다. 목적은 탐지(Detection), 조사(Investigation), 책임 추적(Accountability) 및 사고 후 분석(Post-Incident Analysis)을 지원하는 것이다.

ROS 2 보안 감사 아키텍처(Security Audit Architecture)는 하나의 로깅 소스만으로 시스템 동작 전체를 파악할 수 없기 때문에 여러 계층에서 정보를 수집해야 한다. 관련 증거는 ROS 2 노드(Node), SROS2 보안 메커니즘, DDS 미들웨어(Middleware), 운영체제(Operating System), 네트워크 인프라, 컨테이너(Container), 게이트웨이(Gateway), 인증 서비스 및 로봇 관리 애플리케이션에서 생성될 수 있다. 이러한 소스를 상호 연관(Correlation)하면 개별적으로는 서로 관계없는 로컬 장애처럼 보이는 사건들을 하나의 연속된 이벤트로 재구성할 수 있다.

SROS2 및 DDS 보안(DDS Security) 이벤트는 분산 통신 주변에서 발생하는 보안 의사결정을 설명하므로 특히 중요하다. 유용한 기록에는 참여자 인증(Participant Authentication) 결과, 인증서 검증 실패(Certificate Validation Failure), 권한 부여 거부, 권한 불일치(Permission Mismatch), 보안 플러그인 오류 및 설정된 정책을 위반하는 통신 시도가 포함될 수 있다. 실제로 사용할 수 있는 정보는 DDS 구현체와 로깅 설정에 따라 달라지므로 감사 설계에서는 배포 플랫폼에서 어떤 미들웨어 이벤트를 관찰할 수 있는지 문서화해야 한다.

애플리케이션 수준 ROS 2 로그(Application-Level ROS 2 Log)는 보안 이벤트에 대한 운영적 맥락(Operational Context)을 제공한다. DDS 통신 거부 이벤트는 노드 시작, 수명주기 전환(Lifecycle Transition), 구성 재로딩, 서비스 요청 또는 예상하지 못한 프로세스 재시작과 연계하여 분석할 때 더 큰 의미를 가질 수 있다. 따라서 노드는 비밀 정보를 노출하지 않으면서 보안과 관련된 중요한 상태 변화를 기록해야 한다. 로그에는 중요한 이벤트와 결과를 기록하되 개인키(Private Key), 비밀번호, 토큰(Token), 전체 자격 증명 또는 새로운 보안 위험을 만들 수 있는 기타 민감한 정보를 포함하지 않아야 한다.

운영체제 로그(Operating-System Log)는 ROS 2보다 하위 계층의 가시성(Visibility)을 확장한다. 프로세스 생성, 사용자 로그인, 권한 변경, 서비스 시작, 패키지 설치, 장치 접근, 방화벽 이벤트, 커널 메시지 및 인증 실패는 ROS 그래프(Graph) 수준에서는 보이지 않는 활동을 보여줄 수 있다. 따라서 리눅스(Linux) 기반 로봇에서는 시스템 로깅 및 감사 메커니즘이 ROS 2와 DDS의 증거를 보완할 수 있다. 호스트 수준 기록(Host-Level Record)은 통신 이상이 침해된 프로세스 또는 잘못 구성된 프로세스에서 발생했는지를 조사할 때 특히 중요하다.

네트워크 텔레메트리(Network Telemetry)는 또 다른 필수적인 관점을 제공한다. 방화벽 로그, VPN 이벤트, 스위치 정보, 무선 연결 기록(Wireless Association Record), 플로 메타데이터(Flow Metadata) 및 선택적인 패킷 캡처(Packet Capture)는 통신이 어디에서 시작되었으며 어떤 네트워크 경계를 통과했는지를 보여줄 수 있다. 분할된 로봇 아키텍처에서는 이러한 기록을 이용해 DDS 구성 문제와 차단된 경로 또는 승인되지 않은 영역 간 연결을 구분할 수 있다. 무제한 패킷 캡처는 상당한 저장 공간을 사용하고 민감한 데이터를 노출할 수 있으므로 운영상의 필요에 따라 네트워크 증거를 수집해야 한다.

분산 구성요소 전반에서 타임스탬프(Timestamp)를 신뢰할 수 있고 일관되게 유지할 때 감사 기록의 활용 가치가 크게 높아진다. 로봇, 운영자 스테이션(Operator Station), 플릿 서버(Fleet Server), 게이트웨이 및 모니터링 시스템은 NTP, PTP, GNSS 기반 시간(GNSS-Derived Time) 또는 기타 통제된 기준을 이용한 적절한 시간 동기화 아키텍처(Time Synchronization Architecture)를 공유해야 한다. 일관된 시간이 없으면 서로 다른 장치에서 수집된 이벤트 순서가 잘못 표시되어 인증 실패가 재시작, 네트워크 중단 또는 제어 이상보다 먼저 발생했는지를 판단하기 어려워질 수 있다.

각 보안 이벤트(Security Event)는 불필요하게 장황해지지 않으면서도 상호 연관 분석에 충분한 맥락을 포함해야 한다. 유용한 필드에는 타임스탬프, 로봇 식별자, 호스트, 프로세스 또는 노드 신원(Node Identity), 보안 엔클레이브(Security Enclave), 이벤트 유형, 결과, 출발지 및 목적지 정보, 심각도(Severity), 관련 정책 참조가 포함될 수 있다. 로그를 자동으로 처리할 경우 구조화된 로깅 형식(Structured Logging Format)이 적합하다. 일관된 필드를 사용하면 중앙 시스템이 여러 로봇과 소프트웨어 구성요소에 걸쳐 검색, 필터링, 상호 연관, 집계 및 경보 생성을 수행할 수 있다.

로그 심각도(Log Severity)는 단순한 소프트웨어 구현 세부사항이 아니라 운영적 의미를 반영해야 한다. 일상적인 정상 인증은 정보 수준(Informational)으로 처리할 수 있고, 반복되는 권한 부여 실패는 경고(Warning) 또는 보안 이벤트로 분류할 수 있으며, 신뢰된 제어 통신에 영향을 주는 장애는 즉각적인 대응이 필요할 수 있다. 동일한 유형의 이벤트가 한 로봇에서는 사소한 메시지로 표시되고 다른 로봇에서는 심각한 경보로 표시되지 않도록 플릿 전체에서 심각도 정의를 표준화해야 한다.

중앙 집중식 로그 수집(Centralized Log Collection)은 분산 ROS 2 배포 환경에서 가시성을 향상시킨다. 개별 로봇에 저장된 파일에만 의존하는 대신 선택된 감사 기록을 온프레미스 모니터링 서버(On-Premise Monitoring Server), 플릿 관리 플랫폼, 보안 정보 및 이벤트 관리 시스템(Security Information and Event Management, SIEM) 또는 기타 보호된 저장소로 전달할 수 있다. 중앙 수집은 로봇이 사용할 수 없는 상태가 되더라도 증거를 보존하고 로봇 간 분석을 가능하게 하지만, 무선 또는 상위 네트워크 연결이 일시적으로 중단될 경우를 대비하여 로컬 버퍼링(Local Buffering)도 필요하다.

실용적인 로깅 파이프라인(Logging Pipeline)은 간헐적인 통신 장애를 견딜 수 있어야 한다. 이동 로봇은 Wi-Fi가 불안정한 영역을 통과하거나 액세스 포인트(Access Point) 사이를 전환하거나 외부 연결 없이 일시적으로 운용될 수 있다. 따라서 로컬 로깅 서브시스템은 중요한 기록을 버퍼링하고 연결이 복구되면 이를 전달해야 한다. 네트워크 장애로 인해 로그가 무제한으로 증가하거나 로봇 운영에 필요한 저장 공간을 소비하지 않도록 큐 제한(Queue Limit), 저장 공간 할당량(Storage Quota), 우선순위 및 재시도 동작을 정의해야 한다.

ROS 2 시스템은 많은 양의 진단 정보를 생성할 수 있기 때문에 저장 공간 관리(Storage Management)가 특히 중요하다. 모든 토픽 메시지 또는 성공한 모든 미들웨어 상호작용을 기록하는 것은 일반적으로 보안 감사 추적(Security Audit Trail)에 적합하지 않다. 수집 정책은 가치가 높은 보안 이벤트와 대용량 운영 텔레메트리를 구분해야 한다. 조사 요구사항, 규제 요구사항, 사용 가능한 저장 공간 및 플릿 규모에 따라 로그 순환(Log Rotation), 압축, 보존 기간(Retention Period), 아카이브 규칙 및 삭제 정책을 설정해야 한다.

공격자가 시스템 접근 권한을 획득한 후 증거를 삭제하거나 변경하려 할 수 있으므로 감사 로그 자체도 보호해야 한다. 파일 권한(File Permission), 제한된 관리자 접근, 보호된 원격 전송, 중앙 저장소, 추가 기록 중심 메커니즘(Append-Oriented Mechanism), 무결성 검증(Integrity Verification) 또는 기타 변조 방지(Tamper Resistance) 기술을 이용하여 로그의 신뢰성을 높일 수 있다. 또한 로깅 구성을 변경할 수 있는 사용자를 통제해야 한다. 감사 수집을 조용히 비활성화하거나 축소하는 행위는 이후의 사고 조사를 심각하게 방해할 수 있기 때문이다.

무결성(Integrity)과 동시에 기밀성(Confidentiality)도 고려해야 한다. 로그에는 로봇 식별자, 네트워크 주소, 소프트웨어 버전, 토폴로지(Topology) 정보, 사용자 이름, 운영 위치, 오류 세부정보 및 애플리케이션 데이터가 포함될 수 있다. 이러한 기록은 방어자에게 유용하지만 공격자에게도 가치 있는 정보를 제공할 수 있다. 따라서 감사 시스템은 불필요한 민감 정보를 최소화하고 역할(Role)에 따라 접근을 제한하며 전송 및 저장 과정에서 로그를 보호하고 포함된 데이터에 적절한 보존 정책을 정의해야 한다.

보안 모니터링(Security Monitoring)은 수집된 로그를 실행 가능한 탐지(Actionable Detection) 정보로 변환할 수 있다. 개별 이벤트 하나는 큰 의미가 없을 수 있지만 반복되는 패턴은 비정상 동작을 나타낼 수 있다. 반복적인 인증 실패, 예상하지 않은 네트워크 영역에서의 접근 시도, 비정상적인 인증서 오류, 반복적인 노드 재시작, 갑작스러운 정책 거부 또는 유지보수 시간 외의 관리 변경은 조사를 유발할 수 있다. 탐지 규칙(Detection Rule)은 일반적인 기업 보안 가정에만 의존하지 않고 로봇의 정상 운영 모델을 반영해야 한다.

ROS 2 그래프 및 미들웨어 관찰 정보도 이상 탐지(Anomaly Detection)에 활용할 수 있다. 예상하지 않은 참여자, 익숙하지 않은 노드 신원, 비정상적인 디스커버리 동작, 예상된 도메인 외부의 통신 시도 또는 발행자(Publisher)와 구독자(Subscriber) 관계의 변화는 구성 오류나 승인되지 않은 활동을 나타낼 수 있다. 그러나 동적인 ROS 2 시스템에서는 엔티티(Entity)가 정상적으로 생성되고 제거되므로 모니터링 로직은 과도한 오탐(False Alarm)을 방지하기 위해 예상되는 수명주기 동작을 이해해야 한다.

다중 로봇 배포(Multi-Robot Deployment)는 플릿 전체의 이벤트를 상호 연관함으로써 더 큰 이점을 얻을 수 있다. 한 로봇에서 발생한 인증서 오류는 로컬 구성 문제일 수 있지만 동일한 오류가 여러 로봇에서 동시에 발생한다면 만료된 자격 증명, 배포 실패, 인프라 문제 또는 보다 광범위한 보안 이벤트를 의미할 수 있다. 중앙 분석(Central Analysis)은 독립적인 로봇 로그보다 이러한 패턴을 효과적으로 식별할 수 있다. 따라서 로봇 식별자, 소프트웨어 버전, 배포 그룹 및 구성 리비전(Configuration Revision)을 검색 가능한 감사 맥락에 포함해야 한다.

감사 로깅은 취약점 및 패치 관리(Vulnerability and Patch Management)와 통합되어야 한다. CVE를 해결하기 위해 소프트웨어를 업데이트할 때 로그를 이용하여 새로운 버전이 배포되었는지, 서비스가 정상적으로 재시작되었는지, 보안 정책이 성공적으로 로딩되었는지, 업데이트 이후 예상하지 못한 장애가 발생하지 않았는지를 검증할 수 있다. 이후 사고가 발생할 경우 배포 및 구성 기록을 이용하여 해당 시점에 어떤 로봇이 영향받는 버전을 실행하고 있었으며 임시 완화 조치가 활성화되어 있었는지를 확인할 수 있다.

구성 변경(Configuration Change)은 별도의 감사 대상으로 취급해야 한다. SROS2 거버넌스(Governance) 또는 권한 파일(Permissions File), 인증서, ROS 도메인 설정, 방화벽 규칙, 네트워크 인터페이스, DDS 구성, 컨테이너, 실행 파일(Launch File) 및 권한이 높은 서비스의 변경은 로봇의 보안 상태(Security Posture)를 크게 변화시킬 수 있다. 중요한 구성이 언제 변경되었는지, 어떤 버전이 활성화되었는지, 그리고 해당 정보를 확인할 수 있는 경우 어떤 승인된 프로세스 또는 관리자가 변경을 시작했는지를 기록해야 한다.

감사 데이터(Audit Data)는 침투 테스트(Penetration Testing)와 보안 검증(Security Validation)에서도 유용하다. 통제된 테스트에서는 의도적으로 통신 거부, 인증 실패, 네트워크 정책 위반 또는 비정상 접근 패턴을 발생시킨 후 모니터링 아키텍처가 이를 기록하고 탐지하는지 확인할 수 있다. 이는 보안 통제 자체만을 시험하는 것이 아니라 방어자가 해당 이벤트를 실제로 관찰할 수 있는지도 검증한다. 공격이 차단되었더라도 사용할 수 있는 증거가 남지 않는다면 이는 중요한 모니터링 취약점(Monitoring Weakness)이 될 수 있다.

자동화된 테스트(Automated Testing)는 지속적 통합 및 배포(Continuous Integration and Deployment) 과정에서 감사 기능을 검증할 수 있다. 테스트 환경에서 대표적인 보안 이벤트를 생성하고 필요한 필드, 타임스탬프, 심각도 수준 및 정책 식별자가 올바르게 나타나는지 확인할 수 있다. 미들웨어 버전이나 로깅 구성의 변경으로 중요한 증거가 예고 없이 사라질 수도 있다. 감사 동작을 테스트 가능한 시스템 요구사항(Testable System Requirement)으로 취급하면 ROS 2 소프트웨어 스택이 발전하더라도 모니터링 기능을 지속적으로 유지할 수 있다.

사고 조사(Incident Investigation)에서는 관련 증거를 보존해야 한다. 의심스러운 동작이 발생하면 관련 ROS 2 로그, DDS 보안 기록, 시스템 로그, 네트워크 이벤트, 구성 버전, 소프트웨어 인벤토리 및 배포 이력을 보존해야 한다. 문제 해결 과정에서 증거를 변경하기보다 정해진 절차에 따라 복사하고 보호해야 한다. 명확한 보존 및 접근 절차는 이후 근본 원인 분석(Root-Cause Analysis)과 보안 검토(Security Review)의 신뢰성을 향상시킨다.

감사 대시보드(Audit Dashboard)는 수집된 모든 이벤트를 동일하게 표시하기보다 운영 우선순위에 따라 정보를 제공해야 한다. 운영자는 로봇 상태와 즉각적인 보안 경보가 필요할 수 있으며, 보안 엔지니어는 인증 추세, 정책 위반, 네트워크 이상 및 과거 검색 기능이 필요하다. 개발자는 상세한 미들웨어 진단 정보를 필요로 할 수 있다. 이러한 화면을 분리하면 동일한 감사 인프라가 각 사용자 그룹에 불필요한 정보를 과도하게 제공하지 않으면서 운영, 사이버보안, 유지보수 및 엔지니어링을 모두 지원할 수 있다.

감사 로깅의 효과성(Effectiveness)은 주기적으로 검토해야 한다. 예상된 로그 소스가 계속 데이터를 보고하고 있는지, 시스템 시계가 동기화되어 있는지, 저장 용량이 충분한지, 보존 정책이 정상적으로 작동하는지, 경보가 책임자에게 전달되는지, 그리고 사용 가능한 증거를 통해 중요 이벤트를 재구성할 수 있는지를 확인해야 한다. 새로운 로봇 기능, 네트워크 세그먼트, DDS 구성 또는 외부 인터페이스가 추가되면 어떤 보안 이벤트를 추가로 수집해야 하는지도 함께 검토해야 한다.

궁극적으로 ROS 2 보안 감사 로그 수집(ROS 2 Security Audit Log Collection)은 분산된 보안 이벤트를 추적 가능한 운영 증거(Traceable Operational Evidence)로 변환한다. ROS 2, SROS2, DDS, 운영체제, 네트워크, 게이트웨이 및 관리 로그를 동기화된 타임스탬프, 구조화된 이벤트 형식, 보호된 저장소, 통제된 보존 정책 및 중앙 상호 연관 분석을 통해 결합해야 한다. 수집 기능이 모니터링, 경보(Alerting), 취약점 관리, 구성 추적(Configuration Tracking) 및 사고 대응(Incident Response)과 연결될 때 감사 로깅은 단순한 수동적 기록 저장소가 아니라 지속적으로 작동하는 보안 역량(Continuous Security Capability)이 된다.

## 06.09 ROS2 Security Test Automation Methodology

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 보안 테스트 자동화(Security Test Automation)는 로봇 소프트웨어, 미들웨어(Middleware), 구성(Configuration), 배포 환경이 변화하더라도 보안 통제(Security Control)가 지속적으로 유효한지를 반복 가능하게 검증하는 방법을 제공한다. 수동 보안 테스트(Manual Security Testing)는 설계 검토와 침투 평가(Penetration Assessment)에서 유용하지만 모든 소프트웨어 릴리스를 안정적으로 검증하기에는 한계가 있다. 자동화된 테스트는 중요한 보안 요구사항을 개발, 지속적 통합(Continuous Integration), 시스템 검증 및 플릿 배포(Fleet Deployment) 과정에서 실행할 수 있는 검증 항목으로 변환한다.

이 방법론은 로봇 보안 아키텍처(Robot Security Architecture)를 명시적인 테스트 요구사항(Test Requirement)으로 변환하는 것에서 시작한다. 각 요구사항은 승인된 노드의 성공적인 인증, 승인되지 않은 참여자의 거부, 금지된 토픽 접근의 차단, 민감한 통신의 보호 또는 네트워크 영역 간 격리와 같이 관찰 가능한 조건을 설명해야 한다. 테스트는 단순히 보안 구성 파일이 존재하는지를 확인하는 것이 아니라 실제로 측정할 수 있는 동작을 평가해야 한다.

보안 테스트는 긍정 동작(Positive Behavior)과 부정 동작(Negative Behavior)을 모두 포함해야 한다. 긍정 테스트(Positive Test)는 정상적인 ROS 2 노드가 필요한 피어(Peer)를 발견하고, 허용된 토픽을 교환하며, 승인된 서비스와 액션(Action)을 호출하고, 예상된 수명주기 작업(Lifecycle Operation)을 수행할 수 있는지를 검증한다. 부정 테스트(Negative Test)는 허용되지 않아야 하는 신원, 권한, 도메인 또는 네트워크 경로를 의도적으로 사용한다. 안전한 시스템은 필요한 통신뿐 아니라 정책을 위반하는 통신이 확실하게 거부되는 것도 입증해야 한다.

SROS2 인증(Authentication)은 통제된 조건에서 유효한 보안 신원(Security Identity)과 유효하지 않은 보안 신원을 가진 참여자를 실행하여 테스트할 수 있다. 자동화된 시나리오는 올바르게 프로비저닝(Provisioning)된 노드는 보안 통신을 설정하고, 누락되거나 잘못되었거나 만료되었거나 기타 허용할 수 없는 자격 증명(Credential)을 사용하는 노드는 예상된 정책에 따라 실패하는지를 검증할 수 있다. 테스트 결과에서는 인증 실패와 관련 없는 DDS 디스커버리(Discovery), 애플리케이션 시작 또는 네트워크 연결 문제를 구분해야 한다.

권한 부여 테스트(Authorization Testing)는 인증된 참여자가 할당된 권한으로 제한되는지를 확인하는 데 중점을 둔다. 특정 테스트 신원은 센서 토픽을 구독할 수 있지만 명령을 발행하는 것은 금지될 수 있으며, 진단 서비스 호출은 허용되지만 제어 서비스 접근은 거부될 수 있다. 자동화된 테스트는 허용된 작업과 금지된 작업을 모두 실행하고 관찰된 결과를 의도된 SROS2 권한 정책(Permissions Policy)과 비교해야 한다.

거버넌스(Governance) 및 권한 구성(Permissions Configuration)은 단순한 정적 배포 파일이 아니라 보안 산출물(Security Artifact)로 검증해야 한다. 자동화된 검사는 예상된 도메인, 토픽, 서비스, 액션, 참여자 및 보호 규칙이 올바르게 표현되어 있는지 확인하고, 의도하지 않은 와일드카드 권한(Wildcard Permission)으로 접근 범위가 확대되지 않았는지도 검증할 수 있다. 정책이 구조적으로 유효하더라도 아키텍처에서 요구하는 것보다 더 넓은 권한을 부여할 수 있으므로 구문 검증(Syntax Validation)만으로는 충분하지 않다.

통신 보호(Communication Protection)도 검증해야 한다. 보안 설계에서 암호화되거나 무결성이 보호된 DDS 트래픽을 요구하는 경우, 테스트는 예상된 보안 구성을 사용하여 보호된 통신이 설정되는지를 확인해야 한다. 격리된 테스트 환경에서 네트워크를 관찰하면 민감한 애플리케이션 페이로드(Application Payload)가 일반적인 읽기 가능한 트래픽으로 의도하지 않게 노출되지 않는지를 확인하는 데 도움이 된다. 이러한 검사는 불필요한 민감 운영 정보를 기록하지 않는 방식으로 수행해야 한다.

ROS 2 도메인 격리(Domain Isolation)와 네트워크 분할(Network Segmentation)은 전용 자동화 시나리오가 필요하다. 테스트 시스템은 참여자를 의도된 또는 의도되지 않은 ROS 도메인, VLAN, 서브넷(Subnet), 보안 영역(Security Zone)에 배치하고 예상된 접근 가능성(Reachability)을 검증할 수 있다. 승인된 경계를 통과하는 통신은 정의된 경로나 게이트웨이를 통해 동작해야 하며 금지된 영역 간 통신은 실패해야 한다. 이러한 테스트는 도메인 ID, 방화벽 규칙, 라우팅 및 DDS 디스커버리 설정의 구성 드리프트(Configuration Drift)를 탐지하는 데 도움이 된다.

애플리케이션 통신은 토픽 데이터 교환이 시작되기 전에 실패할 수 있으므로 디스커버리 동작(Discovery Behavior)을 별도로 테스트할 필요가 있다. 자동화된 테스트는 멀티캐스트(Multicast), 구성된 피어(Configured Peer), 디스커버리 서버(Discovery Server) 또는 시스템에서 사용하는 다른 메커니즘을 포함하여 실제 운영 DDS 구성에서 참여자 디스커버리가 정상적으로 동작하는지 검증해야 한다. 특히 다중 로봇 또는 공유 네트워크 환경에서는 의도된 디스커버리 범위 외부의 참여자가 격리된 상태를 유지하는지도 확인해야 한다.

보안 자동화(Security Automation)는 런타임 통신 테스트뿐 아니라 소프트웨어 취약점 검사(Vulnerability Check)도 포함해야 한다. 빌드 파이프라인(Build Pipeline)에서 패키지 인벤토리, 컨테이너 이미지, 운영체제 구성요소, ROS 의존성 및 소프트웨어 자재 명세서(Software Bill of Materials, SBOM)를 알려진 취약점 정보와 비교할 수 있다. 보고된 취약점이 실제 배포 구성에 영향을 미치지 않거나 이미 백포트 수정(Backported Fix)을 포함할 수 있으므로 모든 취약점이 발견될 때마다 빌드를 무조건 실패시키기보다 조직의 위험 기준에 따라 평가해야 한다.

정적 구성 검사(Static Configuration Check)는 런타임 이전에 안전하지 않은 배포 설정을 탐지할 수 있다. 예상하지 않은 특권 컨테이너(Privileged Container), 과도한 파일 권한, 노출된 개인키(Private Key), 안전하지 않은 자격 증명 위치, 불필요한 네트워크 서비스, 승인되지 않은 포트 또는 운영 프로파일에서 비활성화된 보안 기능 등이 이에 해당한다. 애플리케이션 소스 코드가 변경되지 않았더라도 패키징 과정에서 구성 오류가 다시 발생할 수 있으므로 이러한 검사는 특히 중요하다.

자동화된 보안 테스트는 감사 로깅(Audit Logging)도 검증해야 한다. 대표적인 인증 실패, 권한 부여 거부, 구성 변경 및 네트워크 정책 위반을 테스트 환경에서 생성하고 예상된 감사 기록과 비교할 수 있다. 테스트를 통해 타임스탬프(Timestamp), 로봇 식별자, 이벤트 유형, 심각도(Severity), 정책 참조 및 전달 동작을 확인할 수 있다. 이를 통해 보안 통제가 활성화되어 있을 뿐 아니라 모니터링 및 사고 대응 시스템(Incident-Response System)에서 실제로 관찰 가능한지도 보장할 수 있다.

지속적 통합 및 지속적 배포(Continuous Integration and Continuous Deployment, CI/CD) 파이프라인은 이러한 테스트를 실행하기 위한 자연스러운 지점을 제공한다. 소스 변경은 정책 검증, 의존성 스캔(Dependency Scanning), 단위 수준 보안 테스트, 컨테이너 검사 및 선택된 ROS 2 통합 시나리오를 실행하도록 구성할 수 있다. 더 많은 자원이 필요한 테스트는 릴리스가 운영 환경으로 승격되기 전에 예약된 파이프라인, 시뮬레이션 환경, 하드웨어 인더루프(Hardware-in-the-Loop, HIL) 시스템 또는 전용 로봇 테스트베드(Testbed)에서 실행할 수 있다.

모든 검사가 물리적 하드웨어를 필요로 하는 것은 아니므로 보안 테스트를 여러 계층으로 구성해야 한다. 정책 구문, 인증서 구조, 구성 규칙, 의존성 스캔 및 많은 권한 부여 시나리오는 가상 또는 컨테이너 기반 환경에서 실행할 수 있다. 반면 미들웨어 상호운용성(Interoperability), 네트워크 분할, 실시간 영향, 하드웨어 인터페이스, 무선 동작 및 물리적 안전 상호작용은 대표적인 실제 하드웨어가 필요할 수 있다. 계층화된 테스트(Layered Testing)는 필요한 부분의 현실적인 검증을 유지하면서 전체 실행 시간을 단축한다.

의미 있는 자동화를 위해서는 재현 가능한 테스트 환경(Reproducible Test Environment)이 필수적이다. ROS 2 배포판, DDS 구현체, 운영체제 버전, 컨테이너 이미지, 보안 산출물, 네트워크 토폴로지(Network Topology) 및 테스트 신원을 통제하고 기록해야 한다. 그렇지 않으면 기본 환경이 변경되어 동일한 테스트에서도 서로 다른 결과가 발생할 수 있다. 코드형 인프라(Infrastructure as Code)와 버전 관리된 구성(Version-Controlled Configuration)을 사용하면 개발자 환경, CI, 실험실 및 릴리스 환경에서 동일한 보안 조건을 재현하는 데 도움이 된다.

테스트 데이터와 자격 증명 자체도 안전하게 관리해야 한다. 자동화에서는 운영 환경의 비밀 정보(Production Secret)를 CI 시스템으로 복사하는 대신 전용 테스트 인증서, 키, 계정 및 권한을 사용해야 한다. 테스트 인프라에 필요한 비밀 정보는 승인된 비밀 관리 메커니즘(Secret-Management Mechanism)을 통해 저장하고 승인된 작업에만 제공해야 한다. 생성된 산출물과 로그에도 개인키 또는 자격 증명이 실수로 남아 있지 않은지 확인해야 한다.

보안 회귀 테스트(Security Regression Testing)는 미들웨어, 운영체제, DDS 또는 SROS2 업데이트 이후 특히 중요하다. 패치는 애플리케이션 인터페이스가 변경되지 않더라도 디스커버리 동작, 인증서 처리, 정책 해석 또는 전송 구성을 변경할 수 있다. 의존성 업데이트 이후 안정된 보안 테스트 모음을 다시 실행하면 이전에 집행되던 인증, 권한 부여, 격리, 암호화 및 로깅 동작이 그대로 유지되는지를 입증할 수 있다.

로봇에 실시간 요구사항(Real-Time Requirement)이 있는 경우 기능적 보안 테스트와 함께 성능 회귀(Performance Regression)도 검증해야 한다. 인증, 암호화, 추가 방화벽 처리, 로깅 또는 새로운 모니터링 에이전트는 CPU, 메모리, 네트워크 대역폭 및 저장 자원을 소비할 수 있다. 자동화된 벤치마크(Benchmark)는 지연 시간(Latency), 지터(Jitter), 처리량(Throughput), 디스커버리 시간 및 자원 사용량을 설정된 한계와 비교할 수 있다. 로봇이 요구되는 제어 및 인지 성능을 유지할 수 있을 때 보안도 운영적으로 효과적이라고 할 수 있다.

가능한 경우 장애 및 복구 시나리오(Fault and Recovery Scenario)도 자동화해야 한다. 테스트에서는 통신 중단, 인증서 거부, 보안 서비스 장애, 로그 서버 사용 불가 또는 일시적인 네트워크 격리 이후의 동작을 평가할 수 있다. 목적은 로봇이 예상된 상태로 전환하고 서비스가 복구되었을 때 예측 가능한 방식으로 정상화되는지를 확인하는 것이다. 보안 장애가 통제되지 않은 자원 증가, 반복적인 재시작 루프(Restart Loop) 또는 중요한 안전 동작의 예상하지 못한 손실을 발생시켜서는 안 된다.

다중 로봇 환경(Multi-Robot Environment)에서는 격리와 확장성(Scalability)을 동시에 검증하는 테스트가 필요하다. 자동화된 시나리오는 여러 로봇 신원을 생성하고 각 로봇이 자신에게 할당된 통신 자원에만 접근하는지를 확인할 수 있다. 테스트에서는 의도하지 않은 로봇 간 명령 접근, 공유 자격 증명, 지나치게 광범위한 권한 또는 예상하지 않은 디스커버리 관계를 탐지해야 한다. 참여자 수를 증가시키면 단일 로봇 테스트에서는 나타나지 않는 디스커버리 또는 보안 처리 특성도 확인할 수 있다.

파이프라인이 릴리스가 정의된 보안 기준을 충족하는지 자동으로 판단할 수 있도록 테스트 결과는 기계 판독 가능(Machine-Readable)해야 한다. 보고서에는 테스트 환경, 소프트웨어 버전, 정책 리비전(Policy Revision), 테스트 신원, 예상 동작, 관찰된 동작, 타임스탬프 및 관련 증거를 기록해야 한다. 사람이 읽을 수 있는 요약도 엔지니어에게 유용하지만 구조화된 결과는 과거 비교, 추세 분석, 감사 검토(Audit Review) 및 릴리스 게이팅(Release Gating)을 훨씬 쉽게 만든다.

릴리스 게이트(Release Gate)는 필수 보안 요구사항과 권고 수준의 발견 사항을 구분해야 한다. 중요한 인증, 권한 부여, 격리 또는 자격 증명 보호 테스트의 실패는 배포를 차단할 수 있지만 낮은 위험의 관찰 사항은 릴리스를 자동으로 중단하지 않고 검토 작업을 생성하도록 할 수 있다. 일정 압박 상황에서 즉흥적으로 릴리스 결정을 내리지 않도록 게이트 기준(Gate Criteria)은 실제 실패가 발생하기 전에 정의해야 한다.

자동화된 테스트가 주기적인 수동 보안 평가(Manual Security Assessment)를 대체해서는 안 된다. 자동화는 알려진 조건과 회귀를 탐지하는 데 강점을 가지지만 아키텍처 검토, 위협 모델링(Threat Modeling), 침투 테스트(Penetration Testing) 및 전문가 분석은 기존 테스트가 표현하지 못하는 새로운 공격 경로나 잘못된 가정을 식별할 수 있다. 수동 평가에서 발견된 사항은 자동화 테스트 모음으로 다시 반영하여 기술적으로 가능한 경우 발견된 취약점이 영구적인 회귀 테스트가 되도록 해야 한다.

보안 테스트 모음(Security Test Suite) 자체도 수명주기 관리(Lifecycle Management)가 필요하다. 새로운 노드, 토픽, 서비스, 액션, DDS 구성, 네트워크 영역, 로봇 모델, 외부 인터페이스 또는 보안 정책이 도입될 때 테스트를 검토해야 한다. 더 이상 유효하지 않은 테스트는 업데이트하거나 제거하고 테스트 커버리지 공백(Coverage Gap)을 추적해야 한다. 테스트 모음이 현재 아키텍처와 위협 모델을 지속적으로 반영할 때에만 모든 테스트가 통과했다는 결과가 의미를 가진다.

궁극적으로 ROS 2 보안 테스트 자동화(ROS 2 Security Test Automation)는 보안 아키텍처와 지속적인 엔지니어링 증거(Continuous Engineering Evidence)를 연결한다. 인증, 권한 부여, 통신 보호, 도메인 격리, 네트워크 분할, 취약점 검사, 감사 로깅, 성능, 복구 및 다중 로봇 동작을 개발 수명주기 전반에서 반복적으로 검증할 수 있다. 이를 CI/CD, 통제된 테스트 환경, 릴리스 게이트 및 주기적인 전문가 평가와 통합하면 ROS 2 로봇 시스템이 발전하더라도 측정 가능하고 재현 가능한 보안 상태(Measurable and Reproducible Security Posture)를 지속적으로 유지할 수 있다.

## 06.10 ROS2 IEC 62443 Compliance Strategy

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

IEC 62443는 산업 자동화 및 제어 시스템(Industrial Automation and Control Systems)을 위한 사이버보안 프레임워크(Cybersecurity Framework)를 제공하며, 공장, 물류 시설, 인프라 및 기타 운영기술(Operational Technology, OT) 환경에서 동작하는 ROS 2 로봇과 높은 관련성을 가진다. ROS 2 규정 준수 전략(Compliance Strategy)에서는 IEC 62443를 ROS 2 자체에 대한 인증으로 간주해서는 안 된다. 대신 전체 로봇 시스템이 수명주기 전반에서 어떻게 설계, 개발, 통합, 운영, 유지보수 및 평가되어야 하는지를 안내하는 기준으로 활용해야 한다.

첫 번째 단계는 평가 대상 시스템(System Under Consideration)과 운영 경계(Operational Boundary)를 정의하는 것이다. 로봇에는 센서, 액추에이터(Actuator), 모터 제어기(Motor Controller), 임베디드 컴퓨터(Embedded Computer), ROS 2 노드, DDS 미들웨어(Middleware), 무선 인터페이스, 유지보수 포트, 안전 제어기(Safety Controller), 외부 플릿 연결(Fleet Connection)이 포함될 수 있다. IEC 62443 기반 분석에서는 어떤 구성요소가 로봇 제어 시스템에 속하는지, 어떤 외부 시스템이 상호작용하는지, 그리고 제품, 서브시스템, 통합업체, 운영자 및 서비스 제공자 사이에서 사이버보안 책임이 어디에서 변경되는지를 식별해야 한다.

영역과 통로(Zones and Conduits)는 ROS 2 배포를 위한 유용한 아키텍처 모델을 제공한다. 유사한 보안 요구사항을 가진 구성요소를 로봇 제어, 인지(Perception), 안전, 관리, 플릿 협조(Fleet Coordination), 개발 또는 기업 연결(Enterprise Connectivity)과 같은 영역(Zone)으로 그룹화할 수 있다. 통로(Conduit)는 이러한 영역 사이의 통제된 통신 경로를 의미한다. ROS 2 도메인 격리(Domain Isolation), VLAN, 라우팅된 서브넷(Routed Subnet), 방화벽(Firewall), 게이트웨이(Gateway), VPN 및 DDS 구성을 활용하여 이러한 아키텍처를 지원할 수 있지만, 어느 하나의 메커니즘만으로 충분하다고 간주해서는 안 된다.

위험 평가(Risk Assessment)를 통해 각 영역과 통로에 할당할 사이버보안 요구사항을 결정해야 한다. 위협 시나리오(Threat Scenario)에는 승인되지 않은 명령 발행, 센서 데이터 조작, 자격 증명 탈취, 서비스 거부(Denial of Service), 승인되지 않은 소프트웨어 변경, 로봇 간 횡적 이동(Lateral Movement), 원격 유지보수를 통한 침해 등이 포함될 수 있다. 분석에서는 기밀성(Confidentiality)과 데이터 무결성(Data Integrity)뿐 아니라 가용성(Availability), 결정론적 동작(Deterministic Operation), 안전상의 결과, 생산 중단 및 잠재적인 물리적 영향도 함께 고려해야 한다.

IEC 62443 보안 수준(Security Level)은 서로 다른 공격자 유형과 공격 역량에 대해 요구되는 저항 수준을 구조적으로 표현하는 방법을 제공한다. 보안 수준은 모든 로봇에 가장 높은 수준이 필요하다는 가정이 아니라 위험 평가 결과에 따라 선택해야 한다. 노출 정도, 사고 발생 시 결과, 연결성 및 운영 역할에 따라 서로 다른 영역에 서로 다른 목표 보안 수준(Target Security Level)을 적용할 수 있다. 이렇게 도출된 요구사항은 구체적인 시스템 및 구성요소 수준의 보안 통제로 변환해야 한다.

식별 및 인증 제어(Identification and Authentication Control)는 ROS 2 보안 아키텍처의 기본 요소이다. SROS2와 DDS 보안(DDS Security)은 인증서와 암호학적 자격 증명(Cryptographic Credential)을 사용하여 분산 참여자에 대한 신원 기반 인증(Identity-Based Authentication)을 제공할 수 있다. 관리자 사용자, 유지보수 시스템, 게이트웨이 및 외부 서비스에도 이에 대응하는 신원 통제가 필요하다. 가능한 경우 자격 증명은 고유하게 프로비저닝(Provisioning)하고 안전하게 저장하며 필요할 때 교체하고, 더 이상 신뢰해서는 안 되는 장치, 사용자 또는 서비스에 대해서는 폐기(Revocation)해야 한다.

사용 통제(Use Control)는 인증된 엔티티(Entity)가 할당된 기능에 필요한 권한만 갖도록 제한하는 것을 요구한다. SROS2 권한(Permissions)은 토픽(Topic), 서비스(Service), 액션(Action) 통신을 제한할 수 있으며, 운영체제 권한, 컨테이너 통제(Container Control), 방화벽 규칙 및 애플리케이션 권한 부여(Application Authorization)는 다른 계층에서 제한을 집행할 수 있다. 인지 노드가 동일한 로봇 네트워크를 공유한다는 이유만으로 제어 권한을 자동으로 획득해서는 안 된다. 최소 권한(Least Privilege)은 ROS 2 통신과 지원 인프라 전반에서 유지되어야 한다.

시스템 무결성(System Integrity)은 소프트웨어, 구성, 메시지 및 보안 산출물(Security Artifact)의 승인되지 않은 변경으로부터 시스템을 보호하는 것을 요구한다. 안전한 소프트웨어 배포, 신뢰할 수 있는 업데이트 메커니즘, 통제된 구성 관리(Configuration Management), 파일 권한, 이미지 무결성(Image Integrity), 필요한 경우 서명된 산출물(Signed Artifact), DDS 통신 보호 등이 이러한 목표에 기여할 수 있다. 무결성 요구사항은 소스 및 빌드 환경부터 배포 패키지를 거쳐 각 로봇에서 실제 실행되는 소프트웨어와 구성까지 확장되어야 한다.

데이터 기밀성(Data Confidentiality)은 모든 통신 흐름에 동일하게 적용한다고 가정하기보다 위험에 따라 적용해야 한다. DDS 보안 암호화는 선택된 ROS 2 통신을 보호할 수 있으며, VPN 또는 기타 보호된 채널을 통해 외부 네트워크를 통과하는 트래픽을 보호할 수 있다. 민감한 자격 증명, 운영 정보, 지도, 진단 데이터 또는 독점적인 센서 정보는 더 강력한 보호가 필요할 수 있다. 암호화 결정에서는 처리 오버헤드(Processing Overhead), 지연 시간(Latency), 대역폭 및 실시간 요구사항도 함께 고려해야 한다.

제한된 데이터 흐름(Restricted Data Flow)은 로봇 네트워크 분할(Network Segmentation) 및 ROS 2 도메인 격리와 밀접하게 연관된다. 모든 장치 사이에 제한 없는 연결을 허용하는 대신 정의된 통로를 통해 통신하도록 해야 한다. VLAN, 방화벽, ROS_DOMAIN_ID 할당, 게이트웨이, DDS 디스커버리 제어(Discovery Control), SROS2 권한을 결합하여 통신 경로를 제한할 수 있다. 개발자 노트북, 기업 시스템, 외부 서비스 및 유지보수 네트워크가 명시적으로 요구되고 통제되지 않는 한 모션 중요 구성요소(Motion-Critical Component)에 직접 접근해서는 안 된다.

이벤트에 대한 적시 대응(Timely Response to Events)을 위해서는 보안 관련 동작을 관찰할 수 있어야 한다. ROS 2 애플리케이션 로그, SROS2 이벤트, DDS 보안 기록, 운영체제 로그, 방화벽 이벤트, 게이트웨이 활동 및 플릿 관리 기록을 감사 아키텍처(Audit Architecture)에 활용할 수 있다. 일관된 타임스탬프(Timestamp)와 중앙 집중식 수집(Centralized Collection)은 이벤트 상호 연관 분석을 향상시킨다. 모니터링에서는 반복적인 인증 실패, 정책 위반, 예상하지 않은 참여자, 구성 변경 또는 비정상적인 통신 패턴과 같은 의미 있는 상태를 탐지해야 한다.

자원 가용성(Resource Availability)은 사이버보안 통제가 필요한 운영 동작을 방해해서는 안 되기 때문에 로봇에서 특히 중요하다. 네트워크 플러딩(Network Flooding), 과도한 디스커버리 트래픽, 비정상 입력, 로그 저장 공간 고갈 또는 계산량이 많은 보안 처리는 제어 및 인지 워크로드에 영향을 줄 수 있다. 용량 계획(Capacity Planning), 네트워크 분할, 자원 제한, 속도 제어(Rate Control), 모니터링 및 성능 테스트를 통해 중요 로봇 기능에 충분한 CPU, 메모리, 저장 공간 및 통신 자원이 유지되는지를 입증해야 한다.

IEC 62443 규정 준수는 안전한 개발 수명주기(Secure Development Lifecycle) 실행에도 크게 의존한다. 보안 요구사항은 아키텍처 개발 과정에서 정의하고 구현, 검증, 릴리스, 유지보수 및 폐기(Decommissioning)에 이르는 전체 과정에서 유지해야 한다. 위협 모델링(Threat Modeling), 안전한 코딩(Secure Coding), 의존성 관리(Dependency Management), 취약점 분석(Vulnerability Analysis), 보안 테스트, 구성 검토 및 통제된 변경 관리는 배포 직전에 한 번 수행하는 보안 검사가 아니라 반복 가능한 엔지니어링 활동이 되어야 한다.

따라서 소프트웨어 공급망 관리(Software Supply-Chain Management)는 전략의 중요한 부분이다. ROS 2 시스템은 일반적으로 오픈소스 패키지, DDS 구현체, 운영체제 구성요소, 장치 드라이버, AI 프레임워크, 컨테이너 및 공급업체 소프트웨어를 결합한다. 소프트웨어 자재 명세서(Software Bill of Materials, SBOM)를 사용하면 이러한 의존성을 기록하고 취약점을 추적할 수 있다. 알려진 CVE는 설치된 버전, 실제 노출 상태, 사용 가능한 패치, 보완 통제(Compensating Control) 및 운영상의 결과를 기준으로 평가해야 한다.

패치 및 취약점 관리(Patch and Vulnerability Management)는 배포 이후에도 계속되어야 한다. ROS 2, DDS 미들웨어, 운영체제, 라이브러리, 컨테이너 및 하드웨어 관련 소프트웨어에 대한 보안 권고(Security Advisory)를 지원되는 제품 수명주기 전체에서 모니터링해야 한다. 패치는 신뢰할 수 있는 출처에서 확보하고 사이버보안 효과와 로봇 호환성을 검증한 후 통제된 롤아웃(Controlled Rollout)을 통해 배포해야 한다. 즉각적인 패치가 불가능한 경우 임시 완화 조치(Temporary Mitigation)와 문서화된 위험 처리(Risk Treatment)를 수립해야 한다.

보안 검증(Security Verification)은 요구사항이 올바르게 구현되었다는 객관적인 증거를 제공해야 한다. 자동화된 ROS 2 보안 테스트를 이용하여 인증, 권한 부여, 통신 보호, 도메인 격리, 방화벽 동작, 감사 로깅(Audit Logging), 취약점 상태 및 구성 규칙을 검증할 수 있다. 정상 동작이 성공한다고 해서 금지된 통신이 실제로 차단된다는 것을 의미하지 않으므로 부정 테스트(Negative Testing)가 특히 중요하다. 테스트 결과는 반복 가능해야 하며 특정 요구사항 및 구성 리비전(Configuration Revision)과 연결되어야 한다.

로봇 시스템에서는 보안 통제가 기능 및 실시간 요구사항을 손상시키지 않는지도 검증해야 한다. 암호화, 인증, 방화벽, 모니터링 에이전트(Monitoring Agent), 게이트웨이 및 로깅은 처리 오버헤드 또는 통신 지연을 증가시킬 수 있다. 대표적인 테스트에서는 지연 시간, 지터(Jitter), 처리량(Throughput), 디스커버리 동작, CPU 사용률, 메모리 소비 및 복구 특성을 측정해야 한다. 따라서 보안 요구사항과 운영 성능 요구사항은 독립적으로 검증하기보다 함께 검증해야 한다.

구성 관리(Configuration Management)는 장기간 규정 준수 상태를 유지하는 데 필요한 추적성(Traceability)을 제공한다. SROS2 거버넌스 및 권한 파일, 인증서, DDS 구성, ROS 도메인 할당, 네트워크 정책, 컨테이너 정의, 운영체제 설정 및 소프트웨어 버전을 통제하고 버전 관리해야 한다. 변경은 식별 가능한 책임자, 검토, 테스트, 배포 기록 및 롤백(Rollback) 기능을 포함하는 승인된 프로세스를 따라야 하며, 이를 통해 각 로봇의 보안 상태를 다시 재구성할 수 있어야 한다.

IEC 62443 정합성(IEC 62443 Alignment)은 단순히 보안 소프트웨어가 설치되어 있다는 사실만으로 입증할 수 없기 때문에 문서화(Documentation)와 증거(Evidence)가 필수적이다. 아키텍처 설명, 위험 평가, 영역 및 통로 모델(Zone-and-Conduit Model), 보안 요구사항, 위협 모델, 테스트 기록, 취약점 관련 의사결정, 패치 이력, 구성 기준선(Configuration Baseline), 감사 기록 및 수명주기 절차가 결합되어 사이버보안이 어떻게 관리되는지를 보여준다. 증거는 식별된 위험에서 구현된 통제를 거쳐 검증 결과까지 추적 가능해야 한다.

역할과 책임(Roles and Responsibilities)은 구성요소 공급업체, 로봇 제조업체, 시스템 통합업체(System Integrator), 자산 소유자(Asset Owner), 운영자 및 유지보수 조직 사이에서 명확하게 구분해야 한다. DDS 공급업체는 미들웨어 보안을 유지하고 로봇 제조업체는 SROS2 정책을 정의하며 시스템 통합업체는 네트워크 영역을 구성할 수 있다. 자산 소유자는 사용자 접근, 패치 일정, 모니터링 및 사고 대응을 관리할 수 있다. 책임이 명확하지 않으면 개별 구성요소가 기술적으로 보안 기능을 제공하더라도 일부 보안 요구사항이 관리되지 않은 상태로 남을 수 있다.

사고 대응(Incident Response)은 운영 보안 수명주기(Operational Security Lifecycle)에 통합해야 한다. 조직은 보안 이벤트를 어떻게 탐지, 분류, 조사, 격리, 복구 및 문서화할 것인지 정의해야 한다. 로봇 특화 절차에서는 영향을 받은 시스템이 계속 운용될 수 있는지, 제한 모드(Restricted Mode)로 전환해야 하는지, 안전하게 정지해야 하는지 또는 플릿에서 격리해야 하는지를 고려해야 한다. 로그, 동기화된 타임스탬프, 구성 이력, 소프트웨어 인벤토리 및 네트워크 증거는 신뢰할 수 있는 조사와 복구에 필요한 정보를 제공한다.

심층 방어(Defense in Depth)는 이러한 통제를 하나의 복원력 있는 아키텍처(Resilient Architecture)로 연결한다. ROS_DOMAIN_ID는 논리적인 DDS 분리를 제공하고, 네트워크 분할은 접근 가능성을 제한하며, SROS2는 통신을 인증하고 권한을 부여한다. 암호화는 선택된 데이터를 보호하고, 운영체제 통제는 프로세스를 제한하며, 모니터링은 비정상 동작을 탐지하고, 수명주기 프로세스는 취약점과 변경을 관리한다. 하나의 보안 계층이 실패하더라도 전체 로봇 시스템 또는 플릿이 자동으로 노출되지 않도록 설계해야 한다.

실용적인 IEC 62443 전략은 일회성 규정 준수 체크리스트(Compliance Checklist)가 아니라 지속적인 엔지니어링 수명주기(Continuous Engineering Lifecycle)로 표현되어야 한다. 시스템을 정의하고, 위험을 평가하며, 영역과 통로를 설정하고, 보안 요구사항을 결정한 후 계층화된 통제를 구현하고 검증하며 증거를 수집해야 한다. 이후 운영 상태를 모니터링하고 취약점을 관리하며 중요한 변경이 발생할 때 다시 평가해야 한다. 이러한 접근을 통해 ROS 2 보안 메커니즘을 보다 광범위한 산업 사이버보안 프레임워크의 일부로 통합하고 안전하고 유지보수 가능하며 복원력 있는 로봇 시스템(Secure, Maintainable, and Resilient Robotic System)을 구축할 수 있다.
