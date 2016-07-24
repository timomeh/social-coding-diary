# Social Coding Diary 📚

<!-- TOC depthFrom:2 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Sessions](#sessions)
	- [#01 Git Basics](#01-git-basics)
	- [Own adventures - omg a pull request](#own-adventures-omg-a-pull-request)
	- [#02 Hacking & Contributing](#02-hacking-contributing)
	- [#03 Deploy](#03-deploy)
	- [#04 cologne.rb](#04-colognerb)
	- [#05 Talk Prepartion](#05-talk-prepartion)
	- [#06 Talks & Golf](#06-talks-golf)
	- [#07 Bugfixing & Testing](#07-bugfixing-testing)

<!-- /TOC -->

## Sessions

### #01 Git Basics

*20. April*

In der ersten Sessions wurden alle Basics zu git erklärt. Im Folgenden eine Auflistung:

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
Social Coding Diary 📚

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
Social Coding Diary 📖
```

### Own adventures - omg a pull request

In einem von mir nicht mehr gepflegten Repo wurde ein [Pull Request](https://github.com/timomeh/ejs-helper/pull/3) eröffnet. \\o/ Zwar wurden hier schonmal Pull Requests erstellt, doch habe ich ihn diesmal als *Merge and squash* durchgeführt. Die Änderungen waren nicht dramatisch genug, um alle 3 Commits des Forks mit in die History zu übernehmen. Außerdem war ich dabei dieses mal nicht so angespannt, dass ich mich gefühlt habe, als würde ich gleich in Flammen aufgehen.

Jetzt hab ich auch endlich rausgefunden, woher dieses "*user* committed with *user*" kommt!  
![](https://cloud.githubusercontent.com/assets/4227520/14833542/1ba2f954-0c00-11e6-8bc4-42ed31dbc198.png)

Nach dem Merge habe ich das im Repo vernachlässigte git-flow wieder anständig zurechtgebogen, die Version-Number gebumped, das alles gepushed und auf npm published.

Yeeeah.

### #02 Hacking & Contributing

*27. April*

Ziel der Session war es, bei einem vorhandenen Projekt mitzuwirken. Am besten natürlich etwas hacken und einen Pull Request dazu eröffnen, aber auch Issues erstellen und bei Issues helfen ist ein nicht zu unterschätzender Teil beim Contributing.

#### hacken.in

Eine mögliche Unterstützung wäre bei der 404-Page für hacken.in. Hierfür gibt es auch [Issue #630](https://github.com/hacken-in/website/issues/630), bei dem ich meine Unterstützung angeboten habe und positives Feedback bekommen habe. Problematisch wurde es dann allerdings beim Vagrant Setup, bei dem es Probleme mit der Ruby Version gab. Dazu habe ich einen weiteren [Issue](https://github.com/hacken-in/website/issues/642) eröffnet. Um das Problem zu beheben, wurde ein [Pull Request](https://github.com/hacken-in/website/pull/644) gestellt, der noch immer offen ist. Scheinbar gibt es noch Probleme mit dem Setup (siehe [#646](https://github.com/hacken-in/website/issues/646)).

#### atom-material-ui

Das [Material Theme für Atom](https://github.com/atom-material/atom-material-ui/) bekam ein größeres Update, bei dem selektierbare Elemente der Command Palette nicht als selektiert angezeigt wurden. Dazu hatte ich einen [Issue](https://github.com/atom-material/atom-material-ui/issues/265) erstellt. Nachdem der Repo-Eigentümer sich selbst assigned hat, wollte ich nicht direkt selbst den Issue beheben und "ihm in sein Handwerk pfuschen" – außerdem habe ich nicht viel Erfahrung, wie Atom Themes im HTML und CSS aufgebaut sind. Vor allem bei dem Material Theme ist das CSS nicht leicht zu durchblicken. Ich habe mich zwar dran versucht, den Fehler zu beheben (auch erstmal vorrübergehend für mich), und konnte die Schriftfarbe für ein selektiertes Element anpassen. Da das visuell nicht wirklich gut war, habe ich dazu keinen PR erstellt.
Der Issue wurde in der darauf folgenden Woche vom Repo-Eigentümer gefixt.

### #03 Deploy

*4. Mai*

#### Vagrant

Um lokal zu entwickeln, empfiehlt sich eine Virtualisierung mit [Vagrant](https://www.vagrantup.com). Vagrant ermöglicht es, mit einer Konfigurationsdatei eine lauffähige Virtuelle Maschine zu starten. Vagrant bedient sich hierbei anderen Virtualisierungs-Providern wie vmware oder VirtualBox.

Die Virtualisierung hat den Vorteil, dass man keine Pakete direkt auf dem eigenen Betriebssystem installieren muss. Man kann Konfigurationen anpassen und darf auch mal Fehler machen – im schlimmsten Fall muss man die Virtuelle Maschine nur löschen und neu installieren. Auch kann man, sollte man an einem Projekt länger nicht arbeiten, es einfach vom eigenen Rechner löschen und wieder installieren, sobald man es braucht; bei einer gut konfigurierten Vagrant-Box kann man einfach dort weitermachen, wo man zuvor aufgehört hat, ohne einen Unterschied zu merken.

#### heroku

Um Deploying zu veranschaulichen, hat Dirk ein [deploy_example](https://github.com/wpf-social-coding/deploy_example) zur Verfügung gestellt. Das ganze konnte auf heroku deployed werden.

Der Deploy auf heroku ist recht einfach. Nach einloggen (`heroku login`) und erstellen der App (`heroku create` erstellt eine "leere Umgebung"), kann direkt mit git auf heroku gepushed werden (`git push heroku master`). Nach dem Push erkennt heroku, um welche Programmierumgebung es sich handelt und führt dementsprechend den Task/die Tasks aus, um die Applikation zu builden und zu starten.  
Da das deploy_example mit einer Postgresql-Datenbank ausgestattet war, musste die Datenbank noch migriert werden (`heroku run rake db:migrate`). Je nach Umgebung muss möglicherweise der Datenbank-Provider auf heroku ausgewählt werden.

#### Selbst hosten

Heroku ist zwar einfach zu bedienen und bietet einen kostenlosen Plan an, allerdings mit Einschränkungen: Die Applikation schläft nach einer gewissen Zeit ein und muss bei einem Request erst aufwachen, was ein bisschen Zeit in Anspruch nimmt. Außerdem hat jeder Account nur 1000 Stunden kostenlose Nutzung (nicht-verifizierte Account 550 Stunden) für alle Free-Dynos.  
Eine eigene Serverumgebung kann in manchen Fällen die wirtschaflichere Lösung sein.

Um zu veranschaulichen, wie man einen eigenen Server aufsetzen kann, wurde ein Droplet auf Digital Ocean erstellt. Das erstellen eines Droplets ist ebenfalls recht einfach und kostet im Monat ab 5$ (im Vergleich zu Heroku, wo ein Dyno 7$ kostet).

Zwar hat man dadurch schnell einen Server zur Verfügung, muss ihn aber auch selbst konfigurieren und zuerst alle benötigten Pakete installieren. Dirk hat veranschaulicht, dass das sehr viel Zeit in Anspruch nehmen kann – vor allem, wenn man es immer wieder machen muss.

Abhilfe schafft eine Automation wie [Ansible](https://www.ansible.com/). Ansible besitzt Konfigurationsdateien, mit denen Pakete installiert werden können, Konfigurationsdateien angepasst werden können und alle notwendigen Dienste gestartet werden können.  
Auch hat das den Vorteil, dass die Ansible-Konfiguration sowohl mit Vagrant in einer VM als auch auf dem Server genutzt werden kann.


### #04 cologne.rb

*18. Mai*

Zu dieser Session waren wir bei der cologne.rb Usergroup, um den typischen Ablauf einer Usergroup zu erleben. In einer gemütlichen Runde wurden 3 Talks gehalten.

Der erste Talk handelte von Public/Private-Key Verschlüsselung aus mathematischer Sicht.

Beim zweiten Talk wurde das Golfen vorgestellt. Ziel beim Golfen ist es, kleine Programme in so wenig Zeichen wie möglich zu schreiben. (z.B. 140 Zeichen, um es in einen Tweet senden zu können.) Der Code soll nicht schön oder lesbar sein, sondern schlichtweg so kurz wie möglich.

Beim dritten Talk wurde eine Rails-App "gegolft". Am Schluss konnte eine ganze lauffähige Rails-Applikation in ~140 Zeichen erstellt werden. Natürlich besitzt eine solche Applikation dann keine besonderen Funktionen; es hilft aber stark dabei zu verstehen, wie eine größere Architektur wie Rails aufgebaut ist: welche Komponenten wann und wie greifen.

### #05 Talk Prepartion

*25. Mai*

In dieser Session wurde gezeigt, wie man einen eigenen Talk halten kann und sich darauf vorbereitet. Talks können sowohl in kleinerem Rahmen gehalten werden (Firmenintern oder auf Usergroups) oder auch auf größeren Konferenzen.

Hilfreich ist es, am Anfang eine Folie zu haben ("Folie 0"), die für sich alleine dastehen kann, bevor der eigentliche Talk beginnt: z.B. ein subtiles, passendes gif.  
Zu Beginn sollte man das Thema nennen und sich selbst vorstellen.  
Eine Folie mit Twitter-Handle o.ä. eignet sich auch als Schluss-Folie, die länger stehen bleiben kann.  
Im Mittelteil hilft es, wenn man seine Slides strukturiert darstellt und wiederkehrende visuelle Elemente einsetzt, um die Strukturierung des Talks zu unterstützen.

Beim Erstellen des Talks sollte man sich zuerst eine Outline zurecht legen. In der Outline wird grob notiert, in welcher Reihenfolge welche (groben) Themen angesprochen werden. Das muss nicht allzu genau sein.

#### Eigener Talk

Zum Ende sollte ein eigenes Thema für einen Talk gefunden werden und eine Outline verfasst werden.

Als Thema habe ich mir das Module Loading System in Node.js ausgesucht und dazu eine [talk-outline.md](Outline) erstellt.

In der Retrospektiven waren einige Teile der Outline zu genau und andere etwas zu unüberlegt. Auch war das Wording von Themenblöcken nicht final, weshalb ich mich beim Erstellen des Talks außerdem noch darum kümmern musste.

### #06 Talks & Golf

*1. Juni*

#### Talks

Zu Beginn der 6. Session wurden die vorbereiteten Talks gehalten.

**Kritik am eigenen Talk:** Ich hatte meine Outline nochmals kurzfristig umgestellt, wodurch leider der rote Faden des Talks verloren ging. (Ursprünglich wollte ich den Weg zeigen, wie ich versucht habe, ein eigenes Module Loading System zu bauen und dann zufällig das native Module Loading System grob nachgebaut habe.) Außerdem hatte ich keine Folien, um mich vorzustellen. Auch war die Abschluss-Folie etwas seltsam gewählt, was gewirkt hatte, als würde ich mitten im Thema aufhören. Es war kein echter Schluss erkennbar.

#### Golf

Im Anschluss haben wir gemeinsam versucht, das im Golf-Talk von [Session #4](#04-colognerb) gezeigte golfen selbst umzusetzen.

Mein Ergebnis war ein HTTP Server in Node, der als Proxy Bilder von [loremflickr.com](http://loremflickr.com) ausgegeben hat.

```sh
node -e "h=require('http');h.createServer((a,b)=>h.get('http://loremflickr'+'.com/320/240'+a.url,r=>r.pipe(b)).end()).listen(1337)"
```

Zusammen mit dem Hashtag #nodejs hat das in [einen Tweet](https://twitter.com/timomeh/status/738021433939505152) gepasst.

### #07 Bugfixing & Testing

*8. Juni*

In dieser Session haben wir uns gemeinsam Bugfixing und Testing angeschaut. Dazu hat Dirk zum austoben [batman-ipsum](https://github.com/wpf-social-coding/batman-ipsum) zur Verfügung gestellt. Auf drei Branches waren drei verschiedene Bugs, die wir gefixt haben und dazu jeweils einen Pull Request geöffnet haben.

Um Anschluss wurde uns Testing am Beispiel von Ruby mit batman-ipsum gezeigt. Die Konzepte davon sind aber auf die meisten Programmiersprachen übertragbar.

Oft hat man `describe` und `it` zur Verfügung.

Mit `describe` kann man einen Block mit mehreren Tests erstellen. Ein Test befindet sich jeweils in einem `it`.

In dieser Session haben wir vorwiegend um Unit Tests gekümmert.
