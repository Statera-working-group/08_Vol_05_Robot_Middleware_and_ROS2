**Volume 05 Robot Middleware and ROS2**

# 11. ROS2 for Legged Robots

## 11.01 Legged Robot ROS2 SW Stack Special Requirements

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

다족보행 로봇(Legged Robot)은 이동이 몸체(Body), 여러 개의 관절형 다리(Articulated Legs), 불확실한 환경(Environment) 사이의 지속적인 상호작용에 의존하기 때문에 바퀴형 이동 로봇(Wheeled Mobile Robot)과 근본적으로 다른 소프트웨어 요구사항을 가진다. 따라서 이러한 시스템의 ROS2 소프트웨어 스택(ROS2 Software Stack)은 분산 미들웨어(Distributed Middleware)의 유연성과 결정론적 저수준 제어(Deterministic Low-Level Control), 동기화된 센싱(Synchronized Sensing), 고속 상태 추정(State Estimation), 접촉 상태 기반 동작(Contact-Dependent Behavior)의 명시적 관리를 함께 제공해야 한다.

아키텍처(Architecture)는 일반적으로 모든 기능을 동일한 ROS2 노드(Node)로 취급하기보다 제어 중요도(Control Criticality)에 따라 구분된다. 모터 전류(Current), 토크(Torque), 위치(Position), 관절 수준 서보 루프(Joint-Level Servo Loop)는 전용 모터 제어기(Motor Controller), 마이크로컨트롤러(MCU), FPGA 또는 실시간 프로세서(Real-Time Processor)에서 실행될 수 있으며, ROS2는 보행 생성(Gait Generation), 상태 추정, 인지(Perception), 내비게이션(Navigation), 진단(Diagnostics), 정책 추론(Policy Inference)과 같은 상위 기능을 조정한다. 이러한 분리는 미들웨어의 타이밍 변동성이 보행 안정성을 직접 저해하는 것을 방지한다.

다족보행(Locomotion)은 일반적인 내비게이션 워크로드(Navigation Workload)보다 훨씬 높은 갱신 주기(Update Rate)와 엄격한 타이밍 일관성(Timing Consistency)을 요구한다. 관절 센싱(Joint Sensing), 관성 측정(Inertial Measurement), 액추에이터 피드백(Actuator Feedback), 발 접촉 상태(Foot Contact State), 추정된 몸체 움직임(Estimated Body Motion)은 거의 동일한 물리적 시점(Physical Instant)을 표현해야 한다. 따라서 타임스탬프 출처(Timestamp Provenance)가 매우 중요하며, 하드웨어 타임스탬프(Hardware Timestamp), 동기화된 클록(Synchronized Clock), 제한된 통신 지연(Bounded Communication Latency), 정교하게 설계된 센서 획득 파이프라인(Sensor Acquisition Pipeline)이 필요하다.

관절 상태 통신(Joint-State Communication)은 소프트웨어 스택의 핵심 데이터 경로(Data Path) 중 하나를 형성한다. 위치, 속도(Velocity), 힘(Effort), 온도(Temperature), 액추에이터 상태(Actuator Status), 명령 기준값(Commanded Reference)이 하드웨어 인터페이스(Hardware Interface)와 상위 제어기 사이에서 교환될 수 있다. sensor_msgs/JointState와 같은 표준 ROS2 인터페이스는 상호운용성(Interoperability)을 제공하지만, 고주파 제어(High-Frequency Control)에서는 불필요한 직렬화(Serialization), 메모리 할당(Allocation), 복사(Copying), 스케줄링 지터(Scheduling Jitter)를 줄이기 위해 최적화된 내부 표현(Internal Representation), 공유 메모리(Shared Memory), 실시간 안전 버퍼(Real-Time-Safe Buffer)가 필요할 수 있다.

로봇 모델(Robot Model)은 거의 모든 보행 하위 시스템이 일관된 운동학적·동역학적 관계(Kinematic and Dynamic Relationships)에 의존하기 때문에 또 다른 핵심 요구사항이다. URDF는 링크(Link), 관절(Joint), 관성 파라미터(Inertial Parameter), 충돌 형상(Collision Geometry), 센서(Sensor), ROS2 통합을 표현할 수 있으며, MJCF와 같은 보완적 표현은 물리 시뮬레이션(Physics Simulation)과 학습 환경(Learning Environment)을 지원할 수 있다. 소프트웨어 아키텍처는 시뮬레이션 모델(Simulation Model), 제어기 모델(Controller Model), 시각화 모델(Visualization Model), 실제 로봇 구성(Physical Robot Configuration) 사이의 통제된 일관성을 유지해야 한다.

좌표 프레임 관리(Coordinate-Frame Management)는 다족보행 로봇이 움직이는 부동 베이스(Floating Base)와 동시에 움직이는 여러 접촉 체인(Contact Chain)을 포함하기 때문에 특히 까다롭다. tf2 트리(tf2 Tree)는 일반적으로 월드(World), 맵(Map), 오도메트리(Odometry), 베이스(Base), 몸통(Torso), 엉덩이(Hip), 무릎(Knee), 발목(Ankle), 발(Foot), 센서(Sensor), 페이로드(Payload) 프레임을 연결한다. 오래되거나 일관되지 않은 변환(Transform)은 지형 인지(Terrain Perception), 발 배치(Foot Placement), 위치 추정(Localization), 조작(Manipulation), 전신 운동 계산(Whole-Body Motion Calculation)을 손상시킬 수 있으므로 변환 발행 주기와 타임스탬프를 신중하게 설계해야 한다.

발 접촉(Foot Contact)은 단순한 또 하나의 센서 신호가 아니라 로봇의 동역학적 제약조건(Dynamic Constraint)을 변화시키는 이산적인 물리 상태(Discrete Physical Condition)이다. 접촉 추정(Contact Estimation)은 힘 또는 토크 센싱(Force or Torque Sensing), 관절 정보(Joint Information), 모터 전류(Motor Current), IMU 관측, 촉각 센싱(Tactile Sensing), 모델 기반 추정기(Model-Based Estimator)를 결합할 수 있다. ROS2 인터페이스는 접촉 상태(Contact State), 신뢰도(Confidence), 접촉 위치(Contact Location), 접촉력 정보(Contact Force Information), 타이밍 정보를 표현하여 보행 제어기, 상태 추정기, 지형 적응 모듈(Terrain Adaptation Module), 안전 기능(Safety Function)이 지지 상태(Support Condition)를 일관되게 해석하도록 해야 한다.

보행 하위 시스템(Gait Subsystem)은 일반적인 선속도(Linear Velocity)와 각속도(Angular Velocity) 명령 이상의 정보를 표현할 수 있는 인터페이스를 요구한다. 제어기는 보행 유형(Gait Type), 명령된 몸체 속도(Commanded Body Velocity), 몸체 높이(Body Height), 입각 파라미터(Stance Parameter), 스텝 주파수(Step Frequency), 스윙 지속시간(Swing Duration), 발 궤적 목표(Foot Trajectory Target), 지형 적응 파라미터, 전환 요청(Transition Request)을 필요로 할 수 있다. ROS2는 안정적인 감독 인터페이스(Supervisory Interface)를 제공하면서 고대역폭 궤적 생성(High-Bandwidth Trajectory Generation)과 피드백 제어(Feedback Control)는 결정론적 실행 계층(Deterministic Execution Layer)에 가깝게 유지해야 한다.

상태 추정(State Estimation)은 로봇의 베이스가 일반적으로 월드(World)에 대해 직접 구동되지 않으며 그 자세(Pose)를 여러 정보원으로부터 추론해야 하기 때문에 특히 중요하다. IMU 데이터, 관절 엔코더(Joint Encoder), 접촉 제약조건(Contact Constraint), 비주얼 오도메트리(Visual Odometry), 라이다 오도메트리(LiDAR Odometry), 위성항법시스템(GNSS) 등의 관측값이 위치(Position), 자세(Orientation), 속도(Velocity), 지지 상태(Support State)의 추정에 사용될 수 있다. 생성된 추정값은 특히 접촉 전환(Contact Transition), 충격(Impact), 미끄러짐(Slipping), 빠른 몸체 움직임 상황에서 제어기 입력과 시간적으로 일관되어야 한다.

ROS2 실행기 구성(ROS2 Executor Configuration)은 콜백(Callback)의 서로 다른 중요도를 반영해야 한다. 높은 우선순위의 상태 갱신(State Update)과 제어기 인터페이스는 시각화(Visualization), 로깅(Logging), 진단 또는 계산량이 많은 인지 콜백에 의해 지연되어서는 안 된다. 필요에 따라 콜백 그룹(Callback Group), 전용 실행기(Dedicated Executor), CPU 친화성(CPU Affinity), 스레드 우선순위(Thread Priority), PREEMPT_RT, 메모리 잠금(Memory Locking), 사전 할당(Pre-Allocation)을 결합할 수 있다. 목표는 모든 ROS2 동작을 하드 실시간(Hard Real-Time)으로 가정하는 것이 아니라 핵심 경로(Critical Path)에 대해 제한된 지연과 지터를 확보하는 것이다.

DDS 서비스 품질 정책(QoS Policy) 역시 각 데이터 스트림(Data Stream)의 의미에 따라 선택해야 한다. 고주파 센서 또는 상태 토픽(Topic)은 데이터 최신성(Freshness)과 제한된 큐(Bounded Queue)를 우선할 수 있는 반면, 구성(Configuration), 모드 변경(Mode Change), 안전 이벤트(Safety Event), 생명주기 정보(Lifecycle Information)는 더 강한 신뢰성(Reliability)을 요구할 수 있다. 대용량 카메라 및 포인트 클라우드(Point Cloud) 스트림이 제어 관련 정보를 전달하는 통신 경로를 혼잡하게 만들어서는 안 된다. 네트워크 분할(Network Segmentation), 프로세스 구성(Process Composition), 프로세스 내부 통신(Intra-Process Communication), 공유 메모리 메커니즘(Shared-Memory Mechanism)을 통해 이러한 간섭을 줄일 수 있다.

학습 기반 보행(Learning-Based Locomotion)은 추가적인 실행 경로(Execution Path)를 도입한다. 강화학습 정책(Reinforcement-Learning Policy)은 관절 상태, 추정된 베이스 움직임, 이전 행동(Previous Action), 접촉 정보, 지형 관측(Terrain Observation), 사용자 명령(User Command)을 입력받아 관절 목표(Joint Target) 또는 잠재 보행 명령(Latent Locomotion Command)을 생성할 수 있다. ROS2는 추론(Inference), 모니터링(Monitoring), 정책 관리(Policy Management)를 위한 실용적인 통합 경계(Integration Boundary)를 제공하지만, 추론 지연(Inference Latency), 관측 순서(Observation Ordering), 정규화(Normalization), 행동 제한(Action Limit), 데드라인 누락(Missed Deadline)에 대한 동작을 명확하게 제어해야 한다.

안전 메커니즘(Safety Mechanism)은 상위 ROS2 노드, AI 정책(AI Policy), 인지 모듈, 네트워크 연결이 실패하는 상황에서도 계속 유효해야 한다. 명령 감시기(Command Watchdog), 액추에이터 제한(Actuator Limit), 관절 한계 보호(Joint-Limit Protection), 토크 및 속도 제한, 넘어짐 감지(Fall Detection), 통신 타임아웃 처리(Communication Timeout Handling), 비상 상태(Emergency State), 제어된 종료 동작(Controlled Shutdown Behavior)을 적절한 하드웨어 및 소프트웨어 계층에 구현해야 한다. 따라서 학습된 정책은 액추에이터에 대한 유일한 제어 권한이 되는 것이 아니라 독립적으로 강제되는 안전 경계(Safety Envelope) 내부에서 동작해야 한다.

생명주기 관리(Lifecycle Management) 또한 단순히 모든 노드를 동시에 시작하는 것보다 복잡하다. 하드웨어 인터페이스, 시간 동기화(Time Synchronization), 센서, 상태 추정, 제어기, 인지, 보행 정책(Locomotion Policy), 내비게이션 기능 사이에는 안전한 활성화 순서(Safe Activation Order)를 결정하는 의존관계(Dependency)가 존재한다. ROS2 생명주기 메커니즘은 설정(Configuration), 활성화(Activation), 비활성화(Deactivation), 복구(Recovery), 종료(Shutdown)를 조정할 수 있으며, 상태 점검(Health Check)을 통해 필요한 센서, 변환, 추정기, 액추에이터 인터페이스가 정상화된 이후에만 보행을 활성화하도록 해야 한다.

진단(Diagnostics)은 일반적인 CPU, 메모리, 네트워크, 노드 상태뿐만 아니라 보행에 특화된 상태도 포착해야 한다. 중요한 관측 항목에는 제어 루프 주파수(Control-Loop Frequency), 데드라인 누락, 관절 추종 오차(Joint Tracking Error), 액추에이터 포화(Actuator Saturation), 모터 온도(Motor Temperature), 접촉 일관성(Contact Consistency), 추정기 신뢰도(Estimator Confidence), 추론 지연, 통신 지터(Communication Jitter), 배터리 상태(Battery Condition), 비정상 충격(Abnormal Impact)이 포함된다. 로깅은 장애 발생 이후 인지, 추정, 정책, 제어, 하드웨어 계층 전체의 동작을 재구성할 수 있도록 동기화된 정보를 보존해야 한다.

시뮬레이션(Simulation)과 실제 하드웨어 배포(Hardware Deployment)는 가능한 범위에서 호환되는 ROS2 인터페이스를 제공해야 한다. Gazebo, Isaac Sim, MuJoCo 기반 환경이나 다른 물리 플랫폼(Physics Platform)은 실제 로봇과 유사한 인터페이스를 유지하면서 센서, 관절, 접촉, 액추에이터 동작을 모사할 수 있다. 이를 통해 제어기, 보행, 인지, 학습 구성요소가 큰 아키텍처 변경 없이 시뮬레이션에서 소프트웨어 인 더 루프(Software-in-the-Loop), 하드웨어 인 더 루프(Hardware-in-the-Loop) 시험을 거쳐 실제 시스템 배포로 발전할 수 있다.

궁극적으로 ROS2는 모든 결정론적 제어 메커니즘(Deterministic Control Mechanism)을 대체하는 시스템이라기보다 시간 임계적 보행 코어(Time-Critical Locomotion Core)를 둘러싼 통합 및 오케스트레이션 기반(Integration and Orchestration Fabric)으로 활용될 때 가장 효과적이다. 견고한 다족보행 로봇 스택은 실시간 하드웨어 제어(Real-Time Hardware Control), 동기화된 센싱, 접촉 인지 상태 추정(Contact-Aware State Estimation), 보행 및 전신 제어(Whole-Body Control), 인지, 학습 기반 정책, 진단, 생명주기 관리, 안전 감독(Safety Supervision)을 명확한 인터페이스로 결합한다. 이러한 계층형 접근법(Layered Approach)은 ROS2를 임베디드 제어(Embedded Control), 피지컬 AI(Physical AI), 내비게이션, 시스템 수준 지능(System-Level Intelligence)을 연결하는 로봇 미들웨어(Robot Middleware) 계층으로 확장할 수 있게 한다.

## 11.02 Joint State Publish / Subscribe: JointState Usage [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS2 기반 다족보행 로봇(ROS2-Based Legged Robot)에서 관절 상태 통신(Joint-State Communication)은 물리적 액추에이터(Physical Actuator), 센싱 하드웨어(Sensing Hardware), 로봇 모델(Robot Model), 제어기(Controller), 시각화 시스템(Visualization System), 상위 지능(Higher-Level Intelligence)을 연결하는 기본적인 소프트웨어 인터페이스(Software Interface)를 제공한다. 표준 sensor_msgs/msg/JointState 메시지는 관절 이름(Joint Name), 위치(Position), 속도(Velocity), 힘 또는 토크(Effort)의 동기화된 배열을 사용하여 여러 관절의 순간 상태를 표현함으로써 서로 다른 구성요소가 관절 운동(Articulated Motion)을 공통된 방식으로 교환할 수 있도록 한다.

JointState 메시지는 표준 헤더(Header)와 함께 이름(Name), 위치(Position), 속도(Velocity), 힘 또는 토크(Effort) 배열을 포함한다. 헤더의 타임스탬프(Timestamp)는 표현된 관절 측정값이 유효했던 시점을 나타내며, 이름 배열은 각각의 수치 요소를 특정 로봇 관절과 연결한다. 위치 값은 일반적으로 관절 변위(Joint Displacement)를 나타내고, 속도는 그 변화율을 표현하며, 힘 또는 토크는 해당 관절의 물리적 특성에 따라 힘(Force)이나 토크(Torque)를 나타낸다.

배열 간의 정확한 대응 관계(Correspondence)는 매우 중요하다. 구독자(Subscriber)는 동일한 인덱스(Index)의 관절 이름에 따라 위치, 속도, 힘 또는 토크 값을 해석하기 때문이다. 예를 들어 왼쪽 무릎 관절(Left Knee Joint)에 해당하는 값은 채워진 모든 배열에서 일관되게 정렬되어야 한다. 따라서 구현에서는 드라이버(Driver), 컨테이너(Container), 검색 메커니즘(Discovery Mechanism)이 우연히 생성한 순서에 의존하지 않고 관리된 구성(Configuration)이나 하드웨어 매핑(Hardware Mapping)을 기준으로 관절 순서를 결정해야 한다.

타임스탬프(Timestamp)는 관절 측정값이 IMU, 힘 센서(Force Sensor), 촉각 센서(Tactile Sensor), 카메라(Camera), 라이다(LiDAR) 관측값과 자주 융합되는 다족보행 로봇에서 특히 중요하다. 타임스탬프는 단순히 ROS2 발행자(Publisher)가 실행된 순간이 아니라 하드웨어 아키텍처가 허용하는 범위에서 실제 측정 시점(Measurement Time)을 최대한 정확하게 표현해야 한다. 하드웨어 기반 타임스탬프(Hardware-Derived Timestamp)와 동기화된 클록(Synchronized Clock)은 상태 추정(State Estimation), 접촉 감지(Contact Detection), 몸체 움직임 재구성(Body-Motion Reconstruction), 제어(Control)의 시간적 정렬을 향상시킨다.

일반적인 관절 상태 발행자(Joint-State Publisher)는 엔코더(Encoder), 모터 드라이브(Motor Drive), ros2_control 하드웨어 인터페이스(Hardware Interface)로부터 원시 측정값(Raw Measurement)을 획득하고, 하드웨어 단위를 로봇 모델에서 사용하는 규칙으로 변환한 후 JointState 메시지를 구성하여 지정된 토픽(Topic)에 발행한다. 구독자는 robot_state_publisher, 모니터링 도구(Monitoring Tool), 제어기, 추정기(Estimator), 데이터 로거(Data Logger), 진단 노드(Diagnostic Node), 학습 정책 인터페이스(Learning-Policy Interface), 기타 관절 상태가 필요한 응용 구성요소가 될 수 있다.

