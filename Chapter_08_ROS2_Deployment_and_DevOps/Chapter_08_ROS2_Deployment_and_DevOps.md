**Volume 05 Robot Middleware and ROS2**


# 08. ROS2 Deployment and DevOps

##  

## 08.01 ROS2 Containerization: Dockerfile Best Practices [w/Code]

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

Containerization provides a controlled deployment boundary for ROS 2 software by packaging application binaries, runtime libraries, ROS distributions, and configuration dependencies into reproducible images. In robotics, this reduces differences between developer workstations, CI environments, simulation systems, edge computers, and production robots while preserving a consistent software baseline across deployments.

A ROS 2 Dockerfile should begin from the smallest trustworthy base image that still satisfies the application requirements. The selected image should explicitly match the intended ROS 2 distribution, operating-system release, and processor architecture. Production images should avoid unnecessary development utilities because every additional package increases image size, build time, dependency complexity, and the potential security attack surface.

Reproducibility requires dependency installation to be deterministic rather than dependent on the changing state of external repositories. Package versions should be constrained where practical, repository configuration should be explicit, and rosdep should resolve package dependencies from a well-defined workspace. Docker base images can also be pinned by digest when strict traceability is required for validated robot software releases.

Docker layer organization strongly affects build efficiency. Instructions that change infrequently, such as operating-system and ROS dependency installation, should appear before frequently modified application source code. Docker can then reuse cached dependency layers while rebuilding only the application-specific layers. This becomes particularly important for large ROS 2 workspaces containing perception, navigation, control, and AI packages.

Multi-stage builds provide an effective method for separating compilation from execution. A builder stage may contain compilers, colcon, development headers, testing utilities, and source code, while the final runtime stage receives only the installed ROS 2 artifacts and required runtime libraries. This approach produces smaller production images and prevents build tools and temporary files from unnecessarily becoming part of deployed robot software.

ROS 2 workspaces should normally be built using colcon inside the container so that compilation occurs against exactly the libraries used by the target runtime environment. The source workspace can be copied into a dedicated build stage, dependencies resolved through rosdep, and packages compiled with an appropriate build type. The resulting install directory then becomes the primary software artifact transferred into the runtime image.

Dockerfile instructions should minimize unnecessary filesystem state. Package-manager metadata and temporary files should be removed in the same layer in which they are created, and broad COPY operations should be avoided. A carefully designed .dockerignore file should exclude build, install, log, version-control, dataset, model-cache, and temporary directories unless they are explicitly required by the containerized application.

ROS 2 containers require deliberate handling of environment initialization. The container entrypoint should source the ROS distribution setup script and, when applicable, the workspace install/setup script before executing the requested process. This ensures that package paths, shared libraries, interface definitions, and ROS-specific environment variables are consistently available without requiring every command to repeat environment initialization logic.

Configuration should generally remain separate from immutable application binaries. Robot-specific parameters, calibration data, DDS configuration, security artifacts, maps, and deployment settings may be supplied through mounted volumes, secrets, or environment-specific configuration mechanisms. This separation allows the same tested image to operate across multiple robots without rebuilding an image simply because a robot identifier or deployment parameter changes.

ROS 2 networking requires additional consideration because DDS discovery and data transport interact with container networking. Multicast discovery, UDP traffic, interface selection, ROS_DOMAIN_ID, DDS implementation settings, and host networking behavior must be evaluated for the actual deployment environment. A container that operates correctly on a developer workstation may behave differently across robot computers, VLANs, Wi-Fi networks, or multi-robot systems.

Hardware access should follow the principle of minimum privilege. Cameras, LiDAR interfaces, serial devices, CAN adapters, GPUs, and other accelerators should expose only the devices and permissions required by the ROS 2 process. Running every robot container with privileged mode may simplify initial integration, but it weakens isolation and makes security auditing difficult. Explicit device mappings and narrowly defined Linux capabilities are preferable.

GPU-based ROS 2 workloads introduce another compatibility boundary involving the host driver, container runtime, CUDA libraries, and application framework. Perception and Physical AI containers should therefore separate hardware-dependent runtime requirements from application dependencies wherever possible. Image variants may be maintained for CPU, NVIDIA GPU, or Jetson targets while preserving common ROS 2 interfaces and application-level behavior.

Containers should not automatically imply that every ROS 2 node requires an independent image. Node composition, process boundaries, failure isolation, hardware ownership, communication overhead, update frequency, and lifecycle requirements should determine container boundaries. Closely coupled nodes may reasonably share a container, while perception, navigation, hardware interfaces, AI inference, diagnostics, and fleet gateways may require different deployment units.

Production images should execute ROS 2 applications as non-root users whenever hardware and operating requirements permit. File permissions, mounted volumes, device groups, Linux capabilities, read-only filesystems, and writable runtime directories should be designed explicitly. Container security should complement SROS2 and DDS security rather than replace them, because process isolation and communication authentication address different layers of the robot security architecture.

Health management should distinguish container availability from ROS 2 application readiness. A running container does not guarantee that DDS discovery succeeded, sensors initialized correctly, lifecycle nodes reached the active state, or required dependencies became available. Deployment systems should therefore combine container-level health checks with ROS 2 lifecycle state, diagnostics, topic activity, service readiness, and hardware status where operationally appropriate.

Logging should be designed for persistent operation rather than interactive development. ROS 2 console output, application logs, diagnostics, and container runtime logs need controlled retention and aggregation so that storage does not grow indefinitely on edge computers. Important operational information should remain accessible after container restart, while transient build files and debugging artifacts should not become permanent parts of production images.

Image tagging must support traceability. Mutable labels such as latest are convenient during development but insufficient for controlled fleet deployment. Production releases should identify the application version, ROS distribution, architecture, and relevant build revision through immutable tags, digests, or associated metadata. This allows operators to determine exactly which software image is running on each robot and enables reliable rollback after deployment failures.

CI/CD pipelines should build ROS 2 images from version-controlled Dockerfiles and verify them before fleet deployment. Typical validation can include dependency resolution, colcon compilation, unit tests, integration tests, security scanning, image metadata inspection, and simulation-based startup tests. The same validated artifact should progress through staging and production whenever possible instead of rebuilding nominally identical software at each deployment stage.

Containerization is therefore most effective when treated as part of the ROS 2 deployment architecture rather than merely as a packaging technique. A well-designed Dockerfile establishes reproducible dependencies, controlled runtime privileges, efficient builds, hardware-aware execution, observable health, and traceable releases. These properties create the foundation for subsequent Docker Compose, CI/CD, OTA, monitoring, and large-scale fleet deployment mechanisms.

컨테이너화(Containerization)는 애플리케이션 바이너리(Application Binary), 런타임 라이브러리(Runtime Library), ROS 배포판(ROS Distribution), 설정 의존성(Configuration Dependency)을 재현 가능한 이미지(Reproducible Image)로 패키징함으로써 ROS 2 소프트웨어에 통제된 배포 경계(Deployment Boundary)를 제공한다. 로보틱스(Robotics)에서는 개발 워크스테이션, CI 환경, 시뮬레이션 시스템, 엣지 컴퓨터(Edge Computer), 실제 운영 로봇 사이의 환경 차이를 줄이면서 일관된 소프트웨어 기준선(Software Baseline)을 유지할 수 있다.

ROS 2 도커파일(Dockerfile)은 애플리케이션 요구사항을 충족하면서 가능한 한 작고 신뢰할 수 있는 베이스 이미지(Base Image)에서 시작하는 것이 바람직하다. 선택한 이미지는 대상 ROS 2 배포판, 운영체제(OS) 릴리스, 프로세서 아키텍처(Processor Architecture)를 명확하게 일치시켜야 한다. 운영용 이미지(Production Image)에는 불필요한 개발 도구를 포함하지 않아야 하며, 추가 패키지는 이미지 크기와 빌드 시간뿐 아니라 의존성 복잡성과 잠재적인 보안 공격 표면(Security Attack Surface)을 증가시킨다.

재현성(Reproducibility)을 확보하려면 의존성 설치가 외부 저장소(External Repository)의 변화하는 상태에 따라 달라지지 않도록 결정적(Deterministic)으로 구성해야 한다. 가능한 경우 패키지 버전을 제한하고 저장소 설정을 명시적으로 관리하며, rosdep이 잘 정의된 워크스페이스(Workspace)를 기준으로 패키지 의존성을 해결하도록 해야 한다. 엄격한 추적성(Traceability)이 필요한 검증된 로봇 소프트웨어 릴리스에서는 도커 베이스 이미지를 다이제스트(Digest)로 고정할 수도 있다.

도커 레이어(Docker Layer)의 구성 순서는 빌드 효율에 큰 영향을 준다. 운영체제 및 ROS 의존성 설치처럼 자주 변경되지 않는 명령은 빈번하게 수정되는 애플리케이션 소스 코드보다 앞에 배치하는 것이 좋다. 그러면 도커(Docker)는 기존 의존성 레이어를 캐시(Cache)에서 재사용하고 애플리케이션 관련 레이어만 다시 빌드할 수 있다. 이러한 방식은 인지(Perception), 내비게이션(Navigation), 제어(Control), AI 패키지가 포함된 대규모 ROS 2 워크스페이스에서 특히 중요하다.

다단계 빌드(Multi-Stage Build)는 컴파일(Compilation) 환경과 실행(Runtime) 환경을 분리하는 효과적인 방법이다. 빌더 단계(Builder Stage)에는 컴파일러, colcon, 개발 헤더, 테스트 도구, 소스 코드를 포함할 수 있지만 최종 런타임 단계에는 설치된 ROS 2 산출물과 필요한 런타임 라이브러리만 전달한다. 이 방식은 운영 이미지의 크기를 줄이는 동시에 빌드 도구와 임시 파일이 실제 로봇에 배포되는 것을 방지한다.

ROS 2 워크스페이스는 일반적으로 컨테이너 내부에서 colcon으로 빌드하여 대상 런타임 환경과 정확히 동일한 라이브러리를 기준으로 컴파일되도록 구성하는 것이 좋다. 소스 워크스페이스를 전용 빌드 단계로 복사하고 rosdep으로 의존성을 해결한 다음 적절한 빌드 유형(Build Type)으로 패키지를 컴파일할 수 있다. 이렇게 생성된 설치 디렉터리(Install Directory)가 최종 런타임 이미지로 전달되는 핵심 소프트웨어 산출물이 된다.

도커파일 명령은 불필요한 파일시스템 상태(Filesystem State)를 최소화하도록 작성해야 한다. 패키지 관리자 메타데이터와 임시 파일은 생성된 동일 레이어에서 제거하는 것이 좋으며, 지나치게 광범위한 COPY 명령은 피해야 한다. 세밀하게 구성한 .dockerignore 파일을 사용하여 빌드, 설치, 로그, 버전 관리, 데이터셋, 모델 캐시(Model Cache), 임시 디렉터리를 제외하고 컨테이너 애플리케이션에 실제로 필요한 항목만 포함해야 한다.

ROS 2 컨테이너에서는 환경 초기화(Environment Initialization)를 명시적으로 처리해야 한다. 컨테이너 엔트리포인트(Entrypoint)는 요청된 프로세스를 실행하기 전에 ROS 배포판의 설정 스크립트(Setup Script)를 로드하고, 필요한 경우 워크스페이스의 install/setup 스크립트도 함께 로드해야 한다. 이를 통해 패키지 경로, 공유 라이브러리, 인터페이스 정의, ROS 관련 환경 변수를 일관되게 사용할 수 있으며 각 실행 명령에서 환경 초기화 과정을 반복할 필요가 없어진다.

설정(Configuration)은 일반적으로 변경 불가능한 애플리케이션 바이너리와 분리하는 것이 바람직하다. 로봇별 파라미터(Parameter), 캘리브레이션(Calibration) 데이터, DDS 설정, 보안 산출물(Security Artifact), 지도(Map), 배포 설정은 마운트 볼륨(Mounted Volume), 시크릿(Secret), 환경별 설정 메커니즘을 통해 제공할 수 있다. 이러한 분리를 적용하면 로봇 식별자나 배포 파라미터가 변경되었다는 이유만으로 이미지를 다시 빌드하지 않고 동일하게 검증된 이미지를 여러 로봇에서 사용할 수 있다.

ROS 2 네트워킹(Networking)은 DDS 검색(Discovery)과 데이터 전송이 컨테이너 네트워크와 상호작용하기 때문에 추가적인 고려가 필요하다. 멀티캐스트 검색(Multicast Discovery), UDP 트래픽, 네트워크 인터페이스 선택, ROS_DOMAIN_ID, DDS 구현체 설정, 호스트 네트워킹(Host Networking)의 동작을 실제 배포 환경에서 평가해야 한다. 개발자 워크스테이션에서 정상 동작한 컨테이너라도 로봇 컴퓨터, VLAN, Wi-Fi 네트워크, 다중 로봇 시스템에서는 다른 동작을 보일 수 있다.

하드웨어 접근(Hardware Access)은 최소 권한 원칙(Principle of Least Privilege)을 따라야 한다. 카메라, 라이다(LiDAR), 직렬 장치(Serial Device), CAN 어댑터, GPU 및 기타 가속기는 ROS 2 프로세스에 필요한 장치와 권한만 노출해야 한다. 모든 로봇 컨테이너를 특권 모드(Privileged Mode)로 실행하면 초기 통합은 간단해질 수 있지만 격리(Isolation)를 약화시키고 보안 감사를 어렵게 한다. 따라서 명시적인 장치 매핑(Device Mapping)과 제한적으로 정의된 리눅스 기능(Linux Capability)을 사용하는 것이 바람직하다.

GPU 기반 ROS 2 워크로드(Workload)는 호스트 드라이버(Host Driver), 컨테이너 런타임(Container Runtime), CUDA 라이브러리, 애플리케이션 프레임워크 사이에 또 하나의 호환성 경계(Compatibility Boundary)를 만든다. 따라서 인지 및 피지컬 AI(Physical AI) 컨테이너는 가능한 경우 하드웨어 의존적인 런타임 요구사항과 애플리케이션 의존성을 분리해야 한다. CPU, NVIDIA GPU, Jetson 대상별 이미지 변형(Image Variant)을 유지하면서 공통 ROS 2 인터페이스와 애플리케이션 수준의 동작을 유지할 수 있다.

컨테이너화를 적용한다고 해서 모든 ROS 2 노드(Node)가 각각 독립적인 이미지를 가져야 하는 것은 아니다. 노드 컴포지션(Node Composition), 프로세스 경계(Process Boundary), 장애 격리(Failure Isolation), 하드웨어 소유권, 통신 오버헤드, 업데이트 주기, 라이프사이클(Lifecycle) 요구사항을 기준으로 컨테이너 경계를 결정해야 한다. 밀접하게 결합된 노드는 하나의 컨테이너를 공유할 수 있지만 인지, 내비게이션, 하드웨어 인터페이스, AI 추론, 진단, 플릿 게이트웨이(Fleet Gateway)는 서로 다른 배포 단위가 필요할 수 있다.

운영용 이미지는 하드웨어 및 운영 요구사항이 허용하는 범위에서 ROS 2 애플리케이션을 비루트 사용자(Non-Root User)로 실행하는 것이 좋다. 파일 권한, 마운트 볼륨, 장치 그룹(Device Group), 리눅스 기능, 읽기 전용 파일시스템(Read-Only Filesystem), 쓰기 가능한 런타임 디렉터리를 명시적으로 설계해야 한다. 컨테이너 보안은 SROS2 및 DDS 보안을 대체하는 것이 아니라 보완해야 하며, 프로세스 격리와 통신 인증(Authentication)은 로봇 보안 아키텍처의 서로 다른 계층을 담당하기 때문이다.

상태 관리(Health Management)는 컨테이너의 실행 여부와 ROS 2 애플리케이션의 준비 상태(Readiness)를 구분해야 한다. 컨테이너가 실행 중이라는 사실만으로 DDS 검색이 성공하거나 센서가 정상 초기화되고 라이프사이클 노드가 활성 상태(Active State)에 도달했음을 보장하지 않는다. 따라서 배포 시스템은 필요에 따라 컨테이너 수준 상태 확인(Health Check)과 ROS 2 라이프사이클 상태, 진단(Diagnostics), 토픽 활동, 서비스 준비 상태, 하드웨어 상태를 함께 확인해야 한다.

로깅(Logging)은 대화형 개발 환경이 아니라 장기간 운영 환경을 기준으로 설계해야 한다. ROS 2 콘솔 출력, 애플리케이션 로그, 진단 정보, 컨테이너 런타임 로그에는 적절한 보존 및 집계(Aggregation) 정책이 필요하며, 이를 통해 엣지 컴퓨터의 저장공간이 무제한으로 증가하는 것을 방지해야 한다. 중요한 운영 정보는 컨테이너 재시작 이후에도 접근할 수 있어야 하지만 임시 빌드 파일이나 디버깅 산출물은 운영 이미지의 영구적인 구성 요소가 되어서는 안 된다.

이미지 태깅(Image Tagging)은 추적성을 지원하도록 설계해야 한다. latest와 같은 변경 가능한 태그(Mutable Tag)는 개발 단계에서는 편리하지만 통제된 플릿 배포(Fleet Deployment)에는 충분하지 않다. 운영 릴리스에서는 변경 불가능한 태그(Immutable Tag), 다이제스트 또는 관련 메타데이터를 이용하여 애플리케이션 버전, ROS 배포판, 아키텍처, 빌드 리비전(Build Revision)을 식별해야 한다. 이를 통해 각 로봇에서 정확히 어떤 소프트웨어 이미지가 실행되고 있는지 확인하고 배포 실패 시 안정적인 롤백(Rollback)을 수행할 수 있다.

CI/CD 파이프라인은 버전 관리되는 도커파일에서 ROS 2 이미지를 빌드하고 플릿에 배포하기 전에 이를 검증해야 한다. 일반적인 검증 과정에는 의존성 해결, colcon 컴파일, 단위 테스트(Unit Test), 통합 테스트(Integration Test), 보안 스캐닝(Security Scanning), 이미지 메타데이터 검사, 시뮬레이션 기반 시작 테스트 등이 포함될 수 있다. 가능하면 동일한 검증 산출물이 스테이징(Staging)에서 운영 환경까지 진행되어야 하며 각 배포 단계에서 명목상 동일한 소프트웨어를 다시 빌드하는 방식은 피하는 것이 좋다.

따라서 컨테이너화는 단순한 패키징 기법이 아니라 ROS 2 배포 아키텍처(Deployment Architecture)의 일부로 다룰 때 가장 효과적이다. 잘 설계된 도커파일은 재현 가능한 의존성, 통제된 런타임 권한, 효율적인 빌드, 하드웨어 인식 실행(Hardware-Aware Execution), 관찰 가능한 상태(Observable Health), 추적 가능한 릴리스를 확립한다. 이러한 특성은 이후 도커 컴포즈(Docker Compose), CI/CD, 무선 업데이트(OTA), 모니터링(Monitoring), 대규모 플릿 배포로 확장하기 위한 기반을 형성한다.

##  

## 08.02 Docker Compose-Based Robot SW Stack Deployment [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

Docker Compose provides a declarative method for deploying a complete ROS 2 robot software stack as a coordinated collection of containers. Instead of manually starting perception, navigation, control, diagnostics, and application processes, a Compose file describes their images, runtime parameters, networks, volumes, devices, and dependencies. This converts robot deployment into a version-controlled system configuration that can be reproduced across development and production platforms.

A robot software stack should be divided into services according to operational responsibility rather than simply assigning one container to every ROS 2 node. Typical boundaries may separate hardware interfaces, perception pipelines, localization, navigation, AI inference, diagnostics, and external gateways. Closely coupled ROS 2 components can remain within one service when composition improves performance, while independently updated or failure-sensitive functions benefit from separate containers.

The Compose YAML file becomes the deployment manifest describing how these services operate together. Each service can reference a prebuilt image or Dockerfile, define its command and entrypoint, specify environment variables, attach volumes, expose hardware devices, and configure restart behavior. Keeping this configuration in version control allows the deployed topology to evolve together with ROS 2 packages while preserving a traceable history of system-level changes.

Image versions should be explicitly controlled because a multi-container robot depends on compatibility among several software components. Mutable tags such as latest can cause different robots to obtain different implementations even when they use the same Compose file. Versioned tags or immutable image digests provide stronger reproducibility and make it possible to reconstruct a fleet configuration, verify a release, and roll back individual services when deployment problems occur.

ROS 2 communication introduces special networking requirements because DDS discovery and transport must operate across service boundaries. Compose networks can isolate application groups, but multicast, UDP communication, interface selection, ROS_DOMAIN_ID, and DDS configuration must remain compatible with the intended topology. Host networking can simplify some robot deployments, although it reduces network isolation and should therefore be selected according to operational requirements rather than used automatically.

Multi-robot systems require deliberate separation of communication domains and naming spaces. Environment variables and external configuration can assign ROS_DOMAIN_ID values, robot namespaces, DDS profiles, and unique robot identifiers without modifying container images. This allows a common software image set to be deployed across many robots while Compose configuration provides robot-specific identity and communication behavior appropriate to each physical platform.

Hardware-facing services require explicit access to physical resources. Compose can map cameras, serial ports, CAN interfaces, LiDAR devices, USB peripherals, GPUs, and other accelerators into selected containers rather than exposing all host hardware to the complete software stack. Device mappings, group permissions, runtime configuration, and Linux capabilities should follow minimum-privilege principles so that a failure in one service does not unnecessarily compromise other robot functions.

Persistent and configurable data should be separated from immutable container images through volumes and bind mounts. Calibration files, maps, ROS 2 parameter files, DDS profiles, security material, AI models, and operational data can therefore survive container replacement or be changed independently from application binaries. Writable directories should be deliberately defined because unrestricted persistence makes deployment behavior difficult to reproduce and complicates rollback between software versions.

Service dependencies require more than specifying a simple startup order. A container may have started while its ROS 2 node is still initializing, waiting for hardware, discovering DDS participants, loading a map, or transitioning through lifecycle states. Consequently, dependency management should combine Compose startup relationships with health checks, ROS 2 lifecycle orchestration, application readiness, and retry mechanisms instead of assuming that a running container represents a ready subsystem.

Health checks provide a bridge between container supervision and robot application state. Basic checks may verify that a process remains alive, while stronger checks can confirm expected services, device availability, application endpoints, or ROS 2 subsystem readiness. Safety-critical decisions should not depend solely on Docker health status, however, because container supervision does not replace dedicated robot diagnostics, safety controllers, watchdogs, or lifecycle-based safe-state mechanisms.

Restart policies can improve availability when processes terminate unexpectedly, but automatic restart must be designed according to subsystem behavior. Restarting a visualization or telemetry service may be straightforward, whereas repeatedly restarting a hardware driver or motion-related component could create undesirable state transitions. Recovery policies should therefore coordinate restart limits, lifecycle initialization, hardware state, dependency recovery, and escalation to a controlled robot safe state when automatic recovery fails.

Environment-specific configuration should be separated from the primary Compose definition whenever practical. A common base configuration can describe the standard ROS 2 stack, while environment files or Compose overrides adapt it for development computers, simulation, laboratory robots, GPU platforms, and production systems. This prevents duplicated deployment manifests from gradually diverging and supports a consistent architecture across heterogeneous robot computing environments.

Resource management becomes increasingly important when perception and Physical AI workloads share an edge computer with navigation and control services. CPU allocation, memory limits, GPU access, shared-memory capacity, process priorities, and storage consumption should be considered explicitly. Containers provide useful isolation boundaries, but inappropriate resource competition can still increase ROS 2 callback latency, degrade sensor processing, or cause AI workloads to interfere with time-sensitive robot functions.

Logging should be coordinated at the stack level because each service produces independent container and ROS 2 logs. Log rotation, maximum file size, persistent diagnostic storage, timestamps, and centralized collection should be configured so that long-running robots do not exhaust local storage. Service names, container versions, robot identifiers, and ROS 2 node information should remain correlated, allowing operators to reconstruct events across perception, navigation, hardware, and application services.

Secrets and security credentials should not be embedded directly into Compose files or container images. SROS2 keystores, certificates, registry credentials, API tokens, and other sensitive material should be injected through controlled secret-management mechanisms or protected mounts. Services should normally execute as non-root users, receive only required devices and capabilities, and expose only necessary network interfaces, reducing the security impact of a compromised container.

Profiles and optional services can make one Compose architecture usable for several operating modes. Development deployments may include visualization, debugging, tracing, and simulation services, while production robots activate only operational components. GPU inference, alternative sensor drivers, or maintenance tools can similarly be enabled selectively. This approach preserves a common deployment model without forcing every robot or development environment to run an identical set of containers.

The Compose deployment process should integrate with CI/CD rather than depend on manually built images. Each service image can be compiled, tested, scanned, versioned, and published to a registry before the Compose manifest references the validated release. Integration testing can then launch the complete stack in simulation or a hardware test environment, verifying communication, dependencies, lifecycle behavior, configuration compatibility, and startup sequencing before deployment to physical robots.

Rollback should be treated as a system-level capability because robot behavior depends on interactions among multiple services. Updating only one image may introduce interface, message, parameter, or behavioral incompatibilities with the remaining stack. A release manifest should therefore record the compatible image versions and configuration revisions forming a validated robot software baseline, enabling the entire deployment or selected compatible components to return to a known operational state.

Docker Compose is particularly suitable for single-robot computers, development systems, laboratory platforms, and moderately complex edge deployments where a lightweight orchestration mechanism is sufficient. As fleet scale and distributed infrastructure requirements increase, Kubernetes or K3s may provide stronger scheduling and cluster-management capabilities. Compose nevertheless remains valuable as a transparent deployment layer for defining and validating the service architecture of an individual ROS 2 robot.

A well-designed Docker Compose architecture therefore transforms ROS 2 deployment from a collection of startup commands into a reproducible robot software system. Container boundaries, networking, hardware mapping, persistent configuration, health supervision, security, resource control, versioning, CI/CD, and rollback become explicit parts of the deployment model. This creates a stable foundation for OTA updates, monitoring, fleet operations, and subsequent large-scale ROS 2 deployment mechanisms.

