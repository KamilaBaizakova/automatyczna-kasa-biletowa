   Tytuł modelu
Model automatycznej kasy biletowej

   Dane studenta
* Imię i nazwisko:Kamila Baizakova
* E-mail: baizakova@student.agh.edu.pl

     Opis modelowanego systemu
Model przedstawia architekturę systemu automatycznej kasy biletowej zaprojektowanej w języku AADL (Architecture Analysis & Design Language). System umożliwia zakup biletów przez pasażera bez pomocy kasjera. Składa się z interfejsu użytkownika (ekran dotykowy, panel płatności), kontrolera przetwarzającego dane, modułu płatności, drukarki biletów oraz komponentów sprzętowych (procesor, pamięć, magistrala danych).

     Opis ogólny
System został podzielony na trzy główne jednostki funkcjonalne:
* UserInterfaceUnit – odpowiada za interakcję z użytkownikiem (wybór trasy, daty, liczby biletów), 
* Controller – przetwarza dane wejściowe, komunikuje się z systemem płatności i podejmuje decyzje (np. wydruk biletu), 
* OutputUnit – drukuje bilety oraz potwierdzenia. 
Wszystkie komponenty komunikują się za pośrednictwem magistrali systemowej MainBus. Procesy realizowane są przez dedykowane wątki pracujące na procesorze, a dane są przechowywane w pamięci RAM.

     Opis dla użytkownika
Użytkownik korzysta z ekranu dotykowego w celu:
1. Wybrania miejsca docelowego i daty podróży, 
2. Określenia liczby pasażerów i klasy biletu, 
3. Uiszczenia opłaty (gotówką lub kartą), 
4. Odebrania biletu i/lub potwierdzenia transakcji. 
System automatycznie prowadzi użytkownika przez wszystkie etapy, oferując intuicyjny interfejs i obsługę w wielu językach.

   Spis komponentów AADL z komentarzem