다족보행 로봇에서는 데이터를 사용하는 기능에 따라 발행 주기(Publication Frequency)를 결정해야 한다. 시각화(Visualization)는 비교적 낮은 갱신 주기(Update Rate)를 허용할 수 있지만, 상태 추정, 보행 분석(Gait Analysis), 접촉 추론(Contact Reasoning), 정책 추론(Policy Inference)은 훨씬 최신의 정보를 요구할 수 있다. 높은 주기로 발행하는 것만으로 유용한 실시간 동작(Real-Time Behavior)이 보장되는 것은 아니며, 종단 간 지연(End-to-End Latency), 스케줄링 지터(Scheduling Jitter), 타임스탬프 정확도, DDS 전송(DDS Transport), 콜백 실행(Callback Execution), 데이터 최신성(Data Freshness)을 함께 고려해야 한다.

ROS2 서비스 품질(Quality of Service, QoS) 설정은 관절 상태의 연속적이고 시간에 민감한 특성을 반영해야 한다. 많은 응용에서는 과거의 모든 샘플을 복구하는 것보다 가장 최신 상태를 수신하는 것이 더 중요하다. 따라서 구독자가 일시적으로 처리 속도를 따라가지 못할 때 오래된 측정값이 누적되지 않도록 큐 깊이(Queue Depth)를 제한할 수 있다. 신뢰성(Reliability) 요구사항은 네트워크 상태(Network Condition), 컴퓨팅 아키텍처(Computing Architecture), 그리고 지연된 상태 전달보다 일부 샘플 손실(Sample Loss)을 허용하는 것이 더 적절한지에 따라 선택해야 한다.

JointState는 표준화된 관측 인터페이스(Standardized Observation Interface)로 매우 유용하지만 모든 내부 고주파 서보 동작(High-Frequency Servo Operation)의 전송 수단으로 자동 적용해서는 안 된다. 수백 Hz 또는 kHz 수준으로 동작하는 토크, 전류, 위치 루프는 결정론적 공유 메모리(Deterministic Shared Memory), 실시간 버퍼(Real-Time Buffer), 필드버스 통신(Fieldbus Communication), 직접 하드웨어 인터페이스(Direct Hardware Interface)가 필요할 수 있다. ROS2 JointState 발행은 최저 수준 제어 루프를 일반적인 미들웨어 경로로 강제하지 않으면서 그 결과 상태를 외부에 제공할 수 있다.

측정 상태(Measured State)와 명령 상태(Commanded State)의 구분은 명확하게 유지되어야 한다. JointState는 일반적으로 실제 또는 시뮬레이션 로봇의 관측값을 전달하며, 목표 위치(Desired Position), 목표 속도(Desired Velocity), 목표 토크(Desired Torque), 궤적(Trajectory), 제어기 기준값(Controller Reference)은 적절한 명령 인터페이스(Command Interface)를 사용해야 한다. 측정량과 명령량을 모호한 토픽 의미(Topic Semantics) 아래 혼합하면 디버깅(Debugging)이 어려워지고 추정기, 모니터 또는 학습 구성요소가 목표 움직임을 실제 물리적 움직임으로 잘못 해석할 수 있다.

ros2_control 아키텍처에서는 하드웨어 구성요소(Hardware Component)가 상태 인터페이스(State Interface)를 노출하고 제어기 및 브로드캐스터(Broadcaster)가 이를 컨트롤러 관리자(Controller Manager)를 통해 사용할 수 있다. 관절 상태 브로드캐스터(Joint State Broadcaster)는 이러한 상태 인터페이스를 표준 JointState 정보로 발행하여 하드웨어별 데이터 획득과 ROS2 수준 구독자 사이에 명확한 경계를 형성한다. 이를 통해 중복된 드라이버 로직(Driver Logic)을 줄이고 제어기, 시각화 도구, 로깅 시스템, 응용 노드가 일관된 관절 측정 표현을 공유할 수 있다.

로봇 모델링(Robot Modeling)과 관절 상태 통신은 밀접하게 연결되어 있다. 런타임 시스템(Runtime System)에서 발행되는 관절 이름은 URDF 및 관련 제어 구성(Control Configuration)에 정의된 관절과 일치해야 한다. robot_state_publisher는 관절 위치와 로봇의 운동학 모델(Kinematic Model)을 결합하여 링크 변환(Link Transform)을 계산하고 tf2를 통해 변환 트리(Transform Tree)를 발행할 수 있다. 잘못된 이름, 누락된 관절, 유효하지 않은 값 또는 일관되지 않은 모델은 시각화, 인지(Perception), 위치 추정(Localization), 운동 계산(Motion Computation)에 연쇄적인 문제를 발생시킬 수 있다.

다족보행 로봇은 많은 관절이 동시에 몸체를 지지하고 이동시키기 때문에 추가적인 복잡성을 가진다. 4족보행 로봇(Quadruped Robot)은 각 다리에 엉덩이(Hip), 허벅지(Thigh), 무릎(Knee) 관절을 포함할 수 있으며, 휴머노이드(Humanoid)는 다리, 몸통(Torso), 팔(Arm), 손(Hand), 목(Neck), 기타 추가 기구를 포함할 수 있다. 따라서 관절 상태 인터페이스는 확장 가능하고 체계적으로 유지되어야 하며, 명명 규칙(Naming Convention)은 특정 로봇 구성에 불필요하게 종속되지 않으면서 각 관절의 물리적 정체성을 명확하게 표현해야 한다.

구독자는 불완전하거나 지연되거나 비정상적인 데이터도 방어적으로 처리해야 한다. 견고한 구성요소는 예상되는 관절 이름을 검증하고, 배열 크기(Array Dimension)를 확인하며, NaN이나 유효하지 않은 수치 값을 감지하고, 타임스탬프의 경과 시간(Timestamp Age)을 감시하며, 갱신이 중단된 관절을 식별할 수 있어야 한다. 이러한 검사는 균형 제어(Balance Control)와 보행 시스템에서 특히 중요하다. 오래된 측정값은 수치적으로 정상처럼 보이면서도 현재 로봇의 실제 기계적 구성(Mechanical Configuration)을 더 이상 나타내지 않을 수 있기 때문이다.

JointState 데이터는 일반적인 운동 재구성(Motion Reconstruction)을 넘어 진단(Diagnostics)에도 활용할 수 있다. 명령 위치와 측정 위치의 차이는 추종 오차(Tracking Error)를 나타낼 수 있으며, 속도와 힘 또는 토크의 변화 추세는 비정상적인 저항(Abnormal Resistance), 충격(Impact), 액추에이터 포화(Actuator Saturation), 기계적 열화(Mechanical Degradation)를 나타낼 수 있다. 모터 온도, 전류(Current), 고장 코드(Fault Code), 접촉 정보와 결합하면 관절 상태 모니터링은 단순한 시각화 데이터 소스를 넘어 광범위한 상태 관리 아키텍처(Health-Management Architecture)의 일부가 된다.

학습 기반 보행(Learning-Based Locomotion) 역시 신뢰할 수 있는 관절 관측(Joint Observation)에 크게 의존한다. 강화학습 정책(Reinforcement-Learning Policy)은 일반적으로 관절 위치와 속도를 IMU 측정값, 이전 행동(Previous Action), 접촉 상태(Contact State), 명령 정보(Command Information)와 함께 관측 벡터(Observation Vector)의 일부로 사용한다. ROS2 적응 노드(Adaptation Node)는 JointState 데이터를 추론 모델(Inference Model)이 요구하는 순서(Ordering), 정규화(Normalization), 스케일링(Scaling), 텐서 표현(Tensor Representation)으로 변환하면서 타임스탬프를 보존하고 관측값의 완전성(Observation Completeness)을 검증할 수 있다.

시뮬레이션(Simulation)은 실제 플랫폼에서 기대되는 것과 동일한 관절 상태 의미(Joint-State Semantics)를 재현해야 한다. Gazebo, Isaac Sim, MuJoCo 기반 환경 또는 사용자 정의 시뮬레이터(Custom Simulator)는 실제 로봇과 호환되는 인터페이스를 통해 시뮬레이션된 관절 측정값을 발행할 수 있다. 일관된 이름, 단위(Unit), 좌표 규칙(Coordinate Convention), 발행 동작(Publication Behavior), 제어기 인터페이스를 유지하면 알고리즘을 시뮬레이션에서 실제 하드웨어로 이전할 때 필요한 변경을 줄이고 소프트웨어 인 더 루프(Software-in-the-Loop)와 하드웨어 인 더 루프(Hardware-in-the-Loop) 검증을 체계적으로 수행할 수 있다.

멀티스레드 실행(Multi-Threaded Execution)에서는 JointState 구독자가 갱신한 정보를 제어기, 추정기 또는 추론 콜백(Inference Callback)이 동시에 사용할 수 있기 때문에 추가적인 주의가 필요하다. 공유 데이터(Shared Data)는 중요한 실행 경로를 불필요하게 차단하지 않으면서 적절한 동기화(Synchronization) 또는 실시간 안전 데이터 교환 방식(Real-Time-Safe Exchange Pattern)으로 보호해야 한다. 시간 민감 시스템에서는 사전 할당된 버퍼(Preallocated Buffer)와 최신 샘플 교환 메커니즘(Latest-Sample Exchange Mechanism)을 사용하여 오래된 메시지를 유지하지 않으면서 예측 가능한 동작을 제공할 수 있다.

관절 상태 로깅(Joint-State Logging)은 보행 동작을 사후 분석(Post-Run Analysis)하는 데 필수적이다. JointState를 IMU, 접촉 정보, 제어 명령(Control Command), 변환(Transform), 진단 정보, 인지 출력(Perception Output), 정책 행동(Policy Action)과 함께 기록하면 미끄러짐(Slipping), 불안정한 접촉 전환(Unstable Contact Transition), 예상하지 못한 충격(Unexpected Impact), 넘어짐(Fall)과 같은 사건을 재구성할 수 있다. 정확한 타임스탬프를 사용하면 관찰된 장애가 센싱, 추정, 추론, 통신, 제어 또는 기계적 반응(Mechanical Response) 중 어느 단계에서 시작되었는지 분석할 수 있다.

따라서 가장 효과적인 JointState 아키텍처는 이 메시지를 관절형 로봇 상태(Articulated Robot State)를 위한 표준화된 경계(Standardized Boundary)로 사용하면서 필요한 경우 그 하위 계층에 결정론적 메커니즘(Deterministic Mechanism)을 유지하는 것이다. 일관된 명명, 정확한 타임스탬프, 제어된 발행 주기, 적절한 QoS, 검증된 구독자, ros2_control 통합, 모델 일관성(Model Consistency), 동기화된 로깅(Synchronized Logging)을 통해 JointState 통신은 하드웨어 제어(Hardware Control)를 상태 추정, 보행 제어(Gait Control), 시각화, 진단, 시뮬레이션 및 피지컬 AI(Physical AI) 응용과 효과적으로 연결할 수 있다.

## 11.03 Robot URDF / MJCF Modeling and ROS2 Integration [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 모델링(Robot Modeling)은 기계 설계(Mechanical Design), 제어(Control), 시뮬레이션(Simulation), 시각화(Visualization), ROS2 소프트웨어 통합(ROS2 Software Integration)을 연결하는 구조적 기반을 제공한다. 다족보행 로봇(Legged Robot)의 모델은 링크(Link)와 관절(Joint)의 형상뿐만 아니라 관성 특성(Inertial Properties), 관절 제약조건(Joint Constraints), 액추에이터 관계(Actuator Relationships), 충돌 형상(Collision Geometry), 센서(Sensor), 좌표 프레임(Coordinate Frames)까지 표현해야 한다. URDF와 MJCF는 이 문제의 일부 영역을 공통으로 다루면서도 로보틱스 소프트웨어 생태계(Robotics Software Ecosystem)에서 서로 다른 역할을 수행한다.

통합 로봇 기술 형식(Unified Robot Description Format, URDF)은 ROS와 ROS2에서 널리 사용되는 표준 로봇 기술 메커니즘(Robot-Description Mechanism)이다. URDF 모델은 관절로 연결된 링크들의 트리(Tree) 구조로 로봇을 표현한다. 각 링크에는 시각 형상(Visual Geometry), 충돌 형상, 질량(Mass), 질량 중심(Center of Mass), 관성(Inertia) 정보를 포함할 수 있으며, 각 관절에는 부모 링크(Parent Link)와 자식 링크(Child Link), 원점(Origin), 축(Axis), 운동 유형(Motion Type), 한계(Limits), 동역학(Dynamics) 및 ROS2 구성요소에 필요한 기타 특성을 정의할 수 있다.

다족보행 로봇의 URDF 계층구조(Hierarchy)는 일반적으로 베이스(Base) 또는 몸통 링크(Torso Link)에서 시작하여 여러 운동학 체인(Kinematic Chain)으로 확장된다. 4족보행 로봇(Quadruped Robot)은 네 개의 다리에 엉덩이(Hip), 허벅지(Thigh), 종아리(Calf), 발(Foot) 링크를 정의할 수 있으며, 휴머노이드(Humanoid)는 골반(Pelvis), 몸통(Torso), 목(Neck), 팔(Arm), 손(Hand), 관절형 발(Articulated Foot)을 추가로 포함할 수 있다. 제어기, JointState 메시지, tf2 변환(Transform), 시각화 및 설정 파일(Configuration File)이 이러한 식별자를 직접 참조하는 경우가 많기 때문에 일관된 명명과 계층구조가 중요하다.

시각 표현(Visual Representation)과 충돌 표현(Collision Representation)은 서로 관련되어 있지만 구별되는 모델 구성요소로 다루어야 한다. 세부적인 메시(Detailed Mesh)는 RViz2 또는 시뮬레이션에서 사실적인 시각화를 제공할 수 있는 반면, 단순화된 충돌 형상(Simplified Collision Shape)은 접촉 감지(Contact Detection), 경로 계획(Planning), 물리 계산(Physics Calculation)의 연산 비용을 줄일 수 있다. 두 표현을 분리하면 모든 충돌 및 동역학 연산에서 불필요하게 복잡한 형상 메시를 처리하지 않고도 정확한 시각적 외형을 유지할 수 있다.

정확한 관성 파라미터(Inertial Parameters)는 균형(Balance), 접촉력(Contact Force), 전신 제어(Whole-Body Control), 동역학 시뮬레이션(Dynamic Simulation)이 로봇 전체의 질량 분포(Mass Distribution)에 의존하기 때문에 다족보행에서 특히 중요하다. 따라서 각 링크에는 적절한 질량, 질량 중심 위치(Center-of-Mass Location), 관성 텐서(Inertia Tensor) 정보가 포함되어야 한다. 잘못된 관성 값은 기하학적으로는 정확해 보이지만 시뮬레이션에서 비현실적으로 동작하거나 모델 기반 제어기(Model-Based Controller)에서 부정확한 결과를 생성하는 모델을 만들 수 있다.

관절 정의(Joint Definition)는 로봇의 자유도(Degrees of Freedom)와 운동학적 관계(Kinematic Relationships)를 결정한다. 회전 관절(Revolute Joint), 연속 관절(Continuous Joint), 직동 관절(Prismatic Joint), 고정 관절(Fixed Joint) 등의 지원되는 관절 유형은 인접한 링크가 서로 어떻게 움직일 수 있는지를 정의한다. 구동되는 다리 관절에서는 위치, 속도, 힘 또는 토크(Effort)의 제한값이 실제 기구와 액추에이터 성능에 대응해야 한다. 작은 모델링 오류도 순기구학(Forward Kinematics)과 역기구학(Inverse Kinematics) 전체에 전파될 수 있으므로 관절 축과 원점도 일관된 좌표 규칙(Coordinate Convention)을 따라야 한다.

URDF는 ROS2 변환 생태계(Transform Ecosystem)와 밀접하게 통합된다. robot_state_publisher는 로봇 기술 정보(Robot Description)와 측정된 관절 위치를 결합하여 개별 링크의 자세(Pose)를 계산하고 tf2를 통해 좌표 관계를 발행한다. 고정 관절은 정적 변환(Static Transform)으로 발행할 수 있으며 움직이는 관절은 현재 관절 상태에 따라 지속적으로 갱신된다. 이를 통해 인지(Perception), 시각화, 위치 추정(Localization), 계획, 제어 구성요소가 공유하는 공통 공간 기준(Common Spatial Reference)이 형성된다.

robot_description 파라미터(Parameter)는 일반적으로 로봇 모델 정보가 필요한 ROS2 노드에 URDF 표현을 제공한다. 각각의 구성요소 내부에서 별도의 모델 기술을 관리하는 대신 통제된 모델 소스(Controlled Model Source)를 사용하면 시각화, 제어기, 플래너(Planner), 기타 소프트웨어가 일관된 형상과 관절 정의를 사용하도록 할 수 있다. 특히 네 개의 유사한 다리와 같이 반복적인 구조를 포함하는 로봇에서는 URDF를 체계적으로 생성하기 위해 Xacro가 자주 사용된다.

Xacro 기반 모델링(Xacro-Based Modeling)은 매크로(Macro), 파라미터, 수식(Expression), 재사용 가능한 구성요소(Reusable Component)를 사용하여 반복되는 로봇 구조를 생성함으로써 유지보수성(Maintainability)을 향상시킨다. 예를 들어 하나의 다리 정의를 한 번 작성한 후 적절한 원점과 명명 접두어(Naming Prefix)를 적용하여 전방 좌측(Front-Left), 전방 우측(Front-Right), 후방 좌측(Rear-Left), 후방 우측(Rear-Right) 다리로 생성할 수 있다. 이를 통해 중복된 XML을 줄이고 기계 설계 변경 사항을 전체 로봇 모델에 일관되게 반영하기 쉬워진다.

ros2_control은 로봇 기술 정보와 실제 구동(Physical Actuation) 사이에 또 하나의 중요한 통합 계층(Integration Layer)을 제공한다. 제어 관련 기술 정보(Control-Related Description)는 관절을 위치, 속도, 힘 또는 토크와 같은 명령 인터페이스(Command Interface) 및 상태 인터페이스(State Interface)와 연결할 수 있다. 하드웨어 플러그인(Hardware Plugin)은 이러한 추상 인터페이스(Abstract Interface)를 모터 드라이브(Motor Drive), 필드버스 장치(Fieldbus Device), 시뮬레이션 시스템 또는 사용자 정의 전자장치(Custom Electronics)에 연결한다. 이에 따라 제어기는 특정 액추에이터 구현에 대한 종속성을 줄이면서 표준화된 인터페이스를 통해 동작할 수 있다.

MuJoCo 모델링 XML 형식(MuJoCo Modeling XML Format, MJCF)은 물리 시뮬레이션(Physics Simulation), 접촉 중심 동역학(Contact-Rich Dynamics), 학습 기반 로보틱스(Learning-Based Robotics)에 특히 유용하다. MJCF는 MuJoCo 물리 엔진(Physics Engine)에 밀접하게 대응하는 형식으로 관절형 몸체(Articulated Body), 관절, 관성 특성, 액추에이터, 센서, 제약조건(Constraint), 접촉(Contact), 시뮬레이션 전용 파라미터를 표현할 수 있다. 이러한 특성으로 인해 MJCF는 다족보행 연구, 강화학습(Reinforcement Learning), 동적 행동 개발(Dynamic Behavior Development), 대규모 시뮬레이션 실험(Large-Scale Simulation Experiment)에 유용하다.

