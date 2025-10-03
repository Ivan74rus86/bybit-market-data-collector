# Получение данных свечей Bybit

[![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Комплексный Python инструмент для получения и обработки исторических данных свечей с криптовалютной биржи Bybit. Инструмент автоматически загружает, очищает и сжимает рыночные данные для всех доступных торговых пар.

## Особенности

- **Автоматическое получение данных**: Загружает исторические данные свечей для всех USDT торговых пар
- **Обработка данных**: Очищает и валидирует данные, удаляет дубликаты, заполняет пропущенные временные интервалы
- **Сжатие**: Конвертирует данные в эффективный формат Parquet со сжатием ZSTD
- **Возможность возобновления**: Продолжает с того места, где остановился, если процесс прерван
- **Обработка ошибок**: Надежная обработка ошибок с автоматическими механизмами повтора
- **Отслеживание прогресса**: Отображение прогресса в реальном времени во время сбора данных
- **Поддержка разных типов торговли**: Спотовая торговля и деривативы

## Установка

### Предварительные требования

- Python 3.7 или выше
- Стабильное интернет-соединение
- Минимум 10 ГБ свободного места на диске

### Настройка

1. Клонируйте репозиторий:
```bash
git clone https://github.com/Ivan74rus86/bybit-market-data-collector.git
cd bybit-market-data-collector
```

2. Установите зависимости:
```bash
pip install -r requirements.txt
```

## Использование

### Базовое использование

Запустите скрипт для загрузки данных всех доступных торговых пар:

```bash
python main.py
```

### Расширенное использование

Укажите категорию торговли:

```bash
# Для спотовой торговли
python main.py spot

# Для торговли деривативами
python main.py linear
```

## Структура проекта

```
bybit-market-data-collector/
├── main.py              # Основной скрипт для получения данных
├── preprocessing.py     # Модуль для обработки и сжатия данных
├── requirements.txt     # Зависимости Python
├── README.md           # Документация
├── data/               # Сырые CSV файлы (создается автоматически)
└── compressed/         # Сжатые Parquet файлы (создается автоматически)
```

## Выходные данные

Скрипт создает две директории:

- `data/` - Сырые CSV файлы с историческими данными
- `compressed/` - Сжатые Parquet файлы

### Формат данных

Каждый файл содержит следующие колонки:

- `open_time` - Время открытия свечи (Unix timestamp в миллисекундах)
- `open` - Цена открытия
- `high` - Наивысшая цена
- `low` - Наинизшая цена
- `close` - Цена закрытия
- `volume` - Объем торгов
- `quote_asset_volume` - Объем базового актива

## Конфигурация

### Настройки по умолчанию

- **Временной интервал**: 10 минут
- **Дата начала**: 1 января 2022
- **Сжатие**: ZSTD уровень 7
- **Размер пакета**: 1000 свечей за запрос
- **Таймаут запроса**: 30 секунд
- **Задержка при ошибках**: 5 минут

### Настройка

Вы можете изменить следующие переменные в `main.py`:

```python
# Временной интервал в минутах
interval = '10'

# Временная метка начала (по умолчанию 5 минут назад)
START_TIME = (datetime.now() - timedelta(hours=0, minutes=5))

# API endpoint
API_BASE = 'https://api.bybit.com/v5/'

# Размер пакета для запросов
limit = 1000
```

## Обработка данных

Модуль `preprocessing.py` предоставляет следующие функции:

- `quick_clean()` - Удаляет дубликаты и сортирует данные
- `process_raw_to_1m()` - Конвертирует данные в 1-минутные интервалы
- `add_missing_minutes()` - Заполняет пропущенные временные интервалы
- `write_raw_to_parquet()` - Записывает данные в сжатый формат Parquet
- `set_dtypes_compressed()` - Оптимизирует типы данных для экономии памяти

## Обработка ошибок

Скрипт включает надежную обработку ошибок для:

- Таймаутов соединения
- Сетевых прерываний
- Ограничений скорости API
- Неверных ответов от API

При возникновении ошибок скрипт будет:

1. Ждать 5 минут перед повторной попыткой
2. Продолжать с последней успешной точки
3. Записывать детали ошибки в консоль

## Производительность

- **Эффективное использование памяти**: Использует потоковую обработку данных
- **Экономия места**: Сжатие Parquet уменьшает размеры файлов на ~80%
- **Скорость**: Обрабатывает данные последовательно с оптимальными задержками
- **Надежность**: Автоматическое возобновление при сбоях

## Ограничения скорости API

Скрипт соблюдает ограничения скорости API Bybit путем:

- Реализации задержек между запросами
- Корректной обработки ответов об ограничении скорости
- Автоматического повтора с экспоненциальной задержкой

## Устранение неполадок

### Распространенные проблемы

1. **Ошибки соединения**
   - Проверьте интернет-соединение
   - Проверьте доступность API endpoint Bybit
   - Попробуйте запустить скрипт позже

2. **Проблемы с памятью**
   - Уменьшите размер пакета в функции `get_batch()`
   - Обрабатывайте данные меньшими порциями

3. **Недостаток места на диске**
   - Следите за доступным местом на диске
   - Используйте сжатие для долгосрочного хранения
   - Удалите старые CSV файлы после создания Parquet

4. **Медленная работа**
   - Проверьте скорость интернет-соединения
   - Убедитесь, что нет ограничений со стороны провайдера
   - Рассмотрите возможность запуска в облаке

## Вклад в проект

Вклад в проект приветствуется! Пожалуйста:

1. Форкните репозиторий
2. Создайте ветку для новой функции (`git checkout -b feature/amazing-feature`)
3. Внесите изменения
4. Добавьте тесты, если применимо
5. Отправьте pull request

## Лицензия

Этот проект лицензирован под лицензией MIT - см. файл [LICENSE](LICENSE) для деталей.

## Отказ от ответственности

Этот инструмент предназначен только для образовательных и исследовательских целей. Пожалуйста, соблюдайте условия обслуживания Bybit и политики использования API. Авторы не несут ответственности за неправильное использование этого инструмента.

## Поддержка

Если у вас возникли проблемы или вопросы:

- Откройте issue на GitHub
- Проверьте документацию
- Изучите комментарии в коде
- Проверьте раздел "Устранение неполадок"

---

**Примечание**: Этот инструмент требует стабильного интернет-соединения и может занять несколько часов для полной загрузки данных в зависимости от количества торговых пар и скорости вашего соединения.

---

# Bybit Candlestick Data Retriever

[![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A comprehensive Python tool for retrieving and processing historical candlestick data from the Bybit cryptocurrency exchange. The tool automatically downloads, cleans, and compresses market data for all available trading pairs.

## Features

- **Automatic Data Retrieval**: Downloads historical candlestick data for all USDT trading pairs
- **Data Processing**: Cleans and validates data, removes duplicates, fills missing time intervals
- **Compression**: Converts data to efficient Parquet format with ZSTD compression
- **Resume Capability**: Continues from where it left off if interrupted
- **Error Handling**: Robust error handling with automatic retry mechanisms
- **Progress Tracking**: Real-time progress display during data collection
- **Multi-Category Support**: Supports both spot trading and derivatives

## Installation

### Prerequisites

- Python 3.7 or higher
- Stable internet connection
- Minimum 10 GB free disk space

### Setup

1. Clone the repository:
```bash
git clone https://github.com/Ivan74rus86/bybit-market-data-collector.git
cd bybit-market-data-collector
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

## Usage

### Basic Usage

Run the script to download data for all available trading pairs:

```bash
python main.py
```

### Advanced Usage

Specify the trading category:

```bash
# For spot trading
python main.py spot

# For derivatives trading
python main.py linear
```

## Project Structure

```
bybit-market-data-collector/
├── main.py              # Main script for data retrieval
├── preprocessing.py     # Data processing and compression module
├── requirements.txt     # Python dependencies
├── README.md           # Documentation
├── data/               # Raw CSV files (created automatically)
└── compressed/         # Compressed Parquet files (created automatically)
```

## Output

The script creates two directories:

- `data/` - Raw CSV files with historical data
- `compressed/` - Compressed Parquet files

### Data Format

Each file contains the following columns:

- `open_time` - Opening time of the candlestick (Unix timestamp in milliseconds)
- `open` - Opening price
- `high` - Highest price
- `low` - Lowest price
- `close` - Closing price
- `volume` - Trading volume
- `quote_asset_volume` - Quote asset volume

## Configuration

### Default Settings

- **Time Interval**: 10 minutes
- **Start Date**: January 1, 2022
- **Compression**: ZSTD level 7
- **Batch Size**: 1000 candles per request
- **Request Timeout**: 30 seconds
- **Error Retry Delay**: 5 minutes

### Customization

You can modify the following variables in `main.py`:

```python
# Time interval in minutes
interval = '10'

# Start timestamp (default: 5 minutes ago)
START_TIME = (datetime.now() - timedelta(hours=0, minutes=5))

# API endpoint
API_BASE = 'https://api.bybit.com/v5/'

# Batch size for requests
limit = 1000
```

## Data Processing

The `preprocessing.py` module provides the following functions:

- `quick_clean()` - Removes duplicates and sorts data
- `process_raw_to_1m()` - Converts data to 1-minute intervals
- `add_missing_minutes()` - Fills missing time intervals
- `write_raw_to_parquet()` - Writes data to compressed Parquet format
- `set_dtypes_compressed()` - Optimizes data types for memory efficiency

## Error Handling

The script includes robust error handling for:

- Connection timeouts
- Network interruptions
- API rate limiting
- Invalid responses

When errors occur, the script will:

1. Wait 5 minutes before retrying
2. Continue from the last successful point
3. Log error details to console

## Performance

- **Memory Efficient**: Uses streaming data processing
- **Disk Space**: Parquet compression reduces file sizes by ~80%
- **Speed**: Processes data sequentially with optimal delays
- **Reliability**: Automatic resumption on failures

## API Rate Limits

The script respects Bybit's API rate limits by:

- Implementing delays between requests
- Handling rate limit responses gracefully
- Automatic retry with exponential backoff

## Troubleshooting

### Common Issues

1. **Connection Errors**
   - Check internet connection
   - Verify API endpoint accessibility
   - Try running the script later

2. **Memory Issues**
   - Reduce batch size in `get_batch()` function
   - Process data in smaller chunks

3. **Disk Space**
   - Monitor available disk space
   - Use compression for long-term storage
   - Remove old CSV files after creating Parquet

4. **Slow Performance**
   - Check internet connection speed
   - Ensure no provider restrictions
   - Consider running in the cloud

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Disclaimer

This tool is for educational and research purposes only. Please respect Bybit's terms of service and API usage policies. The authors are not responsible for any misuse of this tool.

## Support

If you encounter any issues or have questions:

- Open an issue on GitHub
- Check the documentation
- Review the code comments
- Check the "Troubleshooting" section

---

**Note**: This tool requires a stable internet connection and may take several hours to complete a full data download depending on the number of trading pairs and your connection speed.