pipeline {
    agent any

    stages {

        stage('Get Code') {
            steps {
                git 'https://github.com/sergiozapatero/cp1-2.git'
                sh 'pwd'
                sh 'ls -l'
            }
        }

        stage('Unit') {
            steps {
                sh 'python3 -m pytest test/unit --junitxml=result-unit.xml'
            }
            post {
                always {
                    junit 'result-unit.xml'
                }
            }
        }

        stage('Rest') {
            steps {
                sh 'python3 -m pytest test/rest --junitxml=result-rest.xml'
            }
            post {
                always {
                    junit 'result-rest.xml'
                }
            }
        }

        stage('Static') {
            steps {
                sh 'flake8 app --exit-zero --format=pylint > flake8-report.txt'
                recordIssues(
                    tools: [pyLint(pattern: 'flake8-report.txt')],
                    qualityGates: [
                        [threshold: 8, type: 'TOTAL', unstable: true],
                        [threshold: 10, type: 'TOTAL', failed: true]
                    ]
                )
            }
        }

        stage('Security Test') {
            steps {
                sh 'bandit -r app -f txt -o bandit-report.txt || true'

                recordIssues(
                    tools: [pyLint(pattern: 'bandit-report.txt')],
                    qualityGates: [
                        [threshold: 2, type: 'TOTAL', unstable: true],
                        [threshold: 4, type: 'TOTAL', failed: true]
                    ]
                )
            }
        }

        stage('Performance') {
            steps {
                sh 'jmeter -n -t performance/test.jmx -l result.jtl'
            }
        }

        stage('Coverage') {
            steps {
                sh '''
                coverage run --branch -m pytest test/unit
                coverage xml -o coverage.xml
                coverage report
                '''
            }
        }
    }

    post {
    always {
        script {
            if (fileExists('result.jtl')) {
                perfReport sourceDataFiles: 'result.jtl'
            } else {
                echo "Skipping perfReport: no JTL generated"
            }
        }
    }
}
}