URDF와 MJCF를 반드시 경쟁적인 형식으로 볼 필요는 없다. 실제 ROS2 다족보행 로봇 워크플로(Workflow)에서 URDF는 robot_state_publisher, tf2, RViz2, ros2_control 및 ROS2 패키지(Package)를 위한 주요 통합 표현(Integration Representation)으로 사용될 수 있으며, MJCF는 고정밀 동역학(High-Fidelity Dynamics)과 학습 환경을 지원할 수 있다. 핵심적인 엔지니어링 과제는 두 표현 사이에서 관절 이름, 좌표 규칙, 형상, 질량 특성, 제한값, 액추에이터 가정(Actuator Assumption)을 일관되게 유지하는 것이다.

두 개의 로봇 기술 정보를 독립적으로 관리하면 모델 드리프트(Model Drift)가 발생할 수 있다. 링크 크기, 관절 위치, 질량 분포, 액추에이터 제한 등의 기계적 변경 사항이 한 표현에는 적용되지만 다른 표현에서는 누락될 수 있기 때문이다. 따라서 견고한 개발 프로세스(Development Process)는 권위 있는 모델 파라미터(Authoritative Model Parameters)를 정의하고 가능한 경우 통제된 소스로부터 하위 표현을 생성하거나 검증해야 한다. 자동화된 일관성 검사(Automated Consistency Check)를 통해 이러한 차이가 시뮬레이션이나 실제 하드웨어 실험에 영향을 주기 전에 발견할 수 있다.

모델 형식 간 변환이나 동기화 과정에서는 좌표 규칙에 특별한 주의가 필요하다. 몸체 방향(Body Orientation), 관절 축 정의(Joint-Axis Definition), 기준 프레임(Reference Frame), 쿼터니언 규칙(Quaternion Convention), 베이스 프레임 가정(Base-Frame Assumption)의 차이는 시각적으로 발견하기 어려운 오류를 발생시킬 수 있다. 따라서 모델 검증(Model Validation)은 단순히 렌더링된 로봇이 정상적으로 보이는지만 확인해서는 안 되며, 알려진 관절 구성(Known Joint Configuration), 변환 비교(Transform Comparison), 질량 중심 검사, 순기구학 검증, 통제된 운동 시험(Controlled Motion Test)을 포함해야 한다.

시뮬레이션 통합(Simulation Integration)은 단순히 형상 데이터를 물리 엔진에 로딩하는 것 이상의 작업을 요구한다. 액추에이터 동작(Actuator Behavior), 관절 감쇠(Joint Damping), 마찰(Friction), 접촉 파라미터(Contact Parameter), 센서 노이즈(Sensor Noise), 지연(Latency), 제어 주기(Control Frequency), 지면 상호작용(Ground Interaction)은 시뮬레이션된 보행이 실제 플랫폼과 얼마나 유사한지에 영향을 준다. 강화학습 응용에서는 비현실적인 동역학이 시뮬레이션에서는 우수하지만 실제 배포에서 실패하는 정책(Policy)을 만들 수 있으므로 모델 보정(Model Calibration)과 도메인 랜덤화(Domain Randomization)는 시뮬레이션-현실 전이(Sim-to-Real) 파이프라인의 중요한 요소가 된다.

ROS2는 시뮬레이션과 실제 하드웨어 사이에 동일한 관절 상태(Joint State), 명령(Command), 센서, 변환, 제어기 인터페이스를 제공함으로써 안정적인 소프트웨어 경계(Software Boundary)를 형성할 수 있다. 이러한 인터페이스를 사용하는 알고리즘은 기반 구현이 Gazebo, Isaac Sim, MuJoCo 또는 다른 시뮬레이터에서 실제 로봇으로 변경되더라도 최소한의 아키텍처 수정만 필요하도록 설계되어야 한다. 이러한 인터페이스 일관성(Interface Consistency)은 소프트웨어 인 더 루프(Software-in-the-Loop), 하드웨어 인 더 루프(Hardware-in-the-Loop), 제어기 검증(Controller Validation), 단계적 배포(Progressive Deployment)를 지원한다.

모델 검증은 복잡한 보행 알고리즘을 도입하기 전에 수행해야 한다. 관절 이름과 방향, 링크 변환, 운동 범위(Motion Range), 충돌 형상, 관성 파라미터, 제어기 인터페이스, 센서 프레임(Sensor Frame)을 체계적으로 검증해야 한다. RViz2는 구조 및 변환 오류를 확인하는 데 사용할 수 있으며, 물리 시뮬레이션은 불안정한 관성 또는 접촉 구성을 발견하는 데 도움이 된다. 자동화 시험(Automated Test)은 지속적 통합(Continuous Integration) 과정에서 예상 자세, 변환, 관절 제한, 모델 메타데이터(Model Metadata)를 추가로 비교할 수 있다.

로봇 기술 정보는 사실상 기계 엔지니어링(Mechanical Engineering)과 소프트웨어 사이의 인터페이스 계약(Interface Contract)이 되므로 버전 관리(Version Management) 역시 중요하다. 모델 개정(Model Revision)은 하드웨어 구성(Hardware Configuration), 액추에이터 버전, 센서 배치(Sensor Placement), 보정 데이터(Calibration Data), 제어기 파라미터, 시뮬레이션 자산(Simulation Asset)과 연계되어야 한다. 따라서 소프트웨어 릴리스(Software Release)는 배포된 ROS2 노드가 오래된 기계적 가정(Mechanical Assumption)을 기반으로 동작하지 않도록 호환되는 로봇 모델 버전을 명확하게 식별해야 한다.

피지컬 AI(Physical AI)와 학습 기반 시스템(Learning-Based System)에서 로봇 모델은 전통적인 시각화와 제어를 넘어 구조화된 정보(Structured Information)를 제공할 수도 있다. 관절 토폴로지(Joint Topology), 제한값, 운동학적 관계, 접촉 위치(Contact Location), 몸체 형상(Body Geometry)은 관측값 구성(Observation Construction), 정책 적응(Policy Adaptation), 모션 리타게팅(Motion Retargeting), 시뮬레이션 생성(Simulation Generation), 로봇 간 학습(Cross-Robot Learning)을 지원할 수 있다. 따라서 ROS2 시스템이 전통적인 로보틱스 알고리즘과 학습 정책 및 파운데이션 모델 기반 지능(Foundation-Model-Based Intelligence)을 통합할수록 잘 정의된 모델 계층(Model Layer)의 중요성은 더욱 커진다.

효과적인 다족보행 로봇 모델링 아키텍처는 URDF, MJCF, ROS2 인터페이스, 제어 구성(Control Configuration), 실제 하드웨어를 동일한 체화 시스템(Embodied System)의 동기화된 표현(Synchronized Representation)으로 다룬다. URDF는 ROS2 통합과 운동학 구조(Kinematic Structure)의 중심 역할을 하고, MJCF는 접촉 중심 시뮬레이션과 학습을 지원하며, ros2_control은 모델의 관절을 실제 구동과 연결하고, tf2는 공간 관계(Spatial Relationship)를 배포한다. 일관된 파라미터, 자동화된 검증, 통제된 버전 관리, 시뮬레이션과 하드웨어 간 동등한 인터페이스를 구축함으로써 로봇 모델은 단순한 정적 기술 파일(Static Description File)이 아니라 전체 시스템을 연결하는 핵심 시스템 수준 계약(System-Level Contract)으로 발전한다.

## 11.04 tf2 Transform Tree Design for Multi-Legged Robot [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

tf2 변환 트리(tf2 Transform Tree)는 거의 모든 인지(Perception), 상태 추정(State Estimation), 제어(Control), 내비게이션(Navigation) 기능이 좌표 프레임(Coordinate Frame) 간의 일관된 관계에 의존하기 때문에 ROS2 기반 다족보행 로봇(ROS2-Based Multi-Legged Robot)의 핵심 공간 인프라(Spatial Infrastructure)이다. 단순한 이동 베이스(Mobile Base)와 달리 다족보행 로봇은 부동 몸체(Floating Body)와 여러 개의 관절형 운동학 체인(Articulated Kinematic Chain)을 가지며, 관절이 움직이고 발이 환경과 접촉하거나 접촉을 해제함에 따라 이들의 변환(Transform)은 지속적으로 변화한다.

잘 설계된 변환 트리(Transform Tree)는 전역(Global), 지역(Local), 몸체(Body), 사지(Limb), 센서(Sensor) 프레임의 명확한 계층구조(Hierarchy)에서 시작한다. 대표적인 전역 프레임에는 map과 odom이 있으며, base_link는 주요 몸체 기준(Body Reference)을 나타낸다. base_link에서 엉덩이(Hip), 허벅지(Thigh), 무릎(Knee), 발목(Ankle), 발(Foot) 프레임으로 이어지는 별도의 운동학 분기(Kinematic Branch)가 확장된다. 카메라(Camera), 라이다(LiDAR), IMU, 매니퓰레이터(Manipulator), 기타 페이로드 센서(Payload Sensor)는 각각 보정된 변환(Calibrated Transform)을 통해 적절한 몸체 링크에 연결된다.

map, odom, base_link의 관계는 명확하게 정의된 위치 추정 의미(Localization Semantics)를 따라야 한다. map 프레임은 일반적으로 내비게이션에 사용되는 전역적으로 일관된 기준(Global Consistent Reference)을 나타내며, odom은 단기 움직임 추정에 적합한 지역적으로 연속적인 기준(Locally Continuous Reference)을 제공한다. odom에서 base_link로의 변환은 지역적으로 추정된 로봇 움직임을 나타내며, map에서 odom으로의 변환은 SLAM, 위성항법시스템(GNSS), 비주얼 위치 추정(Visual Localization) 또는 기타 전역 보정(Global Correction)을 사용하여 누적된 드리프트(Accumulated Drift)를 보상할 수 있다.

다족보행 로봇에서 base_link는 단순히 시각화 편의를 위해 선택되는 것이 아니라 안정적이고 명확하게 문서화된 몸체 기준(Body Reference)을 나타내야 한다. 플랫폼에 따라 base, pelvis, torso, base_footprint, base_stabilized와 같은 추가 프레임이 도입될 수 있다. 제어기(Controller), 상태 추정기(Estimator), 플래너(Planner), 인지 파이프라인(Perception Pipeline), 학습 정책(Learning Policy)이 프레임 의미가 일관되지 않을 경우 몸체 위치와 자세(Orientation)를 서로 다르게 해석할 수 있으므로 각 프레임의 의미는 명확하게 유지되어야 한다.

각 다리(Leg)는 로봇 모델(Robot Model)과 현재 관절 구성(Joint Configuration)으로부터 계산되는 관절형 변환 체인(Articulated Transform Chain)을 형성한다. 4족보행 로봇(Quadruped Robot)은 전방 좌측(Front-Left), 전방 우측(Front-Right), 후방 좌측(Rear-Left), 후방 우측(Rear-Right) 분기를 포함할 수 있으며, 각각의 분기는 몸체에서 엉덩이와 무릎 관절을 거쳐 발 프레임까지 이어진다. 휴머노이드 로봇(Humanoid Robot)은 동일한 개념을 두 개의 관절형 다리에 적용하면서 몸통(Torso), 머리(Head), 팔(Arm), 손(Hand), 추가 말단장치 프레임(End-Effector Frame)까지 포함한다.

링크 수가 증가할수록 일관된 프레임 명명(Frame Naming)이 더욱 중요해진다. 이름은 모호한 약어에 의존하지 않으면서 물리적 구성요소와 해당 위치를 모두 식별할 수 있어야 한다. 예를 들어 체계적인 접두어(Prefix) 또는 접미어(Suffix)를 사용하여 좌측과 우측, 전방과 후방의 사지를 구분할 수 있다. 불필요한 변환 계층(Translation Layer)이 증가하는 것을 방지하기 위해 이러한 명명 체계는 URDF 관절 및 링크 이름, 제어기 구성, 센서 정의, 로깅 도구(Logging Tool), 시뮬레이션 모델과 일치해야 한다.

robot_state_publisher는 로봇 모델과 동적 tf2 트리(Dynamic tf2 Tree)를 연결하는 주요 연결부 역할을 한다. 이 구성요소는 관절 위치(Joint Position)를 수신하고 URDF에 정의된 운동학적 관계(Kinematic Relationship)를 해석하여 링크 자세(Link Pose)를 계산한 후 해당 변환을 발행한다. 강체로 장착된 구성요소 사이의 고정 관계는 정적 변환(Static Transform)으로 배포할 수 있으며, 관절형 관계는 실제 로봇이나 시뮬레이터가 제공하는 JointState 정보에 따라 지속적으로 변화한다.

정적 변환(Static Transform)과 동적 변환(Dynamic Transform)은 물리적 의미에 따라 구분해야 한다. 몸통에 강체로 장착된 카메라는 일반적으로 기계 설계와 보정(Calibration)을 통해 결정되는 정적 변환을 가지지만, 여러 개의 구동 관절을 통해 연결된 발은 동적 변환을 가진다. 물리적으로 변화하는 관계를 정적 변환으로 발행하면 잘못된 기하학적 관계가 생성될 수 있으며, 반대로 일정한 변환을 불필요하게 동적으로 발행하면 유용한 정보를 추가하지 않으면서 통신 및 연산 부하만 증가시킨다.

시간 일관성(Time Consistency)은 다족보행 로봇의 tf2 아키텍처에서 가장 중요한 요구사항 중 하나이다. 변환은 특정 시점의 공간적 관계(Spatial Relationship)를 나타내며, 몸체나 사지가 빠르게 움직이는 경우 비교적 작은 타임스탬프(Timestamp) 오차도 큰 영향을 줄 수 있다. 따라서 관절 상태(Joint State), IMU 측정값, 인지 데이터, 변환은 동기화된 시간 기준(Synchronized Time Reference)을 사용해야 하며, 이를 통해 하위 알고리즘이 실제 센서 측정 시점에 해당하는 로봇 구성을 재구성할 수 있어야 한다.

tf2는 시간 버퍼 기반 변환 이력(Time-Buffered Transform History)을 유지하므로 소프트웨어는 특정 타임스탬프에 해당하는 공간적 관계를 요청할 수 있다. 이는 카메라 이미지(Camera Image), 라이다 스캔(LiDAR Scan), 관절 측정값, 상태 추정값이 서로 다른 통신 및 처리 지연(Processing Delay)을 가지고 도착할 때 특히 유용하다. 인지 소프트웨어는 오래된 센서 관측에 단순히 최신 변환을 적용하는 대신 해당 관측의 타임스탬프에 대응하는 변환을 조회함으로써 움직임으로 인한 공간 오차(Motion-Induced Spatial Error)를 줄일 수 있다.

변환 가용성(Transform Availability) 또한 런타임 의존성(Runtime Dependency)으로 다루어야 한다. 인지 또는 제어 노드는 시스템 시작 직후 필요한 모든 프레임을 즉시 사용할 수 있다고 가정해서는 안 된다. 하드웨어 초기화(Hardware Initialization), JointState 발행, 위치 추정, 생명주기 전환(Lifecycle Transition)은 서로 다른 시점에 발생할 수 있다. 따라서 노드는 중요한 실행 경로를 무기한 차단하거나 통제되지 않은 명령을 생성하지 않으면서 누락된 변환, 조회 타임아웃(Lookup Timeout), 지연된 초기화, 일시적인 프레임 불연속(Frame Discontinuity)을 처리해야 한다.

발 프레임(Foot Frame)은 환경과의 접촉이 일시적인 물리적 제약조건(Physical Constraint)을 형성하기 때문에 특별한 의미를 가진다. 입각기(Stance) 동안에는 몸체가 움직이는 동안 발이 지면에 대해 거의 정지된 상태를 유지할 수 있으며, 유각기(Swing) 동안에는 동일한 발이 공간에서 빠르게 이동한다. 상태 추정기와 보행 제어기(Locomotion Controller)는 이러한 관계를 활용할 수 있지만, tf2 자체는 발이 현재 로봇을 지지하고 있는지에 대한 가정을 포함하기보다 공간 변환을 표현하는 역할에 집중해야 한다.

따라서 접촉 상태(Contact State)는 전용 메시지(Dedicated Message) 또는 상태 인터페이스(State Interface)를 통해 전달하고 발 변환과 함께 해석해야 한다. 제어기는 발 프레임 자세(Foot Frame Pose)를 힘 센싱(Force Sensing) 또는 접촉 추정(Contact Estimation)과 결합하여 활성 지지 형상(Active Support Geometry)을 결정할 수 있다. 이러한 분리를 통해 좌표 표현(Coordinate Representation)을 접촉 의미(Contact Semantics)와 독립적으로 유지할 수 있으며, 기본적인 변환 계층구조를 변경하지 않고도 다양한 접촉 추정 방법을 발전시킬 수 있다.

센서 프레임(Sensor Frame)은 실제 장착 위치와 보정 상태를 정확하게 반영해야 한다. IMU 프레임은 관성 센서(Inertial Sensor)의 방향과 위치를 로봇 몸체에 대해 표현해야 하며, 카메라와 라이다 프레임은 각각의 ROS2 인터페이스에서 요구하는 좌표 규칙을 따라야 한다. 이러한 변환의 보정 오류(Calibration Error)는 하위 단계에서 위치 추정 드리프트(Localization Drift), 왜곡된 포인트 클라우드(Distorted Point Cloud), 잘못된 지형 형상(Terrain Geometry), 발 배치 계획(Foot-Placement Planning)의 체계적인 오차로 나타날 수 있다.

고주파 변환 발행(High-Frequency Transform Publication)은 신중한 성능 설계(Performance Design)를 요구한다. 모든 변환을 불필요하게 높은 주기로 발행하면 특히 많은 관절을 가진 휴머노이드에서 CPU 사용률, 직렬화 오버헤드(Serialization Overhead), DDS 트래픽(Traffic)이 증가할 수 있다. 따라서 갱신 주기는 실제 움직임의 동역학과 데이터 소비자의 요구사항에 맞추어야 한다. 프로세스 내부 통신(Intra-Process Communication), 컴포지션(Composition), 실행기 구성(Executor Configuration), 효율적인 JointState 처리를 통해 복잡한 변환 트리의 지연과 연산 부하를 추가로 줄일 수 있다.

tf2 트리는 서로 경쟁하는 변환 권한(Transform Authority)의 집합이 아니라 하나의 트리 구조를 유지해야 한다. 여러 노드가 동일한 부모-자식 관계(Parent-Child Relationship)에 대해 서로 다른 변환을 독립적으로 발행해서는 안 된다. 이러한 충돌은 불안정하거나 예측하기 어려운 공간 결과를 만들 수 있기 때문이다. 각 변환에 대한 책임은 robot_state_publisher, 위치 추정 시스템(Localization System), 상태 추정기, 정적 변환 발행자(Static Transform Publisher), 센서 드라이버(Sensor Driver), 기타 구성요소 사이에서 명확하게 할당해야 한다.

시뮬레이션(Simulation)과 실제 하드웨어(Physical Hardware)는 가능한 범위에서 동일한 기본 프레임 계층구조를 유지해야 한다. Gazebo, Isaac Sim, MuJoCo 기반 환경과 하드웨어 인터페이스는 서로 호환되는 베이스(Base), 관절(Joint), 발(Foot), 센서 프레임을 제공해야 한다. 이러한 대응 관계를 유지하면 인지, 제어, 시각화, 내비게이션, 학습 소프트웨어가 공간 구조에 대한 서로 다른 가정을 사용하지 않고 시뮬레이션과 실제 로봇 사이에서 이동할 수 있다.

