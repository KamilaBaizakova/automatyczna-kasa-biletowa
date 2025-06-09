---
title: "**Automatyczna kasa biletowa -- model architektury w języku
  AADL**"
---

**2. Opis modelowanego systemu**

**2.1. Opis ogólny**

Model architektury AADL przedstawia uproszczoną automatykę stacjonarnej
kasy biletowej, składającą się z 10 podstawowych komponentów:

### 1. `ui_dev` – TouchScreen (device)
- **Porty:**
  - `sel_out : out data port TicketRequest`
  - `disp_in : in data port DisplayMessage`
- **Opis:**  
  Urządzenie wejściowe z ekranem dotykowym. Umożliwia użytkownikowi wybór trasy i rodzaju biletu. Odbiera komunikaty zwrotne (np. o statusie płatności, błędach) i przekazuje je na ekran.

---

### 2. `ui_proc` – UserInputProc (process)
- **Porty:**
  - `sel_in`, `sel_out` – przetwarza wybór użytkownika  
  - `disp_out` – wysyła komunikaty do wyświetlacza  
  - `pay_status_in` – odbiera status płatności  
  - `log_out` – przekazuje logi do rejestratora
- **Opis:**  
  Logika interfejsu użytkownika. Przetwarza wybór biletu i przekazuje go dalej. Odbiera wyniki płatności i drukowania oraz przygotowuje komunikaty dla użytkownika.

---

### 3. `card_dev` – CardReader (device)
- **Porty:**
  - `pay_out : out data port PaymentInfo`
- **Opis:**  
  Czytnik kart płatniczych. Wysyła dane o płatności (np. numer karty, potwierdzenie) do logiki przetwarzającej płatność.

---

### 4. `cash_dev` – CashAcceptor (device)
- **Porty:**
  - `pay_out : out data port PaymentInfo`
- **Opis:**  
  Urządzenie do przyjmowania gotówki. Przesyła informację o włożonych banknotach lub monetach do procesora płatności.

---

### 5. `pay_proc` – PaymentProc (process)
- **Porty:**
  - `pay1_in`, `pay2_in` – odbiera dane z czytnika kart i akceptora gotówki  
  - `ticket_out` – generuje dane biletu  
  - `pay_status_out` – przesyła status płatności  
  - `log_out` – wysyła logi
- **Opis:**  
  Główna logika płatności. Przyjmuje dane płatnicze z dwóch źródeł, weryfikuje płatność i przesyła wynik do UI oraz dane biletu dalej.

---

### 6. `ticket_proc` – TicketProc (process)
- **Porty:**
  - `ticket_in` – odbiera dane biletu  
  - `print_cmd_out` – generuje polecenie drukowania  
  - `prt_status_in` – odbiera status drukowania  
  - `log_out` – przesyła dane do logowania
- **Opis:**  
  Obsługuje proces przygotowania biletu do druku i reaguje na status wydruku. Rejestruje każde zdarzenie.

---

### 7. `prt_dev` – PrinterDevice (device)
- **Porty:**
  - `ticket_in`, `cmd_in`, `status_out`
- **Opis:**  
  Fizyczna drukarka. Odbiera dane biletu i polecenie drukowania, następnie zwraca status wykonania operacji (np. sukces, błąd, brak papieru).

---

### 8. `disp_dev` – DisplayDevice (device)
- **Porty:**
  - `msg_in : in data port DisplayMessage`
- **Opis:**  
  Zewnętrzny wyświetlacz pokazujący komunikaty (np. potwierdzenie płatności, błąd systemu, instrukcje).

---

### 9. `log_proc` – LoggerProc (process)
- **Porty:**
  - `log_in`, `log_out : event data port LogEntry`
- **Opis:**  
  Centralny komponent zbierający logi z UI, płatności i drukarki. Może filtrować, agregować lub po prostu przekazywać dalej.

---

### 10. `net_dev` – NetworkInterface (device)
- **Porty:**
  - `net_in : in event data port LogEntry`
- **Opis:**  
  Interfejs sieciowy do zdalnego przesyłania logów (np. do serwera administracyjnego). Finalny punkt w łańcuchu rejestrowania zdarzeń.


Komponenty są połączone portami zdarzeń i danych w topologii „gwiazda":
ui → tp →pay→ tp→ prt → tp. Model uwzględnia podstawowe przepływy
zdarzeń („wybór biletu", „polecenie płatności", „status płatności",
„polecenie druku", „status druku") i stanowi bazę do dalszej analizy
opóźnień (latency) oraz budżetu zasobów (CPU, pamięć, magistrala).

**2.2. Opis działania z perspektywy użytkownika

