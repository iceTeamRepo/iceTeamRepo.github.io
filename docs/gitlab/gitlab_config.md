---
layout: default
title: Gitlab Configuration
permalink: gitlab/gitlab_config
nav_order: 2
---

# Gitlab Configuration

Gitlab Configuration 에 대한 설명 페이지


## [Nginx](https://docs.gitlab.com/omnibus/settings/nginx.html)
- 설명
    - GitLab의 NGINX 설정은 다양한 서비스에 대해 개별적으로 구성할 수 있음
    - 이 설정은 GitLab 자체와 Mattermost, Registry, Pages와 같은 다른 서비스에 대해 조정 가능
    - 기본적으로 제공되는 NGINX 설정 외에도, 복잡한 환경이나 리버스 프록시, 로드 밸런서를 사용하는 경우 설정을 수정하여 최적화할 수 있음
- 기본 설정
    - **NGINX Listen 주소 설정**
        - 기본적으로 NGINX는 모든 로컬 IPv4 주소에서 수신
        - 이를 변경하려면 `listen_addresses`를 설정
            
            ```ruby
            nginx['listen_addresses'] = ["0.0.0.0", "[::]"]
            registry_nginx['listen_addresses'] = ['*', '[::]']
            mattermost_nginx['listen_addresses'] = ['*', '[::]']
            pages_nginx['listen_addresses'] = ['*', '[::]']
            ```
            
    - **NGINX Listen 포트 설정**
        - 기본적으로 NGINX는 `external_url`에 설정된 포트나 HTTP(80) 또는 HTTPS(443)를 사용하여 수신
        - 이를 변경하려면 `listen_port`를 설정
            
            ```ruby
            nginx['listen_port'] = 8081
            ```
            
    - **NGINX 로그 레벨 변경**
        - 기본적으로 NGINX는 에러 로그 레벨을 사용
        - 로그 레벨을 `debug`로 변경하려면 `error_log_level`을 설정
            
            ```ruby
            nginx['error_log_level'] = "debug"
            ```
            
    - **Referrer-Policy 헤더 설정**
        - 기본적으로 GitLab은 `strict-origin-when-cross-origin`으로 `Referrer-Policy` 헤더를 설정
        - 이를 변경하려면 `referrer_policy`를 설정
            
            ```ruby
            # 설정 변경
            nginx['referrer_policy'] = 'same-origin'
            
            # 비활성화
            nginx['referrer_policy'] = false
            ```
            
    - **Gzip 압축 비활성화**
        - 기본적으로 GitLab은 10,240 바이트 이상인 텍스트 데이터를 Gzip으로 압축
        - 이를 비활성화하려면 `gzip_enabled`를 `false`로 설정
            
            ```ruby
            nginx['gzip_enabled'] = false
            ```
            
    - **프록시 요청 버퍼링 비활성화**
        - 특정 경로에 대해 프록시 요청 버퍼링을 비활성화하려면 `request_buffering_off_path_regex`를 설정
            
            ```ruby
            nginx['request_buffering_off_path_regex'] = "/api/v\\d/jobs/\\d+/artifacts$|/import/gitlab_project$|\\.git/git-receive-pack$|\\.git/ssh-receive-pack$|\\.git/ssh-upload-pack$|\\.git/gitlab-lfs/objects|\\.git/info/lfs/objects/batch$"
            ```
            
    - **NGINX 설정 변경 사항 적용**
        - `gitlab.rb` 파일에서 설정을 변경한 후, 설정을 적용하려면 GitLab을 재구성
            
            ```bash
            sudo gitlab-ctl reconfigure
            ```
            
    - **NGINX 설정 변경 사항을 부드럽게 적용**
        - 설정을 재구성하지 않고 NGINX 구성을 부드럽게 적용하려면 `hup` 명령을 사용할 수 있음
            
            ```bash
            sudo gitlab-ctl hup nginx
            ```
            
    - **`robots.txt` 파일 구성**
        - 사용자 정의 `robots.txt` 파일 생성 후 경로 확인
            
            ```ruby
            nginx['custom_gitlab_server_config'] = "\nlocation =/robots.txt { alias /path/to/custom/robots.txt; }\n"
            ```
            
            - `/path/to/custom/robots.txt`를 실제 경로로 교체
    - **GitLab 서버 블록에 맞춤형 NGINX 설정 추가**
        - `/etc/gitlab/gitlab.rb` 파일 편집하여 사용자 정의 설정 추가
            
            ```ruby
            nginx['custom_gitlab_server_config'] = "location ^~ /foo-namespace/bar-project/raw/ {\n deny all;\n}\n"
            ```
            
        - **주의 사항**:
            - `/assets` 및 루트 `/` 위치는 이미 정의되어 있어 추가 불가
            - 새 위치 추가 시 `proxy_cache off;`, `proxy_http_version 1.1;`, `proxy_pass http://gitlab-workhorse;` 포함 필요
    - **NGINX 구성에 맞춤형 설정 추가**
        - `/etc/gitlab/gitlab.rb` 파일 편집하여 추가 설정 정의
            
            ```ruby
            nginx['custom_nginx_config'] = "include /etc/gitlab/nginx/sites-enabled/*.conf;"
            ```
            
        - 서버 블록 생성
            - `/etc/gitlab/nginx/sites-available/`에서 서버 블록 파일 생성
            - `/etc/gitlab/nginx/sites-enabled/`에 심볼릭 링크 생성
                
                ```bash
                sudo ln -s /etc/gitlab/nginx/sites-available/example.conf /etc/gitlab/nginx/sites-enabled/example.con
                ```
                
        - NGINX 설정 재적용
            
            ```bash
            sudo gitlab-ctl hup nginx   # 설정을 재로드
            # 또는
            sudo gitlab-ctl restart nginx  # NGINX 재시작
            ```
            
    - **맞춤형 오류 페이지 구성**
        - `/etc/gitlab/gitlab.rb` 파일 편집하여 오류 페이지 설정 추가.
            
            ```ruby
            nginx['custom_error_pages'] = {
              '404' => {
                'title' => 'Example title',
                'header' => 'Example header',
                'message' => 'Example message'
              }
            }
            ```
            
