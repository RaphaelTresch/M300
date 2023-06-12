![](../images/AWS_36x36.png "AWS Cloud") 05 - AWS Cloud
======

### Grundlagen
***

**Root Account** <br>
Bezeichnet den Inhaber des AWS-Benutzerkontos. Für den Root sind alle Funktionen in der Cloud freigeschaltet, weshalb mit diesem Benutzer nicht direkt gearbeitet werden soll.

**Regionen** <br>
AWS hat unabhängige Rechenzentren in unterschiedlichen Regionen der Welt, z.B. Irland, Frankfurt, Virginia

**IAM User** <br>
Identity-Management (IAM) ist ein Verwaltungssystem, welches dem Root erlaubt, eigenständige User anzulegen und mit unterschiedlichen Rechten (Permissions & Policies) auszustatten. 

**Network and Security** <br>
Bei AWS gibt es eine Funktion in der EC2-Konsole, welche es erlaubt Security Groups, Key Pairs etc. zu verwalten.

*Security Groups* legen fest welche Ports nach aussen offen sind und können für mehrere VMs gleichzeitig eingerichtet werden.

*Key Pairs* sind Private & Public Keys. Wobei der Public Key bei Amazon verbleibt und der Private Key vom User lokal abgelegt wird um damit auf die VMs in der Cloud zugreifen zu können. 

**AWS Images** <br>
Es gibt vorbereitete VM-Images von AWS, welche einfach über die EC2-Konsole instanziert werden können.


### Vagrant & AWS
***

**Vorbereitungen** <br>
1. Zuerst ist das AWS Vagrant Plugin zu installieren(in der AWS Console):
```Shell
    $ vagrant plugin install vagrant-aws
```
2. Anschliessend ein Dummy-Image downloaden:
```Shell
    $ vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
```

Das Dummy-Image wird als Stellvertreter für das effektive Image in der Amazon Cloud gebraucht.

**AWS einrichten** <br>
1. Als erstes ist unter https://aws.amazon.com/de/ ein Root Account einzurichten
    * Dazu braucht es eine gültige Kreditkarte und eine direkte Telefonnummer! 
2. IAM:
    * Nun folgt das Einrichten des Vagrant-Users mit den Rechten, eine EC2 Instanz (AWS-Image) zu instanzieren
    * Dazu ist auf `Identity and Access Management` zu wechseln
    * Auf `Create New Users` zu klicken:
        * **Enter User Names:** vagrant-user
        * **Haken setzen bei:** [X] Generate an access for each user
    * Zurück auf `User` und dem vagrant-user die `EC2FullAccess Policy` erteilen
3. Network and Security:
    * In der EC2 Konsole (wechseln mittels Quadrat links oben, EC2 Anwahl) eine neue Security Group einrichten und unter `Inbound` mindestens die Ports 22 und 80 für `Anywhere` freigeben
    * Wechseln auf `Key Pair` und ein neues Key Paar anlegen. Der Private Key ist **sicher** lokal abzulegen!

**AWS Images** <br>
Nun kann das gewünschte AWS Image unter `Images - AMIs` gesucht werden, um anschliessend die AWI ID (z.B. ami-26c43149) zu notieren.

Einfacher geht es auch mit `Instance - Launch Instance`.

**Konfiguration** <br>
Für die Konfiguration von Vagrant müssen folgende zwei Dateien in einem neuen Verzeichnis angelegt werden. Zusätzlich ist der Private Key (Key Pair) in dieses Verzeichnis zu kopieren.

Die Einträge access_key, secret_key, ec2_keypair und security_group müssen entsprechend angepasst werden.

config.rb
```Ruby
    $aws_options = {}
    # Access und Secret Key vom User vagrant-user
    $aws_options[:access_key] = ""
    $aws_options[:secret_key] = ""
    # Der Name des erstellten Key Pairs
    $aws_options[:ec2_keypair] = "aws-frankfurt-linux.pem"
    # Region Frankfurt 
    $aws_options[:region] = "eu-central-1"
    # Ubuntu 14.04 Images 
    $aws_options[:ami_id] = "ami-26c43149"
    $aws_options[:instance_type] = "t2.micro"
    # Der Name der erstellten Security Group
    $aws_options[:security_group] = "IoTKit Server"
```