System automatycznej kasy biletowej działa w sposób intuicyjny i sekwencyjny. Poniżej przedstawiono krok po kroku, co dzieje się od momentu wejścia użytkownika w interakcję z ekranem aż do zakończenia transakcji:

---

### 🔹 1. Wybór biletu
- Użytkownik korzysta z **panelu dotykowego** (`ui_dev`) do wyboru typu biletu i trasy.
- Wybrana opcja zostaje przekazana jako sygnał przez port `sel_out` do **logiki interfejsu** (`ui_proc`), która następnie przesyła to dalej do **modułu płatności** (`pay_proc`).

---

### 🔹 2. Autoryzacja płatności
- **Procesor płatności (`pay_proc`)** odbiera żądanie i czeka na dane z:
  - **czytnika kart (`card_dev`)**
  - lub **akceptora gotówki (`cash_dev`)**
- Odpowiednie informacje trafiają do portów `pay1_in` / `pay2_in`.
- Po sprawdzeniu autoryzacji, `pay_proc`:
  - wysyła sygnał `pay_status_out` do `ui_proc` (czy płatność się powiodła),
  - oraz tworzy dane biletu i przesyła je do `ticket_proc`.

---

### 🔹 3. Drukowanie biletu
- **Procesor biletu (`ticket_proc`)** odbiera dane biletu i generuje polecenie wydruku (`print_cmd_out`) dla **drukarki (`prt_dev`)**.
- Po zakończeniu druku, drukarka odsyła `status_out` do `ticket_proc`, który:
  - wysyła status dalej do `ui_proc`,
  - loguje zdarzenie (np. druk zakończony sukcesem lub błąd).

---

### 🔹 4. Wyświetlanie komunikatów
- **`ui_proc`** na podstawie otrzymanych statusów generuje odpowiednie komunikaty (`disp_out`), które są przesyłane do **wyświetlacza (`disp_dev`)**.
- Użytkownik widzi informację o sukcesie, błędzie lub wymaganym działaniu (np. „brak papieru”).

---

### 🔹 5. Logowanie zdarzeń
- W międzyczasie wszystkie istotne operacje (wybór, płatność, druk, błędy) są logowane.
- Dane są przesyłane przez `log_out` z każdego procesu (`ui_proc`, `pay_proc`, `ticket_proc`) do **loggera (`log_proc`)**, który agreguje je i przekazuje do **interfejsu sieciowego (`net_dev`)**.

---

### 🎯 Efekt końcowy
Użytkownik:
- wybiera bilet → płaci → otrzymuje potwierdzenie i wydrukowany bilet,  
a system:
- rejestruje wszystko w logach,
- wykrywa i obsługuje ewentualne błędy.


  **Obsługa błędów**

    - W przypadku odrzucenia karty lub braku papieru w drukarce,
      procesor generuje odpowiedni komunikat zwrotny przez port msg_in i
      wyświetla go w interfejsie.

Cały cykl trwa od momentu wyboru biletu do wydruku i sygnalizacji
zakończenia operacji.


## 3. Spis komponentów AADL z komentarzem

| **Komponent**             | **Typ**   | **Interfejs (porty)**                                                                                                                                                                                                                                                                                                               | **Opis działania**                                                                                                                                  |
|---------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| **ui_dev : TouchScreen**  | device    | - `sel_out : out data port TicketRequest`  <br> - `disp_in : in data port DisplayMessage`                                                                                                                                                                                                                                          | Ekran dotykowy – umożliwia wybór biletu i odbiera komunikaty do wyświetlenia                                                                       |
| **ui_proc : UserInputProc** | process | - `sel_in : in data port TicketRequest` <br> - `sel_out : out data port TicketRequest` <br> - `disp_out : out data port DisplayMessage` <br> - `pay_status_in : in data port PaymentStatus` <br> - `log_out : out event data port LogEntry`                                                 | Logika UI – przetwarza wybór, reaguje na statusy i generuje komunikaty                                                                             |
| **card_dev : CardReader** | device    | - `pay_out : out data port PaymentInfo`                                                                                                                                                                                                                                                                                           | Czytnik kart płatniczych                                                                                                                            |
| **cash_dev : CashAcceptor** | device  | - `pay_out : out data port PaymentInfo`                                                                                                                                                                                                                                                                                           | Akceptor gotówki                                                                                                                                    |
| **pay_proc : PaymentProc** | process  | - `sel_in : in data port TicketRequest` <br> - `pay1_in/pay2_in : in data port PaymentInfo` <br> - `ticket_out : out data port TicketData` <br> - `pay_status_out : out data port PaymentStatus` <br> - `log_out : out event data port LogEntry`                                          | Obsługuje płatności, generuje bilety, informuje o wyniku i rejestruje operacje                                                                     |
| **ticket_proc : TicketProc** | process | - `ticket_in : in data port TicketData` <br> - `prt_status_in : in data port PaymentStatus` <br> - `print_cmd_out : out data port PrintCommand` <br> - `log_out : out event data port LogEntry`                                                                                           | Obsługuje druk biletu, generuje polecenia i loguje status                                                                                           |
| **prt_dev : PrinterDevice** | device | - `ticket_in : in data port TicketData` <br> - `cmd_in : in data port PrintCommand` <br> - `status_out : out data port PaymentStatus`                                                                                                                                                                                             | Drukarka – wykonuje druk na podstawie danych i zgłasza status                                                                                       |
| **disp_dev : DisplayDevice** | device | - `msg_in : in data port DisplayMessage`                                                                                                                                                                                                                                                                                          | Wyświetlacz komunikatów tekstowych                                                                                                                  |
| **log_proc : LoggerProc** | process   | - `log_in : in event data port LogEntry` <br> - `log_out : out event data port LogEntry`                                                                                                                                                                                                                                          | Agreguje logi z całego systemu i przekazuje je dalej                                                                                                |
| **net_dev : NetworkInterface** | device | - `net_in : in event data port LogEntry`                                                                                                                                                                                                                                                                                          | Interfejs do przesyłania logów do systemu zewnętrznego                                                                                              |
                                      |

