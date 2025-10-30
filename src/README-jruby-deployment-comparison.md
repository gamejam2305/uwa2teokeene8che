# Сравнение методов развертывания JRuby в Docker

В этом документе сравниваются три различных подхода к развертыванию JRuby приложений в Docker контейнерах.

## Доступные варианты развертывания

### 1. 🚀 JRuby + Puma (Простой)

**Файлы**: `Dockerfile.jruby`, `start-jruby.sh`, `Gemfile.jruby`

**Плюсы:**
- ✅ Простота конфигурации и отладки
- ✅ Быстрая сборка Docker образа
- ✅ Минимальные зависимости
- ✅ Прямое управление процессом Puma
- ✅ Отлично подходит для разработки и тестирования

**Минусы:**
- ❌ Нет автоматической балансировки нагрузки
- ❌ Ограниченные возможности мониторинга
- ❌ Нет встроенной обработки статических файлов
- ❌ Требует ручной настройки reverse proxy для production

**Использование:**
```bash
# Сборка
docker build -f Dockerfile.jruby -t monitus-jruby .

# Запуск
docker run -p 8080:8080 monitus-jruby
```

**Лучше всего для:** Разработка, тестирование, простые deployments

### 2. 🔄 JRuby + Nginx Reverse Proxy (Рекомендуемый)

**Файлы**: `Dockerfile.jruby-nginx`, `nginx-jruby-proxy.conf`, `supervisord-jruby.conf`

**Плюсы:**
- ✅ Nginx обрабатывает статические файлы
- ✅ Встроенная балансировка нагрузки
- ✅ Supervisor управляет процессами
- ✅ Отличная производительность
- ✅ Легкая масштабируемость
- ✅ Гибкая конфигурация Nginx
- ✅ WebSocket поддержка

**Минусы:**
- ⚠️ Более сложная конфигурация
- ⚠️ Больший размер образа
- ⚠️ Требует понимания Nginx и Supervisor

**Использование:**
```bash
# Сборка
docker build -f Dockerfile.jruby-nginx -t monitus-jruby-nginx .

# Запуск
docker run -p 8080:80 \
  -e PUMA_WORKERS=0 \
  -e PUMA_THREADS_MIN=8 \
  -e PUMA_THREADS_MAX=32 \
  monitus-jruby-nginx
```

**Лучше всего для:** Production развертывания, высоконагруженные системы

### 3. 🏭 JRuby + Phusion Passenger (Промышленный)

**Файлы**: `Dockerfile.jruby-passenger`, `nginx-jruby.conf`, `passenger-jruby.conf`

**Плюсы:**
- ✅ Полноценный application server
- ✅ Автоматическое управление процессами
- ✅ Интеллектуальное масштабирование
- ✅ Встроенный мониторинг
- ✅ Автоматический перезапуск при сбоях
- ✅ Zero-downtime deployments
- ✅ Оптимизирован для production
- ✅ Использует официальные Docker паттерны

**Минусы:**
- ⚠️ Сложная настройка
- ⚠️ Большой размер образа
- ⚠️ Требует глубокого понимания Passenger
- ⚠️ Дольше время сборки

**Использование:**
```bash
# Сборка
docker build -f Dockerfile.jruby-passenger -t monitus-jruby-passenger .

# Запуск
docker run -p 8080:80 \
  -e PASSENGER_MIN_INSTANCES=2 \
  -e PASSENGER_MAX_INSTANCES=8 \
  -e PASSENGER_THREAD_COUNT=16 \
  monitus-jruby-passenger
```

**Лучше всего для:** Крупные production системы, enterprise приложения

## Сравнительная таблица