변환 검증(Transform Validation)은 개발 및 시험 과정에 통합되어야 한다. RViz2를 사용하면 뒤집힌 축(Reversed Axis), 잘못 배치된 센서(Misplaced Sensor), 잘못된 관절 원점(Joint Origin), 예상하지 못한 프레임 움직임을 시각적으로 확인할 수 있으며, tf2 진단 도구(Diagnostic Tool)를 통해 누락된 프레임, 과도한 지연, 연결되지 않은 분기(Disconnected Branch)를 식별할 수 있다. 자동화 시험(Automated Test)은 예상되는 부모-자식 관계, 알려진 변환, 프레임 명명 규칙, 발행 타이밍(Publication Timing), 해당 URDF 모델과의 일관성을 검증할 수 있다.

학습 기반 보행(Learning-Based Locomotion)과 피지컬 AI(Physical AI)에서는 관측값이 추론 모델(Inference Model)에 입력되기 전에 월드(World), 몸체(Body), 센서(Sensor), 지역 지형(Local Terrain) 좌표 사이에서 변환될 수 있기 때문에 프레임 일관성이 더욱 중요하다. 하나의 프레임 규칙으로 학습된 정책(Policy)은 다른 규칙으로 배포될 경우 잘못 동작할 수 있다. 따라서 명시적인 프레임 정의와 통제된 변환 파이프라인(Controlled Transformation Pipeline)은 ROS2 인지, 전통적 제어(Classical Control), 시뮬레이션, 학습 기반 지능(Learned Intelligence)을 연결하는 인터페이스 계약(Interface Contract)의 일부가 된다.

견고한 다족보행 로봇 tf2 아키텍처는 궁극적으로 전체 소프트웨어 스택이 공유하는 하나의 일관된 공간 표현(Coherent Spatial Representation)을 제공한다. 전역 위치 추정(Global Localization)은 map과 odom의 관계를 설정하고, 상태 추정은 몸체 움직임을 결정하며, URDF와 JointState는 관절형 링크 변환을 정의하고, 보정된 정적 변환은 센서 위치를 결정하며, 발 프레임은 사지 형상(Limb Geometry)을 표현한다. 명확한 소유권(Ownership), 동기화된 타임스탬프, 체계적인 명명, 검증, 시뮬레이션-하드웨어 일관성(Simulation-to-Hardware Consistency)을 확보함으로써 이 변환 트리는 보행, 인지, 내비게이션, 전신 제어(Whole-Body Control), 피지컬 AI를 위한 신뢰성 높은 공간적 백본(Spatial Backbone)이 된다.

## 11.05 Foot Contact Detection Message Design [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

발 접촉 감지(Foot Contact Detection)는 발이 환경과 접촉을 형성하거나 유지하고, 미끄러지거나 접촉을 해제할 때마다 보행 동역학(Locomotion Dynamics)이 변화하기 때문에 다족보행 로보틱스(Legged Robotics)의 핵심 기능이다. ROS2 아키텍처에서는 원시 센싱(Raw Sensing)과 추정된 접촉 상태(Estimated Contact State)를 분리하는 명확한 인터페이스를 통해 접촉 정보를 표현해야 하며, 이를 통해 보행 제어(Gait Control), 상태 추정(State Estimation), 전신 제어(Whole-Body Control), 진단(Diagnostics), 학습 기반 정책(Learning-Based Policy)이 일관된 접촉 정보를 사용할 수 있어야 한다.

접촉 감지(Contact Detection)는 로봇 하드웨어에 따라 여러 종류의 센싱 메커니즘(Sensing Mechanism)으로부터 수행할 수 있다. 발 힘 센서(Foot Force Sensor), 6축 힘/토크 센서(Six-Axis Force/Torque Sensor), 촉각 배열(Tactile Array), 관절 토크 측정(Joint Torque Measurement), 모터 전류(Motor Current), 액추에이터 부하 추정(Actuator Load Estimation), 운동학적 관측(Kinematic Observation)은 모두 지면 상호작용(Ground Interaction)의 증거를 제공할 수 있다. IMU 측정값은 충격이나 예상하지 못한 몸체 가속도를 감지하여 추가 정보를 제공하며, 전용 발 센서가 없는 경우에는 모델 기반 추정기(Model-Based Estimator)를 통해 접촉을 추론할 수도 있다.

원시 센서 측정값(Raw Sensor Measurement)은 일반적으로 해석된 접촉 상태(Interpreted Contact State)와 분리하여 유지해야 한다. 힘 센서는 측정된 힘을 지속적으로 발행할 수 있으며, 접촉 추정 노드(Contact-Estimation Node)는 이러한 측정값이 안정적인 지지(Stable Support), 불확실한 접촉(Uncertain Contact), 충격(Impact), 미끄러짐(Slipping), 비접촉(No Contact) 중 어떤 상태에 해당하는지를 판단한다. 이러한 분리는 센서 드라이버(Sensor Driver)를 하드웨어 중심으로 유지하면서 로봇 형태(Morphology), 지형(Terrain), 제어기 설계, 운용 요구사항에 따라 접촉 알고리즘을 독립적으로 발전시킬 수 있게 한다.

유용한 ROS2 발 접촉 메시지(Foot-Contact Message)는 해당 발을 명확하게 식별하고 접촉 관측이 유효했던 시점을 기록해야 한다. 메시지는 타임스탬프(Timestamp)와 프레임 식별자(Frame Identifier)를 포함하는 표준 헤더(Header)를 비롯하여 접촉 상태(Contact Status), 신뢰도(Confidence), 접촉력(Contact Force), 접촉 위치(Contact Position), 표면 법선(Surface Normal), 기타 하위 구성요소에서 요구하는 정보를 포함할 수 있다. 모든 로봇이 모든 필드를 필요로 하는 것은 아니므로 인터페이스는 필수적인 접촉 의미(Contact Semantics)와 선택적인 상세 측정값을 구분해야 한다.

접촉 전환(Contact Transition)은 빠르게 발생하고 보행 제어에 직접적인 영향을 주기 때문에 타임스탬프 정확도(Timestamp Accuracy)가 특히 중요하다. 실제 물리적 이벤트가 발생한 후 상당한 시간이 지나 생성된 메시지는 제어기나 추정기가 잘못된 지지 구성(Support Configuration)을 사용하도록 만들 수 있다. 가능한 경우 타임스탬프는 ROS2 콜백(Callback)이 결과 메시지를 발행한 시간이 아니라 원래의 센서 측정 시점 또는 감지된 접촉 이벤트(Contact Event)의 발생 시점을 나타내야 한다.

프레임 식별자(Frame Identifier)는 힘, 접촉 위치, 표면 법선과 같은 벡터량(Vector Quantity)이 표현되는 좌표계(Coordinate System)를 정의한다. 발 프레임(Foot Frame)은 지역적인 접촉 측정(Local Contact Measurement)에 편리할 수 있으며, 베이스(Base) 또는 월드(World) 좌표는 전신 동역학(Whole-Body Dynamics)과 지형 추론(Terrain Reasoning)에 유용할 수 있다. 선택된 좌표 규칙(Coordinate Convention)은 명확하게 문서화해야 하며, 하위 소프트웨어에서 동일한 정보를 다른 좌표 프레임으로 요구할 경우 tf2를 사용하여 변환해야 한다.

이진 접촉 상태(Binary Contact State)는 단순하지만 실제 환경의 보행을 표현하기에는 충분하지 않을 수 있다. 접촉을 단순히 참(True) 또는 거짓(False)으로 표현하는 대신 추가적인 신뢰도 또는 접촉 품질(Contact Quality) 정보를 유지할 수 있다. 이러한 정보는 착지(Touchdown)와 이지(Liftoff) 전후, 낮은 접촉력, 유연한 표면(Compliant Surface), 센서 측정에 노이즈가 존재하는 상황에서 특히 유용하다. 이를 통해 제어기는 모든 임계값 변화에 동일하게 반응하지 않고 강한 지지 증거와 불확실한 전환을 구분할 수 있다.

직접적인 힘 측정값을 사용할 수 있는 경우 임계값 기반 감지(Threshold-Based Detection)는 비교적 간단한 구현 방법을 제공한다. 측정된 수직력(Normal Force)이 설정된 임계값을 초과하면 접촉 상태를 선언하고 다른 임계값 아래로 떨어지면 접촉을 해제할 수 있다. 접촉 활성화(Contact Activation)와 해제(Contact Release)에 서로 다른 임계값을 사용하는 히스테리시스(Hysteresis)를 적용하면 측정값이 하나의 판단 경계 주변에서 변동할 때 접촉과 비접촉 상태가 빠르게 반복되는 현상을 줄일 수 있다.

발의 충격과 액추에이터 진동(Actuator Vibration)은 짧은 시간 동안 측정값의 급격한 피크(Spike)를 발생시킬 수 있기 때문에 필터링(Filtering)이 필요할 수도 있다. 저역통과 필터(Low-Pass Filter), 이동 윈도우(Moving Window), 지속 조건(Persistence Condition), 디바운스 로직(Debounce Logic)을 사용하면 순간적인 노이즈가 실제 접촉 전환으로 잘못 해석되는 것을 방지할 수 있다. 그러나 과도한 평활화(Smoothing)는 지연을 발생시키고 착지 또는 이지 감지 지연은 보행 타이밍(Gait Timing), 상태 추정, 균형 제어(Balance Control)를 저하시킬 수 있으므로 필터링 지연은 제한되어야 한다.

더 발전된 시스템에서는 여러 관측값을 융합하여 강건성(Robustness)을 향상시킬 수 있다. 힘 측정값을 관절 위치(Joint Position), 관절 속도(Joint Velocity), 모터 토크(Motor Torque), 발 속도(Foot Velocity), IMU 가속도, 예상 보행 위상(Expected Gait Phase)과 결합할 수 있다. 모델 기반 또는 확률적 추정기(Probabilistic Estimator)는 단일 센서 임계값에 의존하지 않고 접촉 가능성(Contact Likelihood)을 계산할 수 있으며, 이러한 접근법은 충격, 부분 접촉(Partial Contact), 미끄러짐, 표면 탄성(Compliance) 때문에 단순한 감지 가정이 신뢰하기 어려운 불규칙 지형(Irregular Terrain)에서 유용하다.

접촉 상태는 보행 제어기(Gait Controller)와 밀접하게 연결된다. 계획된 입각기(Stance Phase) 동안 제어기는 특정 발이 몸체를 지지할 것으로 예상하며, 유각 발(Swing Foot)은 일반적으로 착지 전까지 큰 외력을 받지 않아야 한다. 예상 접촉(Expected Contact)과 측정 접촉(Measured Contact)을 비교하면 조기 착지(Early Touchdown), 지연 착지(Late Touchdown), 예상하지 못한 충돌(Unexpected Collision), 지지 손실(Loss of Support)을 감지할 수 있다. 이러한 이벤트는 보행 적응(Gait Adaptation), 궤적 수정(Trajectory Modification), 복구 동작(Recovery Behavior), 보다 안전한 운용 모드로의 전환을 유발할 수 있다.

상태 추정 역시 신뢰할 수 있는 접촉 정보에 크게 의존한다. 발이 지면에 안정적으로 고정되어 있다고 높은 신뢰도로 판단되는 경우 추정기는 해당 접촉을 운동학적 제약조건(Kinematic Constraint)으로 사용하여 몸체 속도와 자세(Pose)의 추정 성능을 향상시킬 수 있다. 발이 미끄러지고 있는데도 추정기가 계속 고정된 상태로 판단하면 상당한 상태 추정 오차가 누적될 수 있다. 따라서 접촉 메시지는 신뢰하기 어려운 측정값을 절대적인 사실로 표현하기보다 불확실성과 잠재적인 비정상 상태(Abnormal Condition)를 함께 제공해야 한다.

전신 제어(Whole-Body Control)는 접촉 정보를 사용하여 최적화 문제(Optimization Problem)에 포함해야 하는 환경 제약조건(Environmental Constraint)과 힘을 결정한다. 로봇이 보행 위상 사이를 전환함에 따라 활성 접촉 집합(Active Contact Set)이 변경되며, 이에 따라 제어기는 지지 제약조건(Support Constraint), 허용 접촉력(Allowable Contact Force), 마찰 조건(Friction Condition), 몸체 목표(Body Objective)를 갱신해야 한다. 정확한 접촉 타이밍은 더 이상 지지하지 않는 발을 통해 힘을 명령하거나 아직 안정적인 접촉을 형성하지 않은 발에 조기에 하중을 가하는 것을 방지한다.

미끄러짐 감지(Slip Detection)는 단순한 지지 여부를 넘어 접촉 메시지의 기능을 확장할 수 있다. 발이 하중을 받고 있으면서 지면에 대해 움직이는 경우 접촉은 존재하지만 안정적인 상태는 아니다. 추정된 발 움직임, 접선력(Tangential Force), 수직력, 마찰 가정(Friction Assumption), 몸체 움직임을 비교하여 잠재적인 미끄러짐을 식별할 수 있다. 따라서 메시지는 접촉 존재(Contact Presence)와 접촉 품질을 구분하여 보행 소프트웨어가 불안정한 지형 상호작용에 적절하게 대응하도록 할 수 있다.

4족보행 로봇(Quadruped Robot)과 휴머노이드(Humanoid)에서는 접촉 정보가 여러 사지(Limb)에 자연스럽게 확장될 수 있어야 한다. 각 발이 독립적인 접촉 상태를 발행하거나 하나의 통합 메시지(Aggregated Message)가 일관되게 정렬된 구조를 사용하여 모든 발의 상태를 표현할 수 있다. 독립적인 메시지는 모듈식 처리(Modular Processing)를 단순화하고, 통합 방식은 전체 지지 구성을 동기화된 하나의 스냅샷(Synchronized Snapshot)으로 제공할 수 있다. 어떤 방식을 선택하더라도 명확한 발 식별과 일관된 타임스탬프를 유지해야 한다.

ROS2 서비스 품질(Quality of Service, QoS) 설정은 접촉 상태가 시간에 민감하면서 운용적으로 중요하다는 특성을 반영해야 한다. 고주파 접촉 관측에서는 오래된 접촉 상태가 일부 샘플 손실보다 더 위험할 수 있기 때문에 일반적으로 최신 데이터(Fresh Data)와 제한된 큐(Bounded Queue)를 우선한다. 중요한 이산 전환(Discrete Transition)이나 안전 관련 이벤트(Safety-Related Event)는 핵심 상태 변화가 고주파 텔레메트리(High-Frequency Telemetry)에 묻히지 않도록 보다 강한 전달 보장 또는 별도의 이벤트 채널(Event Channel)을 사용할 수 있다.

진단(Diagnostics)은 접촉 추정 품질(Contact-Estimation Quality)뿐만 아니라 기반 센싱 체인(Sensing Chain)의 상태도 감시해야 한다. 유용한 지표에는 센서 가용성(Sensor Availability), 메시지 경과 시간(Message Age), 접촉 전환 빈도(Contact Transition Frequency), 힘 포화(Force Saturation), 유효하지 않은 수치 값, 예상 접촉과 측정 접촉 사이의 불일치, 지속적인 미끄러짐, 중복 센서(Redundant Sensor) 사이의 상태 불일치가 포함된다. 이러한 진단은 기계적 문제, 센서 고장, 보정 오류(Calibration Error), 통신 지연, 알고리즘 추정 실패를 구분하는 데 도움을 준다.

시뮬레이션(Simulation)은 실제 로봇과 호환되는 의미 체계(Semantics)를 사용하여 접촉 정보를 제공해야 한다. Gazebo, Isaac Sim, MuJoCo 또는 기타 물리 환경(Physics Environment)은 접촉력과 충돌 정보(Collision Information)를 제공하고 이를 실제 하드웨어와 동일한 ROS2 접촉 표현으로 변환할 수 있다. 이러한 인터페이스 일관성(Interface Consistency)을 유지하면 보행 제어기, 추정기, 학습 정책, 진단 및 로깅 구성요소가 최소한의 소프트웨어 변경으로 시뮬레이션과 실제 배포 환경에서 동작할 수 있다.

학습 기반 보행(Learning-Based Locomotion)은 접촉 상태를 관측값(Observation) 또는 학습 신호(Training Signal)로 자주 사용한다. 강화학습 정책(Reinforcement-Learning Policy)은 현재 또는 최근의 접촉 정보를 관절 상태(Joint State), IMU 측정값, 명령(Command), 이전 행동(Previous Action)과 함께 입력으로 사용할 수 있다. 접촉 이벤트는 보상 함수(Reward Function), 보행 위상 추정(Gait-Phase Estimation), 지형 적응(Terrain Adaptation), 정책 평가(Policy Evaluation)에도 활용할 수 있다. ROS2 통합 계층은 접촉 정보를 모델 입력으로 변환할 때 순서, 타이밍, 신뢰도, 프레임 의미를 보존해야 한다.

접촉 정보를 JointState, IMU, 액추에이터 명령(Actuator Command), tf2 변환, 제어기 상태(Controller State), 정책 출력(Policy Output)과 함께 로깅하면 보행 실패(Locomotion Failure)를 상세하게 재구성할 수 있다. 엔지니어는 불안정성이 예상하지 못한 충격, 착지 감지 실패(Missed Touchdown), 발 미끄러짐, 메시지 지연, 잘못된 상태 추정, 부적절한 제어기 반응 중 어디에서 시작되었는지를 분석할 수 있다. 따라서 정확하게 동기화된 로그(Synchronized Log)는 접촉 인터페이스를 런타임 운용뿐만 아니라 사후 이벤트 분석(Post-Event Analysis)에도 중요한 정보원으로 만든다.

견고한 발 접촉 메시지 설계(Foot-Contact Message Design)는 궁극적으로 접촉을 단순한 불리언 센서 출력(Boolean Sensor Output)이 아니라 시간에 따라 변화하는 물리적 상태(Time-Dependent Physical State)로 다룬다. 명확한 발 식별(Foot Identity), 정확한 타임스탬프, 정의된 좌표 프레임, 신뢰도 정보, 힘 및 접촉 메타데이터(Contact Metadata), 필터링, 진단, 일관된 시뮬레이션 인터페이스를 통해 ROS2 구성요소는 로봇과 환경의 상호작용(Robot-Environment Interaction)을 신뢰성 있게 공유할 수 있다. 이러한 인터페이스는 센싱, 상태 추정, 보행 제어, 전신 제어, 안전 감독(Safety Supervision), 시뮬레이션, 피지컬 AI(Physical AI)를 연결하는 핵심 연결 계층이 된다.

## 11.06 Gait Controller ROS2 Interface Design [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

보행 제어기(Gait Controller)는 상위 수준의 이동 의도(High-Level Motion Intention)를 여러 다리와 몸체의 협조된 동작으로 변환하는 핵심 보행 구성요소(Locomotion Component)이다. ROS2 기반 다족보행 로봇에서 보행 제어기 인터페이스는 내비게이션(Navigation), 운용자 명령(Operator Command), 상태 추정(State Estimation), 접촉 감지(Contact Detection), 궤적 생성(Trajectory Generation), 전신 제어(Whole-Body Control), 액추에이터 제어(Actuator Control)를 연결하면서 비결정론적인 미들웨어 동작(Non-Deterministic Middleware Operation)이 시간 임계적인 피드백 루프(Time-Critical Feedback Loop)에 직접 개입하지 않도록 설계해야 한다.

