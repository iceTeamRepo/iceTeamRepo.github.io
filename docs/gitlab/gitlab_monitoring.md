---
layout: default
title: Configure Gitlab Monitoring
permalink: gitlab/gitlab_monitoring
nav_order: 6
---


# Configure Gitlab Monitoring
- GitLab Performance Monitoring 는 GitLab에서 제공하는 애플리케이션 성능 측정 시스템
- 커뮤니티와 엔터프라이즈 에디션 모두에서 사용 가능

## 주요 측정 항목
- 트랜잭션 시간: 웹 요청 또는 Sidekiq 작업 완료 시간
- SQL 쿼리 및 HAML 뷰 렌더링 시간
- Ruby 메서드 실행 시간
- Ruby 객체 할당 및 유지 객체
- 시스템 통계: 프로세스의 메모리 사용량, 열린 파일 디스크립터 등
- Ruby 가비지 컬렉션 통계

## 측정되는 메트릭 종류
- 트랜잭션 메트릭
    - 개별 트랜잭션과 연관된 메트릭
    - 예: 웹 요청, SQL 쿼리 시간, HAML 렌더링 시간
- 샘플링 메트릭
    - 특정 트랜잭션과 연관되지 않은 메트릭
    - 예: 가비지 컬렉션 통계, 유지된 Ruby 객체 등. 일정 간격으로 수집되며, 실제 간격은 사용자 정의 값에 랜덤한 오프셋이 추가된 형태
- 샘플링 간격
    - 샘플링 메트릭은 사용자 정의 간격을 기반으로 수집
        - 사용자 정의 간격 설정
            - 환경 변수
                - 샘플링 간격을 제어하는 환경 변수
                    - `RUBY_SAMPLER_INTERVAL_SECONDS`
                    - `DATABASE_SAMPLER_INTERVAL_SECONDS`
                    - `ACTION_CABLE_SAMPLER_INTERVAL_SECONDS`
                    - `PUMA_SAMPLER_INTERVAL_SECONDS`
                    - `THREADS_SAMPLER_INTERVAL_SECONDS`
                    - `GLOBAL_SEARCH_SAMPLER_INTERVAL_SECONDS`
        - 랜덤 오프셋
            - 자동으로 설정
            - 사용자 정의 간격이 15초, 랜덤 오프셋이 +/-50%  경우, 실제 간격은 7.5초(15초 - 7.5초) 에서 22.5초(15초 + 7.5초) 사이로 변동
            - 같은 오프셋 값은 두 번 연속으로 사용되지 않으며, 이를 통해 샘플링 간격의 예측 불가능성을 유지하고 시스템 성능 분석에 유연성을 더함

## Prometheus 
- 메트릭 수집
    - 대상 시스템 (Target System)
        - MySQL, Tomcat, VM 등 다양한 자원
    - Exporter
        - 대상 시스템에서 메트릭을 프로메테우스로 전송하는 에이전트
    - 풀링 방식 (Pulling) 사용
        - 프로메테우스가 주기적으로 Exporter로부터 메트릭을 수집
        - 풀링 방식의 한계: 가변적인 모니터링 대상을 추적하는 데 어려움
    - 서비스 디스커버리 (Service Discovery)
        - DNS, Consul, Kubernetes 등을 통해 동적 서비스 목록을 가져와 풀링 방식으로 메트릭을 수집
        - 예
            - 내부 DNS에 등록된 VM 목록을 통해 동적으로 IP를 추적
- 프로메테우스 구성 요소
    - Exporter
        - 대상 시스템에서 메트릭을 읽어 HTTP GET 요청으로 프로메테우스에 반환
        - 프로메테우스는 단순히 데이터를 읽고 저장하며, 데이터를 처리하거나 히스토리를 저장하지 않음
    - Retrieval
        - 서비스 디스커버리 시스템을 통해 모니터링 대상을 수집하고, Exporter로부터 메트릭을 주기적으로 수집
    - 저장
        - 수집된 메트릭은 로컬 디스크와 메모리에 저장
        - 장점: 설치가 간단
        - 단점: 스케일링이 어려움 (디스크 공간 증가로 해결)
        - HA 및 클러스터링
            - 기본적으로 HA와 클러스터링을 지원하지 않음, 샤딩 방식 사용
        - Thano
            - HA와 클러스터링 문제 해결을 위한 오픈 소스
    - 서빙
        - 저장된 메트릭은 PromQL을 통해 조회 가능
        - 그라파나와 연동하여 대시보드 구성 가능