도커 컴포즈(Docker Compose)는 완전한 ROS 2 로봇 소프트웨어 스택(Robot Software Stack)을 여러 컨테이너(Container)의 조정된 집합으로 배포하기 위한 선언적 방법(Declarative Method)을 제공한다. 인지(Perception), 내비게이션(Navigation), 제어(Control), 진단(Diagnostics), 애플리케이션 프로세스를 수동으로 시작하는 대신 컴포즈 파일(Compose File)에 이미지, 런타임 파라미터, 네트워크, 볼륨, 장치, 의존성을 기술한다. 이를 통해 로봇 배포를 개발 및 운영 플랫폼에서 재현 가능한 버전 관리 시스템 구성으로 전환할 수 있다.

로봇 소프트웨어 스택은 모든 ROS 2 노드(Node)에 하나씩 컨테이너를 할당하는 방식이 아니라 운영 책임(Operational Responsibility)을 기준으로 서비스(Service)를 구분해야 한다. 일반적인 경계는 하드웨어 인터페이스, 인지 파이프라인, 위치추정(Localization), 내비게이션, AI 추론(Inference), 진단, 외부 게이트웨이(Gateway) 등으로 나눌 수 있다. 밀접하게 결합된 ROS 2 컴포넌트는 컴포지션(Composition)이 성능 향상에 유리한 경우 하나의 서비스에 유지할 수 있으며, 독립적인 업데이트나 장애 격리가 필요한 기능은 별도의 컨테이너로 구성하는 것이 유리하다.

컴포즈 YAML 파일은 이러한 서비스들이 함께 동작하는 방식을 정의하는 배포 매니페스트(Deployment Manifest)가 된다. 각 서비스는 사전 빌드된 이미지 또는 도커파일(Dockerfile)을 참조하고 명령과 엔트리포인트(Entrypoint)를 정의하며, 환경 변수, 볼륨, 하드웨어 장치, 재시작 동작을 설정할 수 있다. 이 구성을 버전 관리에 포함하면 ROS 2 패키지와 함께 배포 토폴로지(Deployment Topology)를 발전시키면서 시스템 수준 변경 이력을 추적할 수 있다.

다중 컨테이너 로봇은 여러 소프트웨어 구성요소 사이의 호환성에 의존하므로 이미지 버전(Image Version)을 명확하게 통제해야 한다. latest와 같은 변경 가능한 태그(Mutable Tag)를 사용하면 동일한 컴포즈 파일을 사용하는 로봇에서도 서로 다른 구현이 배포될 수 있다. 버전이 지정된 태그나 변경 불가능한 이미지 다이제스트(Immutable Image Digest)를 사용하면 재현성이 향상되고 플릿(Fleet) 구성을 복원하거나 릴리스를 검증하며 배포 문제 발생 시 개별 서비스를 롤백(Rollback)할 수 있다.

ROS 2 통신에는 DDS 검색(Discovery)과 전송이 서비스 경계를 넘어 동작해야 하므로 특별한 네트워킹(Networking) 요구사항이 존재한다. 컴포즈 네트워크는 애플리케이션 그룹을 격리할 수 있지만 멀티캐스트(Multicast), UDP 통신, 인터페이스 선택, ROS_DOMAIN_ID, DDS 설정은 의도된 토폴로지와 호환되어야 한다. 호스트 네트워킹(Host Networking)은 일부 로봇 배포를 단순화할 수 있지만 네트워크 격리를 감소시키므로 자동으로 적용하기보다 실제 운영 요구사항에 따라 선택해야 한다.

다중 로봇 시스템(Multi-Robot System)에서는 통신 도메인(Communication Domain)과 네임스페이스(Namespace)를 의도적으로 분리해야 한다. 환경 변수와 외부 설정을 이용하면 컨테이너 이미지를 수정하지 않고 ROS_DOMAIN_ID, 로봇 네임스페이스, DDS 프로파일(Profile), 고유 로봇 식별자를 지정할 수 있다. 이를 통해 공통 소프트웨어 이미지 집합을 여러 로봇에 배포하면서 컴포즈 설정을 이용해 각 물리 플랫폼에 적합한 로봇별 식별 정보와 통신 동작을 제공할 수 있다.

하드웨어를 직접 사용하는 서비스에는 물리적 자원에 대한 명시적인 접근 권한이 필요하다. 컴포즈를 사용하면 카메라, 직렬 포트(Serial Port), CAN 인터페이스, 라이다(LiDAR), USB 주변장치, GPU 및 기타 가속기를 전체 소프트웨어 스택에 노출하는 대신 필요한 컨테이너에만 매핑할 수 있다. 장치 매핑(Device Mapping), 그룹 권한, 런타임 설정, 리눅스 기능(Linux Capability)은 최소 권한 원칙(Principle of Least Privilege)을 따라야 하며, 한 서비스의 장애가 다른 로봇 기능에 불필요한 영향을 주지 않도록 해야 한다.

영속 데이터(Persistent Data)와 변경 가능한 설정 데이터는 볼륨(Volume)과 바인드 마운트(Bind Mount)를 이용하여 변경 불가능한 컨테이너 이미지와 분리해야 한다. 캘리브레이션(Calibration) 파일, 지도, ROS 2 파라미터 파일, DDS 프로파일, 보안 자료, AI 모델, 운영 데이터는 컨테이너가 교체되어도 유지하거나 애플리케이션 바이너리와 독립적으로 변경할 수 있다. 쓰기 가능한 디렉터리는 명확하게 정의해야 하며, 무제한적인 영속성을 허용하면 배포 동작의 재현성과 소프트웨어 버전 간 롤백 관리가 어려워질 수 있다.

서비스 의존성(Service Dependency)은 단순한 시작 순서 지정만으로 관리할 수 없다. 컨테이너가 시작되었더라도 ROS 2 노드는 아직 초기화 중이거나 하드웨어를 기다리고 있을 수 있으며, DDS 참여자(Participant)를 검색하거나 지도를 로딩하거나 라이프사이클 상태(Lifecycle State)를 전환하고 있을 수 있다. 따라서 의존성 관리는 실행 중인 컨테이너를 준비된 서브시스템으로 간주하지 말고 컴포즈 시작 관계, 상태 확인(Health Check), ROS 2 라이프사이클 오케스트레이션(Lifecycle Orchestration), 애플리케이션 준비 상태(Readiness), 재시도 메커니즘을 함께 사용해야 한다.

상태 확인(Health Check)은 컨테이너 감독(Container Supervision)과 로봇 애플리케이션 상태를 연결하는 역할을 한다. 기본적인 검사는 프로세스가 실행 중인지 확인하고, 보다 강력한 검사는 예상 서비스, 장치 가용성, 애플리케이션 엔드포인트(Endpoint), ROS 2 서브시스템의 준비 상태를 확인할 수 있다. 그러나 안전 필수(Safety-Critical) 의사결정은 도커 상태만을 기준으로 해서는 안 되며, 컨테이너 감독은 전용 로봇 진단, 안전 제어기, 워치독(Watchdog), 라이프사이클 기반 안전 상태(Safe State) 메커니즘을 대체하지 않는다.

재시작 정책(Restart Policy)은 프로세스가 예기치 않게 종료될 때 가용성(Availability)을 향상시킬 수 있지만 서브시스템의 특성에 맞게 설계해야 한다. 시각화나 텔레메트리(Telemetry) 서비스를 재시작하는 것은 비교적 단순하지만 하드웨어 드라이버나 모션(Motion) 관련 구성요소를 반복적으로 재시작하면 바람직하지 않은 상태 전환이 발생할 수 있다. 따라서 복구 정책은 재시작 제한, 라이프사이클 초기화, 하드웨어 상태, 의존성 복구를 조정하고 자동 복구 실패 시 통제된 로봇 안전 상태로 전환하도록 설계해야 한다.

환경별 설정(Environment-Specific Configuration)은 가능한 경우 기본 컴포즈 정의와 분리하는 것이 좋다. 공통 기본 설정은 표준 ROS 2 스택을 정의하고 환경 파일(Environment File)이나 컴포즈 오버라이드(Compose Override)를 통해 개발 컴퓨터, 시뮬레이션, 실험실 로봇, GPU 플랫폼, 운영 시스템에 맞게 조정할 수 있다. 이렇게 하면 중복된 배포 매니페스트가 서로 다르게 변화하는 것을 방지하면서 이기종 로봇 컴퓨팅 환경에서도 일관된 아키텍처를 유지할 수 있다.

인지 및 피지컬 AI(Physical AI) 워크로드가 내비게이션과 제어 서비스와 동일한 엣지 컴퓨터(Edge Computer)를 공유할 경우 자원 관리(Resource Management)가 더욱 중요해진다. CPU 할당, 메모리 제한, GPU 접근, 공유 메모리(Shared Memory) 용량, 프로세스 우선순위, 저장공간 사용량을 명시적으로 고려해야 한다. 컨테이너는 유용한 격리 경계를 제공하지만 부적절한 자원 경쟁은 ROS 2 콜백 지연시간(Callback Latency)을 증가시키거나 센서 처리를 저하시키고 AI 워크로드가 시간 민감형 로봇 기능에 영향을 줄 수 있다.

로깅(Logging)은 각 서비스가 독립적인 컨테이너 로그와 ROS 2 로그를 생성하므로 전체 스택 수준에서 조정해야 한다. 장기간 운영되는 로봇의 로컬 저장공간이 고갈되지 않도록 로그 순환(Log Rotation), 최대 파일 크기, 영속 진단 저장소, 타임스탬프(Timestamp), 중앙 집중식 수집을 구성해야 한다. 서비스 이름, 컨테이너 버전, 로봇 식별자, ROS 2 노드 정보를 연계하여 인지, 내비게이션, 하드웨어, 애플리케이션 서비스 전반에서 발생한 이벤트를 재구성할 수 있어야 한다.

시크릿(Secret)과 보안 자격 증명(Security Credential)은 컴포즈 파일이나 컨테이너 이미지 내부에 직접 포함해서는 안 된다. SROS2 키스토어(Keystore), 인증서(Certificate), 레지스트리 자격 증명, API 토큰 및 기타 민감한 자료는 통제된 시크릿 관리 메커니즘이나 보호된 마운트를 통해 주입해야 한다. 서비스는 일반적으로 비루트 사용자(Non-Root User)로 실행하고 필요한 장치와 기능만 제공하며 필요한 네트워크 인터페이스만 노출하여 컨테이너 침해 시 보안 영향을 줄여야 한다.

프로파일(Profile)과 선택적 서비스(Optional Service)를 이용하면 하나의 컴포즈 아키텍처를 여러 운영 모드에 사용할 수 있다. 개발 환경에서는 시각화, 디버깅(Debugging), 트레이싱(Tracing), 시뮬레이션 서비스를 포함하고 운영 로봇에서는 실제 운용에 필요한 구성요소만 활성화할 수 있다. GPU 추론, 대체 센서 드라이버, 유지보수 도구도 선택적으로 활성화하여 모든 로봇과 개발 환경에 동일한 컨테이너 집합을 강제로 실행하지 않으면서 공통 배포 모델을 유지할 수 있다.

컴포즈 배포 과정은 수동으로 빌드한 이미지에 의존하지 않고 CI/CD와 통합되어야 한다. 각 서비스 이미지는 컴파일, 테스트, 보안 스캔(Security Scan), 버전 지정, 레지스트리(Registry) 게시 과정을 거친 후 컴포즈 매니페스트에서 검증된 릴리스를 참조할 수 있다. 이후 통합 테스트는 시뮬레이션 또는 하드웨어 시험 환경에서 전체 스택을 실행하여 물리 로봇에 배포하기 전에 통신, 의존성, 라이프사이클 동작, 설정 호환성, 시작 순서를 검증할 수 있다.

롤백은 로봇 동작이 여러 서비스 사이의 상호작용에 의존하므로 시스템 수준 기능(System-Level Capability)으로 다루어야 한다. 하나의 이미지만 업데이트하더라도 나머지 스택과 인터페이스, 메시지, 파라미터 또는 동작 호환성 문제가 발생할 수 있다. 따라서 릴리스 매니페스트(Release Manifest)는 검증된 로봇 소프트웨어 기준선(Baseline)을 구성하는 호환 이미지 버전과 설정 리비전(Configuration Revision)을 기록하여 전체 배포 또는 선택된 호환 구성요소를 알려진 정상 상태로 복원할 수 있어야 한다.

도커 컴포즈는 경량 오케스트레이션(Lightweight Orchestration) 메커니즘으로 충분한 단일 로봇 컴퓨터, 개발 시스템, 실험실 플랫폼, 중간 수준 복잡도의 엣지 배포에 특히 적합하다. 플릿 규모와 분산 인프라 요구사항이 증가하면 쿠버네티스(Kubernetes) 또는 K3s가 더욱 강력한 스케줄링(Scheduling)과 클러스터 관리 기능을 제공할 수 있다. 그럼에도 컴포즈는 개별 ROS 2 로봇의 서비스 아키텍처를 정의하고 검증하기 위한 투명하고 실용적인 배포 계층으로 중요한 가치를 가진다.

따라서 잘 설계된 도커 컴포즈 아키텍처는 ROS 2 배포를 단순한 시작 명령의 집합에서 재현 가능한 로봇 소프트웨어 시스템으로 전환한다. 컨테이너 경계, 네트워킹, 하드웨어 매핑, 영속 설정, 상태 감독, 보안, 자원 제어, 버전 관리, CI/CD, 롤백이 배포 모델의 명시적인 구성요소가 된다. 이는 무선 업데이트(OTA), 모니터링(Monitoring), 플릿 운영(Fleet Operations), 이후의 대규모 ROS 2 배포 메커니즘으로 확장하기 위한 안정적인 기반을 형성한다.

##  

## 08.03 Kubernetes / K3s-Based Robot Deployment [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

Kubernetes provides a declarative orchestration layer for deploying, supervising, and updating containerized ROS 2 software across distributed robot and edge-computing systems. While Docker Compose is effective for managing a stack on an individual robot computer, Kubernetes introduces cluster-level scheduling, service management, recovery, configuration, and rollout mechanisms. K3s provides a lightweight Kubernetes distribution that makes these capabilities practical for resource-constrained robotics environments.

A robot deployment can model each physical robot, edge computer, or on-premise server as a Kubernetes node participating in a common cluster. Workloads are represented as Pods containing one or more closely related containers, while higher-level controllers maintain their desired operating state. This architecture separates application deployment intent from individual host configuration and allows robot software to be managed through a consistent cluster control plane.

K3s is particularly attractive for robotics because it reduces the operational footprint associated with conventional Kubernetes installations. It packages essential cluster components into a compact distribution while retaining standard Kubernetes APIs and deployment concepts. This allows laboratories, AMR fleets, inspection robots, and edge AI platforms to adopt Kubernetes-compatible orchestration without requiring the infrastructure normally associated with large cloud data centers.

The fundamental deployment unit is the Pod, but ROS 2 software boundaries should not automatically correspond to individual ROS 2 nodes. Closely coupled components requiring shared memory or synchronized execution may belong in the same Pod, whereas perception, navigation, AI inference, diagnostics, gateways, and supporting services can be separated according to lifecycle, resource, update, and fault-isolation requirements. Container boundaries should therefore reflect system architecture rather than arbitrary process granularity.

Deployments and other Kubernetes workload controllers describe the desired number and version of application instances. For stateless supporting services, Kubernetes can automatically replace failed Pods and maintain the requested replica count. Robot-specific components require greater caution because hardware drivers, motion controllers, and localization processes are associated with physical devices and state. Automatic rescheduling must respect the relationship between software workloads and the robot hardware they control.

Node labels, selectors, affinity rules, and taints can constrain workloads to appropriate computing platforms. A perception Pod may require an NVIDIA GPU, a hardware-interface Pod may need access to a specific robot computer, and an AI inference workload may be assigned to a high-performance edge server. These scheduling mechanisms allow heterogeneous CPU, GPU, Jetson, and server resources to coexist within a common deployment architecture while maintaining explicit placement policies.

ROS 2 communication over DDS introduces networking considerations that differ from typical web-oriented Kubernetes applications. DDS discovery may depend on multicast, UDP transport, specific network interfaces, and direct connectivity among participants. Overlay networks and network address translation can interfere with expected discovery behavior. Robotics deployments must therefore validate host networking, DDS discovery servers, interface configuration, ROS_DOMAIN_ID, and the selected DDS implementation against the actual cluster topology.

Multi-robot deployments require isolation and identity management beyond basic Kubernetes service discovery. Robot namespaces, ROS_DOMAIN_ID values, DDS profiles, network policies, and robot identifiers can be injected through environment variables and configuration resources. A common container image can therefore support multiple physical robots while deployment manifests determine each robot\'s communication domain, namespace, hardware association, and operational role without rebuilding application software.

ConfigMaps provide a mechanism for separating non-sensitive configuration from container images. ROS 2 parameter files, DDS profiles, application settings, robot identifiers, and environment-specific values can be supplied dynamically to Pods. Sensitive information such as credentials, certificates, API tokens, and SROS2 security material should instead be handled through Kubernetes Secrets or an external secret-management system, with access restricted to only the workloads that require it.

Persistent storage requires special treatment because robot workloads generate maps, logs, rosbag or MCAP recordings, calibration information, AI data, and diagnostic histories. Persistent Volumes and storage classes can separate this data from ephemeral Pod filesystems. Edge deployments must nevertheless account for intermittent connectivity and local storage constraints, ensuring that critical operational data remains available even when robots temporarily lose access to centralized storage infrastructure.

Hardware access is one of the most important differences between conventional Kubernetes workloads and robotic applications. Cameras, LiDAR sensors, CAN interfaces, serial devices, motor controllers, and GPUs are physically attached to particular machines. Device plugins, host device mappings, runtime classes, privileged capabilities, and node placement policies may therefore be required. Access should remain narrowly scoped so that container orchestration does not unnecessarily weaken the host security boundary.

Resource requests and limits allow Kubernetes to describe CPU and memory requirements for each workload, while specialized mechanisms can allocate GPU resources. This is useful when perception, navigation, simulation, diagnostics, and Physical AI inference share heterogeneous computing infrastructure. Resource policies should be validated carefully, however, because excessive CPU contention, memory pressure, or GPU competition can increase ROS 2 latency and degrade time-sensitive robot behavior.

Real-time control should remain separated from assumptions made by general-purpose cluster orchestration. Kubernetes can supervise deployment and availability, but it does not transform ordinary container scheduling into deterministic real-time execution. High-frequency motor control, safety loops, and strict timing functions should remain within appropriately designed real-time processes, controllers, or embedded systems. Kubernetes should primarily orchestrate higher-level ROS 2 workloads around these deterministic control boundaries.

Health management can combine Kubernetes probes with ROS 2 application-level diagnostics. Liveness probes determine whether a workload should be restarted, readiness probes determine whether it is prepared to receive operational traffic, and startup probes accommodate components with long initialization periods. For ROS 2 systems, these mechanisms can be supplemented with lifecycle state, sensor availability, DDS connectivity, diagnostics, and subsystem readiness rather than relying solely on process existence.

Kubernetes self-healing is useful but must be constrained by physical-system semantics. Automatically restarting a telemetry service is fundamentally different from restarting a motor interface or relocating a hardware-dependent Pod to another node. Recovery policies should distinguish stateless services from robot-bound workloads and coordinate restart behavior with lifecycle management, watchdogs, hardware initialization, fault diagnostics, and safe-state transitions when automatic recovery is inappropriate.

Rolling updates allow new software versions to be introduced gradually instead of replacing an entire deployment simultaneously. This is valuable for infrastructure and stateless services, while physical robot software requires additional operational safeguards. Deployment strategies should consider robot mission state, charging status, maintenance windows, hardware compatibility, and validated software baselines. Updates should be stoppable and reversible when health indicators or integration tests reveal unexpected behavior.

Namespaces and role-based access control provide organizational and security boundaries within the cluster. Development, simulation, staging, and production workloads can be separated, while service accounts and RBAC policies restrict which components may modify deployments or access configuration. Network policies can further limit communication paths. These mechanisms complement SROS2 and DDS security, which continue to protect ROS 2 identities, permissions, and communications at the middleware layer.

Observability becomes essential as deployment expands from one robot to a distributed fleet. Kubernetes exposes workload state, restart events, resource utilization, and scheduling information, while ROS 2 contributes node diagnostics, lifecycle states, topic behavior, sensor status, and application logs. Combining infrastructure-level and robot-level telemetry allows operators to distinguish container failures, network problems, resource exhaustion, DDS issues, hardware faults, and application-level degradation.

CI/CD pipelines can generate container images and Kubernetes manifests as version-controlled deployment artifacts. Images should be compiled, tested, security-scanned, tagged, and published to a registry before being referenced by validated manifests. Simulation and hardware-in-the-loop testing can then exercise complete deployments before release. GitOps-style practices can additionally maintain the desired fleet configuration as auditable repository state rather than relying on manual cluster modifications.

K3s can support hierarchical robot infrastructure in which individual robots execute local workloads while edge servers or on-premise computers provide heavier perception, AI inference, data aggregation, and fleet services. Placement policies can determine which computations remain on the robot and which execute on more powerful nodes. This creates a flexible foundation for combining edge intelligence, on-premise AI, and multi-robot coordination within a common deployment framework.

Connectivity loss must be treated as a normal robotics operating condition rather than an exceptional cluster failure. A mobile robot may temporarily lose Wi-Fi, private 5G, or access to the Kubernetes control plane while continuing its mission. Essential navigation, control, perception, and safety functions should therefore remain locally executable. Cluster orchestration should enhance management and recovery without making basic robot autonomy dependent on continuous infrastructure connectivity.

Kubernetes and K3s consequently extend ROS 2 deployment from container management on individual computers toward coordinated orchestration across robots, edge nodes, and shared infrastructure. Their value comes from declarative configuration, controlled scheduling, resource management, health supervision, security boundaries, observable operations, and managed updates. When combined with ROS 2 lifecycle management and robot-aware safety design, they provide a scalable deployment foundation for increasingly distributed robotic systems.

쿠버네티스(Kubernetes)는 분산된 로봇 및 엣지 컴퓨팅(Edge Computing) 시스템 전반에서 컨테이너화된 ROS 2 소프트웨어를 배포, 감독, 업데이트하기 위한 선언적 오케스트레이션 계층(Declarative Orchestration Layer)을 제공한다. 도커 컴포즈(Docker Compose)가 개별 로봇 컴퓨터의 스택 관리에 효과적인 반면, 쿠버네티스는 클러스터 수준 스케줄링(Cluster-Level Scheduling), 서비스 관리, 복구, 설정, 롤아웃(Rollout) 메커니즘을 제공한다. K3s는 이러한 기능을 자원이 제한된 로보틱스 환경에서도 실용적으로 사용할 수 있도록 하는 경량 쿠버네티스 배포판(Lightweight Kubernetes Distribution)이다.

로봇 배포에서는 각각의 물리 로봇, 엣지 컴퓨터 또는 온프레미스 서버(On-Premise Server)를 공통 클러스터에 참여하는 쿠버네티스 노드(Kubernetes Node)로 모델링할 수 있다. 워크로드(Workload)는 하나 이상의 밀접하게 관련된 컨테이너를 포함하는 파드(Pod)로 표현되며, 상위 수준 컨트롤러(Controller)가 원하는 운영 상태를 유지한다. 이러한 아키텍처는 애플리케이션의 배포 의도와 개별 호스트 설정을 분리하고 일관된 클러스터 제어 평면(Control Plane)을 통해 로봇 소프트웨어를 관리할 수 있게 한다.

K3s는 기존 쿠버네티스 설치에서 요구되는 운영 부담을 줄이기 때문에 로보틱스 분야에 특히 적합하다. 필수적인 클러스터 구성요소를 소형 배포판으로 통합하면서도 표준 쿠버네티스 API와 배포 개념을 유지한다. 이를 통해 연구실, AMR 플릿(Fleet), 검사 로봇, 엣지 AI 플랫폼(Edge AI Platform)은 대규모 클라우드 데이터센터 수준의 인프라를 구축하지 않고도 쿠버네티스 호환 오케스트레이션을 도입할 수 있다.

기본적인 배포 단위는 파드이지만 ROS 2 소프트웨어의 경계를 개별 ROS 2 노드(Node)와 자동으로 일치시켜서는 안 된다. 공유 메모리(Shared Memory)나 동기화된 실행이 필요한 밀접한 구성요소는 동일한 파드에 포함할 수 있으며, 인지(Perception), 내비게이션(Navigation), AI 추론(Inference), 진단(Diagnostics), 게이트웨이(Gateway), 지원 서비스는 라이프사이클(Lifecycle), 자원, 업데이트, 장애 격리 요구사항에 따라 분리할 수 있다. 따라서 컨테이너 경계는 임의적인 프로세스 단위가 아니라 시스템 아키텍처를 반영해야 한다.

디플로이먼트(Deployment)와 기타 쿠버네티스 워크로드 컨트롤러는 원하는 애플리케이션 인스턴스의 개수와 버전을 정의한다. 상태 비저장 지원 서비스(Stateless Supporting Service)의 경우 쿠버네티스는 장애가 발생한 파드를 자동으로 교체하고 지정된 복제본 수(Replica Count)를 유지할 수 있다. 반면 로봇 전용 구성요소는 하드웨어 드라이버, 모션 제어기(Motion Controller), 위치추정 프로세스가 물리 장치 및 상태와 연결되므로 더 신중하게 관리해야 한다. 자동 재스케줄링(Automatic Rescheduling)은 소프트웨어 워크로드와 이를 제어하는 로봇 하드웨어 사이의 관계를 고려해야 한다.

노드 레이블(Node Label), 셀렉터(Selector), 어피니티 규칙(Affinity Rule), 테인트(Taint)를 사용하여 워크로드를 적절한 컴퓨팅 플랫폼으로 제한할 수 있다. 인지 파드는 NVIDIA GPU가 필요할 수 있고, 하드웨어 인터페이스 파드는 특정 로봇 컴퓨터에 접근해야 하며, AI 추론 워크로드는 고성능 엣지 서버에 배치될 수 있다. 이러한 스케줄링 메커니즘을 통해 이기종 CPU, GPU, Jetson, 서버 자원을 공통 배포 아키텍처에서 함께 사용하면서 명확한 배치 정책(Placement Policy)을 유지할 수 있다.

