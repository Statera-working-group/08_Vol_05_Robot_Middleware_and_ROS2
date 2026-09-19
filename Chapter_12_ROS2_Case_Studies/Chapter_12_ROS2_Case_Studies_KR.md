**Volume 05 Robot Middleware and ROS2**

# 12. ROS2 Case Studies

## 12.01 Indoor AMR Production Robot: ROS2 Humble Case

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 Humble 기반의 양산형 실내 자율이동로봇(Indoor AMR)은 범용 로봇 미들웨어(Robotics Middleware)가 단순한 연구용 노드(Node)의 집합을 넘어 안정적인 산업용 실행 환경(Industrial Runtime)으로 전환되는 과정을 보여준다. Humble 배포판(Distribution)은 Ubuntu 기반 배포를 위한 장기 지원 기준(Long-Term Support Baseline)을 제공하며, 양산 아키텍처(Production Architecture)는 센싱(Sensing), 위치추정(Localization), 내비게이션(Navigation), 주행 제어(Drive Control), 안전 감독(Safety Supervision), 진단(Diagnostics), 플릿 연결(Fleet Connectivity)을 명확하게 관리되는 서브시스템(Subsystem)으로 구성한다.

로봇 소프트웨어(Robot Software)는 일반적으로 실행 책임(Execution Responsibility)에 따라 구분된다. 하드웨어 연계 노드(Hardware-Facing Node)는 LiDAR, 카메라(Camera), 엔코더(Encoder), 관성측정장치(IMU), 배터리(Battery), 모터 컨트롤러(Motor Controller), 안전 컨트롤러(Safety Controller)의 정보를 수집하며, 상위 계층은 제조사별 프로토콜(Vendor-Specific Protocol)이 아닌 표준화된 ROS 2 인터페이스(Interface)를 사용한다. 이러한 분리는 내비게이션 로직(Navigation Logic)을 재설계하지 않고도 센서나 컨트롤러를 교체할 수 있게 하며, 장치 통합(Device Integration)과 자율 행동(Autonomous Behavior) 사이에 명확한 경계를 제공한다.

이동 계층(Mobility Layer)에서는 ros2_control을 사용하여 ROS 2 소프트웨어 환경과 실제 주행 시스템(Physical Drive System) 사이에 표준화된 연결을 제공할 수 있다. 휠 속도 명령(Wheel Velocity Command), 엔코더 피드백(Encoder Feedback), 조인트 상태(Joint State), 컨트롤러 상태(Controller Status)는 하드웨어 인터페이스(Hardware Interface)를 통해 교환되며, 결정론적 모터 제어(Deterministic Motor Regulation)는 모터 컨트롤러 또는 하위 실시간 서브시스템(Real-Time Subsystem) 내부에 유지된다. 따라서 ROS 2는 엄격한 실시간 전기 제어(Hard Real-Time Electrical Control)를 미들웨어 프로세스로 불필요하게 이동시키지 않으면서 모션(Motion)을 조정한다.

위치추정(Localization)은 휠 오도메트리(Wheel Odometry), 관성 측정(Inertial Measurement), 환경 관측(Environmental Observation)을 결합하여 운용 지도(Operating Map) 내에서 로봇의 자세(Pose)를 지속적으로 유지한다. 양산 시스템은 정상적인 위치추정 정확도뿐만 아니라 초기화(Initialization), 일시적인 센서 성능 저하(Sensor Degradation), 위치추정 신뢰도(Localization Confidence), 지도 변경(Map Change), 위치 상실(Pose Loss) 이후 복구까지 명시적으로 관리해야 한다. 이렇게 생성되는 map-to-odom 및 odom-to-base 변환(Transform)은 내비게이션과 모니터링 구성요소에 신뢰할 수 있는 공간 기준(Spatial Reference)을 제공한다.

Navigation2는 전역 경로 계획(Global Planning), 지역 궤적 생성(Local Trajectory Generation), 코스트맵(Costmap), 행동 실행(Behavior Execution), 복구 메커니즘(Recovery Mechanism), 목표 관리(Goal Management)를 통합하는 핵심 자율 내비게이션 프레임워크(Autonomous Navigation Framework)를 제공한다. 양산 설정(Production Configuration)은 알고리즘의 최대 복잡성보다 예측 가능한 행동(Predictable Behavior)을 중요하게 다룬다. 플래너(Planner)와 컨트롤러(Controller)의 파라미터(Parameter)는 배포 전에 차량 형상(Vehicle Geometry), 적재 하중(Payload), 정지 거리(Stopping Distance), 통로 폭(Aisle Width), 보행자 상호작용(Pedestrian Interaction), 바닥 조건(Floor Condition), 운용 속도 제한(Operational Speed Limit)을 기준으로 검증된다.

ROS 2 통신(Communication)은 각 데이터 스트림(Data Stream)의 의미적 특성에 따라 설정된다. 고주기 센서 정보(High-Rate Sensor Information)는 재전송보다 최신성이 중요한 경우 최선 노력 전송(Best-Effort Delivery)을 사용할 수 있으며, 명령(Command), 운용 상태(Operational State), 경보(Alarm), 일부 제어 정보(Control Information)는 신뢰성 있는 전송(Reliable Delivery)을 요구할 수 있다. 큐 깊이(Queue Depth), 히스토리(History), 내구성(Durability), 데드라인(Deadline), 활성 상태(Liveliness)는 부적절한 DDS 서비스 품질(QoS) 설정이 지연(Latency), 오래된 데이터(Stale Data), 불필요한 네트워크 부하(Network Load)를 발생시킬 수 있기 때문에 아키텍처 파라미터(Architectural Parameter)로 관리된다.

수명주기 관리(Lifecycle Management)는 AMR이 부팅(Boot) 상태에서 자율 서비스(Autonomous Service) 상태로 전환될 때 특히 중요하다. 센서 드라이버(Sensor Driver), 위치추정(Localization), 지도 서비스(Map Service), 내비게이션 구성요소(Navigation Component), 도킹 기능(Docking Function), 플릿 인터페이스(Fleet Interface)는 임의의 순서로 동작 상태가 되어서는 안 된다. 관리형 노드(Managed Node)는 종속성(Dependency)에 따라 설정(Configured), 활성화(Activated), 비활성화(Deactivated), 정리(Cleaned Up), 재시작(Restarted)될 수 있으며, 이를 통해 기동 오케스트레이션(Startup Orchestration)은 모션 기능(Motion Capability)을 활성화하기 전에 선행 조건(Prerequisite)을 확인하고 복구 과정에서 장애 구성요소를 격리할 수 있다.

안전(Safety)은 일반적인 내비게이션 판단(Navigation Decision)과 독립적으로 유지된다. 비상 정지 회로(Emergency Stop Circuit), 보호용 스캐너(Protective Scanner), 안전 PLC(Safety PLC), 드라이브 활성화 신호(Drive Enable Signal), 인증된 안전 기능(Certified Safety Function)은 ROS 2 소프트웨어가 실패하더라도 제어 권한(Control Authority)을 유지해야 한다. ROS 2는 안전 상태 정보(Safety State Information)를 수신하고 이에 따라 자율 행동을 조정하지만, 미들웨어 통신(Middleware Communication)이 안전 등급 제어 체인(Safety-Rated Control Chain)을 대체하지는 않는다. 이러한 분리는 내비게이션 또는 애플리케이션 장애(Application Failure)가 장비 정지를 책임지는 유일한 메커니즘이 되는 것을 방지한다.

자동 도킹(Auto-Docking)은 내비게이션을 정밀한 종단 행동(Precision Terminal Behavior)으로 확장한다. 일반적인 시퀀스(Sequence)는 AMR을 대기 자세(Staging Pose)로 이동시키고, 도킹 기준(Docking Reference)을 획득하며, 저속 정렬(Low-Speed Alignment)을 수행하고, 물리적 접촉 또는 충전 상태(Charging Condition)를 확인한 후 정상적인 충전을 검증하여 임무(Mission)를 완료한다. 각 전환(Transition)은 타임아웃 처리(Timeout Handling)와 복구 로직(Recovery Logic)을 필요로 하며, 감지 실패(Detection Failure), 접근 경로 차단(Blocked Access), 충전기 장애(Charger Fault), 정렬 오류(Alignment Error)가 무한 실행으로 이어지지 않고 제어된 재시도(Controlled Retry) 또는 상위 단계 보고(Escalation)로 처리되도록 해야 한다.

양산형 AMR은 개별 로봇을 넘어서는 통신도 필요로 한다. 로봇 네임스페이스(Robot Namespace)는 여러 장비가 동일한 ROS 2 환경을 공유할 때 토픽(Topic), 서비스(Service), 액션(Action), 파라미터(Parameter), 변환(Transform)을 격리하며, 플릿 계층 인터페이스(Fleet-Level Interface)는 임무 요청(Mission Request), 로봇 상태(Robot State), 교통 정보(Traffic Information), 운용 이벤트(Operational Event)를 교환한다. 상호운용성(Interoperability)이 요구되는 환경에서는 어댑터(Adapter)가 내부 노드 토폴로지(Node Topology)를 직접 외부에 노출하지 않으면서 내부 ROS 2 표현과 VDA 5050 같은 외부 플릿 프로토콜(Fleet Protocol) 사이를 변환할 수 있다.

진단(Diagnostics)은 분산 소프트웨어(Distributed Software)를 관찰 가능한 운용 시스템(Observable Operational System)으로 전환한다. 각 노드는 센서 가용성(Sensor Availability), 위치추정 품질(Localization Quality), 컨트롤러 상태(Controller State), CPU 및 메모리 사용률(Memory Utilization), 네트워크 상태(Network Health), 배터리 상태(Battery Condition), 내비게이션 장애(Navigation Failure), 통신 이상(Communication Anomaly)을 지속적으로 보고한다. 진단 집계(Diagnostic Aggregation)는 이러한 신호를 로봇 수준의 상태(Health State)로 변환하며, 로그(Log), 메트릭(Metric), 기록된 ROS 2 데이터는 실험실에서 쉽게 재현하기 어려운 간헐적 장애(Intermittent Failure)를 분석하기 위한 근거를 제공한다.

배포 구성(Deployment Configuration)은 전체 양산 플릿(Production Fleet)에서 재현 가능해야 한다. ROS 2 패키지(Package), 파라미터 파일(Parameter File), 런치 설정(Launch Configuration), 펌웨어 종속성(Firmware Dependency), 지도(Map), 보안 정책(Security Policy), 애플리케이션 버전(Application Version)은 통제된 소프트웨어 산출물(Controlled Software Artifact)로 관리된다. 지속적 통합 및 배포(CI/CD) 테스트는 릴리스(Release) 전에 빌드(Build)와 인터페이스를 검증할 수 있으며, 단계적 배포(Staged Deployment)와 롤백(Rollback) 메커니즘은 업데이트의 운용 위험(Operational Risk)을 감소시킨다. 이는 AMR 런타임(Runtime)을 보다 광범위한 ROS 2 배포 및 데브옵스(DevOps) 구조와 연결한다.

로봇이 공장 네트워크(Factory Network) 또는 플릿 서버(Fleet Server)를 통해 통신할수록 보안 강화(Security Hardening)는 더욱 중요해진다. DDS 도메인 분리(Domain Separation), 호스트 방화벽(Host Firewall), 제한된 서비스 노출(Restricted Service Exposure), 인증된 업데이트 메커니즘(Authenticated Update Mechanism), SROS2/DDS Security는 비인가 접근 경로(Unauthorized Access Path)를 줄일 수 있다. 그러나 암호화(Encryption), 인증(Authentication), 디스커버리 동작(Discovery Behavior), 인증서 관리(Certificate Management)가 임베디드 컴퓨터(Embedded Computer)의 기동 시간, 대역폭(Bandwidth), CPU 사용량, 통신 지연에 영향을 줄 수 있으므로 보안 설정 역시 성능 벤치마크(Performance Benchmark)를 통해 검증해야 한다.

장애 복구(Fault Recovery)는 모든 문제에 대해 전체 프로세스를 재시작하는 방식이 아니라 계층적 구조(Hierarchical Structure)로 설계된다. 일시적인 센서 통신 문제는 드라이버 재연결(Driver Reconnection)로 처리할 수 있고, 위치추정 성능 저하는 제어된 재위치추정(Relocalization)을 실행할 수 있으며, 내비게이션 실패는 행동 트리 복구(Behavior-Tree Recovery)를 호출할 수 있다. 더 심각한 장애에서는 자율 주행(Autonomous Motion)을 비활성화하고 작업자 개입(Operator Intervention)을 요청할 수 있다. 이러한 계층형 전략은 경미한 장애로 인해 전체 로봇을 불필요하게 정지시키는 것을 방지하면서 불확실한 시스템 상태가 그대로 자율 운용을 지속하지 못하도록 한다.

결과적으로 ROS 2 Humble 기반 양산형 AMR은 단순히 Nav2를 탑재한 이동 플랫폼(Mobile Platform)이 아니라 통합된 사이버 물리 소프트웨어 시스템(Cyber-Physical Software System)이다. ROS 2는 표준화된 통신(Standardized Communication), 수명주기 제어(Lifecycle Control), 하드웨어 추상화(Hardware Abstraction), 진단(Diagnostics), 확장성(Extensibility)을 제공하며, 산업 시스템 엔지니어링(Industrial Systems Engineering)은 결정론적 경계(Deterministic Boundary), 독립적인 안전 체계(Safety Independence), 배포 거버넌스(Deployment Governance), 복구 정책(Recovery Policy), 사이버보안(Cybersecurity), 플릿 관찰성(Fleet Observability)을 추가한다. 이러한 요소가 결합됨으로써 다수의 양산 로봇을 지속적으로 운용할 수 있는 유지보수 가능한 아키텍처(Maintainable Architecture)가 구축된다.

## 12.02 Outdoor AMR ROS2 Multi-Robot Fleet Operation

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

실외 자율이동로봇(Outdoor AMR)의 플릿 운영(Fleet Operation)은 ROS 2를 단일 로봇 미들웨어(Single-Robot Middleware) 환경에서 여러 자율 로봇이 임무(Mission), 경로(Route), 자원(Resource), 안전 상태(Safety State), 운용 정보(Operational Information)를 조정해야 하는 분산 모빌리티 인프라(Distributed Mobility Infrastructure)로 확장한다. 아키텍처는 로봇 로컬 자율성(Robot-Local Autonomy)과 플릿 수준 지능(Fleet-Level Intelligence)을 분리하여 중앙 플릿 시스템과의 통신이 일시적으로 저하되더라도 각 AMR이 필수적인 내비게이션(Navigation)과 안전 기능(Safety Function)을 지속할 수 있도록 한다.

각 실외 AMR은 센서 드라이버(Sensor Driver), 위치추정(Localization), 인지(Perception), 내비게이션(Navigation), 모션 제어(Motion Control), 진단(Diagnostics), 임무 실행(Mission Execution) 구성요소를 포함하는 독립적인 ROS 2 소프트웨어 스택(Software Stack)을 운용한다. 로봇 로컬 처리(Robot-Local Processing)는 지속적인 서버 연결에 대한 의존성을 최소화하고 지연 시간에 민감한 의사결정(Latency-Sensitive Decision)을 실제 플랫폼 가까이에서 수행하도록 한다. 따라서 플릿 서비스(Fleet Service)는 이동 중 수행되는 모든 저수준 제어 판단을 직접 담당하지 않고 로봇을 감독하고 조정한다.

실외 위치추정(Outdoor Localization)은 로봇이 도로, 산업 단지, 캠퍼스, 물류 시설 및 기타 지리적으로 분산된 공간에서 운용될 수 있기 때문에 일반적인 실내 내비게이션보다 광범위한 센서 아키텍처(Sensor Architecture)를 필요로 한다. GNSS 또는 실시간 이동측위 GNSS(RTK-GNSS)는 전역 위치(Global Positioning)를 제공할 수 있으며, 관성측정장치(IMU), 휠 오도메트리(Wheel Odometry), LiDAR, 카메라(Camera)는 로컬 이동 추정(Local Motion Estimation)과 환경 정합(Environmental Alignment)을 지원한다. ROS 2는 타임스탬프 메시지(Timestamped Message)와 일관된 좌표 프레임(Coordinate Frame) 관계를 통해 이러한 정보원을 조정한다.

변환 아키텍처(Transform Architecture)는 전역 기준(Global Reference)과 로봇 상대 좌표계(Robot-Relative Coordinate System)를 모두 표현해야 한다. 각 AMR은 자체 베이스(Base), 오도메트리(Odometry), 센서(Sensor), 로컬 지도(Local Map) 프레임을 유지하는 반면, 플릿 애플리케이션(Fleet Application)은 로봇 위치를 해석하기 위한 공통 공간 기준(Common Spatial Reference)을 필요로 한다. 올바른 네임스페이스(Namespace)와 tf2 설계는 여러 로봇이 유사한 프레임 이름을 발행할 때 변환 충돌(Transform Collision)을 방지하고, 모니터링 시스템이 서로 다른 차량의 좌표 트리(Coordinate Tree)를 혼동하지 않고 전체 플릿을 시각화할 수 있도록 한다.

