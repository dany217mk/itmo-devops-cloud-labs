# 3 Лабораторная  
## Требования

1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. Прочитать историю про Васю (она быстрая, забавная и того стоит): https://habr.com/ru/articles/689234/

# CI/CD это кто?
![img.png](assets/img.png)

CI/CD — это способ автоматизировать процесс создания, проверки и доставки кода до пользователей, чтобы все работало быстро и без ошибок. CI (Continuous Integration) — это когда разработчики регулярно добавляют изменения в общий проект, а система автоматически проверяет, не сломалось ли что-то. CD (Continuous Delivery) — это следующий этап, где всё настроено так, чтобы изменения автоматически проходили тесты и могли быть сразу отправлены пользователям. Это как конвейер: написал, проверил, отправил — всё без лишней возни вручную.

## 1. Плохой CI/CD файл  
![img.png](assets/img1.png)

Мы создадим CI/CD файл с убогими практиками, которые ну вообще не очень.  

```yaml
name: Bad CI/CD!

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y python3 python3-pip

      - name: Add test placeholder(та самая заглушка) 
        run: |
          mkdir -p tests
          echo "def test_placeholder(): assert True" > tests/test_placeholder.py
      
      - name: Run tests
        run: |
          pip install pytest
          pytest tests/
      
      - name: Add source placeholder
        run: |
          mkdir -p src
          echo "print('Hello, World!')" > src/main.py
          
      - name: Build and deploy
        run: |
          echo "Building application..."
          mkdir dist
          cp -r src/* dist/
          echo "Deploying application..."
          echo "Deployed successfully!"
```
### 1.1 Использование ubuntu-latest
Использование ubuntu-latest может привести к проблемам совместимости, так как версия системы может измениться.

### 1.2 Отсутствие версионирования в checkout
Указание только `@v2` вместо конкретной версии может привести к тому, что при изменении самого действия что-то сломается. Лучше явно указать версию или использовать зафиксированный коммит, чтобы всегда получать одинаковый результат.

### 1.3 Неоптимальная установка зависимостей
Использование `apt-get` на каждом запуске — плохая идея, потому что это медленно, и GitHub Actions не сможет закэшировать эти шаги. Из-за этого каждый раз ты теряешь время. Да и вообще, использование `apt-get` менее гибкое решение, которое не позволяет точно контроллировать версию python/

### 1.4 Работа без изоляции
Установка зависимостей и запуск тестов без виртуального окружения `venv` — плохая темка. Это может вызвать конфликты между версиями библиотек, особенно если ты переносишь проект между разными машинами или окружениями.

### 1.5 Все в одном шаге
Сборка и деплой в одном месте — это дурдом какой-то. Если что-то пойдет не так, будет сложно понять, где именно ошибка. Лучше разделять задачи, чтобы каждая выполняла свою функцию.

### 1.6 Нет проверки ошибок
Даже если тесты упадут, пайплайн все равно продолжит работать. Это очень опасно, потому что деплой может произойти с нерабочим кодом. Нужно явно проверять статус каждого шага.

### Проблемы и ошибки при выполнении
В первый раз мы запушили плохой файл на гитхаб и он выдал ошибку:
![img.png](assets/error.png)
Это фиксится добавлением `sudo`
####
Далее вылезла ошибка об отсутствии тестиков, мы добавили заглушку.
![img_1.png](assets/img2.png)
####
И вот, спустя куеву хучу ошибок, оно заработало 🎉🎉🎉
![img.png](assets/img3.png)

## 2. Хороший CI/CD файл
![computer-break-computer.gif](assets%2Fcomputer-break-computer.gif)

Теперь, когда плохой файл готов, самое время сделать легендарный, потрясающий, красивейший, хороший CI/CD файл.

```yaml
name: Good CI/CD

on:
  push:
    branches:
      - main
      - lab-3

jobs:
  build:
    runs-on: ubuntu-20.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Add test placeholder
        run: |
          mkdir -p tests
          echo "def test_placeholder(): assert True" > tests/test_placeholder.py

      - name: Run tests
        run: |
          pip install pytest
          pytest tests/ --maxfail=1

      - name: Add source placeholder
        run: |
          mkdir -p src
          echo "print('Hello, World!')" > src/main.py

      - name: Build application
        run: |
          echo "Building application..."
          mkdir -p dist
          if [ -d "src" ]; then
            cp -r src/* dist/
          else
            echo "Warning: 'src/' directory not found. Skipping copy step."
          fi

      - name: Deploy application
        run: |
          echo "Deploying application..."
          echo "Deployed successfully!"
```

### 2.1 Использование конкректной версии `ubuntu`
 Использована конкректная версия `ubuntu` что исключит конфликты разных версий.

### 2.2 Использование актуальных версий GitHub Actions
 Заменены версии `actions/checkout@v2` на `actions/checkout@v3` и `actions/setup-python@v4`, чтобы использовать последние стабильные версии.

### 2.3 Полный контроль за версией python
 Вместо `apt-get` используется `actions/setup-python@v4`, которая позволяет задать точную версию Python и предоставляет оптимизированное окружение для выполнения пайплайна.

### 2.4 Работаем в изоляции
 В обновленном файле используется `actions/setup-python@v4`, которая автоматически создает изолированное окружение.

### 2.5 Все теперь не в одном шаге
 Сборка `Build application` и деплой `Deploy application` разделены на два отдельных шага.

### 2.6 Нормальная обработка ошибок
 Тесты `Run tests` завершаются с ошибкой, если какой-либо тест падает, благодаря встроенному поведению pytest. Также добавлен флаг --maxfail=1, чтобы пайплайн останавливался при первой ошибке.

 Каждый шаг четко фиксирует свой статус, а деплой запускается только после успешного выполнения предыдущих шагов.
### Запускаем темку
В этот раз (Слава Богу) все заработало с первого раза. А как известно, если все заработало, то доволен программист, недоволен компьютер (ему работать) => можно идти пить яблочный сок.

<p align="center">
    <img src="assets/img4.png">
    <img src="assets/dantedance.gif">
</p>