DDS 기반 ROS 2 통신은 일반적인 웹 중심 쿠버네티스 애플리케이션과 다른 네트워킹(Networking) 특성을 가진다. DDS 검색(Discovery)은 멀티캐스트(Multicast), UDP 전송, 특정 네트워크 인터페이스, 참여자 간 직접 연결에 의존할 수 있다. 오버레이 네트워크(Overlay Network)와 네트워크 주소 변환(Network Address Translation)은 예상되는 검색 동작을 방해할 수 있다. 따라서 로봇 배포에서는 호스트 네트워킹(Host Networking), DDS 디스커버리 서버(Discovery Server), 인터페이스 설정, ROS_DOMAIN_ID, 선택한 DDS 구현체를 실제 클러스터 토폴로지에서 검증해야 한다.

다중 로봇 배포(Multi-Robot Deployment)에는 기본적인 쿠버네티스 서비스 검색 이상의 격리와 식별 관리가 필요하다. 로봇 네임스페이스(Namespace), ROS_DOMAIN_ID, DDS 프로파일(Profile), 네트워크 정책(Network Policy), 로봇 식별자를 환경 변수와 설정 리소스를 통해 주입할 수 있다. 따라서 공통 컨테이너 이미지를 여러 물리 로봇에서 사용하면서 배포 매니페스트(Deployment Manifest)를 통해 각 로봇의 통신 도메인, 네임스페이스, 하드웨어 연결 관계, 운영 역할을 결정할 수 있다.

컨피그맵(ConfigMap)은 민감하지 않은 설정을 컨테이너 이미지와 분리하기 위한 메커니즘을 제공한다. ROS 2 파라미터 파일, DDS 프로파일, 애플리케이션 설정, 로봇 식별자, 환경별 값을 동적으로 파드에 제공할 수 있다. 자격 증명, 인증서(Certificate), API 토큰, SROS2 보안 자료와 같은 민감한 정보는 쿠버네티스 시크릿(Kubernetes Secret) 또는 외부 시크릿 관리 시스템을 통해 처리하고 실제로 필요한 워크로드에만 접근 권한을 제한해야 한다.

로봇 워크로드는 지도, 로그, rosbag 또는 MCAP 기록, 캘리브레이션(Calibration) 정보, AI 데이터, 진단 이력을 생성하므로 영속 저장소(Persistent Storage)를 별도로 관리해야 한다. 퍼시스턴트 볼륨(Persistent Volume)과 스토리지 클래스(Storage Class)를 사용하여 이러한 데이터를 일시적인 파드 파일시스템에서 분리할 수 있다. 그러나 엣지 배포에서는 간헐적인 연결과 로컬 저장공간 제약을 고려하여 로봇이 중앙 저장 인프라와 일시적으로 연결되지 않더라도 중요한 운영 데이터를 사용할 수 있도록 해야 한다.

하드웨어 접근(Hardware Access)은 일반적인 쿠버네티스 워크로드와 로봇 애플리케이션을 구분하는 가장 중요한 차이점 중 하나이다. 카메라, 라이다(LiDAR), CAN 인터페이스, 직렬 장치(Serial Device), 모터 제어기, GPU는 특정 컴퓨터에 물리적으로 연결된다. 따라서 디바이스 플러그인(Device Plugin), 호스트 장치 매핑, 런타임 클래스(Runtime Class), 특권 기능(Privileged Capability), 노드 배치 정책이 필요할 수 있다. 컨테이너 오케스트레이션이 호스트 보안 경계를 불필요하게 약화시키지 않도록 접근 권한은 최소 범위로 제한해야 한다.

자원 요청(Resource Request)과 제한(Limit)을 이용하면 각 워크로드의 CPU 및 메모리 요구사항을 정의할 수 있으며, 특수한 메커니즘을 통해 GPU 자원도 할당할 수 있다. 이는 인지, 내비게이션, 시뮬레이션, 진단, 피지컬 AI(Physical AI) 추론이 이기종 컴퓨팅 인프라를 공유할 때 유용하다. 그러나 과도한 CPU 경쟁, 메모리 압박, GPU 경쟁은 ROS 2 지연시간(Latency)을 증가시키고 시간 민감형 로봇 동작을 저하시킬 수 있으므로 자원 정책을 신중하게 검증해야 한다.

실시간 제어(Real-Time Control)는 범용 클러스터 오케스트레이션의 가정과 분리하여 설계해야 한다. 쿠버네티스는 배포와 가용성을 감독할 수 있지만 일반적인 컨테이너 스케줄링을 결정적 실시간 실행(Deterministic Real-Time Execution)으로 변환하지는 않는다. 고주파 모터 제어, 안전 루프(Safety Loop), 엄격한 타이밍 기능은 적절하게 설계된 실시간 프로세스, 제어기 또는 임베디드 시스템에 유지해야 한다. 쿠버네티스는 이러한 결정적 제어 경계 주변의 상위 수준 ROS 2 워크로드를 주로 오케스트레이션해야 한다.

상태 관리(Health Management)는 쿠버네티스 프로브(Kubernetes Probe)와 ROS 2 애플리케이션 수준 진단을 결합할 수 있다. 라이브니스 프로브(Liveness Probe)는 워크로드를 재시작해야 하는지 판단하고, 레디니스 프로브(Readiness Probe)는 운영 트래픽을 처리할 준비가 되었는지 판단하며, 스타트업 프로브(Startup Probe)는 초기화 시간이 긴 구성요소를 지원한다. ROS 2 시스템에서는 단순한 프로세스 존재 여부뿐 아니라 라이프사이클 상태, 센서 가용성, DDS 연결, 진단 정보, 서브시스템 준비 상태를 함께 사용할 수 있다.

쿠버네티스의 자가 치유(Self-Healing)는 유용하지만 물리 시스템의 의미론에 따라 제한되어야 한다. 텔레메트리(Telemetry) 서비스를 자동 재시작하는 것과 모터 인터페이스를 재시작하거나 하드웨어 의존 파드를 다른 노드로 이동하는 것은 근본적으로 다르다. 복구 정책은 상태 비저장 서비스와 로봇 종속 워크로드를 구분하고 라이프사이클 관리, 워치독(Watchdog), 하드웨어 초기화, 장애 진단, 자동 복구가 부적절한 경우의 안전 상태(Safe State) 전환과 재시작 동작을 조정해야 한다.

롤링 업데이트(Rolling Update)는 전체 배포를 동시에 교체하지 않고 새로운 소프트웨어 버전을 점진적으로 도입할 수 있게 한다. 이는 인프라 및 상태 비저장 서비스에 유용하지만 물리 로봇 소프트웨어에는 추가적인 운영 안전장치가 필요하다. 배포 전략은 로봇의 임무 상태, 충전 상태, 유지보수 시간대(Maintenance Window), 하드웨어 호환성, 검증된 소프트웨어 기준선(Baseline)을 고려해야 한다. 상태 지표나 통합 테스트에서 예상하지 못한 동작이 발견되면 업데이트를 중단하고 이전 상태로 복구할 수 있어야 한다.

네임스페이스와 역할 기반 접근 제어(Role-Based Access Control, RBAC)는 클러스터 내부에 조직적 및 보안 경계를 제공한다. 개발, 시뮬레이션, 스테이징(Staging), 운영 워크로드를 분리할 수 있으며 서비스 계정(Service Account)과 RBAC 정책을 통해 어떤 구성요소가 배포를 수정하거나 설정에 접근할 수 있는지 제한할 수 있다. 네트워크 정책을 통해 통신 경로도 추가로 제한할 수 있다. 이러한 메커니즘은 ROS 2 미들웨어 계층에서 신원, 권한, 통신을 보호하는 SROS2 및 DDS 보안을 보완한다.

배포가 단일 로봇에서 분산 플릿으로 확장되면 관찰 가능성(Observability)이 필수적이다. 쿠버네티스는 워크로드 상태, 재시작 이벤트, 자원 사용량, 스케줄링 정보를 제공하며 ROS 2는 노드 진단, 라이프사이클 상태, 토픽 동작, 센서 상태, 애플리케이션 로그를 제공한다. 인프라 수준 텔레메트리와 로봇 수준 텔레메트리를 결합하면 컨테이너 장애, 네트워크 문제, 자원 고갈, DDS 문제, 하드웨어 장애, 애플리케이션 성능 저하를 구분할 수 있다.

CI/CD 파이프라인은 컨테이너 이미지와 쿠버네티스 매니페스트를 버전 관리되는 배포 산출물로 생성할 수 있다. 이미지는 검증된 매니페스트에서 참조하기 전에 컴파일, 테스트, 보안 스캔(Security Scan), 태깅(Tagging), 레지스트리(Registry) 게시 과정을 거쳐야 한다. 이후 시뮬레이션과 하드웨어 인 더 루프 테스트(Hardware-in-the-Loop Testing)를 통해 전체 배포를 릴리스 전에 검증할 수 있다. 깃옵스(GitOps) 방식은 수동 클러스터 변경 대신 원하는 플릿 구성을 감사 가능한 저장소 상태로 관리할 수 있게 한다.

K3s는 개별 로봇이 로컬 워크로드를 실행하고 엣지 서버 또는 온프레미스 컴퓨터가 더 무거운 인지, AI 추론, 데이터 집계, 플릿 서비스를 제공하는 계층형 로봇 인프라(Hierarchical Robot Infrastructure)를 지원할 수 있다. 배치 정책을 이용하여 어떤 연산을 로봇에 유지하고 어떤 연산을 더 강력한 노드에서 실행할지 결정할 수 있다. 이는 엣지 지능(Edge Intelligence), 온프레미스 AI(On-Premise AI), 다중 로봇 협조(Multi-Robot Coordination)를 공통 배포 프레임워크에서 결합하기 위한 유연한 기반을 제공한다.

연결 손실(Connectivity Loss)은 예외적인 클러스터 장애가 아니라 정상적인 로봇 운영 조건으로 다루어야 한다. 이동 로봇은 임무를 계속 수행하면서 Wi-Fi, 프라이빗 5G(Private 5G), 쿠버네티스 제어 평면과의 연결을 일시적으로 잃을 수 있다. 따라서 필수적인 내비게이션, 제어, 인지, 안전 기능은 로컬에서 계속 실행할 수 있어야 한다. 클러스터 오케스트레이션은 기본적인 로봇 자율성이 지속적인 인프라 연결에 의존하도록 만들지 않으면서 관리와 복구 기능을 강화해야 한다.

따라서 쿠버네티스와 K3s는 ROS 2 배포를 개별 컴퓨터의 컨테이너 관리에서 로봇, 엣지 노드, 공유 인프라 전반의 협조된 오케스트레이션(Coordinated Orchestration)으로 확장한다. 그 가치는 선언적 설정, 통제된 스케줄링, 자원 관리, 상태 감독, 보안 경계, 관찰 가능한 운영, 관리형 업데이트에서 나온다. ROS 2 라이프사이클 관리와 로봇 인식 안전 설계(Robot-Aware Safety Design)를 결합하면 점차 분산화되는 로봇 시스템을 위한 확장 가능한 배포 기반을 제공할 수 있다.

##  

## 08.04 ROS2 CI/CD Pipeline: GitHub Actions [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

Continuous Integration and Continuous Delivery establish a repeatable mechanism for validating and releasing ROS 2 software whenever source code changes. GitHub Actions can automate this workflow directly from a repository by reacting to pushes, pull requests, tags, or manually triggered events. For robotics teams, the pipeline becomes a quality gate that verifies software before it reaches simulation, laboratory hardware, edge computers, or production robots.

A ROS 2 CI/CD workflow is normally defined as version-controlled YAML under the repository's GitHub Actions configuration. The workflow specifies triggers, jobs, execution environments, permissions, environment variables, and individual validation steps. Keeping pipeline definitions beside application source code allows build and test procedures to evolve with the software while making changes reviewable through the same pull-request process used for ROS 2 packages.

Pipeline triggers should reflect the development and release strategy. Pull requests can initiate fast compilation, linting, and unit tests before code is merged, while commits to protected branches can execute broader integration tests and container builds. Version tags or approved release events can initiate packaging and deployment stages. Separating validation from release triggers prevents ordinary development activity from unintentionally publishing or deploying production artifacts.

The build environment must reproduce the intended ROS 2 distribution, operating-system version, and architecture as closely as practical. GitHub-hosted runners can provide standardized environments for common builds, while containers can establish a more controlled ROS 2 toolchain. Self-hosted runners become useful when workflows require specialized GPUs, ARM or Jetson platforms, private networks, proprietary dependencies, or direct access to robotics test infrastructure.

Dependency installation should be automated and deterministic. A workflow can create the ROS 2 workspace, import required repositories, update rosdep information, resolve system dependencies, and install packages before invoking colcon. Dependency versions should be constrained where reproducibility matters because uncontrolled external updates can cause a previously successful pipeline to fail even when application source code has not changed.

The primary compilation stage should execute colcon build with options appropriate to the target workspace and development policy. Build failures, compiler warnings, missing dependencies, interface-generation problems, and package-ordering issues can therefore be detected before integration. Clean builds remain important even when caching is enabled because the pipeline must demonstrate that the repository can produce valid ROS 2 artifacts without relying on undeclared state from a developer workstation.

Caching can substantially reduce CI execution time for large ROS 2 projects, but cached data should be selected carefully. Package downloads, compiler caches, or stable dependency layers may provide useful acceleration, whereas blindly caching build and install directories can conceal dependency problems. Cache keys should incorporate relevant operating-system, ROS distribution, compiler, dependency, and source information so that incompatible artifacts are not reused across different build configurations.

Static analysis and formatting checks provide an early quality barrier before computationally expensive tests execute. ROS 2 packages can integrate linters, compiler diagnostics, formatting rules, copyright checks, and language-specific analysis through the ament ecosystem or dedicated tools. Running these checks consistently in CI reduces differences between developers and prevents style, maintainability, and common implementation defects from accumulating unnoticed.

Unit testing should execute automatically after successful compilation. C++ packages can use gtest through ament_cmake_gtest, while Python packages can use appropriate testing frameworks integrated with the ROS 2 build system. Test results generated by colcon should be collected and reported even when failures occur. This gives developers immediate information about which package or component failed rather than presenting the pipeline only as a binary success or failure indicator.

Integration testing verifies behavior that cannot be established from isolated packages. A workflow may launch multiple ROS 2 nodes and validate topic communication, services, actions, parameters, lifecycle transitions, or expected message flows. Test environments can also start Docker Compose stacks or temporary container networks. Timeouts and cleanup procedures are essential because distributed ROS 2 tests can otherwise leave processes running or cause CI jobs to wait indefinitely.

Simulation provides an important intermediate validation layer between software-only testing and physical robot deployment. Headless Gazebo, other simulators, or application-specific test environments can execute navigation, perception, control, and mission scenarios without requiring physical hardware for every commit. Simulation does not replace hardware testing, but it allows major integration failures and behavioral regressions to be detected earlier and at significantly greater execution frequency.

A matrix build can test the same ROS 2 source against multiple distributions, operating systems, middleware implementations, or build configurations. This is particularly useful for reusable libraries and robot platforms that must support heterogeneous deployment targets. Matrix coverage should remain intentional because every additional combination increases CI cost and maintenance effort. Critical production combinations deserve stronger coverage than experimental environments.

Container image generation can follow successful ROS 2 build and test stages. The pipeline can build Docker images using validated Dockerfiles, assign immutable version identifiers, and publish them to an authorized container registry. Multi-stage Docker builds help ensure that development dependencies remain outside production images. Image metadata should connect the deployed artifact to the source commit, release version, architecture, and ROS distribution from which it was produced.

Security checks should be incorporated before artifacts become eligible for deployment. Workflows can scan source dependencies, container images, credentials exposure, and known vulnerabilities while enforcing repository permissions and protected release processes. GitHub Actions permissions should follow least-privilege principles, and long-lived credentials should not be embedded in workflow files. Repository secrets or federated identity mechanisms should provide controlled access to registries and deployment infrastructure.

Artifacts provide traceability between CI execution and robot deployment. Test reports, coverage information, compiled packages, container metadata, configuration manifests, and selected diagnostic outputs can be retained according to project policy. A release should be reconstructable from its source revision and associated artifacts. This becomes especially important when diagnosing a robot failure months after deployment or demonstrating which software baseline was used during validation.

Self-hosted runners extend CI from cloud-based software validation into physical robotics infrastructure. A laboratory runner can access GPUs, Jetson devices, CAN interfaces, sensors, or complete robots that are unavailable to standard hosted environments. Such runners should be isolated and carefully secured because executing repository workflows on machines connected to physical hardware introduces substantially greater operational risk than ordinary software compilation.

Hardware-in-the-loop testing should be positioned after lower-cost validation stages have succeeded. The pipeline can deploy a candidate build to a controlled robot or test bench, initialize required ROS 2 subsystems, execute predefined scenarios, collect diagnostics, and evaluate expected outcomes. Hardware access should be serialized when necessary so that concurrent jobs do not attempt to control the same device, and failures should return the test system to a known safe state.

Deployment should promote previously validated artifacts rather than rebuild source code independently for each environment. A candidate container image can move from development validation to simulation, staging, laboratory robots, and eventually production while retaining the same immutable digest. Environment-specific parameters remain external to the image. This approach reduces uncertainty about whether the software tested in CI is actually identical to the software executing on the robot.

Production deployment requires stronger controls than ordinary CI execution. Protected environments, approval gates, release branches, signed artifacts, maintenance windows, and staged rollout policies can prevent unreviewed changes from reaching operational robots. Deployment workflows should record which software version was delivered to each target and preserve a rollback path to a validated baseline when post-deployment health checks reveal unexpected behavior.

For robot fleets, CI/CD should coordinate software versioning with configuration compatibility. A release may include ROS 2 packages, container images, parameter files, DDS profiles, lifecycle settings, and deployment manifests that have been validated together. Treating these elements as a coherent release baseline prevents partial updates from introducing incompatible interfaces or behavior across perception, navigation, control, AI inference, and fleet-management services.

Observability closes the loop between deployment and subsequent development. CI/CD can verify immediate deployment health, while operational monitoring provides evidence about crashes, resource consumption, ROS 2 diagnostics, communication failures, and application performance after release. These observations can become inputs for regression tests and future pipeline improvements, turning field experience into repeatable validation rather than leaving operational knowledge outside the development process.

A mature GitHub Actions pipeline therefore connects source control, reproducible builds, static analysis, automated testing, simulation, container generation, security validation, hardware testing, release management, and controlled deployment. For ROS 2 robotics, CI/CD is not simply an automation convenience; it becomes part of the software assurance architecture that establishes traceability from a source commit to the exact validated software operating on a physical robot.

지속적 통합(Continuous Integration, CI)과 지속적 전달(Continuous Delivery, CD)은 소스 코드가 변경될 때마다 ROS 2 소프트웨어를 검증하고 릴리스하기 위한 반복 가능한 메커니즘을 구축한다. 깃허브 액션(GitHub Actions)은 푸시(Push), 풀 리퀘스트(Pull Request), 태그(Tag), 수동 실행 이벤트에 반응하여 저장소(Repository)에서 직접 이러한 워크플로(Workflow)를 자동화할 수 있다. 로보틱스 팀에서 이 파이프라인은 소프트웨어가 시뮬레이션, 실험실 하드웨어, 엣지 컴퓨터(Edge Computer), 운영 로봇에 도달하기 전에 검증하는 품질 게이트(Quality Gate)가 된다.

ROS 2 CI/CD 워크플로는 일반적으로 저장소의 깃허브 액션 설정에 버전 관리되는 YAML 파일로 정의한다. 워크플로에는 트리거(Trigger), 작업(Job), 실행 환경, 권한, 환경 변수, 개별 검증 단계가 지정된다. 파이프라인 정의를 애플리케이션 소스 코드와 함께 관리하면 ROS 2 패키지와 함께 빌드 및 테스트 절차를 발전시킬 수 있으며, ROS 2 패키지에 사용하는 것과 동일한 풀 리퀘스트 검토 과정에서 파이프라인 변경사항도 검토할 수 있다.

파이프라인 트리거는 개발 및 릴리스 전략을 반영해야 한다. 풀 리퀘스트에서는 코드가 병합되기 전에 빠른 컴파일, 린팅(Linting), 단위 테스트(Unit Test)를 실행할 수 있으며, 보호된 브랜치(Protected Branch)에 대한 커밋은 보다 광범위한 통합 테스트와 컨테이너 빌드를 실행할 수 있다. 버전 태그나 승인된 릴리스 이벤트는 패키징 및 배포 단계를 시작할 수 있다. 검증과 릴리스 트리거를 분리하면 일반적인 개발 활동으로 운영 산출물이 의도치 않게 게시되거나 배포되는 것을 방지할 수 있다.

빌드 환경(Build Environment)은 가능한 범위에서 대상 ROS 2 배포판, 운영체제 버전, 아키텍처를 재현해야 한다. 깃허브 호스티드 러너(GitHub-Hosted Runner)는 일반적인 빌드를 위한 표준화된 환경을 제공할 수 있으며, 컨테이너를 사용하면 더욱 통제된 ROS 2 툴체인(Toolchain)을 구축할 수 있다. 특수 GPU, ARM 또는 Jetson 플랫폼, 사설 네트워크, 독점 의존성, 로봇 시험 인프라에 직접 접근해야 하는 경우에는 셀프 호스티드 러너(Self-Hosted Runner)가 유용하다.

의존성 설치(Dependency Installation)는 자동화되고 결정적(Deterministic)이어야 한다. 워크플로는 ROS 2 워크스페이스(Workspace)를 생성하고 필요한 저장소를 가져오며 rosdep 정보를 업데이트하고 시스템 의존성을 해결한 후 colcon을 실행할 수 있다. 재현성이 중요한 경우에는 의존성 버전을 제한해야 한다. 통제되지 않은 외부 업데이트로 인해 애플리케이션 소스 코드가 변경되지 않았음에도 이전에 성공했던 파이프라인이 실패할 수 있기 때문이다.

주요 컴파일 단계에서는 대상 워크스페이스와 개발 정책에 적합한 옵션으로 colcon build를 실행해야 한다. 이를 통해 통합 이전에 빌드 실패, 컴파일러 경고, 누락된 의존성, 인터페이스 생성 문제, 패키지 순서 문제를 발견할 수 있다. 캐싱(Caching)을 사용하더라도 클린 빌드(Clean Build)는 중요하다. 파이프라인은 개발자 워크스테이션의 선언되지 않은 상태에 의존하지 않고 저장소 자체에서 유효한 ROS 2 산출물을 생성할 수 있음을 입증해야 한다.

캐싱은 대규모 ROS 2 프로젝트의 CI 실행 시간을 크게 단축할 수 있지만 캐시할 데이터는 신중하게 선택해야 한다. 패키지 다운로드, 컴파일러 캐시 또는 안정적인 의존성 레이어는 유용한 가속 효과를 제공할 수 있지만 build 및 install 디렉터리를 무분별하게 캐시하면 의존성 문제를 숨길 수 있다. 캐시 키(Cache Key)는 관련 운영체제, ROS 배포판, 컴파일러, 의존성, 소스 정보를 반영하여 서로 다른 빌드 구성에서 호환되지 않는 산출물이 재사용되지 않도록 해야 한다.

정적 분석(Static Analysis)과 포맷 검사(Formatting Check)는 계산 비용이 높은 테스트를 실행하기 전에 초기 품질 장벽을 제공한다. ROS 2 패키지는 ament 생태계 또는 전용 도구를 통해 린터(Linter), 컴파일러 진단, 포맷 규칙, 저작권 검사, 언어별 분석을 통합할 수 있다. 이러한 검사를 CI에서 일관되게 실행하면 개발자 환경 사이의 차이를 줄이고 스타일, 유지보수성, 일반적인 구현 결함이 발견되지 않은 채 누적되는 것을 방지할 수 있다.

단위 테스트는 성공적인 컴파일 이후 자동으로 실행해야 한다. C++ 패키지는 ament_cmake_gtest를 통해 gtest를 사용할 수 있으며, Python 패키지는 ROS 2 빌드 시스템과 통합된 적절한 테스트 프레임워크를 사용할 수 있다. colcon에서 생성한 테스트 결과는 실패가 발생하더라도 수집하고 보고해야 한다. 이를 통해 개발자는 단순히 파이프라인의 성공 또는 실패만 확인하는 것이 아니라 어떤 패키지나 구성요소에서 문제가 발생했는지 즉시 파악할 수 있다.

통합 테스트(Integration Testing)는 개별 패키지 테스트만으로 확인할 수 없는 동작을 검증한다. 워크플로에서 여러 ROS 2 노드를 실행하고 토픽(Topic) 통신, 서비스(Service), 액션(Action), 파라미터(Parameter), 라이프사이클 전환(Lifecycle Transition), 예상 메시지 흐름을 검증할 수 있다. 테스트 환경에서는 도커 컴포즈(Docker Compose) 스택이나 임시 컨테이너 네트워크를 실행할 수도 있다. 분산 ROS 2 테스트가 프로세스를 남겨두거나 CI 작업을 무기한 대기시키지 않도록 타임아웃(Timeout)과 정리 절차가 필수적이다.

시뮬레이션(Simulation)은 소프트웨어 전용 테스트와 물리 로봇 배포 사이에서 중요한 중간 검증 계층을 제공한다. 헤드리스 Gazebo(Headless Gazebo), 기타 시뮬레이터 또는 애플리케이션별 시험 환경을 이용하여 모든 커밋마다 물리 하드웨어를 사용하지 않고도 내비게이션, 인지(Perception), 제어(Control), 임무 시나리오를 실행할 수 있다. 시뮬레이션이 하드웨어 테스트를 대체하지는 않지만 주요 통합 장애와 동작 회귀(Behavioral Regression)를 더 이른 단계에서 훨씬 높은 빈도로 발견할 수 있다.

