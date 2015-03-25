# mypopen_mypclose
similar to popen(3)/ pclose(3)

popen(3) / pclose(3)

Schreiben Sie eine kleine Library mit den zwei Funktionen mypopen() und mypclose(), die sich wie popen(3) bzw. pclose(3) verhalten (siehe Skriptum oder Online-Manual). Implementieren Sie diese beiden Funktionen in einer eigenen Übersetzungseinheit namens mypopen.c und exportieren Sie alle nötigen Informationen (z.B., die extern Deklarationen) in einem zugehörigen Headerfile namens mypopen.h.

Mit Hilfe der Funktionen mypopen() und mypclose() können Sie relativ einfach ein Shell-Kommando ausführen und das Ergebnis direkt in ein Programm einlesen und weiterverarbeiten bzw. Daten, aus einem Programm heraus, an dieses Kommando übergeben.

popen(command, "r") liefert einen Filepointer auf eine pipe, von der die Ausgabe des Befehls command gelesen werden kann. popen(command, "w") liefert einen Filepointer auf eine pipe, auf die die Eingabe für den Befehl command geschrieben werden kann.

Anleitung

mypopen() muß zunächst eine Pipe einrichten (pipe(2)) und dann einen Kindprozeß generieren (fork(2)). Im Kindprozeß ist das richtige Ende der Pipe ("r" oder "w") mit stdin bzw. stdout zu assoziieren (dup2(2)) und das im 1. Argument spezifizierte Kommando auszuführen (execl(3) oder execv(3)). Verwenden Sie - wie die Funktion popen(3) - zum Ausführen des Kommandos die Shell sh(1). Als letztes muß mypopen() von einem Filedeskriptor einen passenden FILE * mit fdopen(3) erzeugen.

Bei Aufruf von mypclose() soll der aufrufende Prozeß auf die Terminierung des Kindprozesses warten (waitpid(2)). Zur Vereinfachung soll immer nur höchstens eine Pipe mit mypopen() geöffnet werden können. Stellen Sie dies in ihrer Implementierung sicher.

Signalbehandlung ist für dieses Beispiel nicht notwendig.

Fehlerbehandlung

mypopen() soll sich (analog zu popen(3)) wie eine Libraryfunktion verhalten. D.h. es wird niemals irgendwas auf stderr (oder sonst wohin) geschrieben - die Entscheidung, ob das Nicht-funktionieren (bzw. auch das Funktionieren) der Libraryfunktion einen Fehler für die Applikation darstellt, obliegt immer der Applikation. Selbiges gilt auch für Fehlermeldungen bzw. die Art und Weise derselben.

Jeglicher schief gegangene System-Call sowie sonstige Fehler sind über den Returnwert der Funktion "nach oben" zu melden. Im Falle von mypopen() ist auch errno passend zu erhalten (falls ein System-Call einen Fehler meldet) bzw. direkt zu setzen (z.B. falls unerwartete/falsche Parameter übergeben werden ist EINVAL ein guter Wert dafür).

Weiters fork(2)t die Libraryfunktion auch, und auch im Kindprozeß können Fehler auftreten (z.B. könnte execv(3) schief gehen). Im Kindprozeß darf nie ein return passieren, weil dann die mypopen() Funktion 2 Mal zurückkommt - einmal im Elternprozeß und einmal im Kindprozeß - und damit rechnet keine Applikation.

Testen

popentest

Zum Testen Ihrer mypopen()/mypclose() Implementierung steht Ihnen am annuminas eine automatisierte Test-Suite Library namens libpopentest.a in /usr/local/lib zur Verfügung (Sourcen liegen in /usr/local/src/libpopentest.zip bzw. /usr/local/src/libpopentest.tar.gz), welche einige sinnvolle Tests implementiert. 
Die Library libpopentest.a enthält dabei das gesamte Testprogramm (inkl. der main()-Funktion) jedoch ohne mypopen() und ohne myclose() - diese kommen ja von Ihnen. 
Linken Sie Ihr mypopen()/mypclose() Modul (mypopen.o) gegen diese Library, um ein Executable (popentest) zu erhalten, welches ohne Argumente aufgerufen werden kann und die in der Library implementierten Tests durchführt. Dies machen Sie auf der Commandline z.B. mit der Zeile


    $ gcc -o popentest mypopen.o -lpopentest
    
