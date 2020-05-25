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
Ein Anwendungsfall ist zum Beispiel, dass man eine Utility-Implementierung korrigieren möchte, bevor sie
in aufgerufen wurde.

	git rebase -i master

Tausche die Zeilen

- `:/"Fix toUpper by using default local"` und
- `:/"use toUpper in Main output"`.

## Commits zusammenführen
Es gibt zwei rebase Befehle um Commits zusammenzuführen.

### squash
Die Commit-Nachrichten werden kombiniert.

	git rebase -i master
	# squash :/"add toUpp"

### fix
Die Commit-Nachricht des Fixes wird verworfen

	git rebase -i master
	# fix :/"Fix toUpper by using default local"

Im Zweifel merkt man sich `squash`, da damit nichts kaputt geht.

## Einen Commit aufteilen
Wir wollen Änderungen, die nicht in einen Commit gehören, in zwei Commits
aufteilen.

Editiere den Commit:

	git rebase -i master
	# edit :/"add toLower"
	git reset HEAD~

Jetzt sind wir in einem rebase und habe die Änderungen, die wir auftrennen
wollen ungetrackt. Diese Änderungen können wir jetzt einzeln Commit, z. B.
mit `git add -p` oder `git add -i`:

Änderungen in zwei Commits committen

	git add -p
	git commit # first partial changes
	git add -p
	git commit # more partial changes
	# repeat partial commit until done
	git rebase --continue

## Dateien eines Branches formatieren

Liste aller modifizierten Dateien:

	# Alle Datei, die in einem Branch geändert wurden.
	git log master..HEAD --pretty=format: --name-only

	# oder eindeutig
	git log master..HEAD --pretty=format: --name-only | sort | uniq

Damit lassen sich Datei direkt im Editor eurer Wahl öffnen.

	# öffnen in IntelliJ; mit Filter, da gelöschte Dateien nicht mehr geöffnet werden können.
	git log master..HEAD --pretty=format: --name-only --diff-filter=d | xargs idea

	# öffnen in Eclipse
	git log master..HEAD --pretty=format: --name-only --diff-filter=d | xargs eclipse --launcher.openFile

Die Dateien können jetzt der Reihe nach im Editor formatiert werden.
Besser wäre jedoch ein Formatierungstools wie Spotless, `gofmt` etc.
Danach werden die Dateien in einem Formatierungs-Commit committet.

Man kann auch den Anspruch stellen Dateien nie unformatiert committed zu haben.
Hier gibt es zwei Fälle, die in getrennt beschreiben werde. Allerdings sollte
man beide Fällen in einem Rebase behandeln.

-	Code formatieren
-	Einen Formatierungspatch erzeugen, **modifizierte und neue Patches getrennt**
-	`rebase -i master` und Formatierungs-Patch an den Anfang **kopieren**
	-	Durch das kopieren bleibt ein Formatierungs commit am Ende aber nur mit Formatierungen von neuem Code

### Modifizierten Code formatieren
Wir haben den Anspruch Dateien, die unformatiert zu committen. Nun kann es aber sein, dass
Dateien, die wir in dem Branch bearbeiten mussten, bereits in master unformatiert waren.

In diesem Fall sind wir gezwungen einen extra Formatierungs-Commit zu machen.
Diesen könnten wir wieder als letzten Commit unseres Branches machen. Allerdings
hätten wir dann streng genommen, die Datei auch zwischen durch unformatiert gelassen.

Es kann auch sein, dass wir beim Bearbeiten von Dateien, die komplette Datei
formatiert haben. Dies wird oft von IDEs beim Speichern getan. In diesem Fall
würde ein Patch aus einer Code-Änderung und aus Whitespace-Änderungen bestehen.
Der Vorteil eines Formatierungs-Commits am Anfang der Branch-Historie ist, dass
die Whitespace-Änderungen verschwinden.