매트릭스 빌드(Matrix Build)를 이용하면 동일한 ROS 2 소스를 여러 배포판, 운영체제, 미들웨어(Middleware) 구현체 또는 빌드 구성에서 테스트할 수 있다. 이는 이기종 배포 대상을 지원해야 하는 재사용 가능 라이브러리와 로봇 플랫폼에 특히 유용하다. 그러나 조합이 추가될 때마다 CI 비용과 유지보수 노력이 증가하므로 매트릭스 범위는 의도적으로 결정해야 한다. 실험 환경보다 실제 운영에 사용되는 핵심 조합에 더 강력한 검증 범위를 제공하는 것이 중요하다.

ROS 2 빌드 및 테스트 단계가 성공하면 컨테이너 이미지(Container Image)를 생성할 수 있다. 파이프라인은 검증된 도커파일(Dockerfile)을 이용해 도커 이미지를 빌드하고 변경 불가능한 버전 식별자를 부여한 후 승인된 컨테이너 레지스트리(Container Registry)에 게시할 수 있다. 다단계 도커 빌드(Multi-Stage Docker Build)를 이용하면 개발 의존성이 운영 이미지에 포함되는 것을 방지할 수 있다. 이미지 메타데이터는 배포 산출물을 소스 커밋, 릴리스 버전, 아키텍처, ROS 배포판과 연결해야 한다.

산출물이 배포 가능한 상태가 되기 전에 보안 검사(Security Check)를 통합해야 한다. 워크플로에서는 소스 의존성, 컨테이너 이미지, 자격 증명 노출, 알려진 취약점을 검사하면서 저장소 권한과 보호된 릴리스 절차를 적용할 수 있다. 깃허브 액션 권한은 최소 권한 원칙(Least-Privilege Principle)을 따라야 하며 장기 자격 증명을 워크플로 파일에 포함해서는 안 된다. 저장소 시크릿(Repository Secret)이나 연합 신원(Federated Identity) 메커니즘을 통해 레지스트리 및 배포 인프라에 대한 접근을 통제해야 한다.

산출물(Artifact)은 CI 실행과 로봇 배포 사이의 추적성(Traceability)을 제공한다. 테스트 보고서, 커버리지(Coverage) 정보, 컴파일된 패키지, 컨테이너 메타데이터, 설정 매니페스트(Configuration Manifest), 선택된 진단 출력은 프로젝트 정책에 따라 보존할 수 있다. 릴리스는 해당 소스 리비전(Source Revision)과 관련 산출물을 이용해 재구성할 수 있어야 한다. 이는 배포 후 수개월이 지난 로봇 장애를 분석하거나 검증 과정에서 어떤 소프트웨어 기준선(Baseline)이 사용되었는지 입증할 때 특히 중요하다.

셀프 호스티드 러너는 CI를 클라우드 기반 소프트웨어 검증에서 물리 로보틱스 인프라까지 확장한다. 실험실 러너는 일반적인 호스티드 환경에서 사용할 수 없는 GPU, Jetson 장치, CAN 인터페이스, 센서 또는 완전한 로봇에 접근할 수 있다. 이러한 러너는 신중하게 격리하고 보안을 강화해야 한다. 물리 하드웨어에 연결된 컴퓨터에서 저장소 워크플로를 실행하는 것은 일반적인 소프트웨어 컴파일보다 훨씬 큰 운영 위험을 발생시키기 때문이다.

하드웨어 인 더 루프 테스트(Hardware-in-the-Loop Testing)는 비용이 낮은 검증 단계가 성공한 이후에 배치해야 한다. 파이프라인은 후보 빌드를 통제된 로봇이나 시험 벤치(Test Bench)에 배포하고 필요한 ROS 2 서브시스템을 초기화한 후 사전에 정의된 시나리오를 실행하여 진단 정보를 수집하고 예상 결과를 평가할 수 있다. 여러 작업이 동일한 장치를 동시에 제어하지 않도록 필요에 따라 하드웨어 접근을 직렬화(Serialization)해야 하며, 실패가 발생하면 시험 시스템을 알려진 안전 상태(Safe State)로 복귀시켜야 한다.

배포(Deployment)는 각 환경에서 소스 코드를 독립적으로 다시 빌드하는 대신 이미 검증된 산출물을 승격(Promotion)하는 방식으로 이루어져야 한다. 후보 컨테이너 이미지는 동일한 변경 불가능한 다이제스트(Immutable Digest)를 유지한 상태에서 개발 검증, 시뮬레이션, 스테이징(Staging), 실험실 로봇, 최종 운영 환경으로 이동할 수 있다. 환경별 파라미터는 이미지 외부에 유지한다. 이를 통해 CI에서 테스트한 소프트웨어와 실제 로봇에서 실행되는 소프트웨어가 동일한지에 대한 불확실성을 줄일 수 있다.

운영 배포(Production Deployment)에는 일반적인 CI 실행보다 강력한 통제가 필요하다. 보호된 환경(Protected Environment), 승인 게이트(Approval Gate), 릴리스 브랜치, 서명된 산출물(Signed Artifact), 유지보수 시간대(Maintenance Window), 단계적 롤아웃(Staged Rollout) 정책을 통해 검토되지 않은 변경사항이 실제 운영 로봇에 도달하는 것을 방지할 수 있다. 배포 워크플로는 각 대상에 전달된 소프트웨어 버전을 기록하고 배포 후 상태 검사에서 예상하지 못한 동작이 발견될 경우 검증된 기준선으로 롤백(Rollback)할 수 있는 경로를 유지해야 한다.

로봇 플릿(Robot Fleet)에서는 CI/CD가 소프트웨어 버전 관리와 설정 호환성을 함께 조정해야 한다. 하나의 릴리스에는 함께 검증된 ROS 2 패키지, 컨테이너 이미지, 파라미터 파일, DDS 프로파일(Profile), 라이프사이클 설정, 배포 매니페스트가 포함될 수 있다. 이러한 요소를 하나의 일관된 릴리스 기준선으로 관리하면 부분적인 업데이트로 인해 인지, 내비게이션, 제어, AI 추론, 플릿 관리 서비스 사이에 호환되지 않는 인터페이스나 동작이 발생하는 것을 방지할 수 있다.

관찰 가능성(Observability)은 배포와 이후 개발 사이의 피드백 루프(Feedback Loop)를 완성한다. CI/CD는 배포 직후의 상태를 검증할 수 있으며, 운영 모니터링은 릴리스 이후 발생하는 충돌(Crash), 자원 소비, ROS 2 진단, 통신 장애, 애플리케이션 성능에 대한 증거를 제공한다. 이러한 관찰 결과를 회귀 테스트(Regression Test)와 향후 파이프라인 개선에 활용하면 현장 경험을 개발 과정 외부의 지식으로 남겨두지 않고 반복 가능한 검증 절차로 전환할 수 있다.

따라서 성숙한 깃허브 액션 파이프라인은 소스 제어(Source Control), 재현 가능한 빌드, 정적 분석, 자동화 테스트, 시뮬레이션, 컨테이너 생성, 보안 검증, 하드웨어 테스트, 릴리스 관리, 통제된 배포를 하나의 연속적인 과정으로 연결한다. ROS 2 로보틱스에서 CI/CD는 단순한 자동화 편의 기능이 아니라 소스 커밋에서 실제 물리 로봇에서 동작하는 정확한 검증 소프트웨어까지의 추적성을 확립하는 소프트웨어 보증 아키텍처(Software Assurance Architecture)의 일부가 된다.

##  

## 08.05 ROS2 Package Distribution: bloom / ros.infrastructure

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 package distribution transforms source repositories into installable software that users can obtain through standard operating-system package managers. Rather than requiring every robot developer to clone repositories and compile workspaces manually, the ROS infrastructure provides a release process that generates binary packages for supported ROS distributions and platforms. This creates a consistent path from package development to reproducible installation and deployment across robotics systems.

A ROS 2 package normally begins in a source repository containing package.xml, build-system configuration, source code, interfaces, tests, and documentation. The package manifest defines metadata such as package name, version, maintainers, licenses, build dependencies, execution dependencies, and export information. Accurate metadata is essential because the release infrastructure uses this information to resolve dependencies and construct packages for downstream users.

Source development and package release are intentionally separated. Developers can continue normal development in the primary repository while a specific source revision is selected for an official release. This separation prevents the distribution process from depending directly on a changing development branch and provides a stable relationship between a released version, its source tag, generated packaging information, and resulting binary artifacts.

Bloom is the primary ROS release automation tool used to convert ROS package metadata into distribution-specific packaging information. It reads package manifests and repository configuration, processes dependency information, and generates the files required by the target packaging ecosystem. For Ubuntu-based ROS deployments, this typically means generating Debian packaging metadata that can later be built into binary .deb packages by the ROS build infrastructure.

The release process commonly uses a dedicated release repository generated and maintained through bloom. This repository is different from the normal source repository because it contains generated packaging branches and metadata corresponding to released versions. Keeping generated release information separate prevents packaging-specific files from cluttering application development while preserving a traceable history of how each ROS package version was prepared for distribution.

The ROS distribution index, commonly represented through rosdistro metadata, connects package repositories with a particular ROS distribution. Repository entries describe source, release, and documentation information together with version and status data. When maintainers prepare a package for a ROS distribution, the corresponding metadata must eventually be updated so that the central infrastructure recognizes the new release and can schedule the required build operations.

Dependency resolution is a central part of this process. Package dependencies declared in package.xml are mapped to platform-specific system packages through rosdep rules and distribution metadata. Bloom uses this information when producing packaging configuration, while build systems later install the required dependencies before compilation. Incorrect or incomplete dependency declarations can therefore cause failures even when the package builds successfully inside the maintainer\'s existing development workspace.

Package versioning should follow a disciplined release policy because versions become part of the public distribution interface. A release identifies a specific implementation that downstream packages and robot deployments may depend upon. Changing source code without changing the corresponding package version can undermine traceability, while uncontrolled version changes can complicate dependency management. Release tags and package.xml versions should therefore remain synchronized.

When a maintainer initiates a bloom release, the tool processes the selected source version, updates the release repository, and prepares changes required for the ROS distribution metadata. The resulting update is typically submitted for review before becoming part of the distribution index. This review step separates package-maintainer actions from modifications to the shared ROS distribution and provides an opportunity to detect metadata or release problems before automated builds begin.

After release metadata is accepted, ROS build infrastructure can generate platform-specific binary packages. Build jobs obtain the released source and packaging information, install dependencies in controlled environments, compile the software, execute applicable packaging steps, and produce repository artifacts. Building packages centrally reduces dependence on individual developer machines and helps ensure that binaries distributed to users correspond to known source and dependency configurations.

Binary repositories provide the user-facing endpoint of the ROS distribution pipeline. Once successfully built and synchronized, ROS 2 packages can be installed using standard operating-system package-management commands rather than compiling from source. This greatly simplifies provisioning of development computers, CI runners, edge systems, and robot computers because common ROS packages can be installed using the same dependency-management mechanisms as other system software.

Package naming follows conventions that connect the ROS distribution and package identity to the generated operating-system package. A ROS package with a source-level name may therefore appear as a distribution-specific Debian package with a ROS-prefixed name. Understanding this distinction is useful when diagnosing rosdep resolution, apt installation, dependency conflicts, or differences between package names used inside a ROS workspace and names visible to the operating system.

The distribution pipeline must account for multiple supported platforms and architectures. A package that compiles successfully on one developer computer may fail on another operating-system version or processor architecture because of compiler behavior, dependency availability, or platform-specific code. Automated build infrastructure exposes these problems before users encounter them and encourages maintainers to avoid assumptions that exist only in their local development environment.

Package maintainers should treat build failures as release-quality signals rather than merely infrastructure problems. Missing dependencies, undeclared system libraries, incorrect install rules, generated-interface errors, and architecture assumptions frequently become visible only in clean build environments. Reproducing these failures locally with clean containers or CI environments can help identify hidden dependencies and improve the portability of the ROS 2 package.

Release automation works best when integrated with normal CI practices. Source repositories should already pass compilation, linting, unit testing, and integration testing before a version is selected for distribution. Bloom and the ROS build infrastructure then address packaging and distribution rather than replacing software validation. This establishes a layered process in which CI validates source quality and the release infrastructure validates package construction and distribution readiness.

Public ROS distribution is not the only useful application of these concepts. Robotics organizations can maintain internal package repositories for proprietary drivers, robot applications, AI components, and company-specific middleware. The same principles of explicit package metadata, versioning, dependency management, controlled builds, repository publication, and traceable releases can support private ROS 2 distribution even when packages are not submitted to the public ROS ecosystem.

Private distribution is particularly useful for production robots because a complete system often combines public ROS packages with proprietary software. Public dependencies may come from standard ROS repositories while organization-specific packages are delivered through authenticated internal repositories or container images. Release baselines should record both sets of dependencies so that a deployed robot can be reconstructed without relying on an undocumented developer workspace.

Package signing and repository trust are important when binary packages become part of production deployment. Robot computers should obtain software only from authorized repositories using verified repository configuration and package-management security mechanisms. Distribution security must also be coordinated with CI/CD, container security, OTA policies, and software bill of materials practices because package provenance becomes part of the overall robot software supply chain.

ROS 2 distribution should distinguish reusable middleware packages from robot-specific configuration. Libraries, drivers, interfaces, algorithms, and stable application components are suitable candidates for versioned package distribution, while frequently changing maps, calibration files, robot identities, secrets, and environment-specific parameters may require separate configuration mechanisms. This separation prevents unnecessary package releases whenever operational configuration changes.

A release should remain traceable from installed binary back to package version, release repository, and original source revision. This traceability becomes important when investigating field failures, reproducing older robot configurations, managing security vulnerabilities, or determining which fleets contain an affected software version. Package distribution therefore contributes not only installation convenience but also configuration management and long-term maintainability.

The ROS release ecosystem creates a structured chain connecting source repositories, bloom, release repositories, rosdistro metadata, automated build infrastructure, binary repositories, and end-user installation. Each stage has a distinct responsibility, allowing development, packaging, building, and distribution to remain separated while still forming a reproducible pipeline. This modular structure supports both community-scale ROS software sharing and disciplined industrial robot software delivery.

For large robotic systems, package distribution can coexist with Docker, Kubernetes, and OTA mechanisms rather than competing with them. ROS packages may be installed while constructing a container image, containers may then be validated through CI/CD, and validated images may later be deployed to robots or fleets. Bloom and ROS infrastructure therefore occupy the package-release layer within a broader deployment architecture that can extend from source code to production robot operation.

ROS 2 패키지 배포(Package Distribution)는 소스 저장소(Source Repository)를 사용자가 표준 운영체제 패키지 관리자(Package Manager)를 통해 설치할 수 있는 소프트웨어로 변환한다. 모든 로봇 개발자가 저장소를 복제하고 워크스페이스를 직접 컴파일하는 대신 ROS 인프라는 지원되는 ROS 배포판과 플랫폼을 위한 바이너리 패키지(Binary Package)를 생성하는 릴리스 프로세스를 제공한다. 이를 통해 패키지 개발부터 재현 가능한 설치와 로봇 시스템 배포까지 일관된 경로를 구축할 수 있다.

ROS 2 패키지는 일반적으로 package.xml, 빌드 시스템 설정, 소스 코드, 인터페이스, 테스트, 문서를 포함하는 소스 저장소에서 시작한다. 패키지 매니페스트(Package Manifest)는 패키지 이름, 버전, 유지관리자, 라이선스, 빌드 의존성, 실행 의존성, 내보내기 정보 등의 메타데이터(Metadata)를 정의한다. 릴리스 인프라는 이 정보를 이용하여 의존성을 해결하고 최종 사용자를 위한 패키지를 구성하므로 정확한 메타데이터 정의가 필수적이다.

소스 개발(Source Development)과 패키지 릴리스(Package Release)는 의도적으로 분리된다. 개발자는 기본 저장소에서 일반적인 개발을 계속할 수 있으며, 공식 릴리스에는 특정 소스 리비전(Source Revision)을 선택한다. 이러한 분리는 배포 프로세스가 지속적으로 변경되는 개발 브랜치에 직접 의존하는 것을 방지하고 릴리스 버전, 소스 태그(Source Tag), 생성된 패키징 정보, 최종 바이너리 산출물 사이에 안정적인 관계를 제공한다.

블룸(Bloom)은 ROS 패키지 메타데이터를 배포판별 패키징 정보로 변환하는 데 사용되는 주요 ROS 릴리스 자동화 도구(Release Automation Tool)이다. 블룸은 패키지 매니페스트와 저장소 설정을 읽고 의존성 정보를 처리하여 대상 패키징 생태계에 필요한 파일을 생성한다. Ubuntu 기반 ROS 배포에서는 일반적으로 이후 ROS 빌드 인프라에서 바이너리 .deb 패키지로 빌드할 수 있는 데비안 패키징 메타데이터(Debian Packaging Metadata)를 생성한다.

릴리스 프로세스에서는 일반적으로 블룸을 통해 생성되고 관리되는 별도의 릴리스 저장소(Release Repository)를 사용한다. 이 저장소는 릴리스된 버전에 대응하는 생성된 패키징 브랜치와 메타데이터를 포함한다는 점에서 일반 소스 저장소와 다르다. 생성된 릴리스 정보를 분리하면 패키징 전용 파일이 애플리케이션 개발 저장소를 복잡하게 만드는 것을 방지하면서 각 ROS 패키지 버전이 배포용으로 준비된 과정을 추적할 수 있다.

일반적으로 rosdistro 메타데이터로 표현되는 ROS 배포판 인덱스(ROS Distribution Index)는 패키지 저장소를 특정 ROS 배포판과 연결한다. 저장소 항목은 버전 및 상태 데이터와 함께 소스(Source), 릴리스(Release), 문서(Documentation) 정보를 정의한다. 유지관리자가 특정 ROS 배포판용 패키지를 준비하면 중앙 인프라가 새로운 릴리스를 인식하고 필요한 빌드 작업을 예약할 수 있도록 관련 메타데이터를 최종적으로 업데이트해야 한다.

의존성 해결(Dependency Resolution)은 이 과정의 핵심 부분이다. package.xml에 선언된 패키지 의존성은 rosdep 규칙과 배포판 메타데이터를 통해 플랫폼별 시스템 패키지에 매핑된다. 블룸은 패키징 설정을 생성할 때 이 정보를 사용하며, 이후 빌드 시스템은 컴파일 전에 필요한 의존성을 설치한다. 따라서 유지관리자의 기존 개발 워크스페이스에서 정상적으로 빌드되더라도 의존성 선언이 잘못되거나 누락되면 릴리스 빌드가 실패할 수 있다.

패키지 버전 관리(Package Versioning)는 버전 자체가 공개 배포 인터페이스의 일부가 되므로 체계적인 릴리스 정책을 따라야 한다. 릴리스는 다른 패키지와 로봇 배포가 의존할 수 있는 특정 구현을 식별한다. 패키지 버전을 변경하지 않고 소스 코드만 변경하면 추적성이 손상될 수 있으며, 통제되지 않은 버전 변경은 의존성 관리를 복잡하게 만들 수 있다. 따라서 릴리스 태그와 package.xml 버전은 서로 동기화된 상태를 유지해야 한다.

유지관리자가 블룸 릴리스를 시작하면 도구는 선택된 소스 버전을 처리하고 릴리스 저장소를 업데이트하며 ROS 배포판 메타데이터에 필요한 변경사항을 준비한다. 생성된 업데이트는 일반적으로 배포판 인덱스에 포함되기 전에 검토를 위해 제출된다. 이러한 검토 단계는 패키지 유지관리자의 작업과 공유 ROS 배포판의 변경을 분리하며, 자동 빌드가 시작되기 전에 메타데이터 또는 릴리스 문제를 발견할 기회를 제공한다.

릴리스 메타데이터가 승인되면 ROS 빌드 인프라(Build Infrastructure)가 플랫폼별 바이너리 패키지를 생성할 수 있다. 빌드 작업은 릴리스된 소스와 패키징 정보를 가져오고 통제된 환경에서 의존성을 설치한 다음 소프트웨어를 컴파일하고 필요한 패키징 단계를 수행하여 저장소 산출물을 생성한다. 중앙에서 패키지를 빌드하면 개별 개발자 컴퓨터에 대한 의존성을 줄이고 사용자에게 배포되는 바이너리가 알려진 소스 및 의존성 구성에 대응하도록 할 수 있다.

바이너리 저장소(Binary Repository)는 ROS 배포 파이프라인에서 사용자가 직접 접하는 최종 지점을 제공한다. 빌드와 동기화가 성공적으로 완료되면 ROS 2 패키지를 소스에서 직접 컴파일하지 않고 표준 운영체제 패키지 관리 명령을 사용하여 설치할 수 있다. 따라서 개발 컴퓨터, CI 러너(Runner), 엣지 시스템, 로봇 컴퓨터에서도 일반적인 시스템 소프트웨어와 동일한 의존성 관리 메커니즘을 사용하여 공통 ROS 패키지를 설치할 수 있다.

패키지 이름 지정(Package Naming)은 ROS 배포판과 패키지 식별자를 생성되는 운영체제 패키지와 연결하는 규칙을 따른다. 따라서 소스 수준의 ROS 패키지 이름은 ROS 접두사가 포함된 배포판별 데비안 패키지 이름으로 나타날 수 있다. 이러한 차이를 이해하면 rosdep 의존성 해결, apt 설치, 의존성 충돌 또는 ROS 워크스페이스 내부에서 사용하는 패키지 이름과 운영체제에 표시되는 패키지 이름 사이의 차이를 분석하는 데 도움이 된다.

배포 파이프라인은 지원되는 여러 플랫폼과 아키텍처(Architecture)를 고려해야 한다. 한 개발자 컴퓨터에서 정상적으로 컴파일되는 패키지도 컴파일러 동작, 의존성 가용성, 플랫폼별 코드 차이로 인해 다른 운영체제 버전이나 프로세서 아키텍처에서는 실패할 수 있다. 자동화된 빌드 인프라는 사용자가 이러한 문제를 경험하기 전에 문제를 발견하고 유지관리자가 자신의 로컬 개발 환경에만 존재하는 가정을 피하도록 한다.

패키지 유지관리자는 빌드 실패를 단순한 인프라 문제가 아니라 릴리스 품질 신호(Release-Quality Signal)로 다루어야 한다. 누락된 의존성, 선언되지 않은 시스템 라이브러리, 잘못된 설치 규칙, 생성 인터페이스 오류, 아키텍처 관련 가정은 깨끗한 빌드 환경에서만 발견되는 경우가 많다. 깨끗한 컨테이너 또는 CI 환경에서 이러한 실패를 로컬로 재현하면 숨겨진 의존성을 식별하고 ROS 2 패키지의 이식성(Portability)을 향상시킬 수 있다.

릴리스 자동화(Release Automation)는 일반적인 CI 방식과 통합할 때 가장 효과적으로 작동한다. 소스 저장소는 배포할 버전을 선택하기 전에 이미 컴파일, 린팅(Linting), 단위 테스트(Unit Testing), 통합 테스트(Integration Testing)를 통과해야 한다. 이후 블룸과 ROS 빌드 인프라는 소프트웨어 검증을 대체하는 것이 아니라 패키징과 배포를 담당한다. 이를 통해 CI는 소스 품질을 검증하고 릴리스 인프라는 패키지 구성과 배포 준비 상태를 검증하는 계층화된 프로세스를 구축할 수 있다.

공개 ROS 배포만이 이러한 개념을 활용할 수 있는 것은 아니다. 로보틱스 조직은 독점 드라이버, 로봇 애플리케이션, AI 구성요소, 회사 전용 미들웨어를 위한 내부 패키지 저장소(Internal Package Repository)를 운영할 수 있다. 명시적인 패키지 메타데이터, 버전 관리, 의존성 관리, 통제된 빌드, 저장소 게시, 추적 가능한 릴리스라는 동일한 원칙을 적용하면 패키지를 공개 ROS 생태계에 제출하지 않더라도 사설 ROS 2 배포(Private ROS 2 Distribution)를 지원할 수 있다.

사설 배포(Private Distribution)는 완전한 로봇 시스템이 공개 ROS 패키지와 독점 소프트웨어를 함께 사용하는 경우가 많기 때문에 운영 로봇에서 특히 유용하다. 공개 의존성은 표준 ROS 저장소에서 가져오고 조직 전용 패키지는 인증된 내부 저장소나 컨테이너 이미지를 통해 제공할 수 있다. 배포된 로봇을 문서화되지 않은 개발자 워크스페이스에 의존하지 않고 재구성할 수 있도록 릴리스 기준선(Release Baseline)에 두 종류의 의존성을 모두 기록해야 한다.

바이너리 패키지가 운영 배포의 일부가 되면 패키지 서명(Package Signing)과 저장소 신뢰(Repository Trust)가 중요해진다. 로봇 컴퓨터는 검증된 저장소 설정과 패키지 관리 보안 메커니즘을 사용하여 승인된 저장소에서만 소프트웨어를 가져와야 한다. 패키지 출처(Provenance)는 전체 로봇 소프트웨어 공급망(Software Supply Chain)의 일부가 되므로 배포 보안은 CI/CD, 컨테이너 보안, 무선 업데이트(OTA) 정책, 소프트웨어 자재 명세서(Software Bill of Materials, SBOM) 관리와 연계해야 한다.

ROS 2 배포에서는 재사용 가능한 미들웨어 패키지와 로봇별 설정(Robot-Specific Configuration)을 구분해야 한다. 라이브러리, 드라이버, 인터페이스, 알고리즘, 안정적인 애플리케이션 구성요소는 버전이 지정된 패키지 배포에 적합하다. 반면 자주 변경되는 지도, 캘리브레이션 파일, 로봇 식별 정보, 시크릿(Secret), 환경별 파라미터는 별도의 설정 메커니즘을 사용하는 것이 적절하다. 이러한 분리를 통해 운영 설정이 변경될 때마다 불필요하게 패키지를 다시 릴리스하는 것을 방지할 수 있다.

