M300 - 25 Infrastruktur-Sicherheit Meine Arbeit
======

### Firewall

Als erstes habe ich mit Vagrant eine Firewall auf Ubuntu erstellt. Ich habe mir auch schon gedanken zur IP-Adressen vergabe gemacht. Ich werde mich im Class B bewewgen. Ich habe bereits am Afang begonnen, eine Netzwerkdokumentation zu ertellen, damit ich immmer einen Überblick habe, welche IP-Adresse ich vergeben habe, und wie es aufgebaut ist. Der Firewall habe ich die IP 172.16.1.10 gegeben. Als Firewall habe ich die UFW Firewall installiert. Ich habe die Firewall so Konfiguriert, dass sie die den Datenverkehr welcher auf den Port 80 eingeht erlaubt. Ausserdem habe ich den Datenverkehr auf den TCP Port 3306 von der IP 172.16.1.200 (meine Test VM) erlaubt.


### Reverse Proxy
Im Punkt 20-Infrastruktur habe ich einen Webserver Konfiguriert, dieser möchte ich hinter dem Reverseproxy "Verstecken". Eingehender Datenverkehr vom Port 10443 wird auf den port 80 Umgeleitet