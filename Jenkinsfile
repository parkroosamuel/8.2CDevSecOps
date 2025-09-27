pipeline {
  agent any

  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
    # Path to nvm and Node.js for all shell steps
    NVM_DIR = "${env.HOME}/.nvm"
    NODE_SETUP = '''
      [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
      nvm install --lts
      nvm use --lts
    '''
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/parkroosamuel/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh """
          ${NODE_SETUP}
          npm ci || npm install
        """
      }
    }

    stage('Run Tests') {
      steps {
        sh """
          ${NODE_SETUP}
          npm test || true
        """
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh """
          ${NODE_SETUP}
          npm run coverage || true
        """
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh """
          ${NODE_SETUP}
          npm audit || true
        """
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        sh """
          ${NODE_SETUP}
          set -e
          echo "=== SonarCloud analysis starting ==="
          node -v || true
          npm -v || true
          java -version 2>&1 || true
          pwd
          ls -la

          if [ ! -f coverage/lcov.info ]; then
            echo "WARNING: coverage/lcov.info not found. Sonar will run but coverage metrics will be missing."
          else
            echo "Found coverage/lcov.info"
          fi

          npx --yes sonar-scanner \
            -Dsonar.projectKey=parkroosamuel_8.2CDevSecOps \
            -Dsonar.organization=parkroosamuel \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.login=${SONAR_TOKEN} \
            -Dsonar.sources=. \
            -Dsonar.exclusions=node_modules/**,test/** \
            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info || true

          echo "=== SonarCloud analysis finished ==="
        """
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Check SonarCloud dashboard for analysis results."
    }
  }
}