릴리스는 설치된 바이너리에서 패키지 버전, 릴리스 저장소, 원본 소스 리비전까지 추적할 수 있어야 한다. 이러한 추적성은 현장 장애 조사, 이전 로봇 구성 재현, 보안 취약점 관리, 영향을 받는 소프트웨어 버전을 사용하는 플릿 식별에 중요하다. 따라서 패키지 배포는 단순한 설치 편의성뿐 아니라 구성 관리(Configuration Management)와 장기 유지보수성(Long-Term Maintainability)에도 기여한다.

ROS 릴리스 생태계는 소스 저장소, 블룸, 릴리스 저장소, rosdistro 메타데이터, 자동화된 빌드 인프라, 바이너리 저장소, 최종 사용자 설치를 연결하는 구조화된 체인을 형성한다. 각 단계는 서로 다른 책임을 가지므로 개발, 패키징, 빌드, 배포를 분리하면서도 재현 가능한 하나의 파이프라인으로 연결할 수 있다. 이러한 모듈식 구조는 커뮤니티 규모의 ROS 소프트웨어 공유뿐 아니라 체계적인 산업용 로봇 소프트웨어 전달에도 적용할 수 있다.

대규모 로봇 시스템에서는 패키지 배포가 도커(Docker), 쿠버네티스(Kubernetes), 무선 업데이트(OTA) 메커니즘과 경쟁하는 것이 아니라 함께 사용될 수 있다. 컨테이너 이미지를 생성할 때 ROS 패키지를 설치하고, 생성된 컨테이너를 CI/CD를 통해 검증한 후 검증된 이미지를 로봇 또는 플릿에 배포할 수 있다. 따라서 블룸과 ROS 인프라는 소스 코드에서 운영 로봇까지 확장되는 더 광범위한 배포 아키텍처 내부에서 패키지 릴리스 계층(Package-Release Layer)을 담당한다.

##  

## 08.06 Robot OTA and ROS2 Package Update Integration [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

Robot Over-the-Air update architecture provides a controlled mechanism for delivering new ROS 2 software, configuration, security fixes, and system components to deployed robots without requiring direct physical access. OTA becomes especially important when robots operate as fleets across factories, logistics centers, outdoor facilities, or remote sites. The update system must preserve availability, traceability, compatibility, and safe robot behavior throughout the complete deployment lifecycle.

ROS 2 software can be updated at several layers, including Debian packages, application binaries, container images, configuration files, AI models, and complete operating-system images. These mechanisms should not be treated as interchangeable. Package-level updates provide fine-grained changes, containers provide stronger application isolation, while system-image approaches support atomic platform replacement. A practical robot may combine several methods under one coordinated OTA policy.

The update pipeline should begin with a clearly identified software release rather than an arbitrary collection of recently built files. ROS 2 packages, container images, parameter sets, DDS profiles, firmware dependencies, and deployment manifests should be associated with a validated release baseline. This prevents partial deployment of incompatible components and allows operators to determine exactly which software configuration is intended for a particular robot model or fleet group.

ROS 2 package distribution can integrate naturally with OTA by publishing validated packages to controlled public or private repositories. Robot computers can obtain versioned packages through standard package-management mechanisms while the OTA controller determines when and where installation occurs. Package repositories provide dependency resolution and version management, whereas the OTA layer adds fleet targeting, scheduling, policy enforcement, health verification, and recovery around the package installation process.

Container-based robots can instead distribute application updates as immutable container images. CI/CD builds and validates an image, publishes it to an authorized registry, and records its digest in a release manifest. Robots then retrieve exactly that artifact rather than rebuilding software locally. Docker Compose, Kubernetes, or K3s can activate the new image while external configuration remains separated, providing a clear boundary between validated application software and robot-specific operational parameters.

An OTA update should progress through staged deployment rather than immediately reaching every robot. A candidate release may first pass CI, simulation, hardware-in-the-loop testing, laboratory validation, and a limited pilot deployment. After operational health is confirmed, the release can expand to larger fleet groups. Progressive rollout reduces the number of robots exposed to an unexpected defect and provides measurable checkpoints at which deployment can be stopped.

Fleet targeting requires explicit knowledge of robot identity and compatibility. Update policies may consider robot model, hardware revision, processor architecture, ROS 2 distribution, sensor configuration, GPU platform, geographic deployment group, or operational role. A package compiled for an x86 edge computer must not be delivered to an incompatible ARM target, and a configuration intended for one sensor suite should not silently replace parameters on a different robot variant.

Pre-update validation should determine whether the robot is in an appropriate state for software modification. The system may verify battery level, charging state, mission status, network quality, available storage, package compatibility, hardware readiness, and current software version. Motion-related updates should normally occur only when the robot is stopped or placed into a controlled maintenance state. OTA orchestration must therefore understand robot operational state rather than treating the robot as an ordinary unattended server.

Update artifacts must be authenticated and integrity-checked before installation. Signed packages, trusted repositories, secure container registries, checksums, certificates, and verified manifests can establish confidence that downloaded software is authorized and unchanged. Transport encryption protects communication in transit, but artifact verification remains necessary because OTA security depends on both secure delivery and trustworthy software provenance across the entire supply chain.

Download and installation should be designed for unreliable connectivity. Mobile robots may move between access points, operate over private 5G, experience temporary wireless loss, or work in environments with limited bandwidth. Large container images, AI models, or system images should support resumable or cached transfer where the selected update technology permits it. A temporary network interruption should delay deployment rather than leave the robot in an undefined software state.

Atomicity is an important property for critical updates. If several packages or configuration elements must change together, installing only part of the release can create incompatible interfaces or dependencies. Container image replacement naturally provides a stronger immutable unit, while package-based systems require careful transaction and dependency management. System-level A/B partition schemes can provide even stronger atomic rollback by maintaining both current and candidate operating-system environments.

ROS 2 lifecycle management can coordinate application shutdown and restart during an update. Managed nodes may transition from active operation through deactivation, cleanup, and shutdown before their software is replaced. After installation, nodes can be reconfigured and activated in a controlled sequence. This is safer than abruptly terminating processes because hardware interfaces, navigation components, and other stateful subsystems can release resources and preserve expected system transitions.

Post-update validation must verify robot functionality rather than merely confirm successful package installation. The system can check container health, process startup, ROS 2 lifecycle states, DDS discovery, required topics, services, actions, sensor availability, diagnostics, resource utilization, and hardware communication. A deployment should be considered successful only when the robot reaches a defined operational state and satisfies the health criteria associated with the release.

Rollback must be designed before deployment begins. The OTA system should retain sufficient information to restore the previous validated package set, container image digest, configuration revision, or system partition. Automatic rollback can be triggered when startup fails or health checks exceed predefined limits, while manual rollback may be initiated by an operator. The previous state must itself be known and validated rather than reconstructed from uncertain local files.

Configuration updates require particular caution because parameters can change robot behavior without modifying executable software. Velocity limits, navigation parameters, sensor calibration, safety zones, DDS settings, and AI thresholds may significantly influence system operation. Configuration should therefore be versioned, validated, associated with compatible software releases, and included in rollback planning. Secrets and robot identities should remain separated from ordinary configuration packages whenever possible.

AI-enabled robots add model artifacts to the OTA architecture. Perception networks, foundation-model adapters, policy models, embeddings, or other learned components may be much larger than conventional ROS 2 packages and may have strict GPU or runtime compatibility requirements. Model versions should be associated with the application version, expected accelerator, preprocessing pipeline, and validation results so that updating an AI model does not silently invalidate the surrounding ROS 2 software stack.

OTA should distinguish application recovery from functional safety. Restarting a failed container or reverting a package does not itself guarantee that the physical robot is safe. Independent safety controllers, emergency-stop functions, watchdogs, motion limits, and safe-state mechanisms must remain effective during update and recovery operations. Software deployment should never bypass the safety architecture merely because the robot is temporarily operating in maintenance mode.

Observability is essential throughout the OTA process. The management system should record download progress, installation events, software versions, configuration revisions, health results, rollback actions, and final deployment state. ROS 2 diagnostics and infrastructure telemetry can be correlated with update events to determine whether a newly introduced release causes increased crashes, communication failures, latency, CPU load, GPU pressure, or sensor initialization problems.

Fleet-scale deployment requires bandwidth and infrastructure planning. Updating hundreds of robots simultaneously can overload wireless networks, registries, edge servers, or internet connections. Updates can therefore be scheduled in waves, distributed through local caches, or staged on on-premise infrastructure before installation. Edge-local distribution can reduce external bandwidth consumption while preserving centralized release governance and software-version visibility across the fleet.

CI/CD and OTA should form one continuous software-delivery chain. CI validates source code, builds ROS 2 packages or containers, performs security checks, executes simulation and hardware tests, and produces immutable release artifacts. OTA consumes these already validated artifacts and controls their deployment to physical robots. Rebuilding software during the OTA stage should be avoided because it breaks the direct relationship between the artifact that was tested and the artifact that is deployed.

Release metadata should provide end-to-end traceability from a physical robot back to the originating source revision. Operators should be able to identify the installed ROS 2 package versions, container digests, configuration revision, model version, deployment time, and release identifier for each robot. This information supports field troubleshooting, vulnerability response, regulatory evidence, maintenance planning, and reconstruction of historical robot configurations.

A mature OTA architecture therefore combines ROS 2 package management, container deployment, lifecycle coordination, security verification, staged rollout, health monitoring, rollback, configuration control, and fleet-level observability. The objective is not simply to transfer new files to a robot, but to move a physical system safely and reproducibly from one validated software baseline to another while maintaining a complete record of the transition.

##  

## 08.07 ROS2 Log Aggregation: ELK Stack Integration [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 log aggregation provides a centralized mechanism for collecting, storing, searching, and analyzing runtime information generated by distributed robot software. A single robot may execute many nodes across several processes or containers, while a fleet may contain hundreds of such systems. Without centralized aggregation, troubleshooting requires manually inspecting individual machines. An ELK-based architecture converts these fragmented records into searchable operational evidence.

ROS 2 applications generate logs through the ROS client libraries and logging infrastructure, typically including timestamps, severity levels, logger names, node context, and application messages. Additional information may come from operating-system services, containers, device drivers, middleware, safety processes, and AI workloads. Effective observability requires these heterogeneous sources to be correlated rather than treating ROS 2 application logs as an isolated diagnostic channel.

The ELK stack traditionally combines Elasticsearch, Logstash, and Kibana. Elasticsearch provides distributed indexing, storage, and search; Logstash performs ingestion, parsing, filtering, and transformation; Kibana provides visualization, exploration, and dashboards. In practical deployments, lightweight collection agents such as Beats or similar forwarders may gather logs on robot computers and transmit them to centralized or edge-hosted ingestion services.

Log collection should begin as close as practical to the source while minimizing interference with robot workloads. ROS 2 console output, log files, systemd journal records, container stdout and stderr streams, kernel messages, and application-specific files can be captured by local agents. The collector should operate with bounded CPU, memory, disk, and network consumption because diagnostic infrastructure must not compete significantly with navigation, perception, control, or safety-related computation.

Structured logging greatly improves aggregation quality. Instead of relying only on free-form text, records can include fields such as timestamp, robot_id, node_name, package, severity, process_id, container_id, software_version, mission_id, and subsystem. Structured fields allow operators to filter and correlate events across large fleets. A message can then be analyzed not merely as text but as an event associated with a particular robot, release, mission, and software component.

Time synchronization is fundamental when logs from multiple computers must be correlated. Robot computers, edge servers, sensors, and centralized infrastructure should maintain sufficiently consistent clocks through mechanisms such as NTP or PTP according to system requirements. Incorrect timestamps can make causal analysis misleading, particularly when investigating DDS communication failures, sensor delays, distributed state transitions, or events that occurred within milliseconds across several ROS 2 processes.

Log severity should be used consistently across packages. DEBUG messages support detailed development analysis, INFO records describe normal operational transitions, WARN indicates abnormal but potentially recoverable conditions, ERROR identifies significant failures, and FATAL represents conditions preventing continued operation of a component. Poor severity discipline produces either excessive noise or insufficient evidence, reducing the effectiveness of automated searching and operational dashboards.

A local forwarding layer should tolerate temporary network loss because mobile robots cannot assume continuous connectivity to centralized logging infrastructure. Logs can be buffered on local storage and transmitted when connectivity returns. Buffer limits and retention policies are necessary to prevent diagnostic data from consuming the robot filesystem indefinitely. Critical events may receive higher retention or transmission priority than verbose debugging information.

Logstash or an equivalent ingestion pipeline can normalize records arriving from different sources. Parsers can extract ROS 2 metadata, convert timestamps, identify robot and subsystem fields, classify events, enrich records with deployment information, and discard unnecessary content. Consistent normalization allows logs from ROS 2 nodes, Docker containers, Kubernetes workloads, operating-system services, and edge applications to be searched using a common operational vocabulary.

Elasticsearch stores normalized events in indexes designed for efficient search and aggregation. Index organization should consider time range, fleet size, robot identity, environment, and retention requirements. High-frequency logs can create substantial storage demand, so index lifecycle policies can move older records, reduce retention, or delete data according to operational value. Logging architecture must therefore balance diagnostic depth against storage, bandwidth, and infrastructure cost.

Kibana provides an interactive layer for investigating robot behavior. Dashboards can summarize error rates, warning counts, node failures, restart events, communication problems, software versions, and fleet-wide event patterns. Operators can begin with a high-level fleet view and then filter to a specific robot, mission, subsystem, container, or time interval. This significantly reduces the time required to locate relevant evidence during field incidents.

ROS 2 logs become more valuable when correlated with diagnostics and lifecycle state. A navigation failure may coincide with a sensor warning, DDS discovery problem, lifecycle transition, CPU overload, or container restart. Recording common identifiers and synchronized timestamps allows these events to be reconstructed as one operational sequence. Central aggregation therefore supports root-cause analysis that would be difficult from isolated node log files.

Containerized ROS 2 deployments require awareness of container logging behavior. Application output may be captured by Docker logging drivers or Kubernetes container-log mechanisms before being forwarded to the aggregation pipeline. Container identity, image version, Pod name, namespace, and node information should be retained because application failures may depend on the deployment environment rather than the ROS 2 node implementation itself.

For Kubernetes or K3s robot infrastructure, collectors can operate on cluster nodes and gather logs from multiple workloads. Kubernetes metadata can enrich each record with Pod, Deployment, namespace, and physical-node identity. This allows an operator to distinguish an application exception from a container restart, scheduling event, resource eviction, or infrastructure failure while still correlating those events with ROS 2 diagnostics and robot behavior.

Fleet-wide logging requires explicit robot identity. Every record should be attributable to the physical system that generated it even when identical container images and ROS 2 packages execute across the fleet. Robot identifiers, hardware variants, site information, and software release versions can be added during collection or ingestion. These dimensions allow comparisons such as whether an error affects one robot, one hardware revision, one site, or an entire software release.

Release metadata should be connected to operational logs so that CI/CD and OTA deployments can be evaluated after installation. When a new ROS 2 package or container version is released, log analysis can reveal whether warning frequency, crashes, initialization failures, or communication errors increase. Deployment events should therefore be searchable alongside application events, enabling operators to compare behavior before and after a software update.

Security and privacy must be considered before centralizing robot logs. Messages can accidentally contain credentials, tokens, network information, user data, or proprietary operational details. Logging policies should prevent sensitive values from being emitted whenever possible, while ingestion filters can redact selected fields. Transport encryption, authenticated agents, role-based access, index permissions, and controlled retention protect centralized diagnostic information.

Log aggregation should complement metrics rather than replace them. Logs provide detailed event context, while metrics efficiently represent continuous numerical behavior such as CPU load, memory usage, latency, message rate, temperature, or GPU utilization. A monitoring dashboard may identify an abnormal metric first and then link the operator to logs from the same robot and time interval. Together they provide stronger observability than either mechanism alone.

Alerts can be generated from recurring or high-severity log patterns, but alert rules should avoid reacting to every isolated message. Useful conditions may include repeated node crashes, persistent DDS discovery failures, sensor initialization errors, excessive restart counts, or a sudden fleet-wide increase in ERROR events after deployment. Alerting should identify actionable conditions while preserving the underlying records for later investigation and correlation.

Retention policies should reflect the operational importance of different data. High-volume DEBUG logs may be retained briefly, while ERROR, FATAL, deployment, security, and safety-related events may require longer storage. Older records can be archived to lower-cost storage when long-term evidence is needed. A deliberate lifecycle policy prevents Elasticsearch from becoming an unlimited data store while preserving information required for maintenance and incident analysis.

A scalable architecture may aggregate logs hierarchically. Robots can buffer locally, site edge servers can receive and preprocess records, and an on-premise or cloud platform can maintain long-term fleet indexes and dashboards. This structure reduces dependence on continuous external connectivity and allows local troubleshooting to continue during network isolation while still providing centralized visibility when communication is available.

A mature ROS 2 logging architecture therefore connects application logs, operating-system events, container records, deployment metadata, robot identity, synchronized time, secure transport, centralized indexing, visualization, and alerting. ELK integration transforms individual diagnostic messages into fleet-level operational knowledge, enabling developers and operators to trace failures from a dashboard back to the exact robot, node, software release, and event sequence that produced them.

ROS 2 로그 집계(Log Aggregation)는 분산된 로봇 소프트웨어에서 생성되는 런타임 정보를 중앙에서 수집, 저장, 검색, 분석하기 위한 메커니즘을 제공한다. 하나의 로봇에서도 여러 프로세스나 컨테이너에 걸쳐 많은 노드가 실행될 수 있으며, 플릿(Fleet)은 수백 대의 이러한 시스템으로 구성될 수 있다. 중앙 집중식 집계가 없다면 문제 해결을 위해 개별 컴퓨터를 직접 확인해야 한다. ELK 기반 아키텍처는 이렇게 분산된 기록을 검색 가능한 운영 증거(Operational Evidence)로 변환한다.

ROS 2 애플리케이션은 ROS 클라이언트 라이브러리(Client Library)와 로깅 인프라(Logging Infrastructure)를 통해 로그를 생성하며, 일반적으로 타임스탬프(Timestamp), 심각도 수준(Severity Level), 로거 이름(Logger Name), 노드 컨텍스트(Node Context), 애플리케이션 메시지를 포함한다. 추가 정보는 운영체제 서비스, 컨테이너, 장치 드라이버, 미들웨어(Middleware), 안전 프로세스, AI 워크로드에서 생성될 수 있다. 효과적인 관찰 가능성(Observability)을 위해서는 ROS 2 애플리케이션 로그를 독립된 진단 채널로 취급하지 않고 이러한 이기종 소스를 서로 연계해야 한다.

ELK 스택(ELK Stack)은 전통적으로 엘라스틱서치(Elasticsearch), 로그스태시(Logstash), 키바나(Kibana)를 결합한다. 엘라스틱서치는 분산 인덱싱(Distributed Indexing), 저장, 검색을 제공하고, 로그스태시는 수집(Ingestion), 파싱(Parsing), 필터링, 변환을 수행하며, 키바나는 시각화, 탐색, 대시보드를 제공한다. 실제 배포에서는 비츠(Beats) 또는 유사한 경량 포워더(Forwarder)가 로봇 컴퓨터에서 로그를 수집하여 중앙 또는 엣지 기반 수집 서비스로 전달할 수 있다.

로그 수집(Log Collection)은 로봇 워크로드에 미치는 영향을 최소화하면서 가능한 한 데이터 소스 가까이에서 시작해야 한다. ROS 2 콘솔 출력, 로그 파일, systemd 저널 기록, 컨테이너 표준 출력(stdout)과 표준 오류(stderr) 스트림, 커널 메시지, 애플리케이션별 파일을 로컬 에이전트(Local Agent)가 수집할 수 있다. 진단 인프라가 내비게이션, 인지(Perception), 제어(Control), 안전 관련 연산과 크게 경쟁하지 않도록 수집기의 CPU, 메모리, 디스크, 네트워크 사용량을 제한해야 한다.

구조화된 로깅(Structured Logging)은 로그 집계 품질을 크게 향상시킨다. 자유 형식 텍스트에만 의존하는 대신 timestamp, robot_id, node_name, package, severity, process_id, container_id, software_version, mission_id, subsystem 등의 필드를 레코드에 포함할 수 있다. 구조화된 필드를 사용하면 운영자가 대규모 플릿의 이벤트를 필터링하고 연계할 수 있으며, 메시지를 단순한 텍스트가 아니라 특정 로봇, 릴리스, 임무, 소프트웨어 구성요소와 연결된 이벤트로 분석할 수 있다.

여러 컴퓨터에서 생성된 로그를 연계해야 할 경우 시간 동기화(Time Synchronization)는 필수적이다. 로봇 컴퓨터, 엣지 서버, 센서, 중앙 인프라는 시스템 요구사항에 따라 NTP 또는 PTP와 같은 메커니즘을 사용하여 충분히 일관된 시계를 유지해야 한다. 잘못된 타임스탬프는 DDS 통신 장애, 센서 지연, 분산 상태 전환 또는 여러 ROS 2 프로세스에서 밀리초 단위로 발생한 이벤트를 조사할 때 인과관계 분석(Causal Analysis)을 왜곡할 수 있다.

로그 심각도(Log Severity)는 패키지 전체에서 일관되게 사용해야 한다. DEBUG 메시지는 상세한 개발 분석을 지원하고, INFO 기록은 정상적인 운영 상태 전환을 설명하며, WARN은 비정상이지만 복구 가능성이 있는 상태를 나타낸다. ERROR는 중대한 장애를 나타내고 FATAL은 구성요소가 계속 동작할 수 없는 상태를 의미한다. 심각도 관리가 부적절하면 지나치게 많은 잡음 또는 부족한 진단 정보가 발생하여 자동 검색과 운영 대시보드의 효과가 감소한다.

로컬 포워딩 계층(Local Forwarding Layer)은 이동 로봇이 중앙 로깅 인프라와 지속적으로 연결되어 있다고 가정할 수 없으므로 일시적인 네트워크 손실을 견딜 수 있어야 한다. 로그를 로컬 저장소에 버퍼링(Buffering)하고 연결이 복구되면 전송할 수 있다. 진단 데이터가 로봇 파일 시스템을 무제한으로 소비하지 않도록 버퍼 크기와 보존 정책(Retention Policy)을 설정해야 한다. 중요 이벤트에는 상세 디버깅 정보보다 높은 보존 또는 전송 우선순위를 부여할 수 있다.

로그스태시 또는 동등한 수집 파이프라인(Ingestion Pipeline)은 서로 다른 소스에서 전달되는 레코드를 정규화(Normalization)할 수 있다. 파서는 ROS 2 메타데이터를 추출하고 타임스탬프를 변환하며 로봇 및 서브시스템 필드를 식별하고 이벤트를 분류하며 배포 정보를 추가하고 불필요한 내용을 제거할 수 있다. 일관된 정규화를 통해 ROS 2 노드, 도커(Docker) 컨테이너, 쿠버네티스(Kubernetes) 워크로드, 운영체제 서비스, 엣지 애플리케이션의 로그를 공통 운영 용어로 검색할 수 있다.

엘라스틱서치는 정규화된 이벤트를 효율적인 검색과 집계를 위해 설계된 인덱스(Index)에 저장한다. 인덱스 구성은 시간 범위, 플릿 규모, 로봇 식별 정보, 운영 환경, 보존 요구사항을 고려해야 한다. 고빈도 로그는 상당한 저장공간을 요구할 수 있으므로 인덱스 수명주기 정책(Index Lifecycle Policy)을 이용해 오래된 레코드를 이동하거나 보존 기간을 단축하거나 운영 가치에 따라 삭제할 수 있다. 따라서 로깅 아키텍처는 진단 깊이와 저장공간, 대역폭, 인프라 비용 사이의 균형을 유지해야 한다.

키바나는 로봇 동작을 조사하기 위한 대화형 계층(Interactive Layer)을 제공한다. 대시보드는 오류 발생률, 경고 횟수, 노드 장애, 재시작 이벤트, 통신 문제, 소프트웨어 버전, 플릿 전체의 이벤트 패턴을 요약할 수 있다. 운영자는 상위 수준의 플릿 화면에서 시작하여 특정 로봇, 임무, 서브시스템, 컨테이너 또는 시간 구간으로 필터링할 수 있다. 이를 통해 현장 장애 발생 시 관련 증거를 찾는 데 필요한 시간을 크게 줄일 수 있다.

ROS 2 로그는 진단(Diagnostics) 및 라이프사이클 상태(Lifecycle State)와 연계될 때 더 큰 가치를 가진다. 내비게이션 장애는 센서 경고, DDS 디스커버리(Discovery) 문제, 라이프사이클 전환, CPU 과부하 또는 컨테이너 재시작과 동시에 발생할 수 있다. 공통 식별자와 동기화된 타임스탬프를 기록하면 이러한 이벤트를 하나의 운영 시퀀스(Operational Sequence)로 재구성할 수 있다. 따라서 중앙 집중식 집계는 개별 노드 로그 파일만으로는 어려운 근본 원인 분석(Root-Cause Analysis)을 지원한다.

컨테이너화된 ROS 2 배포(Containerized ROS 2 Deployment)에서는 컨테이너 로깅 동작을 고려해야 한다. 애플리케이션 출력은 집계 파이프라인으로 전달되기 전에 도커 로깅 드라이버(Docker Logging Driver) 또는 쿠버네티스 컨테이너 로그 메커니즘에 의해 수집될 수 있다. 애플리케이션 장애가 ROS 2 노드 구현 자체가 아니라 배포 환경에 따라 발생할 수 있으므로 컨테이너 식별자, 이미지 버전, 파드(Pod) 이름, 네임스페이스(Namespace), 노드 정보를 유지해야 한다.

쿠버네티스 또는 K3s 기반 로봇 인프라에서는 수집기(Collector)가 클러스터 노드에서 동작하면서 여러 워크로드의 로그를 수집할 수 있다. 쿠버네티스 메타데이터를 이용하여 각 레코드에 파드, 디플로이먼트(Deployment), 네임스페이스, 물리 노드 식별 정보를 추가할 수 있다. 이를 통해 운영자는 애플리케이션 예외와 컨테이너 재시작, 스케줄링 이벤트, 자원 축출(Resource Eviction), 인프라 장애를 구분하면서 이러한 이벤트를 ROS 2 진단 및 로봇 동작과 연계할 수 있다.