| Характеристика | Puma | Nginx + Puma | Passenger |
|----------------|------|---------------|------------|
| **Сложность** | 🟢 Простая | 🟡 Средняя | 🔴 Сложная |
| **Размер образа** | 🟢 ~800MB | 🟡 ~900MB | 🔴 ~1.2GB |
| **Время сборки** | 🟢 ~3 мин | 🟡 ~5 мин | 🔴 ~8 мин |
| **Производительность** | 🟡 Хорошая | 🟢 Отличная | 🟢 Отличная |
| **Масштабируемость** | 🔴 Ограниченная | 🟢 Хорошая | 🟢 Отличная |
| **Мониторинг** | 🔴 Базовый | 🟡 Средний | 🟢 Продвинутый |
| **Статические файлы** | 🔴 Нет | 🟢 Nginx | 🟢 Nginx |
| **WebSocket** | 🟢 Да | 🟢 Да | 🟢 Да |
| **Load Balancing** | 🔴 Нет | 🟢 Да | 🟢 Автоматический |
| **Zero Downtime** | 🔴 Нет | 🟡 Частично | 🟢 Да |
| **Подходит для dev** | 🟢 Отлично | 🟡 Хорошо | 🔴 Сложно |
| **Подходит для prod** | 🔴 Ограниченно | 🟢 Отлично | 🟢 Идеально |

## Рекомендации по выбору

### Для разработки и тестирования:
```bash
# Используйте JRuby + Puma
docker run -p 8080:8080 monitus-jruby
```

### Для production (средние нагрузки):
```bash
# Используйте JRuby + Nginx
docker run -p 80:80 monitus-jruby-nginx
```

### Для enterprise production:
```bash
# Используйте JRuby + Passenger
docker run -p 80:80 monitus-jruby-passenger
```

## Конфигурация производительности

### JRuby оптимизации (для всех вариантов):
```bash
# JVM настройки
JAVA_OPTS="-Xmx2G -Xms512M -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# JRuby настройки
JRUBY_OPTS="-Xcompile.invokedynamic=true -J-Djnr.ffi.asm.enabled=false"

# Системные настройки
MALLOC_ARENA_MAX=2
```

### Масштабирование по ресурсам:

**Для 1 CPU core, 1GB RAM:**
```bash
# Puma
PUMA_THREADS_MIN=4
PUMA_THREADS_MAX=16

# Passenger
PASSENGER_MIN_INSTANCES=1
PASSENGER_MAX_INSTANCES=2
PASSENGER_THREAD_COUNT=8
```

**Для 2 CPU cores, 2GB RAM:**
```bash
# Puma
PUMA_THREADS_MIN=8
PUMA_THREADS_MAX=32

# Passenger
PASSENGER_MIN_INSTANCES=2
PASSENGER_MAX_INSTANCES=4
PASSENGER_THREAD_COUNT=16
```

**Для 4+ CPU cores, 4GB+ RAM:**
```bash
# Puma
PUMA_THREADS_MIN=16
PUMA_THREADS_MAX=64

# Passenger
PASSENGER_MIN_INSTANCES=4
PASSENGER_MAX_INSTANCES=8
PASSENGER_THREAD_COUNT=32
```

## Мониторинг и отладка

### Для всех вариантов:
```bash
# Проверка здоровья
curl http://localhost:PORT/health

# Метрики Prometheus
curl http://localhost:PORT/metrics

# Логи контейнера
docker logs container_name
```

### Специфичные команды:

**Puma:**
```bash
# Статистика Puma
docker exec container_name pumactl stats
```

**Passenger:**
```bash
# Статус Passenger
docker exec container_name passenger-status

# Память процессов
docker exec container_name passenger-memory-stats
```

## Docker Compose примеры

В директории `test/` доступны готовые конфигурации:
- `docker-compose-jruby.ci.yaml` - для Puma
- `docker-compose-jruby-passenger.yml` - для Passenger

## Тестирование

Для каждого варианта доступны тестовые скрипты:
- `test/test-docker-jruby.sh` - тест Puma версии
- `test/test-jruby-passenger.sh` - тест Passenger версии

## Заключение

**Выбирайте подход в зависимости от ваших потребностей:**

1. **Начинающие / Разработка** → JRuby + Puma
2. **Production / Средние нагрузки** → JRuby + Nginx
3. **Enterprise / Высокие нагрузки** → JRuby + Passenger

Все три варианта полностью функциональны и оптимизированы для JRuby 9.4.14.0 с Java 17.
