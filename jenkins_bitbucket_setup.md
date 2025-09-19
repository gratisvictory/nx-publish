# 🔧 Настройка Jenkins + Bitbucket для автоматического релиза

## 📋 1. Подготовка Jenkins

### 1.1 Установка плагинов
```bash
# В Jenkins → Manage Jenkins → Plugins
Установите:
- NodeJS Plugin
- Bitbucket Plugin  
- Pipeline Plugin
- Credentials Plugin
- Slack Notification Plugin (опционально)
- Email Extension Plugin (опционально)
```

### 1.2 Настройка Node.js
```bash
# Jenkins → Manage Jenkins → Tools → NodeJS
Name: Node 18
Version: 18.x.x (последняя LTS)
Global npm packages: nx@latest
```

### 1.3 Настройка Credentials
```bash
# Jenkins → Manage Jenkins → Credentials → Global → Add Credentials

1. NPM Token:
   Kind: Secret text
   ID: npm-token
   Secret: your_npm_automation_token

2. Bitbucket App Password:
   Kind: Username with password
   ID: bitbucket-app-password
   Username: your_bitbucket_username
   Password: your_bitbucket_app_password

3. NX Cloud Token (опционально):
   Kind: Secret text
   ID: nx-cloud-token
   Secret: your_nx_cloud_token
```

## 📋 2. Создание Jenkins Job

### 2.1 Создание Pipeline Job
```bash
# Jenkins Dashboard → New Item
Item name: auto-release-pipeline
Type: Pipeline
OK
```

### 2.2 Настройка Job
```bash
# В конфигурации Pipeline Job:

General:
☑️ Discard old builds
  Days to keep builds: 30
  Max # of builds to keep: 50

Build Triggers:
☑️ Build when a change is pushed to BitBucket
☑️ Poll SCM: H/2 * * * * (проверка каждые 2 минуты)

Pipeline:
  Definition: Pipeline script from SCM
  SCM: Git
  Repository URL: https://bitbucket.org/yourcompany/yourrepo.git
  Credentials: bitbucket-app-password
  Branch Specifier: */main
  Script Path: Jenkinsfile

Advanced:
☑️ Lightweight checkout
```

## 📋 3. Настройка Bitbucket

### 3.1 Создание App Password
```bash
# Bitbucket → Personal settings → App passwords → Create app password
Label: Jenkins Auto Release
Permissions:
☑️ Repositories: Read, Write
☑️ Pull requests: Read
☑️ Webhooks: Read and write

# Сохраните сгенерированный пароль!
```

### 3.2 Настройка Webhook
```bash
# Bitbucket → Repository → Settings → Webhooks → Add webhook
Title: Jenkins Auto Release
URL: http://your-jenkins-url/bitbucket-hook/
Events:
☑️ Repository → Push
☑️ Pull Request → Merged

# Для более точного контроля:
URL: http://your-jenkins-url/job/auto-release-pipeline/build?token=YOUR_BUILD_TOKEN
```

### 3.3 Настройка Branch Permissions (рекомендуется)
```bash
# Bitbucket → Repository → Settings → Branch permissions
Branch: main
Type: Restrict pushes
Restrictions:
☑️ Allow only pull request merges
☑️ Require approval
```

## 📋 4. Обновление конфигураций проекта

### 4.1 Создайте Jenkinsfile в корне проекта
```bash
# Используйте содержимое из артефакта "Jenkinsfile - автоматический релиз"
# НЕ ЗАБУДЬТЕ изменить:
# - yourcompany/yourrepo на ваши реальные значения
# - email адреса для уведомлений
```

### 4.2 Обновите package.json
```bash
# Добавьте/обновите scripts из предыдущих артефактов:
"ci:install": "npm ci",
"ci:test": "npm run lint:affected && npm run test:affected", 
"ci:build": "npm run build:affected",
"ci:release": "npm run release:auto",
"ci:publish": "npm run release:publish"
```

## 📋 5. Первый запуск

### 5.1 Подготовка experimental ветки
```bash
# В вашей experimental ветке:
git add .
npm run commit
# Выберите: feat(ci): setup Jenkins automatic release
git push origin experimental
```

### 5.2 Тестовый мерж
```bash
# Создайте Pull Request: experimental → main
# После аппрува - мерж

# Или локально:
git checkout main
git pull origin main
git merge experimental
git push origin main

# Jenkins должен автоматически запуститься!
```

## 📋 6. Мониторинг и проверка

### 6.1 Проверка в Jenkins
```bash
# Jenkins Dashboard → auto-release-pipeline
# Должны увидеть:
1. Build запустился автоматически
2. Все этапы прошли успешно
3. В Console Output логи процесса
4. В конце сообщение о успешном релизе
```

### 6.2 Проверка результата
```bash
# В Bitbucket:
1. Новые коммиты с chore(release)
2. Новые git теги

# В npm:
npm view your-package-name versions --json

# Локально:
git pull origin main --tags
git tag --sort=-creatordate | head -5
```

## 📋 7. Troubleshooting

### Проблема: Jenkins не запускается при push
```bash
Проверьте:
1. Webhook URL правильный и доступен
2. Bitbucket App Password корректный
3. Credentials в Jenkins настроены правильно
4. Branch Specifier: */main (не main)
```

### Проблема: Git push не работает в Jenkins
```bash
# В Jenkinsfile проверьте что credentials правильные:
withCredentials([usernamePassword(credentialsId: 'bitbucket-app-password', ...

# Проверьте формат URL:
git remote set-url origin https://${BB_USER}:${BB_PASS}@bitbucket.org/workspace/repo.git
```

### Проблема: NPM публикация не работает
```bash
# Проверьте:
1. NPM_TOKEN в Jenkins credentials правильный
2. package.json содержит правильные поля (name, version)
3. Версии уникальные (не опубликованы ранее)
```

## 📋 8. Опциональные улучшения

### 8.1 Slack уведомления
```bash
# Раскомментируйте в Jenkinsfile:
slackSend(
    channel: '#releases',
    message: "🚀 New release ${env.LATEST_TAG} published!"
)
```

### 8.2 Email уведомления
```bash
# Раскомментируйте в Jenkinsfile:
emailext(
    to: 'team@yourcompany.com',
    subject: "✅ Release ${env.LATEST_TAG} Published",
    body: "Automatic release completed successfully."
)
```

### 8.3 Параллельная сборка
```bash
# В Jenkinsfile уже настроена параллельная сборка:
parallel {
    stage('Lint') { ... }
    stage('Tests') { ... }
    stage('Build') { ... }
}
```

## ✅ Финальная проверка

После настройки у вас должно быть:
- ✅ Jenkins Job настроен с правильными credentials
- ✅ Bitbucket webhook настроен
- ✅ Jenkinsfile в корне проекта
- ✅ package.json обновлен с CI скриптами
- ✅ npm token и bitbucket credentials в Jenkins
- ✅ Первый автоматический релиз прошел успешно

**Теперь при каждом push в main → Jenkins автоматически создаст релиз!** 🚀