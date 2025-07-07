pipeline {
    agent { label 'agent' }

    stages {
        stage('Checkout') {
            steps {
                echo "Branch Name: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running Unit Tests..."
                sh 'mvn clean test'
            }
        }

        stage('Security Scan') {
            steps {
                echo "Running Trivy Security Scan..."
                sh '''
                    wget -q https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O html.tpl
                    trivy fs --format template --template "@html.tpl" -o report.html .
                '''
            }
        }

        stage('Build') {
            when {
                branch 'main'
            }
            steps {
                echo "Running Build on main branch..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo "Deploying application on main branch..."
                sh '''
                    if pgrep -f "java -jar java-sample-21-1.0.0.jar" > /dev/null; then
                        pkill -f "java -jar java-sample-21-1.0.0.jar"
                        echo "App was running and has been killed"
                    else
                        echo "App is not running"
                    fi

                    JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > app.log 2>&1 &
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace"
            cleanWs()
        }
    }
}
