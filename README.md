# Semantyki żonglerki

Dokumentacja kodu użytego podczas realizacji projektu "Semantyki żonglerki. Poszukiwanie znaczeń, kontekstów i formy". Niniejsze repozytorium zawiera wykorzystany projekt piłki do żonglowania wyposażonej w moduł IMU, który przesyła dane inercyjne przez IP/UDP do komputera, gdzie są one wizualizowane i przekształcane w dźwięk. 

### Pliki 3D
Pliki 3D w formacie STL, przeznaczone do druku, znajdują się w folderze `3dfiles`. Obejmują one model piłki oraz obudowę na elektronikę. Pliki zostały wygenerowane przy użyciu CadQuery.

### Hardware
- Płytka: ESP32-C6 DevKitC-1
- IMU: BMI160, adres I2C: `0x69`
- Połączenia: SCL → GPIO20, SDA → GPIO19  
  
### Odbieranie danych na komputerze
Upewnij się, że komputer odbierający dane jest dostępny pod adresem ustawionym w `MASTER_HOST` i na porcie `MASTER_PORT`.
Aby podejrzeć surowe dane z czujników oraz wynik działania filtra orientacji, uruchom:
```bash
python read.py
```

Przykładowe dane wyjściowe:

```bash
14873,584,2,3996,-0.98681640625,-0.1142578125,0.056640625,0.244140625,0.0,0.1220703125,0,0,1767274563.513852
14873,584,2,3996,-0.986328125,-0.11376953125,0.05810546875,0.244140625,-0.06103515625,0.06103515625,1,0,1767274563.5138826
14873,584,2,3996,-0.98583984375,-0.11376953125,0.05908203125,0.244140625,-0.06103515625,0.06103515625,2,0,1767274563.5138955
...
```

Każdy wiersz wypisywany w konsoli to dane IMU w formacie CSV, z następującymi kolumnami:

```bash
timestamp_ms,device_sequence_number,sensor_id,battery_mv,accel_x,accel_y,accel_z,gyro_x,gyro_y,gyro_z,sequence_number,checksum_good,host_timestamp
```

Dane z akcelerometru są podawane w jednostkach g, a dane z żyroskopu w deg/s, czyli stopniach na sekundę.

Aby uruchomić prostą wizualizację danych, użyj:

```bash
python read.py | python plot_raw_vis.py
```

### Development

Projekt jest rozwijany przy użyciu PlatformIO na platformie Espressif. Był testowany na Ubuntu i MacOS, ale nie był jeszcze testowany na Windowsie.

### Konfiguracja danych dostępowych i komputera odbierającego dane

Plik src/wifi_secret.h jest ignorowany przez Gita. Przed zbudowaniem projektu utwórz go lokalnie:

```bash
#define WIFI_SSID "your-ssid"
#define WIFI_PASS "your-password"
// Komputer odbierający pakiety UDP z danymi IMU
#define MASTER_HOST "host-or-ip"
#define MASTER_PORT 50555
```

Uwaga: src/main.cpp oczekuje wartości MASTER_HOST oraz MASTER_PORT. Domyślne wartości fallback definiują MASTER_*, ale te makra nie są obecnie używane w kodzie, dlatego ustaw właściwe wartości w swoim pliku wifi_secret.h.

## Budowa z PlatformIO
Repozytorium zawiera plik `platformio.ini`:

```bash
pio run -e esp32-c6-devkitc-1
pio run -e esp32-c6-devkitc-1 --target upload --upload-port /dev/ttyACM0
pio device monitor -e esp32-c6-devkitc-1
```
