pipeline {
	agent any
	parameters {
		choice(name: 'VERSION', choices: ['1.1.0','1.2.0','1.3.0'], description: '')
		booleanParam(name: 'executeTests', defaultValue: true, description: '')
	}
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
		stage("test") {
			when {
				expression {
					params.executeTests
				}
			}
			steps {
				script {
					gv.testApp()
				}
			}
		}
		stage("deploy") {
			steps {
				// sh "docker-compose up -d"
				// 1) 기존 서비스 내리기 (실행 중인 컨테이너 삭제)
				sh 'docker compose down || true'
				// 2) 새 이미지로 빌드 & 기동
				sh 'docker compose build web'
				sh 'docker compose up -d --force-recreate --remove-orphans'
			}
		}
	}
}