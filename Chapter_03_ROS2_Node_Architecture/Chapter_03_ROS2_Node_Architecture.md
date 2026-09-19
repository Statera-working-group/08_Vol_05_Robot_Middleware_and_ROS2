**Volume 05 Robot Middleware and ROS2**


# 03. ROS2 Node Architecture

##  

## 03.01 ROS2 Lifecycle Node State Machine [w/Code]

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

A ROS 2 Lifecycle Node is a managed node whose operational behavior is controlled through a standardized state machine rather than beginning full execution immediately after process creation. This architecture separates node construction, resource configuration, activation, deactivation, cleanup, and shutdown into explicit stages. It enables a robotic system to determine not only whether a process exists, but whether the node is actually prepared and authorized to perform its intended function.

The lifecycle mechanism is especially valuable in complex robotic systems containing perception, localization, planning, control, diagnostics, and hardware-interface nodes. Ordinary ROS 2 nodes usually initialize their resources and begin operation according to application-specific logic. Lifecycle nodes instead expose predictable management interfaces, allowing an external supervisor or orchestration layer to coordinate startup and shutdown sequences across many interdependent components.

The state machine distinguishes primary states from transitions between those states. The principal stable states are Unconfigured, Inactive, Active, and Finalized. When a lifecycle node is first instantiated, it normally enters the Unconfigured state. At this stage, the ROS 2 node object exists, but application-specific resources required for normal operation have not yet been fully initialized, making the node available for controlled configuration without allowing mission execution.

A Configure transition moves the node from Unconfigured toward Inactive and invokes the configuration callback. During this operation, the node can load parameters, allocate memory, initialize algorithms, establish device interfaces, prepare publishers and subscribers, or create other required resources. Successful completion places the node in Inactive. If configuration fails, the lifecycle framework can redirect execution through defined failure handling rather than silently leaving the component in an uncertain operational condition.

The Inactive state represents a configured but non-operational condition. Resources may already exist, communication interfaces may be prepared, and internal data structures may be initialized, yet the node should not perform its primary mission behavior. This distinction is important for coordinated systems because multiple nodes can be prepared before any of them begin producing operational outputs. A supervisor can therefore establish a deterministic readiness boundary before activating the complete processing chain.

Activation moves a lifecycle node from Inactive to Active through the activation callback. The node is now permitted to perform its normal operational role, such as publishing sensor data, generating localization estimates, producing trajectories, or participating in autonomous control. Lifecycle-aware publishers can be activated specifically during this transition, preventing meaningful output from being transmitted before the component has reached its intended operational state.

Deactivation provides the reverse controlled transition from Active to Inactive. Instead of destroying the node, the system temporarily suspends its operational behavior while preserving configured resources that may be needed again. This capability is useful during mission pauses, mode changes, sensor reconfiguration, subsystem isolation, or controlled recovery procedures. A node can later be reactivated without repeating every initialization operation, reducing restart latency and unnecessary resource reconstruction.

The Cleanup transition normally moves a node from Inactive back to Unconfigured. During cleanup, resources acquired during configuration can be released, device sessions can be closed, dynamically allocated structures can be discarded, and configuration-dependent state can be reset. The node process can remain alive while returning to a condition suitable for a new configuration cycle, which supports controlled reinitialization without requiring complete process termination and recreation.

Shutdown transitions move the lifecycle toward the Finalized state. Shutdown can be requested from several stable states because a system may need to terminate a component regardless of whether it is currently unconfigured, inactive, or active. Finalized represents the end of managed operation. This explicit terminal state allows supervisors, diagnostics systems, and launch infrastructure to distinguish intentional lifecycle termination from uncontrolled disappearance caused by process crashes or communication failures.

Lifecycle transitions are implemented through callbacks such as on_configure(), on_activate(), on_deactivate(), on_cleanup(), on_shutdown(), and on_error(). Each callback should perform only the work appropriate to its transition and return a result describing success, failure, or error. Keeping responsibilities aligned with lifecycle boundaries improves architectural clarity because resource ownership, operational permission, recovery behavior, and termination logic become visible parts of the node design rather than hidden application conventions.

Error handling is an essential part of the lifecycle state machine. A failure during configuration, activation, deactivation, cleanup, or runtime operation should not automatically be treated as equivalent to process termination. The lifecycle model provides structured paths through which an error callback can attempt recovery, release resources, report diagnostic information, or move the node toward a safe state. The exact recovery policy remains application dependent and must be designed according to subsystem criticality.

A lifecycle manager can control multiple managed nodes through ROS 2 lifecycle services and coordinate their transitions according to dependency relationships. For example, a hardware driver may need to reach Active before a perception node begins processing its data, while localization may need valid sensor streams before navigation is activated. Coordinated transitions therefore transform node startup from an arbitrary timing problem into an explicit system-level sequencing problem with observable state and controllable progression.

This approach becomes particularly important in autonomous mobile robots, manipulators, UAVs, and distributed Physical AI systems. Consider an AMR in which motor interfaces, LiDAR drivers, localization, obstacle perception, path planning, and motion control form a dependency chain. Activating motion control before localization and safety perception are ready could create unsafe behavior. Lifecycle management allows the architecture to configure components first, verify readiness, and activate them according to defined dependencies.

Lifecycle nodes also improve maintenance and fault recovery because individual subsystems can be restarted or reconfigured without necessarily restarting the entire ROS 2 graph. A malfunctioning sensor-processing node may be deactivated, cleaned up, configured again, and reactivated while unrelated services continue operating. This controlled restart pattern can reduce downtime and supports resilient architectures in which faults are isolated at component boundaries instead of propagating immediately into system-wide restarts.

The lifecycle state should not be confused with application-level mission state. Active indicates that a node is operationally enabled, but it does not necessarily mean that a robot is driving, manipulating an object, or executing a mission. Mission states such as Idle, Navigate, Inspect, Dock, Charge, or Emergency Stop belong to higher-level behavioral logic. Separating node lifecycle management from mission behavior prevents infrastructure state and robot task state from becoming tightly coupled.

Similarly, lifecycle management does not replace functional safety mechanisms. An Active controller may still require independent emergency-stop circuits, watchdogs, command validity checks, communication timeout handling, actuator limits, and safety-rated hardware. Lifecycle state provides software orchestration and operational readiness management, while safety functions must be implemented according to the hazards, safety requirements, and applicable standards of the physical system.

In distributed ROS 2 deployments, lifecycle information can also support observability. Supervisors can query node states, monitor transition results, correlate lifecycle changes with diagnostics, and record state histories in system logs. When a robot fails during deployment, engineers can determine whether a component failed to configure, never reached Active, unexpectedly returned to Inactive, or entered an error path. Such information substantially improves debugging compared with merely knowing whether a process was running.

A robust lifecycle architecture therefore treats transitions as contracts. Configuration should establish resources and prerequisites, activation should enable operational behavior, deactivation should suspend that behavior predictably, cleanup should release configuration-dependent resources, and shutdown should terminate managed operation safely. Transition callbacks should be bounded, observable, and designed to avoid leaving hardware or software resources in ambiguous intermediate conditions when operations fail.

At the system level, the ROS 2 Lifecycle Node State Machine provides a foundation for deterministic orchestration. It converts component readiness from an implicit assumption into an explicit state that other software can inspect and control. Combined with dependency management, diagnostics, watchdogs, fault recovery, and mission-level supervision, lifecycle nodes help transform a collection of independently executing ROS 2 processes into a coordinated robotic software architecture suitable for increasingly complex autonomous systems.

ROS 2 수명주기 노드(Lifecycle Node)는 프로세스가 생성된 직후 전체 기능을 실행하는 대신, 표준화된 상태 머신(State Machine)을 통해 동작 상태가 제어되는 관리형 노드(Managed Node)이다. 이 아키텍처는 노드 생성(Node Construction), 자원 구성(Resource Configuration), 활성화(Activation), 비활성화(Deactivation), 정리(Cleanup), 종료(Shutdown)를 명확한 단계로 분리한다. 이를 통해 로봇 시스템은 프로세스가 존재하는지만 확인하는 것이 아니라, 해당 노드가 실제 기능을 수행할 준비와 권한을 갖추었는지 판단할 수 있다.

수명주기 메커니즘(Lifecycle Mechanism)은 인지(Perception), 위치추정(Localization), 계획(Planning), 제어(Control), 진단(Diagnostics), 하드웨어 인터페이스(Hardware Interface) 노드를 포함하는 복잡한 로봇 시스템에서 특히 유용하다. 일반적인 ROS 2 노드는 응용 프로그램별 로직에 따라 자원을 초기화하고 동작을 시작한다. 반면 수명주기 노드는 예측 가능한 관리 인터페이스(Management Interface)를 제공하여 외부 감독기(Supervisor)나 오케스트레이션 계층(Orchestration Layer)이 상호 의존적인 여러 구성요소의 시작과 종료 순서를 조정할 수 있도록 한다.

상태 머신(State Machine)은 주요 상태(Primary State)와 이들 상태 사이의 전이(Transition)를 구분한다. 대표적인 안정 상태는 미구성(Unconfigured), 비활성(Inactive), 활성(Active), 최종화(Finalized)이다. 수명주기 노드가 처음 인스턴스화되면 일반적으로 미구성(Unconfigured) 상태에 진입한다. 이 단계에서는 ROS 2 노드 객체가 존재하지만 정상 동작에 필요한 응용 프로그램별 자원이 아직 완전히 초기화되지 않았으므로, 임무 실행을 허용하지 않은 상태에서 제어된 구성을 수행할 수 있다.

구성 전이(Configure Transition)는 노드를 미구성(Unconfigured) 상태에서 비활성(Inactive) 상태로 이동시키며 구성 콜백(Configuration Callback)을 호출한다. 이 과정에서 노드는 매개변수(Parameter)를 로드하고, 메모리를 할당하며, 알고리즘을 초기화하고, 장치 인터페이스를 설정하고, 발행자(Publisher)와 구독자(Subscriber)를 준비하거나 필요한 다른 자원을 생성할 수 있다. 구성이 성공적으로 완료되면 노드는 비활성(Inactive) 상태가 된다. 구성이 실패하면 수명주기 프레임워크(Lifecycle Framework)는 구성요소를 불확실한 동작 상태에 그대로 남겨두는 대신 정의된 실패 처리 경로로 실행을 전환할 수 있다.

비활성 상태(Inactive State)는 구성이 완료되었지만 실제 동작은 수행하지 않는 상태를 의미한다. 필요한 자원이 이미 존재하고 통신 인터페이스가 준비되며 내부 데이터 구조가 초기화되었더라도, 노드는 주요 임무 동작을 수행해서는 안 된다. 이러한 구분은 여러 노드가 동작 출력을 생성하기 전에 모두 준비될 수 있도록 하기 때문에 협조형 시스템(Coordinated System)에서 중요하다. 따라서 감독기(Supervisor)는 전체 처리 체인을 활성화하기 전에 결정론적인 준비 완료 경계(Deterministic Readiness Boundary)를 설정할 수 있다.

활성화(Activation)는 활성화 콜백(Activation Callback)을 통해 수명주기 노드를 비활성(Inactive) 상태에서 활성(Active) 상태로 이동시킨다. 이제 노드는 센서 데이터 발행, 위치추정 결과 생성, 궤적(Trajectory) 생성 또는 자율 제어 참여와 같은 정상적인 동작 역할을 수행할 수 있다. 수명주기를 인식하는 발행자(Lifecycle-aware Publisher)는 이 전이 과정에서 명시적으로 활성화될 수 있으므로, 구성요소가 의도한 동작 상태에 도달하기 전에 의미 있는 출력이 전송되는 것을 방지할 수 있다.

비활성화(Deactivation)는 활성(Active) 상태에서 비활성(Inactive) 상태로 되돌아가는 제어된 역전이를 제공한다. 시스템은 노드를 제거하는 대신 향후 다시 사용할 수 있는 구성 자원을 유지하면서 동작 기능만 일시적으로 중단한다. 이 기능은 임무 일시정지(Mission Pause), 모드 변경(Mode Change), 센서 재구성(Sensor Reconfiguration), 하위 시스템 격리(Subsystem Isolation), 제어된 복구(Controlled Recovery) 과정에서 유용하다. 이후 노드는 모든 초기화 작업을 반복하지 않고 다시 활성화될 수 있으므로 재시작 지연과 불필요한 자원 재구성을 줄일 수 있다.

정리 전이(Cleanup Transition)는 일반적으로 노드를 비활성(Inactive) 상태에서 다시 미구성(Unconfigured) 상태로 이동시킨다. 정리 과정에서는 구성 단계에서 획득한 자원을 해제하고, 장치 세션(Device Session)을 종료하며, 동적으로 할당된 구조를 제거하고, 구성에 종속된 상태를 초기화할 수 있다. 노드 프로세스 자체는 계속 실행되면서 새로운 구성 주기에 적합한 상태로 되돌아갈 수 있으므로, 전체 프로세스를 종료하고 다시 생성하지 않고도 제어된 재초기화(Controlled Reinitialization)를 수행할 수 있다.

종료 전이(Shutdown Transition)는 수명주기를 최종화(Finalized) 상태로 이동시킨다. 시스템은 노드가 현재 미구성(Unconfigured), 비활성(Inactive), 활성(Active) 중 어느 상태에 있더라도 구성요소를 종료해야 할 수 있기 때문에 여러 안정 상태에서 종료를 요청할 수 있다. 최종화(Finalized)는 관리형 동작의 종료를 의미한다. 이러한 명시적인 최종 상태를 통해 감독기, 진단 시스템 및 실행 인프라(Launch Infrastructure)는 의도적인 수명주기 종료와 프로세스 충돌(Process Crash) 또는 통신 장애로 발생한 비정상적인 소멸을 구분할 수 있다.

수명주기 전이(Lifecycle Transition)는 on_configure(), on_activate(), on_deactivate(), on_cleanup(), on_shutdown(), on_error()와 같은 콜백(Callback)을 통해 구현된다. 각 콜백은 해당 전이에 적합한 작업만 수행하고 성공(Success), 실패(Failure), 오류(Error)를 나타내는 결과를 반환해야 한다. 책임을 수명주기 경계와 일치시키면 자원 소유권(Resource Ownership), 동작 권한(Operational Permission), 복구 동작(Recovery Behavior), 종료 로직(Termination Logic)이 숨겨진 응용 프로그램 규칙이 아니라 노드 설계의 명확한 요소가 되므로 아키텍처의 명료성이 향상된다.

오류 처리(Error Handling)는 수명주기 상태 머신의 핵심 요소이다. 구성, 활성화, 비활성화, 정리 또는 런타임(Runtime) 동작 중 발생한 실패를 항상 프로세스 종료와 동일하게 취급해서는 안 된다. 수명주기 모델은 오류 콜백(Error Callback)이 복구를 시도하거나, 자원을 해제하거나, 진단 정보를 보고하거나, 노드를 안전 상태(Safe State)로 이동시킬 수 있는 구조화된 경로를 제공한다. 구체적인 복구 정책(Recovery Policy)은 응용 프로그램에 따라 달라지며 하위 시스템의 중요도에 맞게 설계되어야 한다.

수명주기 관리자(Lifecycle Manager)는 ROS 2 수명주기 서비스(Lifecycle Service)를 통해 여러 관리형 노드를 제어하고 의존 관계(Dependency Relationship)에 따라 상태 전이를 조정할 수 있다. 예를 들어 인지 노드가 데이터를 처리하기 전에 하드웨어 드라이버가 활성(Active) 상태에 도달해야 할 수 있으며, 내비게이션(Navigation)이 활성화되기 전에 위치추정(Localization)이 유효한 센서 스트림을 필요로 할 수 있다. 따라서 조정된 상태 전이는 노드 시작 과정을 임의적인 타이밍 문제가 아니라 관찰 가능한 상태와 제어 가능한 진행 과정을 갖춘 명시적인 시스템 수준 순서 제어(System-level Sequencing) 문제로 전환한다.

이러한 접근법은 자율이동로봇(Autonomous Mobile Robot, AMR), 매니퓰레이터(Manipulator), 무인항공기(Unmanned Aerial Vehicle, UAV), 분산형 피지컬 AI 시스템(Distributed Physical AI System)에서 특히 중요하다. 예를 들어 모터 인터페이스, LiDAR 드라이버, 위치추정, 장애물 인지, 경로 계획(Path Planning), 모션 제어(Motion Control)가 의존성 체인을 구성하는 AMR을 생각할 수 있다. 위치추정과 안전 인지(Safety Perception)가 준비되기 전에 모션 제어를 활성화하면 위험한 동작이 발생할 수 있다. 수명주기 관리를 이용하면 먼저 구성요소를 설정하고 준비 상태를 검증한 다음 정의된 의존 관계에 따라 활성화할 수 있다.

수명주기 노드는 전체 ROS 2 그래프(Graph)를 반드시 다시 시작하지 않고도 개별 하위 시스템을 재시작하거나 재구성할 수 있기 때문에 유지보수와 장애 복구(Fault Recovery)도 향상시킨다. 오작동하는 센서 처리 노드는 관련 없는 서비스가 계속 동작하는 동안 비활성화되고, 정리되고, 다시 구성된 후 재활성화될 수 있다. 이러한 제어된 재시작 패턴(Controlled Restart Pattern)은 가동 중단 시간을 줄이고 장애가 즉시 시스템 전체의 재시작으로 확산되는 대신 구성요소 경계에서 격리되는 회복탄력적 아키텍처(Resilient Architecture)를 지원한다.

수명주기 상태(Lifecycle State)를 응용 프로그램 수준의 임무 상태(Mission State)와 혼동해서는 안 된다. 활성(Active)은 노드가 동작 가능하도록 허용되었다는 의미이지만, 반드시 로봇이 주행하거나 물체를 조작하거나 임무를 수행하고 있다는 의미는 아니다. 대기(Idle), 주행(Navigate), 검사(Inspect), 도킹(Dock), 충전(Charge), 비상정지(Emergency Stop)와 같은 임무 상태는 상위 수준의 행동 로직(Behavioral Logic)에 속한다. 노드 수명주기 관리와 임무 행동을 분리하면 인프라 상태(Infrastructure State)와 로봇 작업 상태가 지나치게 강하게 결합되는 것을 방지할 수 있다.

마찬가지로 수명주기 관리(Lifecycle Management)는 기능 안전(Functional Safety) 메커니즘을 대체하지 않는다. 활성(Active) 상태의 제어기라도 독립적인 비상정지 회로(Emergency-stop Circuit), 감시기(Watchdog), 명령 유효성 검사(Command Validity Check), 통신 타임아웃 처리(Communication Timeout Handling), 액추에이터 제한(Actuator Limit), 안전 등급 하드웨어(Safety-rated Hardware)가 필요할 수 있다. 수명주기 상태는 소프트웨어 오케스트레이션(Software Orchestration)과 동작 준비 상태를 관리하며, 안전 기능은 물리 시스템의 위험요소, 안전 요구사항 및 적용 가능한 표준에 따라 별도로 구현되어야 한다.

분산형 ROS 2 배포(Distributed ROS 2 Deployment)에서 수명주기 정보는 관측 가능성(Observability)을 지원할 수도 있다. 감독기는 노드 상태를 조회하고, 상태 전이 결과를 모니터링하며, 수명주기 변화와 진단 정보를 연계하고, 상태 이력을 시스템 로그(System Log)에 기록할 수 있다. 로봇이 실제 배포 환경에서 장애를 일으켰을 때 엔지니어는 구성요소가 구성에 실패했는지, 활성 상태에 도달하지 못했는지, 예상치 못하게 비활성 상태로 돌아갔는지 또는 오류 경로(Error Path)에 진입했는지를 판단할 수 있다. 이러한 정보는 단순히 프로세스의 실행 여부만 파악하는 것보다 디버깅(Debugging) 능력을 크게 향상시킨다.

강건한 수명주기 아키텍처(Robust Lifecycle Architecture)는 따라서 상태 전이를 계약(Contract)으로 취급한다. 구성(Configuration)은 자원과 선행조건을 확립하고, 활성화(Activation)는 동작 기능을 허용하며, 비활성화(Deactivation)는 해당 동작을 예측 가능한 방식으로 중단하고, 정리(Cleanup)는 구성에 종속된 자원을 해제하며, 종료(Shutdown)는 관리형 동작을 안전하게 끝내야 한다. 상태 전이 콜백은 실행 범위가 제한되고 관찰 가능해야 하며, 작업 실패 시 하드웨어나 소프트웨어 자원을 모호한 중간 상태에 남겨두지 않도록 설계되어야 한다.

시스템 수준에서 ROS 2 수명주기 노드 상태 머신(ROS 2 Lifecycle Node State Machine)은 결정론적 오케스트레이션(Deterministic Orchestration)을 위한 기반을 제공한다. 이는 구성요소의 준비 상태를 암묵적인 가정에서 다른 소프트웨어가 검사하고 제어할 수 있는 명시적인 상태로 변환한다. 의존성 관리(Dependency Management), 진단(Diagnostics), 감시기(Watchdog), 장애 복구(Fault Recovery), 임무 수준 감독(Mission-level Supervision)과 결합하면 수명주기 노드는 독립적으로 실행되는 ROS 2 프로세스의 집합을 점점 복잡해지는 자율 시스템에 적합한 조정된 로봇 소프트웨어 아키텍처로 발전시키는 데 기여한다.

##  

## 03.02 Composition Node: Intraprocess Communication [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 composition allows multiple nodes to execute within a single process instead of launching every node as an independent operating-system process. These composable nodes are implemented as loadable components and hosted by a component container. The approach preserves the logical separation of ROS 2 nodes while reducing process-management overhead and enabling communication optimizations that are particularly valuable for high-bandwidth robotic data pipelines.

A conventional ROS 2 deployment often maps one node to one process, providing strong isolation and straightforward fault boundaries. However, data exchanged between separate processes must normally pass through middleware transport mechanisms involving serialization, memory copying, and operating-system networking facilities. For large camera images, point clouds, tensors, or other high-rate sensor messages, these operations can consume significant CPU time and memory bandwidth.

Composition changes the deployment boundary without requiring the software architecture to abandon node modularity. Perception, preprocessing, filtering, localization, or control components can remain distinct ROS 2 nodes with their own publishers, subscribers, parameters, and callbacks while sharing one executable process. The component container dynamically loads these node implementations and manages their execution through ROS 2 component infrastructure.

A composable node is commonly implemented as a class derived from rclcpp::Node and registered as a component so that it can be discovered and loaded at runtime. Instead of defining an independent main() function for every executable, the node implementation is packaged as a shared library. A container process then instantiates one or more components, allowing deployment topology to be changed without fundamentally rewriting the node algorithms.

ROS 2 component containers provide an important separation between software implementation and runtime deployment. The same functional node can potentially run in a dedicated process when isolation is important or share a process with related components when communication efficiency is the priority. This flexibility allows architects to optimize deployment according to message size, communication frequency, latency requirements, resource constraints, and failure-isolation requirements.

Intra-process communication is one of the major performance advantages associated with composition. When compatible publishers and subscribers reside in the same process and intra-process communication is enabled, ROS 2 can avoid portions of the conventional middleware communication path. Messages can be transferred through process-local mechanisms rather than being serialized, transmitted through DDS, deserialized, and copied in the same manner as inter-process communication.

The optimization is especially important for large messages. Consider a camera node publishing high-resolution image frames to preprocessing, neural-network inference, and visualization nodes. Repeated serialization and copying of every image can generate substantial memory traffic. Placing tightly coupled stages in one process and using intra-process communication can reduce unnecessary copies, decrease CPU utilization, and improve end-to-end processing latency.

ROS 2 uses ownership-aware message passing to support efficient intra-process communication. C++ publishers and subscribers can work with smart-pointer-based message ownership, including std::unique_ptr and std::shared_ptr patterns depending on the communication configuration. Where ownership conditions permit, a message can be transferred rather than repeatedly duplicated, allowing large payloads to move between callbacks with substantially lower copying overhead.

The term zero-copy should nevertheless be used carefully. Intra-process communication can eliminate important serialization and copying operations, but it does not guarantee that every message travels through the complete application with absolutely no memory copy. Message ownership, subscriber count, API usage, middleware behavior, message type, and downstream processing can introduce additional copies. Performance should therefore be verified through measurement rather than assumed from architecture alone.

A single publisher may also communicate with multiple subscribers, which complicates exclusive message ownership. A unique message cannot simply be transferred independently to several consumers without considering how each subscriber accesses the data. ROS 2 communication mechanisms must preserve valid ownership and lifetime semantics, and some topologies may consequently require shared ownership or copying. Architecture decisions should therefore consider the complete publisher-subscriber graph rather than an isolated connection.

Composition and intra-process communication are related but distinct concepts. Composition means that multiple ROS 2 nodes share a process, whereas intra-process communication describes an optimized communication path available between compatible endpoints inside that process. Loading nodes into one container creates the opportunity for process-local optimization, but the relevant communication options and publisher-subscriber design must still support the intended intra-process behavior.

The component container also determines how callbacks are scheduled. Multiple composed nodes may share executor resources, meaning that computationally expensive callbacks can influence other components if execution is not designed carefully. A long-running perception callback, for example, may delay time-sensitive callbacks when inappropriate executor or callback-group configurations are used. Communication optimization must therefore be considered together with concurrency and executor architecture.

Single-threaded containers provide relatively simple execution behavior and can be useful when deterministic callback ordering or reduced concurrency complexity is desired. Multi-threaded execution can increase parallelism for independent callbacks but introduces synchronization requirements and possible contention for shared resources. The appropriate choice depends on workload characteristics, callback duration, thread safety, latency requirements, and available CPU cores rather than composition alone.

Process sharing also changes fault-isolation characteristics. Separate ROS 2 processes provide natural operating-system boundaries, so the failure of one process does not necessarily terminate unrelated nodes. When several nodes are composed into one container, a fatal error affecting the process can potentially remove all components hosted within it. Architects must balance communication performance against fault containment when deciding which nodes should share a container.

For this reason, composition boundaries should normally follow functional coupling and data-flow characteristics. Nodes exchanging large, high-frequency messages are strong candidates for composition, particularly when they form a tightly connected processing pipeline. Components with different safety criticality, unstable third-party libraries, independent restart requirements, or limited communication volume may be better deployed in separate processes even if composition is technically possible.

A typical perception pipeline may compose camera acquisition, image rectification, preprocessing, and inference nodes because large image messages flow continuously among them. Higher-level mission planning or fleet communication nodes may remain in separate processes because their messages are smaller and their failure domains differ. Such hybrid deployment combines efficient local data movement with intentional isolation between major robotic subsystems.

Composition can also reduce deployment overhead on resource-constrained edge computers. Fewer operating-system processes can mean fewer duplicated runtime structures, reduced process startup overhead, and more efficient sharing of computational resources. These benefits are relevant to embedded robotic computers and GPU-equipped edge platforms where perception, localization, planning, and AI inference must coexist under constrained CPU, memory, power, and thermal budgets.

Performance evaluation should include more than average message latency. Engineers should examine CPU utilization, memory consumption, memory bandwidth, callback latency, message throughput, jitter, and worst-case behavior under realistic workloads. Comparisons between separate-process communication and composed intra-process communication are particularly useful for image and PointCloud2 pipelines because payload size and publication frequency can strongly influence the benefit obtained.

