Documentatie Praktijk Toets Ianthe

LINUX
Changing hostname and IP
For IP:
Remove as much unnecessary tools as possible and add: IPADDR=, NETMASK=,GATEWAY=,DNS=.
nano /etc/sysconfig/network-scripts/if network adapter
sudo reboot

For name:
nano /etc/hostname
remove old name, add new name
nano /etc/hosts
change all to new name
sudo reboot

DHCP
1.	 sudo dnf install dhcp-server bind httpd samba installeert de DHCP, DNS, WEB en linux SMB
2.	⁠cp dhcpd.conf naar dhcpd.conf.old⁠ (voor backup)
3.	 met nano of vim  dhcpd.conf aanpassen met eigen gegevens


 
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;


# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;
 option broadcast-address 10.254.239.31;

# A slightly different configuration for an internal subnet.
subnet 10.1.32.0 netmask 255.255.255.0 {
  range 10.1.32.50 10.1.32.80;
  option domain-name-servers 10.1.32.1;
  option domain-name "toets.nu";
  option routers 10.1.32.1;
  option broadcast-address 10.1.32.255;
  default-lease-time 600;
  max-lease-time 7200;
}

4.	 
firewall-cmd --list-all
firewall-cmd --add-service=dhcp --permanent
firewall-cmd --reload 
5.	systemctl

Mocht de DHCP moeten worden gebruikt op een client:
-	Config file BOOTPROTO=dhcp. TYPE=Ethernet. ONBOOT=yes.
-	Restart het netwerk.
 
DNS
cp named.empty domain.test
kopieer de name.conf naar document voor backup

chown root.named domain.test
maak de named groep eigenaar van domain.local file

nano domail.test
pas de SOA aan /var/named
(haal een kopie uit /var/named/named.empty)
verander de serial naar de datum (mag niet te lang zijn)
NS is voor nslookup naar computernaam in domein
IN A IP van server
srv4 IN A IP een IP in IP range
ftp & www IN CNAME computer voorvoegsel naam
CNAME is canonical name. daarmee maak je een verwijzings naam

nano (of vim) /etc/named.conf
verander de listen-on port naar any
en de allow-query naar any
zone aanmaken, let op ‘;’

zone "bmc.test" IN {
	type master;
	file "bmc.test";
};

systemctl enable named
systemctl start named

firewall-cmd --add-service=dns --permanent
hiermee zet je het open voor dns verkeer

Netwerk configuratie maken dat de linux servers zelf gebruik kunnen maken van de DNS server.
r /etc/resolv.conf
nano /etc/resolv.conf⁠ IP van DNS invullen.
chattr +i /etc/resolv.conf
reboot now

Troubleshooting:
In systemctl status named: network unreachable resolving.
Zet OPTION=”-4” in /etc/sysconfig/named
(dit configureerd de IPv6)
service named restart

 
Troubleshooting van Tijn:
Named-checkconf
named-checkzone Tinydevs.test /var/named/Tinydevs.test
Reverse zone
Om te kopieren naar nieuwe /var/named file:
$TTL 1D
@        IN        SOA  localhost. root.localhost. (
                        2007091701 ; serial
                        30800      ; refresh
                        7200       ; retry
                        604800     ; expire
                        300 )      ; minimum
         IN        NS    localhost.
1        IN        PTR   localhost.
Verander aan de hand van eigen IP en PTR naar domein naam. 1 moet in IP de 2 zijn als het IP 192.168.2.3 is.
Zone “0.0.10.in-addr.arpa” IN {
Type master;
File “reverse.test”;
Allow-update { none; };
};

Setting up partitions
1.	Add disk in VMware (sata is most logical)
2.	Fdisk /dev/sdb waar sdb de schijf is
3.	pvcreate /dev/sdx creates a physical volume
4.	vgcreate ‘name’ /dev/sdx
5.	lvcreate -L (volume in GB) -n ‘name lv’ ‘name vg’
6.	mkfs -t xfs /dev/namevg/namelg
7.	mount -t xfs /dev/namevg/namelv /data1
Will need to make a directory first to mount this to.
8.	add new name lv in /etc/fstab
df -h
fdisk -l
lsblk
lvextent om logisch volume groter te maken.
Xfs_growfs /dev/… 
Useful commands and locations:

