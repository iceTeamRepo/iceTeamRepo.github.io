---
layout: default
title: Gitlab CICD
permalink: gitlab/gitlab_cicd
nav_order: 8
---

# Gitlab CICD

Gitlab CICD Keywords 에 대한 설명 페이지

- GitlabCI 은 Gitlab Repository 의 이벤트를 받아서 Gitlab Runner 에서 Script 를 실행하게 만드는 도구로 `.gitlab-ci.yaml` 파일을 생성하여 작성한다.  
- 좌측 컨텍스트 메뉴의 **Build → Pipeline** 에서 `.gitlab-ci.yaml` 파일의 실행 결과를 확인할 수 있다.


## Job

- 특정 컨테이너 이미지를 통해 정의한 명령어들을 순차적으로 실행
- 각 Job은 다른 환경에서 실행되며 기본적으로 병렬 수행
- Job 간의 파일 공유 또는 순서를 지정하기 위한 다양한 키워드:
    - **artifacts**: Job의 결과물 저장
    - **cache**: 재사용할 수 있는 데이터 저장
    - **needs**: 특정 Job의 완료를 기다림
- `exit 0` 이외의 종료는 모두 실패로 간주
	
## 문법

- [GitLab CI/CD YAML 문서](https://docs.gitlab.com/ee/ci/yaml/)
- `pipeline` vs `job`
        
| 특성 | 파이프라인 (Pipeline) | 작업 (Job) |
| --- | --- | --- |
| **개념** | 여러 개의 단계(stage)와 작업(job)들이 포함된 전체 프로세스 | CI/CD 프로세스 내에서 단계(stage)의 한 작업 단위 |
| **역할** | 작업을 실행하는 흐름을 정의하며, 전체 CI/CD 파이프라인을 구성 | 실제로 실행되는 개별적인 명령어 또는 스크립트의 집합 |
| **구성 요소** | 여러 단계와 각 단계에 포함된 작업들로 구성 | 각 작업은 stage, script와 같은 세부 설정을 가짐 |
| **실행 순서** | 순차적으로 실행되는 여러 단계의 집합 | 각 단계 내에서 정의된 작업이 순차적 또는 병렬로 실행 |
| **종류** | 파이프라인 자체는 하나만 존재하며, 다양한 파이프라인을 실행 가능 | 하나의 파이프라인에 여러 개의 작업이 존재 |
| **상태** | 파이프라인의 상태는 성공, 실패, 진행 중 등으로 구분 | 각 작업은 성공, 실패, 진행 중 등으로 구분 |
| **트리거** | 코드 푸시, 머지 요청 등으로 트리거 | 파이프라인이 트리거되면 각 작업이 실행 |


## [키워드](https://docs.gitlab.com/ee/ci/yaml/#keywords)

### 설정 샘플
    ```bash
    default:
      image: registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest
      id_tokens:
        GITLAB_OIDC_TOKEN:
          aud: https://gitlab.com
      before_script:
        - echo "before"
      after_script:
        - echo "after"
    
    variables:
      VAR: test-pipeline-var
    
    first:
      image: 
        name: zricethezav/gitleaks:latest
        entrypoint: [""]
      script:
        - echo ${VAR}
        - echo ${GITLAB_OIDC_TOKEN}
    
    second:
      variables:
        VAR: test
      script:
        - echo ${VAR}
        - aws
    
    ```

### Global keywords

- **default**
    - 기능
        - `default` 키워드는 파이프라인 내에서 공통적으로 적용될 기본값을 설정
        - 이 기본값은 특정 잡에서 따로 설정하지 않으면 자동으로 사용됨
    - 주의사항
        - 이미 해당 잡에서 특정 설정이 정의되어 있으면, `default`에서 설정한 값은 적용되지 않으며, 그 잡에 정의된 값이 우선
    - `default`에 설정할 수 있는 주요 키워드
        - **image**: 잡에서 사용할 도커 이미지. 예를 들어, `ruby:3.0`처럼 지정할 수 있습니다.
        - **retry**: 실패한 작업을 재시도할 횟수. 예를 들어, `retry: 2`로 설정하면 실패 시 최대 2번 재시도합니다.
        - **before_script**: 각 잡에서 실행할 스크립트 전에 실행될 기본 스크립트.
        - **after_script**: 각 잡에서 실행 후에 실행될 기본 스크립트.
        - **artifacts**: 작업이 성공적으로 완료된 후, 생성된 파일을 저장할 경로.
        - **cache**: 파이프라인 실행 간 공유할 캐시의 설정.
        - **hooks**: 다양한 시점에서 실행할 훅(hook) 설정.
        - **id_tokens**: 인증을 위한 ID 토큰 관련 설정.
        - **interruptible**: 잡을 중단할 수 있는지 여부 설정.
        - **services**: 파이프라인 실행 시 필요한 외부 서비스 설정.
        - **tags**: CI/CD 러너에 태그를 지정하여 특정 러너에서만 작업을 실행하게 할 수 있습니다.
        - **timeout**: 잡의 실행 시간 제한. (현재 이슈가 있으며 해당 키워드는 효과 없음)
    - `inherit:default`
        - default 키워드에 정의된 값을 사용할지 여부 결정
            
            ```yaml
            job1:
              script: echo "This job does not inherit any default keywords."
              inherit:
                default: false
            ```
            
    - 사용
        
        ```yaml
        default:
          키워드1: 값1
          키워드2: 값2
          ...
        ```
        
    - 예시
        
        ```yaml
        # default 섹션은 모든 잡에 기본값을 제공합니다. 
        # 여기서는 image: ruby:3.0과 retry: 2가 기본값으로 설정되었습니다.
        # rspec 잡은 image와 retry를 명시하지 않아서 default에서 설정한 값들이 자동으로 적용됩니다.
        # rspec 2.7 잡은 image를 ruby:2.7로 설정했지만, retry는 따로 설정하지 않았기 때문에 default에서 설정한 retry: 2가 적용됩니다.
        default:
          image: ruby:3.0
          retry: 2
        
        rspec:
          script: bundle exec rspec
        
        rspec 2.7:
          image: ruby:2.7
          script: bundle exec rspec
        
        nodefault:
          script: echo "This job does not inherit any default keywords."
          inherit:
            default: false
        ```
        
- **include**
    
    ```yaml
    include:
      - local: 'path/to/another/file.yml'
    ```
    
    - **설명**
        - GitLab은 `.gitlab-ci.yml` 파일 내에서 외부 YAML 파일을 포함할 수 있도록 `include` 키워드를 제공
        - CI/CD 구성을 더 잘 조직하고 읽기 쉽게 만들거나, 여러 곳에서 동일한 구성을 중복 없이 사용가능
        - `include`  파일들은 항상 `.gitlab-ci.yml` 파일보다 먼저 평가됨
            - include 로 포함된 YAML 파일에 정의된 `build` 작업이 `.gitlab-ci.yml` 파일에 있는 `build` 작업으로 덮어씌워지거나 default 키워드로 덮어씌워질 수 있음
        - 포함된 파일들이 30초 내에 해결되지 않으면, `include` 처리에 실패
    - **`include` 사용의 이점**
        - 가독성 향상
            - 긴 `.gitlab-ci.yml` 파일을 더 작은, 관리하기 쉬운 파일로 나누어 가독성을 높힘.
        - 중복 제거
            - 여러 프로젝트나 파이프라인에서 공통적으로 사용하는 빌드, 배포 등의 설정을 재사용
        - 중앙 집중화된 템플릿
            - 템플릿 파일을 중앙 저장소에 보관하여, 파일 업데이트를 간편하게 하고 여러 프로젝트에서 일관성 있는 구성을 유지가능
    - **`include` 파일 유형**
        - `include:component`
            - 설명
                - 다른 프로젝트나 모듈에서 CI/CD 구성 요소를 포함
                - 큰 파이프라인을 여러 개의 컴포넌트로 나누거나, 다른 GitLab CI/CD 구성 요소에서 설정을 재사용할 때 유용
                - 이 키워드를 사용하면 외부 저장소에 저장된 CI/CD 구성 요소(예: 보안 검사, 코드 품질 검사, 테스트 등을 처리하는 컴포넌트)를 파이프라인에 통합
            - 구문
                - <fully-qualified-domain-name>/<project-path>/<component-name>@<specific-version>
            - 예시
                
                ```yaml
                include:
                  - component: $CI_SERVER_FQDN/my-org/security-components/secret-detection@1.0
                ```
                
        - `include:local`
            - 설명
                - 로컬 파일을 포함하는 방법
                - 동일한 레포지토리와 브랜치 내에서 상대 경로를 사용해 YAML 파일을 포함
                - 예를 들어, 프로젝트 내의 `ci-templates` 폴더에 있는 YAML 파일을 포함할 수 있음
                - 파일 포함은 항상 현재 구성 파일의 위치를 기준으로 평가됨
            - 입력형식
                - 루트 디렉터리(`/`)에서의 상대 경로
                - `.yml` 또는 `.yaml` 확장자 필요
                - `` 및 `*` 와일드카드 사용 가능
                - 특정 CI/CD 변수 사용 가능
            - 예시
                
                ```yaml
                # 긴 경로
                include:
                  - local: '/templates/.gitlab-ci-template.yml'
                  
                # 짧은 경로  
                include: '.gitlab-ci-production.yml'
                ```
                
        - `include:project`
            - 설명
                - 동일한 Gitlab 인스턴스에 있는 다른 GitLab 프로젝트에서 파일을 포함
                - 이를 통해 다른 프로젝트에서 관리하는 공통적인 CI/CD 구성을 재사용
                - 파이프라인을 실행하는 사용자가 두 프로젝트에 대한 적절한 권한을 가져야 함
            - 입력 형식
                - `project`: 전체 GitLab 프로젝트 경로
                - `file`: 프로젝트 루트에서의 상대 경로 (배열 형식도 가능)
                - `ref` (옵션): 브랜치, 태그 또는 커밋 SHA
                    - 보안 관점에서 타 프로젝트의 CI/CD 구성을 포함할 때 SHA를 사용하여 고정된 버전을 참조하는 것이 안정적
            - 예시
                
                ```yaml
                # 다른 프로젝트의 head 에 있는 GitlabCICD 파일을 포함
                include:
                  - project: 'my-group/my-project'
                    file: '/templates/.gitlab-ci-template.yml'
                  - project: 'my-group/my-subgroup/my-project-2'
                    file:
                      - '/templates/.builds.yml'
                      - '/templates/.tests.yml'
                      
                # 특정 ref 를 지정하여 특정 버전을 포함
                include:
                  - project: 'my-group/my-project'
                    ref: main
                    file: '/templates/.gitlab-ci-template.yml'
                  - project: 'my-group/my-project'
                    ref: v1.0.0
                    file: '/templates/.gitlab-ci-template.yml'
                  - project: 'my-group/my-project'
                    ref: 787123b47f14b552955ca2786bc9542ae66fee5b
                    file: '/templates/.gitlab-ci-template.yml'
                ```
                
            - 
        - `include:remote`
            - 설명
                - 외부 URL에서 YAML 파일을 포함
                - 예를 들어, 인터넷 상의 공용 GitLab 템플릿 파일을 사용
                - 이 방법은 다른 GitLab 인스턴스나 원격 서버에서 파일을 불러오는 경우 유용
                - 모든 중첩된 포함은 공용 사용자로 실행되며, 따라서 공용 프로젝트 또는 템플릿만 포함 가능
                - 보안 관점에서 타 프로젝트의 구성 파일 변경 시 알림이 없으므로 관리가 필요
                - GitLab 프로젝트를 포함할 때는 보호된 브랜치 및 태그를 사용하는 것이 권장됨
            - 입력 형식
                - HTTP/HTTPS GET 요청으로 접근 가능한 전체 URL
                - `.yml` 또는 `.yaml` 확장자 필요
                - 특정 CI/CD 변수 사용 가능
            - 예시
                
                ```yaml
                include:
                  - remote: 'https://gitlab.com/example-project/-/raw/main/.gitlab-ci.yml'
                ```
                
        - `include:template`
            - 설명
                - GitLab에서 제공하는 기본 템플릿 파일을 포함
                - 예를 들어, GitLab이 제공하는 공식 CI/CD 템플릿을 사용하여 빠르게 파이프라인을 설정
            - 입력 형식
                - GitLab CI/CD 템플릿 파일 이름
                - 템플릿 목록은 `lib/gitlab/ci/templates`에서 확인 가능
                - 특정 CI/CD 변수 사용 가능
                - 모든 중첩 포함은 공용 사용자로 실행되므로, 공용 프로젝트 또는 템플릿만 포함 가능
                - 중첩된 템플릿에서는 변수 사용 불가
            - 예시
                
                ```yaml
                # GitLab 템플릿 파일 포함
                include:
                  - template: Auto-DevOps.gitlab-ci.yml
                  
                # 여러 템플릿을 포함
                include:
                  - template: Android-Fastlane.gitlab-ci.yml
                  - template: Auto-DevOps.gitlab-ci.yml
                
                ```
                
        - `include:inputs`
            - 설명
                - 포함된 파일에서 사용할 수 있는 변수나 파라미터를 정의
                - 이를 통해 포함된 구성 파일에서 외부 값을 동적으로 설정
                - `spec:inputs:type`을 사용하는 경우, 입력 값은 지정된 타입과 일치해야 함
                - `spec:inputs:options`를 사용하는 경우, 입력 값은 지정된 옵션 중 하나여야 함
            - 입력 형식
                - 문자열, 숫자, 또는 불리언 값
            - 예시
                
                ```yaml
                
                # custom_configuration.yml은 spec:inputs를 사용해 website 값이 "My website"로 설정됨
                include:
                  - local: 'custom_configuration.yml'
                    inputs:
                      website: "My website"
                ```
                
        - `include:rules`
            - 설명
                - 조건부 규칙을 사용해 특정 상황에서만 다른 구성 파일을 포함
            - 입력 형식
                - `rules:if`: 특정 조건이 참일 때 포함
                - `rules:exists`: 특정 파일이 존재할 때 포함
                - `rules:changes`: 특정 파일에 변경 사항이 있을 때 포함
            - 예시
                
                ```yaml
                # INCLUDE_BUILDS 변수가 "true"이면 build_jobs.yml이 포함
                include:
                  - local: build_jobs.yml
                    rules:
                      - if: $INCLUDE_BUILDS == "true"
                
                test-job:
                  stage: test
                  script: echo "This is a test job"
                ```
                
            - 일부 CI/CD 변수 지원
    - **`include` 추가 세부 사항**
        - GitLab의 `include` 기능을 사용할 때, 몇 가지 중요한 사항과 제한이 존재
        - CI/CD 변수 사용 제한
            - `include` 키워드와 함께 사용할 수 있는 CI/CD 변수는 제한적
            - 일부 변수는 외부 YAML 파일에서 사용할 수 없을 수 있으므로, 필요한 변수에 대한 문서화를 참조 필요
        - 병합(Merging)으로 포함된 설정 덮어쓰기
            - `.gitlab-ci.yml` 파일에서 동일한 job 이름이나  global keyword 를 사용하면, 포함된 설정을 덮어씀
        - 두 개의 설정이 병합되어, `.gitlab-ci.yml` 파일에 정의된 설정이 포함된 설정보다 우선 적용
            - 예를 들어, 포함된 YAML 파일에 정의된 `build` 작업이 `.gitlab-ci.yml` 파일에 있는 `build` 작업으로 덮어씌워짐
        - 파이프라인 실행 중의 `include` 파일 처리
            - 작업(Job) 재실행 시
                - 파이프라인의 작업을 재실행하더라도, 포함된 파일은 다시 로드되지 않음
                - 모든 작업은 파이프라인이 처음 실행될 때 가져온 설정을 사용
                - 즉, 이후에 변경된 포함 파일은 작업 재실행 시 영향을 미치지 않음
            - 파이프라인 재실행 시
                - 파이프라인을 새로 실행하면 포함된 파일을 다시 가져옴
                - 이전 파이프라인에서 포함된 파일이 변경된 경우, 새 파이프라인에서는 최신 파일을 반영한 구성이 적용됨
            - `include` 파일 제한
                - 기본적으로 한 파이프라인당 최대 150개의 `include`를 사용할 수 있으며 중첩된 `include`도 포함
                    - GitLab 16.0 이상 (자체 호스팅 사용자): 최대 `include` 수를 변경할 수 있음
                    - GitLab 15.10 이상: 최대 150개의 `include`를 지원. 중첩된 `include`에서 동일한 파일을 여러 번 포함할 수 있지만, 중복된 파일 포함은 제한에 포함
                    - GitLab 14.9 ~ 15.9: 최대 100개의 `include`를 지원하며, 중복된 `include`는 무시
- **stages**
    - 설명
        - 파이프라인의 Job 들을 stages 로 그룹핑하여 병렬실행
    - 기본값
        - `.gitlab-ci.yml` 파일에 `stages`가 정의되지 않으면 아래 stages 를 자동으로 포함하며 순차 실행
            1. `.pre`
            2. `build`
            3. `test`
            4. `deploy`
            5. `.post`
        - 위 stages 로 따로 정의 없이 사용 가능
        - 이외 내맘대로 stages 를 추가할 수 있고 이 경우 내가 정의한 순서대로 실행되며 암묵적인 stages 인 build,test,deploy 는 사용할 수 없음
    - 예시
        
        ```yaml
        # 내가 정의한 stages 이므로 build,test 역시 암묵적인 build,test 아니고 실행 순서는 내가 정의한 순서에 따름
        stages:
          - lint
          - build
          - test
          - security
          - package
          - deploy
          - cleanup
        
        # 코드 스타일 검사 단계
        code-lint:
          stage: lint
          script:
            - echo "Running code linting..."
            - npm run lint
          only:
            - merge_requests
          allow_failure: true  # 린트는 실패해도 다음 단계 진행
        
        # 빌드 단계
        build-app:
          stage: build
          script:
            - echo "Building application..."
            - npm install
            - npm run build
          artifacts:
            paths:
              - dist/  # 빌드 결과물 저장
        
        # 병렬 테스트
        unit-tests:
          stage: test
          script:
            - echo "Running unit tests..."
            - npm run test:unit
          coverage: '/Coverage: \d+\.\d+%/'  # 테스트 커버리지 정보 출력
        
        integration-tests:
          stage: test
          script:
            - echo "Running integration tests..."
            - npm run test:integration
        
        # 보안 검사
        security-scan:
          stage: security
          script:
            - echo "Running security scan..."
            - npm run scan
          rules:
            - if: $CI_COMMIT_BRANCH == "main"  # 메인 브랜치에서만 실행
        
        # 패키지 생성
        create-package:
          stage: package
          script:
            - echo "Creating deployable package..."
            - tar -czf app.tar.gz dist/
          artifacts:
            paths:
              - app.tar.gz  # 배포 파일 생성
        
        # 배포 단계
        deploy-staging:
          stage: deploy
          script:
            - echo "Deploying to staging..."
            - scp app.tar.gz user@staging-server:/var/www/app/
          environment:
            name: staging
            url: https://staging.example.com
        
        deploy-production:
          stage: deploy
          script:
            - echo "Deploying to production..."
            - scp app.tar.gz user@production-server:/var/www/app/
          environment:
            name: production
            url: https://example.com
          rules:
            - if: $CI_COMMIT_BRANCH == "main"  # 메인 브랜치에서만 실행
            - when: manual  # 수동 승인 필요
        
        # 정리 단계
        cleanup-temp-files:
          stage: cleanup
          script:
            - echo "Cleaning up temporary files..."
            - rm -rf dist/ app.tar.gz
          when: always  # 성공, 실패 관계없이 실행
        
        ```
        
    - 예시 2 - stages 내 직렬실행은 needs 사용
        
        ```yaml
        stages:
          - test
        
        test-job-1:
          stage: test
          script:
            - echo "Running test-job-1"
        
        test-job-2:
          stage: test
          script:
            - echo "Running test-job-2"
            - sleep 5  # Simulating a delay
          **needs: [test-job-1]**  
        ```
        
    - 실행규칙
        - 병렬 실행
            - 동일한 `stage`에 있는 작업들은 병렬로 실행
        - 순차 실행
            - 이전 `stage`의 작업이 모두 성공해야 다음 `stage`가 실행
            - 하나의 작업이라도 실패하면, 이후 단계는 실행되지 않으며 파이프라인은 실패로 표시
        - 특별 단계
            - `.pre`
                - 파이프라인의 모든 작업 전에 실행.
            - `.post`
                - 파이프라인의 모든 작업 후에 실행.
            - 주의
                - `.pre`와 `.post`만 포함된 파이프라인은 실행되지 않음
- **workflow**
    - 설명
        - 파이프라인의 실행 조건을 제어
        - 파이프라인 이름, 실행 조건 및 자동 취소 규칙 등을 정의
    - `workflow` 사용 유형
        - `workflow:auto_cancel:on_new_commit`
            - 설명
                - **새 커밋이 푸시될 때** 기존 파이프라인의 실행을 자동으로 취소하는 동작을 제어
            - 입력값
                - `conservative`
                    - 파이프라인이 취소되지만, `interruptible: false`가 지정된 작업이 시작되기 전에만 취소됨 (기본값)
                - `interruptible`
                    - `interruptible: true`로 설정된 Job 이 실행되기 전에만 취소됨
                    - `interruptible: true`로 설정된 Job 이 실행 중인 경우 그대로 실행됨
                - `none`
                    - 아무 작업도 취소되지 않음
            - 예시
                
                ```yaml
                
                # 첫 번째 커밋을 푸시하고 job1, job2, job3가 실행 중이라고 가정.
                # 새로운 커밋이 푸시되면:
                #    job1과 job3는 interruptible: true이지만, 이미 실행 중인 경우 취소되지 않음.
                #    job2는 interruptible: false이므로 취소되지 않음.
                workflow:
                  auto_cancel:
                    on_new_commit: conservative
                job1:
                  interruptible: true
                  script: sleep 60
                job2:
                  interruptible: false
                  script: sleep 60
                job3:
                  interruptible: false
                  script: slep 60
                ```
                
                ```yaml
                # 첫 번째 커밋을 푸시하고 job1, job2, job3가 실행 중이라고 가정.
                # 새로운 커밋이 푸시되면:
                #    job1과 job3는 interruptible: true이므로 취소됨.
                #    job2는 interruptible: false이므로 취소되지 않음.
                workflow:
                  auto_cancel:
                    on_new_commit: interruptible
                job1:
                  interruptible: true
                  script: sleep 60
                job2:
                  interruptible: false
                  script: sleep 60
                job3:
                  interruptible: false
                  script: sleep 60
                ```
                
        - `workflow:auto_cancel:on_job_failure`
            - 설명
                - 파이프라인에서 **작업 실패 시** 실행 중인 다른 작업을 자동으로 취소하는 동작을 제어
            - 입력값
                - `all`: 하나의 작업이 실패하면, 해당 파이프라인에 있는 모든 작업이 취소됨.
                - `none`: 아무 작업도 취소되지 않음.
            - 예시
                
                ```yaml
                # job2가 실패하면, job1이 실행 중이라면 취소되고, job3는 시작되지 않음
                stages: [stage_a, stage_b]
                
                workflow:
                  auto_cancel:
                    on_job_failure: all
                
                job1:
                  stage: stage_a
                  script: sleep 60
                
                job2:
                  stage: stage_a
                  script:
                    - sleep 30
                    - exit 1  # 실패 발생
                
                job3:
                  stage: stage_b
                  script:
                    - sleep 30
                ```
                
        - `workflow:rules`
            - 설명
                - 파이프라인이 실행될지 말지를 **조건부로 결정**하는 규칙을 설정하는 데 사용
                - 조건이 만족되면 파이프라인이 실행되며, 만족되지 않으면 파이프라인이 실행되지 않음
            - 입력 값
                - `if`: 특정 조건에 맞을 때
                - `changes`: 파일 변경 사항을 기반으로 파이프라인 실행 여부 결정
                - `exists`: 특정 파일이 존재하는지 여부에 따라 파이프라인 실행 여부 결정
                - `when`: `always` 또는 `never`
            - 예시
                
                ```yaml
                workflow:
                  rules:
                    - if: $CI_COMMIT_TITLE =~ /-draft$/
                      when: never  # 커밋 메시지가 -draft로 끝나면 파이프라인 실행 안 함.
                    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
                
                ```
                
        - `workflow:rules:variables`
            - 설명
                - 조건이 일치할 때 **특정 변수를 설정**하여 파이프라인 및 작업에서 사용할 수 있게 만듬.
                - 변수는 파이프라인 내 모든 작업에서 사용할 수 있으며, 동일한 변수가 상위 또는 하위 파이프라인에서 사용되면 **상위 값이 우선** 적용
            - 예시
                
                ```yaml
                # 본 브랜치(예: main 또는 master)에서 파이프라인이 실행될 때:
                #    job1의 DEPLOY_VARIABLE은 "job1-deploy-production"로 설정
                #    job2의 DEPLOY_VARIABLE은 "deploy-production"으로 설
                # 브랜치가 feature(예: feature/my-new-feature)일 경우:
                #    job1의 DEPLOY_VARIABLE은 "job1-default-deploy"로 설정
                #    job1의 IS_A_FEATURE는 true로 설정됩니다.
                #    job2의 DEPLOY_VARIABLE은 "default-deploy"로 설정
                #    job2의 IS_A_FEATURE도 true로 설정됩니다.
                # 기본 브랜치나 feature 브랜치 외의 다른 브랜치(예: hotfix나 release 등)일 경우:
                #    job1의 DEPLOY_VARIABLE은 "job1-default-deploy"로 설정
                #    job2의 DEPLOY_VARIABLE은 "default-deploy"로 설정
                # 
                variables:
                  DEPLOY_VARIABLE: "default-deploy"
                
                workflow:
                  rules:
                    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
                      variables:
                        DEPLOY_VARIABLE: "deploy-production"
                    - if: $CI_COMMIT_BRANCH =~ /feature/
                      variables:
                        IS_A_FEATURE: "true"
                    - if: $CI_COMMIT_BRANCH
                
                job1:
                  variables:
                    DEPLOY_VARIABLE: "job1-default-deploy"
                  rules:
                    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
                      variables:
                        DEPLOY_VARIABLE: "job1-deploy-production"
                    - when: on_success
                  script:
                    - echo "Run script with $DEPLOY_VARIABLE as an argument"
                    - echo "Run another script if $IS_A_FEATURE exists"
                
                job2:
                  script:
                    - echo "Run script with $DEPLOY_VARIABLE as an argument"
                    - echo "Run another script if $IS_A_FEATURE exists"
                ```
                
        - `workflow:name`
            - 설명
                - **파이프라인에 이름을 지정**
                - 이름은 **CI/CD 변수**를 사용하여 동적으로 설정할 수도 있음
            - 예시
                
                ```yaml
                # 단순 파이프라인 이름 설정
                workflow:
                  name: 'Pipeline for branch: $CI_COMMIT_BRANCH'
                
                ---
                # 조건에 맞는 경우 파이프라인 이름이 동적으로 설정
                variables:
                  PROJECT1_PIPELINE_NAME: 'Default pipeline name'
                
                workflow:
                  name: '$PROJECT1_PIPELINE_NAME'
                  rules:
                    - if: '$CI_MERGE_REQUEST_LABELS =~ /pipeline:run-in-ruby3/'
                      variables:
                        PROJECT1_PIPELINE_NAME: 'Ruby 3 pipeline'
                    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
                      variables:
                        PROJECT1_PIPELINE_NAME: 'MR pipeline: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME'
                    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH  # 기본 브랜치 파이프라인을 위한 이름 설정
                
                ```
                
        - `workflow:rules:auto_cancel`
            - 설명
                - **`auto_cancel` 규칙**을 조건에 맞게 설정
            - 예시
                
                ```yaml
                
                # 보호된 브랜치에서는 on_new_commit과 on_job_failure 규칙이 none으로 설정되며, 
                # 다른 브랜치에서는 기본적으로 interruptible과 all로 설정 
                
                # 비보호된 브랜치에서 새 커밋이 푸시될 때:
                #   test-job1과 test-job2가 모두 실행 중일 때, 새 커밋이 푸시되면:
                #     test-job1은 계속 실행(중단 가능한 작업이므로, 중단하지 않음).
                #     test-job2는 취소됩니다(중단 불가능한 작업이므로, 중단되지 않음).
                # 보호된 브랜치에서 새 커밋이 푸시될 때:
                #  test-job1과 test-job2가 모두 실행 중일 때, 새 커밋이 푸시되면:
                #    test-job1과 test-job2 모두 계속 실행
                
                # 보호된 브랜치에서는 on_new_commit: none과 on_job_failure: none이 설정되었기 때문에, 
                # 새 커밋이 푸시되더라도 중단이나 취소가 이루어지지 않음
                
                workflow:
                  auto_cancel:
                    on_new_commit: interruptible
                    on_job_failure: all
                  rules:
                    - if: $CI_COMMIT_REF_PROTECTED == 'true'
                      auto_cancel:
                        on_new_commit: none
                        on_job_failure: none
                    - when: always
                
                test-job1:
                  script: sleep 10
                  interruptible: false
                
                test-job2:
                  script: sleep 10
                  interruptible: true
                ```


### Header keywords

- **Spec Keyword**
    - 기본구문
        - 설명
            - `spec` 키워드는 YAML 파일의 헤더 섹션에 정의되어야 하며, 파이프라인의 설정에서 입력 파라미터를 정의하고 이를 포함한 구성 파일의 동작을 제어하는 데 사용
        - 기본 구조
            
            ```yaml
            spec:
              inputs:
                input_name:
                  type: <input_type>
                  default: <default_value>    # (옵션) 기본값 설정
                  options:                    # (옵션) 허용 가능한 값 목록
                    - option1
                    - option2
                  regex: <정규 표현식>        # (옵션) 입력 값에 대한 정규 표현식
                  description: <설명>         # (옵션) 입력에 대한 설명
            ---
            # 나머지 파이프라인 구성
            ```
            
        - 예시
            
            ```yaml
            # inputs에서 정의된 job-stage와 environment는 실제 파이프라인에서 값을 받아 사용
            # inputs는 기본적으로 문자열 값을 요구하며, 다른 데이터 유형을 지정하려면 type을 사용해야 함
            spec:
              inputs:
                environment:
                job-stage:
            ---
            scan-website:
              stage: $[[ inputs.job-stage ]]
              script: ./scan-website $[[ inputs.environment ]]
            
            ```
            
            ```yaml
            # 종합 예시
            spec:
              inputs:
                environment:
                  options:
                    - development
                    - staging
                    - production
                version:
                  regex: ^v\d\.\d+(\.\d+)$
                  default: 'v1.0'
                deploy_flag:
                  description: 'Flag to enable deployment'
                  type: boolean
                  default: false
            ---
            deploy-job:
              stage: deploy
              script:
                - echo "Deploying to $[[ inputs.environment ]]"
                - echo "Version: $[[ inputs.version ]]"
                - |
                  if [[ $[[ inputs.deploy_flag ]] == true ]]; then
                    echo "Deploying application"
                  else
                    echo "Skipping deployment"
                  fi
            ```
            
    - Spec 유형
        - `spec:inputs:default`
            - 설명
                - inputs 값이 필수적일 때, 기본값을 설정
                - inputs 값이 제공되지 않으면 이 기본값이 사용
            - 예시
                
                ```yaml
                
                spec:
                  inputs:
                    website:
                    user:
                      default: 'test-user'
                    flags:
                      default: ''
                ---
                # 파이프라인 구성
                ```
                
                - `website`는 필수 입력 값이며, `user`는 기본값 `'test-user'`가 설정되어 있어 사용자가 값을 제공하지 않으면 `'test-user'`가 사용
                - `flags`는 기본값이 빈 문자열로 설정되어 있으므로 값이 제공되지 않으면 빈 문자열이 사용
        - `spec:inputs:description`
            - 설명
                - inputs 값에 대한 설명을 제공
                - 이 설명은 파이프라인 실행에 영향을 미치지 않으며, 주로 파일을 읽는 사람을 위한 설명
            - 예시
                
                ```yaml
                spec:
                  inputs:
                    flags:
                      description: 'This input controls the flags for the script execution.'
                ---
                # 파이프라인 구성
                ```
                
        - `spec:inputs:options`
            - 설명
                - inputs 값에 대해 허용되는 값을 제한
                - 최대 50개의 옵션을 정의
            - 예시
                
                ```yaml
                spec:
                  inputs:
                    environment:
                      options:
                        - development
                        - staging
                        - production
                ---
                # 파이프라인 구성
                ```
                
        - `spec:inputs:regex`
            - 설명
                - 입력 값이 정규 표현식을 만족해야 하는 경우에 사용
                - 정규 표현식은 RE2 구문을 사용
            - 예시
                
                ```yaml
                # version은 v1.0, v1.2.3 등과 같은 형식만 유효하며, v1.A.B와 같은 형식은 유효하지 않음
                spec:
                  inputs:
                    version:
                      regex: ^v\d\.\d+(\.\d+)$
                ---
                # 파이프라인 구성
                ```
                
        - `spec:inputs:type`
            - 설명
                - 기본적으로 입력 값은 문자열을 요구하지만, 다른 타입을 지정가능
                - 예를 들어 배열, 숫자, 불리언 값 등을 사용할 수 있음
            - 예시
                
                ```yaml
                # website는 문자열, port는 숫자, available은 불리언, array_input은 배열 형식의 값을 요구
                spec:
                  inputs:
                    job_name:
                    website:
                      type: string
                    port:
                      type: number
                    available:
                      type: boolean
                    array_input:
                      type: array
                ---
                # 파이프라인 구성
                ```

### Job Keywords

- 스크립트 실행 및 실행 순서
    - **after_script**
        - 설명
            - GitLab CI/CD 파이프라인에서 작업(job) 후에 실행될 명령어들을 정의하는 데 사용
        - 주요 특징
            - 다른 셸에서 실행
                - `after_script`는 별도의 셸에서 실행
                - 따라서 `before_script`와 `script`에서 설정한 변수나 변경 사항은 접근할 수 없음
                - 현재 작업 디렉터리가 기본 값으로 돌아가며, 실행 중인 작업과 독립된 환경에서 실행
            - 타임아웃
                - `after_script`는 GitLab Runner 16.4 이상에서 5분의 기본 타임아웃을 가짐
                    - 이 타임아웃은 `RUNNER_AFTER_SCRIPT_TIMEOUT` 변수를 사용해 조정가능
                - `after_script` 명령이 실패하거나 타임아웃되더라도 작업의 종료 코드에는 영향을 미치지 않음
                    - `script`가 성공적으로 완료되면 `after_script`가 실패하더라도 전체 작업은 성공으로 간주
            - 작업 취소와 관련된 이슈
                - 작업이 취소되면 CI/CD 작업 토큰이 즉시 무효화
                - 이로 인해 `after_script`에서 작업 토큰을 사용할 때 문제가 발생할 수 있음
            - 시간 초과 시 `after_script` 실행 안 함
                - 작업이 시간 초과되면 `after_script`는 실행되지 않음
            - 결과 코드와 무관
                - `after_script`의 실행 결과가 작업의 종료 코드에 영향을 미치지 않음
                - 즉, `after_script`가 실패해도 전체 작업이 성공으로 처리
        - 실행 상황
            - `script` 섹션이나 `before_script` 섹션이 끝난 후에 실행되며, 다음과 같은 상황에서도 실행
                - 작업이 취소될 때
                    - `before_script`나 `script` 섹션이 실행 중에 작업이 취소되면 `after_script`는 여전히 실행
                - 작업 실패 시
                    - 작업의 실제 실행 흐름과 관련이 있는 실패만을 고려하기 때문에 조건에 따라 실행 여부가 다름
                    - script_failure 로 실패한 경우 - 정상적으로 실행
                        - `script_failure`는 `script` 섹션에서 실제 명령이 실패했을 때 발생
                        - 예를 들어, `script` 명령어 중 하나가 비정상적으로 종료되거나 오류를 반환하는 경우
                        - 이 실패 유형에서는 `after_script`가 정상적으로 실행
                        - 즉, `script`가 실패하면 `after_script`가 실행되어 후속 작업을 처리
                    - job_failure -  실행되지 않음
                        - `job_failure`는 전체 작업이 실패할 때 발생하는 실패 유형
                        - 이는 `script`의 실패 외에도 다양한 이유로 작업이 실패할 수 있음
                            - 예를 들어, 작업이 실행 중에 외부 서비스와의 연결이 끊어졌거나, 권한 문제가 발생한 경우에도 `job_failure`가 발생
                        - `job_failure`에 해당하는 실패는 `after_script`의 실행을 트리거하지 않으며, 후속 작업을 실행할 수 없음
                    - manual_failure - 실행되지 않음
                        - 사용자가 수동으로 실패를 지정한 경우
                            - 예를 들어, GitLab에서는 사용자가 `manual` 단계를 추가하여 `script`가 실행되지 않도록 설정
                        - 실패가 수동으로 지정된 상태에서 `after_script`는 실행되지 않음
        - 예시
            
            ```yaml
            # 기본
            job:
              script:
                - echo "This is the script section"
              after_script: # 실행됨
                - echo "This runs after the script section finishes"
            
            # 작업 취소 
            job:
              script:
                - echo "Running the main script"
                - sleep 30  # 30초 동안 대기
              after_script: # 30초 전에 작업 취소해도 실행됨
                - echo "Running after_script, even if the job is canceled"
            
            # 실패시 
            job:
              script:
                - exit 1  # 고의로 실패시킴
              after_script:
                - echo "This runs after script fails"
                
            # 타임아웃 
            job:
              script:
                - echo "Running script with long process"
                - sleep 1800  # 30분 동안 대기
              after_script: # 실행되지 않음 
                - echo "This will not run if the job times out"
            
            # after_script에서 환경 변수 사용
            job:
              script:
                - echo "Main script is running"
                - export VAR="Value from script"
              after_script:
                - echo "After script: $VAR"  # `VAR`는 after_script에서 접근할 수 없음
            
            # 작업 취소 시 after_script 건너뛰기
            job:
              script:
                - echo "This job will run and may be canceled"
              after_script:
                - echo "This will not run if the job is canceled"
              interruptible: true  # 이 작업은 취소 가능
            ```
            
    - **before_script**
        - 설명
            - `before_script`는 각 작업(Job)이 실행되기 전에, 실제 `script` 명령이 실행되기 직전에 실행
            - 주로 여러 작업에서 공통적으로 실행해야 할 명령들을 설정할 때 유용
                - 예를 들어, 환경설정, 종속성 설치 등을 `before_script`에서 실행
            - `before_script`에서도 CI/CD 변수를 사용 가능
        - 위치
            - `before_script`는 `job` 내에서 또는 `default` 섹션에서 사용될 수 있음
        - 실행 시점
            - `before_script`는 각 작업의 `script` 명령보다 먼저 실행
            - 다만 `artifacts`가 복원된 이후 실행
        - 구성
            - `before_script`는 명령어 배열을 포함하며, 각 명령어는 쉘에서 순차적으로 실행
        - 특징
            - `before_script`의 명령어들은 **`script` 명령어와 결합되어** 하나의 셸에서 실행
            - 여러 작업에서 공통적으로 사용할 명령어들을 `default` 섹션에 설정하면 모든 작업에서 동일하게 실행
        - 예시
            
            ```yaml
            job:
              before_script:
                - echo "This runs before the main script commands."
                - echo "It is executed after artifacts are restored."
              script:
                - echo "This runs after the before_script."
            
            ---
            # default 섹션에서의 사용시 모든 job 에서 실행됨
            default:
              before_script:
                - echo "This will run before every job's script."
            
            job1:
              script:
                - echo "Job 1's specific commands"
            
            job2:
              script:
                - echo "Job 2's specific commands"
            
            ```
            
    - **script**
        - 설명
            - GitLab CI/CD에서 runner가 실행할 명령어를 지정하는 데 사용되는 키워드
            - 이 키워드는 모든 job에서 필수적이며, 각 job 내에서 실행할 쉘 명령어를 정의
        - 주요 개념
            - `script`: GitLab CI/CD에서 job 내에서 실행할 명령어를 지정하는 키워드
                - 모든 job에서 필요하며, job이 `trigger`인 경우를 제외하고 반드시 사용해야 함
                - `script`는 단일 명령어, 여러 줄의 명령어, YAML 앵커 등을 포함할 수 있음
                - GitLab CI/CD에서는 CI/CD 변수도 `script` 내에서 사용 가능
        - 가능 입력 값
            - 단일 명령어
                - 하나의 명령어를 실행하려면 문자열로 명령어를 작성
                    
                    ```yaml
                    script: "bundle exec rspec"
                    ```
                    
            - 여러 줄 명령어
                - 여러 명령어를 순차적으로 실행하려면 명령어들을 배열로 작성
                    
                    ```yaml
                    script:
                      - uname -a
                      - bundle exec rspec
                    ```
                    
            - YAML 앵커 사용
                - YAML 앵커를 사용해 명령어들을 재사용할 수 있음
                - 예를 들어, 동일한 명령어 조합을 여러 job에서 재사용할 수 있음
                    
                    ```yaml
                    stages:
                      - build
                      - test
                    
                    # 공통 명령어를 앵커로 정의
                    common_script: &common_script
                      - echo "This is a common script"
                      - bundle install
                      - bundle exec rspec
                    
                    build_job:
                      stage: build
                      script:
                        *common_script  # 앵커된 공통 명령어 사용
                    
                    test_job:
                      stage: test
                      script:
                        *common_script  # 동일한 앵커된 공통 명령어 재사용
                    ```
                    
            - CI/CD 변수
                - GitLab CI/CD에서는 **CI/CD 변수**를 `script` 내에서 사용할 수 있음
                - 변수는 `$CI_COMMIT_REF_NAME`과 같은 방식으로 참조
                    
                    ```yaml
                    script:
                      - echo "Deploying to $CI_ENVIRONMENT_NAME"
                    ```
                    
            
    - **dependency**
        - 설명
            - GitLab CI/CD에서 특정 작업으로부터 아티팩트를 가져오는 작업을 정의할 때 사용
            - 이전 단계에 정의된 작업들로부터 아티팩트를 가져오거나, 아티팩트를 전혀 다운로드하지 않도록 설정할 수 있음
            - `dependencies`를 정의하지 않으면, 기본적으로 이전 단계의 모든 작업에서 아티팩트를 가져옴
        - 주요사항
            - 기본 동작
                - `dependencies`가 정의되지 않은 경우, 현재 작업은 이전 단계에 정의된 모든 작업에서 아티팩트를 다운로드
            - 명시적 정의
                - `dependencies`를 사용하면 특정 작업에서 아티팩트를 다운로드할 작업을 명시적으로 설정
                - 작업이 특정 아티팩트만 필요할 경우, 이를 명확하게 정의
            - 빈 배열 설정
                - `dependencies`를 빈 배열(`[]`)로 설정하면, 해당 작업은 이전 단계의 작업에서 아티팩트를 다운로드하지 않음
        - 예시
            
            ```yaml
            
            # build osx와 build linux는 각각의 빌드 작업 후 binaries/ 디렉토리에 아티팩트를 생성
            build osx:
              stage: build
              script: make build:osx
              artifacts:
                paths:
                  - binaries/
            
            build linux:
              stage: build
              script: make build:linux
              artifacts:
                paths:
                  - binaries/
            
            # test osx는 dependencies에 build osx를 지정하여, build osx 작업의 아티팩트를 다운로드 
            test osx:
              stage: test
              script: make test:osx
              dependencies:
                - build osx
            
            # test linux는 dependencies에 build linux를 지정하여, build linux 작업의 아티팩트를 다운로드
            test linux:
              stage: test
              script: make test:linux
              dependencies:
                - build linux
            
            # deploy 작업은 이전 단계에서 생성된 모든 아티팩트를 다운로드
            deploy:
              stage: deploy
              script: make deploy
              environment: production
            ```
            
    - **inherit**
        - 설명
            - `inherit` 키워드는 GitLab CI/CD에서  상속(Inheritance)을 제어하는 데 사용
            - 이를 통해 기본 설정(default)이나 전역 변수(global variables)의 상속을 선택적으로 관리할 수 있음
            - 즉, 작업(Job)이 상속받을 설정이나 변수를 조정할 수 있게 함
        - inherit 키워드
            - `inherit:default`
                - 설명
                    - `inherit:default`는 기본 설정(default keywords)의 상속 여부를 제어
                - 사용법
                    - `true` (기본값): 모든 기본 설정을 상속
                    - `false`: 기본 설정을 상속받지 않음
                    - 기본 설정 목록을 지정하여 특정 설정만 상속할 수 있음
                - 예시
                    
                    ```yaml
                    default:
                      retry: 2
                      image: ruby:3.0
                      interruptible: true
                    
                    job1:
                      script: echo "This job does not inherit any default keywords."
                      inherit:
                        default: false  # 모든 기본 설정을 상속하지 않음
                    
                    job2:
                      script: echo "This job inherits only 'retry' and 'image', not 'interruptible'."
                      inherit:
                        default:
                          - retry
                          - image  # 'retry'와 'image'만 상속
                    ```
                    
            - `inherit:variables`
                - 설명
                    - `inherit:variables`는 전역 변수(global variables)의 상속 여부를 제어
                - 사용법
                    - `true` (기본값): 모든 전역 변수를 상속
                    - `false`: 전역 변수를 상속받지 않음
                    - 특정 변수 목록을 지정하여 원하는 변수만 상속할 수 있음
                - 예시
                    
                    ```yaml
                    variables:
                      VARIABLE1: "This is variable 1"
                      VARIABLE2: "This is variable 2"
                      VARIABLE3: "This is variable 3"
                    
                    job1:
                      script: echo "This job does not inherit any global variables."
                      inherit:
                        variables: false  # 모든 전역 변수를 상속하지 않음
                    
                    job2:
                      script: echo "This job inherits only 'VARIABLE1' and 'VARIABLE2', not 'VARIABLE3'."
                      inherit:
                        variables:
                          - VARIABLE1
                          - VARIABLE2  # 'VARIABLE1'과 'VARIABLE2'만 상속
                    ```
                    
    - **retry**
        - `retry`
            - 설명
                - **GitLab CI/CD**에서 작업이 실패했을 때 자동으로 재시도할 횟수를 설정하는 데 사용
                - 기본값  `retry`는 0으로 설정됨
                - `retry`는 기본적으로 **모든 실패 유형**에 대해 재시도를 시도합니다. 예를 들어, 네트워크 문제나 임시적인 오류로 인한 실패가 있을 때 자동으로 작업을 재시도하려면 `retry`를 설정하면 됨
                - 특정 조건에서만 재시도하고 싶다면 `retry:when` 또는 `retry:exit_codes`와 같은 추가 설정을 사용
                - 최대 2번까지만 가능
            - 예시
                
                ```yaml
                # 기본적인 retry 설정
                test:
                  script: rspec
                  retry: 2
                
                # retry와 when, exit_codes 사용
                # 러너 시스템 오류인 경우 이때 exit code 가 137인 경우만 재시도
                test_advanced:
                  script:
                    - echo "Run a script that results in exit code 137."
                    - exit 137
                  retry:
                    max: 2
                    when: runner_system_failure
                    exit_codes: 137
                ```
                
        - `retry:when`
            - 설명
                - GitLab CI/CD에서 작업이 실패했을 때 특정 실패 케이스에 대해서만 재시도할 수 있도록 하는 기능
            - 가능한 입력값
                - `always`: 모든 실패에 대해 재시도 (기본값)
                - `unknown_failure`: 실패 원인이 불명확한 경우
                - `script_failure`: 스크립트가 실패한 경우 (예: 스크립트 실행 실패, Docker 이미지 다운로드 실패 등)
                - `api_failure`: API 실패로 인해 작업이 실패한 경우
                - `stuck_or_timeout_failure`: 작업이 멈추거나 타임아웃이 발생한 경우
                - `runner_system_failure`: 러너 시스템 오류 발생 (예: 러너 설정 실패 등)
                - `runner_unsupported`: 러너가 지원되지 않는 경우
                - `stale_schedule`: 지연된 작업을 실행할 수 없는 경우
                - `job_execution_timeout`: 작업이 설정된 최대 실행 시간 초과 시
                - `archived_failure`: 작업이 아카이브되어 실행할 수 없는 경우
                - `unmet_prerequisites`: 작업이 선행 작업을 완료하지 못한 경우
                - `scheduler_failure`: 스케줄러가 작업을 실행할 러너에 할당하지 못한 경우
                - `data_integrity_failure`: 데이터 무결성 문제로 실패한 경우
            - 예시
                
                ```yaml
                test:
                  script: rspec
                  retry:
                    max: 2
                    when:
                      - runner_system_failure
                      - stuck_or_timeout_failure
                ```
                
        - `retry:exit_codes`
            - 설명
                - 특정 종료 코드에 대해 재시도를 설정하는 기능
            - 가능한 입력값
                - 단일 종료 코드: 숫자 하나로 지정 (예: `137`)
                - 종료 코드 배열: 여러 종료 코드 지정 가능 (예: `[137, 255]`)
            - 예시
                
                ```yaml
                test_job_2:
                  script:
                    - echo "Run a script that results in exit code 137. This job will be retried."
                    - exit 137
                  retry:
                    max: 1
                    exit_codes:
                      - 255
                      - 137
                ```
                
    - **run**
        - 설명
            - `run` 키워드는 GitLab CI/CD에서 한 개의 job 내에서 여러 단계를 정의할 수 있게 해주는 기능
            - 각 단계는 스크립트(shell 명령어) 또는 미리 정의된 단계(GitLab CI 도구)를 실행할 수 있음
            - 이 기능은 job을 세분화하고, 자주 사용되는 공통 작업을 미리 정의된 단계로 처리할 때 유용
            - 이 기능은 job을 세분화하고, 외부 도구를 활용하거나 환경 변수와 입력값을 동적으로 설정하는 데 유용
        - 주요 개념
            - `run`: job 내에서 실행할 일련의 단계를 정의. 각 단계는 하나의 작업을 의미
            - `name`: 각 단계의 이름을 지정
            - `script`: 실행할 쉘 명령어를 설정. 문자열 형태로 작성할 수 있고, 여러 줄의 스크립트는 YAML의 블록 스칼라 구문을 사용하여 정의할 수 있음
            - `step`: 미리 정의된 외부 GitLab CI 도구를 실행하는 경우 사용
            - `env`: 각 단계에 대해 지정할 수 있는 환경 변수
            - `inputs`: 미리 정의된 단계에서 사용할 수 있는 입력 파라미터
        - 입력값의 구성
            - 배열로 여러 단계를 정의할 수 있으며 각 단계는 해시 형태로 정의되며, 각 해시에는 `name`, `script` 또는 `step`(둘 중 하나만 사용 가능) 항목이 포함
            - `script`: 쉘 명령어를 실행하는 단계
            - `step`: 미리 정의된 GitLab CI 도구를 실행하는 단계
            - `env`: 단계에 사용할 환경 변수들
        - 추가 세부사항
            - 스크립트와 단계 선택
                - 하나의 단계는 `script` 또는 `step` 키만 포함할 수 있으며, 둘을 동시에 사용할 수 없음
            - 기존 `script`와의 충돌
                - `run` 설정은 기존의 `script` 키워드와 함께 사용할 수 없음
                - `run`을 사용하면 `script`는 더 이상 사용되지 않으며, 대신 여러 단계를 정의할 수 있음
            - 멀티라인 스크립트
                - 여러 줄의 스크립트는 YAML의 블록 스칼라 구문을 사용하여 정의할 수 있음
        - 예시
            
            ```yaml
            
            # hello_steps 단계는 간단한 쉘 명령어 echo "hello from step1"을 실행 
            # bye_steps 단계는 미리 정의된 단계 gitlab.com/gitlab-org/ci-cd/runner-tools/echo-step@main을 실행
            # 이 단계는 입력 파라미터(echo: 'bye steps!')와 환경 변수(var1: 'value 1')를 사용 
            job:
              run:
                - name: 'hello_steps'
                  script: 'echo "hello from step1"'
                - name: 'bye_steps'
                  step: gitlab.com/gitlab-org/ci-cd/runner-tools/echo-step@main
                  inputs:
                    echo: 'bye steps!'
                  env:
                    var1: 'value 1'
            ```
            
    - **timeout**
        - 설명
            - 특정 작업에 대해 제한 시간을 설정하는 데 사용
            - 설정된 시간이 지나면, 해당 작업은 실패로 처리
            - 이 키워드는 작업 내에서만 사용하거나 default 섹션에 포함하여 기본값을 설정할 수 있음
        - 사용 방법
            - 작업이 지정된 시간 내에 완료되지 않으면 자동으로 실패 처리
            - 프로젝트 수준의 타임아웃보다 길게 설정할 수 있으나, 러너의 타임아웃을 초과할 수는 없음
        - 가능한 입력
            - `3600 seconds` (초 단위)
            - `60 minutes` (분 단위)
            - `one hour` (시간 단위)
            - `3 hours 30 minutes` (혼합된 시간 형태)
        - 세부 사항
            - `timeout`은 작업별로 설정할 수 있음
            - 프로젝트 전체의 타임아웃보다 긴 값을 설정할 수 있으나, 러너의 설정된 타임아웃 값보다 더 길게 설정할 수는 없음
        - 예시
            
            ```yaml
            build:
              script: build.sh
              timeout: 3 hours 30 minutes
            
            test:
              script: rspec
              timeout: 3h 30m
            ```
            
- 실행 조건 및 제어
    - **allow_failure**
        - 설명
            - CI/CD 파이프라인에서 **작업이 실패했을 때 후속 작업을 어떻게 처리할지** 결정하는 옵션
            - 이 키워드를 사용하면, 특정 작업이 실패하더라도 파이프라인이 계속 실행되도록 설정하거나, 실패한 경우 파이프라인 실행을 멈추도록 설정 가능
        - 기본 동작
            - `allow_failure: true`
                - 작업이 실패하더라도, 후속 작업이 계속 실행
                - 실패한 작업은 **주황색 경고**로 표시되지만, 파이프라인은 여전히 **성공으로 간주**되며 해당 커밋은 **성공**으로 표시
            - `allow_failure: false`
                - 작업이 실패하면 후속 작업이 실행되지 않고, 파이프라인 전체가 실패로 간주
        - 기본 값
            - `when: manual` 시 → 기본값은 `allow_failure: true`
                - 사용자가 수동으로 작업을 시작하기 전까지 파이프라인이 실패 상태로 간주되지 않도록 하려는 의도
            - `when: manual`이 `rules` 내에서 사용 시 →  기본값은 `allow_failure: false`
            - 그외 모든 경우 → 기본값은 `allow_failure: false`
                - 작업이 실패하면 파이프라인이 실패로 간주
        - 예시
            
            ```yaml
            
            # job1과 job2는 병렬로 실행 
            #   job1은 test 단계에서 실행되고, job2는 동일한 test 단계에서 병렬로 실행 
            #   job2가 실패하더라도 파이프라인은 계속 진행됩니다 (allow_failure: true).
            # job3과 job4는 deploy 단계에서 병렬로 실행 
            #   둘 다 when: manual로 설정되어 있어 수동으로 트리거될 때 실행
            #   job3이나 job4가 실패하더라도 (allow_failure: true로 설정되어 있음), 파이프라인은 성공으로 표시
            job1:
              stage: test
              script:
                - echo "Execute script for job1" #  allow_failure: false 적용됨
              
            job2:
              stage: test
              script:
                - echo "Execute script for job2"
              allow_failure: true  # 실패해도 파이프라인이 계속 진행됨
            
            job3:
              stage: deploy
              script:
                - echo "Deploy to staging"
              when: manual  # 수동으로 실행해야 하는 작업 , allow_failure: true 적용됨
            
            job4:
              stage: deploy
              script:
                - echo "Deploy to production"
              allow_failure: true  # 실패해도 파이프라인 성공으로 간주됨
              when: manual  # 수동으로 실행해야 하는 작업
            
            ```
            
    - **interruptible**
        - 설명
            - `interruptible` 키워드는 GitLab CI/CD에서 중복 파이프라인 자동 취소(Auto-cancel redundant pipelines) 기능을 제어하는 데 사용
                - 자동 취소 기능은 GitLab에서 동일한 커밋 참조에 대해 여러 파이프라인이 실행될 때, 이전 파이프라인에서 실행 중인 작업들을 새로운 파이프라인이 시작되면 자동으로 취소하도록 설정
            - 이 기능은 같은 참조(ref)에 대해 새로 시작된 파이프라인이 이전 파이프라인을 완료하기 전에 실행 중인 작업을 취소할 수 있도록 함
            - 새로운 커밋에 대한 파이프라인이 시작되면, 이전 파이프라인에서 실행 중인 작업들이 중복된 작업이라 판단되어 자동으로 취소될 수 있음
            - `interruptible: true`로 설정된 작업은 새로운 파이프라인이 시작되면 취소될 수 있음
        - `interruptible` 사용법
            - `true`: 해당 작업은 새로운 커밋에 대해 실행되는 파이프라인이 시작되면 취소될 수 있음
            - `false` (기본값): 해당 작업은 자동으로 취소되지 않음
        - 주의사항
            - 배포 작업에서는 `interruptible: true`를 설정하지 않는 것이 좋음
            - 배포 중에 작업이 취소되면 부분적인 배포가 발생할 수 있기 때문
        - 기본 동작
            - `workflow:auto_cancel:on_new_commit: conservative` 설정이 기본값
            - 이 설정은 새로운 파이프라인이 실행되었을 때 아직 시작되지 않은 작업은 항상 취소 가능(`interruptible: true`)으로 간주하고, 이미 실행 중인 작업은 `interruptible: true`가 설정된 경우에만 취소
            - 실행 중인 작업에 `interruptible: false`가 설정되면, 그 이후의 모든 작업은 취소되지 않으며, 파이프라인 자체가 중단되지 않음
        - 예시
            
            ```yaml
            # 기본 동작 (on_new_commit: conservative 설정)
            #   step-1과 step-3은 새 파이프라인이 시작되면 취소될 수 있음
            #   step-2는 interruptible: false로 설정되어 있기 때문에 실행이 시작되면 취소되지 않으며, 이로 인해 step-3은 취소되지 않음
            workflow:
              auto_cancel:
                on_new_commit: conservative  # 기본값
            
            stages:
              - stage1
              - stage2
              - stage3
            
            step-1:
              stage: stage1
              script:
                - echo "Can be canceled."
              interruptible: true  # 이 단계는 새 파이프라인 시작 시 취소 가능
            
            step-2:
              stage: stage2
              script:
                - echo "Can not be canceled."
              interruptible: false  # 이 단계는 취소되지 않음
            
            step-3:
              stage: stage3
              script:
                - echo "This step will not be canceled, even though it's set as interruptible."
              interruptible: true  # 그러나 step-2가 interruptible: false이므로 취소되지 않음
            
            ---
            # on_new_commit: interruptible 설정
            #   step-1과 step-3은 새 파이프라인이 시작되면 취소 
            #   step-2는 interruptible: false로 설정되어 있어, 새 파이프라인이 시작되더라도 취소되지 않음
            workflow:
              auto_cancel:
                on_new_commit: interruptible  # 새 커밋에 대해 모든 취소 가능
            
            stages:
              - stage1
              - stage2
              - stage3
            
            step-1:
              stage: stage1
              script:
                - echo "Can be canceled."
              interruptible: true  # 새 파이프라인 시작 시 취소 가능
            
            step-2:
              stage: stage2
              script:
                - echo "Can not be canceled."
              interruptible: false  # 취소 불가
            
            step-3:
              stage: stage3
              script:
                - echo "Can be canceled."
              interruptible: true  # 새 파이프라인 시작 시 취소 가능
            ```
            
    - **manual_confirmation**
        - 설명
            - `when: manual`이 설정된 수동 작업에 대해 사용자에게 확인 메시지를 표시하는 데 사용
        - 사용 방법
            - 이 키워드는 오직 수동 작업(when: manual)에만 사용
            - 입력 값으로 **확인 메시지**를 문자열로 지정
        - 예시
            
            ```yaml
            delete_job:
              stage: post-deployment
              script:
                - make delete
              when: manual
              manual_confirmation: 'Are you sure you want to delete this environment?'
            ```
            
    - **rules**
        - 설명
            - 요약
                - GitLab CI/CD에서 파이프라인에서 특정 작업이 실행될지 여부를 결정하는 데 사용되는 구성 요소
                - 여러 개의 규칙을 정의하여 작업의 실행 여부를 동적으로 설정할 수 있음
                - 각 규칙은 조건이 일치하는지 여부에 따라 작업을 포함하거나 제외
            - 동작 방식
                - 규칙의 평가
                    - `rules`는 파이프라인이 생성될 때 평가되며, 규칙은 순차적으로 처리
                    - 첫 번째로 일치하는 규칙이 발견되면, 그 규칙에 따라 작업이 포함되거나 제외됩
                    - 만약 규칙이 하나도 일치하지 않으면, 해당 작업은 파이프라인에 포함되지 않음
                - 규칙의 처리 우선순위
                    - 규칙이 여러 개 있을 경우, 평가가 순차적으로 이루어지며, 규칙을 통해 작업이 포함되거나 제외.
                    - 첫 번째로 일치한 규칙에 따라 처리되고, 이후의 규칙은 더 이상 확인되지 않음
            - 구성요소
                - `rules`는 배열 형식으로 여러 규칙을 나열할 수 있으며 각 규칙은 최소한 하나의 조건을 가져야 함
                    - `if`: 조건을 참/거짓으로 평가하여 작업을 실행할지 말지 결정
                    - `changes`: 파일 변경을 기준으로 작업 실행 여부를 결정
                    - `exists`: 특정 파일이나 디렉토리의 존재 여부를 확인하여 작업을 실행할지 말지 결정
                    - `when`: 작업이 실행되는 시점을 결정. 기본값은 `on_success`
            - 선택적인 추가 조건
                - `allow_failure`: 작업이 실패해도 파이프라인 전체가 실패하지 않도록 설정할 수 있음
                - `needs`: 다른 작업에 의존성이 있을 때 사용하며 이 조건을 사용하면 해당 작업이 다른 작업이 성공한 후에 실행
                - `variables`: 특정 CI/CD 변수가 존재할 때 작업이 실행되도록 설정
                - `interruptible`: 작업이 실행 중일 때 다른 파이프라인에 의해 중단될 수 있도록 설정
            - `when` 값
                - `when` 값은 작업이 실행되는 시점을 정의하며 기본값은 `on_success`
                    - `on_success`: 작업이 성공할 때 실행. 기본 동작
                    - `delayed`: 지연된 실행이 필요할 때 사용
                    - `always`: 작업이 실패해도 항상 실행
                    - `never`: 조건에 맞지 않으면 작업을 실행하지 않음
            - 예시
                
                ```yaml
                # 조건에 맞는 경우 작업을 실행하는 예시
                job:
                  script: echo "This job runs if the branch is 'main'"
                  rules:
                    - if: '$CI_COMMIT_BRANCH == "main"'
                
                #  파일 변경에 따라 작업을 실행하는 예시
                job:
                  script: echo "This job runs if certain files are modified"
                  rules:
                    - changes:
                        - src/*
                        - config/*
                
                # 파일 존재 여부에 따라 작업을 실행하는 예시
                job:
                  script: echo "This job runs if a specific file exists"
                  rules:
                    - exists:
                        - path/to/file.txt
                
                # 여러 규칙을 결합한 예시(아래 두 조건중 하나라도 걸리면 실행됨)
                #  커밋이 머지 요청에서 발생했을 때
                #  docs 디렉토리 내 파일이 변경되었을 때
                job:
                  script: echo "This job runs if the commit is from a merge request"
                  rules:
                    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
                    - changes:
                        - docs/*
                ```
                
        - rules 키워드
            - `rules:if`
                - 설명
                    - CI/CD 파이프라인에서 조건에 따라 특정 작업을 실행할지 말지를 결정하는 규칙을 정의하는 방식
                    - 이 규칙은 변수나 CI/CD의 미리 정의된 변수 값을 바탕으로 작업을 파이프라인에 추가할지 여부를 지정
                - 동작
                    - 조건이 참일 때 작업을 파이프라인에 추가:
                        - `if` 조건이 참이면 해당 작업을 파이프라인에 추가
                    - 조건이 참이지만 `when: never`가 지정된 경우 작업을 추가하지 않음
                        - 조건이 참이지만 `when: never`가 설정된 경우 작업을 파이프라인에 추가하지 않음
                    - 조건이 거짓일 경우, 다음 규칙을 체크
                        - 조건이 거짓이면, 추가적으로 정의된 다른 `rules` 항목을 확인
                    - 규칙은 순차적으로 평가됨
                        - 규칙은 정의된 순서대로 평가되며, 규칙에 따라 작업의 실행 여부가 결정
                - 규칙 항목
                    - **CI/CD 변수**
                        - `rules:if`는 CI/CD 변수나 미리 정의된 CI/CD 변수를 기반으로 조건을 평가
                        - 단, 일부 예외가 있을 수 있음
                    - **조건문 (if)**
                        - `if` 절은 일반적으로 CI/CD 변수에 대한 조건을 포함
                        - 예를 들어 `if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^feature/`와 같은 방식으로 조건을 정의할 수 있음
                    - `when`과의 결합
                        - `rules:if` 조건이 참일 경우, `when`을 사용하여 조건이 만족했을 때 작업의 상태를 정의할 수 있음
                        - 기본값은 `when: on_success`
                        - `when: never`
                            - 작업을 실행하지 않음.
                        - `when: manual`
                            - 수동으로 작업을 실행하도록 설정.
                        - `when: always`
                            - 조건과 관계없이 항상 실행하도록 설정.
                    - `allow_failure` 과의 결합
                        - `allow_failure: true`를 사용하면 해당 작업이 실패해도 파이프라인을 실패로 간주하지 않도록 설정할 수 있음
                - 추가 세부 사항
                    - `rules:if`에서는 중첩된 변수를 사용할 수 없음
                    - `rules:if`는 `include`와 함께 사용하여 조건적으로 다른 구성 파일을 포함할 수도 있음
                - 예시
                    
                    ```yaml
                    job:
                      script: echo "Hello, Rules!"
                      rules:
                        # source 브랜치가 feature로 시작하고
                        # target 브랜치가 기본 브랜치가 아닌 경우에 해당 작업을 실행하지 않음
                        - if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^feature/ && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME != $CI_DEFAULT_BRANCH
                          when: never
                        # source 브랜치가 feature로 시작하는 경우에 작업을 수동으로 실행하도록 설정
                        # 실패해도 파이프라인 실패로 간주하지 않음 (allow_failure: true).
                        - if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^feature/
                          when: manual
                          allow_failure: true
                        # 단순히 source 브랜치가 존재하는지 확인하는 조건
                        # 조건이 참이라면 해당 작업이 실행
                        - if: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME
                    ```
                    
            - `rules:changes`
                - 설명
                    - 특정 파일에 대한 변경 여부를 기반으로 작업을 파이프라인에 추가할지 말지를 결정하는 규칙
                    - 이를 통해, 파일 변경이 있을 때만 작업을 실행하거나, 특정 파일이 변경되지 않으면 작업을 건너뛰도록 설정할 수 있음
                    - `rules:changes`는 `merge request`나 `branch` 파이프라인에서 유용하게 사용
                - 동작
                    - 변경 사항에 따라 작업 실행
                        - `rules:changes`는 특정 파일들이 변경된 경우에만 작업을 파이프라인에 추가
                        - 파일 경로를 기준으로 변경된 파일들을 확인
                    - 새 브랜치 및 Git 푸시 이벤트 없음
                        - 새 브랜치 파이프라인이나 Git 푸시 이벤트가 없는 경우, `rules:changes`는 항상 `true`로 평가되며 작업이 항상 실행됨
                        - 예를 들어, 태그 파이프라인, 스케줄된 파이프라인, 수동 파이프라인 등은 Git 푸시 이벤트와 연결되지 않기 때문에, `compare_to`를 사용하여 비교할 브랜치를 지정해야 할 수 있음
                    - 병합 요청(Merge Request) 파이프라인
                        - 병합 요청 파이프라인에서는 `rules:changes`가 타겟 브랜치와의 변경 사항을 비교하여 해당 작업을 실행할지 결정
                    - 브랜치 파이프라인
                        - 브랜치 파이프라인에서는 `rules:changes`가 이전 커밋과의 변경 사항을 비교하여 작업을 실행할지 결정
                - 사용 가능한 입력
                    - 파일 경로
                        - 특정 파일을 지정할 수 있음
                        - 예: `Dockerfile`
                    - 와일드카드 경로
                        - 디렉토리 또는 파일 패턴을 와일드카드로 지정할 수 있음
                        - `path/to/directory/*` : 특정 디렉토리의 모든 파일
                        - `path/to/directory/**/*` : 하위 디렉토리를 포함한 모든 파일
                        - `.md` : 특정 확장자를 가진 모든 파일
                        - `"**/*.json"` : 모든 디렉토리에서 `.json` 파일을 포함하는 경로
                    - CI/CD 변수 사용 가능
                        - CI/CD 변수도 경로로 포함할 수 있음
                            
                            ```yaml
                            job:
                              script: echo "Building Docker image"
                              rules:
                                - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                                  changes:
                                    - $CI_PROJECT_DIR/Dockerfile
                            ```
                            
                - 예시
                    
                    ```yaml
                    
                    # CI_PIPELINE_SOURCE가 merge_request_event일 때,
                    # Dockerfile에 변경이 있으면 해당 작업을 수동으로 실행하도록 설정하고(when: manual), 
                    # 작업 실패 시 파이프라인이 계속 실행되도록(allow_failure: true) 설정
                    docker build:
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            - Dockerfile
                          when: manual
                          allow_failure: true
                    
                    # CI_PIPELINE_SOURCE가 merge_request_event일 때, 
                    # path/to/dockerfiles/ 디렉토리 내 모든 파일에 변경이 있으면 작업이 실행
                    docker build alternative:
                      variables:
                        DOCKERFILES_DIR: 'path/to/dockerfiles'
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            - $DOCKERFILES_DIR/**/*
                    
                    # 지정된 파일이나 패턴에 변경 사항이 없다면, 작업은 파이프라인에 추가되지 않음
                    ```
                    
            - `rules:changes:paths`
                - 설명
                    - `rules:changes:paths`는 `rules:changes`와 동일한 기능을 수행하며, 추가적인 하위 키 없이 파일 경로를 지정하는 방식
                - 예시
                    
                    ```yaml
                    # 아래 두 설정은 동작이 동일
                    docker-build-1:
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            - Dockerfile
                    
                    docker-build-2:
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            paths:
                              - Dockerfile
                    
                    ```
                    
            - `rules:changes:compare_to`
                - 설명
                    - 파일 변경을 비교할 기준이 되는 참조(ref)를 지정하는 키워드
                - 가능한 입력
                    - 브랜치 이름 (예: `main`, `branch1`)
                    - 태그 이름 (예: `tag1`)
                    - 커밋 SHA (예: `2fg31ga14b`)
                    - CI/CD 변수 사용
                - 추가 세부 사항
                    - 파일 경로 패턴 해석
                        - `rules:changes`에서 사용하는 파일 경로는 Ruby의 `File.fnmatch`를 사용하여 해석되며, `File::FNM_PATHNAME`, `File::FNM_DOTMATCH`, `File::FNM_EXTGLOB` 플래그가 적용
                    - 최대 50개의 패턴
                        - `rules:changes`에서 정의할 수 있는 파일 경로 패턴은 최대 50개로 제한
                    - 변경된 파일이 하나라도 있으면
                        - 나열된 파일 중 하나라도 변경되면 조건이 참이 되어 작업이 실행 (OR 연산)
                - 주의사항
                    
                    ```yaml
                    # 병합된 결과에서 변경된 파일을 정확히 반영하지 못함
                    # 시나리오
                    #   feature 브랜치에서 index.html 파일을 수정하고, main 브랜치에 병합하려고 합
                    #   이때 병합된 결과 파이프라인은 index.html 파일이 병합된 커밋에서 변경되었는지 확인하는 대신
                    #   병합 요청을 생성한 커밋과 비교할 수 있음
                    #   만약 index.html 파일이 병합 커밋에서만 변경되었고, 
                    #   이전 커밋에서는 변경되지 않았다면, rules:changes는 변경 사항을 감지하지 못하고 
                    #   작업이 실행되지 않을 수 있음
                    job:
                      script: echo "Deploy the website"
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            - index.html
                    
                    ---
                    # rules:changes:compare_to를 사용할 경우, 
                    # 병합된 결과 파이프라인에서 비교하는 기준을 잘못 설정하면 의도한 대로 변경 사항을 
                    # 감지하지못할 수 있음
                    job:
                      script: echo "Run tests"
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            paths:
                              - src/**/*  # src 디렉토리 내 모든 파일 변경 여부
                            compare_to: "refs/heads/main"  # main 브랜치를 기준으로 비교
                    
                    # 해결방안 : 병합된 결과를 기준으로 비교할 수 있도록 규칙을 정확하게 설정
                    job:
                      script: echo "Run tests"
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            paths:
                              - src/**/*  # src 디렉토리 내 모든 파일 변경 여부
                            compare_to: $CI_COMMIT_SHA  # 병합된 커밋을 기준으로 비교
                    ```
                    
                    - GitLab에서 생성된 병합된 결과 파이프라인에서는 예기치 않은 결과가 발생할 수 있음
                - 예시
                    
                    ```yaml
                    # Dockerfile에 변경 사항이 있을 경우, 
                    # refs/heads/branch1을 기준으로 변경된 파일을 비교하고, 그 결과에 따라 작업을 실행
                    docker build:
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          changes:
                            paths:
                              - Dockerfile
                            compare_to: 'refs/heads/branch1'
                    ```
                    
            - `rules:exists`
                - 설명
                    - GitLab CI/CD에서 특정 파일이 리포지토리 내에 존재하는지 확인하여, 해당 파일이 있을 경우에만 작업을 실행하도록 설정하는 규칙
                    - 이 규칙을 사용하면, 특정 파일이나 디렉토리가 존재하는 경우에만 파이프라인에서 특정 작업을 실행할 수 있음
                    - 이 규칙은 파일 경로를 기준으로 판단하며, 파일이 존재하면 조건이 참으로 평가되어 해당 작업이 실행
                - 사용 방법
                    - `rules:exists`는 배열 형식으로 파일 경로를 나열
                    - 경로는 프로젝트 디렉토리(즉, `$CI_PROJECT_DIR`)를 기준으로 하며, 프로젝트 외부의 경로를 참조할 수 없음
                    - 경로에는 와일드카드(glob 패턴)과 CI/CD 변수를 사용할 수 있음
                - 추가 세부 사항
                    - 글로브 패턴
                        - `rules:exists`에서 사용되는 파일 경로는 Ruby의 `File.fnmatch` 메서드를 사용하여 해석되며, `FNM_PATHNAME`, `FNM_DOTMATCH`, `FNM_EXTGLOB` 플래그가 적용
                        - 예를 들어, `*/Dockerfile`과 같은 패턴을 사용하여 모든 서브디렉토리에서 `Dockerfile`을 찾을 수 있음
                    - 파일 제한
                        - GitLab은 `exists` 규칙을 평가할 때 최대 10,000번의 체크를 수행할 수 있음
                        - 만약 프로젝트에 10,000개 이상의 파일이 존재하면, GitLab은 패턴이 항상 일치한다고 간주하여 성능을 최적화
                        - 여러 개의 패턴을 사용하는 경우, 체크할 수 있는 파일 수는 10,000을 패턴의 수로 나눈 값
                            - 예를 들어, 4개의 패턴이 있다면 각 패턴에 대해 2,500개의 파일을 체크할 수 있음
                    - 파일 경로 제한
                        - `rules:exists`에서 사용할 수 있는 파일 경로는 최대 50개까지 정의할 수 있음
                    - 다른 `include` 파일에서의 검색
                        - `rules:exists`를 `include`와 함께 사용할 때, `include` 파일의 프로젝트와 ref에서 파일이 존재하는지 확인
                        - 즉, 병합 요청을 사용한 `include`에서, 파일이 다른 프로젝트나 브랜치에 있을 경우 그 파일이 존재하는지 검색할 수 있음
                    - 아티팩트 검색 불가
                        - `rules:exists`는 아티팩트를 검색할 수 없음
                        - 왜냐하면 이 규칙은 작업 실행 전에 평가되므로, 아티팩트는 이미 만들어진 파일들이므로 사용할 수 없음
                - 예시
                    
                    ```yaml
                    # 기본 예시: Dockerfile이 존재하는 경우에만 작업 실행
                    job:
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - exists:
                            - Dockerfile
                    
                    # 변수를 사용한 예시: 특정 경로에 Dockerfile이 존재하는 경우
                    #  이 예시에서는 DOCKERPATH라는 변수를 사용하여,
                    #   레포지토리 내의 모든 경로에 있는 Dockerfile이 존재할 경우에만 해당 작업이 실행
                    job2:
                      variables:
                        DOCKERPATH: "**/Dockerfile"
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - exists:
                            - $DOCKERPATH
                    
                    # 여러 파일 경로 확인
                    #  Dockerfile, src/main.py, config/settings.yml 중 하나라도 존재하면 해당 작업이 실행
                    job:
                      script: echo "Building Docker image"
                      rules:
                        - exists:
                            - Dockerfile
                            - src/main.py
                            - config/settings.yml
                    ```
                    
            - `rules:exists:paths`
                - 설명
                    - `rules:exists:paths`는 `rules:exists`와 동일한 기능을 제공하지만, `paths`라는 키워드를 사용하여 파일 경로를 지정하는 방식으로 더 직관적으로 표현한 규칙
                    - 이 규칙은 파일 경로를 기반으로 하며, 경로는 프로젝트 디렉토리 내에서만 유효하고 외부 경로를 참조할 수 없음
                    - `rules:exists:paths`는 파일이 존재하는지 여부를 확인하고, 존재하면 규칙이 참으로 평가되어 해당 작업을 실행
                - 사용 방법
                    - `rules:exists:paths`는 **배열 형식**으로 파일 경로를 나열
                    - 경로는 **프로젝트 디렉토리** 내에서 상대 경로로 설정하며, **CI/CD 변수**나 **글로브 패턴**을 사용 가능
                - 예시
                    
                    ```yaml
                    docker-build-2:
                      variables:
                        DOCKERPATH: "**/Dockerfile"
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "merge_request_event"
                          exists:
                            paths:
                              - $DOCKERPATH
                    
                    job:
                      script: echo "Building Docker image"
                      rules:
                        - exists:
                            paths:
                              - Dockerfile
                              - src/main.py
                              - config/settings.yml
                    ```
                    
            - `rules:exists:project`
                - 설명
                    - GitLab CI/CD에서 다른 프로젝트에서 특정 파일이 존재하는지 확인하여, 해당 파일이 존재할 경우에만 작업을 실행하도록 설정하는 규칙
                - 사용 방법
                    - `exists:project`
                        - 파일을 검색할 다른 GitLab 프로젝트를 지정
                        - 경로는 프로젝트 경로로 입력되며, 이 경로는 네임스페이스와 그룹을 포함해야 함
                    - `exists:ref`
                        - 파일을 검색할 커밋 참조(브랜치, 태그, 또는 커밋 SHA)를 지정할 수 있음
                        - 이 값이 제공되지 않으면, 기본적으로 HEAD(가장 최신 커밋)에서 파일을 검색
                - 추가 세부사항
                    - 다른 프로젝트 파일 검색
                        - `rules:exists:project`는 파일이 다른 프로젝트에 존재하는지 확인하는 데 사용
                        - 예를 들어, `my-group/my-project`라는 다른 프로젝트 내에서 파일이 있는지 검색할 수 있음
                    - `exists:ref` 사용
                        - `exists:ref`는 검색할 커밋 참조를 지정
                        - 예를 들어, 특정 태그(`v1.0.0`), 브랜치(`main`), 또는 커밋 SHA를 지정하여 파일을 검색할 수 있음
                        - `ref` 값을 지정하지 않으면 HEAD(가장 최신 커밋)에서 검색
                    - `rules:exists:paths`와 결합
                        - `rules:exists:project`는 항상 `rules:exists:paths`와 함께 사용해야 함
                        - `paths`는 검색할 파일 경로를 지정하고, `project`는 그 파일을 검색할 프로젝트를 지정하는 역할을 함
                - 예시
                    
                    ```yaml
                    # 다른 프로젝트에서 특정 파일 존재 여부 확인
                    docker build:
                      script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
                      rules:
                        - exists:
                            paths:
                              - Dockerfile
                            project: my-group/my-project
                            ref: v1.0.0
                    ```
                    
            - `rules:when`
                - 설명
                    - GitLab CI/CD에서 파이프라인에 작업을 추가할 조건을 제어하는 데 사용되는 규칙
                    - `rules:when`은 `when` 키워드와 비슷하지만, 조건을 설정할 수 있는 옵션이 다소 다름
                    - `rules:when`은 조건을 더 세밀하게 조정할 수 있으며, 다른 조건과 결합하여 사용할 수 있음
                - 주요 특징
                    - `rules:when` 규칙이 `if`, `changes`, `exists`와 결합되지 않으면, 항상 그 규칙에 도달했을 때 해당 조건을 매칭
                    - 다른 규칙들과 함께 사용 가능
                        - `rules:when`은 다른 규칙들(`if`, `changes`, `exists`)과 함께 사용할 수 있으며, 조건에 맞을 경우 해당 작업이 실행되도록 함
                - 가능한 입력 값
                    - `on_success` (기본값) : 이전 단계의 모든 작업이 실패하지 않았을 때 작업을 실행
                    - `on_failure`: 이전 단계에서 적어도 하나의 작업이 실패했을 때 작업을 실행
                    - `never`: 이전 단계의 작업 상태와 관계없이 작업을 실행하지 않음
                    - `always`: 이전 단계의 작업 상태와 관계없이 항상 작업을 실행
                    - `manual`: 작업을 수동으로 추가. 이 경우 `allow_failure`의 기본값은 `false`로 설정
                    - `delayed`: 작업을 지연된 작업으로 추가
                - 추가 세부 사항
                    - `on_success`와 `on_failure`의 동작
                        - `on_success`: 이전 단계에서 실패한 작업이 없다면 실행
                        - `on_failure`: 이전 단계에서 적어도 하나의 작업이 실패한 경우 실행. `allow_failure: true`로 설정된 작업은 실패해도 성공으로 간주되어 `on_failure` 조건에 영향을 미치지 않음.
                        - 건너뛰어진 작업
                            - 예를 들어, 수동 작업은 아직 시작되지 않았더라도 성공한 것으로 간주되어 `on_success` 조건에 영향을 미칠 수 있음
                    - `manual` 사용 시 `allow_failure` 기본값
                        - `rules:when: manual`을 사용할 경우, 기본적으로 `allow_failure`가 `false`로 설정.
                        - 이는 `when: manual`을 `job` 수준에서 사용할 때와 다르며 `job` 수준에서 `when: manual`을 사용하면 `allow_failure`가 `true`로 기본 설정
                        - 이를 동일하게 만들려면, `rules: allow_failure: true`를 추가해야 함
                - 예시
                    
                    ```yaml
                    # 기본 브랜치에서는 on_success로 실행
                    # feature 브랜치에서는 작업이 delayed로 실행
                    # 다른 경우에는 작업이 manual로 실행
                    job1:
                      rules:
                        - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
                        - if: $CI_COMMIT_REF_NAME =~ /feature/
                          when: delayed
                        - when: manual
                      script:
                        - echo
                    
                    ```
                    
            - `rules:allow_failure`
                - 설명
                    - GitLab CI/CD에서 Job이 실패해도 파이프라인이 멈추지 않도록 허용하는 설정
                - 주요 설정
                    - `allow_failure: true`: 해당 job이 실패해도 파이프라인 실행을 계속
                    - `allow_failure: false`: 기본값으로, job이 실패하면 파이프라인이 중지
                    - `manual job`과 함께 사용: `allow_failure: true`는 `manual job`과 함께 사용할 수 있으며, 이를 통해 수동 작업이 실패해도 파이프라인을 계속 진행하게 할 수 있음
                    - `rules`와 함께 사용: `rules`에서 특정 조건을 정의하고, 그 조건이 맞으면 `allow_failure: true`를 적용할 수 있음
                - 예시
                    
                    ```yaml
                    # 조건이 맞으면 수동(job) 실행 시 실패해도 파이프라인이 계속 진행
                    job:
                      script: echo "Hello, Rules!"
                      rules:
                        - if: $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == $CI_DEFAULT_BRANCH
                          when: manual
                          allow_failure: true
                    
                    # 이 작업은 수동으로 트리거되어 실행되며, 실패해도 파이프라인은 계속 진행
                    job:
                      script: echo "Manual job starts!"
                      rules:
                        - when: manual
                          allow_failure: true
                    
                    # 여러 규칙에 적용
                    #   develop 브랜치에서는 수동으로 실행할 수 있고, 실패해도 파이프라인이 계속 진행
                    #   main 브랜치에서는 실패하면 파이프라인이 중지  (allow_failure: false, 기본값)
                    job:
                      script: echo "Conditional manual job with failure allowed."
                      rules:
                        - if: '$CI_COMMIT_BRANCH == "develop"'
                          when: manual
                          allow_failure: true
                        - if: '$CI_COMMIT_BRANCH == "main"'
                          allow_failure: false
                    ```
                    
            - `rules:needs`
                - 설명
                    - `needs` 구성을 동적으로 변경(덮어쓰기)하는 기능
                        - `needs`는 특정 job이 실행되기 전에 의존해야 하는 다른 job을 정의하는데 사용
                        - `rules:needs`는 조건에 맞는 경우, 해당 조건에 맞게 job의 `needs` 구성을 완전히 교체하는 기능
                    - `needs` 없이도 사용 가능
                - 주요 개념
                    - `needs`
                        - `needs`는 GitLab CI에서 어떤 job이 다른 job이 끝날 때까지 기다려야 하는지를 설정
                        - 예를 들어, `job A`가 `job B`가 완료된 후에만 실행되어야 한다면, `job A`의 `needs`에 `job B`를 지정할 수 있음
                    - `rules:needs`
                        - `rules:needs`는 특정 조건에 맞을 때만 `needs` 구성을 업데이트
                        - 예를 들어, `CI_COMMIT_BRANCH`가 특정 브랜치일 때 다른 job을 기다리도록 설정할 수 있음
                        - 조건에 맞는 `needs`가 정의되면, 기존 job-level의 `needs`는 덮어쓰여 새로 정의된 `needs`가 적용
                - 입력 가능한 값
                    - 배열 형태: 다른 job들의 이름을 배열로 지정
                        - 예: `needs: ['job1', 'job2']`
                    - 해시 형태: job 이름과 추가 속성들(`artifacts` 등)을 설정하는 방식
                        - 예: `needs: [{ job: 'job1', artifacts: true }]`
                    - 빈 배열: `needs`를 없애는 방법
                        - 예: `needs: []` (조건이 맞으면 이 job은 다른 job을 기다리지 않음)
                - 추가적인 세부 사항
                    - `rules:needs`가 job-level의 `needs`를 덮어씀
                        - `rules:needs`는 job에서 설정한 `needs`를 덮어씀
                        - 만약 job-level에서 `needs: ['job1']`을 설정하고, `rules:needs`에서 다른 조건을 만족할 때 `needs: ['job2']`로 설정하면, 조건이 맞을 때 `job2`가 적용
                - 예시
                    
                    ```yaml
                    # tests job 은 CI_COMMIT_BRANCH가 기본 브랜치가 아닌 경우 build-dev job을, 
                    # 기본 브랜치인 경우 build-prod job을 기다림
                    # 조건에 맞는 needs가 정의되면 해당 needs가 적용되고, 다른 조건에서는 다른 needs가 적용 
                    build-dev:
                      stage: build
                      rules:
                        - if: $CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH
                      script: echo "Feature branch, so building dev version..."
                    
                    build-prod:
                      stage: build
                      rules:
                        - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
                      script: echo "Default branch, so building prod version..."
                    
                    tests:
                      stage: test
                      rules:
                        - if: $CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH
                          needs: ['build-dev']
                        - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
                          needs: ['build-prod']
                      script: echo "Running dev specs by default, or prod specs when default branch..."
                    
                    # 복합 예시
                    #   tests job은 feature-branch에서 build-dev job을 기다리고, 
                    #   main에서는 build-prod job을 기다림
                    #   deploy job 은 tests job을 기다리며, 추가적으로 deploy-dev 또는 deploy-prod를 needs로 기다림
                    #   deploy-dev 는 feature-branch에서, deploy-prod 는 main에서만 실행 
                    deploy-dev:
                      stage: deploy
                      script: echo "Deploying to dev environment"
                    
                    deploy-prod:
                      stage: deploy
                      script: echo "Deploying to production environment"
                    
                    tests:
                      stage: test
                      rules:
                        - if: $CI_COMMIT_BRANCH == 'feature-branch'
                          needs: ['build-dev']
                        - if: $CI_COMMIT_BRANCH == 'main'
                          needs: ['build-prod']
                    
                    deploy:
                      stage: deploy
                      rules:
                        - if: $CI_COMMIT_BRANCH == 'feature-branch'
                          needs: ['tests', 'deploy-dev']
                        - if: $CI_COMMIT_BRANCH == 'main'
                          needs: ['tests', 'deploy-prod']
                      script: echo "Deployment job"
                    
                    # needs 와 rules:needs 함께 사용
                    build-dev:
                      stage: build
                      script: echo "Building for development..."
                    
                    build-prod:
                      stage: build
                      script: echo "Building for production..."
                    
                    tests:
                      stage: test
                      script: echo "Running tests"
                      needs: ['build-dev', 'build-prod']  # 기본적으로 두 개의 빌드 job을 기다림
                      rules:
                        - if: '$CI_COMMIT_BRANCH == "feature-branch"'
                          needs: ['build-dev']  # feature-branch에서만 build-dev만 기다림
                        - if: '$CI_COMMIT_BRANCH == "main"'
                          needs: ['build-prod']  # main 브랜치에서는 build-prod만 기다림
                    
                    deploy:
                      stage: deploy
                      script: echo "Deploying application"
                      needs:
                        - job: 'tests'  # 기본적으로 tests job을 기다립니다.
                      rules:
                        - if: '$CI_COMMIT_BRANCH == "feature-branch"'
                          needs: ['tests', 'build-dev']  # feature-branch에서는 build-dev와 tests를 기다림
                        - if: '$CI_COMMIT_BRANCH == "main"'
                          needs: ['tests', 'build-prod']  # main 브랜치에서는 build-prod와 tests를 기다림
                    
                    ```
                    
            - `rules:variables`
                - 설명
                    - 조건에 따라 특정 변수를 동적으로 설정하거나 덮어쓰는 기능을 제공
                    - 이 기능을 사용하면, 파이프라인 내에서 특정 조건이 만족될 때만 변수를 설정하거나 수정 가능
                - 주요 개념
                    - `variables`: Job 내에서 사용할 변수를 정의하는 키워드
                    - `rules:variables`: 조건에 맞는 경우, 기존에 정의된 변수 값을 덮어쓰거나 새 변수를 정의하는 방법
                        - 이를 통해 특정 조건에서만 변수를 다르게 설정할 수 있음
                - 가능 입력 값
                    - 해시: `VARIABLE-NAME: value` 형태로 변수 이름과 값을 설정
                    - 변수 덮어쓰기: 조건에 맞을 때 기존에 정의된 변수를 덮어쓸 수 있음
                    - 새로운 변수 정의: 조건에 맞을 때 새로운 변수를 정의할 수 있음
                - 예시
                    
                    ```yaml
                    # 변수 덮어쓰기
                    job:
                      variables:
                        DEPLOY_VARIABLE: "default-deploy"  # 기본값 설정
                      rules:
                        - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
                          variables:                             # 기본 변수 덮어쓰기
                            DEPLOY_VARIABLE: "deploy-production"  # 기본값을 배포용 변수로 덮어씀
                        - if: $CI_COMMIT_REF_NAME =~ /feature/      # feature 브랜치에서는 새로운 변수 추가
                          variables:
                            IS_A_FEATURE: "true"                   # 새로운 변수 정의
                      script:
                        - echo "Run script with $DEPLOY_VARIABLE as an argument"
                        - echo "Run another script if $IS_A_FEATURE exists"
                    ```
                    
            - `rules:interruptible`
                - 설명
                    - 특정 조건에서 job의 `interruptible` 값을 변경(덮어쓰기) 할 수 있게 해주는 기능
                    - 이 기능을 사용하면, job이 실행되는 조건에 따라 job을 중단 가능하게 만들거나 중단 불가능하게 설정할 수 있음
                - 사용법
                    - `interruptible`: 기본적으로 `true` 또는 `false` 값으로 설정할 수 있으며, `true`이면 job이 중단 가능하고, `false`이면 중단 불가능
                    - `rules:interruptible`: 조건에 따라 `interruptible` 값을 동적으로 변경하는 기능으로 조건이 맞으면 `interruptible`을 변경할 수 있음
                    - `interruptible`의 기본값: `interruptible`이 job 내에서 설정되지 않으면 기본값은 `false`이며 이를 통해 job을 취소할 수 없는 상태로 설정
                - 예시
                    
                    ```yaml
                    # 기본적으로 interruptible은 true로 설정되어 있어 job은 실행 중에 중단 가능
                    #  CI_COMMIT_REF_NAME이 기본 브랜치 ($CI_DEFAULT_BRANCH)인 경우,
                    #  interruptible을 false로 설정하여 job을 중단할 수 없게 만듦
                    # 두 번째 규칙 (when: on_success)은 job이 성공적으로 완료되었을 때 실행되지만, 
                    # 이 규칙은 interruptible 값을 설정하는 데 영향을 주지 않음
                    job:
                      script: echo "Hello, Rules!"
                      interruptible: true  # 기본적으로 job은 취소 가능한 상태
                      rules:
                        - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
                          interruptible: false  # 기본 브랜치에서는 job을 중단 불가능하도록 설정
                        - when: on_success
                    ```
                    
    - **when**
        - 설명
            - `when` 키워드는 파이프라인에서 특정 작업이 실행될 조건을 설정하는 데 사용
            - 각 작업에 대해 정의할 수 있으며, 이전 단계의 작업이 성공하거나 실패한 상태에 따라 작업을 실행할지 여부를 결정
            - `when` 키워드가 명시되지 않으면 기본값은 `on_success`
        - 가능한 값
            - `on_success` (기본값)
                - 이전 단계의 모든 작업이 실패하지 않은 경우에만 해당 작업이 실행
                - 기본값이므로 `when`을 정의하지 않으면 자동으로 이 값이 적용
            - `on_failure`
                - 이전 단계에서 하나 이상의 작업이 실패한 경우에만 해당 작업이 실행
                - 주로 실패 후 처리 작업(예: 실패한 빌드를 정리하는 작업)에서 사용
            - `never`
                - 해당 작업은 절대 실행되지 않습니다. 이전 단계의 작업 상태와 관계없이 실행되지 않음
                - `workflow: rules` 또는 `rules` 섹션 내에서만 사용할 수 있음
            - `always`
                - 해당 작업은 이전 단계의 작업 상태와 관계없이 항상 실행
                - 주로 마지막 정리 작업이나 알림을 위해 사용
            - `manual`
                - 해당 작업은 파이프라인에 수동으로 추가되며, 사용자가 GitLab UI에서 수동으로 실행할 수 있음
                - 기본적으로 `allow_failure: true`로 설정되어 있어, 이 작업은 실패해도 파이프라인에 영향을 주지 않음
        - 세부 사항
            - `on_success`와 `on_failure` 평가 시
                - `allow_failure: true`로 설정된 이전 단계의 작업은 실패하더라도 성공으로 간주
                - 이전 단계에서 수동으로 실행되지 않은 작업(예: 수동 작업)은 성공으로 간주
            - 기본값
                - `manual`로 설정된 작업의 기본값은 `allow_failure: tru`
                - `rules: when: manual`을 사용할 때는 `allow_failure: false`로 변경
            - `when`은 `rules`와 함께 사용하여 더 동적인 작업 제어가 가능
            - `when`은 `workflow`와 함께 사용하여 파이프라인이 언제 시작될지 제어할 수 있음
        - 예시
            
            ```yaml
            # cleanup_build_job: build_job이 실패할 경우에만 실행됩니다.
            # cleanup_job: 파이프라인의 마지막 단계에서 항상 실행됩니다, 성공/실패에 관계없이.
            # deploy_job: GitLab UI에서 수동으로 실행할 때만 실행됩니다.
            stages:
              - build
              - cleanup_build
              - test
              - deploy
              - cleanup
            
            build_job:
              stage: build
              script:
                - make build
            
            cleanup_build_job:
              stage: cleanup_build
              script:
                - cleanup build when failed
              when: on_failure
            
            test_job:
              stage: test
              script:
                - make test
            
            deploy_job:
              stage: deploy
              script:
                - make deploy
              when: manual
              environment: production
            
            cleanup_job:
              stage: cleanup
              script:
                - cleanup after jobs
              when: always
            ```
            
    - **tag**
        - 설명
            - `tags` 키워드는 특정 러너를 선택하여 작업을 실행하는 데 사용
            - 각 러너는 등록 시 특정 태그를 지정할 수 있으며, 작업을 실행하려면 해당 작업에 정의된 모든 태그를 가진 러너가 필요
        - 사용 방법
            - `tags`는 작업 내에서만 사용 가능합니다. 또는 default 섹션에서 사용 가능
            - 러너는 등록 시 태그를 지정하고, 작업을 실행하려면 작업에 정의된 모든 태그를 가진 러너가 선택되어야 함
        - 가능한 입력
            - 태그 이름 배열
                - 러너가 실행할 작업에 대해 필요한 태그 목록을 지정
                - CI/CD 변수도 사용할 수 있음
        - 추가 세부 사항
            - 태그의 개수는 50개 이하여야 함
            - 병렬 매트릭스 작업에서 각 작업에 대해 다른 러너 태그를 선택할 수 있음
                
                ```yaml
                # test_matrix 작업은 parallel: matrix를 사용하여 두 개의 병렬 작업을 정의 
                # 첫 번째 작업은 ruby와 postgres 태그를 가진 러너에서 실행 
                # 두 번째 작업은 nodejs와 mysql 태그를 가진 러너에서 실행 
                # TAGS는 매트릭스 배열에서 각 작업에 대해 지정된 태그 값을 가져옴
                # $TAGS는 CI/CD에서 해당 매트릭스 값(ruby, postgres 또는 nodejs, mysql)을 동적으로 설정하여
                #  각 작업에 적합한 러너에서 실행되도록 함
                stages:
                  - test
                
                test_matrix:
                  stage: test
                  script:
                    - echo "Testing with different environments"
                  parallel:
                    matrix:
                      - TAGS: ["ruby", "postgres"]
                      - TAGS: ["nodejs", "mysql"]
                  tags:
                    - $TAGS
                ```
                
        - 예시
            
            ```yaml
            # job은 ruby와 postgres 태그를 가진 러너에서만 실행
            # 이 작업을 실행하려면 둘 다의 태그가 있는 러너가 필요
            job:
              tags:
                - ruby
                - postgres
            ```
            
- 아티팩트 및 캐시
    - **artifacts**
        - 설명
            - `artifacts`는 GitLab CI/CD에서 특정 작업(job)의 출력물을 저장하고 관리하는 기능
            - 출력물은 성공, 실패, 혹은 항상 저장될 수 있으며, 다른 후속 작업에서 이를 다운로드하거나 참조할 수 있음
            - `artifacts`는 크기가 설정된 최대치 이하일 경우 저장
            - 기본적으로 후속 작업들은 이전 작업에서 생성된 `artifacts`를 자동으로 다운로드
            - 아티팩트는 GitLab 서버의 파일 스토리지에 저장
            - 기본적으로 30일 동안 보관되며, 이 기간은 설정 가능
            - 각 작업의 결과 페이지에서 artifacts 에 접근하고 다운로드할 수 있음
            - `needs` 키워드를 사용하는 경우, 명시적으로 지정된 작업에서만 `artifacts`를 다운로드 가능
        - artifacts 키워드
            
            ```yaml
            job1:
              stage: test
              script:
                - echo "Build something"
              artifacts:
                paths:
                  - build/
                exclude:
                  - build/*.o  # .o 파일 제외
                name: "build-artifact"
                expire_in: 2 hours
                when: always  # 항상 artifacts 업로드
                expose_as: "Build Artifact"
                untracked: true  # untracked 파일도 포함
            
            ```
            
            - `artifacts:paths`
                - 설명
                    - 저장할 파일이나 디렉토리의 경로를 지정
                    - 경로는 프로젝트 디렉토리(`$CI_PROJECT_DIR`)를 기준으로 작성
                - 예시
                    
                    ```yaml
                    # binaries/ 디렉토리와 .config 파일을 artifacts로 저장
                    job1:
                      script:
                        - echo "Build something"
                      artifacts:
                        paths:
                          - binaries/
                          - .config
                    ```
                    
            - `artifacts:exclude`
                - 설명
                    - `artifacts:paths`에 포함된 파일 중 특정 파일을 제외합니다.
                - 예시
                    
                    ```yaml
                    # binaries/ 디렉토리의 모든 파일을 저장하되, .o 파일은 제외
                    job1:
                      script:
                        - echo "Build something"
                      artifacts:
                        paths:
                          - binaries/
                        exclude:
                          - binaries/**/*.o  # binaries 폴더 내 .o 파일 제외
                    
                    ```
                    
            - `artifacts:expire_in`
                - 설명
                    - `artifacts`의 만료 시간을 설정
                    - 만료되면 자동으로 삭제
                - 예시
                    
                    ```yaml
                    # artifacts를 1주일 동안 보관한 후 삭제하도록 설정
                    job1:
                      script:
                        - echo "Build something"
                      artifacts:
                        expire_in: 1 week
                    ```
                    
            - `artifacts:expose_as`
                - 설명
                    - Merge Request UI에서 `artifacts`를 다운로드할 수 있도록 표시할 이름을 설정
                - 예시
                    
                    ```yaml
                    # output.log 파일을 artifacts로 저장
                    # Merge Request UI에서 "My Artifact"라는 이름으로 표시
                    job1:
                      script:
                        - echo "Generate artifact"
                      artifacts:
                        expose_as: "My Artifact"
                        paths:
                          - output.log
                    
                    ```
                    
            - `artifacts:name`
                - 설명
                    - 생성된 `artifacts` 아카이브의 이름을 정의
                    - 기본값은 `artifacts.zip`
                - 예시
                    
                    ```yaml
                    #  build/ 디렉토리를 custom-artifact-name.zip으로 저장
                    job1:
                      script:
                        - echo "Build something"
                      artifacts:
                        name: "custom-artifact-name"
                        paths:
                          - build/
                    ```
                    
            - `artifacts:public`
                - 설명
                    - 공용 파이프라인에서 `artifacts`의 공개 여부를 설정
                    - 기본값은 `true`로, 누구나 다운로드할 수 있음
                - 예시
                    
                    ```yaml
                     # 이 작업의 artifacts는 비공개
                    job1:
                      script:
                        - echo "Build something"
                      artifacts:
                        public: false 
                    ```
                    
            - `artifacts:access`
                - 설명
                    - `artifacts`에 접근할 수 있는 권한을 설정
                    - `developer`, `none`, `all` 등의 옵션을 사용할 수 있음
                - 예시
                    
                    ```yaml
                    # Developer 이상의 권한을 가진 사용자만 접근 가능
                    job1:
                      script:
                        - echo "Build something"
                      artifacts:
                        access: 'developer' 
                    
                    ```
                    
            - `artifacts:reports`
                - 설명
                    - 특정 보고서 유형을 `artifacts`로 저장
                        - 예를 들어, `junit`, `coverage`, `sast` 등의 보고서를 저장
                - 예시
                    
                    ```yaml
                    # rspec.xml 파일을 JUnit 보고서 형식으로 artifacts로 저장
                    rspec:
                      stage: test
                      script:
                        - bundle install
                        - rspec --format RspecJunitFormatter --out rspec.xml
                      artifacts:
                        reports:
                          junit: rspec.xml
                    ```
                    
            - `artifacts:untracked`
                - 설명
                    - Git에 추적되지 않은 파일을 `artifacts`로 포함
                    - `.gitignore`에 설정된 파일도 포함
                - 예시
                    
                    ```yaml
                    # Git에서 추적되지 않는 파일도 포함
                    job1:
                      script:
                        - echo "Generate untracked files"
                        - touch untracked_file.txt
                      artifacts:
                        untracked: true
                    ```
                    
            - `artifacts:when`
                - 설명
                    - `artifacts`를 업로드할 시점을 지정
                    - `on_success`(기본값), `on_failure`, `always` 옵션을 사용 가능
                - 예시
                    
                    ```yaml
                    job1:
                      script:
                        - echo "Test something"
                      artifacts:
                        when: on_failure  # 실패 시에만 artifacts 업로드
                        paths:
                          - test.log
                    ```
                    
        - 주의사항
            - artifacts 는 caches 이후에 복원됨
            - 기본적으로 job 이 성공한 경우에만 수집 가능
            - needs 와 같은 키워드 사용하지 않으면, 후속 job에 이전 모든 artifacts 가 전달됨
            - paths 는 job 이 실행되는 최초의 경로($CI_PROJECT_DIR) 을 기준으로 상대 경로로 사용해야 함
            - 쉽게 다운로드 받고 확인이 가능하므로 민감한 파일들은 사용 x
            - expire_in 설정하더라도, 가장 최신 artifacts 는 만료되지 않음
    - **cache**
        - 설명
            - `cache`는 작업 간 파일이나 디렉터리를 캐싱하여 빌드 시간 및 리소스를 절약하는 데 사용
            - `cache`를 사용하면 동일한 데이터나 의존성을 반복적으로 다운로드하지 않아도 되므로 효율성이 증가
            - `cache`를 정의하여 job 간에 데이터를 공유
            - `cache`는 효율적이지만, 너무 큰 파일을 캐싱하면 성능에 영향을 미칠 수 있으니 주의
        - 특징
            - 공유 범위
                - 파이프라인 및 작업 간에 캐시가 공유
            - 분리
                - 기본적으로 보호된 브랜치와 보호되지 않은 브랜치 간에 캐시가 공유되지 않음
            - 복원 시점
                - `artifacts` 복원 전에 캐시가 복원
            - 제한
                - 최대 4개의 서로 다른 캐시만 사용할 수 있음
            - 비활성화 가능
                - 특정 작업에서 캐시를 비활성화하거나 기본 캐시를 재정의할 수 있음
        - cache 키워드
            - `cache:paths`
                - 설명
                    - 특정 파일 또는 디렉터리를 캐싱
                - 입력값
                    - 프로젝트 디렉터리(`$CI_PROJECT_DIR`)를 기준으로 한 경로 배열 (와일드카드 사용 가능).
                - 예시
                    
                    ```yaml
                    job:
                      script:
                        - echo "Using cache"
                      cache:
                        key: binaries-cache
                        paths:
                          - binaries/*.apk
                          - .config
                    ```
                    
            - `cache:key`
                - 설명
                    - 캐시의 고유 키를 정의
                - 기본값
                    - default
                - 예시
                    
                    ```yaml
                    job:
                      script:
                        - echo "Using cache with unique key"
                      cache:
                        key: my-cache-$CI_COMMIT_REF_SLUG
                        paths:
                          - binaries/
                    ```
                    
            - `cache:key:files`
                - 설명
                    - 특정 파일이 변경될 때 새로운 캐시를 생성
                - 예시
                    
                    ```yaml
                    job:
                      script:
                        - echo "Cache changes when files are modified"
                      cache:
                        key:
                          files:
                            - Gemfile.lock
                            - package.json
                        paths:
                          - vendor/ruby
                          - node_modules
                    
                    ```
                    
            - `cache:key:prefix`
                - 설명
                    - `cache:key:files`와 조합하여 **고유한 접두사**를 추가
                - 예시
                    
                    ```yaml
                    job:
                      script:
                        - echo "Using prefix for cache key"
                      cache:
                        key:
                          files:
                            - Gemfile.lock
                          prefix: $CI_JOB_NAME
                        paths:
                          - vendor/ruby
                    ```
                    
            - `cache:untracked`
                - 설명
                    - Git 에 추적되지 않은 파일도 캐싱
                - 입려값
                    - true / false
                - 예시
                    
                    ```yaml
                    job:
                      script: test
                      cache:
                        untracked: true
                    ```
                    
            - `cache:unprotect`
                - 설명
                    - 보호된 브랜치와 보호되지 않은 브랜치 간 캐시 공유를 설정
                - 입력값
                    - true / false
                - 예시
                    
                    ```yaml
                    job:
                      script: test
                      cache:
                        unprotect: true
                    ```
                    
            - `cache:when`
                - 설명
                    - 캐시 저장 조건을 설정
                - 옵션
                    - `on_success` (기본값) → 작업 성공 시에만 저장
                    - `on_failure` → 작업 실패 시에만 저장
                    - `always`→ 항상 저장
                - 예시
                    
                    ```yaml
                    job:
                      script: rspec
                      cache:
                        paths:
                          - rspec/
                        when: 'always'
                    ```
                    
            - `cache:policy`
                - 설명
                    - 캐시 업로드 및 다운로드 동작을 정의
                - 옵션
                    - `pull-push` (기본값): 다운로드 및 업로드 모두 수행.
                    - `pull`: 다운로드만 수행.
                    - `push`: 업로드만 수행.
                - 예시
                    
                    ```yaml
                    prepare-job:
                      stage: build
                      cache:
                        key: build-cache
                        paths:
                          - dependencies/
                        policy: push
                    
                    test-job:
                      stage: test
                      cache:
                        key: build-cache
                        paths:
                          - dependencies/
                        policy: pull
                    
                    ```
                    
            - `cache:fallback_keys`
                - 설명
                    - 기본 키가 없을 경우 사용할 **대체 키를 정의**
                - 예시
                    
                    ```yaml
                    job:
                      script: rspec
                      cache:
                        key: gems-$CI_COMMIT_REF_SLUG
                        paths:
                          - rspec/
                        fallback_keys:
                          - gems
                        when: 'always'
                    ```
                    
            - `cache:paths`
                - 설명
                - 예시
                    
                    ```yaml
                    
                    ```
                    
- 변수 및 시크릿
    - **variables**
        - 설명
            - 요약
                - `variables` 키워드는 CI/CD 파이프라인에서 변수들을 정의하는 데 사용
                - 이를 통해 작업에서 사용할 수 있는 변수들을 설정할 수 있음
            - 유형
                - 전역 변수 (Global)
                    - 파이프라인 전체에서 사용할 수 있는 변수들로, 모든 작업에서 접근 가능
                    - 작업에서 동일한 변수를 정의하면 해당 작업의 변수 값이 우선시 됨
                - 작업 변수 (Job)
                    - 특정 작업에서만 사용할 수 있는 변수들로, 해당 작업의 `script`, `before_script`, `after_script` 등에서 사용
                - 사전 정의된 변수 (Predefined Variables)
                    - GitLab CI/CD 런너가 자동으로 생성하여 작업에서 사용할 수 있는 변수
                        
                        ```yaml
                        # CI_PIPELINE_ID: 파이프라인의 고유 ID를 출력 
                        # CI_JOB_NAME: 현재 실행 중인 작업의 이름을 출력 
                        # CI_COMMIT_REF_NAME: 현재 파이프라인이 실행 중인 브랜치 이름을 출력 
                        # CI_COMMIT_SHA: 현재 커밋의 SHA 해시를 출력 
                        job1:
                          script:
                            - echo "Pipeline ID: $CI_PIPELINE_ID"
                            - echo "Job name: $CI_JOB_NAME"
                            - echo "Branch name: $CI_COMMIT_REF_NAME"
                            - echo "Git commit: $CI_COMMIT_SHA"
                        ```
                        
                - 런너 변수
                    - CI/CD 파이프라인의 런너 동작을 설정하는 데 사용되는 변수
                        
                        ```yaml
                        # CI_RUNNER_ID: 작업을 실행하는 GitLab 런너의 고유 ID를 출력 .
                        # CI_RUNNER_TAGS: 해당 런너에 지정된 태그 목록을 출력. 이를 통해 특정 태그가 지정된 러너에서만 작업을 실행할 수 있음
                        # CI_RUNNER_EXECUTOR: 작업을 실행하는 런너의 실행 방법을 출력
                        # 예를 들어, Docker 실행기를 사용하는지, Shell 실행기를 사용하는지 등을 확인 가능
                        job1:
                          script:
                            - echo "Runner ID: $CI_RUNNER_ID"
                            - echo "Runner tags: $CI_RUNNER_TAGS"
                            - echo "Runner executor: $CI_RUNNER_EXECUTOR"
                        ```
                        
            - 세부 사항
                - **Docker 서비스 컨테이너**에도 YAML에서 정의된 변수들이 적용
                - YAML에서 정의된 변수는 **비민감한 프로젝트 설정**에 적합하며, 민감한 정보는 **보호된 변수**나 **CI/CD 비밀**로 저장해야 함
                - 수동 파이프라인 변수와 예약된 파이프라인 변수는 기본적으로 하위 파이프라인에 전달되지 않음
                - `trigger:forward`를 사용하여 이러한 변수를 하위 파이프라인으로 전달할 수 있음
            - 예시
                
                ```yaml
                # variables 섹션에서 DEPLOY_SITE 변수를 전역 변수로 정의 
                # deploy_job 작업은 전역 변수 DEPLOY_SITE를 사용하여 배포 URL을 지정 
                # deploy_review_job 작업은 자체적으로 DEPLOY_SITE 변수와 REVIEW_PATH 변수를 정의하여 사용
                # 이 경우, deploy_review_job에서 정의된 DEPLOY_SITE가 우선 적용 
                variables:
                  DEPLOY_SITE: "https://example.com/"
                
                deploy_job:
                  stage: deploy
                  script:
                    - deploy-script --url $DEPLOY_SITE --path "/"
                  environment: production
                
                deploy_review_job:
                  stage: deploy
                  variables:
                    DEPLOY_SITE: "https://dev.example.com/"
                    REVIEW_PATH: "/review"
                  script:
                    - deploy-review-script --url $DEPLOY_SITE --path $REVIEW_PATH
                  environment: production
                ```
                
        - variables 키워드
            - `variables:description`
                - 설명
                    - 파이프라인 수준에서 사용하는 변수에 대한 설명을 정의할 수 있는 키워드
                - 예시
                    
                    ```yaml
                    variables:
                      DEPLOY_NOTE:
                        description: "The deployment note. Explain the reason for this deployment."
                    ```
                    
            - `variables:value`
                - 설명
                    - 파이프라인 수준에서 사용하는 변수의 기본값을 정의하는 키워드
                    - 이 값은 사용자가 수동으로 파이프라인을 실행할 때 기본값으로 채워짐
                    - `variables:description`과 함께 사용할 경우, 변수의 설명과 기본값을 함께 제공할 수 있음
                - 예시
                    
                    ```yaml
                    variables:
                      DEPLOY_ENVIRONMENT:
                        value: "staging"
                        description: "The deployment target. Change this variable to 'canary' or 'production' if needed."
                    ```
                    
            - `variables:options`
                - 설명
                    - 수동으로 파이프라인을 실행할 때 선택할 수 있는 값들의 목록을 제공하는 키워드
                    - 이 키워드는 `variables:value`와 함께 사용되며, 사용자에게 선택 가능한 값을 보여주는 UI 기능을 제공
                - 예시
                    
                    ```yaml
                    # DEPLOY_ENVIRONMENT 변수에 대해 production, staging, canary 중 하나를 선택할 수 있는 옵션을 제공하며, 기본값은 staging
                    # 수동 파이프라인 실행 시, 사용자는 staging, production, canary 중 하나를 선택할 수 있습니다. staging은 기본값으로 설정
                    variables:
                      DEPLOY_ENVIRONMENT:
                        value: "staging"
                        options:
                          - "production"
                          - "staging"
                          - "canary"
                        description: "The deployment target. Set to 'staging' by default."
                    ```
                    
            - `variables:expand`
                - 설명
                    - 변수가 확장 가능한지 여부를 정의하는 키워드
                    - 기본적으로 변수는 확장 가능
                    - `expand: false`로 설정하면 해당 변수는 확장되지 않으며, `$`로 시작하는 변수를 사용할 수 없음
                - 예시
                    
                    ```yaml
                    # VAR2는 VAR1의 값인 value1을 확장하여 value2 value1로 처리
                    # VAR3는 expand: false로 설정되어 있어, value3 $VAR1 그대로 출력
                    # 즉, VAR1은 확장되지 않고 원래 그대로 문자열로 표시 
                    variables:
                      VAR1: value1
                      VAR2: value2 $VAR1
                      VAR3:
                        value: value3 $VAR1
                        expand: false
                    
                    ```
                    
    - **secrets**
        - 설명
            - 민감한 정보나 비밀 값을 보안이 필요한 변수로 관리하기 위해 **`secrets`**를 사용
            - `secrets`는 외부 시크릿 제공자로부터 값을 가져오고 이를 CI/CD 파이프라인의 변수로 사용하게 해줌
            - GitLab에서는 여러 시크릿 제공자와 연동할 수 있으며, 이들은 Vault, GCP Secret Manager, Azure Key Vault, 파일 타입 시크릿, 그리고 인증 토큰을 통해 제공
        - HashiCorp Vault 사용
            - 설명
                - `secrets:vault`는 HashiCorp Vault에서 시크릿을 가져와 CI/CD 변수로 설정하는 방법
                - KV-V2(기본값), KV-V1, 또는 generic 시크릿 엔진을 지원
                - `secrets:token` - 토큰을 사용하여 인증
                    - `secrets:token`은 Vault와 같은 외부 서비스와 연동할 때 인증 토큰을 CI/CD 변수로 지정하여 인증을 처리하는 방법
            - 예시
                
                ```yaml
                job:
                  id_tokens:
                    VAULT_TOKEN:
                      aud: https://vault.example.com  # Vault의 인증 토큰을 지정합니다.
                  
                  secrets:
                    DATABASE_PASSWORD:  # 이 CI/CD 변수에 Vault의 비밀을 저장
                      vault:  # secret: `ops/data/production/db`, field: `password`
                        engine:
                          name: kv-v2
                          path: ops
                        path: production/db
                        field: password
                      token: $VAULT_TOKEN  # `VAULT_TOKEN`을 사용하여 Vault에 인증
                  
                  resource_group: vault-secrets  # 동일 리소스를 공유하는 작업들이 한 번에 하나씩 실행되도록 설정
                  
                  script:
                    - echo "Fetching VAULT_TOKEN..."  # 토큰을 먼저 확인하는 로그 출력
                    - echo "Token: $VAULT_TOKEN"  # 실제 토큰 값을 확인하는 로그 (테스트 목적)
                    - echo "Using secret: $DATABASE_PASSWORD"  # Vault에서 가져온 비밀 값 사용 예시
                ```
                
        - File 에 Secret 저장
            - 설명
                - `secrets:file`을 사용하면 시크릿을 파일 타입 CI/CD 변수로 전달할 수 있음
                - 기본값으로는 시크릿이 파일 타입 변수로 전달되며, 파일 경로를 통해 접근
                - 시크릿을 파일이 아닌 변수로 직접 사용하고 싶다면, `file: false`로 설정할 수 있음
            - 예시
                
                ```yaml
                job:
                  secrets:
                    DATABASE_PASSWORD:
                      vault: production/db/password@ops  # 비밀의 위치 (Vault에서 가져오기)
                      file: true  # 비밀 값을 파일 형태로 저장 (기본값이므로 명시하지 않아도 됩니다)
                  
                  script:
                    - echo "The secret path is: $DATABASE_PASSWORD"  # 비밀 값의 경로 출력
                    - cat $DATABASE_PASSWORD  # 파일을 출력하거나, 파일의 내용을 사용
                
                ---
                job:
                  secrets:
                    DATABASE_PASSWORD:
                      vault: production/db/password@ops
                      file: false  # 파일이 아닌 직접 값으로 설정
                      
                  script:
                    - echo "The secret is: $DATABASE_PASSWORD"  # DATABASE_PASSWORD를 변수로 참조
                ```
                
- 컨테이너 및 이미지
    - **image**
        - 설명
            - `image` 관련 키워드는 GitLab CI/CD에서 Docker 컨테이너 환경을 정의하고, 다양한 이미지 설정을 통해 유연한 CI/CD 파이프라인 구성을 지원
            - GitLab CI/CD에서 Docker 이미지를 지정하여, 해당 이미지 내에서 작업을 실행하는 데 사용
        - 사용법
            - `image`는 작업 또는 default 섹션 내에서 사용
            - 기본적인 사용법은 `<image-name>` 형식으로 이미지를 지정
            - 필요에 따라 `<image-name>:<tag>` 또는 `<image-name>@<digest>` 형식으로 태그 또는 다이제스트를 지정할 수도 있음
            - CI/CD 변수도 지원
        - image 키워드
            - `image:name`
                - 설명
                    - 작업에 사용할 Docker 이미지를 지정
                - 예시
                    
                    ```yaml
                    default:
                      image: ruby:3.0
                    ```
                    
            - `image:entrypoint`
                - 설명
                    - Docker 이미지의 **entrypoint**를 오버라이드
                - 예시
                    
                    ```yaml
                    test-job:
                      image:
                        name: super/sql:experimental
                        entrypoint: [""]
                      script: echo "Hello world"
                    ```
                    
            - `image:docker`
                - 설명
                    - Docker 실행자에 대한 추가 옵션을 설정
                    - 예를 들어, `platform`과 `user`를 지정하여 이미지를 특정 아키텍처로 풀하거나, 컨테이너 내에서 실행할 사용자를 설정할 수 있음
                - 예시
                    
                    ```yaml
                    arm-sql-job:
                      script: echo "Run sql tests"
                      image:
                        name: super/sql:experimental
                        docker:
                          platform: arm64/v8
                          user: dave
                    ```
                    
            - `image:pull_policy`
                - 설명
                    - Docker 이미지 풀 정책을 설정
                    - 설정 가능값
                        - `always`: 항상 이미지를 풀어 사용.
                        - `if-not-present`: 로컬에 이미지가 없으면 이미지를 풀고, 있으면 사용.
                        - `never`: 로컬 이미지만 사용하고, 풀지 않음.
                - 예시
                    
                    ```yaml
                    job1:
                      script: echo "A single pull policy."
                      image:
                        name: ruby:3.0
                        pull_policy: if-not-present
                    
                    job2:
                      script: echo "Multiple pull policies."
                      image:
                        name: ruby:3.0
                        pull_policy: [always, if-not-present]
                    ```
                    
    - **services**
        - 설명
            - GitLab CI/CD에서 추가적인 Docker 이미지를 지정하는 데 사용
            - 여러 컨테이너가 동시에 실행될 때 서로 통신할 수 있도록 서비스 간 연결이 가능
            - 이 추가적인 서비스 이미지는 기본 이미지(`image`)와 함께 실행되어, 작업을 실행하는 동안 필요한 외부 서비스(예: 데이터베이스, 캐시 서버 등)를 제공할 수 있음
            - 서비스는 별칭(alias)을 사용하여 연결할 수 있음
            - 이를 통해 여러 컨테이너가 동시에 실행되며, 서로 간의 통신이 가능
                
                ```yaml
                # Ruby 이미지를 기본 이미지로 지정하고, PostgreSQL 이미지를 서비스로 추가 
                # PostgreSQL 서비스는 db-postgres라는 별칭을 사용하여 Ruby 컨테이너 내에서 이를 참조할 수 있음
                default:
                  image:
                    name: ruby:2.6  # 기본 Ruby 이미지를 지정
                    entrypoint: ["/bin/bash"]  # entrypoint 설정
                
                  services:
                    - name: my-postgres:11.7  # PostgreSQL 서비스 지정
                      alias: db-postgres  # 이 서비스를 db-postgres라는 별칭으로 사용
                      entrypoint: ["/usr/local/bin/db-postgres"]  # 서비스의 entrypoint 설정
                      command: ["start"]  # PostgreSQL 서비스 실행 명령
                
                  before_script:
                    - bundle install  # 기본 스크립트 실행 전에 필요한 패키지 설치
                
                test:
                  script:
                    - bundle exec rake spec  # 테스트 실행
                ```
                
        - `services:docker`
            - `services:docker`
                - 설명
                    - `services:docker`는 Docker 실행기를 사용할 때 추가적인 Docker 옵션을 제공하는 설정
                    - 이를 사용하여 Docker 컨테이너의 플랫폼(예: ARM 아키텍처 등)이나 사용자를 지정할 수 있음
                - 예시
                    
                    ```yaml
                    # arm64/v8 플랫폼을 지정하고, dave 사용자로 컨테이너를 실행
                    arm-sql-job:
                      script: echo "Run sql tests in service container"
                      image: ruby:2.6
                      services:
                        - name: super/sql:experimental  # SQL 서비스를 실행
                          docker:
                            platform: arm64/v8  # ARM 아키텍처용 이미지
                            user: dave  # 'dave' 사용자로 실행
                    ```
                    
        - `services:pull_policy`
            - 설명
                - `services:pull_policy`는 GitLab Runner가 Docker 이미지를 어떻게 가져올지를 설정하는 옵션
                - 세 가지 정책을 사용
                    - always: 항상 이미지를 새로 가져옴
                    - if-not-present: 이미지가 로컬에 없으면 가져옴
                    - never: 이미지를 가져오지 않음
            - 예시
                
                ```yaml
                job1:
                  script: echo "A single pull policy."
                  services:
                    - name: postgres:11.6
                      pull_policy: if-not-present  # 이미지가 로컬에 없으면 가져옴
                
                job2:
                  script: echo "Multiple pull policies."
                  services:
                    - name: postgres:11.6
                      pull_policy: [always, if-not-present]  # 여러 정책 지정
                ```
                
- 환경 및 배포 관련
    - **environment**
        - 설명
            - GitLab CI/CD 파이프라인에서 배포 작업이 어느 환경에 배포되는지를 정의하는 데 사용
            - 특정 작업이 배포할 환경을 설정할 수 있으며, 다양한 환경 관리 옵션을 제공
            - 이 키워드는 `배포 작업`에만 사용할 수 있음
        - environment 키워드
            - `environment:name`
                - 설명
                    - 배포 대상 환경의 이름을 지정
                    - 환경 이름은 텍스트(예: `production`, `staging`)나 CI/CD 변수를 사용할 수 있음
                    - 지정한 이름의 환경이 이미 존재하지 않으면, 지정된 이름으로 자동 생성
                - 예시
                    
                    ```yaml
                    deploy to production:
                      stage: deploy
                      script: git push production HEAD:main
                      environment:
                        name: production
                    ```
                    
            - `environment:url`
                - 설명
                    - 배포된 환경에 대한 URL을 지정
                    - URL은 고정된 텍스트나 CI/CD 변수를 사용할 수 있음
                    - 배포가 완료되면, 머지 리퀘스트 또는 배포 페이지에서 URL을 확인할 수 있는 버튼이 표시됨
                - 예시
                    
                    ```yaml
                    deploy to production:
                      stage: deploy
                      script: git push production HEAD:main
                      environment:
                        name: production
                        url: https://prod.example.com
                    ```
                    
            - `environemnt:on_stop`
                - 설명
                    - GitLab CI/CD에서 환경을 종료(stop) 하는 작업을 지정
                    - 특정 환경에서 배포 완료 후 해당 환경을 종료할 때, `on_stop`을 사용하여 이를 처리
                - 주요기능
                    - `on_stop`은 배포 환경을 종료하는 작업을 연결하는 데 사용
                    - `on_stop`은 종료 작업을 정의할 수 있으며, 특정 환경을 종료하는 데 유용
                    - `on_stop`을 사용하면 특정 환경이 완료된 후 다른 작업을 통해 해당 환경을 종료
                - 예시
                    
                    ```yaml
                    stages:
                      - deploy
                    
                    # on_stop 을 통해서 stop_test 라는 job 에서 환경 종료 작업을 진행함을 알려줌
                    deploy_test:
                      image: #myimage
                      stage: deploy
                      environment:
                        name: test
                        url: #myurl
                        **on_stop: stop_test**
                      rules:
                        - if: '$CI_COMMIT_BRANCH == "mybranch" && $CI_PIPELINE_SOURCE != "merge_request_event"'
                      script: #myscript
                    
                    # deploy_test 가 끝나면 트리거 되며 action: stop 을 통해 test 환경이 종료됨
                    stop_test:
                      stage: deploy
                      variables:
                        GIT_STRATEGY: none
                      script: #myscript     
                      when: manual
                      environment:
                        name: test
                        **action: stop**
                    
                    ```
                    
            - `environment:action`
                - 설명
                    - GitLab CI/CD에서 **환경과의 상호작용 방식**을 정의하는 데 사용
                    - 이 키워드는 배포 작업에서 **환경의 상태를 어떻게 변경할지** 지정
                    - 각 작업이 환경과 어떻게 상호작용할지에 따라 다섯 가지 액션을 설정할 수 있음
                - 가능한 액션
                    
                    ```yaml
                    stages:
                      - deploy
                      - test
                    
                    # 1. 환경 시작 (배포)
                    deploy_app:
                      stage: deploy
                      script: make deploy-app
                      environment:
                        name: production
                        action: start  # 환경 시작
                    
                    # 2. 환경 준비 (배포하지 않고 준비만)
                    prepare_environment:
                      stage: deploy
                      script: make prepare-environment
                      environment:
                        name: staging
                        action: prepare  # 배포 없이 환경 준비만
                    
                    # 3. 환경 종료 (리뷰 앱 종료)
                    stop_app:
                      stage: deploy
                      script: make stop-app
                      environment:
                        name: review/$CI_COMMIT_REF_SLUG
                        action: stop  # 리뷰 앱 환경 종료
                    
                    # 4. 환경 검증 (배포 없이 검증만)
                    verify_deployment:
                      stage: test
                      script: make verify-deployment
                      environment:
                        name: production
                        action: verify  # 환경 검증
                    
                    # 5. 환경 접근 (배포 없이 환경에 접근)
                    access_production:
                      stage: deploy
                      script: make access-production
                      environment:
                        name: production
                        action: access  # 환경에 접근만
                    ```
                    
                    - `start`
                        - 설명
                            - 환경을 **시작**하는 작업
                            - 이 액션을 사용할 때, 환경은 **배포가 시작되는 시점**에 생성
                            - 이 작업이 실행된 후에 실제 배포가 이루어짐
                            - 예를 들어, 새로운 프로덕션 서버에 애플리케이션을 배포하는 경우, `start` 액션을 사용하여 환경을 시작하고 배포를 실행
                        - 예시
                            
                            ```yaml
                            deploy_to_production:
                              stage: deploy
                              script: make deploy-production
                              environment:
                                name: production
                                action: start
                            ```
                            
                    - `prepare`
                        - 설명
                            - 환경을 준비하는 작업
                            - 이 작업은 배포를 트리거하지 않음
                            - 환경을 설정하거나 필요한 리소스를 준비하는 데 사용
                            - 예를 들어, 환경이 준비될 때 소프트웨어의 의존성을 설치하거나, 특정 환경 변수들을 설정하는 작업을 수행 할 수 있음
                        - 예시
                            
                            ```yaml
                            prepare_environment:
                              stage: deploy
                              script: make prepare-environment
                              environment:
                                name: staging
                                action: prepare
                            ```
                            
                    - `stop`
                        - 설명
                            - 환경을 중지하는 작업
                            - 배포가 완료되었거나, 해당 환경을 더 이상 사용하지 않을 때 환경을 종료하는 데 사용
                            - `on_stop` 키워드와 함께 사용하여 다른 작업에서 환경을 종료하도록 할 수도 있음
                        - 예시
                            
                            ```yaml
                            stages:
                              - deploy
                            
                            # on_stop 을 통해서 stop_test 라는 job 에서 환경 종료 작업을 진행함을 알려줌
                            deploy_test:
                              image: #myimage
                              stage: deploy
                              environment:
                                name: test
                                url: #myurl
                                on_stop: stop_test
                              rules:
                                - if: '$CI_COMMIT_BRANCH == "mybranch" && $CI_PIPELINE_SOURCE != "merge_request_event"'
                              script: #myscript
                            
                            # deploy_test 가 끝나면 트리거 되며 action: stop 을 통해 test 환경이 종료됨
                            stop_test:
                              stage: deploy
                              variables:
                                GIT_STRATEGY: none
                              script: #myscript     
                              when: manual
                              environment:
                                name: test
                                action: stop
                            
                            ```
                            
                    - `verify`
                        - 설명
                            - 환경을 **검증**하는 작업
                            - 이 액션은 배포를 트리거하지 않고, 환경이 예상대로 설정되었는지 확인하는 데 사용
                            - 예를 들어, 애플리케이션이 제대로 작동하는지, 설정 파일이 올바르게 적용되었는지 점검하는 작업을  수행
                        - 예시
                            
                            ```yaml
                            verify_deployment:
                              stage: test
                              script: make verify-deployment
                              environment:
                                name: production
                                action: verify
                            ```
                            
                    - `access`
                        - 설명
                            - 환경을 접속하는 작업
                            - 이 액션은 환경에 접속하여 환경의 상태를 점검하거나 로그를 조회하는 데 사용
                            - 실제 배포나 환경 변경 작업을 수행하지 않고, 단순히 환경에 대한 액세스만 수행
                        - 예시
                            
                            ```yaml
                            access_review_app:
                              stage: deploy
                              script: make access-review-app
                              environment:
                                name: review/$CI_COMMIT_REF_SLUG
                                action: access
                            ```
                            
            - `environment:auto_stop_in`
                - 설명
                    - GitLab CI/CD에서 환경의 자동 종료 시간을 설정하는 데 사용
                    - 이 설정을 통해 특정 환경이 정해진 시간 후에 자동으로 종료되도록 할 수 있음
                    - 이렇게 하면 환경을 수동으로 종료하지 않아도, 지정된 시간이 지나면 GitLab이 자동으로 해당 환경을 정리
                    - 환경을 배포할 때마다 `auto_stop_in` 시간은 리셋
                - 주요 기능
                    - 기능: `auto_stop_in`을 사용하여 환경의 수명을 설정하고, 지정된 시간이 경과하면 GitLab이 자동으로 해당 환경을 중지
                    - 입력값: 자연어로 된 시간(예: "1 day", "7 days", "168 hours", "never") 또는 CI/CD 변수를 사용할 수 있음
                    - 사용 위치: 이 키워드는 Job 내에서만 사용할 수 있음
                    - 지원되는 시간 포맷:
                        - `168 hours` (168시간)
                        - `7 days` (7일)
                        - `1 week` (1주일)
                        - `never` (자동 종료하지 않음)
                        - CI/CD 변수도 사용 가능
                - 예시
                    
                    ```yaml
                    review_app:
                      script: deploy-review-app
                      environment:
                        name: review/$CI_COMMIT_REF_SLUG
                        auto_stop_in: 1 day
                    ```
                    
            - `environment:kubernetes`
                - 설명
                    - **Kubernetes 환경**에 대한 **대시보드 설정**을 구성하는 데 사용
                    - GitLab CI/CD 파이프라인 내에서 Kubernetes 클러스터와의 상호작용을 설정
                - 주요 기능
                    - Kubernetes 대시보드 설정: GitLab이 Kubernetes 환경과 연동될 수 있도록 설정
                    - 필요한 입력값:
                        - agent: Kubernetes 클러스터와 연동하기 위한 GitLab 에이전트 경로. 형식은 `path/to/agent/project:agent-name`
                        - namespace: Kubernetes의 네임스페이스를 설정합니다. `agent`와 함께 사용해야 함
                        - flux_resource_path: Flux 리소스의 전체 경로를 지정합니다. `agent`와 `namespace`와 함께 설정
                - 예시
                    
                    ```yaml
                    # 이 설정은 production 환경에 GitLab 에이전트를 사용해 Kubernetes 환경을 설정하고, 해당 환경에 대한 대시보드를 제공
                    # 이 대시보드를 사용하려면 GitLab 에이전트 for Kubernetes를 설치하고, 환경 프로젝트나 부모 그룹에 대한 user_access 설정이 필요
                    # 권한 설정
                    #   작업을 실행하는 사용자가 클러스터 에이전트에 접근할 수 있어야하며 그렇지 않으면 agent, namespace, flux_resource_path 설정은 무시
                    deploy:
                      stage: deploy
                      script: make deploy-app
                      environment:
                        name: production
                        kubernetes:
                          agent: path/to/agent/project:agent-name
                          namespace: my-namespace
                          flux_resource_path: helm.toolkit.fluxcd.io/v2/namespaces/gitlab-agent/helmreleases/gitlab-agent
                    ```
                    
            - `environment:deployment_tier`
                - 설명
                    - 배포 환경의 수준을 지정하는 데 사용
                    - 이 설정은 배포된 환경이 어떤 수준에 속하는지(예: 프로덕션, 스테이징 등)를 정의
                - 배포 환경 수준 설정
                    - production: 프로덕션 환경
                    - staging: 스테이징 환경
                    - testing: 테스트 환경
                    - development: 개발 환경
                    - other: 기타 환경
                - 예시
                    
                    ```yaml
                    # 이 예시는 customer-portal 환경을 프로덕션 환경으로 지정
                    deploy:
                      script: echo
                      environment:
                        name: customer-portal
                        deployment_tier: production
                    ```
                    
    - **pages**
        - 설명
            - `pages`는 파이프라인에서 정적 웹 콘텐츠를 GitLab Pages로 배포하는 작업을 정의하는 키워드
            - 필수 조건
                - `artifacts`로 콘텐츠 디렉토리를 정의해야 하며, 기본적으로 `public` 디렉토리가 사용
                - `publish`를 사용하여 다른 콘텐츠 디렉토리를 정의할 수 있음
        - pages 키워드
            - `pages`
                - 예시
                    
                    ```yaml
                    # 기본적인 pages 작업
                    # my-html-content 디렉토리의 콘텐츠를 public 디렉토리로 옮긴 후, public을 GitLab Pages에 배포
                    # rules를 사용해 기본 브랜치(CI_DEFAULT_BRANCH)에서만 배포하도록 설정
                    pages:
                      stage: deploy
                      script:
                        - mv my-html-content public  # my-html-content 디렉토리를 public으로 변경
                      artifacts:
                        paths:
                          - public  # public 디렉토리를 아티팩트로 정의
                      rules:
                        - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH  # 기본 브랜치일 때만 배포
                      environment: production  # 환경은 production으로 설정
                    ```
                    
            - `pages:publish`
                - 설명
                    - GitLab Pages에서 사용할 콘텐츠 디렉토리를 설정하는 키워드
                    - 디폴트로`public` 디렉토리가 사용되며, 이 키워드를 사용하여 다른 디렉토리를 지정할 수 있음
                - 예시
                    
                    ```yaml
                    pages:
                      stage: deploy
                      script:
                        - npx @11ty/eleventy --input=path/to/eleventy/root --output=dist  # Eleventy로 웹사이트 생성
                      artifacts:
                        paths:
                          - dist  # dist 디렉토리 아티팩트로 정의
                      publish: dist  # dist 디렉토리를 GitLab Pages로 배포
                      rules:
                        - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH  # 기본 브랜치일 때만 배포
                      environment: production  # 환경은 production으로 설정
                    
                    ```
                    
            - `pages:pages.path_prefix`
                - 설명
                    - GitLab Pages의 병렬 배포를 위한 기능으로, 배포된 콘텐츠의 경로 접두사(path prefix)를 설정
                    - 브랜치별로 다른 경로를 설정하여, 동일한 프로젝트에서 각 브랜치마다 별도의 Pages를 생성할 수 있음
                - 예시
                    
                    ```yaml
                    
                    pages:
                      stage: deploy
                      script:
                        - echo "Pages accessible through ${CI_PAGES_URL}/${CI_COMMIT_BRANCH}"
                      pages:
                        path_prefix: "$CI_COMMIT_BRANCH"  # 각 브랜치에 대해 다른 경로 접두사 설정
                      artifacts:
                        paths:
                          - public  # public 디렉토리를 GitLab Pages에 배포
                    ```
                    
            - `pages:pages.expire_in`
                - 설명
                    - GitLab Pages로 배포된 페이지의 만료 기간을 설정하는 키워드
                    - 만료 기간을 지정하면, 해당 배포는 주기적인 cron 작업에 의해 자동으로 비활성화
                    - 기본적으로 GitLab Pages 에서 배포된 웹사이트는 일정 기간이 지나면 **자동으로 만료**
                        - 디폴트로 이 값이 `expire` 로 지정되기 때문
                    - 배포가 만료되지 않도록 하려면 `never`를 설정할 수 있음
                - 예시
                    
                    ```yaml
                    pages:
                      stage: deploy
                      script:
                        - echo "Pages accessible through ${CI_PAGES_URL}"
                      pages:
                        expire_in: 1 week  # 배포된 페이지는 1주일 후 만료됨
                      artifacts:
                        paths:
                          - public  # public 디렉토리를 아티팩트로 정의
                    ```
                    
- 릴리즈
    - **release**
        - 설명
            - 목적
                - `release`는 GitLab에서 릴리스를 생성하는 작업을 정의하는 키워드
                - 릴리스를 만들기 위해서는 `release-cli`를 사용해야 하며, 이는 `$PATH`에 포함되어 있어야 함
            - 사용법
                - `release`는 GitLab의 CI/CD 파이프라인에서 릴리스를 생성할 때 사용
                - 릴리스를 생성하는 작업은 태그가 푸시될 때 실행
                - Docker 실행기를 사용할 경우 `release-cli`는 GitLab 컨테이너 레지스트리에서 제공하는 이미지를 통해 사용할 수 있음
                    - [registry.gitlab.com/gitlab-org/release-cli:latest](http://registry.gitlab.com/gitlab-org/release-cli:latest)
                - Shell 실행기를 사용할 경우, `release-cli`는 실행기 서버에 설치되어 있어야 함
            - 필수구성
                - 태그가 생성될 때 실행되어야 하므로, `rules` 또는 `only/except` 키워드를 사용해 특정 조건에서만 실행되도록 설정할 수 있음
                - 릴리스 작업은 `script`와 함께 사용해야 하며, `release`는 `script` 실행 후, `after_script` 실행 전에 수행
            - 가능한 입력 값
                - `tag_name`: 릴리스의 태그 이름을 지정. 보통 Git 태그를 사용
                - `tag_message`: (선택 사항) 태그에 대한 메시지를 지정
                - `name`: (선택 사항) 릴리스의 이름을 지정
                - `description`: 릴리스에 대한 설명을 지정
                - `ref`: (선택 사항) 릴리스가 기준으로 하는 Git ref를 지정
                - `milestones`: (선택 사항) 릴리스와 관련된 마일스톤을 지정
                - `released_at`: (선택 사항) 릴리스의 생성 시간을 지정
                - `assets:links`: (선택 사항) 릴리스에 대한 추가 자산 링크를 지정
            - 추가사항
                - `script` 필수
                    - 모든 릴리스 작업은 `script` 키워드를 포함해야 함
                    - 이 필드는 릴리스가 실행되기 전에 필요한 스크립트 명령을 정의
                    - 예를 들어, 릴리스 작업이 성공적으로 실행되면 그 후에 `release`가 실행
                    - `script`가 필요 없으면 기본적인 실행 명령어를 포함하는 `script: echo "release job"`을 추가할 수 있음
                - 기존 릴리스가 존재하면 실패
                    - 동일한 태그 이름에 대해 릴리스를 생성하려고 할 경우, 이미 릴리스가 존재하면 해당 릴리스는 업데이트되지 않으며 릴리스 생성 작업이 실패
                    - 이는 GitLab의 기본 동작
                - 릴리스 생성 조건
                    - 릴리스는 `script` 명령어가 성공적으로 실행된 후에 생성
                        - `release` 작업은 `script`가 성공적으로 실행되어야만 실행
                - 태그 생성 시 실행
                    - 릴리스 작업은 Git 태그가 생성되었을 때만 실행되며, GitLab UI에서 태그를 수동으로 생성할 때도 릴리스가 생성
                - 다수의 릴리스 생성
                    - 한 파이프라인 내에서 여러 릴리스를 생성할 수 있음
                    - 예를 들어, 여러 태그를 다루는 경우, 각 태그에 대해 별도의 릴리스를 생성할 수 있음
                - Custom SSL CA 인증서
                    - `release-cli`를 사용하여 릴리스를 생성할 때, GitLab의 SSL 인증서 또는 커스텀 SSL 인증서를 설정할 수 있는 옵션이 있음
            - 예시
                
                ```yaml
                # 기본 릴리즈 작업
                #   이 예시에서는 Git 태그가 푸시될 때 릴리스 작업이 실행
                #   $CI_COMMIT_TAG는 현재 커밋에 연결된 Git 태그
                #   이를 사용하여 릴리스의 태그 이름, 릴리스 이름, 설명 등을 설정
                release_job:
                  stage: release
                  image: registry.gitlab.com/gitlab-org/release-cli:latest
                  rules:
                    - if: $CI_COMMIT_TAG  # 태그가 생성될 때 실행
                  script:
                    - echo "Running the release job."
                  release:
                    tag_name: $CI_COMMIT_TAG
                    name: 'Release $CI_COMMIT_TAG'
                    description: 'Release created using the release-cli.'
                
                ```
                
        - release 키워드
            - `release:tag_name`
                - 설명
                    - 릴리스를 생성할 때 사용되는 Git 태그를 지정하는 키워드
                    - `tag_name`에 CI/CD 변수를 사용하면, 릴리스가 자동화되어 버전 관리나 태그 생성 과정을 효율적으로 처리할 수 있음
                - 필수 항목
                    - `release:tag_name`은 반드시 제공되어야 하며, 이는 릴리스가 생성될 때 사용될 Git 태그의 이름을 정의
                - 태그 미존재 시 자동 생성
                    - 지정한 태그가 Git 프로젝트 내에 존재하지 않는 경우, 이 태그는 릴리스와 함께 생성됩니다. 새로 생성된 태그는 파이프라인에 연결된 커밋의 SHA를 사용
                - 가능한 입력
                    - 태그 이름을 지정할 수 있음
                    - 이 태그 이름은 CI/CD 변수를 사용하여 동적으로 생성할 수도 있음
                - 예시
                    
                    ```yaml
                    # 새로운 태그가 푸시될 때, 해당 태그에 대한 릴리스를 생성하는 작업
                    #   $CI_COMMIT_TAG는 GitLab CI에서 자동으로 제공되는 CI/CD 변수로, 현재 커밋에 해당하는 태그 이름을 제공
                    #   이 작업은 새로운 태그가 추가될 때만 실행되며, 새로 생성된 태그에 대해 릴리스를 만듦
                    job:
                      script: echo "Running the release job for the new tag."
                      release:
                        tag_name: $CI_COMMIT_TAG  # 새로 푸시된 Git 태그
                        description: 'Release description'
                      rules:
                        - if: $CI_COMMIT_TAG  # 태그가 생성된 경우에만 실행
                    ---
                    # 릴리스와 새 태그를 동시에 생성
                    #   새로운 태그를 생성하고 해당 태그로 릴리스를 생성하는 작업
                    #   이 경우, 새 태그는 세미버전 규칙을 따르는 형태로 생성
                    job:
                      script: echo "Running the release job and creating a new tag."
                      release:
                        tag_name: ${MAJOR}_${MINOR}_${REVISION}  # 세미버전 형식의 새 태그
                        description: 'Release description'
                      rules:
                        - if: $CI_PIPELINE_SOURCE == "schedule"  # 스케줄된 파이프라인에서만 실행
                    
                    ```
                    
            - `release:tag_message`
                - 설명
                    - Git 태그에 메시지를 추가하는 키워드
                    - 이 메시지는 태그가 새로 생성될 때 사용되며, 태그가 존재하지 않을 경우 새로 생성되는 태그에 주석 메시지를 달 수 있음
                    - 이 옵션을 사용하지 않으면, 기본적으로 가벼운 태그(lightweight tag) 가 생성
                        - 가벼운 태그(lightweight tag) 는 기본적으로 커밋을 가리키는 포인터만을 포함하고 있으며, 메시지가 포함되지 않음
                - 예시
                    
                    ```yaml
                    release_job:
                      stage: release
                      release:
                        tag_name: $CI_COMMIT_TAG  # 새 태그 이름
                        description: 'Release description'
                        tag_message: 'Annotated tag message'  # 태그에 추가될 메시지
                    ```
                    
            - `release:name`
                - 설명
                    - 릴리스의 이름을 지정
                    - 릴리스를 생성할 때 필수
                    - 이 옵션을 생략하면, 기본적으로 릴리스 이름은 `release:tag_name`에서 지정된 태그 이름으로 자동 채워짐
                - 예시
                    
                    ```yaml
                    release_job:
                      stage: release
                      release:
                        name: 'Release $CI_COMMIT_TAG'  # 릴리스 이름에 커밋 태그 사용
                        tag_name: $CI_COMMIT_TAG  # 태그 이름도 커밋 태그로 설정
                        description: 'Release description'
                    ```
                    
            - `release:description`
                - 설명
                    - 릴리스의 상세 설명을 정의하는 키워드
                - 입력값
                    - 문자열
                        - 설명을 직접 문자열로 입력할 수 있음
                    - 파일 경로
                        - 설명을 포함하는 파일 경로를 지정할 수도 있음
                        - 이 파일은 프로젝트 디렉토리 내에 위치해야 하며, 상대 경로를 사용해야 함
                        - 또한, 파일은 심볼릭 링크로 지정할 수도 있으며, 이 링크 역시 프로젝트 디렉토리 내에 존재해야 함
                - 제약사항
                    - 파일 경로 또는 파일명에 공백이 포함될 수 없음
                    - 일부 쉘에서는 CI/CD 변수나 특수 문자를 사용할 때 별도의 구문이나 이스케이프 처리가 필요할 수 있음
                - 예시
                    
                    ```yaml
                    job:
                      release:
                        tag_name: ${MAJOR}_${MINOR}_${REVISION}  # 새로운 태그 생성
                        description: './path/to/CHANGELOG.md'  # 파일로부터 설명을 로드
                    ---
                    release_job:
                      stage: release
                      release:
                        tag_name: $CI_COMMIT_TAG
                        description: 'This release includes new features and bug fixes.'
                    
                    ```
                    
            - `release:ref`
                - 설명
                    - 릴리스를 만들 때, 해당 릴리스가 참조할 커밋, 태그 또는 브랜치를 지정하는 키워드
                    - 이 값은 릴리스 태그가 아직 존재하지 않거나, 특정 커밋 또는 브랜치를 참조하고자 할 때 사용
                - 입력값
                    - 커밋 SHA, 다른 태그 이름, 또는 브랜치 이름을 입력할 수 있음
                    - 이 값은 릴리스가 참조할 리소스를 지정
                    - 릴리스의 참조가 특정 커밋이나 브랜치를 가리키도록 할 때 사용
                - 예시
                    
                    ```yaml
                    # v1.0.0 태그로 릴리스를 만들고, master 브랜치에 해당하는 커밋을 참조
                    release_job:
                      stage: release
                      release:
                        tag_name: v1.0.0
                        ref: 'master'  # master 브랜치를 참조
                    ```
                    
            - `release:milestones`
                - 설명
                    - 릴리스가 연관된 마일스톤의 제목을 정의하는 키워드
                    - 프로젝트에서 설정된 마일스톤을 릴리스와 연결할 수 있음
                - 입력값
                    - 마일스톤 제목을 문자열로 입력
                - 예시
                    
                    ```yaml
                    # v1.0.0 릴리스가 v1.0 Launch와 Bug Fixes 마일스톤과 연관됨
                    release_job:
                      stage: release
                      release:
                        tag_name: v1.0.0
                        milestones:
                          - 'v1.0 Launch'
                          - 'Bug Fixes'
                    ```
                    
            - `release:released_at`
                - 설명
                    - 릴리스가 준비된 날짜와 시간을 지정하는 키워드
                    - 기본적으로는 릴리스가 생성된 시점의 날짜와 시간이 사용
                - 입력값
                    - ISO 8601 형식으로 날짜와 시간을 입력
                    - 예를 들어 `'2021-03-15T08:00:00Z'`와 같은 형식으로 작성
                - 예시
                    
                    ```yaml
                    # 릴리스가 2021-03-15T08:00:00Z 시간에 준비되었음을 지정
                    release_job:
                      stage: release
                      release:
                        tag_name: v1.0.0
                        released_at: '2021-03-15T08:00:00Z'
                    ```
                    
            - `release:assets:links`
                - 설명
                    - 릴리스에 첨부할 자산 링크를 정의하는 키워드
                    - 이 기능은 `release-cli` 버전 v0.4.0 이상에서 사용할 수 있음
                - 입력값
                    - 자산의 이름(`name`), URL(`url`), 선택적으로 파일 경로(`filepath`)와 링크 유형(`link_type`)을 포함한 목록을 입력
                - 예시
                    
                    ```yaml
                    release_job:
                      stage: release
                      release:
                        tag_name: v1.0.0
                        assets:
                          links:
                            - name: 'asset1'
                              url: 'https://example.com/assets/1'
                            - name: 'asset2'
                              url: 'https://example.com/assets/2'
                              filepath: '/pretty/url/1'  # 선택적
                              link_type: 'other'  # 선택적
                    ```
                    
- 리소스
    - **resource_group**
        - 설명
            - GitLab CI/CD에서 리소스 그룹을 정의하는 키워드
            - 같은 프로젝트 내에서 서로 다른 파이프라인에 대해 특정 잡(jobs)이 상호 배타적으로 실행되도록 보장
            - 여러 개의 동일한 리소스 그룹에 속한 잡들이 동시에 큐에 들어가게 되면, 하나의 잡만 실행되고 나머지 잡들은 리소스 그룹이 비워질 때까지 대기
            - 이를 통해, 리소스나 환경을 공유하는 여러 작업들 간의 동시 실행을 제어
        - 동작 원리
            - 리소스 그룹은 세마포어(semaphore)와 비슷한 개념으로, 하나의 리소스를 동시에 여러 작업이 접근하는 것을 방지
            - 동일한 리소스 그룹에 속한 작업들이 동시에 실행되지 않도록 보장
            - 예를 들어, 프로덕션 환경에 대한 배포 작업이 여러 파이프라인에서 동시에 발생하는 것을 막을 수 있음
            - 리소스 그룹은 배포와 같은 특정 작업에 대해 유용하게 사용될 수 있으며, 동시 배포를 방지하고자 할 때 유용
        - 프로세스 모드
            - 리소스 그룹의 기본 프로세스 모드는 unordered(순서 없음)
            - 즉, 리소스 그룹에 속한 잡은 순서에 상관없이 실행
            - **순서 지정**이 필요한 경우, 리소스 그룹의 프로세스 모드를 변경하려면 API를 통해 수정 요청을 할 수 있음
            - 이렇게 하면 리소스 그룹에 포함된 잡들이 순차적으로 실행되도록 설정할 수 있음
        - 예시
            
            ```yaml
            # 프로덕션 환경 배포
            #   deploy-to-production-1, deploy-to-production-2 배포 작업을 production이라는 리소스 그룹에 연결
            #   두 작업은 병렬 실행되지만 동일한 리소스 그룹을 사용하기 때문에
            #   한 번에 하나의 파이프라인만 production 리소스를 사용할 수 있게되어 순차 실행됨
            
            ---
            # 첫 번째 파이프라인
            deploy-to-production-pipeline-1:
              script: deploy
              resource_group: production
            
            ---
            # 두 번째 파이프라인
            deploy-to-production-pipeline-2:
              script: deploy
              resource_group: production
            ```
            
- 파이프라인
    - **needs**
        - 설명
            - `needs`는 파이프라인 내에서 작업 간 의존 관계를 정의하는 중요한 키워드
            - `needs`를 사용하면 작업을 순서대로 실행하는 대신, 특정 작업이 완료된 후에만 다른 작업을 실행하도록 설정할 수 있음
            - 이를 통해 작업들을 병렬로 실행하고, 실행 순서를 더욱 세밀하게 조정
        - needs 키워드
            - `needs`
                - 기본 사용법
                    
                    ```yaml
                    # Linux와 macOS 빌드는 서로 독립적으로 실행
                    linux:build:
                      stage: build
                      script: echo "Building linux..."
                    
                    mac:build:
                      stage: build
                      script: echo "Building mac..."
                    
                    # Lint 작업은 needs: []로 설정되어 있어 빌드 작업과 관계 없이 즉시 실행
                    lint:
                      stage: test
                      needs: []  # lint는 바로 시작됨
                      script: echo "Linting..."
                    
                    # 각 플랫폼의 rspec 작업은 해당 빌드가 끝난 후에 실행
                    linux:rspec:
                      stage: test
                      needs: ["linux:build"]
                      script: echo "Running rspec on linux..."
                    
                    mac:rspec:
                      stage: test
                      needs: ["mac:build"]
                      script: echo "Running rspec on mac..."
                    
                    # Production 작업은 모든 이전 작업이 완료된 후 실행
                    production:
                      stage: deploy
                      script: echo "Running production..."
                      environment: production
                    ```
                    
            - `needs:artifacts`
                - 설명
                    - `needs`를 사용할 때 기본적으로 이전 작업의 아티팩트를 다운로드하지 않지만, `needs:artifacts`를 사용하여 필요한 아티팩트를 다운로드할 수 있음
                - 예시
                    
                    ```yaml
                    test-job1:
                      stage: test
                      needs:
                        - job: build_job1
                          artifacts: true  # build_job1의 아티팩트를 다운로드
                    
                    test-job2:
                      stage: test
                      needs:
                        - job: build_job2
                          artifacts: false  # build_job2의 아티팩트를 다운로드하지 않음
                    ```
                    
            - `needs:project`
                - 설명
                    - 다른 프로젝트에서 성공적인 작업의 아티팩트를 가져오는 데 사용
                - 예시
                    
                    ```yaml
                    build_job:
                      stage: build
                      script:
                        - ls -lhR
                      needs:
                        - project: namespace/group/project-name
                          job: build-1
                          ref: main
                          artifacts: true
                    ```
                    
            - `needs:optional`
                - 설명
                    - GitLab CI/CD에서 파이프라인 작업 간 의존성을 설정할 때 유용한 기능으로, 선택적인 작업을 정의하는 데 사용
                    - `optional: true`를 설정하면, 해당 작업이 파이프라인에 존재할 수도 있고 존재하지 않을 수도 있는 경우에 대해 GitLab이 자동으로 처리
                    - `optional: false`가 기본값이며 이 경우, 해당 작업이 파이프라인에 없으면 파이프라인이 실패
                    - `optional: true`를 사용하면, 그 작업이 파이프라인에 없을 때 파이프라인이 실패하지 않고 나머지 작업들이 진행되며, 작업이 존재할 경우에는 완료를 기다림
                - 예시
                    
                    ```yaml
                    build-job:
                      stage: build
                      script: echo "Building project..."
                    
                    test-job1:
                      stage: test
                      script: echo "Running test-job1..."
                    
                    # test-job2 는 $CI_COMMIT_BRANCH가 기본 브랜치인 경우에만 실행
                    test-job2:
                      stage: test
                      rules:
                        - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
                      script: echo "Running test-job2..."
                    
                    # deploy-job은 test-job1만 기다리고, test-job2가 존재하지 않더라도 실패하지 않고 실행
                    deploy-job:
                      stage: deploy
                      needs:
                        - job: test-job2
                          optional: true  # test-job2가 없을 경우에도 실패하지 않음
                        - job: test-job1
                      environment: production
                      script: echo "Deploying to production..."
                    
                    # review-job은 test-job2가 존재하지 않으면 즉시 실행
                    review-job:
                      stage: deploy
                      needs:
                        - job: test-job2
                          optional: true  # test-job2가 없으면 바로 실행
                      environment: review
                      script: echo "Deploying to review environment..."
                    ```
                    
            - `needs:pipeline`
                - 설명
                    - `needs:pipeline`은 업스트림(상위) 파이프라인의 상태를 미러링(복제)하는 기능을 제공
                    - 특정 파이프라인의 상태를 가져와 현재 작업에 반영
                    - 다른 프로젝트 또는 파이프라인에서 실행된 파이프라인의 상태를 사용하여 현재 파이프라인의 작업을 진행하는 데 필요한 정보를 가져올 수 있음
                    - **업스트림(상위) 파이프라인**이 성공적으로 완료되었는지, 실패했는지 등의 상태 정보를 사용하는 데 유용
                - 구성요소
                    - `pipeline`
                        - 업스트림 파이프라인의 ID를 지정
                        - 이는 다른 프로젝트에서 실행된 파이프라인일 수도 있음
                    - `job`
                        - 파이프라인 상태를 참조하려면 `job`도 함께 지정
                        - `job`을 지정하지 않으면, 파이프라인 상태만 미러링됨
                - 사용방법
                    - 다른 프로젝트나 다른 파이프라인의 상태를 가져와야 한다면 `project`, `job`, `ref`와 함께 사용할 수 있음
                    - GitLab CI/CD에서 다른 파이프라인을 미러링하여 그 상태에 맞춰 작업을 진행할 수 있음
                - 예시
                    
                    ```yaml
                    # 다른 파이프라인의 상태를 미러링
                    #   upstream_status 작업은 other/project라는 다른 프로젝트에서 실행된 파이프라인의 상태를 미러링
                    #   이 작업은 상위 파이프라인의 상태에 따라 현재 파이프라인의 실행 여부가 결정
                    upstream_status:
                      stage: test
                      needs:
                        pipeline: other/project  # 다른 프로젝트의 파이프라인 상태를 미러링
                    ---
                    # job과 함께 사용하는 예시
                    #   upstream_status가 other/project 프로젝트의 build-job 작업의 상태를 미러링
                    #   other/project의 build-job이 성공했는지 여부에 따라 upstream_status 작업이 실행
                    upstream_status:
                      stage: test
                      needs:
                        - pipeline: other/project
                          job: build-job  # 다른 프로젝트의 build-job 상태를 미러링
                    ---
                    # ref를 사용하는 예시: 특정 브랜치의 파이프라인 상태 미러링
                    #   ref: main은 other/project의 main 브랜치에서 실행된 build-job의 상태를 미러링한다는 의미
                    upstream_status:
                      stage: test
                      needs:
                        - pipeline: other/project
                          job: build-job
                          ref: main  # 특정 브랜치의 파이프라인 상태를 미러링
                    ```
                    
            - `needs:pipeline:job`
                - 설명
                    - `needs:pipeline:job`는 GitLab의 부모-자식 파이프라인(parent-child pipelines) 구조에서 자식 파이프라인이 부모 파이프라인 또는 다른 자식 파이프라인에서 실행된 작업으로부터 아티팩트를 다운로드할 수 있도록 하는 기능
                - 주요 특성
                    - 파이프라인 ID
                        - `needs:pipeline:job`은 부모-자식 파이프라인 계층 내에서 다른 파이프라인을 참조해야 하며, 해당 파이프라인의 ID를 사용
                        - 자식 파이프라인이 부모 파이프라인의 작업을 참조하여 아티팩트를 다운로드
                        - 현재 파이프라인의 ID는 사용할 수 없으며, 반드시 부모 파이프라인 또는 다른 자식 파이프라인의 파이프라인 ID를 지정해야 함
                    - 작업(job)
                        - 참조하려는 작업의 이름을 지정
                    - 아티팩트 다운로드
                        - 자식 파이프라인의 작업에서 부모 파이프라인이나 다른 자식 파이프라인의 작업 결과로 생성된 아티팩트를 다운로드할 수 있음
                - 주의사항
                    - *현재 파이프라인 ID($CI_PIPELINE_ID)는 사용할 수 없음
                    - 현재 파이프라인에서 작업의 아티팩트를 다운로드하려면 `needs:artifacts`를 사용해야 함
                    - `needs:pipeline:job`은 트리거 작업(trigger job)에서는 사용할 수 없음
                    - 다중 프로젝트 파이프라인에서 아티팩트를 다운로드하려면 `needs:project`를 사용해야 함
                - 예시
                    
                    ```yaml
                    # .gitlab-ci.yml (부모 파이프라인)
                    #   부모 파이프라인에서 자식 파이프라인을 트리거하고
                    #   PARENT_PIPELINE_ID라는 변수를 통해 부모 파이프라인의 ID를 자식 파이프라인에 전달
                    #   자식 파이프라인은 이 값을 사용해 needs:pipeline:job으로 부모 파이프라인의 아티팩트를 참조
                    stages:
                      - build
                      - deploy
                    
                    create-artifact:
                      stage: build
                      script:
                        - echo "sample artifact" > artifact.txt
                      artifacts:
                        paths:
                          - artifact.txt
                          
                    trigger-child-pipeline:
                      stage: deploy
                      trigger:
                        include: child.yml
                        strategy: depend
                      variables:
                        PARENT_PIPELINE_ID: $CI_PIPELINE_ID
                    
                    # child.yml (자식 파이프라인)
                    # 자식 파이프라인은 부모 파이프라인에서 생성된 아티팩트를 다운로드하려면 
                    # 부모 파이프라인의 ID를 pipeline 필드에 사용하고, 
                    # 아티팩트를 생성한 작업(job)을 job 필드에 지정
                    # $PARENT_PIPELINE_ID는 부모 파이프라인의 파이프라인 ID로, 
                    # CI_PIPELINE_ID 변수를 통해 부모 파이프라인 ID를 자식 파이프라인에 전달
                    stages:
                      - test
                    
                    use-artifact:
                      script: cat artifact.txt
                      needs:
                        - pipeline: $PARENT_PIPELINE_ID
                          job: create-artifact
                    
                    ```
                    
            - `needs:parallel:matrix`
                - 설명
                    - `needs:parallel:matrix`는 병렬화된 작업들 간의 의존성을 설정할 때 사용되는 기능
                    - `parallel:matrix` 와 함께 병렬화된 작업들 중 특정 인스턴스가 완료될 때까지 다른 작업을 기다리게 할 수 있도록 하여, 병렬 작업이 완료되는 순서에 따라 다른 작업을 실행하도록 설정
                - 예시
                    
                    ```yaml
                    
                    # linux:build 작업은 aws라는 PROVIDER와 
                    # 3개의 STACK (monitoring, app1, app2)을 사용하는 3개의 병렬 작업을 생성 
                    #    linux:build: [aws, monitoring]
                    #    linux:build: [aws, app1]
                    #    linux:build: [aws, app2]
                    linux:build:
                      stage: build
                      script: echo "Building linux..."
                      parallel:
                        matrix:
                          - PROVIDER: aws
                            STACK:
                              - monitoring
                              - app1
                              - app2
                    
                    # linux:rspec 작업은 linux:build 작업의 결과에 의존하며, 
                    # STACK 값이 app1인 linux:build: [aws, app1] 작업이 완료되면 실행
                    linux:rspec:
                      stage: test
                      needs:
                        - job: linux:build
                          parallel:
                            matrix:
                              - PROVIDER: aws
                                STACK: app1
                      script: echo "Running rspec on linux..."
                    ```
                    
    - **stages**
        - 설명
            - 요약
                - 실행될 단계(stage)를 정의하는 데 사용
                - 동일한 단계에 있는 작업들은 병렬로 실행
                - 작업 내에서만 사용할 수 있음
                - `stage` 값으로는 기본 단계 또는 사용자 정의 단계 문자열을 사용할 수 있음
            - 가능한 입력
                - 기본 단계
                    - 예를 들어 `test` 단계
                - 사용자 정의 단계:
                    - 를 들어 `build`, `deploy` 등
            - 세부 사항
                - 단계 이름은 255자 이하로 제한됩니다
                - 병렬 실행
                    - 각 단계의 작업은 서로 다른 실행 환경에서 실행될 경우 병렬로 실행될 수 있음
                    - 하나의 러너만 있을 경우, 러너의 `concurrent` 설정이 1보다 크면 병렬 실행이 가능
                - `needs` 설정
                    - `needs: []` 설정을 가진 작업은 `.pre` 단계의 작업과 함께 파이프라인이 생성되는 즉시 실행되며 이 경우 `stage` 설정은 무시되고, 작업은 바로 실행
            - 예시
                
                ```yaml
                # job1은 build 단계에서 실행 
                # job2는 test 단계에서 실행 
                # job3도 기본적으로 test 단계에서 실행 
                # job4는 deploy 단계에서 실행 
                stages:
                  - build
                  - test
                  - deploy
                
                job1:
                  stage: build
                  script:
                    - echo "This job compiles code."
                
                job2:
                  stage: test
                  script:
                    - echo "This job tests the compiled code."
                
                job3:
                  script:
                    - echo "This job also runs in the test stage."
                
                job4:
                  stage: deploy
                  script:
                    - echo "This job deploys the code."
                  environment: production
                
                ```
                
        - stages 전후 처리
            - **stage: .pre**
                - 설명
                    - `.pre` 단계는 파이프라인의 첫 번째 단계로, 사용자 정의 단계는 `.pre` 이후에 실행
                    - `.pre` 단계는 자동으로 파이프라인의 첫 번째 단계로 실행되며, 별도로 정의할 필요는 없음
                    - 만약 파이프라인에 `.pre` 또는 `.post` 단계만 포함되어 있으면, 해당 파이프라인은 실행되지 않으며 적어도 하나의 다른 단계가 있어야 동작
                - 예시
                    
                    ```yaml
                    stages:
                      - build
                      - test
                    
                    first-job:
                      stage: .pre
                      script:
                        - echo "This job runs in the .pre stage."
                    
                    job1:
                      stage: build
                      script:
                        - echo "This job runs in the build stage."
                    
                    job2:
                      stage: test
                      script:
                        - echo "This job runs in the test stage."
                    
                    ```
                    
            - **stage: .post**
                - 설명
                    - **`.post`** 단계는 파이프라인의 **마지막** 단계로, 사용자 정의 단계는 `.post` 이전에 실행
                    - `.post` 단계는 **자동으로 파이프라인의 마지막 단계로 실행**되며, 별도로 정의할 필요는 없음
                    - 파이프라인에 `.pre` 또는 `.post` 단계만 포함되어 있으면, 해당 파이프라인은 실행되지 않음
                - 예시
                    
                    ```yaml
                    stages:
                      - build
                      - test
                    
                    last-job:
                      stage: .post
                      script:
                        - echo "This job runs in the .post stage."
                    
                    job1:
                      stage: build
                      script:
                        - echo "This job runs in the build stage."
                    
                    job2:
                      stage: test
                      script:
                        - echo "This job runs in the test stage."
                    ```
                    
    - **trigger**
        - 설명
            - 요약
                - `trigger` 키워드는 트리거 작업을 선언하는 데 사용
                - 이 작업은 하위 파이프라인 또는 다중 프로젝트 파이프라인을 시작
                - 트리거 작업은 특정 파이프라인을 실행하는 역할
                    - 예를 들어, 자식 파이프라인을 시작하거나 다른 프로젝트에서 파이프라인을 트리거할 수 있음
            - 사용 방법
                - 트리거 작업은 하위 파이프라인 또는 다중 프로젝트 파이프라인을 시작할 때 사용
                - 트리거 작업에서 사용할 수 있는 GitLab CI/CD 구성 키워드는 제한적
                    - `allow_failure`, `extends`, `needs`, `only`, `except`, `rules`, `stage`, `trigger`, `variables`, `when`, `resource_group`, `environment`
            - 가능한 입력
                - 다중 프로젝트 파이프라인을 트리거
                    - `trigger: my-group/my-project`와 같은 방식으로 하위 프로젝트의 경로를 지정
                    - CI/CD 변수는 GitLab 15.3 이후 지원
                - 자식 파이프라인을 트리거
                    - `trigger: include`를 사용하여 자식 파이프라인의 구성 파일 경로를 지정
            - 예시
                
                ```yaml
                # 다중 프로젝트 파이프라인 트리거
                #   my-group/my-project라는 경로의 다중 프로젝트 파이프라인을 트리거하는 작업
                trigger-multi-project-pipeline:
                  trigger: my-group/my-project
                
                # 다른 브랜치에서 다중 프로젝트 파이프라인 트리거
                #   my-group/my-project의 development 브랜치에서 파이프라인을 트리거
                trigger-multi-project-pipeline:
                  trigger:
                    project: my-group/my-project
                    branch: development
                
                # 자식 파이프라인 트리거
                #    path/to/child-pipeline.gitlab-ci.yml 파일을 포함하여 자식 파이프라인을 트리거하는 작업
                trigger-child-pipeline:
                  trigger:
                    include: path/to/child-pipeline.gitlab-ci.yml
                
                ```
                
        - trigger 키워드
            - `trigger:include`
                - 설명
                    - `trigger:include`는 자식 파이프라인을 시작하는 트리거 작업을 선언하는 데 사용
                    - 이 작업은 자식 파이프라인의 구성 파일을 지정하여 해당 파이프라인을 트리거
                - 사용 방법
                    - 자식 파이프라인의 경로를 include로 지정하여 트리거 작업을 설정
                    - `trigger:include:artifact`를 사용하면 동적 자식 파이프라인을 트리거
                - 예시
                    
                    ```yaml
                    # path/to/child-pipeline.gitlab-ci.yml 파일을 포함하여 자식 파이프라인을 트리거하는 작업
                    trigger-child-pipeline:
                      trigger:
                        include: path/to/child-pipeline.gitlab-ci.yml
                    ```
                    
            - `trigger:project`
                - 설명
                    - `trigger:project`는 다중 프로젝트 파이프라인을 시작하는 트리거 작업을 선언하는 데 사용
                    - 이 트리거는 다른 GitLab 프로젝트에서 파이프라인을 시작하는 기능을 제공
                - 사용 방법
                    - `trigger:project`에 하위 프로젝트의 경로를 지정하여 다중 프로젝트 파이프라인을 트리거
                    - 기본적으로 다중 프로젝트 파이프라인은 기본 브랜치에서 트리거되며, 다른 브랜치를 지정하려면 `trigger:branch`를 사용
                - 예시
                    
                    ```yaml
                    # 기본 브랜치에서 다중 프로젝트 파이프라인 트리거
                    trigger-multi-project-pipeline:
                      trigger:
                        project: my-group/my-project
                    
                    # 다른 브랜치에서 다중 프로젝트 파이프라인 트리거
                    trigger-multi-project-pipeline:
                      trigger:
                        project: my-group/my-project
                        branch: development
                    ```
                    
            - `trigger:strategy`
                - 설명
                    - `trigger:strategy`는 트리거 작업이 하위 파이프라인이 완료될 때까지 대기하도록 설정하는 데 사용
                    - `strategy: depend`(기본값)
                        - 기본적으로 트리거 작업은 하위 파이프라인이 생성되는 즉시 성공으로 간주되지만, `strategy: depend`(기본값)를 사용하면 하위 파이프라인이 완료된 후에야 트리거 작업이 성공으로 표시
                    - `strategy: async`
                        - 트리거 작업은 하위 파이프라인이 **시작되자마자** 성공으로 간주
                        - 하위 파이프라인의 결과와 관계없이, 하위 파이프라인이 실행되기만 하면 트리거 작업은 바로 성공 상태로 표시
                - 예시
                    
                    ```yaml
                    # depend 사용시
                    #   trigger-job이 실행되어 하위 파이프라인을 트리거
                    #   이때 strategy: depend가 설정되어 있기 때문에 
                    #   하위 파이프라인이 완료되기 전까지는 trigger-job이 성공으로 간주되지 않음
                    #   하위 파이프라인이 성공적으로 완료되면 trigger-job은 성공 상태로 표시되며, 
                    #   그 후에 notify-job이 실행
                    stages:
                      - build
                      - deploy
                      - notify
                    
                    trigger-job:
                      stage: deploy
                      trigger:
                        include: path/to/child-pipeline.yml
                        strategy: depend
                    
                    notify-job:
                      stage: notify
                      script:
                        - echo "Notifying after child pipeline completion."
                    
                    # async 사용시
                    #  build 단계에서 프로젝트를 빌드합니다.
                    #  trigger_child_pipeline 작업은 strategy: async를 사용하여 하위 파이프라인을 트리거 
                    #    이때, trigger_child_pipeline 작업은 하위 파이프라인이 시작되자마자 성공으로 간주
                    #    하위 파이프라인이 끝나지 않아도 이 작업은 바로 성공으로 표시되고 
                    #    후속 작업인 deploy가 병렬로 실행 
                    #  deploy 단계는 trigger_child_pipeline이 시작되자마자 실행 
                    stages:
                      - build
                      - test
                      - deploy
                    
                    build:
                      stage: build
                      script:
                        - echo "Building the project"
                    
                    trigger_child_pipeline:
                      stage: test
                      trigger:
                        include: path/to/child-pipeline.yml
                        strategy: async
                    
                    deploy:
                      stage: deploy
                      script:
                        - echo "Deploying the project"
                    ```
                    
            - `trigger:forward`
                - 설명
                    - `trigger:forward`는 트리거 작업에서 **변수**를 하위 파이프라인으로 전달하는 데 사용
                    - 기본적으로 변수는 하위 파이프라인으로 전달되지 않으며, 이를 제어하기 위해 `trigger:forward`를 사용
                    - `trigger:forward`는 파이프라인 변수와 YAML 변수를 전달할 수 있음
                - 가능한 입력
                    - `yaml_variables`: `true` 또는 `false` (기본값은 `true`). 트리거 작업에서 정의된 변수를 하위 파이프라인으로 전달할지 여부를 설정
                    - `pipeline_variables`: `true` 또는 `false` (기본값은 `true`). 파이프라인 변수를 하위 파이프라인으로 전달할지 여부를 설정
                - 예시
                    
                    ```yaml
                    # 기본 설정: 파이프라인 변수가 하위 파이프라인으로 전달됨
                    #   child2 작업에서 파이프라인 변수를 하위 파이프라인으로 전달하고, 
                    #   child3 작업에서는 YAML 변수를 전달하지 않음
                    child1:
                      trigger:
                        include: .child-pipeline.yml
                    
                    # 파이프라인 변수를 전달
                    child2:
                      trigger:
                        include: .child-pipeline.yml
                        forward:
                          pipeline_variables: true
                    
                    # YAML 변수를 전달하지 않음
                    child3:
                      trigger:
                        include: .child-pipeline.yml
                        forward:
                          yaml_variables: false
                    ```
                    
- 병렬 실행 및 확장
    - **parallel**
        - `parallel`
            - 설명
                - `parallel`을 사용하면 동일한 작업을 여러 번 실행할 수 있음
                - 예를 들어, 테스트를 여러 환경에서 동시에 실행하거나, 여러 번 반복적으로 실행해야 하는 작업을 분리하여 병렬로 처리할 수 있음
                - `parallel`은 숫자 값으로 설정되며, 숫자만큼 동일한 작업을 병렬로 실행
                - 각 작업은 `job_name 1/N`, `job_name 2/N` 등의 이름을 가짐
            - 예시
                
                ```yaml
                #  test 작업이 5번 병렬로 실행되며, 
                #  각각의 작업은 test 1/5, test 2/5, ..., test 5/5와 같은 이름을 가짐
                test:
                  script: rspec
                  parallel: 5
                ```
                
        - `parallel:matrix`
            - 설명
                - 변수와 그 값을 배열로 지정하고, 각 변수 값의 모든 조합에 대해 작업을 병렬로 실행
            - 세부사항
                - 변수
                    - `parallel`로 실행된 각 작업에는 두 개의 주요 CI/CD 변수가 설정됨
                        - `CI_NODE_INDEX`: 현재 실행 중인 병렬 작업의 인덱스 (예: 1, 2, 3, ...)
                        - `CI_NODE_TOTAL`: 전체 병렬 작업의 수 (예: 총 5개의 작업)
                - 병렬 실행 제한
                    - 병렬로 실행할 수 있는 작업 수가 러너의 수보다 많을 경우, 일부 작업은 대기 상태로 대기열에 들어가며 러너가 준비될 때까지 대기
                    - 너무 많은 병렬 작업을 실행하면 `job_activity_limit_exceeded` 에러가 발생할 수 있음
                    - 이는 인스턴스 레벨에서 활성 파이프라인에 존재할 수 있는 최대 작업 수를 초과했을 때 발생
                - 변수 이름 제한
                    - `parallel:matrix`에서 사용되는 변수 이름은 숫자, 문자, 밑줄(_)만 사용할 수 있음
                - 변수 값
                    - 변수 값은 문자열이나 문자열 배열이어야 함
                    - 각 변수 값의 조합은 최대 200개의 조합까지만 생성할 수 있음
                - 작업 이름 제한
                    - 생성된 각 작업의 이름은 최대 255자이어야 하며, `needs`와 함께 사용할 경우 작업 이름은 128자 이하로 제한
                - 중복 이름 처리
                    
                    ```yaml
                    
                    test:
                      parallel:
                        matrix:
                          - OS: [ubuntu]
                            PROVIDER: [aws, gcp]
                          - OS2: [ubuntu]   # 잘못된 예: OS와 OS2의 값이 동일
                            PROVIDER: [aws, gcp]
                    
                    ```
                    
                    - 동일한 변수 값으로 동일한 작업 이름이 생성되면, 이전의 작업이 덮어써지기 때문에 변수 값을 다르게 설정해야 함
            - 예시
                
                ```yaml
                # 예시에서는 PROVIDER와 STACK 변수의 여러 값 조합에 대해 작업을 병렬로 실행
                #  생성된 작업들
                #    deploystacks: [aws, monitoring]
                #    deploystacks: [aws, app1]
                #    deploystacks: [aws, app2]
                #    deploystacks: [ovh, monitoring]
                #    deploystacks: [ovh, backup]
                #    deploystacks: [ovh, app]
                #    deploystacks: [gcp, data]
                #    deploystacks: [gcp, processing]
                #    deploystacks: [vultr, data]
                #    deploystacks: [vultr, processing]
                #    deploystacks:
                  stage: deploy
                  script:
                    - bin/deploy
                  parallel:
                    matrix:
                      - PROVIDER: aws
                        STACK:
                          - monitoring
                          - app1
                          - app2
                      - PROVIDER: ovh
                        STACK: [monitoring, backup, app]
                      - PROVIDER: [gcp, vultr]
                        STACK: [data, processing]
                  environment: $PROVIDER/$STACK
                
                ```
                
    - **extends**
        - 설명
            - GitLab CI/CD 파이프라인에서 구성 재사용을 위한 기능
            - 이 키워드를 사용하면 중복을 줄이고, 여러 작업(Job)에서 공통된 구성을 효율적으로 관리가능
            - `extends`는 YAML의 앵커와 비슷하지만, 더 유연하고 읽기 쉬운 방식으로 작업 구성을 상속받을 수 있음
            - `extends`는 다른 작업(Job)에서 정의된 구성을 상속하여 해당 작업에 적용할 수 있도록 함
            - 이를 통해 중복 코드를 줄이고, 설정을 보다 간결하고 효율적으로 관리
            - 최대 11레벨까지 상속이 가능하지만 3레벨 이상 사용하는 것은 권장하지 않음
            - `.` 이 붙은 작업은 숨겨진 템플릿 작업으로 이를 `extends` 를 통해 사용 가능
        - 주요 특징
            - 상속
                - `extends`는 다른 작업의 구성을 재사용할 수 있게 해 줌
                - 상속받은 작업은 기존의 설정을 그대로 가져오되, 추가적인 값을 덮어쓸 수 있음
            - 직관적이고 간결한 설정
                - Anchor 를 사용하는 것보다 더 읽기 쉽고, 유지보수에 유리
        - 사용 방법
            - 단일 작업
                - 다른 하나의 작업을 상속받을 수 있음
            - 여러 작업
                - 여러 작업을 한 번에 상속받을 수 있음
        - 예시
            
            ```yaml
            # 기본 사용법
            # .tests는 숨겨진 템플릿 작업으로, stage와 image를 정의 
            # rspec 작업은 .tests 작업을 확장(extends: .tests)하여 공통 구성을 재사용하고, 그 위에 script: rake rspec을 덧붙여 정의 
            # rubocop 작업도 마찬가지로 .tests를 확장하고, script: bundle exec rubocop을 추가 
            .tests:
              stage: test
              image: ruby:3.0
              
            rspec:
              extends: .tests
              script: rake rspec
              
            rubocop:
              extends: .tests
              script: bundle exec rubocop
            
            ---
            # 여러 부모 작업 상속
            # .base_job 작업은 공통된 설정을 정의. 여기에는 이미지 설정과 before_script가 포함
            # .tests와 .deploy는 각각 .base_job을 확장하여 image와 before_script를 상속
            # stage와 script만 각각 다르게 설정
            .base_job:
              image: python:3.9
              before_script: ["pip install -r requirements.txt"]
            
            .tests:
              extends: .base_job
              stage: test
              script: pytest
            
            .deploy:
              extends: .base_job
              stage: deploy
              script: deploy.sh
            
            ```
            
- 테스트
    - **coverage**
        - 설명
            - GitLab에서 코드 커버리지(Coverage)를 추출하고 UI에 표시하려면 `coverage` 키를 사용하여 커스텀 정규식을 설정할 수 있음
        - 동작 원리
            1. GitLab은 job 출력 로그에서 사용자가 제공한 정규식(Regex)을 사용해 커버리지 정보를 포함한 라인을 찾음
            2. 찾은 결과에서 작은 정규식 `\d+(?:\.\d+)?`을 사용해 숫자 값을 추출
            3. 커버리지 값은 UI에 표시 
        - 설정 형식
            - 정규식은 **RE2 정규식 형식**이어야 하며, 슬래시(`/`)로 시작하고 끝나야 함
            - 정규식은 커버리지 값을 포함한 문자열 전체를 매칭할 수 있으며, 커버리지 값을 특정 그룹으로 캡처할 필요는 없음
        - 추가정보
            - RE2 정규식 특징
                - RE2는 빠르고 안전한 정규식 엔진입니다.
                - 캡처 그룹 대신 **비캡처 그룹**(`?:`)을 사용하는 것이 중요
                - 예시: `/Coverage: \d+(?:\.\d+)?/`
            - 추출을 위한 팁
                - 로그의 형식이 다를 경우 정규식을 조정해야 함
                    - 예시: `Coverage: 80%` → `/Coverage: \d+%/`
                    - 예시: `Lines covered: 12345` → `/Lines covered: \d+/`
            - 자주 사용하는 로그 출력 및 정규식 예시
                - 출력: `Test coverage: 85.32%`
                    
                    ```yaml
                    coverage: '/Test coverage: \d+(?:\.\d+)?/'
                    ```
                    
                - 출력: `Coverage achieved: 91%`
                    
                    ```yaml
                    coverage: '/Coverage achieved: \d+%/'
                    ```
                    
                - 출력: `Lines covered: 123/150`
                    
                    ```yaml
                    coverage: '/Lines covered: \d+\/\d+/'
                    ```
                    
        - 예시
            
            ```yaml
            job1:
              script: rspec
              coverage: '/Code coverage: \d+(?:\.\d+)?/'
            
            # 출력로그의 예
            Code coverage: 67.89% of lines covered
            ```
            
- 기타
    - **dast_configuration (ultimate only)**
        - 설명
            - Dynamic Application Security Testing(DAST) 작업에 대한 특정 구성을 정의하는데 사용
            - DAST는 웹 애플리케이션의 실행 중 보안 취약점을 찾기 위한 방법으로, 외부에서 공격자처럼 애플리케이션을 분석하고 테스트하는 과정을 포함
            - `dast_configuration` 키워드는 파이프라인에서 DAST 작업을 실행할 때 사용할 사이트 프로필과 스캐너 프로필을 지정할 수 있게 해줌
        - 주요사항
            - Gitlab Ultimate 티어에서 사용 가능
        - 구성요소
            - site_profile
                - 테스트할 웹사이트의 구성 프로파일을 지정
                - 이 프로파일에는 애플리케이션의 URL, 인증 정보 및 기타 설정이 포함
            - scanner_profile
                - DAST 스캐너가 사용하는 프로파일로, 스캐닝 방법이나 정책 등이 포함
        - 예시
            
            ```yaml
            stages:
              - build
              - dast
            
            include:
              - template: DAST.gitlab-ci.yml
            
            dast:
              dast_configuration:
                site_profile: "Example Co"
                scanner_profile: "Quick Passive Test"
            ```
            
    - **hooks**
        - 설명
            - GitLab CI/CD에서 특정 시점에 명령을 실행하도록 설정할 수 있는 기능
            - 이 기능을 사용하여 작업(Job) 실행 전에 또는 특정 단계에서 명령을 실행할 수 있음
            - 예를 들어, Git 저장소를 클론하기 전에 Git 설정을 조정하거나, 트레이싱 변수(export)를 설정하는 데 사용
        - 사용법
            - 작업의 일부분으로 사용되며 GitLab Runner에서 실행되는 **특정 단계 전에** 명령을 실행
            - `hooks`의 주요 용도는 Git 저장소를 가져오기 전에 설정을 조정하거나, 특정 변수를 설정하는 등의 준비 작업을 하는 것
        - 추가사항
            - 명령 목록 사용가능
                - `pre_get_sources_script`는 여러 명령을 실행할 수 있는 배열로 지정
                - 명령은 한 줄로 나열할 수 있으며, 여러 줄에 걸쳐 나열할 수도 있음
            - CI/CD 변수 사용 가능
                - CI/CD 변수 또는 YAML 앵커도 사용할 수 있음
            - GitLab Runner 설정
                - `hooks`는 GitLab Runner의 구성과 관련이 있음
                - GitLab Runner가 작업을 실행할 때, 정의된 훅을 수행
        - 가능한 훅
            - `pre_get_sources_script`
                - Git 저장소와 그 하위 모듈을 클론하기 전에 실행되는 명령들을 지정
        - 예시
            
            ```yaml
            # pre_get_sources_script 훅에서 echo 'hello job1 pre_get_sources_script' 명령을 실행합니다. 이는 Git 저장소를 클론하기 전에 실행
            # script는 일반적인 스크립트 명령으로, echo 'hello job1 script'가 Git 저장소 클론 후에 실행
            job1:
              hooks:
                pre_get_sources_script:
                  - echo 'hello job1 pre_get_sources_script'
              script: echo 'hello job1 script'
            ```
            
    - **id_token**
        - 설명
            - 타사 서비스와의 인증을 위해 JSON Web Tokens (JWT)을 생성하는 기능
            - GitLab CI/CD에서 **OIDC (OpenID Connect)** 인증을 지원하는 외부 서비스와 안전하게 인증할 수 있음
            - 생성된 JWT는 타사 서비스에 인증을 요청할 때 사용
        - 사용법
            - `id_tokens` 키워드는  작업의 일부분으로 사용되며, 특정 서비스와의 인증을 위해 JWT를 생성하는 데 사용
            - `aud` 필드는 JWT에 포함될 **aud claim**을 설정하는 데 사용
        - 가능한 입력값
            - 토큰 이름과 그에 대응하는 aud claim
                - `aud`는 하나의 문자열 또는 문자열 배열일 수 있으며, CI/CD 변수도 지원
        - 예시
            
            ```yaml
            # id_tokens에서 각 토큰은 고유한 이름(ID_TOKEN_1, ID_TOKEN_2, SIGSTORE_ID_TOKEN)을 가지며
            # aud를 설정하여 각 토큰이 인증할 대상을 지정 
            #    ID_TOKEN_1은 https://vault.example.com에 인증을 위해 사용 
            #    ID_TOKEN_2는 https://gcp.com과 https://aws.com에 인증을 위해 사용 
            #    SIGSTORE_ID_TOKEN은 sigstore에 인증을 위해 사용 
            #    script에서 각 토큰을 사용하여 각기 다른 클라우드 서비스(GCP, AWS, Vault 등)에 인증을 수행 
            job_with_id_tokens:
              id_tokens:
                ID_TOKEN_1:
                  aud: https://vault.example.com
                ID_TOKEN_2:
                  aud:
                    - https://gcp.com
                    - https://aws.com
                SIGSTORE_ID_TOKEN:
                  aud: sigstore
              script:
                - command_to_authenticate_with_vault $ID_TOKEN_1
                - command_to_authenticate_with_aws $ID_TOKEN_2
                - command_to_authenticate_with_gcp $ID_TOKEN_2
            
            ```
            
    - **identity (beta)**
        - 설명
            - `identity`는 **타사 서비스와의 인증을 위한 기능**으로, GitLab CI/CD에서 **Identity Federation**을 사용하여 외부 서비스에 인증할 수 있게 해줌
            - 이 기능은 현재 베타 버전으로 제공되며, 주로 Google Cloud와 같은 타사 클라우드 서비스와 연동하여 CI/CD 파이프라인에서 외부 자원에 접근하는 데 사용
        - 사용법
            - 작업의 일부로 사용되며 이를 통해 GitLab Runner가 외부 서비스에 인증하고 필요한 작업을 실행
            - 타사 서비스와 인증을 연결하려면 해당 서비스와의 **Identity Federation**을 설정해야 함
        - 가능한 입력값
            - `google_cloud`
                - Google Cloud에 인증하기 위해 사용
                - 이 경우, Google Cloud IAM(Identity and Access Management) 통합이 필요
        - 예시
            
            ```yaml
            job_with_workload_identity:
              identity: google_cloud
              script:
                - gcloud compute instances list
            ```

### Anchor 
- GitLab CI에서 "anchor"는 YAML 파일에서 중복 코드를 줄이고 구성 요소를 재 사용하기 위해 사용되는 기능
- YAML에서는 앵커와 알리아스를 사용하여 특정 블록을 정의하고 이를 여러 곳에서 참조할 수 있음
- job 앞에 `.` 을 붙여주면 동작하지 않는 job 이 되며 이를 참조하기 위한 알리아스 `&` 로 이 job 의 데이터를 호출 `*` 하는 방식
    - .<동작하지 않는 job 명> = &<알리아스명>
    - *<알리아스명> 으로 호출
            
    ```yaml
        #  "default_job"이라는 앵커를 정의
        .default_job: &default_job
          script:
            - echo "This is a default job"
          tags:
            - docker
        
        job1:
          <<: *default_job #  앵커를 참조하여 default_job의 모든 설정을 가져옴
          script:
            - echo "This is job1"
        
        job2:
          <<: *default_job
          script:
            - echo "This is job2"    
    ```
