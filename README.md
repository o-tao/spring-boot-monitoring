# Spring Boot Monitoring

애플리케이션의 메트릭과 상태를 모니터링하는 것은 애플리케이션의 성능을 개선하고, 보다 나은 관리를 가능하게 하며, 최적화되지 않은 동작을 발견하는데 큰 도움이 됩니다.   
Spring Boot Monitoring 학습을 위해 생성된 프로젝트입니다.   

- **Prometheus**
  - 역할: 애플리케이션 트래픽에 대한 메트릭 데이터를 수집. 이는 CPU 사용량, 메모리 사용률, 요청 수, 응답 시간 등의 애플리케이션 성능 데이터를 포함
  - 설정: `Prometheus`는 애플리케이션의 `/actuator/prometheus` 엔드포인트를 통해 메트릭 데이터를 수집
- **Grafana**
  - 역할: Prometheus에서 수집한 메트릭 정보를 시각화. 그래프, 차트, 대시보드를 통해 사용자에게 직관적으로 성능 정보를 제공하여 실시간 분석이 가능

# Prometheus

## 사용 라이브러리

```Groovy
implementation("org.springframework.boot:spring-boot-starter-web") // HTTP를 통한 접속
implementation("org.springframework.boot:spring-boot-starter-actuator") // 애플리케이션의 성능과 상태를 모니터링할 수 있는 엔드포인트 제공
runtimeOnly("io.micrometer:micrometer-registry-prometheus") // 메트릭 수집과 모니터링을 제공
```

## application.yml 설정

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus

```

- management: SpringBoot의 관리 기능. 애플리케이션은 내장된 모니터링 엔드포인트를 제공
- endpoints: 애플리케이션에서 제공하는 다양한 모니터링 엔드포인트를 정의
- web: endpoints에 정의된 엔드포인트가 `웹 기반`으로 접근 가능한지에 대한 여부, `HTTP` 요청을 통해 엔드포인트에 접근할 수 있도록 설정
- exposure: 접근 제어 정책 설정, 해당 설정을 통해 어떤 엔드포인트가 웹에서 접근 가능하게 될지 정의
- include
  - health: 애플리케이션의 `상태` `/actuator/health`
  - info: 애플리케이션의 `정보` `/actuator/info`
  - metrics: 애플리케이션의 `메트릭 데이터를 수집`하고 제공 `/actuator/metrics`
  - prometheus: `prometheus`와 통합된 메트릭 데이터를 수집, 메트릭 데이터를 수집하고 시각화 `/actuator/prometheus`

## Actuator가 제공하는 URL 확인

/actuator롤 통해 접속합니다. ex) `http://localhost:8080/actuator`      

  <img width="677" alt="http://localhost:8080/actuator" src="https://github.com/user-attachments/assets/04a292c8-ed39-40b4-a348-94ca3b793f27">

## prometheus.yml 설정

`prometheus.yml` 설정은 **Prometheus**가 애플리케이션의 메트릭을 수집하는 방법을 정의합니다.

```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  -   job_name: 'spring-boot-application'
      metrics_path: '/actuator/prometheus'
      static_configs:
        -   targets: [ 'host.docker.internal:8080' ]

```

- scrape_interval: Prometheus가 메트릭을 수집할 주기를 지정
- evaluation_interval: 메트릭 평가 간격을 설정, 수집한 메트릭 데이터를 분석, 경고 규칙을 평가하는 주기

- job_name: 특정 메트릭 수집 작업의 이름을 지정
- metrics_path: Prometheus가 수집할 메트릭 엔드포인트의 경로를 설정
- targets: 메트릭을 수집할 서버 목록을 설정 (Docker 내의 SpringBoot 애플리케이션 서버의 메트릭을 수집)
  - host.docker.internal: Docker 컨테이너 내에서 외부 네트워크에 접근할 수 있는 주소

## docker-compose.yml 설정

`docker-compose.yml` 설정에서는 **Prometheus**와 **Grafana** 컨테이너를 정의합니다.   
`Prometheus`는 메트릭을 수집, `Grafana`는 이를 시작적으로 확인

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    container_name: prometheus

  grafana:
    image: "grafana/grafana:latest"
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=1234
    container_name: grafana

```

- Prometheus
  - image: Docker 컨테이너의 이미지를 정의
    - latest: 최신버전
  - ports: 사용할 포트를 지정
    - localhost:9090을 통해 Prometheus 웹 인터페이스에 접근
  - volumes
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - Prometheus 설정 파일을 프로젝트의 `./prometheus.yml` 경로에서 컨테이너 내의 `/etc/prometheus/prometheus.yml`로 마운트
  - container_name: Docker 컨테이너의 이름을 지정

- Grafana
  - environment
    - GF_SECURITY_ADMIN_USER: 사용자 이름 설정
    - GF_SECURITY_ADMIN_PASSWORD: 사용자 비밀번호 설정

```bash
# docker-compose 실행
$ docker-compose up -d