Navigation2는 로봇 로컬 경로 실행(Robot-Local Path Execution)을 담당하고 플릿 계층(Fleet Layer)은 상위 수준의 교통 및 임무 제약 조건(Traffic and Mission Constraint)을 관리할 수 있다. 플릿 시스템은 목적지(Destination), 통행로(Corridor), 작업 구역(Work Zone), 경로 구간(Route Segment)을 할당할 수 있으며, 이후 개별 AMR은 로컬 코스트맵(Local Costmap)과 장애물 정보(Obstacle Information)를 이용하여 실행 가능한 궤적(Feasible Trajectory)을 생성하고 수행한다. 이러한 분리는 신속한 로컬 장애물 회피(Local Obstacle Avoidance)를 유지하면서 전역 교통 조정(Global Traffic Coordination)이 여러 로봇의 움직임을 동시에 고려할 수 있도록 한다.

다중 로봇 네임스페이스 설계(Multi-Robot Namespace Design)는 플릿 확장성(Fleet Scalability)의 기본 요소이다. 토픽(Topic), 서비스(Service), 액션(Action), 파라미터(Parameter), 수명주기 인터페이스(Lifecycle Interface), 진단(Diagnostics), 변환(Transform)은 로봇별 식별자(Robot-Specific Identity) 아래에 구성되어 동일한 소프트웨어 패키지(Software Package)가 인터페이스 충돌 없이 여러 차량에서 실행될 수 있도록 한다. 표준화된 명명 규칙(Standardized Naming)은 플릿 도구가 모든 차량에 고유한 소프트웨어 구성을 유지하는 대신 로봇 식별자에서 통신 엔드포인트(Communication Endpoint)를 도출할 수 있게 하므로 자동화된 배포(Automated Deployment)도 단순화한다.

DDS 통신(DDS Communication)은 안정적인 유선 연결을 전제로 하지 않고 실외 네트워크 조건(Outdoor Network Condition)에 맞게 설계되어야 한다. Wi-Fi, 사설 5G(Private 5G), 산업용 무선 네트워크(Industrial Wireless Network), 혼합 통신 환경(Mixed Communication Environment)은 가변적인 지연(Variable Latency), 패킷 손실(Packet Loss), 핸드오버(Handover), 일시적인 연결 단절(Temporary Disconnection)을 발생시킬 수 있다. 따라서 ROS 2 서비스 품질 정책(QoS Policy)은 메시지의 목적에 따라 선택하며, 고주기의 일시적인 센서 트래픽(Transient Sensor Traffic)과 더 강력한 전달 보장이 필요한 임무 명령, 로봇 상태, 경보 및 기타 운용 정보를 구분한다.

플릿 통신(Fleet Communication)은 모든 내부 ROS 2 토픽을 무선 인프라(Wireless Infrastructure)를 통해 전송하는 방식을 피해야 한다. 고대역폭 카메라 스트림(High-Bandwidth Camera Stream), 포인트 클라우드(Point Cloud), 원시 센서 메시지(Raw Sensor Message)는 원격 진단(Remote Diagnostics)에서 특별히 요구하지 않는 한 일반적으로 로봇 내부에 유지한다. 대신 플릿 인터페이스(Fleet Interface)는 위치(Pose), 속도(Velocity), 배터리 상태(Battery State), 임무 진행 상태(Mission Progress), 내비게이션 상태(Navigation Status), 안전 상태(Safety Condition), 장애 코드(Fault Code), 연결 품질(Connectivity Quality)과 같은 간결한 운용 정보를 발행하여 플릿 관찰성(Fleet Observability)을 유지하면서 대역폭을 절감한다.

임무 오케스트레이션(Mission Orchestration)은 외부의 운용 요청(Operational Request)을 로봇이 실행할 수 있는 작업으로 변환한다. 플릿 관리자(Fleet Manager)는 임무를 할당하기 전에 로봇 가용성(Robot Availability), 위치(Location), 배터리 상태(Battery Condition), 적재 능력(Payload Capability), 현재 할당 작업(Current Assignment), 운용 제한(Operational Restriction)을 평가한다. 선택된 로봇은 ROS 2 액션(Action) 또는 상태 머신 전환(State-Machine Transition)으로 변환할 수 있는 구조화된 작업(Structured Task)을 수신하며, 진행 및 완료 이벤트는 이후 스케줄링(Scheduling)과 기업 시스템 통합(Enterprise-System Integration)을 위해 플릿 계층으로 반환된다.

여러 AMR이 좁은 통로(Narrow Passage), 교차로(Intersection), 충전소(Charging Station), 게이트(Gate), 적재 구역(Loading Area) 또는 기타 제한된 자원(Constrained Resource)을 공유할 경우 교통 관리(Traffic Management)가 필수적이다. 플릿 수준 조정(Fleet-Level Coordination)은 경로 구간 또는 운용 구역을 예약할 수 있으며, 동시에 로컬 내비게이션은 보행자(Pedestrian), 차량(Vehicle), 예상하지 못한 장애물(Unexpected Obstacle)을 계속 감지한다. 이러한 계층적 접근(Hierarchical Approach)은 전역 스케줄링 로직(Global Scheduling Logic)이 즉각적인 충돌 회피를 대체하는 것을 방지하고, 순수한 로컬 내비게이션만으로 인해 플릿 전체의 교착 상태(Deadlock)나 자원 충돌(Resource Conflict)이 발생하는 것을 방지한다.

실외 플릿 운영은 통신 손실(Communication Loss)을 명시적으로 허용할 수 있도록 설계해야 한다. 플릿 연결을 상실한 로봇은 즉시 제어 불능 상태가 되거나 오래된 임무를 무기한 수행하는 대신 사전에 정의된 성능 저하 운용 모드(Degraded Operating Mode)로 전환해야 한다. 운용 정책(Operational Policy)에 따라 안전하게 정지하거나, 제한된 경로 구간을 완료하거나, 지정된 대기 위치(Waiting Location)로 이동하거나, 재연결을 시도하면서 제한된 시간 동안 로컬 자율성(Local Autonomy)을 유지할 수 있다. 통신이 복구된 후에는 새로운 명령을 수락하기 전에 상태 조정(State Reconciliation)이 필요하다.

수명주기 관리(Lifecycle Management)는 기동(Startup), 종료(Shutdown), 유지보수(Maintenance), 장애 복구(Fault Recovery) 과정에서 분산 로봇 소프트웨어(Distributed Robot Software)를 체계적으로 제어하는 메커니즘을 제공한다. 센서 노드(Sensor Node), 위치추정(Localization), 내비게이션(Navigation), 플릿 어댑터(Fleet Adapter), 임무 관리자(Mission Manager)는 종속성 규칙(Dependency Rule)에 따라 관리되는 상태(Managed State)를 전환할 수 있다. 플릿 규모에서는 수명주기 상태 자체도 운용 텔레메트리(Operational Telemetry)가 되어 원격 시스템이 의도적으로 비활성화된 로봇과 내비게이션 또는 하드웨어 인터페이스가 예기치 않게 실패한 로봇을 구분할 수 있도록 한다.

플릿 조정(Fleet Coordination)이 중앙집중식으로 수행되더라도 안전 권한(Safety Authority)은 각 물리적 AMR에 로컬로 유지된다. 비상 정지(Emergency Stop), 보호 감지(Protective Sensing), 드라이브 억제(Drive Inhibition) 및 기타 안전 관련 기능(Safety-Related Function)은 원격 ROS 2 노드나 무선 네트워크 연결에만 의존해서는 안 된다. 플릿 시스템은 안전 상태를 관찰하고 임무를 중단하며 다른 로봇의 경로를 변경하고 제어된 행동을 요청할 수 있지만, 물리적 로봇 자체는 플릿 연결과 독립적으로 안전 상태(Safe Condition)에 진입할 수 있는 능력을 유지한다.

진단 및 관찰성(Diagnostics and Observability)은 로봇 로컬 정보를 통합하여 플릿 전체의 운용 상황(Operational Picture)을 구성한다. 상태 정보(Health Status), 위치추정 품질(Localization Quality), CPU 및 메모리 사용률(Memory Utilization), 네트워크 품질(Network Quality), 배터리 상태(Battery Condition), 센서 가용성(Sensor Availability), 내비게이션 장애(Navigation Failure), 임무 이벤트(Mission Event)는 표준화된 진단 인터페이스(Standardized Diagnostic Interface)를 통해 수집할 수 있다. 과거 로그(Historical Log)와 ROS 2 데이터 기록(Data Recording)을 통해 엔지니어는 소프트웨어 동작과 환경 조건(Environmental Condition)의 상관관계를 분석하고 대규모 현장 운용에서만 발생하는 반복적인 문제를 식별할 수 있다.

실외 AMR이 창고 관리(Warehouse Management), 제조 실행 시스템(Manufacturing Execution System), 교통 제어(Traffic Control), 보안 시스템(Security System), 타사 플릿 시스템(Third-Party Fleet System)과 통신해야 하는 경우 상호운용성(Interoperability)이 중요해진다. 프로토콜 어댑터(Protocol Adapter)는 내부 ROS 2 인터페이스를 외부 API 및 상호운용성 표준(Interoperability Standard)으로부터 분리할 수 있으며, 적용 가능한 경우 VDA 5050과 같은 플릿 중심 메커니즘(Fleet-Oriented Mechanism)도 사용할 수 있다. 이러한 경계는 외부 운용 시스템과 안정적인 통합 계약(Integration Contract)을 유지하면서 내부 로봇 아키텍처가 지속적으로 발전할 수 있도록 한다.

배포 및 업데이트 관리(Deployment and Update Management)는 모든 로봇을 동시에 업그레이드하는 대신 전체 플릿을 통제된 집단(Controlled Population)으로 관리해야 한다. 소프트웨어 패키지(Software Package), 컨테이너(Container), 구성 파라미터(Configuration Parameter), 지도(Map), 보안 자격증명(Security Credential), 펌웨어 종속성(Firmware Dependency)은 릴리스 전에 버전 관리(Versioning)되고 검증된다. 업데이트는 개발 로봇(Development Robot), 검증 장비(Validation Unit), 제한된 양산 그룹(Limited Production Group), 전체 플릿 순으로 단계적으로 진행할 수 있으며, 상태 모니터링(Health Monitoring)과 롤백(Rollback) 기능을 통해 하나의 소프트웨어 결함이 다수의 로봇을 동시에 비활성화하는 위험을 줄인다.

성숙한 실외 다중 로봇 ROS 2 아키텍처(Outdoor Multi-Robot ROS 2 Architecture)는 결과적으로 분산된 물리적 자율성(Decentralized Physical Autonomy)과 중앙집중식 또는 분산형 플릿 지능(Centralized or Distributed Fleet Intelligence)을 결합한다. 개별 AMR은 인지(Perception), 위치추정(Localization), 내비게이션(Navigation), 제어(Control), 안전(Safety) 기능을 유지하고, 플릿 계층은 임무(Mission), 교통(Traffic), 자원(Resource), 배포(Deployment), 집단 운용 인식(Collective Operational Awareness)을 관리한다. 이러한 역할 분리는 ROS 2를 개별 자율이동로봇에서 더 큰 규모의 다중 로봇 및 플릿 지능 시스템(Multi-Robot and Fleet-Intelligence System)으로 확장하기 위한 확장 가능한 기반(Scalable Foundation)을 제공한다.

## 12.03 7-DOF Manipulator ros2.control Integration Case

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

ros2_control을 통해 ROS 2와 통합된 7자유도 매니퓰레이터(7-DOF Manipulator)는 표준화된 소프트웨어 인터페이스(Standardized Software Interface)를 유지하면서 상위 수준 로봇 애플리케이션(High-Level Robotic Application)을 결정론적 액추에이터 제어(Deterministic Actuator Control)와 분리하는 방법을 보여준다. 7개의 관절(Joint)은 운동학적 여유도(Kinematic Redundancy)를 제공하여 매니퓰레이터가 하나의 목표 말단장치 자세(End-Effector Pose)에 대해 여러 관절 구성(Joint Configuration)을 사용할 수 있도록 한다. ROS 2는 계획(Planning), 감독(Supervision), 센싱(Sensing), 진단(Diagnostics), 명령 전달(Command Delivery)을 조정하고, 하위 하드웨어는 시간 결정적인 서보 기능(Time-Critical Servo Function)을 수행한다.

매니퓰레이터는 링크(Link), 관절(Joint), 좌표 프레임(Coordinate Frame), 제한값(Limit), 관성 특성(Inertial Property), 액추에이터 관련 정보(Actuator-Related Information)를 포함하는 로봇 기술 모델(Robot Description)을 통해 표현된다. URDF와 관련 구성 파일(Configuration File)은 시각화(Visualization), 운동학(Kinematics), 계획(Planning), 제어 통합(Control Integration)을 위한 공통 구조 모델(Common Structural Model)을 제공한다. 7축 로봇팔에서는 계획 알고리즘이 기계적 또는 운용상의 제약을 위반하지 않으면서 여유도를 활용해야 하므로 정확한 관절 제한(Joint Limit)과 변환 관계(Transform Relationship)가 특히 중요하다.

ros2_control 프레임워크(Framework)는 ROS 2 소프트웨어와 실제 매니퓰레이터 사이의 핵심 추상화 경계(Abstraction Boundary)를 형성한다. 하드웨어 인터페이스(Hardware Interface)는 애플리케이션 노드가 서보 드라이브(Servo Drive)와 직접 통신하도록 하는 대신 표준화된 자원 모델(Resource Model)을 통해 관절 명령 및 상태 인터페이스(Joint Command and State Interface)를 제공한다. 따라서 위치(Position), 속도(Velocity), 토크 또는 힘(Effort) 명령을 제조사별 하드웨어 프로토콜(Vendor-Specific Hardware Protocol)에 매핑하고, 측정된 관절 위치와 속도 및 기타 피드백을 일관된 ROS 2 제어 인터페이스를 통해 반환할 수 있다.

일반적인 하드웨어 플러그인(Hardware Plugin)은 매니퓰레이터 인터페이스를 위한 초기화(Initialization), 활성화(Activation), 읽기(Read), 쓰기(Write), 종료(Shutdown) 동작을 구현한다. 각 제어 주기(Control Cycle)에서 읽기 동작은 액추에이터 피드백(Actuator Feedback)을 획득하여 하드웨어 고유 값을 ROS 2 관절 상태(Joint State)로 변환하며, 쓰기 동작은 컨트롤러 출력(Controller Output)을 로봇 하드웨어가 이해할 수 있는 명령으로 변환한다. 이러한 아키텍처는 EtherCAT, CAN, 산업용 이더넷(Industrial Ethernet), 독점 서보 API(Proprietary Servo API) 등의 통신 세부사항을 상위 매니퓰레이션 소프트웨어(Manipulation Software)로부터 격리한다.

컨트롤러 관리자(Controller Manager)는 하드웨어 자원(Hardware Resource)을 조정하고 해당 자원에서 동작하는 컨트롤러를 동적으로 관리한다. 관절 상태 브로드캐스터(Joint State Broadcaster)는 측정된 관절 정보를 ROS 2 시스템의 다른 구성요소에 발행하며, 궤적 또는 위치 컨트롤러(Trajectory or Position Controller)는 필요한 명령 인터페이스(Command Interface)를 점유한다. 자원 소유권(Resource Ownership)은 호환되지 않는 컨트롤러가 동일한 관절을 동시에 명령하는 것을 방지하고, 운용 중 컨트롤러의 로딩(Loading), 설정(Configuration), 활성화(Activation), 비활성화(Deactivation), 전환(Switching)을 제어할 수 있는 메커니즘을 제공한다.

7개 관절의 협조 운동(Coordinated Motion)을 위해 관절 궤적 컨트롤러(Joint Trajectory Controller)는 원하는 관절 위치와 관련 운동 정보를 포함하는 시간 파라미터화 궤적(Time-Parameterized Trajectory)을 수신할 수 있다. 애플리케이션 소프트웨어에서 각 액추에이터를 독립적으로 명령하는 대신 컨트롤러가 궤적을 보간(Interpolation)하고 매니퓰레이터 전체에 동기화된 명령(Synchronized Command)을 생성한다. 이는 부드러운 다축 운동(Multi-Axis Motion)을 유지하는 데 필수적이며 모션 계획 시스템(Motion-Planning System)과 실제 로봇 사이에 표준 실행 인터페이스(Standard Execution Interface)를 제공한다.

상위 수준 매니퓰레이션 계획(Manipulation Planning)은 서보 구현(Servo Implementation)에 직접 의존하지 않고 충돌을 고려한 궤적(Collision-Aware Trajectory)을 생성할 수 있다. 계획 구성요소(Planning Component)는 로봇 모델(Robot Model), 현재 관절 상태(Current Joint State), 목표 자세(Target Pose), 환경 표현(Environmental Representation), 운동학적 제약(Kinematic Constraint)을 이용하여 실행 가능한 궤적을 계산한다. 생성된 관절 공간 궤적(Joint-Space Trajectory)은 ROS 2 제어 인터페이스를 통해 전달되어 모션 계획(Motion Planning), 궤적 실행(Trajectory Execution), 하드웨어 수준 서보 제어(Hardware-Level Servo Regulation) 사이의 명확한 분리를 유지한다.

