# Social Coding Diary 📚

<!-- TOC depthFrom:2 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Sessions](#sessions)
	- [#02 Git Basics](#02-git-basics)
- [Own adventures](#own-adventures)
	- [omg a pull request](#omg-a-pull-request)

<!-- /TOC -->

## Sessions

### #02 Git Basics

#### Verschiedene "Dateispeicher"

- Stash (Ablage für später)
- Working directory
- Lokales Repo
- Remote Repo (z.B. GitHub)
- *(es kann variabel viele remotes geben)*

#### nützliches im .git Ordner

- `hooks` für Skripte, die automatisch an verschiedenen Stellen ausgeführt werden können
- `objects` sind *alle* Dateien, die git zur Versionierung braucht
  - Trees
  - Commits
  - Textdateien
  - Binary Files
- `config` ist eine Konfigurationsdatei pro Repo
- *(globale Konfigurationen in ~/.gitconfig möglich)*


#### nützliche globale Konfigurationen

```git
[alias]
  # alias für einen übersichtlichen git log-graphen
  lg1 = log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all

  # alias für einen alternativen, etwas umfangreicheren git log-graphen
  lg2 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all

  # alias `git lg` verweist auf lg1
  lg = !"git lg1"

[pull]
  # pull nur mit --rebase ausführen
  rebase = true
```

#### HEAD

Der `HEAD` zeigt in den meisten Fällen auf den aktuellsten Commit eines Branches.

`HEAD~1` bzw. `HEAD~` ist ein Commit vor dem Head.  
`HEAD~2` ist zwei Commits vor dem Head.  
`HEAD~3` ist drei Commits vor dem Head.  
...

#### Really basic commands

```sh
# Repo clonen
$ git clone <remote-url>

# Änderungen von remote holen
$ git pull
# ... ist kurz für
$ git fetch && git merge
# Besser ist aber
$ git pull --rebase
# ... ist kurz für
$ git fetch && git rebase

# Status von aktuellem working tree
$ git status

# Lokale Änderung hinzufügen (stagen)
$ git add <my-file>

# Staged files committen
$ git commit -m "Eine hilfreiche Nachricht dazu angeben"

# Commits auf remote übertragen
$ git push
```


#### Basic Branching

```sh
# Übersicht über alle Branches
$ git branch

# Branch erstellen und darauf wechseln
$ git checkout -b <my-new-branch>

# Branch wechseln
$ git checkout <master>

# Auf einen Branch vom remote wechseln, dabei
# einen neuen lokalen Branch anlegen, der auf
# jenen Branch verweist
$ git checkout -t <some-remote-branch>
```


#### ... and merging

Bei einem merge werden ein oder mehrere Commits zusammengefasst.

```sh
# Änderungen von anderem Branch in aktuellen Branch mergen
$ git merge <some-branch>

# Mehrere histories zusammen mergen (Octopus merge)
$ git merge <some-branch> <another-branch> <even-a-commit-sha>
```

#### Signing stuff

Mittels GPG lassen sich Tags und Commits in git "unterzeichnen", also mit einem public-key authentifiziert.

Für Mac ist die [GPG Suite von GPGTOOLS](https://gpgtools.org/) recht angenehm. Der darüber erstellte und veröffentlichte Public Key wird dann in der globalen gitconfig anhand der key-id hinterlegt:

```sh
$ git config --global user.signignkey <key-id>
```

Außerdem muss der Public-GPG-Key in den GitHub Settings hinterlegt werden.

Linus Torvalds sagt [hier](http://git.661346.n2.nabble.com/GPG-signing-for-git-commit-td2582986.html#a2583316), dass es nicht so schlau wäre, wenn man jeden Commit signed. Stattdessen wäre es cool, wenn man Tags signed.

> Signing each commit is totally stupid. [...] Be happy with 'git tag -s'. It really is the right way.

Wenn Linus das sagt, glaube ich ihm mal. Soll ja ein ganz intelligenter Mann sein.

#### Extravaganza

Git ist eigentlich ein simpler key-value-store. Zu einem sha-hash (key) wird der Inhalt gespeichert (value). Um ein Object anzusehen, gibt es `git cat-file`.

Hier ein Beispiel:

```sh
$ git cat-file -t 77e439d # -t gibt type aus, danach commit-sha
commit
$ git cat-file -p 77e439d # -p gibt object lesbar aus
tree da219328a6b5e3b305368ef458397921bb498b49
parent 53851de02ac7ce9cf2f4b30d456c8c6aaefc4377
author Timo Mämecke <timo@maemecke.com> 1461166362 +0200
committer Timo Mämecke <timo@maemecke.com> 1461166362 +0200

Changed emoji
$ git cat-file -p da21932 # sha von "tree"
100644 blob 7609c14e153564914d973c2a4b3e19b6491b36c9    README.md
$ git cat-file -p 7609c14 # sha von der Datei README.md
\# Social Coding Diary 📚

$ git cat-file -t 53851de # sha von "parent"
commit
$ git cat-file -p 53851de
tree c411b63810a9026b9ff14da66a13cf7170399fb9
parent 24f75f2792a99c742936a2131039c98f94139e56
author Timo Mämecke <timo@maemecke.com> 1461165050 +0200
committer Timo Mämecke <timo@maemecke.com> 1461165050 +0200

Added cool emoji
$ git cat-file -p c411b63 # sha von neuem "tree"
100644 blob c5301155139a323b1ff0ef3333b92bbc884d4251    README.md
$ git cat-file -p c530115 # sha von der Datei README.md
\# Social Coding Diary 📖
```

## Own adventures

### omg a pull request

In einem von mir nicht mehr gepflegten Repo wurde ein [Pull Request](https://github.com/timomeh/ejs-helper/pull/3) eröffnet. \\o/ Zwar wurden hier schonmal Pull Requests erstellt, doch habe ich ihn diesmal als *Merge and squash* durchgeführt. Die Änderungen waren nicht dramatisch genug, um alle 3 Commits des Forks mit in die History zu übernehmen. Außerdem war ich dabei dieses mal nicht so angespannt, dass ich mich gefühlt habe, als würde ich gleich in Flammen aufgehen.

Jetzt hab ich auch endlich rausgefunden, woher dieses "<user> committed with <user>" kommt!  
![](https://cloud.githubusercontent.com/assets/4227520/14833542/1ba2f954-0c00-11e6-8bc4-42ed31dbc198.png)

Nach dem Merge habe ich das im Repo vernachlässigte git-flow wieder anständig zurechtgebogen, die Version-Number gebumped, das alles gepushed und auf npm published.

Yeeeah.
