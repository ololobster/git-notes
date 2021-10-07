Заметки по Git
==============

Оригинал — [github.com/ololobster/git-notes](https://github.com/ololobster/git-notes).

1. [Работа с репозиториями](#работа-с-репозиториями):
   [настройки](#настройки),
   [удалённые репозитории](#удалённые-репозитории),
   [дочерние репозитории](#дочерние-репозитории)
1. [Работа с ветками](#работа-с-ветками):
   [merge vs rebase](#merge-vs-rebase)
1. [Работа с коммитами](#работа-с-коммитами):
   [reflog](#reflog)
1. [Работа с тегами](#работа-с-тегами)
1. [Внутреннее устройство](#внутреннее-устройство):
   [объекты](#объекты),
   [прочее](#прочее)
1. [Подходы к выпуску релизов](#подходы-к-выпуску-релизов):
   [GitHub flow](#github-flow),
   [GitLab flow](#gitlab-flow),
   [Git flow](#git-flow).

# Работа с репозиториями

Создать локальную копию другого репозитория:
```
$ git clone ⟨repo⟩ ⟨target directory⟩
```
Примечания:
1. `⟨target directory⟩` можно не указывать, тогда будет соответствовать названию исходного репа.
1. Если клонируем другой локальный реп (т.е. `⟨repo⟩` — это каталог), то используем `--no-hardlinks`.
1. При клонировании из исходного репа закачивается только главная ветка.
   Можно выбрать другую при помощи `-b ⟨branch⟩`.

Создать локальный репозиторий с нуля:
```
$ git init ⟨directory⟩
```
Примечания:
1. Каталог под реп будет создан, если его не существует.
1. Можно не указывать каталог, тогда будет использован текущий.

### Настройки

Вывести/изменить значение параметра для текущего репа (хранится в `.git/config`):
```
$ git config ⟨param⟩
$ git config ⟨param⟩ ⟨value⟩
```
Вывести/изменить глобальное значение параметра (хранится в `~/.gitconfig`):
```
$ git config --global ⟨param⟩
$ git config --global ⟨param⟩ ⟨value⟩
```
Вывести все настройки:
```
$ git config --list --show-origin
```

Некоторые параметры:
1. `user.email` и `user.name` — информация по автору.
1. `core.autocrlf` (`true`, `false` или `input`) — надо ли править переводы строк в зависимости от ОС.
   Просто используем `false` в сочетании с UTF-8 и `\n`.
1. `init.defaultBranch` — название ветки по умолчанию.
   Обычно `master` или `main`.

### Удалённые репозитории

*Удалённый репозиторий* (remote, upstream) — это репозиторий на удалённом сервере (например, на GitHub или на GitLab).
Вывести список удалённых репов:
```
$ git remote -v
```

Добавить удалённый реп:
```
$ git remote add ⟨repo name⟩ ⟨repo URL⟩
```

Изменить URL удалённого репа:
```
$ git remote set-url ⟨repo name⟩ ⟨repo URL⟩
```

Удалить реп:
```
$ git remote remove ⟨repo name⟩
```

### Дочерние репозитории

Клонировать репозиторий и сразу подтянуть его дочерние репозитории:
```
$ git clone --recurse-submodules ⟨repo⟩
```

Подтянуть дочерние репозитории:
```
$ git submodule update --init --recursive
```

Добавить дочерний репозиторий:
1. Перейти в нужную папку.
1. ```
   $ git submodule add ⟨repo⟩
   ```

# Работа с ветками

`git pull`  =  `git fetch`  +  `git merge`.

Создать новую ветку и переключиться на неё:
```
$ git branch ⟨branch⟩
$ git checkout ⟨branch⟩
```

Создать новую ветку от конкретного коммита:
```
$ git branch ⟨branch⟩ ⟨commit⟩
```

Вывести список веток:
```
$ git branch
```

Удалить локальную ветку, которая уже залита в удалённый реп:
```
$ git branch -d ⟨branch⟩
```
Удалить локальную ветку, которая ещё не залита в удалённый реп:
```
$ git branch -D ⟨branch⟩
```

Pushнуть в удалённый реп (если такой ветки нет в удалённом репе, то она создастся):
```
$ git push ⟨repo name⟩ ⟨remote branch⟩
```

Если push'нули неправильный коммит в удалённый реп:
1. Редактируем что надо:
   ```
   $ git commit --amend -m '⟨comment⟩'
   ```
1. Заменяем в удаленном репе:
   ```
   $ git push --force ⟨repo name⟩ ⟨remote branch⟩
   ```

Скачать ветку из удалённого репа (в локальном репе появится tracking-ветка с названием `⟨repo name⟩/⟨remote branch⟩`, связанная с веткой удалённого репа):
```
$ git fetch ⟨repo name⟩ ⟨remote branch⟩
```

### Merge vs rebase

![](img/merge_vs_rebase.svg "merge vs rebase.")  
*Походы к слиянию веток: merge-коммит vs rebase + fast-forward merge.
У merge-коммита 2 родителя вместо одного.
При fast-forward merge кончик ветки просто переставляется на другой коммит, новых коммитов не создаётся.*

Залить указанную ветку в текущую ветку:
```
$ git merge ⟨branch⟩
```
Примечания:
1. `--ff` повелевает использовать fast-forward merge если это возможно.
   Это поведение по умолчанию.
1. `--no-ff` повелевает создать merge-коммит, даже если можно использовать fast-forward merge.
1. `--ff-only` — либо fast-forward merge либо никак.

Выполнить rebase — сделать так, чтобы текущая ветка начиналась от кончика указанной ветки:
```
$ git rebase ⟨branch⟩
```
Примечания:
1. Создание merge-коммита легко отменить, т.к. при этом создаётся 1 запись в reflog.
   А вот `git rebase` может создать десятки записей.

Переключиться на ветку `feature`, найти изменения относительно `main` и залить их в `release`:
```
$ git rebase --onto release main feature
```
Указатель `release` при этом остаётся на месте.

# Работа с коммитами

*Staging area* (ака index) — это новые файлы и модификации отслеживаемых файлов, которые помечены (при помощи `git add`) для внесения в следующий коммит.

![](img/file_states.png "Состояния файла в Git-репозитории.")  
*Состояния файла в репозитории.*

`git commit` создаёт коммит из того, что находится в staging area.
Примечания:
1. `--amend` повелевает перезаписать предыдущий коммит (его изменения сохраняются).
1. `-a` ака `--all` повелевает перенести все модификации отслеживаемых файлов в staging area, чтобы они тоже попали в коммит.

Вывести последние N коммитов:
```
$ git log -n ⟨N⟩
```

Вывести содержимое коммита:
```
$ git diff ⟨commit⟩^!
```

Избавиться от всех изменений:
```
$ git checkout .
```

Избавиться от изменений в 1 файле:
```
$ git checkout -- ⟨file⟩
```

Восстановить файл, который был удален, но это еще не было закоммичено:
```
$ git checkout HEAD ⟨file⟩
```

Search commits by str:
```
$ git log -S'select_column' -- hac_dashboard/utils/dynamic_table.py
```

Git предусматривает для файлов лишь 3 варианта прав: `100644`, `100755` (выполняемый файл), `120000` (символьная ссылка).

### Reflog

Reflog (reference log) — это записи о перемещениях кончиков веток и `HEAD`.
Например, `HEAD@{2}` означает «куда `HEAD` указывал 2 перемещения назад».
Простое переключение с ветки на ветку при помощи `git checkout` также добавляет новые записи для `HEAD`.

Вывести последние N записей reflog для `HEAD`:
```
$ git reflog show -n ⟨N⟩
```
Вывести записи reflog для заданной ветки:
```
$ git reflog show ⟨branch⟩
```

Git ничего не забывает ~~и не прощает~~, например, `git commit --amend` не перезаписывает последний коммит, а создаёт новый ему на замену (старый остаётся в недрах Git).
Это позволяет много чего отменить.

Отменить последний коммит, если нужно сохранить изменения:
```
$ git reset --soft HEAD@{1}
```

Отменить последний коммит, если изменения не нужны:
```
$ git reset --hard HEAD@{1}
```

Отменить ошибочное применение `git commit --amend`:
```
$ git reset --soft HEAD@{1}
$ git commit -C HEAD@{1}
```

# Работа с тегами

Вывести все теги:
```
$ git tag
```

Искать теги по шаблону:
```
$ git tag -l "poppler-20.*"
```

Вывести последний тег:
```
$ git describe --abbrev=0 --tags
```

Создать простой (lightweight) тег на последнем коммите:
```
$ git tag ⟨tag⟩
```

Создать простой тег на заданном коммите:
```
$ git tag ⟨tag⟩ ⟨commit⟩
```

По умолчанию `git push` не заливает теги в удалённый репозиторий.
Для этого надо вызвать:
```
$ git push ⟨repo name⟩ ⟨tag⟩
```

# Внутреннее устройство

### Объекты

Единицей хранения данных является объект, который идентифицируется 40-символьным хешем SHA1.
Т.е. Git является key-value хранилищем.
Объекты хранятся в `.git/objects`.

Типы объектов:
1. BLOB — это содержимое файла (ни имени файла, ни прав доступа тут нет).
   Если добавить в репозиторий 2 копии одного файла, то BLOB-объект будет один, т.к. хеш SHA1 одинаковый.
1. Tree — это каталог, т.е. список дочерних файлов и каталогов, их прав и соответствующих им BLOB-объектов.
1. Коммит, имеющий следующие свойства:
   - `tree` — корневой каталог репозитория (коммит не хранит дельту);
   - `parent` — предшествующий коммит (2 шт. если это merge-коммит);
   - `author`, `committer` — информация по автору;
   - `message`.

   История изменений в Git — это граф из объектов-коммитов.

Узнать тип объекта:
```
$ git cat-file -t ⟨id⟩
```

Пример создания коммита без высокоуровневых команд `git add` и `git commit`:
1. Создаём BLOB-объект:
   ```
   $ echo 'my blob' | git hash-object -w --stdin
   0b729d77bcd6e98e02910a24d781736df537ef13
   $ git cat-file -t 0b729d77
   blob
   $ git cat-file -p 0b729d77
   my blob
   ```
   В `.git/objects` появился новый файл:
   ```
   $ ls .git/objects/0b
   729d77bcd6e98e02910a24d781736df537ef13
   ```
1. Добавляем в корневой каталог новый файл `my_blob.txt` с правами `100644`:
   ```
   $ git update-index --add --cacheinfo 100644 0b729d77bcd6e98e02910a24d781736df537ef13 my_blob.txt
   $ git write-tree
   6fd058384276fe29baced4899f2a06f671989d6e
   $ git cat-file -t 6fd05838
   tree
   $ git cat-file -p 6fd05838
   ...
   100644 blob 0b729d77bcd6e98e02910a24d781736df537ef13 my_blob.txt
   ```
1. Создаём коммит на основе нового дерева `6fd05838` (предшествующий коммит — `3b64db71`):
   ```
   $ git commit-tree 6fd05838 -p 3b64db71 -m 'test'
   73008467263c517658273f774b992ac135d019a6
   $ git cat-file -t 73008467
   commit
   ```
1. Переставляем кончик ветки `main` на новый коммит:
   ```
   $ git update-ref refs/heads/main 73008467
   ```
1. `git log` показывает всё корректно, но файла `my_blob.txt` нету...
   ```
   $ git checkout HEAD -- my_blob.txt
   ```

### Прочее

Ветки — это файлы в каталоге `.git/refs/heads`.
Каждый такой файл содержит ид коммита, который является кончиком ветки.
Можно посмотреть этот ид:
```
$ git rev-parse main
```
Ветке удалённого репозитория соответствует файл `.git/refs/remotes/⟨repo name⟩/⟨remote branch⟩`.
Смотреть аналогично:
```
$ git rev-parse origin/main
```

Текстовый файл `.git/HEAD` указывает на текущий коммит.
Если мы на ветке `main`, то `.git/HEAD` выглядит так:
```
ref: refs/heads/main
```
Если мы переключились на конкретный коммит `c61f0c0a`, то `.git/HEAD` выглядит так:
```
c61f0c0afdf2ae48db1f0b15cf1cc0023d88c203
```

Reflog хранится в `.git/logs`:
- история `HEAD` — в файле `.git/logs/HEAD`,
- история ветки — в файле `.git/logs/refs/heads/⟨branch⟩`.

Тегу соответствует файл в каталоге `.git/refs/tags`.

# Подходы к выпуску релизов

### GitHub flow

Работа над фичами идёт в отдельных ветках.
Ветка `main` всегда должна оставаться стабильной.
Если что-то залито в `main`, то можно и должно деплоить новую версию (по заветам continuous delivery).

Сложность: очень просто.
Подойдёт для SaaS-приложения, которое деплоится каждый день или чаще.

См. также:
1. [Understanding the GitHub flow](https://guides.github.com/introduction/flow/).
1. [GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html) — Scott Chacon.

### GitLab flow

1-я схема: усовершенствованный GitHub flow с ветками `main` (или `development`) и `production`.
Новым релизом считается заливание `main` в `production`.

Сложность: просто.
Удобнее GitHub flow, когда мы не можем мгновенно задеплоить новую версию в любой момент (например, есть deployment windows, есть модерация мобильных приложений в Apple App Store и т.д.), а разработка в `main` не должна останавливаться.

2-я схема: каждая ветка соответствует какому-либо окружению.
При обновлении ветки происходит деплой в соответствующее окружение.
Пример: тестовые окружения `staging` и `pre-prod` + боевое окружение `production`.

![](img/gitlab_environment_branches.png "GitLab flow с ветками, соответствующими окружениям.")  
*GitLab flow с ветками, соответствующими окружениям.*

3-я схема: `main` предназначена для разработки нового релиза.
Когда релиз пора выпускать, из `main` отпочковывается специальная ветка, где его будут тестировать, шлифовать  и поддерживать.
Выпуском новой версии считается изменение в релизной ветке (опционально — добавление тега в релизную ветку).

Сложность: сложно.
Предназначен для ситуаций когда нам надо поддерживать старые версии продолжительное время.

![](img/gitlab_version_branches.png "GitLab flow с ветками, соответствующими релизам.")  
*GitLab flow с ветками, соответствующими релизам.*

См. также:
1. [Introduction to GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html).

### Git flow

Используемые ветки:
- работа над фичами идёт в отдельных ветках, они вливаются в `develop`;
- `develop` предназначена для разработки нового релиза;
- когда релиз пора выпускать, из `develop` отпочковывается специальная ветка, где его будут тестировать и шлифовать (в `develop` продолжается работа над следующим релизом);
- `main` всегда должна оставаться стабильной, заливка релизной ветки в `main` означает выпуск новой версии, соответствующий коммит помечается тегом;
- ветки для оперативного исправления серьёзных багов, которые вливаются и в `main` и в `develop`.

![](img/git_flow.png "Git flow.")  
*Git flow.*

Сложность: очень сложно.

См. также:
1. [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/) — Vincent Driessen.
