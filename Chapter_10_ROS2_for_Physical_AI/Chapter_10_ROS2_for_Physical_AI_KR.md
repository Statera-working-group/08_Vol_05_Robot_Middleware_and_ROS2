**Volume 05 Robot Middleware and ROS2**

# 10. ROS2 for Physical AI

## 10.01 Physical AI Inference and ROS2 Integration Architecture

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

피지컬 AI(Physical AI)는 다중 모달 관측(Multimodal Observation)을 해석하고, 물리적 환경(Physical Environment)을 추론하며, 실제 세계에 영향을 미치는 행동(Action)을 생성하는 학습 모델(Learned Model)을 도입함으로써 기존 로봇 소프트웨어(Conventional Robotic Software)를 확장한다. ROS2 구조에서 이러한 지능(Intelligence)은 미들웨어(Middleware)나 결정론적 제어 계층(Deterministic Control Layer)을 대체해서는 안 된다. 대신 AI 추론(AI Inference)은 명확한 ROS2 인터페이스(Interface)를 통해 센싱(Sensing), 상태 추정(State Estimation), 계획(Planning), 모션 생성(Motion Generation), 제어(Control)에 연결되는 모듈형 연산 계층(Modular Computational Layer)이 된다.

실용적인 아키텍처(Architecture)는 시스템을 센싱(Sensing), 데이터 전처리(Data Preparation), AI 추론(AI Inference), 의사결정 통합(Decision Integration), 물리적 실행(Physical Execution)으로 분리한다. 카메라(Camera), 라이다(LiDAR), 레이더(Radar), 관성측정장치(IMU), 힘 센서(Force Sensor), 관절 인코더(Joint Encoder) 등의 장치는 ROS2 토픽(Topic)을 통해 관측 데이터를 발행한다. 전처리 노드(Preprocessing Node)는 타임스탬프(Timestamp)를 동기화하고 좌표계(Coordinate Frame)를 변환하며, 측정값을 정규화하고 텐서(Tensor)를 생성하여 AI 모델에 필요한 다중 모달 문맥(Multimodal Context)을 구성한다.

추론 계층(Inference Layer)은 하나의 거대한 AI 노드(Monolithic AI Node)가 아니라 여러 전문화된 ROS2 컴포넌트(Component)의 집합으로 구성하는 것이 바람직하다. 개별 추론 노드는 객체 탐지(Object Detection), 의미 이해(Semantic Understanding), 3차원 인식(3D Perception), 지형 분석(Terrain Analysis), 정책 추론(Policy Inference), 궤적 예측(Trajectory Prediction), 언어 조건 행동 생성(Language-Conditioned Action Generation)을 수행할 수 있다. 이러한 분해는 ROS2 노드 아키텍처(Node Architecture)를 따르면서 각 모델이 서로 다른 가속기(Accelerator), 실행 주기(Execution Frequency), 수명주기 정책(Lifecycle Policy), 연산 자원(Computational Resource)을 사용할 수 있게 한다.

ROS2가 제공하는 여러 통신 패턴(Communication Pattern)은 피지컬 AI(Physical AI) 워크로드(Workload)에 자연스럽게 대응한다. 토픽(Topic)은 이미지(Image), 포인트 클라우드(Point Cloud), 임베딩(Embedding), 탐지 결과(Detection), 예측 상태(Predicted State)와 같은 연속 데이터 스트림에 적합하다. 서비스(Service)는 지속적인 스트리밍이 필요하지 않은 요청-응답형(Request-Response) 추론에 사용할 수 있으며, 액션(Action)은 피드백(Feedback), 취소(Cancellation), 완료 상태(Completion Status)가 필요한 장시간 AI 기반 작업에 적합하다. 따라서 인터페이스는 모델 구현의 편의성이 아니라 추론의 의미론(Inference Semantics)에 따라 선택해야 한다.

대규모 AI 페이로드(Large AI Payload)는 특별한 아키텍처적 고려가 필요하다. 원시 이미지(Raw Image), 포인트 클라우드(Point Cloud), 텐서 표현(Tensor Representation), 고차원 임베딩(High-Dimensional Embedding)은 상당한 직렬화(Serialization), 메모리 복사(Memory Copy), 네트워크 부하(Network Overhead)를 발생시킬 수 있다. 피지컬 AI 시스템은 ROS2 메시지(Message), CPU 메모리, 고정 메모리(Pinned Memory), 가속기 메모리(Accelerator Memory) 사이의 불필요한 변환을 최소화해야 한다. 프로세스 내부 통신(Intra-Process Communication), 컴포지션(Composition), 공유 메모리(Shared Memory), 사전 할당 버퍼(Preallocated Buffer), 가속기 인식 파이프라인(Accelerator-Aware Pipeline)을 활용하면 데이터 이동을 줄일 수 있다.

다중 모달 추론(Multimodal Inference)은 여러 관측값이 거의 동일한 물리적 상태(Physical State)를 나타내야 하므로 시간 일관성(Time Consistency) 역시 중요하다. 센서 타임스탬프(Sensor Timestamp)는 전처리와 추론 과정에서 그 출처(Provenance)를 유지해야 하며, AI 노드가 처리를 완료한 시간으로 대체되어서는 안 된다. ROS2 시간 동기화(Time Synchronization), tf2 좌표 변환(Transform), 하드웨어 타임스탬프(Hardware Timestamp), 동기화 센서 획득(Synchronized Sensor Acquisition)을 사용하면 추론 결과를 실제 관측 시점과 연결할 수 있으며, 이는 움직이는 로봇에서 특히 중요하다.

AI 추론(AI Inference)과 실시간 제어(Real-Time Control)는 아키텍처적으로 분리되어야 한다. 신경망 모델(Neural Model)은 의미 상태(Semantic State), 목표 자세(Target Pose), 내비게이션 목표(Navigation Objective), 정책 출력(Policy Output), 모션 기준값(Motion Reference)을 생성할 수 있지만, 안전 필수 액추에이터 루프(Safety-Critical Actuator Loop)는 일반적인 추론 파이프라인보다 높은 시간 결정성(Timing Determinism)을 요구한다. ROS2는 AI가 행동을 제안하고 결정론적 제어 소프트웨어(Deterministic Control Software)가 이를 검증, 제한, 보간 또는 거부한 뒤 모터와 액추에이터에 전달하는 제한된 인터페이스(Bounded Interface)를 통해 두 영역을 연결할 수 있다.

이러한 분리는 GPU 스케줄링(GPU Scheduling), 모델 복잡도(Model Complexity), 열 스로틀링(Thermal Throttling), 메모리 압박(Memory Pressure), 동시 워크로드(Concurrent Workload)로 인해 추론 지연시간(Inference Latency)이 변할 때 특히 중요하다. 모든 AI 출력에는 관측 타임스탬프(Observation Timestamp), 추론 타임스탬프(Inference Timestamp), 모델 식별자(Model Identity), 신뢰도 정보(Confidence Information), 필요하면 시퀀스 식별자(Sequence Identifier)를 포함해야 한다. 이를 통해 하위 노드는 오래된 결과(Stale Result)를 탐지하고 지연된 예측을 그대로 실행하는 대신 타임아웃(Timeout), 폴백(Fallback), 유지(Hold), 정지(Stop), 기존 제어(Conventional Control)를 적용할 수 있다.

수명주기 관리(Lifecycle Management)는 ROS2와 피지컬 AI를 연결하는 또 하나의 중요한 요소다. 추론 노드는 실제 동작 전에 모델 가중치(Model Weight)를 로딩하고, 가속기 메모리(Accelerator Memory)를 할당하며, TensorRT 등의 실행 런타임(Runtime)을 초기화하고, 장치 가용성(Device Availability)을 확인하며, 워밍업 추론(Warm-Up Inference)을 수행해야 할 수 있다. ROS2 수명주기 노드(Lifecycle Node)는 설정(Configuration)과 활성화(Activation)를 체계적으로 분리할 수 있게 하며, 모델 초기화 실패 시 해당 추론 컴포넌트가 활성 시스템 상태(Active System State)로 진입하는 것을 방지할 수 있다.

가능한 경우 모델 배포(Model Deployment)는 논리적인 ROS2 인터페이스(Logical ROS2 Interface)와 독립되어야 한다. 인식 노드(Perception Node)는 내부적으로 PyTorch, ONNX Runtime, TensorRT, CUDA 또는 다른 가속기 런타임(Accelerator Runtime)을 사용하면서도 동일하고 안정적인 ROS2 메시지 계약(Message Contract)을 유지할 수 있다. 이러한 추상화(Abstraction)를 통해 내비게이션, 조작(Manipulation), 플릿(Fleet), 제어 소프트웨어를 변경하지 않고 모델 최적화(Model Optimization)와 하드웨어 마이그레이션(Hardware Migration)을 수행할 수 있다.

피지컬 AI는 하나의 로봇 안에서도 서로 다른 여러 추론 주기(Inference Rate)를 결합한다. 카메라 인식(Camera Perception)은 초당 수십 프레임으로 실행될 수 있고, 대형 다중 모달 모델(Large Multimodal Model)은 더 낮은 주기로, 월드 모델(World Model)은 또 다른 주기로, 저수준 제어(Low-Level Control)는 초당 수백 또는 수천 회의 주기로 실행될 수 있다. ROS2 노드는 이러한 워크로드를 하나의 시간 영역(Timing Domain)에 강제로 통합해서는 안 된다. 대신 비동기 통신(Asynchronous Communication), 타임스탬프 기반 상태 캐시(State Cache), QoS 설정, 명시적인 데이터 최신성 정책(Freshness Policy)을 통해 서로 다른 실행 주기의 정보를 조정해야 한다.

서비스 품질(QoS) 설계는 이러한 환경에서 시스템 수준의 의사결정이 된다. 고주기 센서 스트림(High-Rate Sensor Stream)은 오래된 데이터가 누적되는 것을 방지하기 위해 최선형 전달(Best-Effort Delivery)과 얕은 히스토리(Shallow History)를 사용할 수 있으며, 중요한 상태 전환(State Transition)이나 작은 크기의 추론 결과는 신뢰성 있는 전달(Reliable Delivery)이 필요할 수 있다. 데드라인(Deadline), 수명(Lifespan), 히스토리(History), 지속성(Durability), 신뢰성(Reliability)은 각 데이터 흐름의 물리적 의미에 맞게 설정해야 한다. 목표는 모든 통신에서 최대 신뢰성을 확보하는 것이 아니라 로봇 정보의 시간적 가치(Temporal Value)에 적합한 전달 동작을 구현하는 것이다.

피지컬 AI 통합에는 관측 가능성(Observability)도 포함되어야 한다. 추론 지연시간(Inference Latency), 전처리 시간(Preprocessing Time), 큐 지연(Queue Delay), GPU 사용률(GPU Utilization), 메모리 사용량(Memory Consumption), 입력 및 출력 주기(Input/Output Frequency), 누락 샘플(Dropped Sample), 신뢰도 분포(Confidence Distribution), 타임아웃 이벤트(Timeout Event)를 기존 ROS2 진단(Diagnostics)과 함께 측정할 수 있어야 한다. 이를 통해 엔지니어는 AI 모델을 불투명한 컴포넌트(Opaque Component)로 취급하지 않고 센싱, 미들웨어 통신, 추론 실행, 의사결정 로직, 제어 중 어디에서 성능 저하가 발생하는지 구분할 수 있다.

안전 감독(Safety Supervision)은 학습된 추론 모델(Learned Inference Model)의 권한 밖에 유지되어야 한다. AI가 생성한 모션 요청(Motion Request)은 실행 전에 속도 제한(Velocity Limit), 충돌 제약(Collision Constraint), 운용 영역(Operational Zone), 액추에이터 제한(Actuator Limit), 로봇 상태(Robot State), 독립 안전 신호(Independent Safety Signal)를 기준으로 검증할 수 있다. 추론이 불가능하거나 불확실해지는 경우 로봇은 사전에 정의된 성능 저하 동작(Degradation Behavior)에 따라 전환되어야 한다. 이러한 구조는 확률적 추론(Probabilistic Reasoning)과 물리적 구동(Physical Actuation) 사이에 통제된 경계를 유지한다.

결과적으로 ROS2 아키텍처는 기존 로보틱스(Conventional Robotics)와 학습 기반 지능(Learned Intelligence)을 연결하는 통합 기반(Integration Fabric)을 형성한다. 센서는 타임스탬프가 포함된 물리적 관측(Physical Observation)을 생성하고, 전처리는 이를 모델 입력 표현(Model-Ready Representation)으로 변환하며, 추론 노드는 의미 정보(Semantic Information) 또는 행동 수준 정보(Action-Level Information)를 생성한다. 이후 의사결정 및 모션 노드가 이를 해석하고, 결정론적 제어(Deterministic Control)가 검증된 명령을 실행한다. 진단, 수명주기 관리, QoS, 보안(Security), 로깅(Logging), 배포(Deployment)는 AI 기능 구현 이후 추가되는 요소가 아니라 전체 경로를 둘러싸는 공통 시스템 기능으로 구성되어야 한다.

이러한 위치 설정은 ROS2 미들웨어가 제어 소프트웨어(Control Software), 엣지 컴퓨팅(Edge Computing), 인식(Perception), 피지컬 AI 파운데이션 모델(Physical AI Foundation Model), 로봇 학습(Robot Learning), 비전-언어-행동 시스템(Vision-Language-Action System), 데이터 아키텍처(Data Architecture), 플릿 지능(Fleet Intelligence)을 서로 독립적이면서 상호운용 가능한 영역으로 연결하는 전체 소프트웨어 구조와 일치한다. 따라서 피지컬 AI 추론은 더 큰 소프트웨어 정의 로봇 아키텍처(Software-Defined Robot Architecture)의 하나의 참여 계층으로 자리 잡으며, 인식, 추론, 미들웨어, 안전, 제어를 하나의 분리 불가능한 스택으로 결합하지 않고도 지능 자체를 지속적으로 발전시킬 수 있게 한다.

## 10.02 ROS2 Message Type Extension: Tensor / Embedding [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

피지컬 AI(Physical AI) 시스템에서 기존 ROS2 메시지 정의(Message Definition)는 텐서(Tensor), 임베딩(Embedding), 특징 맵(Feature Map), 잠재 상태(Latent State), 모델 출력(Model Output)과 같은 신경망 데이터(Neural-Network Data)를 표현하기에 충분하지 않은 경우가 많다. 이러한 표현을 위해 ROS2 인터페이스 계층(Interface Layer)을 확장하면 AI 컴포넌트(AI Component)가 센서, 플래너(Planner), 제어기(Controller), 진단(Diagnostics)과 동일한 분산 통신 아키텍처(Distributed Communication Architecture)에 참여하면서 독립적으로 개발된 노드 사이에 명시적인 데이터 계약(Data Contract)을 유지할 수 있다.

텐서 메시지(Tensor Message)는 수치 데이터뿐만 아니라 이를 올바르게 해석하기 위해 필요한 구조도 함께 기술해야 한다. 중요한 메타데이터(Metadata)에는 요소 데이터 형식(Element Data Type), 차원 수(Dimensionality), 형상(Shape), 메모리 레이아웃(Memory Layout), 그리고 필요에 따라 스트라이드 정보(Stride Information)가 포함된다. 이러한 정보가 없는 부동소수점 값의 연속 데이터는 이미지 특징 맵, 행동 벡터(Action Vector), 포인트 특징 텐서(Point Feature Tensor), 언어 임베딩(Language Embedding) 중 무엇을 의미하는지 수신 측에서 판단할 수 없으므로 모호하다.

따라서 텐서 차원(Tensor Dimension)은 노드별 가정(Node-Specific Assumption)에 포함시키지 않고 명시적으로 표현해야 한다. 예를 들어 이미지 텐서(Image Tensor)는 배치(Batch), 채널(Channel), 높이(Height), 너비(Width) 순서를 사용할 수 있지만 다른 모델은 배치, 높이, 너비, 채널 순서를 사용할 수 있다. 형상과 레이아웃 메타데이터(Layout Metadata)를 발행하면 구독자(Subscriber)가 추론 전에 호환성을 검증할 수 있으며, AI 모델이나 전처리 파이프라인(Preprocessing Pipeline)을 교체할 때 발생할 수 있는 암묵적인 해석 오류를 방지할 수 있다.

요소 형식(Element Type) 역시 인터페이스 계약(Interface Contract)의 핵심 요소다. 피지컬 AI 추론 파이프라인(Physical AI Inference Pipeline)은 모델 아키텍처(Model Architecture)와 가속기 최적화(Accelerator Optimization)에 따라 FP32, FP16, BF16, INT8, 부호 있는 정수(Signed Integer) 또는 부호 없는 정수(Unsigned Integer)를 교환할 수 있다. ROS2 메시지 정의는 데이터 표현(Data Representation)을 바이트 페이로드(Byte Payload)와 독립적으로 식별해야 하며, 이를 통해 수신 컴포넌트가 지원하지 않는 형식을 거부하거나 원시 메모리(Raw Memory)를 잘못 해석하는 대신 의도적인 변환을 수행할 수 있다.

임베딩(Embedding)은 이와 관련되지만 의미론적으로는 다른 인터페이스 설계(Interface Design)를 요구한다. 임베딩은 일반적으로 시각적, 언어적, 공간적, 객체 수준 또는 다중 모달 정보(Multimodal Information)를 인코딩하는 학습된 벡터 표현(Learned Vector Representation)이다. 기술적으로는 1차원 텐서로 전송할 수 있지만, 실용적인 ROS2 메시지는 임베딩 차원(Embedding Dimension), 모델 식별자(Model Identifier), 소스 모달리티(Source Modality), 타임스탬프(Timestamp), 필요한 경우 기준 좌표계(Reference Frame), 그리고 해당 임베딩을 생성한 개체(Entity) 또는 관측(Observation)과 같은 의미 메타데이터(Semantic Metadata)를 벡터와 연결해야 한다.

모델 식별 정보(Model Identity)는 동일한 크기의 임베딩이라도 서로 다른 모델에서 생성되었다면 반드시 호환된다고 볼 수 없기 때문에 특히 중요하다. 한 인코더(Encoder)가 생성한 512차원 시각 임베딩(Visual Embedding)을 다른 모델이 생성한 512차원 벡터와 자동으로 비교할 수는 없다. 따라서 메시지 메타데이터는 하위 노드가 유사도 계산(Similarity Calculation), 검색(Retrieval), 융합(Fusion), 정책 추론(Policy Inference)이 의미론적으로 유효한지를 판단할 수 있도록 충분한 모델 및 표현 정보를 제공해야 한다.