7자유도는 여유도 관리(Redundancy Management)를 중요한 통합 요소로 만든다. 동일한 직교 좌표계 말단장치 자세(Cartesian End-Effector Pose)를 만족하는 여러 관절 구성이 존재할 수 있으므로 플래너(Planner)는 팔꿈치 위치(Elbow Position), 관절 한계 회피(Joint-Limit Avoidance), 충돌 여유(Collision Clearance), 조작성(Manipulability), 작업별 자세(Task-Specific Posture)를 최적화할 수 있다. 제어 아키텍처는 계획된 관절 구성을 실행 과정에서 유지하여 명령이 모호한 직교 좌표계 목표(Cartesian Target)로 단순화됨으로써 원하지 않는 대체 자세가 생성되는 것을 방지해야 한다.

제어 업데이트 경로(Control Update Path)는 실시간 동작(Real-Time Behavior)을 고려하여 설계해야 한다. 메모리 할당(Memory Allocation), 블로킹 연산(Blocking Operation), 과도한 로깅(Excessive Logging), 예측할 수 없는 콜백(Unpredictable Callback), 비결정론적 통신(Non-Deterministic Communication)은 가능한 한 지연 시간에 민감한 제어 루프(Latency-Sensitive Control Loop) 외부에 배치해야 한다. ROS 2와 ros2_control은 실시간 지향 실행(Real-Time-Oriented Execution)을 조정할 수 있지만, 엄격한 서보 루프(Hard Servo Loop)는 결정론적 타이밍을 보다 직접적으로 보장할 수 있는 전용 드라이브(Dedicated Drive), 로봇 컨트롤러(Robot Controller), 실시간 컴퓨팅 계층(Real-Time Computing Layer) 내부에 유지될 수 있다.

피드백(Feedback)은 매니퓰레이터에서 더 넓은 ROS 2 시스템으로 지속적으로 전달된다. 관절 위치와 속도는 상태 추정(State Estimation)과 시각화(Visualization)를 지원하며, 컨트롤러 상태(Controller Status), 궤적 실행 상태(Trajectory Execution State), 하드웨어 장애(Hardware Fault), 온도(Temperature), 전류(Current) 또는 기타 사용 가능한 신호는 진단(Diagnostics)에 활용된다. 이러한 양방향 아키텍처(Bidirectional Architecture)를 통해 애플리케이션 소프트웨어는 모든 명령 궤적이 성공적으로 실행되었다고 가정하는 대신 실제 로봇의 동작 상태를 기반으로 판단할 수 있다.

수명주기 및 컨트롤러 상태 관리(Lifecycle and Controller State Management)는 체계적인 기동 및 종료 동작(Startup and Shutdown Behavior)을 제공한다. 하드웨어 인터페이스는 통신 상태, 관절 상태 유효성(Joint-State Validity), 구성(Configuration), 필요한 안전 조건(Safety Condition)이 검증된 이후에만 활성화되어야 한다. 이후 컨트롤러는 정의된 순서에 따라 활성화될 수 있다. 종료 또는 장애 복구 과정에서는 하드웨어 자원을 해제하기 전에 모션 명령(Motion Command)을 비활성화하여 소프트웨어 상태 사이에서 제어되지 않는 전환이 발생할 가능성을 줄인다.

안전 감독(Safety Supervision)은 정상적인 궤적 제어(Trajectory Control)와 분리된 상태로 유지된다. 관절 제한(Joint Limit), 속도 제한(Velocity Limit), 작업공간 제한(Workspace Restriction), 충돌 모니터링(Collision Monitoring), 보호 정지(Protective Stop), 비상 정지(Emergency Stop), 드라이브 수준 안전 기능(Drive-Level Safety Function)은 서로 다른 아키텍처 계층에서 동작할 수 있다. ROS 2는 안전 상태를 모니터링하고 궤적 실행을 중단할 수 있지만, 안전 등급 기능(Safety-Rated Function)은 일반적인 미들웨어 메시지에만 의존해서는 안 된다. 상위 소프트웨어가 실패하더라도 매니퓰레이터는 위험한 움직임을 방지하거나 정지할 수 있는 독립적인 메커니즘을 유지해야 한다.

장애 처리(Fault Handling)는 통신 장애(Communication Fault), 컨트롤러 장애(Controller Fault), 궤적 실행 실패(Trajectory Failure), 하드웨어 경보(Hardware Alarm), 안전 이벤트(Safety Event)를 구분해야 한다. 일시적인 통신 문제는 재연결(Reconnection) 또는 컨트롤러 재활성화(Controller Reactivation)를 허용할 수 있지만, 엔코더 불일치(Encoder Inconsistency), 드라이브 장애(Drive Fault), 예상하지 못한 관절 움직임(Unexpected Joint Motion), 보호 정지 활성화(Protective-Stop Activation)는 모션 억제(Motion Inhibition)와 작업자 개입(Operator Intervention)을 요구할 수 있다. 명시적인 장애 상태 전환(Fault-State Transition)은 실제 시스템이 안전한 것으로 확인되지 않은 상태에서 자동 복구 로직이 매니퓰레이터를 재시작하는 것을 방지한다.

진단 및 추적(Diagnostics and Tracing)은 장애 원인이 계획(Planning), ROS 2 통신, 컨트롤러 스케줄링(Controller Scheduling), 하드웨어 인터페이스, 서보 시스템(Servo System) 등 다양한 영역에서 발생할 수 있기 때문에 통합 과정에서 중요하다. 엔지니어는 궤적 명령, 컨트롤러 업데이트, 관절 피드백, 실행 오류(Execution Error), 타이밍 정보(Timing Information)의 상관관계를 분석하여 비정상적인 움직임의 원인을 찾을 수 있다. 지연 시간 및 지터 측정(Latency and Jitter Measurement)은 성능 제한이 ROS 2 내부에서 발생하는지 또는 미들웨어 경계 아래에서 발생하는지를 판단하는 데 특히 유용하다.

시뮬레이션 및 하드웨어 인더루프 시험(Simulation and Hardware-in-the-Loop Testing)은 실제 로봇팔에 배포하기 전에 위험을 줄인다. 동일한 로봇 기술 모델과 컨트롤러 구성을 시뮬레이션 하드웨어 인터페이스(Simulated Hardware Interface)에 적용하여 관절 명명(Joint Naming), 컨트롤러 자원 소유권, 궤적 인터페이스, 제한값, 애플리케이션 동작을 검증할 수 있다. 이후 하드웨어 인더루프(HIL) 환경에서 전체 기계 시스템을 동작시키기 전에 실제 컨트롤러 또는 통신 장치를 단계적으로 연결하여 통합 아키텍처를 보다 현실적인 조건에서 검증할 수 있다.

따라서 양산 환경에서 전체 시스템은 계층화된 매니퓰레이션 스택(Layered Manipulation Stack)을 형성한다. 작업 애플리케이션(Task Application)과 모션 계획(Motion Planning)은 원하는 동작을 생성하고, ROS 2는 상태와 명령을 전달하며, ros2_control은 컨트롤러와 하드웨어 접근을 표준화하고, 전용 서보 하드웨어(Dedicated Servo Hardware)는 결정론적 액추에이터 제어를 실행한다. 이러한 아키텍처를 통해 7자유도 매니퓰레이터는 여유 운동학(Redundant Kinematics), 재사용 가능한 ROS 2 소프트웨어, 하드웨어 독립성(Hardware Independence), 제어된 수명주기 동작, 진단, 산업 안전 경계(Industrial Safety Boundary)를 하나의 유지보수 가능한 통합 프레임워크(Maintainable Integration Framework) 안에서 결합할 수 있다.

## 12.04 Quadruped ROS2 RL Policy Integration Case

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

강화학습 정책(Reinforcement Learning Policy)을 ROS 2와 통합하는 4족 보행 로봇(Quadruped Robot)은 학습 기반 보행(Learned Locomotion)을 미들웨어 통신(Middleware Communication), 하드웨어 제어(Hardware Control), 인지(Perception), 안전 감독(Safety Supervision)으로부터 분리하는 계층형 아키텍처(Layered Architecture)를 필요로 한다. 강화학습 정책(RL Policy)은 로봇 관측값(Robot Observation)을 기반으로 움직임을 생성하고, ROS 2는 명령(Command), 상태 정보(State Information), 수명주기 관리(Lifecycle Management), 진단(Diagnostics), 상위 내비게이션(Higher-Level Navigation)을 조정한다. 이러한 분리는 학습 제어(Learned Control)를 독립적인 추론 프로그램이 아니라 유지보수 가능한 로봇 소프트웨어 스택(Robot Software Stack)의 하나의 구성요소로 만든다.

물리적 로봇(Physical Robot)은 링크(Link), 관절(Joint), 관성 특성(Inertial Property), 충돌 형상(Collision Geometry), 액추에이터 제한(Actuator Limit), 좌표 프레임(Coordinate Frame)을 기술하는 URDF 또는 MJCF 모델을 통해 표현된다. 4족 보행 시스템(Quadruped System)은 추가적으로 부동 베이스(Floating Base), 고관절(Hip), 대퇴부(Thigh), 하퇴부(Lower Leg), 발(Foot)에 대한 일관된 표현을 필요로 한다. 이 모델은 시뮬레이션(Simulation), 시각화(Visualization), 상태 추정(State Estimation), 제어(Control), 강화학습 환경(Reinforcement Learning Environment)을 연결하여 학습부터 실제 로봇 배포까지 관절 순서(Joint Ordering)와 좌표 규약(Coordinate Convention)이 일관되게 유지되도록 한다.

로봇 관측값(Robot Observation)은 강화학습 정책의 입력 인터페이스(Input Interface)를 구성한다. 일반적인 관측값에는 관절 위치(Joint Position), 관절 속도(Joint Velocity), 베이스 자세(Base Orientation), 각속도(Angular Velocity), 추정 선속도(Estimated Linear Velocity), 명령된 움직임(Commanded Motion), 이전 행동(Previous Action)이 포함되며, 일부 정책은 발 접촉(Foot Contact) 또는 지형 정보(Terrain Information)를 추가로 사용한다. ROS 2 노드는 이러한 신호를 수집하거나 추정하지만 정책 입력은 학습 과정에서 사용한 정규화(Normalization), 순서(Ordering), 단위(Unit), 기준 프레임(Reference Frame), 이력 표현(History Representation)을 정확하게 따라야 한다.

정책 추론 노드(Policy Inference Node)는 ROS 2와 학습된 보행 제어기(Learned Locomotion Controller)를 연결하는 핵심 브리지(Bridge) 역할을 수행한다. 이 노드는 필요한 로봇 상태와 명령 정보를 구독(Subscribe)하고 관측 벡터(Observation Vector)를 구성하며 신경망 추론(Neural-Network Inference)을 수행한 후 정책 출력을 제어 기준값(Control Reference)으로 변환한다. 모델은 PyTorch, ONNX Runtime 또는 최적화된 추론 런타임(Optimized Inference Runtime)을 통해 배포할 수 있으며, 실행 지연(Inference Latency)은 보행 제어 아키텍처와 호환되는 범위로 유지되어야 한다.

정책 출력(Policy Output)은 학습 과정에서 사용된 행동 표현(Action Representation)에 따라 해석해야 한다. 정책은 목표 관절 위치(Desired Joint Position), 위치 오프셋(Position Offset), 관절 속도(Joint Velocity), 토크(Torque) 또는 하위 제어기(Lower-Level Controller)가 사용하는 파라미터를 생성할 수 있다. 따라서 추론 노드는 임의의 신경망 출력값을 액추에이터에 직접 전달해서는 안 된다. 출력 스케일링(Output Scaling), 클리핑(Clipping), 관절 매핑(Joint Mapping), 부호 규약(Sign Convention), 명령 제한(Command Limit)은 명령이 실제 제어 경로에 진입하기 전에 학습 환경을 정확하게 재현해야 한다.

ros2_control은 보행 소프트웨어(Locomotion Software)와 4족 보행 로봇 하드웨어(Quadruped Hardware) 사이의 표준화된 인터페이스를 제공할 수 있다. 하드웨어 인터페이스(Hardware Interface)는 관절 상태와 명령 자원(Command Resource)을 제공하고 컨트롤러(Controller)는 액추에이터 접근을 관리한다. 학습된 정책은 CAN, EtherCAT, 직렬 통신(Serial Communication), 제조사 전용 모터 API(Vendor-Specific Motor API)에 직접 의존하지 않고 적절한 컨트롤러에 기준값을 전달할 수 있으므로 하드웨어 추상화(Hardware Abstraction)를 유지하고 제어 구성요소를 독립적으로 교체하거나 시험할 수 있다.

실제 4족 보행 로봇 아키텍처는 일반적으로 여러 개의 타이밍 계층(Timing Layer)을 사용한다. 모터 전류 또는 토크 제어(Motor Current or Torque Regulation)는 서보 드라이브(Servo Drive)나 임베디드 컨트롤러(Embedded Controller) 내부에서 가장 높은 주기로 실행되며, 관절 수준 제어(Joint-Level Control)와 상태 추정은 또 다른 결정론적 주기(Deterministic Rate)에서 수행된다. 강화학습 정책 추론은 이보다 낮은 주기로 실행될 수 있으며 추론 단계 사이에서는 행동값을 유지하거나 보간(Interpolation)할 수 있다. ROS 2는 모든 미들웨어 콜백이 가장 빠른 서보 주기에 참여하도록 요구하지 않으면서 이러한 제어 루프 주변의 시스템 통합을 담당한다.

상태 추정(State Estimation)은 학습된 정책이 신체 움직임(Body Motion)의 정확한 표현에 의존하기 때문에 매우 중요하다. 관성측정장치(IMU), 관절 엔코더(Joint Encoder), 접촉 정보(Contact Information), 운동학적 관계(Kinematic Relationship)를 융합하여 베이스 자세, 속도 및 직접 측정하기 어려운 기타 상태를 추정할 수 있다. 추정기의 지연(Estimator Latency)과 노이즈 특성(Noise Characteristic)은 정책 학습 과정에서 사용된 조건과 유사해야 하며, 시뮬레이션 관측과 실제 추정값 사이의 체계적인 차이는 보행 동작을 크게 변화시킬 수 있다.

발 접촉 정보(Foot Contact Information)는 물리적 상호작용(Physical Interaction)과 보행 제어를 연결하는 중요한 요소이다. 접촉 상태(Contact State)는 힘 센서(Force Sensor), 관절 토크(Joint Torque), 모터 전류(Motor Current), 운동학(Kinematics) 또는 이러한 신호들의 조합을 통해 추정할 수 있다. ROS 2 인터페이스는 접촉 정보를 모니터링 및 상위 모듈에 전달하며, 실시간 제어 경로(Real-Time Control Path)는 보행 안정화(Gait Stabilization)와 지형 상호작용(Terrain Interaction)에 필요한 충분히 최신의 접촉 정보를 정책 또는 상태 추정기에 제공한다.

상위 내비게이션(Higher-Level Navigation)은 강화학습 보행 정책과 분리된 상태로 유지되어야 한다. 내비게이션 구성요소(Navigation Component)는 로봇이 어디로 이동해야 하는지를 결정하고 속도(Velocity), 방향(Heading), 로컬 이동 명령(Local Motion Command)을 생성하며, 보행 정책은 해당 명령을 실현하기 위해 4족 로봇의 다리와 몸체를 어떻게 협조시킬지를 결정한다. 이러한 계층 구조는 모든 임무 시스템에 대해 정책을 다시 학습하지 않고도 동일한 보행 제어기를 자율 내비게이션(Autonomous Navigation), 원격조작(Teleoperation), 점검 임무(Inspection Mission), 작업 수준 계획(Task-Level Planning)에 사용할 수 있도록 한다.

시뮬레이션-실환경 전이(Sim-to-Real Transfer)는 강화학습 정책 배포의 핵심 과제 중 하나이다. 학습 환경은 일반적으로 질량(Mass), 마찰(Friction), 액추에이터 응답(Actuator Response), 센서 노이즈(Sensor Noise), 지연(Latency), 지형(Terrain), 외부 교란(External Disturbance)에 대해 도메인 랜덤화(Domain Randomization)를 적용하여 정책이 하나의 이상적인 시뮬레이터 설정에 의존하지 않도록 한다. 실제 배포에서는 관측값 전처리(Observation Preprocessing), 행동 스케일링(Action Scaling), 컨트롤러 동역학(Controller Dynamics), 물리 파라미터(Physical Parameter)가 학습 과정에서 경험한 운용 분포(Operating Distribution) 안에 유지되는지를 검증해야 한다.

안전 감독(Safety Supervision)은 정책 출력이 본질적으로 안전하다고 가정하는 대신 학습된 제어기 주변을 독립적으로 보호해야 한다. 관절 위치 및 속도 제한(Joint Position and Velocity Limit), 토크 제한(Torque Limit), 몸체 자세 임계값(Body Orientation Threshold), 통신 감시기(Communication Watchdog), 열 제한(Thermal Constraint), 비상 정지 조건(Emergency-Stop Condition)을 정책 추론과 독립적으로 평가할 수 있다. 허용 범위를 초과한 명령은 액추에이터에 전달되기 전에 제한(Clipping), 거부(Rejection), 또는 제어된 안전 행동(Controlled Safe Behavior)으로 대체할 수 있다.

