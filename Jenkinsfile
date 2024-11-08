def PROJECT_NAME = "Slot-WheelOfFortune"
def UNITY_VERSION = "2022.3.51f1"
def UNITY_INSTALLATION = "C:\\Program Files\\Unity\\Hub\\Editor\\${UNITY_VERSION}\\Editor\\Unity.exe"
def REPO_URL = "git@github.com:DingDingHouse/Slot-WheelOfFortune.git"

pipeline {
    agent any

    options {
        timeout(time: 60, unit: 'MINUTES')
    }

    environment {
        PROJECT_PATH = "C:\\Games\\Slot-WheelOfFortune"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    dir("${PROJECT_PATH}") {
                        bat '''
                        git config --global http.postBuffer 3221225472
                        git clone git@github.com:DingDingHouse/Slot-WheelOfFortune.git C:\\Games\\Slot-WheelOfFortune || echo "Repository already exists, pulling latest changes."
                        cd Slot-WheelOfFortune
                        git checkout main
                        git fetch --all
                        git reset --hard origin/develop
                        git reset --hard origin/main
                        git checkout develop
                        '''
                    }
                }
            }
        }

        stage('Build WebGL') {
            steps {
                script {
                    withEnv(["UNITY_PATH=${UNITY_INSTALLATION}"]) {
                        bat '''
                        "%UNITY_PATH%" -quit -batchmode -projectPath "%PROJECT_PATH%" -executeMethod BuildScript.BuildWebGL -logFile -
                        '''
                    }
                }
            }
        }

        stage('Push Build to GitHub') {
            steps {
                script {
                    dir("${PROJECT_PATH}") {
                        bat '''
                        hostname
                        git stash -u
                        git checkout main
                        git rm -r -f Build
                        git rm -f index.html
                        git commit -m "delete old Builds" || echo "Nothing to commit"
                        git push origin main

                        git checkout main
                        git checkout develop -- Builds
                        robocopy Builds\\WebGL\\ .\\ /move /e /copyall
                        git lfs install
                        git lfs track "*.wasm" "*.data" "*.js"
                        git rm -r -f Builds
                        git add -f Build index.html .gitattributes
                        git commit -m "adding new Builds" || echo "Nothing to commit"
                        git push origin main
                        git checkout develop
                        git pull origin develop
                        '''
                    }
                }
            }
        }
    }
}