타임스탬프 정보(Timestamp Information)는 단순히 ROS2 메시지가 발행된 시간이 아니라 텐서와 연결된 실제 물리적 관측(Physical Observation)의 시점을 나타내야 한다. 카메라 프레임(Camera Frame)으로부터 생성된 특징 텐서(Feature Tensor)는 해당 프레임의 획득 타임스탬프(Acquisition Timestamp)를 유지해야 하며, 파생된 다중 모달 임베딩(Multimodal Embedding)도 원본 관측과의 추적 가능성(Traceability)을 유지해야 한다. 이를 통해 비동기 피지컬 AI 파이프라인에서 동기화(Synchronization), 지연시간 분석(Latency Analysis), 상태 재구성(State Reconstruction), 오래된 데이터 탐지(Stale-Data Detection)가 가능해진다.

학습된 표현(Learned Representation)이 물리적 기하 구조(Physical Geometry)와 연결되는 경우 좌표계 정보(Coordinate-Frame Information)도 중요하다. 포인트 클라우드 특징(Point-Cloud Feature), 점유 텐서(Occupancy Tensor), 복셀 임베딩(Voxel Embedding), 객체 특징(Object Feature), 공간 잠재 표현(Spatial Latent Representation)은 센서, 로봇, 오도메트리(Odometry), 맵(Map) 좌표계와 연결될 수 있다. 표준 ROS2 헤더 규칙(Header Convention)과 tf2 호환 프레임 식별자(Frame Identifier)를 통합하면 학습된 표현과 로봇의 의사결정이 실행되는 물리적 좌표계 사이의 관계를 유지할 수 있다.

일반적인 텐서 메시지(General Tensor Message)는 간결한 메타데이터 영역(Metadata Section)과 그 뒤에 이어지는 연속 페이로드(Contiguous Payload)를 중심으로 설계할 수 있다. 그러나 일반적인 직렬화 ROS2 메시지를 통해 대규모 수치 배열(Numerical Array)을 전송하면 메모리 복사(Memory Copy)와 직렬화 오버헤드(Serialization Overhead)가 발생할 수 있다. 이는 고해상도 특징 맵(High-Resolution Feature Map)이나 고주기 인식 파이프라인(High-Frequency Perception Pipeline)에서 특히 중요하다. 따라서 메시지 형식 설계는 프로세스 내부 통신(Intra-Process Communication), 공유 메모리 전송(Shared-Memory Transport), 노드 컴포지션(Node Composition), 가속기 중심 데이터 이동(Accelerator-Oriented Data Movement)과 함께 고려해야 한다.

동일한 프로세스에서 실행되는 노드의 경우 ROS2 컴포지션(Composition)을 통해 불필요한 통신 오버헤드를 줄일 수 있지만, 애플리케이션 아키텍처(Application Architecture)는 모든 AI 컴포넌트가 항상 동일한 위치에 배치될 것이라고 가정해서는 안 된다. 잘 정의된 텐서 인터페이스(Tensor Interface)는 배포 구조(Deployment Structure)가 하나의 프로세스에서 여러 프로세스 또는 여러 컴퓨터로 변경되더라도 논리적 상호운용성(Logical Interoperability)을 유지한다. 따라서 가능한 경우 전송 최적화(Transport Optimization)는 텐서의 의미론적 정의(Semantic Definition)와 분리되어야 한다.

GPU 간 직접 통신(Direct GPU-to-GPU Communication)은 일반적인 ROS2 바이트 배열(Byte Array)이 가속기 메모리가 아닌 호스트 접근 가능 데이터(Host-Accessible Data)를 나타내기 때문에 더 복잡한 문제를 제기한다. 고성능 구현(High-Performance Implementation)은 핸들(Handle), 공유 버퍼(Shared Buffer), 대여 데이터(Loaned Data), 미들웨어별 제로 카피(Middleware-Specific Zero-Copy) 메커니즘을 도입할 수 있다. 이러한 최적화는 시스템이 이식 가능한 ROS2 데이터 계약(Portable ROS2 Data Contract)과 배포 환경에 종속된 가속기 전송 메커니즘(Accelerator Transport Mechanism)을 구분할 수 있도록 통제된 인터페이스 경계(Controlled Interface Boundary) 뒤에 유지하는 것이 바람직하다.

텐서 메시지는 구독자 경계(Subscriber Boundary)에서의 검증(Validation)도 필요하다. 노드는 수신 데이터를 사용하기 전에 랭크(Rank), 형상(Shape), 요소 형식(Element Type), 예상 모델 식별 정보(Expected Model Identity), 페이로드 크기(Payload Size), 타임스탬프 최신성(Timestamp Freshness), 표현 버전(Representation Version)을 검증할 수 있다. 이러한 검사는 차원이나 전처리 규칙(Preprocessing Convention)이 수신 모델과 일치하지 않을 경우 외형상 정상적인 수치 배열이 그럴듯하지만 잘못된 신경망 결과를 생성할 수 있는 피지컬 AI에서 특히 중요하다.

텐서 및 임베딩 인터페이스가 발전함에 따라 버전 관리(Version Management)도 필요하다. 양자화 파라미터(Quantization Parameter), 모델 출처(Model Provenance), 정규화 메타데이터(Normalization Metadata), 의미 레이블(Semantic Label), 새로운 요소 형식(Element Type)을 추가하면 배포된 로봇 사이의 호환성에 영향을 줄 수 있다. 안정적인 메시지 정의와 명시적인 표현 버전(Representation Version)은 모델 업그레이드(Model Upgrade)가 하위 컴포넌트를 암묵적으로 손상시키는 것을 방지한다. 따라서 기본적인 의미 계약이 유지되는 경우 인터페이스 진화(Interface Evolution)는 개별 모델 릴리스(Model Release)와 독립적으로 관리해야 한다.

양자화 추론(Quantized Inference)은 추가적인 메타데이터 요구사항을 가진다. INT8 또는 기타 압축 표현(Compressed Representation)은 의도된 수치적 의미를 복원하기 위해 스케일 팩터(Scale Factor), 제로 포인트(Zero Point), 양자화 축(Quantization Axis), 캘리브레이션 규칙(Calibration Convention)이 필요할 수 있다. 이러한 파라미터가 누락되면 원시 텐서 값(Raw Tensor Value)을 원래 런타임 외부에서 신뢰성 있게 해석할 수 없다. 재사용 가능한 ROS2 텐서 표현은 필요한 양자화 정보를 포함하거나 참조되는 모델 계약(Referenced Model Contract)을 통해 이를 명시적으로 정의해야 한다.

임베딩 통신(Embedding Communication)은 직접적인 인식 기능을 넘어 더 높은 수준의 피지컬 AI 기능도 지원한다. 시각 및 언어 임베딩(Visual and Language Embedding)은 의미 검색(Semantic Retrieval), 객체 메모리(Object Memory), 장면 연관(Scene Association), 다중 모달 그라운딩(Multimodal Grounding), 작업 계획(Task Planning), 로봇 지식 공유(Robot Knowledge Sharing)에 활용될 수 있다. ROS2는 모든 하위 시스템이 원시 센서 스트림(Raw Sensor Stream)을 직접 소비하지 않아도 이러한 표현이 인식, 메모리, 추론, VLA, 플릿 수준 컴포넌트(Fleet-Level Component) 사이를 이동할 수 있는 통신 기반(Communication Fabric)을 제공한다.

여러 로봇 사이에서 임베딩을 분산할 때에는 더욱 주의해야 한다. 임베딩은 크기가 작기 때문에 공유 의미 표현(Shared Semantic Representation)으로 활용하기에 매력적이지만, 호환성은 공통 인코더 버전(Common Encoder Version), 전처리 규칙, 정규화 규칙(Normalization Convention), 의미 정의(Semantic Definition)에 의존한다. 따라서 플릿 아키텍처(Fleet Architecture)는 단순히 벡터 차원이 동일하다는 이유만으로 상호운용성을 가정해서는 안 되며, 임베딩 스키마(Embedding Schema)와 모델 출처를 분산 인터페이스 계약(Distributed Interface Contract)의 일부로 관리해야 한다.

텐서 및 임베딩 토픽(Topic)에는 적절한 ROS2 서비스 품질(QoS) 설정도 필요하다. 고주기 중간 텐서(High-Frequency Intermediate Tensor)는 새로운 데이터가 도착하면 이전 데이터의 가치가 빠르게 감소하므로 얕은 큐(Shallow Queue)와 최신성 중심 전달(Freshness-Oriented Delivery)을 사용할 수 있다. 반면 작은 의미 임베딩(Semantic Embedding)이나 중요한 추론 상태(Inference State)는 다른 신뢰성 특성(Reliability Characteristic)이 필요할 수 있다. QoS는 모든 AI 메시지에 동일한 설정을 적용하기보다 학습된 표현의 유효 수명(Lifetime)과 운용상 중요도(Operational Significance)를 반영해야 한다.

이러한 확장 메시지 형식(Extended Message Type)에는 관측 가능성(Observability)도 함께 고려되어야 한다. 진단 시스템(Diagnostics)은 텐서 차원, 페이로드 크기, 발행 주기(Publication Rate), 직렬화 시간(Serialization Time), 전송 지연시간(Transport Latency), 누락 샘플(Dropped Sample), 비호환 스키마 이벤트(Incompatible Schema Event), 오래된 임베딩(Stale Embedding)을 추적할 수 있다. 이러한 지표를 추론 지연시간(Inference Latency) 측정과 결합하면 피지컬 AI 성능 저하가 모델 실행 자체에서 발생하는지 또는 모델을 둘러싼 통신 및 표현 파이프라인(Communication and Representation Pipeline)에서 발생하는지 판단할 수 있다.

궁극적으로 ROS2 텐서 및 임베딩 확장(Tensor and Embedding Extension)은 신경망 연산(Neural Computation)과 분산 로봇 소프트웨어(Distributed Robotic Software) 사이에 형식화된 경계(Typed Boundary)를 구축한다. 목표는 단순히 숫자 배열을 전송하는 것이 아니라 학습된 표현이 로봇 시스템을 이동하는 동안 형상(Shape), 형식(Type), 시간(Time), 출처(Provenance), 의미(Semantics), 호환성(Compatibility), 물리적 문맥(Physical Context)을 유지하는 것이다. 이를 통해 피지컬 AI 모델은 모듈형 컴포넌트(Modular Component)로 지속적으로 발전하면서 인식, 추론(Reasoning), 계획, 제어, 데이터 로깅(Data Logging), 다중 로봇 시스템(Multi-Robot System)과 예측 가능한 방식으로 통합될 수 있다.

## 10.03 Camera AI Inference Node Pipeline Design [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS2의 카메라 AI 추론 노드(Camera AI Inference Node)는 연속적인 시각 센싱(Visual Sensing)과 상위 수준 로봇 지능(Higher-Level Robotic Intelligence)을 연결하는 역할을 한다. 주요 책임은 단순히 신경망(Neural Network)을 실행하는 것이 아니라 카메라 획득(Camera Acquisition), 전처리(Preprocessing), 추론(Inference), 후처리(Postprocessing), 구조화된 결과 발행(Publication)에 이르는 전체 경로를 관리하는 것이다. 이 경로를 명시적인 파이프라인(Pipeline)으로 설계하면 시각 지능(Visual Intelligence)을 모듈화하고 관측 가능하며 교체 가능한 형태로 구성하여 전체 피지컬 AI(Physical AI) 소프트웨어 아키텍처와 통합할 수 있다.

파이프라인은 일반적으로 이미지 데이터(Image Data)와 카메라 보정 정보(Camera Calibration Information)를 발행하는 ROS2 카메라 드라이버(Camera Driver)에서 시작한다. 하드웨어에 따라 입력 장치는 RGB 카메라, 스테레오 카메라(Stereo Camera), 깊이 카메라(Depth Camera), 열화상 카메라(Thermal Camera), 동기화된 다중 카메라 시스템(Synchronized Multi-Camera System)이 될 수 있다. 추론 노드는 가능한 경우 표준 ROS2 이미지 인터페이스(Image Interface)를 사용하고 타임스탬프(Timestamp)와 프레임 식별자(Frame Identifier)를 유지하여 시각 추론 결과를 실제 물리적 관측과 추적 가능하도록 해야 한다.

이미지 획득(Image Acquisition)과 AI 추론(AI Inference)은 서로 다른 주기로 동작하는 경우가 많다. 카메라는 초당 30프레임 또는 60프레임을 생성할 수 있지만 대규모 신경망 모델은 훨씬 적은 프레임만 처리할 수 있다. 따라서 노드는 오래된 이미지가 제한 없이 콜백 큐(Callback Queue)에 누적되지 않도록 해야 한다. 큐 깊이(Queue Depth), 콜백 스케줄링(Callback Scheduling), 프레임 드롭(Frame Dropping), 최신성 정책(Freshness Policy)을 명시적으로 설계하여 모델이 점점 오래된 프레임이 아니라 운용상 가장 의미 있는 최신 관측을 처리하도록 해야 한다.

전처리(Preprocessing)는 ROS2 이미지 데이터를 모델이 요구하는 표현으로 변환한다. 일반적인 처리에는 색 공간 변환(Color-Space Conversion), 크기 조정(Resizing), 자르기(Cropping), 정규화(Normalization), 텐서 변환(Tensor Conversion), 채널 재배열(Channel Reordering), 배치 구성(Batch Construction)이 포함된다. 이러한 연산은 모델의 학습 및 배포 조건과 정확하게 일치해야 한다. RGB/BGR 순서, 정규화 상수(Normalization Constant), 이미지 해상도(Image Resolution), 종횡비 처리(Aspect-Ratio Handling), 텐서 레이아웃(Tensor Layout)의 차이는 신경망 자체가 정상적으로 실행되더라도 추론 성능을 크게 저하시킬 수 있다.

기하학적 전처리(Geometric Preprocessing)는 추론 결과를 원본 카메라 이미지로 다시 매핑해야 하는 경우가 많기 때문에 특별한 주의가 필요하다. 추론 전에 이미지의 크기를 변경하거나 자르고 패딩(Padding) 또는 레터박싱(Letterboxing)을 적용했다면 해당 변환 파라미터(Transformation Parameter)를 후처리 단계에서도 유지해야 한다. 이를 통해 바운딩 박스(Bounding Box), 분할 마스크(Segmentation Mask), 키포인트(Keypoint), 깊이 추정(Depth Estimation) 등의 예측 결과를 원본 센서 이미지 좌표계로 정확하게 변환하고 이후 로봇의 물리적 기하 구조와 연결할 수 있다.

추론 단계(Inference Stage)는 가능한 경우 모델 런타임 추상화(Model-Runtime Abstraction) 뒤에 분리되어야 한다. 동일한 ROS2 노드 아키텍처는 외부 ROS2 인터페이스를 변경하지 않고 PyTorch, ONNX Runtime, TensorRT, CUDA 기반 라이브러리 또는 하드웨어 전용 가속기(Hardware-Specific Accelerator)를 이용해 모델을 실행할 수 있다. 모델 실행(Model Execution)과 통신(Communication)을 분리하면 배포 플랫폼과 최적화 방법이 변화하더라도 하위 인식, 계획, 제어 노드는 안정적인 메시지 계약(Message Contract)을 계속 사용할 수 있다.

GPU 실행(GPU Execution)은 추가적인 파이프라인 설계 요소를 요구한다. 호스트-장치 전송(Host-to-Device Transfer), 텐서 할당(Tensor Allocation), 커널 실행(Kernel Execution), 동기화(Synchronization), 장치-호스트 전송(Device-to-Host Transfer)은 신경망 연산 자체 외에도 상당한 시간을 소비할 수 있다. 재사용 버퍼(Reusable Buffer), 고정 메모리(Pinned Memory), 비동기 실행(Asynchronous Execution), 사전 할당 텐서(Preallocated Tensor), 최적화된 CUDA 스트림(CUDA Stream), 가속기 전용 런타임을 사용하면 이러한 오버헤드를 줄일 수 있다. 따라서 성능 분석에서는 모델 커널 실행 시간만이 아니라 추론 노드의 종단간 지연시간(End-to-End Latency)을 측정해야 한다.

후처리(Postprocessing)는 신경망의 원시 출력(Raw Output)을 로봇이 의미 있게 사용할 수 있는 정보로 변환한다. 객체 탐지 네트워크(Detection Network)는 신뢰도 필터링(Confidence Filtering)과 비최대 억제(Non-Maximum Suppression)가 필요할 수 있고, 분할 모델(Segmentation Model)은 마스크 복원(Mask Reconstruction)이 필요하며, 자세 추정 모델(Pose Model)은 키포인트 또는 기하학적 추정값을 생성할 수 있다. 최종 ROS2 메시지는 문서화되지 않은 모델별 배열을 그대로 노출하기보다 객체 클래스(Object Class), 신뢰도(Confidence), 이미지 좌표(Image Coordinate), 물리적 위치(Physical Position), 분할 영역(Segmentation Region), 추적 식별자(Tracking Identity), 모델 출처(Model Provenance)와 같은 운용 의미를 표현해야 한다.

시각 추론 결과(Visual Inference Result)는 시간적 문맥(Temporal Context)과 공간적 문맥(Spatial Context)을 모두 유지해야 한다. 출력에는 원본 관측 타임스탬프(Observation Timestamp)와 적절한 프레임 식별자를 보존하고, 지연시간 측정이 필요한 경우 추론 완료 시간(Inference Completion Time)도 함께 기록해야 한다. 이를 통해 하위 노드는 물리적 장면이 언제 관측되었으며 처리에 얼마나 많은 시간이 소요되었는지 판단할 수 있다. 기하학적 정보가 존재하는 경우 tf2를 이용해 카메라 프레임(Camera Frame)의 탐지 결과를 로봇, 오도메트리(Odometry), 맵(Map) 좌표계와 연결할 수 있다.