플릿 전체 로깅(Fleet-Wide Logging)에는 명확한 로봇 식별 정보가 필요하다. 동일한 컨테이너 이미지와 ROS 2 패키지가 플릿 전체에서 실행되더라도 모든 레코드는 해당 정보를 생성한 물리 시스템과 연결될 수 있어야 한다. 로봇 식별자, 하드웨어 변형(Hardware Variant), 사이트 정보, 소프트웨어 릴리스 버전을 수집 또는 수집 처리 과정에서 추가할 수 있다. 이를 통해 특정 오류가 하나의 로봇, 특정 하드웨어 리비전, 특정 사이트 또는 전체 소프트웨어 릴리스에 영향을 주는지 비교할 수 있다.

릴리스 메타데이터(Release Metadata)는 운영 로그와 연결되어 CI/CD 및 OTA 배포 결과를 설치 이후에도 평가할 수 있어야 한다. 새로운 ROS 2 패키지나 컨테이너 버전이 릴리스되면 로그 분석을 통해 경고 빈도, 충돌(Crash), 초기화 실패, 통신 오류가 증가했는지 확인할 수 있다. 따라서 배포 이벤트를 애플리케이션 이벤트와 함께 검색할 수 있도록 하여 운영자가 소프트웨어 업데이트 전후의 동작을 비교할 수 있어야 한다.

로봇 로그를 중앙에 집중하기 전에 보안(Security)과 개인정보 보호(Privacy)를 고려해야 한다. 메시지에는 실수로 자격 증명, 토큰, 네트워크 정보, 사용자 데이터 또는 독점적인 운영 정보가 포함될 수 있다. 가능한 경우 로깅 정책을 통해 민감한 값이 출력되지 않도록 해야 하며 수집 필터(Ingestion Filter)를 이용해 특정 필드를 마스킹하거나 제거할 수 있다. 전송 암호화, 인증된 에이전트, 역할 기반 접근(Role-Based Access), 인덱스 권한, 통제된 보존 정책을 통해 중앙 진단 정보를 보호해야 한다.

로그 집계는 메트릭(Metrics)을 대체하는 것이 아니라 보완해야 한다. 로그는 상세한 이벤트 컨텍스트를 제공하는 반면 메트릭은 CPU 부하, 메모리 사용량, 지연시간(Latency), 메시지 전송률, 온도, GPU 사용률과 같은 연속적인 수치 동작을 효율적으로 표현한다. 모니터링 대시보드가 먼저 비정상적인 메트릭을 식별한 후 운영자를 동일한 로봇과 시간 구간의 로그로 연결할 수 있다. 두 메커니즘을 함께 사용하면 각각을 독립적으로 사용하는 것보다 강력한 관찰 가능성을 제공한다.

반복되거나 높은 심각도의 로그 패턴을 기반으로 경고(Alert)를 생성할 수 있지만 모든 개별 메시지에 반응하도록 경고 규칙을 구성해서는 안 된다. 반복적인 노드 충돌, 지속적인 DDS 디스커버리 실패, 센서 초기화 오류, 과도한 재시작 횟수 또는 배포 이후 플릿 전체에서 ERROR 이벤트가 급증하는 상황 등이 유용한 조건이 될 수 있다. 경고 시스템은 실제 대응이 필요한 상태를 식별하면서 이후 조사와 연계를 위해 기본 로그 기록을 보존해야 한다.

보존 정책은 서로 다른 데이터의 운영 중요도를 반영해야 한다. 대량으로 발생하는 DEBUG 로그는 짧게 보존할 수 있지만 ERROR, FATAL, 배포, 보안, 안전 관련 이벤트는 더 오랫동안 저장해야 할 수 있다. 장기적인 증거가 필요한 경우 오래된 기록을 저비용 저장소로 아카이빙(Archiving)할 수 있다. 체계적인 수명주기 정책(Lifecycle Policy)은 필요한 유지보수 및 장애 분석 정보를 보존하면서 엘라스틱서치가 무제한 데이터 저장소가 되는 것을 방지한다.

확장 가능한 아키텍처에서는 로그를 계층적으로 집계할 수 있다. 로봇은 로그를 로컬에서 버퍼링하고, 사이트 엣지 서버(Site Edge Server)는 레코드를 수신하여 전처리하며, 온프레미스(On-Premise) 또는 클라우드 플랫폼은 장기 플릿 인덱스와 대시보드를 유지할 수 있다. 이러한 구조는 지속적인 외부 네트워크 연결에 대한 의존성을 줄이고 네트워크가 격리된 상황에서도 로컬 문제 해결을 지속하면서 통신이 가능할 때 중앙 집중식 가시성을 제공한다.

따라서 성숙한 ROS 2 로깅 아키텍처는 애플리케이션 로그, 운영체제 이벤트, 컨테이너 기록, 배포 메타데이터, 로봇 식별 정보, 동기화된 시간, 보안 전송, 중앙 집중식 인덱싱, 시각화, 경고를 하나의 체계로 연결한다. ELK 통합은 개별 진단 메시지를 플릿 수준의 운영 지식(Operational Knowledge)으로 변환하여 개발자와 운영자가 대시보드에서 장애를 추적하고 해당 장애를 발생시킨 정확한 로봇, 노드, 소프트웨어 릴리스, 이벤트 시퀀스까지 분석할 수 있도록 한다.

##  

## 08.08 ROS2 Metrics Monitoring: Prometheus / Grafana [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS 2 metrics monitoring provides continuous numerical visibility into the health and performance of robot software, computing resources, communication paths, and physical subsystems. Unlike logs, which describe individual events in detail, metrics represent behavior over time through measurable values. Prometheus and Grafana can combine these measurements into an operational monitoring architecture suitable for a single robot, distributed edge systems, or large multi-robot fleets.

A useful monitoring architecture begins by defining what must be measured rather than collecting every available value. Robot-level indicators may include CPU and memory utilization, disk usage, network traffic, temperature, battery state, and GPU load. ROS 2 indicators can include node availability, message rates, communication latency, callback execution behavior, DDS status, lifecycle state, and application-specific performance values associated with navigation, perception, control, or AI workloads.

Prometheus is designed around time-series metrics identified by metric names and labels. Each sample represents a numerical value at a particular time, while labels describe dimensions such as robot identity, node, subsystem, software version, site, or deployment environment. This model allows the same metric definition to be reused across many robots while still supporting filtering, aggregation, comparison, and fleet-level analysis without creating separate monitoring logic for every system.

Metrics are commonly exposed through exporters or application endpoints that Prometheus can scrape at configured intervals. Standard exporters can report Linux host and container information, while custom exporters can translate ROS 2 diagnostics or application state into Prometheus-compatible measurements. ROS 2 nodes may also expose selected application metrics directly through a monitoring component, allowing robot-specific information to enter the same time-series infrastructure as operating-system data.

Metric types should reflect the behavior being represented. Gauges are appropriate for values that increase and decrease, such as temperature, battery percentage, memory usage, or queue depth. Counters represent cumulative events such as message errors, node restarts, or failed requests. Histograms and summaries can characterize distributions of latency or execution time, allowing monitoring to capture not only average performance but also slow or abnormal behavior.

Label design requires particular care in large robot deployments. Labels such as robot_id, node_name, subsystem, site, software_version, and hardware_variant can provide valuable operational dimensions. However, labels containing highly variable values such as timestamps, message identifiers, file names, or arbitrary error strings can create excessive time-series cardinality. Uncontrolled cardinality increases memory, storage, and query cost and can destabilize the monitoring infrastructure itself.

ROS 2 communication metrics can reveal problems that ordinary host monitoring cannot detect. Topic publication frequency, subscription behavior, message delay, dropped samples, deadline violations, DDS discovery state, and selected QoS-related events can provide evidence about communication health. These measurements should be interpreted with application context because low message frequency may be normal for an event-driven topic but indicate failure for a high-rate sensor stream.

Node and lifecycle monitoring provide another important dimension. A process may still exist while a ROS 2 component is inactive, unconfigured, degraded, or unable to communicate with required peers. Monitoring should therefore distinguish operating-system process availability from ROS 2 functional readiness. Lifecycle state, diagnostic status, required topic presence, and subsystem-specific readiness indicators can provide a more accurate representation of whether the robot is actually capable of performing its mission.

Infrastructure exporters provide visibility into the computing platform beneath ROS 2. CPU utilization, load average, memory pressure, filesystem capacity, network throughput, process statistics, and thermal information can identify resource limitations before they become application failures. GPU-equipped robots and edge servers should additionally monitor accelerator utilization, memory consumption, temperature, power behavior, and relevant runtime errors for perception and AI inference workloads.

Containerized deployments add container-level dimensions to monitoring. Docker or Kubernetes metrics can expose container CPU and memory usage, restart counts, resource limits, Pod state, scheduling information, and node health. These infrastructure measurements can be correlated with ROS 2 metrics so that an operator can determine whether degraded navigation or perception originated from application behavior, container resource constraints, or the underlying host computer.

Prometheus normally discovers targets and periodically scrapes their metric endpoints. In dynamic Kubernetes or K3s environments, service discovery can automatically identify workloads as they appear or move. Robot fleets operating through intermittent wireless connections require additional design consideration because centralized scraping may not always reach every robot. Edge aggregation, local Prometheus instances, remote-write mechanisms, or gateways can be used where the network architecture requires hierarchical collection.

Grafana provides visualization and exploration over the collected time-series data. Dashboards can show fleet health, individual robot status, resource consumption, ROS 2 subsystem performance, communication behavior, and historical trends. A useful dashboard should move from summary to detail, allowing an operator to identify an abnormal fleet condition and progressively drill down to a particular robot, subsystem, node, container, or metric without navigating unrelated data.

Dashboard design should emphasize operational questions rather than displaying every measurement simultaneously. A fleet overview may show online robots, degraded systems, active missions, battery state, critical alerts, and software versions. A robot-level dashboard can then expose CPU, GPU, memory, network, ROS 2 diagnostics, sensor rates, and navigation state. Specialized dashboards can provide deeper views for perception, localization, motion control, AI inference, or infrastructure engineering.

Alerting converts selected metric conditions into actionable operational signals. Examples include persistent CPU saturation, rapidly decreasing disk capacity, abnormal GPU temperature, missing sensor data, repeated node restarts, excessive communication latency, or a robot remaining in an unexpected lifecycle state. Alert conditions should normally include duration and context so that brief transient behavior does not generate unnecessary notifications while sustained faults receive appropriate attention.

Prometheus alerting can be integrated with an alert-management layer that groups, routes, suppresses, and deduplicates notifications. Fleet-scale systems need this capability because one infrastructure failure can otherwise produce hundreds of nearly identical robot alerts. Grouping related events by site, subsystem, or failure source helps operators recognize a common root cause rather than treating every affected robot as an independent incident.

Metrics and logs provide complementary observability. A Grafana panel may show that navigation latency increased immediately after a software deployment, while centralized logs from the same time interval reveal DDS warnings, container restarts, or sensor errors. Shared dimensions such as robot ID, software version, deployment identifier, and synchronized timestamps allow operators to move between numerical trends and detailed events during root-cause analysis.

Time synchronization remains important even for time-series monitoring. Metrics collected from robot computers, edge servers, and centralized infrastructure must represent sufficiently consistent time to support correlation with logs, traces, deployment events, and physical robot behavior. NTP may be adequate for many infrastructure metrics, while systems requiring precise timing analysis may use PTP or hardware-supported synchronization for measurements associated with sensors and real-time communication.

Retention and sampling policies should reflect the value and cost of metrics. High-frequency measurements may be useful during development but expensive to retain across a large fleet for months. Scrape intervals, aggregation rules, downsampling, and long-term storage can be selected according to operational requirements. Recent high-resolution data can support troubleshooting while older aggregated data supports capacity planning, reliability analysis, and long-term performance comparisons.

Security must protect both metric endpoints and the centralized monitoring platform. Metrics can expose host names, network topology, software versions, hardware information, operational status, or business-sensitive fleet activity. Authentication, transport encryption, network segmentation, access control, and dashboard permissions should therefore be applied according to deployment requirements. Monitoring components should receive only the privileges required to observe their intended resources.

CI/CD and OTA information can enrich monitoring with release context. Dashboards can annotate the time at which a ROS 2 package, container image, configuration revision, or AI model was deployed. Operators can then compare system behavior before and after the change. If error rates, latency, CPU usage, or restart frequency deteriorate following a rollout, monitoring evidence can support deployment suspension or rollback decisions.

Monitoring should also respect the boundary between operational observability and functional safety. Prometheus and Grafana can detect and visualize abnormal behavior, but they are not substitutes for deterministic safety controllers, emergency-stop systems, watchdogs, or certified protection mechanisms. A robot must enter an appropriate safe state through its safety architecture even if the monitoring network, Prometheus server, or visualization system is unavailable.

Fleet-scale monitoring can use a hierarchical architecture in which each robot exposes local metrics, site or edge infrastructure aggregates operational data, and an on-premise or cloud platform provides long-term fleet visibility. Essential robot operation should remain independent of centralized monitoring availability. When connectivity is lost, robots continue their local mission and safety functions while monitoring data can be buffered, aggregated, or synchronized after communication is restored.

A mature ROS 2 metrics architecture therefore connects robot applications, DDS communication, lifecycle state, host resources, containers, GPUs, deployment metadata, Prometheus collection, alert management, and Grafana visualization. Combined with centralized logging, it provides a unified observability foundation that allows developers and operators to detect degradation, analyze trends, compare releases, diagnose failures, and understand the operational behavior of individual robots and entire fleets.

ROS 2 메트릭 모니터링(Metrics Monitoring)은 로봇 소프트웨어, 컴퓨팅 자원, 통신 경로, 물리적 서브시스템의 상태와 성능을 연속적인 수치로 파악할 수 있도록 한다. 개별 이벤트를 상세하게 설명하는 로그(Log)와 달리 메트릭(Metrics)은 측정 가능한 값을 통해 시간에 따른 동작을 표현한다. 프로메테우스(Prometheus)와 그라파나(Grafana)는 이러한 측정값을 결합하여 단일 로봇, 분산 엣지 시스템, 대규모 다중 로봇 플릿에 적용할 수 있는 운영 모니터링 아키텍처를 구성할 수 있다.

유용한 모니터링 아키텍처는 사용 가능한 모든 값을 수집하는 것이 아니라 무엇을 측정해야 하는지를 정의하는 것에서 시작한다. 로봇 수준 지표에는 CPU 및 메모리 사용률, 디스크 사용량, 네트워크 트래픽, 온도, 배터리 상태, GPU 부하가 포함될 수 있다. ROS 2 지표에는 노드 가용성, 메시지 전송률, 통신 지연시간, 콜백 실행 동작, DDS 상태, 라이프사이클 상태, 내비게이션, 인지, 제어, AI 워크로드와 관련된 애플리케이션별 성능 값이 포함될 수 있다.

프로메테우스는 메트릭 이름과 레이블(Label)로 식별되는 시계열 메트릭(Time-Series Metrics)을 중심으로 설계된다. 각 샘플은 특정 시점의 수치 값을 나타내며 레이블은 로봇 식별자, 노드, 서브시스템, 소프트웨어 버전, 사이트 또는 배포 환경과 같은 차원을 설명한다. 이 모델을 사용하면 동일한 메트릭 정의를 여러 로봇에서 재사용하면서 각 시스템마다 별도의 모니터링 로직을 만들지 않고도 필터링, 집계, 비교, 플릿 수준 분석을 수행할 수 있다.

메트릭은 일반적으로 프로메테우스가 설정된 주기에 따라 스크레이프(Scrape)할 수 있는 익스포터(Exporter) 또는 애플리케이션 엔드포인트(Application Endpoint)를 통해 노출된다. 표준 익스포터는 Linux 호스트와 컨테이너 정보를 제공할 수 있으며, 사용자 정의 익스포터(Custom Exporter)는 ROS 2 진단 정보나 애플리케이션 상태를 프로메테우스 호환 측정값으로 변환할 수 있다. ROS 2 노드 역시 선택된 애플리케이션 메트릭을 모니터링 구성요소를 통해 직접 노출하여 로봇별 정보를 운영체제 데이터와 동일한 시계열 인프라에 통합할 수 있다.

메트릭 유형(Metric Type)은 표현하려는 동작 특성에 맞아야 한다. 게이지(Gauge)는 온도, 배터리 잔량, 메모리 사용량, 큐 깊이처럼 증가하거나 감소할 수 있는 값에 적합하다. 카운터(Counter)는 메시지 오류, 노드 재시작, 실패한 요청과 같은 누적 이벤트를 표현한다. 히스토그램(Histogram)과 서머리(Summary)는 지연시간이나 실행시간의 분포를 나타낼 수 있으므로 평균 성능뿐 아니라 느리거나 비정상적인 동작까지 모니터링할 수 있다.

대규모 로봇 배포에서는 레이블 설계(Label Design)에 특별한 주의가 필요하다. robot_id, node_name, subsystem, site, software_version, hardware_variant와 같은 레이블은 유용한 운영 차원을 제공한다. 그러나 타임스탬프, 메시지 식별자, 파일 이름, 임의의 오류 문자열처럼 매우 다양한 값을 가진 레이블은 과도한 시계열 카디널리티(Time-Series Cardinality)를 생성할 수 있다. 통제되지 않은 카디널리티는 메모리, 저장공간, 쿼리 비용을 증가시키고 모니터링 인프라 자체를 불안정하게 만들 수 있다.

ROS 2 통신 메트릭(Communication Metrics)은 일반적인 호스트 모니터링으로는 발견하기 어려운 문제를 보여줄 수 있다. 토픽 발행 주기, 구독 동작, 메시지 지연, 손실된 샘플, 데드라인 위반(Deadline Violation), DDS 디스커버리 상태, 선택된 서비스 품질(Quality of Service, QoS) 관련 이벤트를 통해 통신 상태에 대한 근거를 확보할 수 있다. 이벤트 기반 토픽에서는 낮은 메시지 주기가 정상일 수 있지만 고속 센서 스트림에서는 장애를 의미할 수 있으므로 이러한 측정값은 애플리케이션 컨텍스트와 함께 해석해야 한다.

노드 및 라이프사이클 모니터링(Node and Lifecycle Monitoring)은 또 다른 중요한 관찰 차원을 제공한다. 프로세스가 존재하더라도 ROS 2 구성요소가 비활성 상태이거나 설정되지 않았거나 성능이 저하되었거나 필요한 피어(Peer)와 통신하지 못할 수 있다. 따라서 운영체제 프로세스 가용성과 ROS 2 기능 준비 상태(Functional Readiness)를 구분해야 한다. 라이프사이클 상태, 진단 상태, 필수 토픽 존재 여부, 서브시스템별 준비 상태 지표를 통해 로봇이 실제로 임무를 수행할 수 있는지 더욱 정확하게 표현할 수 있다.

인프라 익스포터(Infrastructure Exporter)는 ROS 2 아래에서 동작하는 컴퓨팅 플랫폼에 대한 가시성을 제공한다. CPU 사용률, 로드 평균(Load Average), 메모리 압력, 파일 시스템 용량, 네트워크 처리량, 프로세스 통계, 열 상태를 통해 자원 제한이 애플리케이션 장애로 발전하기 전에 식별할 수 있다. GPU가 장착된 로봇과 엣지 서버에서는 인지 및 AI 추론 워크로드를 위해 가속기 사용률, 메모리 소비량, 온도, 전력 동작, 관련 런타임 오류도 추가로 모니터링해야 한다.

컨테이너화된 배포(Containerized Deployment)는 모니터링에 컨테이너 수준의 차원을 추가한다. 도커(Docker) 또는 쿠버네티스(Kubernetes) 메트릭은 컨테이너 CPU 및 메모리 사용량, 재시작 횟수, 자원 제한, 파드(Pod) 상태, 스케줄링 정보, 노드 상태를 제공할 수 있다. 이러한 인프라 측정값을 ROS 2 메트릭과 연계하면 내비게이션이나 인지 성능 저하가 애플리케이션 동작, 컨테이너 자원 제한 또는 기반 호스트 컴퓨터 중 어디에서 발생했는지 판단할 수 있다.

프로메테우스는 일반적으로 대상을 검색하고 설정된 주기에 따라 메트릭 엔드포인트를 스크레이프한다. 동적인 쿠버네티스 또는 K3s 환경에서는 서비스 디스커버리(Service Discovery)를 이용해 워크로드가 생성되거나 이동할 때 자동으로 식별할 수 있다. 간헐적인 무선 연결을 사용하는 로봇 플릿에서는 중앙 스크레이핑이 모든 로봇에 항상 접근하지 못할 수 있으므로 추가 설계가 필요하다. 네트워크 아키텍처에 따라 엣지 집계, 로컬 프로메테우스 인스턴스, 원격 쓰기(Remote Write) 메커니즘 또는 게이트웨이를 이용하여 계층적 수집을 구성할 수 있다.

그라파나는 수집된 시계열 데이터를 시각화하고 탐색하기 위한 기능을 제공한다. 대시보드는 플릿 상태, 개별 로봇 상태, 자원 소비, ROS 2 서브시스템 성능, 통신 동작, 과거 추세를 표시할 수 있다. 유용한 대시보드는 요약 정보에서 세부 정보로 이동할 수 있어야 하며, 운영자가 플릿의 비정상 상태를 식별한 후 관련 없는 데이터를 탐색하지 않고 특정 로봇, 서브시스템, 노드, 컨테이너 또는 메트릭까지 단계적으로 상세 분석할 수 있어야 한다.

대시보드 설계(Dashboard Design)는 모든 측정값을 동시에 표시하는 것보다 운영상의 질문에 초점을 맞춰야 한다. 플릿 개요에서는 온라인 로봇, 성능 저하 시스템, 활성 임무, 배터리 상태, 중요 경고, 소프트웨어 버전을 표시할 수 있다. 로봇 수준 대시보드에서는 CPU, GPU, 메모리, 네트워크, ROS 2 진단, 센서 전송률, 내비게이션 상태를 제공할 수 있다. 전문 대시보드는 인지, 위치추정(Localization), 모션 제어, AI 추론 또는 인프라 엔지니어링을 더욱 상세하게 보여줄 수 있다.

경고(Alerting)는 선택된 메트릭 조건을 실제 대응이 가능한 운영 신호로 변환한다. 지속적인 CPU 포화, 빠르게 감소하는 디스크 용량, 비정상적인 GPU 온도, 센서 데이터 누락, 반복적인 노드 재시작, 과도한 통신 지연 또는 예상하지 않은 라이프사이클 상태가 지속되는 로봇 등이 경고 조건이 될 수 있다. 짧은 일시적 동작으로 불필요한 알림이 발생하지 않도록 경고 조건에는 일반적으로 지속시간과 컨텍스트를 포함하고 지속적인 장애에 적절히 대응해야 한다.

프로메테우스 경고는 알림을 그룹화, 라우팅, 억제, 중복 제거하는 경고 관리 계층(Alert-Management Layer)과 통합할 수 있다. 플릿 규모 시스템에서는 하나의 인프라 장애가 수백 개의 거의 동일한 로봇 경고를 생성할 수 있으므로 이러한 기능이 필요하다. 사이트, 서브시스템 또는 장애 원인별로 관련 이벤트를 그룹화하면 운영자는 영향을 받은 모든 로봇을 개별 장애로 처리하는 대신 공통된 근본 원인(Root Cause)을 파악할 수 있다.

메트릭과 로그는 상호 보완적인 관찰 가능성(Observability)을 제공한다. 그라파나 패널에서 소프트웨어 배포 직후 내비게이션 지연시간이 증가한 것을 확인할 수 있으며, 동일한 시간 구간의 중앙 집중식 로그에서는 DDS 경고, 컨테이너 재시작 또는 센서 오류를 확인할 수 있다. 로봇 ID, 소프트웨어 버전, 배포 식별자, 동기화된 타임스탬프와 같은 공통 차원을 사용하면 근본 원인 분석 과정에서 수치 추세와 상세 이벤트 사이를 연결할 수 있다.

시계열 모니터링에서도 시간 동기화(Time Synchronization)는 중요하다. 로봇 컴퓨터, 엣지 서버, 중앙 인프라에서 수집된 메트릭은 로그, 트레이스(Trace), 배포 이벤트, 물리 로봇 동작과 연계할 수 있도록 충분히 일관된 시간을 표현해야 한다. 많은 인프라 메트릭에서는 NTP가 충분할 수 있지만 정밀한 타이밍 분석이 필요한 시스템에서는 센서 및 실시간 통신과 관련된 측정값을 위해 PTP 또는 하드웨어 지원 동기화(Hardware-Supported Synchronization)를 사용할 수 있다.

보존 및 샘플링 정책(Retention and Sampling Policy)은 메트릭의 가치와 비용을 반영해야 한다. 고주파 측정은 개발 과정에서는 유용하지만 대규모 플릿에서 수개월 동안 보존하면 비용이 크게 증가할 수 있다. 운영 요구사항에 따라 스크레이프 주기, 집계 규칙, 다운샘플링(Downsampling), 장기 저장소를 선택할 수 있다. 최근의 고해상도 데이터는 문제 해결에 활용하고 오래된 집계 데이터는 용량 계획, 신뢰성 분석, 장기 성능 비교에 사용할 수 있다.

보안(Security)은 메트릭 엔드포인트와 중앙 모니터링 플랫폼을 모두 보호해야 한다. 메트릭에는 호스트 이름, 네트워크 토폴로지, 소프트웨어 버전, 하드웨어 정보, 운영 상태 또는 비즈니스에 민감한 플릿 활동 정보가 포함될 수 있다. 따라서 배포 요구사항에 따라 인증(Authentication), 전송 암호화, 네트워크 분할(Network Segmentation), 접근 제어, 대시보드 권한을 적용해야 한다. 모니터링 구성요소에는 관찰 대상 자원을 확인하는 데 필요한 최소한의 권한만 부여해야 한다.

