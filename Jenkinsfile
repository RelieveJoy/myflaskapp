pipeline {
    agent any

    // 注意：PYTHON_PATH 用 8.3 短路径，而不是完整路径。
    // 完整路径为 C:\Users\海祇雪莉\AppData\Local\Programs\Python\Python310\python.exe，
    // 但用户名含中文，Jenkins 的 bat 步骤会把脚本写成 .bat 文件并按系统码页(GBK)解释，
    // 导致中文路径变成乱码、报 "The system cannot find the path specified"。
    // 短路径等价且与编码无关，实测 GBK / UTF-8 下均可正常调用。
    environment {
        PYTHON_PATH = 'C:\\Users\\7071~1\\AppData\\Local\\Programs\\Python\\PYTHON~1\\python.exe'
    }

    stages {
        stage('Verify Python Path') {
            steps {
                bat """
                    @echo off
                    echo "=== 验证Python路径及版本 ==="
                    ${PYTHON_PATH} --version
                    echo "Python路径验证通过！"
                """
            }
        }
        stage('Checkout') {
            steps {
                git url: 'https://github.com/RelieveJoy/myflaskapp.git', branch: 'main'
            }
        }
        stage('Fix pip') {
            steps {
                bat """
                    @echo off
                    echo "=== 修复pip环境 ==="
                    ${PYTHON_PATH} -m ensurepip --upgrade
                    ${PYTHON_PATH} -m pip --version
                """
            }
        }
        stage('Install Dependencies') {
            steps {
                bat """
                    @echo off
                    ${PYTHON_PATH} -m pip install --upgrade pip
                    ${PYTHON_PATH} -m pip install -r requirements.txt
                """
            }
        }
        stage('Lint') {
            steps {
                bat "${PYTHON_PATH} -m pip install flake8 && ${PYTHON_PATH} -m flake8 app.py tests/"
            }
        }
        stage('Test') {
            steps {
                bat """
                    ${PYTHON_PATH} -m pip install pytest
                    ${PYTHON_PATH} -m pytest --cov=app tests/ --cov-report=html
                """
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        stage('Build') {
            steps {
                bat """
                    ${PYTHON_PATH} -m pip install pyinstaller
                    ${PYTHON_PATH} -m PyInstaller --onefile app.py
                """
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                bat "start ${PYTHON_PATH} app.py"
            }
        }
    }
    post {
        success { echo 'CI/CD pipeline completed successfully!' }
        failure { echo 'CI/CD pipeline failed!' }
    }
}
