## Задание 1. 
### 1. Минимальная структура проекта

```
textlib/
│   __init__.py          
│   core.py              
tests/
│   conftest.py          # общие фикстуры/параметры
│   test_reverse.py
│   test_vowels.py
│   test_palindrome.py
pytest.ini               # настройки + репорт
requirements.txt
README.md
```

### 2. requirements.txt

```text
pytest>=8.0
pytest-cov     # отчёт о покрытии
pytest-html    # красивый HTML-отчёт 
```

### 3. тest\_reverse.py

```python
import pytest
from textlib.core import reverse_string

@pytest.mark.parametrize(
    'original,expected',
    [
        ('',             ''),               # пустая строка
        ('a',            'a'),              # один символ
        ('hello',        'olleh'),
        ('Привет, мир!', '!рим ,тевирП'),
        ('  spaced  ',   '  decaps  '),     # пробелы сохраняются
    ],
    ids=['empty', 'single', 'latin', 'cyrillic', 'spaces']
)
def test_reverse_string(original, expected):
    assert reverse_string(original) == expected
```

### 4. test\_vowels.py

```python
import pytest
from textlib.core import count_vowels

@pytest.mark.parametrize(
    'text,expected',
    [
        ('',                  0),
        ('AEIOUaeiou',        10),     # регистр
        ('Python',            1),
        ('Мама мыла раму',    5),      # русские гласные
        ('123_!?',            0),
    ]
)
def test_count_vowels(text, expected):
    assert count_vowels(text) == expected
```

### 5. test\_palindrome.py

```python
import pytest
from textlib.core import is_palindrome

@pytest.mark.parametrize(
    'text,expected',
    [
        ('',        True),   
        ('a',       True),
        ('aba',     True),
        ('abba',    True),
        ('A man Panama',  True),
        ('абба',    True),
        ('hello',   False),
    ]
)
def test_is_palindrome(text, expected):
    assert is_palindrome(text) is expected
```

### 6. pytest.ini 

```ini
[pytest]
addopts = -ra -q --cov=textlib --cov-report=term-missing
console_output_style = classic
filterwarnings =
    error
```

### 7. Запуск

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pytest                  # обычный запуск
pytest --html=report.html  # если нужен HTML отчёт
```
---
## Задание 2. 

**Приложение:** Mail.ru Teams (biz.mail.ru/teams).
**Среда:** Windows 10/11.

### 1. Инструменты

| Цель                                 | Библиотека                                                                                    |   
| ------------------------------------ | --------------------------------------------------------------------------------------------- | 
| Загрузка EXE                         | `requests`, `tqdm`                                                                            |                     
| Автоматизация GUI (проверка запуска) | `pywinauto` (Win32)                                                                           |               
| Тест-раннер                          | `pytest`                                                                                      |                    
| Логи/репорт                          | `pytest-html` + сохранение `%APPDATA%\*` логов                                                |                       

### 2. Установка

```python
# tests/conftest.py
import os
import subprocess
import requests
import pytest
from pathlib import Path

DOWNLOAD_URL = "https://dl.mail.ru/teams/windows/teams.exe"
INSTALLER = Path(__file__).parent / "teams.exe"
INSTALL_DIR = Path(os.getenv("ProgramFiles")) / "Mail.Ru" / "Teams"

@pytest.fixture(scope="session")
def installed_app(tmp_path_factory):
    # скачиваем
    if not INSTALLER.exists():
        r = requests.get(DOWNLOAD_URL, stream=True, timeout=120)
        r.raise_for_status()
        with open(INSTALLER, "wb") as f:
            for chunk in r.iter_content(1024 * 1024):
                f.write(chunk)
    subprocess.run([str(INSTALLER), "/S"], check=True)
    yield INSTALL_DIR
    # деинсталляция 
    uninstaller = INSTALL_DIR / "uninstall.exe"
    if uninstaller.exists():
        subprocess.run([str(uninstaller), "/S"], check=True)
```

### 3. Тест запуска

```python
# tests/test_install_run.py
import subprocess
import time
from pywinauto import Application

def test_teams_runs(installed_app):
    exe = installed_app / "teams.exe"
    proc = subprocess.Popen([str(exe)])
    time.sleep(5)                     # даём время на старт
    app = Application().connect(path=str(exe))
    window = app.top_window()
    assert "Teams" in window.window_text()
    proc.terminate()
```
### 4. Отчёт

* `pytest-html` кладёт `report.html`, где видно шаг установки и ассёрты.

### 5. Запуск

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pytest                  # обычный запуск
pytest --html=report.html  # если нужен HTML отчёт
```
