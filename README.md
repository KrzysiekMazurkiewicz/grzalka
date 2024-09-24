## Opis funkcji sterującej grzałką na podstawie danych z fotowoltaiki, baterii i zużycia energii przez dom

Funkcja ta steruje włączaniem i wyłączaniem grzałki na podstawie temperatury bojlera, wolnej mocy dostępnej z fotowoltaiki oraz stanu naładowania baterii. Grzałka zostaje włączona lub wyłączona w zależności od określonych warunków, takich jak ilość dostępnej wolnej mocy, temperatura bojlera i stan baterii. 

### Algorytm działania:
1. **Obliczanie mocy zużywanej przez dom**:
   Moc zużywana przez dom jest sumą trzech składowych:
   - Moc z fotowoltaiki (`inverter_input_power`).
   - Moc pobierana lub oddawana z baterii (`batteries_charge_discharge_power`).
   - Moc eksportowana do sieci (`total_active_power`).
   
   Zużycie domu jest wyliczane w następujący sposób:
   ```javascript
    powerUsedByHouse = inverterPower + (-batteryChargeDischargePower) + (-totalActivePower);

	2.	Obliczanie wolnej mocy:
Wolna moc to różnica między mocą generowaną przez fotowoltaikę (inverter_input_power) a rzeczywistą mocą zużywaną przez dom, przy uwzględnieniu aktualnego stanu grzałki.
	•	Jeśli grzałka jest wyłączona, wolna moc jest obliczana jako:
     _ freePower = inverterPower - powerUsedByHouse;_
  •	Jeśli grzałka jest włączona, do wolnej mocy dodawana jest moc zużywana przez grzałkę (3000 W), aby uwzględnić jej wpływ na zużycie energii:
      _freePower = inverterPower - powerUsedByHouse + heaterPower;_
  Dzięki temu uzyskujemy realną wartość wolnej mocy, niezależnie od stanu grzałki.

	3.	Warunki włączenia grzałki:
Grzałka włącza się, jeśli spełnione są trzy warunki:
	•	Temperatura bojlera jest poniżej 80°C.
	•	Wolna moc jest większa lub równa -500 W (można to skonfigurować).
	•	Pojemność baterii wynosi co najmniej 80% (konfigurowalna wartość).
	4.	Czas między zmianami stanu grzałki (histereza):
Aby uniknąć częstego włączania i wyłączania grzałki, ustawiamy minimalny czas między zmianami stanu (domyślnie 1 minuta). Dzięki temu grzałka nie będzie często przełączać się, jeśli warunki są zmienne.
Kod uwzględniający histerezę wygląda następująco:
 _   if (now - lastToggleTime >= toggleInterval) { ... }_
5.	Warunki wyłączenia grzałki:
Grzałka wyłącza się, jeśli którykolwiek z powyższych warunków nie jest spełniony (np. spada temperatura, brakuje wolnej mocy lub bateria nie jest wystarczająco naładowana).

Zmienne:

	•	tempThreshold: Próg temperatury bojlera (domyślnie 80°C).
	•	minFreePower: Minimalna wymagana wolna moc do włączenia grzałki (domyślnie -500 W).
	•	minBatteryCapacity: Minimalna wymagana pojemność baterii do włączenia grzałki (domyślnie 80%).
	•	toggleInterval: Minimalny czas między zmianami stanu grzałki (domyślnie 1 minuta).

 



 

### Instrukcja instalacji i użycia pliku

#### Instalacja

1. **Zainstaluj Node-RED**:
   Jeśli nie masz zainstalowanego Node-RED, zainstaluj go zgodnie z oficjalną dokumentacją: [Node-RED Documentation](https://nodered.org/docs/getting-started/local)

2. **Integracja z Home Assistant**:
   Jeśli korzystasz z Home Assistant, upewnij się, że masz zainstalowaną i skonfigurowaną integrację z Node-RED:
   - Zainstaluj dodatek Node-RED w Home Assistant za pomocą Supervisor > Add-on Store.
   - Skonfiguruj dostęp do API Home Assistant, aby Node-RED mógł odbierać i wysyłać dane.

3. **Dodanie funkcji do Node-RED**:
   - Skopiuj kod funkcji sterującej grzałką do swojego edytora w Node-RED.
   - Upewnij się, że poprawnie zdefiniowane są encje dla czujników (inverter_input_power, batteries_charge_discharge_power, total_active_power, grzałka itp.) w Home Assistant.
   - Wstaw funkcję do odpowiedniego miejsca w flow Node-RED.

#### Użycie

1. **Konfiguracja parametrów**:
   Możesz dostosować parametry działania funkcji:
   - **`tempThreshold`**: Temperatura bojlera, poniżej której grzałka może być włączona (domyślnie 80°C).
   - **`minFreePower`**: Minimalna wymagana wolna moc, aby włączyć grzałkę (domyślnie -500 W).
   - **`minBatteryCapacity`**: Minimalna pojemność baterii, aby włączyć grzałkę (domyślnie 80%).
   - **`toggleInterval`**: Minimalny czas między włączeniem a wyłączeniem grzałki (domyślnie 1 minuta).

2. **Dodawanie encji**:
   Upewnij się, że poprawnie definiujesz encje czujników z Home Assistant, takie jak:
   - `sensor.inverter_input_power`: Moc generowana przez panele fotowoltaiczne.
   - `sensor.batteries_charge_discharge_power`: Moc pobierana lub oddawana przez baterię.
   - `sensor.total_active_power`: Moc eksportowana do sieci.
   - `switch.cwu_grzalka_e_switch`: Przełącznik sterujący grzałką.

3. **Monitorowanie**:
   W Node-RED możesz monitorować działanie funkcji poprzez narzędzie debugowania, gdzie będą wyświetlane logi dotyczące stanu systemu i decyzji o włączeniu/wyłączeniu grzałki.

#### Przykład użycia:

1. W momencie, gdy panele fotowoltaiczne generują wystarczająco dużo energii, a warunki są spełnione (temperatura bojlera poniżej progu, odpowiednia wolna moc, bateria naładowana), funkcja włączy grzałkę.
2. Gdy warunki przestaną być spełnione (np. spadnie wolna moc), grzałka zostanie wyłączona.

Jeśli masz pytania dotyczące instalacji lub konfiguracji, sprawdź dokumentację Node-RED i Home Assistant, lub skontaktuj się ze mną na GitHubie.