cat -b fileNameHere
useradd -c fullname username -m 
To activate a user account do: passwd username
Chage -l Hiermee krijg je alle opties van wachtwoorden

Symbolic link:
Ln -s ~/bin/test.sh top.sh
Source  Naam

 
WINDOWS
DHCP
Met Powershell
1.	Install-WindowsFeature -Name ‘DHCP’ -IncludeManagementTools
2.	Add-DhcpServerV4Scope -Name "DHCP Scope" -StartRange 192.168.1.150 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0
3.	Set-DhcpServerV4OptionValue -DnsServer 192.168.1.10 -Router 192.168.1.1⁠
4.	Set-DhcpServerv4Scope -ScopeId 192.168.1.10 -LeaseDuration 1.00:00:00
5.	Restart-service dhcpserver
Voor core kan je ook de core toevoegen aan servermanager op een GUI. Als het DC’s zijn van hetzelfde domein. 
Note voor DHCP install met GUI
Let op! Als je de DHCP server op een core server installeerd, maar NIET op de Windows Server waar jij op zit, moet je DHCP Server Tools installeren. 
Dit doe je als volgt: 
-	Add roles and features
-	Role-based or feature-based installation
-	Selecteer de server waar je op zit
-	Selecteer GEEN server roles, klik op Next 
-	Tab features, drop-down menu ‘Remote Administration Tools’, drop-down menu ‘Role Administration Tools’, selecteer ‘DHCP Server Tools’ en installeer deze.
Scope aanmaken
-	Tools » DHCP
-	Selecteer de server
-	Rechtsklikop ‘IPv4’, selecteer ‘New Scope’
-	Geef de range op die gebruikt gaan worden en klik op Next >
-	Geef eventueel een excluded range op, bijvoorbeeld voor management servers
-	Volg de wizard 
WEBSERVER
Installeren op Core met Powershell
Enable-NetFirewallRule -DisplayGroup "Windows Remote Management"
Om remote management toe te laten. Als je het via de management tool doet die je kan downloaden via de servermanager. 

Install-WindowsFeature Web-Server -IncludeManagementTools
Uninstall-WindowsFeature Web-Dir-Browsing
New-WebSite -Name ServerCore -Port 80 -HostHeader www.servercore.net -PhysicalPath "C:\Inetpub/wwwroot/wordpress"
New-WebBinding -Name "ServerCore" -IPAddress "*" -Port 80 -HostHeader servercore.net
Installeren in GUI
Add roles and features
-	Selecteer ‘Web Server (IIS)’
-	Klik op 3x Next >
-	Tab ‘Select role services’, 
o	Drop-down menu ‘Security’, selecteer: ‘Basic Authentication’ && ‘Digest Authentication’ && ‘IP and Domain Restrictions’ && ‘Windows Authentication’
o	Drop-down menu ‘Management Tools’,selecteer: ‘Management Service’
o	Installeer nu IIS.
-	Check de werking door naar http://%SERVER-NAAM%te gaan binnen je browser. Bijvoorbeeld: http://rftech-win2016-gui/
IP restrictie:
-	Tools » Internet Information Services (IIS) Manager
-	Selecteer de serverKlik op ‘IP Address and Domain Restrictions’
-	Configureer hier je restricties naar wens
Authenticatie:
-	Tools » Internet Information Services (IIS) Manager
-	Selecteer de server
-	Klik op ‘Authentication’
-	Configureer hier welke authenticatie je wilt opleggen.
-	Als je een realm wilt instellen: Rechtsklik op de vorm van authenticatie » Vul je tekst in onder ‘Realm:’
Willekeurige kennis:
Gebruikers importeren vanuit csv bestand:
-	Csvde -i -k -v -F C:\Users\Ianthe Westerik\Desktop\testfile.csv
-	Import-csv .\test.csv | New-ADUser powershell
Dsadd, dsmod, dsquery, dsmove
https://learn.microsoft.com/en-gb/archive/blogs/jhoward/sample-scripts-for-dsadd-dsmodify-dsget-dsquery-dsmod-dsmove
/? Voor toelichting in command prompt

