**Volume 05 Robot Middleware and ROS2**


# 04. ROS2 Communication

##  

## 04.01 Topic Communication: Publisher / Subscriber Pattern [w/Code]

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 topic communication is the primary mechanism for continuously exchanging data between distributed nodes. It follows the publisher--subscriber pattern, in which a publisher produces typed messages on a named topic while one or more subscribers independently receive them. This loose coupling allows sensing, perception, planning, control, diagnostics, and visualization components to evolve without direct dependencies between individual nodes.

Unlike a direct function call, topic communication does not require the publisher to know which nodes consume its data. A camera driver may publish image frames while perception, recording, visualization, and monitoring nodes subscribe simultaneously. The publisher continues operating even when subscribers are added or removed, making the pattern particularly suitable for modular robot architectures whose software components may be deployed across different processes or computers.

Every ROS 2 topic is associated with a defined message type that establishes the data contract between publishers and subscribers. Standard interfaces include messages such as \`sensor_msgs/msg/Image\`, \`sensor_msgs/msg/LaserScan\`, \`sensor_msgs/msg/Imu\`, \`nav_msgs/msg/Odometry\`, and \`geometry_msgs/msg/Twist\`. Applications can also define custom message types when domain-specific information must be exchanged, allowing interfaces to remain explicit and strongly structured.

A publisher is created by a ROS 2 node with a topic name, message type, and Quality of Service configuration. During execution, the node constructs a message, populates its fields, and invokes the publish operation. Publishing is conceptually asynchronous: the application does not normally wait for subscribers to finish processing the message. This property enables sensor and control pipelines to continue producing data according to their own execution schedules.

A subscriber registers a callback associated with the same topic and compatible message type. When incoming data becomes available, the ROS 2 executor schedules the corresponding callback according to the node\'s execution configuration. The callback can then perform filtering, state estimation, perception, visualization, logging, or other processing. Consequently, communication behavior is closely related to executor design, callback groups, and concurrency mechanisms discussed in node architecture.

ROS 2 topic communication is implemented through the ROS middleware abstraction layer and commonly uses DDS-based publish--subscribe communication underneath. ROS 2 therefore avoids requiring a centralized ROS master for ordinary topic discovery and data exchange. Participating endpoints discover compatible publishers and subscribers through middleware mechanisms, after which communication can occur locally or across a network according to the selected middleware implementation and configuration.

Quality of Service, or QoS, is fundamental to the publisher--subscriber relationship because communication compatibility involves more than matching topic names and message types. Reliability determines whether delivery is attempted reliably or on a best-effort basis, while durability influences whether previously published data can be available to later subscribers. History and depth determine how samples are retained when producers and consumers operate at different rates.

The correct QoS configuration depends strongly on the semantics of the data. High-rate camera, LiDAR, or other sensor streams may favor best-effort delivery because receiving the newest sample can be more valuable than retransmitting an obsolete one. Configuration or state information may instead require reliable delivery and appropriate durability. ROS 2 therefore allows communication behavior to be adapted to different robotic workloads without changing the fundamental topic abstraction.

Topic frequency is another important architectural property. A mobile robot may publish camera images at tens of frames per second, IMU measurements at hundreds of hertz, localization estimates at a lower rate, and diagnostic information much less frequently. Publishers should avoid transmitting unnecessary data because serialization, middleware processing, network transfer, deserialization, callback execution, and memory operations collectively consume CPU, memory bandwidth, and network capacity.

Topic names provide a logical addressing mechanism for robot data flows. Names such as \`/camera/image_raw\`, \`/scan\`, \`/odom\`, or \`/cmd_vel\` express the role of each communication channel independently of the physical location of its publishers and subscribers. Namespaces and remapping extend this mechanism, enabling reusable nodes to be instantiated for different robots, sensors, or subsystems without modifying their source code.

This abstraction becomes especially important in multi-robot systems. Instead of allowing every robot to publish indistinguishable global topic names, namespaces can organize communication into structures such as \`/robot_01/odom\` and \`/robot_02/odom\`. Fleet-level software can selectively subscribe to relevant information while robot-local components operate within their own namespace. The volume structure therefore treats multi-robot namespace design as a dedicated communication topic later in the chapter.

ROS 2 also supports communication between nodes running in the same process. When composition and intra-process communication are enabled appropriately, message transfer can avoid some serialization and copying operations that would otherwise occur through conventional inter-process middleware paths. This is valuable for high-bandwidth perception pipelines in which large images, point clouds, or AI inference tensors pass rapidly between tightly connected processing stages.

Large messages require particular attention because communication cost grows with payload size and publication rate. Image streams and three-dimensional point clouds can quickly dominate network bandwidth and memory traffic. Robot architects therefore need to consider message representation, transport topology, publication frequency, QoS, intra-process communication, and unnecessary data duplication. The chapter structure consequently separates large-message transfer and serialization optimization into dedicated sections.

Publisher and subscriber rates do not have to be identical. A publisher can produce data faster than a subscriber processes it, causing queued samples to accumulate or older samples to be discarded according to the configured history and depth policies. Conversely, a subscriber may spend much of its time waiting when data arrives slowly. Designing a stable ROS 2 system therefore requires understanding both communication frequency and callback processing capacity.

The publisher--subscriber model naturally supports one-to-many, many-to-one, and many-to-many data flows. A single localization publisher can feed navigation, visualization, monitoring, and logging nodes, while multiple distributed sensors can publish information consumed by fusion components. This scalability is one reason topics are well suited to continuous state and sensor streams, whereas services and actions address different interaction semantics elsewhere in the ROS 2 communication architecture.

Topics should not be treated as a universal replacement for every software interaction. They are most appropriate when data represents an ongoing stream or asynchronously changing state and when the producer does not require an immediate response from a specific consumer. Request--response operations are better represented by services, while long-running tasks requiring feedback, progress monitoring, and cancellation are generally represented by actions. These mechanisms complement rather than replace topics.

Operational debugging begins by observing the ROS graph and topic behavior. ROS 2 command-line tools can list available topics, inspect message types, display published messages, measure publication frequency, and examine communication characteristics. These observations help identify incorrect topic names, missing publishers, incompatible message types, unexpected rates, QoS mismatches, and overloaded processing pipelines before deeper middleware or network diagnostics become necessary.

In production robots, topic design becomes an architectural contract rather than merely a programming convenience. Stable message definitions, predictable naming conventions, appropriate QoS profiles, controlled publication rates, clear ownership, and measurable latency make distributed components easier to integrate and maintain. Poorly designed topics can instead create hidden coupling, excessive bandwidth consumption, stale data processing, and difficult-to-diagnose behavior across the robot software stack.

For autonomous mobile robots and Physical AI systems, topic communication forms the continuous information fabric connecting sensors, localization, perception, world representation, AI inference, planning, control, diagnostics, and fleet interfaces. A well-designed publisher--subscriber architecture allows these components to remain independently deployable while participating in a coherent distributed system, providing the communication foundation on which more advanced ROS 2 services, actions, security, and real-time mechanisms can be built.

ROS 2 토픽 통신(topic communication)은 분산 노드(distributed node) 사이에서 데이터를 지속적으로 교환하기 위한 핵심 메커니즘이다. 이는 발행자-구독자 패턴(publisher--subscriber pattern)을 따르며, 발행자(publisher)는 이름이 지정된 토픽(topic)에 형식화된 메시지를 생성하고 하나 이상의 구독자(subscriber)는 이를 독립적으로 수신한다. 이러한 느슨한 결합(loose coupling)을 통해 센싱(sensing), 인지(perception), 계획(planning), 제어(control), 진단(diagnostics), 시각화(visualization) 구성요소를 개별 노드 사이의 직접적인 종속성 없이 발전시킬 수 있다.

직접 함수 호출(direct function call)과 달리 토픽 통신에서는 발행자가 어떤 노드가 자신의 데이터를 사용하는지 알 필요가 없다. 카메라 드라이버(camera driver)가 영상 프레임(image frame)을 발행하면 인지, 기록, 시각화 및 모니터링 노드가 동시에 이를 구독할 수 있다. 구독자가 추가되거나 제거되어도 발행자는 계속 동작하므로, 이 패턴은 서로 다른 프로세스(process)나 컴퓨터에 소프트웨어 구성요소를 배치할 수 있는 모듈형 로봇 아키텍처(modular robot architecture)에 특히 적합하다.

모든 ROS 2 토픽에는 발행자와 구독자 사이의 데이터 계약(data contract)을 정의하는 메시지 형식(message type)이 연결된다. 표준 인터페이스(standard interface)에는 \`sensor_msgs/msg/Image\`, \`sensor_msgs/msg/LaserScan\`, \`sensor_msgs/msg/Imu\`, \`nav_msgs/msg/Odometry\`, \`geometry_msgs/msg/Twist\`와 같은 메시지가 포함된다. 도메인 특화 정보(domain-specific information)를 교환해야 하는 경우에는 사용자 정의 메시지 형식(custom message type)을 정의하여 명시적이고 강력하게 구조화된 인터페이스를 구성할 수 있다.

발행자(publisher)는 ROS 2 노드에서 토픽 이름(topic name), 메시지 형식(message type), 서비스 품질(Quality of Service, QoS) 설정과 함께 생성된다. 실행 중 노드는 메시지를 생성하고 필드(field)를 채운 다음 발행 연산(publish operation)을 호출한다. 발행은 개념적으로 비동기식(asynchronous)이며, 일반적으로 응용 프로그램(application)은 구독자가 메시지 처리를 완료할 때까지 기다리지 않는다. 따라서 센서 및 제어 파이프라인(sensor and control pipeline)은 각각의 실행 주기에 따라 지속적으로 데이터를 생성할 수 있다.

구독자(subscriber)는 동일한 토픽과 호환되는 메시지 형식에 연결된 콜백(callback)을 등록한다. 수신 데이터가 도착하면 ROS 2 실행기(executor)는 노드의 실행 설정에 따라 해당 콜백을 스케줄링(scheduling)한다. 콜백은 필터링(filtering), 상태 추정(state estimation), 인지, 시각화, 로깅(logging) 등의 처리를 수행할 수 있다. 따라서 통신 동작은 노드 아키텍처에서 다루는 실행기 설계(executor design), 콜백 그룹(callback group), 동시성 메커니즘(concurrency mechanism)과 밀접하게 관련된다.

ROS 2 토픽 통신은 ROS 미들웨어 추상화 계층(ROS middleware abstraction layer)을 통해 구현되며, 일반적으로 내부적으로 DDS 기반 발행-구독 통신(DDS-based publish--subscribe communication)을 사용한다. 따라서 ROS 2의 일반적인 토픽 검색(discovery)과 데이터 교환에는 중앙 집중형 ROS 마스터(centralized ROS master)가 필요하지 않다. 참여 엔드포인트(endpoint)는 미들웨어 메커니즘을 통해 호환되는 발행자와 구독자를 검색하며, 이후 선택된 미들웨어 구현 및 설정에 따라 로컬 또는 네트워크를 통해 통신한다.

서비스 품질(Quality of Service, QoS)은 발행자-구독자 관계의 핵심 요소이다. 통신 호환성은 단순히 토픽 이름과 메시지 형식을 일치시키는 것 이상을 의미하기 때문이다. 신뢰성(reliability)은 신뢰성 있는 전달 또는 최선형 전달(best-effort delivery) 여부를 결정하며, 내구성(durability)은 이전에 발행된 데이터를 나중에 연결된 구독자가 받을 수 있는지에 영향을 준다. 이력(history)과 깊이(depth)는 생산자와 소비자가 서로 다른 속도로 동작할 때 메시지 샘플을 어떻게 유지할지를 결정한다.

적절한 QoS 설정은 데이터의 의미적 특성에 크게 좌우된다. 고속 카메라, LiDAR 또는 기타 센서 스트림(sensor stream)은 오래된 데이터를 재전송하는 것보다 최신 샘플을 수신하는 것이 더 중요할 수 있으므로 최선형 전달(best-effort delivery)이 적합할 수 있다. 반면 설정이나 상태 정보는 신뢰성 있는 전달(reliable delivery)과 적절한 내구성을 요구할 수 있다. ROS 2는 기본적인 토픽 추상화(topic abstraction)를 변경하지 않고도 다양한 로봇 작업 부하(robotic workload)에 맞게 통신 동작을 조정할 수 있다.

토픽 주기(topic frequency) 역시 중요한 아키텍처 특성이다. 모바일 로봇(mobile robot)은 카메라 영상을 초당 수십 프레임으로, 관성 측정 장치(Inertial Measurement Unit, IMU) 데이터를 수백 헤르츠로, 위치 추정(localization estimate)을 더 낮은 주기로, 진단 정보는 훨씬 낮은 빈도로 발행할 수 있다. 직렬화(serialization), 미들웨어 처리, 네트워크 전송, 역직렬화(deserialization), 콜백 실행, 메모리 연산이 CPU와 메모리 대역폭 및 네트워크 용량을 소비하므로 불필요한 데이터 전송은 피해야 한다.

토픽 이름(topic name)은 로봇 데이터 흐름(robot data flow)을 위한 논리적 주소 지정 메커니즘(logical addressing mechanism)을 제공한다. \`/camera/image_raw\`, \`/scan\`, \`/odom\`, \`/cmd_vel\`과 같은 이름은 발행자와 구독자의 물리적 위치와 관계없이 각 통신 채널의 역할을 나타낸다. 네임스페이스(namespace)와 재매핑(remapping)을 활용하면 소스 코드(source code)를 수정하지 않고도 재사용 가능한 노드를 서로 다른 로봇, 센서 또는 하위 시스템에 적용할 수 있다.

이러한 추상화는 다중 로봇 시스템(multi-robot system)에서 특히 중요하다. 모든 로봇이 구분되지 않는 전역 토픽 이름(global topic name)을 사용하는 대신 네임스페이스를 적용하여 \`/robot_01/odom\`, \`/robot_02/odom\`과 같은 구조로 통신을 구성할 수 있다. 플릿 수준 소프트웨어(fleet-level software)는 필요한 정보만 선택적으로 구독하고, 각 로봇의 로컬 구성요소는 자체 네임스페이스에서 동작할 수 있다. 따라서 이 장에서는 다중 로봇 네임스페이스 설계(multi-robot namespace design)를 별도의 통신 주제로 다룬다.

ROS 2는 동일한 프로세스에서 실행되는 노드 사이의 통신도 지원한다. 컴포지션(composition)과 프로세스 내부 통신(intra-process communication)을 적절하게 활성화하면 일반적인 프로세스 간 미들웨어 경로에서 발생하는 일부 직렬화 및 데이터 복사 작업을 줄일 수 있다. 이는 대용량 영상, 포인트 클라우드(point cloud), AI 추론 텐서(AI inference tensor)가 긴밀하게 연결된 처리 단계 사이를 빠르게 이동하는 고대역폭 인지 파이프라인(high-bandwidth perception pipeline)에서 특히 유용하다.

대용량 메시지(large message)는 페이로드 크기(payload size)와 발행 주기에 따라 통신 비용이 증가하므로 특별한 주의가 필요하다. 영상 스트림과 3차원 포인트 클라우드는 네트워크 대역폭과 메모리 트래픽을 빠르게 점유할 수 있다. 따라서 로봇 아키텍트(robot architect)는 메시지 표현(message representation), 전송 토폴로지(transport topology), 발행 주기, QoS, 프로세스 내부 통신, 불필요한 데이터 복제를 함께 고려해야 한다. 이에 따라 이 장에서는 대용량 메시지 전송과 직렬화 최적화를 별도의 절에서 다룬다.

발행자와 구독자의 처리 주기는 동일할 필요가 없다. 발행자가 구독자의 처리 속도보다 빠르게 데이터를 생성하면 설정된 이력(history) 및 깊이(depth) 정책에 따라 메시지 샘플이 큐(queue)에 누적되거나 오래된 샘플이 삭제될 수 있다. 반대로 데이터가 느리게 도착하면 구독자는 상당한 시간을 대기할 수 있다. 따라서 안정적인 ROS 2 시스템을 설계하려면 통신 주기와 콜백 처리 용량(callback processing capacity)을 함께 이해해야 한다.

발행자-구독자 모델은 자연스럽게 일대다(one-to-many), 다대일(many-to-one), 다대다(many-to-many) 데이터 흐름을 지원한다. 하나의 위치 추정 발행자가 내비게이션(navigation), 시각화, 모니터링, 로깅 노드에 데이터를 제공할 수 있으며, 여러 분산 센서가 발행한 정보를 센서 융합(sensor fusion) 구성요소가 수신할 수도 있다. 이러한 확장성은 토픽이 지속적인 상태 및 센서 스트림에 적합한 주요 이유이며, 서비스(service)와 액션(action)은 ROS 2 통신 아키텍처에서 서로 다른 상호작용 의미를 담당한다.

토픽은 모든 소프트웨어 상호작용을 대체하는 범용 메커니즘으로 간주해서는 안 된다. 토픽은 데이터가 지속적인 스트림 또는 비동기적으로 변화하는 상태를 나타내고, 생산자가 특정 소비자로부터 즉각적인 응답을 요구하지 않는 경우에 가장 적합하다. 요청-응답(request--response) 연산은 서비스(service)가 적합하며, 피드백, 진행 상태 모니터링, 취소 기능이 필요한 장시간 실행 작업(long-running task)은 일반적으로 액션(action)이 적합하다. 이들 통신 방식은 토픽을 대체하는 것이 아니라 상호 보완한다.

운영 환경의 디버깅(debugging)은 ROS 그래프(ROS graph)와 토픽 동작을 관찰하는 것에서 시작한다. ROS 2 명령줄 도구(command-line tool)를 사용하면 사용 가능한 토픽을 나열하고, 메시지 형식을 확인하고, 발행된 메시지를 표시하며, 발행 주기와 통신 특성을 측정할 수 있다. 이러한 관찰을 통해 심층적인 미들웨어 또는 네트워크 진단에 앞서 잘못된 토픽 이름, 누락된 발행자, 호환되지 않는 메시지 형식, 비정상적인 발행 주기, QoS 불일치, 과부하된 처리 파이프라인 등을 식별할 수 있다.

실제 운용 로봇(production robot)에서 토픽 설계는 단순한 프로그래밍 편의 기능을 넘어 아키텍처 계약(architectural contract)이 된다. 안정적인 메시지 정의, 예측 가능한 명명 규칙(naming convention), 적절한 QoS 프로파일(QoS profile), 제어된 발행 주기, 명확한 소유권, 측정 가능한 지연시간(latency)은 분산 구성요소의 통합과 유지보수를 용이하게 한다. 반대로 잘못 설계된 토픽은 숨겨진 결합(hidden coupling), 과도한 대역폭 소비, 오래된 데이터 처리, 로봇 소프트웨어 스택 전반에서 진단하기 어려운 동작을 초래할 수 있다.

자율이동로봇(Autonomous Mobile Robot, AMR)과 피지컬 AI(Physical AI) 시스템에서 토픽 통신은 센서, 위치 추정, 인지, 세계 표현(world representation), AI 추론(AI inference), 계획, 제어, 진단, 플릿 인터페이스(fleet interface)를 연결하는 지속적인 정보 전달 기반을 형성한다. 잘 설계된 발행자-구독자 아키텍처는 이러한 구성요소가 독립적으로 배포되면서도 하나의 일관된 분산 시스템에 참여하도록 하며, 고급 ROS 2 서비스, 액션, 보안 및 실시간 메커니즘을 구축하기 위한 통신 기반을 제공한다.

##  

## 04.02 Service Communication: Client / Server Sync Call [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 service communication provides a request--response mechanism for interactions in which one node asks another node to perform a specific operation and return a defined result. The client sends a typed request to a named service, while the server processes that request and produces a typed response. This model is appropriate for discrete operations rather than the continuous data streams handled by topic-based publisher--subscriber communication.

A service interface defines both sides of the communication contract. Unlike a topic message, which represents a single data structure, a service definition contains a request section and a response section. The request specifies the information required to perform an operation, while the response represents its result, status, or returned data. This explicit interface allows independently implemented ROS 2 nodes to interact through a predictable and strongly typed software boundary.

The client represents the requesting side of the service pattern. It creates a request object according to the service type, populates the required fields, and sends the request to the service server. Conceptually, the client expects one response for each request. Typical robot operations include querying configuration information, resetting internal state, enabling a subsystem, requesting a calculation, changing an operating mode, or initiating a short deterministic operation.

The server represents the processing side and advertises a named service within the ROS 2 graph. When a request arrives, the server\'s callback receives the request data, executes the required logic, populates a response object, and returns the result. Separating the caller from the implementation allows the service provider to change its internal algorithms or hardware interaction while maintaining the same externally visible service interface.

Synchronous service calls express the familiar blocking request--response concept: the calling logic sends a request and waits until the corresponding response becomes available. This programming model can be straightforward because subsequent processing begins only after the requested result has been obtained. However, blocking must be used carefully in distributed robot software because communication, scheduling, server processing, or network delays can extend the waiting period unexpectedly.

ROS 2 execution architecture makes callback context especially important when synchronous behavior is required. A node waiting for a service result may depend on an executor to process the callback that completes that same request. Poor executor, callback-group, or threading design can therefore cause excessive blocking or deadlock-like behavior. Service communication must consequently be designed together with the callback groups, executors, and concurrency mechanisms introduced in the node architecture chapter.

For this reason, asynchronous service APIs are frequently useful even when the application logically requires synchronous sequencing. A client can issue a request and obtain a future representing the eventual response, allowing the executor to continue processing other callbacks. Application logic can later examine or wait for that future under controlled conditions. This separates the semantic requirement for a response from unnecessary blocking of the node\'s overall execution path.

Service discovery is handled through the ROS 2 middleware architecture. A client must locate a compatible server associated with the requested service name and type before useful communication can occur. Applications commonly check or wait for service availability before sending requests. This is important during robot startup because different nodes, hardware interfaces, containers, or distributed computers may become operational at different times.

Service names provide logical addressing in a manner similar to topic names. Namespaces and remapping can therefore be applied to service interfaces so that reusable components can operate within different robots or subsystems. A service such as a reset or mode-control interface can be instantiated independently under robot-specific namespaces. This becomes increasingly important when identical software components are deployed across multiple AMRs, manipulators, or other robotic platforms.

Services differ fundamentally from topics in communication semantics. Topics emphasize decoupled streaming and can naturally distribute the same data to many subscribers. Services establish a request--response transaction between a client and a server for a specific operation. A sensor measurement stream therefore belongs naturally on a topic, whereas requesting a map reset, querying a configuration value, or commanding a short configuration change can be more naturally represented by a service.

Services must also be distinguished from ROS 2 actions. A service is best suited to operations that can normally return a result within a reasonably short period and do not require continuous progress feedback. Actions are designed for long-running operations in which the requester may need intermediate feedback, goal management, or cancellation. Navigation to a distant pose, for example, generally fits the action model better than a blocking service transaction.

Request and response message design strongly affects maintainability. A service should expose the minimum information necessary to express the operation while returning enough information for the client to determine the outcome. Responses commonly include result data together with success, failure, or diagnostic information when appropriate. Clear semantics prevent clients from depending on undocumented server behavior and allow service implementations to evolve without unnecessary coupling.

Failure handling is essential because a distributed service invocation is not equivalent to an ordinary local function call. The server may not yet be available, a process may terminate, network communication may be interrupted, middleware discovery may be incomplete, or processing may take longer than expected. Client logic should therefore account for availability, timeout behavior, error reporting, retries where appropriate, and safe fallback behavior rather than assuming that every request succeeds.

Retry policies require particular care in robotic systems. Repeating a read-only query may be harmless, but automatically repeating a request that changes hardware state can execute an operation more than once. Service semantics should therefore make it clear whether an operation is idempotent, state-changing, or unsafe to repeat. When physical actuators are involved, communication recovery must never be designed independently of system state and safety requirements.

Service callback execution time also influences the responsiveness of the ROS 2 node. A callback that performs lengthy computation, waits indefinitely for hardware, or invokes another blocking dependency can occupy executor resources and delay unrelated callbacks. Short and bounded service processing is generally easier to integrate into responsive distributed systems. Longer operations should be reconsidered as asynchronous workflows, actions, state machines, or independently scheduled tasks.

The ROS 2 command-line interface provides practical mechanisms for inspecting service communication. Developers can discover available services, examine their interface types, inspect service definitions, and manually issue requests during integration and debugging. These capabilities help determine whether a server is visible, whether the expected service type is being used, and whether request and response behavior agrees with the intended software interface.

In a production robot, services often form a management and command layer around continuously operating topic pipelines. Perception and localization nodes may exchange runtime data through topics while exposing services for reset, configuration, calibration, or state queries. This combination preserves the efficiency and loose coupling of streaming communication while providing explicit transactional interfaces for operations that require a definite request and corresponding result.

Service interfaces should nevertheless remain separate from hard real-time control paths whenever blocking or nondeterministic communication could compromise timing guarantees. A motor-control loop operating at hundreds of hertz should not depend on an unpredictable network service transaction for each control cycle. Services are better suited to supervisory operations, configuration changes, initialization, diagnostics, and other interactions whose timing requirements are compatible with distributed middleware behavior.

For autonomous robots and Physical AI systems, client--server services complement topic communication by providing explicit transactional boundaries between distributed components. When service types, namespaces, executor behavior, timeout handling, failure semantics, and operation duration are designed consistently, ROS 2 services provide a clean mechanism for coordinating short request--response interactions while leaving continuous data exchange and long-running tasks to topics and actions respectively.

ROS 2 서비스 통신(service communication)은 하나의 노드가 다른 노드에 특정 작업을 수행하도록 요청하고 정의된 결과를 반환받는 상호작용을 위한 요청-응답 메커니즘(request--response mechanism)을 제공한다. 클라이언트(client)는 이름이 지정된 서비스(service)에 형식화된 요청을 전송하고, 서버(server)는 해당 요청을 처리하여 형식화된 응답을 생성한다. 이 모델은 토픽 기반 발행자-구독자 통신(topic-based publisher--subscriber communication)이 처리하는 연속적인 데이터 스트림보다 개별적인 작업에 적합하다.

서비스 인터페이스(service interface)는 통신 계약(communication contract)의 양쪽을 정의한다. 하나의 데이터 구조를 나타내는 토픽 메시지(topic message)와 달리 서비스 정의(service definition)는 요청(request) 영역과 응답(response) 영역을 포함한다. 요청은 작업 수행에 필요한 정보를 지정하고, 응답은 작업 결과, 상태 또는 반환 데이터를 나타낸다. 이러한 명시적인 인터페이스를 통해 독립적으로 구현된 ROS 2 노드가 예측 가능하고 강력하게 형식화된 소프트웨어 경계(software boundary)를 통해 상호작용할 수 있다.

클라이언트(client)는 서비스 패턴(service pattern)에서 요청을 수행하는 측을 나타낸다. 클라이언트는 서비스 형식(service type)에 따라 요청 객체(request object)를 생성하고 필요한 필드를 채운 다음 서비스 서버(service server)에 요청을 전송한다. 개념적으로 클라이언트는 각 요청에 대해 하나의 응답을 기대한다. 대표적인 로봇 작업에는 설정 정보 조회, 내부 상태 초기화, 하위 시스템 활성화, 계산 요청, 운용 모드 변경 또는 짧고 결정론적인 작업의 시작 등이 포함된다.

서버(server)는 처리 측을 담당하며 ROS 2 그래프(ROS 2 graph) 내에서 이름이 지정된 서비스를 제공한다. 요청이 도착하면 서버의 콜백(callback)은 요청 데이터를 수신하고 필요한 로직을 실행한 다음 응답 객체(response object)를 구성하여 결과를 반환한다. 호출자(caller)와 구현부(implementation)를 분리하면 서비스 제공자는 외부에 공개되는 서비스 인터페이스를 유지하면서 내부 알고리즘이나 하드웨어 상호작용 방식을 변경할 수 있다.

동기식 서비스 호출(synchronous service call)은 익숙한 블로킹 요청-응답(blocking request--response) 개념을 표현한다. 호출 로직은 요청을 전송한 후 해당 응답이 사용 가능해질 때까지 기다린다. 요청 결과가 확보된 이후에 후속 처리가 시작되므로 이러한 프로그래밍 모델은 이해하기 쉽다. 그러나 분산 로봇 소프트웨어에서는 통신, 스케줄링, 서버 처리 또는 네트워크 지연으로 대기 시간이 예상보다 길어질 수 있으므로 블로킹(blocking)을 신중하게 사용해야 한다.

동기식 동작이 필요한 경우 ROS 2 실행 아키텍처(execution architecture)에서는 콜백 컨텍스트(callback context)가 특히 중요하다. 서비스 결과를 기다리는 노드는 동일한 요청을 완료하는 콜백을 처리하기 위해 실행기(executor)에 의존할 수 있다. 따라서 실행기, 콜백 그룹(callback group), 스레딩(threading)을 잘못 설계하면 과도한 블로킹이나 교착 상태와 유사한 동작(deadlock-like behavior)이 발생할 수 있다. 그러므로 서비스 통신은 노드 아키텍처에서 다루는 콜백 그룹, 실행기, 동시성 메커니즘(concurrency mechanism)과 함께 설계해야 한다.

이러한 이유로 응용 프로그램이 논리적으로 동기식 순차 처리(synchronous sequencing)를 요구하는 경우에도 비동기식 서비스 API(asynchronous service API)가 유용하게 사용될 수 있다. 클라이언트는 요청을 전송하고 미래에 도착할 응답을 나타내는 퓨처(future)를 얻을 수 있으므로 실행기는 다른 콜백을 계속 처리할 수 있다. 응용 프로그램 로직은 이후 제어된 조건에서 해당 퓨처를 확인하거나 기다릴 수 있다. 이를 통해 응답이 필요하다는 의미적 요구사항과 노드 전체 실행 경로의 불필요한 블로킹을 분리할 수 있다.

서비스 검색(service discovery)은 ROS 2 미들웨어 아키텍처(middleware architecture)를 통해 처리된다. 유효한 통신이 이루어지기 전에 클라이언트는 요청한 서비스 이름 및 형식과 연결된 호환 가능한 서버를 찾아야 한다. 응용 프로그램은 일반적으로 요청을 보내기 전에 서비스 가용성(service availability)을 확인하거나 서비스가 준비될 때까지 기다린다. 로봇 시작 과정에서는 서로 다른 노드, 하드웨어 인터페이스, 컨테이너(container), 분산 컴퓨터가 서로 다른 시점에 동작을 시작할 수 있으므로 이러한 과정이 중요하다.

서비스 이름(service name)은 토픽 이름과 유사한 방식으로 논리적 주소 지정(logical addressing)을 제공한다. 따라서 서비스 인터페이스에도 네임스페이스(namespace)와 재매핑(remapping)을 적용하여 재사용 가능한 구성요소를 서로 다른 로봇이나 하위 시스템에서 운용할 수 있다. 초기화 또는 모드 제어 인터페이스와 같은 서비스를 로봇별 네임스페이스 아래에서 독립적으로 구성할 수 있으며, 동일한 소프트웨어 구성요소를 여러 자율이동로봇(Autonomous Mobile Robot, AMR), 매니퓰레이터(manipulator) 또는 다른 로봇 플랫폼에 배포할 때 특히 중요하다.

서비스는 통신 의미론(communication semantics) 측면에서 토픽과 근본적으로 다르다. 토픽은 느슨하게 결합된 스트리밍(streaming)을 중심으로 하며 동일한 데이터를 여러 구독자에게 자연스럽게 배포할 수 있다. 서비스는 특정 작업을 위해 클라이언트와 서버 사이에 요청-응답 트랜잭션(request--response transaction)을 구성한다. 따라서 센서 측정 스트림은 토픽에 적합하고, 맵 초기화 요청, 설정값 조회 또는 짧은 설정 변경 명령은 서비스로 표현하는 것이 더 자연스럽다.

서비스는 ROS 2 액션(action)과도 구분해야 한다. 서비스는 일반적으로 합리적으로 짧은 시간 내에 결과를 반환할 수 있고 지속적인 진행 피드백(progress feedback)이 필요하지 않은 작업에 적합하다. 액션은 요청자가 중간 피드백, 목표 관리(goal management), 취소 기능을 필요로 할 수 있는 장시간 실행 작업(long-running operation)을 위해 설계되었다. 예를 들어 먼 목표 위치까지 이동하는 내비게이션 작업은 블로킹 서비스 트랜잭션보다 액션 모델(action model)에 더 적합하다.

요청 및 응답 메시지 설계(request and response message design)는 유지보수성(maintainability)에 큰 영향을 준다. 서비스는 작업을 표현하는 데 필요한 최소한의 정보를 제공하면서 클라이언트가 결과를 판단하기에 충분한 정보를 반환해야 한다. 필요한 경우 응답에는 결과 데이터와 함께 성공, 실패 또는 진단 정보가 포함될 수 있다. 명확한 의미론은 클라이언트가 문서화되지 않은 서버 동작에 의존하는 것을 방지하고 불필요한 결합 없이 서비스 구현을 발전시킬 수 있도록 한다.

분산 서비스 호출(distributed service invocation)은 일반적인 로컬 함수 호출(local function call)과 동일하지 않으므로 실패 처리(failure handling)가 필수적이다. 서버가 아직 준비되지 않았거나 프로세스가 종료될 수 있으며, 네트워크 통신이 중단되거나 미들웨어 검색이 완료되지 않았거나 처리가 예상보다 오래 걸릴 수도 있다. 따라서 클라이언트 로직은 모든 요청이 성공한다고 가정하는 대신 가용성, 타임아웃(timeout), 오류 보고, 필요한 경우 재시도(retry), 안전한 대체 동작(fallback behavior)을 고려해야 한다.

로봇 시스템에서는 재시도 정책(retry policy)을 특히 신중하게 설계해야 한다. 읽기 전용 조회(read-only query)를 반복하는 것은 문제가 없을 수 있지만, 하드웨어 상태를 변경하는 요청을 자동으로 반복하면 동일한 작업이 여러 번 실행될 수 있다. 따라서 서비스 의미론은 작업이 멱등성(idempotent)을 가지는지, 상태 변경 작업인지, 반복 실행이 위험한지를 명확히 해야 한다. 물리적 액추에이터(physical actuator)가 관련된 경우 통신 복구는 시스템 상태와 안전 요구사항을 고려하여 설계해야 한다.

서비스 콜백 실행 시간(service callback execution time)은 ROS 2 노드의 응답성에도 영향을 준다. 장시간 계산을 수행하거나 하드웨어 응답을 무기한 기다리거나 다른 블로킹 종속성(blocking dependency)을 호출하는 콜백은 실행기 자원을 점유하고 관련 없는 다른 콜백의 실행을 지연시킬 수 있다. 짧고 실행 시간이 제한된 서비스 처리가 응답성이 높은 분산 시스템에 통합하기 쉽다. 장시간 작업은 비동기식 워크플로(asynchronous workflow), 액션, 상태 머신(state machine) 또는 독립적으로 스케줄링되는 작업으로 재구성하는 것이 적절하다.

ROS 2 명령줄 인터페이스(command-line interface)는 서비스 통신을 검사하기 위한 실용적인 메커니즘을 제공한다. 개발자는 사용 가능한 서비스를 검색하고 인터페이스 형식을 확인하며 서비스 정의를 검사하고 통합 및 디버깅 과정에서 직접 요청을 전송할 수 있다. 이러한 기능을 통해 서버가 정상적으로 검색되는지, 예상한 서비스 형식이 사용되고 있는지, 요청 및 응답 동작이 의도한 소프트웨어 인터페이스와 일치하는지를 확인할 수 있다.

실제 운용 로봇(production robot)에서 서비스는 지속적으로 동작하는 토픽 파이프라인(topic pipeline)을 둘러싼 관리 및 명령 계층(management and command layer)을 구성하는 경우가 많다. 인지 및 위치 추정 노드는 실행 데이터를 토픽을 통해 교환하면서 초기화, 설정, 보정(calibration), 상태 조회를 위한 서비스를 제공할 수 있다. 이러한 조합은 스트리밍 통신의 효율성과 느슨한 결합을 유지하면서 명확한 요청과 그에 대응하는 결과가 필요한 작업에 명시적인 트랜잭션 인터페이스(transactional interface)를 제공한다.

서비스 인터페이스는 블로킹 또는 비결정론적 통신(nondeterministic communication)이 타이밍 보장에 영향을 줄 수 있는 하드 실시간 제어 경로(hard real-time control path)와 분리해야 한다. 수백 헤르츠로 동작하는 모터 제어 루프(motor-control loop)가 매 제어 주기마다 예측하기 어려운 네트워크 서비스 트랜잭션에 의존해서는 안 된다. 서비스는 감독 작업(supervisory operation), 설정 변경, 초기화, 진단 등 분산 미들웨어 동작과 호환되는 타이밍 요구사항을 가진 상호작용에 더 적합하다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 클라이언트-서버 서비스(client--server service)는 분산 구성요소 사이에 명시적인 트랜잭션 경계(transactional boundary)를 제공함으로써 토픽 통신을 보완한다. 서비스 형식, 네임스페이스, 실행기 동작, 타임아웃 처리, 실패 의미론, 작업 수행 시간을 일관되게 설계하면 ROS 2 서비스는 짧은 요청-응답 상호작용을 조정하는 명확한 메커니즘을 제공하며, 연속적인 데이터 교환은 토픽(topic), 장시간 작업은 액션(action)이 각각 담당하도록 역할을 분리할 수 있다.

##  

## 04.03 Action Communication: Long-Running Async Tasks [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 action communication is designed for long-running operations that cannot be represented effectively as a continuous topic stream or a short request--response service transaction. An action allows a client to submit a goal to an action server, receive intermediate feedback while the operation is executing, obtain a final result, and request cancellation when necessary. This makes actions particularly suitable for navigation, manipulation, docking, calibration, and complex robot behaviors.

The action communication model is organized around three fundamental data elements: goal, feedback, and result. The goal describes what the client wants the server to accomplish, feedback reports intermediate execution information, and the result describes the final outcome. Together, these elements provide a structured asynchronous interaction model in which the requester can observe and influence an operation throughout its execution rather than simply waiting for completion.

An action interface defines these communication elements as a single software contract. The goal section contains parameters required to initiate the task, the result section contains information returned when execution finishes, and the feedback section defines data periodically reported during execution. This explicit structure allows independently developed nodes to agree on task semantics while keeping implementation details hidden behind a stable ROS 2 interface.

The action client represents the component requesting execution of a task. It constructs a goal according to the action type and sends that goal to an action server. Instead of blocking until the entire operation completes, the client can continue processing other events while monitoring goal acceptance, feedback, completion, or cancellation. This asynchronous behavior is essential when robot tasks may require seconds, minutes, or potentially much longer execution periods.

The action server receives goals and determines whether they should be accepted or rejected. An accepted goal becomes an active operation whose execution is managed by the server. During execution, the server can periodically publish feedback describing progress or current state. When processing finishes, it produces a result and terminates the goal with an appropriate status, allowing the client to distinguish successful completion from cancellation, abortion, or other outcomes.

Goal management is a major difference between actions and ordinary service communication. A service request represents a comparatively short transaction, whereas an action goal has a lifecycle that persists during execution. The system must therefore track goal identity, acceptance, execution state, cancellation requests, and terminal status. This lifecycle enables distributed robot components to coordinate tasks whose progress cannot be represented adequately by a single request followed immediately by a response.

Feedback provides visibility into the internal progress of a long-running task without exposing the server\'s complete implementation. A navigation action can report remaining distance or other progress information, while a manipulator action may report execution state or trajectory progress. Feedback allows supervisory software, user interfaces, behavior managers, and other nodes to monitor execution and react to changing conditions before the final result becomes available.

Cancellation is another defining capability of the action model. A client may determine that an active goal is no longer required because the mission has changed, an obstacle has appeared, a higher-priority command has arrived, or a safety condition requires interruption. The client can request cancellation, after which the server determines how to terminate the operation safely. Cancellation therefore requires application-level handling rather than simply terminating a callback or process.

Actions are asynchronous by nature because long-running robot behavior should not prevent a node from processing unrelated communication. Goal submission, feedback reception, cancellation, and result processing are commonly integrated with ROS 2 executors and callbacks. Correct executor and callback-group design remains important because computationally expensive execution or blocking callbacks can delay feedback, cancellation handling, sensor processing, or other communication occurring within the same process.

The action server should normally separate communication handling from the actual long-running workload. If a goal callback directly performs a lengthy operation while monopolizing an executor thread, responsiveness can deteriorate significantly. A more scalable architecture accepts the goal, establishes execution state, and performs the workload through an appropriate asynchronous execution path while allowing ROS 2 callbacks to continue processing communication and lifecycle events.

Action communication is built on ROS 2 communication primitives rather than existing as an entirely separate transport mechanism. At the application level, developers work with goal, feedback, result, and cancellation semantics, while ROS 2 manages the underlying communication required to coordinate them. This abstraction provides a task-oriented interface without requiring application code to manually construct multiple independent communication channels for every long-running operation.

Action names, like topic and service names, participate in the ROS 2 naming and namespace system. A navigation action may therefore exist under robot-specific namespaces in a multi-robot deployment. Namespaces allow identical action server implementations to run on multiple AMRs, manipulators, quadrupeds, or humanoids while remaining logically separated. Fleet or mission-level software can then address the appropriate robot and submit goals without modifying the underlying action implementation.

Actions should be selected according to interaction semantics rather than simply because an operation is computationally complex. Topics are appropriate for continuous asynchronous data streams, and services are appropriate for relatively short request--response interactions. Actions are appropriate when an operation has meaningful duration and benefits from explicit goal management, progress feedback, final results, or cancellation. Maintaining this distinction produces clearer and more predictable ROS 2 interfaces.

Navigation is one of the most representative uses of ROS 2 actions. A client can submit a destination pose as a goal, while the navigation system performs localization-aware path planning, obstacle avoidance, trajectory execution, and recovery behavior over an extended period. During execution, feedback can expose progress, and the goal can be canceled or replaced when mission requirements change. The final result then communicates the terminal outcome to the requesting component.

Manipulator control provides another important use case. A higher-level node may request execution of a trajectory, movement to a target pose, or completion of a manipulation behavior. Such operations take measurable time and may need to be interrupted if perception changes or a safety condition occurs. An action interface allows motion execution to remain encapsulated while exposing the control points needed by planning, supervision, and mission-management software.

Autonomous mobile robots also use actions for docking, charging, inspection, and mission-level behaviors. A docking operation may include searching for the station, alignment, approach, contact verification, and charging confirmation. Representing the entire procedure as a blocking service would provide little visibility during execution. An action can instead maintain a persistent goal while reporting intermediate progress and supporting controlled cancellation or failure recovery.

Failure handling must account for the distributed and stateful nature of action execution. A goal may be rejected before execution, aborted after execution begins, canceled by the client, or interrupted by failures in hardware, communication, perception, or planning. The final status and result should communicate enough information for higher-level software to determine the next step. Mission logic can then retry, select an alternative behavior, enter a safe state, or escalate the failure.

Multiple goals require explicit policy decisions. Depending on the application, an action server may allow concurrent goals, reject new goals while one is active, or replace a previous goal with a newer one. The correct strategy depends on resource ownership and task semantics. A mobile base, for example, generally cannot execute independent conflicting navigation commands simultaneously, whereas other servers may safely process multiple logically independent operations.

Action interfaces also create useful boundaries between mission-level intelligence and lower-level execution. A behavior planner, task manager, or Physical AI agent can express an intended goal without controlling every intermediate actuator command. The action server translates that goal into an execution procedure and continuously reports progress. This separation supports hierarchical robot architectures in which high-level reasoning and low-level execution operate at different timescales.

For production robotics, action design should define goal semantics, acceptance rules, feedback content, cancellation behavior, terminal states, timeout expectations, and resource ownership clearly. Ambiguous action behavior can create difficult integration failures because long-running tasks interact with changing robot state over time. Well-defined action contracts allow navigation, manipulation, docking, inspection, and autonomous behaviors to remain modular while still being observable and controllable.

In autonomous robots and Physical AI systems, ROS 2 actions complete the communication model formed by topics and services. Topics provide continuous data distribution, services provide short transactional request--response operations, and actions provide managed asynchronous execution for long-running tasks. Together, these mechanisms allow distributed robot software to separate streaming information, immediate transactions, and persistent goal-oriented behavior into communication patterns suited to their actual operational semantics.

ROS 2 액션 통신(action communication)은 연속적인 토픽 스트림(topic stream)이나 짧은 요청-응답 서비스 트랜잭션(request--response service transaction)으로 효과적으로 표현하기 어려운 장시간 실행 작업(long-running operation)을 위해 설계되었다. 액션(action)을 사용하면 클라이언트(client)가 액션 서버(action server)에 목표(goal)를 제출하고, 작업 실행 중 중간 피드백(feedback)을 수신하며, 최종 결과(result)를 얻고, 필요한 경우 취소(cancellation)를 요청할 수 있다. 따라서 액션은 내비게이션(navigation), 조작(manipulation), 도킹(docking), 보정(calibration), 복잡한 로봇 행동에 특히 적합하다.

액션 통신 모델(action communication model)은 목표(goal), 피드백(feedback), 결과(result)라는 세 가지 핵심 데이터 요소를 중심으로 구성된다. 목표는 클라이언트가 서버에 수행하도록 요청하는 작업을 정의하고, 피드백은 실행 중간의 정보를 전달하며, 결과는 최종 수행 결과를 나타낸다. 이 요소들은 요청자가 단순히 작업 완료를 기다리는 대신 전체 실행 과정에서 작업을 관찰하고 영향을 줄 수 있는 구조화된 비동기식 상호작용 모델(asynchronous interaction model)을 제공한다.

액션 인터페이스(action interface)는 이러한 통신 요소들을 하나의 소프트웨어 계약(software contract)으로 정의한다. 목표 영역(goal section)은 작업 시작에 필요한 매개변수를 포함하고, 결과 영역(result section)은 실행 완료 시 반환되는 정보를 포함하며, 피드백 영역(feedback section)은 실행 중 주기적으로 보고되는 데이터를 정의한다. 이러한 명시적 구조를 통해 독립적으로 개발된 노드가 안정적인 ROS 2 인터페이스 뒤에 구현 세부사항을 숨기면서 작업의 의미론(task semantics)을 공유할 수 있다.

액션 클라이언트(action client)는 작업 실행을 요청하는 구성요소를 나타낸다. 액션 형식(action type)에 따라 목표를 구성하여 액션 서버로 전송하며, 전체 작업이 완료될 때까지 블로킹(blocking)하는 대신 목표 수락, 피드백, 완료 또는 취소 상태를 모니터링하면서 다른 이벤트를 계속 처리할 수 있다. 이러한 비동기식 동작(asynchronous behavior)은 로봇 작업이 수초, 수분 또는 그보다 훨씬 긴 시간 동안 수행될 수 있는 경우 필수적이다.

액션 서버(action server)는 목표를 수신하고 이를 수락할지 거부할지를 결정한다. 수락된 목표는 서버가 실행을 관리하는 활성 작업(active operation)이 된다. 실행 중 서버는 진행 상황이나 현재 상태를 나타내는 피드백을 주기적으로 발행할 수 있다. 처리가 완료되면 결과를 생성하고 적절한 상태(status)로 목표를 종료하여 클라이언트가 정상 완료, 취소, 중단(abortion) 또는 기타 결과를 구분할 수 있도록 한다.

목표 관리(goal management)는 액션과 일반적인 서비스 통신을 구분하는 중요한 특징이다. 서비스 요청(service request)이 비교적 짧은 트랜잭션을 나타내는 반면, 액션 목표(action goal)는 실행 기간 동안 유지되는 수명주기(lifecycle)를 가진다. 따라서 시스템은 목표 식별자(goal identity), 수락 여부, 실행 상태, 취소 요청, 종료 상태(terminal status)를 추적해야 한다. 이러한 수명주기를 통해 단일 요청과 즉각적인 응답만으로 충분히 표현하기 어려운 작업을 분산 로봇 구성요소가 조정할 수 있다.

피드백(feedback)은 서버의 전체 구현을 외부에 노출하지 않으면서 장시간 작업의 내부 진행 상태를 확인할 수 있게 한다. 내비게이션 액션은 남은 거리나 기타 진행 정보를 보고할 수 있고, 매니퓰레이터 액션(manipulator action)은 실행 상태 또는 궤적 진행 상황(trajectory progress)을 전달할 수 있다. 이를 통해 감독 소프트웨어(supervisory software), 사용자 인터페이스, 행동 관리자(behavior manager), 다른 노드가 최종 결과가 나오기 전에 실행 상태를 모니터링하고 변화하는 조건에 대응할 수 있다.

취소(cancellation)는 액션 모델의 또 다른 핵심 기능이다. 임무가 변경되거나 장애물이 나타나거나 더 높은 우선순위의 명령이 도착하거나 안전 조건으로 인해 작업을 중단해야 하는 경우 클라이언트는 활성 목표가 더 이상 필요하지 않다고 판단할 수 있다. 클라이언트는 취소를 요청하고 서버는 작업을 안전하게 종료하는 방법을 결정한다. 따라서 취소는 단순히 콜백이나 프로세스를 종료하는 것이 아니라 응용 수준 처리(application-level handling)를 필요로 한다.

장시간 실행되는 로봇 행동이 노드의 다른 통신 처리를 방해해서는 안 되므로 액션은 본질적으로 비동기식(asynchronous)이다. 목표 제출, 피드백 수신, 취소, 결과 처리는 일반적으로 ROS 2 실행기(executor) 및 콜백(callback)과 통합된다. 계산량이 많은 실행이나 블로킹 콜백(blocking callback)은 동일한 프로세스에서 수행되는 피드백, 취소 처리, 센서 처리 또는 기타 통신을 지연시킬 수 있으므로 적절한 실행기와 콜백 그룹(callback group) 설계가 중요하다.

액션 서버는 일반적으로 통신 처리와 실제 장시간 실행 작업을 분리해야 한다. 목표 콜백(goal callback)이 실행기 스레드(executor thread)를 독점하면서 장시간 작업을 직접 수행하면 시스템 응답성이 크게 저하될 수 있다. 보다 확장 가능한 아키텍처에서는 목표를 수락하고 실행 상태를 설정한 다음 적절한 비동기식 실행 경로(asynchronous execution path)를 통해 작업을 수행하면서 ROS 2 콜백이 통신 및 수명주기 이벤트를 계속 처리할 수 있도록 한다.

액션 통신은 완전히 독립적인 전송 메커니즘으로 존재하는 것이 아니라 ROS 2 통신 기본 요소(communication primitive)를 기반으로 구성된다. 응용 수준에서 개발자는 목표, 피드백, 결과, 취소의 의미론을 사용하고 ROS 2는 이를 조정하기 위해 필요한 하위 통신을 관리한다. 이러한 추상화(abstraction)는 응용 코드가 모든 장시간 작업을 위해 여러 독립적인 통신 채널을 직접 구성하지 않고도 작업 중심 인터페이스(task-oriented interface)를 사용할 수 있게 한다.

액션 이름(action name)은 토픽 및 서비스 이름과 마찬가지로 ROS 2 이름 지정 및 네임스페이스 시스템(naming and namespace system)에 포함된다. 따라서 다중 로봇 배포(multi-robot deployment)에서는 내비게이션 액션을 로봇별 네임스페이스 아래에 구성할 수 있다. 이를 통해 동일한 액션 서버 구현을 여러 자율이동로봇(Autonomous Mobile Robot, AMR), 매니퓰레이터, 사족보행로봇(quadruped), 휴머노이드(humanoid)에서 논리적으로 분리하여 실행할 수 있다.

액션은 단순히 연산이 복잡하다는 이유가 아니라 상호작용 의미론(interaction semantics)에 따라 선택해야 한다. 토픽(topic)은 지속적인 비동기식 데이터 스트림에 적합하고, 서비스(service)는 비교적 짧은 요청-응답 상호작용에 적합하다. 액션은 작업에 의미 있는 실행 시간이 존재하고 명시적인 목표 관리, 진행 피드백, 최종 결과 또는 취소가 필요한 경우에 적합하다. 이러한 구분을 유지하면 보다 명확하고 예측 가능한 ROS 2 인터페이스를 구성할 수 있다.

내비게이션(navigation)은 ROS 2 액션을 사용하는 가장 대표적인 사례 중 하나이다. 클라이언트는 목적지 자세(destination pose)를 목표로 제출하고, 내비게이션 시스템은 장시간에 걸쳐 위치 추정 기반 경로 계획, 장애물 회피, 궤적 실행, 복구 동작(recovery behavior)을 수행할 수 있다. 실행 중 피드백을 통해 진행 상태를 확인할 수 있으며 임무 요구사항이 변경되면 목표를 취소하거나 교체할 수 있다. 최종 결과는 요청 구성요소에 작업의 종료 결과를 전달한다.

매니퓰레이터 제어(manipulator control) 역시 중요한 활용 사례이다. 상위 수준 노드는 궤적 실행, 목표 자세로의 이동 또는 조작 행동 완료를 요청할 수 있다. 이러한 작업은 일정한 실행 시간이 필요하며 인지 정보가 변경되거나 안전 조건이 발생하면 중단되어야 할 수도 있다. 액션 인터페이스는 모션 실행(motion execution)을 내부에 캡슐화(encapsulation)하면서 계획, 감독, 임무 관리 소프트웨어가 필요로 하는 제어 지점을 외부에 제공한다.

자율이동로봇은 도킹, 충전, 검사 및 임무 수준 행동(mission-level behavior)에도 액션을 사용할 수 있다. 도킹 작업에는 충전소 검색, 정렬, 접근, 접촉 확인, 충전 확인과 같은 여러 단계가 포함될 수 있다. 전체 절차를 블로킹 서비스(blocking service)로 표현하면 실행 중 상태를 충분히 확인하기 어렵다. 액션은 지속적인 목표를 유지하면서 중간 진행 상황을 보고하고 제어된 취소 또는 실패 복구(failure recovery)를 지원할 수 있다.

실패 처리(failure handling)는 액션 실행의 분산적이고 상태를 가지는 특성(stateful nature)을 고려해야 한다. 목표는 실행 전에 거부되거나 실행이 시작된 이후 중단될 수 있으며, 클라이언트가 취소하거나 하드웨어, 통신, 인지, 계획의 오류로 인해 작업이 중단될 수도 있다. 최종 상태와 결과에는 상위 수준 소프트웨어가 다음 동작을 결정하기에 충분한 정보가 포함되어야 하며, 임무 로직은 이를 바탕으로 재시도, 대체 행동 선택, 안전 상태 진입 또는 오류 상위 전달을 수행할 수 있다.

다중 목표(multiple goals)를 처리하려면 명시적인 정책 결정이 필요하다. 응용 분야에 따라 액션 서버는 여러 목표의 동시 실행을 허용하거나 하나의 목표가 활성화된 동안 새로운 목표를 거부하거나 기존 목표를 새로운 목표로 대체할 수 있다. 적절한 전략은 자원 소유권(resource ownership)과 작업 의미론에 따라 결정된다. 예를 들어 모바일 베이스(mobile base)는 일반적으로 서로 충돌하는 독립적인 내비게이션 명령을 동시에 실행할 수 없지만, 다른 서버는 논리적으로 독립된 여러 작업을 안전하게 처리할 수 있다.

액션 인터페이스는 임무 수준 지능(mission-level intelligence)과 하위 수준 실행(low-level execution) 사이에 유용한 경계를 형성한다. 행동 계획기(behavior planner), 작업 관리자(task manager), 피지컬 AI 에이전트(Physical AI agent)는 모든 중간 액추에이터 명령을 직접 제어하지 않고 의도된 목표를 표현할 수 있다. 액션 서버는 해당 목표를 실행 절차로 변환하고 진행 상태를 지속적으로 보고한다. 이러한 분리는 상위 수준 추론과 하위 수준 실행이 서로 다른 시간 척도(timescale)에서 동작하는 계층적 로봇 아키텍처(hierarchical robot architecture)를 지원한다.

실제 운용 로보틱스(production robotics)에서 액션 설계는 목표 의미론, 수락 규칙, 피드백 내용, 취소 동작, 종료 상태, 타임아웃 기대값(timeout expectation), 자원 소유권을 명확하게 정의해야 한다. 장시간 작업은 시간이 흐르면서 변화하는 로봇 상태와 상호작용하기 때문에 모호한 액션 동작은 통합 과정에서 진단하기 어려운 오류를 만들 수 있다. 잘 정의된 액션 계약(action contract)은 내비게이션, 조작, 도킹, 검사 및 자율 행동을 관찰 가능하고 제어 가능한 상태로 유지하면서 모듈화할 수 있게 한다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 ROS 2 액션은 토픽과 서비스로 구성되는 통신 모델을 완성한다. 토픽은 연속적인 데이터 배포를 담당하고, 서비스는 짧은 트랜잭션형 요청-응답 작업을 담당하며, 액션은 장시간 작업을 위한 관리 가능한 비동기식 실행(managed asynchronous execution)을 제공한다. 이 세 가지 메커니즘을 함께 사용하면 분산 로봇 소프트웨어에서 스트리밍 정보, 즉각적인 트랜잭션, 지속적인 목표 지향 행동(goal-oriented behavior)을 실제 운용 의미에 적합한 통신 패턴으로 분리할 수 있다.

##  

## 04.04 QoS Deep Dive: Reliability, Durability, History [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 Quality of Service, or QoS, defines how data is delivered between publishers and subscribers rather than merely what data is exchanged. QoS policies allow communication behavior to be adapted to sensor streams, control commands, state information, diagnostics, and distributed robot workloads. Among these policies, reliability, durability, and history are especially important because they directly determine delivery guarantees, retained samples, and queue behavior.

QoS exists because robotic communication channels have very different requirements. A camera publishing high-rate images may prioritize fresh data and low latency, while a configuration or system-state publisher may require stronger delivery guarantees. Applying the same communication policy everywhere can waste bandwidth, increase latency, or cause unnecessary retransmission. ROS 2 therefore exposes middleware communication characteristics through configurable QoS profiles.

Reliability determines how ROS 2 handles message delivery when packets are lost or communication conditions deteriorate. The two principal reliability policies are Reliable and Best Effort. Reliable communication attempts to ensure that samples reach compatible subscribers, potentially using acknowledgment and retransmission mechanisms provided by the underlying middleware. Best Effort sends data without requiring the same delivery guarantee, favoring lower communication overhead and timely transmission.

Reliable QoS is appropriate when losing information may affect application correctness. Commands, important state transitions, configuration information, or low-rate data that must be processed consistently can benefit from reliable delivery. However, reliability does not make a distributed system failure-proof. Network interruption, process termination, exhausted resources, or incompatible QoS configurations can still prevent communication, so application-level fault handling remains necessary.

Best Effort QoS is particularly useful for high-frequency sensor streams where the newest sample is more valuable than an older sample that was lost in transit. Camera images, LiDAR scans, or rapidly changing measurements may become obsolete quickly. Retransmitting stale data can increase congestion and latency without improving robot behavior. Best Effort communication therefore often provides a practical balance for perception pipelines operating over constrained or variable networks.

The choice between Reliable and Best Effort should consider publication frequency, payload size, network quality, acceptable loss, and the semantic value of individual samples. A low-rate command channel and a high-bandwidth image stream may require completely different policies even though both use ROS 2 topics. QoS should therefore be treated as part of interface design rather than as a middleware setting selected only after software integration.

Durability determines whether data should remain available for subscribers that join after publication. Volatile durability provides only data generated while communication endpoints are participating in the current exchange. A late-joining subscriber does not automatically receive earlier samples. This behavior is suitable for continuously changing information where historical values have little value once newer data becomes available.

Transient Local durability allows a publisher to retain samples so that compatible subscribers joining later can receive previously published information according to the configured history and resource limits. This behavior is useful for information representing persistent or slowly changing state. A subscriber starting after initialization may need the latest relevant state rather than waiting indefinitely for the publisher to generate another identical update.

Durability should not be confused with permanent database storage. Transient Local behavior is associated with middleware communication and the lifetime and resources of participating entities; it is not a substitute for persistent logging, databases, rosbag recording, or fault-tolerant storage. Robot architects should distinguish communication-level sample retention from long-term data persistence when designing state recovery and system restart behavior.

History controls how samples are retained when publishers and subscribers operate at different rates. Keep Last stores a limited number of recent samples, while Keep All attempts to preserve all samples subject to middleware resource limits. Keep Last is commonly paired with a depth value that specifies the size of the retained queue. Together, history and depth determine how much temporary backlog can accumulate.

A Keep Last depth of one emphasizes the newest state because an older queued value can be replaced by a newer one before processing. Larger depths provide additional buffering when callbacks temporarily fall behind publication. However, increasing depth is not automatically beneficial. Deep queues consume memory and may cause subscribers to process stale information long after it has lost operational value, particularly in fast perception and control pipelines.

Keep All can be appropriate when every sample has semantic importance, but it requires careful resource management. If a producer generates data faster than the consumer can process it, retained samples can grow until middleware limits are reached. In a robot system with bounded CPU and memory, an apparently lossless queue can therefore create latency or resource pressure. Communication requirements must be balanced against processing capacity.

Reliability, durability, and history interact rather than operating as completely independent choices. A Reliable publisher with deep history may preserve and retransmit more information but can consume greater bandwidth and memory. A Best Effort publisher with shallow Keep Last history can prioritize freshness and bounded resource usage. A Transient Local publisher combines retained state with late-joining subscriber behavior, making history configuration especially significant.

QoS compatibility between publishers and subscribers is essential. Matching a topic name and message type does not guarantee successful communication when requested and offered QoS policies are incompatible. A publisher offers communication characteristics, while a subscriber requests characteristics it can accept. ROS 2 and the underlying DDS middleware evaluate these policies when establishing communication between endpoints, making QoS mismatch an important source of integration failures.

This compatibility model becomes particularly important when integrating independently developed ROS 2 packages. A sensor driver may use a QoS profile optimized for streaming measurements while a custom subscriber may be created with assumptions appropriate to reliable application data. Both nodes can appear in the ROS graph yet fail to exchange samples as expected. Developers should therefore inspect endpoint QoS settings during communication debugging.

ROS 2 provides predefined QoS profiles and configurable policy combinations for common communication patterns. Sensor-oriented communication generally favors freshness and bounded queues, while other interfaces may emphasize reliability or retained state. These profiles provide useful starting points, but production systems should validate them against actual message rates, payload sizes, processing delays, network topology, and failure conditions rather than assuming one profile is universally appropriate.

Network characteristics strongly influence practical QoS behavior. On a stable wired network, Reliable communication may impose acceptable overhead, while wireless links with packet loss or variable latency can cause retransmissions and queue buildup. Large camera images or point clouds amplify these effects. QoS selection must therefore consider not only ROS 2 software architecture but also Ethernet, Wi-Fi, shared bandwidth, middleware configuration, and deployment topology.

QoS also influences end-to-end latency. A deep history queue can preserve data while simultaneously increasing the age of information processed by a slow subscriber. Reliable retransmission can improve delivery completeness while increasing latency under packet loss. Best Effort can reduce retransmission pressure but permit missing samples. For autonomous robots, the correct optimization target is often useful and timely information rather than maximum delivery completeness alone.

Diagnostics should examine both message transport and application behavior. Topic frequency, bandwidth, callback processing time, dropped samples, queue depth, and endpoint QoS configuration provide complementary evidence. When a subscriber receives no data or experiences unexpected delays, developers should check topic and message compatibility together with reliability, durability, history, depth, network conditions, and executor behavior before attributing the problem to a single layer.

QoS design becomes more complex in multi-robot and distributed Physical AI systems because communication may cross processes, computers, network segments, wireless links, and fleet infrastructure. High-bandwidth perception data may remain local to a robot while compact state information is distributed more broadly. Selecting QoS according to data semantics and communication scope helps prevent unnecessary network traffic while preserving information required for coordination and supervision.

Production ROS 2 systems should define QoS as part of each interface contract. Documentation should identify why a channel uses Reliable or Best Effort delivery, whether late-joining subscribers require retained samples, and how much history is operationally meaningful. Explicit QoS contracts reduce integration ambiguity and make performance testing reproducible across sensors, compute platforms, middleware implementations, and network configurations.

Reliability, durability, and history ultimately represent different dimensions of communication intent. Reliability asks how strongly delivery should be attempted, durability asks whether earlier information should remain available to later participants, and history asks how many samples should be retained while communication proceeds. Understanding these distinctions allows ROS 2 architects to design communication channels according to the actual temporal and operational meaning of robot data.

ROS 2 서비스 품질(Quality of Service, QoS)은 단순히 어떤 데이터를 교환하는지를 넘어 발행자(publisher)와 구독자(subscriber) 사이에서 데이터가 어떻게 전달되는지를 정의한다. QoS 정책(QoS policy)을 통해 센서 스트림(sensor stream), 제어 명령(control command), 상태 정보, 진단 정보, 분산 로봇 작업 부하(distributed robot workload)에 맞게 통신 동작을 조정할 수 있다. 이 가운데 신뢰성(reliability), 내구성(durability), 이력(history)은 전달 보장, 샘플 유지, 큐(queue) 동작을 직접 결정하는 핵심 정책이다.

QoS가 필요한 이유는 로봇의 통신 채널마다 요구사항이 매우 다르기 때문이다. 고속으로 영상을 발행하는 카메라는 최신 데이터와 낮은 지연시간(latency)을 우선할 수 있지만, 설정 정보나 시스템 상태를 발행하는 노드는 더 강력한 전달 보장을 요구할 수 있다. 모든 통신에 동일한 정책을 적용하면 대역폭을 낭비하거나 지연시간을 증가시키고 불필요한 재전송(retransmission)을 발생시킬 수 있다. 따라서 ROS 2는 미들웨어 통신 특성을 설정 가능한 QoS 프로파일(QoS profile)로 제공한다.

신뢰성(reliability)은 패킷(packet)이 손실되거나 통신 환경이 악화될 때 ROS 2가 메시지 전달을 어떻게 처리할지를 결정한다. 대표적인 신뢰성 정책은 신뢰성 전달(Reliable)과 최선형 전달(Best Effort)이다. Reliable 통신은 하위 미들웨어가 제공하는 확인 응답(acknowledgment)과 재전송 메커니즘 등을 활용하여 호환 가능한 구독자에게 샘플이 전달되도록 시도한다. Best Effort는 동일한 수준의 전달 보장을 요구하지 않고 데이터를 전송하여 낮은 통신 부하와 적시성을 우선한다.

신뢰성 QoS(Reliable QoS)는 정보 손실이 응용 프로그램의 정확성에 영향을 줄 수 있는 경우에 적합하다. 명령, 중요한 상태 전환(state transition), 설정 정보 또는 일관되게 처리해야 하는 저주기 데이터는 신뢰성 있는 전달의 이점을 얻을 수 있다. 그러나 신뢰성이 분산 시스템을 완전히 무결하게 만드는 것은 아니다. 네트워크 중단, 프로세스 종료, 자원 고갈 또는 호환되지 않는 QoS 설정으로 인해 여전히 통신이 실패할 수 있으므로 응용 수준의 오류 처리(application-level fault handling)가 필요하다.

최선형 QoS(Best Effort QoS)는 전송 중 손실된 오래된 샘플보다 최신 샘플이 더 중요한 고주파 센서 스트림에 특히 유용하다. 카메라 영상, LiDAR 스캔 또는 빠르게 변화하는 측정값은 짧은 시간 내에 정보 가치가 사라질 수 있다. 오래된 데이터를 재전송하면 로봇 동작을 개선하지 못하면서 네트워크 혼잡과 지연시간만 증가시킬 수 있다. 따라서 Best Effort 통신은 제한적이거나 변동성이 높은 네트워크에서 동작하는 인지 파이프라인(perception pipeline)에 실용적인 균형점을 제공한다.

Reliable과 Best Effort 중 어느 것을 선택할지는 발행 주기(publication frequency), 페이로드 크기(payload size), 네트워크 품질, 허용 가능한 손실률, 개별 샘플의 의미적 가치(semantic value)를 고려해야 한다. 저주기 명령 채널과 고대역폭 영상 스트림은 모두 ROS 2 토픽을 사용하더라도 완전히 다른 정책이 필요할 수 있다. 따라서 QoS는 소프트웨어 통합 이후에 선택하는 단순한 미들웨어 설정이 아니라 인터페이스 설계(interface design)의 일부로 다루어야 한다.

내구성(durability)은 데이터가 발행된 이후에 참여하는 구독자에게 이전 데이터가 제공되어야 하는지를 결정한다. 휘발성 내구성(Volatile durability)은 현재 통신에 참여하고 있는 엔드포인트(endpoint) 사이에서 생성된 데이터만 제공한다. 늦게 참여한 구독자(late-joining subscriber)는 이전에 발행된 샘플을 자동으로 수신하지 않는다. 이러한 동작은 새로운 데이터가 생성되면 과거 값의 의미가 거의 사라지는 지속적으로 변화하는 정보에 적합하다.

일시적 로컬 내구성(Transient Local durability)은 발행자가 샘플을 유지하여 나중에 참여한 호환 가능한 구독자가 설정된 이력과 자원 제한에 따라 이전에 발행된 정보를 받을 수 있도록 한다. 이러한 동작은 지속적이거나 느리게 변화하는 상태를 표현하는 정보에 유용하다. 초기화 이후에 시작된 구독자는 발행자가 동일한 업데이트를 다시 생성할 때까지 기다리는 대신 가장 최근의 유효한 상태를 받을 수 있다.

내구성은 영구적인 데이터베이스 저장소(permanent database storage)와 혼동해서는 안 된다. Transient Local 동작은 미들웨어 통신 및 참여 엔티티(entity)의 수명과 자원에 연관되며, 영구 로깅(persistent logging), 데이터베이스, rosbag 기록 또는 장애 허용 저장소(fault-tolerant storage)를 대체하지 않는다. 로봇 아키텍트는 상태 복구와 시스템 재시작 동작을 설계할 때 통신 수준의 샘플 유지와 장기적인 데이터 영속성(data persistence)을 구분해야 한다.

이력(history)은 발행자와 구독자가 서로 다른 속도로 동작할 때 샘플을 어떻게 유지할지를 제어한다. 최근 항목 유지(Keep Last)는 제한된 수의 최신 샘플을 저장하고, 전체 항목 유지(Keep All)는 미들웨어 자원 제한 범위에서 모든 샘플을 보존하려고 시도한다. Keep Last는 일반적으로 유지되는 큐의 크기를 지정하는 깊이(depth) 값과 함께 사용된다. 이력과 깊이는 함께 일시적으로 얼마나 많은 데이터 적체(backlog)를 허용할지를 결정한다.

Keep Last의 깊이를 1로 설정하면 처리 전에 큐에 존재하는 이전 값이 새로운 값으로 대체될 수 있으므로 최신 상태를 강조할 수 있다. 더 큰 깊이는 콜백(callback)의 처리 속도가 일시적으로 발행 속도를 따라가지 못할 때 추가적인 버퍼링(buffering)을 제공한다. 그러나 깊이를 증가시키는 것이 항상 유리한 것은 아니다. 깊은 큐는 메모리를 소비하며 특히 고속 인지 및 제어 파이프라인에서 이미 운용 가치가 사라진 오래된 정보를 구독자가 뒤늦게 처리하게 만들 수 있다.

전체 항목 유지(Keep All)는 모든 샘플이 의미적으로 중요한 경우 적합할 수 있지만 세심한 자원 관리(resource management)가 필요하다. 생산자가 소비자의 처리 속도보다 빠르게 데이터를 생성하면 미들웨어 제한에 도달할 때까지 유지되는 샘플이 증가할 수 있다. CPU와 메모리가 제한된 로봇 시스템에서는 겉보기에는 손실 없는 큐(lossless queue)가 오히려 지연이나 자원 압박(resource pressure)을 발생시킬 수 있다. 따라서 통신 요구사항과 처리 용량 사이의 균형이 필요하다.

신뢰성, 내구성, 이력은 완전히 독립적으로 동작하기보다 서로 상호작용한다. 깊은 이력과 결합된 Reliable 발행자는 더 많은 정보를 유지하고 재전송할 수 있지만 더 많은 대역폭과 메모리를 소비할 수 있다. 얕은 Keep Last 이력과 결합된 Best Effort 발행자는 최신성과 제한된 자원 사용을 우선할 수 있다. Transient Local 발행자는 유지된 상태와 늦게 참여하는 구독자 동작을 결합하므로 이력 설정이 특히 중요해진다.

발행자와 구독자 사이의 QoS 호환성(QoS compatibility)은 필수적이다. 토픽 이름과 메시지 형식(message type)이 일치한다고 해서 요청 및 제공 QoS 정책이 호환되지 않는 상황에서도 통신이 성공하는 것은 아니다. 발행자는 통신 특성을 제공(offer)하고 구독자는 자신이 수용할 수 있는 특성을 요청(request)한다. ROS 2와 하위 DDS 미들웨어는 엔드포인트 사이의 통신을 설정할 때 이러한 정책을 평가하므로 QoS 불일치는 중요한 통합 실패 원인이 될 수 있다.

이러한 호환성 모델(compatibility model)은 독립적으로 개발된 ROS 2 패키지를 통합할 때 특히 중요하다. 센서 드라이버(sensor driver)는 스트리밍 측정값에 최적화된 QoS 프로파일을 사용할 수 있지만 사용자 정의 구독자는 신뢰성 있는 응용 데이터에 적합한 설정을 가정할 수 있다. 두 노드가 ROS 그래프에 표시되더라도 예상한 방식으로 샘플을 교환하지 못할 수 있다. 따라서 개발자는 통신 문제를 디버깅할 때 엔드포인트의 QoS 설정을 함께 확인해야 한다.

ROS 2는 일반적인 통신 패턴을 위한 사전 정의 QoS 프로파일(predefined QoS profile)과 설정 가능한 정책 조합을 제공한다. 센서 중심 통신은 일반적으로 최신성과 제한된 큐를 우선하고, 다른 인터페이스는 신뢰성 또는 유지된 상태를 강조할 수 있다. 이러한 프로파일은 유용한 출발점이지만 실제 운용 시스템에서는 하나의 프로파일이 모든 상황에 적합하다고 가정하지 말고 실제 메시지 주기, 페이로드 크기, 처리 지연, 네트워크 토폴로지(network topology), 장애 조건을 기준으로 검증해야 한다.

네트워크 특성은 실제 QoS 동작에 큰 영향을 준다. 안정적인 유선 네트워크에서는 Reliable 통신의 추가 부하가 허용될 수 있지만, 패킷 손실이나 가변적인 지연시간이 존재하는 무선 링크에서는 재전송과 큐 적체가 발생할 수 있다. 대용량 카메라 영상이나 포인트 클라우드(point cloud)는 이러한 영향을 더욱 증폭시킨다. 따라서 QoS 선택은 ROS 2 소프트웨어 아키텍처뿐 아니라 이더넷(Ethernet), Wi-Fi, 공유 대역폭, 미들웨어 설정 및 배포 토폴로지를 함께 고려해야 한다.

QoS는 종단 간 지연시간(end-to-end latency)에도 영향을 준다. 깊은 이력 큐는 데이터를 보존하는 동시에 느린 구독자가 처리하는 정보의 시간적 노후도(age)를 증가시킬 수 있다. Reliable 재전송은 전달 완전성을 높이는 대신 패킷 손실 상황에서 지연시간을 증가시킬 수 있다. Best Effort는 재전송 부담을 줄이는 대신 일부 샘플 손실을 허용한다. 자율 로봇에서는 단순한 최대 전달 완전성보다 운용에 유효하면서 적시에 제공되는 정보가 더 중요한 최적화 목표가 되는 경우가 많다.

진단(diagnostics)은 메시지 전송과 응용 프로그램 동작을 함께 검사해야 한다. 토픽 주기, 대역폭, 콜백 처리 시간, 손실된 샘플, 큐 깊이, 엔드포인트 QoS 설정은 서로 보완적인 진단 정보를 제공한다. 구독자가 데이터를 받지 못하거나 예상하지 못한 지연이 발생하면 문제를 하나의 계층에만 귀속시키기 전에 토픽 및 메시지 호환성과 함께 신뢰성, 내구성, 이력, 깊이, 네트워크 상태, 실행기(executor) 동작을 확인해야 한다.

다중 로봇(multi-robot) 및 분산 피지컬 AI(Physical AI) 시스템에서는 통신이 프로세스, 컴퓨터, 네트워크 세그먼트(network segment), 무선 링크, 플릿 인프라(fleet infrastructure)를 가로지를 수 있으므로 QoS 설계가 더욱 복잡해진다. 고대역폭 인지 데이터는 로봇 내부에 유지하고 압축된 상태 정보는 더 넓은 범위로 배포할 수 있다. 데이터 의미와 통신 범위에 따라 QoS를 선택하면 조정 및 감독에 필요한 정보를 유지하면서 불필요한 네트워크 트래픽을 줄일 수 있다.

실제 운용 ROS 2 시스템에서는 QoS를 각 인터페이스 계약(interface contract)의 일부로 정의해야 한다. 문서에는 특정 채널이 Reliable 또는 Best Effort 전달을 사용하는 이유, 늦게 참여하는 구독자에게 유지된 샘플이 필요한지 여부, 어느 정도의 이력이 운용상 의미가 있는지를 명시해야 한다. 명확한 QoS 계약은 통합 과정의 모호성을 줄이고 센서, 컴퓨팅 플랫폼, 미들웨어 구현, 네트워크 구성 전반에서 성능 시험을 재현 가능하게 만든다.

결국 신뢰성(reliability), 내구성(durability), 이력(history)은 서로 다른 차원의 통신 의도(communication intent)를 나타낸다. 신뢰성은 데이터 전달을 얼마나 강하게 보장하려고 할 것인지를 결정하고, 내구성은 이전 정보를 나중에 참여하는 구성요소에게 제공할 것인지를 결정하며, 이력은 통신이 진행되는 동안 몇 개의 샘플을 유지할 것인지를 결정한다. 이러한 차이를 이해하면 ROS 2 아키텍트는 로봇 데이터가 가지는 실제 시간적 의미와 운용 목적에 따라 적절한 통신 채널을 설계할 수 있다.

##  

## 04.05 Large Message Transfer: Image / PointCloud Optimization [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

Large-message transfer is a major performance concern in ROS 2 because robotics applications frequently exchange images, depth maps, point clouds, radar data, occupancy grids, and AI-related tensors whose payloads are much larger than ordinary control or state messages. The communication architecture must therefore consider not only message correctness but also serialization cost, memory copying, bandwidth consumption, latency, and receiver processing capacity.

A camera illustrates the scale of the problem. An uncompressed image contains width × height × channels × bytes-per-channel data, and the resulting payload is multiplied by the frame rate. Multiple cameras can therefore generate hundreds of megabytes per second before protocol overhead is considered. High-resolution RGB-D systems further increase traffic because color images and depth measurements may be transmitted simultaneously through separate ROS 2 topics.

Three-dimensional LiDAR creates a different but similarly demanding workload. A \`sensor_msgs/msg/PointCloud2\` message can contain large numbers of points with coordinates, intensity, timestamps, ring identifiers, semantic labels, or additional application-specific fields. Higher sensor resolution and scan frequency increase both message size and publication rate. When several perception nodes subscribe independently, memory movement and serialization overhead can become as important as raw network bandwidth.

Large-message performance should be analyzed as an end-to-end data path rather than only as network transmission. A typical path includes sensor acquisition, message construction, memory allocation, serialization, middleware queues, transport, deserialization, callback scheduling, and application processing. Any stage can become the dominant bottleneck. Optimizing Ethernet bandwidth alone may therefore provide little improvement when repeated memory copies or slow subscribers dominate total latency.

Serialization converts ROS 2 message structures into a representation suitable for middleware transport, while deserialization reconstructs them at the receiver. For small messages this cost may be negligible, but large images and point clouds can make serialization a significant CPU and memory-bandwidth workload. The cost becomes more visible when one large message must be independently serialized, copied, or delivered to several consumers at high frequency.

Memory copying is particularly important in perception pipelines. A camera frame may move from a device driver buffer into a ROS message, through middleware storage, into a subscriber buffer, and finally into an image-processing or AI inference framework. Repeated copies increase memory bandwidth consumption and latency while producing no additional information. Large-message optimization therefore attempts to reduce unnecessary ownership transitions and data duplication wherever the architecture permits.

ROS 2 composition can reduce this overhead when communicating nodes can execute inside the same process. Components placed in a shared process can use intra-process communication mechanisms that avoid portions of the conventional inter-process data path. When message ownership and API usage permit, data can be passed with fewer copies. This is especially valuable for tightly coupled pipelines such as camera acquisition, preprocessing, perception, and local visualization.

Loaned-message and zero-copy-oriented mechanisms can further reduce memory movement when supported by the selected ROS 2 middleware, message path, and platform. Instead of repeatedly allocating and copying a large payload, middleware-managed memory can potentially be used more directly. However, zero-copy behavior should never be assumed merely because an API exposes related concepts. Actual benefits depend on RMW implementation, DDS capabilities, transport configuration, and deployment topology.

Communication across processes or computers introduces different constraints. Intra-process optimizations cannot eliminate network transfer when publishers and subscribers execute on separate machines. The architecture must then consider physical link capacity, protocol overhead, packet loss, middleware fragmentation, retransmission behavior, operating-system buffers, and competing traffic. Gigabit Ethernet can become a practical limitation surprisingly quickly when multiple high-resolution sensors publish simultaneously.

DDS and RTPS may fragment payloads that exceed practical transport packet sizes. Large ROS 2 messages can therefore become many lower-level fragments whose delivery and reconstruction create additional processing and buffering requirements. Packet loss can be particularly expensive with reliable communication because missing information may trigger recovery or retransmission behavior. Large payloads consequently make QoS and network quality closely coupled to observed application latency.

QoS selection should reflect the temporal value of large sensor data. A camera subscriber performing real-time perception may prefer a shallow queue and Best Effort reliability because processing the newest image is more valuable than receiving an old frame after retransmission. A recording or analysis pipeline may have different requirements and prioritize completeness. The same underlying sensor data can therefore require different communication strategies according to the consumer\'s purpose.

History depth must also remain bounded according to realistic processing capacity. If a camera publishes faster than a perception node can process, a deep queue does not solve the computational deficit; it merely stores increasingly stale frames. For real-time robotic perception, keeping only a small number of recent samples often provides more useful behavior. Queue depth should be chosen according to acceptable data age, callback execution time, and temporary processing jitter.

Reducing publication rate is one of the simplest optimization techniques when downstream components do not require every sensor sample. A camera operating at 60 frames per second does not imply that every consumer needs 60 Hz data. Mapping, visualization, monitoring, AI inference, and logging may operate at different effective rates. Separating these requirements prevents high-rate raw streams from propagating unnecessarily through the complete robot software architecture.

Payload reduction provides another optimization dimension. Images may be resized, cropped to a region of interest, converted to a lower-cost representation, or compressed when the resulting processing and quality trade-offs are acceptable. Point clouds can be filtered, downsampled, cropped spatially, or transformed into more task-specific representations. Such optimization is most effective when unnecessary information is removed near the source rather than after expensive transport.

Compression reduces communication bandwidth at the cost of additional computation and often additional latency. Compressed image transport can be useful across constrained networks, but local GPU perception pipelines may prefer uncompressed or hardware-friendly formats to avoid encode-decode overhead. Point-cloud compression presents similar trade-offs. The correct choice depends on whether the limiting resource is network bandwidth, CPU/GPU processing, memory bandwidth, storage capacity, or latency.

Robot architectures should distinguish local high-bandwidth data from information that genuinely needs system-wide distribution. Raw camera and LiDAR streams may remain within an edge-compute perception domain, while derived objects, tracks, occupancy information, localization states, or compact semantic representations are distributed to planning and fleet systems. This hierarchical approach prevents expensive sensor payloads from becoming global communication dependencies.

Multi-robot systems amplify the importance of this separation. Broadcasting raw sensor streams from every robot across a shared wireless network can consume available capacity rapidly and degrade communication needed for coordination or safety-related state exchange. Robot-local perception should therefore process large data close to its source whenever possible, while fleet communication carries compact mission, state, diagnostic, and coordination information appropriate to the wider network scope.

Subscriber architecture also affects performance because every consumer adds processing demand. Visualization, recording, diagnostics, perception, and debugging tools can unintentionally compete for the same CPU, memory, and network resources. A system that performs well with one subscriber may behave differently when several tools are attached. Performance validation should therefore reproduce realistic production subscriber counts rather than measuring only an isolated publisher-to-subscriber pair.

Monitoring must include message size, publication frequency, effective bandwidth, end-to-end latency, CPU utilization, memory consumption, queue behavior, dropped samples, and subscriber processing time. These measurements help distinguish transport bottlenecks from application bottlenecks. ROS 2 topic inspection, tracing, middleware diagnostics, operating-system monitoring, and network analysis can be combined to identify where large-message latency is actually introduced.

Optimization should be validated under realistic failure and congestion conditions rather than only on an idle laboratory network. Wireless degradation, packet loss, simultaneous sensor activity, recording workloads, AI inference, and multiple subscribers can expose behavior that is invisible in simple tests. Reliable communication may accumulate retransmission pressure, queues may grow, and processing latency may increase even though average bandwidth appears acceptable during nominal operation.

Large-message transfer is ultimately an architectural problem rather than a single ROS 2 parameter to tune. Efficient systems combine appropriate message representations, bounded QoS queues, suitable reliability, composition, intra-process communication, reduced copying, controlled publication rates, payload reduction, and network-aware deployment. These techniques allow images and point clouds to remain useful inputs to high-performance perception without overwhelming the communication infrastructure.

For autonomous robots and Physical AI systems, the preferred architecture keeps high-volume raw perception data close to the computation that consumes it and distributes increasingly compact information toward planning, control, mission management, and fleet layers. Treating bandwidth, memory movement, serialization, latency, and data freshness as a unified design problem enables ROS 2 communication to scale from individual sensors to complex multi-computer and multi-robot systems.

대용량 메시지 전송(large-message transfer)은 ROS 2에서 중요한 성능 고려사항이다. 로보틱스 응용 프로그램은 일반적인 제어 또는 상태 메시지보다 훨씬 큰 페이로드(payload)를 가진 영상(image), 깊이 맵(depth map), 포인트 클라우드(point cloud), 레이더 데이터(radar data), 점유 격자(occupancy grid), AI 관련 텐서(tensor)를 빈번하게 교환한다. 따라서 통신 아키텍처는 메시지의 정확성뿐 아니라 직렬화 비용(serialization cost), 메모리 복사(memory copying), 대역폭 소비, 지연시간(latency), 수신측 처리 용량을 함께 고려해야 한다.

카메라는 이러한 문제의 규모를 보여주는 대표적인 사례이다. 비압축 영상(uncompressed image)의 데이터 크기는 영상 폭 × 높이 × 채널 수 × 채널당 바이트 수로 결정되며, 실제 데이터 전송량은 여기에 프레임률(frame rate)이 곱해진다. 따라서 여러 카메라는 프로토콜 오버헤드(protocol overhead)를 고려하기 전부터 초당 수백 메가바이트의 데이터를 생성할 수 있다. 고해상도 RGB-D 시스템은 컬러 영상과 깊이 측정값을 별도의 ROS 2 토픽으로 동시에 전송할 수 있으므로 트래픽을 더욱 증가시킨다.

3차원 LiDAR는 다른 형태이지만 마찬가지로 높은 통신 부하를 발생시킨다. \`sensor_msgs/msg/PointCloud2\` 메시지는 좌표, 반사 강도(intensity), 타임스탬프(timestamp), 링 식별자(ring identifier), 의미론적 레이블(semantic label), 응용 프로그램별 추가 필드를 가진 많은 수의 포인트를 포함할 수 있다. 센서 해상도와 스캔 주기가 증가하면 메시지 크기와 발행 주기가 모두 증가한다. 여러 인지 노드가 독립적으로 구독하면 메모리 이동과 직렬화 오버헤드가 원시 네트워크 대역폭만큼 중요한 문제가 될 수 있다.

대용량 메시지 성능은 네트워크 전송만이 아니라 종단 간 데이터 경로(end-to-end data path)의 관점에서 분석해야 한다. 일반적인 경로에는 센서 데이터 획득, 메시지 생성, 메모리 할당(memory allocation), 직렬화, 미들웨어 큐(middleware queue), 전송, 역직렬화(deserialization), 콜백 스케줄링(callback scheduling), 응용 프로그램 처리가 포함된다. 이 가운데 어느 단계든 주요 병목(bottleneck)이 될 수 있으므로 반복적인 메모리 복사나 느린 구독자가 전체 지연을 지배한다면 이더넷 대역폭만 최적화해도 성능 개선은 제한적일 수 있다.

직렬화(serialization)는 ROS 2 메시지 구조를 미들웨어 전송에 적합한 표현으로 변환하고, 역직렬화는 수신 측에서 이를 다시 메시지 구조로 복원한다. 작은 메시지에서는 이러한 비용을 무시할 수 있지만 대용량 영상과 포인트 클라우드에서는 직렬화가 상당한 CPU 및 메모리 대역폭 부하를 발생시킬 수 있다. 하나의 대용량 메시지를 여러 소비자에게 높은 주기로 전달하면서 각각 직렬화하거나 복사해야 하는 경우 이러한 비용은 더욱 두드러진다.

메모리 복사(memory copying)는 특히 인지 파이프라인(perception pipeline)에서 중요하다. 카메라 프레임은 장치 드라이버 버퍼(device driver buffer)에서 ROS 메시지로 이동하고, 다시 미들웨어 저장 영역과 구독자 버퍼를 거쳐 영상 처리 또는 AI 추론 프레임워크(AI inference framework)로 전달될 수 있다. 반복적인 복사는 새로운 정보를 생성하지 않으면서 메모리 대역폭 소비와 지연시간을 증가시킨다. 따라서 대용량 메시지 최적화에서는 아키텍처가 허용하는 범위에서 불필요한 소유권 전환(ownership transition)과 데이터 복제를 줄이는 것이 중요하다.

ROS 2 컴포지션(composition)은 통신 노드들이 동일한 프로세스 안에서 실행될 수 있을 때 이러한 오버헤드를 줄일 수 있다. 공유 프로세스에 배치된 컴포넌트(component)는 일반적인 프로세스 간 데이터 경로의 일부를 회피하는 프로세스 내부 통신(intra-process communication)을 사용할 수 있다. 메시지 소유권과 API 사용 방식이 허용한다면 더 적은 복사로 데이터를 전달할 수 있으며, 이는 카메라 획득, 전처리(preprocessing), 인지, 로컬 시각화가 긴밀하게 연결된 파이프라인에서 특히 유용하다.

대여 메시지(loaned message)와 제로 카피 지향 메커니즘(zero-copy-oriented mechanism)은 선택한 ROS 2 미들웨어, 메시지 경로 및 플랫폼에서 지원되는 경우 메모리 이동을 더욱 줄일 수 있다. 대용량 페이로드를 반복적으로 할당하고 복사하는 대신 미들웨어가 관리하는 메모리를 보다 직접적으로 사용할 수 있다. 그러나 관련 API가 제공된다는 이유만으로 실제 제로 카피가 동작한다고 가정해서는 안 된다. 실제 효과는 RMW 구현, DDS 기능, 전송 설정, 배포 토폴로지(deployment topology)에 따라 달라진다.

프로세스 또는 컴퓨터 사이의 통신에서는 다른 제약조건이 발생한다. 발행자와 구독자가 서로 다른 컴퓨터에서 실행된다면 프로세스 내부 최적화로 네트워크 전송을 제거할 수 없다. 이 경우 물리적 링크 용량, 프로토콜 오버헤드, 패킷 손실(packet loss), 미들웨어 단편화(fragmentation), 재전송 동작, 운영체제 버퍼, 경쟁 트래픽을 고려해야 한다. 여러 고해상도 센서가 동시에 데이터를 발행하면 기가비트 이더넷(Gigabit Ethernet)도 예상보다 빠르게 실질적인 병목이 될 수 있다.

DDS와 실시간 발행-구독 프로토콜(Real-Time Publish-Subscribe Protocol, RTPS)은 실제 전송 패킷 크기를 초과하는 페이로드를 여러 조각으로 단편화할 수 있다. 따라서 대용량 ROS 2 메시지는 많은 하위 수준 프래그먼트(fragment)로 나뉘며, 이를 전달하고 다시 조립하는 과정에서 추가적인 처리와 버퍼링이 필요하다. 신뢰성 통신에서는 누락된 정보가 복구 또는 재전송 동작을 유발할 수 있으므로 패킷 손실의 비용이 특히 커질 수 있다. 따라서 대용량 페이로드에서는 QoS와 네트워크 품질이 실제 응용 프로그램 지연시간과 밀접하게 연결된다.

QoS 선택은 대용량 센서 데이터가 가지는 시간적 가치(temporal value)를 반영해야 한다. 실시간 인지를 수행하는 카메라 구독자는 재전송된 오래된 프레임을 처리하는 것보다 최신 영상을 처리하는 것이 중요하므로 얕은 큐와 최선형 신뢰성(Best Effort reliability)을 선호할 수 있다. 반면 기록 또는 분석 파이프라인은 완전성을 더 중요하게 고려할 수 있다. 따라서 동일한 센서 데이터라도 소비자의 목적에 따라 서로 다른 통신 전략이 필요할 수 있다.

이력 깊이(history depth) 역시 실제 처리 용량에 따라 제한해야 한다. 카메라가 인지 노드의 처리 속도보다 빠르게 영상을 발행한다면 깊은 큐를 사용해도 계산 능력 부족 자체가 해결되는 것은 아니며 점점 오래된 프레임을 저장할 뿐이다. 실시간 로봇 인지에서는 소수의 최신 샘플만 유지하는 것이 더 유용한 경우가 많다. 큐 깊이는 허용 가능한 데이터 시간 경과(data age), 콜백 실행 시간, 일시적인 처리 지터(processing jitter)를 기준으로 선택해야 한다.

하위 구성요소가 모든 센서 샘플을 요구하지 않는다면 발행 주기(publication rate)를 줄이는 것이 가장 단순한 최적화 방법 중 하나이다. 카메라가 초당 60프레임으로 동작한다고 해서 모든 소비자가 60 Hz 데이터를 필요로 하는 것은 아니다. 매핑(mapping), 시각화, 모니터링, AI 추론, 로깅(logging)은 서로 다른 유효 주기로 동작할 수 있다. 이러한 요구사항을 분리하면 고주기 원시 데이터 스트림이 전체 로봇 소프트웨어 아키텍처에 불필요하게 전파되는 것을 방지할 수 있다.

페이로드 감소(payload reduction)는 또 다른 최적화 방법이다. 영상은 처리 성능과 품질 사이의 절충이 허용되는 경우 크기 조정(resizing), 관심 영역(region of interest) 자르기, 더 낮은 비용의 표현으로 변환 또는 압축할 수 있다. 포인트 클라우드는 필터링(filtering), 다운샘플링(downsampling), 공간적 크롭(cropping) 또는 작업에 특화된 표현으로 변환할 수 있다. 이러한 최적화는 불필요한 정보를 비싼 전송 과정 이후가 아니라 데이터 소스에 가까운 위치에서 제거할 때 가장 효과적이다.

압축(compression)은 추가적인 계산과 경우에 따라 추가적인 지연시간을 대가로 통신 대역폭을 줄인다. 압축 영상 전송은 제한된 네트워크에서 유용할 수 있지만 로컬 GPU 인지 파이프라인은 인코딩-디코딩(encode-decode) 오버헤드를 피하기 위해 비압축 또는 하드웨어 친화적인 형식을 선호할 수 있다. 포인트 클라우드 압축도 유사한 절충 관계를 가진다. 올바른 선택은 제한 요소가 네트워크 대역폭, CPU/GPU 처리량, 메모리 대역폭, 저장 용량 또는 지연시간 중 무엇인지에 따라 결정된다.

로봇 아키텍처는 로컬 고대역폭 데이터(local high-bandwidth data)와 실제로 시스템 전체에 배포해야 하는 정보를 구분해야 한다. 원시 카메라 및 LiDAR 스트림은 엣지 컴퓨팅 인지 영역(edge-compute perception domain) 내부에 유지하고, 여기에서 생성된 객체, 추적 정보(track), 점유 정보, 위치 추정 상태 또는 압축된 의미론적 표현(compact semantic representation)을 계획 및 플릿 시스템으로 전달할 수 있다. 이러한 계층적 접근 방식은 고비용 센서 페이로드가 전역 통신 종속성이 되는 것을 방지한다.

다중 로봇 시스템(multi-robot system)에서는 이러한 분리의 중요성이 더욱 커진다. 모든 로봇의 원시 센서 스트림을 공유 무선 네트워크로 브로드캐스트하면 가용 대역폭을 빠르게 소비하여 협업이나 안전 관련 상태 교환에 필요한 통신 성능까지 저하시킬 수 있다. 따라서 가능한 경우 로봇 로컬 인지(robot-local perception)는 대용량 데이터를 데이터 소스 가까이에서 처리하고, 플릿 통신(fleet communication)은 넓은 네트워크 범위에 적합한 압축된 임무, 상태, 진단 및 협업 정보를 전달해야 한다.

각 소비자가 추가적인 처리 부하를 발생시키므로 구독자 아키텍처(subscriber architecture)도 성능에 영향을 준다. 시각화, 기록, 진단, 인지 및 디버깅 도구는 의도하지 않게 동일한 CPU, 메모리 및 네트워크 자원을 두고 경쟁할 수 있다. 하나의 구독자가 있을 때 정상적으로 동작하는 시스템도 여러 도구를 연결하면 다른 성능 특성을 보일 수 있다. 따라서 성능 검증은 단순한 발행자-구독자 한 쌍이 아니라 실제 운용 환경의 구독자 수를 재현하여 수행해야 한다.

모니터링(monitoring)은 메시지 크기, 발행 주기, 실제 대역폭, 종단 간 지연시간, CPU 사용률, 메모리 소비량, 큐 동작, 손실된 샘플, 구독자 처리 시간을 포함해야 한다. 이러한 측정값은 전송 병목과 응용 프로그램 병목을 구분하는 데 도움을 준다. ROS 2 토픽 검사, 트레이싱(tracing), 미들웨어 진단, 운영체제 모니터링 및 네트워크 분석을 함께 사용하면 대용량 메시지의 지연이 실제로 어느 지점에서 발생하는지 파악할 수 있다.

최적화는 유휴 상태의 실험실 네트워크에서만 검증할 것이 아니라 실제와 유사한 장애 및 혼잡 조건에서도 검증해야 한다. 무선 통신 품질 저하, 패킷 손실, 여러 센서의 동시 동작, 데이터 기록 부하, AI 추론, 다수 구독자는 단순한 시험에서는 나타나지 않는 동작을 드러낼 수 있다. 정상 상태에서 평균 대역폭이 허용 가능한 수준으로 보이더라도 신뢰성 통신의 재전송 부하가 누적되고 큐가 증가하며 처리 지연시간이 커질 수 있다.

대용량 메시지 전송은 궁극적으로 하나의 ROS 2 매개변수만 조정하여 해결할 수 있는 문제가 아니라 아키텍처 문제이다. 효율적인 시스템은 적절한 메시지 표현, 제한된 QoS 큐, 적합한 신뢰성 정책, 컴포지션, 프로세스 내부 통신, 메모리 복사 감소, 제어된 발행 주기, 페이로드 감소, 네트워크를 고려한 배포를 결합한다. 이러한 기법을 통해 영상과 포인트 클라우드를 고성능 인지의 핵심 입력으로 유지하면서도 통신 인프라가 과부하되는 것을 방지할 수 있다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 바람직한 아키텍처는 대용량 원시 인지 데이터(raw perception data)를 이를 소비하는 연산 장치 가까이에 유지하고, 계획, 제어, 임무 관리 및 플릿 계층으로 갈수록 더욱 압축되고 추상화된 정보를 배포하는 것이다. 대역폭, 메모리 이동, 직렬화, 지연시간, 데이터 최신성(data freshness)을 하나의 통합된 설계 문제로 다루면 ROS 2 통신을 개별 센서 수준에서 복잡한 다중 컴퓨터 및 다중 로봇 시스템까지 확장할 수 있다.

##  

## 04.06 Serialization Optimization: CDR / MCAP [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 serialization is the process of converting structured message data into a byte representation that can be transported, stored, or reconstructed by another communication endpoint. Although serialization is largely hidden behind ROS 2 APIs, its cost becomes important for high-frequency and large-payload workloads. Images, point clouds, maps, and recorded sensor streams can make serialization, memory allocation, copying, and storage throughput significant parts of total system latency.

ROS 2 commonly operates through DDS-based middleware, where message data is represented for transport using the Common Data Representation, or CDR, serialization model. CDR defines how typed fields such as integers, floating-point values, arrays, strings, and nested structures are encoded into a portable binary representation. This allows distributed endpoints with compatible interfaces to reconstruct equivalent message structures while hiding machine-specific memory layouts.

CDR serialization must consider field ordering, alignment, padding, variable-length data, and nested message structures. Primitive values can often be encoded efficiently, while strings, sequences, and dynamically sized arrays introduce additional processing and memory-management requirements. Consequently, two ROS 2 messages containing similar amounts of useful information can exhibit different serialization costs depending on how their interfaces and variable-length fields are structured.

For small control messages, serialization overhead is generally minor compared with application processing and communication latency. The situation changes with \`sensor_msgs/msg/Image\`, \`sensor_msgs/msg/PointCloud2\`, occupancy grids, and other data-intensive interfaces. Repeated serialization of megabyte-scale messages at tens of hertz can consume measurable CPU time and memory bandwidth, particularly when the same data is delivered to multiple subscribers or crosses process boundaries.

Serialization cost is only one component of the complete data path. A message may first require allocation and construction, followed by serialization into middleware buffers, transport, deserialization, additional copying, and conversion into application-specific representations. Optimizing only the serializer can therefore leave the dominant bottleneck untouched. Performance engineering should measure the complete publisher-to-consumer path and identify where time and memory traffic are actually spent.

Message design can reduce serialization overhead by avoiding unnecessary data and excessive dynamic allocation. Large structures should contain information that consumers genuinely require, while redundant fields and repeated conversions should be minimized. Fixed or bounded representations can provide more predictable resource behavior when appropriate. However, interface optimization should preserve semantic clarity rather than producing obscure message formats solely to reduce a small amount of serialization work.

Memory allocation frequently interacts with serialization performance. Repeatedly allocating large buffers for every image or point cloud can increase CPU overhead, memory fragmentation, and latency variability. Reusing buffers, reserving capacity where APIs permit, and avoiding unnecessary intermediate representations can improve predictability. These techniques are particularly valuable in sustained sensor pipelines where essentially the same message structure is processed thousands of times during operation.

Intra-process communication can bypass portions of conventional serialization when compatible ROS 2 components execute within the same process. Rather than converting a message into a transport representation and immediately reconstructing it for another local component, ownership-aware transfer can reduce copying and serialization work. This makes composition especially useful for tightly coupled high-bandwidth pipelines such as camera acquisition, preprocessing, AI inference, and perception.

Loaned messages provide another optimization mechanism when supported by the ROS middleware implementation and transport path. Middleware-managed memory can allow publishers to populate buffers without creating as many intermediate copies, while compatible subscriber paths may further reduce movement of large payloads. The achievable benefit depends on RMW and DDS implementation details, message characteristics, operating system support, and whether communication remains local or crosses a network.

Zero-copy should therefore be treated as a deployment-dependent optimization rather than a universal ROS 2 guarantee. A pipeline described as zero-copy at one layer may still contain copies in a sensor driver, middleware boundary, network stack, accelerator interface, or application framework. Verification requires measurement of the actual deployed data path. Architectural documentation should distinguish theoretical zero-copy capability from demonstrated end-to-end copy reduction.

Serialization requirements change when communication crosses computer boundaries. Data must ultimately be represented in a form suitable for network transport, and large serialized payloads may be fragmented by the middleware and transport protocols. At this point, CPU serialization cost interacts with network bandwidth, packetization, QoS reliability, retransmission, and buffering. Optimizing message representation can therefore improve both processor utilization and network efficiency.

Recording introduces another serialization-related path because ROS 2 messages may also be written to persistent storage for debugging, validation, dataset construction, and later replay. High-rate cameras, LiDARs, IMUs, and robot-state topics can generate large recording workloads. Storage format, indexing strategy, compression, disk throughput, and metadata organization then become part of communication performance rather than merely offline data-management concerns.

MCAP is a container format designed for storing timestamped multimodal data and is widely applicable to robotics logging workflows. It can organize multiple channels and message schemas within a single recording while supporting metadata and indexing needed for efficient access. In ROS 2 environments, MCAP can serve as a rosbag2 storage format, allowing heterogeneous topic streams to be recorded in a structured container for later inspection, replay, conversion, or analysis.

CDR and MCAP address different layers and should not be treated as competing serialization technologies. CDR primarily concerns the binary representation used to encode typed message data for middleware-oriented communication, while MCAP is a container format for organizing recorded messages, schemas, channels, timestamps, and related metadata. An MCAP recording can therefore contain ROS 2 message data whose serialized representation is associated with ROS middleware conventions.

This distinction is important when optimizing a recording pipeline. Changing the container format does not automatically eliminate the computational cost of creating serialized message data, and optimizing CDR transport does not solve storage indexing or disk-throughput limitations. A complete logging architecture should separately consider message serialization, transfer into the recording subsystem, container organization, compression, file writing, indexing, and subsequent replay requirements.

Compression can substantially reduce MCAP or rosbag storage requirements when recorded data contains compressible information, but compression consumes processing resources. The optimal strategy depends on whether the system is constrained primarily by CPU capacity, disk bandwidth, storage capacity, or replay speed. A robot performing intensive AI inference may prefer lower recording CPU overhead, while a long-duration data-collection platform may prioritize reducing storage consumption.

Chunking and indexing influence how recorded data is written and later accessed. Grouping messages into chunks can improve storage efficiency and enable compression over useful blocks of data, while indexes allow tools to locate messages without scanning an entire recording sequentially. These features become particularly important for multi-hour robotics datasets containing synchronized camera, LiDAR, localization, control, and diagnostic streams.

Timestamp integrity is equally important in recorded multimodal systems. Efficient serialization and storage have limited value if sensor streams cannot later be aligned correctly. Recording architectures should preserve the timing information needed to correlate images, point clouds, IMU measurements, robot poses, control commands, and AI outputs. This requirement connects serialization and storage design with the broader ROS 2 topics of synchronization, clock management, and multi-sensor data fusion.

Recording throughput should be validated under the same workload used during actual robot operation. A storage system that handles one camera during development may fail when several cameras, point clouds, and diagnostics are recorded simultaneously while perception and control are active. Monitoring CPU usage, write throughput, queue growth, dropped messages, file size, and latency helps determine whether the bottleneck lies in serialization, compression, storage, or application processing.

For Physical AI dataset collection, MCAP-based recording can provide a practical boundary between online robot execution and offline learning pipelines. Raw sensor observations, robot states, commands, perception outputs, and mission events can be captured together and later transformed into training-specific formats. Maintaining clear schemas and timestamps preserves traceability between what the robot observed, what its software inferred, and what actions were executed.

A scalable ROS 2 architecture therefore separates optimization goals according to the data path. Online communication focuses on latency, bounded queues, reduced copying, appropriate CDR handling, and efficient middleware transport. Recording focuses on sustained throughput, storage organization, indexing, compression, timestamp integrity, and reliable replay. Some optimizations benefit both paths, but each must be measured according to its own operational requirements.

Serialization optimization ultimately requires reducing unnecessary transformations rather than simply making every individual conversion faster. Well-designed systems minimize redundant message fields, avoid repeated allocation and copying, exploit intra-process and loaned-message capabilities where verified, and keep high-bandwidth data close to its consumers. For recorded workloads, efficient MCAP organization complements these techniques by providing structured storage for large, synchronized ROS 2 datasets.

In autonomous robots and Physical AI systems, CDR and MCAP occupy complementary positions in the information lifecycle. CDR supports interoperable typed data representation within ROS 2 middleware communication, while MCAP supports structured preservation of timestamped multimodal streams for replay and analysis. Understanding where serialization, copying, transport, recording, compression, and indexing occur enables engineers to optimize the complete path from live sensor acquisition to long-term robotic data use.

ROS 2 직렬화(serialization)는 구조화된 메시지 데이터를 전송하거나 저장하거나 다른 통신 엔드포인트(communication endpoint)에서 재구성할 수 있는 바이트 표현(byte representation)으로 변환하는 과정이다. 직렬화는 대부분 ROS 2 API 뒤에서 처리되지만, 고주기 및 대용량 페이로드 작업에서는 그 비용이 중요해진다. 영상, 포인트 클라우드(point cloud), 맵(map), 기록된 센서 스트림은 직렬화, 메모리 할당(memory allocation), 복사(copying), 저장 처리량(storage throughput)을 전체 시스템 지연시간의 중요한 요소로 만들 수 있다.

ROS 2는 일반적으로 DDS 기반 미들웨어(DDS-based middleware)를 통해 동작하며, 메시지 데이터는 전송을 위해 공통 데이터 표현(Common Data Representation, CDR) 직렬화 모델을 사용하여 표현된다. CDR은 정수, 부동소수점 값, 배열(array), 문자열(string), 중첩 구조(nested structure)와 같은 형식화된 필드를 이식 가능한 바이너리 표현(portable binary representation)으로 인코딩하는 방법을 정의한다. 이를 통해 호환 가능한 인터페이스를 가진 분산 엔드포인트가 기계별 메모리 배치(machine-specific memory layout)를 숨기면서 동일한 메시지 구조를 재구성할 수 있다.

CDR 직렬화(CDR serialization)는 필드 순서(field ordering), 정렬(alignment), 패딩(padding), 가변 길이 데이터(variable-length data), 중첩 메시지 구조를 고려해야 한다. 기본 자료형(primitive value)은 비교적 효율적으로 인코딩할 수 있지만 문자열, 시퀀스(sequence), 동적 크기 배열(dynamically sized array)은 추가적인 처리와 메모리 관리 요구사항을 발생시킨다. 따라서 유효 정보량이 비슷한 두 ROS 2 메시지도 인터페이스와 가변 길이 필드의 구성 방식에 따라 서로 다른 직렬화 비용을 가질 수 있다.

작은 제어 메시지에서는 직렬화 오버헤드(serialization overhead)가 일반적으로 응용 프로그램 처리 및 통신 지연에 비해 크지 않다. 그러나 \`sensor_msgs/msg/Image\`, \`sensor_msgs/msg/PointCloud2\`, 점유 격자(occupancy grid) 및 기타 데이터 집약적 인터페이스에서는 상황이 달라진다. 메가바이트 규모의 메시지를 초당 수십 회 반복적으로 직렬화하면 상당한 CPU 시간과 메모리 대역폭을 소비할 수 있으며, 동일한 데이터가 여러 구독자에게 전달되거나 프로세스 경계를 통과할 경우 이러한 영향은 더욱 커진다.

직렬화 비용은 전체 데이터 경로를 구성하는 하나의 요소일 뿐이다. 메시지는 먼저 메모리 할당 및 생성 과정을 거친 후 미들웨어 버퍼(middleware buffer)로 직렬화되고, 전송, 역직렬화(deserialization), 추가 복사, 응용 프로그램별 표현(application-specific representation)으로의 변환 과정을 거칠 수 있다. 따라서 직렬화기(serializer)만 최적화하면 실제 주요 병목이 그대로 남을 수 있다. 성능 엔지니어링은 전체 발행자-소비자 경로를 측정하여 시간과 메모리 트래픽이 실제로 어디에서 소비되는지 확인해야 한다.

메시지 설계(message design)는 불필요한 데이터와 과도한 동적 메모리 할당(dynamic allocation)을 방지함으로써 직렬화 오버헤드를 줄일 수 있다. 대규모 구조는 소비자가 실제로 필요로 하는 정보를 포함해야 하며 중복 필드와 반복적인 변환은 최소화해야 한다. 적절한 경우 고정 또는 제한된 표현(fixed or bounded representation)을 사용하여 자원 동작을 더욱 예측 가능하게 만들 수 있다. 그러나 작은 직렬화 비용을 줄이기 위해 의미적으로 불명확한 메시지 형식을 만드는 것보다는 인터페이스의 명확성을 유지하는 것이 중요하다.

메모리 할당은 직렬화 성능과 밀접하게 상호작용하는 경우가 많다. 모든 영상이나 포인트 클라우드에 대해 대용량 버퍼를 반복적으로 할당하면 CPU 오버헤드, 메모리 단편화(memory fragmentation), 지연시간 변동성이 증가할 수 있다. API가 허용하는 경우 버퍼를 재사용하고 용량을 미리 확보(reserve)하며 불필요한 중간 표현(intermediate representation)을 피하면 예측 가능성을 향상시킬 수 있다. 이러한 기법은 동일한 메시지 구조가 운용 중 수천 번 반복 처리되는 지속적인 센서 파이프라인에서 특히 유용하다.

프로세스 내부 통신(intra-process communication)은 호환되는 ROS 2 컴포넌트(component)가 동일한 프로세스 안에서 실행될 때 일반적인 직렬화 과정의 일부를 우회할 수 있다. 메시지를 전송 형식으로 변환한 직후 다른 로컬 컴포넌트에서 다시 복원하는 대신 소유권을 고려한 전송(ownership-aware transfer)을 통해 복사와 직렬화 작업을 줄일 수 있다. 따라서 컴포지션(composition)은 카메라 획득, 전처리(preprocessing), AI 추론(AI inference), 인지(perception)처럼 긴밀하게 연결된 고대역폭 파이프라인에서 특히 유용하다.

대여 메시지(loaned message)는 ROS 미들웨어 구현과 전송 경로에서 지원되는 경우 사용할 수 있는 또 다른 최적화 메커니즘이다. 미들웨어 관리 메모리(middleware-managed memory)를 사용하면 발행자가 다수의 중간 복사본을 생성하지 않고 버퍼를 채울 수 있으며, 호환 가능한 구독자 경로에서는 대용량 페이로드의 이동을 더욱 줄일 수 있다. 실제로 얻을 수 있는 효과는 RMW와 DDS 구현 세부사항, 메시지 특성, 운영체제 지원, 통신이 로컬에 유지되는지 또는 네트워크를 통과하는지에 따라 달라진다.

따라서 제로 카피(zero-copy)는 ROS 2에서 보편적으로 보장되는 기능이 아니라 배포 환경에 따라 달라지는 최적화 기법으로 다루어야 한다. 특정 계층에서 제로 카피로 설명되는 파이프라인도 센서 드라이버, 미들웨어 경계, 네트워크 스택(network stack), 가속기 인터페이스(accelerator interface), 응용 프레임워크 내부에서는 여전히 데이터 복사가 발생할 수 있다. 실제 배포된 데이터 경로를 측정하여 검증해야 하며, 아키텍처 문서에서는 이론적인 제로 카피 기능과 실제로 검증된 종단 간 복사 감소를 구분해야 한다.

통신이 컴퓨터 경계를 넘어가면 직렬화 요구사항도 달라진다. 데이터는 결국 네트워크 전송에 적합한 형태로 표현되어야 하며, 대용량 직렬화 페이로드는 미들웨어와 전송 프로토콜에 의해 단편화(fragmentation)될 수 있다. 이 단계에서 CPU 직렬화 비용은 네트워크 대역폭, 패킷화(packetization), QoS 신뢰성, 재전송, 버퍼링과 상호작용한다. 따라서 메시지 표현을 최적화하면 프로세서 사용률과 네트워크 효율을 동시에 개선할 수 있다.

기록(recording)은 ROS 2 메시지를 디버깅, 검증, 데이터셋 구축, 이후 재생(replay)을 위해 영구 저장소에 기록할 수 있기 때문에 또 다른 직렬화 관련 경로를 형성한다. 고주기 카메라, LiDAR, 관성 측정 장치(Inertial Measurement Unit, IMU), 로봇 상태 토픽은 대규모 기록 부하를 생성할 수 있다. 이 경우 저장 형식(storage format), 인덱싱 전략(indexing strategy), 압축(compression), 디스크 처리량(disk throughput), 메타데이터 구성(metadata organization)이 단순한 오프라인 데이터 관리가 아니라 통신 성능의 일부가 된다.

MCAP은 타임스탬프가 포함된 다중 모달 데이터(timestamped multimodal data)를 저장하도록 설계된 컨테이너 형식(container format)으로 로보틱스 로깅 작업에 폭넓게 활용할 수 있다. 하나의 기록 파일 안에서 여러 채널(channel)과 메시지 스키마(message schema)를 구성하고 효율적인 접근에 필요한 메타데이터와 인덱싱을 지원한다. ROS 2 환경에서 MCAP은 rosbag2 저장 형식(storage format)으로 사용될 수 있으며, 서로 다른 토픽 스트림을 구조화된 컨테이너에 기록하여 이후 검사, 재생, 변환 또는 분석에 활용할 수 있다.

CDR과 MCAP은 서로 다른 계층을 담당하므로 경쟁하는 직렬화 기술로 간주해서는 안 된다. CDR은 주로 미들웨어 기반 통신을 위해 형식화된 메시지 데이터를 인코딩하는 바이너리 표현을 담당하는 반면, MCAP은 기록된 메시지, 스키마, 채널, 타임스탬프 및 관련 메타데이터를 구성하는 컨테이너 형식이다. 따라서 MCAP 기록에는 ROS 미들웨어 규약과 관련된 직렬화 표현을 사용하는 ROS 2 메시지 데이터가 포함될 수 있다.

이러한 구분은 기록 파이프라인(recording pipeline)을 최적화할 때 중요하다. 컨테이너 형식을 변경한다고 해서 직렬화된 메시지 데이터를 생성하는 계산 비용이 자동으로 제거되는 것은 아니며, CDR 전송을 최적화한다고 해서 저장소 인덱싱이나 디스크 처리량의 제한이 해결되는 것도 아니다. 완전한 로깅 아키텍처(logging architecture)는 메시지 직렬화, 기록 하위 시스템으로의 전달, 컨테이너 구성, 압축, 파일 쓰기, 인덱싱, 이후 재생 요구사항을 각각 고려해야 한다.

압축(compression)은 기록 데이터에 압축 가능한 정보가 포함되어 있을 경우 MCAP 또는 rosbag 저장 용량을 크게 줄일 수 있지만 처리 자원을 소비한다. 최적의 전략은 시스템이 주로 CPU 용량, 디스크 대역폭, 저장 용량 또는 재생 속도 가운데 어떤 요소의 제약을 받는지에 따라 달라진다. 고부하 AI 추론을 수행하는 로봇은 낮은 기록 CPU 오버헤드를 선호할 수 있고, 장시간 데이터 수집 플랫폼은 저장 용량 절감을 더 중요하게 고려할 수 있다.

청킹(chunking)과 인덱싱(indexing)은 기록된 데이터가 저장되고 이후 접근되는 방식에 영향을 준다. 메시지를 청크(chunk) 단위로 그룹화하면 저장 효율을 높이고 유효한 데이터 블록 단위의 압축을 가능하게 할 수 있으며, 인덱스를 사용하면 도구가 전체 기록을 순차적으로 스캔하지 않고도 원하는 메시지를 찾을 수 있다. 이러한 기능은 동기화된 카메라, LiDAR, 위치 추정, 제어 및 진단 스트림을 포함하는 수시간 규모의 로보틱스 데이터셋에서 특히 중요하다.

타임스탬프 무결성(timestamp integrity)은 기록된 다중 모달 시스템에서 동일하게 중요하다. 센서 스트림을 이후에 정확하게 정렬할 수 없다면 효율적인 직렬화와 저장의 가치도 제한된다. 기록 아키텍처는 영상, 포인트 클라우드, IMU 측정값, 로봇 자세(robot pose), 제어 명령, AI 출력 사이의 상관관계를 유지하는 데 필요한 시간 정보를 보존해야 한다. 이러한 요구사항은 직렬화 및 저장 설계를 동기화(synchronization), 클록 관리(clock management), 다중 센서 데이터 융합(multi-sensor data fusion)과 연결한다.

기록 처리량(recording throughput)은 실제 로봇 운용에서 사용하는 것과 동일한 작업 부하에서 검증해야 한다. 개발 과정에서 카메라 한 대의 데이터를 처리할 수 있는 저장 시스템도 인지 및 제어가 활성화된 상태에서 여러 카메라, 포인트 클라우드, 진단 정보를 동시에 기록하면 실패할 수 있다. CPU 사용률, 쓰기 처리량(write throughput), 큐 증가, 메시지 손실, 파일 크기, 지연시간을 모니터링하면 병목이 직렬화, 압축, 저장 또는 응용 프로그램 처리 중 어디에 존재하는지 판단할 수 있다.

피지컬 AI(Physical AI) 데이터셋 수집에서 MCAP 기반 기록은 온라인 로봇 실행과 오프라인 학습 파이프라인(offline learning pipeline) 사이에 실용적인 경계를 제공할 수 있다. 원시 센서 관측값(raw sensor observation), 로봇 상태, 명령, 인지 출력, 임무 이벤트를 함께 기록한 후 학습에 특화된 형식으로 변환할 수 있다. 명확한 스키마와 타임스탬프를 유지하면 로봇이 무엇을 관측했고 소프트웨어가 무엇을 추론했으며 어떤 행동이 실행되었는지 사이의 추적성(traceability)을 보존할 수 있다.

따라서 확장 가능한 ROS 2 아키텍처는 데이터 경로에 따라 최적화 목표를 구분한다. 온라인 통신(online communication)은 지연시간, 제한된 큐, 복사 감소, 적절한 CDR 처리, 효율적인 미들웨어 전송에 중점을 둔다. 기록은 지속적인 처리량, 저장소 구성, 인덱싱, 압축, 타임스탬프 무결성, 신뢰성 있는 재생에 중점을 둔다. 일부 최적화 기법은 두 경로 모두에 도움이 되지만 각 경로는 자체적인 운용 요구사항을 기준으로 측정하고 검증해야 한다.

직렬화 최적화(serialization optimization)는 궁극적으로 모든 개별 변환을 단순히 더 빠르게 만드는 것이 아니라 불필요한 변환 자체를 줄이는 것을 목표로 해야 한다. 잘 설계된 시스템은 중복 메시지 필드를 최소화하고 반복적인 메모리 할당과 복사를 피하며, 검증된 경우 프로세스 내부 통신과 대여 메시지 기능을 활용하고, 고대역폭 데이터를 이를 소비하는 구성요소 가까이에 유지한다. 기록 작업에서는 효율적인 MCAP 구성이 이러한 기법을 보완하여 대규모 동기화 ROS 2 데이터셋을 위한 구조화된 저장 방식을 제공한다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 CDR과 MCAP은 정보 수명주기(information lifecycle)의 상호 보완적인 위치를 차지한다. CDR은 ROS 2 미들웨어 통신에서 상호운용 가능한 형식화 데이터 표현을 지원하고, MCAP은 재생과 분석을 위해 타임스탬프 기반 다중 모달 스트림을 구조적으로 보존한다. 직렬화, 복사, 전송, 기록, 압축, 인덱싱이 각각 어디에서 발생하는지 이해하면 실시간 센서 획득에서 장기적인 로봇 데이터 활용까지 전체 경로를 최적화할 수 있다.

##  

## 04.07 ROS2 Bridge: ros1.bridge / External System [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 bridge architecture provides an interoperability layer between ROS 2 and software environments that do not communicate through the same middleware model. A bridge receives information from one communication domain, translates its interface and transport representation, and republishes or forwards it into another domain. This mechanism is particularly important during migration from ROS 1, integration of legacy robot software, and connection to external industrial or cloud systems.

The \`ros1_bridge\` package is the best-known bridge mechanism for connecting ROS 1 and ROS 2 systems. ROS 1 primarily relies on its own communication infrastructure and message transport conventions, whereas ROS 2 is built around an abstraction layer that commonly uses DDS middleware. The bridge allows nodes from both generations to exchange compatible information without requiring the complete ROS 1 software stack to be rewritten immediately.

Conceptually, the bridge operates as an adapter between two communication graphs. On the ROS 1 side, it participates in ROS 1 topic, service, and message mechanisms. On the ROS 2 side, it creates corresponding ROS 2 communication entities. Incoming data is interpreted according to the source interface, converted into the compatible destination representation, and then published or returned through the communication mechanism expected by the receiving environment.

Message compatibility is therefore central to bridge operation. A bridge cannot safely translate arbitrary data merely because two topics have similar names. It must understand how fields in the ROS 1 message correspond to fields in the ROS 2 message. Standard interfaces with known mappings are relatively straightforward, while custom messages require compatible definitions and appropriate build-time support so that conversion rules can be generated or made available.

The dynamic bridge can create communication bridges according to interfaces discovered at runtime when the required message mappings are available. This is useful in development and migration environments because developers do not have to manually instantiate every topic connection. As ROS 1 and ROS 2 endpoints appear, the bridge can establish the necessary forwarding paths according to the supported interfaces and discovered communication graph.

A static bridge represents a more explicitly configured approach in which required bridge connections are determined in advance. This can reduce unnecessary runtime flexibility and make deployment behavior easier to reason about in controlled production systems. The choice between dynamic and static bridging should reflect the deployment environment, interface stability, debugging requirements, resource constraints, and the degree of runtime discovery required by the robot system.

Topic bridging is particularly common because sensors and robot state information are frequently exchanged during migration. A legacy ROS 1 camera driver, LiDAR driver, or localization component may continue publishing data while newly developed perception or planning components operate in ROS 2. The bridge converts compatible messages between the two environments, allowing modernization to proceed subsystem by subsystem rather than requiring a single disruptive platform replacement.

Service bridging extends interoperability to request--response interactions. A ROS 2 component may need to invoke functionality that still resides in a ROS 1 subsystem, or a legacy ROS 1 application may need access to a capability implemented in ROS 2. The bridge translates compatible service requests and responses across the middleware boundary, although developers must still consider timeout behavior, availability, failure propagation, and semantic differences between the connected systems.

Bridging introduces computational and communication overhead because messages may require conversion, serialization, deserialization, memory allocation, and copying. The impact is often modest for small state messages but can become significant for images, point clouds, maps, and other high-bandwidth data. A migration architecture should therefore avoid routing every data stream through a bridge without evaluating message size, publication frequency, CPU utilization, memory bandwidth, and end-to-end latency.

QoS creates another important boundary because ROS 1 and ROS 2 do not expose identical communication semantics. ROS 2 provides explicit DDS-oriented policies for reliability, durability, history, and other delivery behavior, whereas ROS 1 communication follows different mechanisms. A bridge must map communication behavior as appropriately as possible, but developers should not assume that every ROS 2 QoS property has an exact ROS 1 equivalent across the boundary.

For this reason, bridge validation should examine operational semantics rather than only confirming that messages appear on both sides. Engineers should verify message frequency, ordering, latency, loss behavior, startup behavior, late-joining nodes, and failure recovery under realistic network conditions. A bridge that works during a simple command-line test may behave differently when multiple sensors, high data rates, and distributed computers operate simultaneously.

The \`ros1_bridge\` is most valuable as a migration mechanism rather than as a permanent excuse to preserve unnecessary architectural complexity. During a staged ROS 1-to-ROS 2 transition, legacy drivers or mature algorithms can remain operational while surrounding software moves to ROS 2. Over time, components can be replaced or ported until the bridge carries fewer interfaces and can eventually be removed if the complete platform becomes ROS 2 native.

Bridging is not limited conceptually to ROS 1. Modern robots frequently communicate with external systems that use REST APIs, WebSocket, MQTT, gRPC, industrial protocols, proprietary TCP/UDP interfaces, or fleet-management standards. In these cases, a gateway node or adapter performs a role similar to a bridge: it converts ROS 2 topics, services, actions, or internal state into the data model and transport expected by the external environment.

An external-system bridge should separate transport conversion from robot-domain semantics whenever practical. A ROS 2 node may receive internal localization, mission, battery, or diagnostic information and translate it into an external API representation. In the opposite direction, external commands can be validated and converted into ROS 2 interfaces. This separation prevents cloud, enterprise, or vendor-specific protocols from becoming deeply embedded inside navigation, perception, or control nodes.

Fleet management is a representative example. Individual robots may use ROS 2 internally while a fleet manager communicates through a different protocol or standardized interface. A gateway can translate mission requests into robot-level commands and convert robot state into fleet-level status information. The bridge becomes an architectural boundary between high-frequency robot-local communication and lower-frequency coordination information exchanged across the facility or enterprise network.

Industrial integration often requires similar adapters. Programmable logic controllers, manufacturing execution systems, warehouse management systems, inspection equipment, and proprietary machines may not participate directly in the ROS 2 graph. A bridge or gateway can expose selected robot capabilities through protocols appropriate to those systems while keeping ROS 2 as the internal robotics middleware. This approach reduces coupling between robot software and external automation infrastructure.

Data-model translation is usually more difficult than transport translation. An external system may represent a robot mission, pose, status, timestamp, coordinate frame, or error code differently from ROS 2. Simply copying fields is therefore insufficient. A robust bridge defines explicit semantic mappings, unit conversions, coordinate conventions, identifier rules, timestamps, validation requirements, and error behavior so that information retains the same operational meaning across the boundary.

Command bridges require particularly careful validation because external messages may ultimately affect physical motion. Incoming commands should be checked for validity, authorization, current robot state, supported operating mode, and appropriate limits before being forwarded to execution components. Communication interoperability should not bypass the safety state machine, mission arbitration, motion constraints, or hardware protection mechanisms already established inside the robot architecture.

Failure isolation is another important responsibility. If an external cloud service, ROS 1 process, network connection, or enterprise application becomes unavailable, the bridge should prevent that failure from propagating uncontrollably into core robot functions. Robot-local navigation, perception, and safety functions should remain operational when their architecture permits. Reconnection, timeout, buffering, retry, and degraded-mode policies should be explicitly defined according to the importance of each interface.

Security boundaries become increasingly important when a bridge connects the ROS 2 domain to external networks. The gateway may become an entry point between trusted robot-local communication and less trusted enterprise, wireless, or cloud infrastructure. Authentication, authorization, encryption, network segmentation, input validation, logging, and rate limiting should therefore be considered part of bridge architecture rather than optional features added after integration.

Observability is essential because bridge failures can otherwise appear as failures of the systems on either side. Useful diagnostics include connection state, mapped interfaces, message rates, conversion failures, queue utilization, dropped messages, latency, reconnect attempts, and protocol errors. Logging these metrics allows engineers to determine whether a problem originates in ROS 1, ROS 2, the bridge itself, the external protocol, or the underlying network.

A scalable bridge should expose only the information that actually needs to cross a system boundary. Raw camera and LiDAR streams should not automatically be forwarded to fleet or cloud infrastructure merely because they exist in the ROS 2 graph. Local processing can transform high-bandwidth sensor data into compact objects, events, health information, or mission state before bridging. This reduces bandwidth, conversion overhead, security exposure, and external-system dependencies.

For autonomous robots and Physical AI systems, bridging is ultimately an architectural technique for controlled interoperability. \`ros1_bridge\` supports staged coexistence between ROS 1 and ROS 2, while external gateways connect ROS 2 to fleet, cloud, industrial, and enterprise ecosystems. Well-designed bridges preserve interface semantics, isolate failures, control data flow, validate commands, and make heterogeneous systems cooperate without allowing transport differences to contaminate the robot\'s core software architecture.

ROS 2 브리지 아키텍처(bridge architecture)는 ROS 2와 동일한 미들웨어 모델(middleware model)을 사용하지 않는 소프트웨어 환경 사이에 상호운용성 계층(interoperability layer)을 제공한다. 브리지(bridge)는 한 통신 도메인(communication domain)에서 정보를 수신하고 인터페이스(interface)와 전송 표현(transport representation)을 변환한 뒤 다른 도메인으로 다시 발행하거나 전달한다. 이러한 메커니즘은 ROS 1에서의 마이그레이션(migration), 레거시 로봇 소프트웨어(legacy robot software)의 통합, 외부 산업 또는 클라우드 시스템과의 연결에서 특히 중요하다.

\`ros1_bridge\` 패키지는 ROS 1과 ROS 2 시스템을 연결하기 위한 가장 대표적인 브리지 메커니즘(bridge mechanism)이다. ROS 1은 주로 자체 통신 인프라와 메시지 전송 규약(message transport convention)에 의존하는 반면, ROS 2는 일반적으로 DDS 미들웨어를 사용하는 추상화 계층(abstraction layer)을 기반으로 구축된다. 브리지를 사용하면 전체 ROS 1 소프트웨어 스택을 즉시 다시 작성하지 않고도 두 세대의 노드가 호환 가능한 정보를 교환할 수 있다.

개념적으로 브리지는 두 통신 그래프(communication graph) 사이의 어댑터(adapter)로 동작한다. ROS 1 측에서는 ROS 1 토픽(topic), 서비스(service), 메시지 메커니즘에 참여하고, ROS 2 측에서는 이에 대응하는 ROS 2 통신 엔티티(communication entity)를 생성한다. 입력 데이터는 소스 인터페이스(source interface)에 따라 해석되고 호환 가능한 목적지 표현(destination representation)으로 변환된 후 수신 환경이 요구하는 통신 메커니즘을 통해 발행되거나 반환된다.

따라서 메시지 호환성(message compatibility)은 브리지 동작의 핵심이다. 두 토픽의 이름이 유사하다는 이유만으로 브리지가 임의의 데이터를 안전하게 변환할 수 있는 것은 아니다. ROS 1 메시지의 필드가 ROS 2 메시지의 필드와 어떻게 대응하는지 이해해야 한다. 알려진 매핑(mapping)을 가진 표준 인터페이스는 비교적 간단하지만 사용자 정의 메시지(custom message)는 변환 규칙을 생성하거나 사용할 수 있도록 호환 가능한 정의와 적절한 빌드 시점 지원(build-time support)이 필요하다.

동적 브리지(dynamic bridge)는 필요한 메시지 매핑이 제공되는 경우 런타임(runtime)에 발견된 인터페이스에 따라 통신 브리지를 생성할 수 있다. 개발 및 마이그레이션 환경에서는 개발자가 모든 토픽 연결을 수동으로 생성하지 않아도 되므로 유용하다. ROS 1과 ROS 2 엔드포인트(endpoint)가 나타나면 브리지는 지원되는 인터페이스와 검색된 통신 그래프에 따라 필요한 전달 경로(forwarding path)를 구성할 수 있다.

정적 브리지(static bridge)는 필요한 브리지 연결을 사전에 결정하는 보다 명시적으로 구성된 접근 방식이다. 이를 통해 불필요한 런타임 유연성을 줄이고 통제된 운영 시스템에서 배포 동작(deployment behavior)을 보다 쉽게 예측할 수 있다. 동적 브리지와 정적 브리지의 선택은 배포 환경, 인터페이스 안정성, 디버깅 요구사항, 자원 제약, 로봇 시스템이 요구하는 런타임 검색(runtime discovery)의 수준을 고려하여 결정해야 한다.

토픽 브리징(topic bridging)은 마이그레이션 과정에서 센서 및 로봇 상태 정보가 빈번하게 교환되기 때문에 특히 일반적으로 사용된다. 기존 ROS 1 카메라 드라이버, LiDAR 드라이버 또는 위치 추정(localization) 컴포넌트가 계속 데이터를 발행하는 동안 새롭게 개발된 인지(perception) 또는 계획(planning) 컴포넌트는 ROS 2에서 동작할 수 있다. 브리지는 두 환경 사이에서 호환 가능한 메시지를 변환함으로써 한 번에 전체 플랫폼을 교체하지 않고 서브시스템 단위로 현대화를 진행할 수 있게 한다.

서비스 브리징(service bridging)은 요청-응답 상호작용(request-response interaction)까지 상호운용성을 확장한다. ROS 2 컴포넌트가 아직 ROS 1 서브시스템에 존재하는 기능을 호출해야 하거나 기존 ROS 1 응용 프로그램이 ROS 2에서 구현된 기능에 접근해야 할 수 있다. 브리지는 호환 가능한 서비스 요청과 응답을 미들웨어 경계(middleware boundary)를 넘어 변환하지만, 개발자는 연결된 시스템 사이의 타임아웃(timeout), 가용성(availability), 장애 전파(failure propagation), 의미적 차이도 함께 고려해야 한다.

브리징(bridging)은 메시지 변환, 직렬화(serialization), 역직렬화(deserialization), 메모리 할당(memory allocation), 복사(copying)가 필요할 수 있기 때문에 계산 및 통신 오버헤드를 발생시킨다. 작은 상태 메시지에서는 영향이 크지 않을 수 있지만 영상, 포인트 클라우드(point cloud), 맵(map) 및 기타 고대역폭 데이터에서는 상당한 영향을 줄 수 있다. 따라서 마이그레이션 아키텍처는 메시지 크기, 발행 주기, CPU 사용률, 메모리 대역폭, 종단 간 지연시간(end-to-end latency)을 평가하지 않은 상태에서 모든 데이터 스트림을 브리지로 전달해서는 안 된다.

QoS는 ROS 1과 ROS 2가 동일한 통신 의미론(communication semantics)을 제공하지 않기 때문에 또 다른 중요한 경계를 형성한다. ROS 2는 신뢰성(reliability), 내구성(durability), 이력(history) 및 기타 전달 동작에 대한 명시적인 DDS 지향 정책을 제공하지만 ROS 1 통신은 다른 메커니즘을 따른다. 브리지는 가능한 범위에서 통신 동작을 적절하게 매핑해야 하지만 모든 ROS 2 QoS 속성이 ROS 1 측에 정확하게 대응된다고 가정해서는 안 된다.

따라서 브리지 검증(bridge validation)은 메시지가 양쪽에 나타나는지만 확인하는 것이 아니라 실제 운용 의미론(operational semantics)을 검증해야 한다. 엔지니어는 현실적인 네트워크 조건에서 메시지 주기, 순서, 지연시간, 손실 동작, 시작 동작, 지연 참여 노드(late-joining node), 장애 복구를 확인해야 한다. 단순한 명령줄 시험에서 정상적으로 동작하는 브리지도 여러 센서, 높은 데이터 전송률, 분산 컴퓨터가 동시에 동작하는 환경에서는 다른 특성을 나타낼 수 있다.

\`ros1_bridge\`는 불필요한 아키텍처 복잡성을 영구적으로 유지하기 위한 수단보다는 마이그레이션 메커니즘(migration mechanism)으로 사용할 때 가장 큰 가치가 있다. 단계적인 ROS 1에서 ROS 2로의 전환 과정에서 기존 드라이버나 검증된 알고리즘을 계속 운용하면서 주변 소프트웨어를 ROS 2로 이전할 수 있다. 시간이 지나면서 컴포넌트를 교체하거나 포팅(porting)하여 브리지가 전달하는 인터페이스 수를 점차 줄이고, 전체 플랫폼이 ROS 2 네이티브(ROS 2 native) 환경으로 전환되면 최종적으로 브리지를 제거할 수 있다.

브리징의 개념은 ROS 1에만 제한되지 않는다. 현대 로봇은 REST API, 웹소켓(WebSocket), MQTT, gRPC, 산업용 프로토콜(industrial protocol), 독점 TCP/UDP 인터페이스(proprietary TCP/UDP interface), 플릿 관리 표준(fleet-management standard)을 사용하는 외부 시스템과 빈번하게 통신한다. 이러한 경우 게이트웨이 노드(gateway node) 또는 어댑터가 브리지와 유사한 역할을 수행하여 ROS 2 토픽, 서비스, 액션(action) 또는 내부 상태를 외부 환경에서 요구하는 데이터 모델과 전송 방식으로 변환한다.

외부 시스템 브리지(external-system bridge)는 가능한 경우 전송 변환(transport conversion)과 로봇 도메인 의미론(robot-domain semantics)을 분리해야 한다. ROS 2 노드는 내부 위치 추정, 임무(mission), 배터리 또는 진단 정보를 수신하여 외부 API 표현으로 변환할 수 있다. 반대 방향에서는 외부 명령을 검증한 후 ROS 2 인터페이스로 변환할 수 있다. 이러한 분리는 클라우드, 기업 시스템 또는 공급업체별 프로토콜이 내비게이션, 인지 또는 제어 노드 내부에 깊이 결합되는 것을 방지한다.

플릿 관리(fleet management)는 대표적인 사례이다. 개별 로봇은 내부적으로 ROS 2를 사용하는 반면 플릿 관리자(fleet manager)는 다른 프로토콜이나 표준화된 인터페이스를 통해 통신할 수 있다. 게이트웨이는 임무 요청을 로봇 수준의 명령으로 변환하고 로봇 상태를 플릿 수준의 상태 정보로 변환할 수 있다. 이때 브리지는 고주기 로봇 로컬 통신과 시설 또는 기업 네트워크 전체에서 교환되는 저주기 협업 정보 사이의 아키텍처 경계(architectural boundary)가 된다.

산업 시스템 통합(industrial integration)에서도 유사한 어댑터가 필요한 경우가 많다. 프로그래머블 로직 컨트롤러(Programmable Logic Controller, PLC), 제조 실행 시스템(Manufacturing Execution System, MES), 창고 관리 시스템(Warehouse Management System, WMS), 검사 장비 및 독점 기계는 ROS 2 그래프에 직접 참여하지 않을 수 있다. 브리지 또는 게이트웨이는 이러한 시스템에 적합한 프로토콜을 통해 선택된 로봇 기능을 제공하면서 ROS 2를 내부 로보틱스 미들웨어로 유지할 수 있다. 이러한 방식은 로봇 소프트웨어와 외부 자동화 인프라 사이의 결합도(coupling)를 낮춘다.

데이터 모델 변환(data-model translation)은 일반적으로 전송 변환보다 더 어렵다. 외부 시스템은 로봇 임무, 자세(pose), 상태, 타임스탬프(timestamp), 좌표 프레임(coordinate frame), 오류 코드(error code)를 ROS 2와 다르게 표현할 수 있다. 따라서 단순히 필드를 복사하는 것만으로는 충분하지 않다. 견고한 브리지는 명시적인 의미적 매핑(semantic mapping), 단위 변환(unit conversion), 좌표 규약, 식별자 규칙(identifier rule), 타임스탬프, 검증 요구사항, 오류 동작을 정의하여 시스템 경계를 넘어 전달된 정보가 동일한 운용 의미를 유지하도록 해야 한다.

명령 브리지(command bridge)는 외부 메시지가 최종적으로 물리적 움직임에 영향을 줄 수 있으므로 특히 신중한 검증이 필요하다. 입력 명령은 실행 컴포넌트로 전달되기 전에 유효성, 권한(authorization), 현재 로봇 상태, 지원되는 운용 모드, 적절한 제한값을 검사해야 한다. 통신 상호운용성이 로봇 아키텍처 내부에 이미 구축된 안전 상태 머신(safety state machine), 임무 중재(mission arbitration), 운동 제한(motion constraint), 하드웨어 보호 메커니즘을 우회해서는 안 된다.

장애 격리(failure isolation) 역시 중요한 책임이다. 외부 클라우드 서비스, ROS 1 프로세스, 네트워크 연결 또는 기업 응용 프로그램을 사용할 수 없게 되더라도 브리지는 해당 장애가 핵심 로봇 기능으로 통제되지 않은 상태에서 전파되는 것을 방지해야 한다. 아키텍처가 허용하는 경우 로봇 로컬 내비게이션, 인지 및 안전 기능은 계속 동작해야 한다. 재연결(reconnection), 타임아웃, 버퍼링(buffering), 재시도(retry), 성능 저하 모드(degraded mode) 정책은 각 인터페이스의 중요도에 따라 명시적으로 정의해야 한다.

브리지가 ROS 2 도메인을 외부 네트워크와 연결할수록 보안 경계(security boundary)의 중요성이 증가한다. 게이트웨이는 신뢰할 수 있는 로봇 로컬 통신과 상대적으로 신뢰도가 낮은 기업, 무선 또는 클라우드 인프라 사이의 진입점(entry point)이 될 수 있다. 따라서 인증(authentication), 권한 부여(authorization), 암호화(encryption), 네트워크 분할(network segmentation), 입력 검증(input validation), 로깅(logging), 전송률 제한(rate limiting)을 통합 이후 추가되는 선택 기능이 아니라 브리지 아키텍처의 일부로 고려해야 한다.

관측 가능성(observability)은 브리지 장애가 그렇지 않을 경우 양쪽 시스템의 장애처럼 보일 수 있기 때문에 필수적이다. 유용한 진단 정보에는 연결 상태, 매핑된 인터페이스, 메시지 전송률, 변환 실패, 큐 사용량, 손실된 메시지, 지연시간, 재연결 시도, 프로토콜 오류가 포함된다. 이러한 지표를 기록하면 엔지니어는 문제가 ROS 1, ROS 2, 브리지 자체, 외부 프로토콜 또는 기반 네트워크 중 어디에서 발생했는지 판단할 수 있다.

확장 가능한 브리지(scalable bridge)는 시스템 경계를 실제로 넘어가야 하는 정보만 노출해야 한다. 원시 카메라 및 LiDAR 스트림이 ROS 2 그래프에 존재한다는 이유만으로 이를 플릿 또는 클라우드 인프라에 자동 전달해서는 안 된다. 로컬 처리를 통해 고대역폭 센서 데이터를 압축된 객체, 이벤트(event), 상태 정보(health information), 임무 상태로 변환한 후 브리징할 수 있다. 이를 통해 대역폭, 변환 오버헤드, 보안 노출(security exposure), 외부 시스템 종속성을 줄일 수 있다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 브리징은 궁극적으로 통제된 상호운용성(controlled interoperability)을 구현하기 위한 아키텍처 기법이다. \`ros1_bridge\`는 ROS 1과 ROS 2의 단계적 공존을 지원하며, 외부 게이트웨이(external gateway)는 ROS 2를 플릿, 클라우드, 산업 및 기업 생태계와 연결한다. 잘 설계된 브리지는 인터페이스 의미론을 보존하고, 장애를 격리하며, 데이터 흐름을 제어하고, 명령을 검증함으로써 전송 방식의 차이가 로봇의 핵심 소프트웨어 아키텍처를 오염시키지 않으면서 이기종 시스템(heterogeneous system)이 협력할 수 있도록 한다.

##  

## 04.08 ROS2 Multi-Robot Namespace Design [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

Multi-robot ROS 2 systems require a naming architecture that allows many robots to execute similar software without allowing their communication interfaces to collide. A single robot may contain hundreds of topics, services, actions, parameters, and transform frames. When identical software stacks are replicated across a fleet, these names must remain predictable and isolated. ROS 2 namespaces provide the primary mechanism for creating this separation while preserving reusable node and interface definitions.

A namespace is a hierarchical prefix applied to ROS 2 names. Instead of every robot publishing a generic \`/scan\`, \`/camera/image\`, or \`/cmd_vel\` interface at the global level, each robot can operate beneath a unique namespace such as \`/robot_01\`, \`/robot_02\`, or \`/robot_03\`. The resulting interfaces become \`/robot_01/scan\` and \`/robot_02/scan\`, allowing identical sensor and navigation software to operate simultaneously without confusing one robot\'s data with another\'s.

This design is particularly effective when the robot software stack is treated as a reusable template. Nodes, launch files, parameters, and communication interfaces can be defined using relative names rather than embedding a specific robot identifier in source code. At deployment time, the complete stack is placed within the selected namespace. The same executable and configuration structure can then be instantiated repeatedly for different robots with minimal modification.

Relative and absolute ROS 2 names must therefore be used deliberately. An absolute name beginning with \`/\` is resolved from the root namespace and can unintentionally escape a robot-specific namespace. A relative name is resolved within the namespace assigned to the node. Multi-robot software generally benefits from relative naming for robot-local interfaces, while absolute names should be reserved for intentionally shared system-level resources whose scope is clearly defined.

Node names also require consistency. Multiple robots may each execute nodes called \`camera_driver\`, \`localization\`, \`planner\`, and \`controller\` because their namespaces make the fully qualified node names unique. For example, \`/robot_01/localization\` and \`/robot_02/localization\` represent separate entities even though the local node name is identical. This preserves common software configurations while allowing monitoring tools to identify the owning robot from the hierarchical name.

A useful namespace hierarchy reflects operational ownership rather than implementation accidents. The first level can identify the robot, while lower levels can identify subsystems such as sensors, perception, navigation, manipulation, or diagnostics. Interfaces such as \`/robot_01/sensors/lidar/scan\` or \`/robot_01/navigation/status\` communicate both ownership and function. Excessively deep hierarchies should still be avoided because long names increase configuration complexity without necessarily improving semantic clarity.

Launch files are an important mechanism for applying namespaces consistently. A robot launch description can group related nodes under a namespace and pass the robot identifier as a launch argument. This prevents developers from manually editing every topic or node name when another robot is added. A fleet deployment can instantiate the same launch structure several times, each with a different namespace and parameter set corresponding to a particular physical robot.

Remapping complements namespaces when a component\'s generic interface must connect to a deployment-specific name. A reusable controller might subscribe to \`cmd_vel\`, while the launch configuration places the node under \`/robot_03\` or remaps the interface to another local command path. Remapping should be used to adapt reusable components at integration boundaries rather than compensating for an inconsistent naming architecture throughout the entire software system.

Parameters should follow the same ownership principle. Two robots running identical nodes may require different sensor calibrations, controller gains, network addresses, or mechanical dimensions. Namespaced nodes naturally provide separate parameter contexts, allowing common parameter files to be combined with robot-specific overrides. Configuration management should make it clear which values belong to the software type and which values describe an individual physical robot.

Topics within a robot namespace usually represent robot-local data. Camera images, LiDAR scans, odometry, joint states, local maps, velocity commands, battery state, and diagnostic information normally belong to one robot. Keeping these interfaces namespaced reduces accidental cross-connections and makes tools such as topic inspection, visualization, logging, and debugging easier to interpret when several robots are active on the same ROS 2 network.

Some information, however, genuinely belongs to the fleet rather than to an individual robot. Mission assignments, shared traffic information, fleet status, infrastructure state, or coordination events may use explicitly designed fleet-level interfaces. These should not emerge accidentally because a node escaped its namespace. A clear architecture distinguishes robot-local communication from shared coordination communication and defines the data contract at the boundary between the two scopes.

Namespaces alone do not provide network isolation. ROS 2 discovery may still allow participants within the same DDS communication environment to discover endpoints belonging to other robots. The namespace prevents name collisions and organizes the graph, but it does not automatically prevent traffic, discovery load, or unauthorized access. Large fleets may therefore require additional DDS configuration, domain separation, discovery control, network segmentation, or security mechanisms.

ROS domain identifiers can provide a broader communication boundary when groups of robots should not participate in the same DDS domain. Robots in separate test areas, laboratories, customers, or operational zones can be assigned different domain IDs when complete discovery separation is appropriate. However, placing every robot in a different domain can complicate fleet-level communication, so domain design and namespace design should be treated as complementary architectural mechanisms rather than interchangeable solutions.

Transform data requires special attention because TF frame identifiers are not automatically solved merely by placing ROS nodes in namespaces. If several robots publish frames named \`base_link\`, \`odom\`, \`map\`, or \`laser\`, a shared transform environment can contain ambiguous frame identities. Multi-robot architectures therefore need a deliberate TF naming strategy, such as robot-specific frame prefixes or separated transform scopes, consistent with localization, visualization, mapping, and planning requirements.

The global \`map\` concept also requires an explicit design decision. Each robot may maintain its own local map and localization frame, or several robots may operate within a common facility coordinate system. In a shared-map architecture, robot poses must be distinguishable while remaining expressible in the common reference frame. Namespace and TF conventions should therefore be designed together rather than independently after navigation software has already been integrated.

Services and actions need the same namespace discipline as topics. A command such as navigation to a pose, docking, calibration, reset, or inspection must clearly target the intended robot. Names such as \`/robot_02/navigate_to_pose\` or \`/robot_04/dock\` make ownership explicit. This becomes especially important for actions because long-running goals, feedback, cancellation, and results must remain associated with the correct robot throughout their lifecycle.

Multi-robot monitoring becomes much easier when naming conventions are deterministic. Operators and software tools can filter interfaces by robot namespace, aggregate the same diagnostic signal across the fleet, or compare equivalent nodes between robots. A predictable pattern also supports automated logging and observability because dashboards can derive robot identity and subsystem ownership directly from names instead of relying on manually maintained mappings.

Logging architecture should preserve namespace information because identical topic names from different robots otherwise become difficult to distinguish during offline analysis. Recorded datasets should retain robot identity together with timestamps and message schemas. This is particularly important when data from several robots is combined for fleet analytics, failure investigation, mapping, or Physical AI training, where the origin of each observation and action must remain traceable.

Scalability requires controlling what is shared across the fleet. A namespace makes \`/robot_01/camera/image\` different from \`/robot_02/camera/image\`, but publishing both high-bandwidth streams across the same wireless infrastructure may still overload the network. Robot-local perception should therefore process raw data locally where practical, while fleet interfaces distribute compact poses, objects, mission states, health information, and coordination events.

A multi-robot system should also avoid hard-coded assumptions about fleet size. Software that explicitly expects \`robot_01\`, \`robot_02\`, and \`robot_03\` becomes difficult to extend when robots are added, removed, or replaced. Robot identity should be supplied through configuration, launch arguments, discovery, or fleet-management information. The architecture can then instantiate the same software pattern for a changing population without modifying application source code.

Naming conventions should be documented as part of the ROS 2 interface contract. The document should define robot identifiers, namespace hierarchy, node naming, topic and service conventions, action ownership, TF frame strategy, shared fleet interfaces, and rules for absolute names. Consistent rules prevent different teams from independently creating incompatible naming patterns that become difficult to correct after the system has grown.

Testing should include several robot instances rather than validating each stack only in isolation. Engineers should verify that duplicate software instances launch successfully, interfaces remain separated, commands reach only their intended robot, TF trees remain unambiguous, parameter configurations do not leak between instances, and shared fleet interfaces behave correctly. Tests should also evaluate discovery behavior and network load as the number of active robots increases.

Simulation provides an effective environment for validating namespace design before deploying a physical fleet. Multiple simulated robots can run identical ROS 2 stacks with different namespaces, poses, sensor configurations, and missions. Problems involving remapping, TF frames, duplicated names, shared topics, or launch configuration can then be discovered early. The same namespace strategy should transfer to physical deployment so that simulation and real robots preserve equivalent interface structures.

For autonomous fleets and Physical AI systems, namespace design is ultimately a mechanism for expressing ownership, scope, and repeatability within the ROS 2 communication graph. A strong architecture combines robot-specific namespaces, relative interfaces, consistent launch configuration, deliberate TF naming, clearly separated fleet-level communication, and appropriate DDS network boundaries. This allows one reusable robot software stack to scale into many cooperating robots without sacrificing traceability, isolation, or maintainability.

다중 로봇 ROS 2 시스템(multi-robot ROS 2 system)에서는 여러 로봇이 유사한 소프트웨어를 실행하면서도 각 통신 인터페이스가 서로 충돌하지 않도록 하는 명명 아키텍처(naming architecture)가 필요하다. 하나의 로봇에는 수백 개의 토픽(topic), 서비스(service), 액션(action), 매개변수(parameter), 변환 프레임(transform frame)이 존재할 수 있다. 동일한 소프트웨어 스택(software stack)이 플릿(fleet) 전체에 복제될 때 이러한 이름은 예측 가능하면서 서로 격리되어야 한다. ROS 2 네임스페이스(namespace)는 재사용 가능한 노드 및 인터페이스 정의를 유지하면서 이러한 분리를 구현하는 핵심 메커니즘이다.

네임스페이스(namespace)는 ROS 2 이름에 적용되는 계층적 접두사(hierarchical prefix)이다. 모든 로봇이 일반적인 \`/scan\`, \`/camera/image\`, \`/cmd_vel\` 인터페이스를 전역 수준(global level)에 발행하는 대신 각 로봇을 \`/robot_01\`, \`/robot_02\`, \`/robot_03\`과 같은 고유 네임스페이스 아래에서 동작하도록 구성할 수 있다. 그 결과 인터페이스는 \`/robot_01/scan\`, \`/robot_02/scan\`과 같이 구성되며, 동일한 센서 및 내비게이션 소프트웨어가 한 로봇의 데이터를 다른 로봇의 데이터와 혼동하지 않고 동시에 동작할 수 있다.

이러한 설계는 로봇 소프트웨어 스택을 재사용 가능한 템플릿(reusable template)으로 취급할 때 특히 효과적이다. 노드, 실행 파일(launch file), 매개변수, 통신 인터페이스를 소스 코드에 특정 로봇 식별자를 직접 삽입하는 대신 상대 이름(relative name)을 사용하여 정의할 수 있다. 배포 시점에 전체 스택을 선택된 네임스페이스 아래에 배치한다. 그러면 동일한 실행 파일과 설정 구조를 최소한의 수정만으로 서로 다른 로봇에 반복적으로 인스턴스화(instantiate)할 수 있다.

따라서 ROS 2의 상대 이름(relative name)과 절대 이름(absolute name)은 의도적으로 사용해야 한다. \`/\`로 시작하는 절대 이름은 루트 네임스페이스(root namespace)를 기준으로 해석되므로 의도하지 않게 로봇별 네임스페이스를 벗어날 수 있다. 상대 이름은 노드에 할당된 네임스페이스 내부에서 해석된다. 다중 로봇 소프트웨어에서는 일반적으로 로봇 로컬 인터페이스(robot-local interface)에 상대 이름을 사용하는 것이 유리하며, 절대 이름은 범위가 명확하게 정의된 의도적인 시스템 수준 공유 자원에만 사용하는 것이 바람직하다.

노드 이름(node name)에도 일관성이 필요하다. 여러 로봇은 각각 \`camera_driver\`, \`localization\`, \`planner\`, \`controller\`라는 동일한 이름의 노드를 실행할 수 있으며, 네임스페이스를 사용하면 완전한 노드 이름(fully qualified node name)이 서로 고유하게 유지된다. 예를 들어 \`/robot_01/localization\`과 \`/robot_02/localization\`은 로컬 노드 이름이 동일하더라도 서로 다른 엔티티(entity)를 나타낸다. 이를 통해 공통 소프트웨어 설정을 유지하면서 모니터링 도구가 계층적 이름으로 해당 노드가 어느 로봇에 속하는지 식별할 수 있다.

유용한 네임스페이스 계층(namespace hierarchy)은 구현상의 우연한 구조가 아니라 운용 소유권(operational ownership)을 반영해야 한다. 첫 번째 수준에서는 로봇을 식별하고 하위 수준에서는 센서, 인지(perception), 내비게이션(navigation), 조작(manipulation), 진단(diagnostics)과 같은 서브시스템을 구분할 수 있다. \`/robot_01/sensors/lidar/scan\` 또는 \`/robot_01/navigation/status\`와 같은 인터페이스는 소유권과 기능을 동시에 표현한다. 그러나 지나치게 깊은 계층은 의미적 명확성을 높이지 않으면서 설정 복잡성만 증가시킬 수 있으므로 피해야 한다.

실행 파일(launch file)은 네임스페이스를 일관되게 적용하기 위한 중요한 메커니즘이다. 로봇 실행 설명(robot launch description)은 관련 노드를 하나의 네임스페이스 아래에 그룹화하고 로봇 식별자를 실행 인수(launch argument)로 전달할 수 있다. 이를 통해 새로운 로봇이 추가될 때 개발자가 모든 토픽이나 노드 이름을 수동으로 수정할 필요가 없다. 플릿 배포(fleet deployment)에서는 동일한 실행 구조를 여러 번 인스턴스화하면서 각 물리적 로봇에 대응하는 서로 다른 네임스페이스와 매개변수 집합을 적용할 수 있다.

재매핑(remapping)은 컴포넌트의 일반 인터페이스를 배포 환경에 특화된 이름에 연결해야 할 때 네임스페이스를 보완한다. 재사용 가능한 컨트롤러(controller)가 \`cmd_vel\`을 구독하는 경우 실행 설정을 통해 해당 노드를 \`/robot_03\` 아래에 배치하거나 인터페이스를 다른 로컬 명령 경로로 재매핑할 수 있다. 재매핑은 전체 소프트웨어 시스템의 일관성 없는 명명 구조를 보완하기 위한 수단보다는 재사용 가능한 컴포넌트를 통합 경계(integration boundary)에 맞게 조정하기 위한 방법으로 사용하는 것이 바람직하다.

매개변수(parameter)도 동일한 소유권 원칙을 따라야 한다. 동일한 노드를 실행하는 두 로봇이라도 서로 다른 센서 보정값(sensor calibration), 제어기 이득(controller gain), 네트워크 주소, 기계적 치수(mechanical dimension)가 필요할 수 있다. 네임스페이스가 적용된 노드는 자연스럽게 별도의 매개변수 컨텍스트(parameter context)를 제공하므로 공통 매개변수 파일과 로봇별 오버라이드(robot-specific override)를 조합할 수 있다. 설정 관리에서는 어떤 값이 소프트웨어 유형에 속하고 어떤 값이 개별 물리적 로봇을 설명하는지 명확하게 구분해야 한다.

로봇 네임스페이스 내부의 토픽은 일반적으로 로봇 로컬 데이터(robot-local data)를 나타낸다. 카메라 영상, LiDAR 스캔, 오도메트리(odometry), 관절 상태(joint state), 로컬 맵(local map), 속도 명령(velocity command), 배터리 상태, 진단 정보는 일반적으로 하나의 로봇에 속한다. 이러한 인터페이스를 네임스페이스로 구분하면 의도하지 않은 교차 연결을 줄일 수 있으며 여러 로봇이 동일한 ROS 2 네트워크에서 활성화되어 있을 때 토픽 검사, 시각화, 로깅(logging), 디버깅(debugging)을 더욱 쉽게 해석할 수 있다.

그러나 일부 정보는 개별 로봇이 아니라 실제로 플릿 전체에 속한다. 임무 할당(mission assignment), 공유 교통 정보(shared traffic information), 플릿 상태(fleet status), 인프라 상태 또는 협업 이벤트(coordination event)는 명시적으로 설계된 플릿 수준 인터페이스(fleet-level interface)를 사용할 수 있다. 이러한 인터페이스가 어떤 노드가 우연히 자신의 네임스페이스를 벗어났기 때문에 생성되어서는 안 된다. 명확한 아키텍처는 로봇 로컬 통신과 공유 협업 통신을 구분하고 두 범위 사이의 경계에서 데이터 계약(data contract)을 정의한다.

네임스페이스만으로 네트워크 격리(network isolation)가 제공되는 것은 아니다. ROS 2 검색(discovery)은 동일한 DDS 통신 환경에 있는 참여자들이 다른 로봇에 속한 엔드포인트를 계속 검색할 수 있도록 할 수 있다. 네임스페이스는 이름 충돌을 방지하고 그래프를 구조화하지만 트래픽, 검색 부하(discovery load), 비인가 접근(unauthorized access)을 자동으로 차단하지는 않는다. 따라서 대규모 플릿에서는 추가적인 DDS 설정, 도메인 분리(domain separation), 검색 제어(discovery control), 네트워크 세분화(network segmentation), 보안 메커니즘이 필요할 수 있다.

ROS 도메인 식별자(ROS domain identifier)는 로봇 그룹이 동일한 DDS 도메인에 참여하지 않아야 할 때 보다 광범위한 통신 경계를 제공할 수 있다. 서로 다른 시험 구역, 연구실, 고객 또는 운용 구역에 있는 로봇들은 완전한 검색 분리가 필요한 경우 서로 다른 도메인 ID(domain ID)를 할당받을 수 있다. 그러나 모든 로봇을 서로 다른 도메인에 배치하면 플릿 수준 통신이 복잡해질 수 있으므로 도메인 설계와 네임스페이스 설계는 서로 대체 가능한 수단이 아니라 상호 보완적인 아키텍처 메커니즘으로 다루어야 한다.

변환 데이터(transform data)는 ROS 노드를 네임스페이스에 배치하는 것만으로 TF 프레임 식별자(TF frame identifier) 문제가 자동으로 해결되지 않기 때문에 특별한 주의가 필요하다. 여러 로봇이 \`base_link\`, \`odom\`, \`map\`, \`laser\`라는 동일한 이름의 프레임을 발행하면 공유 변환 환경에서 프레임 식별이 모호해질 수 있다. 따라서 다중 로봇 아키텍처에는 위치 추정, 시각화, 매핑(mapping), 계획 요구사항과 일관성을 유지하는 로봇별 프레임 접두사(robot-specific frame prefix) 또는 분리된 변환 범위와 같은 명확한 TF 명명 전략이 필요하다.

전역 \`map\` 개념 역시 명시적인 설계 결정이 필요하다. 각 로봇이 자체 로컬 맵과 위치 추정 프레임(localization frame)을 유지할 수도 있고 여러 로봇이 공통 시설 좌표계(common facility coordinate system) 안에서 동작할 수도 있다. 공유 맵 아키텍처(shared-map architecture)에서는 로봇의 자세를 서로 구분할 수 있으면서 동시에 공통 기준 프레임으로 표현할 수 있어야 한다. 따라서 네임스페이스와 TF 규칙은 내비게이션 소프트웨어 통합 이후 독립적으로 결정하는 것이 아니라 처음부터 함께 설계해야 한다.

서비스(service)와 액션(action)에도 토픽과 동일한 네임스페이스 규칙이 필요하다. 위치 이동, 도킹(docking), 보정(calibration), 리셋(reset), 검사(inspection)와 같은 명령은 대상 로봇을 명확하게 식별해야 한다. \`/robot_02/navigate_to_pose\` 또는 \`/robot_04/dock\`과 같은 이름은 소유권을 명확하게 표현한다. 특히 액션은 장시간 실행되는 목표(goal), 피드백(feedback), 취소(cancellation), 결과(result)가 전체 수명주기 동안 올바른 로봇과 연결되어야 하므로 이러한 구분이 더욱 중요하다.

명명 규칙(naming convention)이 결정적이고 예측 가능하면 다중 로봇 모니터링(multi-robot monitoring)이 훨씬 쉬워진다. 운영자와 소프트웨어 도구는 로봇 네임스페이스를 기준으로 인터페이스를 필터링하고, 플릿 전체에서 동일한 진단 신호를 집계하거나, 서로 다른 로봇의 동일한 노드를 비교할 수 있다. 예측 가능한 패턴은 대시보드가 수동으로 관리되는 매핑에 의존하지 않고 이름에서 직접 로봇 식별자와 서브시스템 소유권을 추출할 수 있으므로 자동 로깅과 관측 가능성(observability)도 지원한다.

로깅 아키텍처(logging architecture)는 서로 다른 로봇의 동일한 토픽 이름을 오프라인 분석에서 구분하기 어려워지는 것을 방지하기 위해 네임스페이스 정보를 보존해야 한다. 기록된 데이터셋은 타임스탬프(timestamp), 메시지 스키마(message schema)와 함께 로봇 식별 정보를 유지해야 한다. 이는 여러 로봇의 데이터를 플릿 분석(fleet analytics), 장애 조사, 매핑 또는 피지컬 AI(Physical AI) 학습에 통합할 때 특히 중요하며, 각 관측값과 행동(action)의 출처를 추적할 수 있어야 한다.

확장성(scalability)을 확보하려면 플릿 전체에서 어떤 데이터를 공유할 것인지 제어해야 한다. 네임스페이스는 \`/robot_01/camera/image\`와 \`/robot_02/camera/image\`를 서로 다른 인터페이스로 만들지만 두 고대역폭 스트림을 동일한 무선 인프라 전체에 발행하면 여전히 네트워크가 과부하될 수 있다. 따라서 가능한 경우 로봇 로컬 인지는 원시 데이터를 로컬에서 처리하고, 플릿 인터페이스는 압축된 자세, 객체, 임무 상태, 상태 정보(health information), 협업 이벤트를 배포해야 한다.

다중 로봇 시스템은 플릿 규모에 대한 하드코딩된 가정(hard-coded assumption)도 피해야 한다. \`robot_01\`, \`robot_02\`, \`robot_03\`이 존재한다고 명시적으로 가정하는 소프트웨어는 로봇이 추가되거나 제거되거나 교체될 때 확장하기 어렵다. 로봇 식별 정보는 설정(configuration), 실행 인수, 검색 또는 플릿 관리 정보를 통해 제공되어야 한다. 그러면 응용 프로그램 소스 코드를 수정하지 않고도 변화하는 로봇 집단에 동일한 소프트웨어 패턴을 인스턴스화할 수 있다.

명명 규칙은 ROS 2 인터페이스 계약(interface contract)의 일부로 문서화해야 한다. 문서에는 로봇 식별자, 네임스페이스 계층, 노드 명명, 토픽 및 서비스 규칙, 액션 소유권, TF 프레임 전략, 공유 플릿 인터페이스, 절대 이름 사용 규칙을 정의해야 한다. 일관된 규칙을 적용하면 서로 다른 팀이 독립적으로 호환되지 않는 명명 패턴을 생성하고 시스템 규모가 커진 이후 이를 수정하기 어려워지는 문제를 예방할 수 있다.

시험(testing)은 각 로봇 스택을 개별적으로 검증하는 데 그치지 않고 여러 로봇 인스턴스(robot instance)를 포함해야 한다. 엔지니어는 동일한 소프트웨어 인스턴스가 동시에 정상적으로 실행되는지, 인터페이스가 서로 분리되는지, 명령이 의도된 로봇에만 전달되는지, TF 트리가 모호하지 않은지, 매개변수 설정이 인스턴스 사이에서 누출되지 않는지, 공유 플릿 인터페이스가 올바르게 동작하는지를 검증해야 한다. 또한 활성 로봇 수가 증가할 때 검색 동작과 네트워크 부하도 평가해야 한다.

시뮬레이션(simulation)은 실제 플릿을 배포하기 전에 네임스페이스 설계를 검증할 수 있는 효과적인 환경을 제공한다. 여러 시뮬레이션 로봇은 서로 다른 네임스페이스, 자세, 센서 설정, 임무를 사용하면서 동일한 ROS 2 스택을 실행할 수 있다. 이를 통해 재매핑, TF 프레임, 중복 이름, 공유 토픽, 실행 설정과 관련된 문제를 조기에 발견할 수 있다. 동일한 네임스페이스 전략을 실제 배포에도 적용하여 시뮬레이션과 실제 로봇이 동등한 인터페이스 구조를 유지하도록 해야 한다.

자율 로봇 플릿(autonomous robot fleet)과 피지컬 AI(Physical AI) 시스템에서 네임스페이스 설계는 궁극적으로 ROS 2 통신 그래프 내부에서 소유권(ownership), 범위(scope), 반복 가능성(repeatability)을 표현하는 메커니즘이다. 견고한 아키텍처는 로봇별 네임스페이스, 상대 인터페이스, 일관된 실행 설정, 명확한 TF 명명, 분리된 플릿 수준 통신, 적절한 DDS 네트워크 경계를 결합한다. 이를 통해 하나의 재사용 가능한 로봇 소프트웨어 스택을 추적성(traceability), 격리(isolation), 유지보수성(maintainability)을 희생하지 않고 여러 협업 로봇으로 확장할 수 있다.

##  

## 04.09 ROS2 Communication Security: SROS2 Config [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 communication security addresses the risk that distributed robot nodes may exchange commands, sensor data, state information, and configuration across networks that cannot always be assumed to be trusted. A robot may communicate over wired Ethernet, Wi-Fi, industrial networks, or multi-robot infrastructure, creating opportunities for unauthorized discovery, message interception, modification, or command injection. SROS2 provides ROS 2 mechanisms for applying security capabilities to this distributed communication environment.

SROS2 builds on security capabilities provided through DDS Security and exposes supporting tools and workflows for ROS 2 deployments. Rather than creating an independent encryption protocol above ROS topics, it uses middleware-level security mechanisms associated with DDS communication. This approach allows authentication, access control, and cryptographic protection to operate close to the communication layer while remaining compatible with the ROS 2 node and interface model.

Authentication establishes whether a communication participant has an approved identity. Without authentication, an unauthorized process could potentially appear on a reachable ROS 2 network and attempt to participate in discovery or communication. A secured deployment associates participants with cryptographic identity material, allowing middleware security mechanisms to verify identities before protected communication relationships are established.

Public-key infrastructure is therefore an important foundation of SROS2 configuration. Certificates, private keys, certificate authorities, and signed security artifacts establish trust relationships between ROS 2 participants. The private key represents sensitive identity material and must be protected carefully, while certificates allow identities to be verified against a trusted authority. Security depends not only on cryptographic algorithms but also on how these credentials are generated, distributed, stored, renewed, and revoked.

SROS2 uses a keystore-oriented workflow to organize security artifacts for a ROS 2 deployment. A keystore can contain the certificate authority information and security material required for individual node identities or enclaves. Tools can be used to create the keystore, generate identities, and prepare the files required by DDS Security. This provides a repeatable structure for managing credentials instead of manually placing unrelated certificate and policy files throughout a robot computer.

An enclave provides a security context that can be associated with one or more ROS 2 processes or nodes according to the deployment design. Enclaves help separate security identity from assumptions based only on executable or node names. A system can organize security artifacts according to functional boundaries such as navigation, perception, fleet integration, or diagnostics, allowing deployment security to evolve without embedding cryptographic details directly into application source code.

Authentication alone is insufficient because a valid participant should not automatically be allowed to access every interface. Authorization and access control determine which communication operations an authenticated participant may perform. A camera process may be permitted to publish image topics but should not necessarily publish velocity commands. A monitoring node may read diagnostic information while being prohibited from issuing navigation or actuator commands.

Security policies therefore express permitted communication behavior. Permissions can constrain which topics, services, actions, or underlying DDS entities a participant may use according to the generated policy and middleware capabilities. The objective is to implement least privilege: each software component receives only the communication permissions required for its assigned function. Limiting privileges reduces the potential impact of compromised, defective, or incorrectly configured components.

Policy generation should begin from a known communication architecture rather than from unrestricted access. Engineers need to understand which nodes publish, subscribe, provide services, invoke services, and participate in actions. Security configuration then becomes an extension of the ROS 2 interface contract. When the communication graph changes, the security policy should be reviewed together with the software interface instead of being treated as unrelated deployment metadata.

Confidentiality protects information from unauthorized observation while it is communicated between secured participants. Encryption can be important when robot data travels over wireless or shared infrastructure, particularly when messages contain operational information, maps, images, mission details, or proprietary data. However, encryption introduces computational and operational costs, so its impact should be evaluated on the actual edge computer and middleware configuration used by the robot.

Integrity protection addresses unauthorized modification of communication. For robotic systems, integrity may be even more operationally important than secrecy for certain interfaces because modified velocity commands, mission requests, localization information, or safety-related state could alter physical behavior. Cryptographic protection helps receivers detect communication that does not satisfy the expected security guarantees, but application-level validation is still required before safety-critical commands are executed.

Secure discovery is also important because ROS 2 relies heavily on distributed participant and endpoint discovery. An unsecured discovery environment can reveal system structure or permit unwanted participants to interact with the graph. DDS Security mechanisms can protect relevant discovery and communication relationships according to the selected implementation and configuration. Engineers should validate discovery behavior explicitly because security failures may appear as ordinary ROS 2 connectivity problems.

SROS2 security is commonly enabled through ROS 2 security configuration and environment settings that identify the keystore, security strategy, and enclave context used when nodes start. Deployment scripts and launch systems should apply these settings consistently rather than relying on developers to configure terminals manually. Reproducible startup configuration is essential because a robot that silently starts without its intended security context can create a dangerous difference between tested and deployed behavior.

A strict security strategy should fail closed when required security artifacts or permissions are unavailable. Allowing a process to fall back silently to unsecured communication can defeat the purpose of the security architecture. Production systems should define whether startup must stop, enter a degraded state, or isolate affected functionality when credentials are missing, expired, corrupted, or inconsistent with policy.

Security configuration interacts with namespaces and multi-robot architecture. A fleet may contain many instances of the same software stack, but each physical robot may require its own identity and trust relationships. Namespaces organize ROS interfaces, whereas cryptographic identities and permissions establish who is allowed to participate. The two mechanisms complement one another: a unique topic name distinguishes ownership semantically, while security policy controls authorized access to that communication.

External bridges and gateways deserve special attention because they connect the ROS 2 domain to cloud, enterprise, fleet, or legacy systems. A gateway may legitimately access interfaces that ordinary nodes cannot, making it a high-value security boundary. Its permissions should be narrowly defined, external input should be validated, and authentication between the gateway and external infrastructure should be handled independently from assumptions about trust inside the ROS 2 network.

Network segmentation remains useful even when SROS2 is enabled. Security should not rely on a single protective mechanism. Separating robot control networks from guest, enterprise, or public networks reduces unnecessary exposure and discovery traffic. Firewalls, VLANs, wireless security, VPNs, DDS configuration, and SROS2 can form complementary layers, with each mechanism reducing the consequences if another layer is incorrectly configured or compromised.

Security also has a performance cost that should be measured rather than assumed. Authentication affects connection establishment, while encryption and integrity protection add processing and message overhead. The impact may be negligible for low-rate control information but more noticeable for high-bandwidth images or point clouds. Benchmarks should measure latency, CPU utilization, throughput, and startup behavior with the same security settings intended for production deployment.

Real-time and safety-sensitive paths require particular care. Cryptographic processing should not introduce uncontrolled latency or resource contention into timing-critical control loops. At the same time, bypassing security for convenience may expose highly consequential interfaces. A robust architecture separates hard real-time motor control from broader distributed communication where appropriate and applies security at boundaries consistent with both timing requirements and the system threat model.

Credential lifecycle management is as important as initial configuration. Certificates and keys may need expiration policies, renewal, replacement, revocation, secure backup, and recovery procedures. A fleet containing tens or hundreds of robots cannot depend on manually generated credentials that remain unchanged indefinitely. Identity provisioning should therefore become part of manufacturing, commissioning, maintenance, software update, and decommissioning processes.

Private keys and security artifacts should be protected at rest as well as during communication. File permissions provide a basic protection layer, while production platforms may use stronger mechanisms such as hardware-backed key storage or trusted security devices when justified by the threat model. Backups must also be protected because copying a credential store without adequate controls can undermine otherwise strong communication security.

Observability is necessary for operating a secured ROS 2 system. Authentication failures, denied permissions, certificate problems, policy mismatches, enclave configuration errors, and secure discovery failures should generate diagnostics that can be distinguished from ordinary network faults. Logs must provide enough information for troubleshooting without exposing private keys, secrets, or other sensitive security material.

Testing should include negative security cases, not only successful communication. Engineers should verify that authorized publishers and subscribers communicate correctly while unauthorized participants are rejected. Tests should examine invalid credentials, forbidden topics, expired or missing security artifacts, incorrect enclave configuration, restart behavior, and network interruption. Demonstrating that prohibited communication fails is an essential part of validating the security architecture.

For autonomous robots and Physical AI systems, SROS2 configuration should be treated as part of system architecture rather than a final deployment option. Authentication establishes trusted identities, access control limits communication privileges, cryptographic protection supports confidentiality and integrity, and disciplined credential management preserves these properties throughout the robot lifecycle. Combined with network segmentation, command validation, monitoring, and least-privilege design, SROS2 helps transform an open ROS 2 communication graph into a controlled distributed robotic system.

ROS 2 통신 보안(communication security)은 분산된 로봇 노드가 항상 신뢰할 수 있다고 가정할 수 없는 네트워크를 통해 명령, 센서 데이터, 상태 정보 및 설정 정보를 교환할 때 발생하는 위험을 다룬다. 로봇은 유선 이더넷(Ethernet), Wi-Fi, 산업용 네트워크 또는 다중 로봇 인프라를 통해 통신할 수 있으며, 이 과정에서 비인가 검색(unauthorized discovery), 메시지 도청(message interception), 변조(modification), 명령 주입(command injection)의 가능성이 발생한다. SROS2는 이러한 분산 통신 환경에 보안 기능을 적용하기 위한 ROS 2 메커니즘을 제공한다.

SROS2는 DDS 보안(DDS Security)을 통해 제공되는 보안 기능을 기반으로 구축되며, ROS 2 배포를 위한 지원 도구와 작업 흐름(workflow)을 제공한다. ROS 토픽 위에 독립적인 암호화 프로토콜을 만드는 대신 DDS 통신과 연계된 미들웨어 수준 보안 메커니즘(middleware-level security mechanism)을 사용한다. 이러한 접근 방식을 통해 인증(authentication), 접근 제어(access control), 암호학적 보호(cryptographic protection)가 ROS 2 노드 및 인터페이스 모델과 호환성을 유지하면서 통신 계층 가까이에서 동작할 수 있다.

인증(authentication)은 통신 참여자(communication participant)가 승인된 신원을 가지고 있는지를 확인한다. 인증이 없다면 비인가 프로세스가 접근 가능한 ROS 2 네트워크에 나타나 검색이나 통신에 참여하려고 시도할 수 있다. 보안이 적용된 배포 환경에서는 참여자에게 암호학적 신원 정보(cryptographic identity material)를 연결하여 보호된 통신 관계가 설정되기 전에 미들웨어 보안 메커니즘이 해당 신원을 검증할 수 있도록 한다.

따라서 공개 키 기반 구조(Public Key Infrastructure, PKI)는 SROS2 설정의 중요한 기반이다. 인증서(certificate), 개인 키(private key), 인증 기관(Certificate Authority, CA), 서명된 보안 아티팩트(signed security artifact)는 ROS 2 참여자 사이의 신뢰 관계를 형성한다. 개인 키는 민감한 신원 정보를 나타내므로 신중하게 보호해야 하며, 인증서는 신뢰할 수 있는 기관을 기준으로 신원을 검증할 수 있게 한다. 보안은 암호 알고리즘뿐 아니라 이러한 자격 증명(credential)을 어떻게 생성, 배포, 저장, 갱신 및 폐기하는지에도 좌우된다.

SROS2는 ROS 2 배포에 필요한 보안 아티팩트를 체계적으로 관리하기 위해 키스토어 기반 작업 흐름(keystore-oriented workflow)을 사용한다. 키스토어(keystore)는 인증 기관 정보와 개별 노드 신원 또는 엔클레이브(enclave)에 필요한 보안 자료를 포함할 수 있다. 도구를 이용하여 키스토어를 생성하고 신원을 생성하며 DDS 보안에 필요한 파일을 준비할 수 있다. 이를 통해 서로 관련 없는 인증서와 정책 파일을 로봇 컴퓨터 곳곳에 수동으로 배치하는 대신 자격 증명을 반복 가능한 구조로 관리할 수 있다.

엔클레이브(enclave)는 배포 설계에 따라 하나 이상의 ROS 2 프로세스 또는 노드와 연결할 수 있는 보안 컨텍스트(security context)를 제공한다. 엔클레이브는 실행 파일이나 노드 이름만을 기준으로 하는 가정으로부터 보안 신원을 분리하는 데 도움을 준다. 시스템은 내비게이션(navigation), 인지(perception), 플릿 통합(fleet integration), 진단(diagnostics)과 같은 기능적 경계에 따라 보안 아티팩트를 구성할 수 있으며, 이를 통해 암호학적 세부사항을 응용 프로그램 소스 코드에 직접 포함하지 않고도 배포 보안을 발전시킬 수 있다.

인증만으로는 충분하지 않다. 유효한 참여자라고 해서 모든 인터페이스에 자동으로 접근할 수 있어서는 안 되기 때문이다. 권한 부여(authorization)와 접근 제어(access control)는 인증된 참여자가 어떤 통신 작업을 수행할 수 있는지를 결정한다. 카메라 프로세스는 영상 토픽을 발행할 수 있지만 속도 명령을 발행할 필요는 없으며, 모니터링 노드는 진단 정보를 읽을 수 있지만 내비게이션이나 액추에이터 명령을 발행하지 못하도록 제한할 수 있다.

따라서 보안 정책(security policy)은 허용되는 통신 동작을 표현한다. 권한(permission)은 생성된 정책과 미들웨어 기능에 따라 참여자가 사용할 수 있는 토픽, 서비스(service), 액션(action) 또는 하위 DDS 엔티티를 제한할 수 있다. 목표는 최소 권한(least privilege)을 구현하는 것이다. 각 소프트웨어 컴포넌트에는 할당된 기능을 수행하는 데 필요한 통신 권한만 제공한다. 권한을 제한하면 침해되거나 결함이 있거나 잘못 설정된 컴포넌트가 시스템에 미치는 잠재적인 영향을 줄일 수 있다.

정책 생성(policy generation)은 무제한 접근을 기본으로 시작하는 것이 아니라 이미 정의된 통신 아키텍처를 기반으로 시작해야 한다. 엔지니어는 어떤 노드가 발행(publish), 구독(subscribe), 서비스 제공, 서비스 호출 및 액션에 참여하는지 이해해야 한다. 그러면 보안 설정은 ROS 2 인터페이스 계약(interface contract)의 확장이 된다. 통신 그래프가 변경될 경우 보안 정책 역시 관련 없는 배포 메타데이터로 취급하는 대신 소프트웨어 인터페이스와 함께 검토해야 한다.

기밀성(confidentiality)은 보안 참여자 사이에서 통신되는 정보를 비인가 관찰로부터 보호한다. 암호화(encryption)는 로봇 데이터가 무선 또는 공유 인프라를 통과할 때 중요할 수 있으며, 특히 메시지에 운용 정보, 맵, 영상, 임무 정보 또는 독점 데이터가 포함되는 경우 중요성이 높아진다. 그러나 암호화에는 계산 및 운용 비용이 추가되므로 실제 로봇에서 사용하는 엣지 컴퓨터(edge computer)와 미들웨어 설정을 기반으로 그 영향을 평가해야 한다.

무결성 보호(integrity protection)는 통신 내용의 비인가 변경을 방지하거나 탐지하는 것을 다룬다. 로봇 시스템의 일부 인터페이스에서는 변조된 속도 명령, 임무 요청, 위치 추정 정보 또는 안전 관련 상태가 물리적 동작을 변경할 수 있으므로 기밀성보다 무결성이 운용 측면에서 더욱 중요할 수도 있다. 암호학적 보호는 예상되는 보안 보장을 충족하지 않는 통신을 수신자가 탐지하도록 지원하지만, 안전 중요 명령을 실행하기 전에는 여전히 응용 프로그램 수준 검증(application-level validation)이 필요하다.

ROS 2는 분산 참여자 및 엔드포인트 검색(discovery)에 크게 의존하므로 보안 검색(secure discovery) 역시 중요하다. 보안이 적용되지 않은 검색 환경은 시스템 구조를 노출하거나 원하지 않는 참여자가 통신 그래프와 상호작용하도록 허용할 수 있다. DDS 보안 메커니즘은 선택한 구현 및 설정에 따라 관련 검색 및 통신 관계를 보호할 수 있다. 보안 장애가 일반적인 ROS 2 연결 문제처럼 보일 수 있으므로 엔지니어는 검색 동작을 명시적으로 검증해야 한다.

SROS2 보안은 일반적으로 노드가 시작될 때 사용할 키스토어, 보안 전략(security strategy), 엔클레이브 컨텍스트를 지정하는 ROS 2 보안 설정 및 환경 설정(environment setting)을 통해 활성화된다. 배포 스크립트(deployment script)와 실행 시스템(launch system)은 개발자가 터미널에서 수동으로 설정하는 방식에 의존하지 않고 이러한 설정을 일관되게 적용해야 한다. 의도된 보안 컨텍스트 없이 로봇이 조용히 시작된다면 시험 환경과 실제 배포 환경 사이에 위험한 차이가 발생할 수 있으므로 재현 가능한 시작 설정이 필수적이다.

엄격한 보안 전략(strict security strategy)은 필요한 보안 아티팩트나 권한을 사용할 수 없는 경우 안전하게 실패(fail closed)해야 한다. 프로세스가 자동으로 비보안 통신으로 전환되도록 허용하면 보안 아키텍처의 목적 자체가 무력화될 수 있다. 생산 시스템은 자격 증명이 누락되거나 만료되거나 손상되거나 정책과 일치하지 않을 경우 시작을 중단할 것인지, 성능 저하 상태(degraded state)로 진입할 것인지, 또는 영향을 받는 기능을 격리할 것인지를 정의해야 한다.

보안 설정은 네임스페이스(namespace) 및 다중 로봇 아키텍처와 상호작용한다. 플릿에는 동일한 소프트웨어 스택의 여러 인스턴스가 존재할 수 있지만 각 물리적 로봇에는 고유한 신원과 신뢰 관계가 필요할 수 있다. 네임스페이스는 ROS 인터페이스를 구조화하는 반면 암호학적 신원과 권한은 누가 통신에 참여할 수 있는지를 결정한다. 두 메커니즘은 상호 보완적이며, 고유한 토픽 이름은 의미적으로 소유권을 구분하고 보안 정책은 해당 통신에 대한 승인된 접근을 제어한다.

외부 브리지(bridge)와 게이트웨이(gateway)는 ROS 2 도메인을 클라우드, 기업 시스템, 플릿 또는 레거시 시스템과 연결하므로 특별한 주의가 필요하다. 게이트웨이는 일반 노드가 접근할 수 없는 인터페이스에 합법적으로 접근할 수 있으므로 중요한 보안 경계(security boundary)가 될 수 있다. 게이트웨이 권한은 가능한 한 좁게 정의하고 외부 입력을 검증해야 하며, 게이트웨이와 외부 인프라 사이의 인증은 ROS 2 네트워크 내부의 신뢰에 대한 가정과 독립적으로 처리해야 한다.

SROS2가 활성화되어 있더라도 네트워크 분할(network segmentation)은 여전히 유용하다. 보안은 하나의 보호 메커니즘에만 의존해서는 안 된다. 로봇 제어 네트워크를 게스트, 기업 또는 공용 네트워크와 분리하면 불필요한 노출과 검색 트래픽을 줄일 수 있다. 방화벽(firewall), VLAN, 무선 보안, 가상 사설망(Virtual Private Network, VPN), DDS 설정, SROS2는 상호 보완적인 계층을 구성할 수 있으며, 하나의 계층이 잘못 설정되거나 침해되더라도 다른 계층이 그 영향을 줄일 수 있다.

보안에는 성능 비용(performance cost)도 발생하므로 이를 추측하기보다는 실제로 측정해야 한다. 인증은 연결 설정에 영향을 주고 암호화와 무결성 보호는 처리 부하와 메시지 오버헤드를 추가한다. 저주기 제어 정보에서는 영향이 미미할 수 있지만 고대역폭 영상이나 포인트 클라우드에서는 더 두드러질 수 있다. 벤치마크(benchmark)는 실제 생산 배포에 사용할 것과 동일한 보안 설정을 적용하여 지연시간, CPU 사용률, 처리량(throughput), 시작 동작을 측정해야 한다.

실시간(real-time) 및 안전 민감 경로(safety-sensitive path)는 특별한 주의가 필요하다. 암호학적 처리가 시간 중요 제어 루프에 통제되지 않은 지연이나 자원 경쟁(resource contention)을 발생시켜서는 안 된다. 동시에 편의를 위해 보안을 우회하면 매우 중요한 인터페이스가 외부에 노출될 수 있다. 견고한 아키텍처는 필요한 경우 하드 실시간 모터 제어(hard real-time motor control)를 광범위한 분산 통신과 분리하고, 시간 요구사항과 시스템 위협 모델(threat model)에 모두 부합하는 경계에서 보안을 적용한다.

자격 증명 수명주기 관리(credential lifecycle management)는 초기 설정만큼 중요하다. 인증서와 키에는 만료 정책, 갱신, 교체, 폐기(revocation), 안전한 백업 및 복구 절차가 필요할 수 있다. 수십 대 또는 수백 대의 로봇으로 구성된 플릿은 수동으로 생성되어 무기한 변경되지 않는 자격 증명에 의존할 수 없다. 따라서 신원 프로비저닝(identity provisioning)은 제조, 시운전(commissioning), 유지보수, 소프트웨어 업데이트 및 폐기(decommissioning) 프로세스의 일부가 되어야 한다.

개인 키와 보안 아티팩트는 통신 중뿐 아니라 저장 상태(at rest)에서도 보호해야 한다. 파일 권한(file permission)은 기본적인 보호 계층을 제공하며, 생산 플랫폼에서는 위협 모델에 따라 필요한 경우 하드웨어 기반 키 저장소(hardware-backed key storage) 또는 신뢰할 수 있는 보안 장치(trusted security device)와 같은 보다 강력한 메커니즘을 사용할 수 있다. 자격 증명 저장소를 충분한 통제 없이 복사하면 강력한 통신 보안도 무력화될 수 있으므로 백업 역시 보호해야 한다.

보안이 적용된 ROS 2 시스템을 운영하려면 관측 가능성(observability)이 필요하다. 인증 실패, 거부된 권한, 인증서 문제, 정책 불일치, 엔클레이브 설정 오류, 보안 검색 실패는 일반적인 네트워크 장애와 구분할 수 있는 진단 정보를 생성해야 한다. 로그(log)는 문제 해결에 충분한 정보를 제공하면서도 개인 키, 비밀 정보(secret), 기타 민감한 보안 자료를 노출해서는 안 된다.

시험(testing)은 성공적인 통신뿐 아니라 부정적 보안 사례(negative security case)도 포함해야 한다. 엔지니어는 승인된 발행자와 구독자가 정상적으로 통신하는 동시에 비인가 참여자가 거부되는지를 검증해야 한다. 잘못된 자격 증명, 금지된 토픽, 만료되거나 누락된 보안 아티팩트, 잘못된 엔클레이브 설정, 재시작 동작, 네트워크 중단 등을 시험해야 한다. 금지된 통신이 실제로 실패한다는 것을 입증하는 것은 보안 아키텍처 검증의 핵심 요소이다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 SROS2 설정은 최종 배포 단계에서 선택적으로 추가하는 기능이 아니라 시스템 아키텍처의 일부로 다루어야 한다. 인증은 신뢰할 수 있는 신원을 확립하고, 접근 제어는 통신 권한을 제한하며, 암호학적 보호는 기밀성과 무결성을 지원하고, 체계적인 자격 증명 관리는 로봇 수명주기 전체에서 이러한 특성을 유지한다. 네트워크 분할, 명령 검증, 모니터링, 최소 권한 설계와 결합된 SROS2는 개방된 ROS 2 통신 그래프를 통제된 분산 로봇 시스템(controlled distributed robotic system)으로 전환하는 데 도움을 준다.

##  

## 04.10 ROS2 Communication Latency Benchmark Methodology [w/Code]

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

Communication latency in ROS 2 is the elapsed time required for information to travel from a producing component to the point where a receiving component can use it. Although a simple publisher-to-subscriber test can produce a latency number, meaningful benchmarking requires a controlled methodology. Message size, publication rate, QoS, middleware, executor behavior, CPU load, network conditions, and timestamp placement can all change the result substantially.

A benchmark must first define exactly what latency is being measured. Transport latency may describe the interval between publication and message reception, while application-level latency can include message construction, serialization, queueing, callback scheduling, deserialization, and processing. End-to-end robotic latency may extend further from sensor acquisition to actuator command generation. These measurements answer different engineering questions and should not be reported as if they were equivalent.

Timestamp placement determines the boundaries of the measurement. A timestamp inserted before message construction measures a different path from one inserted immediately before publication. Similarly, measuring at subscriber callback entry differs from measuring after application processing finishes. Benchmark documentation should identify every measurement point explicitly so another engineer can reproduce the test and understand which parts of the ROS 2 pipeline are included.

One-way latency measurement requires synchronized clocks when publisher and subscriber execute on different computers. Without adequate synchronization, clock offset and drift can be mistaken for communication delay. Protocols such as NTP may be sufficient for some general tests, while higher-precision experiments may require PTP or hardware-supported synchronization. The required clock accuracy should be significantly better than the latency differences the benchmark is intended to distinguish.

Round-trip latency avoids some cross-machine clock synchronization problems by sending a message to another endpoint and returning a response to the original computer. The originating process measures the complete elapsed time using one clock. Dividing the round-trip time by two may provide a rough estimate under symmetric conditions, but it should not automatically be interpreted as exact one-way latency because forward and reverse processing or network paths may differ.

Benchmark messages should represent the workloads expected in the actual robot. Testing only a tiny integer message provides limited information about systems that transmit images, point clouds, maps, trajectories, or large AI outputs. A useful benchmark evaluates several payload classes, ranging from small control and state messages to medium structured data and large sensor payloads. This reveals where serialization, copying, fragmentation, and bandwidth begin to dominate latency.

Publication rate is equally important. A communication path that performs well at 10 Hz may behave very differently at 100 Hz or when several sensors publish simultaneously. Increasing the rate can create queue buildup, CPU contention, network congestion, and subscriber backlog. Benchmarks should therefore test realistic nominal rates and selected stress conditions rather than reporting the best latency observed under an artificially light communication workload.

QoS configuration must be recorded as part of every result. Reliable and Best Effort communication can produce different behavior, especially under packet loss or congestion. History depth, durability, and other relevant policies can influence queueing and startup behavior. Comparing middleware implementations or network configurations while silently changing QoS makes the benchmark difficult to interpret because the communication contract itself has changed.

Reliable communication should be evaluated under both clean and impaired network conditions when the deployment uses distributed computers. On an idle wired network, retransmission behavior may rarely appear. Under packet loss, wireless interference, or congestion, reliability can increase tail latency while attempting to recover missing data. Best Effort may lose samples instead. A meaningful benchmark therefore reports both latency and message-delivery behavior rather than treating them as unrelated metrics.

Average latency alone is insufficient for robotic communication. A system with a low mean value can still experience occasional delays large enough to disrupt perception, planning, or control. Measurements should include distribution-oriented statistics such as median, percentiles, minimum, maximum, and jitter. High-percentile values such as the 95th, 99th, or 99.9th percentile are particularly useful for exposing rare delays hidden by an average.

Jitter describes variation in timing and is often as important as nominal latency. Predictable communication at a slightly higher latency may be easier to engineer around than communication that is usually fast but occasionally stalls. Executor scheduling, operating-system activity, memory allocation, background processes, thermal throttling, network contention, and retransmission can all introduce jitter. Benchmarking should therefore collect enough samples to characterize variability rather than relying on a few message exchanges.

A warm-up period should normally precede measurement. Initial discovery, memory allocation, cache population, dynamic library activity, and connection establishment can distort early samples. Startup latency may be an important metric in its own right, but it should be measured separately from steady-state communication latency. Clearly separating startup and steady-state behavior prevents transient initialization effects from contaminating long-duration performance statistics.

The execution environment should be controlled and documented. CPU model, core count, operating system, ROS 2 distribution, RMW implementation, DDS vendor, network interface, link speed, and relevant middleware settings can affect results. CPU frequency scaling, power modes, containerization, virtualization, and real-time kernel configuration may also matter. Benchmark numbers without platform context are difficult to reproduce or compare.

Executor configuration can strongly influence callback latency. A message may already be available in middleware while the subscriber callback waits because another callback is executing. Single-threaded and multi-threaded executors, callback groups, callback duration, and competing timers can therefore alter observed application latency without changing network transport. Tests should distinguish middleware transport behavior from executor-induced scheduling delay whenever possible.

CPU affinity and background workload should be considered when deterministic behavior is important. Pinning critical benchmark processes to selected cores can reduce scheduler migration and interference, while uncontrolled logging, visualization, recording, AI inference, or system services may increase variability. Both isolated and realistic-load tests are valuable: isolated tests characterize the communication mechanism, whereas loaded tests indicate whether the robot can maintain performance during actual operation.

Intra-process and inter-process communication should be benchmarked separately because their data paths differ substantially. Components within the same process may benefit from ROS 2 intra-process optimization and reduced copying, while separate processes require additional middleware operations. Communication between computers introduces the network stack and physical link. Reporting these configurations independently reveals where architectural boundaries add latency.

Large-message benchmarks should monitor more than elapsed time. CPU utilization, memory bandwidth, network throughput, allocation behavior, dropped samples, queue depth, and subscriber processing rate can explain why latency changes as payload size increases. A sudden increase may indicate fragmentation, serialization overhead, network saturation, or a consumer that cannot keep pace. Supporting resource metrics make latency measurements diagnostically useful rather than merely descriptive.

Tracing provides deeper visibility when simple timestamps cannot identify the source of delay. ROS 2 tracing and operating-system instrumentation can expose callback execution, executor scheduling, publication events, and other relevant transitions. Combined with middleware and network observations, tracing helps decompose end-to-end latency into software stages. This is particularly useful when an occasional long-tail delay cannot be reproduced from application timestamps alone.

Network benchmarks should describe topology and traffic conditions. Direct Ethernet, switched Ethernet, Wi-Fi, VPN, and routed networks have different delay and jitter characteristics. Link speed alone does not describe the environment. Competing traffic, switch behavior, wireless signal quality, packet loss, and multicast discovery traffic can influence ROS 2 performance. Production validation should therefore reproduce the network architecture intended for deployment.

Security configuration must also remain consistent during comparison. SROS2 authentication, encryption, and integrity protection can add processing and communication overhead. Measuring an unsecured development system and assuming identical production latency after security is enabled can produce misleading conclusions. When security is required operationally, final latency benchmarks should use the same security configuration, certificates, middleware settings, and network boundaries as the deployed robot.

Multi-robot benchmarking introduces another dimension because discovery and traffic increase as the fleet grows. A single robot may show excellent communication performance while dozens of participants on the same network increase discovery activity and bandwidth contention. Tests can progressively increase robot or participant count while measuring latency, jitter, packet loss, CPU utilization, and discovery behavior to identify practical scaling limits.

Benchmark experiments should be automated whenever possible. Scripts can establish configurations, launch publishers and subscribers, apply known message sizes and rates, collect measurements, and save environment metadata. Repeated runs reduce the risk that a single favorable or unfavorable execution determines the conclusion. Automated testing also allows communication performance to become part of regression testing when ROS 2, middleware, kernel, or application software changes.

Results should be reported with sufficient context to support engineering decisions. A useful report identifies hardware and software configuration, topology, payload, rate, QoS, sample count, clock method, system load, and statistical metrics. Graphs of latency distributions or latency versus payload and rate can reveal behavior that a single number cannot. Raw measurements should be retained when practical so alternative statistical analysis can be performed later.

For autonomous robots and Physical AI systems, the most valuable benchmark is one that represents the real information path and operational workload. Communication should be measured with realistic sensor rates, perception load, security, network conditions, and multiple active components rather than in an empty demonstration environment. A disciplined methodology transforms ROS 2 latency from an isolated benchmark number into evidence that the complete robotic architecture can satisfy its timing requirements.

ROS 2에서 통신 지연시간(communication latency)은 정보를 생성하는 컴포넌트(component)에서 수신 컴포넌트가 해당 정보를 사용할 수 있는 지점까지 전달되는 데 걸리는 시간이다. 단순한 발행자-구독자(publisher-subscriber) 시험으로도 하나의 지연시간 값을 얻을 수 있지만, 의미 있는 벤치마킹(benchmarking)을 위해서는 통제된 방법론이 필요하다. 메시지 크기, 발행 주기(publication rate), QoS, 미들웨어(middleware), 실행기(executor) 동작, CPU 부하, 네트워크 조건, 타임스탬프(timestamp) 위치에 따라 결과가 크게 달라질 수 있다.

벤치마크(benchmark)는 먼저 정확히 어떤 지연시간을 측정하는지 정의해야 한다. 전송 지연시간(transport latency)은 발행에서 메시지 수신까지의 시간을 의미할 수 있으며, 응용 프로그램 수준 지연시간(application-level latency)은 메시지 생성, 직렬화(serialization), 큐잉(queueing), 콜백 스케줄링(callback scheduling), 역직렬화(deserialization), 처리 과정까지 포함할 수 있다. 종단 간 로봇 지연시간(end-to-end robotic latency)은 센서 데이터 획득에서 액추에이터 명령 생성까지 더 넓은 범위를 포함할 수 있다. 이러한 측정값은 서로 다른 엔지니어링 질문에 답하므로 동일한 값처럼 보고해서는 안 된다.

타임스탬프 위치(timestamp placement)는 측정 범위를 결정한다. 메시지를 생성하기 전에 삽입한 타임스탬프는 발행 직전에 삽입한 타임스탬프와 서로 다른 데이터 경로를 측정한다. 마찬가지로 구독자 콜백(subscriber callback)에 진입하는 시점의 측정과 응용 프로그램 처리가 완료된 이후의 측정도 서로 다르다. 벤치마크 문서에는 모든 측정 지점을 명확하게 정의하여 다른 엔지니어가 시험을 재현하고 ROS 2 파이프라인의 어떤 부분이 포함되는지 이해할 수 있도록 해야 한다.

단방향 지연시간(one-way latency)을 측정할 때 발행자와 구독자가 서로 다른 컴퓨터에서 실행된다면 동기화된 클록(synchronized clock)이 필요하다. 충분한 동기화가 이루어지지 않으면 클록 오프셋(clock offset)과 드리프트(drift)가 통신 지연으로 잘못 측정될 수 있다. 일반적인 시험에서는 네트워크 시간 프로토콜(Network Time Protocol, NTP)이 충분할 수 있지만 더 높은 정밀도가 필요한 실험에서는 정밀 시간 프로토콜(Precision Time Protocol, PTP) 또는 하드웨어 지원 동기화가 필요할 수 있다. 필요한 클록 정확도는 벤치마크에서 구분하려는 지연시간 차이보다 충분히 높아야 한다.

왕복 지연시간(round-trip latency)은 메시지를 다른 엔드포인트로 전송한 뒤 응답을 원래 컴퓨터로 되돌려 하나의 클록으로 전체 경과 시간을 측정하므로 컴퓨터 간 클록 동기화 문제의 일부를 피할 수 있다. 대칭적인 조건에서는 왕복 시간(round-trip time)을 2로 나누어 대략적인 단방향 지연시간을 추정할 수 있다. 그러나 순방향과 역방향의 처리 과정이나 네트워크 경로가 서로 다를 수 있으므로 이를 정확한 단방향 지연시간으로 자동 해석해서는 안 된다.

벤치마크 메시지는 실제 로봇에서 예상되는 작업 부하(workload)를 대표해야 한다. 작은 정수 메시지만 시험하는 것은 영상, 포인트 클라우드(point cloud), 맵(map), 궤적(trajectory), 대용량 AI 출력을 전송하는 시스템에 제한적인 정보만 제공한다. 유용한 벤치마크는 작은 제어 및 상태 메시지에서 중간 크기의 구조화 데이터, 대용량 센서 페이로드(payload)에 이르는 여러 데이터 크기를 평가해야 한다. 이를 통해 어느 지점부터 직렬화, 복사, 단편화(fragmentation), 대역폭이 지연시간을 지배하기 시작하는지 파악할 수 있다.

발행 주기(publication rate) 역시 중요하다. 10 Hz에서 정상적으로 동작하는 통신 경로도 100 Hz 또는 여러 센서가 동시에 데이터를 발행하는 환경에서는 매우 다른 특성을 보일 수 있다. 발행 주기가 증가하면 큐 누적(queue buildup), CPU 경쟁(contention), 네트워크 혼잡(network congestion), 구독자 처리 지연(subscriber backlog)이 발생할 수 있다. 따라서 벤치마크는 인위적으로 가벼운 통신 부하에서 얻은 최상의 지연시간만 보고하는 대신 실제 운용에 가까운 정상 발행 주기와 선택된 스트레스 조건(stress condition)을 함께 시험해야 한다.

QoS 설정은 모든 측정 결과의 일부로 기록해야 한다. 신뢰성 통신(Reliable)과 최선형 통신(Best Effort)은 특히 패킷 손실이나 혼잡 상황에서 서로 다른 동작을 보일 수 있다. 이력 깊이(history depth), 내구성(durability) 및 기타 관련 정책도 큐잉과 시작 동작에 영향을 줄 수 있다. QoS를 명확히 기록하지 않은 상태에서 미들웨어 구현이나 네트워크 설정을 비교하면 통신 계약 자체가 변경된 것이므로 벤치마크 결과를 해석하기 어려워진다.

분산 컴퓨터를 사용하는 배포 환경에서는 신뢰성 통신(reliable communication)을 정상 네트워크와 품질이 저하된 네트워크 조건 모두에서 평가해야 한다. 유휴 상태의 유선 네트워크에서는 재전송 동작이 거의 나타나지 않을 수 있지만 패킷 손실, 무선 간섭 또는 혼잡 상황에서는 누락 데이터를 복구하는 과정에서 신뢰성 통신의 꼬리 지연시간(tail latency)이 증가할 수 있다. 반면 최선형 통신은 샘플을 손실할 수 있다. 따라서 의미 있는 벤치마크에서는 지연시간과 메시지 전달 동작(message-delivery behavior)을 서로 독립된 항목으로 취급하지 않고 함께 보고해야 한다.

평균 지연시간(average latency)만으로는 로봇 통신 성능을 충분히 평가할 수 없다. 평균값이 낮은 시스템도 인지, 계획 또는 제어를 방해할 정도로 큰 지연이 간헐적으로 발생할 수 있다. 측정 결과에는 중앙값(median), 백분위수(percentile), 최소값, 최대값, 지터(jitter)와 같은 분포 기반 통계가 포함되어야 한다. 특히 95번째, 99번째 또는 99.9번째 백분위수와 같은 높은 백분위 값은 평균값에 숨겨진 드문 지연을 확인하는 데 유용하다.

지터(jitter)는 시간 변화량을 나타내며 일반적인 지연시간만큼 중요할 수 있다. 약간 더 높은 지연시간을 가지더라도 예측 가능한 통신은 평소에는 빠르지만 간헐적으로 정지하는 통신보다 시스템 설계가 쉬울 수 있다. 실행기 스케줄링(executor scheduling), 운영체제 활동, 메모리 할당, 백그라운드 프로세스, 열 스로틀링(thermal throttling), 네트워크 경쟁, 재전송 등이 지터를 발생시킬 수 있다. 따라서 몇 번의 메시지 교환만으로 판단하지 말고 충분한 샘플을 수집하여 변동성을 분석해야 한다.

일반적으로 실제 측정 전에 워밍업 구간(warm-up period)을 두어야 한다. 초기 검색(discovery), 메모리 할당, 캐시 채우기(cache population), 동적 라이브러리 활동, 연결 설정이 초기 샘플을 왜곡할 수 있기 때문이다. 시작 지연시간(startup latency) 자체도 중요한 지표가 될 수 있지만 정상 상태 통신 지연시간(steady-state communication latency)과는 별도로 측정해야 한다. 시작 과정과 정상 상태를 명확하게 분리하면 일시적인 초기화 효과가 장시간 성능 통계를 왜곡하는 것을 방지할 수 있다.

실행 환경(execution environment)은 통제되고 문서화되어야 한다. CPU 모델, 코어 수, 운영체제, ROS 2 배포판(distribution), RMW 구현, DDS 공급업체(vendor), 네트워크 인터페이스, 링크 속도 및 관련 미들웨어 설정이 결과에 영향을 줄 수 있다. CPU 주파수 스케일링(frequency scaling), 전원 모드, 컨테이너화(containerization), 가상화(virtualization), 실시간 커널(real-time kernel) 설정도 영향을 줄 수 있다. 플랫폼 정보가 없는 벤치마크 수치는 재현하거나 비교하기 어렵다.

실행기 설정(executor configuration)은 콜백 지연시간에 큰 영향을 줄 수 있다. 메시지가 이미 미들웨어에 도착했더라도 다른 콜백이 실행되고 있다면 구독자 콜백은 대기할 수 있다. 단일 스레드 실행기(single-threaded executor), 다중 스레드 실행기(multi-threaded executor), 콜백 그룹(callback group), 콜백 실행 시간, 경쟁 타이머(timer)는 네트워크 전송이 변하지 않더라도 관측되는 응용 프로그램 지연시간을 변경할 수 있다. 가능한 경우 미들웨어 전송 동작과 실행기에서 발생하는 스케줄링 지연을 구분하여 시험해야 한다.

결정론적 동작(deterministic behavior)이 중요하다면 CPU 친화도(CPU affinity)와 백그라운드 작업 부하도 고려해야 한다. 중요한 벤치마크 프로세스를 특정 코어에 고정하면 스케줄러 이동과 간섭을 줄일 수 있지만, 제어되지 않은 로깅(logging), 시각화, 기록(recording), AI 추론 또는 시스템 서비스는 변동성을 증가시킬 수 있다. 격리된 시험과 실제 부하 시험은 모두 가치가 있으며, 전자는 통신 메커니즘 자체를 평가하고 후자는 실제 로봇 운용 중 성능 유지 가능성을 평가한다.

프로세스 내부 통신(intra-process communication)과 프로세스 간 통신(inter-process communication)은 데이터 경로가 크게 다르므로 별도로 벤치마킹해야 한다. 동일한 프로세스 내부의 컴포넌트는 ROS 2 프로세스 내부 최적화와 복사 감소의 이점을 얻을 수 있지만 별도의 프로세스에서는 추가적인 미들웨어 작업이 필요하다. 컴퓨터 간 통신에는 네트워크 스택(network stack)과 물리적 링크가 추가된다. 이러한 설정을 독립적으로 보고하면 어떤 아키텍처 경계에서 지연시간이 추가되는지 파악할 수 있다.

대용량 메시지 벤치마크(large-message benchmark)에서는 경과 시간뿐 아니라 다른 자원 지표도 모니터링해야 한다. CPU 사용률, 메모리 대역폭, 네트워크 처리량(network throughput), 메모리 할당 동작, 손실된 샘플, 큐 깊이(queue depth), 구독자 처리율을 측정하면 페이로드 크기가 증가할 때 지연시간이 변하는 이유를 설명할 수 있다. 갑작스러운 증가는 단편화, 직렬화 오버헤드, 네트워크 포화(network saturation), 또는 소비자가 처리 속도를 따라가지 못하는 상황을 나타낼 수 있다. 보조 자원 지표를 함께 측정하면 지연시간 결과를 단순한 현상 설명이 아니라 진단에 활용할 수 있다.

단순한 타임스탬프만으로 지연 원인을 파악할 수 없다면 추적(tracing)을 통해 더 깊은 가시성을 확보할 수 있다. ROS 2 추적과 운영체제 계측(instrumentation)은 콜백 실행, 실행기 스케줄링, 발행 이벤트 및 기타 관련 전환 과정을 확인할 수 있다. 미들웨어 및 네트워크 관측 정보와 결합하면 종단 간 지연시간을 여러 소프트웨어 단계로 분해할 수 있다. 이는 응용 프로그램 타임스탬프만으로 원인을 재현하기 어려운 간헐적인 긴 꼬리 지연(long-tail delay)을 분석할 때 특히 유용하다.

네트워크 벤치마크에서는 토폴로지(topology)와 트래픽 조건을 설명해야 한다. 직접 연결 이더넷, 스위치 기반 이더넷(switched Ethernet), Wi-Fi, VPN, 라우팅 네트워크(routed network)는 서로 다른 지연 및 지터 특성을 가진다. 링크 속도만으로 네트워크 환경 전체를 설명할 수 없다. 경쟁 트래픽, 스위치 동작, 무선 신호 품질, 패킷 손실, 멀티캐스트 검색 트래픽(multicast discovery traffic)이 ROS 2 성능에 영향을 줄 수 있다. 따라서 생산 환경 검증에서는 실제 배포에 사용할 네트워크 아키텍처를 재현해야 한다.

비교 시험에서는 보안 설정(security configuration) 역시 동일하게 유지해야 한다. SROS2 인증, 암호화(encryption), 무결성 보호(integrity protection)는 처리 및 통신 오버헤드를 추가할 수 있다. 보안이 비활성화된 개발 시스템을 측정한 후 실제 생산 환경에서 보안을 활성화해도 동일한 지연시간이 유지된다고 가정하면 잘못된 결론을 내릴 수 있다. 실제 운용에 보안이 필요하다면 최종 지연시간 벤치마크에도 배포 로봇과 동일한 보안 설정, 인증서, 미들웨어 설정 및 네트워크 경계를 적용해야 한다.

다중 로봇 벤치마킹(multi-robot benchmarking)에서는 플릿 규모가 증가함에 따라 검색과 트래픽도 증가하므로 또 다른 평가 차원이 추가된다. 단일 로봇에서는 우수한 통신 성능을 보이더라도 동일한 네트워크에서 수십 개의 참여자가 동작하면 검색 활동과 대역폭 경쟁이 증가할 수 있다. 로봇 또는 참여자 수를 단계적으로 증가시키면서 지연시간, 지터, 패킷 손실, CPU 사용률, 검색 동작을 측정하면 실제 시스템의 확장 한계(scaling limit)를 파악할 수 있다.

가능한 경우 벤치마크 실험은 자동화(automation)해야 한다. 스크립트를 사용하여 설정을 구성하고 발행자와 구독자를 실행하며 알려진 메시지 크기와 발행 주기를 적용하고 측정값과 환경 메타데이터(environment metadata)를 저장할 수 있다. 반복 시험을 수행하면 한 번의 유리하거나 불리한 실행 결과가 전체 결론을 결정하는 위험을 줄일 수 있다. 자동화된 시험은 ROS 2, 미들웨어, 커널 또는 응용 소프트웨어가 변경될 때 통신 성능을 회귀 시험(regression testing)의 일부로 포함할 수도 있다.

결과는 엔지니어링 의사결정을 지원할 수 있을 정도로 충분한 컨텍스트(context)와 함께 보고해야 한다. 유용한 보고서는 하드웨어 및 소프트웨어 설정, 토폴로지, 페이로드, 발행 주기, QoS, 샘플 수, 클록 방식, 시스템 부하 및 통계 지표를 명시한다. 지연시간 분포 또는 페이로드와 발행 주기에 따른 지연시간 그래프는 하나의 숫자로 표현할 수 없는 시스템 동작을 보여줄 수 있다. 가능하다면 원시 측정 데이터(raw measurement data)도 보존하여 이후 다른 통계 분석을 수행할 수 있도록 해야 한다.

자율 로봇(autonomous robot)과 피지컬 AI(Physical AI) 시스템에서 가장 가치 있는 벤치마크는 실제 정보 경로와 운용 작업 부하를 대표하는 시험이다. 비어 있는 데모 환경에서 통신을 측정하기보다는 실제와 유사한 센서 주기, 인지 부하(perception load), 보안, 네트워크 조건, 다수의 활성 컴포넌트를 적용하여 측정해야 한다. 체계적인 방법론을 사용하면 ROS 2 지연시간을 단순한 하나의 벤치마크 수치가 아니라 전체 로봇 아키텍처가 요구되는 시간 제약(timing requirement)을 만족할 수 있음을 보여주는 근거로 전환할 수 있다.
