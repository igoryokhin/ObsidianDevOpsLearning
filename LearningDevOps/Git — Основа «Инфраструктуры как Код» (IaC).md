# Установка и «Знакомство» (Configuration)

## **1. Установка**

Для начала установим `git` на наш сервер:
```terminal
sudo apt update
sudo apt install git -y
```


## **2. Настройка личности**

Для того чтобы авторизоваться и пушить комиты своих сборок в GitHub, нам нужно выполнить данные команды:
```terminal
git config --global user.name "igoryokhin"
git config --global user.email "igoryokhin@gmail.com"
```

Комиты это нынешняя версия написанного файла который ты отправляешь в GitHub, без авторизации `git` не даст тебе ничего сохранить

## **3. Проверка**

Теперь проверяем что вышеописанные команды сохранили наши введенные данные:
```terminal
git config --list
```

вот что мы получили:
```terminal
user.name=igoryokhin
user.email=igoryokhin@gmail.com
maki@lablearning:~/GitRepository$ 
```


## Создание первого локального репозитория

Репозиторий - это простая папка которую `git` использует как базу данных (папка `.git`)

Создаем папку в домашней директории:
```terminal
sudo mkdir /home/maki/GitRepositoryDevOpsLab
```

Переходим по этомй пути:
```terminal
cd /home/maki/GitRepositoryDevOpsLab
```

После чего инициализируем `git`:
```terminal
git init
```

Создастся папка `.git` , проверяем статус:
```terminal
git status
```

Получаем такой ответ:
```terminal
maki@lablearning:~/GitRepository/DevOpsLab$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        default

nothing added to commit but untracked files present (use "git add" to track)
```

Я уже скопировал конфигурацию nginx в эту папку для пробы, теперь мы можем добавить его для сохранения в репозиторий:
```terminal
git add default
```

После чего мы комитим наш добавленный файл на отправку в репозиторий:
```terminal
git commit -m "Initial commit: nginx default config"
```

Вот что мы получаем в ответ:
```terminal
[master (root-commit) 24d9bcf] Initial commit: nginx default config
 1 file changed, 91 insertions(+)
 create mode 100644 default
maki@lablearning:~/GitRepository/DevOpsLab$ 
```

Таким образом у нас появился локальный репозиторий на своем сервере!

## Создание связи локального и удаленного репозитория

Нам потребуется зайти на сайт GitHub, авторизоваться и создать пустой репозиторий с аналогичным названием.

Псоле создания репозитория на GitHub нам сам же GitHub предлагает два варианта:

### …or create a new repository on the command line
```terminal
echo "# DevOpsLab" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/igoryokhin/DevOpsLab.git
git push -u origin main
```

### …or push an existing repository from the command line

```terminal
git remote add origin https://github.com/igoryokhin/DevOpsLab.git
git branch -M main
git push -u origin main
```


Раз мы создали уже и опробовали локальный репозиторий, мы пойдем по второму варианту

![[Схема работы с Git.canvas]]
Локальные изменения -> `git add` (индекс) -> `git commit` (локальный сейв) -> `git push` (облачный сейв)

Для того чтобы каждый раз не вводить токен для пуша в облако, нам понадобаться закешировать наш токен. Это делает данной командой:
```terminal
git config --global credential.helper store
```

После чего потребуется еще раз пропушить локальный репозиторий

«Никогда не передавай свои приватные ключи или токены другим людям. Доступ выдается через настройки прав доступа (RBAC/Collaborators), а не через общий пароль»

## Вопросы на засыпку

### Проверочный вопрос (уровень выше)

Если ты:

1. Изменил файл
    
2. Сделал `git add`
    
3. Потом снова изменил файл
    
4. Сделал `git commit`
    

👉 Что попадёт в коммит — первая версия или вторая?

#### Ответ

## Что попадёт в коммит?

Если:

1. Изменил файл
    
2. `git add file`
    
3. Снова изменил файл
    
4. `git commit`
    

👉 В коммит попадёт **первая версия**.

Потому что `git add` кладёт снимок в staging.  
Всё, что изменилось после add — не попадёт, пока ты снова не сделаешь `git add`.