Observability and debugging remain important when multiple nodes occupy one process. ROS 2 graph tools still represent the logical nodes separately, but process-level monitoring may show several components under a common container. Logging should therefore preserve clear node identities, and diagnostics should distinguish component-level health from container-level health. This distinction becomes essential when identifying whether a failure originates from one algorithm or from shared execution infrastructure.

Composition also supports configurable robot software products because deployment can be adjusted without changing the logical interface architecture. A development system may launch components independently for debugging, while a production robot may compose performance-sensitive nodes into optimized containers. The topic, service, action, and parameter interfaces can remain largely consistent, allowing deployment optimization to occur independently from higher-level functional integration.

In a scalable ROS 2 architecture, composition should therefore be treated as a deployment and performance design mechanism rather than simply a method for reducing the number of processes. Intra-process communication provides its greatest value when data locality, ownership semantics, executor behavior, and fault boundaries are considered together. Properly applied, these mechanisms preserve ROS 2 modularity while enabling efficient high-bandwidth communication for demanding robotic and Physical AI workloads.

ROS 2 컴포지션(Composition)은 모든 노드를 독립적인 운영체제 프로세스(Operating-System Process)로 실행하는 대신, 여러 노드를 하나의 프로세스 안에서 실행할 수 있도록 한다. 이러한 컴포저블 노드(Composable Node)는 로드 가능한 컴포넌트(Loadable Component)로 구현되며 컴포넌트 컨테이너(Component Container)에 의해 호스팅된다. 이 방식은 ROS 2 노드의 논리적 분리를 유지하면서 프로세스 관리 오버헤드를 줄이고, 특히 고대역폭 로봇 데이터 파이프라인(High-Bandwidth Robotic Data Pipeline)에 유용한 통신 최적화를 가능하게 한다.

기존의 ROS 2 배포(Deployment)는 하나의 노드를 하나의 프로세스에 대응시키는 경우가 많으며, 이는 강력한 격리(Isolation)와 명확한 장애 경계(Fault Boundary)를 제공한다. 그러나 서로 다른 프로세스 사이에서 교환되는 데이터는 일반적으로 직렬화(Serialization), 메모리 복사(Memory Copying), 운영체제 네트워크 기능을 포함하는 미들웨어 전송 메커니즘(Middleware Transport Mechanism)을 거쳐야 한다. 대용량 카메라 이미지, 포인트 클라우드(Point Cloud), 텐서(Tensor), 기타 고속 센서 메시지에서는 이러한 작업이 상당한 CPU 시간과 메모리 대역폭을 소비할 수 있다.

컴포지션(Composition)은 소프트웨어 아키텍처에서 노드 모듈성(Node Modularity)을 포기하지 않고도 배포 경계(Deployment Boundary)를 변경한다. 인지(Perception), 전처리(Preprocessing), 필터링(Filtering), 위치추정(Localization), 제어(Control) 구성요소는 각각 자체 발행자(Publisher), 구독자(Subscriber), 매개변수(Parameter), 콜백(Callback)을 가진 독립적인 ROS 2 노드로 유지되면서 하나의 실행 프로세스를 공유할 수 있다. 컴포넌트 컨테이너(Component Container)는 이러한 노드 구현을 동적으로 로드하고 ROS 2 컴포넌트 인프라를 통해 실행을 관리한다.

컴포저블 노드(Composable Node)는 일반적으로 rclcpp::Node에서 파생된 클래스(Class)로 구현되고 컴포넌트(Component)로 등록되어 런타임(Runtime)에 검색되고 로드될 수 있도록 한다. 각각의 실행 파일마다 독립적인 main() 함수를 정의하는 대신 노드 구현을 공유 라이브러리(Shared Library) 형태로 패키징한다. 이후 컨테이너 프로세스(Container Process)가 하나 이상의 컴포넌트를 인스턴스화하므로, 노드 알고리즘을 근본적으로 다시 작성하지 않고도 배포 토폴로지(Deployment Topology)를 변경할 수 있다.

ROS 2 컴포넌트 컨테이너(Component Container)는 소프트웨어 구현(Software Implementation)과 런타임 배포(Runtime Deployment)를 분리하는 중요한 역할을 한다. 동일한 기능 노드를 격리가 중요할 때는 전용 프로세스에서 실행하고, 통신 효율이 우선될 때는 관련 구성요소와 하나의 프로세스를 공유하도록 구성할 수 있다. 이러한 유연성을 통해 아키텍트는 메시지 크기, 통신 빈도, 지연시간 요구사항, 자원 제약, 장애 격리 요구사항에 따라 배포 구조를 최적화할 수 있다.

프로세스 내부 통신(Intra-Process Communication)은 컴포지션과 관련된 주요 성능상의 장점 중 하나이다. 호환되는 발행자와 구독자가 동일한 프로세스에 존재하고 프로세스 내부 통신이 활성화되어 있으면, ROS 2는 기존 미들웨어 통신 경로의 일부를 생략할 수 있다. 메시지는 프로세스 간 통신(Inter-Process Communication)처럼 DDS를 통해 직렬화, 전송, 역직렬화(Deserialization), 복사되는 대신 프로세스 로컬 메커니즘(Process-Local Mechanism)을 통해 전달될 수 있다.

이러한 최적화는 대용량 메시지에서 특히 중요하다. 고해상도 이미지 프레임을 전처리, 신경망 추론(Neural-Network Inference), 시각화 노드로 발행하는 카메라 노드를 생각할 수 있다. 모든 이미지의 반복적인 직렬화와 복사는 상당한 메모리 트래픽(Memory Traffic)을 발생시킬 수 있다. 밀접하게 연결된 처리 단계를 하나의 프로세스에 배치하고 프로세스 내부 통신을 사용하면 불필요한 복사를 줄이고 CPU 사용률을 낮추며 종단간 처리 지연시간(End-to-End Processing Latency)을 개선할 수 있다.

ROS 2는 효율적인 프로세스 내부 통신을 지원하기 위해 소유권 인식 메시지 전달(Ownership-Aware Message Passing)을 사용한다. C++ 발행자와 구독자는 통신 구성에 따라 std::unique_ptr 및 std::shared_ptr 패턴을 포함한 스마트 포인터 기반 메시지 소유권(Smart-Pointer-Based Message Ownership)을 사용할 수 있다. 소유권 조건이 허용되는 경우 메시지를 반복적으로 복제하는 대신 소유권 자체를 전달할 수 있으므로, 대용량 페이로드(Payload)를 훨씬 적은 복사 오버헤드로 콜백 사이에서 이동시킬 수 있다.

그러나 제로 카피(Zero-Copy)라는 용어는 신중하게 사용해야 한다. 프로세스 내부 통신은 중요한 직렬화 및 복사 작업을 제거할 수 있지만, 모든 메시지가 전체 응용 프로그램을 통과하면서 메모리 복사가 전혀 발생하지 않는다는 것을 보장하지는 않는다. 메시지 소유권, 구독자 수, API 사용 방식, 미들웨어 동작, 메시지 형식, 후속 처리 과정에 따라 추가 복사가 발생할 수 있다. 따라서 성능은 아키텍처만으로 가정해서는 안 되며 실제 측정(Measurement)을 통해 검증해야 한다.

하나의 발행자(Publisher)는 여러 구독자(Subscriber)와 통신할 수도 있으며, 이 경우 독점적인 메시지 소유권(Exclusive Message Ownership)이 복잡해진다. 각각의 구독자가 데이터에 어떻게 접근하는지를 고려하지 않고 하나의 고유 메시지를 여러 소비자에게 독립적으로 전달할 수는 없다. ROS 2 통신 메커니즘은 유효한 소유권 및 수명 의미론(Ownership and Lifetime Semantics)을 보존해야 하므로, 일부 토폴로지에서는 공유 소유권(Shared Ownership) 또는 복사가 필요할 수 있다. 따라서 아키텍처 설계에서는 하나의 연결만이 아니라 전체 발행자-구독자 그래프(Publisher-Subscriber Graph)를 고려해야 한다.

컴포지션(Composition)과 프로세스 내부 통신(Intra-Process Communication)은 서로 관련되어 있지만 서로 다른 개념이다. 컴포지션은 여러 ROS 2 노드가 하나의 프로세스를 공유하는 것을 의미하며, 프로세스 내부 통신은 해당 프로세스 내부의 호환 가능한 통신 종단점(Endpoint) 사이에서 사용할 수 있는 최적화된 통신 경로를 의미한다. 노드를 하나의 컨테이너에 로드하면 프로세스 로컬 최적화의 기회가 만들어지지만, 관련 통신 옵션과 발행자-구독자 설계 역시 의도한 프로세스 내부 동작을 지원해야 한다.

컴포넌트 컨테이너(Component Container)는 콜백이 어떻게 스케줄링(Scheduling)되는지도 결정한다. 여러 개의 컴포즈된 노드(Composed Node)가 실행기 자원(Executor Resource)을 공유할 수 있으므로, 실행 구조를 적절히 설계하지 않으면 계산량이 많은 콜백이 다른 구성요소에 영향을 줄 수 있다. 예를 들어 부적절한 실행기(Executor) 또는 콜백 그룹(Callback Group) 구성을 사용하면 장시간 실행되는 인지 콜백이 시간 민감형 콜백(Time-Sensitive Callback)을 지연시킬 수 있다. 따라서 통신 최적화는 동시성(Concurrency) 및 실행기 아키텍처와 함께 고려해야 한다.

단일 스레드 컨테이너(Single-Threaded Container)는 비교적 단순한 실행 동작을 제공하며, 결정론적인 콜백 순서(Deterministic Callback Ordering) 또는 동시성 복잡성 감소가 필요한 경우 유용할 수 있다. 다중 스레드 실행(Multi-Threaded Execution)은 독립적인 콜백의 병렬성(Parallelism)을 높일 수 있지만 공유 자원에 대한 동기화(Synchronization) 요구사항과 경합(Contention)을 발생시킨다. 적절한 방식은 컴포지션 자체가 아니라 작업부하 특성, 콜백 실행시간, 스레드 안전성(Thread Safety), 지연시간 요구사항, 사용 가능한 CPU 코어 수에 따라 결정되어야 한다.

프로세스 공유(Process Sharing)는 장애 격리(Fault Isolation)의 특성도 변화시킨다. 서로 분리된 ROS 2 프로세스는 운영체제 수준의 자연스러운 경계를 제공하므로 하나의 프로세스가 실패하더라도 관련 없는 다른 노드가 반드시 종료되는 것은 아니다. 그러나 여러 노드를 하나의 컨테이너에 컴포즈하면 프로세스에 영향을 미치는 치명적인 오류(Fatal Error)가 해당 컨테이너에 호스팅된 모든 구성요소를 동시에 종료시킬 수 있다. 따라서 아키텍트는 어떤 노드가 컨테이너를 공유할 것인지 결정할 때 통신 성능과 장애 격리 사이의 균형을 고려해야 한다.

이러한 이유로 컴포지션 경계(Composition Boundary)는 일반적으로 기능적 결합도(Functional Coupling)와 데이터 흐름(Data Flow)의 특성을 따라 설정하는 것이 적절하다. 크고 빈번한 메시지를 교환하는 노드는 특히 긴밀하게 연결된 처리 파이프라인을 구성하는 경우 컴포지션의 강력한 후보가 된다. 반면 안전 중요도(Safety Criticality)가 서로 다르거나, 불안정한 서드파티 라이브러리(Third-Party Library)를 사용하거나, 독립적인 재시작이 필요하거나, 통신량이 적은 구성요소는 기술적으로 컴포지션이 가능하더라도 별도의 프로세스로 배포하는 것이 더 적합할 수 있다.

일반적인 인지 파이프라인(Perception Pipeline)에서는 대용량 이미지 메시지가 지속적으로 흐르기 때문에 카메라 획득(Camera Acquisition), 이미지 보정(Image Rectification), 전처리(Preprocessing), 추론(Inference) 노드를 하나의 프로세스에 컴포즈할 수 있다. 반면 상위 수준의 임무 계획(Mission Planning)이나 플릿 통신(Fleet Communication) 노드는 메시지 크기가 작고 장애 영역(Failure Domain)이 다르므로 별도의 프로세스에 유지할 수 있다. 이러한 하이브리드 배포(Hybrid Deployment)는 효율적인 로컬 데이터 이동과 주요 로봇 하위 시스템 사이의 의도적인 격리를 결합한다.

컴포지션은 자원이 제한된 엣지 컴퓨터(Edge Computer)의 배포 오버헤드(Deployment Overhead)를 줄이는 데에도 도움이 될 수 있다. 운영체제 프로세스 수가 감소하면 중복된 런타임 구조가 줄어들고 프로세스 시작 오버헤드가 감소하며 계산 자원을 더욱 효율적으로 공유할 수 있다. 이러한 장점은 인지, 위치추정, 계획, AI 추론(AI Inference)이 제한된 CPU, 메모리, 전력, 열 설계 예산(Thermal Budget) 내에서 공존해야 하는 임베디드 로봇 컴퓨터와 GPU 기반 엣지 플랫폼에서 중요하다.

성능 평가(Performance Evaluation)는 평균 메시지 지연시간만을 대상으로 해서는 안 된다. 엔지니어는 CPU 사용률, 메모리 소비량, 메모리 대역폭, 콜백 지연시간, 메시지 처리량(Message Throughput), 지터(Jitter), 실제 작업부하에서의 최악 조건 동작(Worst-Case Behavior)을 함께 검토해야 한다. 특히 이미지 및 PointCloud2 파이프라인에서는 페이로드 크기와 발행 주기가 얻을 수 있는 성능 이점에 큰 영향을 주기 때문에 별도 프로세스 통신과 컴포즈된 프로세스 내부 통신을 비교하는 것이 유용하다.

여러 노드가 하나의 프로세스에 존재하는 경우에도 관측 가능성(Observability)과 디버깅(Debugging)은 중요하다. ROS 2 그래프 도구(Graph Tool)는 여전히 논리적인 노드를 개별적으로 표현하지만, 프로세스 수준 모니터링에서는 여러 구성요소가 하나의 공통 컨테이너 아래에 표시될 수 있다. 따라서 로깅(Logging)은 명확한 노드 식별자를 유지해야 하며, 진단(Diagnostics)은 컴포넌트 수준 상태(Component-Level Health)와 컨테이너 수준 상태(Container-Level Health)를 구분해야 한다. 이러한 구분은 장애가 개별 알고리즘에서 발생했는지 공유 실행 인프라에서 발생했는지를 식별할 때 필수적이다.

컴포지션은 논리적인 인터페이스 아키텍처(Logical Interface Architecture)를 변경하지 않고 배포 구성을 조정할 수 있기 때문에 구성 가능한 로봇 소프트웨어 제품(Configurable Robot Software Product)도 지원한다. 개발 시스템에서는 디버깅을 위해 구성요소를 독립적으로 실행하고, 양산 로봇(Production Robot)에서는 성능에 민감한 노드를 최적화된 컨테이너에 컴포즈할 수 있다. 토픽(Topic), 서비스(Service), 액션(Action), 매개변수(Parameter) 인터페이스는 대부분 일관되게 유지할 수 있으므로 상위 수준 기능 통합과 독립적으로 배포 최적화를 수행할 수 있다.

따라서 확장 가능한 ROS 2 아키텍처(Scalable ROS 2 Architecture)에서 컴포지션(Composition)은 단순히 프로세스 수를 줄이기 위한 방법이 아니라 배포 및 성능 설계 메커니즘(Deployment and Performance Design Mechanism)으로 다루어야 한다. 프로세스 내부 통신(Intra-Process Communication)은 데이터 지역성(Data Locality), 소유권 의미론(Ownership Semantics), 실행기 동작(Executor Behavior), 장애 경계(Fault Boundary)를 함께 고려할 때 가장 큰 가치를 제공한다. 이러한 메커니즘을 적절하게 적용하면 ROS 2의 모듈성을 유지하면서도 고성능 로봇 및 피지컬 AI(Physical AI) 작업부하에 필요한 효율적인 고대역폭 통신을 구현할 수 있다.

##  

## 03.03 Node Parameter System Usage [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

The ROS 2 parameter system provides a standardized mechanism for configuring node behavior without embedding every operational value directly in source code. Parameters belong to individual nodes and represent configurable values such as sensor rates, thresholds, frame identifiers, algorithm options, controller gains, or feature switches. This node-centric model makes configuration explicit while preserving clear ownership of runtime settings.

Unlike a global configuration database, ROS 2 parameters are associated with specific node instances. Each node declares the parameters it supports and can define their names, types, default values, and optional descriptors. This architecture prevents unrelated components from implicitly sharing configuration state and allows developers to understand which node owns a particular setting, an important property in large distributed robotic software systems.

Parameter declaration establishes the configuration contract of a node. In rclcpp, a node can use declare_parameter() to introduce a parameter and provide an initial value or expected type. Declaring parameters before accessing them improves interface clarity because supported configuration options become part of the node design. Undeclared parameter behavior can be controlled through node options, but explicit declaration is generally preferable for predictable production systems.

ROS 2 parameters support fundamental scalar and array data types suitable for most runtime configuration requirements. Typical values include Boolean flags, integers, floating-point numbers, strings, byte arrays, and arrays of primitive types. Complex application structures are normally represented through logically grouped parameter names or external configuration structures rather than treating the parameter mechanism as a general-purpose database for arbitrary application data.

Hierarchical naming conventions make large parameter sets easier to organize. A perception node might use names such as camera.width, camera.height, detection.threshold, and inference.device, while a controller could expose control.frequency, velocity.max_linear, and velocity.max_angular. The dot-separated notation provides a logical namespace-like structure even though parameters ultimately remain individually named values owned by the node.

Default values are important because they define how a node behaves when no external configuration is supplied. A well-designed default should permit safe and understandable initialization whenever practical, while deployment-specific values can override it. Critical hardware identifiers, calibration values, or safety-related limits should not be assigned arbitrary defaults merely for convenience, because an apparently valid configuration can be more dangerous than an explicit configuration failure.

Parameters can be supplied from the command line when a node is launched. This is convenient for testing, debugging, and temporary experiments because individual settings can be changed without editing source files or maintaining another configuration artifact. Command-line overrides are less suitable for large production configurations, however, because long parameter lists become difficult to review, reproduce, version, and maintain consistently across multiple robots.

YAML parameter files provide a practical method for managing larger configurations. A YAML file can associate node names with ros__parameters and define many values in a structured form. Configuration files can be stored with robot software, tracked through version control, reviewed during integration, and selected for different hardware variants or deployment environments. This makes parameter management part of repeatable system configuration rather than an informal runtime procedure.

Launch files can connect parameter definitions to deployment architecture. A ROS 2 launch description may provide parameter files, direct parameter dictionaries, or combinations of default and deployment-specific settings when creating nodes. This allows one software package to support multiple robot variants while keeping algorithm implementation unchanged. Hardware model, sensor configuration, operating environment, and mission profile can therefore influence runtime behavior through deployment configuration.

Parameters can also be inspected and modified through ROS 2 command-line tools. Engineers can list parameters exposed by a node, retrieve current values, examine descriptors, and request runtime changes. These capabilities are useful during commissioning and debugging because the active configuration can be observed directly from the running system instead of being inferred only from source files or expected launch arguments.

Runtime parameter changes introduce additional design responsibilities. A node should not accept a new value merely because its type is syntactically valid. Changing a control frequency, detection threshold, coordinate frame, device identifier, or resource allocation parameter may require validation or may be unsafe while the component is operating. Parameter mutability should therefore be designed according to the operational meaning of each setting.

ROS 2 provides parameter-change callbacks that allow a node to validate requested updates before accepting them. A callback can check numerical ranges, combinations of related values, operating-state restrictions, or hardware constraints and reject invalid requests. This validation layer converts the parameter interface from simple writable storage into a controlled configuration boundary, helping prevent runtime changes from creating inconsistent internal state.

Validation should consider relationships among parameters rather than checking values independently when necessary. For example, a minimum range should remain below a maximum range, image dimensions may need to match a supported camera mode, and controller limits may depend on the physical platform. A request containing several parameter updates should be evaluated as a coherent configuration change when their meanings are interdependent.

Parameter descriptors can provide metadata describing the intended meaning and constraints of a parameter. Descriptions, read-only properties, integer ranges, and floating-point ranges can make configuration interfaces easier to understand and support tooling that presents parameter information to users. Descriptors do not eliminate the need for application-level validation, but they improve self-documentation and reduce ambiguity in reusable robotic software components.

Parameter events provide visibility into configuration changes across the ROS 2 system. When parameters are created, modified, or deleted, parameter-related events can be observed by interested components. Monitoring these events can support diagnostics, configuration auditing, user interfaces, and system supervision. In production robots, recording important parameter changes together with timestamps can be valuable when investigating behavior that changed during operation.

Parameters should be distinguished from topics, services, and actions according to semantic purpose. Parameters represent relatively persistent configuration associated with a node, whereas topics are designed for streams of data, services for request-response interactions, and actions for longer-running goal-oriented operations. Using parameters for high-frequency state exchange or sensor data would misuse the configuration mechanism and create an unsuitable communication architecture.

The parameter system also interacts with node lifecycle management. A lifecycle node may declare parameters during construction but apply resource-dependent configuration during its configuration transition. Some settings may be allowed to change only while the node is Unconfigured or Inactive, while others can safely change during Active operation. Connecting parameter mutability to lifecycle state creates clearer operational rules for reconfiguration.

In an AMR, parameters may configure LiDAR frame names, localization thresholds, planner behavior, maximum velocity, obstacle margins, or diagnostic intervals. In a manipulator, they may define controller gains, joint limits, planning options, and device interfaces. Physical AI nodes may expose inference thresholds, model paths, accelerator selection, preprocessing dimensions, or runtime feature switches, allowing common software components to adapt to different deployment targets.

Large robot fleets require disciplined parameter governance because uncontrolled per-robot modifications can gradually create configuration drift. Parameter files should be versioned, deployment variants should be identifiable, and important runtime overrides should be traceable. A robot should ideally report not only its software version but also the effective configuration used by critical nodes, enabling field behavior to be reproduced during testing and failure analysis.

Security must also be considered because runtime parameter modification can alter robot behavior. A parameter controlling speed limits, perception thresholds, communication endpoints, or safety-related operating conditions should not be treated as harmless configuration. Access to parameter services should follow the security architecture of the ROS 2 deployment, and critical changes may require additional authorization, validation, logging, or higher-level supervisory control.

A robust ROS 2 parameter architecture therefore combines explicit declaration, meaningful defaults, structured YAML configuration, launch-time overrides, runtime inspection, controlled mutation, validation, observability, and configuration versioning. Used correctly, the parameter system separates software logic from deployment-specific settings while maintaining clear node ownership. This enables reusable ROS 2 components to operate predictably across development, simulation, production robots, and scalable Physical AI systems.

ROS 2 매개변수 시스템(Parameter System)은 모든 동작 값을 소스 코드에 직접 포함하지 않고도 노드의 동작을 구성할 수 있도록 표준화된 메커니즘을 제공한다. 매개변수(Parameter)는 개별 노드에 속하며 센서 주기, 임계값, 프레임 식별자(Frame Identifier), 알고리즘 옵션, 제어기 게인(Controller Gain), 기능 스위치(Feature Switch) 등의 설정 가능한 값을 표현한다. 이러한 노드 중심 모델(Node-Centric Model)은 런타임 설정(Runtime Setting)의 명확한 소유권을 유지하면서 구성을 명시적으로 관리할 수 있게 한다.

전역 구성 데이터베이스(Global Configuration Database)와 달리 ROS 2 매개변수는 특정 노드 인스턴스(Node Instance)에 연결된다. 각 노드는 자신이 지원하는 매개변수를 선언하고 해당 매개변수의 이름, 자료형(Type), 기본값(Default Value), 선택적 설명자(Descriptor)를 정의할 수 있다. 이러한 아키텍처는 관련 없는 구성요소가 암묵적으로 구성 상태를 공유하는 것을 방지하고 특정 설정을 어떤 노드가 소유하는지 명확하게 파악할 수 있게 하며, 이는 대규모 분산 로봇 소프트웨어 시스템에서 중요한 특성이다.

매개변수 선언(Parameter Declaration)은 노드의 구성 계약(Configuration Contract)을 정의한다. rclcpp에서는 노드가 declare_parameter()를 사용하여 매개변수를 선언하고 초기값 또는 예상 자료형을 지정할 수 있다. 매개변수를 사용하기 전에 선언하면 지원되는 구성 옵션이 노드 설계의 일부로 명확하게 표현되므로 인터페이스 명료성이 향상된다. 선언되지 않은 매개변수(Undeclared Parameter)의 동작은 노드 옵션(Node Option)을 통해 제어할 수 있지만, 예측 가능한 양산 시스템에서는 일반적으로 명시적인 선언이 더 적합하다.

ROS 2 매개변수는 대부분의 런타임 구성 요구사항에 적합한 기본 스칼라(Scalar) 및 배열(Array) 자료형을 지원한다. 대표적인 값에는 불리언(Boolean) 플래그, 정수(Integer), 부동소수점 수(Floating-Point Number), 문자열(String), 바이트 배열(Byte Array), 기본 자료형 배열이 포함된다. 복잡한 응용 프로그램 구조는 일반적으로 매개변수 메커니즘을 임의의 응용 데이터를 위한 범용 데이터베이스로 사용하는 대신 논리적으로 그룹화된 매개변수 이름이나 외부 구성 구조(External Configuration Structure)를 통해 표현한다.

계층적 명명 규칙(Hierarchical Naming Convention)을 사용하면 대규모 매개변수 집합을 더욱 쉽게 구성할 수 있다. 인지 노드(Perception Node)는 camera.width, camera.height, detection.threshold, inference.device와 같은 이름을 사용할 수 있으며, 제어기(Controller)는 control.frequency, velocity.max_linear, velocity.max_angular 등을 노출할 수 있다. 점으로 구분된 표기법(Dot-Separated Notation)은 매개변수가 궁극적으로 노드가 소유하는 개별 이름의 값으로 유지되면서도 논리적인 네임스페이스(Namespace)와 유사한 구조를 제공한다.

기본값(Default Value)은 외부 구성이 제공되지 않았을 때 노드가 어떻게 동작할지를 정의하기 때문에 중요하다. 잘 설계된 기본값은 가능한 경우 안전하고 이해하기 쉬운 초기화를 가능하게 해야 하며, 배포 환경별 값(Deployment-Specific Value)은 이를 재정의할 수 있다. 중요한 하드웨어 식별자, 보정값(Calibration Value), 안전 관련 제한값에는 단순한 편의를 위해 임의의 기본값을 지정해서는 안 된다. 겉으로는 유효해 보이는 잘못된 구성이 명시적인 구성 실패보다 더 위험할 수 있기 때문이다.

노드가 실행될 때 명령줄(Command Line)을 통해 매개변수를 제공할 수도 있다. 개별 설정을 소스 파일 수정이나 별도의 구성 파일 관리 없이 변경할 수 있으므로 테스트, 디버깅(Debugging), 일시적인 실험에 편리하다. 그러나 긴 매개변수 목록은 검토, 재현, 버전 관리(Versioning), 여러 로봇 간의 일관된 유지보수가 어려워지기 때문에 명령줄 재정의(Command-Line Override)는 대규모 양산 구성에는 상대적으로 적합하지 않다.

