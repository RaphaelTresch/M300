M300 - 25 Infrastruktur-Sicherheit Meine Arbeit
======

### Firewall

Als erstes habe ich mit Vagrant eine Firewall auf Ubuntu erstellt. Ich habe mir auch schon gedanken zur IP-Adressen vergabe gemacht. Ich werde mich im Class B bewewgen. Ich habe bereits am Afang begonnen, eine Netzwerkdokumentation zu ertellen, damit ich immmer einen Überblick habe, welche IP-Adresse ich vergeben habe, und wie es aufgebaut ist. Der Firewall habe ich die IP 172.16.1.10 gegeben. Als Firewall habe ich die UFW Firewall installiert. Ich habe die Firewall so Konfiguriert, dass sie die den Datenverkehr welcher auf den Port 80 eingeht erlaubt. Ausserdem habe ich den Datenverkehr auf den TCP Port 3306 von der IP 172.16.1.200 (meine Test VM) und von der IP 172.16.1.11 Erlaubt.

Meine Web VM bekommt die IP 172.16.1.11 Es wird dort eine Ordnersynchronisation mit einem Ordner auf dem Host eingerichtet, welche das HTML file synchronisiert


### Reverse Proxy
Im Punkt 20-Infrastruktur habe ich einen Webserver Konfiguriert, dieser möchte ich hinter dem Reverseproxy "Verstecken". Eingehender Datenverkehr vom Port 10443 wird auf den port 80 Umgeleitet Ausserdem wurde auch hier eine Ordnersynchronisation erstellt, um die Konfigurationsdatein zu Syncen. 

### Zusammenfügen von ReverseProxy und Firewall
Zum schluss, habe ich alles in ein Vagrant file zusammengesetzt, Dieses Vagrant file, ist auch abgelegt


### Authentifizierung und Autorisierung

Im folgenden Beispiel wird ein Vagrantfile gezeigt, das die Verwendung der "ubuntu/xenial64" Box demonstriert. Die virtuelle Maschine wird mit einer statischen IP-Adresse konfiguriert und ist über die Ports 80 und 443 erreichbar. Der Ordner "config/" auf dem Host-System wird mit dem Ordner "/etc/apache2/sites-available/" auf der virtuellen Maschine synchronisiert, um den Austausch von Konfigurationsdateien für das SSL-Zertifikat zu ermöglichen.

Als erstes wird Apache installiert und konfiguriert, um SSL zu unterstützen und ein selbst signiertes SSL-Zertifikat zu generieren. Die Standardkonfiguration von Apache wird deaktiviert und durch eine neue Konfiguration ersetzt. Optional kann ein Benutzername und das dazugehörige Passwort im Vagrantfile angegeben werden. Mit diesen Anmeldedaten kann man sich dann auf der Website anmelden, um Zugriff zu erhalten. Abschließend wird der Apache-Dienst neu gestartet.


#### Quelle
Da ich im Punkt 25 meine Schwierigkeiten hatte, hat mir Luka Petkovic aus der ST20a geholfen, anhand seiner arbeit und mir dies erlklärt.