pipeline {
  agent any

  environment {
    // Inject the secret text stored in Jenkins (create it with id SONAR_TOKEN).
    // This requires the credential to be a "Secret text" (or compatible) type.
    SONAR_TOKEN = credentials('SONAR_TOKEN')
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/parkroosamuel/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci || npm install'   // npm ci if lockfile exists, fallback to npm install
      }
    }

    stage('Run Tests') {
      steps {
        // continue even if tests fail so we still upload coverage to Sonar
        sh 'npm test || true'
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Ensure coverage report exists; your package.json should have a coverage script
        sh 'npm run coverage || true'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh 'npm audit || true'
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        // This step tries to run sonar-scanner via npx. It falls back to installing
        // sonar-scanner locally if needed. The sonar-project.properties file (in repo)
        // will be read and sonar.login uses the injected SONAR_TOKEN above.
        sh '''
          set -e
          echo "=== SonarCloud analysis starting ==="
          echo "Checking environment..."
          node -v || true
          npm -v || true
          java -version 2>&1 || true
          pwd
          ls -la

          # Ensure coverage file presence - Sonar will want coverage/lcov.info if you configured it
          if [ ! -f coverage/lcov.info ]; then
            echo "WARNING: coverage/lcov.info not found. Sonar will run but coverage metrics will be missing."
          else
            echo "Found coverage/lcov.info"
          fi

          # Run sonar-scanner using npx (preferred: no global install required)
          # We pass key org and host as overrides to be explicit; sonar.login uses injected token.
          npx --yes sonar-scanner \
            -Dsonar.projectKey=parkroosamuel_8.2CDevSecOps \
            -Dsonar.organization=parkroosamuel \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.login=${SONAR_TOKEN} \
            -Dsonar.sources=. \
            -Dsonar.exclusions=node_modules/**,test/** \
            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info || true

          echo "=== SonarCloud analysis finished ==="
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Check SonarCloud dashboard for analysis results."
    }
  }
}

