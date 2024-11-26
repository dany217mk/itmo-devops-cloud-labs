# 3 Лабораторная  
## 1. Плохой CI/CD файл  

Мы создадим CI/CD файл с убогими практиками, которые ну вообще не очень.  

```yaml
name: Bad CI/CD Pipeline

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        run: |
          sudo apt-get update -y
          sudo apt-get install git -y
          git clone https://github.com/${{ github.repository }} repo
      - name: Debug workspace
        run: |
          pwd
          ls -al
      - name: Run tests
        run: echo "Skipping tests..."
      - name: Build application
        run: echo "Skipping build..."
      - name: Deploy to production
        run: echo "Deploying directly to production without checks!"
```
### 1.1 Использование ubuntu-latest
Использование ubuntu-latest может привести к проблемам совместимости, так как версия системы может измениться.

### 1.2 Пропуск тестов
Этап тестирования полностью пропущен (echo "Skipping tests..."), что делает процесс ненадежным.

### 1.3 Пропуск этапа сборки
Сборка также пропущена (echo "Skipping build..."), что делает пайплайн практически бесполезным.

### 1.4 Прямой деплой в production
Код деплоится напрямую без проверки ветки или дополнительных условий (echo "Deploying directly to production without checks!").

### 1.5 Отсутствие логики обработки ошибок
Нет проверок или обработки ошибок на любом из этапов. Если что-то пойдет не так, пайплайн завершится успешно, даже если результат некорректный.


## 2. Хороший CI/CD файл

Теперь, когда плохой файл готов (запускать его, пожалуй не будем) самое время сделать легендарный, потрясающий, красивейший, хороший CI/CD файл.

```yaml
name: Good CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        run: |
          sudo apt-get update -y
          sudo apt-get install git -y
          git clone https://github.com/${{ github.repository }} repo
          cd repo
      - name: Debug workspace
        run: |
          pwd
          ls -al
      - name: Run tests
        run: |
          cd repo
          # команды тестирования проекта
          echo "Running tests..."
          echo "All tests passed!"
      - name: Build application
        run: |
          cd repo
          # команды сборки проекта
          echo "Building the application..."
          echo "Build completed!"
      - name: Deploy to production
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: |
          cd repo
          # команды деплоя
          echo "Deploying to production..."
```

### 2.1 Использование конкректной версии `ubuntu`
 Использована конкректная версия `ubuntu` что исключит конфликты разных версий.

### 2.2 Использование актуальной версии `actions/checkout@v1`
 Использована актуальная версия `actions/checkout@v3`.

### 2.3 Установка Node.js и npm внутри CI/CD пайплайна
 Использован `actions/setup-node@v3` для предварительной настройки Node.js.

### 2.4 Использование `npm ci` вместо `npm install`
 Заменено на `npm ci`, что гарантирует установку зависимостей из `package-lock.json`.

### 2.5 Пропуск тестов
Добавлен этап запуска тестов.

### 2.6 Прямой деплой в production
 Добавлено условие деплоя только из ветки `main` и только при `push` событии.

## З Запускаем темки

![](image.png)

![](image-1.png)
