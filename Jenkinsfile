pipeline {
    agent any

    environment {
        TRIVY_SCAN_TARGET = "${env.WORKSPACE}/image1.tar"
    }

    triggers {
        cron '10 22 * * 1,4' // Runs at 22:10 on Monday and Thursday
    }

    stages {
        stage('Trivy Image Scan') {
            steps {
                sh '''
                set -e  # Exit on any error

                # Check if Trivy is already installed
                if ! command -v trivy > /dev/null; then
                  echo "Installing Trivy..."

                  # Define installation directory
                  INSTALL_DIR=/tmp/trivy-install
                  TRIVY_TAR=/tmp/trivy.tar.gz

                  # Download Trivy
                  mkdir -p ${INSTALL_DIR}
                  echo "Downloading Trivy from GitHub..."
                  curl -fsSL -o ${TRIVY_TAR} https://github.com/aquasecurity/trivy/releases/download/v0.69.2/trivy_0.69.2_Linux-64bit.tar.gz

                  # Verify download succeeded and extract
                  if [ -f ${TRIVY_TAR} ]; then
                    echo "Extracting Trivy..."
                    tar -xzf ${TRIVY_TAR} -C ${INSTALL_DIR}
                    rm -f ${TRIVY_TAR}
                  else
                    echo "ERROR: Failed to download Trivy"
                    exit 1
                  fi

                  # Move trivy binary to current directory and make it executable
                  mv ${INSTALL_DIR}/trivy ./
                  chmod +x ./trivy
                  rm -rf ${INSTALL_DIR}
                fi

                # Scan tarball with Trivy and save SARIF report
                ./trivy image --input ${TRIVY_SCAN_TARGET} --format sarif --output trivy-results.sarif || true
                '''
            }
        }
        stage('Security Scan') {
            steps {
                registerSecurityScan(
                    // Security Scan to include
                    artifacts: "trivy-results.sarif",
                    format: "sarif",
                    archive: true
                )
            }
        }
    }
}