Praktischerweise werden Sie diese Zeile in eine entsprechende Regel in das Makefile schreiben. 
Benutzen Sie diese Test Suite, um Ihre Lösung zu diesem Beispiel zu testen. Jegliche fehlschlagenden Tests deuten auf Fehler in Ihrer Lösung hin, die zu Punkteabzügen führen werden.

test-pipe

Zusätzlich steht Ihnen am annuminas eine interaktive Test-Suite Library namens libtest-pipe.a in /usr/local/lib zur Verfügung (Sourcen liegen in /usr/local/src/libtest-pipe.zip bzw. /usr/local/src/libtest-pipe.tar.gz), welche das interaktive Testen Ihrer mypopen()/mypclose() Implementierung ermöglicht. 
Linken Sie Ihr mypopen()/mypclose() Modul (mypopen.o) gegen diese Library, um ein Executable namens test-pipe zu erhalten, welches ohne Argumente aufgerufen werden kann und die in der Library implementierte interaktive Test-Suite durchführt. Dies machen Sie auf der Commandline z.B. mit der Zeile


        $ gcc -o test-pipe mypopen.o -ltest-pipe
      
Praktischerweise werden Sie diese Zeile in eine entsprechende Regel in das Makefile schreiben. 
Das Programm test-pipe verlangt erst die Eingabe eines Kommandos und danach "r" oder "w", je nachdem ob von der Pipe gelesen oder auf die Pipe geschrieben werden soll. Wurde die Pipe mit "r" geöffnet, werden von der Pipe gelesene Zeichen auf stdout ausgegeben. Wurde sie mit "w" geöffnet werden von stdin gelesene Zeichen in die Pipe kopiert. In jedem Fall wird die Anzahl der gelesenen oder geschriebenen Zeichen auf stdoutausgegeben.

Beispiel Sessions test-pipe

 
    $ ./test-pipe
    Command> ls -al
    Type (r/w)> r
    total 52
    drwxrwxrwx+   3 tom      Domain U        0 Mar 19 00:26 .
    drwxrwxrwx+   8 Administ Domain U        0 Mar 18 09:46 ..
    -rwxrwxrwx    1 tom      Domain U     1863 Mar 19 00:17 Makefile
    drwxrwxrwx+   4 tom      Domain U        0 Mar 19 00:26 doc
    -rwxrwxrwx    1 tom      Domain U     8675 Mar 19 00:25 doxygen.dcf
    -rwxrwxrwx    1 tom      Domain U     5733 Mar 18 23:46 main.c
    -rw-r--r--    1 tom      Domain U     2996 Mar 19 00:17 main.o
    -rwxrwxrwx    1 tom      Domain U     4194 Mar 18 11:26 mypopen.c
    -rwxrwxrwx    1 tom      Domain U     1888 Mar 18 10:44 mypopen.h
    -rw-r--r--    1 tom      Domain U     1632 Mar 19 00:17 mypopen.o
    -rwxr-xr-x    1 tom      Domain U    18051 Mar 19 00:17 test-pipe.exe
    -rwxrwxrwx    1 tom      Domain U     1596 Mar 19 00:00 utils.c
    -rwxrwxrwx    1 tom      Domain U     1389 Mar 19 00:00 utils.h
    -rw-r--r--    1 tom      Domain U      793 Mar 19 00:17 utils.o
    905 characters read.
    

    $ ./test-pipe
    Command> grep foo
    Type (r/w)> w
    Die Betriebssysteme Uebung ist total genial.
    Der Vortragende ist ein foo.
    Der Vortragende ist ein foo.
    Abbrechen mit Control D.
    99 characters written.
    
Zum Vergleich sind vorkompilierte Executables von test-pipe und popentest in /usr/local/bin am annuminas bereitgestellt.

Vorbereitung

3 zentrale Systemcalls dieses Beispiels sind fork(2), execl(3) und dup2(2).

