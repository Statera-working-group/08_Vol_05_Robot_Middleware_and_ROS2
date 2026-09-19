**Volume 05 Robot Middleware and ROS2**

# 07. ROS2 Lifecycle Management

## 07.01 Lifecycle Node State Transition Deep Dive [w/Code]

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 수명주기 노드(Lifecycle Node)는 일반적인 노드(Node) 모델에 명시적으로 관리되는 상태 머신(Managed State Machine)을 추가하여, 소프트웨어 구성요소가 자원 생성(Resource Construction), 구성(Configuration), 실행(Execution), 일시 중지(Suspension), 정리(Cleanup), 종료(Shutdown)를 서로 분리할 수 있도록 한다. 이러한 모델은 센서, 제어기(Controller), 위치추정(Localization) 모듈, 인공지능(AI) 구성요소가 단순히 프로세스가 존재한다는 이유만으로 즉시 동작해서는 안 되는 로봇 시스템에서 특히 중요하다.

수명주기 모델(Lifecycle Model)은 기본 상태(Primary State)와 이들 상태 사이의 전이(Transition)를 구분한다. 주요 안정 상태(Stable State)는 미구성(Unconfigured), 비활성(Inactive), 활성(Active), 최종화(Finalized)이다. 노드는 일반적으로 프로세스는 존재하지만 운용 자원이 아직 완전히 준비되지 않은 미구성(Unconfigured) 상태에서 시작한다. 수명주기 전이(Lifecycle Transition)는 애플리케이션에서 정의한 콜백(Callback)을 호출하면서 노드를 이러한 상태 사이에서 의도적으로 이동시킨다.

구성(Configuration)은 노드를 구성 중(Configuring) 전이를 통해 미구성(Unconfigured) 상태에서 비활성(Inactive) 상태로 이동시킨다. 이 과정에서 노드는 파라미터(Parameter)를 로드하고, 버퍼(Buffer)를 할당하며, 알고리즘을 초기화하고, 장치 인터페이스(Device Interface)를 설정하고, 수명주기 인식 발행자(Lifecycle-aware Publisher)를 생성하거나 구성 파일(Configuration File)을 검증할 수 있다. 성공하면 비활성(Inactive) 상태에 도달하며, 구성 실패 시에는 불완전한 구성요소가 조용히 실행되는 대신 정의된 오류 처리(Error Handling) 동작으로 전환될 수 있다.

비활성(Inactive) 상태는 구성은 완료되었지만 실제 운용은 수행하지 않는 구성요소를 나타낸다. 동작에 필요한 자원이 이미 존재할 수 있지만 노드는 정상적인 런타임 기능(Runtime Function)을 수행해서는 안 된다. 이러한 분리는 전체 서브시스템(Subsystem)을 데이터 생성이나 액추에이터(Actuator) 관련 처리를 시작하기 전에 구성하고 검사할 수 있기 때문에 결정론적 로봇 기동(Deterministic Robot Startup)에 중요하다. 따라서 여러 종속 노드(Dependent Node)를 먼저 준비한 후 일관되게 활성화할 수 있다.

활성화(Activation)는 활성화 중(Activating) 전이를 통해 비활성(Inactive) 노드를 활성(Active) 상태로 변경한다. 관련 콜백은 일반적으로 수명주기 발행자(Lifecycle Publisher), 타이머(Timer), 처리 파이프라인(Processing Pipeline), 하드웨어 연계 동작(Hardware-facing Behavior)과 같이 런타임 동안에만 동작해야 하는 자원을 활성화하는 데 사용된다. 활성화가 성공하면 노드는 본래의 운용 역할을 수행하고 구현된 방식에 따라 ROS 2 통신 아키텍처(Communication Architecture)에 정상적으로 참여한다.

비활성화(Deactivation)는 구성요소를 완전히 제거하지 않고 활성(Active) 상태에서 비활성(Inactive) 상태로 되돌리는 경로를 제공한다. 노드는 빠른 재활성화(Reactivation)에 필요한 구성과 자원을 유지하면서 운용 동작만 중단할 수 있다. 로봇에서는 이것이 프로세스를 종료하는 것과 크게 다르다. 위치추정(Localization), 인지(Perception), 내비게이션(Navigation), 장치 노드(Device Node)는 구조적으로 유지된 상태에서 능동적인 동작만 일시적으로 중지하여 제어된 복구(Controlled Recovery)를 수행할 수 있다.

정리(Cleanup)는 비활성(Inactive) 노드를 다시 미구성(Unconfigured) 상태로 이동시킨다. 정리 콜백(Cleanup Callback)은 구성 과정에서 생성된 자원을 해제하고 구성요소를 다시 구성할 수 있는 상태로 되돌려야 한다. 이를 통해 운영체제 프로세스(Operating-system Process)를 반드시 재시작하지 않고도 구성 변경(Configuration Change), 하드웨어 재연결(Hardware Reconnection), 제어된 서브시스템 재설정(Controlled Subsystem Reset), 반복적인 초기화(Initialization)를 수행할 수 있으므로 수명주기 관리는 장시간 운용되는 로봇 플랫폼에 유용하다.

종료 전이(Shutdown Transition)는 수명주기 동작을 종료하고 최종적으로 노드를 최종화(Finalized) 상태에 배치한다. 최종화(Finalized)는 단순히 프로세스를 강제로 종료하는 것과 개념적으로 다르다. 전이 과정에서 애플리케이션 로직(Application Logic)이 제어된 종료 작업(Controlled Shutdown Activity)을 수행할 기회를 제공하기 때문이다. 주변 프로세스가 최종적으로 제거되거나 재시작되기 전에 하드웨어 명령을 중지하고, 통신 자원을 해제하고, 상태 정보를 보존하며, 진단 이벤트(Diagnostic Event)를 기록할 수 있다.