#docker-compose 정리
$ docker-compose down

```

## Prometheus 접속

`docker-compose`를 실행한 뒤, `http://localhost:9090`을 통해 `Prometheus`에 접속합니다.   

<img width="955" alt="스크린샷 2024-12-09 22 53 50" src="https://github.com/user-attachments/assets/dd267006-5b34-48c9-9a71-af4aad0493dd">

상단 메뉴에서 `Status` → `Target health`로 이동하여 SpringBoot 애플리케이션을 수신하고 있는지 확인이 가능합니다.   

<img width="1903" alt="스크린샷 2024-12-09 22 57 11" src="https://github.com/user-attachments/assets/0ef79e83-f134-4d34-90be-ed1c7bda0465">   

# Grafana

`Grafana`는 오픈 소스 시각화 및 대시보드 도구로, 여러 데이터 소스의 메트릭을 시각적으로 표현하고 분석할 수 있습니다.   
`Prometheus`와 함께 사용하여 애플리케이션의 성능, 상태, 트래픽 데이터를 시각적으로 모니터링할 때 주로 사용됩니다.

`Grafana`도 `Prometheus`와 마찬가지로 `docker-compose`를 통해 관리가 가능합니다. (위 내용에서 설정)

## Grafana 접속

`http://localhost:3000`을 통해 `Grafana`에 접속이 가능합니다.   

<img width="589" alt="스크린샷 2024-12-09 23 04 10" src="https://github.com/user-attachments/assets/3c1e1231-2f6a-461f-a4b3-e8f831a4a99e">   

`docker-compose`에서 지정한 사용자 이름과 비밀번호를 입력하여 로그인합니다.   

## data source 추가

<div>
<img width="1900" alt="스크린샷 2024-12-09 23 06 39" src="https://github.com/user-attachments/assets/f8a53e68-feaf-4441-b67f-1d175a73b8fa">   
<br><br>
<img width="722" alt="스크린샷 2024-12-09 23 08 02" src="https://github.com/user-attachments/assets/aed53076-597f-43e7-a52d-bc645864dd56">   
<br><br>
<img width="1047" alt="스크린샷 2024-12-09 23 09 48" src="https://github.com/user-attachments/assets/c7e39fe7-9b78-4500-bee6-4d4a58ce1205">   
</div>

`http://host.docker.internal:9090`을 입력합니다.   
`Prometheus`의 포트를 앞서 설정하여 `http://localhost:9090`을 통하여 접속했었습니다.   
Docker 컨테이너 내에서 외부 네트워크에 접근할 수 있도록 포트번호를 `Prometheus`의 포트로 설정하여 입력합니다.   

<img width="913" alt="스크린샷 2024-12-09 23 14 48" src="https://github.com/user-attachments/assets/04924e34-53bf-4d1e-8b97-feb475993d95">   

`Save & test`를 클릭해 `Successfully queried the Prometheus API.`문구가 나온다면 정상적으로 추가된 것입니다.   

## Dashboards 추가