YAML 매개변수 파일(YAML Parameter File)은 대규모 구성을 관리하는 실용적인 방법을 제공한다. YAML 파일은 노드 이름과 ros__parameters를 연결하고 여러 값을 구조화된 형태로 정의할 수 있다. 구성 파일(Configuration File)은 로봇 소프트웨어와 함께 저장하고 버전 관리 시스템에서 추적하며 통합 과정에서 검토하고 서로 다른 하드웨어 변형이나 배포 환경에 맞게 선택할 수 있다. 이를 통해 매개변수 관리는 비공식적인 런타임 절차가 아니라 반복 가능한 시스템 구성의 일부가 된다.

실행 파일(Launch File)은 매개변수 정의를 배포 아키텍처(Deployment Architecture)와 연결할 수 있다. ROS 2 실행 기술(Launch Description)은 노드를 생성할 때 매개변수 파일, 직접 정의된 매개변수 딕셔너리(Parameter Dictionary), 또는 기본 설정과 배포별 설정의 조합을 제공할 수 있다. 이를 통해 하나의 소프트웨어 패키지가 알고리즘 구현을 변경하지 않고도 여러 로봇 변형을 지원할 수 있다. 따라서 하드웨어 모델, 센서 구성, 운용 환경, 임무 프로파일(Mission Profile)에 따라 배포 구성을 통해 런타임 동작을 조정할 수 있다.

ROS 2 명령줄 도구(Command-Line Tool)를 통해 매개변수를 검사하고 수정할 수도 있다. 엔지니어는 노드가 노출하는 매개변수를 나열하고, 현재 값을 조회하고, 설명자(Descriptor)를 확인하며, 런타임 변경을 요청할 수 있다. 이러한 기능은 실제 실행 중인 시스템에서 활성 구성을 직접 확인할 수 있기 때문에 시운전(Commissioning)과 디버깅 과정에서 유용하며, 소스 파일이나 예상 실행 인자만으로 현재 구성을 추정할 필요가 없다.

런타임 매개변수 변경(Runtime Parameter Change)은 추가적인 설계 책임을 요구한다. 노드는 새로운 값의 자료형이 문법적으로 유효하다는 이유만으로 해당 값을 받아들여서는 안 된다. 제어 주기, 검출 임계값, 좌표 프레임(Coordinate Frame), 장치 식별자 또는 자원 할당 매개변수를 변경하려면 검증(Validation)이 필요할 수 있으며, 구성요소가 동작 중일 때는 변경 자체가 안전하지 않을 수도 있다. 따라서 매개변수 변경 가능성(Parameter Mutability)은 각 설정의 운용상 의미에 따라 설계해야 한다.

ROS 2는 요청된 변경을 적용하기 전에 노드가 이를 검증할 수 있도록 매개변수 변경 콜백(Parameter-Change Callback)을 제공한다. 콜백은 수치 범위, 서로 관련된 값의 조합, 운용 상태 제한, 하드웨어 제약조건을 확인하고 잘못된 요청을 거부할 수 있다. 이러한 검증 계층(Validation Layer)은 매개변수 인터페이스를 단순한 쓰기 가능한 저장소에서 제어된 구성 경계(Controlled Configuration Boundary)로 전환하여 런타임 변경으로 인해 내부 상태가 불일치하는 것을 방지하는 데 도움을 준다.

필요한 경우 검증은 각 매개변수를 독립적으로 확인하는 것뿐 아니라 매개변수 사이의 관계도 고려해야 한다. 예를 들어 최소 범위는 최대 범위보다 작아야 하고, 이미지 크기는 지원되는 카메라 모드와 일치해야 하며, 제어기 제한값은 물리적 플랫폼의 특성에 따라 달라질 수 있다. 여러 매개변수의 의미가 상호 의존적인 경우에는 다수의 매개변수 업데이트 요청을 하나의 일관된 구성 변경(Coherent Configuration Change)으로 평가해야 한다.

매개변수 설명자(Parameter Descriptor)는 매개변수의 의도된 의미와 제약조건을 설명하는 메타데이터(Metadata)를 제공할 수 있다. 설명(Description), 읽기 전용 속성(Read-Only Property), 정수 범위(Integer Range), 부동소수점 범위(Floating-Point Range)를 제공하면 구성 인터페이스를 더욱 쉽게 이해할 수 있으며 사용자에게 매개변수 정보를 표시하는 도구도 지원할 수 있다. 설명자가 응용 프로그램 수준 검증을 대체하지는 않지만 자체 문서화(Self-Documentation)를 향상시키고 재사용 가능한 로봇 소프트웨어 구성요소의 모호성을 줄여준다.

매개변수 이벤트(Parameter Event)는 ROS 2 시스템 전체에서 발생하는 구성 변경에 대한 가시성(Visibility)을 제공한다. 매개변수가 생성, 수정 또는 삭제되면 관련 구성요소가 매개변수 관련 이벤트를 관찰할 수 있다. 이러한 이벤트 모니터링은 진단(Diagnostics), 구성 감사(Configuration Auditing), 사용자 인터페이스(User Interface), 시스템 감독(System Supervision)을 지원할 수 있다. 양산 로봇에서는 중요한 매개변수 변경을 타임스탬프(Timestamp)와 함께 기록하면 운용 중 변화한 동작을 조사할 때 유용하다.

매개변수(Parameter)는 의미적인 목적에 따라 토픽(Topic), 서비스(Service), 액션(Action)과 구분해야 한다. 매개변수는 노드와 연관된 비교적 지속적인 구성(Persistent Configuration)을 나타내는 반면, 토픽은 데이터 스트림(Data Stream), 서비스는 요청-응답 상호작용(Request-Response Interaction), 액션은 장시간 수행되는 목표 지향적 작업(Goal-Oriented Operation)을 위해 설계된다. 고주파 상태 교환이나 센서 데이터에 매개변수를 사용하면 구성 메커니즘의 목적에 맞지 않으며 부적절한 통신 아키텍처가 된다.

매개변수 시스템은 노드 수명주기 관리(Node Lifecycle Management)와도 상호작용한다. 수명주기 노드(Lifecycle Node)는 생성 과정에서 매개변수를 선언할 수 있지만, 자원에 종속된 구성은 구성 전이(Configuration Transition) 과정에서 적용할 수 있다. 일부 설정은 노드가 미구성(Unconfigured) 또는 비활성(Inactive) 상태일 때만 변경하도록 허용할 수 있으며, 다른 설정은 활성(Active) 동작 중에도 안전하게 변경할 수 있다. 매개변수 변경 가능성을 수명주기 상태와 연결하면 재구성(Reconfiguration)에 대한 운용 규칙이 더욱 명확해진다.

자율이동로봇(Autonomous Mobile Robot, AMR)에서 매개변수는 LiDAR 프레임 이름, 위치추정 임계값, 플래너(Planner) 동작, 최대 속도, 장애물 여유 거리(Obstacle Margin), 진단 주기 등을 구성할 수 있다. 매니퓰레이터(Manipulator)에서는 제어기 게인, 관절 제한(Joint Limit), 계획 옵션, 장치 인터페이스를 정의할 수 있다. 피지컬 AI(Physical AI) 노드는 추론 임계값, 모델 경로(Model Path), 가속기 선택(Accelerator Selection), 전처리 크기, 런타임 기능 스위치를 노출하여 공통 소프트웨어 구성요소를 서로 다른 배포 대상에 맞게 조정할 수 있다.

대규모 로봇 플릿(Robot Fleet)에서는 통제되지 않은 개별 로봇별 수정이 점차 구성 드리프트(Configuration Drift)를 발생시킬 수 있기 때문에 체계적인 매개변수 거버넌스(Parameter Governance)가 필요하다. 매개변수 파일은 버전 관리되어야 하고, 배포 변형(Deployment Variant)은 식별 가능해야 하며, 중요한 런타임 재정의는 추적할 수 있어야 한다. 이상적으로 로봇은 소프트웨어 버전뿐 아니라 중요 노드가 실제 사용하고 있는 유효 구성(Effective Configuration)도 보고하여 현장 동작을 테스트 및 장애 분석 과정에서 재현할 수 있어야 한다.

런타임 매개변수 수정은 로봇의 동작을 변화시킬 수 있기 때문에 보안(Security)도 고려해야 한다. 속도 제한, 인지 임계값, 통신 종단점(Communication Endpoint), 안전 관련 운용 조건을 제어하는 매개변수를 단순하고 무해한 구성값으로 취급해서는 안 된다. 매개변수 서비스(Parameter Service)에 대한 접근은 ROS 2 배포의 보안 아키텍처를 따라야 하며, 중요한 변경에는 추가적인 권한 부여(Authorization), 검증, 로깅 또는 상위 수준 감독 제어(Supervisory Control)가 필요할 수 있다.

따라서 강건한 ROS 2 매개변수 아키텍처(Robust ROS 2 Parameter Architecture)는 명시적 선언(Explicit Declaration), 의미 있는 기본값, 구조화된 YAML 구성, 실행 시점 재정의(Launch-Time Override), 런타임 검사(Runtime Inspection), 제어된 변경(Controlled Mutation), 검증, 관측 가능성(Observability), 구성 버전 관리(Configuration Versioning)를 결합한다. 매개변수 시스템을 올바르게 사용하면 명확한 노드 소유권을 유지하면서 소프트웨어 로직과 배포별 설정을 분리할 수 있다. 이를 통해 재사용 가능한 ROS 2 구성요소가 개발, 시뮬레이션, 양산 로봇 및 확장 가능한 피지컬 AI 시스템 전반에서 예측 가능하게 동작할 수 있다.

##  

## 03.04 Pluginlib-Based Extensible Node Design [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 pluginlib provides a runtime plugin mechanism that allows software components to select and load implementations without statically binding every implementation into the application architecture. A node can depend on an abstract interface while concrete algorithms are supplied as dynamically loadable plugins. This pattern separates stable application logic from replaceable functionality and enables extensible robotic software that can evolve without repeatedly modifying the core node.

The architectural foundation of pluginlib is interface-based design. A developer first defines a base class representing the contract that every plugin implementation must satisfy. The interface normally contains virtual methods describing required behavior while avoiding implementation-specific assumptions. Navigation algorithms, planners, controllers, filters, cost functions, perception processors, or hardware strategies can therefore share one common interface while providing substantially different internal algorithms.

Concrete plugin classes derive from the common base interface and implement its required functions. The host node does not need detailed knowledge of these classes during its normal control flow. Instead, it interacts with objects through pointers or references to the base interface. This use of polymorphism reduces coupling because application logic depends on an abstraction rather than directly depending on each algorithm implementation.

Plugins are commonly compiled into shared libraries that can be discovered and loaded at runtime. The plugin implementation is exported using pluginlib-compatible registration mechanisms so that the framework can associate a plugin class with its base type. The host application can then request a particular implementation by its registered identifier instead of constructing the concrete class directly in source code.

Plugin description files provide metadata connecting symbolic plugin names to their implementation classes and shared libraries. These descriptions allow pluginlib to determine which library contains a requested class and which base interface the implementation supports. Package metadata exports the plugin description so that other ROS 2 packages can discover available implementations through the ROS package environment without hard-coded absolute library paths.

The plugin loader, typically represented by pluginlib::ClassLoader in C++, forms the runtime bridge between the host node and dynamically loaded implementations. It is configured with the package and base-class type associated with a plugin family. The application can query declared classes and instantiate a selected implementation. The resulting object is accessed through the common interface, keeping plugin-specific construction details outside the main algorithmic logic.

Configuration and plugin selection are often connected through ROS 2 parameters. A node may expose a parameter such as planner_plugin, controller_plugin, filter_type, or perception_backend whose value identifies the implementation to load. YAML configuration or launch files can then select different plugins for different robots or operating environments while the executable and high-level node logic remain unchanged.

This architecture is particularly useful when several algorithms provide the same functional role. A navigation system may support multiple global planners or local controllers, while a perception pipeline may offer alternative filtering, segmentation, or detection strategies. Instead of implementing a large conditional structure inside one node, each strategy can be encapsulated as an independent plugin implementing the same interface contract.

Plugin-based design also improves package modularity. A core package can contain interfaces and common infrastructure while separate packages provide concrete implementations. New algorithms can therefore be added as additional packages without requiring direct modification of the host package. This separation supports independent development, testing, versioning, and distribution, which becomes increasingly valuable as a robotic software ecosystem grows.

Runtime extensibility does not necessarily imply unrestricted hot swapping during active robot operation. Loading a new implementation may require resource initialization, parameter validation, memory allocation, hardware access, or reconstruction of algorithm state. If plugins must be replaced while the system is running, the architecture should define controlled transitions that prevent callbacks from accessing objects while they are being destroyed or replaced.

Lifecycle nodes can provide a useful structure for controlled plugin management. A plugin may be selected and instantiated during the Configure transition, activated when the node enters Active, suspended during Deactivate, and released during Cleanup. This approach aligns plugin resource ownership with explicit node states and makes reconfiguration behavior easier to reason about than replacing active algorithm objects at arbitrary execution points.

Thread safety becomes important when plugin methods are invoked from callbacks managed by ROS 2 executors. A plugin object may be accessed by subscriptions, timers, services, or actions executing concurrently, particularly under a MultiThreadedExecutor. The plugin interface and implementation must clearly define whether concurrent invocation is permitted and which internal resources require synchronization to prevent races or inconsistent state.

Failure handling must also be designed at the plugin boundary. A requested plugin may not exist, its shared library may fail to load, construction may fail, or initialization may reject the current configuration. The host node should detect these conditions and produce useful diagnostic information rather than continuing with an invalid object. Critical systems may define fallback implementations, but fallback behavior should be explicit and validated rather than silently changing algorithms.

Interface stability is one of the most important requirements for a sustainable plugin architecture. When the base interface changes, existing plugin implementations may require modification and rebuilding. Interfaces should therefore capture stable functional contracts and avoid exposing unnecessary implementation details. Versioning strategies become necessary when major interface changes cannot remain backward compatible across independently maintained plugin packages.

Plugin interfaces should also define ownership and lifetime expectations for exchanged data. Passing raw implementation-specific structures across the abstraction boundary can tightly couple plugins to the host application. Preferable interfaces use stable ROS messages, standard C++ types, clearly defined domain structures, or carefully designed abstract data contracts. This allows implementations to evolve internally without forcing corresponding changes throughout the surrounding node architecture.

Testing benefits from the separation created by pluginlib. Individual plugin implementations can be unit-tested against the base interface, while the host node can be tested using simple mock or test plugins. Integration tests can verify plugin discovery, library loading, configuration, initialization, and execution. This separation makes it easier to determine whether a failure originates from core node orchestration or from a specific algorithm implementation.

Performance considerations remain relevant because abstraction does not eliminate computational cost. Virtual function dispatch and plugin loading usually represent small overhead compared with perception, planning, or control algorithms, but initialization time, memory allocation, and library dependencies can affect startup behavior. Performance-sensitive applications should measure complete execution paths and avoid assuming that either static integration or plugin-based integration is inherently faster.

Security and deployment governance are also important in production systems. Dynamically loading arbitrary libraries can create significant risk if plugin selection or installed packages are not controlled. Production robots should load only trusted and validated plugin implementations from managed software distributions. Plugin configuration, package versions, and library integrity should be included in deployment and software update policies.

A practical AMR architecture might define common interfaces for global planning, local control, docking behavior, obstacle processing, or recovery strategies. Different robot models can then select implementations according to drivetrain geometry, sensor configuration, operating environment, or mission requirements. The same host architecture can support indoor logistics robots, outdoor AMRs, or specialized inspection platforms without embedding every platform-specific algorithm into one monolithic node.

Physical AI systems can apply the same pattern to inference backends, preprocessing strategies, model adapters, semantic reasoning modules, or action-generation components. A common interface may allow one deployment to use a lightweight edge inference implementation while another uses a larger GPU-accelerated model. Plugin selection can therefore become a controlled mechanism for adapting a common ROS 2 software architecture to different compute capabilities.

The plugin pattern should not be applied indiscriminately. Functions that are permanently fixed, extremely simple, or tightly coupled to core invariants may gain little from dynamic extensibility. Excessive plugin boundaries can increase configuration complexity and make dependencies harder to understand. Pluginlib is most effective where multiple interchangeable implementations genuinely exist or where future extension is an explicit architectural requirement.

A robust pluginlib-based ROS 2 architecture therefore combines stable abstract interfaces, independently packaged implementations, runtime discovery, configuration-driven selection, controlled lifecycle management, explicit failure handling, thread-safe execution, testing, and deployment governance. Applied to appropriate extension points, pluginlib transforms algorithm variability from hard-coded branching into a modular architectural capability, enabling ROS 2 nodes to remain reusable as robotic platforms and Physical AI functions evolve.

ROS 2 플러그인라이브러리(pluginlib)는 모든 구현을 응용 프로그램 아키텍처에 정적으로 결합하지 않고도 소프트웨어 구성요소가 런타임(Runtime)에 구현을 선택하고 로드할 수 있도록 하는 플러그인 메커니즘(Plugin Mechanism)을 제공한다. 노드는 추상 인터페이스(Abstract Interface)에 의존하고, 구체적인 알고리즘은 동적으로 로드 가능한 플러그인(Dynamically Loadable Plugin)으로 제공할 수 있다. 이 패턴은 안정적인 응용 프로그램 로직과 교체 가능한 기능을 분리하여 핵심 노드를 반복적으로 수정하지 않고도 확장 가능한 로봇 소프트웨어를 발전시킬 수 있도록 한다.

플러그인라이브러리(pluginlib)의 아키텍처 기반은 인터페이스 기반 설계(Interface-Based Design)이다. 개발자는 먼저 모든 플러그인 구현이 준수해야 하는 계약을 나타내는 기반 클래스(Base Class)를 정의한다. 이 인터페이스는 일반적으로 구현별 가정을 최소화하면서 필요한 동작을 정의하는 가상 함수(Virtual Method)를 포함한다. 따라서 내비게이션 알고리즘, 플래너(Planner), 제어기(Controller), 필터(Filter), 비용 함수(Cost Function), 인지 처리기(Perception Processor), 하드웨어 전략 등이 하나의 공통 인터페이스를 공유하면서 서로 크게 다른 내부 알고리즘을 제공할 수 있다.

구체적인 플러그인 클래스(Concrete Plugin Class)는 공통 기반 인터페이스를 상속하고 필요한 함수를 구현한다. 호스트 노드(Host Node)는 일반적인 제어 흐름에서 이러한 클래스의 세부 내용을 알 필요가 없다. 대신 기반 인터페이스에 대한 포인터(Pointer) 또는 참조(Reference)를 통해 객체와 상호작용한다. 이러한 다형성(Polymorphism)의 활용은 응용 프로그램 로직이 각각의 알고리즘 구현에 직접 의존하는 대신 추상화(Abstraction)에 의존하도록 하여 결합도(Coupling)를 낮춘다.

플러그인은 일반적으로 런타임에 검색하고 로드할 수 있는 공유 라이브러리(Shared Library)로 컴파일된다. 플러그인 구현은 플러그인라이브러리와 호환되는 등록 메커니즘(Registration Mechanism)을 통해 내보내져 프레임워크가 플러그인 클래스와 기반 자료형(Base Type)을 연결할 수 있도록 한다. 이후 호스트 응용 프로그램은 소스 코드에서 구체적인 클래스를 직접 생성하는 대신 등록된 식별자(Registered Identifier)를 사용하여 특정 구현을 요청할 수 있다.

플러그인 설명 파일(Plugin Description File)은 기호화된 플러그인 이름(Symbolic Plugin Name)을 구현 클래스와 공유 라이브러리에 연결하는 메타데이터(Metadata)를 제공한다. 이러한 설명을 통해 플러그인라이브러리는 요청된 클래스가 어느 라이브러리에 포함되어 있으며 해당 구현이 어떤 기반 인터페이스를 지원하는지 판단할 수 있다. 패키지 메타데이터(Package Metadata)는 플러그인 설명을 내보내므로 다른 ROS 2 패키지가 절대 라이브러리 경로를 하드코딩하지 않고도 ROS 패키지 환경을 통해 사용 가능한 구현을 검색할 수 있다.

C++에서 일반적으로 pluginlib::ClassLoader로 표현되는 플러그인 로더(Plugin Loader)는 호스트 노드와 동적으로 로드되는 구현 사이의 런타임 연결 역할을 한다. 플러그인 로더는 플러그인 계열(Plugin Family)에 연결된 패키지와 기반 클래스 자료형으로 구성된다. 응용 프로그램은 선언된 클래스를 조회하고 선택한 구현을 인스턴스화할 수 있다. 생성된 객체는 공통 인터페이스를 통해 접근되므로 플러그인별 생성 세부사항이 주요 알고리즘 로직과 분리된다.

구성과 플러그인 선택은 ROS 2 매개변수(Parameter)를 통해 연결되는 경우가 많다. 노드는 planner_plugin, controller_plugin, filter_type, perception_backend와 같이 로드할 구현을 식별하는 매개변수를 노출할 수 있다. 이후 YAML 구성이나 실행 파일(Launch File)을 사용하여 실행 파일과 상위 수준 노드 로직을 변경하지 않고도 서로 다른 로봇 또는 운용 환경에 맞는 플러그인을 선택할 수 있다.

이 아키텍처는 여러 알고리즘이 동일한 기능적 역할을 제공하는 경우 특히 유용하다. 내비게이션 시스템은 여러 전역 플래너(Global Planner) 또는 로컬 제어기(Local Controller)를 지원할 수 있으며, 인지 파이프라인(Perception Pipeline)은 서로 다른 필터링, 세그멘테이션(Segmentation), 검출(Detection) 전략을 제공할 수 있다. 하나의 노드 내부에 거대한 조건문 구조를 구현하는 대신 각각의 전략을 동일한 인터페이스 계약을 구현하는 독립적인 플러그인으로 캡슐화(Encapsulation)할 수 있다.

플러그인 기반 설계(Plugin-Based Design)는 패키지 모듈성(Package Modularity)도 향상시킨다. 핵심 패키지는 인터페이스와 공통 인프라를 포함하고 별도의 패키지가 구체적인 구현을 제공할 수 있다. 따라서 호스트 패키지를 직접 수정하지 않고도 새로운 알고리즘을 추가 패키지 형태로 확장할 수 있다. 이러한 분리는 독립적인 개발, 테스트, 버전 관리(Versioning), 배포(Distribution)를 지원하며 로봇 소프트웨어 생태계가 확장될수록 그 가치가 커진다.

런타임 확장성(Runtime Extensibility)이 반드시 로봇이 동작 중인 상태에서 제한 없는 핫 스와핑(Hot Swapping)을 의미하는 것은 아니다. 새로운 구현을 로드하려면 자원 초기화, 매개변수 검증, 메모리 할당, 하드웨어 접근 또는 알고리즘 상태 재구성이 필요할 수 있다. 시스템이 실행되는 동안 플러그인을 교체해야 한다면 콜백이 제거되거나 교체되는 객체에 접근하지 못하도록 방지하는 제어된 전이(Controlled Transition)를 아키텍처에서 정의해야 한다.

수명주기 노드(Lifecycle Node)는 제어된 플러그인 관리를 위한 유용한 구조를 제공할 수 있다. 플러그인은 구성 전이(Configure Transition)에서 선택되고 인스턴스화될 수 있으며, 노드가 활성(Active) 상태에 진입할 때 활성화되고 비활성화(Deactivate) 과정에서 동작이 중단되며 정리(Cleanup) 과정에서 해제될 수 있다. 이러한 접근법은 플러그인의 자원 소유권(Resource Ownership)을 명시적인 노드 상태와 일치시키며, 활성 알고리즘 객체를 임의의 실행 시점에서 교체하는 방식보다 재구성 동작을 명확하게 이해할 수 있도록 한다.

ROS 2 실행기(Executor)가 관리하는 콜백에서 플러그인 메서드가 호출되는 경우 스레드 안전성(Thread Safety)이 중요해진다. 특히 다중 스레드 실행기(MultiThreadedExecutor)를 사용하는 경우 플러그인 객체는 동시에 실행되는 구독(Subscription), 타이머(Timer), 서비스(Service), 액션(Action)에서 접근될 수 있다. 플러그인 인터페이스와 구현은 동시 호출이 허용되는지 명확하게 정의해야 하며, 경쟁 상태(Race Condition)나 일관되지 않은 상태를 방지하기 위해 어떤 내부 자원에 동기화(Synchronization)가 필요한지 규정해야 한다.

장애 처리(Failure Handling)도 플러그인 경계(Plugin Boundary)에서 설계되어야 한다. 요청된 플러그인이 존재하지 않거나, 공유 라이브러리 로드에 실패하거나, 객체 생성이 실패하거나, 초기화 과정에서 현재 구성이 거부될 수 있다. 호스트 노드는 이러한 조건을 감지하고 유효하지 않은 객체를 사용하여 계속 실행하는 대신 유용한 진단 정보를 제공해야 한다. 중요 시스템에서는 대체 구현(Fallback Implementation)을 정의할 수 있지만, 대체 동작은 알고리즘을 암묵적으로 변경하는 방식이 아니라 명시적으로 정의되고 검증되어야 한다.

인터페이스 안정성(Interface Stability)은 지속 가능한 플러그인 아키텍처에서 가장 중요한 요구사항 중 하나이다. 기반 인터페이스가 변경되면 기존 플러그인 구현을 수정하고 다시 빌드해야 할 수 있다. 따라서 인터페이스는 안정적인 기능 계약(Functional Contract)을 표현해야 하며 불필요한 구현 세부사항을 노출하지 않아야 한다. 독립적으로 유지되는 여러 플러그인 패키지 사이에서 주요 인터페이스 변경에 대한 하위 호환성(Backward Compatibility)을 유지할 수 없는 경우에는 버전 관리 전략이 필요하다.

플러그인 인터페이스는 교환되는 데이터의 소유권(Ownership)과 수명(Lifetime)에 대한 기대사항도 정의해야 한다. 구현별 원시 데이터 구조를 추상화 경계를 통해 전달하면 플러그인과 호스트 응용 프로그램이 강하게 결합될 수 있다. 안정적인 ROS 메시지, 표준 C++ 자료형, 명확하게 정의된 도메인 구조(Domain Structure) 또는 신중하게 설계된 추상 데이터 계약(Abstract Data Contract)을 사용하는 것이 바람직하다. 이를 통해 주변 노드 아키텍처 전체를 변경하지 않고도 각 구현의 내부 구조를 발전시킬 수 있다.

테스트(Testing)는 플러그인라이브러리가 제공하는 분리 구조의 이점을 얻을 수 있다. 개별 플러그인 구현은 기반 인터페이스를 기준으로 단위 테스트(Unit Test)할 수 있으며, 호스트 노드는 간단한 모의 플러그인(Mock Plugin)이나 테스트 플러그인을 사용하여 검증할 수 있다. 통합 테스트(Integration Test)는 플러그인 검색, 라이브러리 로드, 구성, 초기화, 실행을 검증할 수 있다. 이러한 분리를 통해 장애가 핵심 노드 오케스트레이션(Core Node Orchestration)에서 발생했는지 특정 알고리즘 구현에서 발생했는지 더욱 쉽게 판단할 수 있다.

