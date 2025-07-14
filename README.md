# 프로젝트 CI/CD 파이프라인 구성

이 문서는 GitHub, Jenkins, Docker Compose, ngrok을 활용하여 FastAPI 애플리케이션을 자동 빌드·테스트·배포하는 CI/CD 워크플로우를 설명합니다.

---

## 📁 파일 구조

```plaintext
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── script.groovy
├── requirements.txt
└── app
    └── main.py
```

- **Dockerfile**: Python 3.9 기반 이미지에 FastAPI 앱 복사 및 의존성 설치 후 Uvicorn 실행
- **docker-compose.yml**: `web` 서비스(FastAPI 앱)를 컨테이너로 정의, 포트 매핑 및 볼륨 마운트 설정
- **Jenkinsfile**: Pipeline as Code 정의 (Build, Test, Deploy 단계)
- **script.groovy**: 파이프라인 내 빌드/테스트/배포 로직을 메서드로 구현
- **requirements.txt**: Python 패키지 목록
- **app/main.py**: FastAPI 애플리케이션 엔트리포인트

---

## 🔧 사전 준비

1. **GitHub** 레포지토리 생성 및 코드 푸시, webhooks 추가
2. **Jenkins** 설치 (WSL2 또는 서버), jenkins GitHub Plugin, Git Plugin 설치
3. **GitHub Personal Access Token** 발급 후 Jenkins Credentials에 등록(`jenkinscicd`)
4. **ngrok** wsl2 우분투 시스템에 설치 및 인증 토큰 설정 (설치 홈페이지 참고)
---

## 🔗 GitHub ↔ Jenkins 연동

1. Jenkins **Pipeline** 아이템 생성
   - Definition: *Pipeline script from SCM*
   - SCM: Git → 레포 URL(`https://github.com/your-org/your-repo.git`)
   - Credentials: Credentials 추가(Secret text 선택, Secret에 GitHub Personal access tokens 기재)
   - Branch Specifier: `*/main`
   - Script Path: `Jenkinsfile`
2. Build Triggers: **GitHub hook trigger for GITScm polling** 활성화
3. GitHub 레포 → Settings → Webhooks → Add webhook
   - Payload URL: `https://<NGROK_DOMAIN>/github-webhook/`
   - Content type: `application/json`
   - 이벤트: `Just the push event.`

---

## 🚀 ngrok으로 외부 접근

WSL2 또는 Jenkins 호스트에서 다음 명령어 실행:

```bash
ngrok http http://localhost:8080
```

- ngrok이 발급한 도메인을 Webhook Payload URL로 사용

---

## 📜 Jenkinsfile 파이프라인

```groovy
pipeline {
  agent any
  stages {
    stage('Init') {
      steps { script { gv = load 'script.groovy' } }
    }
    stage('Build') {
      steps {
        script { gv.buildApp() }
        sh 'docker compose build web'
      }
    }
    stage('Test') {
      when { expression { params.executeTests } }
      steps { script { gv.testApp() } }
    }
    stage('Deploy') {
      steps {
			// 1) 기존 서비스 내리기 (실행 중인 컨테이너 삭제)
			sh 'docker compose down || true'
			// 2) 새 이미지로 빌드 & 기동
			sh 'docker compose build web'
			sh 'docker compose up -d --force-recreate --remove-orphans'
      }
    }
  }
}
```

1. **Init**: `script.groovy` 로직 로드
2. **Build**: Docker 이미지 빌드
3. **Test** (옵션): 테스트 스크립트 실행
4. **Deploy**: 컨테이너 및 오퍼런 컨테이너 정리 후 재배포

---

## 🔄 전체 워크플로우

1. 코드 변경 → `git add && git commit && git push origin main`
2. GitHub → ngrok → Jenkins Webhook 전달
3. Jenkins 파이프라인 실행
4. Docker 이미지 빌드 → 컨테이너 기동 → 최신 버전 서비스 활성화
5. 성공 시 Jenkins 콘솔에 로그 확인
<img width="1122" height="611" alt="image" src="https://github.com/user-attachments/assets/5f520669-3984-4fd0-87ce-114e0cc2af65" />

<img width="215" height="94" alt="image" src="https://github.com/user-attachments/assets/ee228417-1421-4626-a77f-2424a9b592d1" />

---

## 👀 참고

- **docker compose up --build**: 변경된 레이어만 재빌드

---

> 이제 코드 푸시 한 번으로 빌드·테스트·배포가 자동화됩니다! 🎉