[Grafana Labs](https://grafana.com/grafana/dashboards)에 접속합니다.   

<img width="1242" alt="스크린샷 2024-12-09 23 22 48" src="https://github.com/user-attachments/assets/b183ee11-fe8d-4fbe-89ba-76ea56c5a39f">   

`Grafana Labs`에 접속하여 `SpringBoot`검색 → `JVM 선택`

<img width="303" alt="스크린샷 2024-12-09 23 24 01" src="https://github.com/user-attachments/assets/23afde36-c030-41a6-b1fd-97bc31b68b38">   

스크롤을 내리면 나오는 우측의 `ID` 값 복사   

<img width="784" alt="스크린샷 2024-12-09 23 25 37" src="https://github.com/user-attachments/assets/43f7ae1b-0117-484b-ae3f-911588fae7e9">

다시 `Grafana`로 이동하여 `Import dashboard` 검색

<img width="628" alt="스크린샷 2024-12-09 23 26 25" src="https://github.com/user-attachments/assets/fdecf67d-29d7-4ad2-81f9-5bea0c4221af">

복사한 `ID` 입력 후 `Load` 클릭

<img width="808" alt="스크린샷 2024-12-09 23 28 26" src="https://github.com/user-attachments/assets/e1247c86-7697-4e08-9346-8fbf569c66de">   

`Prometheus`로 앞서 생성한 `data sources` 선택 후 `import`   

<img width="1901" alt="스크린샷 2024-12-09 23 39 55" src="https://github.com/user-attachments/assets/8c8201c5-6f5d-4377-82ef-70041f32e297">   

`Prometheus`의 정보를 바탕으로 대시보드가 표현됩니다.   

<img width="372" alt="스크린샷 2024-12-09 23 57 26" src="https://github.com/user-attachments/assets/2de8e6ce-8e5c-4aec-bacc-a9de65fae791">   

## Grafana 대시보드 메트릭

| **카테고리**                | **메트릭**                    | **설명**                                                                                 |
|-----------------------------|-------------------------------|------------------------------------------------------------------------------------------|
| **Quick Facts**             | Uptime                       | 애플리케이션이 시작된 이후 경과 시간 (초 단위)                                           |
|                             | Start Time                   | 애플리케이션이 시작된 시간                                                              |
|                             | Heap Used                    | JVM에서 사용 중인 힙 메모리 크기 (MB 단위)                                               |
|                             | Non-Heap Used                | JVM에서 사용 중인 논힙 메모리 크기 (MB 단위)                                            |
| **I/O Overview**            | Rate                        | I/O 작업의 초당 요청 속도                                                               |
|                             | Errors                      | 발생한 I/O 작업 오류 수                                                                 |
|                             | Duration                    | I/O 작업의 평균 실행 시간                                                               |
|                             | Utilisation                 | I/O 리소스의 사용률                                                                     |
| **JVM Memory**              | JVM Heap                    | JVM에서 힙 메모리 사용량                                                                |
|                             | JVM Non-Heap                | JVM에서 논힙 메모리 사용량                                                              |
|                             | JVM Total                   | JVM에서 전체 메모리 사용량                                                              |
|                             | JVM Process Memory          | JVM 프로세스가 사용하는 메모리 양                                                       |
| **JVM Misc**                | CPU Usage                   | 애플리케이션의 CPU 사용량                                                               |
|                             | Load                        | 시스템의 CPU 부하                                                                       |
|                             | Threads                     | JVM에서 실행 중인 스레드 수                                                             |
|                             | Thread States               | JVM 스레드의 상태 (RUNNABLE, WAITING 등)                                               |
|                             | GC Pressure                 | 가비지 컬렉션 활동의 빈도                                                              |
|                             | Log Events                  | 로깅 이벤트 수                                                                          |
|                             | File Descriptors            | JVM에서 열려 있는 파일 디스크립터 수                                                   |
| **JVM Memory Pools (Heap)** | G1 Eden Space               | G1 GC의 Eden 메모리 공간                                                               |
|                             | G1 Old Gen                  | G1 GC의 Old Generation 메모리 공간                                                    |
|                             | G1 Survivor Space           | G1 GC의 Survivor 메모리 공간                                                          |
| **JVM Memory Pools (Non-Heap)** | Metaspace                  | JVM Metaspace 메모리 공간                                                              |
|                             | Compressed Class Space      | JVM에서 압축된 클래스 공간                                                              |
|                             | CodeCache                   | JVM 코드 캐시 메모리                                                                   |
| **Garbage Collection**      | Collections                 | GC가 실행된 횟수                                                                       |
|                             | Pause Durations             | GC로 인해 발생한 애플리케이션 일시 중지 시간                                          |
|                             | Allocated/Promoted          | GC에 의해 힙 메모리에서 이동된 객체 크기                                              |
| **Classloading**            | Classes Loaded              | JVM에서 로드된 클래스 수                                                               |
|                             | Class Delta                 | 클래스 로드 및 언로드의 변화량                                                        |
| **Buffer Pools**            | Direct                      | JVM Direct Buffer Pool 사용량                                                         |
|                             | Mapped                      | JVM Mapped Buffer Pool 사용량                                                         |
|                             | Mapped - 'Non-Volatile Memory' | JVM Mapped Buffer Pool 중 비휘발성 메모리 사용량                                        |


해당 학습을 통해 SpringBoot 애플리케이션의 모니터링이 어떻게 이루어지는지 이해하고 `Prometheus`와 `Grafana`를 활용한 데이터 수집 및 시각화 방법을 알 수 있었습니다.   
애플리케이션의 성능과 상태를 지속적으로 모니터링하는 것은 안정성과 확장성을 유지하는 데 매우 중요한 역할을 합니다. `Prometheus`와 `Grafana`는 이를 효과적으로 지원하는 도구입니다.
