# Инструкция: подключение удалённого проекта PyCharm через JetBrains Gateway

Цель: открыть удалённую папку проекта на сервере как полноценный PyCharm-проект через **JetBrains Gateway / Remote Development**.

Удалённый проект:

```text
/home/ubuntu/rag-vector-db
```

SSH-подключение:

```text
Host: 62.84.115.245
User: ubuntu
Port: 22
Private key: C:\.ssh\id_ed25519
```

---

## 1. Проверить SSH-доступ

На локальной машине Windows в PowerShell:

```powershell
ssh -i C:\.ssh\id_ed25519 -o IdentitiesOnly=yes ubuntu@62.84.115.245
```

Если подключение успешно, можно проверить проект:

```bash
ls -la /home/ubuntu/rag-vector-db
```

---

## 2. Настройка JetBrains Gateway

1. Включить VPN на Windows.
2. Открыть **JetBrains Gateway**.
3. Выбрать **SSH**-подключение.
4. Указать:

   ```text
   Host: 62.84.115.245
   User: ubuntu
   Port: 22
   Authentication: Key pair
   Private key: C:\.ssh\id_ed25519
   ```

5. После успешного подключения выбрать IDE:

   ```text
   PyCharm / PyCharm Professional
   ```

---


## 3. Установка PyCharm из локального архива

Скачать архив PyCharm на Windows:

```powershell
Invoke-WebRequest `
  -Uri "https://download.jetbrains.com/python/pycharm-2026.2.4.tar.gz" `
  -OutFile "$env:USERPROFILE\Downloads\pycharm-2026.2.4.tar.gz"
```

Проверить размер файла:

```powershell
(Get-Item "$env:USERPROFILE\Downloads\pycharm-2026.2.4.tar.gz").Length
```

В нашем случае размер был:

```text
1298582879 bytes
```

Дальше в **JetBrains Gateway** выполнить следующие действия:

1. Создать или открыть SSH-подключение к серверу.
2. Выбрать удалённый проект:

   ```text
   /home/ubuntu/rag-vector-db
   ```

3. На шаге выбора IDE выбрать PyCharm.
4. Вместо автоматической загрузки с сервера JetBrains выбрать вариант установки из локального архива, например:

   ```text
   Install from local file
   Upload installer file
   Select local archive
   ```

   Точное название пункта может отличаться в зависимости от версии Gateway.

5. Указать локальный файл:

   ```text
   C:\Users\<USER>\Downloads\pycharm-2026.2.4.tar.gz
   ```

6. Продолжить подключение.

Gateway должен сам загрузить этот архив на сервер и распаковать его.

## 4. Возможная финальная ошибка с JetBrains Client

Если backend уже запустился, но Gateway падает с ошибкой:

```text
Cannot download 'https://download.jetbrains.com/idea/code-with-me/JetBrainsClient-262.11879.98.jbr.win.zip'
Connection refused: getsockopt
```

это означает, что локальная Windows-машина не может скачать JetBrains Client.

Решение:

1. Включить VPN на Windows.
2. Проверить доступность URL через PowerShell.
3. Нажать `Retry` в Gateway.

## 5. Подключение репозитория GitHub по SSH

Этот раздел нужен для разворачивания другого проекта из репозитория GitHub.

Все команды ниже выполняются **на удалённом сервере** после подключения по SSH:

```powershell
ssh -i C:\.ssh\id_ed25519 -o IdentitiesOnly=yes ubuntu@62.84.115.245
```

### 5.1. Установить Git

```bash
sudo apt update
sudo apt install -y git
```

Проверить установку:

```bash
git --version
```

### 5.2. Создать SSH-ключ для GitHub

На сервере создать отдельный SSH-ключ для доступа к GitHub:

```bash
ssh-keygen -t ed25519 -C "github-deploy-key" -f ~/.ssh/github_ed25519
```

Если будет задан вопрос про парольную фразу, можно нажать `Enter`, чтобы оставить её пустой.

После создания ключа должны появиться два файла:

```text
~/.ssh/github_ed25519
~/.ssh/github_ed25519.pub
```

Закрытый ключ `~/.ssh/github_ed25519` никому не передавать.

### 5.3. Добавить ключ в SSH config на сервере

Создать или отредактировать файл:

```bash
nano ~/.ssh/config
```