CI/CD와 OTA 정보는 릴리스 컨텍스트(Release Context)를 추가하여 모니터링을 강화할 수 있다. 대시보드에는 ROS 2 패키지, 컨테이너 이미지, 설정 리비전 또는 AI 모델이 배포된 시점을 표시할 수 있다. 운영자는 이를 이용하여 변경 전후의 시스템 동작을 비교할 수 있다. 롤아웃 이후 오류율, 지연시간, CPU 사용량 또는 재시작 빈도가 악화되면 모니터링 증거를 기반으로 배포 중단이나 롤백(Rollback)을 판단할 수 있다.

모니터링에서는 운영 관찰 가능성과 기능 안전(Functional Safety)의 경계도 유지해야 한다. 프로메테우스와 그라파나는 비정상 동작을 탐지하고 시각화할 수 있지만 결정론적 안전 제어기(Deterministic Safety Controller), 비상 정지(Emergency Stop), 워치독(Watchdog), 인증된 보호 메커니즘을 대체하지 않는다. 모니터링 네트워크, 프로메테우스 서버 또는 시각화 시스템을 사용할 수 없는 경우에도 로봇은 자체 안전 아키텍처를 통해 적절한 안전 상태(Safe State)로 전환할 수 있어야 한다.

플릿 규모 모니터링(Fleet-Scale Monitoring)은 각 로봇이 로컬 메트릭을 제공하고 사이트 또는 엣지 인프라가 운영 데이터를 집계하며 온프레미스(On-Premise) 또는 클라우드 플랫폼이 장기적인 플릿 가시성을 제공하는 계층적 아키텍처를 사용할 수 있다. 핵심 로봇 동작은 중앙 모니터링 시스템의 가용성과 독립적이어야 한다. 연결이 끊어져도 로봇은 로컬 임무와 안전 기능을 계속 수행하고, 모니터링 데이터는 통신 복구 이후 버퍼링, 집계 또는 동기화할 수 있다.

따라서 성숙한 ROS 2 메트릭 아키텍처는 로봇 애플리케이션, DDS 통신, 라이프사이클 상태, 호스트 자원, 컨테이너, GPU, 배포 메타데이터, 프로메테우스 수집, 경고 관리, 그라파나 시각화를 하나의 체계로 연결한다. 중앙 집중식 로깅(Centralized Logging)과 결합하면 개발자와 운영자가 성능 저하를 탐지하고 추세를 분석하며 릴리스를 비교하고 장애를 진단하여 개별 로봇에서 전체 플릿까지의 운영 동작을 이해할 수 있는 통합 관찰 가능성 기반(Unified Observability Foundation)을 제공한다.

##  

## 08.09 ROS2 Deployment Environment Security Hardening

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

A ROS 2 deployment environment must be hardened as a complete operational platform rather than securing only individual ROS nodes. Production robots combine Linux hosts, ROS 2 processes, DDS communication, containers, device interfaces, remote management, update mechanisms, and network services. Weakness in any layer can expose the entire robot. Security hardening therefore applies defense in depth from the operating system and network boundary through middleware, applications, credentials, and deployment infrastructure.

The first principle is to minimize the attack surface. Robot computers should run only the packages, services, ports, protocols, and management interfaces required for their operational role. Development utilities, unused network daemons, unnecessary compilers, debugging interfaces, and obsolete packages should be removed from production systems. A smaller software footprint reduces potential vulnerabilities while also simplifying patch management, configuration auditing, and long-term maintenance.

Operating-system hardening establishes the foundation beneath ROS 2. Production hosts should use supported operating-system releases, security updates, controlled repositories, restricted administrative access, secure boot mechanisms where appropriate, and well-defined user accounts. File permissions, process ownership, system services, kernel configuration, and remote-login policies should follow least-privilege principles so that compromise of one application does not automatically provide control of the entire robot computer.

ROS 2 applications should avoid unnecessary execution with root privileges. Nodes normally require only the permissions needed to access their files, network interfaces, or hardware devices. Dedicated service accounts and carefully configured groups can provide access to cameras, serial ports, CAN interfaces, LiDAR devices, or GPUs without granting unrestricted administrative authority. Privileged execution should be treated as an exceptional requirement and documented whenever it cannot be eliminated.

DDS forms a major communication surface in ROS 2 because discovery and data exchange can expose nodes, topics, services, and actions across reachable networks. Production deployment should therefore control which network interfaces carry DDS traffic and which systems can participate in the communication domain. Network segmentation, firewall rules, ROS_DOMAIN_ID planning, DDS configuration, and controlled discovery mechanisms can reduce unintended communication between development, test, production, and unrelated robot networks.

SROS2 extends ROS 2 with security mechanisms based on DDS Security. It can provide participant authentication, communication encryption, and access control for ROS graph entities. Security enclaves and policy artifacts can define which nodes are permitted to publish, subscribe, provide services, or perform actions. These controls are most effective when generated from an explicit communication architecture rather than applying broad permissions that effectively recreate an unrestricted ROS graph.

Cryptographic identities and security artifacts require lifecycle management. Certificates, private keys, governance files, permission files, and related credentials should be generated, distributed, stored, rotated, and revoked through controlled procedures. Private keys should not be committed to source repositories or embedded directly in container images. Compromise of a robot credential should be recoverable without requiring replacement of the security identity for every robot in the fleet.

Network architecture should separate robot operational traffic from general enterprise or public networks whenever practical. VLANs, dedicated wireless networks, private 5G segments, VPNs, firewalls, gateways, and site-level security zones can limit communication paths. Remote access should enter through controlled management interfaces rather than exposing ROS 2 or DDS services directly to untrusted networks. The robot should remain protected even when connected to infrastructure outside the development laboratory.

Containerization provides useful isolation but does not automatically create a secure deployment. Production ROS 2 containers should use minimal base images, trusted registries, non-root users, read-only filesystems where practical, restricted Linux capabilities, and explicit device mappings. Privileged containers and unrestricted host mounts should be avoided because they can bypass much of the isolation provided by the container runtime and expose the underlying robot computer.

Container images should be treated as immutable and traceable release artifacts. Images can be scanned for known vulnerabilities, unnecessary packages, exposed credentials, and dependency risks before deployment. Version tags should correspond to validated releases, while immutable digests provide stronger identification of the exact image being executed. Production robots should retrieve images only from authorized registries using authenticated and encrypted communication paths.

Kubernetes or K3s deployments introduce additional security boundaries. Namespaces, service accounts, role-based access control, network policies, admission controls, secrets management, Pod security settings, and node permissions should restrict what each workload can access. Robot-specific workloads may require hardware privileges, but these permissions should be limited to the necessary devices and nodes rather than granting cluster-wide administrative capability to an application container.

Secrets must be separated from ordinary application configuration. Passwords, API tokens, private keys, registry credentials, VPN credentials, and SROS2 security material should not appear in source code, public configuration files, container layers, or diagnostic logs. Deployment systems can inject secrets at runtime through controlled mechanisms. Access should be auditable and limited to the specific service requiring the credential, reducing the consequences of accidental disclosure.

Software supply-chain security begins before deployment. ROS 2 packages, third-party libraries, operating-system dependencies, container images, firmware, and AI frameworks may introduce vulnerabilities or compromised components. CI/CD pipelines should use trusted sources, dependency scanning, artifact verification, software bills of materials, and controlled build environments. A production release should identify not only application source code but also the dependencies and build artifacts from which it was constructed.

OTA and package-update mechanisms represent powerful administrative interfaces and therefore require strong protection. Update manifests, ROS 2 packages, container images, configuration revisions, and system images should be authenticated and integrity-checked before activation. The robot should reject unauthorized or corrupted artifacts. Update infrastructure should also preserve rollback capability so that security patches can be deployed without sacrificing operational recovery when an update causes unexpected behavior.

Remote administration should be minimized and strongly authenticated. SSH, web interfaces, fleet-management agents, diagnostic endpoints, and maintenance APIs should be restricted to approved networks and identities. Shared passwords should be avoided, and administrative actions should be attributable to individual operators or services where practical. Idle services and temporary maintenance ports should not remain enabled simply because they were useful during development or commissioning.

Logging and monitoring are essential for detecting security-relevant behavior. Authentication failures, unexpected service starts, privilege changes, configuration modifications, container restarts, network anomalies, software updates, and ROS 2 security events should be collected with appropriate context. Centralized logging and metrics can reveal patterns that are difficult to identify on one robot, particularly when the same abnormal behavior appears across multiple systems after a deployment or network event.

Security monitoring must itself be protected. Logs and metrics may reveal robot identities, network topology, software versions, operational schedules, site information, or security events. Transport encryption, authenticated collectors, access-controlled dashboards, retention policies, and redaction of sensitive fields should be applied. Diagnostic convenience should not create a secondary information channel that exposes details useful to an attacker.

Physical access must also be considered because deployed robots may operate outside controlled data centers. Accessible USB ports, removable storage, exposed Ethernet connectors, recovery consoles, debug headers, and local boot options can bypass network security assumptions. Production designs can restrict unused interfaces, protect service ports, control boot configuration, and define maintenance procedures that distinguish authorized physical servicing from normal robot operation.

Hardening must preserve the functional and real-time requirements of the robot. Aggressive firewall rules, encryption, container restrictions, security monitoring, or endpoint protection can introduce latency, block DDS discovery, interfere with hardware access, or consume resources required by control workloads. Security changes should therefore be validated through performance, communication, hardware-in-the-loop, and operational testing rather than assuming that a configuration suitable for ordinary IT servers is automatically suitable for robotics.

Security and functional safety remain related but distinct concerns. Authentication and encryption can prevent unauthorized communication, but they do not replace emergency-stop circuits, safety controllers, watchdogs, motion limits, or safe-state mechanisms. Conversely, a functionally safe motion controller does not guarantee cybersecurity. Deployment architecture should ensure that security controls cannot disable essential safety functions and that cybersecurity failures lead to predictable operational responses.

Fleet environments require consistent security baselines. Host configuration, firewall rules, ROS 2 security policies, container versions, certificates, access permissions, and monitoring agents should be managed through reproducible deployment mechanisms rather than manual configuration of individual robots. Configuration drift can otherwise create robots with different exposure levels even when they nominally run the same software release.

Security hardening is a continuous lifecycle rather than a one-time commissioning activity. New vulnerabilities, dependency changes, expired certificates, altered network environments, and newly introduced robot capabilities can invalidate earlier assumptions. Asset inventories, vulnerability management, patching, credential rotation, configuration auditing, incident response, and periodic security testing should therefore continue throughout the operational life of the robot fleet.

A mature ROS 2 deployment environment combines hardened hosts, least-privilege processes, controlled DDS communication, SROS2 policies, container isolation, network segmentation, protected credentials, trusted software supply chains, secure OTA, monitoring, and reproducible fleet configuration. The objective is to reduce exposure while preserving robot performance and safety, creating layered protection from the physical robot computer through ROS 2 middleware to edge, on-premise, and fleet-management infrastructure.

ROS 2 배포 환경(Deployment Environment)은 개별 ROS 노드만 보호하는 것이 아니라 전체 운영 플랫폼을 하나의 시스템으로 보고 보안을 강화해야 한다. 운영 로봇은 Linux 호스트, ROS 2 프로세스, DDS 통신, 컨테이너, 장치 인터페이스, 원격 관리, 업데이트 메커니즘, 네트워크 서비스를 결합한다. 어느 한 계층의 취약점도 전체 로봇을 노출시킬 수 있다. 따라서 보안 강화(Security Hardening)는 운영체제와 네트워크 경계에서 미들웨어, 애플리케이션, 자격 증명, 배포 인프라까지 심층 방어(Defense in Depth)를 적용해야 한다.

첫 번째 원칙은 공격 표면(Attack Surface)을 최소화하는 것이다. 로봇 컴퓨터에서는 운영 역할에 필요한 패키지, 서비스, 포트, 프로토콜, 관리 인터페이스만 실행해야 한다. 개발 도구, 사용하지 않는 네트워크 데몬(Network Daemon), 불필요한 컴파일러, 디버깅 인터페이스, 오래된 패키지는 운영 시스템에서 제거해야 한다. 소프트웨어 구성요소를 줄이면 잠재적인 취약점을 감소시키는 동시에 패치 관리, 설정 감사(Configuration Auditing), 장기 유지보수를 단순화할 수 있다.

운영체제 보안 강화(Operating-System Hardening)는 ROS 2 아래의 기반을 형성한다. 운영 호스트는 지원되는 운영체제 릴리스, 보안 업데이트, 통제된 저장소, 제한된 관리자 접근, 필요한 경우 보안 부팅(Secure Boot), 명확하게 정의된 사용자 계정을 사용해야 한다. 파일 권한, 프로세스 소유권, 시스템 서비스, 커널 설정, 원격 로그인 정책은 최소 권한 원칙(Least-Privilege Principle)을 따라야 하며, 하나의 애플리케이션이 침해되더라도 전체 로봇 컴퓨터의 제어권을 자동으로 획득하지 못하도록 해야 한다.

ROS 2 애플리케이션은 불필요하게 루트 권한(Root Privilege)으로 실행하지 않아야 한다. 일반적으로 노드는 필요한 파일, 네트워크 인터페이스 또는 하드웨어 장치에 접근하는 데 필요한 권한만 가져야 한다. 전용 서비스 계정(Service Account)과 세밀하게 설정된 그룹을 사용하면 무제한 관리자 권한을 부여하지 않고도 카메라, 직렬 포트, CAN 인터페이스, LiDAR 장치, GPU에 접근할 수 있다. 권한 상승 실행(Privileged Execution)은 예외적인 요구사항으로 취급하고 제거할 수 없는 경우 그 이유를 문서화해야 한다.

DDS는 디스커버리(Discovery)와 데이터 교환 과정에서 접근 가능한 네트워크에 노드, 토픽, 서비스, 액션을 노출할 수 있으므로 ROS 2의 주요 통신 공격 표면을 형성한다. 따라서 운영 배포에서는 어떤 네트워크 인터페이스가 DDS 트래픽을 전달하고 어떤 시스템이 통신 도메인에 참여할 수 있는지를 통제해야 한다. 네트워크 분할(Network Segmentation), 방화벽 규칙, ROS_DOMAIN_ID 계획, DDS 설정, 통제된 디스커버리 메커니즘을 이용하면 개발, 시험, 운영 및 관계없는 로봇 네트워크 사이에서 의도하지 않은 통신을 줄일 수 있다.

SROS2는 DDS 보안(DDS Security)을 기반으로 하는 보안 메커니즘을 ROS 2에 확장한다. 이를 통해 참여자 인증(Participant Authentication), 통신 암호화, ROS 그래프(Graph) 엔터티에 대한 접근 제어를 제공할 수 있다. 보안 엔클레이브(Security Enclave)와 정책 산출물(Policy Artifact)은 어떤 노드가 발행(Publish), 구독(Subscribe), 서비스 제공 또는 액션 수행을 허용받는지 정의할 수 있다. 이러한 제어는 사실상 제한 없는 ROS 그래프를 다시 만드는 광범위한 권한을 적용하는 대신 명시적인 통신 아키텍처를 기반으로 생성할 때 가장 효과적이다.

암호학적 신원(Cryptographic Identity)과 보안 산출물에는 수명주기 관리(Lifecycle Management)가 필요하다. 인증서, 개인 키(Private Key), 거버넌스 파일(Governance File), 권한 파일(Permission File), 관련 자격 증명은 통제된 절차를 통해 생성, 배포, 저장, 교체, 폐기해야 한다. 개인 키를 소스 저장소에 커밋하거나 컨테이너 이미지에 직접 포함해서는 안 된다. 하나의 로봇 자격 증명이 침해되더라도 전체 플릿의 모든 로봇 보안 신원을 교체하지 않고 해당 자격 증명을 복구하거나 폐기할 수 있어야 한다.

네트워크 아키텍처(Network Architecture)는 가능한 경우 로봇 운영 트래픽을 일반 기업 네트워크나 공용 네트워크에서 분리해야 한다. VLAN, 전용 무선 네트워크, 사설 5G 세그먼트, VPN, 방화벽, 게이트웨이, 사이트 수준 보안 영역(Security Zone)을 이용하여 통신 경로를 제한할 수 있다. 원격 접근은 ROS 2 또는 DDS 서비스를 신뢰할 수 없는 네트워크에 직접 노출하는 대신 통제된 관리 인터페이스를 통해 이루어져야 한다. 로봇은 개발 실험실 외부의 인프라에 연결된 경우에도 보호될 수 있어야 한다.

컨테이너화(Containerization)는 유용한 격리 기능을 제공하지만 그 자체만으로 안전한 배포 환경을 만드는 것은 아니다. 운영 ROS 2 컨테이너는 최소화된 베이스 이미지(Base Image), 신뢰할 수 있는 레지스트리, 비루트 사용자(Non-Root User), 가능한 경우 읽기 전용 파일 시스템(Read-Only Filesystem), 제한된 Linux 기능(Capability), 명시적인 장치 매핑(Device Mapping)을 사용해야 한다. 권한 컨테이너(Privileged Container)와 제한 없는 호스트 마운트(Host Mount)는 컨테이너 런타임이 제공하는 격리 기능 대부분을 우회하고 기반 로봇 컴퓨터를 노출할 수 있으므로 피해야 한다.

컨테이너 이미지는 변경 불가능하고 추적 가능한 릴리스 산출물(Immutable and Traceable Release Artifact)로 취급해야 한다. 배포 전에 알려진 취약점, 불필요한 패키지, 노출된 자격 증명, 의존성 위험을 검사할 수 있다. 버전 태그는 검증된 릴리스에 대응해야 하며 변경 불가능한 다이제스트(Immutable Digest)는 실제 실행 중인 정확한 이미지를 더욱 강력하게 식별한다. 운영 로봇은 인증되고 암호화된 통신 경로를 이용하여 승인된 레지스트리에서만 이미지를 가져와야 한다.

쿠버네티스(Kubernetes) 또는 K3s 배포에서는 추가적인 보안 경계가 도입된다. 네임스페이스(Namespace), 서비스 계정, 역할 기반 접근 제어(Role-Based Access Control, RBAC), 네트워크 정책(Network Policy), 승인 제어(Admission Control), 시크릿 관리(Secrets Management), 파드 보안 설정(Pod Security Setting), 노드 권한을 이용하여 각 워크로드가 접근할 수 있는 범위를 제한해야 한다. 로봇별 워크로드에 하드웨어 권한이 필요할 수 있지만 애플리케이션 컨테이너에 클러스터 전체 관리자 권한을 부여하는 대신 필요한 장치와 노드에만 해당 권한을 제한해야 한다.

시크릿(Secret)은 일반적인 애플리케이션 설정과 분리해야 한다. 비밀번호, API 토큰, 개인 키, 레지스트리 자격 증명, VPN 자격 증명, SROS2 보안 자료를 소스 코드, 공개 설정 파일, 컨테이너 레이어 또는 진단 로그에 포함해서는 안 된다. 배포 시스템은 통제된 메커니즘을 통해 런타임(Runtime)에 시크릿을 주입할 수 있다. 접근 과정은 감사 가능해야 하며 해당 자격 증명이 필요한 특정 서비스로만 제한하여 우발적인 노출로 인한 영향을 줄여야 한다.

소프트웨어 공급망 보안(Software Supply-Chain Security)은 배포 이전부터 시작된다. ROS 2 패키지, 서드파티 라이브러리, 운영체제 의존성, 컨테이너 이미지, 펌웨어, AI 프레임워크는 취약하거나 침해된 구성요소를 포함할 가능성이 있다. CI/CD 파이프라인은 신뢰할 수 있는 소스, 의존성 검사(Dependency Scanning), 산출물 검증(Artifact Verification), 소프트웨어 자재 명세서(Software Bill of Materials, SBOM), 통제된 빌드 환경을 사용해야 한다. 운영 릴리스에서는 애플리케이션 소스 코드뿐 아니라 이를 구성하는 의존성과 빌드 산출물까지 식별할 수 있어야 한다.

무선 업데이트(Over-the-Air Update, OTA)와 패키지 업데이트 메커니즘은 강력한 관리자 인터페이스이므로 높은 수준의 보호가 필요하다. 업데이트 매니페스트, ROS 2 패키지, 컨테이너 이미지, 설정 리비전(Configuration Revision), 시스템 이미지는 활성화 전에 인증과 무결성 검증을 거쳐야 한다. 로봇은 승인되지 않았거나 손상된 산출물을 거부해야 한다. 또한 보안 패치를 배포하면서 업데이트로 예상하지 못한 동작이 발생하는 경우 운영 상태를 복구할 수 있도록 업데이트 인프라는 롤백(Rollback) 기능을 유지해야 한다.

원격 관리(Remote Administration)는 최소화하고 강력한 인증을 적용해야 한다. SSH, 웹 인터페이스, 플릿 관리 에이전트(Fleet-Management Agent), 진단 엔드포인트, 유지보수 API는 승인된 네트워크와 신원으로 접근을 제한해야 한다. 공유 비밀번호는 피해야 하며 가능한 경우 관리 작업을 개별 운영자 또는 서비스와 연결하여 추적할 수 있어야 한다. 개발이나 초기 시운전(Commissioning) 과정에서 유용했다는 이유만으로 사용하지 않는 서비스와 임시 유지보수 포트를 계속 활성화해서는 안 된다.

로깅(Logging)과 모니터링(Monitoring)은 보안 관련 동작을 탐지하는 데 필수적이다. 인증 실패, 예상하지 않은 서비스 시작, 권한 변경, 설정 변경, 컨테이너 재시작, 네트워크 이상, 소프트웨어 업데이트, ROS 2 보안 이벤트를 적절한 컨텍스트와 함께 수집해야 한다. 중앙 집중식 로깅과 메트릭을 이용하면 하나의 로봇에서는 식별하기 어려운 패턴을 발견할 수 있으며, 특히 배포나 네트워크 이벤트 이후 여러 시스템에서 동일한 비정상 동작이 발생할 때 효과적이다.

보안 모니터링(Security Monitoring) 자체도 보호되어야 한다. 로그와 메트릭에는 로봇 식별 정보, 네트워크 토폴로지, 소프트웨어 버전, 운영 일정, 사이트 정보 또는 보안 이벤트가 포함될 수 있다. 전송 암호화, 인증된 수집기(Authenticated Collector), 접근이 통제된 대시보드, 보존 정책(Retention Policy), 민감한 필드의 마스킹 또는 제거를 적용해야 한다. 진단 편의성을 위해 공격자에게 유용한 정보를 노출하는 부가적인 정보 채널을 만들어서는 안 된다.

배포된 로봇은 통제된 데이터센터 외부에서 작동할 수 있으므로 물리적 접근(Physical Access)도 고려해야 한다. 접근 가능한 USB 포트, 이동식 저장장치, 노출된 Ethernet 커넥터, 복구 콘솔, 디버그 헤더(Debug Header), 로컬 부팅 옵션은 네트워크 보안 가정을 우회할 수 있다. 운영 설계에서는 사용하지 않는 인터페이스를 제한하고 서비스 포트를 보호하며 부팅 설정을 통제하고, 승인된 물리적 유지보수와 정상 로봇 운영을 구분하는 유지보수 절차를 정의할 수 있다.

보안 강화는 로봇의 기능 및 실시간 요구사항(Real-Time Requirement)을 유지해야 한다. 지나치게 강력한 방화벽 규칙, 암호화, 컨테이너 제한, 보안 모니터링 또는 엔드포인트 보호는 지연시간을 증가시키거나 DDS 디스커버리를 차단하거나 하드웨어 접근을 방해하거나 제어 워크로드에 필요한 자원을 소비할 수 있다. 따라서 일반적인 IT 서버에 적합한 설정이 로보틱스에도 자동으로 적합하다고 가정하지 말고 성능, 통신, 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL), 운영 테스트를 통해 보안 변경사항을 검증해야 한다.

사이버보안(Cybersecurity)과 기능 안전(Functional Safety)은 서로 관련되어 있지만 구분되는 영역이다. 인증과 암호화는 승인되지 않은 통신을 방지할 수 있지만 비상 정지(Emergency Stop), 안전 제어기(Safety Controller), 워치독(Watchdog), 모션 제한(Motion Limit), 안전 상태(Safe-State) 메커니즘을 대체하지 않는다. 반대로 기능적으로 안전한 모션 제어기가 사이버보안을 보장하는 것도 아니다. 배포 아키텍처는 보안 제어가 필수 안전 기능을 비활성화하지 않도록 하고 사이버보안 장애가 발생해도 예측 가능한 운영 대응이 이루어지도록 해야 한다.

플릿 환경(Fleet Environment)에서는 일관된 보안 기준선(Security Baseline)이 필요하다. 호스트 설정, 방화벽 규칙, ROS 2 보안 정책, 컨테이너 버전, 인증서, 접근 권한, 모니터링 에이전트는 개별 로봇을 수동으로 설정하는 대신 재현 가능한 배포 메커니즘을 통해 관리해야 한다. 그렇지 않으면 명목상 동일한 소프트웨어 릴리스를 실행하더라도 설정 드리프트(Configuration Drift)로 인해 로봇마다 서로 다른 보안 노출 수준이 발생할 수 있다.

보안 강화는 일회성 시운전 작업이 아니라 지속적인 수명주기(Continuous Lifecycle)이다. 새로운 취약점, 의존성 변경, 만료된 인증서, 변화된 네트워크 환경, 새롭게 추가된 로봇 기능으로 인해 기존의 보안 가정이 더 이상 유효하지 않을 수 있다. 따라서 자산 목록(Asset Inventory), 취약점 관리, 패치 적용, 자격 증명 교체, 설정 감사, 사고 대응(Incident Response), 정기적인 보안 테스트를 로봇 플릿의 전체 운영 수명 동안 지속해야 한다.

