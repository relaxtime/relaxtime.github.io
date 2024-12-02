+++
date = '2024-12-02T12:39:26+03:00'
draft = false
title = 'Создаем сайт на Github Pages с помощью Hugo'
summary = 'Создаем сайт на Github Pages с помощью Hugo'
+++

Создаем репозиторий на GitHub
![](/posts/attachments/Pasted%20image%2020241202112823.png)

Скачиваем Hugo по ссылке https://github.com/gohugoio/hugo/releases/tag/v0.139.3

Cоздаем новый сайт
```bash
.\hugo.exe new site relaxtime.github.io
cd relaxtime.github.io
```

Инициализируем репозиторий
```bash
git init 
git commit --allow-empty -m "Initializing master branch"
```

Добавляем удаленный репозиторий
```bash
git remote add origin https://github.com/relaxtime/relaxtime.github.io.git
```

Генерируем ssh-ключи, публичный ключ добавляем в репозиторий
```bash
ssh-keygen -t ed25519 -C "relaxtimezzz@gmail.com"
type relaxtime.pub
```
![](/posts/attachments/Pasted%20image%2020241202120711.png)

Сгенерированные ключи relaxtime и relaxtime.pub перемещаем в другую директорию

Запускаем ssh-агент от имени администратора
```bash
Set-Service -Name ssh-agent -StartupType Manual
Start-Service ssh-agent
```
Посмотреть все ключи в ssh-агенте
```bash
ssh-add -l
```

Удалить все ключи из ssh-агента
```bash
ssh-add -D
```

Записываем ssh-ключ в агент
```bash
ssh-add D:\hugo_0.139.3_windows-amd64\relaxtime
```
![](/posts/attachments/Pasted%20image%2020241202121336.png)

Отправляем изменения из ветки master в GitHub
```bash
git push -u origin master
```

![](/posts/attachments/Pasted%20image%2020241202121742.png)

Создаем ветку devel, в которой будем хранить кодовую базу, исходники из которых генерируется сайт

```bash
git checkout -b devel
```

Добавляем все файлы проекта в ветку devel
```bash
git add --all .
git commit -m 'Initial commit'
```

Отправляем изменения из ветки devel в GitHub
```bash
git push -u origin devel
```
![](/posts/attachments/Pasted%20image%2020241202121903.png)

Устанавливаем тему
```bash
git submodule add https://github.com/nodejh/hugo-theme-mini.git themes/mini
```

Содержимое файла hugo.toml:
```bash
baseURL = 'https://relaxtime.github.io/'
languageCode = 'ru-ru'
title = 'Test Site'
theme = 'mini'
publishdir = './public/'
```


Исходный код и сам сайт хранятся в разных ветках (devel и master соответственно). Переключаться между ними и переносить изменения - неудобно. Поэтому подключим ветку `master` в рабочую директорию в каталог `public` командами:
```bash
git worktree add -B master public origin/master
echo "public" >> .gitignore
git add .gitignore
git commit -m 'Add .gitignore'
```
![](/posts/attachments/Pasted%20image%2020241202122100.png)

Чтобы не было ошибок с social, нужно в файле `D:\hugo_0.139.3_windows-amd64\relaxtime.github.io\themes\mini\layouts\partials\footer.html` изменить `.Site.Social` на `.Site.Params.Social`

Генерируем сайт
```bash
..\hugo.exe
```
Загружаем на GitHub
```bash
pushd public
git add .
git commit -m 'Add index page'
git push
popd
```
![](/posts/attachments/Pasted%20image%2020241202122651.png)

Проверяем доступность сайта
![](/posts/attachments/Pasted%20image%2020241202122749.png)

Создаем первый пост
```bash
..\hugo.exe new posts/create-site.md
```

Содержимое create-site.md:
```bash
+++
date = '2024-12-02T07:13:31+03:00'
draft = false
title = 'Создаем сайт на Github Pages с помощью Hugo'
summary = 'Создаем сайт на Github Pages с помощью Hugo'
+++

Текст
```

Для медиа создаем директорию  `D:\hugo_0.139.3_windows-amd64\relaxtime.github.io\content\posts\attachments`
Ссылка на медиа указывается в таком виде `![](/posts/attachments/name.png)`

Сохраняем изменения
```bash
git add .
git commit -m 'Add index page, first post'
git push
..\hugo.exe
```
![](/posts/attachments/Pasted%20image%2020241202130357.png)
```bash
pushd public
git add .
git commit -m 'Publish first post'
git push
popd
```
![](/posts/attachments/Pasted%20image%2020241202130423.png)


Проверяем доступность
![](/posts/attachments/Pasted%20image%2020241202130531.png)