카메라 드라이버, 전처리 단계, 추론 컴포넌트가 동일한 컴퓨터에서 실행되는 경우 ROS2 컴포지션(Composition)을 통해 성능을 향상시킬 수 있다. 프로세스 내부 통신(Intra-Process Communication)은 특히 대용량 이미지 메시지에서 직렬화(Serialization)와 메모리 복사(Memory Copy) 오버헤드를 줄일 수 있다. 그러나 시스템이 이후 여러 프로세스 또는 여러 컴퓨터에 걸쳐 컴포넌트를 분리하더라도 전체 인식 아키텍처를 다시 설계하지 않도록 각 단계 사이의 논리적 인터페이스(Logical Interface)는 명확하게 정의되어야 한다.

서비스 품질(QoS) 설정은 카메라 추론의 시간적 특성을 반영해야 한다. 고주기 이미지(High-Rate Image)는 움직이는 로봇에서 오래된 프레임의 가치가 빠르게 감소하므로 제한된 히스토리(Limited History)가 일반적으로 유리하다. 일부 센서 스트림에는 최선형 통신(Best-Effort Communication)이 적합할 수 있으며, 크기가 작은 추론 출력은 운용상 중요도에 따라 신뢰성 있는 전달(Reliable Delivery)을 사용할 수 있다. QoS는 하나의 범용 설정을 사용하는 대신 센서 특성, 네트워크 조건, 처리 주기, 하위 시스템 요구사항에 따라 결정해야 한다.

동시성(Concurrency) 역시 의도적으로 제어해야 한다. 이미지 수신, 전처리, GPU 추론, 결과 발행, 진단, 파라미터 업데이트(Parameter Update)가 콜백 스케줄링이나 공유 자원을 통해 서로 간섭할 수 있기 때문이다. ROS2 콜백 그룹(Callback Group)과 실행기(Executor)를 이용해 워크로드를 분리하고 제한된 큐(Bounded Queue)를 사용하여 과부하를 방지할 수 있다. GPU 파이프라인에서는 제어되지 않은 동시 추론 요청이 유용한 처리량을 증가시키기보다 메모리 사용량과 지연시간을 증가시킬 수 있다.

수명주기 관리(Lifecycle Management)는 카메라 AI 노드를 체계적으로 초기화하는 방법을 제공한다. 설정(Configuration) 단계에서 노드는 모델 파일(Model File)을 로딩하고, 추론 런타임을 초기화하며, GPU 메모리를 할당하고, 파라미터를 검증하고, 구독자와 발행자를 설정할 수 있다. 활성화(Activation)는 이러한 자원이 준비된 이후 실제 이미지 처리를 시작하도록 한다. 비활성화(Deactivation)는 설정을 제거하지 않고 추론을 중단할 수 있으며, 정리(Cleanup)는 가속기 자원을 해제하여 제어된 시작, 재시작, 모델 교체, 장애 복구(Fault Recovery)를 지원한다.

런타임 파라미터(Runtime Parameter)는 노드를 다시 빌드하지 않고도 안전하게 변경할 수 있는 운용 설정을 제공해야 한다. 여기에는 신뢰도 임계값(Confidence Threshold), 추론 주기(Inference Frequency), 입력 해상도 선택(Input Resolution Selection), 활성 출력 클래스(Enabled Output Class), 시각화 옵션(Visualization Option), 진단 제한값(Diagnostic Limit) 등이 포함될 수 있다. 모델의 기본 텐서 구조나 안전 동작에 영향을 주는 파라미터는 더욱 강력한 검증이 필요하다. 동적 설정 가능성(Dynamic Configurability)은 파라미터 변경이 추론 파이프라인의 기본 가정을 암묵적으로 위반하지 않을 때에만 유용하다.

관측 가능성(Observability)은 카메라 AI 성능이 모델 정확도(Model Accuracy)만으로 결정되지 않기 때문에 필수적이다. 노드는 카메라 입력 주기(Camera Input Rate), 수용 및 드롭된 프레임(Accepted and Dropped Frame), 전처리 지연시간(Preprocessing Latency), 추론 지연시간(Inference Latency), 후처리 지연시간(Postprocessing Latency), 전체 종단간 지연시간, 출력 주기(Output Rate), 큐 점유율(Queue Occupancy), GPU 사용률(GPU Utilization), 메모리 사용량(Memory Consumption)을 측정해야 한다. 타임스탬프 기반 측정을 통해 성능 저하가 센서 전달, ROS2 스케줄링, 데이터 이동, 모델 실행 또는 하위 통신 중 어디에서 발생하는지 식별할 수 있다.

장애 처리(Fault Handling)는 이미지 누락(Missing Image), 잘못된 인코딩(Invalid Encoding), 카메라 연결 해제(Camera Disconnection), GPU 메모리 할당 실패, 모델 로딩 오류(Model Loading Error), 추론 예외(Inference Exception), 과도한 지연시간(Excessive Latency), 잘못된 출력(Invalid Output)에 대한 동작을 정의해야 한다. 카메라 AI 노드는 이러한 상태에서도 정상적인 결과처럼 보이는 데이터를 계속 발행하기보다 진단 및 수명주기 상태 전환(Lifecycle State Transition)을 통해 문제를 보고해야 한다. 이를 통해 하위 컴포넌트는 정상적인 인식과 성능이 저하되거나 사용할 수 없는 AI 기능을 구분하고 적절한 폴백 동작(Fallback Behavior)을 선택할 수 있다.

다중 카메라 로봇(Multi-Camera Robot)의 경우 아키텍처는 이미지와 추론 출력 사이에 모호한 관계가 발생하지 않도록 해야 한다. 네임스페이스(Namespace), 카메라 식별자(Camera Identifier), 프레임 식별자, 타임스탬프, 모델 인스턴스(Model Instance)를 사용하여 각각의 처리 경로를 명확하게 구분해야 한다. 일부 시스템은 각 카메라마다 독립적인 추론 노드를 사용할 수 있으며, 다른 시스템은 동기화된 여러 이미지를 하나의 다중 모달 모델(Multimodal Model)에 배치로 입력할 수 있다. 내부 추론 전략과 관계없이 인터페이스 설계는 센서 출처(Sensor Provenance)를 유지해야 한다.

전체 카메라 AI 파이프라인(Camera AI Pipeline)은 센싱(Sensing), 전처리, 가속 추론(Accelerated Inference), 후처리, ROS2 통신, 진단, 수명주기 관리를 하나의 통제된 데이터 경로(Controlled Data Path)로 연결한다. 이 아키텍처의 목표는 단순히 신경망 처리량(Neural-Network Throughput)을 최대화하는 것이 아니라 예측 가능한 통합(Predictable Integration)을 구현하는 것이다. 시간(Time), 기하 구조(Geometry), 모델 출처, 자원 사용(Resource Usage), 장애 동작(Failure Behavior)을 명시적으로 유지하면 카메라 기반 피지컬 AI(Camera-Based Physical AI)는 인식, 내비게이션(Navigation), 조작(Manipulation), VLA 추론(VLA Reasoning), 자율 로봇 행동(Autonomous Robot Behavior)을 지원하는 신뢰성 높은 모듈형 하위 시스템(Modular Subsystem)으로 동작할 수 있다.

## 10.04 LiDAR 3D Perception AI Node Integration [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

라이다 기반 3D 인식(LiDAR-Based 3D Perception)은 로봇에 주변 물리 환경(Physical Environment)에 대한 직접적인 기하학적 관측(Geometric Observation)을 제공하므로 자율 내비게이션(Autonomous Navigation)과 피지컬 AI(Physical AI)의 핵심 구성 요소다. ROS2 아키텍처에서 3D 인식 AI 노드(3D Perception AI Node)는 원시 포인트 클라우드 스트림(Raw Point-Cloud Stream)을 객체(Object), 표면(Surface), 장애물(Obstacle), 의미 영역(Semantic Region), 점유 정보(Occupancy Information), 학습된 공간 특징(Learned Spatial Feature)과 같은 구조화된 표현으로 변환하여 상위 로봇 소프트웨어가 활용할 수 있도록 한다.

처리 파이프라인(Processing Pipeline)은 일반적으로 PointCloud2와 같은 표준 ROS2 인터페이스를 통해 포인트 클라우드(Point Cloud)를 발행하는 라이다 드라이버(LiDAR Driver)에서 시작한다. 각 측정 데이터에는 3차원 좌표가 포함되며 추가적으로 강도(Intensity), 반사도(Reflectivity), 링 인덱스(Ring Index), 리턴 정보(Return Information), 센서별 속성(Sensor-Specific Attribute)이 포함될 수 있다. 원본 타임스탬프(Timestamp)와 프레임 식별자(Frame Identifier)를 유지하는 것은 이후 인식 결과를 생성한 실제 물리적 관측 및 좌표계와 연결하기 위해 필수적이다.