수명주기 관리(Lifecycle Management)는 로봇이 준비되기 전에 강화학습 제어기가 활성화되는 것을 방지하는 데 유용하다. 센서 인터페이스(Sensor Interface), 상태 추정(State Estimation), 하드웨어 통신(Hardware Communication), 정책 로딩(Policy Loading), 컨트롤러 구성(Controller Configuration)을 초기화하고 검증한 이후 보행 기능을 활성화할 수 있다. 종료 또는 장애 복구 과정에서는 액추에이터 인터페이스를 해제하기 전에 정책을 비활성화하여 능동 보행(Active Locomotion) 상태에서 정지 상태 또는 사전에 정의된 안전 상태(Safe State)로 제어된 전환을 수행할 수 있다.

장애 처리(Fault Handling)는 일반적인 로봇 장애뿐만 아니라 학습 기반 제어에 특화된 장애도 고려해야 한다. 관측값 누락(Missing Observation), 오래된 타임스탬프(Stale Timestamp), 추론 시간 초과(Inference Overrun), 유효하지 않은 수치 출력(Invalid Numerical Output), 모델 로딩 실패(Model-Loading Failure), 상태 추정기 발산(Estimator Divergence), 액추에이터 장애(Actuator Fault), 예상하지 못한 몸체 자세(Unexpected Body Attitude)는 서로 다른 대응을 요구한다. 감시기(Watchdog)와 유효성 검사(Validity Check)는 손상된 관측값이나 지연된 정책 결과가 정상적인 액추에이터 명령으로 해석되어 제어 시스템으로 전달되는 것을 방지할 수 있다.

진단 및 추적(Diagnostics and Tracing)은 개발 및 현장 운용 과정에서 강화학습 통합 시스템을 관찰 가능하게 만든다. 정책 추론 시간(Policy Inference Time), 관측값 경과 시간(Observation Age), 행동값(Action Value), 제어 루프 타이밍(Control-Loop Timing), 관절 추종 오차(Joint Tracking Error), 접촉 상태(Contact State), 상태 추정 품질(Estimator Quality), 하드웨어 상태(Hardware Status)를 ROS 2 메시지와 함께 기록할 수 있다. 이러한 동기화 기록(Synchronized Record)은 불안정한 동작의 원인이 신경망 정책, 상태 추정, 미들웨어 지연, 액추에이터 동역학, 환경 조건 또는 인터페이스 구성 중 어디에서 발생하는지를 엔지니어가 구분할 수 있도록 한다.

검증(Validation)은 시뮬레이션에서 시작하여 점진적으로 실제 하드웨어에 가까운 단계로 진행해야 한다. 정책은 먼저 학습 시뮬레이터(Training Simulator)에서 실행한 후 시뮬레이션 하드웨어 인터페이스를 사용하는 ROS 2 환경에서 검증하고, 이후 하드웨어 인더루프 시험(Hardware-in-the-Loop Testing)과 제한된 실제 로봇 시험(Restrained Physical Experiment)으로 확장할 수 있다. 타이밍, 명령 제한, 비상 동작(Emergency Behavior), 상태 추정, 복구 메커니즘이 검증된 이후에만 제한 없는 보행(Unrestricted Locomotion)과 보다 복잡한 지형으로 시험 범위를 확대해야 한다.

결과적으로 이 시스템은 강화학습(Reinforcement Learning)을 보다 광범위한 4족 보행 로봇 ROS 2 스택(Quadruped ROS 2 Stack) 내부에서 제어되는 하나의 지능 구성요소(Intelligence Component)로 취급한다. ROS 2는 통신, 수명주기, 진단, 내비게이션 통합(Navigation Integration), 하드웨어 추상화를 제공하고, 강화학습 정책은 학습된 보행 행동(Learned Locomotion Behavior)을 제공한다. 실시간 제어(Real-Time Control), 상태 추정, 안전 감독, 시뮬레이션 검증(Simulation Validation), 장애 처리가 정책을 둘러싸는 구조를 통해 학습된 보행을 시뮬레이션에서 신뢰성 있는 실제 로봇 운용(Reliable Physical Robot Operation)으로 전환할 수 있는 배포 가능한 아키텍처(Deployable Architecture)를 구성한다.

## 12.05 Humanoid ROS2 Full Stack System Integration Case

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

완전한 ROS 2 시스템으로 통합된 휴머노이드 로봇(Humanoid Robot)은 보행(Locomotion), 균형(Balance), 조작(Manipulation), 인지(Perception), 계획(Planning), 상호작용(Interaction), 안전(Safety)이 하나의 조정된 물리 시스템으로 동작해야 하기 때문에 가장 복잡한 미들웨어 아키텍처(Middleware Architecture) 중 하나를 구성한다. 이동 플랫폼이나 고정형 매니퓰레이터와 달리 휴머노이드는 다수의 결합된 관절을 제어하면서 접촉 조건(Contact Condition)을 지속적으로 변경한다. 따라서 ROS 2는 명확한 인터페이스와 수명주기 경계(Lifecycle Boundary)를 통해 서로 다른 실시간 및 비실시간 서브시스템(Subsystem)을 연결하는 통합 백본(Integration Backbone) 역할을 한다.

로봇 기술 모델(Robot Description)은 전체 소프트웨어 스택의 구조적 기반을 제공한다. URDF, Xacro 또는 보완적인 로봇 모델은 몸통(Torso), 골반(Pelvis), 머리(Head), 팔(Arm), 손(Hand), 다리(Leg), 발(Foot), 관절(Joint), 관성 특성(Inertial Property), 충돌 형상(Collision Geometry), 제한값(Limit), 센서 프레임(Sensor Frame)을 정의한다. 동일한 모델이 시각화(Visualization), 운동학(Kinematics), 상태 추정(State Estimation), 모션 계획(Motion Planning), 충돌 검사(Collision Checking), ros2_control 및 상위 휴머노이드 애플리케이션에서 사용되므로 일관된 관절 명명(Joint Naming)과 좌표 규약(Coordinate Convention)이 필수적이다.

하드웨어 추상화(Hardware Abstraction)는 휴머노이드 소프트웨어를 개별 액추에이터(Actuator)와 임베디드 장치(Embedded Device)로부터 분리한다. ros2_control 하드웨어 인터페이스는 관절 위치(Position), 속도(Velocity), 힘 또는 토크(Effort), 온도(Temperature), 전류(Current), 명령 자원(Command Resource)을 표준화된 인터페이스를 통해 제공할 수 있다. EtherCAT, CAN, 산업용 이더넷(Industrial Ethernet), 제조사 전용 액추에이터 프로토콜(Proprietary Actuator Protocol)과 같은 통신은 이 경계 아래에 유지되므로 상위 제어 계층 전체에 장치별 통신 로직을 포함할 필요가 없다.

휴머노이드 제어(Humanoid Control)는 일반적으로 하나의 ROS 2 실행 주파수 대신 여러 타이밍 영역(Timing Domain)을 사용한다. 모터 전류 및 토크 루프(Motor Current and Torque Loop)는 드라이브나 임베디드 컨트롤러 내부에서 실행될 수 있으며, 관절 제어(Joint Control), 상태 추정, 균형 제어(Balance Control), 전신 제어(Whole-Body Control)는 각 기능에 적합한 결정론적 주기(Deterministic Rate)에서 동작한다. 계획, 인지, 작업 추론(Task Reasoning), 사용자 상호작용(User Interaction)은 상대적으로 낮은 주기로 실행될 수 있다. ROS 2는 모든 서브시스템을 하나의 제어 루프로 강제하지 않으면서 이러한 타이밍 영역 사이의 정보 교환을 조정한다.

상태 추정(State Estimation)은 안정적인 전신 동작(Whole-Body Behavior)에 필요한 동적 표현(Dynamic Representation)을 생성한다. 관성측정장치(IMU), 관절 엔코더(Joint Encoder), 발 접촉 정보(Foot Contact Information), 힘 또는 토크 센싱(Force or Torque Sensing), 운동학적 제약(Kinematic Constraint)을 융합하여 베이스 자세(Base Orientation), 속도(Velocity), 접촉 상태(Contact State), 신체 구성(Body Configuration)을 추정할 수 있다. 지연되거나 시간적으로 정렬되지 않은 측정값은 부동 베이스(Floating Base), 지지 발(Supporting Foot), 질량중심(Center of Mass), 주변 환경 사이의 추정 관계를 왜곡할 수 있으므로 타임스탬프 일관성(Timestamp Consistency)이 특히 중요하다.

보행 제어(Locomotion Control)는 원하는 로봇 움직임을 다리와 몸체의 협조된 동작으로 변환한다. 보행 시스템(Walking System)은 발걸음(Footstep), 질량중심 궤적(Center-of-Mass Trajectory), 스윙 발 궤적(Swing-Foot Trajectory), 균형 기준값(Balance Reference)을 생성하면서 측정된 상태에 지속적으로 적응할 수 있다. ROS 2는 보행 서브시스템 주변에서 명령과 모니터링 인터페이스를 제공하고, 지연 시간에 민감한 안정화(Stabilization)는 적절한 결정론적 제어 계층 내부에서 수행된다. 이를 통해 모듈성(Modularity)과 동적 균형 유지에 필요한 응답 속도를 동시에 확보할 수 있다.

전신 제어(Whole-Body Control)는 여러 목표와 제약 조건 아래에서 휴머노이드의 다수 자유도(Degrees of Freedom)를 통합적으로 조정한다. 컨트롤러는 관절, 토크, 마찰(Friction), 접촉 제약(Contact Constraint)을 만족하면서 질량중심 운동, 발 접촉, 몸통 자세(Torso Orientation), 손 목표(Hand Target), 관절 자세(Joint Posture), 운동량(Momentum)을 제어할 수 있다. 팔과 다리를 독립적인 기구로 취급하는 대신 전신 제어는 서로 경쟁하는 목표를 조정하여 조작이나 상체 움직임이 의도하지 않게 보행 안정성을 저해하지 않도록 한다.

조작(Manipulation)은 전체 스택에 또 하나의 협조된 서브시스템을 추가한다. 모션 계획은 현재 로봇 상태와 환경 표현(Environmental Representation)을 이용하여 팔, 손 또는 전신 궤적(Whole-Body Trajectory)을 생성할 수 있으며, ros2_control 또는 전용 관절 컨트롤러(Dedicated Joint Controller)가 생성된 기준값을 실행한다. 서 있거나 걷는 동안 수행되는 작업에서는 조작 명령이 균형 및 접촉 제약과 호환되어야 하므로 작업 수준 계획(Task-Level Planning), 팔 제어(Arm Control), 보행, 전신 제어 사이의 조정이 필요하다.

인지(Perception)는 내비게이션과 조작에 필요한 환경적 맥락(Environmental Context)을 제공한다. 카메라(Camera), 깊이 센서(Depth Sensor), LiDAR, 힘 센서(Force Sensor) 및 기타 장치를 ROS 2 드라이버와 처리 노드를 통해 통합하여 객체 정보(Object Information), 지형 형상(Terrain Geometry), 장애물(Obstacle), 사람 위치(Human Location), 의미론적 관측(Semantic Observation)을 제공할 수 있다. 인지 아키텍처는 모든 서브시스템이 원시 센서 스트림(Raw Sensor Stream)을 직접 처리하지 않도록 고대역폭 센서 처리(High-Bandwidth Sensor Processing)와 압축된 월드 상태 표현(Compact World-State Representation)을 구분해야 한다.

내비게이션(Navigation)은 개별 다리 관절을 직접 명령하는 대신 휴머노이드가 어디로 이동해야 하는지를 결정함으로써 보행 제어기보다 상위 계층에서 동작한다. 전역 및 로컬 계획(Global and Local Planning)은 경로(Path), 목표 자세(Target Pose), 속도 기준값(Velocity Reference)을 생성할 수 있으며, 보행 서브시스템은 이러한 목표를 실행 가능한 발걸음과 신체 움직임으로 변환한다. 이러한 계층 구조는 내비게이션 알고리즘이 보행 생성(Gait Generation)과 독립적으로 발전할 수 있도록 하며 서로 다른 보행 제어기가 동일한 임무 수준 내비게이션 아키텍처를 공유할 수 있게 한다.

작업 실행(Task Execution)은 보행, 조작, 인지, 상호작용을 완전한 로봇 행동(Robot Behavior)으로 통합한다. 상태 머신(State Machine), 행동 트리(Behavior Tree), ROS 2 액션(Action), 서비스(Service), 토픽(Topic)을 사용하여 작업 장소 접근, 객체 탐색, 파지(Grasping), 운반, 장비와의 상호작용 같은 장시간 실행 작업(Long-Running Operation)을 표현할 수 있다. 휴머노이드 작업은 여러 서브시스템에 걸쳐 수행되므로 성공(Success), 실패(Failure), 취소(Cancellation), 타임아웃(Timeout), 복구(Recovery) 상태를 명확하게 정의하여 장애를 일관된 방식으로 처리해야 한다.

수명주기 관리(Lifecycle Management)는 전체 스택의 제어된 기동 및 종료를 확립한다. 하드웨어 인터페이스, 센서, 상태 추정, 인지, 컨트롤러, 보행, 계획, 작업 구성요소는 임의적인 프로세스 순서가 아니라 종속 관계(Dependency Relationship)에 따라 활성화되어야 한다. 움직임을 허용하기 전에 통신, 관절 상태, 센서 유효성(Sensor Validity), 컨트롤러 준비 상태(Controller Readiness), 안전 조건을 검증할 수 있다. 종료 과정에서는 이러한 종속 관계를 역순으로 처리하여 하위 자원이 사라지기 전에 모션 기능(Motion Capability)을 먼저 제거한다.

안전 감독(Safety Supervision)은 정상적인 자율 행동을 무시하고 개입할 수 있는 권한을 유지해야 한다. 관절 및 토크 제한, 충돌 모니터링(Collision Monitoring), 낙상 감지(Fall Detection), 몸체 자세 임계값(Body-Orientation Threshold), 열 상태(Thermal Condition), 통신 감시기(Communication Watchdog), 보호 정지(Protective Stop), 비상 정지(Emergency Stop)는 서로 다른 아키텍처 계층에서 평가할 수 있다. ROS 2는 안전 상태를 전달하고 행동적 대응을 조정하지만, 안전이 중요한 정지 메커니즘(Safety-Critical Stopping Mechanism)은 미들웨어, 인지, 계획 또는 AI 구성요소가 실패하더라도 독립적인 제어 권한을 유지해야 한다.

낙상 관리(Fall Management)는 균형 상실이 일반적인 컨트롤러 오류로만 처리될 수 없기 때문에 별도로 다루어야 한다. 시스템은 자세(Orientation), 각속도(Angular Velocity), 접촉 상태, 지지 조건(Support Condition)을 통해 불안정성을 감지하고 정상 제어에서 보호 대응(Protective Response)으로 전환할 수 있다. 낙상 이후에는 복구 또는 일어서기 동작(Stand-Up Behavior)을 허용하기 전에 하드웨어 무결성(Hardware Integrity), 관절 상태, 센서 유효성, 주변 공간(Environmental Clearance), 컨트롤러 상태를 평가해야 한다.

진단 및 추적(Diagnostics and Tracing)은 높은 상호 결합성을 가진 전체 시스템에 대한 가시성(Visibility)을 제공한다. 관절 추종 오차(Joint Tracking Error), 제어 루프 타이밍(Control-Loop Timing), 상태 추정기 품질(State-Estimator Quality), 접촉 상태, 인지 지연(Perception Latency), 계획 결과(Planning Result), CPU 및 GPU 사용률, 액추에이터 상태, 네트워크 동작(Network Behavior), 수명주기 전환(Lifecycle Transition)을 동기화된 타임스탬프와 함께 기록할 수 있다. 관찰되는 보행이나 조작 장애의 실제 원인이 처리 체인의 여러 단계 이전에 위치한 다른 서브시스템일 수 있으므로 이러한 신호의 상관관계를 분석하는 것이 중요하다.

시뮬레이션(Simulation)은 제한되지 않은 실제 로봇 운용 전에 휴머노이드 스택을 통합하기 위한 가장 안전한 환경을 제공한다. 로봇 모델, 컨트롤러, 인지 파이프라인(Perception Pipeline), 내비게이션, 조작, 작업 행동을 먼저 시뮬레이션 하드웨어와 환경에서 검증할 수 있다. 이후 하드웨어 인더루프 시험(Hardware-in-the-Loop Testing)을 통해 실제 컨트롤러나 컴퓨팅 장치를 도입하고, 단계적으로 제한된 실제 로봇 시험을 수행하여 복잡한 자율 작업에 진입하기 전에 타이밍, 제한값, 균형 동작, 비상 대응(Emergency Handling), 복구 기능을 검증한다.

완성된 아키텍처는 휴머노이드를 단순히 다수의 관절에 연결된 하나의 컨트롤러가 아니라 분산 사이버 물리 시스템(Distributed Cyber-Physical System)으로 취급한다. ROS 2는 통신(Communication), 하드웨어 추상화, 수명주기 조정(Lifecycle Coordination), 진단, 모듈형 통합(Modular Integration)을 제공하며, 전문화된 계층은 인지, 상태 추정, 보행, 전신 제어, 조작, 내비게이션, 작업 지능(Task Intelligence)을 제공한다. 안전, 실시간 경계(Real-Time Boundary), 시뮬레이션, 장애 복구(Fault Recovery)가 이러한 기능을 둘러싸면서 더 높은 수준의 자율성을 갖는 휴머노이드 시스템으로 확장할 수 있는 기반을 구성한다.

