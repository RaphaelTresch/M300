![](../images/Packer_36x36.png "Packer") Packerproblem bei Windows
======

**Probleme mit Packer und Windows**

Packer kann in Zusammenarbeit mit Windows zu Problemen führen. Es kann also sein, das Packker nur auf einem Linux oder MAC OS läuft. Folgend ist aber die Anleitung, wie man Packer installieren und nutzen kann.

*** Bei Problemen mit der Installation Foglendes testen ***
Zuerst müssen wir das paket Unzip installieren
```Shell
$ sudo apt-get install unzip
```
Danach laden wir Packer von der offiziellen website herunter, Link dazu hier [Hier](https://developer.hashicorp.com/packer/downloads)

```Shell
$ wget https://www.packer.io/downloads
```

Nun müssen wir die Datei entpacken
```Shell
$ sudo unzip packer_*_linux_amd64.zip -d /usr/local/bin/
```

Um Packer nun zu starten, geht man im Verzeichnis einen Ort zurück und führt die .json datei aus
```Shell
$ Packer build ubuntu-18.04-vagrant.json
```a


### Installation
***
1. Packer herunterladen von: https://www.packer.io/
    * Auf Button `DOWNLOAD ...` klicken
    * Windows 64 oder macOS 64-Bit anwählen
    * Warten, bis Datei heruntergeladen wurde
2. Heruntergeladene Datei `packer_....zip` entpacken
3. Terminal (*Bash*) öffnen & Ordner erstellen:
```Shell
    $ sudo mkdir ~/packer
    $ cd ~/packer 

    $ pwd                               #Gibt den Pfad des Ordners aus
    
    /Users[Dein-Benutzername]/packer
```
4. Entpackte Datei `packer` in das erstelltes Verzeichnis kopieren
```Shell
    $ cp packer ~/packer
```
5. Pfad in der Path-Variable hinterlegen (damit das System das Command kennt):
```Shell
    $ export PATH="$PATH:/Users[Dein-Benutzername]/packer"
```

> #### Änderung und anzeigen der Umgebungsvariablen
>
> Umgebungsvariablen können folgendermassen gesetzt und den anderen Prozessen innerhalb des Betriebssystems bekannt gemacht werden:
>
> Setzen der Variable: <Variablenname>=<Variableninhalt>
> Bekanntmachen der Variable: export <Variablenname>
> Löschen der Variable: unset <Variablenname>
> Abfrage der Variable: env
>
> Weitere Informationen zu Umgebungsvariablen:   
> https://de.wikipedia.org/wiki/Umgebungsvariable#%C3%84nderung_der_Umgebungsvariablen
>

6. Funktion von Packer testen:
```Shell
    $ packer

    Usage: packer [--version] [--help] <command> [<args>]

    Available commands are:
        build       build image(s) from template
        fix         fixes templates from old versions of packer
        inspect     see components of a template
        validate    check that a template is valid
        version     Prints the Packer version
```
7. Terminal (*Bash*) wieder schliessen & mit dieser Dokumentation fortfahren ...
   
### Image erstellen
***

Im nachfolgenden Abschnitt soll in Oracle VirtualBox ein Ubuntu Linux Image erstellt werden.

Was wir dazu benötigen ist eine Packer Konfiguration in JSON-Format und eine ISO-Datei mit einem Ubuntu Image.

Die Packer Konfigurationsdatei kann mit einem normalen Texteditor erzeugt werden und die ISO-Datei finden wir im Downloadbereich von ubuntu.com.

**Post-processors** <br>
Grob zusammengefasst holt Packer die ISO-Datei vom Internet erstellt einen leere VM mit angehängter ISO-Datei und versucht ohne Interaktion vom User ein Ubuntu Linux System zu installieren.

Damit Ubuntu Linux ohne User-Interaktion installiert werden kann braucht es noch eine zusätzliche PreSeed Konfigurationsdatei. Diese gibt dem Installer Anweisungen wie er standardmässig Verfahren soll.

```JSON
    "builders": [
        {
        "type": "virtualbox-iso",
    "boot_command": [
        " preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ubuntu-preseed.cfg<wait>",
    ],
```

Ein Beispiel:
```Shell
    debconf debconf/frontend select Noninteractive
    choose-mirror-bin mirror/http/proxy string
    d-i base-installer/kernel/override-image string linux-server
    # Default user
    d-i passwd/user-fullname string vagrant
    d-i passwd/username string vagrant
    d-i passwd/user-password password vagrant
    d-i passwd/user-password-again password vagrant
    d-i passwd/username string vagrant

    # Minimum packages (see postinstall.sh)
    d-i pkgsel/include string openssh-server
    d-i pkgsel/install-language-support boolean false
    d-i pkgsel/update-policy select none
    d-i pkgsel/upgrade select none
```

**Provisioning** <br>
Nach der Installation von Ubuntu Linux werden die Anweisungen in der provisioners Sektion ausgeführt. Hier eine Reihe von vorbereiteten Shell Scripts:
```JSON
    "provisioners": [
        {
        "type": "shell",
        "execute_command": "echo 'vagrant'|sudo -S sh '{{.Path}}'",
        "override": {
            "virtualbox-iso": {
            "scripts": [
                "scripts/server/base.sh",
```

**Post-processors** <br>
Nach Installation und Feintuning wird die Sektion `post-processor` abgearbeitet und ein aufbereitetes Images für den gewünschten Provider, hier Oracle VirtualBox erzeugt:
```JSON
    "post-processors": [
        {
        "type": "vagrant",
        "override": {
            "virtualbox": {
            "output": "ubuntu-server-amd64-virtualbox.box"
            }
        }
    }
```


### Sharing
***
Sobald eine Vagrant-Box gebaut wurde, ist die nächste Herausforderung diese Box mit anderen teilen.

Bei der Freigabe von Vagrant-Boxen gibt es ein paar Dinge zu berücksichtigen:
* Für wen ist die Box erstellt worden? 
    * Für die Öffentlichkeit den Vertrieb, speziell für ein Entwicklungs-Team?
* Wie gross ist die Box? 
    * Die meisten öffentlichen Basis Boxen sind eher klein. Boxen für Entwicklungs-Teams können eine Vielzahl von Tools enthalten und recht gross wird (z.B. IoTKit ist Box 15 GB gross).
* Beinhaltet die Box sensitive Daten wie z.B. Private Keys?

Anstelle der Möglichkeit eine Box auf einem (lokalen) HTTP-Server zu speichern, gibt es auch die Möglichkeit diese beim Entwickler von Vagrant unter https://vagrantcloud.com/ zu speichern.

**HTTP Server Variante** <br>
Nachdem die Box mittels Packer erstellt wurde, wird sie z.B. via SCP (Secure Copy) auf einen HTTP-Server (z.B. lokaler Apache-Webserver) kopiert.

Von dort kann sie dann mittels folgenden Einträgen im Vagrantfile angesprochen werden:
```Ruby
    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
        config.vm.box = "ubuntu-server-amd64-virtualbox.box"
        config.vm.box_url = "http://localhost:8080/ubuntu-server-amd64-virtualbox.box"
    end
```