성숙한 ROS 2 배포 환경은 보안 강화된 호스트(Hardened Host), 최소 권한 프로세스, 통제된 DDS 통신, SROS2 정책, 컨테이너 격리, 네트워크 분할, 보호된 자격 증명, 신뢰할 수 있는 소프트웨어 공급망, 안전한 OTA, 모니터링, 재현 가능한 플릿 설정을 하나의 체계로 결합한다. 목표는 로봇의 성능과 안전을 유지하면서 보안 노출을 줄이고, 물리 로봇 컴퓨터에서 ROS 2 미들웨어를 거쳐 엣지(Edge), 온프레미스(On-Premise), 플릿 관리 인프라까지 이어지는 계층화된 보호(Layered Protection)를 구축하는 것이다.

##  

## 08.10 Large-Scale Fleet ROS2 Deployment Operations

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

Large-scale ROS 2 fleet deployment transforms software delivery from the management of individual robots into the coordinated operation of hundreds or thousands of heterogeneous physical systems. Each robot combines computing hardware, sensors, actuators, ROS 2 nodes, middleware, configuration, and network connectivity. Fleet operations must therefore manage software consistency, robot identity, deployment state, observability, security, and recovery while preserving local autonomy and physical safety.

A scalable fleet architecture separates robot-local execution from centralized management. Navigation, perception, motion control, hardware interfaces, and essential safety-related functions remain on the robot or nearby edge computer. Fleet services provide configuration, mission coordination, software distribution, monitoring, analytics, and operational governance. This separation ensures that temporary loss of connectivity to central infrastructure does not immediately prevent a robot from executing safe local behavior.

Every robot requires a persistent and unique identity that connects the physical machine with its digital configuration. Fleet metadata can include robot_id, model, hardware revision, processor architecture, sensor configuration, site, operational role, ROS 2 distribution, and software baseline. This information enables deployment systems to determine which packages, container images, parameters, AI models, and security credentials are compatible with each physical robot.

Standardized software baselines reduce configuration drift across the fleet. A baseline can define ROS 2 package versions, container digests, operating-system revision, DDS profiles, configuration schemas, AI models, and deployment manifests. Robots may still require individual calibration or site-specific parameters, but these differences should be represented explicitly. Operators must be able to distinguish intentional configuration variation from accidental divergence caused by manual maintenance.

ROS 2 namespaces, ROS_DOMAIN_ID values, DDS configuration, and network segmentation help organize communication across multiple robots. A fleet should not assume that every ROS participant must discover every other participant. Local robot communication, site-level coordination, and centralized fleet services can use controlled communication boundaries. This reduces unnecessary DDS discovery traffic, limits fault propagation, and makes robot-to-robot or robot-to-infrastructure communication easier to secure and diagnose.

Deployment infrastructure may combine ROS 2 packages, Docker containers, Docker Compose, Kubernetes, or K3s according to system scale and operational requirements. Individual robots can run a stable local container stack while edge or on-premise clusters host shared perception, AI inference, mapping, data processing, or fleet services. The deployment architecture should assign workloads according to hardware attachment, latency requirements, accelerator availability, reliability, and connectivity.

Fleet software releases should be immutable and traceable. CI/CD pipelines build and test ROS 2 packages or container images, record source revisions and dependencies, perform security checks, and publish validated artifacts to controlled repositories or registries. Deployment systems should install exactly these artifacts rather than rebuilding software on robots. This maintains a direct relationship between the software that passed validation and the software executing in the field.

Large-scale rollout should use deployment groups rather than updating every robot simultaneously. Robots can be organized by laboratory, pilot group, site, hardware variant, customer environment, or operational role. A release may progress from simulation and hardware-in-the-loop testing to a small canary group before expanding through successive fleet waves. Health evidence from each stage provides a decision point for continuing, pausing, or reversing deployment.

Robot operational state must influence deployment scheduling. Updating a stationary robot in a maintenance area is fundamentally different from modifying software while the robot is carrying material, inspecting infrastructure, or navigating outdoors. Fleet systems should consider mission state, charging condition, battery level, network quality, maintenance windows, and safety state before activation. Deployment orchestration must therefore integrate with robot operations rather than behaving like ordinary server patch management.

OTA mechanisms provide the execution layer for controlled fleet updates. Packages, containers, configuration revisions, AI models, and system images can be downloaded in advance and activated when operational conditions are satisfied. Large artifacts may be cached on site-level edge servers to reduce repeated wide-area transfers. Resumable downloads and local buffering are important because mobile robots frequently operate with intermittent or variable-quality network connectivity.

Rollback capability is essential when deployment affects physical systems. Each robot should retain sufficient information to restore the previous validated package set, container digest, configuration revision, model version, or operating-system image. Rollback criteria may include failed startup, missing ROS 2 nodes, abnormal diagnostics, communication loss, excessive resource usage, or inability to reach the expected operational state after an update.

Fleet health cannot be represented by simple online or offline status. A robot may be connected to the network while navigation is unavailable, a sensor has failed, a lifecycle node remains inactive, or DDS communication is degraded. Operational health should combine infrastructure metrics, ROS 2 lifecycle states, diagnostics, required topic availability, sensor status, mission readiness, battery condition, and application-specific indicators into a meaningful representation of functional readiness.

Centralized observability becomes increasingly important as fleet size grows. Prometheus and Grafana can provide numerical monitoring of computing resources, communication behavior, robot health, and historical trends, while centralized logging systems such as ELK preserve detailed events. Shared robot identifiers, software versions, deployment IDs, and synchronized timestamps allow operators to correlate metrics, logs, updates, and physical behavior across the fleet.

Fleet dashboards should support progressive investigation from system-wide status to individual robot detail. Operators may begin with robot availability, mission state, battery condition, software version, active alerts, and site status. They can then drill down into a particular robot to examine ROS 2 nodes, containers, CPU and GPU utilization, DDS communication, sensors, lifecycle states, recent updates, and logs without manually connecting to the robot computer.

Alert management must account for correlated failures. If a site network fails, hundreds of robots may simultaneously report communication loss. If a faulty software release is deployed, the same node error may appear across an entire rollout group. Fleet monitoring should group and correlate related alerts by site, release, subsystem, or infrastructure dependency so that operators identify common causes instead of processing every robot notification as an independent incident.

Capacity planning includes both robot resources and shared infrastructure. Container registries, package repositories, monitoring systems, log storage, wireless networks, edge servers, databases, and fleet-management services must scale with robot count and telemetry volume. Deployment waves and local caching can prevent software distribution from saturating site networks, while retention and aggregation policies prevent operational data from growing without control.

Security policies must remain consistent across the fleet. Robot certificates, SROS2 identities, VPN credentials, container-registry access, service accounts, firewall rules, and administrative permissions require centralized lifecycle management. Credentials should support rotation and revocation at the level of an individual robot or group. A compromised robot should be isolated without forcing unnecessary credential replacement or operational interruption across the entire fleet.

Configuration management should treat fleet state as declarative whenever practical. Desired software versions, deployment manifests, security policies, robot groups, and environment-specific settings can be maintained in version-controlled repositories. GitOps-style workflows can compare desired state with observed deployment state and provide an auditable history of changes. Manual modification on individual robots should be minimized because it creates undocumented configuration drift.

Heterogeneous fleets require compatibility-aware orchestration. Different robots may use x86 or ARM processors, NVIDIA or other accelerators, different sensor suites, and multiple hardware revisions. Deployment metadata must therefore describe compatibility constraints explicitly. A common application can have several validated artifacts or profiles, while fleet management selects the correct variant according to the physical capabilities and software baseline of each robot.

Shared edge and on-premise computing can extend robot-local intelligence. Computationally expensive perception, map processing, model adaptation, data aggregation, or multi-robot optimization may execute on site infrastructure when latency and connectivity permit. Workload placement should preserve clear boundaries between functions that can migrate to shared infrastructure and functions that must remain local because of real-time, safety, or connectivity requirements.

Fleet operations must be designed for disconnected operation. Loss of Wi-Fi, private 5G, WAN connectivity, an edge server, or the central management platform should not automatically stop essential robot functions. Robots should maintain local navigation, perception, control, and safety capabilities for an appropriate operational period. Telemetry, logs, metrics, and deployment status can be buffered locally and synchronized after connectivity is restored.

Incident response becomes a fleet-level discipline rather than an individual debugging activity. Operators need to identify affected robots, software releases, hardware variants, sites, and time ranges quickly. A problematic deployment may be paused globally, isolated to a group, or rolled back selectively. Complete traceability from robot state to source revision, package version, container digest, configuration, and deployment event substantially reduces the time required to investigate field failures.

Operational procedures should define roles and authority for deployment, rollback, emergency intervention, security response, and maintenance. Automation can execute routine actions consistently, but physical robots still require clear governance because software changes can influence motion and interaction with the environment. High-impact actions should be auditable, attributable, and constrained by appropriate authorization and robot safety state.

Large-scale ROS 2 operations ultimately require a hierarchical management architecture connecting robot-local autonomy, site edge infrastructure, on-premise services, and centralized fleet governance. ROS 2 provides distributed robotic communication, while containers, OTA, CI/CD, monitoring, logging, security, and orchestration provide the operational framework around it. Together these mechanisms enable reproducible software delivery and coordinated lifecycle management without making safe robot operation dependent on continuous central connectivity.

A mature fleet deployment system therefore treats every robot as both an autonomous physical machine and a managed software platform. Identity, configuration, software baselines, staged updates, rollback, observability, security, compatibility, and operational state are managed as connected parts of one lifecycle. The objective is not merely to deploy ROS 2 software at scale, but to maintain a reliable, secure, diagnosable, and continuously evolvable fleet throughout years of real-world operation.

대규모 ROS 2 플릿 배포(Large-Scale ROS 2 Fleet Deployment)는 소프트웨어 배포를 개별 로봇 관리에서 수백 또는 수천 대의 이기종 물리 시스템을 조정하여 운영하는 수준으로 확장한다. 각 로봇은 컴퓨팅 하드웨어, 센서, 액추에이터, ROS 2 노드, 미들웨어, 설정, 네트워크 연결을 결합한다. 따라서 플릿 운영(Fleet Operations)은 로컬 자율성과 물리적 안전을 유지하면서 소프트웨어 일관성, 로봇 식별, 배포 상태, 관찰 가능성(Observability), 보안, 복구를 관리해야 한다.

확장 가능한 플릿 아키텍처(Scalable Fleet Architecture)는 로봇 로컬 실행(Robot-Local Execution)과 중앙 집중식 관리(Centralized Management)를 분리한다. 내비게이션, 인지, 모션 제어, 하드웨어 인터페이스, 필수 안전 관련 기능은 로봇 또는 인접한 엣지 컴퓨터(Edge Computer)에 유지된다. 플릿 서비스는 설정, 임무 조정, 소프트웨어 배포, 모니터링, 분석, 운영 거버넌스(Operational Governance)를 제공한다. 이러한 분리를 통해 중앙 인프라와 일시적으로 연결이 끊어져도 로봇은 안전한 로컬 동작을 계속 수행할 수 있다.

모든 로봇에는 물리적 장비와 디지털 설정을 연결하는 지속적이고 고유한 식별자(Persistent Unique Identity)가 필요하다. 플릿 메타데이터(Fleet Metadata)에는 robot_id, 모델, 하드웨어 리비전, 프로세서 아키텍처, 센서 구성, 사이트, 운영 역할, ROS 2 배포판(Distribution), 소프트웨어 기준선(Software Baseline)이 포함될 수 있다. 이를 통해 배포 시스템은 각 물리 로봇과 호환되는 패키지, 컨테이너 이미지, 파라미터, AI 모델, 보안 자격 증명을 결정할 수 있다.

표준화된 소프트웨어 기준선(Standardized Software Baseline)은 플릿 전체의 설정 드리프트(Configuration Drift)를 줄인다. 기준선에는 ROS 2 패키지 버전, 컨테이너 다이제스트(Container Digest), 운영체제 리비전, DDS 프로파일, 설정 스키마(Configuration Schema), AI 모델, 배포 매니페스트(Deployment Manifest)를 정의할 수 있다. 개별 로봇에 고유한 보정 또는 사이트별 파라미터가 필요할 수 있지만 이러한 차이는 명시적으로 표현해야 한다. 운영자는 의도된 설정 차이와 수동 유지보수로 발생한 우발적 차이를 구분할 수 있어야 한다.

ROS 2 네임스페이스(Namespace), ROS_DOMAIN_ID 값, DDS 설정, 네트워크 분할(Network Segmentation)은 다중 로봇 환경에서 통신을 체계적으로 구성하는 데 도움을 준다. 플릿은 모든 ROS 참여자가 서로를 반드시 검색해야 한다고 가정해서는 안 된다. 로봇 로컬 통신, 사이트 수준 조정, 중앙 플릿 서비스는 통제된 통신 경계(Communication Boundary)를 사용할 수 있다. 이를 통해 불필요한 DDS 디스커버리 트래픽을 줄이고 장애 전파를 제한하며 로봇 간 또는 로봇과 인프라 간 통신의 보안과 진단을 용이하게 할 수 있다.

배포 인프라(Deployment Infrastructure)는 시스템 규모와 운영 요구사항에 따라 ROS 2 패키지, 도커(Docker) 컨테이너, 도커 컴포즈(Docker Compose), 쿠버네티스(Kubernetes), K3s를 조합할 수 있다. 개별 로봇은 안정적인 로컬 컨테이너 스택을 실행하고 엣지 또는 온프레미스(On-Premise) 클러스터는 공유 인지, AI 추론, 매핑, 데이터 처리, 플릿 서비스를 실행할 수 있다. 배포 아키텍처는 하드웨어 연결성, 지연시간 요구사항, 가속기 가용성, 신뢰성, 네트워크 연결성을 고려하여 워크로드를 배치해야 한다.

플릿 소프트웨어 릴리스(Fleet Software Release)는 변경 불가능하고 추적 가능해야 한다. CI/CD 파이프라인은 ROS 2 패키지 또는 컨테이너 이미지를 빌드하고 테스트하며 소스 리비전과 의존성을 기록하고 보안 검사를 수행한 후 검증된 산출물을 통제된 저장소 또는 레지스트리에 게시한다. 배포 시스템은 로봇에서 소프트웨어를 다시 빌드하는 대신 정확히 이러한 산출물을 설치해야 한다. 이를 통해 검증을 통과한 소프트웨어와 실제 현장에서 실행되는 소프트웨어 사이의 직접적인 관계를 유지할 수 있다.

대규모 롤아웃(Rollout)은 모든 로봇을 동시에 업데이트하는 대신 배포 그룹(Deployment Group)을 사용해야 한다. 로봇은 연구실, 파일럿 그룹, 사이트, 하드웨어 변형, 고객 환경 또는 운영 역할에 따라 구성할 수 있다. 릴리스는 시뮬레이션과 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 시험을 거쳐 소규모 카나리 그룹(Canary Group)에 먼저 적용한 후 연속적인 플릿 배포 단계로 확대할 수 있다. 각 단계에서 확보한 상태 정보를 바탕으로 배포의 계속, 일시 중지 또는 되돌리기를 결정할 수 있다.

로봇의 운영 상태(Operational State)는 배포 스케줄링(Deployment Scheduling)에 반영되어야 한다. 유지보수 구역에 정지해 있는 로봇을 업데이트하는 것과 자재를 운반하거나 시설을 검사하거나 실외에서 주행 중인 로봇의 소프트웨어를 변경하는 것은 근본적으로 다르다. 플릿 시스템은 활성화 전에 임무 상태, 충전 상태, 배터리 수준, 네트워크 품질, 유지보수 시간대(Maintenance Window), 안전 상태를 고려해야 한다. 따라서 배포 오케스트레이션(Deployment Orchestration)은 일반적인 서버 패치 관리처럼 동작하는 것이 아니라 로봇 운영과 통합되어야 한다.

무선 업데이트(Over-the-Air Update, OTA) 메커니즘은 통제된 플릿 업데이트를 실행하는 계층을 제공한다. 패키지, 컨테이너, 설정 리비전, AI 모델, 시스템 이미지를 미리 다운로드한 후 운영 조건이 충족되면 활성화할 수 있다. 대용량 산출물은 사이트 수준 엣지 서버에 캐시(Cache)하여 반복적인 광역 네트워크 전송을 줄일 수 있다. 모바일 로봇은 간헐적이거나 품질이 변하는 네트워크 환경에서 동작하는 경우가 많으므로 이어받기 가능한 다운로드(Resumable Download)와 로컬 버퍼링(Local Buffering)이 중요하다.

배포가 물리 시스템에 영향을 미치므로 롤백 기능(Rollback Capability)은 필수적이다. 각 로봇은 이전에 검증된 패키지 세트, 컨테이너 다이제스트, 설정 리비전, 모델 버전 또는 운영체제 이미지로 복원하는 데 필요한 충분한 정보를 유지해야 한다. 롤백 기준에는 시작 실패, 필수 ROS 2 노드 누락, 비정상 진단 상태, 통신 손실, 과도한 자원 사용 또는 업데이트 이후 예상된 운영 상태에 도달하지 못하는 상황 등이 포함될 수 있다.

플릿 상태(Fleet Health)는 단순한 온라인 또는 오프라인 상태만으로 표현할 수 없다. 로봇이 네트워크에는 연결되어 있더라도 내비게이션을 사용할 수 없거나 센서가 고장났거나 라이프사이클 노드가 비활성 상태에 머물거나 DDS 통신이 저하될 수 있다. 운영 상태는 인프라 메트릭, ROS 2 라이프사이클 상태, 진단 정보, 필수 토픽 가용성, 센서 상태, 임무 준비 상태, 배터리 상태, 애플리케이션별 지표를 결합하여 실질적인 기능 준비 상태(Functional Readiness)를 표현해야 한다.

플릿 규모가 증가할수록 중앙 집중식 관찰 가능성(Centralized Observability)의 중요성도 증가한다. 프로메테우스(Prometheus)와 그라파나(Grafana)는 컴퓨팅 자원, 통신 동작, 로봇 상태, 장기 추세에 대한 수치 기반 모니터링을 제공할 수 있으며 ELK와 같은 중앙 집중식 로깅(Centralized Logging) 시스템은 상세 이벤트를 보존할 수 있다. 공통 로봇 식별자, 소프트웨어 버전, 배포 ID, 동기화된 타임스탬프를 사용하면 플릿 전체에서 메트릭, 로그, 업데이트, 물리적 동작을 상호 연계할 수 있다.

플릿 대시보드(Fleet Dashboard)는 전체 시스템 상태에서 개별 로봇의 상세 정보까지 단계적으로 조사할 수 있도록 구성해야 한다. 운영자는 로봇 가용성, 임무 상태, 배터리 상태, 소프트웨어 버전, 활성 경고, 사이트 상태부터 확인할 수 있다. 이후 특정 로봇으로 드릴다운(Drill Down)하여 로봇 컴퓨터에 직접 접속하지 않고도 ROS 2 노드, 컨테이너, CPU와 GPU 사용률, DDS 통신, 센서, 라이프사이클 상태, 최근 업데이트, 로그를 조사할 수 있다.

경고 관리(Alert Management)는 상호 연관된 장애를 고려해야 한다. 사이트 네트워크에 장애가 발생하면 수백 대의 로봇이 동시에 통신 손실을 보고할 수 있다. 결함이 있는 소프트웨어 릴리스가 배포되면 동일한 노드 오류가 전체 롤아웃 그룹에서 발생할 수 있다. 플릿 모니터링은 사이트, 릴리스, 서브시스템 또는 인프라 의존성을 기준으로 관련 경고를 그룹화하고 상관 분석하여 운영자가 각 로봇 알림을 독립적인 장애로 처리하는 대신 공통 원인을 식별하도록 해야 한다.

용량 계획(Capacity Planning)은 로봇 자원뿐 아니라 공유 인프라도 포함한다. 컨테이너 레지스트리, 패키지 저장소, 모니터링 시스템, 로그 저장소, 무선 네트워크, 엣지 서버, 데이터베이스, 플릿 관리 서비스는 로봇 수와 텔레메트리(Telemetry) 양의 증가에 맞추어 확장되어야 한다. 단계적 배포와 로컬 캐싱(Local Caching)은 소프트웨어 배포가 사이트 네트워크를 포화시키는 것을 방지하며 보존 및 집계 정책은 운영 데이터가 통제 없이 증가하는 것을 방지한다.

보안 정책(Security Policy)은 플릿 전체에서 일관성을 유지해야 한다. 로봇 인증서, SROS2 신원, VPN 자격 증명, 컨테이너 레지스트리 접근 권한, 서비스 계정, 방화벽 규칙, 관리자 권한에는 중앙 집중식 수명주기 관리(Centralized Lifecycle Management)가 필요하다. 자격 증명은 개별 로봇 또는 그룹 단위로 교체와 폐기가 가능해야 한다. 하나의 로봇이 침해되더라도 전체 플릿에서 불필요한 자격 증명 교체나 운영 중단을 발생시키지 않고 해당 로봇을 격리할 수 있어야 한다.

설정 관리(Configuration Management)는 가능한 경우 플릿 상태를 선언적 방식(Declarative Approach)으로 다루어야 한다. 원하는 소프트웨어 버전, 배포 매니페스트, 보안 정책, 로봇 그룹, 환경별 설정을 버전 관리 저장소에서 유지할 수 있다. GitOps 방식의 워크플로(GitOps-Style Workflow)는 원하는 상태(Desired State)와 관측된 배포 상태(Observed Deployment State)를 비교하고 변경 이력을 감사 가능하게 제공할 수 있다. 개별 로봇에 대한 수동 변경은 문서화되지 않은 설정 드리프트를 발생시키므로 최소화해야 한다.

이기종 플릿(Heterogeneous Fleet)에는 호환성을 고려한 오케스트레이션(Compatibility-Aware Orchestration)이 필요하다. 서로 다른 로봇은 x86 또는 ARM 프로세서, NVIDIA 또는 다른 가속기, 서로 다른 센서 구성과 다양한 하드웨어 리비전을 사용할 수 있다. 따라서 배포 메타데이터는 호환성 제약조건(Compatibility Constraint)을 명시적으로 표현해야 한다. 하나의 공통 애플리케이션에 여러 개의 검증된 산출물 또는 프로파일을 제공하고 플릿 관리 시스템이 각 로봇의 물리적 기능과 소프트웨어 기준선에 따라 올바른 변형을 선택할 수 있다.

공유 엣지(Edge) 및 온프레미스(On-Premise) 컴퓨팅은 로봇 로컬 지능을 확장할 수 있다. 연산량이 많은 인지, 지도 처리, 모델 적응(Model Adaptation), 데이터 집계, 다중 로봇 최적화는 지연시간과 연결성이 허용되는 경우 사이트 인프라에서 실행할 수 있다. 워크로드 배치(Workload Placement)는 공유 인프라로 이동할 수 있는 기능과 실시간성, 안전 또는 연결성 요구사항으로 인해 반드시 로컬에 유지해야 하는 기능 사이에 명확한 경계를 유지해야 한다.

플릿 운영은 연결 단절 상태(Disconnected Operation)를 고려하여 설계해야 한다. Wi-Fi, 사설 5G, WAN 연결, 엣지 서버 또는 중앙 관리 플랫폼이 중단되더라도 필수 로봇 기능이 자동으로 정지해서는 안 된다. 로봇은 적절한 운영 기간 동안 로컬 내비게이션, 인지, 제어, 안전 기능을 유지해야 한다. 텔레메트리, 로그, 메트릭, 배포 상태는 로컬에 버퍼링하고 연결이 복구된 이후 중앙 시스템과 동기화할 수 있다.

사고 대응(Incident Response)은 개별 로봇 디버깅이 아니라 플릿 수준의 운영 체계가 된다. 운영자는 영향을 받은 로봇, 소프트웨어 릴리스, 하드웨어 변형, 사이트, 시간 범위를 신속하게 식별할 수 있어야 한다. 문제가 있는 배포는 전체적으로 중단하거나 특정 그룹에 격리하거나 선택적으로 롤백할 수 있다. 로봇 상태에서 소스 리비전, 패키지 버전, 컨테이너 다이제스트, 설정, 배포 이벤트까지 완전한 추적성(Traceability)을 확보하면 현장 장애 조사 시간을 크게 줄일 수 있다.

운영 절차(Operational Procedure)는 배포, 롤백, 긴급 개입, 보안 대응, 유지보수에 대한 역할과 권한을 정의해야 한다. 자동화는 반복적인 작업을 일관되게 수행할 수 있지만 소프트웨어 변경이 로봇의 움직임과 주변 환경과의 상호작용에 영향을 미칠 수 있으므로 물리 로봇에는 여전히 명확한 거버넌스가 필요하다. 영향이 큰 작업은 감사 가능하고 수행 주체를 추적할 수 있어야 하며 적절한 권한과 로봇 안전 상태에 따라 제한되어야 한다.

대규모 ROS 2 운영은 궁극적으로 로봇 로컬 자율성(Robot-Local Autonomy), 사이트 엣지 인프라(Site Edge Infrastructure), 온프레미스 서비스, 중앙 플릿 거버넌스(Centralized Fleet Governance)를 연결하는 계층형 관리 아키텍처(Hierarchical Management Architecture)를 필요로 한다. ROS 2는 분산 로봇 통신을 제공하며 컨테이너, OTA, CI/CD, 모니터링, 로깅, 보안, 오케스트레이션은 이를 둘러싼 운영 프레임워크를 제공한다. 이들을 결합하면 안전한 로봇 운영을 지속적인 중앙 연결에 의존시키지 않으면서 재현 가능한 소프트웨어 배포와 조정된 수명주기 관리를 수행할 수 있다.

성숙한 플릿 배포 시스템(Mature Fleet Deployment System)은 모든 로봇을 자율적인 물리 기계이면서 동시에 관리되는 소프트웨어 플랫폼으로 취급한다. 식별, 설정, 소프트웨어 기준선, 단계적 업데이트, 롤백, 관찰 가능성, 보안, 호환성, 운영 상태를 하나의 수명주기를 구성하는 상호 연결된 요소로 관리한다. 목표는 단순히 ROS 2 소프트웨어를 대규모로 배포하는 것이 아니라 수년간의 실제 운영 기간 동안 신뢰할 수 있고 안전하며 진단 가능하고 지속적으로 진화할 수 있는 플릿을 유지하는 것이다.
