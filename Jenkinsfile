pipeline {
    agent any

    tools {
    // ТОЧНО ОДНО ИЗ ЭТИХ:
    'hudson.plugins.sonar.SonarRunnerInstallation' 'SonarQube Scanner'
    // ИЛИ
    // 'hudson.plugins.sonar.SonarRunnerInstallation' 'SonarQube Scanner'
}
    
    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Клонирование репозитория...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Wotfisher/system-design-primer.git',
                        credentialsId: ''
                    ]]
                ])
                sh 'echo "Репозиторий успешно склонирован"'
                sh 'ls -la'
            }
        }

        stage('Static Analysis') {
            steps {
                echo '🔍 Статический анализ кода...'
                sh '''
                echo "=== Анализ структуры проекта ==="
                echo "Всего файлов: $(find . -type f | wc -l)"
                echo "Python файлов: $(find . -name "*.py" | wc -l)"
                echo "Markdown файлов: $(find . -name "*.md" | wc -l)"
                echo "JavaScript файлов: $(find . -name "*.js" | wc -l)"
                
                echo "=== Поиск потенциальных уязвимостей ==="
                echo "1. Поиск hardcoded паролей..."
                grep -r -i "password\\|passwd\\|secret\\|token" --include="*.py" --include="*.js" --include="*.json" . || echo "Не найдено"
                
                echo "2. Поиск SQL запросов..."
                find . -type f -name "*.py" -exec grep -l "SELECT\\|INSERT\\|UPDATE\\|DELETE" {} \\; || echo "Не найдено"
                
                echo "3. Проверка конфигурационных файлов..."
                find . -name "*.env*" -o -name "*.config*" -o -name "*.conf*" | head -10
                '''
            }
        }

        stage('Simple Tests') {
            steps {
                echo '🧪 Простое тестирование...'
                sh '''
                echo "=== Тест 1: Проверка README ==="
                if [ -f "README.md" ]; then
                    echo "README.md существует"
                    echo "Размер: $(wc -l < README.md) строк"
                else
                    echo "ОШИБКА: README.md не найден"
                fi
                
                echo "=== Тест 2: Проверка структуры ==="
                if [ -d "solutions" ] || [ -d "src" ] || [ -d "lib" ]; then
                    echo "Структура проекта корректна"
                else
                    echo "Предупреждение: нестандартная структура"
                fi
                
                echo "=== Тест 3: Проверка зависимостей ==="
                if [ -f "requirements.txt" ]; then
                    echo "Найдены Python зависимости:"
                    head -20 requirements.txt
                elif [ -f "package.json" ]; then
                    echo "Найдены Node.js зависимости"
                else
                    echo "Файлы зависимостей не найдены"
                fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '🛡️ Запуск анализа SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=system-design-primer-audit \
                      -Dsonar.projectName="System Design Primer Audit" \
                      -Dsonar.projectVersion=1.0 \
                      -Dsonar.sources=. \
                      -Dsonar.sourceEncoding=UTF-8 \
                      -Dsonar.host.url=http://sonarqube:9000 \
                      -Dsonar.login=squ_f0f2295a2535257442dad1ec568848439415a74e
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline успешно завершён!'
            sh 'echo "Результаты анализа доступны в SonarQube"'
        }
        failure {
            echo '❌ Pipeline завершился с ошибкой'
        }
        always {
            echo '📊 Сборка завершена'
            archiveArtifacts artifacts: '**/*.txt, **/*.log', allowEmptyArchive: true
        }
    }
}