인터페이스는 감독 수준 명령(Supervisory Command)과 고주파 제어 신호(High-Frequency Control Signal)를 분리해야 한다. ROS2 토픽(Topic), 서비스(Service), 액션(Action)을 통해 목표 속도(Desired Velocity), 보행 모드(Gait Mode), 몸체 자세(Body Posture), 이동 상태(Locomotion State), 전환 요청(Transition Request)을 전달할 수 있으며, 결정론적 제어 프로세스(Deterministic Control Process)는 훨씬 높은 주기로 관절 또는 액추에이터 기준값을 생성한다. 이러한 분리는 ROS2의 유연성을 유지하면서 통신 지연과 실행기 지터(Executor Jitter)가 보행 안정성을 저해하는 것을 방지한다.

기본적인 명령 인터페이스(Command Interface)는 이동 로봇에서 사용하는 일반적인 선속도(Linear Velocity) 및 각속도(Angular Velocity) 표현을 확장할 수 있다. 목표 전진 속도, 횡방향 속도(Lateral Velocity), 요 속도(Yaw Velocity)는 직관적인 이동 명령을 제공하지만, 다족보행에는 몸체 높이(Body Height), 롤 및 피치 목표(Roll and Pitch Target), 입각 폭(Stance Width), 스텝 주파수(Step Frequency), 보폭(Stride Length), 지형 관련 파라미터(Terrain-Related Parameter)가 추가로 필요할 수 있다. 이러한 값에는 명확한 단위, 좌표 프레임, 제한값, 타임스탬프 의미(Timestamp Semantics)를 정의해야 한다.

보행 선택(Gait Selection)을 위해서는 명확하게 정의된 모드 인터페이스(Mode Interface)가 필요하다. 4족보행 로봇(Quadruped Robot)은 정지(Stand), 걷기(Walk), 트로트(Trot), 페이스(Pace), 바운드(Bound), 크롤(Crawl), 복구(Recovery), 특수 지형 모드(Specialized Terrain Mode)를 지원할 수 있으며, 휴머노이드 시스템(Humanoid System)은 서기(Standing), 걷기(Walking), 스테핑(Stepping), 회전(Turning), 복구 동작을 제공할 수 있다. 안전하게 새로운 모드가 활성화되기 전에 접촉 위상(Contact Phase)과 동기화된 전환이 필요할 수 있으므로 요청된 보행(Requested Gait)과 현재 활성화된 보행(Active Gait)을 구분해야 한다.

따라서 보행 전환(Gait Transition)은 즉각적인 파라미터 변경이 아니라 제어된 상태 변화(Controlled State Change)로 다루어야 한다. 정지 상태에서 트로트로 변경하거나 속도 범위를 변경하고 복구 동작으로 진입하는 과정에는 지지 안정성(Support Stability)을 유지하기 위한 중간 단계가 필요할 수 있다. 보행 관리자(Gait Manager)는 요청을 검증하고 전환 조건이 충족되었는지 판단하며 전환 시퀀스(Transition Sequence)를 실행하고, 활성(Active), 대기(Pending), 거부(Rejected), 고장(Faulted) 상태를 상위 ROS2 구성요소에 전달할 수 있다.

보행 제어기는 상태 추정 입력(State-Estimation Input)에 크게 의존한다. 추정된 몸체 위치(Body Position), 자세(Orientation), 선속도 및 각속도, 관절 상태(Joint State), 필요한 경우 지형 상대 정보(Terrain-Relative Information)는 안정적인 보행을 생성하는 데 필요한 피드백을 제공한다. 입력 메시지는 정확한 타임스탬프와 명확하게 정의된 좌표 프레임을 포함해야 하며, 이를 통해 제어기는 새로운 보행 명령을 계산하기 전에 실제 로봇 움직임과 오래되거나 공간적으로 일관되지 않은 측정값을 구분할 수 있어야 한다.

발 접촉 정보(Foot-Contact Information)는 또 다른 핵심 인터페이스를 제공한다. 제어기는 어떤 발이 입각 상태(Stance)에 있어야 하는지와 실제로 어떤 접촉이 감지되었는지를 알아야 한다. 계획된 접촉 상태와 측정된 접촉 상태를 비교하면 조기 착지(Early Touchdown), 지연 착지(Late Touchdown), 예상하지 못한 충격(Unexpected Impact), 지지 손실(Loss of Support), 미끄러짐(Slipping)을 감지할 수 있다. 이러한 이벤트는 지형 조건이 기존 계획과 다를 때 위상 타이밍(Phase Timing), 유각 궤적(Swing Trajectory), 몸체 안정화(Body Stabilization), 전체 보행 모드를 변경하는 데 활용할 수 있다.

보행 생성(Gait Generation)은 일반적으로 각 다리에 대한 위상 표현(Phase Representation)을 내부적으로 유지한다. 위상은 다리가 입각(Stance), 유각(Swing), 전환(Transition), 기타 보행별 상태 중 어디에 있는지를 결정하며 궤적 생성에 필요한 타이밍 정보를 제공한다. ROS2 진단 또는 모니터링 인터페이스를 통해 이러한 위상을 외부에 제공할 수 있지만, 고주파 위상 갱신(High-Frequency Phase Update) 자체는 네트워크 메시지 전달에 의존하기보다 일반적으로 실시간 보행 프로세스(Real-Time Locomotion Process) 내부에 유지해야 한다.

유각 발 궤적 생성(Swing-Foot Trajectory Generation)은 보행 타이밍과 목표 움직임을 공간적인 발 궤적(Spatial Foot Trajectory)으로 변환한다. 입력에는 현재 발 자세(Current Foot Pose), 목표 착지점(Desired Foothold), 지형 높이(Terrain Height), 장애물 정보(Obstacle Information), 유각 지속시간(Swing Duration), 발 들림 높이(Clearance Requirement)가 포함될 수 있다. 출력은 일반적으로 시간에 따른 위치, 속도, 필요에 따라 가속도 기준값을 정의한다. 예상하지 못한 접촉 이후에도 궤적을 조정할 수 있어야 하며 운동학 및 액추에이터 제약조건을 만족하면서 부드러운 움직임을 유지해야 한다.

입각 다리(Stance Leg)는 지지하는 발이 환경과 직접 상호작용하기 때문에 서로 다른 요구사항을 가진다. 보행 제어기는 목표 접촉 위치(Desired Contact Location), 지지력(Support Force), 몸체 움직임 목표(Body Motion Objective), 제약조건을 전신 제어기에 제공할 수 있다. 모델 예측 제어(Model Predictive Control) 또는 최적화 기반 보행(Optimization-Based Locomotion)에서는 접촉 스케줄(Contact Schedule), 마찰 가정(Friction Assumption), 예측 몸체 궤적(Predicted Body Trajectory), 유한 계획 구간(Finite Planning Horizon)의 힘 분배(Force Distribution)를 추가로 전달할 수 있다.

보행 제어와 전신 제어 사이의 경계는 명확하게 유지해야 한다. 보행 계층(Gait Layer)은 이동 구조(Locomotion Structure), 위상 타이밍, 목표 착지점, 몸체 목표를 결정하고, 전신 제어기는 이러한 목표를 동역학적으로 일관된 관절 가속도(Joint Acceleration), 토크(Torque), 기타 액추에이터 수준 기준값으로 변환한다. 이러한 분리를 유지하면 서로 다른 보행 플래너(Gait Planner), 최적화 방법(Optimization Method), 액추에이터 제어기를 모든 하위 시스템과 강하게 결합하지 않고 독립적으로 발전시킬 수 있다.

명령 유효성(Command Validity)은 지속적으로 감독해야 한다. 운용자 연결, 내비게이션 노드, 네트워크 링크가 실패하면 속도 명령이 오래된 상태(Stale State)가 될 수 있다. 보행 제어기는 명령 타임스탬프를 감시하고 속도 감소, 정지 자세로의 전환, 안전 정지(Safe Stop) 요청과 같은 정의된 타임아웃 동작(Timeout Behavior)을 수행해야 한다. 유효하지 않은 수치 값, 설정된 한계를 초과하는 명령, 지원되지 않는 보행 요청은 보행 코어(Locomotion Core)에 도달하기 전에 거부하거나 제한해야 한다.

ROS2 서비스 품질(Quality of Service, QoS) 설정은 각 인터페이스의 의미에 맞게 구성해야 한다. 연속적인 속도 명령과 보행 상태는 일반적으로 최신 데이터와 제한된 큐(Bounded Queue)를 우선하지만, 이산적인 모드 전환, 구성 변경(Configuration Change), 안전 관련 요청은 신뢰성 높은 전달(Reliable Delivery)이 필요할 수 있다. 특히 인지 데이터와 보행 트래픽이 동일한 컴퓨팅 네트워크를 공유하는 경우 고주파 텔레메트리(High-Frequency Telemetry)가 중요한 전환 또는 안전 정보 채널을 혼잡하게 만들어서는 안 된다.

실시간 실행(Real-Time Execution)을 위해서는 ROS2 콜백과 결정론적 제어기 갱신(Deterministic Controller Update)을 신중하게 분리해야 한다. 구독자 콜백(Subscriber Callback)은 검증된 최신 명령이나 상태 추정값을 실시간 안전 버퍼(Real-Time-Safe Buffer)에 저장하고, 보행 루프는 차단 없이 해당 버퍼를 읽을 수 있다. 동적 메모리 할당(Dynamic Memory Allocation), 긴 뮤텍스 대기(Mutex Wait), 로깅 동작, 비용이 큰 직렬화(Serialization)는 핵심 루프에서 피해야 한다. CPU 친화성(CPU Affinity)과 실시간 스케줄링(Real-Time Scheduling)을 통해 타이밍 예측 가능성을 추가로 향상시킬 수 있다.

제어기 출력 인터페이스(Controller Output Interface)는 보행 계층 아래의 아키텍처에 따라 달라진다. 일부 시스템은 관절 위치 또는 속도 목표를 생성하며, 토크 제어 로봇(Torque-Controlled Robot)은 전신 제어를 통해 목표 토크(Desired Torque)를 생성할 수 있다. 다른 아키텍처에서는 발 궤적과 몸체 목표를 중간 제어기(Intermediate Controller)에 전달한다. ros2_control은 표준화된 명령 및 상태 인터페이스를 제공할 수 있지만, 최종 서보 루프(Servo Loop)는 전용 실시간 프로세서, 모터 제어기 또는 필드버스 하드웨어(Fieldbus Hardware)에 유지될 수 있다.

안전 감독(Safety Supervision)은 보행 알고리즘 자체와 독립적으로 유지되어야 한다. 관절 제한(Joint Limit), 토크 제한(Torque Limit), 속도 제약조건(Velocity Constraint), 몸체 자세 임계값(Body Orientation Threshold), 통신 감시기(Communication Watchdog), 액추에이터 고장(Actuator Fault), 넘어짐 감지(Fall Detection)는 필요한 경우 보행 명령을 무시하고 시스템을 보호할 수 있어야 한다. 보행 제어기가 공격적인 움직임을 요청하더라도 독립적으로 강제되는 안전 메커니즘이 최종 명령이 로봇의 허용 운용 범위(Operating Envelope) 내에 있는지를 결정해야 한다.

생명주기 관리(Lifecycle Management)는 보행 제어기의 가용성을 체계적으로 관리하는 방법을 제공한다. 필요한 관절 인터페이스, 상태 추정, 접촉 센싱(Contact Sensing), 변환(Transform), 액추에이터 시스템이 정상적인 상태가 되기 전에는 제어기가 활성 보행 상태(Active Locomotion State)에 진입해서는 안 된다. 설정(Configuration), 활성화(Activation), 비활성화(Deactivation), 오류 처리(Error Processing), 복구(Recovery)를 ROS2 생명주기 개념을 통해 조정함으로써 불완전하거나 일관되지 않은 시스템 상태에서 보행이 시작되는 것을 방지할 수 있다.

진단(Diagnostics)은 제어 루프에 영향을 주지 않으면서 보행 동작을 이해하는 데 충분한 정보를 제공해야 한다. 유용한 데이터에는 요청된 보행(Requested Gait)과 활성 보행, 위상 상태(Phase State), 명령 경과 시간(Command Age), 제어기 주파수(Controller Frequency), 실행 지연(Execution Latency), 접촉 불일치(Contact Mismatch), 착지점 조정(Foothold Adjustment), 포화 상태(Saturation Condition), 추종 오차(Tracking Error), 전환 상태(Transition Status)가 포함된다. 타임스탬프가 포함된 진단 정보는 JointState, IMU, 접촉, tf2, 액추에이터, 인지 로그와 연계할 때 특히 유용하다.

시뮬레이션(Simulation)은 실제 하드웨어에서 사용하는 것과 동일한 감독 수준 보행 인터페이스(Supervisory Gait Interface)를 제공해야 한다. Gazebo, Isaac Sim, MuJoCo 또는 기타 환경은 동일한 보행 및 속도 명령을 수신하면서 호환되는 상태, 접촉, 관절 인터페이스를 제공할 수 있다. 이러한 일관성은 소프트웨어 인 더 루프(Software-in-the-Loop) 및 하드웨어 인 더 루프(Hardware-in-the-Loop) 검증을 가능하게 하며, ROS2 기반 제어 아키텍처를 재설계하지 않고 보행 알고리즘을 실제 로봇 배포 단계로 발전시킬 수 있게 한다.

학습 기반 보행(Learning-Based Locomotion)도 동일한 인터페이스 계약(Interface Contract) 내부에 통합할 수 있다. 강화학습 정책(Reinforcement-Learning Policy)은 기존의 보행 생성을 대체하거나 보완하면서 관절 목표(Joint Target), 잠재 행동(Latent Action), 착지점 조정, 보행 파라미터를 생성할 수 있다. ROS2는 정책 선택(Policy Selection), 명령, 모니터링, 진단을 관리하며, 결정론적 어댑터(Deterministic Adapter)는 관측 타이밍을 검증하고 입력을 정규화(Normalization)하며 행동을 제한하고 추론 데드라인(Inference Deadline)을 놓쳤을 때 대체 동작(Fallback Behavior)을 실행할 수 있다.

견고한 ROS2 보행 제어기 인터페이스는 궁극적으로 상위 수준의 이동 의도와 시간 임계적인 물리적 실행(Time-Critical Physical Execution)을 연결하는 통제된 경계(Controlled Boundary) 역할을 한다. 명확한 명령 의미(Command Semantics), 보행 상태 관리(Gait-State Management), 동기화된 상태 추정 및 접촉 입력, 실시간 안전 데이터 교환(Real-Time-Safe Data Exchange), 명확한 전신 제어 경계, 안전 감독, 생명주기 관리, 진단, 시뮬레이션 호환성(Simulation Compatibility)을 통해 동일한 아키텍처에서 전통적인 보행 알고리즘(Classical Gait Algorithm), 최적화 기반 제어(Optimization-Based Control), 학습 기반 피지컬 AI(Learning-Based Physical AI)를 함께 지원할 수 있다.

## 11.07 RL Policy Inference Node Integration [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

강화학습 정책 추론 노드(RL Policy Inference Node)는 학습된 강화학습 정책(Reinforcement-Learning Policy)과 다족보행 로봇의 ROS2 보행 아키텍처(ROS2 Locomotion Architecture)를 연결하는 런타임 가교(Runtime Bridge)를 제공한다. 주요 역할은 동기화된 로봇 관측값(Robot Observation)을 획득하고, 이를 학습된 모델이 요구하는 표현으로 변환하며, 제한된 시간 예산(Time Budget) 내에서 추론을 실행하고, 결과 행동(Action)을 검증한 후 안전한 명령을 하위 보행, 전신 제어(Whole-Body Control), 액추에이터 제어(Actuator Control) 구성요소에 전달하는 것이다.

추론 노드(Inference Node)는 ROS2 기반 인지 및 상태 인터페이스와 결정론적 보행 실행 계층(Deterministic Locomotion Execution Layer) 사이에 배치해야 한다. 입력은 JointState, IMU, 발 접촉(Foot Contact), 상태 추정(State Estimation), 명령(Command), 지형 인지(Terrain Perception) 토픽을 통해 전달될 수 있으며, 출력은 관절 위치(Joint Position), 관절 속도(Joint Velocity), 토크(Torque), 목표 발 위치(Desired Foot Location), 보행 파라미터(Gait Parameter), 잠재 행동(Latent Action)을 나타낼 수 있다. 정확한 경계는 학습 정책 내부에 어느 수준까지 보행 지능(Locomotion Intelligence)이 포함되어 있는지에 따라 결정된다.

관측값 구성(Observation Construction)은 가장 중요한 통합 책임 중 하나이다. 정책은 관절 위치 및 속도, 베이스 자세(Base Orientation), 각속도(Angular Velocity), 투영 중력(Projected Gravity), 이동 명령(Commanded Motion), 이전 행동(Previous Action), 접촉 상태(Contact State), 지형 특징(Terrain Feature), 고유감각 이력(Proprioceptive History)을 요구할 수 있다. 따라서 ROS2 메시지는 학습 시 사용된 순서, 차원(Dimension), 단위(Unit), 좌표 프레임(Coordinate Frame), 정규화 규칙(Normalization Rule), 시간적 해석(Temporal Interpretation)과 정확하게 일치하는 고정 관측 벡터(Fixed Observation Vector)로 변환되어야 한다.

신경망(Neural Network)은 각 입력 차원을 학습 시 정의된 의미에 따라 해석하므로 관측값 순서(Observation Ordering)는 결정론적으로 유지되어야 한다. URDF 관절 순서, JointState 발행 순서, 센서 구성 또는 로봇 버전이 변경되더라도 정책 입력의 배열 순서가 암묵적으로 변경되어서는 안 된다. 추론 노드는 ROS2 관절 이름과 정책 인덱스(Policy Index) 사이의 명시적인 매핑(Mapping)을 사용하고 초기화 과정에서 예상 입력을 검증하며, 보행이 활성화되기 전에 호환되지 않는 구성을 거부해야 한다.

정규화 파라미터(Normalization Parameter)는 선택적인 전처리 세부사항이 아니라 배포된 정책 인터페이스(Deployed Policy Interface)의 일부이다. 관절 위치, 속도, 명령, 지형 관측 및 기타 값은 학습 과정에서 스케일링(Scaling), 클리핑(Clipping), 중심화(Centering), 변환(Transformation)이 적용되었을 수 있다. 런타임 노드는 이러한 연산을 일관되게 재현해야 한다. 따라서 평균(Mean), 표준편차(Standard Deviation), 스케일 계수(Scaling Factor), 클리핑 범위(Clipping Range), 관측 규칙(Observation Convention)은 학습된 모델 및 배포 구성과 함께 버전 관리되어야 한다.

여러 ROS2 소스에서 관측값이 생성되는 경우 시간적 일관성(Temporal Consistency)이 특히 중요하다. 관절 상태, IMU 데이터, 접촉 추정(Contact Estimation), 명령, 지형 정보는 서로 다른 주기와 통신 지연을 가지고 도착할 수 있다. 추론 노드는 추론이 시작되는 순간 단순히 사용 가능한 메시지를 조합하는 대신 타임스탬프(Timestamp), 동기화 버퍼(Synchronized Buffer), 최신 유효 샘플 정책(Latest-Valid-Sample Policy), 명시적 보간(Explicit Interpolation)을 사용하여 충분히 일관된 물리적 시점(Physical Instant)을 나타내는 관측값을 구성해야 한다.