OU met powershell:  
New-ADOrganizationalUnit Name
-	Name “Name
-	Path “OU=Sales,OU=Administration,DC=DOMAIN,DC=local”
Bij de laatste twee commands bepaal je waar hij komt te staan. Als je hem normaal wil doe je het zonder de twee toevoegingen.

Homefolder/profilepath instellen met wildcard
Properties van een gebruiker. De wildcard stel je in door het pad aan te geven als volgt:
\\Path\To\Share\%username% Laat %username% zo staan! Dit is je wildcard. Denk ook aan de share rechten!

Delegation of control instellen
http://www.itingredients.com/how-to-delegate-control-in-active-directory-users-and-computers/ 
-	Maak eerst een groep aan binnen een OU (rechtsklik » New... » Group)
-	Geef de groep een naam, group scope = global en group type = security.
-	Klik op de properties van de groep » tab ‘Members’ en voeg de benodigde gebruikers toe
-	Klik met rechts op de desbetreffende OU en selecteer ‘Delegate Control...’
-	Klik op next » add... » selecteer de groep die zojuist is gemaakt
-	Selecteer de benodigde attributen && Finish.

 
Veel voorkomende group policies
Naar settings, en dan vind je de computer config of userconfig ect. Als je naar beneden scrolt. Rechter muisknop en edit.

-	Laatst ingelogde gebruiker verbergen: Computer configuration » Policies » Windows Settings » Security Settings » Local Policies » Security Option >>  Interactive logon: do not display last user name
-	Group policy loopback processing (mode replace): Computer Configuration » Policies » Administrative Templates » System » Group Policy >> Configure user Group Policy loopback processing mode
-	Alle desktop iconen verbergen: User Configuration » Policies » Administrative Templates » Desktop >> Hide and disable all items on the desktop
-	Control panel toegang revoken: User configuration » Policies » Administrative Templates » Control Panel >> Prohibit access to Control Panel and PC settings
-	Run command revoken: User Configuration » Policies » Administrative Templates » Start Menu and Taskbar >> Remove Run menu from Start Menu
-	Folder redirection: Maak eerst een share aan en zorg bijvoorbeeld dat een groep de correcte rechten heeft.
o	User Configuration » Policies »Windows Settings » Folder redirection » Documents » Properties >> Basic –Redirect everyone’s folder to the same location >> Verwijs naar de share (bijvoorbeeld \\BMC-DC1\ShareX)
-	Software packages deployen: Computer Configuration » Policies » Software Settings » Software installation >> Rechtsklik >> New package
-	Alle GPO’s back-uppen (PowerShell): Backup-GPO –All –Path %insert_path%
-	Enkele GPO back-uppen(PowerShell): Backup-GPO –Name <string> -Path %insert_path%
-	GPO restoren(PowerShell): Restore-GPO –All –Path %insert_path%



 
Core server configuration
Dit kan ook in AD Server Manager
Set-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\WebManagement\Server-Name
EnableRemoteManagement -Value 1 Allow remote management.
Rename-Computer –newname %NAAM%
Bijvoorbeeld: Rename-Computer –newname BIPL-CORE-AD01
Stel een statisch IP in via sconfig
Restart-computer 
Get-Timezone
Set-TimeZone –id “Greenwich Mean Time”
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Get-Service adws,kdc,netlogon,dns
Dit kan ook in AD Server Manager
Choose from:
Add-ADDSReadOnlyDomainControllerAccount Read only
Install-ADDSDomain Child of tree domain
Install-ADDSDomainController bestaand domein
Install-ADDSForest Nieuw domein

Peer domain controller OF domain controller voor subdomein toevoegen
Stel het IP adres in binnen de range van die van de primary domain controller, met de primary domain controller als gateway. Verander ook de naam van de server naar iets herkenbaars, én maak de server lid van het bestaande domein. Log in bij de server met een administrator account van het domein.
 
https://www.integraxor.com/add-client-to-active-directory-domain-part-2/ 
