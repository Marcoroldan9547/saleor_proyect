pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }
  environment {
    SONARQUBE_ENV = 'sonarqube'   // Nombre EXACTO que configuraste en Manage Jenkins → System
    SCANNER = 'SonarScanner'      // Nombre del tool en Global Tool Configuration
    VENV = '.venv'                // Carpeta para el virtualenv de Python
  }
  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Setup Python & Deps') {
      steps {
        bat """
          py -3 --version
          py -3 -m venv %VENV%
          call %VENV%\\Scripts\\activate
          python -m pip install --upgrade pip
          if exist requirements.txt pip install -r requirements.txt
          pip install pytest pytest-cov pytest-django coverage flake8
        """
      }
    }

    stage('Lint') {
      steps {
        bat """
          call %VENV%\\Scripts\\activate
          flake8 saleor || exit /b 0
        """
      }
    }

    stage('Tests + Coverage') {
      steps {
        bat """
          call %VENV%\\Scripts\\activate
          pytest -q --maxfail=1 --junitxml=pytest.xml --cov=saleor --cov-report=xml:coverage.xml
        """
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'pytest.xml'
          archiveArtifacts artifacts: 'coverage.xml,pytest.xml', fingerprint: true, onlyIfSuccessful: false
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_ENV}") {
          withEnv(["PATH+SCAN=${tool(SCANNER)}\\bin"]) {
            bat 'sonar-scanner.bat -Dsonar.projectKey=saleor-backend'
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') error "Quality Gate: ${qg.status}"
          }
        }
      }
    }

    stage('Deploy (by branch)') {
      when { anyOf { branch 'DEV'; branch 'QA'; branch 'PROD' } }
      steps {
        script {
          if (env.BRANCH_NAME == 'DEV')  { echo 'Deploy a DEV (placeholder)'  }
          if (env.BRANCH_NAME == 'QA')   { echo 'Deploy a QA (placeholder)'   }
          if (env.BRANCH_NAME == 'PROD') { echo 'Deploy a PROD (placeholder)' }
        }
      }
    }
  }
  post { always { cleanWs() } }
}