추상화가 계산 비용을 제거하는 것은 아니기 때문에 성능 고려사항(Performance Consideration)도 여전히 중요하다. 가상 함수 디스패치(Virtual Function Dispatch)와 플러그인 로딩의 오버헤드는 일반적으로 인지, 계획 또는 제어 알고리즘의 계산량과 비교하면 작지만 초기화 시간, 메모리 할당, 라이브러리 의존성은 시작 동작에 영향을 줄 수 있다. 성능에 민감한 응용 프로그램에서는 전체 실행 경로를 측정해야 하며 정적 통합 또는 플러그인 기반 통합 중 어느 한쪽이 본질적으로 더 빠르다고 가정해서는 안 된다.

양산 시스템에서는 보안(Security)과 배포 거버넌스(Deployment Governance) 역시 중요하다. 플러그인 선택이나 설치된 패키지가 통제되지 않는 상황에서 임의의 라이브러리를 동적으로 로드하면 상당한 위험이 발생할 수 있다. 양산 로봇은 관리되는 소프트웨어 배포판(Managed Software Distribution)에서 제공되는 신뢰할 수 있고 검증된 플러그인 구현만 로드해야 한다. 플러그인 구성, 패키지 버전, 라이브러리 무결성(Library Integrity)은 배포 및 소프트웨어 업데이트 정책에 포함되어야 한다.

실제 자율이동로봇(Autonomous Mobile Robot, AMR) 아키텍처에서는 전역 계획(Global Planning), 로컬 제어(Local Control), 도킹 동작(Docking Behavior), 장애물 처리(Obstacle Processing), 복구 전략(Recovery Strategy)을 위한 공통 인터페이스를 정의할 수 있다. 이후 서로 다른 로봇 모델은 구동계 형상(Drivetrain Geometry), 센서 구성, 운용 환경, 임무 요구사항에 따라 구현을 선택할 수 있다. 동일한 호스트 아키텍처가 모든 플랫폼별 알고리즘을 하나의 거대한 노드에 포함하지 않고도 실내 물류 로봇, 실외 AMR 또는 특수 검사 플랫폼을 지원할 수 있다.

피지컬 AI 시스템(Physical AI System)은 추론 백엔드(Inference Backend), 전처리 전략(Preprocessing Strategy), 모델 어댑터(Model Adapter), 의미론적 추론 모듈(Semantic Reasoning Module), 행동 생성 구성요소(Action-Generation Component)에 동일한 패턴을 적용할 수 있다. 공통 인터페이스를 사용하면 한 배포 환경에서는 경량 엣지 추론(Lightweight Edge Inference) 구현을 사용하고 다른 환경에서는 대규모 GPU 가속 모델을 사용할 수 있다. 따라서 플러그인 선택은 공통 ROS 2 소프트웨어 아키텍처를 서로 다른 컴퓨팅 성능에 맞게 조정하는 제어된 메커니즘이 될 수 있다.

플러그인 패턴(Plugin Pattern)을 모든 기능에 무분별하게 적용해서는 안 된다. 영구적으로 고정되어 있거나 매우 단순하거나 핵심 불변조건(Core Invariant)과 긴밀하게 결합된 기능은 동적 확장성으로 얻을 수 있는 이점이 적을 수 있다. 지나치게 많은 플러그인 경계는 구성 복잡성을 증가시키고 의존 관계를 이해하기 어렵게 만들 수 있다. 플러그인라이브러리는 실제로 여러 교체 가능한 구현이 존재하거나 향후 확장이 명시적인 아키텍처 요구사항인 영역에서 가장 효과적이다.

따라서 강건한 플러그인라이브러리 기반 ROS 2 아키텍처(Robust Pluginlib-Based ROS 2 Architecture)는 안정적인 추상 인터페이스, 독립적으로 패키징된 구현, 런타임 검색(Runtime Discovery), 구성 기반 선택(Configuration-Driven Selection), 제어된 수명주기 관리, 명시적인 장애 처리, 스레드 안전 실행(Thread-Safe Execution), 테스트 및 배포 거버넌스를 결합한다. 적절한 확장 지점(Extension Point)에 적용하면 플러그인라이브러리는 알고리즘의 다양성을 하드코딩된 조건 분기에서 모듈형 아키텍처 기능으로 전환하여 로봇 플랫폼과 피지컬 AI 기능이 발전하더라도 ROS 2 노드를 지속적으로 재사용할 수 있도록 한다.

##  

## 03.05 Node Callback Groups and Executor Design [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 callback groups and executors define how callbacks are scheduled and allowed to execute relative to one another. A node may contain subscriptions, timers, services, clients, and actions that generate executable callbacks, but their timing behavior is not determined by the node abstraction alone. Executors select ready work, while callback groups express concurrency constraints among callbacks, providing an architectural mechanism for controlling execution behavior.

An executor connects ROS 2 communication events to application callback execution. It monitors entities associated with one or more nodes and determines which ready callback should execute. Rather than creating a dedicated application thread for every subscription or timer, ROS 2 centralizes callback scheduling through executors. This model allows developers to control threading and concurrency independently from the logical communication interfaces exposed by individual nodes.

The SingleThreadedExecutor executes callbacks using one execution thread. Only one callback handled by that executor can execute at a time, which simplifies reasoning about shared state and eliminates many forms of simultaneous access. This model is useful for relatively simple nodes or systems where callback execution times are short, but a long-running callback can delay every other callback waiting on the same executor.

The MultiThreadedExecutor provides multiple worker threads that can execute eligible callbacks concurrently. This can improve responsiveness and computational utilization when callbacks are independent or frequently wait for external operations. However, adding executor threads does not automatically make every callback parallel. Actual concurrency depends on callback-group configuration, available ready work, synchronization, and whether application code can safely execute simultaneously.

Callback groups define concurrency relationships within nodes. ROS 2 provides two principal group types: MutuallyExclusive and Reentrant. Entities such as subscriptions, timers, service servers, clients, and action-related callbacks are associated with callback groups. The executor considers these group semantics when selecting work, allowing developers to express which callbacks must avoid overlapping execution and which callbacks may execute concurrently.

A MutuallyExclusive callback group permits only one callback belonging to that group to execute at a time. Even when a MultiThreadedExecutor has several available worker threads, callbacks in the same mutually exclusive group are serialized relative to one another. This behavior is useful when callbacks share state that should not be accessed concurrently or when an algorithm requires a predictable sequence of operations without explicit fine-grained locking.

A Reentrant callback group permits multiple callbacks from the group to execute concurrently when executor resources are available. This can include different callbacks and potentially multiple invocations of the same callback. Reentrant groups are therefore appropriate only when the corresponding application logic is designed for concurrent execution. Shared mutable state must be protected, eliminated, or structured so that overlapping callback execution cannot produce races.

Every callback-capable entity belongs to a callback group, either one explicitly assigned by the developer or a default group associated with the node. This detail is important because using a MultiThreadedExecutor alone may provide little expected concurrency if all relevant callbacks remain in one mutually exclusive default group. Effective executor design therefore requires intentional callback-group assignment rather than simply increasing the number of executor threads.

Consider a node containing a high-rate sensor subscription, a periodic control timer, and a diagnostic service. If all callbacks share one mutually exclusive group, expensive sensor processing may delay the control timer and service response. Separating them into appropriate groups can allow independent execution under a MultiThreadedExecutor, provided that shared data is synchronized and the underlying algorithms support concurrent access.

Callback groups can therefore represent architectural execution domains. A control-related group may serialize state estimation and command generation, while a diagnostic group executes independently. A computational perception callback may be placed in another group so that it does not unnecessarily block lightweight communication callbacks. This organization makes concurrency policy visible in the node structure rather than scattering thread-management decisions throughout application code.

Concurrency introduces race conditions when callbacks access shared mutable data without correct synchronization. A subscriber may update a state estimate while a timer reads it, or a service may modify configuration while another callback uses the same object. Mutexes, atomics, immutable data, ownership transfer, message passing, or carefully designed state snapshots can be used depending on the access pattern and required timing behavior.

Synchronization itself can introduce latency and contention. Protecting a large shared structure with one coarse mutex may prevent data corruption but serialize callbacks that were intended to run concurrently. Fine-grained locking can increase parallelism but also increases implementation complexity and deadlock risk. Executor architecture should therefore be designed together with data ownership rather than treating callback groups and synchronization as unrelated concerns.

Blocking operations require particular attention. A callback that waits synchronously for a service response, action result, device operation, or other event may occupy an executor thread for an extended period. If completion requires another callback that cannot execute because of callback-group restrictions or exhausted executor threads, the system can deadlock or suffer severe latency. Asynchronous interaction patterns are often preferable inside callback-driven architectures.

Deadlocks can occur even when several executor threads exist. For example, a callback may make a synchronous request and wait for a completion callback associated with the same MutuallyExclusive group. Because the waiting callback still occupies the group\'s execution permission, the completion callback cannot run. Understanding callback-group relationships is therefore essential when using synchronous service or action operations from within callbacks.

Executor design also affects latency, jitter, and responsiveness. A high-priority conceptual function does not automatically receive operating-system real-time priority merely because it is placed in a separate callback group. Callback groups control eligibility for concurrent execution, while thread scheduling is governed by the executor implementation and operating system. Systems with strict real-time requirements may require specialized executors, thread configuration, CPU affinity, and real-time operating-system policies.

Long-running computation should be evaluated carefully before placing it directly inside a callback. Neural-network inference, large point-cloud processing, optimization, or planning may consume enough time to interfere with communication and control callbacks. Such workloads can be separated into dedicated execution contexts, worker threads, processes, or specialized pipelines while ROS 2 callbacks perform bounded data transfer and coordination.

Composition further increases the importance of executor design because multiple nodes can share one process and potentially one executor. A composed perception pipeline may contain camera, preprocessing, inference, and visualization nodes whose callbacks compete for the same worker threads. Efficient intra-process communication can reduce data-transfer overhead, but poor callback scheduling can still create latency, starvation, or contention within the shared process.

Multiple executors can be used when stronger execution separation is required. Different nodes or callback groups may be assigned to separate executor contexts so that control, perception, diagnostics, and background processing do not all compete within one scheduling domain. This approach can provide clearer resource boundaries, although additional executors and threads increase system complexity and must still be coordinated with shared resources.

Observability is essential when tuning callback execution. Engineers should measure callback duration, scheduling delay, execution frequency, queueing behavior, CPU utilization, and end-to-end latency under realistic load. Tracing tools can reveal whether a callback is delayed because another callback is executing, because no executor thread is available, or because synchronization blocks progress. Executor configuration should be validated empirically rather than selected only from theoretical expectations.

For an AMR, callback architecture might isolate motion-control timers from computationally intensive LiDAR or camera processing while allowing diagnostics and fleet communication to proceed independently. Manipulators may separate high-rate state updates from planning services, while Physical AI systems may isolate GPU inference coordination from safety monitoring. The exact grouping should reflect timing dependencies, shared state, computational load, and failure consequences.

A robust ROS 2 execution architecture therefore combines intentional callback-group assignment, appropriate executor selection, explicit data ownership, bounded callback behavior, careful synchronization, and performance measurement. Single-threaded execution offers simplicity, while multi-threaded execution provides controlled concurrency when the software is designed to support it. Together, callback groups and executors transform callback scheduling into an explicit architectural decision for scalable robotic and Physical AI systems.

ROS 2 콜백 그룹(Callback Group)과 실행기(Executor)는 콜백(Callback)이 어떻게 스케줄링되고 서로 어떤 관계로 실행될 수 있는지를 정의한다. 하나의 노드는 실행 가능한 콜백을 생성하는 구독(Subscription), 타이머(Timer), 서비스(Service), 클라이언트(Client), 액션(Action)을 포함할 수 있지만, 이들의 시간적 실행 특성은 노드 추상화(Node Abstraction)만으로 결정되지 않는다. 실행기는 준비된 작업을 선택하고, 콜백 그룹은 콜백 사이의 동시성 제약조건(Concurrency Constraint)을 표현하여 실행 동작을 제어하는 아키텍처 메커니즘을 제공한다.

실행기(Executor)는 ROS 2 통신 이벤트(Communication Event)를 응용 프로그램 콜백 실행과 연결한다. 실행기는 하나 이상의 노드와 연결된 엔티티(Entity)를 감시하고 준비된 콜백 중 어떤 것을 실행할지 결정한다. 각각의 구독이나 타이머마다 전용 응용 프로그램 스레드(Thread)를 생성하는 대신 ROS 2는 실행기를 통해 콜백 스케줄링(Callback Scheduling)을 중앙화한다. 이 모델을 통해 개발자는 개별 노드가 제공하는 논리적 통신 인터페이스와 독립적으로 스레딩(Threading)과 동시성(Concurrency)을 제어할 수 있다.

단일 스레드 실행기(SingleThreadedExecutor)는 하나의 실행 스레드를 사용하여 콜백을 실행한다. 해당 실행기가 처리하는 콜백 중 한 번에 하나만 실행될 수 있으므로 공유 상태(Shared State)를 비교적 쉽게 추론할 수 있으며 여러 형태의 동시 접근을 제거할 수 있다. 이 모델은 비교적 단순한 노드나 콜백 실행시간이 짧은 시스템에 유용하지만, 장시간 실행되는 하나의 콜백이 동일한 실행기에서 대기하는 다른 모든 콜백을 지연시킬 수 있다.

다중 스레드 실행기(MultiThreadedExecutor)는 여러 작업 스레드(Worker Thread)를 제공하여 실행 가능한 콜백을 동시에 처리할 수 있도록 한다. 콜백들이 서로 독립적이거나 외부 작업을 자주 기다리는 경우 응답성과 계산 자원 활용도를 향상시킬 수 있다. 그러나 실행기 스레드 수를 증가시킨다고 해서 모든 콜백이 자동으로 병렬 실행되는 것은 아니다. 실제 동시성은 콜백 그룹 구성, 실행 준비 상태의 작업, 동기화(Synchronization), 응용 프로그램 코드의 동시 실행 안전성에 따라 결정된다.

콜백 그룹(Callback Group)은 노드 내부에서 콜백 사이의 동시성 관계를 정의한다. ROS 2는 상호 배타적(MutuallyExclusive) 그룹과 재진입 가능(Reentrant) 그룹이라는 두 가지 주요 그룹 유형을 제공한다. 구독, 타이머, 서비스 서버(Service Server), 클라이언트 및 액션 관련 콜백과 같은 엔티티는 콜백 그룹과 연결된다. 실행기는 작업을 선택할 때 이러한 그룹의 의미론(Semantics)을 고려하여 어떤 콜백이 중첩 실행을 피해야 하고 어떤 콜백이 동시에 실행될 수 있는지를 표현할 수 있도록 한다.

상호 배타적 콜백 그룹(MutuallyExclusive Callback Group)은 해당 그룹에 속하는 콜백 중 한 번에 하나만 실행되도록 허용한다. 다중 스레드 실행기에서 여러 작업 스레드를 사용할 수 있더라도 동일한 상호 배타적 그룹에 속한 콜백은 서로 직렬화(Serialization)되어 실행된다. 이러한 동작은 콜백들이 동시에 접근해서는 안 되는 상태를 공유하거나, 명시적인 세분화 잠금(Fine-Grained Locking) 없이 알고리즘이 예측 가능한 작업 순서를 요구하는 경우 유용하다.

재진입 가능 콜백 그룹(Reentrant Callback Group)은 실행기 자원을 사용할 수 있을 때 그룹 내부의 여러 콜백을 동시에 실행할 수 있도록 허용한다. 여기에는 서로 다른 콜백뿐 아니라 잠재적으로 동일한 콜백의 여러 호출도 포함될 수 있다. 따라서 재진입 가능 그룹은 해당 응용 프로그램 로직이 동시 실행을 지원하도록 설계된 경우에만 적합하다. 공유 변경 가능 상태(Shared Mutable State)는 보호하거나 제거하거나 중첩된 콜백 실행으로 경쟁 상태(Race Condition)가 발생하지 않도록 구조화해야 한다.

콜백 실행이 가능한 모든 엔티티는 개발자가 명시적으로 지정한 콜백 그룹 또는 노드와 연결된 기본 그룹(Default Group)에 속한다. 이는 다중 스레드 실행기만 사용한다고 해서 예상한 수준의 동시성이 반드시 제공되는 것은 아니라는 점에서 중요하다. 관련 콜백이 모두 하나의 상호 배타적인 기본 그룹에 남아 있다면 다중 스레드 실행기의 효과가 매우 제한될 수 있다. 따라서 효과적인 실행기 설계에는 단순히 실행기 스레드 수를 늘리는 것이 아니라 의도적인 콜백 그룹 할당이 필요하다.

고주기 센서 구독(High-Rate Sensor Subscription), 주기적인 제어 타이머(Control Timer), 진단 서비스(Diagnostic Service)를 포함하는 노드를 생각할 수 있다. 모든 콜백이 하나의 상호 배타적 그룹을 공유하면 계산량이 많은 센서 처리가 제어 타이머와 서비스 응답을 지연시킬 수 있다. 공유 데이터가 적절히 동기화되고 기본 알고리즘이 동시 접근을 지원한다면 이들을 적절한 그룹으로 분리하여 다중 스레드 실행기에서 독립적으로 실행할 수 있다.

따라서 콜백 그룹은 아키텍처 수준의 실행 도메인(Execution Domain)을 나타낼 수 있다. 제어 관련 그룹에서는 상태추정(State Estimation)과 명령 생성을 직렬화할 수 있으며, 진단 그룹은 이와 독립적으로 실행할 수 있다. 계산량이 많은 인지 콜백(Perception Callback)은 별도의 그룹에 배치하여 가벼운 통신 콜백을 불필요하게 차단하지 않도록 할 수 있다. 이러한 구성은 스레드 관리 결정을 응용 프로그램 코드 곳곳에 분산시키는 대신 노드 구조에서 동시성 정책을 명확하게 표현한다.

콜백이 적절한 동기화 없이 공유 변경 가능 데이터에 접근하면 동시성으로 인해 경쟁 상태(Race Condition)가 발생한다. 구독 콜백이 상태추정 값을 갱신하는 동안 타이머가 해당 값을 읽거나, 서비스가 구성을 수정하는 동안 다른 콜백이 동일한 객체를 사용할 수 있다. 접근 패턴과 요구되는 시간적 동작에 따라 뮤텍스(Mutex), 원자적 연산(Atomic Operation), 불변 데이터(Immutable Data), 소유권 전달(Ownership Transfer), 메시지 전달(Message Passing), 신중하게 설계된 상태 스냅샷(State Snapshot)을 사용할 수 있다.

동기화(Synchronization) 자체도 지연시간과 경합(Contention)을 발생시킬 수 있다. 하나의 거친 단위 뮤텍스(Coarse-Grained Mutex)로 대규모 공유 구조를 보호하면 데이터 손상을 방지할 수 있지만, 원래 동시에 실행하려던 콜백까지 직렬화할 수 있다. 세분화된 잠금(Fine-Grained Locking)은 병렬성을 높일 수 있지만 구현 복잡성과 교착상태(Deadlock) 위험도 증가시킨다. 따라서 실행기 아키텍처는 콜백 그룹과 동기화를 별개의 문제로 취급하지 않고 데이터 소유권(Data Ownership)과 함께 설계해야 한다.

블로킹 연산(Blocking Operation)은 특별한 주의가 필요하다. 서비스 응답, 액션 결과, 장치 동작 또는 다른 이벤트를 동기적으로 기다리는 콜백은 오랜 시간 동안 실행기 스레드를 점유할 수 있다. 작업 완료를 위해 필요한 다른 콜백이 콜백 그룹 제한이나 실행기 스레드 부족으로 실행될 수 없다면 시스템에 교착상태가 발생하거나 심각한 지연시간이 나타날 수 있다. 따라서 콜백 기반 아키텍처에서는 비동기 상호작용 패턴(Asynchronous Interaction Pattern)이 더 적합한 경우가 많다.

여러 실행기 스레드가 존재하더라도 교착상태(Deadlock)는 발생할 수 있다. 예를 들어 하나의 콜백이 동기 요청(Synchronous Request)을 수행한 후 동일한 상호 배타적 그룹에 연결된 완료 콜백(Completion Callback)을 기다릴 수 있다. 대기 중인 콜백이 여전히 해당 그룹의 실행 권한을 점유하고 있기 때문에 완료 콜백은 실행될 수 없다. 따라서 콜백 내부에서 동기 서비스 또는 액션 작업을 사용할 때는 콜백 그룹 사이의 관계를 이해하는 것이 필수적이다.

실행기 설계(Executor Design)는 지연시간(Latency), 지터(Jitter), 응답성(Responsiveness)에도 영향을 미친다. 개념적으로 높은 우선순위를 가진 기능을 별도의 콜백 그룹에 배치한다고 해서 운영체제의 실시간 우선순위(Real-Time Priority)가 자동으로 부여되는 것은 아니다. 콜백 그룹은 동시 실행 가능 여부를 제어하는 반면 스레드 스케줄링(Thread Scheduling)은 실행기 구현과 운영체제에 의해 관리된다. 엄격한 실시간 요구사항을 가진 시스템에서는 특수 실행기, 스레드 구성, CPU 친화도(CPU Affinity), 실시간 운영체제 정책이 필요할 수 있다.

장시간 실행되는 계산 작업(Long-Running Computation)을 콜백 내부에 직접 배치하기 전에 신중하게 평가해야 한다. 신경망 추론(Neural-Network Inference), 대규모 포인트 클라우드 처리(Point-Cloud Processing), 최적화(Optimization), 계획(Planning)은 통신 및 제어 콜백을 방해할 정도로 많은 시간을 소비할 수 있다. 이러한 작업은 전용 실행 컨텍스트(Execution Context), 작업 스레드, 별도 프로세스 또는 특수 처리 파이프라인으로 분리하고 ROS 2 콜백은 제한된 데이터 전달과 조정 역할을 수행하도록 설계할 수 있다.

컴포지션(Composition)은 여러 노드가 하나의 프로세스와 잠재적으로 하나의 실행기를 공유할 수 있기 때문에 실행기 설계의 중요성을 더욱 높인다. 컴포즈된 인지 파이프라인은 카메라, 전처리, 추론, 시각화 노드를 포함할 수 있으며 이들의 콜백이 동일한 작업 스레드를 두고 경쟁할 수 있다. 효율적인 프로세스 내부 통신(Intra-Process Communication)이 데이터 전송 오버헤드를 줄일 수 있지만, 잘못된 콜백 스케줄링은 공유 프로세스 내부에서 여전히 지연, 기아 상태(Starvation), 경합을 발생시킬 수 있다.

더 강한 실행 분리(Execution Separation)가 필요한 경우 여러 실행기(Multiple Executors)를 사용할 수 있다. 서로 다른 노드 또는 콜백 그룹을 별도의 실행기 컨텍스트에 할당하여 제어, 인지, 진단, 백그라운드 처리(Background Processing)가 하나의 스케줄링 도메인에서 모두 경쟁하지 않도록 할 수 있다. 이러한 방식은 보다 명확한 자원 경계(Resource Boundary)를 제공할 수 있지만, 추가적인 실행기와 스레드는 시스템 복잡성을 증가시키며 공유 자원과의 조정도 여전히 필요하다.

콜백 실행을 조정할 때는 관측 가능성(Observability)이 필수적이다. 엔지니어는 실제 작업부하에서 콜백 실행시간, 스케줄링 지연(Scheduling Delay), 실행 빈도, 대기열 동작(Queueing Behavior), CPU 사용률, 종단간 지연시간(End-to-End Latency)을 측정해야 한다. 추적 도구(Tracing Tool)를 사용하면 다른 콜백이 실행 중이어서 지연되는지, 사용 가능한 실행기 스레드가 없어서 지연되는지, 또는 동기화로 인해 진행이 차단되는지 확인할 수 있다. 실행기 구성은 이론적인 예상만으로 선택하지 말고 실제 측정을 통해 검증해야 한다.

자율이동로봇(Autonomous Mobile Robot, AMR)에서는 모션 제어 타이머(Motion-Control Timer)를 계산량이 많은 LiDAR 또는 카메라 처리와 격리하면서 진단 및 플릿 통신(Fleet Communication)이 독립적으로 진행되도록 콜백 아키텍처를 구성할 수 있다. 매니퓰레이터(Manipulator)는 고주기 상태 갱신을 계획 서비스(Planning Service)와 분리할 수 있으며, 피지컬 AI 시스템(Physical AI System)은 GPU 추론 조정(GPU Inference Coordination)을 안전 모니터링(Safety Monitoring)과 격리할 수 있다. 정확한 그룹 구성은 시간적 의존성, 공유 상태, 계산 부하, 장애 결과를 반영해야 한다.

따라서 강건한 ROS 2 실행 아키텍처(Robust ROS 2 Execution Architecture)는 의도적인 콜백 그룹 할당, 적절한 실행기 선택, 명시적인 데이터 소유권, 제한된 콜백 동작(Bounded Callback Behavior), 신중한 동기화, 성능 측정을 결합한다. 단일 스레드 실행은 단순성을 제공하고, 다중 스레드 실행은 소프트웨어가 이를 지원하도록 설계되었을 때 제어된 동시성을 제공한다. 콜백 그룹과 실행기는 함께 콜백 스케줄링을 확장 가능한 로봇 및 피지컬 AI 시스템을 위한 명시적인 아키텍처 설계 요소로 전환한다.

##  

## 03.06 Multi-Thread Executor and Callback Concurrency [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

The ROS 2 MultiThreadedExecutor enables multiple eligible callbacks to execute concurrently through a pool of worker threads. It is designed for nodes and composed systems in which subscriptions, timers, services, actions, and other callback-driven operations may need to progress simultaneously. Unlike single-threaded execution, it introduces controlled parallel execution, but actual concurrency depends on callback groups, available work, synchronization, and application design.

The executor continuously identifies executable entities whose conditions are ready, such as a subscription with available data or a timer whose period has expired. Ready work is assigned to available worker threads according to executor scheduling behavior and callback-group constraints. The number of threads therefore defines potential execution capacity, while the callback architecture determines whether that capacity can actually be used.

A MultiThreadedExecutor does not automatically execute every callback in parallel. If callbacks belong to the same MutuallyExclusive callback group, only one of them may execute at a time even when several worker threads are idle. Conversely, callbacks assigned to compatible groups or a Reentrant group can execute simultaneously when resources are available. Callback-group design is therefore a fundamental part of multi-threaded execution.

MutuallyExclusive groups are useful when callbacks access common state that should be processed sequentially. For example, a localization update and a control-state modification may share data structures whose consistency is easier to guarantee through serialized execution. Placing these callbacks in one mutually exclusive group creates a logical critical execution region without requiring every relationship to be expressed through explicit application-level locks.

Reentrant callback groups permit concurrent callback execution and are appropriate for operations that are thread-safe by design. Multiple sensor-processing requests, independent service operations, or stateless transformations may benefit from this model. Reentrancy must be a property of the implementation rather than merely a configuration choice, because simultaneous invocation can corrupt internal state when an algorithm assumes exclusive access.