원시 라이다 데이터(Raw LiDAR Data)는 일반적으로 신경망 추론(Neural-Network Inference) 전에 전처리(Preprocessing)가 필요하다. 대표적인 연산에는 유효하지 않은 포인트 제거(Invalid-Point Removal), 거리 필터링(Range Filtering), 관심 영역 선택(Region-of-Interest Selection), 지면 필터링(Ground Filtering), 좌표 변환(Coordinate Transformation), 다운샘플링(Downsampling), 특징 생성(Feature Construction)이 포함된다. AI 아키텍처에 따라 포인트 클라우드는 추론 런타임(Inference Runtime)으로 전달되기 전에 원시 포인트(Raw Point), 필러(Pillar), 복셀(Voxel), 거리 이미지(Range Image), 희소 텐서(Sparse Tensor), 조감도 특징(Bird\'s-Eye-View Feature)으로 표현될 수 있다.

전처리 설계(Preprocessing Design)는 정확도(Accuracy)와 연산 비용(Computational Cost)에 직접적인 영향을 미친다. 과도한 다운샘플링은 지연시간(Latency)과 GPU 메모리 사용량을 줄일 수 있지만 작거나 멀리 있는 객체를 제거할 수 있으며, 고밀도 표현(Dense Representation)은 기하학적 세부 정보를 유지하는 대신 더 많은 연산 비용을 요구한다. 따라서 전처리 노드는 명시적인 공간 범위(Spatial Range), 복셀 해상도(Voxel Resolution), 샘플링 규칙(Sampling Rule), 특징 정의(Feature Definition)를 제공하여 실행 동작이 모델 학습 시 사용된 가정과 일치하도록 해야 한다.

모든 예측 결과가 실제 공간적 의미(Physical Spatial Meaning)를 가지므로 3D 인식에서는 좌표계 관리(Coordinate-Frame Management)가 특히 중요하다. 센서 측정값은 라이다 프레임(LiDAR Frame)에서 생성될 수 있지만 내비게이션과 계획(Planning)은 베이스(Base), 오도메트리(Odometry), 맵(Map) 좌표계에서 동작할 수 있다. ROS2 tf2 변환(Transformation)은 올바른 타임스탬프와 함께 적용되어야 하며, 추론이 완료된 이후의 최신 로봇 자세가 아니라 원본 스캔이 획득된 시점의 로봇 자세를 기준으로 탐지 결과를 변환해야 한다.

AI 추론 단계(AI Inference Stage)는 다양한 종류의 3D 인식 모델(3D Perception Model)을 지원할 수 있다. 노드는 3D 객체 탐지(3D Object Detection), 의미 분할(Semantic Segmentation), 포인트 분류(Point Classification), 주행 가능성 추정(Traversability Estimation), 지형 이해(Terrain Understanding), 점유 예측(Occupancy Prediction), 학습 기반 특징 추출(Learned Feature Extraction)을 수행할 수 있다. 더욱 발전된 피지컬 AI 시스템에서는 라이다 특징을 카메라, 레이더 또는 언어 조건 표현(Language-Conditioned Representation)과 결합하여 다중 모달 장면 이해(Multimodal Scene Understanding)와 후속 추론에 활용할 수 있다.

추론 실행(Inference Execution)은 외부 ROS2 메시지 계약(Message Contract)과 분리된 상태로 유지하는 것이 바람직하다. 내부 구현은 PyTorch, ONNX Runtime, TensorRT, CUDA, 희소 합성곱 라이브러리(Sparse Convolution Library), 전용 가속기 프레임워크(Dedicated Accelerator Framework)를 사용할 수 있지만 하위 노드에는 일관된 출력을 발행해야 한다. 이러한 분리를 통해 내비게이션, 매핑(Mapping), 계획, 플릿(Fleet) 소프트웨어를 다시 설계하지 않고도 모델, 추론 엔진, 정밀도 모드(Precision Mode), 연산 하드웨어를 변경할 수 있다.

대규모 포인트 클라우드(Large Point Cloud)는 상당한 메모리 및 통신 오버헤드(Communication Overhead)를 발생시킨다. PointCloud2 데이터를 반복적으로 직렬화(Serialization), 역직렬화(Deserialization), 복사하면 특히 고채널 라이다(High-Channel-Count LiDAR)에서 종단간 지연시간(End-to-End Latency)의 상당 부분을 차지할 수 있다. 전처리와 추론이 동일한 엣지 컴퓨터(Edge Computer)에서 실행되는 경우 ROS2 컴포지션(Composition), 프로세스 내부 통신(Intra-Process Communication), 공유 메모리(Shared Memory), 재사용 버퍼(Reusable Buffer), 적절한 데이터 소유권(Data Ownership) 설계를 통해 불필요한 데이터 이동을 줄일 수 있다.

GPU 중심 파이프라인(GPU-Oriented Pipeline)은 포인트 클라우드 표현 사이의 변환을 최소화해야 한다. 비효율적인 경로에서는 ROS2 데이터를 CPU 구조로 역직렬화하고, 다시 전처리 버퍼에 복사하며, 텐서를 생성한 후 GPU 메모리로 전송하고, 후처리 과정에서 유사한 복사를 반복할 수 있다. 효율적인 구현에서는 가능한 경우 메모리를 재사용하고, 변환 작업을 일괄 처리하며, 텐서를 사전 할당(Preallocation)하고, 중간 표현(Intermediate Representation)을 가속기 가까이에 유지하여 데이터 이동 비용을 최소화한다.

후처리(Postprocessing)는 모델별 출력(Model-Specific Output)을 로봇 수준의 공간 정보(Robot-Level Spatial Information)로 변환한다. 3D 탐지기는 객체 클래스(Object Class), 신뢰도(Confidence), 중심 위치(Center), 크기(Dimension), 방향(Orientation), 속도(Velocity)를 생성할 수 있으며, 분할 네트워크(Segmentation Network)는 포인트 또는 복셀에 의미 레이블(Semantic Label)을 할당할 수 있다. 점유 기반 모델(Occupancy-Oriented Model)은 자유 공간(Free), 점유 공간(Occupied), 미확인 공간(Unknown), 의미 공간 상태(Semantic Spatial State)를 생성할 수 있다. ROS2 출력 인터페이스는 내부 신경망 텐서 형식을 구독자가 직접 해석하도록 요구하기보다 이러한 운용 의미를 명시적으로 제공해야 한다.

로봇 또는 주변 객체가 추론 과정에서 이동할 수 있기 때문에 시간 일관성(Temporal Consistency)을 유지하는 것은 어려운 문제다. 따라서 발행되는 탐지 결과의 타임스탬프는 관측 시점(Observation Time)을 나타내야 하며, 추가적인 처리 타임스탬프(Processing Timestamp)를 통해 추론 완료 시점을 표시할 수 있다. 하위 시스템은 이 정보를 이용하여 자기 이동(Ego-Motion)을 보정하고, 오래된 예측(Stale Prediction)을 거부하며, 여러 스캔 사이의 탐지 결과를 연관시키거나 AI 결과가 내비게이션과 제어에 사용하기에 충분히 최신인지 판단할 수 있다.

다중 센서 융합(Multi-Sensor Fusion)은 더욱 엄격한 동기화를 요구한다. 카메라 이미지, 라이다 스캔, 레이더 측정, IMU 데이터는 서로 다른 주기와 전송 지연시간으로 도착할 수 있다. 융합 노드(Fusion Node)는 ROS2 콜백 도착 순서가 아니라 획득 시간(Acquisition Time)과 보정된 좌표 변환(Calibrated Transformation)을 기준으로 측정값을 연결해야 한다. 하드웨어 타임스탬프(Hardware Timestamp), 동기화된 클록(Synchronized Clock), tf2 히스토리(History), 명시적인 시간 허용 오차(Temporal Tolerance)를 사용하면 모달리티 사이의 물리적으로 의미 있는 대응 관계를 유지할 수 있다.

ROS2 서비스 품질(QoS) 설정은 라이다 스트림의 대용량 및 시간 민감 특성을 반영해야 한다. 깊은 큐(Deep Queue)는 운용 가치가 사라진 오래된 포인트 클라우드가 계속 연산 자원을 소비하게 만들 수 있으므로 문제가 될 수 있다. 센서 스트림은 얕은 히스토리(Shallow History)와 최신성 중심 동작(Freshness-Oriented Behavior)이 적합할 수 있으며, 작은 크기의 탐지 또는 점유 출력은 하위 시스템 요구사항에 따라 다른 신뢰성 정책(Reliability Policy)을 적용할 수 있다. 따라서 QoS는 각 인터페이스의 특성에 맞게 개별적으로 설계해야 한다.

동시성(Concurrency)은 고주기 포인트 클라우드 수신이 추론, 진단(Diagnostics), 제어 관련 통신을 차단하지 않도록 설계되어야 한다. 콜백 그룹(Callback Group)과 적절한 ROS2 실행기(Executor)를 사용하여 각 기능을 분리할 수 있으며, 제한된 처리 큐(Bounded Processing Queue)를 통해 통제되지 않은 데이터 적체를 방지할 수 있다. 추론 처리 속도가 라이다 획득 주기를 따라가지 못하는 경우 지연시간을 암묵적으로 누적하는 것보다 명시적인 스캔 선택(Scan Selection) 또는 프레임 건너뛰기(Frame-Skipping) 정책을 사용하는 것이 바람직하다.

수명주기 관리(Lifecycle Management)는 연산 비용이 큰 3D 인식 자원을 제어하는 데 유용하다. 설정(Configuration) 단계에서 노드는 모델 가중치(Model Weight)를 로딩하고, 희소 합성곱(Sparse Convolution) 또는 TensorRT 엔진을 초기화하며, GPU 메모리를 할당하고, 라이다 파라미터를 검증하고, 발행자와 구독자를 구성할 수 있다. 초기화가 성공한 이후 활성화(Activation)를 통해 실제 인식을 시작하며, 비활성화(Deactivation)와 정리(Cleanup)를 이용하여 추론 중단, 자원 해제, 장애 복구(Fault Recovery), 모델 교체를 통제된 방식으로 수행할 수 있다.

관측 가능성(Observability)은 신경망 실행만이 아니라 전체 라이다 AI 파이프라인을 대상으로 해야 한다. 주요 측정 항목에는 포인트 클라우드 입력 주기(Point-Cloud Input Rate), 포인트 수(Point Count), 필터링된 포인트 수(Filtered Point Count), 전처리 지연시간(Preprocessing Latency), 호스트-장치 전송 시간(Host-to-Device Transfer Time), 추론 지연시간(Inference Latency), 후처리 지연시간(Postprocessing Latency), 출력 주기(Output Rate), 큐 점유율(Queue Occupancy), GPU 사용률(GPU Utilization), 메모리 사용량(Memory Consumption), 누락 스캔(Dropped Scan), 발행 결과의 종단간 경과 시간(End-to-End Age)이 포함된다. 이를 통해 센싱, 미들웨어, 전처리, 추론, 발행 과정의 병목을 식별할 수 있다.

장애 처리(Fault Handling)는 잘못된 포인트 클라우드(Malformed Point Cloud), 누락된 스캔(Missing Scan), 잘못된 프레임 변환(Incorrect Frame Transformation), 타임스탬프 불연속(Timestamp Discontinuity), 모델 초기화 실패(Model Initialization Failure), GPU 오류, 과도한 추론 지연시간, 잘못된 출력(Invalid Output)을 고려해야 한다. 진단 상태(Diagnostic State)는 인식 기능이 정상(Healthy), 성능 저하(Degraded), 사용 불가(Unavailable) 중 어느 상태인지 명확하게 나타내야 한다. 이를 통해 내비게이션 및 제어 컴포넌트는 누락되거나 오래된 AI 인식 결과를 정상적인 자유 공간 정보로 처리하지 않고 보수적인 폴백 동작(Conservative Fallback Behavior)을 선택할 수 있다.

다중 라이다 로봇(Multi-LiDAR Robot)의 경우 네임스페이스(Namespace)와 프레임 식별자를 사용하여 센서 출처(Sensor Provenance)를 명확하게 유지해야 한다. 개별 센서는 전방, 후방, 측면 또는 상부의 서로 다른 시야 영역(Field of View)을 담당할 수 있으며, 각 스트림을 독립적으로 처리하거나 하나의 공통 표현(Common Representation)으로 융합할 수 있다. 여러 포인트 클라우드가 공유 3D 환경 모델(Shared 3D Environmental Model)에 기여하는 경우 보정(Calibration), 동기화, 중첩 영역 처리(Overlap Handling), 중복 객체 관리(Duplicate-Object Management)가 통합 아키텍처의 일부가 된다.

전체 ROS2 라이다 3D 인식 아키텍처(ROS2 LiDAR 3D Perception Architecture)는 물리적 센싱(Physical Sensing), 공간 전처리(Spatial Preprocessing), 가속기 기반 AI 추론(Accelerator-Based AI Inference), 기하학적 후처리(Geometric Postprocessing), 좌표 변환, 미들웨어 통신(Middleware Communication), 진단, 수명주기 관리를 하나의 통제된 파이프라인(Controlled Pipeline)으로 연결한다. 시간(Time), 기하 구조(Geometry), 모델 출처(Model Provenance), 데이터 최신성(Data Freshness), 장애 상태(Failure State)를 유지함으로써 인식 노드는 SLAM, 내비게이션, 조작(Manipulation), 월드 모델(World Model), VLA 시스템(VLA System), 자율 피지컬 AI 행동(Autonomous Physical AI Behavior)에 신뢰성 있는 공간 지능(Spatial Intelligence)을 제공할 수 있다.

## 10.05 VLA Model Inference: ROS2 Service Wrapper [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

비전-언어-행동 모델(Vision-Language-Action Model)은 시각 인식(Visual Perception)과 자연어 명령(Natural-Language Instruction)을 로봇 중심의 행동(Robot-Oriented Action)에 직접 연결하며, 피지컬 AI(Physical AI) 시스템의 중요한 구성 요소가 된다. ROS2에서는 VLA 모델을 내비게이션(Navigation)이나 제어 노드(Control Node)에 긴밀하게 결합하기보다 잘 정의된 소프트웨어 인터페이스(Software Interface) 뒤에 격리하는 것이 바람직하다. ROS2 서비스 래퍼(Service Wrapper)는 로봇 요청을 모델 입력으로 변환하고 추론 결과를 구조화된 응답으로 변환함으로써 이러한 경계를 제공한다.

서비스 래퍼(Service Wrapper)는 ROS2 의미 체계(Semantics)와 VLA 런타임(Runtime)이 요구하는 내부 표현 사이의 적응 계층(Adaptation Layer)으로 동작한다. 요청 노드(Requesting Node)는 토크나이저(Tokenizer), 비전 인코더(Vision Encoder), 텐서 레이아웃(Tensor Layout), GPU 메모리 또는 모델별 API를 이해할 필요가 없어야 한다. 대신 명령(Instruction), 관측 참조(Observation Reference), 로봇 상태(Robot State), 작업 문맥(Task Context), 실행 제약(Execution Constraint)과 같은 정보를 포함하는 명확하게 정의된 요청을 전송하고 명시적으로 형식화된 결과를 수신한다.

서비스 기반 통신(Service-Based Communication)은 VLA 추론이 제한된 요청-응답 작업(Bounded Request-Response Operation)으로 동작할 때 적합하다. 플래너(Planner)는 현재 장면의 해석, 조작 목표(Manipulation Target), 상위 수준 행동(High-Level Action)을 요청하고 결과를 기다린 후 다음 작업을 진행할 수 있다. 그러나 ROS2 서비스가 모든 VLA 상호작용을 표현하는 데 적합한 것은 아니다. 지속적인 피드백(Continuous Feedback), 취소(Cancellation), 중간 진행 상태(Intermediate Progress)가 필요한 장시간 작업은 일반적으로 ROS2 액션(Action)이나 상위 수준 작업 오케스트레이션(Task Orchestration)을 사용하는 것이 더 적합하다.

서비스 요청(Service Request)은 물리적으로 의미 있는 추론을 위해 충분한 문맥(Context)을 유지해야 한다. 시각 관측(Visual Observation)에는 RGB 또는 깊이 이미지(Depth Image), 최근 획득된 센서 데이터에 대한 참조, 객체 정보(Object Information), 인식 노드에서 생성된 임베딩(Embedding)이 포함될 수 있다. 언어 입력(Language Input)은 운영자 명령(Operator Command), 임무 설명(Mission Description), 작업 제약(Task Constraint)을 나타낼 수 있다. 로봇 상태 정보에는 자세(Pose), 관절 상태(Joint State), 그리퍼 상태(Gripper State), 사용 가능한 기능(Capability), 환경 문맥(Environmental Context)이 포함될 수 있으며, 이를 통해 모델이 로봇이 수행할 수 없는 행동을 추론하는 것을 제한할 수 있다.

대규모 시각 입력(Large Visual Input)은 세심한 인터페이스 설계(Interface Design)를 요구한다. 모든 서비스 요청에 전체 고해상도 이미지(High-Resolution Image)를 직접 포함하면 불필요한 직렬화(Serialization)와 메모리 복사(Memory Copy) 오버헤드가 발생할 수 있다. 배포 아키텍처(Deployment Architecture)에 따라 래퍼는 대신 동기화된 토픽 데이터(Synchronized Topic Data), 캐시된 관측(Cached Observation), 공유 메모리 참조(Shared-Memory Reference), 또는 압축된 학습 임베딩(Compact Learned Embedding)을 사용할 수 있다. 선택한 방식은 요청과 실제 추론에 사용된 정확한 관측 사이의 관계를 유지해야 한다.

VLA 추론은 변화하는 물리적 환경(Physical Environment)을 기반으로 하기 때문에 타임스탬프(Timestamp)와 출처 정보(Provenance Information)가 필수적이다. 래퍼는 시각 관측이 언제 획득되었는지, 어떤 센서가 생성했는지, 그리고 해당 관측과 어떤 로봇 상태가 대응되는지를 알아야 한다. 추론에 상당한 시간이 필요한 경우 응답에서는 관측 시간(Observation Time)과 추론 완료 시간(Inference Completion Time)을 구분하여 하위 컴포넌트가 결과가 여전히 실행에 유효한지 판단할 수 있도록 해야 한다.

래퍼 내부에서 전처리(Preprocessing)는 ROS2 수준 입력을 모델별 다중 모달 표현(Model-Specific Multimodal Representation)으로 변환한다. 이미지는 크기 조정(Resizing), 정규화(Normalization), 텐서 변환(Tensor Conversion), 비전 인코더 처리가 필요할 수 있으며, 언어 명령은 토큰화(Tokenization)와 프롬프트 구성(Prompt Construction)이 필요할 수 있다. 로봇 상태는 정규화하거나 구조화된 토큰(Structured Token)으로 인코딩할 수 있다. 이러한 변환은 모델 구현 변경이 ROS2 소프트웨어 스택 전체로 전파되지 않도록 래퍼 내부에 유지되어야 한다.

모델 런타임 인터페이스(Model-Runtime Interface)도 마찬가지로 ROS2 서비스 정의와 분리되어 추상화되어야 한다. VLA 구현은 PyTorch, ONNX Runtime, TensorRT, 가속기 전용 라이브러리(Accelerator-Specific Library), 원격 추론 백엔드(Remote Inference Backend)를 사용할 수 있다. 내부 모델, 런타임, 정밀도 모드(Precision Mode), 연산 플랫폼이 변경되더라도 서비스 계약(Service Contract)은 안정적으로 유지될 수 있다. 이러한 분리는 모든 ROS2 요청 노드를 변경하지 않고 모델 업그레이드(Model Upgrade)와 하드웨어 마이그레이션(Hardware Migration)을 수행할 수 있게 한다.

VLA 출력은 로봇에 제공되기 전에 명시적인 의미 해석(Semantic Interpretation)이 필요하다. 모델 내부에서는 행동 토큰(Action Token), 텍스트 설명(Textual Description), 이산 스킬 식별자(Discrete Skill Identifier), 목표 자세(Target Pose), 웨이포인트 시퀀스(Waypoint Sequence), 연속 행동 벡터(Continuous Action Vector)가 생성될 수 있다. 래퍼는 문서화되지 않은 신경망 출력 형식을 그대로 노출하기보다 이러한 모델별 출력을 로봇이 무엇을 수행해야 하는지를 명확하게 전달하는 ROS2 표현으로 변환해야 한다.

VLA 추론 결과를 액추에이터 명령(Actuator Command)으로 직접 변환하는 방식은 일반적으로 피해야 한다. 서비스 응답은 하위 계획 및 제어 컴포넌트가 검증할 수 있는 행동 제안(Action Proposal), 작업 결정(Task Decision), 목표 상태(Target State), 스킬 선택(Skill Selection), 모션 목표(Motion Objective)로 해석하는 것이 바람직하다. 이후 결정론적 소프트웨어(Deterministic Software)가 운동학적 제약(Kinematic Constraint), 충돌 검사(Collision Checking), 작업 공간 제한(Workspace Limit), 속도 제한(Velocity Restriction), 안전 정책(Safety Policy)을 적용함으로써 학습 기반 추론과 물리적 구동(Physical Actuation) 사이에 통제된 경계를 유지할 수 있다.

모델이나 애플리케이션이 의미 있는 측정값을 제공할 수 있는 경우 VLA 결과에는 신뢰도(Confidence)와 유효성 정보(Validity Information)가 함께 포함되어야 한다. 응답에는 추론 성공 여부, 결과를 생성한 모델, 사용된 관측, 파싱(Parsing) 또는 검증 과정에서 지원되지 않는 행동이 발견되었는지를 나타낼 수 있다. 단순히 서비스 호출이 정상적으로 완료되었다는 이유만으로 유효한 명령으로 처리하는 것보다 구조화된 상태 필드(Structured Status Field)를 사용하는 것이 바람직하다. 연산 성공이 반드시 물리적으로 실행 가능한 결과를 의미하는 것은 아니기 때문이다.

VLA 모델은 기존 ROS2 기능보다 훨씬 크고 변동성이 높은 지연시간(Latency)을 가질 수 있으므로 타임아웃 처리(Timeout Handling)가 특히 중요하다. 서비스 클라이언트(Service Client)는 결과가 유효하게 사용될 수 있는 시간을 정의해야 하며, 래퍼는 과부하된 추론 큐(Inference Queue) 또는 정지된 런타임(Stalled Runtime)을 탐지해야 한다. 응답이 도착하기 전에 관련 장면이 변경되었다면 하위 소프트웨어는 오래된 환경 정보에서 생성된 행동을 실행하지 않고 해당 결과를 거부할 수 있어야 한다.

여러 ROS2 노드가 동시에 VLA 추론을 요청하는 경우 동시성 정책(Concurrency Policy)을 명시적으로 정의해야 한다. 통제되지 않은 병렬 요청(Parallel Request)은 특히 대규모 다중 모달 모델(Large Multimodal Model)에서 GPU 메모리를 고갈시키거나 예측하기 어려운 지연시간을 발생시킬 수 있다. 래퍼는 요청을 직렬화(Serialization)하거나 제한된 큐(Bounded Queue)를 유지하고, 제어된 배치 처리(Controlled Batching)를 지원하거나 여러 추론 작업자(Inference Worker)에 요청을 분배할 수 있다. 선택 전략은 응답 지연시간, 처리량(Throughput), 공정성(Fairness), 자원 격리(Resource Isolation) 중 어떤 요소가 우선되는지에 따라 결정해야 한다.

수명주기 관리(Lifecycle Management)는 VLA 서비스를 제어하는 데 유용한 메커니즘을 제공한다. 설정(Configuration) 단계에서 래퍼는 모델 가중치(Model Weight)를 로딩하고, 토크나이저와 비전 인코더를 초기화하며, 가속기 메모리(Accelerator Memory)를 할당하고, 모델 파일을 검증하며, 워밍업 추론(Warm-Up Inference)을 수행할 수 있다. 이러한 작업이 성공한 이후에만 서비스가 실제 동작 상태로 전환되어야 한다. 비활성화(Deactivation)를 통해 새로운 추론 요청의 수신을 중단할 수 있으며, 정리(Cleanup)는 제어된 재시작 또는 교체를 위해 모델 및 가속기 자원을 해제한다.

런타임 모델 교체(Runtime Model Replacement)는 서비스가 제공하는 ROS2 아키텍처를 변경하지 않고 지원할 수 있다. 새로운 VLA 모델을 로딩하고 검증하더라도 의미 계약(Semantic Contract)이 호환되는 경우 클라이언트는 동일한 논리적 요청 및 응답 정의를 계속 사용할 수 있다. 결과를 생성한 정확한 모델 설정(Model Configuration)을 추적하고 호환되지 않는 표현을 탐지할 수 있도록 모델 식별자(Model Identifier)와 인터페이스 버전(Interface Version)을 기록해야 한다.

관측 가능성(Observability)은 전체 서비스 경로(Service Path)를 대상으로 해야 한다. 주요 측정 항목에는 요청 주기(Request Rate), 큐 대기 시간(Queue Waiting Time), 전처리 지연시간(Preprocessing Latency), 모델 추론 지연시간(Model Inference Latency), 후처리 시간(Postprocessing Time), 전체 응답 지연시간(Total Response Latency), 타임아웃 횟수(Timeout Count), 실패율(Failure Rate), GPU 사용률(GPU Utilization), 메모리 사용량(Memory Consumption), 모델 식별 정보(Model Identity)가 포함된다. 이러한 지표를 통해 느린 VLA 동작이 ROS2 통신, 스케줄링, 전처리, 가속기 실행 또는 출력 해석 중 어디에서 발생하는지 구분할 수 있다.

장애 처리(Fault Handling)는 잘못 구성된 요청(Malformed Request), 누락된 관측(Missing Observation), 유효하지 않은 언어 입력(Invalid Language Input), 사용할 수 없는 모델 파일, GPU 메모리 할당 실패, 추론 예외(Inference Exception), 지원되지 않는 행동 출력(Unsupported Action Output), 과도한 지연시간(Excessive Latency)을 처리해야 한다. 래퍼는 부분적으로 유효한 행동 데이터를 생성하기보다 명시적인 오류 상태(Error State)를 반환하고 진단 정보(Diagnostic Information)를 발행해야 한다. 상위 오케스트레이션(Orchestration)은 이를 기반으로 재시도하거나 다른 모델을 선택하고, 기존 플래너(Conventional Planner)를 호출하거나 운영자 지원(Operator Assistance)을 요청하거나 사전에 정의된 안전 동작(Safe Behavior)으로 전환할 수 있다.

언어 명령(Language Instruction)이 물리적 행동에 영향을 줄 수 있기 때문에 보안(Security) 역시 중요하다. ROS2 접근 제어(Access Control)는 어떤 노드가 VLA 서비스를 호출할 수 있는지 제한해야 하며, 분산 배포(Distributed Deployment) 환경에서는 요청과 응답에 인증(Authentication)과 보호된 통신(Protected Communication)이 필요할 수 있다. 래퍼는 언어 입력을 신뢰되지 않은 작업 데이터(Untrusted Task Data)로 취급하고, 모델이 생성한 출력이 실행되기 전에 로봇 기능(Capability), 운용 조건(Operational Constraint), 안전 검증(Safety Validation)을 반드시 통과하도록 해야 한다.

결과적으로 VLA ROS2 서비스 래퍼(VLA ROS2 Service Wrapper)는 다중 모달 파운데이션 모델 추론(Multimodal Foundation-Model Inference)과 기존 로봇 소프트웨어(Conventional Robotic Software) 사이에 통제된 경계를 형성한다. 전처리, 모델 런타임, 자원 관리(Resource Management), 검증(Validation), 시간 관리(Timing), 진단, 장애 처리를 내부에 캡슐화(Encapsulation)하면서 안정적인 ROS2 의미 체계를 외부에 제공한다. 이를 통해 VLA 지능은 독립적으로 발전할 수 있으며, 내비게이션, 조작(Manipulation), 작업 계획(Task Planning), 안전(Safety), 제어(Control)는 신뢰할 수 있는 피지컬 AI 아키텍처의 모듈형 컴포넌트로 유지될 수 있다.

## 10.06 AI Inference Result to Motion Command Node [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

AI 추론 결과-모션 명령 노드(AI Inference Result-to-Motion Command Node)는 확률적 피지컬 AI 추론(Probabilistic Physical AI Reasoning)과 결정론적 로봇 모션(Deterministic Robot Motion) 사이의 핵심적인 전환 지점을 형성한다. 이 노드의 목적은 신경망 출력을 액추에이터(Actuator)에 직접 전달하는 것이 아니라 AI가 생성한 결과를 해석하고 검증하며 제약하고 변환하여 기존 ROS2 계획 및 제어 컴포넌트가 안전하게 실행할 수 있는 모션 목표(Motion Objective)로 만드는 것이다. 이러한 경계를 통해 학습 기반 지능(Learned Intelligence)과 물리적 제어(Physical Control) 사이의 모듈성을 유지할 수 있다.

노드의 입력은 객체 탐지(Object Detection), 3D 인식(3D Perception), VLA 모델(VLA Model), 학습된 정책(Learned Policy), 궤적 예측(Trajectory Prediction), 의미 기반 내비게이션(Semantic Navigation), 월드 모델 추론(World-Model Inference)에서 생성될 수 있다. 이러한 상위 컴포넌트는 목표 자세(Target Pose), 객체 위치(Object Location), 웨이포인트(Waypoint), 속도 제안(Velocity Suggestion), 스킬 식별자(Skill Identifier), 파지 후보(Grasp Candidate), 연속 행동 벡터(Continuous Action Vector)를 생성할 수 있다. 변환 노드는 하위 모션 요청을 생성하기 전에 각 표현의 의미론적 의미(Semantic Meaning)를 이해해야 한다.

여러 AI 모델이 동일한 모션 아키텍처(Motion Architecture)에 연결되는 경우 정규화된 중간 표현(Normalized Intermediate Representation)을 사용하는 것이 유용하다. 각 모델마다 별도의 제어 인터페이스(Control Interface)를 구현하는 대신 노드는 서로 다른 출력을 목표 자세, 목표 속도(Target Velocity), 궤적 기준(Trajectory Reference), 조작 목표(Manipulation Goal), 내비게이션 웨이포인트(Navigation Waypoint), 스킬 요청(Skill Request)과 같은 공통 개념으로 변환할 수 있다. 이러한 추상화(Abstraction)는 빠르게 변화하는 AI 모델과 상대적으로 안정적인 로봇 계획 및 제어 소프트웨어 사이의 결합도를 낮춘다.

모든 추론 결과(Inference Result)는 모션 변환을 시작하기 전에 검증되어야 한다. 검증에는 메시지 무결성(Message Integrity), 모델 식별 정보(Model Identity), 신뢰도(Confidence), 타임스탬프 최신성(Timestamp Freshness), 좌표계(Coordinate Frame), 차원 일관성(Dimensional Consistency), 허용된 행동 유형(Allowed Action Type), 필요한 로봇 상태(Required Robot State)가 포함될 수 있다. 수치적으로 유효한 출력이라도 오래된 관측, 알 수 없는 좌표계, 지원되지 않는 로봇 기능 또는 허용된 작업 공간 밖의 목표를 참조한다면 운용상 유효하지 않을 수 있다.

AI 추론은 가변적인 지연시간(Variable Latency)을 발생시키기 때문에 시간적 유효성(Temporal Validity)이 특히 중요하다. 노드는 추론 결과를 생성한 관측의 타임스탬프를 유지하고 이를 현재 로봇 상태(Current Robot State)와 비교해야 한다. 추론 과정에서 로봇이나 목표 객체가 크게 이동했다면 제안된 행동은 현재의 물리적 상황과 더 이상 일치하지 않을 수 있다. 데이터 유효 시간 제한(Age Limit), 타임아웃 정책(Timeout Policy), 상태 재조정(State Reconciliation), 재추론(Re-Inference)을 통해 오래된 예측이 모션 명령으로 변환되는 것을 방지할 수 있다.

AI 출력과 제어 컴포넌트가 서로 다른 좌표계에서 동작하는 경우 좌표 변환(Coordinate Transformation)이 필요하다. 카메라 모델은 객체 위치를 카메라 좌표계(Camera Frame)에서 예측할 수 있지만 매니퓰레이터(Manipulator)는 자신의 베이스 좌표계(Base Frame)에 정의된 목표가 필요하고, 모바일 로봇 플래너(Mobile Robot Planner)는 맵 좌표(Map Coordinate)를 요구할 수 있다. ROS2 tf2를 이용하여 타임스탬프와 일관된 좌표 변환을 수행함으로써 인식 결과와 물리적 실행 사이의 기하학적 관계(Geometric Relationship)를 유지할 수 있다.

신뢰도 값(Confidence Value)은 의사결정 로직(Decision Logic)에 활용될 수 있지만 보편적인 안전 보장(Universal Safety Guarantee)으로 해석해서는 안 된다. 서로 다른 모델이 생성하는 신뢰도는 의미와 보정 특성(Calibration Characteristic)이 서로 다를 수 있다. 노드는 모델별 허용 임계값(Acceptance Threshold)이나 불확실성 정책(Uncertainty Policy)을 적용할 수 있지만, 허용된 결과 역시 독립적인 기하학적, 운용적, 안전 검증을 거쳐야 한다. 낮은 신뢰도의 출력은 거부, 재관측(Re-Observation), 저속 동작(Slower Behavior), 다른 의사결정 메커니즘으로의 전환을 유발할 수 있다.

AI 결과를 모션 목표로 변환하는 방법은 로봇 유형(Robot Type)에 따라 달라진다. 모바일 로봇(Mobile Robot)의 경우 의미 기반 추론 결과가 내비게이션 목표(Navigation Goal), 로컬 웨이포인트(Local Waypoint), 방향 기준(Heading Reference), 제한된 속도 제안(Bounded Velocity Suggestion)으로 변환될 수 있다. 매니퓰레이터의 경우 말단장치 자세(End-Effector Pose), 파지 후보, 관절 공간 목표(Joint-Space Objective), 명명된 스킬(Named Skill)이 될 수 있다. 보행 로봇(Legged Robot)과 무인항공기(UAV)는 직접적인 액추에이터 수준 명령보다 몸체 자세(Body Pose), 발 디딤 목표(Foothold Target), 궤적(Trajectory), 비행 목표(Flight Objective)를 요구할 수 있다.

변환된 목표를 하위 시스템으로 전달하기 전에 모션 제약(Motion Constraint)을 적용해야 한다. 이러한 제약에는 속도 및 가속도 제한(Velocity and Acceleration Limit), 관절 제한(Joint Limit), 작업 공간 경계(Workspace Boundary), 최소 장애물 이격거리(Minimum Obstacle Clearance), 방향 제한(Orientation Restriction), 지형 제약(Terrain Constraint), 페이로드 조건(Payload Condition), 임무별 운용 영역(Mission-Specific Operational Zone)이 포함될 수 있다. AI 출력은 로봇의 물리적 제한을 무시할 수 있는 권한이 아니라 이러한 경계 내에서 제시되는 행동 제안(Action Proposal)으로 해석해야 한다.

이 노드는 일반적으로 저수준 액추에이터 신호를 직접 생성하기보다 기존 모션 생성 컴포넌트(Motion-Generation Component)와 연동해야 한다. 내비게이션 목표는 내비게이션 플래너(Navigation Planner), 조작 목표는 모션 플래너(Motion Planner), 궤적 기준은 적절한 ROS2 토픽(Topic), 서비스(Service), 액션(Action)을 통해 제어기(Controller)에 전달할 수 있다. 이러한 아키텍처를 통해 AI 기반 의사결정 이후 기존 알고리즘이 경로 생성(Path Generation), 역기구학(Inverse Kinematics), 충돌 검사(Collision Checking), 궤적 최적화(Trajectory Optimization), 피드백 제어(Feedback Control)를 수행할 수 있다.

ROS2 액션(Action)은 AI에서 생성된 모션 요청이 상당한 시간이 필요한 작업을 시작하는 경우 특히 유용하다. 명령 노드는 내비게이션, 조작, 도킹(Docking), 점검(Inspection) 목표를 제출하고 실행 진행 상황에 대한 피드백을 받을 수 있다. 새로운 인식 결과로 기존 결정이 더 이상 유효하지 않게 되면 취소(Cancellation)와 선점(Preemption)을 수행할 수 있다. 반면 제한된 속도 명령처럼 짧은 수명의 기준값은 연속 스트리밍이 적절한 경우 토픽을 사용할 수 있다.

안전 감독(Safety Supervision)은 AI-모션 변환 경로와 독립적으로 유지되어야 한다. 의미적 및 기하학적 검증을 통과한 이후에도 모션 요청은 충돌 모니터(Collision Monitor), 안전 제어기(Safety Controller), 속도 및 분리 감시(Speed-and-Separation Logic), 지오펜싱(Geofencing), 액추에이터 보호(Actuator Protection), 비상 정지(Emergency Stop) 메커니즘을 통과해야 할 수 있다. 명령 노드는 AI 추론의 성공을 이러한 계층을 우회할 수 있는 권한으로 해석해서는 안 된다. 학습 기반 지능은 행동을 제안하지만 안전 메커니즘은 물리적 실행에 대한 권한을 유지한다.

주기 불일치(Rate Mismatch) 역시 고려해야 한다. AI 추론은 초당 몇 회 수준으로 업데이트될 수 있지만 모션 제어기(Motion Controller)는 초당 수백 또는 수천 회 동작할 수 있다. 전체 제어 아키텍처가 이를 위해 명시적으로 설계되지 않은 경우 변환 노드는 저주기 신경망 예측으로 고주기 피드백 루프(High-Frequency Feedback Loop)를 대체해서는 안 된다. 대신 AI 결과는 현재 센서 피드백을 이용하는 결정론적 제어기가 지속적으로 추종할 수 있는 기준값(Reference)이나 목표(Objective)를 설정하는 데 사용될 수 있다.

상태 머신(State Machine) 또는 행동 오케스트레이션(Behavior Orchestration)은 추론 결과가 실행 가능한 명령으로 변환되는 과정을 추가적으로 제어할 수 있다. 목표는 로봇이 적절한 운용 상태(Operational State)에 있고, 관련 센서가 정상이며, 위치 추정(Localization)이 유효하고, 충돌하는 다른 임무가 존재하지 않을 때에만 허용될 수 있다. 대기(Waiting), 검증(Validating), 명령(Commanding), 실행(Executing), 완료(Completed), 거부(Rejected), 장애(Faulted)와 같은 명시적인 상태를 사용하면 AI 추론과 로봇 행동 사이의 상호작용을 쉽게 이해하고 진단할 수 있다.

여러 AI 컴포넌트가 서로 경쟁하는 모션 제안(Motion Proposal)을 생성하는 경우 동시성(Concurrency)이 중요해진다. 인식 모델이 장애물 회피(Obstacle Avoidance)를 요청하는 동시에 VLA 시스템이 조작을 제안하고 임무 플래너(Mission Planner)가 내비게이션을 요청할 수 있다. 이러한 상황에서는 제어되지 않은 콜백 순서가 아니라 중재(Arbitration) 또는 우선순위 결정(Prioritization)이 필요하다. 변환 노드는 각 제안 행동의 명령 출처(Command Source), 우선순위(Priority), 유효 기간(Validity Interval), 요청 기능(Requested Capability), 실행 상태(Execution Status)를 식별함으로써 이 과정에 참여할 수 있다.

수명주기 관리(Lifecycle Management)를 이용하면 필요한 종속 시스템(Dependency)이 준비된 이후에만 모션 변환 기능을 활성화할 수 있다. 설정(Configuration) 단계에서는 좌표계, 파라미터(Parameter), 로봇 제한(Robot Limit), 하위 액션 서버(Action Server), 안전 인터페이스(Safety Interface)를 검증할 수 있다. 활성화(Activation)는 명령 생성을 허용하고, 비활성화(Deactivation)는 새로운 AI 기반 모션 요청이 전달되는 것을 차단한다. 이를 통해 시스템 시작, 유지보수(Maintenance), 모델 교체(Model Replacement), 시스템 장애 복구(System Fault Recovery)를 통제된 방식으로 수행할 수 있다.

관측 가능성(Observability)은 추론 결과에서 실제 물리 명령으로 이어지는 전체 전환 과정을 포함해야 한다. 유용한 측정 항목에는 입력 결과 주기(Input Result Rate), 허용 및 거부된 제안(Accepted and Rejected Proposal), 추론 결과의 경과 시간(Inference Age), 검증 시간(Validation Time), 좌표 변환 지연시간(Transformation Latency), 명령 생성 시간(Command Generation Time), 하위 시스템 수락 여부(Downstream Acceptance), 실행 완료(Execution Completion), 취소 빈도(Cancellation Frequency), 안전 거부 이벤트(Safety Rejection Event)가 포함된다. 모델 식별 정보와 원본 타임스탬프를 기록하면 특정 모션 목표가 생성된 이유를 추적할 수 있다.

장애 처리(Fault Handling)는 누락된 좌표 변환(Missing Transformation), 오래된 추론(Stale Inference), 잘못된 메시지(Malformed Message), 유효하지 않은 목표(Invalid Target), 도달 불가능한 자세(Unreachable Pose), 사용할 수 없는 플래너(Unavailable Planner), 제어기 장애(Controller Fault), 안전 시스템 거부(Safety-System Rejection)를 명시적으로 처리해야 한다. 노드는 문제가 있는 결과를 조용히 폐기하거나 잘못된 명령을 반복적으로 발행하기보다 명확한 상태 정보를 반환하거나 발행해야 한다. 로봇 상태에 따라 폴백 동작(Fallback Behavior)은 위치 유지(Hold Position), 정지(Stop), 새로운 인식 요청(Request New Perception), 다른 플래너 선택(Alternative Planner Selection), 상위 감독 로직(Supervisory Logic)으로의 전환을 포함할 수 있다.

결과적으로 이 아키텍처는 AI 인식 및 추론(AI Perception and Reasoning)에서 로봇 움직임(Robot Movement)으로 이어지는 통제된 의미론적 연결(Semantic Bridge)을 형성한다. AI 모델은 학습된 해석(Learned Interpretation)과 행동 제안을 제공하고, 변환 노드는 시간적, 기하학적, 운용적, 기능적 유효성을 검증하며, 계획 컴포넌트(Planning Component)는 실행 가능한 모션을 생성한다. 이후 결정론적 제어기(Deterministic Controller)가 궤적을 실행하고 독립적인 안전 메커니즘이 물리 시스템을 감독한다. ROS2는 이러한 책임을 명확하게 분리하면서도 하나의 피지컬 AI 파이프라인(Physical AI Pipeline)으로 통합하여 동작할 수 있도록 하는 인터페이스를 제공한다.

## 10.07 AI Model Hot Swap: Runtime Model Replacement [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

AI 모델 핫 스왑(AI Model Hot Swap)은 ROS2 기반 피지컬 AI(Physical AI) 시스템이 계속 운용되는 동안 추론 모델(Inference Model)을 교체할 수 있는 기능이다. 전체 로봇 소프트웨어 스택을 중지하는 대신 모델 로딩(Model Loading), 검증(Validation), 활성화(Activation), 폐기(Retirement)를 통제된 런타임 경계(Runtime Boundary) 내부에 격리한다. 이를 통해 인식(Perception), 추론(Reasoning), 정책 모델(Policy Model)을 독립적으로 발전시키면서 주변 로봇 컴포넌트에는 안정적인 ROS2 인터페이스를 지속적으로 제공할 수 있다.

런타임 모델 교체(Runtime Model Replacement)는 피지컬 AI 모델이 기존 로봇 인터페이스보다 훨씬 빈번하게 변경되기 때문에 중요하다. 새로운 탐지기(Detector)는 환경 인식 성능을 향상시킬 수 있고, VLA 모델(VLA Model)은 추가적인 스킬을 확보할 수 있으며, 최적화된 TensorRT 엔진은 추론 지연시간(Inference Latency)을 줄일 수 있다. 모든 모델 업데이트마다 내비게이션(Navigation), 제어(Control), 센싱(Sensing), 미들웨어(Middleware) 컴포넌트를 재시작해야 한다면 배포 과정에 큰 중단이 발생한다. 핫 스왑은 모델의 진화와 전체 시스템 가용성(System Availability)을 분리한다.

기본적인 설계 원칙은 논리적 추론 인터페이스(Logical Inference Interface)와 현재 활성화된 모델 구현(Active Model Implementation)을 분리하는 것이다. ROS2 토픽(Topic), 서비스(Service), 액션(Action)은 안정적인 입력 및 출력 계약(Input and Output Contract)을 제공하고, 내부 모델 관리자(Model Manager)가 각 요청을 어떤 모델 인스턴스(Model Instance)가 처리할지 결정해야 한다. 따라서 클라이언트는 특정 신경망 객체, 런타임 엔진(Runtime Engine), 모델 파일을 직접 지정하지 않고 지속적으로 유지되는 추론 엔드포인트(Inference Endpoint)와 통신한다.

모델 레지스트리(Model Registry)는 여러 모델 버전을 관리하는 데 필요한 정보를 제공한다. 등록된 각 모델에는 고유 식별자(Unique Identifier), 시맨틱 버전(Semantic Version), 아키텍처 유형(Architecture Type), 입력 및 출력 스키마(Input and Output Schema), 정밀도 모드(Precision Mode), 런타임 종속성(Runtime Dependency), 가속기 요구사항(Accelerator Requirement), 체크섬(Checksum), 배포 상태(Deployment Status)를 포함할 수 있다. 추가 메타데이터에는 학습 출처(Training Provenance), 지원 센서 구성(Sensor Configuration), 전처리 프로파일(Preprocessing Profile), 예상 지연시간(Expected Latency), 특정 로봇 기능과의 호환성(Compatibility)을 기록할 수 있다.

핫 스왑은 활성 모델을 즉시 교체하는 방식이 아니라 통제된 로딩 단계(Controlled Loading Phase)에서 시작해야 한다. 기존 모델이 추론 요청을 계속 처리하는 동안 후보 모델(Candidate Model)을 CPU 또는 가속기 메모리(Accelerator Memory)에 로딩할 수 있다. 후보 모델을 활성화하기 전에 모델 파일, 메타데이터, 런타임 라이브러리, 텐서 차원(Tensor Dimension), 전처리 설정(Preprocessing Configuration), 필요한 하드웨어 자원을 검증해야 한다.

가속기 기반 추론(Accelerator-Based Inference)에서는 워밍업 단계(Warm-Up Phase)가 중요하다. CUDA 초기화, TensorRT 엔진 준비, 커널 선택(Kernel Selection), 메모리 할당(Memory Allocation), 그래프 컴파일(Graph Compilation), 캐시 생성(Cache Construction)으로 인해 최초 추론은 이후의 추론보다 상당히 느릴 수 있다. 활성화 전에 대표적인 워밍업 입력(Warm-Up Input)을 실행하면 이러한 초기화 비용을 실제 운용 추론 경로 외부에서 처리할 수 있으며, 트래픽이 전환되기 전에 런타임 장애(Runtime Failure)를 탐지할 기회도 제공한다.

호환성 검증(Compatibility Validation)은 단순히 모델 파일을 로딩할 수 있는지 확인하는 수준을 넘어야 한다. 후보 모델은 입력 해상도(Input Resolution), 텐서 레이아웃(Tensor Layout), 출력 클래스(Output Class), 좌표 규칙(Coordinate Convention), 임베딩 차원(Embedding Dimension), 행동 표현(Action Representation), 정규화 규칙(Normalization Rule), 필수 메타데이터를 포함한 기존 ROS2 의미 계약(Semantic Contract)에 대해 검증되어야 한다. 모델이 정상적으로 실행되더라도 출력의 의미가 변경되었다면 하위 노드와 호환되지 않을 수 있다.

기존 모델과 새로운 모델이 일시적으로 공존하는 경우 자원 검증(Resource Validation)도 매우 중요하다. 대규모 모델은 상당한 GPU 메모리를 사용할 수 있으며, 활성 모델과 후보 모델을 동시에 로딩하면 가속기의 용량을 초과할 수 있다. 모델 관리자는 교체를 시작하기 전에 사용 가능한 메모리, 연산 요구량(Compute Requirement), 런타임 종속성, 예상 동시성(Concurrency)을 평가해야 한다. 자원이 제한된 시스템에서는 단계적인 언로딩(Staged Unloading)이나 임시 폴백 추론 경로(Fallback Inference Path)가 필요할 수 있다.

실제 전환(Switchover)은 원자적 전환(Atomic Transition) 또는 논리적으로 원자적인 전환 방식으로 수행되어야 한다. 검증이 성공한 이후에만 새로운 추론 요청을 후보 모델로 전달하고, 이전 모델에서 이미 처리 중인 요청은 완료하도록 허용하거나 명시적인 취소 정책(Cancellation Policy)에 따라 처리한다. 이를 통해 하나의 요청이 서로 다른 모델 버전에 의해 부분적으로 처리되는 것을 방지하고, 발행되는 모든 결과에 대해 명확한 모델 출처(Model Provenance)를 유지할 수 있다.

이중 버퍼(Double Buffer) 또는 액티브-스탠바이 아키텍처(Active-Standby Architecture)를 사용하면 서비스 중단을 최소화할 수 있다. 하나의 모델 인스턴스가 활성 상태를 유지하는 동안 두 번째 슬롯에서 후보 모델을 준비한다. 검증과 워밍업이 완료되면 모델 관리자가 활성 모델 참조(Active-Model Reference) 또는 라우팅 테이블(Routing Table)을 변경하여 이후의 요청을 새로운 인스턴스로 전달한다. 진행 중인 작업이 완료되고 롤백 신뢰성이 확보될 때까지 이전 모델을 일시적으로 유지한 후 해당 자원을 해제할 수 있다.

모델 교체 과정과 교체 이후의 추론 출력에는 모델 출처 정보(Model Provenance)가 포함되어야 한다. 특히 하위 시스템의 동작이 학습된 표현(Learned Representation)에 의존하는 경우 각 결과에는 해당 결과를 생성한 모델 버전 또는 배포 식별자(Deployment Identifier)가 기록되어야 한다. 이러한 정보가 없으면 ROS2 인터페이스 수준에서는 동일하게 보이는 여러 모델의 출력이 로그에 혼재되어 성능 분석, 디버깅(Debugging), 사고 재구성(Incident Reconstruction), 회귀 문제 조사(Regression Investigation)가 크게 어려워질 수 있다.

상태 기반 AI 모델(Stateful AI Model)은 추가적인 상태 마이그레이션 로직(State Migration Logic)을 필요로 한다. 순환 정책(Recurrent Policy), 시간 기반 인식 네트워크(Temporal Perception Network), 월드 모델(World Model), 추적기(Tracker), 메모리 증강 시스템(Memory-Augmented System)은 새로운 모델로 단순히 이전할 수 없는 은닉 상태(Hidden State)를 유지할 수 있다. 모델 교체 과정에서는 상태를 초기화하거나, 호환 가능한 상태 표현을 변환하거나, 최근 관측으로부터 문맥을 재구성하거나, 새로운 모델이 충분한 시간적 이력(Temporal History)을 축적할 때까지 출력을 일시적으로 보류해야 할 수 있다.

수명주기 관리(Lifecycle Management)는 런타임 모델 교체를 전체 ROS2 시스템과 조정하는 데 활용할 수 있다. 수명주기를 인식하는 모델 관리자(Lifecycle-Aware Model Manager)는 후보 모델을 설정하고, 종속성을 검증하고, 활성화한 후 이전 인스턴스를 비활성화하고 마지막으로 사용하지 않는 자원을 정리할 수 있다. 이러한 전환은 관련 없는 센싱 또는 제어 노드의 수명주기와 분리하여 AI 모델 변경 때문에 정상적으로 동작하는 다른 컴포넌트까지 불필요하게 재시작되지 않도록 해야 한다.

전환 과정에서는 추론 트래픽(Inference Traffic)을 제어해야 한다. 고주기 카메라 또는 LiDAR 파이프라인은 후보 모델이 로딩되는 동안에도 계속 요청을 생성할 수 있으며, VLA 서비스에는 이미 실행 중인 장시간 요청이 존재할 수 있다. 제한된 큐(Bounded Queue), 승인 제어(Admission Control), 요청 드레이닝(Request Draining), 임시 라우팅 정책(Temporary Routing Policy)을 사용하면 통제되지 않은 요청 적체를 방지할 수 있다. 전환 메커니즘은 활성화 이전, 전환 중, 활성화 직후에 도착하는 요청을 어떻게 처리할지 명확하게 정의해야 한다.

롤백(Rollback)은 선택적인 복구 기능이 아니라 기본적인 요구사항이다. 새로운 모델이 유효하지 않은 출력, 과도한 지연시간, 메모리 압박(Memory Pressure), 런타임 예외(Runtime Exception), 허용할 수 없는 진단 상태를 발생시키는 경우 시스템은 이전에 검증된 모델을 복원할 수 있어야 한다. 일정한 안정화 기간(Stabilization Period) 동안 이전 인스턴스를 유지하면 전체 모델 로딩 및 초기화 과정을 반복하지 않고도 신속한 롤백을 수행할 수 있다.

후보 모델이 활성화된 이후에도 상태 모니터링(Health Monitoring)은 계속되어야 한다. 성공적인 로딩만으로 모델이 실제 센서 입력과 운용 워크로드(Operational Workload)에서 올바르게 동작한다는 것을 보장할 수 없다. 시스템은 추론 지연시간, 출력 주기(Output Rate), 유효하지 않은 결과 발생 빈도, GPU 메모리, 가속기 사용률(Accelerator Utilization), 큐 깊이(Queue Depth), 타임아웃 비율(Timeout Rate), 하위 시스템 거부 이벤트(Downstream Rejection Event)를 모니터링할 수 있다. 이러한 측정값은 교체된 모델이 지속적으로 정상 운용되는지 판단하기 위한 근거를 제공한다.

안전 민감 시스템(Safety-Sensitive System)에서는 모델 활성화와 물리적 움직임에 영향을 미칠 수 있는 권한(Authority)을 추가적으로 분리해야 한다. 새롭게 활성화된 인식 또는 정책 모델도 이전 모델과 동일한 검증, 계획, 제약, 안전 계층을 통과해야 한다. 핫 스왑은 학습 기반 추론의 출처를 변경하는 것이며 충돌 검사(Collision Checking), 액추에이터 제한(Actuator Limit), 비상 정지(Emergency Stop), 독립적인 안전 감독(Independent Safety Supervision)을 자동으로 변경해서는 안 된다.

런타임 모델 교체는 실행 가능한 AI 아티팩트(Executable AI Artifact)가 로봇 내부로 진입하는 경로를 추가하므로 보안(Security)과 무결성 제어(Integrity Control)도 필요하다. 모델 패키지는 배포 보안 아키텍처(Deployment Security Architecture)에 따라 인증(Authentication) 또는 다른 방식으로 검증되어야 하며, 체크섬을 이용하여 파일 손상(Corruption)을 탐지할 수 있다. 일반 ROS2 클라이언트가 운용 중인 지능 모델을 임의로 교체할 수 없도록 모델 등록, 활성화, 롤백, 제거 작업에 대한 접근 권한을 제한해야 한다.

관측 가능성(Observability)은 전체 모델 교체 과정을 기록해야 한다. 유용한 이벤트에는 후보 모델 등록(Candidate Registration), 로딩 시작 및 완료, 검증 결과, 워밍업 지연시간(Warm-Up Latency), 메모리 할당, 활성화 시간, 이전 모델 폐기(Previous-Model Retirement), 롤백 이벤트, 실패 원인(Failure Reason)이 포함된다. 이러한 이벤트를 추론 지표(Inference Metric) 및 로봇 동작과 연계하면 운용상의 변화가 소프트웨어 상태, 하드웨어 자원 또는 새롭게 배포된 모델 중 어디에서 발생했는지 판단할 수 있다.

결과적으로 이 아키텍처는 AI 모델을 영구적으로 내장된 컴포넌트가 아니라 교체 가능한 런타임 자원(Replaceable Runtime Resource)으로 취급한다. 안정적인 ROS2 추론 계약(Inference Contract)이 레지스트리, 로딩, 검증, 워밍업, 라우팅(Routing), 모니터링, 롤백, 정리 메커니즘을 포함하는 관리형 모델 런타임(Managed Model Runtime)을 둘러싸는 구조를 형성한다. 이를 통해 피지컬 AI 시스템은 추적 가능성(Traceability), 자원 제어(Resource Control), 결정론적 통합 경계(Deterministic Integration Boundary), 독립적인 로봇 안전 메커니즘을 유지하면서도 학습된 기능을 최소한의 시스템 중단으로 업데이트할 수 있다.

## 10.08 AI Inference Latency Monitoring Integration [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

피지컬 AI(Physical AI)에서 AI 추론 지연시간(AI Inference Latency)은 매우 중요한 런타임 특성(Runtime Property)이다. 추론 결과는 현재의 물리적 상황을 나타낼 수 있을 만큼 충분히 빠르게 도착할 때만 유용하기 때문이다. 매우 정확한 인지(Perception) 또는 정책 모델(Policy Model)이라도 과도한 지연으로 인해 출력이 이미 지나간 장면을 설명한다면 실제 운용에서는 효과가 크게 감소할 수 있다. 따라서 ROS2 지연시간 모니터링(Latency Monitoring)은 추론 시간을 단순한 모델 성능 벤치마크(Model-Performance Benchmark)가 아니라 전체 시스템 동작(System Behavior)의 일부로 다루어야 한다.

종단간 지연시간(End-to-End Latency)은 하나의 추론 시간 값으로 표현하기보다 측정 가능한 여러 단계로 분해해야 한다. 일반적인 파이프라인에는 센서 획득(Sensor Acquisition), ROS2 전송(ROS2 Transport), 입력 동기화(Input Synchronization), 전처리(Preprocessing), 큐 대기(Queue Waiting), 호스트-디바이스 전송(Host-to-Device Transfer), 모델 실행(Model Execution), 디바이스-호스트 전송(Device-to-Host Transfer), 후처리(Postprocessing), 결과 발행(Result Publication), 다운스트림 소비(Downstream Consumption)가 포함된다. 각 단계를 개별적으로 측정하면 지연이 신경망 모델, 미들웨어(Middleware), 스케줄링(Scheduling), 메모리 이동 또는 주변 소프트웨어 중 어디에서 발생하는지 식별할 수 있다.

타임스탬프 설계(Timestamp Design)는 의미 있는 측정의 기본 요소이다. 센서 메시지는 획득 타임스탬프(Acquisition Timestamp)를 보존해야 하며, 추가적인 타임스탬프를 통해 전처리 시작, 추론 제출(Inference Submission), 추론 완료, 결과 발행, 다운스트림 수신 시점을 식별할 수 있다. 이러한 시간 지점은 실제 물리적 관측에서 AI 출력까지 추적 가능한 지연시간 경로(Traceable Latency Path)를 생성한다. 일관된 타임스탬프가 없으면 모델 실행 시간과 큐 대기, 통신 또는 콜백 스케줄링(Callback Scheduling) 지연이 혼동될 수 있다.

여러 컴퓨터 또는 가속기 시스템(Accelerator System)에 걸쳐 지연시간을 측정하는 경우 클록 동기화(Clock Synchronization)가 필요하다. 서로 다른 엣지 컴퓨터(Edge Computer), 온프레미스 서버(On-Premise Server), 로봇 컨트롤러에서 실행되는 ROS2 노드는 시간 오프셋이 서로 다른 클록으로 생성된 타임스탬프를 비교할 수 있기 때문이다. PTP, gPTP 또는 적절한 다른 동기화 메커니즘을 이용하여 공통 시간 기준(Shared Time Reference)을 설정할 수 있다. 센서 획득 시점과 AI 처리 시점을 정밀하게 연계해야 하는 경우 하드웨어 타임스탬프(Hardware Timestamp)가 특히 유용하다.

모니터링 아키텍처(Monitoring Architecture)는 모델 추론 지연시간(Model Inference Latency)과 파이프라인 지연시간(Pipeline Latency)을 구분해야 한다. 모델 지연시간은 신경망 자체의 실행 구간을 측정하는 반면, 파이프라인 지연시간에는 전처리, 스케줄링, 데이터 전송, 추론, 후처리, ROS2 통신이 포함된다. 모델이 20밀리초(ms)에 실행되더라도 전체 결과가 소비자(Consumer)에 도달하는 데 80밀리초가 걸릴 수 있다. 따라서 모델만 최적화하면 실제 운용 지연의 대부분을 간과할 수 있다.

큐잉 지연시간(Queueing Latency)도 중요한 측정 항목이다. 과부하된 추론 노드(Inference Node)는 계산 자체는 빠르더라도 요청이 실행되기 전에 큐에서 오랫동안 대기할 수 있기 때문이다. 카메라 스트림, LiDAR 프레임 또는 여러 로봇 클라이언트가 가속기가 처리할 수 있는 속도보다 빠르게 작업을 생성할 수 있다. 큐 깊이(Queue Depth), 요청 경과시간(Request Age), 삭제된 입력(Dropped Input), 대기시간을 모니터링하면 평균 모델 실행시간만으로는 확인할 수 없는 포화 상태(Saturation Condition)를 발견할 수 있다.

GPU 타이밍(GPU Timing)은 가속기 실행이 흔히 비동기식(Asynchronous)이기 때문에 주의가 필요하다. 추론 API 호출 전후의 CPU 시간만 측정하면 GPU 계산이 완료되기 전에 호출이 반환되는 경우 실제보다 지나치게 짧은 시간이 측정될 수 있다. CUDA 이벤트(CUDA Event), 런타임별 프로파일링 메커니즘(Runtime-Specific Profiling Mechanism), 명시적인 동기화 지점(Explicit Synchronization Point)을 사용하면 가속기 실행시간을 보다 정확하게 측정할 수 있다. 그러나 계측(Instrumentation) 자체가 관측하려는 시간 동작을 변경하지 않도록 불필요한 동기화는 피해야 한다.

지연시간 통계(Latency Statistics)는 평균 성능뿐만 아니라 변동성(Variability)도 표현해야 한다. 평균 지연시간은 로봇 운용에서 중요한 간헐적인 장시간 지연을 숨길 수 있다. 중앙값(Median)과 함께 P95 및 P99와 같은 상위 백분위수(Upper Percentile), 최대 지연시간, 지터(Jitter), 타임아웃 발생 빈도, 데드라인 미스(Deadline Miss)를 모니터링하면 보다 완전한 상태를 파악할 수 있다. 드물게 발생하는 지연으로 오래된 인지 결과나 지연된 모션 판단이 발생할 수 있기 때문에 꼬리 지연시간(Tail Latency)은 특히 중요하다.

서로 다른 AI 워크로드(AI Workload)는 서로 다른 지연시간 예산(Latency Budget)을 필요로 한다. 장애물 대응에 사용되는 객체 탐지(Object Detection)는 의미론적 지도작성(Semantic Mapping)보다 엄격한 데드라인을 요구할 수 있으며, 고수준 작업 추론에 사용되는 시각-언어-행동 모델(Vision-Language-Action Model, VLA)은 훨씬 긴 추론 시간을 허용할 수 있다. 따라서 모니터링 시스템은 모든 AI 구성요소에 하나의 공통 지연 임계값을 적용하기보다 측정값을 워크로드 식별자, 모델 버전, 로봇 기능, 예상 데드라인과 연계해야 한다.

데이터 신선도(Data Freshness)는 처리 시간과 독립적으로 모니터링해야 한다. 추론 요청 자체는 빠르게 실행되더라도 해당 입력 관측 데이터가 큐 또는 동기화 버퍼(Synchronization Buffer)에서 과도한 시간을 보냈을 수 있다. 결과가 소비되는 시점에서 원본 관측 데이터가 얼마나 오래되었는지가 추론 실행시간 자체보다 더 중요할 수 있다. 신선도 메트릭(Freshness Metric)은 AI 결과를 해당 결과의 근거가 되는 센서 정보가 실제 물리적으로 획득된 시점과 연결한다.

ROS2 트레이싱(ROS2 Tracing)은 소프트웨어 측 시간 동작에 대한 상세한 가시성을 제공할 수 있다. 트레이스포인트(Tracepoint)를 이용하면 추론 노드 주변의 콜백 스케줄링, 실행기(Executor) 동작, 메시지 발행, 메시지 수신, 통신 경로를 확인할 수 있다. 이러한 정보를 모델 런타임 타이밍(Model-Runtime Timing)과 결합하면 ROS2 스케줄링 지연과 가속기 지연을 구분하는 데 도움이 된다. 신경망 모델과 입력 크기가 변경되지 않았는데도 추론 성능이 달라지는 경우 이러한 상관 분석이 특히 유용하다.

모니터링에서는 지연시간과 자원 사용률(Resource Utilization)의 상관관계도 분석해야 한다. GPU 사용률, GPU 메모리 소비량, CPU 부하, 메모리 압력(Memory Pressure), 온도, 전력 상태, 네트워크 트래픽, 추론 큐 깊이는 무작위로 보일 수 있는 시간 변화의 원인을 설명할 수 있다. 예를 들어 지연시간 증가는 모델 계산 자체의 변화가 아니라 여러 모델 사이의 GPU 경합(GPU Contention), CPU 전처리 포화, 열 스로틀링(Thermal Throttling), 네트워크 혼잡(Network Congestion)으로 인해 발생할 수 있다.

전용 ROS2 모니터링 노드(Dedicated ROS2 Monitoring Node)를 사용하면 진단 기능을 추론 구현과 강하게 결합하지 않고도 타이밍 정보를 수집할 수 있다. 추론 노드는 모델 식별자, 타임스탬프, 큐 지연, 전처리 시간, 실행시간, 전체 지연시간을 포함하는 구조화된 지연시간 메트릭 또는 진단 메시지를 발행할 수 있다. 모니터링 노드는 여러 모델과 로봇의 값을 통합하면서 각 추론 노드가 본래의 계산 기능에 집중할 수 있도록 한다.

임계값(Threshold)과 데드라인 정책(Deadline Policy)은 수동적인 측정 기능을 능동적인 운용 감시(Operational Supervision)로 전환할 수 있다. 지연시간이 허용 범위를 초과하면 시스템은 경고를 발행하거나 추론 결과를 오래된 상태(Stale)로 표시하고, 지연된 출력을 거부하거나 요청 주기를 낮추며, 모델 설정을 변경하거나 폴백 경로(Fallback Path)를 활성화할 수 있다. 임계값은 단순한 명목상 벤치마크 성능이 아니라 지연이 물리 시스템에 미치는 결과를 반영하여 결정해야 한다.

적응형 동작(Adaptive Behavior)은 지연시간 피드백을 이용하여 시스템 응답성을 유지할 수 있다. 추론 파이프라인에 과부하가 발생하면 이미지 해상도를 낮추거나 센서 처리 주기를 감소시키고, 더 가벼운 모델을 선택하거나 배치 크기(Batch Size)를 변경하며, 추론을 다른 가속기로 이동하거나 안전 중요 워크로드(Safety-Critical Workload)에 우선순위를 부여할 수 있다. 이러한 적응은 성능 최적화가 다운스트림 로봇 구성요소가 기대하는 의미적 동작을 암묵적으로 변경하지 않도록 명시적인 정책에 따라 관리되어야 한다.

모델 핫 스와핑(Model Hot Swapping)은 대체 모델의 ROS2 인터페이스가 동일하더라도 런타임 특성이 변경될 수 있으므로 지연시간 모니터링과 통합해야 한다. 활성화하기 전에 후보 모델(Candidate Model)을 워밍업(Warm-Up) 및 검증 과정에서 벤치마크할 수 있다. 활성화 이후에는 해당 모델의 지연시간 분포를 운용 한계와 비교해야 한다. 과도한 추론 또는 큐 지연은 이전에 검증된 모델로 롤백(Rollback)하는 조건 중 하나가 될 수 있다.

분산 피지컬 AI 시스템(Distributed Physical AI System)은 엣지(Edge), 온프레미스(On-Premise), 네트워크 경계를 가로지르는 상관 분석이 필요하다. 센서 프레임이 로봇에서 생성되고 엣지 컴퓨터에서 전처리된 후 더 큰 가속기에서 처리되어 ROS2 결과로 다시 반환될 수 있다. 모니터링에서는 이러한 단계 전체에 걸쳐 요청 식별자(Request Identifier) 또는 추적 식별자(Trace Identifier)를 유지하여 엔지니어가 서로 관련 없는 로컬 타이밍 측정값을 개별적으로 분석하는 대신 하나의 종단간 트랜잭션(End-to-End Transaction)을 재구성할 수 있도록 해야 한다.

모든 타이밍 이벤트를 최대 상세 수준으로 로깅하면 특히 고주기 인지 파이프라인에서 로깅 자체가 오버헤드(Overhead)를 발생시킬 수 있다. 따라서 모니터링 설계는 샘플링(Sampling), 집계(Aggregation), 설정 가능한 추적 수준(Configurable Trace Level), 이벤트 기반 상세 기록(Event-Triggered Detailed Recording)을 통해 진단 해상도와 런타임 비용 사이의 균형을 유지해야 한다. 정상 운용에서는 간결한 통계만 유지하고 비정상적인 지연 상태가 발생하면 보다 상세한 트레이싱을 활성화하여 추론 파이프라인에 지속적인 부담을 주지 않으면서 문제를 분석할 수 있다.

시각화(Visualization)와 이력 분석(Historical Analysis)은 원시 타이밍 측정값을 엔지니어링 근거(Engineering Evidence)로 변환한다. 대시보드는 시간에 따른 지연시간 분포, 데드라인 미스, 큐 깊이, 처리량(Throughput), 모델 버전, 하드웨어 사용률을 표시할 수 있다. 저장된 측정값은 소프트웨어, 모델, 드라이버, ROS2 설정 또는 하드웨어 변경 이후의 회귀 분석(Regression Analysis)에도 활용할 수 있다. 이를 통해 지연시간 모니터링은 일회성 성능 최적화가 아니라 지속적인 검증(Continuous Validation)의 일부가 된다.

통합 지연시간 모니터링 아키텍처(Integrated Latency-Monitoring Architecture)는 동기화된 타임스탬프, ROS2 트레이싱, 가속기 측정값, 큐 통계, 자원 텔레메트리(Resource Telemetry), 데드라인 정책, 모델 식별 정보를 하나의 관측 가능한 추론 경로(Observable Inference Path)로 연결한다. 이를 통해 엔지니어는 모델의 계산 시간이 얼마나 긴지만이 아니라 로봇이 결과를 수신하는 시점에 해당 정보가 얼마나 오래되었는지도 판단할 수 있다. 이러한 구분은 시간 특성이 인지, 추론, 계획, 물리적 행동의 유효성에 직접적인 영향을 미치는 신뢰성 높은 피지컬 AI(Dependable Physical AI)를 구현하는 데 필수적이다.

## 10.09 Physical AI Data Logging: MCAP Format [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

피지컬 AI(Physical AI) 시스템은 로봇이 무엇을 인식하고, 추론하고, 결정하고, 실행했는지를 엔지니어가 재구성할 수 있도록 서로 다른 유형의 데이터를 함께 보존해야 한다. ROS2 토픽(Topic)에는 카메라 이미지, LiDAR 포인트 클라우드(Point Cloud), 좌표 변환(Transform), 위치 추정(Localization), 관절 상태(Joint State), AI 추론 출력, 명령(Command), 진단(Diagnostics), 시간 정보(Timing Information)가 포함될 수 있다. MCAP은 이러한 동기화된 다중 모달 스트림(Multimodal Stream)을 재사용 가능한 로봇 데이터셋으로 기록하는 데 적합한 구조화된 로깅 컨테이너(Logging Container)를 제공한다.

MCAP은 단순히 센서 메시지를 저장하기 위한 형식 이상으로 이해해야 한다. 피지컬 AI 아키텍처(Physical AI Architecture)에서 로그(Log)는 관측(Observation)을 모델 출력(Model Output) 및 물리적 행동(Physical Action)과 연결하는 증거 기록(Evidence Record)이 된다. 따라서 유용한 기록에는 원시 센서 데이터뿐만 아니라 파생된 인식 결과(Derived Perception), 모델 식별 정보(Model Identity), 로봇 상태(Robot State), 작업 문맥(Task Context), 계획 결과(Planning Result), 제어 명령(Control Command), 안전 이벤트(Safety Event), 그리고 로봇이 특정 방식으로 동작한 이유를 설명하는 데 필요한 시스템 진단 정보가 포함되어야 한다.

ROS2 통합(ROS2 Integration)을 이용하면 MCAP 기록 기능을 기존 통신 그래프(Communication Graph)에 밀접하게 배치할 수 있다. 기록 대상으로 선택된 토픽은 ROS2 메시지 스키마(Message Schema)와 타임스탬프(Timestamp)를 유지할 수 있으므로 나중에 재생(Playback)하고 분석할 때에도 기록된 정보를 해석할 수 있다. 이를 통해 각 서브시스템마다 별도의 로깅 인터페이스(Logging Interface)를 생성할 필요가 줄어들며, 센싱(Sensing), AI 추론, 계획(Planning), 제어(Control), 모니터링(Monitoring) 컴포넌트가 기존 ROS2 통신 경로를 통해 데이터를 제공할 수 있다.

피지컬 AI 데이터는 물리 세계에서 발생하는 이벤트 사이의 관계를 표현하기 때문에 타임스탬프 무결성(Timestamp Integrity)이 매우 중요하다. 카메라 프레임, LiDAR 스캔, IMU 측정값, 좌표 변환, AI 출력, 모션 명령(Motion Command)은 시간적으로 상호 연관되어야 한다. 로그는 가능한 경우 소스 타임스탬프(Source Timestamp)를 보존하고 이를 기록 시간(Recording Time) 또는 수신 시간(Reception Time)과 구분해야 한다. 여러 컴퓨터가 참여하는 경우 동기화된 클록(Synchronized Clock)은 의미 있는 재구성에 필요한 공통 시간 기준(Common Temporal Reference)을 제공한다.

센서 출처(Sensor Provenance) 역시 식별 가능한 상태로 유지되어야 한다. 기록된 이미지는 해당 이미지를 생성한 카메라와 보정 정보(Calibration), 좌표계(Coordinate Frame)에 연결되어야 하며, 포인트 클라우드는 관련 LiDAR 좌표계와 시간 정보를 유지해야 한다. 좌표 변환 데이터는 이러한 관측을 로봇 좌표계(Robot Frame), 오도메트리 좌표계(Odometry Frame), 맵 좌표계(Map Frame)와 연결한다. 이러한 문맥을 보존하면 기록된 센서 값이 물리적 의미를 신뢰성 있게 복원할 수 없는 고립된 배열(Array)로 변하는 것을 방지할 수 있다.

AI 추론 결과(Inference Result)는 기존 센서 기록보다 추가적인 메타데이터(Metadata)를 필요로 한다. 각 결과에는 해당 결과를 생성한 모델 또는 배포 버전(Deployment Version)이 식별되어야 하며, 가능한 경우 결과가 파생된 관측 또는 요청(Request)도 연결되어야 한다. 신뢰도(Confidence), 추론 타임스탬프(Inference Timestamp), 지연시간(Latency), 전처리 프로파일(Preprocessing Profile), 런타임 설정(Runtime Configuration), 출력 의미(Output Semantics)도 기록할 수 있다. 이러한 필드는 배포 이후 모델 동작을 비교하고 추론 조건을 재현할 수 있도록 한다.

모델 입력과 출력을 기록하면 데이터셋 개발(Dataset Development)을 위한 중요한 기반을 구축할 수 있다. 엔지니어는 인식이 성공하거나 실패한 사례를 추출하고, 어려운 환경 조건을 식별하며, 새로운 학습 또는 평가 데이터 서브셋(Subset)을 생성할 수 있다. 현장 운용(Field Operation)과 AI 개발을 서로 분리된 활동으로 취급하는 대신 MCAP 기록을 통해 로봇 운용이 모델 분석, 재학습(Retraining), 검증(Validation), 향후 배포를 위한 근거를 지속적으로 생성하는 데이터 루프(Data Loop)를 구성할 수 있다.

고해상도 카메라와 고밀도 LiDAR 스트림은 매우 큰 로그를 생성할 수 있으므로 데이터 용량(Data Volume)을 신중하게 관리해야 한다. 모든 토픽을 최대 주기로 기록하면 저장장치 대역폭(Storage Bandwidth)을 초과하거나 불필요하게 큰 아카이브를 생성할 수 있다. 로깅 정책(Logging Policy)은 필수 토픽을 선택하고 중요도가 낮은 스트림의 기록 주기를 줄이며 적절한 압축(Compression)을 적용하고, 기록을 관리 가능한 파일로 분할하며, 각 데이터 범주의 가치에 따라 보존 정책(Retention Rule)을 적용할 수 있다.

압축은 저장 용량 감소(Storage Reduction), 연산 오버헤드(Computational Overhead), 향후 데이터 접근성(Future Accessibility) 사이의 균형을 고려해야 한다. 대규모 이미지와 포인트 클라우드 스트림이 일반적으로 기록 용량의 대부분을 차지하는 반면 상태, 명령, 진단 메시지는 상대적으로 작다. 따라서 압축 전략(Compression Strategy)은 사용 가능한 CPU 자원, 디스크 처리량(Disk Throughput), 기록 주기, 향후 분석 요구사항을 고려해야 한다. 로깅 작업이 실시간 로봇 운용(Real-Time Robot Operation)을 방해할 정도로 많은 연산 자원을 소비해서는 안 된다.

청킹(Chunking)과 파일 순환(File Rotation)은 장시간 임무에서 운용 강건성(Operational Robustness)을 향상시킨다. 하나의 매우 큰 기록 파일을 생성하는 대신 시간, 크기, 임무 단계(Mission Phase), 로봇 식별자(Robot Identifier), 운용 이벤트(Operational Event)를 기준으로 데이터를 분할할 수 있다. 작은 세그먼트(Segment)는 전송, 인덱싱(Indexing), 보존, 기록 중단 시 복구를 단순화한다. 매니페스트(Manifest)를 이용해 이러한 세그먼트를 임무 메타데이터와 연결하면 전체 운용 시퀀스(Operational Sequence)를 계속 검색하고 추적할 수 있다.

이벤트 트리거 기록(Event-Triggered Recording)은 연속 로깅(Continuous Logging)을 보완할 수 있다. 로봇은 최근 관측을 롤링 버퍼(Rolling Buffer)에 유지하다가 장애물 조우(Obstacle Encounter), 추론 실패(Inference Failure), 안전 개입(Safety Intervention), 위치 추정 이상(Localization Anomaly), 운영자 오버라이드(Operator Override), 예상하지 못한 움직임과 같은 중요한 조건이 발생하면 상세 데이터를 영구 저장할 수 있다. 이러한 접근법은 모든 고대역폭 스트림을 무기한 저장하지 않고도 드물게 발생하는 중요한 이벤트 주변의 고가치 문맥을 보존한다.

피지컬 AI 로그에는 성공적인 동작과 실패한 동작을 모두 포함해야 한다. 정상 운용만 포함하는 데이터셋으로는 장애를 진단하거나 강건성(Robustness)을 향상시키기에 충분하지 않다. 거부된 AI 제안(Rejected AI Proposal), 플래너 실패(Planner Failure), 오래된 추론 결과(Stale Inference Result), 비상 정지(Emergency Stop), 성능이 저하된 센서(Degraded Sensor), 타임아웃 이벤트(Timeout Event), 폴백 동작(Fallback Action)은 특히 가치 있는 학습 및 검증 사례를 제공할 수 있다. 거부 이유를 기록하는 것은 원래 모델 출력을 기록하는 것만큼 중요할 수 있다.

MCAP 재생(MCAP Playback)을 사용하면 기록된 로봇 운용 데이터를 개발 및 시험에 다시 활용할 수 있다. 센서 스트림을 인식 및 AI 노드에 재생하여 서로 다른 알고리즘이 동일한 관측 데이터를 처리하도록 할 수 있다. 엔지니어는 물리적 임무를 다시 수행하지 않고도 모델 버전을 비교하고, 결함(Defect)을 재현하며, 파라미터 변경(Parameter Change)을 평가하고, 하위 시스템 동작(Downstream Behavior)을 시험할 수 있다. 따라서 재생 기능은 현장 데이터를 반복 가능한 소프트웨어 통합(Software Integration) 및 회귀 시험(Regression Testing) 자원으로 변환한다.

결정론적 재현(Deterministic Reproduction)을 위해서는 메시지 내용 이외의 정보도 고려해야 한다. 소프트웨어 버전(Software Version), 모델 버전(Model Version), 파라미터(Parameter), 보정 정보(Calibration), 좌표계, 런타임 설정, 관련 환경 정보(Environment Information)를 기록과 연결해야 한다. 비동기 스케줄링(Asynchronous Scheduling)이나 하드웨어 동작으로 인해 완전히 동일한 실행 결과를 얻지 못할 수 있지만, 포괄적인 메타데이터는 결과가 생성된 조건을 이해하고 재현하는 능력을 크게 향상시킨다.

로깅은 AI 추론 지연시간 모니터링(AI Inference Latency Monitoring)과 통합되어야 한다. 기록에는 관측 타임스탬프, 큐 지연시간(Queue Delay), 전처리 시간(Preprocessing Time), 모델 실행 시간(Model Execution Time), 후처리 시간(Postprocessing Time), 발행 시간(Publication Time)을 추론 결과와 함께 보존할 수 있다. 오프라인 분석(Offline Analysis) 과정에서 엔지니어는 잘못된 로봇 응답이 모델 추론 자체의 문제에서 발생했는지, 아니면 올바른 결과가 너무 늦게 도착하여 변화하는 환경에서 더 이상 유효하지 않았기 때문인지 판단할 수 있다.

모델 핫 스왑(Model Hot Swap) 이벤트 역시 운용 로그(Operational Log)에 포함되어야 한다. 등록(Registration), 후보 모델 로딩(Candidate Loading), 검증, 활성화, 롤백(Rollback), 폐기(Retirement) 이벤트를 기록하면 각 시간 구간에 어떤 모델이 활성화되어 있었는지를 확인할 수 있다. 하나의 임무 중 여러 모델 버전이 사용되는 경우 이러한 타임라인(Timeline)을 통해 모든 추론 결과를 정확한 런타임 설정과 연결하고 서로 다른 배포 버전의 성능 측정값이 잘못 통합되는 것을 방지할 수 있다.

보안 및 개인정보 보호 정책(Security and Privacy Policy)은 기록된 정보의 전체 수명주기(Lifecycle)를 관리해야 한다. 피지컬 AI 로그에는 사람의 이미지, 시설 배치(Facility Layout), 운용 명령(Operational Command), 위치 정보(Location Information), 독점적인 산업 공정(Proprietary Industrial Process)이 포함될 수 있다. 따라서 접근 제어(Access Control), 필요한 경우의 암호화(Encryption), 무결성 검증(Integrity Verification), 보존 기간(Retention Period), 통제된 데이터 반출 절차(Controlled Export Procedure)를 데이터 수집 이후에 추가하는 것이 아니라 로깅 아키텍처 자체의 일부로 고려해야 한다.

기록 장치(Recorder) 자체의 관측 가능성(Observability)도 필요하다. 로깅 장애(Logging Failure)가 발생하면 이후 진단에 필요한 증거가 아무런 경고 없이 사라질 수 있기 때문이다. 시스템은 기록 상태(Recording Status), 저장장치 용량(Storage Capacity), 쓰기 처리량(Write Throughput), 누락 메시지(Dropped Message), 압축 부하(Compression Load), 파일 순환 상태, 기록 오류(Recording Error)를 모니터링해야 한다. 진단 정보는 저장 공간 고갈(Storage Exhaustion)이나 성능 저하로 중요한 운용 데이터가 손실되기 전에 상위 감독 소프트웨어(Supervisory Software)에 경고할 수 있다.

확장 가능한 배포(Scalable Deployment)에서는 일관된 명명 규칙(Naming Convention), 메타데이터, 매니페스트, 데이터셋 레지스트리(Dataset Registry)를 통해 MCAP 기록을 체계적으로 구성할 수 있다. 로봇 식별자, 임무 식별자(Mission Identifier), 날짜, 환경(Environment), 소프트웨어 릴리스(Software Release), 모델 버전, 센서 구성, 이벤트 태그(Event Tag)는 대규모 아카이브에서 검색 가능한 기준을 제공한다. 이러한 구성은 다수의 로봇에서 수집된 데이터를 플릿 분석(Fleet Analytics), 집단 학습(Collective Learning), 시뮬레이션(Simulation), 파운데이션 모델 개발(Foundation-Model Development)에 활용할수록 더욱 중요해진다.

결과적으로 MCAP 기반 로깅 아키텍처(MCAP-Based Logging Architecture)는 로봇 운용과 피지컬 AI 개발 사이에 지속적인 연결 고리(Persistent Bridge)를 제공한다. 동기화된 센서 관측, 로봇 상태, AI 추론, 모델 출처, 계획 결정(Planning Decision), 명령, 안전 이벤트, 진단, 시간 정보를 함께 보존함으로써 체화된 행동(Embodied Behavior)을 재구성할 수 있는 기록을 생성한다. 이러한 기록은 디버깅, 재생, 벤치마킹(Benchmarking), 데이터셋 큐레이션(Dataset Curation), 모델 개선(Model Improvement), 검증, 그리고 피지컬 AI 전체 수명주기에 걸친 지속 학습(Continuous Learning)을 지원한다.

## 10.10 Robotics Physical AI ROS2 Integration Architecture

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

힐스 로보틱스 피지컬 AI ROS2 통합 아키텍처(Hills Robotics Physical AI ROS2 Integration Architecture)는 물리적 센싱(Physical Sensing), 실시간 로봇 기능(Real-Time Robot Function), AI 추론(AI Inference), 임무 수준 지능(Mission-Level Intelligence), 플릿 운용(Fleet Operation)을 연결하는 계층형 로봇 지능 프레임워크(Layered Robotic Intelligence Framework)로 구성할 수 있다. ROS2는 이러한 계층 전반에서 공통 소프트웨어 통신 기반을 제공하며, 인식(Perception), 내비게이션(Navigation), 조작(Manipulation), 학습 기반 지능(Learned Intelligence), 안전(Safety), 시스템 관리(System Management)가 모듈성을 유지하면서 하나의 통합된 피지컬 AI 실행 환경에 참여할 수 있도록 한다.

물리 계층(Physical Layer)에서 서로 다른 유형의 로봇은 카메라(Camera), LiDAR, 깊이 센서(Depth Sensor), 열화상 센서(Thermal Sensor), GNSS/RTK, IMU, 관절 센서(Joint Sensor), 기타 플랫폼별 장치(Platform-Specific Device)를 통해 환경과 상호작용한다. 모바일 로봇(Mobile Robot), 매니퓰레이터(Manipulator), 보행 플랫폼(Legged Platform), 향후의 항공 시스템(Aerial System)은 서로 다른 하드웨어 인터페이스를 제공할 수 있지만, ROS2 디바이스 노드(Device Node)는 관측과 상태를 표준화된 통신 경로로 정규화하여 상위 소프트웨어 계층이 하드웨어에 직접 종속되지 않고 이를 사용할 수 있도록 한다.

로봇 제어 계층(Robot-Control Layer)은 AI 모델의 변경 여부와 관계없이 신뢰성 있게 유지되어야 하는 결정론적 기능(Deterministic Function)을 포함한다. 위치 추정(Localization), 매핑(Mapping), 내비게이션, 궤적 생성(Trajectory Generation), 기구학(Kinematics), 액추에이터 제어(Actuator Control), 차량 제어(Vehicle Control), 안전 관련 감독(Safety-Related Supervision)이 주로 이 계층에 속한다. AI 컴포넌트는 의미 정보(Semantic Information)나 모션 목표(Motion Objective)를 제공할 수 있지만, 기존 플래너(Planner)와 제어기(Controller)가 이러한 목표를 물리적으로 실행 가능한 궤적과 폐루프 실행(Closed-Loop Execution)으로 변환하는 책임을 유지한다.

ROS2 미들웨어(ROS2 Middleware)는 센싱, 제어, 지능 사이의 통신 백본(Communication Backbone)을 형성한다. 토픽(Topic)은 연속적인 센서 및 상태 스트림을 지원하고, 서비스(Service)는 제한된 요청-응답 작업(Request-Response Operation)을 제공하며, 액션(Action)은 피드백(Feedback), 취소(Cancellation), 선점(Preemption)이 필요한 장시간 작업을 지원한다. 파라미터(Parameter)와 수명주기 메커니즘(Lifecycle Mechanism)은 설정 및 운용 상태 관리를 제공하며, DDS 통신은 분산된 노드가 로봇 컴퓨터와 지원 엣지 시스템(Edge System)에 걸쳐 동작할 수 있도록 한다.

인식 계층(Perception Layer)은 원시 물리 관측(Raw Physical Observation)을 구조화된 환경 정보(Structured Environmental Information)로 변환한다. 카메라 및 LiDAR 파이프라인은 객체 탐지(Object Detection), 세그멘테이션(Segmentation), 추적(Tracking), 3D 인식(3D Perception), 자유 공간 추정(Free-Space Estimation), 의미 지도 작성(Semantic Mapping), 환경 이해(Environmental Understanding)를 수행할 수 있다. 기존 기하학적 알고리즘(Geometric Algorithm)과 학습 기반 인식 모델(Learned Perception Model)은 ROS2 인터페이스 뒤에서 공존할 수 있으며, 하위 컴포넌트는 내부 추론 구현에 종속되지 않고 객체, 자세(Pose), 지도(Map), 임베딩(Embedding), 의미 정보를 사용할 수 있다.

피지컬 AI 추론 계층(Physical AI Inference Layer)은 다중 모달 지능(Multimodal Intelligence)과 학습 기반 지능을 통해 이러한 인식 아키텍처를 확장한다. 비전-언어-행동 모델(Vision-Language-Action Model), 학습된 정책(Learned Policy), 월드 모델(World Model), 의미 추론 모델(Semantic Reasoning Model), 작업 지향 추론 서비스(Task-Oriented Inference Service)는 센서 관측을 언어 명령(Language Instruction) 및 로봇 문맥(Robot Context)과 함께 해석할 수 있다. PyTorch, ONNX, TensorRT, 가속기 전용 구현(Accelerator-Specific Implementation), 향후의 추론 엔진이 독립적으로 발전할 수 있도록 모델 런타임(Model Runtime)은 안정적인 ROS2 인터페이스 뒤에 캡슐화되어야 한다.

AI 출력은 물리적 움직임에 영향을 주기 전에 통제된 의사결정 경계(Controlled Decision Boundary)를 통과해야 한다. 추론 결과는 행동 제안(Action Proposal), 목표 자세(Target Pose), 내비게이션 목표(Navigation Goal), 조작 목표(Manipulation Target), 스킬 요청(Skill Request), 궤적 목표(Trajectory Objective)로 정규화할 수 있다. 이후 검증 과정에서는 신뢰도(Confidence), 타임스탬프(Timestamp), 좌표계(Coordinate Frame), 로봇 기능(Robot Capability), 작업 공간 제약(Workspace Constraint), 운용 상태(Operational State)를 확인한다. 이를 통해 문서화되지 않은 신경망 출력을 액추에이터 명령으로 직접 취급하는 것을 방지한다.

모션 실행(Motion Execution)은 계획 및 제어 인터페이스(Planning and Control Interface)를 통해 AI 추론과 분리된 상태로 유지된다. 내비게이션 목표는 모바일 로봇 플래너(Mobile-Robot Planner), 조작 목표는 모션 계획 시스템(Motion-Planning System), 플랫폼별 기준값(Platform-Specific Reference)은 적절한 궤적 제어기(Trajectory Controller)로 전달할 수 있다. AI 추론 이후에도 충돌 검사(Collision Checking), 기구학적 실행 가능성(Kinematic Feasibility), 속도 및 가속도 제약(Velocity and Acceleration Constraint), 지오펜싱(Geofencing), 독립적인 안전 감독(Independent Safety Supervision)이 계속 적용되어 학습 기반 행동 주위에 결정론적 실행 경계(Deterministic Execution Boundary)를 유지한다.

임무 수준 오케스트레이션(Mission-Level Orchestration)은 개별 지능 기능을 완전한 로봇 작업으로 조정한다. 임무 관리자(Mission Manager)는 작업 상태(Task State)에 따라 내비게이션, 점검(Inspection), 조작, 인식, VLA 추론(VLA Reasoning), 복구 행동(Recovery Behavior), 운영자 상호작용(Operator Interaction)을 결합할 수 있다. 개별 AI 노드 내부에 전체 임무 로직을 포함시키는 대신 오케스트레이션은 명시적인 작업 진행(Task Progression)을 유지하고 특정 기능을 언제 호출, 취소, 재시도, 교체 또는 상위 시스템으로 전달할지를 결정한다.

동일한 아키텍처는 단일 로봇에서 다중 로봇 운용(Multi-Robot Operation)으로 확장될 수 있다. 플릿(Fleet) 및 RMS 컴포넌트는 임무를 분배하고, 로봇 상태를 모니터링하며, 교통을 조정하고, 충전(Charging)을 관리하며, 운용 가용성(Operational Availability)을 감독할 수 있다. 상위 수준 지능은 여러 로봇에서 수집된 정보를 이용하여 공유 지도(Shared Map), 분산 관측(Distributed Observation), 작업량 할당(Workload Allocation), 집단 의사결정 지원(Collective Decision Support)을 제공할 수 있으며, 각 로봇은 즉각적인 물리적 실행에 필요한 로컬 제어(Local Control)와 안전 기능을 유지한다.

모델 관리(Model Management)는 개발 단계에만 필요한 기능이 아니라 운용 서브시스템(Operational Subsystem)으로 취급해야 한다. 모델 레지스트리(Model Registry)는 모델 식별 정보, 버전, 입출력 계약(Input and Output Contract), 런타임 요구사항(Runtime Requirement), 가속기 호환성(Accelerator Compatibility), 배포 상태(Deployment Status)를 추적할 수 있다. 런타임 핫 스왑(Runtime Hot Swap)을 이용하면 안정적인 ROS2 추론 인터페이스를 계속 운용하고 관련 없는 로봇 기능을 유지하면서 후보 모델을 로딩하고, 검증하고, 워밍업(Warm-Up)하고, 활성화하고, 모니터링하며, 필요할 경우 롤백(Rollback)할 수 있다.

지연시간 모니터링(Latency Monitoring)은 AI 결과가 물리적 행동에 여전히 유효한지를 판단하기 위한 시간적 근거(Temporal Evidence)를 제공한다. 센서 획득 시간(Sensor Acquisition Time), ROS2 전송(ROS2 Transport), 동기화(Synchronization), 전처리(Preprocessing), 큐 지연(Queue Delay), GPU 실행(GPU Execution), 후처리(Postprocessing), 발행(Publication)을 각각 독립적인 단계로 측정할 수 있다. PTP 또는 다른 공유 시간 인프라(Shared Time Infrastructure)를 사용하면 분산 컴퓨터 사이의 이벤트를 상호 연관시켜 모델 추론 시간만이 아니라 종단간 데이터 최신성(End-to-End Data Freshness)을 평가할 수 있다.

관측 가능성(Observability)은 ROS2 동작, AI 런타임 성능(AI Runtime Performance), 로봇 실행을 연결해야 한다. 측정 항목에는 메시지 주기(Message Rate), 콜백 지연시간(Callback Latency), 추론 지연시간(Inference Latency), 큐 깊이(Queue Depth), GPU 및 CPU 사용률(Utilization), 메모리 사용량(Memory Consumption), 모델 식별 정보, 거부된 AI 출력(Rejected AI Output), 플래너 상태(Planner Status), 제어기 상태(Controller State), 안전 이벤트가 포함될 수 있다. 통합 진단(Unified Diagnostics)을 통해 엔지니어는 성능 저하가 센서, 미들웨어, AI 모델, 컴퓨팅 자원, 계획, 통신 또는 물리적 실행 중 어디에서 발생했는지를 판단할 수 있다.

MCAP 기반 로깅(MCAP-Based Logging)은 이러한 계층 전체에 걸쳐 지속적인 운용 기록(Persistent Operational Record)을 제공한다. 센서 관측, 좌표 변환(Transform), 로봇 상태, AI 입력 및 출력, 모델 버전, 임무 결정(Mission Decision), 모션 명령(Motion Command), 진단, 지연시간 측정값, 안전 이벤트를 동기화된 시간 정보와 함께 기록할 수 있다. 재생(Playback)을 통해 동일한 관측 데이터를 디버깅(Debugging), 회귀 시험(Regression Testing), 모델 비교(Model Comparison), 데이터셋 큐레이션(Dataset Curation), 중요한 현장 이벤트 재구성(Field Event Reconstruction)에 다시 활용할 수 있다.

기록된 운용 데이터는 지속적인 피지컬 AI 개선 루프(Continuous Physical AI Improvement Loop)를 구축할 수 있다. 현장 로그(Field Log)는 성공적인 운용, 어려운 환경, 거부된 행동, 인식 실패(Perception Failure), 안전 개입(Safety Intervention), 비정상적인 로봇 행동에 대한 사례를 제공한다. 선택된 데이터는 학습 및 평가 데이터셋으로 큐레이션하고, 인식 또는 추론 모델 개선에 사용하며, 시뮬레이션(Simulation)과 재생을 통해 검증한 후 통제된 모델 관리 절차를 통해 다시 배포할 수 있다.

보안(Security)은 통신, 모델 배포(Model Deployment), 운용 명령(Operational Command), 기록 데이터 전반에 적용되어야 한다. ROS2 보안 메커니즘(Security Mechanism)은 노드 간 통신을 제한할 수 있으며, 모델 패키지는 활성화 전에 무결성(Integrity)과 권한(Authorization)을 검증해야 한다. 임무 명령과 AI 서비스는 허가된 컴포넌트만 접근할 수 있어야 하며, 운용 로그에는 민감한 환경 및 로봇 정보가 포함될 수 있으므로 통제된 접근(Controlled Access), 암호화(Encryption), 보존 정책(Retention Policy), 감사 가능성(Auditability)이 필요할 수 있다.

안전(Safety)은 전체 아키텍처에서 독립적인 권한(Independent Authority)으로 유지된다. AI 모델은 인식, 추론, 적응(Adaptation), 자율성(Autonomy)을 향상시킬 수 있지만 충돌 보호(Collision Protection), 액추에이터 제한(Actuator Limit), 비상 정지(Emergency Stop), 운용 제약(Operational Constraint), 플랫폼별 안전 메커니즘을 우회해서는 안 된다. AI 신뢰도가 감소하거나 데이터가 오래되고, 추론이 실패하거나 통신이 손실되는 경우 결정론적 폴백 동작(Deterministic Fallback Behavior)을 통해 로봇을 정의된 안전 운용 상태(Safe Operational Condition)로 유지하거나 전환해야 한다.

엣지 및 온프레미스 컴퓨팅(Edge and On-Premise Computing)에 걸친 배포는 동일한 논리적 ROS2 계약(Logical ROS2 Contract)을 유지하면서 지연시간과 연산 요구사항에 따라 워크로드(Workload)를 배치할 수 있어야 한다. 시간에 민감한 인식과 제어는 로봇 가까이에 유지할 수 있으며, 더 무거운 모델 적응(Model Adaptation), 플릿 분석(Fleet Analytics), 데이터셋 처리(Dataset Processing), 시뮬레이션, 집단 지능(Collective Intelligence)은 더 큰 컴퓨팅 자원을 활용할 수 있다. 명확한 인터페이스는 물리적 로봇 운용이 하나의 특정 컴퓨팅 위치에 종속되는 것을 방지한다.

결과적으로 힐스 로보틱스 아키텍처(Hills Robotics Architecture)는 ROS2 미들웨어, 결정론적 로봇 소프트웨어(Deterministic Robot Software), 피지컬 AI 추론, 모델 수명주기 관리(Model Lifecycle Management), 동기화된 시간 체계(Synchronized Timing), 관측 가능성, MCAP 데이터 로깅(Data Logging), 임무 오케스트레이션, 플릿 수준 지능(Fleet-Level Intelligence)을 하나의 확장 가능한 프레임워크(Extensible Framework)로 통합한다. 핵심 원칙은 통제된 분리(Controlled Separation)이며, 학습 기반 지능은 빠르게 발전할 수 있는 반면 안정적인 인터페이스, 계획, 제어, 안전, 운용 증거(Operational Evidence)는 점점 더 고도화되는 자율 로봇 시스템 전반에서 신뢰할 수 있는 물리적 실행을 유지한다.