- 모니터링 설정
    - **NGINX 상태 모니터링 설정**
        - GitLab은 기본적으로 **127.0.0.1:8060/nginx_status**에서 NGINX 서버 상태를 모니터링할 수 있는 엔드포인트를 제공
        - 이 엔드포인트는 NGINX의 현재 상태에 대한 정보를 표시
        - 상태 정보
            - **Active connections**: 열린 연결 수
            - **Accepts**: 수락된 연결 수
            - **Handled**: 처리된 연결 수
            - **Requests**: 총 요청 수
            - **Reading**: 요청 헤더를 읽고 있는 연결 수
            - **Writing**: 요청 본문을 읽거나 응답을 보내고 있는 연결 수
            - **Waiting**: 유지되는 연결 수 (keep-alive 연결)
        - 상태 엔드포인트 설정
            - `/etc/gitlab/gitlab.rb` 파일을 수정하여 NGINX 상태 엔드포인트를 설정
                
                ```ruby
                # 활성화
                nginx['status'] = {
                  "listen_addresses" => ["127.0.0.1"],
                  "fqdn" => "dev.example.com",  # 서버의 FQDN
                  "port" => 9999,               # 포트 번호
                  "options" => {
                    "access_log" => "on",      # 통계 로그 활성화
                    "allow" => "127.0.0.1",    # localhost만 접근 허용
                    "deny" => "all"            # 다른 모든 IP 차단
                  }
                }
                
                # 비활성화
                nginx['status'] = {
                  'enable' => false           # 상태 엔드포인트 비활성화
                }
                ```
                
        - 설정을 저장하고 GitLab을 재구성
            
            ```bash
            sudo gitlab-ctl reconfigure
            ```
            
    - **NGINX VTS 모듈을 사용한 고급 성능 메트릭 설정**
        - GitLab에서는 **NGINX VTS (Virtual host Traffic Status)** 모듈을 사용하여 추가적인 성능 메트릭, 특히 지연(latency) 백분위수(latency percentiles)를 제공
        - 이 모듈을 활성화하면 메모리 사용량이 증가하고, 각 요청에 대해 CPU가 소모될 수 있음
        - Prometheus에서 메트릭을 수집하려면 추가 저장 공간이 필요
        - 설정
            - Gitlab
                1. VTS 설정 파일 생성
                    
                    ```bash
                    sudo mkdir -p /etc/gitlab/nginx/conf.d/
                    sudo vim /etc/gitlab/nginx/conf.d/vts-custom.conf
                    ```
                    
                2.  설정 파일에 히스토그램 버킷과 필터링을 활성화 옵션 삽입
                    
                    ```
                    vhost_traffic_status_histogram_buckets 0.005 0.01 0.05 0.1 0.25 0.5 1 2.5 5 10;
                    vhost_traffic_status_filter_by_host on;
                    vhost_traffic_status_filter on;
                    vhost_traffic_status_filter_by_set_key $server_name server::*;
                    ```
                    
                3. GitLab 설정 파일 수정
                    
                    ```ruby
                    nginx['custom_nginx_config'] = "include /etc/gitlab/nginx/conf.d/vts-custom.conf;"
                    ```
                    
                4. GitLab 재구성 및 NGINX 재시작
                    
                    ```bash
                    sudo gitlab-ctl reconfigure
                    sudo gitlab-ctl restart nginx
                    ```
                    
            - Prometheus 쿼리 예시
                - NGINX VTS 모듈에서 제공하는 성능 지표를 Prometheus를 통해 모니터링
                    
                    ```bash
                    # 평균 응답시간
                    rate(nginx_vts_server_request_seconds_total[5m]) / rate(nginx_vts_server_requests_total{code=~"2xx|3xx|4xx|5xx"}[5m])
                    
                    # P90 지연
                    histogram_quantile(0.90, rate(nginx_vts_server_request_duration_seconds_bucket[5m]))
                    
                    # P99 지연
                    histogram_quantile(0.99, rate(nginx_vts_server_request_duration_seconds_bucket[5m]))
                    
                    # 업스트림 평균 응답 시간
                    rate(nginx_vts_upstream_response_seconds_total[5m]) / rate(nginx_vts_upstream_requests_total{code=~"2xx|3xx|4xx|5xx"}[5m])
                    
                    # P90 업스트림 지연
                    histogram_quantile(0.90, rate(nginx_vts_upstream_response_duration_seconds_bucket[5m]))
                    
                    # P99 업스트림 지연
                    histogram_quantile(0.99, rate(nginx_vts_upstream_response_duration_seconds_bucket[5m]))
                    
                    # GitLab Workhorse에 대한 메트릭----
                    #  GitLab Workhorse의 90번째 백분위수 업스트림 지연
                    histogram_quantile(0.90, rate(nginx_vts_upstream_response_duration_seconds_bucket{upstream="gitlab-workhorse"}[5m]))
                    
                    # GitLab Workhorse의 평균 업스트림 응답 시간
                    rate(nginx_vts_upstream_response_seconds_total{upstream="gitlab-workhorse"}[5m]) / 
                    rate(nginx_vts_upstream_requests_total{upstream="gitlab-workhorse",code=~"2xx|3xx|4xx|5xx"}[5m])
                    ```
                    
    - **GitLab에서 사용자 업로드 파일을 NGINX 웹 서버가 제대로 접근하고 사용할 수 있도록 설정**
        - GitLab은 파일 업로드를 처리할 때, NGINX(웹 서버)를 사용하여 해당 파일들을 서비스
        - GitLab 내부에서 업로드되는 파일은 특정 그룹(`gitlab-www`)에 속한 사용자만 접근할 수 있도록 권한이 설정됨
        - NGINX가 `gitlab-www` 그룹에 속하지 않으면 파일에 접근할 수 없게 됨
        - 설정
            
            ```bash
            # www-data 사용자(NGINX 사용자가 GitLab 업로드 파일에 접근할 수 있도록)
            # gitlab-www 그룹에 www-data 사용자를 추가하는 명령어
            sudo usermod -aG gitlab-www www-data
            ```
            
