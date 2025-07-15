pipeline {
	agent any
	stages {
		stage("init") {
			steps {
				script {
					gv = load "script.groovy"
				}
			}
		}
		stage("Checkout") {
			steps {
				checkout scm
			}
		}
		stage("Build") {
			steps {
				sh 'docker-compose build web'
			}
		}
		// stage("test") {
		// 	when {
		// 		expression {
		// 			params.executeTests
		// 		}
		// 	}
		// 	steps {
		// 		script {
		// 			gv.testApp()
		// 		}
		// 	}
		// }
		stage("deploy") {
			steps {
			// 1) 기존 컨테이너 내리기 (실행 중인 컨테이너 삭제)
			sh 'docker compose down --remove-orphans || true'
			// 2) 변경된 레이어만 빌드 및 컨테이너 기동
			sh 'docker compose up -d --build --force-recreate --remove-orphans'
			// 3) dangling 이미지 정리
			sh 'docker image prune -f'
			}
		}
	}
}