정책 실행 주기(Policy Execution Rate)는 하위 수준 서보 주기(Lower-Level Servo Frequency)와 구분해야 한다. 학습 기반 보행 정책은 수십에서 수백 Hz 수준으로 실행될 수 있지만, 모터 전류, 토크 또는 위치 루프는 실시간 제어기(Real-Time Controller)에서 훨씬 높은 주기로 동작할 수 있다. 정책 갱신 사이에는 하위 제어기가 가장 최근의 유효 행동을 유지하거나 보간(Interpolation), 필터링(Filtering), 추종(Tracking)할 수 있다. 이러한 계층적 설계는 신경망 추론 타이밍이 모든 액추에이터 서보 동작의 직접적인 시간 기준이 되는 것을 방지한다.

추론 지연(Inference Latency)은 단순한 신경망 실행 시간이 아니라 종단 간 지연(End-to-End Latency)으로 측정해야 한다. 관측값 획득, 전처리(Preprocessing), 텐서 변환(Tensor Conversion), CPU-GPU 전송, 모델 실행(Model Execution), 출력 전송, 후처리(Post-Processing), 명령 발행(Command Publication)이 모두 실제 지연에 영향을 준다. 따라서 런타임 진단(Runtime Diagnostics)은 현실적인 동시 시스템 부하(Concurrent System Workload)에서 정책 실행 시간, 전체 파이프라인 지연(Pipeline Latency), 데드라인 미준수(Deadline Miss), 관측값 경과 시간(Observation Age), 행동 경과 시간(Action Age), 추론 주파수(Inference Frequency)를 기록해야 한다.

GPU 가속(GPU Acceleration)은 추론 성능을 크게 향상시킬 수 있지만 데이터 이동(Data Movement)과 스케줄링 오버헤드(Scheduling Overhead)를 함께 고려해야 한다. 반복적인 메모리 할당과 불필요한 호스트-디바이스 복사(Host-Device Copy)는 사용 가능한 시간 예산의 상당 부분을 소비할 수 있다. 사전 할당된 텐서(Preallocated Tensor), 영구 버퍼(Persistent Buffer), 필요한 경우 고정 메모리(Pinned Memory), 최적화된 추론 런타임(Optimized Inference Runtime), 신중한 디바이스 배치(Device Placement)를 사용하여 오버헤드를 줄일 수 있다. 최적의 구현은 정책 크기, 관측 차원, 갱신 주기, 사용 가능한 엣지 컴퓨팅 하드웨어(Edge-Computing Hardware)에 따라 달라진다.

추론 콜백(Inference Callback)은 중요한 실시간 경로(Real-Time Path) 내부에서 통제되지 않은 작업을 직접 수행해서는 안 된다. ROS2 콜백은 실시간 안전 관측 버퍼(Real-Time-Safe Observation Buffer)를 최신 데이터로 갱신하고, 전용 추론 루프(Dedicated Inference Loop)는 정의된 일정에 따라 검증된 스냅샷(Snapshot)을 사용할 수 있다. 생성된 행동은 다시 제한된 버퍼(Bounded Buffer)에 저장하여 하위 제어기에 전달할 수 있다. 이러한 구조는 긴 뮤텍스 대기(Mutex Wait), 동적 메모리 할당(Dynamic Memory Allocation), 블로킹 동작(Blocking Operation)을 피하면서 미들웨어 스케줄링 변동성을 제어기로부터 격리한다.

정책 출력(Policy Output)은 로봇에 적용하기 전에 검증해야 한다. 신경망 출력은 물리적인 관절 또는 보행 명령으로 변환하기 위해 먼저 스케일링해야 하는 정규화 행동(Normalized Action)일 수 있다. 통합 계층(Integration Layer)은 유한 수치(Finite Numerical Value), 차원 일관성(Dimensional Consistency), 설정 범위(Configured Range), 변화율 제한(Rate Limit), 관절 제한(Joint Limit), 토크 제약조건(Torque Constraint), 기타 안전 조건을 검증해야 한다. 모델이 성공적으로 실행되었다는 이유만으로 비정상적이거나 극단적인 출력을 액추에이터에 직접 전달해서는 안 된다.

연속된 정책 출력 사이에 물리적 기구가 안전하게 추종하기 어려운 불연속성(Discontinuity)이나 고주파 변화가 존재하는 경우 행동 평활화(Action Smoothing)가 필요할 수 있다. 변화율 제한, 보간, 저역통과 필터(Low-Pass Filter), 하위 궤적 추종(Downstream Trajectory Tracking)을 사용하여 보다 부드러운 액추에이터 기준값을 제공할 수 있다. 그러나 과도한 필터링은 학습 과정에서 획득된 정책의 동작 특성을 변화시킬 수 있으므로 배포 파이프라인은 실제 플랫폼에 필요한 물리적 제약조건만 적용하면서 정책이 의도한 동역학을 유지해야 한다.

이전 행동(Previous Action)은 강화학습 관측 벡터에 자주 포함된다. 추론 노드는 이 필드가 원시 정책 출력(Raw Policy Output), 클리핑된 행동(Clipped Action), 필터링된 명령(Filtered Command), 또는 하위 제어기가 실제로 수용한 행동(Accepted Action) 중 무엇을 의미하는지 명확하게 정의해야 한다. 학습 시 사용한 의미와 다른 표현을 사용하면 관측 불일치(Observation Mismatch)가 발생할 수 있다. 따라서 선택된 행동 표현은 시뮬레이션, 학습, 평가(Evaluation), 하드웨어 배포 전 과정에서 문서화되고 일관되게 유지되어야 한다.

정책 생명주기 관리(Policy Lifecycle Management)는 모델 로딩(Model Loading)과 보행 활성화(Locomotion Activation)를 조정해야 한다. 설정 단계에서 노드는 모델, 정규화 파라미터, 관절 매핑, 디바이스 구성(Device Configuration), 메타데이터(Metadata)를 로드하고 로봇과의 호환성을 검증할 수 있다. 필요한 상태, 접촉, 명령, 변환(Transform) 입력이 준비된 이후에만 활성화해야 한다. 비활성화(Deactivation) 시에는 행동 생성을 안전하게 중단해야 하며, 오류 처리에서는 마지막 행동을 무기한 유지하지 않고 미리 정의된 대체 동작(Fallback)으로 시스템을 전환해야 한다.

신경망 파일 하나만으로는 배포 가능한 정책을 완전히 정의할 수 없으므로 모델 버전 관리(Model Versioning)가 매우 중요하다. 정책 패키지(Policy Package)는 관측 스키마(Observation Schema), 행동 스키마(Action Schema), 관절 순서, 정규화 파라미터, 제어 주기(Control Frequency), 예상 로봇 모델(Expected Robot Model), 학습 환경(Training Environment), 소프트웨어 의존성(Software Dependency), 관련 안전 제한(Safety Limit)을 식별해야 한다. 런타임 소프트웨어는 이러한 메타데이터를 현재 로봇 구성과 비교하여 호환되지 않는 모델과 하드웨어 조합의 배포를 방지할 수 있다.

안전 감독(Safety Supervision)은 학습된 정책과 독립적으로 유지되어야 한다. 감시기(Watchdog)는 추론 데드라인 미준수, 오래된 관측값(Stale Observation), 통신 손실(Communication Loss), 유효하지 않은 행동, 비정상적인 몸체 자세(Abnormal Body Orientation), 액추에이터 고장(Actuator Fault), 과도한 추종 오차(Tracking Error)를 감지할 수 있다. 이러한 조건이 발생하면 시스템은 로봇의 안전 아키텍처에 따라 안전 명령을 유지하거나 전통적인 제어기(Conventional Controller)로 전환하고, 정지 모드(Standing Mode)에 진입하거나 액추에이터 권한(Actuator Authority)을 줄이고, 필요한 경우 비상 대응(Emergency Response)을 시작할 수 있다.

여러 학습 행동(Learned Behavior)을 사용할 수 있는 경우 정책 전환(Policy Switching)에는 통제된 상태 관리(Controlled State Management)가 필요하다. 로봇은 평지 보행, 거친 지형(Rough Terrain), 복구(Recovery), 계단(Stairs), 특수 기동(Specialized Maneuver)을 위한 여러 정책을 포함할 수 있다. 각 정책이 서로 다른 내부 상태나 관측 이력(Observation History)을 가정할 수 있기 때문에 모델을 즉시 전환하면 행동 불연속성이 발생할 수 있다. 따라서 정책 관리자(Policy Manager)는 전환 조건을 검증하고 필요에 따라 블렌딩(Blending), 중립 상태(Neutral State), 보행 위상 정렬(Gait-Phase Alignment), 기타 전환 메커니즘을 조정해야 한다.

가능한 경우 시뮬레이션과 하드웨어는 동일한 정책 인터페이스(Policy-Facing Interface)를 제공해야 한다. MuJoCo, Isaac Sim, Gazebo 또는 다른 환경은 하드웨어 추론 노드가 기대하는 것과 동일한 의미 체계를 사용하여 JointState, IMU, 접촉, 명령, 지형 관측을 제공할 수 있다. 이를 통해 정책은 기본적인 ROS2 관측 및 행동 계약(Observation and Action Contract)을 변경하지 않고 시뮬레이션, 소프트웨어 인 더 루프(Software-in-the-Loop), 하드웨어 인 더 루프(Hardware-in-the-Loop), 실제 로봇 배포 단계로 발전할 수 있다.

시뮬레이션-현실 전이(Sim-to-Real) 배포에는 단순히 메시지 유형(Message Type)을 동일하게 유지하는 것 이상의 작업이 필요하다. 좌표 규칙(Coordinate Convention), 센서 스케일링(Sensor Scaling), 액추에이터 응답(Actuator Response), 제어 주기, 지연, 접촉 의미(Contact Semantics), 관측 노이즈(Observation Noise), 행동 해석(Action Interpretation)이 학습 조건과 충분히 일치해야 한다. 도메인 랜덤화(Domain Randomization)는 모델링 불확실성에 대한 강건성(Robustness)을 높일 수 있지만, 런타임 통합 계층은 방지 가능한 인터페이스 차이가 추가적인 정책 실패 원인이 되지 않도록 해야 한다.

진단(Diagnostics)과 동기화된 로깅(Synchronized Logging)은 학습 기반 보행을 평가하는 데 필수적이다. 관측 벡터, 정책 행동, 원시 ROS2 입력, 처리된 상태(Processed State), 추론 지연, 제어기 출력, 접촉 이벤트(Contact Event), 안전 개입(Safety Intervention)을 호환 가능한 타임스탬프와 함께 기록해야 한다. 이를 통해 엔지니어는 예상하지 못한 동작이 학습 정책 자체, 관측 불일치, 오래된 데이터, 추론 지연, 행동 처리(Action Processing), 하위 제어, 물리적 상호작용 중 어디에서 발생했는지를 판단할 수 있다.

견고한 강화학습 정책 추론 노드는 단순한 신경망 모델의 래퍼(Wrapper)가 아니라 통제된 적응 계층(Controlled Adaptation Layer)으로 기능한다. 결정론적인 관측값 구성, 동기화된 입력, 학습과 일치하는 정규화, 제한된 추론 지연, 명확한 행동 의미(Action Semantics), 실시간 안전 버퍼링(Real-Time-Safe Buffering), 출력 검증, 생명주기 제어, 모델 버전 관리, 독립적인 안전 감독, 시뮬레이션-하드웨어 일관성(Simulation-to-Hardware Consistency)을 통해 강화학습 정책을 ROS2 기반 피지컬 AI 보행 스택(Physical AI Locomotion Stack)의 신뢰성 있는 구성요소로 통합할 수 있다.

## 11.08 RViz2-Based Legged Robot Monitoring [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

RViz2는 ROS2 기반 다족보행 로봇(Legged Robot)의 공간적 상태(Spatial State)와 동작을 이해하기 위한 그래픽 모니터링 환경(Graphical Monitoring Environment)을 제공한다. 주로 수치 데이터를 표시하는 일반적인 대시보드와 달리 RViz2는 로봇 모델(Robot Model), 좌표 프레임(Coordinate Frame), 센서 관측값(Sensor Observation), 궤적(Trajectory), 접촉(Contact), 내비게이션 정보를 하나의 공통 3차원 장면(Three-Dimensional Scene)에 통합한다. 따라서 보행(Locomotion), 인지(Perception), 상태 추정(State Estimation), 제어(Control) 사이의 복잡한 관계를 진단하는 데 특히 유용하다.

시각화(Visualization)는 일반적으로 URDF 또는 Xacro로 정의된 로봇 모델에서 시작한다. robot_description 파라미터는 로봇 상태 발행기(robot_state_publisher)에서 사용되며, 로봇 상태 발행기는 운동학 모델(Kinematic Model)과 입력되는 관절 상태(JointState) 데이터를 결합하여 해당 tf2 변환(Transform)을 발행한다. 이후 RViz2는 전체 로봇의 현재 형상을 재구성하여 엔지니어가 관절 움직임, 링크 방향(Link Orientation), 다리 기하구조(Limb Geometry), 몸체와 지지 다리 사이의 관계를 시각적으로 확인할 수 있게 한다.

정확한 고정 프레임(Fixed Frame)은 의미 있는 시각화를 위해 필수적이다. 모니터링 목적에 따라 RViz2는 map, odom 또는 base_link를 고정 기준으로 사용할 수 있다. map 프레임은 전역 내비게이션(Global Navigation)과 환경 확인에 유용하고, odom은 국부적으로 연속적인 움직임 분석(Local Continuous Motion Analysis)을 지원하며, base_link는 로봇 상대 동작(Robot-Relative Behavior)을 강조한다. 잘못된 프레임을 선택하면 실제 제어기가 정상적으로 동작하더라도 유효한 데이터가 불안정하거나 분리되거나 잘못된 위치에 표시될 수 있다.

tf2 변환 트리(Transform Tree)는 모니터링 시스템의 공간적 기반(Spatial Backbone)을 제공한다. 베이스(Base), 관절(Joint), 발(Foot), IMU, 카메라(Camera), 라이다(LiDAR), 기타 센서의 프레임을 함께 검사하여 기하학적 관계를 검증할 수 있다. RViz2는 누락된 변환(Missing Transform), 잘못된 부모-자식 관계(Parent-Child Relationship), 반전된 축(Reversed Axis), 캘리브레이션 오류(Calibration Error), 비정상적인 프레임 움직임을 발견하는 데 도움을 준다. 작은 공간적 불일치도 접촉 추정과 보행 제어에 영향을 줄 수 있으므로 이러한 문제는 다족보행 로봇에서 특히 중요하다.

관절 상태 시각화(JointState Visualization)는 로봇의 명령 자세(Commanded Posture)와 측정 자세(Measured Posture)를 즉각적으로 확인할 수 있게 한다. 관절 위치가 올바르게 발행되면 표시된 URDF 모델은 실제 또는 시뮬레이션 로봇의 기구 움직임을 따라간다. 이를 명령된 자세와 비교하면 관절 이름 불일치(Joint-Name Mismatch), 잘못된 부호 규칙(Sign Convention), 오프셋 오류(Offset Error), 오래된 데이터(Stale Data)를 발견할 수 있다. 수치적인 검사가 필요한 경우에는 보완적인 진단 도구를 통해 관절 속도와 노력값(Effort)을 추가로 모니터링할 수 있다.

발 접촉 모니터링(Foot-Contact Monitoring)은 각 발 프레임 또는 추정된 접촉 위치에 배치된 RViz2 마커(Marker)를 사용하여 표현할 수 있다. 마커의 색상, 형태, 크기 또는 텍스트를 이용해 입각(Stance), 유각(Swing), 불확실 접촉(Uncertain Contact), 충격(Impact), 미끄러짐(Slipping) 상태를 구분할 수 있다. 접촉력(Contact Force)은 추정된 지면 반력(Ground Reaction Force)의 방향과 크기를 나타내는 벡터로 시각화할 수 있다. 이러한 공간 표현은 계획된 접촉 스케줄과 실제 측정된 물리적 상호작용 사이의 불일치를 훨씬 쉽게 파악할 수 있게 한다.

보행 위상 정보(Gait Phase Information)도 시각화에 통합할 수 있다. 각 다리는 현재 입각 또는 유각 위상을 표시할 수 있으며, 마커를 이용하여 계획된 착지점(Planned Foothold), 현재 발 위치(Current Foot Position), 미래 유각 궤적(Future Swing Trajectory)을 나타낼 수 있다. 4족보행 로봇(Quadruped Robot)에서는 트로트(Trot) 동작 중 대각선 지지 패턴(Diagonal Support Pattern)이나 걷기 중 순차적 접촉 패턴을 확인할 수 있다. 휴머노이드(Humanoid)에서는 단일 지지(Single Support), 이중 지지(Double Support), 발 배치(Foot Placement), 스텝 전환(Step Transition) 동작을 분석하는 데 활용할 수 있다.

궤적 시각화(Trajectory Visualization)는 몸체 움직임과 개별 다리 모두에 유용하다. 계획된 베이스 궤적(Base Trajectory)은 Path 메시지를 사용하여 표시할 수 있으며, 유각 발 궤적과 목표 착지점은 Marker 또는 MarkerArray 메시지를 통해 표현할 수 있다. 목표 궤적과 측정 궤적을 비교하면 추종 품질(Tracking Quality)을 직관적으로 파악할 수 있다. 큰 공간적 편차가 발생하면 상태 추정 오류, 액추에이터 한계(Actuator Limitation), 제어기 포화(Controller Saturation), 잘못된 지형 가정(Terrain Assumption) 등을 원인으로 검토할 수 있다.

상태 추정 결과는 가능한 경우 원시 정보(Raw Information) 또는 독립적으로 생성된 정보와 함께 시각화해야 한다. 추정된 베이스 위치(Base Pose), 속도 방향(Velocity Direction), 자세(Orientation), 오도메트리 궤적(Odometry Trajectory)을 센서 관측값 또는 외부 위치 추정 기준(External Localization Reference)과 비교할 수 있다. 지원되는 경우 공분산 정보(Covariance Information)도 표시할 수 있다. 이를 통해 갑작스러운 위치 변화, 자세 드리프트(Orientation Drift), 일관되지 않은 속도 방향, 프레임 불연속(Frame Discontinuity)을 더 심각한 보행 문제로 발전하기 전에 시각적으로 발견할 수 있다.

IMU 모니터링은 몸체 자세와 각운동(Angular Motion)이 균형 제어(Balance Control)에 큰 영향을 주기 때문에 특히 중요하다. RViz2는 IMU 프레임과 관련된 자세 및 base_link와의 관계를 시각화할 수 있다. 이를 tf2 검사와 함께 사용하면 센서 장착 방향(Sensor Mounting Orientation)과 좌표 규칙(Coordinate Convention)을 검증하는 데 도움이 된다. 잘못된 축 매핑(Axis Mapping)이나 변환 캘리브레이션 오류는 수치적으로는 그럴듯한 측정값을 생성하면서도 근본적으로 잘못된 상태 추정 결과를 만들 수 있다.

