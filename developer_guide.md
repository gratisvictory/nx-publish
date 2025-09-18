# 🚀 Автоматический релиз - Инструкция для разработчиков

## 📋 Что происходит автоматически

**После правильного коммита в `main`:**
1. ✅ **GitHub Actions автоматически запускается**
2. ✅ **Анализирует ваши коммиты** (conventional commits)
3. ✅ **Определяет какие проекты изменились** (affected)  
4. ✅ **Выбирает тип версии** (patch/minor/major на основе коммитов)
5. ✅ **Обновляет версии** в package.json
6. ✅ **Генерирует CHANGELOG.md**
7. ✅ **Создает git теги** (v1.2.3)
8. ✅ **Коммитит изменения** обратно в репозиторий
9. ✅ **Собирает проекты**
10. ✅ **Публикует в npm**
11. ✅ **Создает GitHub Release**

**Вам нужно только правильно коммитить!**

---

## 🎯 Правила коммитов (Conventional Commits)

### **Типы коммитов и что они делают:**

```bash
# 🔧 PATCH версия (0.0.X) - багфиксы
git commit -m "fix(my-app): resolve login redirect issue"
git commit -m "fix(ui-lib): button hover state"

# ✨ MINOR версия (0.X.0) - новые фичи  
git commit -m "feat(my-app): add dark theme support"
git commit -m "feat(core-lib): add new validation helper"

# 💥 MAJOR версия (X.0.0) - breaking changes
git commit -m "feat!(auth-lib): change login API interface"
git commit -m "fix!(utils): remove deprecated parseDate method"

# 🚫 НЕ создает релиз - служебные коммиты
git commit -m "docs(readme): update installation guide"
git commit -m "test(core): add unit tests for validator"
git commit -m "chore: update dependencies" 
git commit -m "ci: fix workflow configuration"
git commit -m "style(lint): fix formatting"
git commit -m "refactor(utils): improve code structure"
```

### **Формат коммита:**
```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Примеры:**
```bash
# Простой
feat(dashboard): add user analytics widget

# С телом и breaking change
feat(api)!: redesign authentication system

BREAKING CHANGE: The login method now requires email instead of username.
Users need to update their login calls from login(username) to login(email).

# Множественные изменения
feat(ui): add new button variants

- Add primary, secondary, danger variants
- Include hover and focus states  
- Update Storybook documentation
```

---

## 🛠 Workflow для разработчика

### **1. Обычная разработка:**
```bash
# Создаем ветку
git checkout -b feature/user-profile

# Работаем...
# Изменяем файлы в apps/my-app, libs/ui-components, etc.

# Коммитим ПРАВИЛЬНО (это ключевое!)
git add .
git commit -m "feat(my-app): add user profile page"

# Пушим
git push origin feature/user-profile

# Создаем PR, ревью, мержим в main
```

### **2. После мержа в main - всё автоматом!**
```bash
# Через ~2-3 минуты в GitHub Actions увидите:
# ✅ Auto Release workflow completed
# ✅ Packages published  
# ✅ New release v1.2.0 created
```

### **3. Помощники для правильных коммитов:**

**Способ А: Интерактивный помощник**
```bash
# Установите один раз (после настройки)
npm run commit

# Вам зададут вопросы:
# ? Select the type of change: feat
# ? What is the scope: my-app  
# ? Write a short description: add user profile page
```

**Способ Б: Ручные коммиты (если знаете формат)**
```bash
git commit -m "feat(my-app): add user profile page"
```

---

## 🔍 Проверка перед коммитом

### **Автоматические проверки (Husky hooks):**
- **Pre-commit**: линтеры, форматеры, сборка
- **Commit-msg**: проверка формата коммита
- **Pre-push**: тесты

### **Ручная проверка:**
```bash
# Посмотреть что изменилось
npm run show:affected

# Dry run релиза (безопасно)
npm run release:dry

# Проверить последние коммиты
npm run show:commits-since-release

# Валидировать коммиты 
npm run validate:commits
```

---

## 🎨 Шпаргалка по типам изменений

| Тип изменения | Conventional Commit | Версия | Пример |
|---------------|-------------------|---------|---------|
| Новая фича | `feat(scope): ...` | minor | `feat(auth): add 2FA` |
| Багфикс | `fix(scope): ...` | patch | `fix(ui): button alignment` |
| Breaking change | `feat!(scope): ...` | major | `feat!(api)!: new auth` |
| Улучшение производительности | `perf(scope): ...` | patch | `perf(db): optimize queries` |
| Откат изменений | `revert(scope): ...` | patch | `revert: add feature X` |
| Документация | `docs(scope): ...` | ❌ нет | `docs: update README` |
| Тесты | `test(scope): ...` | ❌ нет | `test: add unit tests` |
| Рефакторинг | `refactor(scope): ...` | ❌ нет | `refactor: improve code` |
| Стили/форматирование | `style(scope): ...` | ❌ нет | `style: fix linting` |
| Настройки CI/CD | `ci(scope): ...` | ❌ нет | `ci: update workflow` |
| Обновление зависимостей | `chore(scope): ...` | ❌ нет | `chore: update deps` |

---

## ⚠️ Частые ошибки

```bash
# ❌ НЕ ПРАВИЛЬНО - не создаст релиз
git commit -m "fixed bug"
git commit -m "added new feature"  
git commit -m "updated component"

# ✅ ПРАВИЛЬНО - создаст релиз
git commit -m "fix(my-app): resolve login bug"
git commit -m "feat(ui): add new button component"
git commit -m "perf(core): optimize data processing"
```

---

## 🚨 Что делать если что-то пошло не так

### **Релиз создался, но неправильная версия:**
```bash
# Откатить последний тег локально
git tag -d v1.2.0
git push --delete origin v1.2.0

# Исправить коммит и запустить релиз заново
npm run release:auto
```

### **Нужно срочно остановить автоматический релиз:**
```bash
# Добавить [skip ci] в коммит
git commit -m "hotfix: critical bug [skip ci]"

# Или переименовать/удалить workflow временно в GitHub
```

### **Проверить что будет релизиться:**
```bash
# Всегда можно проверить dry run
npm run release:dry
```

---

## 🎯 Итого: Ваши действия

1. **Работаете как обычно** в своих ветках
2. **Коммитите правильно**: `feat(scope): description`
3. **Мержите в main** через PR
4. **Всё остальное происходит автоматически!**

**Система сама определит:**
- 📦 Какие пакеты релизить  
- 📈 Какую версию поставить
- 🔗 Какие зависимости обновить
- 📝 Что написать в CHANGELOG
- 🚀 Когда опубликовать