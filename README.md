Заметки по Git
==============

1. [Работа с репозиториями](#работа-с-репозиториями):
   [настройки](#настройки),
   [дочерние репозитории](#дочерние-репозитории)
1. [Работа с ветками](#работа-с-ветками):
   [merge vs rebase](#merge-vs-rebase)
1. [Работа с коммитами](#работа-с-коммитами)
1. [Внутреннее устройство](#внутреннее-устройство)
1. [Подходы к выпуску релизов](#подходы-к-выпуску-релизов):
   [git-flow](#git-flow),
   [GitHub flow](#github-flow),
   [GitLab flow](#gitLab-flow)

# Работа с репозиториями

Инициализировать репозиторий:
```
$ git init ⟨directory⟩
```
Папка под репозиторий будет создана, если её не существует.

Клонировать локальный репозиторий:
```
$ git clone --no-hardlinks ⟨repo directory⟩
```

Клонировать заданную ветку репозитория:
```
$ git clone -b ⟨branch⟩ ⟨repo⟩
```

Вывести список удалённых репов:
```
$ git remote -v
```

Добавить реп:
```
$ git remote add ⟨repo name⟩ ⟨repo URL⟩
```

Изменить URL репа:
```
$ git remote set-url ⟨repo name⟩ ⟨repo URL⟩
```

Удалить реп из списка:
```
$ git remote remove ⟨repo name⟩
```

### Настройки

Вывести все настройки:
```
$ git config --list --show-origin
```
Вывести глобальное значение параметра (хранится в `~/.gitconfig`):
```
$ git config --global user.email
```
Вывести значение параметра для текущего репа (хранится в `.git/config`):
```
$ git config user.email
```
Изменить значение параметра:
```
$ git config user.name "John Doe"
```
Добавляем `--global` для глобальных изменений.

Do not fuck with end of line:
```
$ git config --global core.autocrlf false
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

# Ветки

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

Push'нуть в удалённый реп (если такой ветки нет в удалённом репе, то она создастся):
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

Скачать ветку, которой ещё нет в локальном репе:
```
$ git fetch ⟨repo name⟩ ⟨remote branch⟩
```

Checking out a local branch from a remote-tracking branch automatically creates what is called a “tracking branch” (and the branch it tracks is called an “upstream branch”).
Tracking branches are local branches that have a direct relationship to a remote branch.

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

Переключиться на ветку `feature`, найти изменения относительно `master` и залить их в `release`:
```
$ git rebase --onto release master feature
```
Указатель `release` при этом остаётся на месте.

# Работа с коммитами

Отменить последний коммит:
1. Если нужно сохранить изменения последнего коммита:
   ```
   $ git reset --soft HEAD~1
   ```
1. Если изменения последнего коммита не нужны:
   ```
   $ git reset --hard HEAD~1
   ```

Отменить ошибочное применение `git commit --amend`:
```
$ git reset --soft HEAD@{1}
$ git commit -C HEAD@{1}
```

Вывести содержимое коммита:
```
$ git diff ⟨commit⟩^!
```

Вывести последние N коммитов:
```
$ git log -n ⟨N⟩
```

Избавиться от всех изменений:
```
$ git checkout .
```

Избавиться от изменений в 1 файле:
```
$ git checkout -- computer/img/pipelining_example.dia
```

Восстановить файл, который был удален, но это еще не было закоммичено:
```
$ git checkout HEAD ⟨file⟩
```

Search commits by str:
```
$ git log -S'select_column' -- hac_dashboard/utils/dynamic_table.py
```

Вывести последний тег:
```
$ git describe --abbrev=0
```

# Внутреннее устройство

Единицей хранения данных является объект, который идентифицируется 40-символьным хешем SHA1.
Объекты хранятся в `.git/objects`.
Основные типы объектов:
1. BLOB — содержимое файла (ни имени файла, ни прав доступа тут нет).
1. Tree — каталог.
1. Коммит, имеющий следующие свойства:
   - `tree` — корневой каталог репозитория;
   - `message`;
   - `author`, `commiter`;
   - родительский коммит или 2 родительских коммита (если это merge-коммит).
   Коммит не хранит дельту.

Узнать тип объекта:
```
$ git cat-file -t ⟨id⟩
```

Ветки — это файлы в каталоге `.git/refs/heads`.
Каждый такой файл содержит ид коммита, который является кончиком ветки.
Можно посмотреть этот ид:
```
$ git rev-parse master
```
Ветке удалённого репозитория соответствует файл `.git/refs/remotes/⟨repo name⟩/⟨remote branch⟩`.
Смотреть аналогично:
```
$ git rev-parse origin/master
```

Тегу соответствует файл в каталоге `.git/refs/tags`.

Текстовый файл `.git/HEAD` указывает на текущий коммит.
Если мы на ветке `master`, то `.git/HEAD` выглядит так:
```
ref: refs/heads/master
```
Если мы переключились на конкретный коммит `c61f0c0a`, то `.git/HEAD` выглядит так:
```
c61f0c0afdf2ae48db1f0b15cf1cc0023d88c203
```

# Подходы к выпуску релизов

### Git-flow

Простейший вариант git-flow подразумевает 2 основные ветки:
- в `develop` идёт разработка нового релиза;
- обновление `master` означает выпуск нового релиза, соответствующий коммит помечается тегом;
- фичи разрабатываются отдельных ветках, которые заливаются в `develop`.

![](img/simple_git_flow.png "Простейший вариант git-flow.")

Опционально:
- ветки для оперативного исправления серьёзных багов, которые заливаются и в `master` и в `develop`;
- ветки для подготовки релизов.

![](img/git_flow.png "Git-flow.")

Особенности:
1. Сложно.

См. также:
1. [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/) by Vincent Driessen.

### GitHub flow

Continuous delivery — это практика, при которой разработчики выпускают обновления непосредственно в production (путём слияний с `master`) в автоматическом режиме.

Ветка `master` всегда должна оставаться стабильной.

Особенности:
1. Очень просто.
1. Норм. для SaaS-приложений.

См. также:
1. [Understanding the GitHub flow](https://guides.github.com/introduction/flow/).
1. [GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html) by Scott Chacon.
1. [Simple Git workflow is simple](https://www.atlassian.com/blog/git/simple-git-workflow-is-simple) by Atlassian.

### GitLab flow

См. также:
1. [Introduction to GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