인지 정보(Perception Information)는 PointCloud2, LaserScan, Image, 점유 지도(Occupancy Map), 고도 지도(Elevation Map), 기타 ROS2 호환 표현을 사용하여 동일한 장면에 통합할 수 있다. 불규칙 지형(Uneven Terrain)을 주행하는 다족보행 로봇은 주변 지형 형상과 함께 후보 착지점(Candidate Foothold) 및 몸체 궤적을 표시할 수 있다. 따라서 엔지니어는 보행 결정이 인지된 장애물, 경사면(Slope), 계단(Step), 틈(Gap), 기타 지형 구조에 적절하게 대응하는지를 확인할 수 있다.

내비게이션 모니터링(Navigation Monitoring)은 개별 보행 주기를 넘어 시각화 범위를 확장한다. 전역 경로(Global Path), 지역 궤적(Local Trajectory), 목표 자세(Goal Pose), 장애물 지도(Obstacle Map), 로봇 풋프린트(Robot Footprint)를 map 및 odom 프레임을 기준으로 표시할 수 있다. 이를 통해 내비게이션 수준의 의도와 보행 수준의 실행을 직접 연결할 수 있다. 로봇이 경로를 제대로 추종하지 못할 경우 RViz2를 이용해 문제가 위치 추정(Localization), 경로 계획(Path Planning), 지역 움직임 생성(Local Motion Generation), 하위 보행 실행 중 어느 부분에서 발생했는지를 분석할 수 있다.

학습 기반 보행(Learning-Based Locomotion)에서도 내부 신경망(Neural Network)을 직접 노출하지 않고 동일한 시각화 인프라를 활용할 수 있다. 정책 명령(Policy Command), 선택된 이동 모드(Locomotion Mode), 예측 착지점(Predicted Foothold), 행동 기반 목표(Action-Derived Target), 선택된 관측 특징(Observation Feature)을 시각화 마커로 변환할 수 있다. 목적은 모든 신경망 값을 표시하는 것이 아니라 학습된 의사결정이 로봇 움직임과 물리적 상호작용에 어떠한 영향을 주는지를 이해할 수 있도록 물리적으로 해석 가능한 값을 제공하는 것이다.

RViz2는 결정론적 실시간 제어 경로(Deterministic Real-Time Control Path)의 외부에 유지해야 한다. 대규모 포인트 클라우드(Point Cloud), 이미지, 고밀도 마커(Dense Marker), 고주파 변환(High-Frequency Transform)을 동시에 표시하는 경우 시각화 트래픽은 상당한 네트워크 대역폭을 사용할 수 있다. 제어 노드는 운용과 진단에 필요한 정보만 발행하고, 시각화 어댑터(Visualization Adapter)가 내부 데이터를 적절한 주기로 RViz2 친화적인 표현으로 변환하도록 구성할 수 있다. 모니터링 작업이 제어 데드라인(Control Deadline)의 충족 여부에 영향을 주어서는 안 된다.

서비스 품질(Quality of Service, QoS) 설정은 시스템 부하가 높은 상황에서 시각화의 응답성을 유지하는 데 영향을 준다. 고주파 상태 및 센서 스트림은 제한된 큐(Bounded Queue)와 최신 데이터 중심의 동작이 유리한 반면, 정적이거나 천천히 변화하는 구성 정보는 보다 신뢰성 높은 전달(Reliable Delivery)을 사용할 수 있다. 시각화 구독자(Visualization Subscriber)는 처리 속도가 데이터 입력 속도를 따라가지 못할 때 오래된 데이터를 계속 누적하지 않도록 해야 한다. 실시간 보행 모니터링에서는 오래된 데이터를 순차적으로 재생하는 것보다 최신의 의미 있는 상태를 표시하는 것이 일반적으로 더 유용하다.

RViz2 구성(RViz2 Configuration)은 일시적인 사용자 인터페이스 설정이 아니라 재사용 가능한 엔지니어링 자산(Reusable Engineering Asset)으로 관리해야 한다. 디스플레이 선택(Display Selection), 고정 프레임, 토픽, 마커 네임스페이스(Marker Namespace), 카메라 시점(Camera Viewpoint), 그리드 파라미터(Grid Parameter), 시각화 스케일(Visualization Scale)을 RViz 구성 파일에 저장할 수 있다. 보행 개발, 접촉 디버깅(Contact Debugging), 인지 통합, 내비게이션 시험, 현장 진단(Field Diagnostics)을 위한 표준 구성을 마련하면 서로 다른 엔지니어가 일관된 모니터링 환경에서 로봇을 관찰할 수 있다.

여러 다족보행 로봇을 모니터링할 때는 네임스페이스(Namespace)가 중요해진다. RViz2가 각 로봇의 데이터를 명확하게 구분할 수 있도록 로봇별 토픽, 변환, 마커, 진단, 로봇 설명(Robot Description)을 분리해야 한다. 다중 로봇 시각화(Multi-Robot Visualization)는 공유 환경에서 각 로봇의 궤적, 지역 지도(Local Map), 접촉 상태, 작업 정보를 함께 표시할 수 있다. 일관된 명명 규칙(Naming Convention)은 프레임 충돌(Frame Collision)을 방지하고 기본 시각화 아키텍처를 변경하지 않으면서 플릿 수준 모니터링(Fleet-Level Monitoring)을 가능하게 한다.

가능한 경우 시뮬레이션과 실제 하드웨어는 호환 가능한 RViz2 모니터링 구성을 사용해야 한다. Gazebo, Isaac Sim, MuJoCo 또는 기타 시뮬레이션 환경에서 동작하는 로봇은 실제 시스템과 동일한 로봇 상태, 변환, 접촉, 궤적, 인지 표현을 발행할 수 있다. 따라서 엔지니어는 공통 시각 인터페이스(Common Visual Interface)를 통해 시뮬레이션과 실제 동작을 비교할 수 있으며, 캘리브레이션, 동역학(Dynamics), 타이밍, 제어 응답(Control Response)의 차이를 더욱 쉽게 식별할 수 있다.

RViz2는 단독 디버깅 메커니즘으로 사용하는 것보다 타임스탬프 기반 진단(Timestamped Diagnostics), rosbag 기록(Recording), 정량적 분석(Quantitative Analysis)과 결합할 때 가장 효과적이다. 시각적인 이상 현상(Visual Anomaly)을 통해 문제가 언제 어디에서 발생했는지를 확인하고, 기록된 JointState, tf2, IMU, 접촉, 제어기, 정책, 액추에이터 데이터를 이용해 그 수치적인 원인을 분석할 수 있다. 이러한 조합은 시각적 문제 발견에서 재현 가능한 엔지니어링 분석(Reproducible Engineering Analysis)으로 이어지는 실용적인 작업 흐름을 제공한다.

잘 설계된 RViz2 모니터링 아키텍처는 궁극적으로 전체 ROS2 다족보행 로봇 스택을 위한 관측 가능성 계층(Observability Layer)으로 기능한다. 로봇 모델, tf2 프레임, 관절 상태, 접촉, 보행 위상, 궤적, 상태 추정, 인지, 내비게이션, 학습 기반 출력을 하나의 공간적 맥락(Spatial Context)에 통합함으로써 엔지니어가 소프트웨어 상태와 실제 물리적 동작을 연결하여 이해할 수 있게 하며, 동시에 시각화 기능을 결정론적 제어(Deterministic Control) 및 안전 중요 실행(Safety-Critical Execution)으로부터 분리할 수 있다.

## 11.09 Quadruped Full ROS2 Stack Configuration Case [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

완전한 4족보행 로봇 ROS2 스택(Quadruped ROS2 Stack)은 일관된 ROS2 인터페이스를 중심으로 보행 제어(Locomotion Control), 상태 추정(State Estimation), 인지(Perception), 내비게이션(Navigation), 하드웨어 인터페이스(Hardware Interface), 안전(Safety), 진단(Diagnostics), 시스템 관리(System Management)를 통합한다. 목표는 단순히 개별 노드를 연결하는 것이 아니라 상위 수준 자율 기능(High-Level Autonomy)이 독립적으로 발전할 수 있으면서 결정론적 하위 제어(Deterministic Low-Level Control)가 미들웨어 지연과 연산 변동성(Computational Variability)으로부터 보호되는 계층형 아키텍처(Layered Architecture)를 구축하는 것이다.

하드웨어 계층(Hardware Layer)에서 로봇은 관절 액추에이터(Joint Actuator), 모터 드라이브(Motor Drive), 엔코더(Encoder), IMU, 발 접촉 센서(Foot-Contact Sensor), 카메라(Camera), 라이다(LiDAR), 필요한 경우 GNSS 또는 추가 환경 센서(Environmental Sensor)를 포함한다. 전용 임베디드 제어기(Embedded Controller) 또는 실시간 프로세서(Real-Time Processor)는 가장 빠른 모터 및 안전 루프를 실행한다. ROS2 인터페이스는 최고 주파수의 서보 루프(Servo Loop)가 DDS 통신에 직접 의존하지 않도록 하면서 검증된 관절 상태, 센서 측정값, 액추에이터 상태, 명령 채널을 제공한다.

로봇 모델(Robot Model)은 전체 스택을 위한 공통 기하학적 정의(Common Geometric Definition)를 제공한다. URDF 또는 Xacro는 베이스(Base), 네 개의 다리 체인(Leg Chain), 관절 제한(Joint Limit), 관성 특성(Inertial Property), 충돌 형상(Collision Geometry), 센서 프레임(Sensor Frame)을 정의한다. 로봇 상태 발행기(robot_state_publisher)는 이 모델과 관절 상태(JointState) 데이터를 결합하여 tf2 트리(Tree)를 생성한다. 앞쪽 왼쪽(Front-Left), 앞쪽 오른쪽(Front-Right), 뒤쪽 왼쪽(Rear-Left), 뒤쪽 오른쪽(Rear-Right)과 같은 일관된 명칭은 URDF, 제어기, 접촉 메시지, 정책 매핑(Policy Mapping), 진단, 시각화 전반에서 동일하게 유지해야 한다.

일반적인 변환 계층(Transform Hierarchy)은 map, odom, base_link를 연결한 다음 개별 다리와 센서 방향으로 분기한다. 정적 변환(Static Transform)은 고정된 센서 장착 관계를 표현하고, 동적 변환(Dynamic Transform)은 관절 움직임과 추정된 로봇 자세를 나타낸다. 상태 추정, 지형 인지(Terrain Perception), 내비게이션, 발 배치(Foot Placement), RViz2가 모두 동일한 물리적 로봇을 기준으로 공간적으로 일관된 데이터에 의존하므로 정확한 타임스탬프와 명확하게 지정된 변환 권한(Transform Authority)이 필수적이다.

상태 추정은 IMU 측정값, 관절 상태, 운동학적 제약조건(Kinematic Constraint), 발 접촉, 선택적인 외부 위치 추정(External Localization)을 결합하여 베이스 자세, 위치, 속도, 환경에 대한 상대 움직임을 추정한다. 입각(Stance) 중에는 신뢰할 수 있는 발 접촉을 몸체 움직임의 유용한 제약조건으로 사용할 수 있지만, 미끄러지거나 불확실한 접촉은 다르게 처리해야 한다. 추정기는 일관된 로봇 상태를 발행하며, 이는 보행 생성(Gait Generation), 전신 제어(Whole-Body Control), 내비게이션, 학습 정책(Learned Policy)의 핵심 입력이 된다.

발 접촉 처리(Foot-Contact Processing)는 원시 센싱(Raw Sensing)과 보행 로직(Locomotion Logic) 사이에서 독립적인 하위 시스템을 구성한다. 힘 센서(Force Sensor), 모터 전류(Motor Current), 관절 동역학(Joint Dynamics), 운동학(Kinematics), 학습 기반 추정기(Learned Estimator)가 접촉 감지에 사용될 수 있다. 결과 인터페이스는 각 발의 접촉 상태, 신뢰도(Confidence), 힘, 위치, 필요한 경우 미끄러짐 정보를 식별한다. 예상 접촉과 측정 접촉을 비교하면 조기 착지(Early Touchdown), 지연 접촉(Delayed Contact), 충격(Impact), 지지 손실(Loss of Support)에 대응할 수 있다.

보행 제어 계층(Gait-Control Layer)은 목표 로봇 움직임을 네 다리의 협조된 동작으로 변환한다. 속도 명령(Velocity Command), 몸체 자세 요청(Body Posture Request), 보행 모드(Gait Mode), 지형 파라미터(Terrain Parameter), 상태 추정값이 보행 제어기로 입력되며, 제어기는 각 다리의 입각 및 유각(Swing) 위상을 관리한다. 걷기(Walking), 트로트(Trotting), 페이스(Pacing), 바운드(Bounding), 크롤(Crawling), 정지(Standing), 복구(Recovery)는 동일한 감독 인터페이스(Supervisory Interface)를 공유하면서 서로 다른 내부 알고리즘을 통해 타이밍, 착지점(Foothold), 몸체 움직임 목표를 생성할 수 있다.

유각 궤적 생성(Swing Trajectory Generation)은 현재 발 위치에서 다음 착지점까지 스텝 높이(Step Height), 지형 여유 높이(Terrain Clearance), 속도, 접촉 이벤트를 고려하여 부드러운 기준 궤적을 생성한다. 입각 동작(Stance Behavior)은 지지 제약조건(Support Constraint)을 정의하고 몸체 안정화(Body Stabilization)에 기여한다. 예상하지 못한 지형 상호작용은 계획된 궤적이나 보행 위상을 변경할 수 있다. 이를 통해 4족보행 로봇은 최초 접촉 스케줄을 변경할 수 없는 고정 시퀀스로 취급하지 않고 보행을 지속적으로 적응시킬 수 있다.

전신 제어기(Whole-Body Controller)는 보행 목표를 전체 기구(Mechanism)에 대해 동역학적으로 일관된 명령(Dynamically Consistent Command)으로 변환한다. 목표 몸체 움직임, 발 궤적, 접촉 제약조건, 힘 분배(Force Distribution), 관절 제한, 액추에이터 성능을 결합하여 관절 수준 기준값(Joint-Level Reference)을 계산할 수 있다. 로봇에 따라 이러한 출력은 관절 위치, 속도, 가속도 또는 토크가 될 수 있다. ros2_control은 이 계층과 하드웨어 사이에 표준화된 상태 및 명령 인터페이스를 제공할 수 있다.

전체 스택은 감독 수준 ROS2 처리(Supervisory ROS2 Processing)와 결정론적 제어 실행(Deterministic Control Execution) 사이에 엄격한 경계를 유지해야 한다. 내비게이션, 인지, 시각화, 로깅(Logging), 구성(Configuration)은 일반적인 ROS2 노드로 실행할 수 있지만, 고주파 보행, 전신 제어, 액추에이터 루프는 실시간 안전 실행 패턴(Real-Time-Safe Execution Pattern)을 사용해야 한다. 공유 버퍼(Shared Buffer), 사전 할당 메모리(Preallocated Memory), 제한된 연산(Bounded Computation), 스레드 우선순위(Thread Priority), 신중하게 설계된 실행기(Executor) 구조를 통해 비핵심 ROS2 작업이 보행 타이밍을 방해하는 것을 방지할 수 있다.

인지 기능은 고유감각 센싱(Proprioceptive Sensing)을 넘어 주변 환경에 대한 정보를 제공한다. 카메라와 라이다는 포인트 클라우드(Point Cloud), 깊이 정보(Depth Information), 장애물 표현(Obstacle Representation), 의미론적 관측(Semantic Observation), 고도 지도(Elevation Map)를 생성할 수 있다. 지형 처리(Terrain Processing)는 이러한 측정값을 표면 높이, 경사(Slope), 주행 가능성(Traversability), 후보 착지점(Candidate Foothold) 등 보행에 유용한 구조로 변환한다. 이후 보행 또는 발걸음 플래너(Footstep Planner)는 지역 지형에 따라 발 배치와 몸체 움직임을 조정할 수 있다.

내비게이션은 보행 계층보다 상위에서 동작하며 개별 관절이나 발이 아니라 목표 움직임을 명령해야 한다. 내비게이션 구성요소는 지도, 목표, 장애물, 로봇 성능을 기반으로 전역 경로(Global Path)와 지역 속도 기준(Local Velocity Reference)을 생성할 수 있다. 보행 스택은 이러한 기준값을 물리적으로 실현 가능한 4족보행 움직임으로 변환한다. 안정성, 지형 난이도(Terrain Difficulty), 속도 제한, 보행 고장에 대한 피드백을 상위로 반환하여 내비게이션이 동작을 조정하도록 할 수 있다.

강화학습 정책(Reinforcement-Learning Policy)은 주변 ROS2 아키텍처를 교체하지 않고 동일한 스택 내부에 삽입할 수 있다. 정책 추론 노드(Policy Inference Node)는 학습 모델을 실행하기 전에 JointState, IMU, 접촉, 상태 추정, 명령, 지형 정보로부터 결정론적인 관측값(Deterministic Observation)을 구성한다. 출력은 보행 파라미터를 수정하거나 발 목표(Foot Target)를 생성하거나 관절 수준 행동(Joint-Level Action)을 제공할 수 있으며, 독립적인 검증 및 안전 계층이 정책 출력을 제한한 후 하드웨어로 전달한다.

안전 감독(Safety Supervision)은 모터 제어기 내부에만 존재하는 것이 아니라 전체 스택에 걸쳐 적용되어야 한다. 관절 제한, 토크 및 속도 제약조건, 몸체 자세, 접촉 이상(Contact Abnormality), 명령 타임아웃(Command Timeout), 통신 상태, 액추에이터 고장, 열 상태(Thermal Condition), 넘어짐 감지(Fall Detection)가 안전 판단에 사용될 수 있다. 심각도에 따라 시스템은 속도를 줄이고, 정지 자세로 전환하고, 제어기를 변경하거나, 보행을 비활성화하거나, 독립적으로 보호된 경로를 통해 비상 정지(Emergency Stop)를 실행할 수 있다.

생명주기 관리(Lifecycle Management)는 노드 간 시작 및 종료 의존성(Startup and Shutdown Dependency)을 조정한다. 관절 상태 처리, 변환, 상태 추정, 보행 제어, 정책 추론이 활성화되기 전에 하드웨어 인터페이스가 정상 상태가 되어야 한다. 필요한 센서, 변환, 상태 추정, 접촉, 액추에이터 인터페이스가 정의된 검사를 통과한 경우에만 보행을 시작해야 한다. 제어기를 갑자기 중단하면 서 있는 로봇의 안정성을 잃을 수 있으므로 통제된 비활성화(Controlled Deactivation)와 복구 시퀀스(Recovery Sequence)도 동일하게 중요하다.

ROS2 서비스 품질(Quality of Service, QoS)은 모든 데이터에 동일하게 적용하는 것이 아니라 데이터의 의미에 따라 선택해야 한다. 고주파 JointState, IMU, 접촉, 인지 스트림은 일반적으로 데이터 최신성(Freshness)과 제한된 큐(Bounded Queue)를 중요하게 처리하는 반면, 구성 변경, 생명주기 이벤트(Lifecycle Event), 중요한 모드 전환은 신뢰성 높은 전달(Reliable Delivery)이 필요할 수 있다. 네트워크 구성 또는 통신 정책을 통해 대용량 센서 트래픽과 시간 민감형 제어 정보(Time-Sensitive Control Information)를 분리하면 시스템 동작의 예측 가능성을 향상시킬 수 있다.