Concurrency and parallelism should also be distinguished. Concurrency means that multiple callback operations can make progress during overlapping periods, while true parallelism requires execution on multiple CPU cores or other processing resources at the same time. A MultiThreadedExecutor creates the opportunity for both, but operating-system scheduling, processor availability, callback behavior, and synchronization determine the actual execution pattern.

Thread count is an important design parameter but should not simply be maximized. Too few worker threads can leave independent callbacks waiting unnecessarily, while excessive threads may increase context switching, cache interference, memory pressure, and synchronization overhead. The appropriate number depends on available CPU cores, callback workload, blocking behavior, computational intensity, and whether other processes require guaranteed computing resources.

CPU-bound and I/O-bound callbacks behave differently under multi-threaded execution. CPU-intensive perception, planning, or numerical processing can consume entire cores for significant periods, whereas callbacks waiting for device communication or external services may leave processing resources underutilized. Understanding this distinction helps determine whether additional executor threads improve throughput or merely create more scheduling competition.

Shared mutable data is the primary source of concurrency hazards. Two callbacks may read and modify robot state, maps, command buffers, configuration objects, or algorithm state simultaneously. Without appropriate protection, operations can interleave unpredictably and create data races, corrupted values, inconsistent decisions, or intermittent failures that are difficult to reproduce. Multi-threaded executor design must therefore include an explicit shared-state strategy.

Mutexes provide a common mechanism for protecting shared resources, but their scope and duration directly influence performance. Holding a mutex during expensive computation or blocking communication can prevent other callbacks from progressing and eliminate the benefit of concurrency. Critical sections should therefore be kept as short as practical, and expensive work should preferably operate on local or immutable snapshots when the application architecture permits.

Deadlocks arise when callbacks or threads wait indefinitely for resources or events that cannot become available. Multiple mutexes acquired in inconsistent order are a traditional source of deadlock, but ROS 2 callback interactions can create additional cases. A callback that performs a synchronous request and waits for another callback whose execution is prevented by group restrictions can block the system even when a MultiThreadedExecutor is being used.

Asynchronous communication is often better suited to concurrent callback architectures. Instead of occupying a worker thread while waiting synchronously for a service or action result, a node can initiate the request and process completion through another callback. This approach allows executor threads to continue handling other ready work, although callback groups and object lifetimes must still be designed so that completion processing can execute safely.

Long-running callbacks can create scheduling latency even when no explicit synchronization problem exists. If all executor threads become occupied by expensive perception or planning tasks, short control, diagnostic, or communication callbacks may wait for a worker to become available. Multi-threading reduces some blocking effects but does not guarantee bounded response time. Computational workloads must therefore be separated according to their timing requirements.

Callback starvation is another possible concern. Frequently ready or computationally expensive callbacks may consume execution opportunities while lower-frequency work experiences excessive delay. The standard executor should not be interpreted as a general real-time priority scheduler. Systems requiring strict temporal guarantees may need dedicated execution contexts, specialized executors, operating-system scheduling policies, or separation of critical functions into independent processes.

Multi-threaded execution becomes especially important in composed ROS 2 systems. Several nodes loaded into one component container may share an executor while using efficient intra-process communication. Camera acquisition, preprocessing, inference, localization, and visualization can then execute with lower communication overhead, but they also compete for shared executor threads. Composition and executor configuration must consequently be designed as one integrated performance problem.

A useful architecture can separate callback groups according to timing and data dependencies. High-rate control callbacks may occupy a mutually exclusive group, sensor ingestion may use another group, and independent diagnostics may use a reentrant or separate group. Computationally expensive AI inference can be isolated so that its execution does not unnecessarily serialize lightweight callbacks. The exact arrangement should reflect measured workload rather than generic grouping rules.

Multiple executors may be preferable when callback groups alone do not provide sufficient isolation. A robot can place time-sensitive control nodes on one executor and computational perception nodes on another, potentially assigning their threads to different processor resources. This creates stronger scheduling boundaries, although communication, shared-memory access, and operating-system scheduling must still be considered when evaluating end-to-end behavior.

CPU affinity can further control where executor threads run by restricting selected threads or processes to particular processor cores. In systems with demanding control and perception workloads, affinity can reduce interference and improve repeatability by preventing unrelated computation from migrating across critical cores. Affinity alone does not guarantee real-time behavior, but it can become part of a broader deterministic execution strategy.

Priority management requires similar caution. ROS 2 callback groups do not inherently assign operating-system thread priorities to individual callbacks. If safety monitoring or motion control requires stronger scheduling guarantees than background perception, the architecture may need separate threads, executors, or processes configured with appropriate operating-system policies. Logical callback importance and actual CPU scheduling priority are different concepts.

Tracing and profiling are essential for understanding real callback concurrency. Measurements should capture callback start and completion times, scheduling delay, executor utilization, thread activity, synchronization waits, and end-to-end message latency. These observations reveal whether additional threads improve throughput, whether mutex contention dominates execution, or whether computational callbacks are delaying time-sensitive work.

Testing should include worst-case concurrent workloads rather than only isolated node operation. A system may behave correctly when perception, control, services, and diagnostics are tested separately but fail when all callbacks become active simultaneously. Stress tests should exercise high sensor rates, service requests, parameter updates, planning activity, and fault conditions together to expose races, starvation, excessive jitter, or hidden blocking dependencies.

In an AMR, a MultiThreadedExecutor can allow LiDAR processing, camera ingestion, localization updates, control timers, and diagnostics to progress concurrently while callback groups protect critical shared state. Manipulators can separate high-rate joint-state processing from planning and monitoring. Physical AI platforms can coordinate GPU inference, sensor pipelines, world-model updates, and supervisory functions without forcing every activity through a single sequential callback path.

A robust MultiThreadedExecutor architecture therefore requires more than selecting a larger thread count. It combines intentional callback grouping, thread-safe algorithms, controlled data ownership, minimal blocking, careful synchronization, workload isolation, tracing, and realistic stress testing. When these elements are designed together, callback concurrency can improve responsiveness and resource utilization while preserving predictable behavior in scalable ROS 2 robotic and Physical AI systems.

ROS 2 다중 스레드 실행기(MultiThreadedExecutor)는 여러 작업 스레드(Worker Thread)를 사용하여 실행 가능한 다수의 콜백(Callback)을 동시에 처리할 수 있도록 한다. 이는 구독(Subscription), 타이머(Timer), 서비스(Service), 액션(Action) 및 기타 콜백 기반 작업이 동시에 진행되어야 하는 노드와 컴포즈 시스템(Composed System)을 위해 설계된다. 단일 스레드 실행과 달리 제어된 병렬 실행을 제공하지만, 실제 동시성은 콜백 그룹(Callback Group), 준비된 작업, 동기화(Synchronization), 응용 프로그램 설계에 따라 결정된다.

실행기(Executor)는 구독 데이터가 도착하거나 타이머 주기가 만료되는 것처럼 실행 조건이 충족된 엔티티(Entity)를 지속적으로 식별한다. 준비된 작업은 실행기의 스케줄링 동작과 콜백 그룹 제약조건에 따라 사용 가능한 작업 스레드에 할당된다. 따라서 스레드 수는 잠재적인 실행 용량을 정의하지만, 실제로 그 용량을 활용할 수 있는지는 콜백 아키텍처(Callback Architecture)에 의해 결정된다.

다중 스레드 실행기(MultiThreadedExecutor)가 모든 콜백을 자동으로 병렬 실행하는 것은 아니다. 콜백이 동일한 상호 배타적 콜백 그룹(MutuallyExclusive Callback Group)에 속하면 여러 작업 스레드가 유휴 상태이더라도 한 번에 하나의 콜백만 실행할 수 있다. 반대로 호환 가능한 서로 다른 그룹이나 재진입 가능 그룹(Reentrant Group)에 할당된 콜백은 자원을 사용할 수 있을 때 동시에 실행될 수 있다. 따라서 콜백 그룹 설계는 다중 스레드 실행의 핵심 요소이다.

상호 배타적 그룹(MutuallyExclusive Group)은 순차적으로 처리되어야 하는 공통 상태에 여러 콜백이 접근하는 경우 유용하다. 예를 들어 위치추정 갱신(Localization Update)과 제어 상태 수정(Control-State Modification)이 데이터 구조를 공유한다면 직렬 실행을 통해 데이터 일관성을 보다 쉽게 보장할 수 있다. 이러한 콜백을 하나의 상호 배타적 그룹에 배치하면 모든 관계를 명시적인 응용 프로그램 수준 잠금으로 표현하지 않고도 논리적인 임계 실행 영역(Critical Execution Region)을 구성할 수 있다.

재진입 가능 콜백 그룹(Reentrant Callback Group)은 동시 콜백 실행을 허용하며 설계 자체가 스레드 안전(Thread-Safe)한 작업에 적합하다. 여러 센서 처리 요청, 독립적인 서비스 작업 또는 상태를 보유하지 않는 변환(Stateless Transformation)은 이 모델의 이점을 얻을 수 있다. 재진입성(Reentrancy)은 단순한 구성 선택이 아니라 구현 자체의 특성이어야 하며, 알고리즘이 배타적 접근을 가정하는 경우 동시 호출로 내부 상태가 손상될 수 있다.

동시성(Concurrency)과 병렬성(Parallelism)도 구분해야 한다. 동시성은 여러 콜백 작업이 서로 중첩되는 시간 동안 진행될 수 있음을 의미하는 반면, 진정한 병렬성은 여러 CPU 코어나 다른 처리 자원에서 실제로 동시에 실행되는 것을 요구한다. 다중 스레드 실행기는 두 가지 모두가 가능한 환경을 제공하지만 실제 실행 형태는 운영체제 스케줄링, 프로세서 가용성, 콜백 동작, 동기화에 의해 결정된다.

스레드 수(Thread Count)는 중요한 설계 매개변수이지만 단순히 최대화해서는 안 된다. 작업 스레드가 너무 적으면 독립적인 콜백이 불필요하게 대기할 수 있으며, 지나치게 많으면 컨텍스트 스위칭(Context Switching), 캐시 간섭(Cache Interference), 메모리 부담, 동기화 오버헤드가 증가할 수 있다. 적절한 스레드 수는 사용 가능한 CPU 코어, 콜백 작업부하, 블로킹 동작, 계산 집약도, 다른 프로세스가 요구하는 계산 자원에 따라 결정해야 한다.

CPU 중심 콜백(CPU-Bound Callback)과 입출력 중심 콜백(I/O-Bound Callback)은 다중 스레드 실행 환경에서 서로 다르게 동작한다. 계산 집약적인 인지(Perception), 계획(Planning), 수치 처리(Numerical Processing)는 상당한 시간 동안 CPU 코어 전체를 사용할 수 있지만 장치 통신이나 외부 서비스를 기다리는 콜백은 처리 자원을 충분히 활용하지 못할 수 있다. 이러한 차이를 이해하면 추가 실행기 스레드가 처리량을 높이는지 아니면 단순히 스케줄링 경쟁만 증가시키는지 판단할 수 있다.

공유 변경 가능 데이터(Shared Mutable Data)는 동시성 위험의 주요 원인이다. 두 콜백이 로봇 상태, 지도, 명령 버퍼(Command Buffer), 구성 객체(Configuration Object), 알고리즘 상태를 동시에 읽거나 수정할 수 있다. 적절한 보호가 없다면 연산 순서가 예측할 수 없게 뒤섞여 데이터 경쟁(Data Race), 값 손상, 일관되지 않은 판단 또는 재현하기 어려운 간헐적 장애가 발생할 수 있다. 따라서 다중 스레드 실행기 설계에는 명시적인 공유 상태 관리 전략이 포함되어야 한다.

뮤텍스(Mutex)는 공유 자원을 보호하는 일반적인 방법이지만 적용 범위와 유지 시간이 성능에 직접적인 영향을 미친다. 고비용 계산이나 블로킹 통신을 수행하는 동안 뮤텍스를 유지하면 다른 콜백의 진행을 막아 동시성의 장점을 제거할 수 있다. 따라서 임계 구역(Critical Section)은 가능한 짧게 유지해야 하며, 응용 프로그램 아키텍처가 허용한다면 고비용 작업은 로컬 데이터 또는 불변 스냅샷(Immutable Snapshot)을 기반으로 수행하는 것이 바람직하다.

교착상태(Deadlock)는 콜백이나 스레드가 사용 가능해질 수 없는 자원 또는 이벤트를 무기한 기다릴 때 발생한다. 여러 뮤텍스를 일관되지 않은 순서로 획득하는 것은 전통적인 교착상태 원인이지만 ROS 2의 콜백 상호작용에서도 추가적인 문제가 발생할 수 있다. 콜백이 동기 요청(Synchronous Request)을 수행한 후 콜백 그룹 제한 때문에 실행될 수 없는 다른 콜백을 기다리면 다중 스레드 실행기를 사용하더라도 시스템이 정지할 수 있다.

비동기 통신(Asynchronous Communication)은 동시 콜백 아키텍처에 더 적합한 경우가 많다. 서비스 또는 액션 결과를 동기적으로 기다리면서 작업 스레드를 점유하는 대신 노드는 요청을 시작하고 이후 다른 콜백을 통해 완료 결과를 처리할 수 있다. 이러한 방식은 실행기 스레드가 다른 준비된 작업을 계속 처리하도록 하지만, 완료 처리 과정이 안전하게 실행될 수 있도록 콜백 그룹과 객체 수명(Object Lifetime)을 함께 설계해야 한다.

명시적인 동기화 문제가 없더라도 장시간 실행되는 콜백(Long-Running Callback)은 스케줄링 지연(Scheduling Latency)을 발생시킬 수 있다. 모든 실행기 스레드가 고비용 인지 또는 계획 작업에 사용되면 짧은 제어, 진단, 통신 콜백도 사용 가능한 작업 스레드가 생길 때까지 기다려야 한다. 다중 스레딩은 일부 블로킹 효과를 줄이지만 제한된 응답시간(Bounded Response Time)을 보장하지 않으므로 계산 작업은 시간 요구조건에 따라 분리해야 한다.

콜백 기아 상태(Callback Starvation)도 고려해야 할 문제이다. 빈번하게 준비 상태가 되거나 계산량이 많은 콜백이 실행 기회를 지속적으로 소비하면 낮은 빈도의 작업이 과도하게 지연될 수 있다. 표준 실행기(Standard Executor)를 일반적인 실시간 우선순위 스케줄러(Real-Time Priority Scheduler)로 해석해서는 안 된다. 엄격한 시간 보장이 필요한 시스템은 전용 실행 컨텍스트, 특수 실행기, 운영체제 스케줄링 정책 또는 중요 기능의 독립 프로세스 분리가 필요할 수 있다.

다중 스레드 실행은 컴포즈된 ROS 2 시스템(Composed ROS 2 System)에서 특히 중요하다. 하나의 컴포넌트 컨테이너(Component Container)에 로드된 여러 노드는 효율적인 프로세스 내부 통신(Intra-Process Communication)을 사용하면서 하나의 실행기를 공유할 수 있다. 카메라 획득, 전처리, 추론, 위치추정, 시각화가 낮은 통신 오버헤드로 실행될 수 있지만 동시에 공유 실행기 스레드를 두고 경쟁하므로 컴포지션(Composition)과 실행기 구성은 하나의 통합된 성능 문제로 설계해야 한다.

유용한 아키텍처에서는 시간적 요구사항과 데이터 의존성에 따라 콜백 그룹을 분리할 수 있다. 고주기 제어 콜백은 하나의 상호 배타적 그룹에 배치하고, 센서 입력은 다른 그룹을 사용하며, 독립적인 진단 작업은 재진입 가능 그룹 또는 별도의 그룹에 배치할 수 있다. 계산량이 많은 AI 추론(AI Inference)은 가벼운 콜백을 불필요하게 직렬화하지 않도록 격리할 수 있다. 구체적인 구성은 일반적인 규칙보다 실제 측정된 작업부하를 반영해야 한다.

콜백 그룹만으로 충분한 격리를 제공할 수 없다면 다중 실행기(Multiple Executors)를 사용하는 것이 더 적절할 수 있다. 로봇은 시간에 민감한 제어 노드를 하나의 실행기에 배치하고 계산 집약적인 인지 노드를 다른 실행기에 배치하여 각각의 스레드를 서로 다른 프로세서 자원에 할당할 수 있다. 이는 더 강한 스케줄링 경계(Scheduling Boundary)를 제공하지만 종단간 동작을 평가할 때 통신, 공유 메모리 접근, 운영체제 스케줄링을 함께 고려해야 한다.

CPU 친화도(CPU Affinity)를 사용하면 선택된 스레드 또는 프로세스가 특정 프로세서 코어에서 실행되도록 제한하여 실행 위치를 더욱 세밀하게 제어할 수 있다. 높은 제어 및 인지 작업부하를 가진 시스템에서는 관련 없는 계산이 중요한 코어 사이를 이동하는 것을 방지하여 간섭을 줄이고 반복성을 향상시킬 수 있다. CPU 친화도만으로 실시간 동작을 보장할 수는 없지만 보다 광범위한 결정론적 실행 전략(Deterministic Execution Strategy)의 일부가 될 수 있다.

우선순위 관리(Priority Management)에도 동일한 주의가 필요하다. ROS 2 콜백 그룹은 개별 콜백에 운영체제 수준의 스레드 우선순위를 본질적으로 부여하지 않는다. 안전 모니터링(Safety Monitoring)이나 모션 제어(Motion Control)가 백그라운드 인지 작업보다 강력한 스케줄링 보장을 요구한다면 적절한 운영체제 정책으로 구성된 별도의 스레드, 실행기 또는 프로세스가 필요할 수 있다. 논리적인 콜백 중요도와 실제 CPU 스케줄링 우선순위는 서로 다른 개념이다.

추적(Tracing)과 프로파일링(Profiling)은 실제 콜백 동시성을 이해하는 데 필수적이다. 측정 항목에는 콜백 시작 및 완료 시간, 스케줄링 지연, 실행기 활용률, 스레드 활동, 동기화 대기시간, 종단간 메시지 지연시간이 포함되어야 한다. 이러한 관측을 통해 추가 스레드가 실제 처리량을 향상시키는지, 뮤텍스 경합(Mutex Contention)이 실행 성능을 지배하는지, 계산 집약적인 콜백이 시간에 민감한 작업을 지연시키는지 확인할 수 있다.

테스트(Testing)는 개별 노드 동작만이 아니라 최악 조건의 동시 작업부하(Worst-Case Concurrent Workload)를 포함해야 한다. 인지, 제어, 서비스, 진단을 개별적으로 테스트할 때는 정상적으로 동작하더라도 모든 콜백이 동시에 활성화되면 시스템이 실패할 수 있다. 스트레스 테스트(Stress Test)는 높은 센서 데이터 속도, 서비스 요청, 매개변수 갱신, 계획 작업, 장애 조건을 동시에 발생시켜 경쟁 상태, 기아 상태, 과도한 지터 또는 숨겨진 블로킹 의존성을 확인해야 한다.

자율이동로봇(Autonomous Mobile Robot, AMR)에서는 다중 스레드 실행기를 통해 LiDAR 처리, 카메라 입력, 위치추정 갱신, 제어 타이머, 진단 작업을 동시에 진행하면서 콜백 그룹을 사용하여 중요한 공유 상태를 보호할 수 있다. 매니퓰레이터(Manipulator)는 고주기 관절 상태 처리(Joint-State Processing)를 계획 및 모니터링 작업과 분리할 수 있다. 피지컬 AI 플랫폼(Physical AI Platform)은 모든 활동을 하나의 순차적인 콜백 경로로 제한하지 않고 GPU 추론, 센서 파이프라인, 월드 모델 갱신(World-Model Update), 감독 기능(Supervisory Function)을 조정할 수 있다.

따라서 강건한 다중 스레드 실행기 아키텍처(Robust MultiThreadedExecutor Architecture)는 단순히 더 많은 스레드 수를 선택하는 것 이상을 요구한다. 의도적인 콜백 그룹 구성, 스레드 안전 알고리즘(Thread-Safe Algorithm), 제어된 데이터 소유권(Data Ownership), 최소화된 블로킹, 신중한 동기화, 작업부하 격리(Workload Isolation), 추적, 현실적인 스트레스 테스트를 함께 결합해야 한다. 이러한 요소를 통합적으로 설계하면 콜백 동시성은 확장 가능한 ROS 2 로봇 및 피지컬 AI 시스템에서 예측 가능한 동작을 유지하면서 응답성과 자원 활용도를 향상시킬 수 있다.

##  

## 03.07 Node State Machine-Based Behavior Management [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

State-machine-based behavior management organizes robot behavior as a set of explicit states, transitions, events, and transition conditions rather than distributing behavioral decisions across loosely connected callbacks. In a ROS 2 node, the state machine can coordinate perception, planning, control, communication, and recovery activities according to the robot's current operational context. This makes behavior flow visible, testable, and easier to reason about as system complexity grows.

A state represents a meaningful behavioral condition in which the node performs a defined set of activities. An AMR might use Idle, Navigate, Dock, Charge, Recover, and Emergency states, while a manipulator could use Standby, Approach, Grasp, Transfer, and Release. Each state should describe what the node is permitted to do, what inputs it processes, what outputs it produces, and which events can cause it to leave that state.

Transitions define the permitted movement between states. Instead of allowing arbitrary behavioral changes from any callback, the state machine specifies valid source and destination relationships. A navigation robot may transition from Idle to Navigate after receiving a valid mission, from Navigate to Dock after reaching a target region, or from any operational state to Emergency when a safety event occurs. Explicit transitions prevent undefined behavioral sequences.

Events provide the stimuli that request or trigger transitions. They may originate from ROS 2 subscriptions, timers, services, actions, sensor conditions, internal algorithms, or supervisory commands. Examples include mission_received, goal_reached, obstacle_blocked, battery_low, docking_complete, timeout, and emergency_stop. Converting heterogeneous inputs into clearly defined behavioral events separates communication mechanisms from higher-level decision logic.

A transition usually requires more than the presence of an event. Guard conditions can determine whether the requested transition is currently valid by checking robot state, sensor quality, resource availability, mission context, or safety constraints. A Navigate transition might require localization confidence above a threshold and an available route, while charging behavior may require confirmed docking alignment before electrical power transfer is enabled.

States can define entry, execution, and exit behavior. Entry logic performs initialization required when entering a state, such as resetting counters, configuring planners, or starting a device. Execution logic performs the ongoing behavior associated with the state, while exit logic releases resources or terminates activities before transition. Separating these responsibilities reduces duplicated initialization and cleanup code across behavioral paths.

State-machine execution should remain coordinated with ROS 2 callback processing. Subscriptions and services may generate events asynchronously, but state transitions should occur through a controlled mechanism rather than allowing multiple callbacks to modify behavioral state independently. Event queues, protected transition functions, serialized callback groups, or dedicated behavior-management callbacks can prevent race conditions when several events arrive nearly simultaneously.

Concurrency requires special attention when the node uses a MultiThreadedExecutor. A sensor callback may report an obstacle while a mission callback requests a new goal and a timer detects a timeout. If each callback can directly change the active state, the resulting transition may depend on thread timing. Centralizing transition decisions allows the state machine to define deterministic precedence and synchronization rules for competing events.

Event priority becomes important in safety-related robotic behavior. An emergency-stop event should normally override routine mission progression, while a low-battery event may take precedence over accepting a new noncritical mission. Priority does not need to be implemented as a single global numerical ranking, but the architecture should clearly define how conflicting events are resolved so behavior remains understandable under abnormal conditions.

Hierarchical state machines can reduce complexity when a robot contains many related behaviors. A high-level Operational state may contain Navigate, Inspect, and Dock substates, while a separate Fault state contains Recoverable and Critical conditions. Shared behavior can then be defined at the parent level instead of repeated in every child state. Hierarchy helps preserve conceptual structure as mission logic expands.

History and contextual information may also influence behavior without becoming states themselves. Mission identifiers, retry counts, navigation goals, battery measurements, fault codes, or perception results are better represented as state-machine context when they describe data rather than distinct modes of operation. Excessive conversion of every variable into a state can create a combinatorial state explosion and make the behavior model difficult to maintain.

Behavioral state machines should be distinguished from ROS 2 managed-node lifecycle states. Lifecycle states such as Unconfigured, Inactive, and Active describe the operational readiness and resource-management condition of a node. Behavioral states such as Navigate, Dock, Inspect, or Recover describe what the robot is doing while the node is operational. A node can therefore be lifecycle Active while its internal behavior state changes repeatedly during a mission.

The state machine can integrate naturally with ROS 2 actions for long-running behaviors. Entering a Navigate state may initiate a navigation action, while action feedback updates contextual information and the final result generates goal_reached or navigation_failed events. Cancellation can be issued when the state machine transitions away from Navigate. This separates action transport and feedback handling from the behavioral decision about what should happen next.

Services and topics serve complementary roles in this architecture. Topics are suitable for continuous observations and asynchronous events, while services can expose explicit management operations such as requesting a reset or querying state. The state machine should avoid becoming dependent on transport-specific details. Communication callbacks ideally translate ROS 2 inputs into internal events and translate behavioral outputs into appropriate ROS 2 messages or commands.

Failure and recovery behavior should be modeled explicitly rather than embedded as scattered exception paths. A Recover state can stop unsafe motion, inspect fault information, clear temporary conditions, retry limited operations, or request supervisory assistance. Recovery attempts should have defined limits and exit criteria so repeated failures cannot create endless loops. Unrecoverable conditions should transition toward a safe fault or emergency behavior.

Timeouts are particularly useful for detecting behaviors that fail to make progress. A docking state may require completion within a defined interval, while a navigation state can monitor progress rather than waiting indefinitely for a result. Timer-generated events allow the state machine to distinguish normal execution from stalled behavior and select recovery actions. Timeout values should reflect physical system dynamics and measured operational performance.

Observability improves significantly when the current behavioral state and transition history are exposed through ROS 2 diagnostics, topics, logs, or tracing. Operators can determine not only that a robot stopped but whether it was waiting, recovering, docking, or responding to a safety condition. Recording transition timestamps, triggering events, source states, destination states, and failure reasons provides valuable evidence for debugging and fleet-level analysis.

Testing can be structured around states and transitions instead of requiring complete physical missions for every scenario. Unit tests can inject events and verify expected transitions, guard behavior, entry actions, and invalid-transition rejection. Integration tests can connect actual ROS 2 communication interfaces and confirm that sensor messages, action results, timeouts, and service requests generate the intended behavioral events and resulting outputs.

Determinism is improved when identical state, context, and event conditions produce the same transition decision. External sensor timing can still be nondeterministic, but the decision logic itself should avoid hidden dependencies on callback ordering wherever possible. Explicit event handling, transition tables, guard conditions, and controlled shared-state access make behavior reproducible enough for systematic debugging and safety validation.

An AMR can use this architecture to coordinate mission acceptance, autonomous navigation, obstacle handling, docking, charging, and recovery. A manipulator can manage approach, grasp, verification, transfer, and release, while a UAV can represent takeoff, cruise, mission execution, return, landing, and emergency behavior. Physical AI systems can similarly coordinate perception-driven decisions while preserving deterministic supervisory boundaries around learned components.

A robust state-machine-based ROS 2 behavior architecture therefore combines explicit states, controlled transitions, well-defined events, guard conditions, contextual data, failure handling, concurrency control, observability, and systematic testing. The state machine becomes the behavioral coordination layer rather than a replacement for perception, planning, or control algorithms. This separation allows increasingly intelligent robot functions to operate within understandable and governable execution logic.