Man kann über die Strategie streiten. Zu Übungszwecken, wollen wir aber einen
Formatierungs-Patch für modifizierten Code am Anfang der Historie.

Man kopiert den Formatierungs-Commit vom Ende der Historie an den Anfang.

	git rebase -i origin/master
	# Kopiere den letzten Commit an den Anfang

Dies führt sehr Wahrscheinlich zu Konflikten. Allerdings sind dies
Pseudo-Konflikte, da wir diese automatisch lösen können. Im Konfliktfall
merken, wir uns die Dateien und verwerfen alle Änderungen.

	git reset -- .
	git checkout -- .

Jetzt werden alle Dateien, die Konflikte hatten formatiert und committen.

	# format files
	git commit -a -m 'formatting'
	git rebase --continue

Für weitere Konflikte verfahren wir so wie im Fall vom Formatieren von neuem Code.

### Neuen Code formatieren
Formatierungs-Patch für neuen Code wollen wir beim Hinzufügen fixen.

Anders: Wir wollen den Code nie falsch formatiert haben.

Dafür gehen wir genauso vor, wie beim Formatieren von modifiziertem Code, d. h.
kopieren den Formatierungs-Commit vom Ende an den Anfang. Dies führt wieder zu
Konflikten, da der neue Code, der Formatiert wurde, noch gar nicht existiert.
Nachdem wir die Änderungen wieder verwerfen

	git reset -- .
	git checkout -- .

Jetzt ist der neue Code formatiert aber ungetrackt. `git` kann uns nun helfen,
die Commits zu finden in den wir formatieren hätten sollen.

	git rebase --continue

Findet git einen Konflikt, ist dieser wieder einfach zu beheben. Wir merken uns
die Datei, die einen Konflikt hat, und holen uns den Stand der angewendet werden soll.
Den Hash kann man zum Beispiel `git status` entnehmen:

	Last commands done (6 commands done):
	  pick 91f2c5c Fix toUpper by using default local
	  edit 33846bb add toLower and remove empty line

Also hier

	git co 91f2c5c -- .

Jetzt kann die Datei formatiert werden gefolgt von

	git add .
	git rebase --continue

Dieser Prozess wiederholt sich, bis der rebase durch ist.

## Jeden Commit testen
Dadurch stellt man sicher(er), dass man weiter mit bisect arbeiten kann.

	git rebase -i --exec './gradlew test' master

## Debugging mit bisect
Dies hat jetzt nicht direkt etwas mit rebasen zu tun, da Bug gerne mal gemeldet werden, nachdem
gemerget wurde.

Es wurde ein Bug gemeldet. Ja ein seltenes Szenario aber dennoch.
Wir wissen das der Bug noch nicht in Version 1 war. Wir wollen wissen,
wann der Bug beim Entwickeln von Version 2 reinkam.

„Titel werden nicht mehr UpperCase dargestellt.“

Als guter Entwickler schreiben wir natürlich **als erstes** einen Test,
der den Bug reproduziert.

	cat > java-project/src/test/java/MyTest.java <<EOF
	import org.junit.Test;
	import static org.junit.Assert.assertEquals;

	import de.sernet.talks.git.rebase.Util;

	public class MyTest {

			@Test
			public void test() {
					assertEquals("FOO", Util.toUpper("foo"));
			}
	}
	EOF

**WICHTIG** Diesen Test noch nicht committen. Solange der Test untracked ist,
können wir ihn bei bisect benutzten. Bisect führt eine Binäre-Suche zwischen
einem guten Commit (Version 1) und einem schlechten Commit (Version 2) durch.

	git bisect start
	git bisect bad
	git bisect good :/'Fix toUpper'

Git checkt automatisch einen Commit in der Mitte aus. Jetzt wieder holen wir

	./gradlew test
	git bisect bad|good # je nach Testergebnis

Besser jedoch ist

	git bisect run ./gradlew test

## Tipps

	git rebase --edit-todo
	git pull -r # git fetch && git rebase origin/<branch>