4\. Model -- rysunek

![Obraz zawierający tekst, diagram, zrzut ekranu, linia Zawartość
wygenerowana przez sztuczną inteligencję może być
niepoprawna.](IMG_0430.jpeg) podstawowa wersja 

## Rys. 1. Diagram instancji Ticket Machine (rozszerzona)

![Diagram instancji](TicketMachine%202/Untitled.png)

Rys. 2. Raport spójności portów -- 0 niez 1

5.  **Proponowane metody analizy modelu w OSATE i wyniki**

**Check Model (sprawdzanie spójności składniowej)**

1.  **Opis:** Wbudowana w OSATE weryfikacja poprawności AADL (Xtext
    Check, Semantic Checks).

2.  **Wynik:** „No problems found" -- brak błędów składniowych,
    wszystkie referencje do typów i portów rozwiązane poprawnie.

**Flow Consistency Analysis**

3.  **Opis:** Sprawdzenie zgodności definicji przepływów danych
    (FlowSpecification) i ich odwzorowania na rzeczywiste połączenia
    portów między komponentami.

4.  **Wynik:** Brak cyklicznych zależności ani niespójności w
    przepływach; każdy zadeklarowany flow ma odpowiadające mu połączenie
    c1...c5.

**Port Connection Consistency**

1.  **Opis:** Weryfikacja, czy kierunki portów (in/out) są zgodne z
    kierunkiem połączeń oraz czy typy portów są kompatybilne.

2.  **Wynik:** Wszystkie połączenia ui→tp, tp→pay, pay→tp, tp→prt,
    prt→tp są poprawne -- brak ostrzeżeń.

**Latency Analysis (analiza opóźnień)**

1.  **Opis:** Przy użyciu wtyczki Cheddar lub wbudowanych w OSATE
    narzędzi timingowych można przydzielić czasy wykonania
    poszczególnych komponentów i obliczyć maksymalne opóźnienie od
    wejścia do wyjścia.

2.  **Wynik (symulacja wstępna):** Bez realnych wartości czasów
    aktywacji i priorytetów wszystkie komponenty traktowane są jako
    natychmiastowe, więc opóźnienie wynosi 0 ms. Po dodaniu właściwości
    Compute_Execution_Time i harmonogramu RR\--FIFO (np. WiPoc) uzyskamy
    rzeczywiste wartości.

**Resource Budget Analysis (analiza obciążenia zasobów)**

1.  **Opis:** Ocena procentowego zużycia CPU i pamięci przez każdy
    subkomponent (współpraca z narzędziem Cheddar lub OSATE Budget).

2.  **Wynik (domyślne ustawienia bez obciążeń):** 0 % użycia CPU i
    pamięci -- model wymaga nadania właściwości zasobowych (m.in. WCET),
    aby uzyskać realistyczne wyniki.

**Safety / Error Propagation Checks**

1.  **Opis:** (Opcjonalnie) analiza bezpieczeństwa przepływów błędów --
    czy sygnał błędu (np. msg_in z UI) trafia do wszystkich
    zainteresowanych komponentów i jak jest propagowany.

2.  **Wynik:** Przy braku formalnie zdefiniowanych flow dla sygnałów
    awarii nie wykonano tej analizy; można ją dołożyć, definiując
    dodatkowe porty „error" i flow.