진단은 개별 노드의 상태만 확인하는 것이 아니라 전체 4족보행 로봇을 통합적으로 파악할 수 있어야 한다. 유용한 정보에는 제어 주파수(Control Frequency), 메시지 경과 시간(Message Age), 추정기 상태(Estimator Status), 변환 가용성(Transform Availability), 접촉 일관성(Contact Consistency), 보행 위상, 추종 오차(Tracking Error), 액추에이터 포화(Actuator Saturation), 정책 지연(Policy Latency), 센서 상태, CPU 또는 GPU 부하, 통신 상태, 안전 이벤트가 포함된다. 타임스탬프 기반 진단과 rosbag 기록을 사용하면 현장 시험 이후 여러 계층에 걸쳐 발생한 고장을 재구성할 수 있다.

RViz2는 전체 스택의 공간적 관측 가능성 계층(Spatial Observability Layer)을 제공한다. URDF 모델, tf2 프레임, 관절 구성, 발 접촉, 힘 벡터(Force Vector), 보행 위상, 계획 착지점, 몸체 궤적, 지형 포인트 클라우드, 고도 지도, 내비게이션 경로, 정책 기반 목표(Policy-Derived Target)를 하나의 환경에서 시각화할 수 있다. 시각화 복잡성이나 렌더링 부하(Rendering Load)가 보행 데드라인에 영향을 주지 않도록 RViz2는 실시간 제어 경로 외부에 유지해야 한다.

시뮬레이션은 실제 하드웨어에서 사용하는 것과 동일한 ROS2 인터페이스를 재현해야 한다. Gazebo, Isaac Sim, MuJoCo 또는 다른 시뮬레이터는 호환 가능한 관절, IMU, 접촉, 인지, 변환 정보를 제공하면서 동일한 보행 명령을 수신할 수 있다. 인터페이스 동등성(Interface Equivalence)을 유지하면 소프트웨어 인 더 루프(Software-in-the-Loop) 및 하드웨어 인 더 루프(Hardware-in-the-Loop) 시험을 지원할 수 있으며, 전통적인 제어기와 학습 정책을 아키텍처별 변경을 최소화하면서 실제 배포 단계로 발전시킬 수 있다.

배포 구성(Deployment Configuration)은 단순히 실행 가능한 노드 이름만 정의해서는 안 된다. 로봇 모델 버전, 제어기 파라미터, 관절 매핑, 센서 캘리브레이션(Sensor Calibration), QoS 프로파일(QoS Profile), 정책 버전, 안전 제한, 네임스페이스(Namespace), 프레임 정의, 네트워크 설정, 실행 의존성(Launch Dependency)을 통제된 구성 자산(Controlled Configuration Artifact)으로 관리해야 한다. 재현 가능한 실행 구조(Reproducible Launch Structure)를 구축하면 개발 컴퓨터, 엣지 시스템(Edge System), 시뮬레이션, 실제 로봇에 동일한 4족보행 소프트웨어 스택을 일관되게 배포할 수 있다.

최종 아키텍처는 센싱(Sensing)이 로봇과 환경을 표현하고, 상태 추정이 일관된 물리적 상태를 생성하며, 지능(Intelligence)이 움직임을 선택하고, 보행이 의도를 접촉 인지 행동(Contact-Aware Behavior)으로 변환하며, 제어가 액추에이터 명령을 생성하고, 실제 로봇이 다음 관측값을 만들어 내는 폐루프 피지컬 AI(Closed Physical AI Loop)를 형성한다. ROS2는 이 루프를 둘러싼 통합 및 관측 가능성 프레임워크(Integration and Observability Framework)를 제공하며, 결정론적 제어와 독립적인 안전 메커니즘(Independent Safety Mechanism)은 실제 물리적 실행(Physical Execution)을 보호한다.

## 11.10 Humanoid ROS2 Stack Configuration Case

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

완전한 휴머노이드 ROS2 스택(Humanoid ROS2 Stack)은 통합된 아키텍처 내에서 인지(Perception), 상태 추정(State Estimation), 균형 제어(Balance Control), 보행(Walking), 전신 움직임(Whole-Body Motion), 조작(Manipulation), 하드웨어 제어(Hardware Control), 안전(Safety), 시스템 감독(System Supervision)을 조정해야 한다. 4족보행 로봇(Quadruped)과 비교하면 휴머노이드는 더 높은 자유도(Degrees of Freedom), 더 작은 지지 영역(Support Region), 빈번한 단일 지지(Single Support)와 이중 지지(Double Support) 전환, 보행과 상체 움직임 사이의 강한 결합을 가지므로 인터페이스 일관성과 타이밍 관리가 특히 중요하다.

하드웨어 계층(Hardware Layer)은 일반적으로 다리, 몸통(Torso), 팔, 목, 손을 위한 구동 관절(Actuated Joint)과 함께 엔코더(Encoder), 모터 드라이브(Motor Drive), IMU, 발 힘 또는 토크 센서(Foot Force or Torque Sensor), 카메라(Camera), 깊이 센서(Depth Sensor), 라이다(LiDAR)를 포함한다. 임베디드 제어기(Embedded Controller)는 고주파 모터 전류, 토크, 위치 및 보호 루프(Protection Loop)를 실행한다. ROS2는 이러한 결정론적 루프 상위에서 검증된 상태 및 명령 인터페이스를 제공하여 미들웨어 타이밍 변동성이 액추에이터 안정성을 직접 결정하지 않도록 한다.

URDF 또는 Xacro는 휴머노이드의 공통 기하학 및 운동학적 모델(Geometric and Kinematic Description)을 제공한다. 모델은 링크(Link), 관절(Joint), 관성 특성(Inertial Property), 충돌 형상(Collision Geometry), 관절 제한(Joint Limit), 트랜스미션(Transmission), 센서 장착 프레임(Sensor Mounting Frame)을 정의한다. 로봇 상태 발행기(robot_state_publisher)는 모델과 관절 상태(JointState) 정보를 결합하여 tf2 변환을 생성한다. 일관된 관절 명칭과 순서는 로봇 모델, 제어기, 상태 추정, 움직임 계획(Motion Planning), 학습 정책(Learned Policy), 진단(Diagnostics), 시각화 전반에서 유지되어야 한다.

변환 아키텍처(Transform Architecture)는 일반적으로 map, odom, base 또는 pelvis 프레임을 연결한 후 왼쪽과 오른쪽 다리, 몸통, 팔, 머리, 센서 프레임으로 분기한다. 발 및 손 프레임은 환경과의 물리적 상호작용 지점(Physical Interaction Point)을 정의하기 때문에 특히 중요하다. 정확한 타임스탬프, 보정된 정적 변환(Calibrated Static Transform), 명확하게 지정된 변환 권한(Transform Authority)을 통해 보행, 조작, 인지, 시각화 구성요소가 동일한 로봇 구성을 일관되게 해석할 수 있다.

휴머노이드 상태 추정(Humanoid State Estimation)은 IMU 측정값, 관절 상태, 운동학(Kinematics), 발 접촉(Foot Contact), 선택적으로 비전(Visual), 라이다 또는 외부 위치 추정(External Localization)을 결합한다. 추정기는 골반(Pelvis) 또는 베이스의 자세, 위치, 선속도 및 각속도, 로봇과 환경 사이의 관계를 결정한다. 보행 중에는 왼발 지지, 오른발 지지, 이중 지지 상태 사이의 전환을 추정기가 정확하게 해석해야 하므로 접촉 정보가 특히 중요하다.

발 접촉 처리(Foot-Contact Processing)는 힘 또는 토크 측정값, 압력 정보(Pressure Information), 모터 신호, 운동학적 정보를 안정적인 접촉 상태(Contact State)로 변환한다. 시스템은 예상된 지지(Expected Support)와 불확실하거나 미끄러지는 접촉을 구분하고, 가능한 경우 접촉력(Contact Force), 압력 중심(Center of Pressure), 접촉 신뢰도(Contact Confidence)를 추정해야 한다. 이러한 출력은 균형 추정(Balance Estimation), 보행 위상 전환(Walking-Phase Transition), 전신 제어(Whole-Body Control), 넘어짐 방지(Fall Prevention), 불규칙한 지면에 대한 적응을 지원한다.

균형 제어는 질량 중심(Center of Mass)이 사용 가능한 지지 영역과 동역학적으로 일관된 상태를 유지해야 하기 때문에 휴머노이드 스택의 핵심 계층이다. 제어기는 질량 중심, 영 모멘트 점(Zero Moment Point), 캡처 포인트(Capture Point), 발산 운동 성분(Divergent Component of Motion), 중심 운동량(Centroidal Momentum) 또는 관련 표현을 사용할 수 있다. ROS2는 기준값과 진단 정보를 전달할 수 있지만 시간 민감도가 높은 균형 계산은 결정론적 제어 프로세스(Deterministic Control Process)에서 실행해야 한다.

보행 제어기(Walking Controller)는 목표 속도, 방향, 몸체 자세, 내비게이션 명령을 협조된 스텝 동작(Coordinated Stepping Behavior)으로 변환한다. 제어기는 서기(Standing), 보행 시작(Walking Initiation), 단일 지지, 이중 지지, 정지(Stopping), 회전(Turning), 복구(Recovery) 전환을 관리한다. 발걸음 타이밍(Footstep Timing)과 배치는 균형 제약조건과 현재 접촉 상태에 일관되어야 한다. 따라서 보행 제어기는 보행을 서로 독립적인 다리 궤적의 연속이 아니라 제어된 상태 기계(Controlled State Machine)로 다루어야 한다.

발걸음 플래너(Footstep Planner)는 명령된 움직임, 운동학적 도달 가능성(Kinematic Reachability), 지형 형상(Terrain Geometry), 충돌 제약조건(Collision Constraint), 균형 요구사항을 기반으로 미래의 지지 위치를 결정한다. 평지에서는 규칙적인 스텝 패턴으로 충분할 수 있지만 계단, 경사면, 장애물, 불규칙 지면에서는 인지 기반 배치(Perception-Aware Placement)가 필요하다. 계획된 발걸음은 적절한 전역 또는 지역 프레임으로 표현하여 명확하게 정의된 ROS2 인터페이스를 통해 궤적 및 균형 구성요소에 전달할 수 있다.

유각 발 궤적 생성(Swing-Foot Trajectory Generation)은 계획된 각 스텝을 현재 지지 구성에서 목표 착지점(Target Foothold)까지의 부드러운 움직임으로 변환한다. 궤적은 관절 제한, 속도 제약조건, 충돌 회피(Collision Avoidance)를 만족하면서 충분한 지면 여유 높이(Ground Clearance)를 확보해야 한다. 조기 착지(Early Touchdown), 지연 접촉(Delayed Contact), 예상하지 못한 지형 높이가 발생하면 실제 물리적 조건과 더 이상 일치하지 않는 기존 궤적을 강제로 실행하지 않고 제어된 적응(Controlled Adaptation)을 수행해야 한다.

전신 제어는 여러 동시 목표를 만족하면서 다리, 골반, 몸통, 팔, 머리를 조정한다. 보행 중 제어기는 질량 중심 움직임, 발 자세(Foot Pose), 몸통 방향(Torso Orientation), 팔 자세(Arm Posture), 접촉 제약조건, 관절 제한을 동시에 추종할 수 있다. 최적화 기반 접근법(Optimization-Based Approach)은 작업 우선순위를 설정하고 활성 접촉 사이에 힘을 분배할 수 있다. 이 계층은 상위 움직임 목표를 동역학적으로 일관된 관절 가속도, 위치, 속도 또는 토크로 변환한다.

상체 제어(Upper-Body Control)는 보행과 완전히 독립된 기능으로 취급하기보다 보행과 통합해야 한다. 팔 움직임은 보행 중 각운동량(Angular Momentum)을 보상하거나 운반 물체를 안정화하고, 조작을 수행하거나 의도적으로 전신 동역학을 변화시킬 수 있다. 조작 작업은 지지 구조를 확대하거나 변경하는 손 접촉(Hand Contact)을 추가할 수도 있다. 통합된 전신 인터페이스(Unified Whole-Body Interface)를 사용하면 보행과 조작 목표가 서로 충돌하는 관절 명령을 생성하지 않으면서 로봇 자원을 공유할 수 있다.

ros2_control은 전신 제어와 액추에이터 하드웨어 사이에 표준화된 하드웨어 상태 및 명령 인터페이스를 제공할 수 있다. 서로 다른 관절 그룹은 기계적 설계와 제어기 요구사항에 따라 위치, 속도 또는 토크 인터페이스를 사용할 수 있다. 최고 주파수의 서보 루프는 전용 실시간 하드웨어에 유지하면서 ROS2는 적절한 주기에서 제어기 구성, 상태 발행, 모드 변경, 진단, 감독 수준 명령 흐름(Supervisory Command Flow)을 관리할 수 있다.

인지 기능은 보행과 조작 모두를 지원한다. 카메라, 깊이 센서, 라이다는 포인트 클라우드(Point Cloud), 장애물 표현(Obstacle Representation), 의미론적 정보(Semantic Information), 지형 지도(Terrain Map), 객체 자세(Object Pose), 사람 관련 관측(Human-Related Observation)을 생성할 수 있다. 보행 구성요소는 이 정보를 사용하여 주행 가능한 영역과 발걸음을 결정하고, 조작 구성요소는 객체와 상호작용 목표를 찾는 데 사용한다. 공유 공간 프레임(Shared Spatial Frame)을 사용하면 두 기능이 동일한 환경을 기준으로 추론할 수 있다.

내비게이션(Navigation)은 휴머노이드 관절을 직접 제어하는 대신 실현 가능한 몸체 움직임 또는 보행 목표를 명령해야 한다. 전역 계획(Global Planning)은 환경을 통과하는 경로를 결정하고, 지역 계획(Local Planning)은 장애물, 지형, 가능한 보행 방향, 로봇의 안정성 제한을 고려한다. 보행 및 발걸음 계층은 이러한 의도를 동역학적으로 실현 가능한 스텝으로 변환한다. 균형 상태, 차단된 착지점(Blocked Foothold), 복구 상태, 이동성 감소(Reduced Mobility)에 대한 피드백을 통해 내비게이션 동작을 조정할 수 있다.

학습 기반 정책(Learning-Based Policy)은 여러 계층에서 기존 휴머노이드 제어를 보완할 수 있다. 강화학습 정책(RL Policy)은 동기화된 관측값으로부터 보행 행동, 잔차 보정(Residual Correction), 발걸음 조정, 전신 기준값, 복구 동작을 생성할 수 있다. ROS2 추론 노드(Inference Node)는 관측값 구성, 정규화(Normalization), 정책 실행, 행동 검증(Action Validation)을 관리하며, 결정론적 제어기와 독립적인 안전 메커니즘이 학습된 출력을 제한한 후 실제 액추에이터에 영향을 주도록 해야 한다.

안전 감독(Safety Supervision)은 높은 자유도를 가진 키가 큰 기구가 균형을 잃었을 때 발생할 수 있는 결과를 고려해야 한다. 관절 및 토크 제한, 몸체 자세, 지지 상태(Support State), 질량 중심 동작, 액추에이터 고장, 통신 타임아웃(Communication Timeout), 열 상태(Thermal Condition), 충돌 정보(Collision Information), 넘어짐 감지를 지속적으로 평가해야 한다. 대응 방식에는 움직임 감소, 지지 영역 확대, 정지, 보호 자세(Protective Posture) 진입, 제어기 전환, 비상 종료(Emergency Shutdown)가 포함될 수 있다.

생명주기 관리(Lifecycle Management)는 휴머노이드 움직임이 안전하게 시작되기 위해 필요한 많은 의존성을 조정한다. 보행이 활성화되기 전에 하드웨어 인터페이스, 로봇 모델, 변환, 센서, 상태 추정, 접촉 감지, 균형 제어, 액추에이터 제어기가 정상 상태가 되어야 한다. 조작 또는 학습 정책 노드는 추가적인 의존성을 가질 수 있다. 통제된 활성화(Activation), 비활성화(Deactivation), 고장 처리(Fault Handling), 복구를 통해 부분적으로 초기화된 구성요소가 물리적으로 일관되지 않은 명령을 생성하는 것을 방지한다.

실시간 아키텍처(Real-Time Architecture)는 결정론적인 균형, 전신 및 액추에이터 제어 루프를 시각화, 로깅(Logging), 인지 처리 및 기타 가변적인 작업 부하(Variable Workload)로부터 격리해야 한다. 실시간 안전 버퍼(Real-Time-Safe Buffer), 사전 할당 메모리(Preallocated Memory), 제한된 연산(Bounded Computation), CPU 스케줄링, 신중하게 분리된 실행기(Executor)를 통해 타이밍 간섭을 줄일 수 있다. ROS2는 통합 프레임워크로 유지되지만 제어 안정성이 예측 불가능한 콜백 실행이나 대규모 센서 메시지 처리에 의존해서는 안 된다.

진단과 RViz2는 상호 보완적인 관측 가능성(Observability)을 제공한다. 수치 진단은 제어 주파수, 접촉 상태, 균형 여유도(Balance Margin), 추종 오차(Tracking Error), 액추에이터 포화(Actuator Saturation), 추론 지연(Inference Latency), 센서 상태, 안전 이벤트를 보고할 수 있다. 동시에 RViz2는 휴머노이드 모델, tf2 프레임, 지지 발, 질량 중심 궤적(Center-of-Mass Trajectory), 계획된 발걸음, 유각 궤적, 인지 데이터, 내비게이션 경로, 조작 목표를 하나의 공통 공간적 맥락(Spatial Context)에서 표시할 수 있다.

시뮬레이션은 실제 휴머노이드에서 사용하는 것과 동일한 ROS2 인터페이스를 재현해야 한다. Gazebo, Isaac Sim, MuJoCo 또는 다른 동역학 환경(Dynamics Environment)은 동일한 감독 수준 명령을 수신하면서 호환 가능한 관절, IMU, 접촉, 인지, 변환 데이터를 제공할 수 있다. 소프트웨어 인 더 루프(Software-in-the-Loop) 및 하드웨어 인 더 루프(Hardware-in-the-Loop) 시험을 통해 위험성이 높은 실제 실험에 앞서 보행, 균형, 전신 제어, 학습 정책, 생명주기 전환, 고장 처리를 검증할 수 있다.

완전한 휴머노이드 ROS2 구성은 궁극적으로 인지가 환경을 해석하고, 상태 추정이 물리적 상태를 재구성하며, 계획(Planning)이 발걸음과 작업을 선택하고, 균형 및 전신 지능(Whole-Body Intelligence)이 전체 기구를 조정하며, 결정론적 제어가 안전한 액추에이터 명령을 실행하는 폐루프 피지컬 AI 시스템(Closed Physical AI System)을 형성한다. ROS2는 명시적인 인터페이스를 통해 이러한 기능을 연결하며, 관측 가능성, 생명주기 관리, 시뮬레이션, 독립적인 안전 메커니즘을 통해 자율 보행(Autonomous Walking)에서 통합 이동 조작(Integrated Mobile Manipulation)으로의 신뢰성 있는 발전을 지원한다.