## Gitlab 에서 Prometheus 설정

### Gitlab 인스턴스 내에 설치된 Prometheus 사용시 설정
- 설명
    - GitLab은 기본적으로 Prometheus 모니터링을 제공
- 설정
    - 기본 설정
        - Prometheus와 그 exporters는 기본적으로 활성화되어 있고 `gitlab-prometheus` 사용자로 실행됨
        - Prometheus는 기본적으로 `http://localhost:9090`에서 실행되며, 기본적으로 GitLab 서버 자체에서만 접근할 수 있음
        - 비활성화 시
            
            ```ruby
            # /etc/gitlab/gitlab.rb
            prometheus_monitoring['enable'] = false
            sidekiq['metrics_enabled'] = false
            puma['exporter_enabled'] = false  # 기본값은 이미 false로 설정되어 있지만, 확실히 비활성화하려면 명시적으로 설정
            
            ```
            
    - 자체 설치 시
        - GitLab을 자체적으로 컴파일한 경우, Prometheus를 직접 설치하고 구성해야 함
    - Exporter 설정
        - 각 exporter는 자동으로 Prometheus의 모니터링 대상에 추가됨
        - 개별적으로 비활성화할 수 있음
    - 포트 및 주소 변경
        
        ```ruby
        # /etc/gitlab/gitlab.rb
        # prometheus['listen_address'] = 'localhost:9090' # Default 
        prometheus['listen_address'] = '0.0.0.0:9090'  # 모든 호스트에서 접근 허용
        ```
        
        - Prometheus가 수신하는 포트를 변경하는 것은 다른 서비스와 충돌할 수 있어 권장되지 않으며, 변경 시 주의가 필요
    - 커스텀 스크랩 구성 추가
        
        ```ruby
        # /etc/gitlab/gitlab.rb
        # http://1.1.1.1:8060/probe?param_a=test&param_b=additional_test 를 스크랩
        #  job_name: 스크랩 작업 이름 (예: custom-scrape).
        #  metrics_path: 스크랩할 경로 (예: /probe).
        #  params: 쿼리 파라미터를 추가할 수 있습니다 (예: param_a와 param_b).
        #  static_configs: 스크랩할 타겟 IP와 포트 (예: 1.1.1.1:8060).
        prometheus['scrape_configs'] = [
          {
            'job_name': 'custom-scrape',
            'metrics_path': '/probe',
            'params' => {
              'param_a' => ['test'],
              'param_b' => ['additional_test'],
            },
            'static_configs' => [
              'targets' => ['1.1.1.1:8060'],
            ],
          },
        ]
        ```
        
    - Prometheus 저장소 보유 크기 구성
        
        ```ruby
        prometheus['flags'] = {
          'storage.tsdb.path' => "/var/opt/gitlab/prometheus/data",  # 데이터 저장 경로
          'storage.tsdb.retention.time' => "7d",  # 7일 동안 데이터 보유
          'storage.tsdb.retention.size' => "2GB", # 저장할 수 있는 최대 데이터 블록 크기
          'config.file' => "/var/opt/gitlab/prometheus/prometheus.yml"  # 구성 파일 경로
        }
        
        $ sudo gitlab-ctl reconfigure
        ```
        
    - 메트릭 보기
        - Prometheus는 기본적으로 `http://localhost:9090`에서 제공하는 대시보드를 통해 성능 메트릭을 확인
        - 접근 시 주의사항
            - **SSL 사용 시 문제**: GitLab 인스턴스에서 SSL이 활성화되어 있으면, 동일한 FQDN을 사용하는 GitLab과 Prometheus에 한 브라우저에서 동시에 접근할 수 없을 수 있습니다. 이는 HSTS(HTTP Strict Transport Security)로 인해 발생합니다.
            - **해결 방법**:
                - 별도의 FQDN 사용
                - 서버 IP를 통해 접근
                - 다른 브라우저를 사용하여 Prometheus에 접근
                - HSTS 리셋
                - NGINX를 프록시로 설정하여 접근