상태 머신 기반 행동 관리(State-Machine-Based Behavior Management)는 행동 결정을 서로 느슨하게 연결된 콜백(Callback)에 분산시키는 대신, 로봇의 행동을 명시적인 상태(State), 전이(Transition), 이벤트(Event), 전이 조건(Transition Condition)의 집합으로 구성한다. ROS 2 노드에서 상태 머신(State Machine)은 로봇의 현재 운용 상황에 따라 인지(Perception), 계획(Planning), 제어(Control), 통신(Communication), 복구(Recovery) 활동을 조정할 수 있다. 이를 통해 시스템 복잡성이 증가하더라도 행동 흐름을 명확하게 파악하고 테스트하며 논리적으로 분석하기 쉬워진다.

상태(State)는 노드가 정의된 일련의 활동을 수행하는 의미 있는 행동 조건을 나타낸다. 자율이동로봇(Autonomous Mobile Robot, AMR)은 대기(Idle), 주행(Navigate), 도킹(Dock), 충전(Charge), 복구(Recover), 비상(Emergency) 상태를 사용할 수 있으며, 매니퓰레이터(Manipulator)는 대기(Standby), 접근(Approach), 파지(Grasp), 이송(Transfer), 해제(Release) 상태를 사용할 수 있다. 각 상태는 노드가 수행할 수 있는 작업, 처리하는 입력, 생성하는 출력, 해당 상태를 벗어나게 하는 이벤트를 정의해야 한다.

전이(Transition)는 상태 사이에서 허용되는 이동을 정의한다. 모든 콜백에서 임의로 행동을 변경하도록 허용하는 대신 상태 머신은 유효한 출발 상태(Source State)와 목적 상태(Destination State)의 관계를 명시한다. 내비게이션 로봇은 유효한 임무를 수신하면 대기(Idle)에서 주행(Navigate)으로, 목표 영역에 도달하면 주행에서 도킹(Dock)으로 전이할 수 있으며, 안전 이벤트가 발생하면 모든 운용 상태에서 비상(Emergency)으로 전이할 수 있다. 명시적인 전이는 정의되지 않은 행동 순서를 방지한다.

이벤트(Event)는 상태 전이를 요청하거나 발생시키는 자극(Stimulus)을 제공한다. 이벤트는 ROS 2 구독(Subscription), 타이머(Timer), 서비스(Service), 액션(Action), 센서 조건, 내부 알고리즘 또는 상위 감독 명령(Supervisory Command)에서 발생할 수 있다. 예를 들어 임무 수신(mission_received), 목표 도달(goal_reached), 장애물 차단(obstacle_blocked), 배터리 부족(battery_low), 도킹 완료(docking_complete), 시간 초과(timeout), 비상 정지(emergency_stop) 등이 있다. 서로 다른 입력을 명확하게 정의된 행동 이벤트로 변환하면 통신 메커니즘과 상위 수준 의사결정 로직을 분리할 수 있다.

일반적으로 상태 전이에는 이벤트의 발생만으로 충분하지 않다. 가드 조건(Guard Condition)은 로봇 상태, 센서 품질, 자원 가용성, 임무 상황 또는 안전 제약조건을 검사하여 요청된 전이가 현재 유효한지 결정할 수 있다. 예를 들어 주행(Navigate) 상태로의 전이는 위치추정 신뢰도(Localization Confidence)가 임계값 이상이고 사용 가능한 경로가 존재해야 할 수 있으며, 충전 행동은 전력 전달을 활성화하기 전에 도킹 정렬(Docking Alignment)이 확인되어야 할 수 있다.

상태는 진입(Entry), 실행(Execution), 종료(Exit) 동작을 정의할 수 있다. 진입 로직은 카운터 초기화, 플래너(Planner) 구성 또는 장치 시작과 같이 해당 상태에 진입할 때 필요한 초기화를 수행한다. 실행 로직은 상태와 연관된 지속적인 행동을 수행하며, 종료 로직은 전이 전에 자원을 해제하거나 진행 중인 활동을 종료한다. 이러한 책임 분리는 여러 행동 경로에서 초기화 및 정리 코드를 반복적으로 구현하는 문제를 줄여준다.

상태 머신 실행은 ROS 2 콜백 처리와 조정되어야 한다. 구독과 서비스는 비동기적으로 이벤트를 생성할 수 있지만, 여러 콜백이 독립적으로 행동 상태를 변경하도록 허용하기보다는 제어된 메커니즘을 통해 상태 전이가 수행되어야 한다. 이벤트 큐(Event Queue), 보호된 전이 함수(Protected Transition Function), 직렬화된 콜백 그룹(Serialized Callback Group), 전용 행동 관리 콜백 등을 사용하면 여러 이벤트가 거의 동시에 도착할 때 발생할 수 있는 경쟁 상태(Race Condition)를 방지할 수 있다.

노드가 다중 스레드 실행기(MultiThreadedExecutor)를 사용할 때는 동시성(Concurrency)에 특별한 주의가 필요하다. 센서 콜백이 장애물을 보고하는 동시에 임무 콜백이 새로운 목표를 요청하고 타이머가 시간 초과를 감지할 수 있다. 각각의 콜백이 활성 상태를 직접 변경할 수 있다면 최종 전이는 스레드 실행 시점에 따라 달라질 수 있다. 전이 결정을 중앙화하면 상태 머신이 서로 경쟁하는 이벤트에 대해 결정론적 우선순위와 동기화 규칙을 정의할 수 있다.

안전과 관련된 로봇 행동에서는 이벤트 우선순위(Event Priority)가 중요해진다. 비상 정지 이벤트(Emergency-Stop Event)는 일반적인 임무 진행보다 우선해야 하며, 배터리 부족 이벤트는 중요하지 않은 새로운 임무를 수락하는 것보다 우선할 수 있다. 우선순위를 반드시 하나의 전역 숫자 순위로 구현할 필요는 없지만, 비정상적인 상황에서도 행동을 이해할 수 있도록 충돌하는 이벤트를 어떻게 해결할 것인지 아키텍처에서 명확하게 정의해야 한다.

계층형 상태 머신(Hierarchical State Machine)은 로봇에 서로 연관된 행동이 많을 때 복잡성을 줄일 수 있다. 상위 수준의 운용(Operational) 상태는 주행(Navigate), 검사(Inspect), 도킹(Dock) 하위 상태(Substate)를 포함할 수 있으며, 별도의 고장(Fault) 상태는 복구 가능(Recoverable) 및 치명적(Critical) 조건을 포함할 수 있다. 공통 행동을 모든 하위 상태에서 반복하지 않고 부모 상태(Parent State) 수준에서 정의할 수 있으므로 임무 로직이 확장되더라도 개념적 구조를 유지할 수 있다.

이력(History)과 상황 정보(Contextual Information)는 그 자체가 상태가 되지 않으면서 행동에 영향을 줄 수 있다. 임무 식별자, 재시도 횟수, 내비게이션 목표, 배터리 측정값, 고장 코드, 인지 결과는 별개의 운용 모드가 아니라 데이터를 설명하므로 상태 머신 컨텍스트(State-Machine Context)로 표현하는 것이 적절하다. 모든 변수를 상태로 변환하면 조합적 상태 폭발(Combinatorial State Explosion)이 발생하여 행동 모델을 유지하기 어려워질 수 있다.

행동 상태 머신(Behavioral State Machine)은 ROS 2 관리 노드 수명주기 상태(Managed-Node Lifecycle State)와 구분해야 한다. 미구성(Unconfigured), 비활성(Inactive), 활성(Active)과 같은 수명주기 상태는 노드의 운용 준비도와 자원 관리 상태를 나타낸다. 반면 주행(Navigate), 도킹(Dock), 검사(Inspect), 복구(Recover)와 같은 행동 상태는 노드가 운용 중일 때 로봇이 무엇을 수행하고 있는지를 나타낸다. 따라서 노드는 수명주기 활성(Active) 상태를 유지하면서 임무 수행 중 내부 행동 상태를 반복적으로 변경할 수 있다.

상태 머신은 장시간 수행되는 행동을 위해 ROS 2 액션(Action)과 자연스럽게 통합될 수 있다. 주행(Navigate) 상태에 진입하면 내비게이션 액션을 시작하고, 액션 피드백(Action Feedback)은 상황 정보를 갱신하며, 최종 결과는 목표 도달(goal_reached) 또는 내비게이션 실패(navigation_failed) 이벤트를 생성할 수 있다. 상태 머신이 주행 상태에서 다른 상태로 전이하면 취소(Cancellation)를 요청할 수 있다. 이를 통해 액션 전송과 피드백 처리를 다음 행동을 결정하는 의사결정 로직과 분리할 수 있다.

서비스(Service)와 토픽(Topic)은 이 아키텍처에서 상호 보완적인 역할을 한다. 토픽은 연속적인 관측 정보와 비동기 이벤트에 적합하며, 서비스는 재설정 요청이나 상태 조회와 같은 명시적인 관리 작업을 제공할 수 있다. 상태 머신은 특정 전송 방식(Transport)에 종속되지 않는 것이 바람직하다. 통신 콜백은 ROS 2 입력을 내부 이벤트로 변환하고, 행동 출력을 적절한 ROS 2 메시지 또는 명령으로 변환하는 역할을 수행하는 것이 이상적이다.

고장 및 복구 행동(Failure and Recovery Behavior)은 분산된 예외 처리 경로에 포함시키기보다 명시적으로 모델링해야 한다. 복구(Recover) 상태는 위험한 움직임을 중단하고, 고장 정보를 검사하고, 일시적인 조건을 제거하고, 제한된 횟수의 작업을 재시도하거나 상위 감독 시스템에 지원을 요청할 수 있다. 반복되는 장애가 무한 루프를 생성하지 않도록 복구 시도에는 명확한 제한과 종료 조건이 필요하다. 복구할 수 없는 조건은 안전한 고장(Fault) 또는 비상(Emergency) 행동으로 전이되어야 한다.

시간 초과(Timeout)는 정상적으로 진행되지 않는 행동을 감지하는 데 특히 유용하다. 도킹 상태는 정의된 시간 안에 완료되어야 할 수 있으며, 주행 상태는 결과를 무기한 기다리는 대신 진행 상태를 감시할 수 있다. 타이머가 생성하는 이벤트를 사용하면 상태 머신이 정상적인 실행과 정체된 행동(Stalled Behavior)을 구분하고 적절한 복구 작업을 선택할 수 있다. 시간 초과 값은 실제 물리 시스템의 동역학과 측정된 운용 성능을 반영해야 한다.

현재 행동 상태와 전이 이력(Transition History)을 ROS 2 진단(Diagnostics), 토픽, 로그(Log), 추적(Tracing)을 통해 노출하면 관측 가능성(Observability)이 크게 향상된다. 운영자는 로봇이 단순히 정지했다는 사실뿐 아니라 대기 중인지, 복구 중인지, 도킹 중인지 또는 안전 조건에 대응하고 있는지를 확인할 수 있다. 전이 시간, 전이를 발생시킨 이벤트, 출발 상태, 목적 상태, 실패 원인을 기록하면 디버깅과 플릿 수준 분석(Fleet-Level Analysis)에 유용한 근거를 제공한다.

테스트(Testing)는 모든 시나리오에 대해 완전한 물리적 임무를 실행하는 대신 상태와 전이를 중심으로 구성할 수 있다. 단위 테스트(Unit Test)는 이벤트를 주입하고 예상되는 전이, 가드 조건 동작, 진입 동작, 유효하지 않은 전이 거부를 검증할 수 있다. 통합 테스트(Integration Test)는 실제 ROS 2 통신 인터페이스를 연결하여 센서 메시지, 액션 결과, 시간 초과, 서비스 요청이 의도된 행동 이벤트와 결과 출력을 생성하는지 확인할 수 있다.

동일한 상태, 컨텍스트, 이벤트 조건이 동일한 전이 결정을 생성하도록 설계하면 결정론(Determinism)이 향상된다. 외부 센서의 입력 시점 자체는 여전히 비결정적일 수 있지만, 의사결정 로직은 가능한 한 콜백 실행 순서에 대한 숨겨진 의존성을 피해야 한다. 명시적인 이벤트 처리, 전이 테이블(Transition Table), 가드 조건, 제어된 공유 상태 접근을 사용하면 체계적인 디버깅과 안전 검증(Safety Validation)에 충분한 수준으로 행동을 재현할 수 있다.

자율이동로봇(Autonomous Mobile Robot, AMR)은 이 아키텍처를 사용하여 임무 수락, 자율 주행, 장애물 처리, 도킹, 충전, 복구를 조정할 수 있다. 매니퓰레이터(Manipulator)는 접근, 파지, 검증, 이송, 해제를 관리할 수 있으며, 무인항공기(Unmanned Aerial Vehicle, UAV)는 이륙(Takeoff), 순항(Cruise), 임무 실행, 복귀(Return), 착륙(Landing), 비상 행동을 상태로 표현할 수 있다. 피지컬 AI 시스템(Physical AI System) 역시 학습된 구성요소 주변에 결정론적인 감독 경계를 유지하면서 인지 기반 의사결정을 조정할 수 있다.

따라서 강건한 상태 머신 기반 ROS 2 행동 아키텍처(Robust State-Machine-Based ROS 2 Behavior Architecture)는 명시적인 상태, 제어된 전이, 명확하게 정의된 이벤트, 가드 조건, 컨텍스트 데이터, 장애 처리, 동시성 제어, 관측 가능성, 체계적인 테스트를 결합한다. 상태 머신은 인지, 계획 또는 제어 알고리즘을 대체하는 것이 아니라 행동 조정 계층(Behavioral Coordination Layer)의 역할을 수행한다. 이러한 분리를 통해 더욱 지능적인 로봇 기능이 발전하더라도 이해 가능하고 관리 가능한 실행 로직 안에서 동작하도록 구성할 수 있다.

##  

## 03.08 Node Diagnostics: ros2.diagnostics Integration [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

Diagnostics in ROS 2 provide a structured mechanism for reporting the operational health of nodes, sensors, actuators, communication interfaces, and higher-level robot functions. Instead of relying only on console logs, a node can publish machine-readable diagnostic information describing whether a component is operating normally, approaching an abnormal condition, or experiencing a fault. This creates a common health-monitoring layer that can be consumed by operators, supervisors, and fleet systems.

The ROS diagnostics architecture commonly uses diagnostic_msgs messages together with utilities provided by diagnostic_updater and related diagnostic packages. Individual nodes generate diagnostic status information, while aggregation components can combine many reports into a hierarchical system view. This separates local health evaluation from centralized presentation and allows each component to describe its own condition using a standardized communication format.

A DiagnosticStatus represents the health of a particular component or function. It contains a diagnostic level, descriptive message, component name, hardware identifier, and optional key-value information. The conventional levels indicate OK, WARN, ERROR, or STALE conditions. These levels provide a compact summary, while the associated message and values explain the measured condition that caused the status.

The distinction between WARN and ERROR should reflect operational consequences rather than arbitrary numerical thresholds. A temperature approaching its recommended limit may generate WARN while the device continues operating, whereas communication loss with a safety-critical actuator may require ERROR. STALE indicates that expected diagnostic information is no longer current, which is particularly useful for identifying nodes, sensors, or communication paths that have stopped reporting.

Key-value fields allow diagnostic messages to include measurements and contextual information such as temperature, voltage, current, packet loss, sensor frequency, localization quality, memory utilization, error counters, or firmware versions. These values make the diagnostic status useful for engineering analysis rather than merely displaying a colored health indicator. Names and units should remain consistent so automated monitoring tools can interpret them reliably.

The diagnostic_updater package helps ROS nodes publish diagnostic information in a repeatable manner. A node can create an updater, identify its hardware or logical component, and register diagnostic tasks that evaluate specific health conditions. The updater periodically invokes these tasks and publishes their resulting status. This approach keeps diagnostic evaluation organized and avoids manually constructing complete diagnostic messages throughout application code.

Diagnostic tasks should evaluate conditions that have meaningful operational significance. A camera node may report image frequency, frame drops, device temperature, and connection state, while a motor controller node may monitor voltage, current, temperature, communication errors, and controller fault codes. A localization node can expose update rate, covariance, map availability, or confidence indicators relevant to determining whether navigation can continue safely.

Frequency monitoring is particularly useful for periodic sensors and data pipelines. A camera expected to publish at 30 Hz may still appear connected even when its effective rate falls substantially because of hardware, network, or processing problems. Diagnostic utilities can compare observed frequency against an acceptable range and produce warning or error states when message timing violates expected behavior.

Timestamp monitoring complements frequency checking by detecting delayed or incorrectly timestamped data. Sensor messages may continue arriving at an expected rate while their timestamps indicate excessive latency or synchronization problems. For perception and sensor-fusion systems, stale measurements can be as harmful as missing measurements. Diagnostics should therefore consider freshness, delay, and clock consistency where temporal accuracy affects robot behavior.

Diagnostics should distinguish raw measurements from evaluated health. Publishing CPU temperature or battery voltage alone does not state whether the value is acceptable. A diagnostic task interprets measurements according to defined operating limits and reports an appropriate status. Thresholds should come from hardware specifications, system requirements, validated operational data, or safety analysis rather than being selected only for convenient visualization.

A node can also use diagnostic information internally without allowing diagnostics to become an uncontrolled safety mechanism. For example, persistent localization degradation may inform a supervisory state machine that navigation should enter a recovery state. The behavioral controller should make the transition according to defined policies, while diagnostics provide structured evidence about the underlying condition. This preserves separation between health reporting and behavioral authority.

Lifecycle nodes can coordinate diagnostic reporting with their operational state. An Unconfigured node may report that hardware has not yet been initialized, an Inactive node may report readiness without active data production, and an Active node can perform full runtime monitoring. During cleanup or shutdown, diagnostic tasks can be disabled or updated so operators do not mistake an intentionally inactive component for an unexpected failure.

Diagnostic aggregation becomes valuable as robot complexity increases. A mobile robot may contain dozens of nodes reporting sensor, compute, network, power, localization, navigation, and actuator health. An aggregator can organize these reports into logical categories and summarize lower-level conditions. Operators can then inspect an overall robot status and progressively drill down toward the component responsible for a warning or error.

Aggregation should preserve root-cause information rather than hiding detailed failures behind one overall status. If navigation is degraded because localization quality is poor, and localization is degraded because LiDAR messages are stale, the diagnostic hierarchy should make that dependency understandable. Clear naming and consistent grouping help operators distinguish primary faults from secondary symptoms generated by dependent software components.

Diagnostic reporting must also account for transient faults. A single dropped packet or brief scheduling delay may not justify escalating an entire robot to an error state. Debouncing, persistence timers, moving statistics, hysteresis, or consecutive-failure counters can prevent unstable status changes. At the same time, excessive filtering can hide rapidly developing faults, so persistence policies should reflect the dynamics and consequences of each monitored condition.

Diagnostics and logging serve different but complementary purposes. Logs provide detailed event records useful for software debugging, while diagnostics summarize the current operational health in a structured format. A diagnostic ERROR may be accompanied by detailed logs explaining the failure sequence, but operators should not need to search large log files simply to determine whether a sensor or actuator is healthy at the present moment.

Tracing and performance monitoring can complement diagnostics when the health problem involves timing rather than explicit component failure. Callback latency, executor starvation, CPU saturation, or communication congestion may cause a node to miss expected deadlines even though no hardware error exists. Diagnostic tasks can expose summarized performance indicators, while tracing tools provide deeper evidence for identifying the scheduling or execution mechanism responsible.

Diagnostic callbacks should remain lightweight and should not create significant interference with the function they monitor. Expensive hardware queries, blocking network operations, or large computations performed inside diagnostic updates can introduce latency into normal node execution. Measurements can instead be collected during normal operation and stored in thread-safe state, allowing diagnostic tasks to evaluate recent values efficiently when the updater requests a report.

Thread safety becomes important when diagnostic tasks read data updated by subscriptions, timers, device threads, or control callbacks. Shared measurements should be protected with suitable synchronization or represented through atomic values and immutable snapshots. Diagnostic code must not introduce data races merely because it is considered secondary functionality. The monitoring path should follow the same concurrency discipline as the operational path.

Testing should verify diagnostic behavior as deliberately as functional behavior. Unit tests can inject nominal, warning, failure, stale, and boundary conditions and verify the expected diagnostic levels and messages. Integration tests can disconnect devices, delay messages, reduce publication rates, or inject simulated fault codes. This confirms that the diagnostic system detects realistic failures and recovers correctly when normal operation returns.

For an AMR, diagnostics can combine battery health, motor-controller state, wheel feedback, LiDAR frequency, camera availability, GNSS or localization quality, network connectivity, CPU/GPU temperature, navigation status, and emergency-system information. Manipulators can monitor joint drives and end effectors, while Physical AI systems can add inference latency, accelerator utilization, model availability, and perception-pipeline freshness.

At fleet scale, standardized diagnostics provide a foundation for remote maintenance and operational analytics. A fleet manager can identify recurring sensor failures, thermal problems, communication degradation, or robots that repeatedly enter warning states. Historical diagnostic records can support preventive maintenance and reliability analysis, provided that component names, severity definitions, timestamps, units, and software versions remain sufficiently consistent across deployed platforms.

A robust ROS 2 diagnostics architecture therefore combines standardized diagnostic messages, diagnostic_updater tasks, meaningful health thresholds, frequency and timestamp monitoring, hierarchical aggregation, lifecycle awareness, concurrency safety, logging, tracing, and systematic fault testing. When integrated into each node as an intentional design responsibility, diagnostics transform scattered runtime symptoms into observable system health that can support reliable robots, maintenance workflows, and scalable Physical AI operations.

ROS 2의 진단(Diagnostics)은 노드, 센서, 액추에이터(Actuator), 통신 인터페이스(Communication Interface), 상위 수준 로봇 기능의 운용 상태를 보고하기 위한 구조화된 메커니즘을 제공한다. 콘솔 로그(Console Log)에만 의존하는 대신 노드는 구성요소가 정상적으로 동작하는지, 비정상 상태에 접근하고 있는지 또는 고장이 발생했는지를 설명하는 기계 판독 가능 진단 정보(Machine-Readable Diagnostic Information)를 발행할 수 있다. 이를 통해 운영자, 감독 시스템(Supervisor), 플릿 시스템(Fleet System)이 공통으로 활용할 수 있는 상태 모니터링 계층(Health-Monitoring Layer)을 구성할 수 있다.

ROS 진단 아키텍처(ROS Diagnostics Architecture)는 일반적으로 diagnostic_msgs 메시지와 diagnostic_updater 및 관련 진단 패키지가 제공하는 유틸리티(Utility)를 함께 사용한다. 개별 노드는 진단 상태 정보를 생성하고, 집계 구성요소(Aggregation Component)는 여러 보고를 결합하여 계층적인 시스템 상태를 구성할 수 있다. 이러한 구조는 로컬 상태 평가(Local Health Evaluation)를 중앙화된 표현 방식과 분리하며, 각 구성요소가 표준화된 통신 형식을 통해 자신의 상태를 설명할 수 있도록 한다.

진단 상태(DiagnosticStatus)는 특정 구성요소 또는 기능의 상태를 나타낸다. 여기에는 진단 수준(Diagnostic Level), 설명 메시지, 구성요소 이름, 하드웨어 식별자(Hardware Identifier), 선택적인 키-값 정보(Key-Value Information)가 포함된다. 일반적으로 사용하는 수준은 정상(OK), 경고(WARN), 오류(ERROR), 오래된 정보(STALE) 상태를 나타낸다. 이러한 수준은 간결한 상태 요약을 제공하며, 관련 메시지와 값은 해당 상태가 발생한 측정 조건을 설명한다.

경고(WARN)와 오류(ERROR)의 구분은 임의의 수치 임계값이 아니라 실제 운용 결과(Operational Consequence)를 반영해야 한다. 권장 한계에 가까워지는 온도는 장치가 계속 동작하는 동안 경고를 발생시킬 수 있지만, 안전에 중요한 액추에이터와의 통신 손실은 오류 상태를 요구할 수 있다. 오래된 정보(STALE)는 예상된 진단 정보가 더 이상 최신 상태가 아님을 나타내며, 보고가 중단된 노드, 센서 또는 통신 경로를 식별하는 데 특히 유용하다.

키-값 필드(Key-Value Field)를 사용하면 진단 메시지에 온도, 전압, 전류, 패킷 손실(Packet Loss), 센서 주파수, 위치추정 품질(Localization Quality), 메모리 사용률, 오류 카운터(Error Counter), 펌웨어 버전(Firmware Version)과 같은 측정값과 상황 정보를 포함할 수 있다. 이러한 값은 진단 상태를 단순한 색상 기반 상태 표시가 아니라 엔지니어링 분석에 유용한 정보로 만든다. 자동화된 모니터링 도구가 신뢰성 있게 해석할 수 있도록 이름과 단위는 일관성을 유지해야 한다.

diagnostic_updater 패키지는 ROS 노드가 진단 정보를 일관된 방식으로 발행할 수 있도록 지원한다. 노드는 업데이터(Updater)를 생성하고 하드웨어 또는 논리적 구성요소를 식별한 후 특정 상태 조건을 평가하는 진단 작업(Diagnostic Task)을 등록할 수 있다. 업데이터는 이러한 작업을 주기적으로 호출하고 그 결과 상태를 발행한다. 이 방식은 진단 평가를 체계적으로 구성하며 응용 프로그램 코드 전체에서 완전한 진단 메시지를 직접 반복 생성하는 문제를 방지한다.

진단 작업(Diagnostic Task)은 실제 운용에 의미가 있는 조건을 평가해야 한다. 카메라 노드는 영상 주파수, 프레임 손실(Frame Drop), 장치 온도, 연결 상태를 보고할 수 있으며, 모터 제어기(Motor Controller) 노드는 전압, 전류, 온도, 통신 오류, 제어기 고장 코드를 감시할 수 있다. 위치추정 노드는 갱신 주기, 공분산(Covariance), 지도 가용성 또는 내비게이션을 안전하게 계속할 수 있는지 판단하는 데 필요한 신뢰도 지표를 제공할 수 있다.

주파수 모니터링(Frequency Monitoring)은 주기적인 센서와 데이터 파이프라인(Data Pipeline)에 특히 유용하다. 30 Hz로 발행되어야 하는 카메라는 하드웨어, 네트워크 또는 처리 문제로 실제 발행률이 크게 감소하더라도 연결 자체는 유지된 것처럼 보일 수 있다. 진단 유틸리티는 관측된 주파수를 허용 가능한 범위와 비교하여 메시지 타이밍이 예상 동작을 벗어날 경우 경고 또는 오류 상태를 생성할 수 있다.

타임스탬프 모니터링(Timestamp Monitoring)은 지연되거나 잘못된 시간정보를 가진 데이터를 감지함으로써 주파수 검사를 보완한다. 센서 메시지가 예상된 속도로 계속 도착하더라도 타임스탬프가 과도한 지연이나 시간 동기화 문제를 나타낼 수 있다. 인지(Perception) 및 센서 융합(Sensor Fusion) 시스템에서는 오래된 측정값이 누락된 측정값만큼 위험할 수 있다. 따라서 시간 정확도가 로봇 행동에 영향을 주는 경우 진단은 데이터 최신성(Freshness), 지연, 시계 일관성(Clock Consistency)을 고려해야 한다.