## 12.06 ROS2 Security Hardened Operation Case

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

보안 강화 ROS 2 운용(Security-Hardened ROS 2 Operation)은 미들웨어를 암묵적으로 신뢰되는 로봇 네트워크(Implicitly Trusted Robot Network)에서 신원(Identity), 권한(Permission), 통신 경로(Communication Path), 소프트웨어 버전(Software Version), 운용 이벤트(Operational Event)가 명시적으로 관리되는 통제된 분산 시스템(Controlled Distributed System)으로 전환한다. 보안은 DDS 트래픽 암호화에만 국한되지 않고 전체 로봇 수명주기(Robot Lifecycle)를 포괄해야 한다. 따라서 아키텍처는 SROS2, DDS Security, 네트워크 분할(Network Segmentation), 호스트 강화(Host Hardening), 안전한 배포(Secure Deployment), 모니터링(Monitoring), 복구(Recovery)를 상호 연계된 보호 계층으로 결합한다.

보안 아키텍처(Security Architecture)는 로봇, 운영자(Operator), 플릿 서버(Fleet Server), 엔지니어링 컴퓨터(Engineering Computer), 외부 서비스(External Service)에 대한 명확한 신뢰 모델(Trust Model)을 정의하는 것에서 시작한다. 각 참여자(Participant)는 ROS 2 그래프 전체에 무제한으로 접근하는 대신 명시적인 신원과 제한된 운용 역할(Operational Role)을 가져야 한다. 시스템 기능에 따라 보안 경계(Security Boundary)를 설정하여 내비게이션(Navigation), 제어(Control), 진단(Diagnostics), 플릿 관리(Fleet Management), 개발(Development), 외부 통합(External Integration) 환경을 실제 통신 요구사항에 맞게 분리할 수 있다.

SROS2는 노드(Node)와 통신 엔드포인트(Communication Endpoint)에 DDS Security 기능을 적용하기 위한 ROS 2 도구를 제공한다. 보안 키스토어(Security Keystore)는 인증된 통신을 설정하는 데 필요한 거버넌스 정보(Governance Information), 권한(Permission), 인증서(Certificate), 개인 키(Private Key) 및 관련 보안 산출물을 포함한다. 노드는 할당된 보안 엔클레이브(Security Enclave)를 사용하여 모든 프로세스를 하나의 완전히 신뢰된 참여자로 취급하지 않고 기능적 경계에 따라 자격증명(Credential)과 정책(Policy)을 적용할 수 있다.

인증(Authentication)은 신뢰할 수 있는 통신이 설정되기 전에 DDS 참여자의 신원을 검증한다. 인증서 기반 자격증명(Certificate-Based Credential)을 사용하면 노드가 자신의 신원을 증명할 수 있으며, 비인가 프로세스(Unauthorized Process)가 단순히 ROS 2 도메인에 참여할 가능성을 줄일 수 있다. 따라서 인증서 발급(Issuance), 배포(Distribution), 갱신(Renewal), 폐기(Revocation), 만료(Expiration), 안전한 개인 키 저장(Secure Private-Key Storage)은 양산 로봇의 전체 수명주기에 걸쳐 관리해야 하는 운용 책임이 된다.

접근 제어(Access Control)는 인증된 참여자가 실제로 수행할 수 있는 작업을 결정한다. 권한 정책(Permission Policy)은 토픽(Topic) 또는 도메인 요구사항에 따라 발행(Publishing), 구독(Subscribing), 서비스 상호작용(Service Interaction), 기타 DDS 통신을 제한할 수 있다. 예를 들어 인지 노드(Perception Node)가 동일한 로봇에 속한다는 이유만으로 모션 명령(Motion Command)을 발행할 권한까지 자동으로 가질 필요는 없다. 최소 권한 정책(Least-Privilege Policy)은 구성요소가 침해되거나 잘못 설정되었을 때 발생할 수 있는 영향을 줄인다.

DDS Security는 기밀성(Confidentiality)과 무결성(Integrity)을 위한 암호화 메커니즘(Cryptographic Mechanism)을 사용하여 전송 중 데이터(Data in Transit)를 보호할 수 있다. 암호화(Encryption)는 수동적인 네트워크 감시로부터 민감한 로봇 정보가 노출되는 것을 줄이며, 무결성 보호(Integrity Protection)는 비인가 변경(Unauthorized Modification)을 탐지하는 데 도움을 준다. 그러나 암호 처리는 계산 및 통신 오버헤드(Computational and Communication Overhead)를 발생시키므로 양산 배포 전에 실제 메시지 주기, 페이로드 크기(Payload Size), 프로세서 부하(Processor Load), 네트워크 조건을 반영하여 보안 설정의 성능을 검증해야 한다.

네트워크 분할(Network Segmentation)은 통신이 물리적 또는 논리적으로 이동할 수 있는 범위를 제한하여 미들웨어 수준 보안을 보완한다. 로봇 제어 네트워크(Robot Control Network), 센서 네트워크(Sensor Network), 플릿 통신(Fleet Communication), 유지보수 인터페이스(Maintenance Interface), 기업 시스템(Enterprise System), 외부 연결(External Connectivity)은 VLAN, 방화벽(Firewall), 라우팅 정책(Routing Policy), 전용 인터페이스(Dedicated Interface)를 사용하여 분리할 수 있다. DDS 도메인은 추가적인 논리적 분리를 제공할 수 있지만 도메인 식별자(Domain Identifier) 자체를 강제 가능한 사이버보안 경계(Cybersecurity Boundary)와 동일하게 취급해서는 안 된다.

DDS 디스커버리(Discovery)는 자동 참여자 탐색이 시스템 구조를 노출하거나 의도하지 않은 네트워크 구간 사이에 통신을 발생시킬 수 있기 때문에 특별한 주의가 필요하다. 멀티캐스트 동작(Multicast Behavior), 디스커버리 서버(Discovery Server), 인터페이스 바인딩(Interface Binding), 라우팅(Routing), 허용된 피어 관계(Allowed Peer Relationship)는 배포 토폴로지(Deployment Topology)에 따라 구성해야 한다. 양산 시스템에서는 네트워크에서 접근 가능한 모든 장치가 전체 운용 네트워크의 디스커버리에 자유롭게 참여하도록 허용하는 것보다 예측 가능한 통신 경로(Predictable Communication Path)를 구성하는 것이 중요하다.

호스트 강화(Host Hardening)는 ROS 2가 실행되는 컴퓨팅 플랫폼을 보호한다. 불필요한 서비스와 포트(Port)를 비활성화하고, 소프트웨어 권한(Software Privilege)을 최소화하며, 관리자 접근(Administrative Access)을 통제하고, 운영체제 패키지를 정의된 업데이트 정책(Update Policy)에 따라 관리해야 한다. 파일 권한(File Permission)은 보안 산출물과 구성 데이터를 보호해야 하며, 애플리케이션 프로세스는 기능 수행에 필요한 최소한의 권한만 사용해야 한다. 미들웨어 보안만으로는 무제한적인 로컬 침해(Local Compromise)를 허용하는 호스트를 보호할 수 없다.

컨테이너화 또는 패키지 기반 배포(Containerized or Packaged Deployment)는 또 다른 보안 경계를 형성한다. ROS 2 애플리케이션, 종속성(Dependency), 구성 파일(Configuration File), 런타임 권한(Runtime Permission)은 통제된 소스로부터 빌드되어 식별 가능한 소프트웨어 산출물(Software Artifact)로 배포되어야 한다. 버전 관리(Versioning), 체크섬 또는 서명(Checksum or Signature), 소프트웨어 인벤토리(Software Inventory), 통제된 저장소(Controlled Repository)는 각 로봇에서 실제로 어떤 소프트웨어가 실행되고 있는지를 확인하는 데 도움을 준다. 배포 파이프라인(Deployment Pipeline)은 검토되지 않은 바이너리나 구성 변경이 양산 시스템에 조용히 유입되는 것을 방지해야 한다.

안전한 업데이트 메커니즘(Secure Update Mechanism)은 장기간 운용되는 로봇이 필연적으로 취약점 패치(Vulnerability Patch)와 소프트웨어 변경을 필요로 하기 때문에 필수적이다. 업데이트 과정은 업데이트 출처(Update Source)를 인증하고 산출물 무결성을 검증하며 호환 가능한 구성을 유지하고 배포 실패 시 롤백(Rollback)을 제공해야 한다. 단계적 배포(Staged Rollout)는 전체 플릿(Fleet)에 적용하기 전에 개발 로봇 또는 일부 양산 로봇에서 릴리스를 검증함으로써 결함이 있거나 침해된 업데이트가 초래할 수 있는 영향을 줄인다.

수명주기 관리(Lifecycle Management)는 보호된 구성요소가 언제 운용 상태로 전환될지를 통제함으로써 보안을 강화할 수 있다. 노드가 활성 상태(Active State)로 전환되기 전에 보안 자격증명, 통신 종속성(Communication Dependency), 하드웨어 인터페이스(Hardware Interface), 필수 서비스(Required Service)를 검증할 수 있다. 인증에 실패하거나 정책 파일이 유효하지 않거나 필요한 보안 서비스가 사용할 수 없는 경우, 시스템은 부분적으로 구성된 노드가 통제되지 않은 가정 아래 통신하도록 허용하는 대신 자율 운용(Autonomous Operation)을 차단할 수 있다.

보안 모니터링(Security Monitoring)은 보호 메커니즘을 관찰 가능한 운용 프로세스(Observable Operational Process)로 전환한다. 인증 실패(Authentication Failure), 거부된 권한(Denied Permission), 예상하지 못한 참여자(Unexpected Participant), 구성 변경(Configuration Change), 반복적인 연결 시도(Connection Attempt), 노드 재시작(Node Restart), 네트워크 이상(Network Anomaly), 업데이트 이벤트(Update Event)를 일반적인 ROS 2 진단 정보와 함께 수집할 수 있다. 중앙집중식 로그와 메트릭(Metric)을 통해 운영자는 일반적인 소프트웨어 장애와 구성 오류, 비인가 활동 또는 침입 시도(Attempted Intrusion)를 나타낼 수 있는 동작을 구분할 수 있다.

감사 로깅(Audit Logging)은 저장 공간을 과도하게 사용하거나 불필요한 민감 정보를 노출하지 않으면서 중요한 보안 이벤트를 재구성할 수 있을 정도의 충분한 맥락(Context)을 보존해야 한다. 관련 기록에는 타임스탬프(Timestamp), 참여자 신원(Participant Identity), 정책 결정(Policy Decision), 소프트웨어 버전, 관리자 작업(Administrative Action), 통신 실패(Communication Failure)가 포함될 수 있다. 여러 호스트의 시계가 크게 다르면 수집된 이벤트를 신뢰성 있게 연관 분석할 수 없으므로 로봇과 인프라 전체의 시간 동기화(Time Synchronization)가 중요하다.

보안 사고 대응(Security Incident Handling)은 물리적 안전(Physical Safety)과 함께 동작해야 한다. 사이버보안 이상을 탐지했다고 해서 모든 프로세스를 즉시 종료하는 것은 로봇이 이동 중이거나 화물을 운반하거나 사람 주변에서 동작하는 경우 적절하지 않을 수 있다. 시스템은 안전 중요 제어 기능(Safety-Critical Control Function)을 유지하면서 외부 통신 제한, 신규 임무 거부, 성능 저하 운용(Degraded Operation), 안전 정지(Safe Stop), 영향을 받은 서비스 격리(Service Isolation), 작업자 개입(Operator Intervention) 요청과 같은 제어된 대응을 정의해야 한다.

복구(Recovery)는 사고 발생 이후 ROS 2 노드를 단순히 재시작하는 것 이상의 절차를 필요로 한다. 자격증명을 폐기하거나 교체해야 할 수 있고, 침해된 소프트웨어를 신뢰된 산출물(Trusted Artifact)에서 복원해야 하며, 네트워크 접근을 격리하고 구성 무결성(Configuration Integrity)을 다시 검증해야 할 수 있다. 자율 운용을 재개하기 전에 하드웨어 상태, 소프트웨어 버전, 보안 정책, 통신 신뢰(Communication Trust), 수명주기 상태, 관련 안전 조건을 확인하여 복구 과정에서 기존 취약점이 다시 활성화되지 않도록 해야 한다.

보안 시험(Security Testing)은 시스템 통합(System Integration)과 릴리스 검증(Release Validation)에 포함되어야 한다. 시험을 통해 인증 동작, 통신 거부(Denied Communication), 인증서 오류(Certificate Failure), 만료된 자격증명(Expired Credential), 네트워크 격리(Network Isolation), 정책 적용(Policy Enforcement), 안전한 기동(Secure Startup), 로깅, 업데이트 롤백을 검증할 수 있다. 또한 성능 시험은 보안 메커니즘이 디스커버리 시간, CPU 부하, 대역폭(Bandwidth), 메시지 지연(Message Latency), 실시간 민감 통신에 미치는 영향을 측정하여 보안 기능 자체가 인지되지 않은 운용 장애를 발생시키지 않는지 확인해야 한다.

따라서 양산형 보안 강화 ROS 2 시스템(Production Security-Hardened ROS 2 System)은 하나의 보안 메커니즘에 의존하지 않고 심층 방어(Defense in Depth)를 적용한다. SROS2와 DDS Security는 미들웨어 신원과 통신을 보호하고, 네트워크 제어(Network Control)는 연결 범위를 제한하며, 호스트 및 배포 강화(Host and Deployment Hardening)는 실행 환경을 보호하고, 모니터링은 운용 가시성(Operational Visibility)을 제공한다. 여기에 수명주기 관리, 사고 대응(Incident Response), 복구, 지속적인 검증(Continuous Validation)이 결합되어 전체 ROS 2 보안 및 배포 아키텍처와 연계되는 다계층 보안 체계(Multi-Layer Security Architecture)를 완성한다.

## 12.07 ROS1 to ROS2 Migration Project Case

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

양산 로봇(Production Robot)을 ROS 1에서 ROS 2로 마이그레이션(Migration)하는 것은 단순한 소스 코드 변환 작업이 아니다. 이는 ROS Master 중심의 통신 모델에서 DDS 기반 분산 미들웨어 환경(Distributed Middleware Environment)으로 전환하는 아키텍처 변화이며, 노드 API(Node API), 서비스 품질(Quality of Service) 의미 체계, 수명주기 기능(Lifecycle Capability), 보안 메커니즘(Security Mechanism), 런치 시스템(Launch System), 배포 방식(Deployment Practice)도 함께 변화한다. 따라서 성공적인 마이그레이션은 개별 소프트웨어 구성요소를 교체하기 전에 기존 로봇의 전체 운용 동작을 이해하는 것에서 시작한다.

첫 번째 작업은 ROS 1 시스템에 대한 인벤토리(Inventory)를 구축하는 것이다. 노드(Node), 토픽(Topic), 서비스(Service), 액션(Action), 파라미터(Parameter), 런치 파일(Launch File), 메시지 정의(Message Definition), 장치 드라이버(Device Driver), 서드파티 패키지(Third-Party Package), 동적 재구성 인터페이스(Dynamic Reconfigure Interface), 진단(Diagnostics), 외부 애플리케이션(External Application)을 종속 관계와 함께 매핑한다. 구성요소는 비즈니스 중요도(Business Importance), 실시간 민감도(Real-Time Sensitivity), 하드웨어 종속성(Hardware Dependency), 교체 난이도(Replacement Difficulty), ROS 2 지원 여부에 따라 분류하여 단순한 패키지 개수가 아니라 운용 위험을 기준으로 마이그레이션 우선순위를 결정한다.

통신 그래프(Communication Graph)는 명시적인 인터페이스 계약(Interface Contract)으로 분석해야 한다. ROS 1 퍼블리셔(Publisher)와 서브스크라이버(Subscriber)는 ROS 2 토픽으로 매핑하고, 서비스와 액션은 상호작용 패턴(Interaction Pattern)과 타이밍 요구사항(Timing Requirement)에 따라 검토한다. 사용자 정의 메시지 패키지(Custom Message Package)는 호환되는 ROS 2 인터페이스 정의와 빌드 구성(Build Configuration)을 필요로 한다. 이 단계에서는 기존의 모든 ROS 1 통신 종속성을 그대로 재현하는 대신 사용되지 않는 인터페이스를 제거하고 각 인터페이스의 소유권(Ownership)을 명확하게 정의할 수 있다.

DDS는 의도적인 서비스 품질(QoS) 구성을 필요로 하는 새로운 통신 동작을 도입한다. ROS 2는 신뢰성(Reliability), 내구성(Durability), 히스토리(History), 깊이(Depth), 데드라인(Deadline), 활성 상태(Liveliness) 정책을 지원하기 때문에 ROS 1의 통신 가정을 항상 그대로 이전할 수 있는 것은 아니다. 센서 스트림(Sensor Stream)은 최선 노력 전송(Best-Effort Delivery)을 통해 최신성을 우선할 수 있는 반면, 명령이나 중요한 상태 정보는 신뢰성 있는 전송(Reliable Delivery)을 요구할 수 있다. 따라서 QoS 호환성은 최종 배포 단계의 조정 사항이 아니라 통신 노드 사이에서 직접 시험하고 검증해야 한다.

