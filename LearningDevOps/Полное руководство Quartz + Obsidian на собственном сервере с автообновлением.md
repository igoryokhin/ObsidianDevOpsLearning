## 1. Подготовка сервера

Предполагается, что у вас есть сервер с Ubuntu (22.04/24.04) и доступ по SSH. Выполните начальную настройку:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl docker.io -y
```

Убедитесь, что Docker запущен:
```bash
sudo systemctl enable --now docker
```

### 1.1. Установка Node.js (рекомендуемая версия 22)
```bash
curl -fsSL https://fnm.vercel.app/install | bash
source ~/.bashrc
fnm install 22
fnm default 22
node --version  # должно быть v22.x
```

### 1.2. Настройка домена
Направьте домен (например, `web.obsidian.my.to`) на IP вашего сервера. Убедитесь, что порты 80 и 443 открыты.

---

## 2. Установка Quartz

Склонируйте репозиторий Quartz (форк или оригинал) и установите зависимости:

```bash
cd ~
git clone https://github.com/jackyzha0/quartz.git quartz-src
cd quartz-src
npm install
```

---

## 3. Репозиторий с заметками

Создайте отдельный репозиторий для Obsidian (например, `ObsidianDevOpsLearning`). В нём должна быть папка `content` (или вы можете настроить путь в `quartz.config.ts`). Удобно хранить заметки в корне репозитория.

Пример структуры:
```
ObsidianDevOpsLearning/
├── index.md                # главная страница (шаблон)
├── LearningDevOps/         # папка с заметками
│   ├── Topic Linux Fundamentals.md
│   ├── Git — Основа «Инфраструктуры как Код» (IaC).md
│   └── ...
└── .gitignore
```

Клонируйте этот репозиторий на сервер:
```bash
cd ~
git clone https://github.com/ваш-username/ObsidianDevOpsLearning.git
```

---

## 4. Настройка Quartz

### 4.1. Файл `quartz.config.ts`
Отредактируйте его под свои нужды. Минимально нужно указать `baseUrl` и отключить ненужные плагины. Пример:

```typescript
import { QuartzConfig } from "./quartz/cfg"
import * as Plugin from "./quartz/plugins"

const config: QuartzConfig = {
  configuration: {
    pageTitle: "Obsidian DevOps Learning",
    enableSPA: true,
    enablePopovers: true,
    analytics: { provider: "plausible" },
    locale: "ru-RU",
    baseUrl: "/quartz",   // важно!
    ignorePatterns: ["private", "templates", "_templates"],
    defaultDateType: "created",
    theme: { ... } // ваши настройки темы
  },
  plugins: {
    transformers: [ ... ],
    filters: [Plugin.RemoveDrafts()],
    emitters: [
      Plugin.AliasRedirects(),
      Plugin.ComponentResources(),
      Plugin.ContentPage(),
      // Plugin.FolderPage(),   // отключаем, чтобы избежать лишних страниц папок
      Plugin.TagPage(),
      Plugin.ContentIndex({ enableSiteMap: true }),
      Plugin.Assets(),
      Plugin.Static(),
      Plugin.NotFoundPage(),
    ],
  },
}
export default config
```

### 4.2. Файл `quartz.layout.ts`
Настройте отображение компонентов. Убедитесь, что `Graph` имеет параметр `depth: -1` (показывать все узлы). Пример:

```typescript
export const defaultContentPageLayout: PageLayout = {
  beforeBody: [ ... ],
  left: [ ... ],
  right: [
    Component.Graph({
      depth: -1,      // показывать все страницы, даже без связей
    }),
    Component.DesktopOnly(Component.TableOfContents()),
    Component.Backlinks(),
  ],
}
```

---

## 5. Первая сборка

Перейдите в директорию Quartz и выполните сборку, указав путь к вашему репозиторию с заметками и выходную папку (например, `~/obsidian-site/site/quartz`):

```bash
cd ~/quartz-src
npx quartz build --directory ~/ObsidianDevOpsLearning --output ~/obsidian-site/site/quartz
```

После сборки проверьте, что в `~/obsidian-site/site/quartz` появились файлы: `index.html`, папка `LearningDevOps` с HTML-файлами, `static`, `index.css` и т.д.

---

## 6. Настройка веб-сервера Caddy

### 6.1. Установка Caddy (через Docker)

Создайте директорию для конфигурации:
```bash
mkdir -p ~/ObsidianLiveSync/caddy_config
```

### 6.2. Файл `Caddyfile`
Создайте файл `~/ObsidianLiveSync/caddy_config/Caddyfile` со следующим содержимым:

```
# CouchDB / LiveSync (если есть)
db.obsidian.my.to {
    reverse_proxy obsidian-livesync:5984
    encode zstd gzip
    log {
        output file /var/log/caddy/db_access.log
    }
}