Experimentieren Sie mit dem fork(2) Systemcall:
Erarbeiten Sie ein Testprogramm, daß einen Kindprozeß in die Welt setzt und (in beiden Prozessen) die eigene Prozeßid (getpid(2)) und die des Elternprozesses (getppid(2)) ausgibt.
Für Fortgeschrittene: Erarbeiten Sie ein Testprogramm, daß zählt und ausgibt, wieviele Prozesse angelegt werden können. Dazu implementieren Sie eine Funktion, die einen Kindprozeß erzeugt und - nur wenn das erfolgreich war - auf dessen Terminierung wartet (z.B. mit wait(2)). In dem Kindprozeß rufen sie rekursiv obige Funktion auf. 
Die Abbruchbedingung der Rekursion ist ein Fehler beim Systemcall fork(2). Führen Sie einen Zähler mit und geben Sie diesen beim Abbruch aus. 
Um eine klassische "fork()-Bombe" zu vermeiden bzw. auch im Falle des Falles die Chance zu haben, diese selber wieder zu entsorgen, machen Sie am besten vor jedem fork(2) eine Pause von einer halben Sekunde mit usleep(500000).
Filedeskriptoren (wie sie z.B. open(2) liefert) sind normale int Variablen und der Index in der Filedescriptortable (im Kernel-Teil) des Prozesses. Schreiben Sie ein Programm, daß dup2(2) ausprobiert und gehen Sie dabei wie folgt vor:
Legen Sie sich mit Ihrem Lieblingseditor ein kurzes Textfile (kurzes_textfile.txt) an (wenige Zeilen Text genügen).
Öffnen Sie in Ihrem Programm ein beliebiges (anderes) File (beliebiges_file.txt) zum Schreiben (mit open(2)).
Schreiben Sie in Ihrem Programm eine Zeile Text in das File beliebiges_file.txt (mit write(2)).
Duplizieren Sie in Ihrem Programm den Filedeskriptor auf STDOUT_FILENO (== 1) mit dup2(2).
Schließen Sie in Ihrem Programm den ursprünglich geöffneten Filedeskriptor (den den Sie als Rückgabewert von open(2) erhalten haben) mit Hilfe von close(2) wieder.
Führen Sie das in Ihrem Programm das Programm /bin/cat mit execl(3) aus und übergeben Sie als einzigen Parameter den Filenamen des kurzen Textfiles (kurzes_textfile.txt).
Was fällt ihnen am Inhalt des beliebigen Files (beliebiges_file.txt) auf?
Testsuites für mypopen()/mypclose():
Nehmen Sie die Test-Suites für mypopen()/mypclose() dadurch in Betrieb, daß Sie die in der stdio Library vorhandenen Implementierungen von popen(3) und pclose(3) mit eigenen Funktionen namens mypopen()/mypclose() wrappen und dann diese Wrapper gegen die Libraries der Test-Suites linken.

    #include <stdio.h>
    
    FILE *mypopen(const char *command, const char *type)
    {
        return popen(command, type);
    }
	      
    int mypclose(FILE *stream)
    {
        return pclose(stream);
    }
	    
Erstellen Sie Regeln im Makefile, die mit obigen 2 Wrapper-Funktionen die Binaries test-pipe und popentest erzeugen und probieren Sie sie aus. Dabei werden nicht alle Tests bei popentest funktionieren (weil das Fehlerchecking am Beginn von popen(3) ziemlich minimal ist). 
In Folge ersetzen Sie die Aufrufe von popen(3) und pclose(3) durch Ihre eigene Implementierung der 2 Funktionen.
Zusätzliche Richtlinien

Neben den üblichen C Richtlinien gilt für dieses Beispiel folgendes:

Vergessen Sie auch bei diesem Beispiel nicht auf die Argumentbehandlung (auch bei einem Programm welches keine Argumente erhält ist eine Argumentbehandlung durchzuführen)!
Falls Sie Pipes erzeugen, sollte das wie im Übungsskriptum beschrieben geschehen. Für sämtliche I/O-Operationen (Lesen/Schreiben von stdin, stdout Pipes usw.) sind ausschließlich die Standard I/O-Funktionen (fdopen(3), fopen(3), fgets(3), etc.) zu verwenden. Bedenken Sie hierbei, dass die Standard I/O Funktionen gepuffert sind.
Alle Ressourcen (wie z.B. Pipes) müssen ordnungsgemäß vor Terminierung entfernt werden.
Die Terminierung aller Kindprozesse sicherzustellen ohne kill(2) oder killpg(2) zu verwenden. Der Exit Status beendeter Kindprozesse muß vom Vaterprozess abgeholt werden (wait(2), waitpid(2), wait3(2)).
Signalbehandlung ist für dieses Beispiel nicht erforderlich!