소스 마이그레이션(Source Migration)은 노드 초기화(Node Initialization), 퍼블리셔, 서브스크라이버, 서비스, 타이머(Timer), 파라미터, 실행기(Executor), 로깅(Logging), 종료 동작(Shutdown Behavior)의 변경을 필요로 한다. C++ 애플리케이션은 roscpp 개념에서 rclcpp로 이동하고, Python 노드는 rospy에서 rclpy로 마이그레이션한다. API 호출을 한 줄씩 기계적으로 변환하는 대신 콜백 그룹(Callback Group), 실행기, 컴포지션(Composition), 수명주기 관리와 같은 ROS 2 개념을 적용하여 기존 아키텍처를 단순화하거나 강화할 수 있는 부분을 식별해야 한다.

빌드 및 패키지 인프라(Build and Package Infrastructure) 역시 크게 변경된다. ROS 1의 catkin 패키지는 ament_cmake 또는 ament_python을 사용하는 ROS 2 패키지로 마이그레이션하며, 패키지 매니페스트(Package Manifest), 종속성 선언(Dependency Declaration), 인터페이스 생성(Interface Generation), 설치 규칙(Installation Rule), 워크스페이스 절차(Workspace Procedure)도 이에 맞게 변경한다. 마이그레이션 프로젝트 전반에서 재현 가능한 빌드(Reproducible Build)를 유지하여 개발자가 소프트웨어 변환 문제와 종속성 또는 개발 환경 불일치 문제를 명확하게 구분할 수 있도록 해야 한다.

런치 구성(Launch Configuration)은 ROS 2 런치가 다른 실행 모델과 Python 기반 기능을 제공하기 때문에 재설계가 필요하다. 기존 ROS 1 XML 런치 파일은 먼저 노드 종속성, 네임스페이스(Namespace), 리매핑(Remapping), 파라미터, 머신 가정(Machine Assumption), 기동 순서(Startup Ordering)를 기준으로 분석해야 한다. 이후 ROS 2 런치 아키텍처를 통해 필요한 동작을 재현하면서 재사용 가능한 런치 기술(Reusable Launch Description), 조건부 실행(Conditional Execution), 네임스페이스, 이벤트 핸들러(Event Handler), 구조화된 구성(Structured Configuration)을 활용하여 모듈성을 향상시킬 수 있다.

파라미터 처리(Parameter Handling) 역시 중요한 아키텍처 차이이다. ROS 1은 일반적으로 중앙집중식 파라미터 서버(Centralized Parameter Server)에 의존하는 반면, ROS 2 파라미터는 개별 노드에 속하며 노드 인터페이스(Node Interface)를 통해 접근한다. 따라서 파라미터 네임스페이스, YAML 파일, 런타임 업데이트(Runtime Update), 검증(Validation), 소유권을 다시 설계해야 한다. 동적 구성 동작(Dynamic Configuration Behavior)은 ROS 1의 dynamic_reconfigure 메커니즘이 그대로 유지된다고 가정하지 않고 적절한 ROS 2 파라미터 콜백(Parameter Callback)이나 애플리케이션 인터페이스를 통해 구현해야 한다.

하드웨어 통합(Hardware Integration)은 기존 드라이버가 수년에 걸쳐 축적된 장치별 지식을 포함할 수 있기 때문에 마이그레이션에서 가장 위험도가 높은 영역 중 하나이다. 센서 및 액추에이터 드라이버는 유지보수되는 ROS 2 구현이 존재하는지, 기존 ROS 1 드라이버를 포팅(Porting)해야 하는지, 또는 하드웨어 인터페이스를 재설계해야 하는지를 개별적으로 평가해야 한다. 제어 가능한 하드웨어의 경우 ros2_control을 활용하여 애플리케이션 수준 제어와 제조사별 통신(Vendor-Specific Communication) 사이에 표준화된 경계를 구성할 수 있다.

단계적 마이그레이션(Staged Migration)은 일반적으로 전체 양산 스택을 한 번에 교체하는 것보다 안전하다. 구성요소를 센싱(Sensing), 위치추정(Localization), 내비게이션(Navigation), 조작(Manipulation), 제어(Control), 진단, 플릿 통합(Fleet Integration)과 같은 기능 영역으로 분류할 수 있다. 각 영역을 정의된 인터페이스에 대해 마이그레이션하고 검증한 이후 다음 종속 영역을 변환한다. 이러한 접근은 측정 가능한 중간 상태(Intermediate State)를 생성하며 시스템 통합 과정에서 예상하지 못한 동작이 발생했을 때 동시에 분석해야 하는 변수의 수를 줄인다.

단계적 프로젝트에서는 ROS 1과 ROS 2의 일시적인 공존(Temporary Coexistence)이 필요할 수 있다. 기술적으로 적절하고 선택한 배포판(Distribution)에서 지원되는 경우 브리지(Bridge)를 사용하여 두 환경 사이에서 선택된 토픽이나 인터페이스를 교환할 수 있다. 그러나 추가적인 변환(Translation), 배포 복잡성, 인터페이스 제한(Interface Limitation), 문제 해결 경계(Troubleshooting Boundary)로 인해 장기적인 혼합 운용은 유지보수가 어려워질 수 있으므로 브리지는 최종 아키텍처가 아니라 마이그레이션 메커니즘(Migration Mechanism)으로 취급해야 한다.

내비게이션 마이그레이션(Navigation Migration)은 단순히 하나의 패키지 이름을 다른 이름으로 교체하는 작업이 아니다. ROS 1 내비게이션 스택에는 수년간 조정된 지도(Map), 코스트맵 파라미터(Costmap Parameter), 플래너(Planner), 복구 동작(Recovery Behavior), 위치추정 설정, 로봇별 수정 사항이 포함될 수 있다. Navigation2로 마이그레이션할 때는 수명주기 관리 서버(Lifecycle-Managed Server), 액션, 플러그인(Plugin), 행동 트리(Behavior Tree), 변경된 파라미터 구조와 같은 ROS 2 개념으로 구성을 변환하면서 기존 시스템의 의도된 운용 동작을 유지해야 한다.

수명주기 관리(Lifecycle Management)는 이전에 스크립트나 암묵적인 런치 순서로 구현했을 수 있는 기동 및 복구 동작을 개선할 기회를 제공한다. 센서, 위치추정, 내비게이션, 컨트롤러(Controller), 애플리케이션 구성요소는 검증된 종속 관계에 따라 설정 상태(Configured State)와 활성 상태(Active State)를 전환할 수 있다. 따라서 마이그레이션된 시스템은 자율 주행을 활성화하기 전에 명시적인 준비 조건(Readiness Condition)을 설정하고 개별 서브시스템에 장애가 발생했을 때 보다 구조화된 복구를 수행할 수 있다.

시험(Testing)은 마이그레이션된 노드가 오류 없이 실행되는지만 확인하는 것이 아니라 실제 동작을 비교해야 한다. 기록된 데이터셋(Recorded Dataset), 시뮬레이션 시나리오(Simulation Scenario), 하드웨어 인더루프 환경(Hardware-in-the-Loop Environment), 실제 로봇 시험을 사용하여 ROS 1 기준 시스템과 ROS 2 구현 사이의 위치추정 정확도, 내비게이션 완료율, 제어 응답(Control Response), CPU 사용률, 네트워크 동작, 기동 시간, 장애 복구를 비교할 수 있다. 기존 시스템을 폐기하기 전에 회귀 기준(Regression Criteria)을 설정하여 기능적 차이를 명확하게 확인하고 측정할 수 있도록 해야 한다.

미들웨어 동작, 실행기, 직렬화(Serialization), DDS 구성, 노드 컴포지션(Node Composition)이 ROS 1과 다르기 때문에 성능 검증(Performance Validation)이 특히 중요하다. 대표적인 작업 부하(Representative Workload)에서 종단간 지연(End-to-End Latency), 메시지 지터(Message Jitter), CPU 사용량, 메모리 사용률(Memory Utilization), 네트워크 대역폭(Network Bandwidth), 콜백 실행(Callback Execution)을 측정해야 한다. ros2_tracing 및 관련 관찰성 메커니즘(Observability Mechanism)을 활용하면 예상하지 못한 타이밍 문제가 애플리케이션 코드, 실행기 스케줄링(Executor Scheduling), DDS 통신 또는 하드웨어 인터페이스 중 어디에서 발생하는지를 식별할 수 있다.

보안 및 배포 방식(Security and Deployment Practice)도 마이그레이션 과정에서 현대화할 수 있다. ROS 2 시스템에는 SROS2 및 DDS Security와 함께 네트워크 분할(Network Segmentation), 통제된 권한(Controlled Permission), 호스트 강화(Host Hardening), 버전 관리된 구성(Versioned Configuration), 안전한 업데이트 절차(Secure Update Procedure)를 적용할 수 있다. 이러한 기능은 인증서 관리(Certificate Management), 디스커버리 동작(Discovery Behavior), 계산 오버헤드(Computational Overhead), 운용 복구(Operational Recovery)를 평가하지 않고 단순히 보안을 활성화할 경우 새로운 통합 문제가 발생할 수 있으므로 명확한 배포 전략에 따라 도입해야 한다.

최종 전환(Final Cutover)은 ROS 2 시스템이 정상 임무와 정의된 장애 시나리오에서 기존과 동등하거나 의도적으로 개선된 운용 동작을 입증한 이후에만 수행해야 한다. 전환 기간 동안 배포 시스템은 롤백 기능(Rollback Capability), 구성 추적성(Configuration Traceability), 소프트웨어 버전 기록(Software Version Record), 모니터링을 유지해야 한다. ROS 1 종속성과 임시 브리지를 모두 제거한 이후에는 완성된 ROS 2 아키텍처가 후속 개발, 배포, 보안 강화(Security Hardening), 플릿 규모 확장(Fleet-Scale Evolution)을 위한 새로운 통제 기준선(Controlled Baseline)이 된다.

## 12.08 ROS2 CI/CD Build and Operation Case

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 지속적 통합 및 지속적 배포(CI/CD) 빌드와 운용 사례는 소스 코드 변경(Source-Code Change)에서 검증된 로봇 배포(Validated Robot Deployment)에 이르는 통제된 소프트웨어 전달 경로(Software Delivery Path)를 구축한다. 로보틱스에서 지속적 통합(Continuous Integration)과 지속적 전달(Continuous Delivery)은 애플리케이션 컴파일뿐 아니라 미들웨어 인터페이스, 하드웨어 종속성, 구성 파일, 컨테이너, 지도, 보안 산출물, 런타임 동작까지 다루어야 한다. 따라서 파이프라인(Pipeline)은 단순한 개발 편의 기능이 아니라 운용 아키텍처(Operational Architecture)의 일부가 된다.

워크플로(Workflow)는 패키지, 인터페이스 정의(Interface Definition), 런치 파일(Launch File), 파라미터(Parameter), 시험(Test), 배포 매니페스트(Deployment Manifest), 관련 구성을 포함하는 버전 관리 ROS 2 소스 저장소(Version-Controlled ROS 2 Source Repository)에서 시작한다. 개발자는 브랜치(Branch) 또는 통제된 변경 요청(Controlled Change Request)을 통해 작업하며, 저장소는 소스 리비전(Source Revision)과 릴리스된 로봇 소프트웨어 사이의 추적 가능한 관계를 유지한다. 이에 따라 커밋 식별자(Commit Identifier)는 빌드 재현, 현장 문제 조사, 실제 양산 로봇에 적용된 소프트웨어 확인을 위한 기준이 된다.

지속적 통합(Continuous Integration)은 관련 소스 또는 구성 변경이 제출될 때마다 시작된다. CI 환경은 깨끗한 워크스페이스(Clean Workspace)를 생성하고 선언된 종속성(Declared Dependency)을 설치한 후 colcon과 패키지 빌드 시스템(Package Build System)을 사용하여 ROS 2 빌드를 수행한다. ament_cmake 또는 ament_python 기반 패키지는 매니페스트에 따라 컴파일되고 설치된다. 깨끗한 환경에서 빌드하면 개발자 워크스테이션에서는 숨겨져 있지만 다른 컴퓨터나 로봇에서는 실패할 수 있는 미선언 종속성(Undeclared Dependency)을 탐지하는 데 도움이 된다.

정적 검사(Static Check)와 패키지 수준 시험(Package-Level Test)은 비용이 큰 시스템 검증을 시작하기 전에 실행해야 한다. 포매팅(Formatting), 린팅(Linting), 종속성 검사(Dependency Check), 인터페이스 검증(Interface Validation), C++ 단위 시험(Unit Test), Python 시험, 패키지별 검증을 통해 명백한 결함을 신속하게 차단할 수 있다. gtest와 ament_cmake_gtest를 사용하는 ROS 2 패키지는 이러한 프로세스에 단위 시험을 통합할 수 있다. 조기 실패(Early Failure)는 이후 시뮬레이션, 하드웨어 인더루프(Hardware-in-the-Loop), 실제 로봇 시험에서 단순한 소프트웨어 결함을 발견하는 비용을 줄인다.

인터페이스 호환성(Interface Compatibility)은 분산 ROS 2 시스템에서 특히 중요하다. 메시지(Message), 서비스(Service), 액션(Action), 파라미터, 토픽 이름(Topic Name), 네임스페이스(Namespace), QoS 정책의 변경은 정상적으로 컴파일되더라도 런타임에서 다른 노드를 중단시킬 수 있다. 따라서 CI는 종속 패키지 사이의 인터페이스 계약(Interface Contract)을 검증하고 예상되는 퍼블리셔(Publisher), 서브스크라이버(Subscriber), 서비스, 액션 동작을 확인하는 통합 시험(Integration Test)을 수행해야 한다. 이러한 호환성 검사는 독립적으로 유지보수되는 로봇 서브시스템들이 동일한 미들웨어 환경을 공유할수록 더욱 중요해진다.

성공적인 빌드는 개발자 워크스페이스를 직접 배포 소스로 사용하는 대신 변경 불가능하고 식별 가능한 산출물(Immutable and Identifiable Artifact)을 생성해야 한다. 배포 전략에 따라 이러한 산출물에는 ROS 2 패키지, 바이너리 아카이브(Binary Archive), 컨테이너 이미지(Container Image), 구성 번들(Configuration Bundle), 완전한 로봇 소프트웨어 릴리스가 포함될 수 있다. 각 산출물은 버전, 소스 리비전, 빌드 환경(Build Environment), 종속성 집합(Dependency Set), 시험 결과와 연결되어야 하며, 이를 통해 배포된 시스템을 이후에 재구성하고 감사(Audit)할 수 있다.

컨테이너화(Containerization)는 ROS 2 애플리케이션을 필요한 사용자 공간 종속성(User-Space Dependency) 및 런타임 구성과 함께 패키징하여 재현성(Reproducibility)을 향상시킬 수 있다. Docker 이미지는 CI 검증 이후 자동으로 빌드되어 통제된 레지스트리(Controlled Registry)에 저장될 수 있다. 컨테이너 경계(Container Boundary)는 인지(Perception), 내비게이션(Navigation), 플릿 연결(Fleet Connectivity), 모니터링(Monitoring) 등의 애플리케이션 그룹을 분리할 수 있지만 하드웨어 접근, DDS 네트워킹, 실시간 요구사항, 장치 권한(Device Permission)은 컨테이너가 자동으로 올바른 로봇 동작을 제공한다고 가정하지 않고 명시적으로 설계해야 한다.

시뮬레이션(Simulation)은 여러 ROS 2 구성요소를 함께 실행할 수 있는 첫 번째 시스템 수준 환경(System-Level Environment)을 제공한다. 자동화된 시나리오(Automated Scenario)는 로봇 모델, 컨트롤러(Controller), 센서, 내비게이션 구성요소, 애플리케이션 노드를 시작하고 반복 가능한 조건에서 예상 동작을 검증할 수 있다. 시뮬레이션이 실제 물리 검증을 대체할 수는 없지만 후보 릴리스(Candidate Release)에 실제 로봇 하드웨어를 할당하기 전에 기동 종속성, 토픽 연결성(Topic Connectivity), 파라미터 구성, 내비게이션 동작, 장애 처리(Failure Handling)를 시험할 수 있다.

하드웨어 인더루프 시험(Hardware-in-the-Loop Testing)은 시뮬레이션만으로 완전히 재현하기 어려운 물리적 인터페이스까지 CI를 확장한다. 실제 컨트롤러, 임베디드 컴퓨터(Embedded Computer), 센서, 네트워크 장치, 액추에이터 인터페이스(Actuator Interface)를 자동화된 시험 인프라에 연결하면서 위험한 기계적 움직임은 제한하거나 격리할 수 있다. 이 단계는 소프트웨어가 제한 없는 로봇 운용에 승인되기 전에 타이밍 문제, 드라이버 비호환성, 펌웨어 종속성(Firmware Dependency), 통신 장애, 하드웨어 특화 동작을 탐지하는 데 유용하다.