진단은 원시 측정값(Raw Measurement)과 평가된 상태(Evaluated Health)를 구분해야 한다. CPU 온도 또는 배터리 전압만 발행하는 것으로는 해당 값이 허용 가능한 상태인지 알 수 없다. 진단 작업은 정의된 운용 한계에 따라 측정값을 해석하고 적절한 상태를 보고한다. 임계값은 단순히 시각화를 편리하게 하기 위해 선택하는 것이 아니라 하드웨어 사양, 시스템 요구사항, 검증된 운용 데이터 또는 안전 분석(Safety Analysis)을 기반으로 설정해야 한다.

노드는 진단 정보를 내부적으로 사용할 수도 있지만 진단 자체가 통제되지 않은 안전 메커니즘(Uncontrolled Safety Mechanism)이 되어서는 안 된다. 예를 들어 지속적인 위치추정 성능 저하는 감독 상태 머신(Supervisory State Machine)에 내비게이션이 복구 상태로 진입해야 한다는 정보를 제공할 수 있다. 행동 제어기(Behavioral Controller)는 정의된 정책에 따라 상태 전이를 결정하고, 진단은 그 판단의 기반이 되는 상태 정보를 구조화하여 제공해야 한다. 이를 통해 상태 보고와 행동 결정 권한을 분리할 수 있다.

수명주기 노드(Lifecycle Node)는 진단 보고를 자신의 운용 상태와 조정할 수 있다. 미구성(Unconfigured) 노드는 하드웨어가 아직 초기화되지 않았음을 보고하고, 비활성(Inactive) 노드는 실제 데이터를 생성하지 않지만 준비 상태임을 나타낼 수 있으며, 활성(Active) 노드는 전체 런타임 모니터링(Runtime Monitoring)을 수행할 수 있다. 정리(Cleanup) 또는 종료(Shutdown) 과정에서는 운영자가 의도적으로 비활성화된 구성요소를 예상치 못한 장애로 오인하지 않도록 진단 작업을 비활성화하거나 상태를 적절히 갱신할 수 있다.

로봇의 복잡성이 증가하면 진단 집계(Diagnostic Aggregation)의 가치도 커진다. 이동 로봇은 센서, 컴퓨팅, 네트워크, 전원, 위치추정, 내비게이션, 액추에이터 상태를 보고하는 수십 개의 노드를 포함할 수 있다. 집계기(Aggregator)는 이러한 보고를 논리적인 범주로 구성하고 하위 수준의 상태를 요약할 수 있다. 이를 통해 운영자는 전체 로봇 상태를 확인한 후 경고나 오류를 발생시킨 개별 구성요소까지 단계적으로 추적할 수 있다.

집계(Aggregation)는 하나의 전체 상태 뒤에 세부 장애를 숨기는 대신 근본 원인 정보(Root-Cause Information)를 유지해야 한다. 위치추정 품질이 저하되어 내비게이션 성능이 떨어지고, 그 위치추정 문제가 LiDAR 메시지의 오래된 데이터 때문에 발생했다면 진단 계층은 이러한 의존 관계를 이해할 수 있도록 표현해야 한다. 명확한 명명 규칙과 일관된 그룹화는 운영자가 주요 고장(Primary Fault)과 종속 소프트웨어 구성요소에서 발생한 이차적 증상(Secondary Symptom)을 구분하는 데 도움이 된다.

진단 보고는 일시적 장애(Transient Fault)도 고려해야 한다. 하나의 패킷 손실이나 짧은 스케줄링 지연만으로 로봇 전체를 오류 상태로 전환하는 것은 적절하지 않을 수 있다. 디바운싱(Debouncing), 지속 시간 타이머(Persistence Timer), 이동 통계(Moving Statistics), 히스테리시스(Hysteresis), 연속 실패 카운터(Consecutive-Failure Counter)를 사용하면 불안정한 상태 변화를 방지할 수 있다. 그러나 지나친 필터링은 빠르게 악화되는 장애를 숨길 수 있으므로 지속성 정책은 각 감시 조건의 동역학과 장애 결과를 반영해야 한다.

진단(Diagnostics)과 로깅(Logging)은 서로 다른 목적을 가지면서 상호 보완적인 역할을 한다. 로그는 소프트웨어 디버깅에 유용한 상세 이벤트 기록을 제공하는 반면, 진단은 현재의 운용 상태를 구조화된 형식으로 요약한다. 진단 오류(ERROR)는 장애 발생 과정을 설명하는 상세 로그와 함께 제공될 수 있지만, 운영자가 현재 센서나 액추에이터가 정상인지 판단하기 위해 대규모 로그 파일을 직접 검색해야 하는 구조는 바람직하지 않다.

상태 문제가 명시적인 구성요소 장애가 아니라 시간적 실행 문제와 관련된 경우 추적(Tracing)과 성능 모니터링(Performance Monitoring)이 진단을 보완할 수 있다. 콜백 지연시간(Callback Latency), 실행기 기아 상태(Executor Starvation), CPU 포화(CPU Saturation), 통신 혼잡(Communication Congestion)은 하드웨어 오류가 없더라도 노드가 예상된 마감시간을 지키지 못하게 할 수 있다. 진단 작업은 요약된 성능 지표를 제공하고, 추적 도구는 문제를 일으킨 스케줄링 또는 실행 메커니즘을 식별하기 위한 보다 상세한 근거를 제공할 수 있다.

진단 콜백(Diagnostic Callback)은 가볍게 유지되어야 하며 감시 대상 기능에 상당한 간섭을 발생시키지 않아야 한다. 진단 갱신 내부에서 수행되는 고비용 하드웨어 질의, 블로킹 네트워크 작업 또는 대규모 계산은 정상적인 노드 실행에 지연을 유발할 수 있다. 대신 정상 운용 과정에서 측정값을 수집하여 스레드 안전 상태(Thread-Safe State)에 저장하고, 업데이터가 보고를 요청할 때 진단 작업이 최신 값을 효율적으로 평가하도록 구성할 수 있다.

진단 작업이 구독, 타이머, 장치 스레드 또는 제어 콜백에 의해 갱신되는 데이터를 읽을 때는 스레드 안전성(Thread Safety)이 중요하다. 공유 측정값은 적절한 동기화(Synchronization)를 통해 보호하거나 원자적 값(Atomic Value)과 불변 스냅샷(Immutable Snapshot)으로 표현해야 한다. 진단 코드가 부가적인 기능이라는 이유로 데이터 경쟁(Data Race)을 발생시켜서는 안 된다. 모니터링 경로도 실제 운용 경로와 동일한 수준의 동시성 규칙을 따라야 한다.

테스트(Testing)는 기능 동작과 마찬가지로 진단 동작도 의도적으로 검증해야 한다. 단위 테스트(Unit Test)는 정상, 경고, 장애, 오래된 정보, 경계 조건을 주입하여 예상되는 진단 수준과 메시지를 확인할 수 있다. 통합 테스트(Integration Test)는 장치를 분리하고, 메시지를 지연시키고, 발행 주파수를 낮추거나 모의 고장 코드(Simulated Fault Code)를 주입할 수 있다. 이를 통해 진단 시스템이 실제적인 장애를 감지하고 정상 운용이 복구되었을 때 올바르게 상태를 회복하는지 검증할 수 있다.

자율이동로봇(Autonomous Mobile Robot, AMR)에서는 배터리 상태, 모터 제어기 상태, 휠 피드백(Wheel Feedback), LiDAR 주파수, 카메라 가용성, GNSS 또는 위치추정 품질, 네트워크 연결성, CPU/GPU 온도, 내비게이션 상태, 비상 시스템 정보를 진단 체계에 통합할 수 있다. 매니퓰레이터(Manipulator)는 관절 구동기(Joint Drive)와 엔드 이펙터(End Effector)를 감시할 수 있으며, 피지컬 AI 시스템(Physical AI System)은 추론 지연시간, 가속기 활용률, 모델 가용성, 인지 파이프라인 최신성(Perception-Pipeline Freshness)을 추가할 수 있다.

플릿 규모(Fleet Scale)에서는 표준화된 진단 정보가 원격 유지보수(Remote Maintenance)와 운용 분석(Operational Analytics)의 기반을 제공한다. 플릿 관리자(Fleet Manager)는 반복적으로 발생하는 센서 장애, 열 문제, 통신 성능 저하 또는 지속적으로 경고 상태에 진입하는 로봇을 식별할 수 있다. 구성요소 이름, 심각도 정의, 타임스탬프, 단위, 소프트웨어 버전이 배포된 플랫폼 전체에서 충분한 일관성을 유지한다면 과거 진단 기록은 예방 정비(Predictive Maintenance)와 신뢰성 분석(Reliability Analysis)을 지원할 수 있다.

따라서 강건한 ROS 2 진단 아키텍처(Robust ROS 2 Diagnostics Architecture)는 표준화된 진단 메시지, diagnostic_updater 작업, 의미 있는 상태 임계값, 주파수 및 타임스탬프 모니터링, 계층형 집계(Hierarchical Aggregation), 수명주기 인식(Lifecycle Awareness), 동시성 안전성, 로깅, 추적, 체계적인 장애 테스트를 결합한다. 진단을 각 노드의 명시적인 설계 책임으로 통합하면 분산된 런타임 증상을 관측 가능한 시스템 상태로 변환하여 신뢰성 높은 로봇, 유지보수 워크플로(Maintenance Workflow), 확장 가능한 피지컬 AI 운용을 지원할 수 있다.

##  

## 03.09 Node Unit Testing: gtest / ament.cmake.gtest [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

Unit testing in ROS 2 verifies small software units independently before they are combined into complete robotic behaviors. A node may interact with topics, services, parameters, timers, algorithms, and hardware interfaces, making failures difficult to isolate after full integration. GoogleTest, together with ament_cmake_gtest, provides a standard C++ testing workflow for validating node logic, utility classes, interfaces, and failure handling during normal package builds.

GoogleTest, commonly called gtest, is a C++ testing framework that organizes verification around test cases and assertions. Tests execute application code with controlled inputs and compare observed behavior against expected results. Assertions such as EXPECT_EQ, EXPECT_TRUE, EXPECT_NEAR, and ASSERT_\* allow developers to express precise conditions. Failed expectations identify the location and nature of incorrect behavior without requiring an entire robot application to run.

ROS 2 integrates GoogleTest into the ament build system through ament_cmake_gtest. This package simplifies declaring gtest executables in CMake and registering them with the ROS 2 testing infrastructure. Tests can then be built with colcon and executed as part of the package test process. The integration allows unit testing to follow the same dependency, package, and workspace conventions used by the production ROS 2 software.

A typical CMakeLists.txt enables testing and locates ament_cmake_gtest before registering a test target with ament_add_gtest. The test source is compiled into an executable and linked against the libraries or components being tested. Required ROS 2 dependencies are attached to the test target in the same manner as other package targets. This makes test compilation an effective check that public interfaces and dependency declarations remain valid.

Package design strongly affects testability. If all planning, validation, state management, and communication logic is implemented directly inside one large node class, tests may require extensive ROS initialization and asynchronous communication. Separating algorithmic logic into ordinary C++ classes or libraries allows most behavior to be tested without spinning a complete node. The ROS 2 node can then remain a relatively thin integration layer around independently verified functionality.

Pure functions and deterministic components are particularly suitable for unit tests. Coordinate conversions, command limits, parameter validation, state transitions, trajectory checks, filtering rules, and message interpretation can be exercised using fixed input data. Tests should include nominal cases as well as boundaries and invalid inputs. Deterministic tests provide fast feedback and make failures easier to reproduce than scenarios dependent on physical sensors or timing.

Node-level unit tests can initialize an rclcpp context and construct the node directly when ROS-specific behavior must be examined. The test can inspect declared parameters, create messages, invoke accessible interfaces, or execute callbacks through controlled communication. ROS initialization and shutdown should be managed consistently so multiple tests do not interfere with one another through residual global state or incorrectly shared contexts.

Test fixtures are useful when several tests require the same initialization. A GoogleTest fixture derives from testing::Test and provides SetUp and TearDown operations for constructing reusable objects, initializing state, or releasing resources. For ROS 2 tests, fixtures can manage nodes, executors, test publishers, temporary configuration, and contexts. Centralizing this setup reduces duplication and ensures that each test begins from a known condition.

Assertions should verify externally meaningful behavior rather than internal implementation details whenever possible. A test for velocity limiting should verify the resulting command, not the exact sequence of private helper calls used to produce it. Tests that depend heavily on implementation structure become fragile during refactoring even when observable behavior remains correct. Stable tests are based on functional contracts and public interfaces.

Parameter behavior is an important target for node testing. Tests can verify default values, valid overrides, rejection of invalid ranges, and relationships among parameters. If a node uses parameter callbacks, tests should confirm that accepted updates modify configuration correctly while invalid updates preserve the previous valid state. Boundary conditions are particularly important because many field failures occur near configured operational limits.

State-machine-based nodes can be tested by injecting events and verifying state transitions without executing a complete physical mission. Tests can confirm valid transitions, rejected transitions, guard conditions, recovery paths, timeouts, and emergency behavior. When transition logic is separated from ROS transport, the same behavioral model can be tested quickly with ordinary C++ calls while separate integration tests verify topic, service, or action connections.

Mocks, fakes, and test doubles help isolate a unit from dependencies that are expensive, nondeterministic, or unavailable during automated testing. A planner can be tested with a fake map provider, or a behavior manager can use a mock motion interface. Interfaces designed for dependency injection make such substitutions easier. The goal is not to reproduce the entire robot but to control the environment around the specific behavior being verified.

Time-dependent code requires careful design because tests based on real sleep intervals are slow and can become unreliable under different computing loads. Whenever practical, timing logic should depend on controllable clocks, explicit timestamps, or injectable time sources. Tests can then simulate timeout boundaries and delayed events deterministically. ROS time, system time, and simulated time should not be mixed implicitly within logic that requires repeatable verification.

Concurrency also requires dedicated testing. Nodes using MultiThreadedExecutor, Reentrant callback groups, shared state, or worker threads may behave correctly under simple unit tests but fail when operations overlap. Unit tests can validate synchronization primitives and thread-safe data structures, while stress and integration tests should exercise concurrent callbacks repeatedly. Race detection tools can provide additional evidence beyond functional assertions.

Error paths deserve the same attention as normal operation. Tests should deliberately provide malformed input, unavailable resources, invalid parameters, failed plugin creation, communication loss indications, and algorithm errors where applicable. The expected response may be rejection, a diagnostic status, a recovery transition, or a safe output. Explicit failure tests prevent error handling from becoming code that exists syntactically but is rarely exercised.

Diagnostics can also be verified through unit tests. A diagnostic task can be supplied with nominal measurements, warning thresholds, error conditions, and stale timestamps, and the resulting status can be checked. This ensures that operational monitoring follows the same engineering discipline as primary functionality. Changes to thresholds or diagnostic logic can then be reviewed through reproducible test results rather than manual observation.

Test independence is essential for reliable automated execution. One test should not depend on another test having run first or on files, parameters, ports, topics, and global objects left behind by previous tests. Temporary resources should be uniquely created and cleaned up. Tests that pass individually but fail when executed as a suite often indicate hidden shared state, timing assumptions, or incomplete teardown behavior.

The standard ROS 2 workflow uses colcon test to execute registered tests across selected packages or a workspace. Test results can then be summarized with colcon test-result, making failures visible during local development and continuous integration. Developers should run relevant tests after code changes, while CI pipelines can rebuild packages and execute the test suite automatically for commits, merge requests, or release candidates.

Continuous integration turns unit testing from an occasional developer activity into a persistent quality gate. A build server can compile the package on supported environments, execute gtests, inspect failures, and reject changes that violate verified behavior. Combined with static analysis, formatting checks, integration tests, and system tests, unit tests provide the fast inner layer of a broader verification strategy rather than replacing higher-level validation.

Coverage measurements can help identify logic that is rarely exercised, but coverage percentage should not be treated as proof of correctness. Executing a line does not guarantee that meaningful conditions were checked. High-value tests focus on requirements, boundaries, failure modes, state transitions, and safety-relevant decisions. Coverage is most useful as evidence for discovering untested regions and guiding additional engineering review.

For an AMR, unit tests can verify velocity limits, mission validation, navigation-state transitions, docking conditions, parameter ranges, and diagnostic thresholds. Manipulator software can test joint constraints, grasp-state logic, and command validation, while Physical AI nodes can test preprocessing, inference-result interpretation, confidence rules, and fallback decisions without requiring every test to execute a complete learned model.

A robust ROS 2 testing architecture therefore combines testable package structure, GoogleTest assertions and fixtures, ament_cmake_gtest integration, deterministic logic, dependency injection, controlled ROS contexts, failure testing, concurrency verification, and automated execution with colcon and CI. By making unit tests part of node design rather than an activity added after implementation, developers can detect regressions early and evolve complex robotic and Physical AI software with greater confidence.

ROS 2의 단위 테스트(Unit Testing)는 작은 소프트웨어 단위가 완전한 로봇 행동으로 통합되기 전에 각각을 독립적으로 검증한다. 하나의 노드는 토픽(Topic), 서비스(Service), 매개변수(Parameter), 타이머(Timer), 알고리즘(Algorithm), 하드웨어 인터페이스(Hardware Interface)와 상호작용할 수 있으므로 전체 통합 이후에는 장애 원인을 분리하기 어려울 수 있다. GoogleTest와 ament_cmake_gtest를 함께 사용하면 일반적인 패키지 빌드 과정에서 노드 로직, 유틸리티 클래스, 인터페이스, 장애 처리 동작을 검증하는 표준 C++ 테스트 워크플로(Test Workflow)를 구성할 수 있다.

일반적으로 gtest라고 부르는 GoogleTest는 테스트 케이스(Test Case)와 어설션(Assertion)을 중심으로 검증을 구성하는 C++ 테스트 프레임워크(Test Framework)이다. 테스트는 제어된 입력으로 응용 프로그램 코드를 실행하고 관측된 동작을 예상 결과와 비교한다. EXPECT_EQ, EXPECT_TRUE, EXPECT_NEAR, ASSERT_\*와 같은 어설션을 사용하여 개발자는 정확한 검증 조건을 표현할 수 있다. 예상 조건이 실패하면 전체 로봇 응용 프로그램을 실행하지 않고도 잘못된 동작의 위치와 특성을 식별할 수 있다.

ROS 2는 ament_cmake_gtest를 통해 GoogleTest를 ament 빌드 시스템(Build System)에 통합한다. 이 패키지는 CMake에서 gtest 실행 파일을 선언하고 ROS 2 테스트 인프라(Test Infrastructure)에 등록하는 과정을 단순화한다. 이후 테스트는 colcon을 사용하여 빌드되고 패키지 테스트 과정의 일부로 실행될 수 있다. 이러한 통합을 통해 단위 테스트는 실제 ROS 2 소프트웨어에서 사용하는 것과 동일한 의존성, 패키지, 작업공간(Workspace) 규칙을 따를 수 있다.

일반적인 CMakeLists.txt에서는 테스트 기능을 활성화하고 ament_cmake_gtest를 찾은 다음 ament_add_gtest를 사용하여 테스트 대상(Test Target)을 등록한다. 테스트 소스는 실행 파일로 컴파일되고 테스트 대상 라이브러리 또는 구성요소와 링크(Link)된다. 필요한 ROS 2 의존성은 다른 패키지 대상과 동일한 방식으로 테스트 대상에 연결된다. 따라서 테스트 컴파일 과정 자체도 공개 인터페이스(Public Interface)와 의존성 선언이 계속 유효한지를 확인하는 효과적인 검증 수단이 된다.

패키지 설계(Package Design)는 테스트 가능성(Testability)에 큰 영향을 미친다. 계획, 검증, 상태 관리, 통신 로직을 모두 하나의 거대한 노드 클래스 내부에 직접 구현하면 테스트를 위해 광범위한 ROS 초기화와 비동기 통신이 필요할 수 있다. 알고리즘 로직을 일반 C++ 클래스나 라이브러리로 분리하면 완전한 노드를 실행하지 않고도 대부분의 동작을 테스트할 수 있다. ROS 2 노드는 독립적으로 검증된 기능을 둘러싸는 비교적 얇은 통합 계층(Integration Layer)으로 유지할 수 있다.

순수 함수(Pure Function)와 결정론적 구성요소(Deterministic Component)는 단위 테스트에 특히 적합하다. 좌표 변환(Coordinate Conversion), 명령 제한(Command Limit), 매개변수 검증, 상태 전이(State Transition), 궤적 검사(Trajectory Check), 필터링 규칙(Filtering Rule), 메시지 해석(Message Interpretation)을 고정된 입력 데이터로 검증할 수 있다. 테스트에는 정상적인 경우뿐 아니라 경계 조건(Boundary Condition)과 유효하지 않은 입력도 포함해야 한다. 결정론적 테스트는 빠른 피드백을 제공하며 물리 센서나 타이밍에 의존하는 시나리오보다 장애를 쉽게 재현할 수 있다.

ROS 관련 동작을 검증해야 하는 경우 노드 수준 단위 테스트(Node-Level Unit Test)는 rclcpp 컨텍스트(Context)를 초기화하고 노드를 직접 생성할 수 있다. 테스트는 선언된 매개변수를 확인하고, 메시지를 생성하며, 접근 가능한 인터페이스를 호출하거나 제어된 통신을 통해 콜백(Callback)을 실행할 수 있다. 여러 테스트가 잔여 전역 상태(Global State)나 잘못 공유된 컨텍스트 때문에 서로 영향을 주지 않도록 ROS 초기화와 종료(Shutdown)를 일관성 있게 관리해야 한다.

여러 테스트에서 동일한 초기화가 필요한 경우 테스트 픽스처(Test Fixture)가 유용하다. GoogleTest 픽스처는 testing::Test를 상속하고 재사용 가능한 객체 생성, 상태 초기화, 자원 해제를 위한 SetUp과 TearDown 작업을 제공한다. ROS 2 테스트에서는 픽스처가 노드, 실행기(Executor), 테스트 발행자(Test Publisher), 임시 구성, 컨텍스트를 관리할 수 있다. 이러한 설정을 중앙화하면 코드 중복을 줄이고 각각의 테스트가 알려진 상태(Known Condition)에서 시작하도록 보장할 수 있다.

어설션(Assertion)은 가능하면 내부 구현 세부사항보다 외부에서 의미 있는 동작을 검증해야 한다. 속도 제한 테스트는 결과 명령을 검증해야 하며, 해당 결과를 생성하기 위해 호출된 비공개 보조 함수(Private Helper Function)의 정확한 순서를 검사할 필요는 없다. 구현 구조에 지나치게 의존하는 테스트는 관측 가능한 동작이 올바르게 유지되더라도 리팩터링(Refactoring) 과정에서 쉽게 깨질 수 있다. 안정적인 테스트는 기능 계약(Functional Contract)과 공개 인터페이스를 기반으로 해야 한다.

매개변수 동작(Parameter Behavior)은 노드 테스트에서 중요한 검증 대상이다. 테스트는 기본값(Default Value), 유효한 재정의(Override), 허용 범위를 벗어난 값의 거부, 매개변수 사이의 관계를 검증할 수 있다. 노드가 매개변수 콜백(Parameter Callback)을 사용하는 경우 허용된 갱신이 구성을 올바르게 변경하는지, 유효하지 않은 갱신이 기존의 유효한 상태를 유지하는지 확인해야 한다. 실제 현장 장애의 상당수가 설정된 운용 한계 주변에서 발생할 수 있으므로 경계 조건은 특히 중요하다.

상태 머신 기반 노드(State-Machine-Based Node)는 완전한 물리적 임무를 실행하지 않고 이벤트(Event)를 주입하고 상태 전이를 검증하는 방식으로 테스트할 수 있다. 테스트는 유효한 전이, 거부되는 전이, 가드 조건(Guard Condition), 복구 경로(Recovery Path), 시간 초과(Timeout), 비상 동작을 확인할 수 있다. 전이 로직이 ROS 전송 계층(Transport)과 분리되어 있다면 동일한 행동 모델을 일반적인 C++ 호출로 빠르게 테스트하고, 별도의 통합 테스트에서 토픽, 서비스 또는 액션(Action) 연결을 검증할 수 있다.

모의 객체(Mock), 가짜 객체(Fake), 테스트 대역(Test Double)은 비용이 크거나 비결정적이거나 자동화 테스트 환경에서 사용할 수 없는 의존성으로부터 테스트 대상을 격리하는 데 도움이 된다. 플래너(Planner)는 가짜 지도 제공자(Fake Map Provider)를 사용하여 테스트할 수 있고, 행동 관리자(Behavior Manager)는 모의 모션 인터페이스(Mock Motion Interface)를 사용할 수 있다. 의존성 주입(Dependency Injection)을 지원하도록 인터페이스를 설계하면 이러한 대체가 쉬워진다. 목적은 전체 로봇을 재현하는 것이 아니라 검증하려는 특정 동작 주변의 환경을 제어하는 것이다.

시간 의존 코드(Time-Dependent Code)는 실제 대기 시간을 사용하는 테스트가 느리고 컴퓨팅 부하에 따라 불안정해질 수 있으므로 신중한 설계가 필요하다. 가능한 경우 타이밍 로직은 제어 가능한 시계(Controllable Clock), 명시적인 타임스탬프(Timestamp), 주입 가능한 시간 소스(Injectable Time Source)에 의존하도록 설계해야 한다. 그러면 테스트에서 시간 초과 경계와 지연 이벤트를 결정론적으로 모의할 수 있다. 반복 가능한 검증이 필요한 로직에서는 ROS 시간(ROS Time), 시스템 시간(System Time), 시뮬레이션 시간(Simulated Time)을 암묵적으로 혼합해서는 안 된다.

동시성(Concurrency) 역시 전용 테스트가 필요하다. 다중 스레드 실행기(MultiThreadedExecutor), 재진입 가능 콜백 그룹(Reentrant Callback Group), 공유 상태(Shared State), 작업 스레드(Worker Thread)를 사용하는 노드는 단순한 단위 테스트에서는 정상적으로 동작하지만 작업이 중첩될 때 실패할 수 있다. 단위 테스트는 동기화 기본 요소(Synchronization Primitive)와 스레드 안전 데이터 구조(Thread-Safe Data Structure)를 검증할 수 있으며, 스트레스 테스트(Stress Test)와 통합 테스트는 동시 콜백을 반복적으로 실행해야 한다. 경쟁 상태 탐지 도구(Race Detection Tool)는 기능적 어설션 이상의 추가적인 검증 근거를 제공할 수 있다.

오류 경로(Error Path)는 정상 동작과 동일한 수준의 관심을 받아야 한다. 테스트에서는 적용 가능한 경우 잘못된 입력, 사용할 수 없는 자원, 유효하지 않은 매개변수, 플러그인 생성 실패, 통신 손실 표시, 알고리즘 오류를 의도적으로 제공해야 한다. 예상되는 대응은 요청 거부, 진단 상태(Diagnostic Status), 복구 전이 또는 안전 출력(Safe Output)일 수 있다. 명시적인 장애 테스트(Failure Test)를 통해 오류 처리 코드가 문법적으로만 존재하고 실제로는 거의 실행되지 않는 문제를 방지할 수 있다.

