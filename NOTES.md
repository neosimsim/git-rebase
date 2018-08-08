# git ‒ Clean Branching
Ich werde hier bewusst auf git cli Ebene bleiben.
Wenn man das kann, kann man git.

Kernbefehle

	git rebase -i master
	git rebase --continue
	git rebase --abort
	# Force Push
	git push -f

## Einfache Änderung an einem Commit
### Tippfehler in einer Datei
Wir wollen einen Kommentar in einer Java-Datei korrigieren.

	git rebase -i master
	# edit :/"add utility class"
	sed -i 's/clas /class /' ./java-project/src/main/java/de/sernet/talks/git/rebase/Util.java

### Tippfehler in einer Commit-Nachricht
Wir wollen eine Commit-Nachricht korrigieren.

	git rebase -i master
	# reword :/"add toUppr util method"

## Reihenfolge vom Commits ändern
Tausche

- `:/"Fix toUpper by using default local"` und
- `:/"use toUpper in Main output"`.

Man möchte in der Historie vlt. erst alles richtig machen und erst dann verwenden.

## Commit zusammenführen
### squash
Die Commit-Nachrichten werden kombiniert.

	git rebase -i master
	# squash :/"add toUpp"

### fix
Die Commit-Nachricht des Fixes wird verworfen

	git rebase -i master
	# fix :/"Fix toUpper by using default local"

## Einen Commit aufteilen

	git rebase -i master
	# edit :/"add toLower"
	git reset HEAD~

Änderungen in zwei Commits committen

	git commit --amend # first partial changes
	git commit # more partial changes
	git rebase --continue

## Dateien eines Branches formatieren

Liste aller modifizierten Dateien:

	# Alle Datei, die gemerged werden listen
	git log master..HEAD --pretty=format: --name-only

	# oder eindeutig
	git log master..HEAD --pretty=format: --name-only | sort | uniq

	# öffnen; mit Filter, da gelöschte nicht mehr geöffnet werden können.
	git log master..HEAD --pretty=format: --name-only --diff-filter=d | xargs idea

-	Code formatieren
-	Einen Formatierungspatch erzeugen, **modifizierte und neue Patches getrennt**
-	`rebase -i master` und Formatierungs-Patch an den Anfang **kopieren**
	-	Durch das kopieren bleibt ein Formatierungs commit am Ende aber nur mit Formatierungen von neuem Code

### modifizierte Code
Formatierungs-Patch für modifizierten Code wollen wir am Anfang der Historie.

### neuer Code
Formatierungs-Patch für neuen Code wollen wir ans Hinzufügen fixen.

Anders: Wir wollen den Code nie falsch formatiert haben.

## Jeden Commit testen
Dadurch stellt man sicher(er), dass man weiter mit bisect arbeiten kann.

	git rebase -i --exec './gradlew test' master

## Tipps

	git rebase --edit-todo
	git pull -r # git fetch && git rebase origin/<branch>