### 외부 Prometheus (옴니버스 패키지 매니저 사용하여 설치) 사용시 설정
1. GitLab Linux 패키지 설치 
2. Consul 서버의 IP 주소 또는 DNS 레코드 수집 
3. /etc/gitlab/gitlab.rb 파일 편집
    
    ```ruby
    # /etc/gitlab/gitlab.rb
    roles ['monitoring_role']
    
    external_url 'http://gitlab.example.com'
    
    # Prometheus
    prometheus['listen_address'] = '0.0.0.0:9090'
    prometheus['monitor_kubernetes'] = false
    
    # Prometheus를 위한 서비스 발견 활성화
    # consul 설정을 활성화한 후에는 
    # prometheus['scrape_configs'] 설정을 추가하지 않아야 함
    # 두 설정을 동시에 활성화하면 오류가 발생
    consul['enable'] = true
    consul['monitoring_service_discovery'] = true
    consul['configuration'] = {
       retry_join: %w(10.0.0.1 10.0.0.2 10.0.0.3), # IP나 FQDN을 사용
    }
    
    # Nginx - Grafana 접근을 위한 설정
    nginx['enable'] = true
    
    $ sudo gitlab-ctl reconfigure
    
    # /etc/gitlab/gitlab.rb
    # 다른 노드들에 모니터링 노드의 정보를 전달
    gitlab_rails['prometheus_address'] = '10.0.0.1:9090'
    
    $ sudo gitlab-ctl reconfigure
    ```
    
    
### 외부 Prometheus 서버 사용시 설정
- 설명
    - Prometheus와 대부분의 exporters는 인증을 지원하지 않으므로, 외부 네트워크에 노출하는 것을 권장하지 않음
    - GitLab을 외부 Prometheus 서버로 모니터링하려면 몇 가지 설정 변경이 필요
