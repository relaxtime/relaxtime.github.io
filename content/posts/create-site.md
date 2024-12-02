+++
date = '2024-12-02T12:39:26+03:00'
draft = false
title = 'Create Site'
title = 'Создаем сайт на Github Pages с помощью Hugo'
summary = 'Создаем сайт на Github Pages с помощью Hugo'
+++
Создаем репозиторий на GitHub
![](/posts/attachments/Pasted%20image%2020241202112823.png)
Скачиваем Hugo по ссылке https://github.com/gohugoio/hugo/releases/tag/v0.139.3
Cоздаем новый сайт
```
.\hugo.exe new site relaxtime.github.io
cd relaxtime.github.io
```
Инициализируем репозиторий
```
git init 
git commit --allow-empty -m "Initializing master branch"
```
Добавляем удаленный репозиторий
```
git remote add origin https://github.com/relaxtime/relaxtime.github.io.git
```
Генерируем ssh-ключи, публичный ключ добавляем в репозиторий
```
ssh-keygen -t ed25519 -C "relaxtimezzz@gmail.com"
type relaxtime.pub
```
![](/posts/attachments/Pasted%20image%2020241202120711.png)

Сгенерированные ключи relaxtime и relaxtime.pub перемещаем в другую директорию

Запускаем ssh-агент от имени администратора
```
Set-Service -Name ssh-agent -StartupType Manual
Start-Service ssh-agent
```
Посмотреть все ключи в ssh-агенте
```
ssh-add -l
```
Удалить все ключи из ssh-агента
```
ssh-add -D
```
Записываем ssh-ключ в агент
```
ssh-add D:\hugo_0.139.3_windows-amd64\relaxtime
```
![](/posts/attachments/Pasted%20image%2020241202121336.png)

Отправляем изменения из ветки master в GitHub
```
git push -u origin master
```

![](/posts/attachments/Pasted%20image%2020241202121742.png)

Создаем ветку devel, в которой будем хранить кодовую базу, исходники из которых генерируется сайт

```
git checkout -b devel
```

Добавляем все файлы проекта в ветку devel
```
git add --all .
git commit -m 'Initial commit'
```
Отправляем изменения из ветки devel в GitHub
```
git push -u origin devel
```
![](/posts/attachments/Pasted%20image%2020241202121903.png)

Устанавливаем тему
```
git submodule add https://github.com/nodejh/hugo-theme-mini.git themes/mini
```

hugo.toml:
```
baseURL = 'https://relaxtime.github.io/'
languageCode = 'ru-ru'
title = 'Test Site'
theme = 'mini'
publishdir = './public/'
```


Исходный код и сам сайт хранятся в разных ветках (devel и master соответственно). Переключаться между ними и переносить изменения - неудобно. Поэтому подключим ветку `master` в рабочую директорию в каталог `public` командами:
```
git worktree add -B master public origin/master
echo "public" >> .gitignore
git add .gitignore
git commit -m 'Add .gitignore'
```
![](/posts/attachments/Pasted%20image%2020241202122100.png)

Чтобы не было ошибок с social, нужно в файле `C:\Users\1\Desktop\hugo_0.139.3_windows-amd64\relaxtime.github.io\themes\mini\layouts\partials\footer.html` изменить `.Site.Social` на `.Site.Params.Social`

Генерируем сайт
```
..\hugo.exe
```
Загружаем на GitHub
```
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
```
..\hugo.exe new posts/create-site.md
```
Содержимое create-site.md:
```
+++
date = '2024-12-02T07:13:31+03:00'
draft = false
title = 'Создаем сайт на Github Pages с помощью Hugo'
summary = 'Создаем сайт на Github Pages с помощью Hugo'
+++

Текст
```

Для медиа создаем директорию  `D:\hugo_0.139.3_windows-amd64\relaxtime.github.io\content\posts\attachments`
В файле `create-site.md` меняем `attachments/` на `/posts/attachments/`

Сохраняем изменения
```
git add .
git commit -m 'Add index page, first post'
git push
..\hugo.exe
pushd public
git add .
git commit -m 'Publish first post'
git push
popd
```