Vagrant File
```Ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!

    VAGRANTFILE_API_VERSION = "2"
    CONFIG = "#{File.dirname(__FILE__)}/config.rb"
    if File.exist?(CONFIG)
    require CONFIG
    end
    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.define "web" do |web|
        web.vm.box = "dummy"
        web.vm.provider "aws" do |aws, override|
        override.ssh.username = "ubuntu"
        override.ssh.private_key_path = "#{File.dirname(__FILE__)}/aws-frankfurt-linux.pem"
        aws.access_key_id = $aws_options[:access_key]
        aws.secret_access_key = $aws_options[:secret_key]
        aws.keypair_name = $aws_options[:ec2_keypair]
        aws.region = $aws_options[:region]
        aws.ami = $aws_options[:ami_id]
        aws.instance_type = $aws_options[:instance_type]
        aws.security_groups = $aws_options[:security_group]
        aws.tags = {
            'Name' => 'Vagrant Web Server',
            }
        end
    end
        config.vm.provision "shell", inline: <<-SHELL 
        sudo apt-get update
        sudo apt-get -y install apache2
        SHELL
    end
```

**Image erstellen** <br>
Nachdem die Dateien config.rb und Vagrantfile erstellt wurden kann im erstellten Verzeichnis mittels folgendem Befehl die VM in der Cloud erzeugt werden:
```Shell
    $ vagrant up web --provider=aws
```

![](../images/Reflexion_36x36.png) 06 - Reflexion
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)

Unter **[Cloud Computing](https://de.wikipedia.org/wiki/Cloud_Computing)** (deutsch Rechnerwolke) versteht man die Ausführung von Programmen, die nicht auf dem lokalen Rechner installiert sind, sondern auf einem anderen Rechner, der aus der Ferne aufgerufen wird (bspw. über das Internet).

Eine **dynamische Infrastruktur-Plattform** ist ein System, das Rechen-Ressourcen bereitstellt (Virtualisiert),
insbesondere Server (compute), Speicher (storage) und Netzwerke (networks), und diese
Programmgesteuert zuweist und verwaltet, sogenannte **Virtuelle Maschinen** (VM).

Damit "Infrastructure as Code" auf "Dynamic Infrastructure Platforms" genutzt werden können, müssen sie die folgenden Anforderungen erfüllen:

- **Programmierbar** - Ein Userinterface ist zwar angenehm und viele Cloud Anbieter haben eines, aber für "Infrastructure as Code"
muss die Plattform via Programmierschnittstelle ([API](https://de.wikipedia.org/wiki/Programmierschnittstelle)) ansprechbar sein.
- **On-demand** - Ressourcen (Server, Speicher, Netzwerke) schnell erstellen und vernichtet.
- **Self-Service** - Ressourcen anpassen und auf eigene Bedürfnisse zuschneiden.
- **Portabel** - Anbieter von Ressourcen müssen austauschbar sein.
- Sicherheit, Zertifizierungen (z.B. [ISO 27001](https://de.wikipedia.org/wiki/ISO/IEC_27001)), ...


![](../images/Magnifier_36x36.png "Quellenverzeichnis") 08 - Quellenverzeichnis
====== 

> [⇧ **Nach oben**](#inhaltsverzeichnis)

![](../images/Samples_36x36.png "Vagrant") 09 - Beispiele (für LB2)
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)

1.  Terminal (*Bash*) öffnen
2.  In das entsprechende Verzeichnis z.B. `cd M300/vagrant/web` wechseln. `README.md` studieren, z.B. mittels `less README.md`.

* [web - Einfacher Webserver](../vagrant/web/)
* [db - MySQL mit UserInterface](../vagrant/db/)
* [script - Shellscript erstellt mehrere Web Server VM's](../vagrant/script/)
* [mmdb - Multi Machine, Erstellung von mehreren VM's mittels Vagrantfile](../vagrant/mmdb/)
* [lam - Linux, Apache, MySQL, REST Umgebung](../vagrant/lam/)
* [iot - Umfangreicheres Beispiel mit Desktop Umgebung](../vagrant/iot/)
* [cloud-init - Vagrant mit Cloud-init](../vagrant/cloud-init/)
* [MAAS Umgebung in einer VM](../vagrant/maas/)