성능 회귀 시험(Performance Regression Testing)은 기능적으로 정상적인 릴리스가 실시간 동작을 인지하지 못한 상태에서 저하시키는 것을 방지한다. 종단간 지연(End-to-End Latency), 메시지 지터(Message Jitter), CPU 및 메모리 사용률, 네트워크 대역폭(Network Bandwidth), 콜백 실행(Callback Execution), 기동 시간(Startup Time), 선택된 제어 루프 메트릭(Control-Loop Metric)을 허용된 기준선(Baseline)과 비교할 수 있다. ROS 2 트레이싱(Tracing)과 모니터링 메커니즘을 활용하면 변경으로 인해 발생한 미들웨어, 스케줄링, 통신 또는 애플리케이션 수준의 성능 저하를 식별할 수 있다.

보안 검사(Security Check)는 배포 시점까지 연기하지 않고 소프트웨어 전달 파이프라인 안에서 수행해야 한다. 종속성 검토(Dependency Review), 산출물 무결성 검증(Artifact Integrity Verification), 통제된 자격증명(Controlled Credential), 컨테이너 또는 패키지 검사, 배포 정책 검증을 ROS 2 보안 구성과 결합할 수 있다. 양산 산출물은 신뢰할 수 있는 빌드 출력(Trusted Build Output)에서 획득해야 하며, 서명(Signing) 또는 이에 상응하는 무결성 메커니즘(Integrity Mechanism)을 사용하여 CI 환경과 로봇 배포 사이에서 변조된 소프트웨어가 유입되는 것을 방지할 수 있다.

지속적 전달(Continuous Delivery)은 후보 버전이 정의된 품질 게이트(Quality Gate)를 통과한 이후에만 시작된다. 단순히 컴파일에 성공하는 것만으로는 충분하지 않으며 릴리스에는 단위 시험, 통합 시험, 시뮬레이션 시나리오, 보안 검사, 성능 임계값(Performance Threshold), 하드웨어 검증이 요구될 수 있다. 개발(Development), 통합(Integration), 스테이징(Staging), 양산(Production) 환경 사이에서 승격(Promotion)할 때 가능한 한 동일하게 식별되는 산출물을 유지하여 배포 직전에 소프트웨어가 다른 조건으로 다시 빌드되지 않도록 해야 한다.

로봇 배포(Robot Deployment)는 릴리스 실패가 물리적 움직임에 영향을 줄 수 있기 때문에 일반적인 웹 서비스 배포보다 보수적인 통제가 필요하다. 후보 버전은 먼저 개발 로봇(Development Robot)에 설치하고 이후 검증 장비(Validation Unit), 제한된 양산 로봇 그룹을 거쳐 더 넓은 플릿으로 확장할 수 있다. 각 단계에서는 상태 지표(Health Indicator)와 임무 결과(Mission Result)를 관찰한다. 이러한 점진적 배포 전략(Progressive Deployment Strategy)은 현실적인 운용 검증을 수행하면서 결함에 노출되는 로봇 수를 제한한다.

동일한 바이너리(Binary)라도 파라미터, 지도, DDS 설정, 런치 파일, 하드웨어 정의(Hardware Definition)가 변경되면 서로 다르게 동작할 수 있으므로 구성(Configuration)도 소프트웨어와 함께 버전 관리해야 한다. 따라서 양산 릴리스는 실행 가능한 소프트웨어뿐 아니라 해당 소프트웨어가 요구하는 구성 기준선(Configuration Baseline)까지 식별해야 한다. 구성 추적성(Configuration Traceability)을 확보하면 로봇이 서로 다른 동작을 보일 때 운영자가 코드 회귀(Code Regression)와 잘못된 파라미터 또는 환경 변경을 구분할 수 있다.

롤백(Rollback)은 예외적인 복구 절차가 아니라 필수적인 운용 기능(Operational Capability)이다. 모니터링에서 장애 증가, 내비게이션 성능 저하, 비정상적인 자원 사용, 기동 실패 또는 기타 릴리스 관련 문제가 발견되면 배포 시스템은 이전에 검증된 소프트웨어 및 구성 기준선을 복원할 수 있어야 한다. 롤백 절차 자체도 시험해야 하며, 종속성이나 데이터 형식(Data Format) 문제로 이전 버전이 정상적으로 재시작되지 못한다면 이론적인 이전 버전의 존재만으로는 충분한 보호를 제공할 수 없다.

관찰성(Observability)은 배포와 개발 사이의 피드백 루프(Feedback Loop)를 완성한다. ROS 2 로그(Log), 진단(Diagnostics), 시스템 메트릭(System Metric), 수명주기 상태(Lifecycle State), 자원 사용률(Resource Utilization), 네트워크 정보, 애플리케이션 이벤트(Application Event)를 릴리스 이후 집계할 수 있다. CI/CD는 로그 집계(Log Aggregation), 메트릭 모니터링(Metrics Monitoring), 환경 강화(Environment Hardening), OTA 업데이트(OTA Update), 대규모 플릿 운용(Large-Scale Fleet Operation)과 연결되며, 이를 통해 양산 텔레메트리(Production Telemetry)가 독립적인 유지보수 기능이 아니라 이후 소프트웨어 개선을 위한 입력 정보가 된다.

플릿 규모(Fleet Scale)에서 CI/CD는 서로 다른 운용 상태에 있는 다수 로봇의 소프트웨어 일관성(Software Consistency)을 유지하는 메커니즘이 된다. 배포 시스템은 어떤 로봇이 업데이트 대상인지, 각 로봇이 현재 어떤 버전을 실행하는지, 설치가 성공했는지, 배포 후 상태 검사(Post-Deployment Health Check)를 통과했는지를 파악해야 한다. OTA 메커니즘, 단계적 릴리스 그룹(Staged Release Group), 모니터링, 롤백을 결합하면 모든 로봇을 수동으로 유지보수되는 개별 컴퓨터로 취급하지 않고 배포된 ROS 2 시스템을 지속적으로 발전시킬 수 있는 통제된 경로가 형성된다.

최종적인 ROS 2 CI/CD 아키텍처는 소스 변경이 빌드(Build), 시험(Test), 패키징(Packaging), 검증(Validation), 승격(Promotion), 배포(Deployment), 관찰(Observation)을 거쳐 승인되거나 롤백되는 지속적 엔지니어링 루프(Continuous Engineering Loop)를 형성한다. 자동화된 소프트웨어 전달은 물리 로봇에 적합한 시뮬레이션, 하드웨어 검증, 보안, 구성 관리(Configuration Management), 텔레메트리, 운용 통제와 결합된다. 이를 통해 ROS 2 미들웨어 개발은 CI/CD, 컨테이너화, 관찰성, OTA, 플릿 운용이 상호 보완적인 수명주기 역량(Lifecycle Capability)을 구성하는 로봇 데브옵스(Robot DevOps) 체계로 확장된다.

## 12.09 Large-Scale Fleet ROS2 OTA Deployment Case

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

대규모 플릿 ROS 2 OTA 배포(Large-Scale Fleet ROS 2 OTA Deployment)는 일반적인 소프트웨어 전달을 수백 대 또는 수천 대의 로봇을 위한 조정된 운용 시스템(Coordinated Operational System)으로 확장한다. 목표는 단순히 패키지를 원격으로 전송하는 것이 아니라 어떤 로봇이 어떤 소프트웨어를 받을지, 언제 설치할지, 호환성을 어떻게 검증할지, 장애의 영향을 어떻게 제한할지를 통제하는 것이다. 따라서 OTA는 ROS 2 배포, 플릿 관리(Fleet Management), 관찰성(Observability), 보안(Security), 수명주기 제어(Lifecycle Control), 복구(Recovery)를 하나의 관리 프로세스로 결합한다.

아키텍처는 CI/CD 파이프라인 및 신뢰할 수 있는 산출물 저장소(Trusted Artifact Repository)에 연결된 중앙집중식 릴리스 관리 서비스(Centralized Release-Management Service)에서 시작한다. 검증된 ROS 2 패키지, 컨테이너 이미지(Container Image), 구성 번들(Configuration Bundle), 지도(Map), 관련 배포 산출물에는 변경 불가능한 버전(Immutable Version)과 릴리스 메타데이터(Release Metadata)가 할당된다. 플릿 플랫폼은 이러한 릴리스와 개별 로봇 사이의 관계를 관리하여 운영자가 모든 배치 로봇에서 실행되는 정확한 소프트웨어 및 구성 기준선(Configuration Baseline)을 식별할 수 있도록 한다.

플릿 인벤토리(Fleet Inventory)는 통제된 배포의 기반을 제공한다. 각 로봇은 안정적인 신원(Stable Identity)과 함께 하드웨어 유형, 컴퓨팅 플랫폼(Compute Platform), ROS 2 배포판(Distribution), 펌웨어 버전(Firmware Version), 설치된 패키지, 구성 리비전(Configuration Revision), 운용 사이트(Operational Site), 현재 상태(Health State)로 표현되어야 한다. 이를 통해 배포 결정은 접근 가능한 모든 로봇에 동일한 업데이트를 무조건 전송하는 대신 명시적인 적격성 규칙(Eligibility Rule)을 사용할 수 있으며, 이는 여러 하드웨어 또는 소프트웨어 세대가 혼재하는 플릿에서 특히 중요하다.

릴리스 호환성(Release Compatibility)은 업데이트가 로봇에 제공되기 전에 평가해야 한다. 패키지는 특정 센서 드라이버, 펌웨어 리비전, 커널 기능(Kernel Feature), DDS 설정, 메시지 정의(Message Definition), 하드웨어 인터페이스에 의존할 수 있다. 따라서 배포 서비스는 릴리스 요구사항을 로봇 인벤토리와 비교하고 설치 전에 호환되지 않는 조합을 거부해야 한다. 호환성 메타데이터(Compatibility Metadata)는 OTA를 단순한 원격 파일 복사에서 통제된 구성 및 종속성 관리(Configuration and Dependency Management)로 전환한다.

대규모 배포는 즉각적인 전체 플릿 배포 대신 단계적 롤아웃 그룹(Staged Rollout Group)을 사용해야 한다. 릴리스는 개발 로봇(Development Robot)에서 실험실 검증 장비(Laboratory Validation Unit), 선택된 현장 로봇(Field Robot), 제한된 양산 그룹(Limited Production Group), 최종적으로 전체 플릿으로 단계적으로 확대할 수 있다. 각 단계 이후 상태 메트릭(Health Metric)을 평가한 다음 승격(Promotion)을 계속한다. 이를 통해 결함의 운용 영향을 제한하고 CI, 시뮬레이션, 실험실 시험에서는 발견되지 않았던 환경 특화 문제(Environment-Specific Problem)를 탐지할 수 있다.

배포 스케줄링(Deployment Scheduling)은 각 로봇의 물리적 운용 상태(Physical Operating State)를 고려해야 한다. 로봇이 주행하거나 물체를 조작하거나 중요한 조건에서 충전하거나 핵심 임무를 수행하는 동안 소프트웨어를 설치하면 허용하기 어려운 운용 중단이 발생할 수 있다. 따라서 플릿 오케스트레이션(Fleet Orchestration)은 안전한 유지보수 시간대(Safe Maintenance Window)를 식별하고 OTA 작업을 임무 관리(Mission Management)와 조정해야 한다. 로봇은 활성 작업을 완료하고 통제된 상태로 전환한 후 설치를 시작하기 전에 준비 상태를 확인할 수 있다.

ROS 2 수명주기 관리(Lifecycle Management)는 애플리케이션 구성요소를 업데이트에 대비시키는 구조화된 메커니즘을 제공한다. 관리형 노드(Managed Node)는 종속성을 통제된 순서로 종료하면서 활성 운용 상태에서 비활성 상태(Inactive State) 또는 종료 상태(Finalized State)로 전환할 수 있다. 설치 이후 구성요소는 검증된 기동 종속성(Startup Dependency)에 따라 구성되고 활성화될 수 있다. 수명주기 기반 오케스트레이션(Lifecycle-Based Orchestration)은 통제되지 않은 프로세스 종료에 대한 의존성을 줄이고 업데이트 이후 예측 가능한 서비스 복귀(Return to Service)를 지원한다.

OTA 전송(OTA Transport)은 산출물 검증(Artifact Verification)과 분리되어야 한다. 패키지가 클라우드 서비스, 로컬 엣지 서버(Local Edge Server), 사이트 저장소(Site Repository), 기타 네트워크 메커니즘을 통해 전달되는지와 관계없이 로봇은 설치 전에 수신한 산출물의 신원과 무결성(Integrity)을 검증해야 한다. 체크섬(Checksum), 서명(Signature), 신뢰할 수 있는 저장소, 인증된 통신(Authenticated Communication), 통제된 자격증명(Controlled Credential)은 손상되거나 비인가된 소프트웨어가 실행 가능한 로봇 코드가 되는 것을 방지하는 데 도움을 준다.

네트워크 조건(Network Condition)은 플릿 OTA 설계에 큰 영향을 준다. 로봇은 서로 다른 대역폭과 신뢰성을 가진 이더넷(Ethernet), Wi-Fi, 사설 셀룰러 네트워크(Private Cellular Network), 간헐적 외부 연결(Intermittent External Link)을 통해 운용될 수 있다. 대용량 컨테이너 이미지나 소프트웨어 번들은 내비게이션, 텔레메트리(Telemetry), 안전 관련 트래픽과 불필요하게 경쟁해서는 안 된다. 다운로드 속도 제한(Download Throttling), 캐싱(Caching), 로컬 미러(Local Mirror), 재개 가능한 전송(Resumable Transfer), 예약 배포(Scheduled Distribution)를 통해 네트워크 혼잡을 줄이면서 지리적으로 분산된 사이트에서도 배포를 지속할 수 있다.

견고한 로봇 측 업데이트 에이전트(Robot-Side Update Agent)는 로컬 설치 트랜잭션(Local Installation Transaction)을 관리한다. 이 에이전트는 선행 조건(Prerequisite)을 확인하고 사용 가능한 저장 공간을 검사하며 산출물을 다운로드하거나 수신하고 무결성을 검증한 다음 새로운 소프트웨어를 준비하고 이전 상태를 기록하여 배포 정책에 따라 설치를 수행한다. 다운로드, 검증, 설치, 재시작, 검증이 서로 다른 운용 상태이므로 단순히 로봇을 '업데이트 완료'로 표시하지 않도록 에이전트는 상세한 진행 상태를 플릿 관리 시스템에 보고해야 한다.

ROS 2 동작은 파라미터(Parameter), 런치 파일(Launch File), 지도, DDS 설정, 하드웨어 기술(Hardware Description), 보안 정책(Security Policy)에 크게 의존하기 때문에 구성 관리(Configuration Management)는 소프트웨어 배포와 함께 수행되어야 한다. 소프트웨어와 구성 버전은 독립적으로 식별할 수 있어야 하지만 서로 호환되는 기준선(Compatible Baseline)을 통해 릴리스해야 한다. 이를 통해 검증되지 않은 조합을 방지하면서 바이너리를 불필요하게 교체하지 않고도 플릿 운영자가 구성을 업데이트할 수 있다.

설치 후 검증(Post-Installation Validation)은 업데이트가 실제로 운용 가능한 로봇을 만들어냈는지를 판단한다. 시스템은 프로세스 기동, 수명주기 상태(Lifecycle State), ROS 2 그래프 연결성(Graph Connectivity), 필수 토픽 및 서비스, 센서 가용성(Sensor Availability), 컨트롤러 상태(Controller State), 위치추정 준비 상태(Localization Readiness), 자원 사용률(Resource Usage), 진단 상태(Diagnostic Status)를 검증할 수 있다. 따라서 패키지 설치 성공이 배포 성공과 동일한 것은 아니며, 로봇은 다시 자율 임무에 투입되기 전에 정의된 상태 검사(Health Check)를 통과해야 한다.

카나리 배포(Canary Deployment)는 소수의 대표 로봇을 선택하여 양산 환경에 먼저 노출함으로써 추가적인 위험 통제 계층을 제공한다. 이러한 로봇은 단순히 선택하기 편리한 장비가 아니라 의미 있는 하드웨어, 운용 환경, 작업 부하(Workload)의 조합을 대표해야 한다. 이들의 텔레메트리는 릴리스가 더 큰 플릿으로 확대되기 전에 CPU 사용량 증가, 통신 장애, 내비게이션 회귀(Navigation Regression), 반복적인 재시작(Restart Loop), 애플리케이션 오류를 발견하는 데 활용할 수 있다.

대규모 배포에서는 개별 로봇을 수동으로 검사하는 방식에 의존할 수 없으므로 관찰성(Observability)이 필수적이다. 중앙집중식 로그 집계(Centralized Log Aggregation), 메트릭 수집(Metrics Collection), 배포 이벤트(Deployment Event), ROS 2 진단, 수명주기 전환(Lifecycle Transition), 네트워크 상태, 자원 사용률, 임무 결과를 통해 플릿 전체의 운용 상태를 확인할 수 있다. OTA는 로그 집계, Prometheus/Grafana 형태의 모니터링, 환경 강화(Environment Hardening), 대규모 플릿 운용과 연결되어 통합적인 배포 관찰 체계를 구성한다.

롤백(Rollback)은 배포 프로세스에서 정상적인 상태 전환(Normal State Transition)으로 설계해야 한다. 설치가 실패하거나 배포 후 상태 지표가 정의된 한계를 초과하면 로봇은 이전에 검증된 소프트웨어 및 구성 기준선을 복원해야 한다. 플릿 수준에서는 릴리스 관리자(Release Manager)가 추가 롤아웃을 중단하고 영향을 받은 그룹에 대한 롤백을 시작할 수 있다. 자동 롤백(Automated Rollback) 역시 로봇이 물리적 동작을 수행하는 도중 소프트웨어를 갑자기 교체하지 않도록 로봇의 안전과 운용 상태를 고려해야 한다.