- 설정
    1. GitLab 서버에서 Prometheus 비활성화 
        
        ```ruby
        # /etc/gitlab/gitlab.rb
        prometheus['enable'] = false
        ```
        
    2. 각 서비스의 exporter를 네트워크 주소에서 수신하도록 설정
        
        ```ruby
        #### Gitlab Instance
        # 외부 Prometheus 사용
        prometheus['enable'] = false
        prometheus_monitoring['enable'] = false
        gitlab_rails['prometheus_address'] = '${private_ip_prometheus}:9090' #'http://prometheus.${gitlab_host}:9090'
        
        # 외부 Prometheus 사용위한 listen_addr 변경
        prometheus['listen_address'] = '0.0.0.0:9090'
        node_exporter['listen_address'] = '0.0.0.0:9100'
        gitlab_workhorse['prometheus_listen_addr'] = '0.0.0.0:9229'
        gitlab_exporter['listen_address'] = '0.0.0.0'
        gitlab_exporter['listen_port'] = '9168'
        registry['debug_addr'] = '0.0.0.0:5001'
        sidekiq['listen_address'] = '0.0.0.0'
        
        #redis_exporter['listen_address'] = '0.0.0.0:9121' # elasticache 사용시 의미 없음
        #postgres_exporter['listen_address'] = '0.0.0.0:9187' # rds 사용시 의미 없음
        #pgbouncer_exporter['listen_address'] = '0.0.0.0:9188' # rds 사용시 의미 없음
        gitaly['configuration'] = { # gitaly 를 외부 vm 에 배포한 경우 의미 없음 
          prometheus_listen_addr: '0.0.0.0:9236'
        }
        
        # Exporter Enable
        #postgres_exporter['enable'] = true
        #pgbouncer_exporter['enable'] = true
        gitlab_exporter['enable'] = true
        #redis_exporter['enable'] = true
        #puma['exporter_enabled'] = true
        #puma['exporter_address'] = "127.0.0.1"
        #puma['exporter_port'] = 8083
        node_exporter['enable'] = true
         
        # NGINX 매트릭 수집할 경우 NGINX 에 Prometheus 서버 IP 접근 허용
        nginx['status'] = {
          "listen_addresses" => ["0.0.0.0"],
          #"fqdn" => "dev.example.com",  # nginx 서버의 FQDN
          "port" => 8060, # 포트 번호
          "options" => {
            "server_tokens" => "off",
            "access_log" => "on", # 통계 로그 활성화
            "allow" => "${private_ip_prometheus}", # prometheus 서버만 접근 허용
            "deny" => "all" # 다른 모든 IP 차단
          }
        }
        
        # Prometheus 서버가 GitLab의 메트릭 엔드포인트에서 데이터를 가져올 수 있도록 IP 화이트리스트에 추가
        gitlab_rails['monitoring_whitelist'] = ['0.0.0.0/0', '::1/128']
        
        ### Gitaly Instance 
        gitaly['configuration'] = {
          ...
          prometheus_listen_addr: '0.0.0.0:9236', 
          ...
        }
        
        ### Prafect Instances
        praefect['configuration'] = {
          ...
          prometheus_listen_addr: '0.0.0.0:9652',
          prometheus_exclude_database_from_default_metrics: true,
          ...
        }
        
        ### 각 서버들에서 reconfigure
        $ sudo gitlab-ctl reconfigure
        ```
        
    3. Gitlab 인스턴스 사용 포트에 대한 인바운드 방화벽 설정
    4. Prometheus를 설치
    5. Prometheus 서버 설정
        - Prometheus 서버의 설정 파일에서 각 노드의 exporter를 스크랩 대상으로 추가
            - 예시 설정
                
                ```yaml
                # 1.1.1.1 은 Gitlab Server IP
                scrape_configs:
                  - job_name: nginx
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:8060
                  - job_name: redis
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:9121
                  - job_name: postgres
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:9187
                  - job_name: node
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:9100
                  - job_name: gitlab-workhorse
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:9229
                  - job_name: gitlab-rails
                    metrics_path: "/-/metrics"
                    scheme: https
                    static_configs:
                      - targets:
                        - <gitlab URL>
                     tls_config:
                      insecure_skip_verify: true
                  - job_name: gitlab-sidekiq
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:8082
                  - job_name: gitlab_exporter_database
                    metrics_path: "/database"
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:9168
                  - job_name: gitlab_exporter_sidekiq
                    metrics_path: "/sidekiq"
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:9168
                  - job_name: praefect
                    static_configs:
                      - targets: 
                        - <parefects instance1 ip>:9652
                        - <parefects instance2 ip>:9652
                        - <parefects instance2 ip>:9652
                  - job_name: gitaly  
                    static_configs:
                      - targets:
                        - <gitaly instance1 ip>:9236
                        - <gitaly instance2 ip>:9236
                        - <gitaly instance2 ip>:9236
                  - job_name: registry
                    static_configs:
                      - targets:
                        - <gitlab instance ip>:5001
                ```
                
    6. Prometheus 서버 재시작
        - Prometheus 서버를 다시 시작하여 새로운 설정을 적용

## Reference

- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/](https://docs.gitlab.com/ee/administration/monitoring/prometheus/)
- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/web_exporter.html](https://docs.gitlab.com/ee/administration/monitoring/prometheus/web_exporter.html)
- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/redis_exporter.html](https://docs.gitlab.com/ee/administration/monitoring/prometheus/redis_exporter.html)
- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/postgres_exporter.html](https://docs.gitlab.com/ee/administration/monitoring/prometheus/postgres_exporter.html)
- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/pgbouncer_exporter.html](https://docs.gitlab.com/ee/administration/monitoring/prometheus/pgbouncer_exporter.html)
- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/registry_exporter.html](https://docs.gitlab.com/ee/administration/monitoring/prometheus/registry_exporter.html)
- [https://docs.gitlab.com/ee/administration/monitoring/prometheus/gitlab_exporter.html](https://docs.gitlab.com/ee/administration/monitoring/prometheus/gitlab_exporter.html)