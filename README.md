@ -0,0 +1,79 @@
Program "programator_obsz_SST39F" pozwala obsługiwać pamięć FLASH typu SST39SFxxx /max 512 kB/, włożoną do płytki
której piny 2-28 podstawki są połaczone z pinami podstawki U10 CA80.  
Możemy użyć FLASH o mniejszej pojemności ale wówczas musimy zmienić sprawdzanie końca pamięci FLASH, 
etykieta "SEKT_END" /dla 512 kB to 7F, dla 256-3F a 128 - 1F Możemy pamięć zapisywać, odczytywać i 
przeglądać jej zawartość. Pomocny jest tutaj wyświetlacz LCD 2004, /20. znaków, 4. wiersze/ podłączony
do CA80 wg projektu Kol. #ZEGAR na "microgeek.eu", podłączony bezpośrednio do linii danych CA80 (nie przez port 8255), z 
wykorzystaniem tylko układu 74LS02. Wadą (programu) użycia LCD jest to, że program zajmuje dużo więcej 
bajtów niż program bez obsługi LCD. Bez LCD program też będzie działał, tylko nie wszystkie komunikaty będą 
czytelne i niektóre polecenia nie będą widoczne na CA. Również nazwy programów też nie będą widoczne.
 
https://microgeek.eu/viewtopic.php?f=82&t=2513  - podłączenie LCD
https://microgeek.eu/viewtopic.php?f=82&t=2435 - pamięć masowa FLASH


 
 Na LCD możemy wyświetlić więcej czytelnych znaków niż na wyświetlaczu CA80. Jeśli mamy nową kość 
SST39FLxxx, to aby zapisać nasz pierwszy program, musimy na początku FLASH wpisać FD E4 - 
możemy to zrobić zlec. *8 . Program jest "samowystarczalny" 
jeśli idzie o obsługę LCD, gdyż ma potrzebne podprogramy do obsługi LCD.
Format plików z programami w pamięci FLASH, objaśniony jest w pliku *.ASM

Po uruchomieniu programu, na LCD mamy "MENU" a na CA80 napis "FLA_2".
 KLAWISZ F1  TO POWRÓT NA POCZĄTEK PROGRAMU !, podobnie jak "G" w CA88

Zlecenia w MENU:

 0- kasowanie /uwaga !!!/ pamięci FLASH - CAŁEJ /klawisz C/ lub jednego, wybranego sektora /klaw. 2/, 
    wybór w menu na LCD
 1- szukanie numeru, który jest "pierwszy wolny" podczas zapisywania programu- nadanie numeru programu,
    numer zostanie wyświetlony na LCD i CA80
 2- przegląd pamięci /bez możliwości modyfikacji, bo to FLASH/ po podaniu adresu początkowego - max 
    7FFFF, do przodu "=" lub do tyłu "." Adres max 5. cyfrowy, do 7FFFF /dla 512 kB/; adres wyświetlany
    na CA80 i LCD
 4- zapis programu /*4E/ lub obszaru /*47/  do FLASH
    Podczas zapisu programu z CA <CA_OD> <CA_DO> musimy podać [NR] programu, 1-FF. Program 
    sprawdzi, czy wpisany nr jest wolny czy zajęty;  wyświetli się odpowiedni komunikat na CA 
    /na LCD tylko, gdy numer dozwolony jest wolny, na dole LCD, mrugający kursor. Program szuka w pamięci 
    FLASH trzech bajtów: FD E4 FF, oznaczających koniec zapisanych programów w pamięci FLASH. 
    Nasz program będzie wpisany za ostatnim FD E4 - [NR_prog] [CA_OD] i "treść" programu.Potem, po 30 bajtach
    od końca zapisanego programu wpisze FD E4 jako koniec programów. Ten "zapas" 30. bajtów FF na ewentualne 
    poprawki w programie; musimy wówczas CAŁY sektor pzepisać do RAM, zrobić poprawkę /max 30 bajtów!/,
    skasować dany sektor i ponownie cały sektor z RAM wpisać do FLASH. Trochę to skomplikowane ale wykonalne.
    Możemy w ten sposób zapisać też obszar z CA80 do FLASH,
    musimy jednak pamiętać, że podczas odczytu tego programu, będzie on wpisany do CA80 tam skąd został
    zapisany. Jeśli chcemy zapisać obszar ROM z naszego CA80, lepiej przenieść go zlec. *B /z CA80/ do 
    RAM i wówczas zapisać.Po przeniesieniu do RAM, możemy nadać mu nazwę poprzedzoną DDE2 - opis w pliku ASM.
    Jeśli nazwy brak, pojawią się znaki ???.
    po zapisie obszaru z CA do FLASH, pojawi się napis na CA80 "write FD ?" - jeśli wcisniemy "=", to po 30 bajtach
    zostaną  dopisane bajty FD E4   
 5- szukanie programów, program wyszukuje NR_PROG, wyświetla go na CA i LCD, nazwę /nazwa po DD E2/
    na LCD, jeśli nazwy nie ma, na LCD wyświetlają się "? ? ?" i następuje przeskok na wiersz następny na LCD.
    Podczas szukania, na CA pojawia się adres początku /w pam. FLASH/ wyświetlanego programu. Po 
    wyświetleniu 4. linii, program czeka na wciśniecie klawisza: jeśli "=" szuka dalej, jeśli wcisniemy
    0 - do F, pojawia się  napis na LCD i CA z prośbą o podanie numeru programu, który chcemy przepisać do CA.
    Nastąpi  wyszukanie podanego programu w pamięci FLASH, przepisanie do CA i uruchomienie.
 6- odczyt obszaru z pamięci FLASH do CA80; ponieważ obszar 4000h-7FFFh w CA jest "zajęty" przez płytkę
    FLASH, możemy przepisać bajty z FLASH do RAM CA80 od 8000h do DFFFh /28 KB/. Pamiętać należy, że 
    program obsługi tejże pamięci jest od E000h, w moim CA80 - SK.
 7- pozycja programu - wyszukiwanie obszaru, gdzie znajduje się dany program; na CA wyświetlany jest 
    adres uruchomienia w CA i jego długość /oddzielone kropką/ a na LCD położenie w pamięci FLASH 
    i w CA80. Można te informacje z LCD /"FLASH od-do" wykorzystać do przepisania obszaru /programu/ 
    z FLASH do CA80. Długość programu liczona jest od początku szukanego programu w pamięci FLASH 
    /po FD E4 [NR_PR] [CA_OD] / do znalezienia trzech bajtów FF.
 8- wciskając kl. 8 inicjujemy pamięć FLASH, tzn. wpisujemy bajty FD E4 od adr. 0 i potem już możemy wpisywać
    programy.

<img width="1262" height="596" alt="programator_FL" src="https://github.com/user-attachments/assets/5c9fd9b6-1f43-48ca-8ad5-0a354cda2925" />

 3 -  to zapis AT28C256 _EEPROM /32 KB/, adres początku zapisu jest dowolny, nie musimy startować od
     adr. 0; pamiętać należy tylko o tym, że adres końcowy zapisu nie może przekraczać 7FFFh !!
     to zlecenie wyświetla na LCD tylko "USTAW ZWORKI J9 J10 J11" - na płytce, 
     na wyświetlaczu CA80: [CAod], [CAdo], [FLod]
     Jeśli chcielibyśmy zapisa pamięć AT28C64B /16 KB/, należy wówczas w pliku ASM zmienić adresy
     odblokowania i zablokowania pamięci!! wg noty katalogowej
     
;=====
