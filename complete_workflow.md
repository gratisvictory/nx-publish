# 🚀 Полный Workflow: От разработки до релиза

## 📋 Этап 1: Настройка (делается один раз)

### 1.1 Обновите nx.json
```bash
# Скопируйте конфигурацию из артефакта выше в ваш nx.json
```

### 1.2 Создайте GitHub Actions workflows
```bash
# Создайте файлы:
.github/workflows/selective-release.yml  # из артефакта выше
```

### 1.3 Добавьте скрипты в package.json
```bash
# Добавьте скрипты из артефакта выше
```

### 1.4 Настройте секреты в GitHub
- `NPM_TOKEN` - токен для публикации в npm
- `NX_CLOUD_ACCESS_TOKEN` (опционально)

---

## 🛠 Этап 2: Разработка (ваш текущий случай)

### 2.1 Создали ветку и работали
```bash
git checkout -b feature/new-app-feature
# Работали в apps/my-app
# Внесли изменения...
git add .
```

### 2.2 ВАЖНО: Правильный коммит
**Вместо обычного коммита, используйте conventional commits:**

```bash
# ❌ Плохо - не поймет что релизить
git commit -m "fixed bug in app"

# ✅ Хорошо - автоматически определит что релизить
git commit -m "fix(my-app): resolve login issue"
git commit -m "feat(my-app): add new dashboard component"
git commit -m "feat(ui-components): add button variants"
```

### 2.3 Пуш ветки
```bash
git push origin feature/new-app-feature
```

---

## 🔄 Этап 3: Pull Request и Merge

### 3.1 Создайте PR
- GitHub покажет какие файлы изменены
- В PR можно увидеть affected проекты

### 3.2 Перед мержем - проверка
```bash
# Посмотреть что будет релизиться
npm run show:affected

# Dry run релиза (локально)
npm run dry-run:affected
```

### 3.3 Мерж в main
```bash
git checkout main
git pull origin main
git merge feature/new-app-feature
git push origin main
```

---

## 🚀 Этап 4: Релиз (несколько вариантов)

### Вариант А: Автоматический релиз (рекомендуется)

**Если коммиты правильные (conventional commits):**
```bash
# Nx автоматически определит:
# - Какие проекты изменились  
# - Какой тип версии нужен (patch/minor/major)
# - Какие зависимые проекты обновить

npm run release:affected
# или
npx nx release --verbose
```

**Что происходит:**
- Анализирует коммиты с последнего релиза
- Определяет affected проекты
- Автоматически выбирает тип версии
- Обновляет версии в package.json
- Создает git теги
- Генерирует CHANGELOG

### Вариант Б: Ручной выбор проектов

```bash
# Если хотите точно контролировать что релизить
npm run release:project:patch -- my-app,ui-components
npm run release:project:minor -- my-app
npm run release:project:major -- core-lib
```

### Вариант В: Через GitHub Actions (в CI/CD)

1. **Идите в GitHub → Actions → Selective Release**
2. **Нажмите "Run workflow"**
3. **Выберите параметры:**
   - Release type: `affected` (найдет автоматически)
   - Version type: `patch/minor/major`
   - Include dependencies: `true`

### Вариант Г: Smart Script (самый удобный)

```bash
# Создайте файл scripts/release.sh из артефакта выше
chmod +x scripts/release.sh

# Автоматически найдет что релизить
./scripts/release.sh patch

# Или конкретные проекты
./scripts/release.sh minor "my-app,ui-components"

# Dry run для проверки
./scripts/release.sh patch "" true true
```

---

## 📦 Этап 5: Публикация

### 5.1 После версионирования - публикация
```bash
# Опубликовать все что было версионировано
npm run release:publish

# Или только конкретные проекты
npm run publish:projects -- my-app,ui-components

# Или только affected
npm run publish:affected
```

### 5.2 В GitHub Actions
Публикация происходит автоматически в том же workflow после версионирования.

---

## 🎯 Практический пример вашего случая

### Сценарий: Работали в apps/my-app

```bash
# 1. Ваши изменения
git add .
git commit -m "feat(my-app): add user profile page"
git commit -m "fix(ui-components): button color contrast"
git push origin feature/profile-page

# 2. После мержа в main - проверка
npm run show:affected
# Покажет: my-app, ui-components (и возможно зависимые проекты)

# 3. Релиз (выберите один способ)

# Способ 1: Автоматический
npm run release:affected
# Результат: my-app получит minor, ui-components получит patch

# Способ 2: Ручной контроль  
npx nx release version minor --projects=my-app --include-dependencies
# Результат: my-app получит minor + все что от него зависит

# Способ 3: Через GitHub Actions
# Перейти в Actions → Run workflow → выбрать "affected" + "minor"

# 4. Публикация
npm run release:publish
```

---

## 🔍 Полезные команды для проверки

```bash
# Посмотреть граф зависимостей
npm run show:graph

# Посмотреть что изменилось
npm run show:affected

# Посмотреть все библиотеки
npm run show:projects

# Dry run любой операции (добавьте --dry-run)
npx nx release --dry-run --verbose

# Посмотреть текущие версии
cat apps/my-app/package.json | grep version
cat libs/ui-components/package.json | grep version
```

---

## 🎨 Conventional Commits шпаргалка

```bash
# Patch версия (0.0.X)
git commit -m "fix(my-app): resolve login issue"
git commit -m "docs(ui-lib): update README"
git commit -m "style(core): format code"

# Minor версия (0.X.0)
git commit -m "feat(my-app): add dark theme"
git commit -m "feat(ui-components): add date picker"

# Major версия (X.0.0)
git commit -m "feat!(core-lib): change API interface"
git commit -m "fix!(utils): remove deprecated methods"

# Не создает релиз
git commit -m "chore: update dependencies"
git commit -m "ci: update workflow"
git commit -m "test: add unit tests"
```

---

## ✅ Рекомендуемый workflow

1. **Разработка**: Используйте conventional commits
2. **Проверка**: `npm run show:affected` и `npm run dry-run:affected`
3. **Релиз**: `./scripts/release.sh` (автоматический) или GitHub Actions
4. **Публикация**: автоматически через workflow или `npm run release:publish`

**Главное преимущество**: система сама поймет что и как релизить на основе ваших коммитов!