진단(Diagnostics) 역시 단위 테스트를 통해 검증할 수 있다. 진단 작업(Diagnostic Task)에 정상 측정값, 경고 임계값, 오류 조건, 오래된 타임스탬프를 제공하고 결과 상태를 확인할 수 있다. 이를 통해 운용 모니터링(Operational Monitoring)도 주요 기능과 동일한 엔지니어링 원칙을 따르도록 할 수 있다. 임계값이나 진단 로직의 변경 사항은 수동 관찰이 아니라 재현 가능한 테스트 결과를 통해 검토할 수 있다.

신뢰성 높은 자동화 실행을 위해서는 테스트 독립성(Test Independence)이 필수적이다. 하나의 테스트가 다른 테스트가 먼저 실행되었는지 여부에 의존하거나 이전 테스트가 남긴 파일, 매개변수, 포트, 토픽, 전역 객체에 의존해서는 안 된다. 임시 자원은 고유하게 생성하고 실행 후 정리해야 한다. 개별적으로는 통과하지만 전체 테스트 스위트(Test Suite)에서는 실패하는 테스트는 숨겨진 공유 상태, 타이밍 가정 또는 불완전한 종료 처리(TearDown)를 나타내는 경우가 많다.

표준 ROS 2 워크플로에서는 colcon test를 사용하여 선택한 패키지 또는 작업공간에 등록된 테스트를 실행한다. 이후 colcon test-result를 통해 테스트 결과를 요약하여 로컬 개발과 지속적 통합(Continuous Integration, CI) 환경에서 실패를 확인할 수 있다. 개발자는 코드 변경 후 관련 테스트를 실행해야 하며, CI 파이프라인(CI Pipeline)은 커밋(Commit), 병합 요청(Merge Request), 릴리스 후보(Release Candidate)에 대해 패키지를 다시 빌드하고 테스트 스위트를 자동으로 실행할 수 있다.

지속적 통합(Continuous Integration)은 단위 테스트를 간헐적인 개발자 활동에서 지속적인 품질 게이트(Quality Gate)로 전환한다. 빌드 서버(Build Server)는 지원 환경에서 패키지를 컴파일하고 gtest를 실행하며 실패 결과를 검사하고 검증된 동작을 위반하는 변경을 차단할 수 있다. 정적 분석(Static Analysis), 형식 검사(Formatting Check), 통합 테스트, 시스템 테스트와 결합하면 단위 테스트는 상위 수준 검증을 대체하는 것이 아니라 광범위한 검증 전략의 빠른 내부 계층을 제공한다.

코드 커버리지(Code Coverage) 측정은 거의 실행되지 않는 로직을 식별하는 데 도움이 될 수 있지만, 커버리지 비율을 정확성의 증명으로 간주해서는 안 된다. 특정 코드 라인이 실행되었다고 해서 의미 있는 조건이 검증되었다는 의미는 아니다. 가치가 높은 테스트는 요구사항, 경계 조건, 장애 모드(Failure Mode), 상태 전이, 안전 관련 의사결정에 집중한다. 커버리지는 테스트되지 않은 영역을 발견하고 추가적인 엔지니어링 검토를 유도하는 근거로 활용할 때 가장 유용하다.

자율이동로봇(Autonomous Mobile Robot, AMR)의 경우 단위 테스트를 통해 속도 제한, 임무 검증, 내비게이션 상태 전이, 도킹 조건, 매개변수 범위, 진단 임계값을 검증할 수 있다. 매니퓰레이터(Manipulator) 소프트웨어는 관절 제약조건(Joint Constraint), 파지 상태 로직(Grasp-State Logic), 명령 검증을 테스트할 수 있으며, 피지컬 AI 노드(Physical AI Node)는 모든 테스트에서 완전한 학습 모델을 실행하지 않고도 전처리, 추론 결과 해석, 신뢰도 규칙(Confidence Rule), 대체 동작 결정(Fallback Decision)을 검증할 수 있다.

따라서 강건한 ROS 2 테스트 아키텍처(Robust ROS 2 Testing Architecture)는 테스트 가능한 패키지 구조, GoogleTest 어설션과 픽스처, ament_cmake_gtest 통합, 결정론적 로직, 의존성 주입, 제어된 ROS 컨텍스트, 장애 테스트, 동시성 검증, colcon과 CI를 이용한 자동화 실행을 결합한다. 단위 테스트를 구현 이후에 추가하는 활동이 아니라 노드 설계의 일부로 구성하면 개발자는 회귀 오류(Regression)를 조기에 발견하고 복잡한 로봇 및 피지컬 AI 소프트웨어를 더 높은 신뢰성을 유지하면서 지속적으로 발전시킬 수 있다.

##  

## 03.10 Node Performance Profiling: ros2.tracing [w/Code]

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

Performance profiling in ROS 2 examines how node execution consumes time and computing resources under realistic workloads. A robot may function correctly while still suffering from excessive callback latency, scheduling jitter, communication delay, or CPU contention. Profiling converts these timing problems into measurable evidence, allowing engineers to identify where execution time is spent and whether software behavior satisfies the latency and throughput requirements of the robotic system.

Traditional logging is useful for recording events but is limited for detailed timing analysis because adding many log statements can disturb execution and still provide only partial ordering information. ROS 2 tracing provides instrumentation designed to capture runtime events with relatively low overhead. Trace data can reconstruct callback execution, message flow, executor activity, and other middleware-related behavior without requiring developers to manually insert timestamps throughout application code.

The ros2_tracing ecosystem integrates ROS 2 instrumentation with tracing infrastructure such as the Linux Trace Toolkit next generation, commonly known as LTTng. Instrumented ROS 2 layers emit tracepoints when significant events occur. A tracing session records these events with timestamps and contextual information, producing a timeline that can later be analyzed without continuously printing diagnostic information to the console during robot operation.

Tracepoints are predefined instrumentation locations representing events of interest. They can describe node creation, publisher and subscription initialization, callback registration, callback execution, message publication, or other runtime activities depending on the available instrumentation. Because tracepoints are associated with specific events, engineers can reconstruct relationships between ROS entities and execution behavior rather than observing only aggregate CPU utilization.

Callback duration is one of the most useful measurements obtained from tracing. By observing callback start and end events, engineers can determine how long sensor processing, timers, services, or other callbacks occupy execution resources. A callback that occasionally takes much longer than its normal duration may produce control jitter or queue buildup even when its average execution time appears acceptable. Distribution and worst-case behavior are therefore important.

Scheduling latency should be distinguished from callback execution time. A message may become available at one time but its callback may not start until later because an executor thread is busy or another callback prevents execution. Measuring only the callback body would miss this waiting period. Tracing can help identify whether excessive latency originates from the algorithm itself or from executor scheduling, callback-group restrictions, synchronization, or resource contention.

Executor configuration strongly influences observed timing. A SingleThreadedExecutor can create predictable serialization but allows one long callback to delay all other ready work. A MultiThreadedExecutor can reduce some waiting by executing eligible callbacks concurrently, yet mutex contention or MutuallyExclusive callback groups may still serialize important operations. Trace timelines make these relationships visible and help verify whether additional threads actually improve execution.

End-to-end latency is often more meaningful than the execution time of one node. A perception pipeline may pass data through acquisition, preprocessing, inference, filtering, and decision nodes before producing an actuator command. Each stage can contribute computation, communication, queueing, and scheduling delay. Tracing allows engineers to correlate events across components and analyze the complete path from sensor input to resulting system response.

Throughput is another important metric, especially for high-rate cameras, LiDAR, point clouds, and Physical AI pipelines. A node may process individual messages quickly but still fail to sustain the incoming rate when downstream stages or shared resources become saturated. Profiling should compare input rate, processing rate, queue behavior, and dropped or delayed data so bottlenecks can be identified before they cause unstable real-time behavior.

Jitter describes variation in timing rather than simply large average latency. Control timers, localization updates, or sensor-processing callbacks may execute with an acceptable mean period while individual intervals vary significantly. Excessive jitter can degrade control quality, synchronization, and sensor fusion. Trace analysis should therefore examine timing distributions, minimum and maximum intervals, percentiles, and outliers instead of relying only on average values.

CPU utilization provides useful context but does not directly explain ROS 2 execution behavior. High CPU usage may indicate legitimate computation, inefficient algorithms, excessive copying, or too many active threads. Conversely, low overall CPU utilization can coexist with severe callback delays if a critical executor thread is blocked. Profiling should combine system-level resource measurements with ROS-aware tracing rather than interpreting processor utilization in isolation.

Memory behavior can also affect timing performance. Frequent dynamic allocation, large message copies, cache misses, and memory pressure may introduce unpredictable delays. Intra-process communication and appropriate ownership strategies can reduce copying for large data such as images and point clouds, but their benefits should be measured under the actual architecture. Profiling provides evidence for deciding whether communication optimization materially improves end-to-end performance.

Synchronization overhead becomes visible when multi-threaded nodes spend significant time waiting for locks or shared resources. A coarse mutex protecting a large state object may serialize callbacks despite multiple executor threads. Shortening critical sections, using immutable snapshots, separating ownership, or restructuring data flow can reduce contention. Optimization should target measured waiting relationships rather than adding concurrency mechanisms without evidence.

Tracing should be performed under representative system load. A node tested alone on a development computer may show excellent latency but behave differently when perception, navigation, diagnostics, visualization, recording, and networking operate simultaneously. CPU frequency behavior, memory pressure, GPU activity, storage I/O, and competing processes can all influence timing. Representative workload testing is therefore essential for meaningful performance conclusions.

Profiling should begin with a defined performance question rather than collecting traces without a purpose. Engineers may ask why a control timer misses its expected period, why camera-to-command latency increases during inference, or why navigation becomes unstable under high sensor load. A focused question determines which events, nodes, callbacks, and time intervals should be examined and prevents large trace datasets from becoming difficult to interpret.

A useful optimization workflow measures a baseline before changing the implementation. After identifying a bottleneck, engineers can modify callback structure, executor configuration, message ownership, algorithm design, synchronization, or deployment boundaries and then repeat the same measurement. Comparing traces before and after the change demonstrates whether the optimization improved the intended metric or merely shifted latency elsewhere in the system.

Tracing overhead must also be considered. Instrumentation is intended to minimize disturbance, but collecting events still consumes processing, memory, and storage resources. Very high event rates or long recording sessions can produce large datasets and may influence constrained platforms. Engineers should select appropriate tracing duration and event scope, and performance claims should account for the measurement configuration used during data collection.

Custom tracepoints can extend analysis when built-in ROS 2 events do not expose enough application-specific information. A developer may instrument the start and completion of perception stages, planner phases, model inference, or command-generation steps. Application tracepoints can then be correlated with ROS 2 executor and communication events, creating a unified timeline from middleware activity to domain-specific computation.

Visualization and post-processing transform raw trace events into useful engineering information. Timelines can reveal overlapping callbacks, idle periods, delayed execution, and long-running operations, while scripts can calculate latency distributions and callback statistics. The objective is not simply to collect trace files but to derive repeatable measurements that can be compared across software versions, configurations, hardware platforms, and workloads.

Performance regression testing can incorporate selected profiling metrics into continuous engineering workflows. A software change that preserves functional tests may still increase callback duration or end-to-end latency. Repeating controlled benchmarks and comparing timing metrics can detect these regressions before deployment. Performance thresholds should be based on system requirements and realistic variability rather than overly rigid values derived from a single test run.

For an AMR, tracing can analyze LiDAR-to-localization latency, navigation callback timing, control-loop jitter, and executor contention during mapping or obstacle avoidance. Manipulators can profile state feedback and command generation, while Physical AI systems can correlate sensor arrival, preprocessing, GPU inference, reasoning, and action output. These measurements reveal whether intelligent functions fit within the timing budget of physical interaction.

A robust ROS 2 performance-profiling architecture therefore combines ros2_tracing instrumentation, executor-aware timing analysis, end-to-end latency measurement, throughput and jitter evaluation, system-resource monitoring, representative workloads, and repeatable baseline comparisons. Profiling turns performance optimization from intuition into evidence, enabling complex robotic and Physical AI nodes to evolve while maintaining observable and measurable execution behavior.

ROS 2의 성능 프로파일링(Performance Profiling)은 실제적인 작업부하(Workload)에서 노드 실행이 시간과 컴퓨팅 자원을 어떻게 소비하는지를 분석한다. 로봇은 기능적으로 정상 동작하면서도 과도한 콜백 지연시간(Callback Latency), 스케줄링 지터(Scheduling Jitter), 통신 지연(Communication Delay), CPU 경합(CPU Contention)을 겪을 수 있다. 프로파일링은 이러한 시간적 문제를 측정 가능한 근거로 변환하여 실행시간이 어디에서 소비되는지, 그리고 소프트웨어 동작이 로봇 시스템의 지연시간 및 처리량 요구사항을 충족하는지를 엔지니어가 확인할 수 있도록 한다.

전통적인 로깅(Logging)은 이벤트를 기록하는 데 유용하지만, 많은 로그 문장을 추가하면 실행 자체에 영향을 줄 수 있고 이벤트의 전체적인 순서 관계도 부분적으로만 제공하므로 세부적인 타이밍 분석에는 한계가 있다. ROS 2 추적(Tracing)은 비교적 낮은 오버헤드(Overhead)로 런타임 이벤트를 수집하도록 설계된 계측(Instrumentation) 기능을 제공한다. 추적 데이터를 이용하면 응용 프로그램 코드 전체에 타임스탬프를 직접 삽입하지 않고도 콜백 실행, 메시지 흐름, 실행기(Executor) 활동 및 기타 미들웨어 관련 동작을 재구성할 수 있다.

ros2_tracing 생태계는 ROS 2 계측 기능을 일반적으로 LTTng라고 부르는 차세대 리눅스 추적 툴킷(Linux Trace Toolkit next generation)과 같은 추적 인프라(Tracing Infrastructure)에 통합한다. 계측된 ROS 2 계층은 중요한 이벤트가 발생할 때 추적 지점(Tracepoint)을 생성한다. 추적 세션(Tracing Session)은 이러한 이벤트를 타임스탬프와 상황 정보(Contextual Information)와 함께 기록하여, 로봇 운용 중 지속적으로 진단 정보를 콘솔에 출력하지 않고도 이후 분석할 수 있는 시간축(Timeline)을 생성한다.

추적 지점(Tracepoint)은 관심 대상 이벤트를 나타내는 사전에 정의된 계측 위치이다. 사용 가능한 계측 기능에 따라 노드 생성, 발행자(Publisher)와 구독(Subscription) 초기화, 콜백 등록, 콜백 실행, 메시지 발행 및 기타 런타임 활동을 나타낼 수 있다. 추적 지점은 특정 이벤트와 연결되므로 엔지니어는 단순한 전체 CPU 사용률만 관찰하는 것이 아니라 ROS 엔티티(Entity)와 실제 실행 동작 사이의 관계를 재구성할 수 있다.

콜백 실행시간(Callback Duration)은 추적을 통해 얻을 수 있는 가장 유용한 측정값 중 하나이다. 콜백 시작 및 종료 이벤트를 관찰하면 센서 처리, 타이머(Timer), 서비스(Service) 또는 다른 콜백이 실행 자원을 얼마나 오랫동안 점유하는지 확인할 수 있다. 특정 콜백의 평균 실행시간이 허용 가능한 수준이더라도 간헐적으로 평소보다 훨씬 오래 실행되면 제어 지터(Control Jitter)나 대기열 누적(Queue Buildup)을 발생시킬 수 있다. 따라서 실행시간 분포와 최악 조건 동작(Worst-Case Behavior)을 함께 분석하는 것이 중요하다.

스케줄링 지연시간(Scheduling Latency)은 콜백 실행시간과 구분해야 한다. 메시지가 특정 시점에 사용 가능해졌더라도 실행기 스레드가 사용 중이거나 다른 콜백이 실행을 방해하면 실제 콜백 시작 시점은 늦어질 수 있다. 콜백 함수 자체의 실행시간만 측정하면 이러한 대기시간을 확인할 수 없다. 추적을 사용하면 과도한 지연이 알고리즘 자체에서 발생하는지, 실행기 스케줄링, 콜백 그룹(Callback Group) 제약, 동기화(Synchronization), 자원 경합(Resource Contention)에서 발생하는지 분석할 수 있다.

실행기 구성(Executor Configuration)은 관측되는 타이밍에 큰 영향을 미친다. 단일 스레드 실행기(SingleThreadedExecutor)는 예측 가능한 직렬 실행을 제공하지만 하나의 장시간 콜백이 다른 모든 준비된 작업을 지연시킬 수 있다. 다중 스레드 실행기(MultiThreadedExecutor)는 실행 가능한 콜백을 동시에 처리하여 일부 대기시간을 줄일 수 있지만, 뮤텍스 경합(Mutex Contention)이나 상호 배타적 콜백 그룹(MutuallyExclusive Callback Group)이 중요한 작업을 여전히 직렬화할 수 있다. 추적 시간축은 이러한 관계를 시각적으로 확인하고 추가 스레드가 실제로 실행 성능을 개선하는지 검증하는 데 도움이 된다.

종단간 지연시간(End-to-End Latency)은 하나의 노드 실행시간보다 더 중요한 경우가 많다. 인지 파이프라인(Perception Pipeline)은 액추에이터 명령(Actuator Command)을 생성하기 전에 데이터 획득, 전처리, 추론(Inference), 필터링, 의사결정 노드를 거쳐 데이터를 전달할 수 있다. 각 단계는 계산, 통신, 대기열 처리(Queueing), 스케줄링 지연을 발생시킬 수 있다. 추적을 사용하면 여러 구성요소의 이벤트를 연관시켜 센서 입력부터 최종 시스템 응답까지 전체 경로를 분석할 수 있다.

처리량(Throughput) 역시 중요한 성능 지표이며, 특히 고주기 카메라, LiDAR, 포인트 클라우드(Point Cloud), 피지컬 AI 파이프라인(Physical AI Pipeline)에서 중요하다. 하나의 노드가 개별 메시지를 빠르게 처리하더라도 하위 단계나 공유 자원이 포화되면 입력 데이터 속도를 지속적으로 처리하지 못할 수 있다. 프로파일링은 입력률, 처리율, 대기열 동작, 손실되거나 지연된 데이터를 비교하여 병목(Bottleneck)이 불안정한 실시간 동작을 발생시키기 전에 식별할 수 있도록 해야 한다.

지터(Jitter)는 단순히 평균 지연시간이 큰 것이 아니라 타이밍의 변동성을 나타낸다. 제어 타이머, 위치추정 갱신(Localization Update), 센서 처리 콜백은 평균 주기가 허용 가능한 수준이더라도 개별 실행 간격이 크게 달라질 수 있다. 과도한 지터는 제어 품질, 동기화, 센서 융합(Sensor Fusion)을 저하시킬 수 있다. 따라서 추적 분석에서는 평균값에만 의존하지 않고 타이밍 분포, 최소 및 최대 간격, 백분위수(Percentile), 이상치(Outlier)를 함께 분석해야 한다.

CPU 사용률(CPU Utilization)은 유용한 상황 정보를 제공하지만 ROS 2 실행 동작을 직접적으로 설명하지는 않는다. 높은 CPU 사용률은 정상적인 고부하 계산, 비효율적인 알고리즘, 과도한 데이터 복사 또는 지나치게 많은 활성 스레드를 의미할 수 있다. 반대로 전체 CPU 사용률이 낮더라도 중요한 실행기 스레드가 차단되면 심각한 콜백 지연이 발생할 수 있다. 따라서 프로파일링에서는 프로세서 사용률만 단독으로 해석하지 않고 시스템 수준 자원 측정과 ROS 인식 추적(ROS-Aware Tracing)을 결합해야 한다.

메모리 동작(Memory Behavior)도 시간 성능에 영향을 줄 수 있다. 빈번한 동적 메모리 할당(Dynamic Allocation), 대용량 메시지 복사, 캐시 미스(Cache Miss), 메모리 압박(Memory Pressure)은 예측하기 어려운 지연을 발생시킬 수 있다. 프로세스 내부 통신(Intra-Process Communication)과 적절한 소유권 전략(Ownership Strategy)은 이미지나 포인트 클라우드와 같은 대용량 데이터의 복사를 줄일 수 있지만, 실제 아키텍처에서 그 효과를 측정해야 한다. 프로파일링은 통신 최적화가 실제 종단간 성능을 의미 있게 개선하는지를 판단하는 근거를 제공한다.

다중 스레드 노드가 잠금(Lock)이나 공유 자원을 기다리는 데 상당한 시간을 소비하면 동기화 오버헤드(Synchronization Overhead)가 추적 결과에 나타난다. 대규모 상태 객체를 보호하는 하나의 거친 단위 뮤텍스(Coarse Mutex)는 여러 실행기 스레드가 존재하더라도 콜백을 직렬화할 수 있다. 임계 구역(Critical Section)을 단축하고, 불변 스냅샷(Immutable Snapshot)을 사용하고, 소유권을 분리하거나 데이터 흐름을 재구성하면 경합을 줄일 수 있다. 최적화는 근거 없이 동시성 메커니즘을 추가하는 것이 아니라 실제 측정된 대기 관계를 대상으로 수행해야 한다.

추적은 대표적인 시스템 부하(Representative System Load)에서 수행해야 한다. 개발 컴퓨터에서 노드를 단독으로 테스트하면 매우 우수한 지연시간을 보이더라도 인지, 내비게이션, 진단, 시각화, 기록, 네트워킹이 동시에 동작하면 다른 결과가 나타날 수 있다. CPU 주파수 동작, 메모리 압박, GPU 활동, 저장장치 입출력(Storage I/O), 경쟁 프로세스가 모두 타이밍에 영향을 줄 수 있다. 따라서 의미 있는 성능 결론을 얻으려면 실제 운용을 대표하는 작업부하 테스트가 필수적이다.

프로파일링은 목적 없이 추적 데이터를 수집하는 것이 아니라 명확하게 정의된 성능 질문(Performance Question)에서 시작해야 한다. 엔지니어는 제어 타이머가 예상 주기를 지키지 못하는 이유, 추론 중 카메라 입력에서 명령 출력까지의 지연시간이 증가하는 이유, 높은 센서 부하에서 내비게이션이 불안정해지는 이유를 질문할 수 있다. 명확한 질문은 분석해야 할 이벤트, 노드, 콜백, 시간 구간을 결정하고 대규모 추적 데이터가 해석하기 어려워지는 문제를 방지한다.

효과적인 최적화 워크플로(Optimization Workflow)는 구현을 변경하기 전에 기준 성능(Baseline)을 측정하는 것에서 시작한다. 병목을 식별한 후 엔지니어는 콜백 구조, 실행기 구성, 메시지 소유권, 알고리즘 설계, 동기화 또는 배포 경계(Deployment Boundary)를 변경하고 동일한 측정을 반복할 수 있다. 변경 전후의 추적 결과를 비교하면 최적화가 목표 성능 지표를 실제로 개선했는지 아니면 단순히 지연 문제를 시스템의 다른 위치로 이동시켰는지 확인할 수 있다.

추적 오버헤드(Tracing Overhead)도 고려해야 한다. 계측 기능은 실행 간섭을 최소화하도록 설계되지만 이벤트 수집에는 여전히 처리 자원, 메모리, 저장 공간이 사용된다. 매우 높은 이벤트 발생률이나 장시간 기록은 대규모 데이터셋을 생성하고 자원이 제한된 플랫폼의 동작에 영향을 줄 수 있다. 따라서 엔지니어는 적절한 추적 시간과 이벤트 범위를 선택해야 하며, 성능 평가 결과에는 데이터 수집에 사용된 측정 구성을 함께 고려해야 한다.

기본 ROS 2 이벤트만으로 응용 프로그램별 정보를 충분히 확인할 수 없는 경우 사용자 정의 추적 지점(Custom Tracepoint)을 통해 분석 범위를 확장할 수 있다. 개발자는 인지 단계, 플래너 단계(Planner Phase), 모델 추론, 명령 생성 단계의 시작과 완료를 계측할 수 있다. 이러한 응용 프로그램 추적 지점을 ROS 2 실행기 및 통신 이벤트와 연관시키면 미들웨어 활동부터 도메인 특화 계산(Domain-Specific Computation)까지 연결되는 통합 시간축을 구성할 수 있다.

시각화(Visualization)와 후처리(Post-Processing)는 원시 추적 이벤트를 유용한 엔지니어링 정보로 변환한다. 시간축을 통해 중첩되는 콜백, 유휴 구간(Idle Period), 지연된 실행, 장시간 실행 작업을 확인할 수 있으며, 분석 스크립트는 지연시간 분포와 콜백 통계를 계산할 수 있다. 목적은 단순히 추적 파일을 수집하는 것이 아니라 소프트웨어 버전, 구성, 하드웨어 플랫폼, 작업부하 사이에서 비교할 수 있는 반복 가능한 성능 측정값을 도출하는 것이다.

성능 회귀 테스트(Performance Regression Testing)는 선택된 프로파일링 지표를 지속적인 엔지니어링 워크플로에 포함할 수 있다. 기능 테스트를 모두 통과하는 소프트웨어 변경도 콜백 실행시간이나 종단간 지연시간을 증가시킬 수 있다. 제어된 벤치마크(Benchmark)를 반복하고 시간 성능 지표를 비교하면 배포 전에 이러한 회귀를 발견할 수 있다. 성능 임계값은 단일 테스트에서 얻은 지나치게 엄격한 값이 아니라 시스템 요구사항과 현실적인 변동성을 기반으로 설정해야 한다.

자율이동로봇(Autonomous Mobile Robot, AMR)에서는 추적을 통해 LiDAR에서 위치추정까지의 지연시간, 내비게이션 콜백 타이밍, 제어 루프 지터(Control-Loop Jitter), 매핑(Mapping) 또는 장애물 회피 중 실행기 경합을 분석할 수 있다. 매니퓰레이터(Manipulator)는 상태 피드백과 명령 생성을 프로파일링할 수 있으며, 피지컬 AI 시스템(Physical AI System)은 센서 입력, 전처리, GPU 추론, 추론 기반 판단(Reasoning), 행동 출력(Action Output)의 시간 관계를 분석할 수 있다. 이러한 측정을 통해 지능형 기능이 물리적 상호작용에 요구되는 타이밍 예산(Timing Budget) 안에서 동작하는지를 판단할 수 있다.

따라서 강건한 ROS 2 성능 프로파일링 아키텍처(Robust ROS 2 Performance-Profiling Architecture)는 ros2_tracing 계측, 실행기 인식 타이밍 분석(Executor-Aware Timing Analysis), 종단간 지연시간 측정, 처리량과 지터 평가, 시스템 자원 모니터링, 대표 작업부하, 반복 가능한 기준 성능 비교를 결합한다. 프로파일링은 성능 최적화를 직관에 의존하는 작업에서 측정 근거에 기반한 엔지니어링 과정으로 전환하며, 복잡한 로봇 및 피지컬 AI 노드가 관측 가능하고 측정 가능한 실행 동작을 유지하면서 발전할 수 있도록 한다.
