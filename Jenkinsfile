pipeline {
    agent {
        docker {
            image 'node:16-buster-slim'
            args '-p 3000:3000'
        }
    }
    // รับการทริกเกอร์จาก GitHub Actions
    triggers {
        githubPush()
    }
    
    stages {
        // ขั้นตอนนี้ทำเฉพาะกรณีที่ถูกทริกเกอร์โดยตรง ไม่ได้มาจาก GitHub Actions
        stage('Check for GitHub Actions') {
            steps {
                script {
                    // ตรวจสอบว่ามาจาก GitHub Actions หรือไม่
                    env.FROM_GITHUB_ACTIONS = sh(script: 'curl -s $JENKINS_URL/api/json | grep -q "github-actions"', returnStatus: true) == 0
                    echo "FROM_GITHUB_ACTIONS: ${env.FROM_GITHUB_ACTIONS}"
                }
            }
        }
        
        // จะทำ Build และ Test เฉพาะเมื่อไม่ได้มาจาก GitHub Actions
        stage('Build') {
            when { expression { return env.FROM_GITHUB_ACTIONS != 'true' } }
            steps {
                sh 'npm install'
            }
        }
        
        stage('Test') {
            when { expression { return env.FROM_GITHUB_ACTIONS != 'true' } }
            steps {
                sh 'chmod +rx ./jenkins/scripts/*.sh'
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Prepare for Deploy') {
            when {
                branch 'master'
            }
            steps {
                script {
                    // ตรวจสอบว่ามีการ build จาก GitHub Actions หรือไม่
                    def hasBuild = fileExists('build')
                    if (!hasBuild) {
                        echo 'ไม่พบไฟล์ build ที่มาจาก GitHub Actions จะทำการ build ในตัว Jenkins เอง'
                        sh 'npm run build'
                    } else {
                        echo 'พบไฟล์ build จาก GitHub Actions ใช้ build นี้ในการ deploy'
                    }
                }
                
                // เตรียมการ deploy
                sh 'chmod +rx ./jenkins/scripts/*.sh'
            }
        }
        
        stage('Auto Approval') {
            when {
                branch 'master'
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
                echo 'Auto approval for master branch'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
                sh './jenkins/scripts/kill.sh'
                echo 'Successfully deployed the application!'
            }
        }
    }
}