```YAML
version: '3.8'

services:
  # 1. Сервис Ollama: Хранит модели LLM и векторную базу данных
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    hostname: ollama
    # Открываем порт только внутри сети Docker, если не нужен прямой доступ извне
    # Если нужен прямой доступ, раскомментируйте: - "11434:11434"
    volumes:
      # Постоянное хранение моделей и данных Ollama
      - ollama_models:/root/.ollama
      # Используйте GPU, если доступно (раскомментируйте строки ниже)
      # deploy:
      #   resources:
      #     reservations:
      #       devices:
      #         - driver: nvidia
      #           count: all
      #           capabilities: [gpu]
    restart: always

  # 2. Сервис Open WebUI: Веб-интерфейс с поддержкой RAG
  open-webui:
    image: ghcr.io/open-webui/open-webui:latest
    container_name: open-webui
    hostname: open-webui
    ports:
      # Порт, по которому будет доступен Open WebUI (например, http://ваше-имя-домена:3000)
      - "3000:8080"
    environment:
      # Указываем Open WebUI, как найти Ollama
      - OLLAMA_BASE_URL=http://ollama:11434
      # Настройка базы данных и RAG (по умолчанию используется SQLite, что подходит для начала)
      - DATABASE_URL=sqlite:///app/backend/data/webui.db
      - RAG_ENABLE=True
      # Установите безопасный секретный ключ для шифрования токенов и сессий
      - WEBUI_SECRET_KEY=ВАШ_СИЛЬНЫЙ_СЕКРЕТНЫЙ_КЛЮЧ_ДЛЯ_ПРОДАКШНА
    volumes:
      # Постоянное хранение данных интерфейса (пользователи, настройки, RAG-коллекции)
      - open_webui_data:/app/backend/data
      # Подключаем Docker Socket для использования Buildkit (если это нужно, осторожно!)
      # - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - ollama
    restart: always

# Постоянные тома для сохранения данных
volumes:
  ollama_models:
  open_webui_data:
```

gigachat
```YAML
# Добавьте этот сервис в ваш существующий docker-compose.yml
services:
  # ... (существующие сервисы ollama и open-webui)

  # 3. Сервис GigaChat Adapter: Прокси для GigaChat
  gigachat-adapter:
    image: antonk0/gigachat-adapter:latest
    container_name: gigachat-adapter
    hostname: gigachat-adapter
    ports:
      # Внутренний порт, который будет использоваться Open WebUI
      - "8000:8000"
    environment:
      # ! ОБЯЗАТЕЛЬНО ЗАМЕНИТЕ ЭТИ ЗНАЧЕНИЯ !
      - GIGACHAT_CREDENTIALS=ВАШ_СЕКРЕТ_GIGACHAT  # Ваш токен авторизации (например, из OAuth)
      - BEARER_TOKEN=ВАШ_СЕКРЕТНЫЙ_ТОКЕН_ДЛЯ_WEBUI # Любой секретный токен для аутентификации в Open WebUI
      - GIGACHAT_VERIFY_SSL_CERTS=False # Может понадобиться, если есть проблемы с сертификатами
    restart: always
    # Убедитесь, что Open WebUI и адаптер находятся в одной сети
    depends_on:
      - open-webui
```