따라서 수명주기 전이(Lifecycle Transition)는 단순한 상태 라벨(State Label) 이상의 의미를 가진다. 각각의 전이는 소프트웨어가 특정 콜백 로직(Callback Logic)을 실행하고 성공 또는 실패를 보고할 수 있는 제어 경계(Controlled Boundary)를 형성한다. C++에서는 일반적으로 \`rclcpp_lifecycle::LifecycleNode\`를 상속하고 \`on_configure()\`, \`on_activate()\`, \`on_deactivate()\`, \`on_cleanup()\`, \`on_shutdown()\`, \`on_error()\` 등의 콜백을 재정의하여 추상적인 상태 머신(State Machine)을 실제 애플리케이션 동작과 연결한다.

전이 콜백(Transition Callback)은 임의의 초기화 명령을 모아놓은 형태가 아니라 트랜잭션과 유사한 동작(Transaction-like Operation)으로 설계하는 것이 바람직하다. 구성이 부분적으로 성공한 후 실패하는 경우 노드는 모호한 운용 상태(Ambiguous Operational Condition)를 외부에 노출하지 않아야 한다. 실패 이전에 확보된 자원을 주의 깊게 추적해야 하며, 콜백 반환값(Callback Return Value)은 요청된 상태 전이가 완료되었는지, 복구 가능한 방식으로 실패했는지, 또는 복구 불가능한 상황이 발생했는지를 정확하게 나타내야 한다.

수명주기 노드(Lifecycle Node)가 물리적 하드웨어와 상호작용할 때 오류 처리(Error Processing)는 특히 중요해진다. 카메라(Camera)가 열리지 않거나, 라이다(LiDAR)가 응답을 중지하거나, 모터 인터페이스(Motor Interface)가 초기화를 거부하거나, 인공지능 가속기(AI Accelerator)가 필요한 메모리를 할당하지 못할 수 있다. 수명주기 로직(Lifecycle Logic)은 이러한 상황을 탐지하고 무제어 실행을 허용하는 대신 정리(Cleanup), 재구성(Reconfiguration), 외부 복구(External Recovery), 최종화(Finalization) 중 어떤 과정이 뒤따라야 하는지를 결정하기 위한 구조화된 위치를 제공한다.

수명주기 인식 발행자(Lifecycle-aware Publisher)는 구성(Configuration)과 실제 운용(Operation)의 차이를 더욱 명확하게 한다. 발행자(Publisher)는 노드가 구성되는 동안 생성될 수 있지만 노드가 활성(Active) 상태가 되었을 때만 활성화될 수 있다. 비활성화(Deactivation)는 발행자 자체를 제거하지 않고 발행 기능만 중단할 수 있다. 이를 통해 기술적으로 존재하지만 정상적인 로봇 운용에 참여하도록 아직 승인되지 않은 서브시스템의 데이터를 하위 구성요소(Downstream Component)가 유효한 정보로 해석하는 것을 방지할 수 있다.

상태 전이(State Transition)는 외부에서 요청할 수 있으므로 수명주기 노드는 상위 수준 오케스트레이션(Higher-level Orchestration)에 적합하다. 감독기(Supervisor)는 현재 상태를 조회하고, 구성을 요청하고, 종속 구성요소가 준비된 후 노드를 활성화하며, 유지보수 전에 비활성화하고, 복구 과정에서 정리 작업을 시작할 수 있다. 따라서 수명주기 노드는 준비 상태를 간접적으로 추론해야 하는 독립 실행 프로세스가 아니라 명시적으로 관리 가능한 소프트웨어 구성요소(Manageable Software Component)가 된다.

완전한 로봇 시스템에서는 상태 전이(State Transition)가 서브시스템 종속성(Subsystem Dependency)을 반영할 때 수명주기 관리(Lifecycle Management)의 효과가 가장 커진다. 예를 들어 센서 드라이버(Sensor Driver)가 인지(Perception)보다 먼저 활성화되고, 인지가 위치추정(Localization)보다 먼저 활성화되며, 위치추정이 자율 내비게이션(Autonomous Navigation)보다 먼저 활성화되어야 할 수 있다. 종료 과정에서는 반대 순서가 필요할 수 있다. 따라서 이 장의 구조는 개별 수명주기 전이에서 시스템 전체 오케스트레이션(System-wide Orchestration), 종속성 기반 기동(Dependency-based Startup), 전원 시퀀싱(Power Sequencing), 복구(Recovery), 안전 상태 관리(Safe-state Management)로 확장된다.

수명주기 상태(Lifecycle State)를 안전 상태(Safety State)와 혼동해서는 안 된다. 활성(Active), 비활성(Inactive), 최종화(Finalized)는 소프트웨어 관리 상태를 설명하는 반면, 비상 정지(Emergency Stop), 안전 토크 차단(Safe Torque Off), 보호 정지(Protective Stop) 등의 기능 안전(Functional Safety) 상태는 전용 안전 메커니즘(Safety Mechanism)에 속한다. 수명주기 관리는 이러한 메커니즘과 연계될 수 있지만 ROS 2 상태 전이 자체를 위험 에너지를 제거하거나 액추에이터 안전을 보장하는 안전 기능으로 간주해서는 안 된다.

좋은 수명주기 설계(Lifecycle Design)를 위해서는 관측 가능성(Observability)도 필요하다. 전이 요청(Transition Request), 이전 상태와 전이 후 상태, 콜백 실행 시간(Callback Duration), 실패 원인(Failure Reason), 하드웨어 준비 상태(Hardware Readiness), 복구 시도(Recovery Attempt)를 진단 및 로깅 인프라(Diagnostics and Logging Infrastructure)를 통해 확인할 수 있어야 한다. 분산 로봇 시스템에서는 이러한 정보를 통해 운영자와 감독 소프트웨어가 충돌로 종료된 노드와 의도적으로 비활성 상태인 노드, 구성을 기다리는 노드, 장애에서 복구 중인 노드, 종속 구성요소가 준비되지 않아 활성화를 거부한 노드를 구분할 수 있다.

결국 수명주기 전이(Lifecycle Transition)는 구성요소의 존재(Component Existence)와 구성요소의 준비 상태(Component Readiness) 사이에 명확한 관리 계약(Management Contract)을 제공한다. 프로세스 시작이 곧 운용 준비 완료를 의미한다고 가정하는 대신 ROS 2는 준비(Preparation), 활성화(Activation), 일시 중지(Suspension), 재구성(Reconfiguration), 복구(Recovery), 종료(Termination)를 명시적으로 표현할 수 있다. 이러한 특성은 로봇 시스템이 소수의 노드에서 인지(Perception), 제어(Control), 내비게이션(Navigation), 인공지능 추론(AI Inference), 안전 감독(Safety Supervision), 플릿 연계 서비스(Fleet-connected Service)를 포함하는 분산 소프트웨어 스택으로 확장될수록 더욱 중요해진다.

## 07.02 System-Wide Lifecycle Orchestration [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

시스템 전체 수명주기 오케스트레이션(System-wide Lifecycle Orchestration)은 ROS 2 관리형 노드(Managed Node) 개념을 개별 구성요소에서 전체 로봇 소프트웨어 스택의 협조된 운용으로 확장한다. 각 프로세스가 독립적으로 구성되고 활성화되도록 두는 대신, 오케스트레이션 계층(Orchestration Layer)이 여러 노드의 수명주기 상태(Lifecycle State), 전이 요청(Transition Request), 종속성(Dependency), 준비 조건(Readiness Condition), 장애(Failure), 종료 순서(Shutdown Sequence)를 감독한다. 이를 통해 복잡한 분산 로봇 시스템(Distributed Robotic System)에 명시적인 운용 구조를 제공할 수 있다.

로봇은 구성요소들이 임의의 순서로 안전하게 시작할 수 있는 시스템이 아니다. 센서 드라이버(Sensor Driver)는 인지(Perception)가 시작되기 전에 하드웨어 초기화(Hardware Initialization)가 필요할 수 있으며, 위치추정(Localization)은 유효한 센서 스트림(Sensor Stream)과 좌표 변환(Transform)에 의존할 수 있다. 내비게이션(Navigation)은 안정적인 위치추정을 필요로 하고, 모션 제어(Motion Control)는 검증된 내비게이션 출력을 요구할 수 있다. 시스템 전체 오케스트레이션은 이러한 관계를 고정된 시작 지연이나 시간적 가정에 의존하지 않고 제어된 수명주기 종속성(Lifecycle Dependency)으로 변환한다.

오케스트레이션 과정(Orchestration Process)은 일반적으로 관리 대상 노드(Managed Node)를 탐색하거나 등록하고 예상 초기 상태(Expected Initial State)를 결정하는 것으로 시작한다. 이후 각 구성요소의 수명주기 인터페이스(Lifecycle Interface)를 통해 미구성(Unconfigured), 비활성(Inactive), 활성(Active), 최종화(Finalized) 상태를 확인할 수 있다. 오케스트레이터(Orchestrator)는 이러한 상태에 대한 시스템 수준 관점(System-level View)을 유지하면서 현재 어떤 전이가 유효하고 어떤 노드가 종속 구성요소를 기다려야 하는지를 결정한다.

로봇 기동(Robot Startup) 과정에서는 일반적으로 활성화(Activation)보다 구성(Configuration)이 먼저 수행된다. 센서 인터페이스(Sensor Interface), 인지 모듈(Perception Module), 위치추정 서비스(Localization Service), 내비게이션 구성요소(Navigation Component), 제어기(Controller), 진단(Diagnostics), 인공지능 추론 노드(AI Inference Node)를 먼저 미구성(Unconfigured) 상태에서 비활성(Inactive) 상태로 이동시킬 수 있다. 이를 통해 파라미터(Parameter), 메모리(Memory), 하드웨어 인터페이스(Hardware Interface), 통신 엔드포인트(Communication Endpoint), 알고리즘(Algorithm)을 준비하면서 조기 운용 동작을 방지할 수 있다. 따라서 정상 실행을 허용하기 전에 시스템의 준비 상태를 확립할 수 있다.

활성화(Activation)는 이후 종속성을 고려한 순서(Dependency-aware Sequence)에 따라 진행될 수 있다. 카메라(Camera) 또는 라이다(LiDAR) 드라이버가 먼저 활성(Active) 상태가 되고, 그 다음 인지 처리(Perception Processing), 위치추정(Localization), 매핑 또는 내비게이션(Mapping or Navigation), 마지막으로 임무 수준 또는 모션 관련 구성요소(Mission-level or Motion-related Component)가 활성화될 수 있다. 오케스트레이터는 다음 종속 단계로 진행하기 전에 필요한 각 전이를 검증한다. 특정 구성요소의 활성화가 실패하면 하위 종속 노드(Downstream Node)는 잘못된 운용 상태로 진입하지 않고 비활성(Inactive) 상태를 유지할 수 있다.

종속성 모델링(Dependency Modeling)은 단순히 프로세스 실행 순서를 재현하는 것이 아니라 실제 기능적 요구사항(Functional Requirement)을 표현해야 한다. 예를 들어 위치추정 노드(Localization Node)는 활성화된 IMU 및 라이다 소스, 유효한 좌표 변환(Transform), 초기화된 지도(Map)를 필요로 할 수 있다. 내비게이션은 단순히 노드 프로세스가 존재하는지가 아니라 위치추정 품질(Localization Quality)과 비용지도 준비 상태(Costmap Readiness)에 의존할 수 있다. 따라서 효과적인 오케스트레이션은 수명주기 상태 정보와 애플리케이션별 상태 및 준비 조건(Application-specific Health and Readiness Condition)을 결합한다.

시스템 오케스트레이터(System Orchestrator)는 ROS 2 수명주기 서비스(Lifecycle Service)와 이벤트(Event)를 이용하여 전이 정책(Transition Policy)을 구현할 수 있다. 구성(Configure), 활성화(Activate), 비활성화(Deactivate), 정리(Cleanup), 종료(Shutdown) 전이를 요청하면서 응답과 결과 상태를 모니터링할 수 있다. 전이 이벤트(Transition Event)는 상태 변경에 대한 추가적인 가시성을 제공하며 감독 로직(Supervisory Logic)이 시스템 모델을 비동기적으로 갱신할 수 있도록 한다. 완료되지 않는 전이 요청으로 인해 로봇 기동이 무한정 차단되지 않도록 타임아웃(Timeout) 관리도 중요하다.

수명주기 오케스트레이션(Lifecycle Orchestration)은 런타임 복구(Runtime Recovery)를 위한 구조화된 기반도 제공한다. 인지 노드(Perception Node)에서 복구 가능한 하드웨어 또는 처리 장애가 발생하면 시스템은 복구를 시도하기 전에 종속된 위치추정 또는 내비게이션 구성요소를 비활성화할 수 있다. 장애 노드는 정리(Cleanup), 재구성(Reconfiguration), 재활성화(Reactivation)를 거칠 수 있으며, 이후 종속 구성요소도 제어된 순서에 따라 다시 운용 상태로 복귀할 수 있다. 따라서 복구는 통제되지 않은 프로세스 재시작이 아니라 오케스트레이션된 상태 전이 순서(Orchestrated State Sequence)가 된다.

모든 장애가 동일한 복구 전략(Recovery Strategy)을 유발해서는 안 된다. 일시적인 센서 통신 문제는 구성 재시도(Retry Configuration)가 적절할 수 있지만, 반복되는 초기화 실패는 해당 서브시스템을 비활성(Inactive) 상태로 유지하면서 성능 저하 운용(Degraded Operation)을 보고해야 할 수 있다. 중요한 제어기 장애(Critical Controller Failure)는 더 광범위한 시스템 비활성화를 요구할 수 있다. 오케스트레이터는 장애를 분류하고 재시도(Retry), 재시작(Restart), 격리(Isolate), 성능 저하 운용(Degrade), 안전 운용 모드 전환(Safe Operational Mode), 전체 종료(Complete Shutdown) 등의 사전 정의된 정책을 적용해야 한다.

시스템 전체 비활성화(System-wide Deactivation)는 유지보수(Maintenance), 모드 변경(Mode Change), 도킹(Docking), 소프트웨어 업데이트(Software Update), 일시적인 운용 중단(Operational Suspension) 과정에서도 중요하다. 활성 상태의 임무 및 내비게이션 구성요소를 중지하면서 선택된 모니터링 또는 통신 서비스는 계속 유지할 수 있다. 수명주기 노드는 비활성(Inactive) 상태에서 구성된 자원을 유지하기 때문에 매번 모든 프로세스를 제거하고 다시 생성하는 방식보다 로봇의 운용을 더욱 빠르게 재개할 수 있다.

종료(Shutdown)는 일반적으로 종속성 관계를 역순으로 따라야 한다. 상위 수준 임무 실행(Mission Execution)이 내비게이션보다 먼저 중지되고, 내비게이션은 위치추정 및 인지보다 먼저 중지되며, 하드웨어 연계 구성요소(Hardware-facing Component)는 해당 데이터를 사용하는 구성요소가 더 이상 필요로 하지 않을 때 종료될 수 있다. 이를 통해 하위 구성요소가 아직 실행 중인 상황에서 상위 자원이 사라지는 문제를 방지할 수 있다. 같은 원칙을 액추에이터 비활성화(Actuator Disabling), 로그 기록 완료(Log Flushing), 영구 상태 저장(Persistent-state Storage), 통신 종료(Communication Termination), 최종 하드웨어 해제(Final Hardware Release)에도 적용할 수 있다.

오케스트레이터 자체도 중요한 감독 구성요소(Supervisory Component)가 되므로 신중한 아키텍처 설계가 필요하다. 중앙집중형 오케스트레이션(Centralized Orchestration)은 명확한 전역 상태 모델(Global State Model)과 단순한 종속성 관리를 제공하며, 계층형 오케스트레이션(Hierarchical Orchestration)은 인지, 내비게이션, 제어, 인공지능, 하드웨어 서브시스템 사이에 책임을 분배할 수 있다. 대규모 로봇은 모든 수명주기 결정을 하나의 거대한 관리자에 집중시키기보다 상위 감독기(Higher-level Supervisor)가 조정하는 지역 서브시스템 관리자(Local Subsystem Manager)를 사용하는 것이 효과적일 수 있다.

시스템 상태(System State)를 단일 노드의 상태만으로 표현해서는 안 된다. 하나의 로봇에는 수십 개의 수명주기 구성요소가 존재할 수 있으며, 이들의 상태 조합은 기동 중(Booting), 준비 완료(Ready), 운용 중(Operational), 성능 저하(Degraded), 유지보수(Maintenance), 복구 중(Recovering), 종료 중(Shutting Down)과 같은 조건에 대응할 수 있다. 이러한 시스템 수준 조건(System-level Condition)은 여러 노드 상태와 상태 지표(Health Indicator)를 종합하여 도출할 수 있다. 따라서 오케스트레이터는 분산된 구성요소 상태를 임무 관리(Mission Management)와 운영자가 이해할 수 있는 운용 상태로 변환한다.

관측 가능성(Observability)은 오케스트레이션 동작을 진단하는 데 필수적이다. 로그(Log)와 진단(Diagnostics)에는 요청된 전이, 전이 요청의 출처, 이전 및 결과 상태, 콜백 실행 시간(Callback Duration), 타임아웃 이벤트(Timeout Event), 종속성 판단(Dependency Decision), 재시도 횟수(Retry Count), 장애 원인(Failure Cause)이 기록되어야 한다. 타임스탬프 기반 전이 이력(Timestamped Transition History)을 사용하면 로봇이 정상 운용 상태에 도달하지 못한 이유 또는 특정 구성요소의 장애 이후 하위 서브시스템이 자동으로 비활성화된 이유를 재구성할 수 있다.

동시성(Concurrency) 역시 의도적으로 관리해야 한다. 종속성 그래프(Dependency Graph)에서 서로 독립적인 분기는 기동 시간을 단축하기 위해 병렬로 구성하거나 활성화할 수 있지만, 서로 종속된 노드는 순서 제약(Ordering Constraint)을 유지해야 한다. 따라서 병렬 전이(Parallel Transition)는 모든 구성요소를 동시에 실행하는 방식이 아니라 명시적인 종속성 분석(Dependency Analysis)을 기반으로 수행되어야 한다. 이러한 접근 방식은 기동 성능과 결정론적 동작(Deterministic Behavior)의 균형을 제공하며 로봇 소프트웨어 스택이 대규모로 확장될수록 더욱 중요해진다.

수명주기 오케스트레이션은 기능 안전 메커니즘(Functional Safety Mechanism)과 연계되어야 하지만 서로 구분되어야 한다. 오케스트레이터는 위험 상태가 보고되었을 때 소프트웨어 구성요소를 비활성(Inactive) 또는 종료(Shutdown) 상태로 전환하도록 명령할 수 있지만, 수명주기 전이는 인증된 비상 정지(Emergency Stop), 안전 토크 차단(Safe Torque Off), 보호 정지(Protective Stop), 안전 제어기(Safety Controller) 기능을 대체하지 않는다. 안전 메커니즘은 ROS 2 수명주기 관리 계층과 독립적으로 제어 권한을 유지해야 한다.

다중 로봇 및 플릿 환경(Multi-robot and Fleet Environment)에서는 동일한 원칙을 단일 로봇을 넘어 확장할 수 있다. 각 로봇은 자체적인 지역 수명주기 오케스트레이터(Local Lifecycle Orchestrator)를 유지하면서 준비 상태(Readiness), 성능 저하 상태(Degraded Status), 유지보수 상태(Maintenance State), 복구 진행 상황(Recovery Progress)을 요약하여 플릿 관리 시스템(Fleet Management)에 제공할 수 있다. 그러면 플릿 소프트웨어는 모든 내부 노드 전이를 직접 제어하지 않으면서도 필요한 서브시스템을 사용할 수 없는 로봇에 임무를 할당하는 것을 방지할 수 있다.

결국 시스템 전체 수명주기 오케스트레이션(System-wide Lifecycle Orchestration)은 ROS 2 프로세스의 집합을 하나의 협조된 운용 시스템(Coordinated Operational System)으로 전환한다. 명시적인 종속성(Explicit Dependency), 순서화된 전이(Ordered Transition), 준비 상태 검증(Readiness Verification), 장애 격리(Failure Containment), 복구 정책(Recovery Policy), 관측 가능성(Observability), 제어된 종료(Controlled Shutdown)를 통해 기동 및 런타임 동작을 우연에 의존하지 않고 재현 가능하게 만든다. 이는 이후 다루게 될 수명주기 관리자 구현(Lifecycle Manager Implementation), 자동 복구(Automatic Recovery), 종속성 순서 관리(Dependency Ordering), 로봇 전원 시퀀싱(Robot Power Sequencing), 진단(Diagnostics), 다중 로봇 동기화(Multi-robot Synchronization), 안전 상태 통합(Safe-state Integration)의 기반을 제공한다.

## 07.03 ros2.lifecycle.manager Usage [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 수명주기 관리자(Lifecycle Manager)는 각 구성요소가 독립적으로 시작과 종료를 결정하도록 하는 대신, 여러 관리형 노드(Managed Node)의 수명주기 인터페이스(Lifecycle Interface)를 통해 이들을 제어하는 감독 메커니즘(Supervisory Mechanism)을 제공한다. 수명주기 관리 구조(Lifecycle-management Structure)에서 주요 역할은 상태 전이(State Transition)를 요청하고, 결과를 관찰하며, 노드 순서를 조정하고, 전체 로봇 소프트웨어 스택에서 예측 가능한 운용 상태(Operational State)를 유지하는 것이다.

관리자(Manager)는 개별 수명주기 노드(Lifecycle Node)의 상위에서 동작하며 구성(Configure), 활성화(Activate), 비활성화(Deactivate), 정리(Cleanup), 종료(Shutdown) 전이를 통해 이들과 상호작용한다. 각 관리형 노드는 여전히 애플리케이션별 콜백(Application-specific Callback)을 자체적으로 담당하지만, 관리자는 이러한 콜백이 언제 호출되어야 하는지를 결정한다. 이러한 분리는 장치 초기화(Device Initialization), 알고리즘 설정(Algorithm Setup), 자원 처리(Resource Handling)는 노드 내부에 유지하면서 운용 순서와 감독 기능은 중앙에서 관리할 수 있게 한다.

일반적인 구성(Configuration)에서는 관리 대상 그룹에 포함되는 수명주기 지원 노드(Lifecycle-enabled Node)를 지정한다. 해당 노드의 이름과 예상 실행 순서(Expected Order)가 관리자 구성의 일부가 되므로 시스템은 반복 가능한 기동 순서(Repeatable Startup Sequence)를 확립할 수 있다. 따라서 센서 드라이버(Sensor Driver), 위치추정 구성요소(Localization Component), 내비게이션 모듈(Navigation Module), 제어기(Controller), 기타 관리형 서비스(Managed Service)를 서로 독립적인 ROS 2 프로세스가 아니라 하나의 협조된 운용 서브시스템(Coordinated Operational Subsystem)의 구성원으로 관리할 수 있다.

기동(Startup) 과정에서 관리자는 먼저 관리 대상 노드에 구성 전이(Configuration Transition)를 요청한다. 각 노드는 \`on_configure()\` 로직을 실행하면서 미구성(Unconfigured) 상태에서 비활성(Inactive) 상태로 이동하며, 이 과정에서 파라미터(Parameter)를 로드하고, 자원(Resource)을 할당하고, 하드웨어 인터페이스(Hardware Interface)를 초기화하거나 통신 객체(Communication Object)를 준비할 수 있다. 관리자는 해당 구성요소를 다음 단계로 진행할 준비가 완료된 것으로 판단하기 전에 전이가 성공했는지 확인해야 한다.

구성이 성공한 이후 활성화 요청(Activation Request)은 노드를 비활성(Inactive) 상태에서 활성(Active) 상태로 이동시킨다. \`on_activate()\` 콜백은 수명주기 발행자(Lifecycle Publisher), 타이머(Timer), 센서 획득(Sensor Acquisition), 처리 파이프라인(Processing Pipeline), 기타 운용 기능을 활성화한다. 관리자가 활성화를 중앙에서 제어함으로써 상위 수준 구성요소가 자신이 의존하는 하위 수준 서비스와 자원이 필요한 수명주기 상태에 도달하기 전에 동작하는 것을 방지할 수 있다.

수명주기 관리자(Lifecycle Manager)는 기동 순서가 기능적 종속성(Functional Dependency)을 반영할 때 특히 유용하다. 센서 드라이버는 인지(Perception)보다 먼저 활성화되어야 하고, 인지는 위치추정(Localization)보다 먼저, 위치추정은 내비게이션(Navigation)보다 먼저 활성화되어야 할 수 있다. 관리자는 이러한 순서를 정의하거나 따름으로써 준비 상태(Readiness)를 단계적으로 확립할 수 있다. 이는 실행 스크립트(Launch Script)에 임의의 대기 시간(Sleep Period)을 삽입하고 단순히 시간이 경과하면 구성요소가 준비되었다고 가정하는 방식보다 신뢰성이 높다.

자동 시작 동작(Autostart Behavior)은 로봇이 실행된 이후 관리 대상 노드를 자동으로 구성하고 활성화해야 하는 배포 환경(Deployment)을 단순화할 수 있다. 자동 시작(Autostart)이 활성화되면 수명주기 관리자는 별도의 운영자 명령을 기다리지 않고 필요한 전이를 시작할 수 있다. 자동 시작이 비활성화되면 시스템은 명시적인 기동 제어(Explicit Startup Control)를 기다리는 상태를 유지할 수 있으며, 이는 시운전(Commissioning), 디버깅(Debugging), 유지보수(Maintenance), 시험(Testing), 단계적 시스템 초기화(Staged System Initialization)에 유용하다.

자동 기동을 사용할 수 있는 경우에도 외부 제어(External Control)는 중요하다. 감독 소프트웨어(Supervisory Software), 실행 인프라(Launch Infrastructure), 운영자 인터페이스(Operator Interface), 상위 수준 오케스트레이션 로직(Higher-level Orchestration Logic)은 구현 방식에 따라 기동(Startup), 일시 중지(Pause), 재개(Resume), 재설정(Reset), 종료(Shutdown) 동작을 요청할 수 있다. 따라서 수명주기 관리자는 여러 수명주기 노드를 개별적으로 제어하는 대신 하나의 협조된 그룹(Coordinated Group)으로 조작할 수 있는 운용 제어 지점(Operational Control Point)의 역할을 한다.

비활성화(Deactivation)를 사용하면 관리자는 노드 구성을 완전히 제거하지 않고 활성 운용(Active Operation)을 일시 중지할 수 있다. 관리 대상 노드는 활성(Active) 상태에서 비활성(Inactive) 상태로 전이하면서 발행자(Publisher), 타이머, 처리 루프(Processing Loop), 하드웨어 연계 동작(Hardware-facing Activity)을 중지하는 동시에 이후 재활성화(Reactivation)에 필요한 자원은 유지할 수 있다. 이러한 동작은 로봇이 자율 운용을 일시적으로 중단하거나 유지보수 상태로 진입하거나, 운용 모드를 변경하거나, 제어된 복구(Controlled Recovery)를 준비할 때 유용하다.

정리(Cleanup)는 적절한 노드를 비활성(Inactive) 상태에서 미구성(Unconfigured) 상태로 되돌려 더욱 깊은 수준의 재설정(Reset)을 제공한다. \`on_cleanup()\` 콜백은 구성 과정에서 생성된 자원을 해제하고, 인터페이스를 닫고, 내부 데이터 구조(Internal Data Structure)를 초기화하며, 다음 구성 주기를 준비할 수 있다. 단순한 비활성화만으로 충분하지 않고 운영체제 프로세스 전체를 종료하지 않은 상태에서 서브시스템을 다시 구성해야 할 때 수명주기 관리자는 이러한 기능을 활용할 수 있다.

관리자는 전이 요청을 전송했다는 이유만으로 성공했다고 가정하지 않고 전이 응답(Transition Response)을 실제 운용 증거(Operational Evidence)로 취급해야 한다. 장치를 사용할 수 없어 구성이 실패하거나, 필요한 자원이 없어 활성화가 실패하거나, 정리 과정에서 예상하지 못한 문제가 발생할 수 있다. 따라서 견고한 관리(Robust Management)를 위해서는 결과 상태(Resulting State)를 확인하고, 타임아웃(Timeout)이나 실패를 탐지하며, 종속 구성요소가 잘못된 기동 순서를 계속 진행하지 못하도록 방지해야 한다.

수명주기 관리(Lifecycle Management)는 모니터링 로직(Monitoring Logic)과 통합될 경우 구조화된 복구(Structured Recovery)도 지원한다. 관리 대상 구성요소가 비정상 상태가 되면 감독 소프트웨어는 복구 정책(Recovery Policy)에 따라 비활성화, 정리, 재구성(Reconfiguration), 재활성화를 시작할 수 있다. 복구가 진행되는 동안 종속 노드(Dependent Node)를 일시 중지해야 할 수도 있다. 이는 이후 수명주기 관리 주제에서 다루는 런타임 노드 재시작(Runtime Node Restart)과 자동 복구(Automatic Recovery) 메커니즘의 기반을 형성한다.

관리자는 수명주기 상태(Lifecycle State)와 구성요소 건전성(Component Health)을 구분해야 한다. 노드가 기술적으로 활성(Active) 상태를 유지하고 있더라도 센서 스트림이 중단되거나, 위치추정 결과가 저하되거나, 하드웨어 인터페이스가 오류를 보고하고 있을 수 있다. 따라서 효과적인 수명주기 관리자 사용을 위해서는 활성 상태만을 정상적인 로봇 동작의 증거로 간주하지 않고 상태 전이를 진단(Diagnostics), 하트비트 정보(Heartbeat Information), 애플리케이션별 준비 상태 검사(Application-specific Readiness Check), 타임아웃 모니터링(Timeout Monitoring)과 결합해야 한다.

종료 순서(Shutdown Sequencing)는 기동 순서만큼 신중하게 설계해야 한다. 일반적으로 상위 수준 소비자(High-level Consumer)는 자신에게 데이터나 기능을 제공하는 하위 수준 서비스보다 먼저 중지되어야 한다. 임무 실행(Mission Execution)과 내비게이션은 위치추정 및 인지보다 먼저 비활성화할 수 있으며, 하드웨어 연계 노드는 종속된 소프트웨어 구성요소가 더 이상 필요로 하지 않을 때 중지할 수 있다. 이러한 제어된 순서는 일관되지 않은 상태를 줄이고 자원을 예측 가능한 방식으로 해제할 수 있도록 한다.

수명주기 관리자(Lifecycle Manager)는 로봇 전원 시퀀싱(Robot Power Sequencing)에도 참여할 수 있다. 소프트웨어 구성요소는 관련 센서, 컴퓨터, 통신 네트워크, 액추에이터가 물리적으로 준비되기 전에 활성화되어서는 안 되며, 종속된 소프트웨어가 계속 명령을 전송하거나 데이터를 기다리는 동안 하드웨어의 전원을 차단해서도 안 된다. 수명주기 전이는 보다 광범위한 기동 및 종료 절차와 조정할 수 있는 소프트웨어 가시적 동기화 지점(Software-visible Synchronization Point)을 제공한다.

더 큰 규모의 아키텍처에서는 하나의 관리자가 반드시 로봇의 모든 노드를 제어할 필요는 없다. 별도의 수명주기 관리자들이 하드웨어(Hardware), 인지(Perception), 내비게이션(Navigation), 조작(Manipulation), 인공지능 추론(AI Inference), 기타 서브시스템을 각각 감독하고 상위 수준 오케스트레이터(Higher-level Orchestrator)가 이들 그룹을 조정할 수 있다. 이러한 계층형 관리(Hierarchical Management)는 결합도(Coupling)를 줄이고 각 서브시스템이 자체 전이 로직을 유지하면서도 시스템 전체 수명주기 오케스트레이션(System-wide Lifecycle Orchestration)에 참여할 수 있도록 한다.

운용 가시성(Operational Visibility)은 관리자 제어와 함께 제공되어야 한다. 현재 수명주기 상태, 요청된 전이, 전이 소요 시간(Transition Duration), 실패한 콜백(Failed Callback), 복구 시도(Recovery Attempt), 관리자의 결정(Manager Decision)을 로깅(Logging)과 진단을 통해 확인할 수 있어야 한다. 이를 통해 개발자와 운영자는 로봇이 구성을 기다리는 중인지, 부분적으로 활성화된 상태인지, 장애에서 복구 중인지, 의도적으로 일시 중지된 것인지, 또는 예상된 운용 상태에 도달하지 못하고 있는지를 판단할 수 있다.

수명주기 관리자 사용은 기능 안전 권한(Functional Safety Authority)과 분리되어야 한다. 관리자는 장애가 탐지되었을 때 제어기를 비활성화하거나 종료를 요청할 수 있지만 ROS 2 수명주기 전이는 비상 정지 회로(Emergency-stop Circuit), 안전 PLC(Safety PLC), 안전 토크 차단(Safe Torque Off), 기타 전용 안전 메커니즘(Dedicated Safety Mechanism)을 대체하지 않는다. 수명주기 관리는 안전 이벤트 주변의 소프트웨어 동작을 조정하지만 실제 안전 기능(Safety Function)은 독립적으로 강제될 수 있어야 한다.

ROS 2 수명주기 관리자(Lifecycle Manager)를 올바르게 사용하면 수명주기 지원 노드들을 관리 가능한 운용 서브시스템(Manageable Operational Subsystem)으로 전환할 수 있다. 중앙집중형 전이 제어(Central Transition Control), 순서화된 구성 및 활성화(Ordered Configuration and Activation), 제어된 비활성화 및 정리(Controlled Deactivation and Cleanup), 장애 인식 감독(Failure-aware Supervision), 진단과의 통합을 통해 반복 가능한 로봇 기동 및 종료 동작을 구축할 수 있다. 이는 개별 수명주기 노드를 자동 복구, 종속성 기반 기동(Dependency-based Startup), 전원 시퀀싱, 진단 경보(Diagnostic Alert), 시스템 수준 수명주기 오케스트레이션과 연결하는 실질적인 관리 계층(Management Layer)을 형성한다.

## 07.04 Runtime Node Restart and Auto Recovery Design [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

런타임 노드 재시작(Runtime Node Restart)과 자동 복구(Automatic Recovery)는 장시간 운용되는 ROS 2 로봇에서 필수적인 기능이다. 소프트웨어와 하드웨어 장애를 설계 단계의 검증만으로 항상 제거할 수는 없기 때문이다. 센서 통신이 끊기거나, 장치 드라이버(Device Driver)가 응답하지 않거나, 알고리즘이 비정상 상태에 진입하거나, 외부 자원을 일시적으로 사용할 수 없게 될 수 있다. 따라서 복구 아키텍처(Recovery Architecture)는 장애를 체계적으로 탐지하고 격리하며 해결해야 하는 예상 가능한 운용 이벤트(Operational Event)로 취급한다.

복구 설계(Recovery Design)는 노드 건전성(Node Health), 수명주기 상태(Lifecycle State), 프로세스 건전성(Process Health), 서브시스템 준비 상태(Subsystem Readiness)를 구분해야 한다. 노드는 출력이 중단된 상태에서도 활성(Active) 상태를 유지할 수 있으며, 프로세스가 존재하더라도 내부 하드웨어 연결이 실패할 수 있다. 반대로 프로세스 재시작에 성공했더라도 전체 서브시스템이 복원되지 않을 수 있다. 따라서 효과적인 복구는 수명주기 정보와 진단(Diagnostics), 하트비트 모니터링(Heartbeat Monitoring), 통신 활동, 자원 상태(Resource Status), 애플리케이션별 건전성 기준(Application-specific Health Criteria)을 결합한다.

장애 탐지(Fault Detection)는 복구 순서(Recovery Sequence)의 시작점이다. 감독 로직(Supervisory Logic)은 진단 메시지(Diagnostic Message), 하트비트 타임스탬프(Heartbeat Timestamp), 토픽 발행률(Topic Publication Rate), 서비스 응답성(Service Responsiveness), 수명주기 전이 결과(Lifecycle Transition Result), 하드웨어 상태, 콜백 실행 시간(Callback Execution Time), 프로세스 가용성(Process Availability)을 모니터링할 수 있다. 각각의 감시 조건에는 의미 있는 타임아웃(Timeout)이나 임계값(Threshold)을 설정하여 일시적인 지연과 지속적인 장애를 구분하면서 비정상 구성요소가 무기한 운용되는 것을 방지해야 한다.

탐지된 장애는 복구를 시도하기 전에 분류되어야 한다. 복구 가능한 조건(Recoverable Condition)에는 일시적인 네트워크 중단, 사용할 수 없는 센서, 일시적인 초기화 실패(Transient Initialization Failure), 해소 가능한 자원 고갈(Resource Exhaustion) 등이 포함될 수 있다. 반면 지속적인 구성 오류(Configuration Error), 호환되지 않는 파라미터, 손상된 자원, 반복적인 하드웨어 장애는 운영자 개입(Operator Intervention)이 필요할 수 있다. 이러한 분류를 통해 자율적으로 복구할 수 없는 구성요소가 자동 복구 메커니즘에 의해 반복적으로 재시작되는 것을 방지할 수 있다.

일반적으로 가장 영향이 적은 복구 동작(Least Disruptive Recovery Action)을 먼저 시도해야 한다. 활성(Active) 노드가 비활성화와 재활성화만으로 정상 동작을 복원할 수 있다면 프로세스를 제거할 필요가 없다. 더 심각한 장애에서는 비활성화(Deactivate), 정리(Cleanup), 구성(Configure), 활성화(Activate) 전이를 통해 노드 자원을 다시 구성해야 할 수 있다. 수명주기 수준 복구(Lifecycle-level Recovery)로 정상 동작을 복원할 수 없는 경우에만 프로세스 재시작(Process Restart), 서브시스템 재시작(Subsystem Restart), 또는 더 광범위한 시스템 복구(System Recovery)로 확대해야 한다.

수명주기 기반 복구(Lifecycle-based Recovery)는 복구 동작을 명시적으로 수행하고 관찰할 수 있다는 관리형 노드(Managed Node)의 장점을 유지한다. 노드는 활성(Active) 상태에서 비활성(Inactive) 상태로 전이하고, \`on_cleanup()\`을 실행하여 손상된 자원을 해제하고, \`on_configure()\`를 실행하여 인터페이스와 내부 상태를 다시 생성한 후, \`on_activate()\`를 실행하여 운용을 재개할 수 있다. 각각의 전이는 계속 진행하기 전에 성공, 실패, 타임아웃, 진단 정보를 평가할 수 있는 확인 지점(Checkpoint)을 제공한다.

노드가 수명주기 서비스(Lifecycle Service)에 응답할 수 없거나, 실행기(Executor)가 차단되거나, 메모리 손상(Memory Corruption)이 의심되거나, 복구 불가능한 내부 장애로 인해 정상적인 전이를 수행할 수 없는 경우에는 프로세스 수준 재시작(Process-level Restart)이 필요하다. 이 경우 외부 감독기(External Supervisor)가 프로세스를 종료하고 다시 생성해야 한다. 실행 시스템(Launch System), 서비스 관리자(Service Manager), 컨테이너(Container), 전용 감시 프로세스(Watchdog Process)가 이러한 기능을 제공할 수 있으며, ROS 2 수명주기 관리는 주변 서브시스템의 논리적 복구(Logical Recovery)를 조정한다.

하나의 노드를 재시작하면 여러 종속 구성요소(Dependent Component)에 영향을 줄 수 있다. 라이다(LiDAR) 드라이버가 실패하면 인지(Perception)와 위치추정(Localization)은 더 이상 유효한 데이터를 수신하지 못하지만, 내비게이션(Navigation)은 오래된 정보를 사용하여 일시적으로 계속 동작할 수 있다. 따라서 복구 로직은 종속성 관계(Dependency Relationship)를 이용하여 장애 구성요소를 재시작하기 전에 영향을 받는 하위 노드(Downstream Node)를 비활성화하거나 동작을 억제해야 한다. 복구 이후에는 정상적인 종속성 기반 기동 순서(Dependency-aware Startup Order)에 따라 종속 노드를 다시 활성화할 수 있다.

복구 정책(Recovery Policy)은 무제한 재시작 루프(Unlimited Restart Loop)에 의존하는 대신 단계적인 에스컬레이션 수준(Escalation Level)을 정의해야 한다. 첫 번째 장애에서는 수명주기 재활성화(Lifecycle Reactivation)를 시도하고, 반복된 장애에서는 정리와 재구성을 수행하며, 이후에도 장애가 발생하면 프로세스 재시작을 수행할 수 있다. 정의된 재시도 한도(Retry Limit)나 시간 범위를 초과하여 장애가 계속되면 무한한 복구 시도로 자원을 소비하는 대신 서브시스템을 격리(Isolate)하거나 성능 저하(Degraded) 상태로 표시하거나 운영자 제어(Operator Control)로 전환할 수 있다.

재시도 동작(Retry Behavior)에는 제어된 시간 관리가 포함되어야 한다. 장애가 발생한 장치를 즉시 수백 번 재시작하면 하드웨어, 통신 네트워크, 로그 또는 외부 서비스에 과도한 부하를 발생시킬 수 있다. 복구 관리자(Recovery Manager)는 재시도 간격(Retry Interval), 점진적으로 증가하는 백오프 지연(Backoff Delay), 최대 재시도 횟수(Maximum Retry Count), 복구 시간 범위(Recovery Window)를 적용할 수 있다. 일정 기간 동안 안정적으로 동작하면 장애 카운터(Failure Counter)를 초기화하여 간헐적인 일시 장애와 빠르게 반복되는 장애를 서로 다르게 처리할 수 있다.

구성요소를 재시작하기 전에 상태 보존(State Preservation)을 고려해야 한다. 일부 노드는 다시 생성할 수 있는 런타임 상태(Runtime State)만 유지하지만, 다른 노드는 캘리브레이션 데이터(Calibration Data), 지도(Map), 임무 진행 상태(Mission Progress), 학습된 파라미터(Learned Parameter), 하드웨어 구성(Hardware Configuration)을 유지할 수 있다. 복구 로직은 어떤 정보를 재시작 이후에도 보존해야 하고 어떤 정보를 의도적으로 폐기해야 하는지 결정해야 한다. 정보 손실로 정상적인 복구나 임무 재개가 불가능해지는 경우 영구 상태(Persistent State)는 휘발성 노드 메모리 외부에 저장해야 한다.

자동 복구(Automatic Recovery)는 재시작 이후 준비 상태 검증(Readiness Verification)도 필요로 한다. 프로세스가 다시 생성되었다고 해서 반드시 운용 가능한 것은 아니며, 활성(Active) 수명주기 상태만으로도 정상 동작을 증명할 수 없다. 감독기(Supervisor)는 복구 성공을 선언하고 종속 구성요소를 재활성화하기 전에 필요한 토픽(Topic), 센서 데이터율(Sensor Rate), 좌표 변환(Transform), 위치추정 품질(Localization Quality), 하드웨어 상태, 모델 가용성(Model Availability), 기타 애플리케이션별 조건을 검증해야 한다.

복구 과정은 일련의 운용 이벤트로 관찰할 수 있어야 한다. 로그에는 최초 장애(Original Fault), 탐지 시간(Detection Time), 영향을 받은 노드, 이전 수명주기 상태, 복구 동작, 전이 결과, 재시작 횟수(Restart Count), 복구 소요 시간(Elapsed Recovery Time), 일시 중지된 종속 노드, 최종 결과(Final Outcome)를 기록해야 한다. 이러한 기록은 일시적이고 개별적인 장애와 공학적 개선이 필요한 체계적인 신뢰성 문제(Systematic Reliability Problem)를 구분하는 데 필수적이다.

감시기 아키텍처(Watchdog Architecture)는 여러 수준에서 구현할 수 있다. 내부 감시기(Internal Watchdog)는 정지된 콜백이나 누락된 장치 데이터를 탐지할 수 있고, 서브시스템 감독기(Subsystem Supervisor)는 여러 수명주기 노드를 모니터링할 수 있으며, 외부 프로세스 감독기(External Process Supervisor)는 노드의 완전한 장애를 탐지할 수 있다. 상위 수준 로봇 관리 시스템은 서브시스템 준비 상태와 임무 영향을 관찰할 수 있다. 계층화된 감시기(Layered Watchdog)는 감시 대상 구성요소와 함께 실패할 수 있는 단일 메커니즘에 대한 의존성을 줄인다.

복구 동시성(Recovery Concurrency)은 신중하게 제어해야 한다. 서로 독립적인 장애는 종속성이 겹치지 않는 경우 병렬로 복구할 수 있지만, 긴밀하게 결합된 노드를 동시에 재시작하면 경쟁 상태(Race Condition)가 발생하거나 공유 자원(Shared Resource)에 과부하가 발생할 수 있다. 복구 조정기(Recovery Coordinator)는 필요한 경우 전이를 직렬화(Serialize)하고, 공유 복구 영역(Shared Recovery Domain)을 잠그며, 여러 감독기가 동일한 노드 또는 서브시스템에 충돌하는 수명주기 명령을 전달하지 못하도록 해야 한다.

자동 복구는 심각한 장애를 숨겨서는 안 된다. 반복적으로 충돌하고 재시작되는 구성요소는 신뢰성이 악화되고 있음에도 일시적으로 정상 운용되는 것처럼 보일 수 있다. 따라서 복구에 성공한 이후에도 복구 횟수(Recovery Counter), 성능 저하 상태 표시(Degraded-state Indicator), 장애 이력(Fault History), 에스컬레이션 알림(Escalation Notification)을 계속 확인할 수 있어야 한다. 목표는 모든 노드를 근본적인 상태와 관계없이 단순히 활성(Active) 상태로 되돌리는 것이 아니라 장애 투명성(Fault Transparency)을 유지하면서 운용 연속성(Operational Continuity)을 확보하는 것이다.

안전 관련 구성요소(Safety-related Component)에는 더욱 엄격한 복구 정책이 필요하다. 자동 재시작은 비상 정지(Emergency Stop), 안전 인터록(Safety Interlock), 안전 토크 차단(Safe Torque Off), 하드웨어 장애 조건(Hardware Fault Condition)을 절대로 우회해서는 안 된다. 복구된 제어기(Controller)는 독립적인 안전 조건이 운용을 허용하고 필요한 준비 상태 검사가 성공할 때까지 액추에이터 명령을 다시 시작해서는 안 된다. 수명주기 복구는 소프트웨어 복원을 조정할 수 있지만 위험한 동작(Hazardous Motion)에 대한 허가는 적절한 안전 아키텍처(Safety Architecture)의 통제 아래 유지되어야 한다.

다중 로봇 시스템(Multi-robot System)에서는 일반적으로 복구를 장애가 발생한 로봇 내부에 국한하면서 플릿 관리 시스템(Fleet Management)에 요약된 가용성 정보(Availability Information)를 제공해야 한다. 복구 중인 로봇은 일시적으로 새로운 임무를 거부하거나 성능 저하 능력(Degraded Capability)을 보고할 수 있다. 복구가 실패하면 플릿 관리자(Fleet Manager)는 유지보수를 요청하면서 다른 로봇에 작업을 재분배할 수 있으며, 이를 통해 하나의 노드 장애가 전체 플릿 운용을 불필요하게 방해하는 것을 방지할 수 있다.

견고한 런타임 재시작 및 자동 복구 설계(Runtime Restart and Auto-recovery Design)는 결국 장애 탐지(Fault Detection), 장애 분류(Classification), 수명주기 전이(Lifecycle Transition), 프로세스 감독(Process Supervision), 종속성 관리(Dependency Management), 재시도 제한(Retry Limit), 상태 보존(State Preservation), 준비 상태 검증(Readiness Verification), 관측 가능성(Observability), 에스컬레이션(Escalation)을 결합한다. 이를 통해 재시작을 임시방편적인 대응이 아니라 제어된 상태 관리 절차(Controlled State-management Procedure)로 전환하며, 종속성 기반 노드 기동, 전원 시퀀싱(Power Sequencing), 진단 경보(Diagnostic Alert), 안전 상태 진입(Safe-state Entry), 수명주기 통합 시험(Lifecycle Integration Testing)을 위한 기반을 제공한다.

## 07.05 Dependency-Based Node Start Order Management [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

종속성 기반 노드 시작 순서 관리(Dependency-based Node Start Order Management)는 ROS 2 구성요소가 자신에게 필요한 서비스(Service), 데이터 소스(Data Source), 하드웨어 자원(Hardware Resource), 소프트웨어 기능(Software Capability)이 실제로 준비된 이후에만 운용 상태로 진입하도록 보장한다. 복잡한 로봇에서는 실행된 노드가 여전히 초기화 중일 수 있으므로 단순한 프로세스 생성 순서만으로는 충분하지 않다. 따라서 명시적 종속성 관리(Explicit Dependency Management)는 시간에 대한 가정을 구성요소 사이의 검증 가능한 준비 상태 관계(Verifiable Readiness Relationship)로 대체한다.

종속성(Dependency)은 단순히 실행 파일(Launch File)에서 먼저 시작된 프로세스가 아니라 기능적 선행 조건(Functional Prerequisite)을 의미한다. 위치추정 노드(Localization Node)는 활성화된 라이다(LiDAR) 및 IMU 드라이버, 유효한 TF 좌표 변환(TF Transform), 로드된 지도(Map)를 필요로 할 수 있으며, 내비게이션(Navigation)은 위치추정, 비용지도(Costmap), 제어기 인터페이스(Controller Interface)를 필요로 할 수 있다. 따라서 시작 순서 관리는 프로세스가 존재하면 운용 준비가 완료되었다고 가정하는 대신 각 구성요소가 활성화되기 전에 실제로 무엇을 필요로 하는지를 모델링해야 한다.

이러한 관계는 노드가 관리형 ROS 2 구성요소를 나타내고 연결선이 선행 조건 관계(Prerequisite Relationship)를 나타내는 방향성 종속성 그래프(Directed Dependency Graph)로 표현할 수 있다. 해결되지 않은 선행 조건이 없는 구성요소는 가장 초기의 기동 계층(Startup Layer)을 형성하며, 종속 구성요소는 이후 계층에 위치한다. 이러한 그래프는 기동 로직(Startup Logic)을 명확하게 하고 검증을 지원하며 유효한 구성(Configuration), 활성화(Activation), 복구(Recovery), 종료(Shutdown) 순서를 계산하기 위한 기반을 제공한다.

수명주기 상태(Lifecycle State)는 종속성 관리를 위한 유용한 동기화 지점(Synchronization Point)을 제공한다. 구성요소는 먼저 미구성(Unconfigured) 상태에서 비활성(Inactive) 상태로 전이하면서 자원을 할당하고 구성을 검증할 수 있지만, 하위 종속 구성요소(Downstream Component)가 아직 운용 상태로 진입할 필요는 없다. 필요한 선행 구성요소가 예상 상태와 준비 조건에 성공적으로 도달하면 오케스트레이터(Orchestrator)는 다음 종속성 계층의 활성화를 제어된 순서로 요청할 수 있다.

필요한 경우 구성 종속성(Configuration Dependency)과 활성화 종속성(Activation Dependency)을 구분해야 한다. 구성 단계에서는 파라미터를 로드하고 메모리를 할당하기만 하므로 상위 데이터 소스가 활성(Active) 상태가 되기 전에도 노드를 구성할 수 있다. 그러나 활성화에는 실제 센서 데이터나 정상적으로 동작하는 하드웨어 인터페이스가 필요할 수 있다. 이러한 조건을 분리하면 운용의 정확성을 유지하면서 기동 병렬성(Startup Parallelism)을 높이고 전체 로봇 소프트웨어 스택을 불필요하게 직렬화(Serialize)하는 것을 방지할 수 있다.

종속성 그래프에서 서로 독립적인 분기(Independent Branch)는 동시에 처리할 수 있다. 예를 들어 카메라(Camera)와 라이다 드라이버가 서로 충돌하는 자원을 공유하지 않는다면 병렬로 구성할 수 있으며, 인지 구성요소(Perception Component)는 각각 필요한 입력이 준비될 때까지 기다릴 수 있다. 제어된 병렬 처리(Controlled Parallelism)는 결정론적 종속성 동작(Deterministic Dependency Behavior)을 유지하면서 전체 기동 시간을 줄인다. 따라서 오케스트레이터는 실제 종속성이나 공유 자원 요구사항에 의해 제약되는 전이만 직렬화해야 한다.

준비 상태(Readiness)는 단순한 수명주기 상태보다 더 강한 조건이어야 한다. 활성(Active) 상태의 센서 드라이버가 아직 첫 번째 유효 측정값을 기다리고 있을 수 있으며, 활성 상태의 위치추정 노드가 아직 신뢰할 수 있는 자세 추정(Pose Estimate)을 생성하지 못했을 수 있다. 따라서 종속성 조건에는 토픽 가용성(Topic Availability), 최소 발행률(Minimum Publication Rate), 유효한 타임스탬프(Valid Timestamp), TF 가용성, 하드웨어 상태, 진단 수준(Diagnostic Level), 지도 준비 상태(Map Readiness), 위치추정 품질(Localization Quality), 애플리케이션별 확인 신호(Application-specific Confirmation Signal) 등이 포함될 수 있다.

타임아웃 처리(Timeout Handling)는 특정 종속성으로 인해 기동 과정이 무기한 차단되는 것을 방지한다. 실패할 수 있는 모든 준비 조건에는 적절한 대기 정책(Waiting Policy)과 타임아웃을 설정해야 한다. 선행 조건이 준비되지 않으면 오케스트레이터는 시스템 정책에 따라 해당 전이를 재시도하고, 복구를 시작하거나, 서브시스템을 성능 저하(Degraded) 상태로 표시하거나, 선택적 기능(Optional Functionality)을 건너뛰거나, 기동을 중단할 수 있다. 장애 정보에는 사용할 수 없는 선행 조건과 그로 인해 시작하지 못한 노드가 모두 명확하게 나타나야 한다.

종속성은 필수 종속성(Mandatory Dependency)과 선택적 종속성(Optional Dependency)으로 구분할 수 있다. 필수 종속성을 사용할 수 없는 경우 정상 동작을 보장할 수 없으므로 활성화를 차단한다. 선택적 종속성은 사용할 수 있을 때 향상된 기능을 제공하지만 사용할 수 없더라도 제한된 기능 모드(Reduced-capability Mode)를 허용할 수 있다. 이러한 차이를 명시적으로 모델링하면 모든 주변장치 장애를 전체 시스템의 기동 실패로 처리하지 않고 정의된 성능 저하 모드(Degraded Mode)로 로봇을 계속 운용할 수 있다.

조건부 종속성(Conditional Dependency)은 로봇이 여러 운용 모드(Operating Mode)를 지원할 때 유용하다. 지도 생성 운용(Map-building Operation)에서는 매핑(Mapping)이 필요할 수 있지만 기존에 저장된 지도를 사용하여 주행할 때는 필요하지 않을 수 있다. 조작기(Manipulator)는 물체 취급 임무(Handling Mission)에서는 필요하지만 점검 임무(Inspection Mission)에서는 필요하지 않을 수 있다. 따라서 활성 종속성 그래프(Active Dependency Graph)는 임무 모드, 하드웨어 구성, 배포 프로파일(Deployment Profile), 사용 가능한 기능(Capability)에 따라 선택하거나 변경할 수 있다.

순환 종속성(Circular Dependency)은 가능한 경우 런타임 이전에 탐지해야 한다. 노드 A가 활성 상태가 되기 위해 노드 B를 필요로 하고 동시에 노드 B가 노드 A를 필요로 한다면 어느 쪽도 다음 단계로 진행할 수 없다. 종속성 검증(Dependency Validation)은 구성 또는 통합 시험(Integration Testing) 과정에서 순환(Cycle), 누락된 노드 참조(Missing Node Reference), 불가능한 상태 요구사항(Impossible State Requirement), 서로 모순되는 준비 조건(Contradictory Readiness Condition)을 식별해야 한다. 잘못된 그래프를 사전에 방지하는 것이 배포된 로봇에서 기동 교착 상태(Startup Deadlock)를 진단하는 것보다 훨씬 신뢰성이 높다.

시작 순서 로직(Start-order Logic)은 하드웨어 준비 상태(Hardware Readiness)와도 연계되어야 한다. ROS 2 드라이버는 물리적 장치의 전원이 공급되고, 통신 링크가 설정되며, 필요한 펌웨어(Firmware)가 정상적으로 응답하기 전에 활성화되어서는 안 된다. 마찬가지로 상위 수준 제어 노드(Control Node)는 액추에이터 인터페이스(Actuator Interface)가 초기화되기 전에 운용 상태로 진입해서는 안 된다. 따라서 소프트웨어 종속성은 ROS 2 노드를 넘어 전원 제어기(Power Controller), 네트워크(Network), 필드버스(Fieldbus), 센서, 가속기(Accelerator), 외부 서비스(External Service)까지 확장되는 경우가 많다.

기동 중 발생한 장애에는 종속성을 고려한 롤백 또는 격리(Dependency-aware Rollback or Containment)가 필요하다. 센서 드라이버가 활성화된 이후 인지 기능이 실패하면 위치추정과 내비게이션은 비활성 상태를 유지해야 하지만, 관련이 없는 서브시스템은 계속 동작할 수 있다. 정책에 따라 이미 활성화된 선행 구성요소를 진단 목적으로 계속 유지하거나 역순으로 비활성화할 수 있다. 이를 통해 하나의 기동 장애가 로봇의 모든 구성요소를 자동으로 불안정하게 만드는 것을 방지한다.

런타임 복구(Runtime Recovery)에서도 동일한 종속성 정보를 사용한다. 운용 중 상위 노드(Upstream Node)에 장애가 발생하면 하위 소비자(Downstream Consumer)를 즉시 식별하여 누락되거나 오래된 정보를 사용하기 전에 동작을 중지시킬 수 있다. 장애 노드가 복구되고 준비 상태가 검증된 이후에는 원래의 순서 규칙에 따라 종속 구성요소를 다시 활성화할 수 있다. 따라서 종속성 그래프는 초기 기동뿐만 아니라 장애 복구에도 활용된다.

종료 순서(Shutdown Ordering)는 일반적으로 종속성 관계를 역순으로 적용하여 결정한다. 임무 실행(Mission Execution)과 내비게이션은 위치추정보다 먼저 중지되어야 하고, 인지는 해당 센서 소스가 해제되기 전에 중지되어야 하며, 하드웨어 연계 구성요소(Hardware-facing Component)는 해당 자원을 사용하는 구성요소의 종료가 완료될 때까지 유지되어야 한다. 역방향 종속성 순서(Reverse Dependency Ordering)는 로봇 전원 종료 과정에서 처리되지 않은 요청, 잘못된 데이터 접근, 미완료 명령, 통제되지 않은 자원 제거를 줄여준다.

계층형 아키텍처(Hierarchical Architecture)는 대규모 로봇의 종속성 관리를 단순화할 수 있다. 지역 관리자(Local Manager)는 센서, 인지, 내비게이션, 조작(Manipulation), 제어, 인공지능 서브시스템을 각각 관리하고 시스템 수준 오케스트레이터(System-level Orchestrator)가 이러한 그룹 사이의 종속성을 관리할 수 있다. 이를 통해 지나치게 상세한 하나의 전역 실행 순서(Global Sequence)를 구성하는 것을 피하고 각 서브시스템 개발팀이 명확하게 정의된 준비 상태 인터페이스(Readiness Interface)를 통해 내부 종속성 규칙을 독립적으로 관리할 수 있다.

종속성 상태(Dependency State)는 개발자와 운영자가 관찰할 수 있어야 한다. 진단 시스템은 어떤 선행 조건이 준비 완료(Ready), 대기 중(Waiting), 실패(Failed), 선택 사항(Optional), 복구 중(Recovering)인지 표시하고 특정 노드가 활성화되지 않은 이유를 설명할 수 있어야 한다. 전이 타임스탬프(Transition Timestamp), 종속성 평가 결과(Dependency Evaluation Result), 타임아웃 원인, 기동 소요 시간(Startup Duration)은 느린 초기화, 잘못된 구성, 불안정한 하드웨어, 구성요소 사이의 숨겨진 결합(Hidden Coupling)을 식별하는 데 중요한 정보를 제공한다.

종속성 기반 기동(Dependency-based Startup)은 기능 안전(Functional Safety)과 연계되어야 하지만 이를 대체해서는 안 된다. 제어기(Controller)는 활성화되기 전에 안전 시스템(Safety System)이 운용 허가를 보고하는 것에 의존할 수 있지만 ROS 2 종속성 로직 자체가 인증된 안전 메커니즘(Certified Safety Mechanism)은 아니다. 비상 정지(Emergency Stop), 안전 토크 차단(Safe Torque Off), 보호 정지(Protective Stop), 기타 안전 기능은 해당 상태가 수명주기 준비 상태 판단에 사용되는 경우에도 독립적인 제어 권한을 유지해야 한다.

다중 로봇 시스템(Multi-robot System)에서는 각 로봇마다 자체적인 하드웨어와 소프트웨어 준비 조건이 있으므로 종속성 관리는 기본적으로 로봇 내부에서 수행되어야 한다. 플릿 관리 시스템(Fleet Management)은 모든 내부 전이를 직접 제어하는 대신 준비 완료(Ready), 성능 저하(Degraded), 복구 중(Recovering), 사용 불가(Unavailable)와 같은 요약 상태를 사용할 수 있다. 이를 통해 로봇의 자율적인 내부 관리를 유지하면서 성공적으로 종속성 체인을 완료한 기능을 기준으로 임무를 할당할 수 있다.

결국 종속성 기반 노드 시작 순서 관리(Dependency-based Node Start Order Management)는 로봇 기동을 단순한 프로세스 실행 순서에서 검증된 기능의 단계적 진행(Verified Progression of Capabilities)으로 전환한다. 명시적인 그래프(Explicit Graph), 수명주기 전이(Lifecycle Transition), 준비 상태 검사(Readiness Check), 독립 분기의 병렬 처리(Parallel Independent Branch), 타임아웃, 선택적 종속성, 장애 격리(Failure Containment), 복구, 역순 종료(Reverse-order Shutdown)를 통해 예측 가능한 시스템 동작을 구축하며, 로봇 전원 켜기 및 끄기 시퀀싱(Robot Power-on/off Sequencing)과 수명주기 기반 안전 운용(Lifecycle-based Safe Operation)을 위한 직접적인 기반을 제공한다.

## 07.06 Robot Power On/Off Sequence and ROS2 Integration [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 전원 켜기 및 끄기 시퀀싱(Robot Power-on and Power-off Sequencing)은 물리적인 에너지 상태(Physical Energy State)와 ROS 2 소프트웨어 수명주기 상태(Software Lifecycle State)를 연계하여 하드웨어와 소프트웨어가 제어되고 예측 가능한 순서로 운용 상태에 진입하도록 한다. 로봇은 단순히 모든 장치에 동시에 전원을 공급하고 모든 노드를 한꺼번에 실행해서는 안 된다. 컴퓨팅 시스템, 네트워크, 센서, 액추에이터(Actuator), 제어기(Controller), 애플리케이션 노드는 서로 다른 초기화 요구사항을 가지므로 자율 운용이 시작되기 전에 이를 동기화해야 한다.

전원 시퀀싱(Power Sequencing)은 ROS 2 소프트웨어 계층보다 하위 수준에서 시작된다. 주 전원 분배(Main Power Distribution), DC/DC 변환기(DC/DC Converter), 임베디드 컴퓨터(Embedded Computer), 네트워크 스위치(Network Switch), 센서 전원 레일(Sensor Power Rail), 모터 드라이브(Motor Drive), 안전 제어기(Safety Controller)는 서로 다른 활성화 지연 시간과 준비 상태 검사가 필요할 수 있다. 따라서 소프트웨어 아키텍처는 물리적 전원 영역(Power Domain)을 인식하고 그 상태를 감독 로직(Supervisory Logic)에 제공해야 하며, 단순히 전원이 공급되었다는 이유만으로 장치가 즉시 통신이나 제어에 사용 가능하다고 가정해서는 안 된다.

가장 초기의 기동 단계(Startup Stage)에서는 안전 및 인프라 기반(Safety and Infrastructure Foundation)을 확립해야 한다. 안전 제어기, 비상 정지 회로(Emergency-stop Circuit), 전원 모니터링(Power Monitoring), 통신 인프라(Communication Infrastructure), 필수 컴퓨팅 자원(Essential Computing Resource)은 모션 관련 소프트웨어(Motion-related Software)의 운용이 허가되기 전에 준비되어야 한다. ROS 2 프로세스는 운영체제(Operating System), 네트워크 인터페이스(Network Interface), 시간 동기화(Time Synchronization), 저장장치(Storage), 필요한 미들웨어 서비스(Middleware Service)가 분산 통신에 적합한 안정적인 상태에 도달한 이후에 시작할 수 있다.

컴퓨팅 인프라가 준비되면 ROS 2 수명주기 노드(Lifecycle Node)는 구성 단계(Configuration Phase)에 진입할 수 있다. 하드웨어 드라이버(Hardware Driver)는 미구성(Unconfigured) 상태에서 비활성(Inactive) 상태로 전이하면서 파라미터(Parameter)를 로드하고, 버퍼(Buffer)를 할당하고, 통신 인터페이스(Communication Interface)를 열고, 장치 식별 정보(Device Identity)나 펌웨어 상태(Firmware Status)를 확인할 수 있다. 이 단계에서 노드를 비활성 상태로 유지하면 물리적 하드웨어와 관련 소프트웨어 자원이 검증되기 전에 데이터 발행이나 제어 동작이 시작되는 것을 방지할 수 있다.

센서 전원(Sensor Power)과 소프트웨어 활성화(Software Activation)는 명시적인 준비 상태 조건(Readiness Condition)을 통해 조정되어야 한다. 라이다(LiDAR)는 전원이 공급된 이후 유효한 측정값을 생성하기까지 수 초가 필요할 수 있으며, 카메라(Camera), GNSS 수신기(GNSS Receiver), IMU, 열화상 센서(Thermal Sensor), 깊이 센서(Depth Sensor)는 서로 다른 초기화 특성을 가질 수 있다. 따라서 해당 ROS 2 드라이버는 통신, 진단(Diagnostics), 타임스탬프(Timestamp), 필요한 데이터 스트림(Data Stream)이 정의된 준비 상태 기준을 만족한 이후에만 활성화되어야 한다.

상위 수준 구성요소(Higher-level Component)는 종속성 기반 활성화(Dependency-based Activation)를 따라야 한다. 인지(Perception)는 필요한 센서 스트림이 유효해진 이후 시작되어야 하며, 위치추정(Localization)은 인지 또는 위치추정용 센서와 좌표 변환(Transform)을 사용할 수 있게 된 이후 시작되어야 한다. 내비게이션(Navigation)은 신뢰할 수 있는 자세(Pose)와 지도 환경(Map Environment)이 확립된 이후 시작되어야 한다. 임무 실행(Mission Execution)과 자율 제어(Autonomous Control)는 선택된 운용 모드에 필요한 모든 필수 기능이 준비 상태 검사를 통과한 이후에만 활성(Active) 상태가 되어야 한다.

액추에이터 관련 활성화(Actuator-related Activation)는 소프트웨어 준비 상태가 물리적 움직임의 허가를 자동으로 의미하지 않으므로 추가적인 제어가 필요하다. 모터 제어기(Motor Controller)와 드라이브(Drive)는 전원이 공급된 상태에서도 억제(Inhibited), 비활성(Disabled), 또는 비동작 상태(Non-motion State)를 유지할 수 있다. ROS 2 제어 구성요소는 움직임이 허가되기 전에 구성할 수 있지만 액추에이터 활성화(Actuator Enablement)는 제어 인터페이스, 명령 경로(Command Path), 피드백 신호(Feedback Signal), 안전 조건(Safety Condition), 시스템 수준 준비 상태가 검증된 이후에만 이루어져야 한다.

시스템 수준 오케스트레이터(System-level Orchestrator)는 전원 상태(Power State)와 수명주기 전이(Lifecycle Transition)를 연결할 수 있다. 오케스트레이터는 구성(Configure) 또는 활성화(Activate) 전이를 요청하기 전에 전원 제어기 피드백(Power-controller Feedback), 하드웨어 진단, 네트워크 연결 상태(Network Connectivity), 수명주기 상태를 관찰할 수 있다. 고정된 지연 시간에 의존하는 대신 명시적인 조건이 충족될 때 다음 기동 단계로 진행하고, 타임아웃(Timeout)을 이용하여 허용된 초기화 시간 내에 준비되지 못한 하드웨어 또는 소프트웨어 구성요소를 탐지할 수 있다.

전원 켜기 시퀀싱(Power-on Sequencing)은 종속성이 허용하는 경우 병렬 초기화(Parallel Initialization)를 지원해야 한다. 서로 독립적인 센서 전원 영역이나 컴퓨팅 서비스는 동시에 시작할 수 있지만 제한된 전원 공급 장치, 통신 버스(Communication Bus), 열 자원(Thermal Resource)을 공유하는 구성요소는 단계적인 활성화가 필요할 수 있다. 제어된 병렬 처리(Controlled Parallelism)는 기동 시간을 단축하지만 어떤 장치를 함께 안전하게 시작할 수 있는지를 정의할 때 돌입 전류(Current Surge), 네트워크 혼잡(Network Congestion), 초기화 트래픽(Initialization Traffic), 공유 자원 충돌(Shared-resource Conflict)을 고려해야 한다.

전원 켜기 과정에서 장애가 발생하면 로봇이 불완전한 운용 상태로 계속 진행하도록 허용하는 대신 적절한 경계에서 진행을 중지해야 한다. 필수 센서(Mandatory Sensor)가 초기화되지 않으면 종속된 위치추정 또는 내비게이션 노드는 비활성(Inactive) 상태를 유지해야 한다. 반면 선택적 장치(Optional Device)는 사용할 수 없는 상태로 표시하면서 로봇이 정의된 성능 저하 모드(Degraded Mode)로 진입하도록 할 수 있다. 진단 정보에는 장애가 발생한 전원 영역, 하드웨어 구성요소, 수명주기 노드, 차단된 종속성(Blocked Dependency)이 명확하게 표시되어야 한다.

런타임 전원 관리(Runtime Power Management)에서도 동일한 통합 원칙을 사용할 수 있다. 로봇은 에너지 소비를 줄이기 위해 사용하지 않는 센서, 가속기(Accelerator), 페이로드(Payload), 통신 장치의 전원을 선택적으로 차단하면서 해당 ROS 2 노드를 비활성(Inactive) 또는 미구성(Unconfigured) 상태로 전환할 수 있다. 해당 기능이 다시 필요해지면 전체 로봇을 재시작하지 않고 하드웨어 전원을 복구하고 준비 상태를 검증한 후 관련 노드를 재구성(Reconfigure)하거나 재활성화(Reactivation)할 수 있다.

전원 끄기 시퀀싱(Power-off Sequencing)은 단순히 전기 전원을 차단하는 대신 일반적으로 기능적 종속성 순서(Functional Dependency Order)를 역방향으로 적용해야 한다. 임무 실행을 가장 먼저 중지하고 그 다음 내비게이션과 상위 수준 자율 동작(Autonomous Behavior)을 중지해야 한다. 이후 인지와 위치추정을 비활성화하고, 센서 데이터 획득(Sensor Acquisition)을 중지하고, 하드웨어 인터페이스를 정리할 수 있다. 소프트웨어 소비자(Software Consumer)가 종속성을 해제한 이후에만 해당 물리 장치와 전원 영역의 전원을 차단해야 한다.

액추에이터는 특히 제어된 종료 경로(Controlled Shutdown Path)가 필요하다. 모션 명령(Motion Command)을 중단하고, 로봇이 요구되는 정지 상태(Stopped Condition)에 도달하고, 제어기가 적절한 비운용 상태(Non-operational State)로 전환된 후 시스템 설계에 따라 드라이브 활성화 신호(Drive Enable Signal)를 제거해야 한다. 이후 ROS 2 노드는 인터페이스를 비활성화하고 정리할 수 있다. 하드웨어가 제어된 동작을 위해 활성 소프트웨어 명령에 더 이상 의존하지 않는 상태가 된 이후에만 전기적 전원 차단(Electrical Power Removal)을 수행해야 한다.

컴퓨팅 플랫폼(Computing Platform)은 일반적으로 애플리케이션 종료(Application Shutdown), 로그 기록 완료(Log Flushing), 영구 상태 저장(Persistent-state Storage), 수명주기 정리(Lifecycle Cleanup), 진단 보고(Diagnostic Reporting)가 완료될 때까지 전원이 유지되어야 한다. 메인 컴퓨터의 전원을 갑자기 제거하면 장애 정보를 잃거나, 완료되지 않은 데이터 트랜잭션(Data Transaction)이 손상되거나, 올바른 하드웨어 종료를 수행하지 못할 수 있다. 따라서 최종 소프트웨어 단계에서는 전원 제어기가 비필수 컴퓨팅 전원을 차단하기 전에 영구 저장 작업이 완료되었는지 확인해야 한다.

수명주기 콜백(Lifecycle Callback)은 전원 통합을 위한 유용한 경계를 제공한다. \`on_configure()\`는 필요한 하드웨어가 존재하는지 검증할 수 있고, \`on_activate()\`는 운용 통신(Operational Communication)을 활성화할 수 있으며, \`on_deactivate()\`는 런타임 활동을 중지하고, \`on_cleanup()\`은 인터페이스를 닫고 자원을 해제할 수 있다. 그러나 수명주기 콜백이 하드 실시간 전기 제어(Hard Real-time Electrical Control)나 인증된 안전 동작(Certified Safety Behavior)을 제공한다고 가정해서는 안 되며, 이는 전용 하드웨어 메커니즘을 중심으로 소프트웨어 상태를 조정하는 역할을 한다.

전원 시퀀싱은 기능 안전(Functional Safety)과 명확하게 구분되어야 한다. 비상 정지 회로, 안전 토크 차단(Safe Torque Off), 안전 PLC(Safety PLC), 접촉기(Contactor), 기타 안전 메커니즘은 ROS 2와 독립적으로 위험 에너지(Hazardous Energy)를 제거하거나 억제할 수 있어야 한다. 수명주기 오케스트레이션(Lifecycle Orchestration)은 안전 상태를 관찰하고 소프트웨어를 비활성화하여 대응할 수 있지만, 안전 시스템이 보호 기능을 수행하기 전에 소프트웨어 전이 완료를 기다리도록 해서는 절대 안 된다.

관측 가능성(Observability)은 기동 및 종료 장애가 하드웨어와 소프트웨어 경계를 넘나드는 경우가 많기 때문에 매우 중요하다. 로그는 일관된 타임스탬프를 사용하여 전원 명령(Power Command), 전압 또는 장치 준비 신호(Device-ready Signal), 네트워크 가용성(Network Availability), ROS 2 수명주기 전이, 준비 상태 검사, 타임아웃 이벤트, 장애 원인을 서로 연계해야 한다. 이러한 상관관계를 통해 엔지니어는 노드 장애가 소프트웨어 로직 때문인지 또는 물리적 장치의 전원이 정상적으로 공급되거나 초기화되지 않았기 때문인지 판단할 수 있다.

예기치 않은 전원 중단(Unexpected Power Interruption) 이후의 복구는 이전 소프트웨어 상태가 여전히 유효하다고 가정하는 대신 정의된 재진입 절차(Re-entry Procedure)를 따라야 한다. 영향을 받은 노드는 비활성화하고, 오래된 인터페이스(Stale Interface)를 정리하고, 하드웨어 전원이 복원될 때까지 기다린 후 통신을 재구성하고, 장치 상태를 검증하고, 다시 활성화해야 할 수 있다. 종속 구성요소는 복원된 기능이 정상적인 기동 과정에서 사용했던 것과 동일한 준비 상태 검사를 통과할 때까지 일시 중지된 상태를 유지해야 한다.

다중 로봇 시스템(Multi-robot System)에서는 전원 영역과 하드웨어 종속성이 각 로봇 내부에 존재하므로 일반적으로 각 로봇이 자체적인 내부 전원 및 수명주기 시퀀스를 관리해야 한다. 플릿 관리 시스템(Fleet Management)은 기동 중(Booting), 준비 완료(Ready), 운용 중(Operational), 성능 저하(Degraded), 종료 중(Shutting Down), 사용 불가(Unavailable)와 같은 요약 상태를 전달받을 수 있다. 이를 통해 플릿 수준 제어기(Fleet-level Controller)에 모든 내부 전원 전이를 노출하지 않고도 임무 할당과 충전 운용(Charging Operation)에 로봇 준비 상태를 반영할 수 있다.

통합 로봇 전원 시퀀싱(Integrated Robot Power Sequencing)은 궁극적으로 전기적 준비 상태(Electrical Readiness), 하드웨어 초기화(Hardware Initialization), ROS 2 수명주기 관리(Lifecycle Management), 종속성 순서 관리(Dependency Ordering), 안전 감독(Safety Supervision), 애플리케이션 활성화(Application Activation)를 하나의 협조된 운용 절차(Coordinated Operational Procedure)로 연결한다. 잘 설계된 시퀀스는 결정론적 기동(Deterministic Startup), 제어된 종료(Controlled Shutdown), 에너지 인식 런타임 운용(Energy-aware Runtime Operation), 장애 격리(Fault Containment), 신뢰성 있는 복구(Reliable Recovery)를 가능하게 하며, 진단 경보(Diagnostic Alert), 수명주기 모니터링(Lifecycle Monitoring), 다중 로봇 동기화(Multi-robot Synchronization), 안전 상태 통합(Safe-state Integration)을 위한 기반을 제공한다.

## 07.07 Lifecycle Event-Based Diagnostic Alert System [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

수명주기 이벤트 기반 진단 경보 시스템(Lifecycle Event-based Diagnostic Alert System)은 ROS 2 관리형 노드(Managed Node)의 상태 정보와 런타임 건전성 모니터링(Runtime Health Monitoring)을 결합하여 비정상 조건을 운용 상황에 맞게 탐지할 수 있도록 한다. 구성요소가 기동 중인지, 활성 상태인지, 복구 중인지, 종료 중인지 알지 못한 채 개별 오류만 보고하는 대신 진단 계층(Diagnostic Layer)은 장애를 수명주기 상태(Lifecycle State) 및 전이(Transition)와 연계한다. 이를 통해 운영자와 자동 감독기(Automated Supervisor)가 경보의 의미를 더욱 정확하게 이해할 수 있다.

수명주기 이벤트(Lifecycle Event)는 미구성(Unconfigured), 비활성(Inactive), 활성(Active), 최종화(Finalized)와 같은 상태 사이의 변화를 명시적으로 나타내는 정보 흐름을 제공한다. 구성(Configure), 활성화(Activate), 비활성화(Deactivate), 정리(Cleanup), 종료(Shutdown), 오류(Error) 전이는 노드가 어떤 작업을 시도하고 있으며 언제 운용 역할이 변경되는지를 나타낸다. 이러한 이벤트를 모니터링하면 정상적인 상태 변화와 예상하지 못한 전이, 장애, 완료되지 않은 전이, 비정상적인 복구 동작을 구분할 수 있다.

수명주기 상태만으로 구성요소의 건전성(Component Health)을 판단하기에는 충분하지 않다. 센서 데이터가 중단되거나, 처리 지연 시간(Processing Latency)이 허용 한계를 초과하거나, 하드웨어 인터페이스(Hardware Interface)를 사용할 수 없는 상황에서도 노드는 활성(Active) 상태를 유지할 수 있다. 따라서 진단 모니터링은 수명주기 이벤트와 함께 하트비트 상태(Heartbeat Status), 토픽 주기(Topic Frequency), 서비스 응답성(Service Responsiveness), 하드웨어 진단, 자원 사용량(Resource Usage), 데이터 유효성(Data Validity), 콜백 실행 동작(Callback Execution Behavior)과 같은 애플리케이션 수준의 증거를 결합해야 한다.

각 진단 관측(Diagnostic Observation)은 노드의 현재 수명주기 상태에 따라 해석되어야 한다. 노드가 비활성(Inactive) 상태일 때 센서 데이터가 없는 것은 정상일 수 있지만 동일한 노드가 활성(Active) 상태일 때 데이터가 없다면 심각한 문제가 될 수 있다. 구성 중에 데이터를 발행하지 않는 것은 정상일 수 있지만 비활성화 이후에도 데이터가 계속 발행된다면 잘못된 수명주기 동작을 의미할 수 있다. 상태 인식 해석(State-aware Interpretation)은 현재 구성요소가 존재하는 운용 상황에 맞게 진단 규칙을 평가하므로 잘못된 경보(False Alarm)를 줄일 수 있다.

이벤트 기반 아키텍처(Event-based Architecture)는 수명주기 전이를 진단 평가(Diagnostic Evaluation)의 트리거(Trigger)로 사용할 수 있다. 노드가 활성(Active) 상태에 진입하면 모니터링 시스템은 예상 데이터율(Expected Data Rate), 하드웨어 응답, 애플리케이션 건전성을 검사하기 시작할 수 있다. 노드가 비활성(Inactive) 상태가 되면 런타임 검사를 중단하거나 자원 유지 상태 검사(Resource-retention Check)로 대체할 수 있다. 구성 또는 정리 과정에서는 전이 소요 시간(Transition Duration)과 콜백 완료 여부를 모니터링하여 타임아웃이나 장애를 탐지할 수 있다.

진단 심각도(Diagnostic Severity)는 탐지된 조건과 그 조건이 운용에 미치는 영향을 함께 반영해야 한다. 정보 이벤트(Informational Event)는 정상적인 수명주기 전이를 기록할 수 있고, 경고(Warning)는 준비 지연이나 선택적 기능의 성능 저하를 나타낼 수 있으며, 오류(Error)는 필수 기능의 손실을 의미할 수 있다. 치명적 조건(Critical Condition)은 시스템 수준의 개입이 필요한 장애를 나타낼 수 있다. 심각도는 단순한 수명주기 상태가 아니라 정의된 시스템 정책(System Policy)에 따라 결정되어야 한다.

경보 생성(Alert Generation)에는 즉각적인 진단을 지원할 수 있는 충분한 상황 정보(Context)가 포함되어야 한다. 유용한 경보에는 노드 이름, 타임스탬프(Timestamp), 수명주기 상태, 요청된 전이, 진단 조건(Diagnostic Condition), 심각도, 영향을 받는 기능(Affected Capability), 재시도 횟수(Retry Count), 관련 종속성(Dependency)이 포함될 수 있다. 가능하다면 장애가 기동(Startup), 정상 운용(Normal Operation), 복구(Recovery), 유지보수(Maintenance), 종료(Shutdown) 중 어느 과정에서 발생했는지도 식별하여 운영자가 전체 시스템 이력을 직접 재구성하지 않고도 이벤트를 이해할 수 있도록 해야 한다.

전이 타임아웃(Transition Timeout)은 중요한 수명주기 경보의 원인이다. 구성 과정은 하드웨어를 기다리면서 차단될 수 있고, 활성화는 사용할 수 없는 자원을 기다릴 수 있으며, 정리는 인터페이스 해제에 실패할 수 있다. 모니터링 시스템은 예상 한계(Expected Limit)를 기준으로 전이 소요 시간을 측정하고 콜백이 허용 시간을 초과하면 경보를 생성해야 한다. 이후 타임아웃 이벤트는 시스템을 중간 상태에 무기한 남겨두는 대신 복구 로직(Recovery Logic)을 시작하는 데 사용할 수 있다.

반복적인 수명주기 전이(Repeated Lifecycle Transition)는 개별 복구 시도가 성공하더라도 시스템의 불안정성을 나타낼 수 있다. 노드가 활성(Active)에서 비활성(Inactive)으로 전환된 후 다시 활성 상태로 반복적으로 이동한다면 간헐적인 하드웨어, 통신, 소프트웨어 문제를 의미할 수 있다. 따라서 진단 시스템은 활성 상태로 성공적으로 복귀한 각각의 상황을 근본적인 문제가 해결되었다는 증거로 간주하지 않고 시간에 따른 전이 빈도(Transition Frequency), 재시작 횟수(Restart Count), 복구 횟수(Recovery Count), 장애 재발(Fault Recurrence)을 추적해야 한다.

진단 경보(Diagnostic Alert)는 자동 복구 정책(Automatic Recovery Policy)의 입력으로 활용할 수 있다. 경고는 단순한 관찰만 필요할 수 있지만 오류는 비활성화와 재활성화를 시작할 수 있다. 더욱 지속적인 장애는 정리 및 재구성(Reconfiguration), 프로세스 재시작(Process Restart), 서브시스템 격리(Subsystem Isolation), 성능 저하 운용(Degraded Operation)을 유발할 수 있다. 경보 시스템은 복구 관리자가 비구조화된 로그 메시지가 아니라 분류된 조건(Classified Condition)을 기반으로 복구 결정을 내릴 수 있도록 구조화된 장애 정보(Structured Fault Information)를 제공해야 한다.

종속성 정보(Dependency Information)는 경보 해석을 향상시킨다. 상위 라이다(LiDAR) 드라이버에 장애가 발생하면 하위 인지(Perception), 위치추정(Localization), 내비게이션(Navigation) 노드는 각각 독립적으로 장애가 발생한 것이 아니라 그 결과로 사용할 수 없게 될 수 있다. 진단 시스템은 장애의 잠재적인 최초 발생 지점(Probable Originating Fault)을 식별하고 주요 경보(Primary Alert)와 종속성에 의해 전파된 영향(Propagated Dependency Effect)을 구분해야 한다. 이를 통해 경보 폭주(Alert Storm)를 줄이고 운영자가 실제로 조치가 필요한 구성요소에 집중할 수 있다.

대규모 ROS 2 시스템에서는 경보 억제(Alert Suppression)와 집계(Aggregation)가 필요하다. 하나의 하드웨어 장애가 종속 노드 전체에서 수백 개의 관련 경고를 발생시킬 수 있다. 상태 인식 규칙(State-aware Rule)은 서브시스템이 의도적으로 비활성(Inactive) 또는 복구 중(Recovering)일 때 예상되는 이차 경보(Secondary Alert)를 억제할 수 있으며, 집계를 통해 반복되는 이벤트를 하나의 요약된 사고(Summarized Incident)로 결합할 수 있다. 그러나 중요한 심각도 변화나 복구 결과는 억제 로직에 의해 숨겨지지 않고 계속 확인할 수 있어야 한다.

수명주기 기반 진단(Lifecycle-based Diagnostics)은 타임스탬프 기반 이벤트 이력(Timestamped Event History)을 유지해야 한다. 상태 전이, 진단 상태 변화, 복구 명령(Recovery Command), 프로세스 재시작, 준비 상태 결과(Readiness Result), 운영자 조치(Operator Action)를 서로 연계된 타임라인(Timeline)으로 저장할 수 있다. 이를 통해 장애가 로봇 전체에 어떻게 전파되었는지, 자동 복구가 올바르게 동작했는지를 사후 분석할 수 있다. 여러 컴퓨터나 분산 서브시스템에서 이벤트가 발생하는 경우 일관된 시간 동기화(Time Synchronization)가 특히 중요하다.

진단 시스템은 구성요소 수준(Component-level)과 시스템 수준(System-level)의 관점을 모두 제공해야 한다. 개발자는 개별 수명주기 콜백과 노드 진단에 대한 상세 정보가 필요할 수 있지만 운영자는 준비 완료(Ready), 운용 중(Operational), 성능 저하(Degraded), 복구 중(Recovering), 유지보수(Maintenance), 사용 불가(Unavailable)와 같은 요약된 상태가 필요하다. 감독 계층(Supervisory Layer)은 여러 수명주기 상태, 건전성 지표(Health Indicator), 종속성 관계, 해결되지 않은 경보를 종합하여 이러한 시스템 상태를 도출할 수 있다.

관측 가능성 인터페이스(Observability Interface)에는 ROS 2 진단 토픽(Diagnostic Topic), 수명주기 전이 이벤트, 구조화된 로그(Structured Log), 대시보드(Dashboard), 영구 이벤트 저장소(Persistent Event Storage), 원격 모니터링 서비스(Remote Monitoring Service)가 포함될 수 있다. 목적은 단순히 더 많은 메시지를 생성하는 것이 아니라 상태, 건전성, 시간, 운용 영향(Operational Consequence) 사이의 관계를 보존하는 것이다. 일관된 식별자(Identifier)와 타임스탬프를 사용하면 디버깅과 플릿 운용 과정에서 이러한 다양한 채널의 정보를 서로 연계할 수 있다.

진단 경보 로직(Diagnostic Alert Logic) 자체도 모니터링되어야 한다. 모니터링 노드가 수명주기 이벤트 수신을 중단하거나, 서브시스템과의 통신을 상실하거나, 과부하 상태가 되더라도 경보가 발생하지 않는 상황을 정상 운용의 증거로 해석해서는 안 된다. 하트비트(Heartbeat), 감시기(Watchdog), 중복 감독(Redundant Supervision), 외부 프로세스 모니터링(External Process Monitoring)을 통해 로봇이 운용되는 동안 진단 인프라가 계속 사용 가능한지 검증할 수 있다.

안전 관련 경보(Safety-related Alert)는 진단 알림(Diagnostic Notification)과 안전 권한(Safety Authority) 사이의 명확한 경계를 필요로 한다. 진단 시스템은 비상 정지(Emergency Stop), 안전 인터록(Safety Interlock), 안전 토크 차단(Safe Torque Off), 안전 제어기 장애(Safety-controller Fault)를 보고하고 소프트웨어 비활성화를 조정할 수 있지만 독립적인 안전 메커니즘을 대체해서는 안 된다. 안전 기능은 ROS 2 이벤트 처리, 경보 생성, 수명주기 전이 처리가 가능한지 여부와 관계없이 동작해야 한다.

다중 로봇 배포(Multi-robot Deployment)에서는 지역 진단 시스템(Local Diagnostic System)이 상세한 수명주기 이벤트를 처리하면서 요약된 사고 정보(Incident)와 로봇 가용성(Robot Availability)을 플릿 관리 시스템(Fleet Management)에 전달할 수 있다. 이를 통해 플릿 인프라가 모든 내부 노드 이벤트로 과부하되는 것을 방지할 수 있다. 플릿 수준 모니터링은 준비 완료(Ready), 성능 저하(Degraded), 복구 중(Recovering), 사용 불가(Unavailable) 상태의 로봇을 식별하고 이 정보를 임무 할당(Mission Allocation), 유지보수 계획(Maintenance Planning), 운용 감독(Operational Supervision)에 활용할 수 있다.

결국 수명주기 이벤트 기반 진단 경보 시스템(Lifecycle Event-based Diagnostic Alert System)은 수명주기 전이와 런타임 건전성 신호(Runtime Health Signal)를 상황 인식형 운용 정보(Contextual Operational Intelligence)로 변환한다. 상태 인식 규칙(State-aware Rule), 전이 모니터링(Transition Monitoring), 심각도 분류(Severity Classification), 종속성 상관관계(Dependency Correlation), 경보 집계(Alert Aggregation), 이벤트 이력(Event History), 복구 통합(Recovery Integration), 시스템 수준 보고(System-level Reporting)를 통해 장애를 더욱 쉽게 탐지하고 해석할 수 있다. 이는 다중 로봇 수명주기 동기화(Multi-robot Lifecycle Synchronization), 안전 상태 진입(Safe-state Entry), 종료 복구(Shutdown Recovery), 수명주기 통합 시험(Lifecycle Integration Testing)을 위한 기반을 제공한다.

## 07.08 Multi-Robot Lifecycle Synchronization Strategy

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

다중 로봇 수명주기 동기화(Multi-robot Lifecycle Synchronization)는 ROS 2 수명주기 관리(Lifecycle Management)를 단일 로봇에서 여러 로봇이 협조하는 플릿(Fleet)으로 확장하며, 각 로봇은 로컬 운용 제어(Local Operational Control)를 유지하면서 선택된 수명주기 정보를 상위 수준 오케스트레이션(Higher-level Orchestration)과 공유한다. 목적은 모든 로봇의 모든 노드를 동일한 상태로 강제하는 것이 아니라, 협조 동작에 공통된 운용 조건이 필요한 경우 준비 상태(Readiness), 임무 참여(Mission Participation), 복구(Recovery), 유지보수(Maintenance), 종료(Shutdown)를 동기화하는 것이다.

각 로봇은 일반적으로 자체적인 로컬 수명주기 관리자(Local Lifecycle Manager)를 유지해야 한다. 하드웨어 초기화(Hardware Initialization), 센서 준비 상태(Sensor Readiness), 제어기 활성화(Controller Activation), 내부 종속성(Internal Dependency)은 해당 로봇에 속하기 때문이다. 플릿 계층(Fleet Layer)은 모든 내부 수명주기 노드를 직접 관리하지 않아야 한다. 대신 각 로봇은 전체적인 임무 수행 능력을 나타내기 위해 기동 중(Booting), 준비 완료(Ready), 운용 중(Operational), 성능 저하(Degraded), 복구 중(Recovering), 유지보수(Maintenance), 종료 중(Shutting Down), 사용 불가(Unavailable)와 같은 요약 상태를 외부에 제공할 수 있다.

계층형 아키텍처(Hierarchical Architecture)는 실용적인 동기화 구조를 제공한다. 개별 수명주기 노드는 인지(Perception), 내비게이션(Navigation), 제어(Control), 조작(Manipulation), AI 처리(AI Processing)를 담당하는 서브시스템 관리자(Subsystem Manager)에 의해 관리된다. 이러한 관리자는 서브시스템 준비 상태를 로봇 수준 오케스트레이터(Robot-level Orchestrator)에 보고하고, 오케스트레이터는 로봇의 운용 상태를 도출한다. 이후 플릿 수준 조정기(Fleet-level Coordinator)는 로봇 수준 상태를 받아 공유 임무, 자원, 집단 동작에 영향을 주는 전이만 조정한다.

동기화는 단순한 프로세스 존재 여부가 아니라 기능(Capability)을 기반으로 해야 한다. 로봇은 필수 센서, 위치추정(Localization), 내비게이션, 통신, 안전 관련 선행 조건(Safety-related Prerequisite)이 정의된 준비 기준을 만족한 이후에만 준비 완료(Ready)를 보고할 수 있다. 다른 로봇은 선택적 센서(Optional Sensor)를 사용할 수 없어 성능 저하(Degraded) 상태이더라도 특정 임무는 수행할 수 있다. 따라서 기능 중심 동기화(Capability-oriented Synchronization)를 사용하면 이기종 로봇(Heterogeneous Robot)이 실제 수행 가능한 기능에 따라 임무에 참여할 수 있다.

시간 동기화(Time Synchronization)는 여러 로봇에서 발생하는 수명주기 이벤트를 해석하기 위한 중요한 기반이다. 전이 타임스탬프(Transition Timestamp), 진단 경보(Diagnostic Alert), 복구 이벤트(Recovery Event), 임무 명령(Mission Command), 준비 상태 보고(Readiness Report)는 충분히 일관된 시간 기준(Time Reference)을 사용해야 한다. 시스템 요구사항에 따라 PTP, GNSS 기반 타이밍(GNSS-derived Timing), NTP 또는 기타 동기화 메커니즘을 선택할 수 있다. 일관된 시간이 없으면 분산 이벤트의 발생 순서를 재구성하고 협조 장애(Coordination Failure)를 진단하기 어려워진다.

플릿 동기화(Fleet Synchronization)는 고정된 대기 시간(Fixed Waiting Period)이 아니라 명시적인 상태 보고(State Report)와 확인 응답(Acknowledgement)을 사용해야 한다. 협조 임무에 여러 로봇이 필요한 경우 플릿 관리자는 준비를 요청하고 필요한 각 로봇이 요구되는 준비 조건을 보고할 때까지 기다릴 수 있다. 동기화 정책(Synchronization Policy)이 충족된 이후에만 임무 실행을 시작한다. 이를 통해 미리 정의된 기동 시간이 지났다는 이유만으로 모든 로봇이 준비되었다고 가정하는 문제를 방지할 수 있다.

모든 다중 로봇 운용에서 모든 로봇이 동시에 동일한 상태에 도달할 필요는 없다. 일부 임무에서는 참여하는 모든 로봇이 준비 완료(Ready)가 되어야 실행을 시작할 수 있는 배리어 방식 동기화 지점(Barrier-style Synchronization Point)이 필요할 수 있다. 다른 임무에서는 최소 로봇 수 또는 필요한 기능 조합만 충족하면 되는 부분 동기화(Partial Synchronization)를 사용할 수 있다. 따라서 동기화 전략은 불필요한 전역 배리어(Global Barrier)를 강제하는 대신 임무 의미론(Mission Semantics)을 반영해야 한다.

배리어 동기화(Barrier Synchronization)는 협동 운송(Cooperative Transport), 대형 이동(Formation Movement), 동기화 검사(Synchronized Inspection), 공유 조작(Shared Manipulation)과 같이 긴밀하게 협조되는 작업에 유용하다. 각 참여자는 정의된 준비 상태에 도달한 후 필요한 그룹이 모두 준비될 때까지 논리적 배리어(Logical Barrier)에서 대기한다. 이후 조정기(Coordinator)가 그룹을 다음 운용 단계로 진행시킨다. 하나의 사용 불가능한 로봇이 전체 임무를 무기한 차단하지 않도록 타임아웃(Timeout)을 반드시 적용해야 한다.

느슨한 동기화(Loose Synchronization)는 독립적이거나 결합도가 낮은 플릿 작업에 더 적합하다. 로봇은 서로 다른 시간에 준비 완료 상태가 되어 임무를 수락할 수 있으며, 플릿 관리자는 자원 가용성(Resource Availability)을 지속적으로 갱신한다. 복구 또는 유지보수 중인 로봇은 관련 없는 다른 로봇의 수명주기 상태를 변경하도록 강제하지 않고 일시적으로 가용 로봇 풀(Available Pool)에서 제외될 수 있다. 이 모델은 실제 운용 종속성이 존재하는 부분에만 동기화를 적용하므로 확장성(Scalability)을 향상시킨다.

임무 결정이 세부 기능에 의존하는 경우 로봇 상태 보고(Robot State Report)에는 하나의 수명주기 레이블보다 많은 정보가 포함되어야 한다. 간결한 상태 메시지(Status Message)에는 로봇 식별 정보(Robot Identity), 시스템 상태(System State), 사용 가능한 기능(Available Capability), 성능 저하 기능(Degraded Function), 현재 복구 상태(Current Recovery State), 타임스탬프, 활성 임무(Active Mission), 준비 상태 세대 또는 시퀀스 정보(Readiness Generation or Sequence Information)를 포함할 수 있다. 이러한 메타데이터(Metadata)는 플릿 조정기가 최신 정보와 오래된 보고(Stale Report)를 구분하고 특정 임무 요구사항을 로봇이 충족할 수 있는지 판단하도록 지원한다.

통신 손실(Communication Loss)은 명시적으로 처리되어야 한다. 플릿 조정기는 마지막으로 보고된 상태가 무기한 유효하다고 가정할 수 없기 때문이다. 하트비트(Heartbeat), 리스(Lease), 상태 만료 시간(Status Expiration Time), 세션 메커니즘(Session Mechanism)을 사용하여 로봇이 보고한 준비 상태를 얼마 동안 신뢰할 수 있는지 정의할 수 있다. 통신이 만료되면 플릿은 해당 로봇을 사용 불가(Unavailable) 또는 알 수 없음(Unknown)으로 표시하고, 로봇의 로컬 감독기(Local Supervisor)는 현재 임무를 안전하게 계속 수행할 수 있는지 독립적으로 판단한다.

동기화 로직(Synchronization Logic)은 지연되거나 중복되거나 순서가 뒤바뀐 분산 메시지를 허용할 수 있어야 한다. 상태 보고와 전이 요청에는 타임스탬프, 시퀀스 번호(Sequence Number), 트랜잭션 식별자(Transaction Identifier), 세대 카운터(Generation Counter)를 포함하여 오래된 정보가 최신 상태를 덮어쓰지 않도록 할 수 있다. 특히 멱등성 명령(Idempotent Command)은 네트워크 재시도(Network Retry)가 발생하더라도 반복된 요청이 안전하지 않거나 일관되지 않은 전이를 발생시키지 않도록 하기 때문에 중요하다.

하나의 로봇에서 발생한 장애는 일반적으로 관련 없는 다른 로봇으로부터 격리되어야 한다. 느슨하게 결합된 검사 임무(Loosely Coupled Inspection Mission)에서 하나의 로봇이 복구 중(Recovering) 상태에 진입하면 플릿 관리자는 해당 로봇의 미완료 작업을 재할당하면서 다른 로봇의 운용을 계속할 수 있다. 긴밀하게 결합된 임무에서는 영향을 받은 그룹이 일시 중지하거나 재구성하거나 정의된 동기화 지점으로 복귀해야 할 수 있다. 따라서 복구 정책(Recovery Policy)은 임무 참여자 사이의 결합도(Coupling)에 따라 결정되어야 한다.

복구 이후 로봇은 단순히 프로세스가 재시작되었다는 이유만으로 즉시 협조 운용에 다시 참여해서는 안 된다. 로컬 수명주기 관리자는 필요한 노드를 복원하고, 하드웨어 및 소프트웨어 준비 상태를 검증하고, 진단 상태를 확인한 후 갱신된 로봇 수준 상태를 발행해야 한다. 플릿 조정기는 현재 임무에서 요구하는 기능이 다시 검증된 이후에만 해당 로봇을 운용에 재투입할 수 있다.

플릿 전체 기동(Fleet-wide Startup)은 내부 노드를 직접 제어하지 않고도 단계적으로 구성할 수 있다. 먼저 인프라 서비스(Infrastructure Service)와 통신을 사용할 수 있도록 한 다음 각각의 로봇이 독립적으로 기동할 수 있다. 각 로봇은 자체적인 종속성 기반 수명주기 시퀀스(Dependency-based Lifecycle Sequence)를 수행하고 완료되면 준비 완료(Ready)를 보고한다. 이후 플릿 계층은 필요한 로봇 수 또는 기능 집합(Capability Set)이 지정된 동기화 조건에 도달하면 임무 할당이나 집단 운용(Collective Operation)을 활성화한다.

협조 종료(Coordinated Shutdown)도 유사한 계층적 원칙을 따른다. 플릿 관리자는 새로운 임무 할당을 중단하고 참여 로봇에게 현재 작업을 완료하거나 중단하도록 요청한 후 종료 준비 상태(Shutdown-ready Condition)로 전환하도록 명령할 수 있다. 이후 각 로봇은 자체적인 역방향 종속성 시퀀스(Reverse Dependency Sequence)를 수행하여 필요에 따라 임무 소프트웨어, 내비게이션, 인지, 센서, 하드웨어 인터페이스를 비활성화한 다음 자체 전원 종료 절차(Power-off Procedure)를 완료한다.

시스템 요구사항이 지속적인 로컬 자율성(Local Autonomy)을 요구하는 경우 다중 로봇 동기화는 단일 운용 장애 지점(Single Point of Operational Failure)을 만들지 않아야 한다. 플릿 조정기가 손실되더라도 자동으로 제어되지 않는 동작이 발생해서는 안 된다. 로봇은 애플리케이션 및 안전 아키텍처(Safety Architecture)에 따라 안전한 로컬 작업 완료, 임무 실행 중단, 대기 상태 복귀, 사전에 정의된 안전 상태(Safe Condition) 진입과 같은 통신 손실 대응 정책을 가져야 한다.

수명주기 관리가 여러 시스템으로 확장될수록 관측 가능성(Observability)은 더욱 중요해진다. 플릿 모니터링(Fleet Monitoring)은 일관된 식별자와 타임스탬프를 사용하여 로봇 수준 상태, 전이 요청, 확인 응답, 타임아웃, 통신 품질(Communication Quality), 복구 이벤트, 임무 결정을 서로 연계해야 한다. 운영자는 모든 내부 노드를 개별적으로 조사하지 않고도 어떤 로봇이 동기화를 차단했는지, 어떤 기능을 사용할 수 없었는지, 플릿이 이에 어떻게 대응했는지를 판단할 수 있어야 한다.

확장성을 확보하려면 플릿 수준에서 불필요한 수명주기 트래픽(Lifecycle Traffic)을 제한해야 한다. 상세한 노드 전이와 진단 정보는 로컬에 유지하면서 요약된 로봇 상태와 중요한 이벤트만 상위 계층으로 전송할 수 있다. 이벤트 기반 갱신(Event-driven Update)은 중요한 변화를 즉시 보고하고, 주기적인 하트비트는 지속적인 가용성을 확인한다. 이러한 계층적 필터링(Hierarchical Filtering)은 수백 대의 로봇과 수천 개의 노드가 중앙 조정 인프라(Central Coordination Infrastructure)에 과도한 부하를 발생시키는 것을 방지한다.

안전 권한(Safety Authority)은 로컬에 유지되어야 하며 플릿 동기화와 독립적으로 동작해야 한다. 플릿 명령은 비상 정지(Emergency Stop), 안전 토크 차단(Safe Torque Off), 보호 정지(Protective Stop), 로컬 장애물 보호(Local Obstacle Protection), 기타 전용 안전 기능을 절대로 무시하거나 재정의해서는 안 된다. 플릿 수명주기 조정(Fleet Lifecycle Coordination)은 운용 전이를 요청하고 안전 상태에 대응할 수 있지만, 네트워크 통신이나 중앙 오케스트레이션(Centralized Orchestration)을 사용할 수 없는 상황에서도 각 로봇의 안전 메커니즘은 자체적인 권한을 유지해야 한다.

결국 다중 로봇 수명주기 동기화(Multi-robot Lifecycle Synchronization)는 노드 상태(Node State)에서 서브시스템 준비 상태(Subsystem Readiness), 로봇 기능(Robot Capability), 플릿 조정(Fleet Coordination)으로 이어지는 계층 구조를 형성한다. 로컬 수명주기 자율성(Local Lifecycle Autonomy), 기능 기반 상태(Capability-based Status), 동기화된 시간(Synchronized Time), 배리어(Barrier), 확인 응답(Acknowledgement), 통신 리스(Communication Lease), 장애 격리(Failure Isolation), 복구 검증(Recovery Validation), 확장 가능한 보고(Scalable Reporting), 독립적인 안전 권한을 통해 모든 내부 구성요소를 중앙에서 직접 제어하지 않고도 여러 로봇이 예측 가능하고 협조적으로 동작할 수 있다.

## 07.09 Lifecycle-Based Safe State Entry Design [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

수명주기 기반 안전 상태 진입 설계(Lifecycle-based Safe-state Entry Design)는 로봇이 정상 운용 상태에서 벗어나 제어된 비운용 상태(Controlled Non-operational Condition)로 진입해야 할 때 ROS 2 관리형 노드(Managed Node)의 전이를 이용하여 소프트웨어 동작을 조정한다. 안전 상태(Safe State)는 장애, 통신 손실, 기능 저하, 운영자 명령, 유지보수 또는 종료로 인해 요청될 수 있다. 수명주기 관리(Lifecycle Management)는 서로 종속된 로봇 구성요소 전반에서 이러한 전이를 조정하기 위한 명시적인 소프트웨어 상태 경계(Software-state Boundary)를 제공한다.

안전 상태(Safe State)는 단순히 ROS 2 비활성(Inactive) 상태와 동일한 것으로 간주하는 것이 아니라 공학적으로 설계된 시스템 상태(Engineered System Condition)로 정의해야 한다. 로봇에 따라 정지된 움직임(Stopped Motion), 억제된 액추에이터(Inhibited Actuator), 중단된 임무 실행(Suspended Mission Execution), 유지되는 위치추정(Localization), 지속적인 진단(Diagnostics), 또는 선택된 센서의 지속적인 운용이 필요할 수 있다. 따라서 개별 노드에 수명주기 전이를 할당하기 전에 시스템 수준에서 요구되는 조건을 정의해야 한다.

수명주기 상태(Lifecycle State)는 소프트웨어 가용성(Software Availability)을 나타내는 반면 안전 상태(Safety State)는 허용 가능한 물리적 동작과 위험 제어(Risk Control)를 나타낸다. 이 두 개념은 서로 관련되어 있지만 동일하지 않다. 노드가 비활성(Inactive) 상태로 진입하면 데이터 발행이나 처리를 중지할 수 있지만 이것만으로 모터가 토크 생성을 중지했다는 것을 보장하지 않는다. 비상 정지(Emergency Stop), 보호 정지(Protective Stop), 안전 토크 차단(Safe Torque Off), 안전 PLC(Safety PLC) 로직, 하드웨어 인터록(Hardware Interlock)과 같은 전용 안전 기능은 ROS 2 수명주기 실행과 독립적으로 유지되어야 한다.

안전 상태 진입(Safe-state Entry)은 일반적으로 감독(Supervisory), 진단(Diagnostic), 운용(Operational), 안전 관련 메커니즘에서 탐지된 트리거(Trigger)로 시작된다. 위치추정 손실, 반복적인 제어기 장애, 유효하지 않은 센서 데이터, 통신 타임아웃(Communication Timeout), 과도한 자원 사용량, 임무 취소, 외부 안전 이벤트 등이 이에 해당한다. 시스템이 제어된 성능 저하(Controlled Degradation), 순차적 비활성화(Orderly Deactivation), 즉각적인 움직임 억제(Immediate Motion Inhibition), 독립적인 안전 동작 중 무엇이 필요한지를 결정할 수 있도록 트리거를 분류해야 한다.

안전 상태 관리자(Safe-state Manager)는 분류된 트리거를 조정된 수명주기 동작(Coordinated Lifecycle Action)으로 변환할 수 있다. 모든 노드에 독립적으로 명령하는 대신 종속성 정보(Dependency Information)를 이용하여 어떤 기능을 먼저 중지해야 하고 어떤 서비스를 계속 사용할 수 있어야 하는지 결정할 수 있다. 일반적으로 임무 실행(Mission Execution)과 자율 명령 생성(Autonomous Command Generation)은 초기 단계에서 중단하며, 감독과 복구를 지원하기 위해 모니터링, 진단, 통신, 위치추정 또는 선택된 인지(Perception) 기능은 활성 상태로 유지할 수 있다.

구성요소를 잘못된 순서로 비활성화하면 로봇을 올바르게 정지시키는 데 필요한 정보가 제거될 수 있으므로 전이 순서(Transition Ordering)가 중요하다. 상위 수준 계획기(Planner)와 임무 제어기(Mission Controller)는 먼저 새로운 명령 생성을 중단하고, 그 다음 모션 제어기(Motion Controller)가 정의된 정지 상태에 진입하도록 할 수 있다. 제어된 움직임이 중단된 이후에만 선택된 안전 상태 정책에서 제거가 요구되는 인지, 위치추정 또는 하드웨어 인터페이스를 비활성화해야 한다.

액추에이터 처리(Actuator Handling)에서는 소프트웨어 조정(Software Coordination)과 물리적 안전 집행(Physical Safety Enforcement)을 명확하게 분리해야 한다. 수명주기 전이는 명령 생성을 중단하고, 제어기를 비활성화하고, 소프트웨어 인터페이스를 종료할 수 있지만 위험한 움직임(Hazardous Motion)의 차단이 성공적인 콜백 실행에만 의존해서는 안 된다. ROS 2 노드가 차단되거나, 충돌하거나, 과부하되거나, 요청된 전이를 완료할 수 없는 경우에도 필요하면 안전 등급 메커니즘(Safety-rated Mechanism)이 독립적으로 위험 에너지를 억제하거나 제거해야 한다.

모든 장애가 동일한 안전 상태를 요구하는 것은 아니다. 내비게이션 센서 장애는 통신과 진단 기능을 활성 상태로 유지하면서 로봇을 정지시키는 것을 허용할 수 있다. 선택적 검사 카메라(Optional Inspection Camera)의 장애는 성능 저하 모드(Degraded Mode)에서 계속 이동하도록 허용할 수 있다. 중요한 액추에이터 또는 안전 제어기 상태는 즉각적인 움직임 억제를 요구할 수 있다. 따라서 안전 상태 정책(Safe-state Policy)은 하나의 보편적인 종료 대응을 적용하는 대신 장애 등급(Fault Class)과 운용 상황(Operational Context)을 서로 다른 목표 상태(Target State)에 매핑해야 한다.

수명주기 전이 요청(Lifecycle Transition Request)은 완료(Completion), 타임아웃(Timeout), 거부(Rejection), 실패(Failure)를 모니터링해야 한다. 노드가 허용된 시간 내에 비활성화되지 않는 경우 안전 상태 관리자는 무기한 기다려서는 안 된다. 프로세스 종료(Process Termination), 서브시스템 격리(Subsystem Isolation), 하드웨어 억제(Hardware Inhibition), 또는 사전에 정의된 다른 폴백 메커니즘(Fallback Mechanism)으로 단계적으로 대응할 수 있다. 이러한 에스컬레이션(Escalation)은 안전 상태로 진입하기 위한 소프트웨어 메커니즘 자체가 실패하더라도 로봇이 제어되지 않는 중간 상태에 남지 않도록 한다.

종속성 인식 안전 상태 진입(Dependency-aware Safe-state Entry)은 불필요한 연쇄 종료(Cascading Shutdown)도 방지한다. 선택적 인지 구성요소에 장애가 발생하면 해당 구성요소에 종속된 기능만 사용할 수 없게 만들면 될 수 있다. 시스템 정책이 허용하는 경우 다른 독립적인 기능은 계속 운용할 수 있다. 반대로 위치추정과 같은 핵심 기능(Fundamental Capability)에 장애가 발생하면 내비게이션, 자율 임무 실행, 관련 제어기가 함께 비운용 상태로 전이되어야 할 수 있다.

시스템은 안전 상태 진입(Safe-state Entry)과 비상 대응(Emergency Response)을 구분해야 한다. 제어된 안전 상태 진입은 구성요소가 정지하고, 데이터를 기록하고, 자원을 해제할 시간을 제공하기 때문에 수백 밀리초에서 수 초가 걸리는 순차적 절차를 포함할 수 있다. 반면 비상 안전 기능(Emergency Safety Function)은 독립적으로 훨씬 빠르게 반응해야 할 수 있다. 이후 ROS 2 수명주기 오케스트레이션(Lifecycle Orchestration)은 안전 시스템이 확립한 물리적 상태와 소프트웨어 상태를 다시 정합시킬 수 있다.

목표 안전 상태(Target Safe State)에 진입한 이후에도 가능하다면 로봇은 해당 전이를 발생시킨 조건을 계속 모니터링해야 한다. 진단 노드(Diagnostic Node)는 활성(Active) 상태를 유지할 수 있고, 운영자 또는 플릿 관리 시스템(Fleet Management)과의 통신을 계속할 수 있으며, 선택된 센서는 환경 또는 복구 정보를 제공할 수 있다. 관측 가능성(Observability)을 유지하면 장애가 해소되었는지, 수동 개입(Manual Intervention)이 필요한지, 자동 복구(Automatic Recovery)를 안전하게 시도할 수 있는지를 판단하는 데 도움이 된다.

안전 상태는 시스템 수준에서 명시적으로 표현되어야 한다. 운용 중(Operational), 성능 저하(Degraded), 안전 정지(Safe Stop), 복구 중(Recovering), 유지보수(Maintenance), 사용 불가(Unavailable)와 같은 상태를 이용하여 여러 수명주기 노드와 물리적 서브시스템의 결합 상태를 요약할 수 있다. 이를 통해 외부 감독기(External Supervisor)가 개별 노드 상태로부터 로봇의 안전 상태나 준비 상태를 추론하지 않아도 되며, 임무 관리자, 운영자, 플릿 수준 조정(Fleet-level Coordination)을 위한 더욱 명확한 인터페이스를 제공할 수 있다.

안전 상태에서의 복구(Recovery)는 단순히 비활성화 명령을 역순으로 실행하는 것이 아니라 새로운 준비 상태 확인 과정(New Readiness Process)으로 취급해야 한다. 먼저 최초 장애가 제거되거나 승인된 성능 저하 정책(Approved Degraded Policy)에 따라 수용되어야 한다. 이후 하드웨어 인터페이스, 센서, 위치추정, 제어기, 상위 수준 애플리케이션을 종속성 순서(Dependency Order)에 따라 복원하고, 각 기능이 정상 운용에 다시 참여하도록 허용하기 전에 준비 상태 검사(Readiness Check)를 수행해야 한다.

자동 복구(Automatic Recovery)는 재시도 제한(Retry Limit)과 에스컬레이션 정책(Escalation Policy)을 사용해야 한다. 장애가 있는 구성요소를 비활성(Inactive)과 활성(Active) 사이에서 반복적으로 전환하면 불안정한 동작을 발생시키고 지속적인 장애를 감출 수 있다. 복구 관리자(Recovery Manager)는 장애 재발(Fault Recurrence), 전이 시도 횟수, 경과된 복구 시간(Elapsed Recovery Time), 이전 복구 결과를 추적해야 한다. 정의된 한계를 초과하면 무한한 재시도를 계속하는 대신 시스템을 안전 상태로 유지하고 운영자 또는 유지보수 개입을 요청할 수 있다.

통신 손실(Communication Loss)은 안전 상태 진입 중 중앙집중식 명령(Centralized Command)을 더 이상 사용할 수 없을 수 있기 때문에 특별한 처리가 필요하다. 각 로봇은 플릿 관리자, 원격 운영자(Remote Operator), 외부 서비스와의 통신이 끊어졌을 때 수행할 동작을 정의하는 로컬 정책(Local Policy)을 가져야 한다. 애플리케이션에 따라 로봇은 정지하거나, 제한된 로컬 작업(Bounded Local Action)을 완료하거나, 지정된 위치로 복귀하거나, 통신이 복원될 때까지 정지된 모니터링 상태(Stationary Monitored Condition)를 유지할 수 있다.

다중 로봇 운용(Multi-robot Operation)에서는 하나의 로봇이 안전 상태에 진입했다고 해서 임무 종속성이 협조 동작을 요구하지 않는 한 모든 로봇을 자동으로 정지시켜서는 안 된다. 느슨하게 결합된 플릿(Loosely Coupled Fleet)은 영향을 받은 로봇을 격리하고 해당 작업을 재할당할 수 있다. 협동 운송(Cooperative Transport)이나 대형 운용(Formation Operation)에서는 참여 그룹 전체가 함께 정지해야 할 수 있다. 따라서 플릿 수준 정책은 로봇의 안전 상태 보고와 임무 결합도(Mission Coupling), 기능 종속성(Capability Dependency)을 함께 고려해야 한다.

관측 가능성(Observability)은 안전 상태 동작을 검증하는 데 필수적이다. 로그는 최초 트리거(Initiating Trigger), 진단 심각도(Diagnostic Severity), 수명주기 상태, 전이 요청, 콜백 결과(Callback Result), 액추에이터 상태, 안전 시스템 상태, 타임스탬프, 에스컬레이션 동작, 최종 시스템 상태를 서로 연계해야 한다. 완전한 이벤트 타임라인(Event Timeline)을 통해 엔지니어는 로봇이 올바른 이유로 정지했는지, 그리고 소프트웨어와 물리적 메커니즘이 의도된 상태에 도달했는지를 검증할 수 있다.

안전 상태 설계(Safe-state Design)는 정상 운용만으로 검증하는 것이 아니라 장애 주입(Fault Injection)과 통합 시험(Integration Testing)을 통해 시험해야 한다. 시험에서는 센서 손실, 통신 중단, 수명주기 콜백 실패, 제어기 타임아웃, 프로세스 충돌(Process Crash), 지연된 전이(Delayed Transition), 복구 실패 등을 발생시킬 수 있다. 예상되는 수명주기 시퀀스, 물리적 응답(Physical Response), 진단 경보, 타임아웃 동작, 최종 시스템 상태가 사전에 정의된 인수 기준(Acceptance Criteria)을 충족하는지 확인해야 한다.

결국 수명주기 기반 안전 상태 진입(Lifecycle-based Safe-state Entry)은 보다 광범위한 로봇 안전 아키텍처(Robot Safety Architecture)를 중심으로 구조화된 소프트웨어 조정 메커니즘을 제공한다. 명시적인 안전 상태 정의, 장애 분류(Fault Classification), 종속성 인식 전이(Dependency-aware Transition), 타임아웃 에스컬레이션(Timeout Escalation), 독립적인 안전 권한(Independent Safety Authority), 지속적인 진단, 제어된 복구(Controlled Recovery), 시스템 수준 보고(System-level Reporting)를 통해 ROS 2 구성요소는 수명주기 관리 자체를 안전 기능으로 간주하지 않으면서도 비정상 조건에 예측 가능하게 대응할 수 있다.

## 07.10 Lifecycle Management System Integration Test

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

수명주기 관리 시스템 통합 시험(Lifecycle Management System Integration Testing)은 ROS 2 관리형 노드(Managed Node), 서브시스템 관리자(Subsystem Manager), 하드웨어 인터페이스(Hardware Interface), 진단(Diagnostics), 복구 메커니즘(Recovery Mechanism), 시스템 수준 오케스트레이션(System-level Orchestration)이 하나의 협조된 로봇 시스템으로 올바르게 동작하는지를 검증한다. 단위 시험(Unit Test)은 개별 콜백을 확인할 수 있지만, 통합 시험은 여러 수명주기 구성요소가 종속성, 비동기 이벤트(Asynchronous Event), 분산 통신(Distributed Communication), 물리 또는 시뮬레이션 장치를 통해 상호작용할 때 나타나는 시스템 동작을 검증해야 한다.

시험 환경(Test Environment)은 현실적인 상호작용을 확인할 수 있을 정도로 실제 운용 아키텍처(Operational Architecture)를 충실하게 표현해야 한다. 실제 로봇 하드웨어, 하드웨어 인더루프 시스템(Hardware-in-the-loop System), 시뮬레이션 센서와 액추에이터, ROS 2 미들웨어(Middleware), 수명주기 관리자(Lifecycle Manager), 진단 서비스, 안전 인터페이스(Safety Interface), 임무 소프트웨어 등이 포함될 수 있다. 소프트웨어 결함을 장치, 네트워크, 타이밍 영향과 구분할 수 있도록 시험은 결정론적 시뮬레이션(Deterministic Simulation)에서 대표적인 실제 하드웨어 구성으로 단계적으로 확장되어야 한다.

기본 통합 시험(Baseline Integration Test)은 전체 정상 수명주기 시퀀스(Nominal Lifecycle Sequence)를 검증해야 한다. 노드는 예상된 초기 상태에서 시작하여 구성 요청(Configuration Request)을 받고, 필요한 자원을 할당하고, 비활성(Inactive) 상태로 전이한 다음 종속성 순서(Dependency Order)에 따라 활성화된다. 시험에서는 발행자(Publisher), 구독자(Subscriber), 서비스(Service), 하드웨어 인터페이스, 제어기가 의도된 단계에서만 운용 상태가 되는지 확인하고 전체 로봇이 정의된 준비 완료(Ready) 또는 운용 중(Operational) 상태에 도달하는지 검증해야 한다.

기동 시험(Startup Testing)은 단순히 상태 전이가 성공했는지만 확인해서는 안 된다. 각 구성요소는 종속 노드가 활성화되기 전에 준비 상태 기준(Readiness Criteria)을 충족해야 한다. 센서 데이터는 유효해야 하고, 좌표 변환(Transform)을 사용할 수 있어야 하며, 필요한 경우 위치추정(Localization)이 수렴해야 하고, 하드웨어 통신이 확립되어야 하며, 제어기가 예상 상태를 보고해야 한다. 따라서 노드가 활성(Active) 상태에 도달했더라도 필요한 기능을 제공하지 못한다면 성공적인 기동이 아니라 통합 실패(Integration Failure)로 판단해야 한다.

종료 시험(Shutdown Testing)은 역방향 종속성 시퀀스(Reverse Dependency Sequence)를 검증하고 자원이 안전하게 해제되는지를 확인해야 한다. 임무 실행(Mission Execution)과 자율 명령 생성(Autonomous Command Generation)은 모션 관련 구성요소가 비활성화되기 전에 중지되어야 한다. 센서 소비자(Sensor Consumer)는 관련 하드웨어 인터페이스가 제거되기 전에 중지되어야 하고, 영구 데이터(Persistent Data)는 기록을 완료해야 하며, 필요한 정리가 완료될 때까지 컴퓨팅 자원을 사용할 수 있어야 한다. 최종 상태와 종료 시간(Shutdown Timing)은 검증을 위해 기록해야 한다.

전이 콜백 동작(Transition Callback Behavior)은 정상 조건과 비정상 조건 모두에서 시험해야 한다. \`on_configure()\`, \`on_activate()\`, \`on_deactivate()\`, \`on_cleanup()\`, \`on_shutdown()\` 및 오류 처리 경로(Error-processing Path)는 정상적인 자원뿐만 아니라 사용할 수 없는 장치, 잘못된 파라미터(Invalid Parameter), 지연된 응답(Delayed Response), 내부 장애 조건에서도 실행되어야 한다. 통합 시험에서는 반환된 전이 결과(Transition Result)를 검증하고 주변 구성요소가 성공, 거부(Rejection), 실패에 올바르게 대응하는지를 확인해야 한다.

종속성 시험(Dependency Testing)은 수명주기 오케스트레이션(Lifecycle Orchestration)이 노드 사이의 기능적 관계(Functional Relationship)를 준수하는지를 검증한다. 위치추정이 센서와 좌표 변환에 의존한다면 이러한 선행 조건을 사용할 수 없을 때 활성화가 진행되어서는 안 된다. 내비게이션(Navigation)이 위치추정에 의존한다면 해당 전이는 정책에 따라 차단되거나 실패해야 한다. 또한 독립적인 분기(Independent Branch)는 관련 없는 구성요소로 인해 불필요하게 지연되지 않고 시작하거나 복구할 수 있는지도 확인해야 한다.

장애 주입(Fault Injection)은 예상된 가정이 깨졌을 때만 나타나는 수명주기 문제가 많기 때문에 필수적이다. 시험에서는 센서 연결을 해제하고, 프로세스를 중지하고, 메시지를 지연시키고, 준비 상태 조건을 손상시키고, 네트워크 연결을 제거하고, 특정 자원을 고갈시키거나, 수명주기 콜백이 강제로 실패하도록 만들 수 있다. 목적은 시스템이 조용히 불일치 상태(Inconsistent State)에 남도록 허용하는 것이 아니라 정의된 정책에 따라 장애가 탐지되고, 전파되고, 격리되고, 처리되는지를 검증하는 것이다.

타임아웃 동작(Timeout Behavior)은 명시적으로 시험해야 한다. 수명주기 콜백, 하드웨어 초기화, 준비 상태 검사, 서비스 요청, 확인 응답(Acknowledgement)은 프로세스가 계속 실행 중이더라도 완료되지 않을 수 있다. 시험 프레임워크(Test Framework)는 설정된 한계를 초과하는 지연을 의도적으로 발생시키고 오케스트레이터가 타임아웃을 탐지하고, 적절한 진단을 생성하고, 종속 구성요소의 활성화를 방지하며, 정의된 복구 또는 에스컬레이션 동작(Escalation Action)을 수행하는지를 검증해야 한다.

복구 시험(Recovery Testing)은 장애가 발생한 프로세스가 재시작되는지만 확인하는 것이 아니라 전체 복구 시퀀스(Recovery Sequence)를 검증해야 한다. 장애 이후 영향을 받은 노드는 비활성화, 정리(Cleanup), 재시작(Restart), 재구성(Reconfigure)을 수행하고 활성(Active) 상태로 복귀하기 전에 준비 상태 검사를 통과해야 할 수 있다. 복구된 기능을 신뢰할 수 있을 때까지 종속 구성요소는 중단 상태를 유지해야 한다. 재시도 횟수(Retry Counter), 에스컬레이션 임계값(Escalation Threshold), 성능 저하 모드 결정(Degraded-mode Decision), 지속적인 장애 동작(Persistent-failure Behavior)도 함께 검증해야 한다.

진단 통합 시험(Diagnostic Integration Testing)은 수명주기 이벤트와 런타임 건전성 정보(Runtime Health Information)가 의미 있는 경보를 생성하는지를 확인한다. 시험에서는 심각도 분류(Severity Classification), 노드 및 서브시스템 식별, 타임스탬프, 전이 상황(Transition Context), 종속성 상관관계(Dependency Correlation), 근본 장애 보고(Root-fault Reporting)를 검증해야 한다. 하나의 상위 장애가 제어되지 않는 경보 폭주(Alert Storm)를 발생시켜서는 안 되며, 의도적으로 비활성(Inactive) 상태인 노드는 현재 수명주기 상태에 부합하는 동작을 하고 있다면 잘못된 데이터 누락 경보를 발생시키지 않아야 한다.

안전 상태 시험(Safe-state Test)은 비정상 조건이 의도된 시스템 수준 대응으로 이어지는지를 검증해야 한다. 로봇이 운용되는 동안 위치추정 손실, 제어기 장애, 통신 중단, 유효하지 않은 중요 센서 데이터 등의 장애를 주입할 수 있다. 시험에서는 명령 억제(Command Suppression), 순차적인 수명주기 전이, 액추에이터 상태 처리(Actuator-state Handling), 지속적인 진단, 최종 시스템 상태를 확인하고 요청된 전이가 허용된 시간 내에 완료되지 않는 경우 에스컬레이션이 수행되는지를 검증해야 한다.

안전 메커니즘(Safety Mechanism)은 수명주기 조정과 별도로 시험하는 동시에 두 시스템 사이의 인터페이스도 함께 검증해야 한다. 비상 정지(Emergency Stop), 보호 정지(Protective Stop), 안전 토크 차단(Safe Torque Off), 안전 PLC(Safety PLC) 로직, 하드웨어 인터록(Hardware Interlock)은 ROS 2 노드가 차단되거나 사용할 수 없는 상황에서도 계속 유효해야 한다. 통합 시험에서는 수명주기 소프트웨어가 안전 상태를 올바르게 관찰하고 적절한 소프트웨어 상태로 전이하면서도 안전 동작 자체의 선행 조건이 되지 않는지를 확인해야 한다.

통신 및 분산 시스템 시험(Communication and Distributed-system Test)에서는 지연 시간(Latency), 패킷 손실(Packet Loss), 일시적인 연결 해제, 중복 요청(Duplicated Request), 지연된 상태 보고를 발생시켜야 한다. 메시지가 재시도되거나 순서가 변경되더라도 수명주기 명령이 일관되지 않은 상태를 만들어서는 안 된다. 하트비트(Heartbeat), 리스(Lease), 시퀀스 번호(Sequence Number), 확인 응답, 타임아웃 정책을 사용하는 경우 이를 검증하여 오래된 정보(Stale Information)가 로봇의 준비 상태나 운용 가용성을 잘못 나타내지 않도록 해야 한다.

반복 주기 시험(Repeated-cycle Testing)은 한 번의 기동 과정에서는 발견하기 어려운 자원 누수(Resource Leak)와 상태 손상(State Corruption)을 탐지할 수 있다. 구성(Configure), 활성화(Activate), 비활성화(Deactivate), 정리(Cleanup), 복구 시퀀스를 여러 번 반복하면서 메모리, 파일 디스크립터(File Descriptor), 스레드(Thread), 미들웨어 엔티티(Middleware Entity), GPU 또는 가속기 자원(Accelerator Resource), 하드웨어 핸들(Hardware Handle), 타이밍을 모니터링해야 한다. 자원 사용량은 제한된 범위 내에서 유지되어야 하며 반복 실행에서도 수명주기 전이는 일관된 결과를 제공해야 한다.

성능 측정(Performance Measurement)은 기능 검증(Functional Verification)과 함께 수행되어야 한다. 중요한 지표에는 기동 시간(Startup Time), 전이 지연 시간(Transition Latency), 준비 상태 탐지 시간(Readiness Detection Time), 복구 시간(Recovery Duration), 종료 시간, 진단 경보 지연 시간(Diagnostic Alert Latency), 반복 실행 간 변동성이 포함된다. 분산 시스템에서는 타임스탬프 정확도(Timestamp Accuracy)와 통신 지연 측정도 필요할 수 있다. 성능 인수 한계(Performance Acceptance Limit)는 단순히 전이가 최종적으로 성공했는지가 아니라 실제 운용 요구사항을 반영해야 한다.

다중 로봇 통합 시험(Multi-robot Integration Testing)은 로봇 및 플릿 수준에서 동기화 정책(Synchronization Policy)을 검증해야 한다. 시험에서는 여러 로봇에 준비 완료(Ready), 성능 저하(Degraded), 복구 중(Recovering), 사용 불가(Unavailable), 통신 손실 조건을 조합할 수 있다. 배리어 동기화(Barrier Synchronization), 부분 동기화(Partial Synchronization), 임무 할당(Mission Allocation), 작업 재할당(Task Reassignment), 플릿 기동(Fleet Startup), 협조 종료(Coordinated Shutdown), 복구 이후 로봇 재진입(Robot Re-entry)은 모든 내부 노드 전이를 플릿 제어에 노출하지 않으면서 임무 종속성에 따라 동작해야 한다.

시험 실행 이후 장애를 재구성할 수 있도록 관측 가능성(Observability)을 시험 프레임워크에 포함해야 한다. 수명주기 이벤트, 전이 요청과 결과, 진단 메시지, 하드웨어 상태, 안전 상태, 네트워크 조건, 복구 동작, 시스템 수준 상태는 일관된 타임스탬프와 식별자(Identifier)를 공유해야 한다. 이후 자동화된 시험 보고서(Automated Test Report)는 실제로 관찰된 이벤트 시퀀스와 예상 시퀀스를 비교하여 동작이 달라지기 시작한 정확한 단계를 식별할 수 있다.

통합 시험은 로그를 육안으로 확인하는 방식에 의존하지 않고 명시적인 통과 및 실패 기준(Pass and Fail Criteria)을 정의해야 한다. 예상 초기 상태, 허용된 전이(Permitted Transition), 최대 전이 시간(Maximum Transition Time), 필요한 준비 상태 조건, 허용 가능한 성능 저하 모드, 복구 제한, 최종 상태, 금지된 동작(Prohibited Behavior)을 시험 실행 전에 정의해야 한다. 이를 통해 지속적 통합(Continuous Integration), 시뮬레이션 연구실, 하드웨어 시험 벤치(Hardware Test Bench), 배포 전 검증(Pre-deployment Validation) 환경에서 시험을 자동으로 수행할 수 있다.

결국 수명주기 관리 시스템 통합 시험(Lifecycle Management System Integration Testing)은 로봇을 독립적으로 동작하는 ROS 2 노드의 집합이 아니라 협조된 상태 관리 시스템(Coordinated State-managed System)으로 검증한다. 정상 시퀀싱(Nominal Sequencing), 종속성 강제(Dependency Enforcement), 장애 주입, 타임아웃 처리, 진단, 복구, 안전 상태 동작, 분산 통신, 반복 주기 시험, 성능 측정, 다중 로봇 조정을 종합적으로 검증함으로써 정상 및 비정상 운용 조건 모두에서 수명주기 오케스트레이션이 예측 가능한 방식으로 유지된다는 근거를 제공한다.