- 기타 설정
    - **NGINX 설정 위치**
        - GitLab 설정 파일인 `/etc/gitlab/gitlab.rb`에서 설정을 수정
        - GitLab Rails 애플리케이션과 다른 서비스별 NGINX 설정은 `nginx['<setting>']`와 `service_nginx['<setting>']` 형식을 사용하여 각각 수정
    - **서비스별 NGINX 설정**
        - GitLab 자체의 NGINX 설정은 `nginx['<setting>']`을 사용하여 수정
        - Mattermost, Registry, Pages와 같은 서비스에 대해서는 `mattermost_nginx['<setting>']`, `registry_nginx['<setting>']` 등으로 별도로 설정을 적용
            - 예: GitLab의 HTTP -> HTTPS 리디렉션을 활성화하려면 다음과 같이 설정
                
                ```bash
                nginx['redirect_http_to_https'] = true
                registry_nginx['redirect_http_to_https'] = true
                mattermost_nginx['redirect_http_to_https'] = true
                ```
                
    - **HTTPS 설정**
        - 기본적으로 Linux 패키지 설치는 HTTPS를 사용하지 않지만, 이를 활성화하려면 다음과 같은 방법을 사용
            - [Let’s Encrypt를 이용해 무료로 자동 HTTPS 인증서를 설정](https://docs.gitlab.com/omnibus/settings/ssl/index.html#enable-the-lets-encrypt-integration)
            - [수동으로 HTTPS 인증서를 설정](https://docs.gitlab.com/omnibus/settings/ssl/index.html#configure-https-manually)
            - 리버스 프록시나 외부 로드 밸런서를 사용하여 SSL 종료를 처리하는 경우
                - [GitLab의 NGINX에서 SSL 처리를 비활성화하고 외부에서 SSL을 처리하도록 설정](https://docs.gitlab.com/omnibus/settings/ssl/index.html#configure-a-reverse-proxy-or-load-balancer-ssl-termination)
    - **프록시 헤더 변경**
        - GitLab이 리버스 프록시 뒤에 있을 때, 기본 프록시 헤더 설정이 자동으로 구성됨
        - 복잡한 환경에서는 기본 설정을 변경해야 할 수 있음
        - 이 설정을 통해 CSRF 오류, 인증 오류 등을 방지할 수 있음
            
            ```bash
            # 리버스 프록시 뒤에서 SSL 종료를 처리하는 경우
            nginx['proxy_set_headers'] = {
              "X-Forwarded-Proto" => "http"
            }
            # 외부 프록시에서 클라이언트의 실제 IP를 전달하려는 경우
            nginx['proxy_set_headers'] = {
              "X-Forwarded-For" => "$remote_addr",
              "X-Real-IP" => "$remote_addr"
            }
            # 리버스 프록시가 특정 포트에서 요청을 전달하는 경우
            nginx['proxy_set_headers'] = {
              "X-Forwarded-Port" => "443"  # 프록시가 SSL을 처리하는 경우
            }
            
            # 기타 설정
            nginx['proxy_set_headers'] = { 
              "CUSTOM_HEADER" => "VALUE"
            }
            ```
            
    - **NGINX에서 실제 Client IP 주소 설정**
        - GitLab에서 실제 클라이언트 IP 주소를 로깅하려면 리버스 프록시를 사용할 때 NGINX의 real_ip 모듈을 설정 필요
        - 기본적으로 GitLab은 연결된 클라이언트의 IP 주소를 기록하지만, 리버스 프록시가 사용될 경우 프록시 서버의 IP가 클라이언트 IP로 기록될 수 있음
        - 이를 방지하고 실제 클라이언트의 IP를 기록하려면 `real_ip_trusted_addresses`를 설정해야 함
        - **설정 방법**
            
            ```bash
            # /etc/gitlab/gitlab.rb 파일 수정
            
            # 리버스 프록시의 IP 주소 설정
            #  예를 들어 프록시 서버가 192.168.1.0/24, 192.168.2.1과 같은 네트워크에 속한다면 이를 설정
            nginx['real_ip_trusted_addresses'] = [ '192.168.1.0/24', '192.168.2.1', '2001:0db8::/32' ]
            
            # X-Forwarded-For 헤더를 사용하여 실제 클라이언트 IP를 확인
            nginx['real_ip_header'] = 'X-Forwarded-For'
            
            # real_ip_recursive = 'on' 옵션은 X-Forwarded-For 헤더에 여러 IP가 나열된 경우, 가장 첫 번째 IP(즉, 실제 클라이언트 IP)를 사용하는 설정
            nginx['real_ip_recursive'] = 'on'
            ```
            
    - **PROXY 프로토콜 사용 설정**
        - NGINX가 리버스 프록시(예: HAProxy)로부터 클라이언트의 원래 IP 주소를 받아들이기 위해 필요한 설정
        - PROXY 프로토콜을 사용하면 리버스 프록시가 클라이언트의 실제 IP를 포함하는 PROXY 헤더를 전송하고, GitLab은 이 헤더를 통해 클라이언트 IP를 확인할 수 있음
        - **설정 방법**
            
            ```bash
            # /etc/gitlab/gitlab.rb 파일 수정
            
            #  PROXY 프로토콜을 활성화
            nginx['proxy_protocol'] = true
            
            # 신뢰할 수 있는 프록시 주소 설정
            nginx['real_ip_trusted_addresses'] = [ "127.0.0.0/8", "IP_OF_THE_PROXY/32" ]
            ```
            
    - **외부 웹 서버 사용 (Apache 또는 외부 NGINX)**
        - GitLab은 번들된 NGINX를 사용하지만, Apache나 외부 NGINX를 사용하려면 별도의 설정 필요
        - 설정 방법
            1. Bundled NGINX 비활성화
                
                ```bash
                nginx['enable'] = false
                ```
                
            2. 웹 서버 사용자 설정
                - GitLab은 외부 웹 서버 사용자(예: Apache의 `www-data`, NGINX의 `nginx`)에게 GitLab 파일에 대한 접근을 허용
                - 예시 (웹 서버 사용자가 `www-data`인 경우)
                    
                    ```bash
                    web_server['external_users'] = ['www-data']
                    ```
                    
            3. SELinux 설정 (선택 사항)
                - 만약 SELinux가 활성화되어 있고 외부 웹 서버가 제한된 SELinux 프로필 하에 있다면, 웹 서버의 프로필을 수정하거나 제한을 풀어야 할 수 있음
            4. `trusted_proxies` 설정
                - 외부 웹 서버가 GitLab과 동일한 서버에 있지 않은 경우, 그 웹 서버의 IP를 신뢰할 수 있는 프록시로 설정
                    
                    ```bash
                    gitlab_rails['trusted_proxies'] = [ '192.168.1.0/24', '192.168.2.1', '2001:0db8::/32' ]
                    ```
                    
            5. GitLab Workhorse 설정 (Apache 사용 시)
                - Apache는 UNIX 소켓을 사용하지 않으므로 TCP 포트를 통해 연결
                - 기본 포트는 8181
                    
                    ```bash
                    gitlab_workhorse['listen_network'] = "tcp"
                    gitlab_workhorse['listen_addr'] = "127.0.0.1:8181"
                    ```
                    
            6. 웹 서버 설정 파일 다운로드
                - GitLab의 [GitLab 레포지토리](https://gitlab.com/)에서 사용하려는 웹 서버에 맞는 설정 파일을 다운로드
                - 이 설정 파일에서 `YOUR_SERVER_FQDN`을 실제 FQDN으로 바꾸고, SSL을 사용하는 경우 SSL 키 위치 등을 지정
    - [GitLab을 기존 Passenger와 NGINX 설치로 호스팅 설정](https://docs.gitlab.com/omnibus/settings/nginx.html#use-an-existing-passenger-and-nginx-installation)
- 설정 적용
    - 설정을 변경한 후에는 GitLab을 재구성하여 변경 사항을 적용
        
        ```yaml
        sudo gitlab-ctl reconfigure
        ```

## Reference

- [https://docs.gitlab.com/ee/administration/configure.html](https://docs.gitlab.com/ee/administration/configure.html)
- [https://docs.gitlab.com/omnibus/settings/](https://docs.gitlab.com/omnibus/settings/)