대규모 배포는 하나의 원자적 트랜잭션(Atomic Transaction)처럼 동작하는 경우가 드물기 때문에 부분적 플릿 장애(Partial Fleet Failure)에 대한 명시적인 처리가 필요하다. 릴리스가 배포될 때 일부 로봇은 오프라인 상태이거나 충전 중이거나 임무를 수행하거나 네트워크에서 단절되어 있거나 일시적으로 비정상 상태일 수 있다. 따라서 플릿 시스템은 로봇별 배포 상태(Deployment State)를 추적하고 통제된 전환 기간 동안 서로 다른 소프트웨어 버전의 혼재를 허용하면서도 로봇이나 인프라 구성요소가 계속 통신하는 경우 필요한 인터페이스 호환성을 유지해야 한다.

배포 규모가 증가할수록 보안(Security)의 중요성도 커진다. 침해된 OTA 메커니즘은 전체 플릿에 비인가 소프트웨어를 배포할 수 있기 때문이다. 릴리스 승인(Release Authorization), 산출물 서명(Artifact Signing), 자격증명 보호(Credential Protection), 역할 기반 관리자 접근(Role-Based Administrative Access), 감사 로깅(Audit Logging), 네트워크 분할(Network Segmentation), 배포 환경 강화(Deployment-Environment Hardening)를 통해 빌드 인프라에서 로봇 설치에 이르는 전체 경로를 보호해야 한다. 따라서 OTA 보안은 독립적인 파일 전송 기능이 아니라 보다 광범위한 ROS 2 배포 및 사이버보안 아키텍처(Cybersecurity Architecture)의 일부가 된다.

완성된 대규모 ROS 2 OTA 아키텍처는 검증된 릴리스를 선택하고 배포(Distribution), 설치(Installation), 검증(Verification), 관찰(Observation)한 후 측정 가능한 플릿 상태(Fleet Health)에 따라 유지하거나 롤백하는 폐쇄형 운용 루프(Closed Operational Loop)를 형성한다. CI/CD, 수명주기 관리, 모니터링, 보안, 구성 제어(Configuration Control), 플릿 오케스트레이션을 OTA와 결합하면 추적성(Traceability)과 통제된 운용 위험(Controlled Operational Risk)을 유지하면서 로봇 소프트웨어를 지속적으로 발전시킬 수 있다. 이는 ROS 2 배포 및 데브옵스(Deployment and DevOps) 구조에서 정의되는 플릿 규모 배포 역량(Fleet-Scale Deployment Capability)을 구현한다.

## 12.10 Future ROS2 Evolution and Robotics Strategy

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2의 미래 발전(Future Evolution)은 로봇 미들웨어 프레임워크(Robot Middleware Framework)에서 점점 더 자율화되는 물리 시스템(Physical System)을 위한 분산 실행 기반(Distributed Execution Foundation)으로 전환하는 과정으로 이해해야 한다. 로봇에 더욱 풍부한 인지(Perception), AI 추론(AI Inference), 적응형 계획(Adaptive Planning), 이기종 가속기(Heterogeneous Accelerator), 플릿 수준 협업(Fleet-Level Coordination)이 통합됨에 따라 ROS 2는 결정론적 장치 제어(Deterministic Device Control)와 데이터 집약적 지능(Data-Intensive Intelligence)을 연결해야 한다. Hills Robotics에게 이러한 발전은 개별 AMR에서 통합 피지컬 AI 시스템(Integrated Physical AI System)으로 확장하기 위한 소프트웨어 기반을 제공한다.

ROS 2는 단일 온보드 컴퓨터(Onboard Computer) 내부가 아니라 점차 이기종 컴퓨팅 환경(Heterogeneous Computing Environment) 전반에서 동작하게 될 것이다. 로봇 아키텍처는 센서, 엣지 컴퓨터(Edge Computer), 온프레미스 인프라(On-Premise Infrastructure), 클라우드 서비스(Cloud Service)에 분산된 마이크로컨트롤러(Microcontroller), 실시간 프로세서(Real-Time Processor), CPU, GPU, NPU, 전용 AI 가속기(Dedicated AI Accelerator)를 결합할 수 있다. ROS 2는 이러한 자원 전반에서 일관된 통신 및 수명주기 계층(Communication and Lifecycle Layer)을 제공하면서 시간 중요 제어와 계산 집약적 AI 작업이 가장 적절한 위치에서 실행될 수 있도록 해야 한다.

AI가 물리적 제어(Physical Control)와 더욱 긴밀하게 연결되면서 실시간 기능(Real-Time Capability)은 계속해서 핵심 요소가 될 것이다. 인지 모델(Perception Model)과 추론 시스템(Reasoning System)은 가변적인 실행 시간을 허용할 수 있지만 모터 제어, 안전 모니터링(Safety Monitoring), 위치추정 업데이트(Localization Update), 동기화 센싱(Synchronized Sensing)은 제한된 타이밍 동작(Bounded Timing Behavior)을 요구한다. 따라서 미래 아키텍처는 결정론적 고속 경로(Deterministic Fast Path)와 계산적으로 유연한 지능 경로(Flexible Intelligence Path)를 분리하고, ROS 2 실행기(Executor), 실시간 운영 환경, DDS QoS, 하드웨어 인터페이스, 동기화된 시계(Synchronized Clock)를 사용하여 예측 가능한 상호작용을 유지해야 한다.

DDS는 계속해서 중요한 분산 통신 기반(Distributed Communication Foundation)을 제공하겠지만 대규모 로봇 시스템에서는 더욱 의도적인 통신 엔지니어링(Communication Engineering)이 필요하다. 신뢰성(Reliability), 내구성(Durability), 데드라인(Deadline), 활성 상태(Liveliness), 멀티캐스트 동작(Multicast Behavior), 디스커버리(Discovery), 네트워크 토폴로지(Network Topology)는 획일적인 기본 설정을 사용하는 대신 작업 부하 특성(Workload Characteristic)에 따라 구성해야 한다. Hills Robotics는 센서, 제어, 진단, AI 추론, 플릿 협업, 인프라 통신을 위한 통신 프로파일(Communication Profile)을 재사용 가능한 아키텍처 자산(Reusable Architectural Asset)으로 관리할 수 있다.

로봇이 여러 카메라, LiDAR, IMU, 액추에이터, 분산 컴퓨터, AI 파이프라인을 통합하면서 시간 동기화(Time Synchronization)는 더욱 중요해질 것이다. 소프트웨어 타임스탬프(Software Timestamp)만으로는 센싱과 물리적 이벤트 사이의 정확한 시간적 상관관계(Temporal Correlation)가 필요한 애플리케이션에 충분하지 않다. 따라서 ROS 2 아키텍처는 PTP, gPTP, PPS, 하드웨어 타임스탬핑(Hardware Timestamping), 하드웨어 트리거링(Hardware Triggering)과 함께 발전하여 분산된 관측(Observation)과 동작(Action)이 일관된 물리적 시간 기준(Physical Time Reference)에 연결될 수 있도록 해야 한다.

ROS 2 수명주기 관리(Lifecycle Management)는 노드 기동 제어(Node Startup Control)에서 보다 광범위한 운용 상태 관리 메커니즘(Operational State-Management Mechanism)으로 발전할 수 있다. 인지, 위치추정(Localization), 내비게이션(Navigation), 조작(Manipulation), AI 정책 실행(AI Policy Execution), 통신, 임무 서비스(Mission Service)는 검증된 종속성과 로봇 준비 상태(Robot Readiness)에 따라 활성화될 수 있다. Hills Robotics는 수명주기 기반 오케스트레이션(Lifecycle-Based Orchestration)을 활용하여 초기화(Initialization), 자율 운용(Autonomous Operation), 성능 저하 운용(Degraded Operation), 유지보수(Maintenance), 소프트웨어 업데이트, 복구(Recovery), 안전 종료(Safe Shutdown) 사이의 명시적인 상태 전환을 구축할 수 있다.

ROS 2와 AI 프레임워크(AI Framework) 사이의 경계 역시 점점 중요해질 것이다. 파운데이션 모델(Foundation Model), 비전-언어 모델(Vision-Language Model), 비전-언어-행동 모델(Vision-Language-Action Model), 월드 모델(World Model), 강화학습 정책(Reinforcement-Learning Policy), 학습 기반 인지 시스템(Learned Perception System)이 미들웨어와 제어 아키텍처를 대체해서는 안 된다. 대신 이러한 요소들은 정의된 인터페이스를 통해 센싱, 상태 추정(State Estimation), 계획(Planning), 제어, 안전 기능과 연결되는 지능 구성요소(Intelligence Component)로 동작함으로써 전체 로봇 소프트웨어 스택을 불안정하게 만들지 않으면서 학습 기반 지능을 발전시킬 수 있다.

월드 모델(World Model)은 로봇, 환경, 객체, 작업, 가능한 미래 상태에 대한 예측 표현(Predictive Representation)을 유지함으로써 중요한 상위 수준 지능 계층(Higher-Level Intelligence Layer)을 제공할 수 있다. ROS 2는 동기화된 관측 정보와 로봇 상태를 제공하고 통제된 인터페이스를 통해 목표(Goal), 예측(Prediction), 정책 출력(Policy Output)을 수신할 수 있다. 이러한 분리는 결정론적 로봇 기능과 적응형 지능(Adaptive Intelligence)이 공존할 수 있도록 하며 모든 서브시스템을 종단간 학습 모델(End-to-End Learned Model)로 변경하지 않고 기존 자율 시스템에서 피지컬 AI(Physical AI)로 발전할 수 있는 실용적인 경로를 제공한다.

다중 로봇 운용(Multi-Robot Operation)은 이러한 아키텍처를 개별 지능(Individual Intelligence)의 범위를 넘어 확장한다. 로봇들은 플릿 및 온프레미스 서비스를 통해 임무 상태(Mission State), 환경 정보(Environmental Information), 자원 가용성(Resource Availability), 작업 진행 상태(Task Progress), 선택된 학습 지식(Learned Knowledge)을 공유할 수 있다. Hills Robotics는 ROS 2를 로봇 수준 실행 기반(Robot-Level Execution Substrate)으로 배치하고 상위 수준 오케스트레이션(Higher-Level Orchestration)을 통해 집단 행동(Collective Behavior)을 조정할 수 있다. 결과적으로 이러한 아키텍처는 로컬 자율성(Local Autonomy)과 플릿 지능(Fleet Intelligence)을 분리하면서 두 수준 사이에 명시적인 인터페이스를 유지한다.

여러 로봇이 공장, 물류 시설, 산업 현장 또는 공공 인프라에서 운용될 경우 온프레미스 지능 계층(On-Premise Intelligence Layer)의 중요성은 더욱 커질 수 있다. 계산 집약적인 월드 모델 적응(World-Model Adaptation), 플릿 최적화(Fleet Optimization), 데이터 집계(Data Aggregation), 시뮬레이션, 모델 평가(Model Evaluation), 정책 배포(Policy Distribution)는 모든 로봇에서 중복 실행하는 대신 공유 GPU 인프라(Shared GPU Infrastructure)에서 실행할 수 있다. ROS 2 게이트웨이(Gateway)와 서비스 인터페이스는 로컬 제어 자율성을 유지하면서 이러한 집단 지능(Collective Intelligence)을 엣지 로봇과 연결할 수 있다.

물리적 로봇은 즉각적인 인지와 제어를 원격 인프라에 완전히 의존할 수 없기 때문에 엣지 컴퓨팅(Edge Computing)은 계속해서 필수적이다. Hills Robotics는 안전 중요 제어(Safety-Critical Control), 위치추정, 장애물 회피(Obstacle Avoidance), 필수 자율 기능(Essential Autonomy)을 로컬에서 유지하고, 보다 무거운 AI 추론과 집단 최적화(Collective Optimization)는 지연(Latency), 대역폭(Bandwidth), 컴퓨팅 가용성(Compute Availability), 운용 위험에 따라 분산하는 계층형 아키텍처(Layered Architecture)를 유지할 수 있다. 이를 통해 외부 연결이 제한되거나 사용할 수 없는 상황에서도 단계적 성능 저하(Graceful Degradation)가 가능하다.

시뮬레이션 및 디지털 트윈 환경(Digital-Twin Environment)은 ROS 2 엔지니어링 수명주기(Engineering Lifecycle)의 필수적인 부분이 될 것이다. 로봇 모델, 센서, 컨트롤러, 내비게이션 소프트웨어, AI 정책, 플릿 동작을 실제 배포 전에 평가하고 이후 현장 텔레메트리(Field Telemetry)와 비교할 수 있다. 따라서 시뮬레이션은 CI/CD, 회귀 시험(Regression Testing), 하드웨어 인더루프 검증(Hardware-in-the-Loop Validation), 운용 데이터와 직접 연결되어 개발 과정이 가상 환경과 실제 배포 로봇 사이를 순환하는 지속적인 루프(Continuous Loop)가 되도록 해야 한다.

점점 복잡해지는 로봇 지능과 함께 배포 아키텍처(Deployment Architecture)도 발전해야 한다. 컨테이너화(Containerization), 재현 가능한 빌드(Reproducible Build), 구성 관리(Configuration Management), CI/CD, OTA 업데이트(OTA Update), 단계적 롤아웃(Staged Rollout), 롤백(Rollback), 플릿 관찰성(Fleet Observability)은 프로젝트별 추가 기능이 아니라 표준 역량(Standard Capability)이 되어야 한다. 모델이 로봇 소프트웨어의 일부가 됨에 따라 모델 버전(Model Version), 보정 데이터(Calibration Data), 추론 구성(Inference Configuration), 정책 산출물(Policy Artifact)도 기존 ROS 2 패키지와 동일한 수준의 추적성(Traceability)과 호환성 관리(Compatibility Management)를 적용해야 한다.

로봇의 연결성과 소프트웨어 정의 특성(Software-Defined Characteristic)이 강화되면서 보안(Security)은 시스템 수준 요구사항(System-Level Requirement)이 될 것이다. SROS2와 DDS Security는 미들웨어 신원과 통신을 보호할 수 있으며, 네트워크 분할(Network Segmentation), 보안 부팅 체인(Secure Boot Chain), 서명된 산출물(Signed Artifact), 통제된 자격증명(Controlled Credential), 호스트 강화(Host Hardening), 감사 로깅(Audit Logging), 안전한 OTA(Secure OTA)는 보다 광범위한 실행 환경을 보호한다. Hills Robotics는 이러한 메커니즘을 운용 수명주기 관리(Operational Lifecycle Management)와 통합하여 개발에서 플릿 배포까지 보안이 지속적으로 적용될 수 있도록 해야 한다.

관찰성(Observability)은 점점 더 자율화되는 시스템을 책임 있게 운용하기 위해 필요한 근거(Evidence)를 제공한다. ROS 2 로그(Log), 트레이스(Trace), 진단(Diagnostics), 수명주기 상태(Lifecycle State), 네트워크 메트릭(Network Metric), AI 추론 성능(AI Inference Performance), 임무 결과(Mission Outcome), 안전 이벤트(Safety Event), 하드웨어 상태(Hardware Health)를 여러 로봇에서 집계할 수 있다. 이러한 정보는 디버깅과 유지보수를 지원하는 동시에 시뮬레이션 개선, 모델 평가, 플릿 최적화, 향후 학습 파이프라인(Learning Pipeline)을 위한 구조화된 운용 데이터(Structured Operational Data)를 생성한다.

따라서 Hills Robotics는 ROS 2를 최종 지능 계층(Final Intelligence Layer)이 아니라 물리적 장치, 결정론적 제어, 자율 기능(Autonomous Function), AI 구성요소, 플릿 인프라를 연결하는 분산 신경계(Distributed Nervous System)로 활용할 수 있다. 로봇 수준 실행은 모듈화되고 하드웨어 인지형(Hardware-Aware)으로 유지하면서 온프레미스 및 상위 수준 지능이 학습과 집단 운용을 조정할 수 있다. 이러한 아키텍처 분리는 결합도(Coupling)를 줄이고 컴퓨팅 플랫폼, AI 모델, 센서, 로봇 유형이 서로 독립적으로 발전할 수 있도록 한다.

장기 전략(Long-Term Strategy)은 ROS 2 기반 자율 로봇에서 엣지 지능(Edge Intelligence), 공유 온프레미스 지능(Shared On-Premise Intelligence), 플릿 오케스트레이션(Fleet Orchestration)이 명시적인 인터페이스를 통해 협력하는 확장 가능한 피지컬 AI 아키텍처(Scalable Physical AI Architecture)로 발전하는 것이다. ROS 2는 통신, 수명주기, 하드웨어 추상화(Hardware Abstraction), 배포, 보안, 관찰성의 기반을 제공하고 상위 지능 계층은 적응(Adaptation), 예측(Prediction), 추론(Reasoning), 집단 행동을 제공한다. Hills Robotics에게 이는 AMR 소프트웨어에서 다중 로봇 피지컬 AI 시스템(Multi-Robot Physical AI System)으로 발전하기 위한 실질적인 진화 경로(Evolutionary Path)를 제공한다.