# Quartz сайт
web.obsidian.my.to {
    root * /var/www/obsidian-web
    encode zstd gzip
    file_server

    # Webhook – должен быть ПЕРВЫМ
    handle /webhook/* {
        reverse_proxy localhost:9000
    }

    # Редирект с корня на /quartz/
    handle / {
        redir /quartz/ 301
    }

    # Редирект для путей без /quartz (например, /LearningDevOps/...)
    handle_path /LearningDevOps* {
        redir /quartz/LearningDevOps{path} 301
    }

    # Редирект для самой папки /quartz/LearningDevOps/ (чтобы не было 404)
    handle_path /quartz/LearningDevOps/* {
        redir /quartz/ 307
    }

    # Основной обработчик для /quartz/
    handle_path /quartz/* {
        root * /var/www/obsidian-web/quartz
        try_files {path}.html {path} {path}/ /index.html
        file_server
    }

    log {
        output file /var/log/caddy/web_access.log
    }
}
```

### 6.3. Запуск Caddy в Docker
```bash
docker run -d \
  --name caddy \
  -p 80:80 -p 443:443 \
  -v ~/ObsidianLiveSync/caddy_config:/etc/caddy \
  -v ~/obsidian-site:/var/www/obsidian-web \
  -v caddy_data:/data \
  -v caddy_config:/config \
  caddy:latest
```

Проверьте, что контейнер запущен:
```bash
docker ps | grep caddy
```

### 6.4. Проверка работы
Откройте в браузере `https://web.obsidian.my.to/` – должен произойти редирект на `/quartz/` и загрузиться главная страница. Перейдите по любой заметке, например `https://web.obsidian.my.to/quartz/LearningDevOps/Topic-Linux-Fundamentals` – страница должна открыться.

---

## 7. Автоматизация обновления главной страницы

Чтобы на главной странице (`index.md`) автоматически появлялись ссылки на все заметки, создадим скрипт-генератор.

### 7.1. Шаблон `index.template.md`
Переименуйте существующий `index.md` в `index.template.md` и уберите из него список заметок, оставив место для вставки:

```bash
cd ~/ObsidianDevOpsLearning
mv index.md index.template.md
nano index.template.md
```

Пример содержимого:
```markdown
# Добро пожаловать

Это главная страница.

## Все заметки

{{LINKS}}
```

### 7.2. Скрипт генерации `generate-index.sh`
Создайте файл `~/generate-index.sh`:

```bash
#!/bin/bash

TEMPLATE="/home/makidocker/ObsidianDevOpsLearning/index.template.md"
OUTPUT="/home/makidocker/ObsidianDevOpsLearning/index.md"
NOTES_DIR="/home/makidocker/ObsidianDevOpsLearning/LearningDevOps"

# Собираем список файлов
LINKS=""
while IFS= read -r -d '' file; do
    name=$(basename "$file" .md)
    LINDS="${LINKS}- [[${name}]]\n"
done < <(find "$NOTES_DIR" -maxdepth 1 -name "*.md" ! -name "index.md" -print0 | sort -z)

# Убираем лишние переводы строк
LINKS=$(echo -e "$LINKS" | sed '/^$/d')

# Создаём временный файл
TMP=$(mktemp)

# Читаем шаблон построчно и заменяем метку
while IFS= read -r line; do
    if [[ "$line" == *"{{LINKS}}"* ]]; then
        echo -e "$LINKS" >> "$TMP"
    else
        echo "$line" >> "$TMP"
    fi
done < "$TEMPLATE"

mv "$TMP" "$OUTPUT"
echo "index.md сгенерирован"
```

Сделайте скрипт исполняемым:
```bash
chmod +x ~/generate-index.sh
```

### 7.3. Добавляем `index.md` в `.gitignore`
Чтобы случайно не закоммитить сгенерированный файл:
```bash
echo "index.md" >> ~/ObsidianDevOpsLearning/.gitignore
```

---

## 8. Скрипт полного обновления сайта

Создайте файл `~/update-quartz.sh`:

```bash
#!/bin/bash
export PATH="/home/makidocker/.local/share/fnm/node-versions/v22.22.1/bin:$PATH"

REPO_DIR="/home/makidocker/ObsidianDevOpsLearning"
QUARTZ_DIR="/home/makidocker/quartz-src"
OUTPUT_DIR="/home/makidocker/obsidian-site/site/quartz"
LOG_FILE="/home/makidocker/quartz-update.log"

echo "$(date): Начало обновления" >> "$LOG_FILE"

cd "$REPO_DIR" || exit
git pull origin main >> "$LOG_FILE" 2>&1

if [ $? -eq 0 ]; then
    echo "$(date): Git pull выполнен, генерируем index.md" >> "$LOG_FILE"
    /home/makidocker/generate-index.sh >> "$LOG_FILE" 2>&1

    echo "$(date): Запускаем сборку Quartz" >> "$LOG_FILE"
    cd "$QUARTZ_DIR" || exit
    npx quartz build --directory "$REPO_DIR" --output "$OUTPUT_DIR" >> "$LOG_FILE" 2>&1

    sleep 2

    # Добавляем тег <base href="/quartz/"> во все HTML-файлы
    cd "$OUTPUT_DIR"
    find . -name "*.html" -exec sed -i 's|<head>|<head><base href="/quartz/">|' {} \; 2>>"$LOG_FILE"

    echo "$(date): Сборка и добавление base-тега завершены" >> "$LOG_FILE"
else
    echo "$(date): Ошибка при git pull" >> "$LOG_FILE"
fi
```

Сделайте его исполняемым:
```bash
chmod +x ~/update-quartz.sh
```

Теперь вы можете вручную запускать обновление:
```bash
~/update-quartz.sh
```

---

## 9. Настройка автоматического обновления через Webhook (GitHub)

### 9.1. Установка webhook
```bash
sudo apt install webhook -y
```

### 9.2. Конфигурация webhook
Создайте файл `/etc/webhook.conf`:

```json
[
  {
    "id": "update-quartz",
    "execute-command": "/home/makidocker/update-quartz.sh",
    "command-working-directory": "/home/makidocker",
    "response-message": "Updating site...",
    "trigger-rule": {
      "match": {
        "type": "payload-hash-sha1",
        "secret": "сгенерируйте-случайную-строку",
        "parameter": {
          "source": "header",
          "name": "X-Hub-Signature"
        }
      }
    }
  }
]
```

Сгенерируйте секрет:
```bash
openssl rand -hex 32
```
Скопируйте результат и вставьте вместо `сгенерируйте-случайную-строку`.

### 9.3. Запуск webhook как сервис systemd
Создайте файл `/etc/systemd/system/webhook.service`:

```ini
[Unit]
Description=Webhook server
After=network.target

[Service]
ExecStart=/usr/bin/webhook -hooks /etc/webhook.conf -verbose
WorkingDirectory=/home/makidocker
Restart=always
User=makidocker
Group=makidocker

[Install]
WantedBy=multi-user.target
```

Запустите и добавьте в автозагрузку:
```bash
sudo systemctl daemon-reload
sudo systemctl enable webhook.service
sudo systemctl start webhook.service
sudo systemctl status webhook.service
```

### 9.4. Настройка вебхука на GitHub
- Зайдите в репозиторий → Settings → Webhooks → Add webhook.
- **Payload URL**: `https://web.obsidian.my.to/webhook/update-quartz`
- **Content type**: `application/json`
- **Secret**: вставьте сгенерированную строку.
- **Which events?** – выберите "Just the push event".
- Нажмите **Add webhook**.

### 9.5. Проверка
Сделайте тестовый пуш в репозиторий. Наблюдайте за логами:
```bash
sudo journalctl -u webhook.service -f
```
Если всё настроено верно, при каждом пуше скрипт `update-quartz.sh` будет запускаться и сайт обновится.

---

## 10. Итог

Теперь у вас есть полностью автоматизированная система:
- Obsidian заметки хранятся в Git-репозитории.
- При пуше в GitHub webhook запускает обновление на сервере.
- Скрипт подтягивает изменения, генерирует `index.md` со ссылками на все заметки, собирает статический сайт с помощью Quartz.
- Caddy обслуживает сайт по HTTPS, корректно обрабатывает пути и редиректы.
- Граф на главной странице отображает все заметки (благодаря `depth: -1` и сгенерированным ссылкам).

Поздравляю! Ваш цифровой сад работает и обновляется автоматически.