Добавить блок:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
```

Выставить корректные права:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/github_ed25519
chmod 644 ~/.ssh/github_ed25519.pub
```

### 5.4. Добавить публичный ключ в GitHub

Вывести публичный ключ на сервере:

```bash
cat ~/.ssh/github_ed25519.pub
```

Скопировать весь вывод, который начинается с `ssh-ed25519`.

Дальше в GitHub:

1. Открыть `https://github.com/`.
2. Перейти в **Settings**.
3. Открыть **SSH and GPG keys**.
4. Нажать **New SSH key**.
5. В поле **Title** указать, например:

   ```text
   remote-ubuntu-server
   ```

6. В поле **Key** вставить содержимое файла `~/.ssh/github_ed25519.pub`.
7. Нажать **Add SSH key**.

Если ключ добавляется только для одного репозитория, можно использовать **Deploy keys** в настройках конкретного репозитория:

```text
Repository -> Settings -> Deploy keys -> Add deploy key
```

Для приватного репозитория без записи достаточно deploy key без флага `Allow write access`.

### 5.5. Проверить SSH-доступ к GitHub

На сервере выполнить:

```bash
ssh -T git@github.com
```

При первом подключении подтвердить fingerprint:

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Ввести:

```text
yes
```

Успешный ответ обычно выглядит так:

```text
Hi <github-user>! You've successfully authenticated, but GitHub does not provide shell access.
```

### 5.6. Склонировать репозиторий

Перейти в домашнюю директорию пользователя:

```bash
cd /home/ubuntu
```

Склонировать репозиторий по SSH:

```bash
git clone git@github.com:<OWNER>/<REPOSITORY>.git
```

Например:

```bash
git clone git@github.com:sourcecraft/rag-vector-db.git
```

Перейти в папку проекта:

```bash
cd /home/ubuntu/<REPOSITORY>
```

Если проект должен лежать по пути из этой инструкции, используйте:

```bash
cd /home/ubuntu
git clone git@github.com:<OWNER>/<REPOSITORY>.git rag-vector-db
cd /home/ubuntu/rag-vector-db
```

---

## 6. Разворачивание Python-окружения и установка requirements.txt

Команды выполняются **на удалённом сервере** в директории проекта.

### 6.1. Установить Python venv

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip
```

### 6.2. Создать и активировать виртуальное окружение

```bash
cd /home/ubuntu/<REPOSITORY>
python3 -m venv .venv
source .venv/bin/activate
```

Если проект находится в `/home/ubuntu/rag-vector-db`:

```bash
cd /home/ubuntu/rag-vector-db
python3 -m venv .venv
source .venv/bin/activate
```

Обновить инструменты установки:

```bash
python -m pip install --upgrade pip setuptools wheel
```

### 6.3. Создать requirements.txt

Если в репозитории уже есть `requirements.txt`, этот шаг можно пропустить.

Если файла нет, создать его:

```bash
cat > requirements.txt <<'EOF'
pymupdf4llm==1.27.2.3
pdfplumber==0.11.9
beautifulsoup4==4.14.3
lxml==6.1.1
trafilatura==2.0.0
charset-normalizer==3.4.7
ftfy==6.3.1
chonkie==1.6.7
tiktoken==0.13.0
razdel==0.5.0
numpy==2.3.4
datasets==4.4.1
sentence-transformers==5.1.2
EOF
```

В списке выше `chonkie==1.6.7` указан один раз, потому что одна и та же зависимость не должна дублироваться в `requirements.txt`.

### 6.4. Установить зависимости из requirements.txt

```bash
pip install -r requirements.txt
```

Эквивалентные команды установки вручную:

```bash
pip install pymupdf4llm==1.27.2.3 pdfplumber==0.11.9 beautifulsoup4==4.14.3 lxml==6.1.1 trafilatura==2.0.0 charset-normalizer==3.4.7 ftfy==6.3.1 chonkie==1.6.7
pip install tiktoken==0.13.0 razdel==0.5.0 chonkie==1.6.7
```

### 6.5. Проверить установку

```bash
python -m pip check
python - <<'PY'
import fitz
import pdfplumber
import bs4
import lxml
import trafilatura
import charset_normalizer
import ftfy
import chonkie
import tiktoken
import razdel

print("Dependencies import OK")
PY
```