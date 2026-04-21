# Proiect InkTime Smartwatch

**Student:** Ghița Maria  
**Grupa:** 331CC

## 1. Descrierea Conceptului

InkTime este un dispozitiv purtabil bazat pe microcontrolerul **nRF52840**, conceput pentru a maximiza autonomia prin utilizarea unui ecran de tip **E-Paper (EPD)** de 1.54 inch. Spre deosebire de ecranele clasice, tehnologia E-Paper consumă energie doar la împrospătarea imaginii, oferind o vizibilitate excelentă în lumină naturală și un mod de operare "Always-on" extrem de eficient.

## 2. Diagrama Bloc (Hardware Architecture)

```text
                                  SISTEM INTELIGENT INKTIME
      SURSA ENERGIE              ___________________________                PERIFERICE (I/O)
    _________________           |                           |             ____________________
   |                 |          |      nRF52840 (SoC)       |            |                    |
   | USB-C Connector |--- 5V ---|   Bluetooth 5.0 Low En.   |--- SPI --->|  E-PAPER DISPLAY   |
   |   (Incarcare)   |          |___________________________|    Bus     |  (1.54" 24-pin)    |
   |_________________|           |  P0.02 (SCK)             |            |____________________|
            |                    |  P0.03 (MOSI)            |
            v                    |  P0.15 (DC)              |             ____________________
    _________________            |  P0.16 (RST)             |            |                    |
   | BQ25180 Charger |           |  P0.17 (BUSY)            |            |                    |
   | (Mngmnt Baterie)|-- VBAT -- |                          |            |                    |
   |_________________|    |      |  P0.26 (SDA) <---I2C---> |----------->|  ACCEL. (BMA421)   |
            ^             |      |  P0.27 (SCL)    Bus      |----------->|  GAUGE (MAX17048)  |
            |             |      |                          |----------->|  DRV2605 (Haptic)  |
    ________|________     |      |                          |            |____________________|
   | Li-Po Battery   |----+      |                          |
   |    250 mAh      |    |      |  P1.11 (PWM) ------------|----------->[ SHAKER / MOTOR ]
   |_________________|    |      |                          |
            |             |      |  P0.13 (UP)              |             ____________________
            v             |      |  P0.14 (ENTER) <---GPIO--|------------|   3x BUTOANE       |
    _________________     |      |  P1.02 (DOWN)            |            |   NAVIGATIE        |
   | RT6160A (Buck-B)|<---+      |                          |            |____________________|
   |  Output: 3.3V   |-----------|  P0.00 / P0.01 ----------|----------->[ CRYSTAL 32.768kHz]
   |_________________|           |  (LFXO Clock)            |            
                                 |__________________________|
```

## 3. Descriere Detaliată Hardware

### 3.1 Microcontroller — nRF52840

Inima sistemului este microcontrollerul Nordic nRF52840, care gestionează toate perifericele, comunicația wireless și logica aplicației.

- **CPU:** ARM Cortex-M4F la frecvență maximă de 64 MHz
- **Memorie:** 1 MB Flash, 256 KB RAM
- **Alimentare:** 3.3V (furnizat de regulatorul DC-DC)
- **Conectivitate:** Bluetooth 5.0 Low Energy, USB 2.0
- **Periferice utilizate:** SPI, I2C, GPIO, PWM, USB

Antena ceramică 2450AT18B100E (2.4 GHz) este acordată printr-o rețea de matching LC calculată pentru impedanță de 50Ω, asigurând performanță RF optimă.

### 3.2 Sistemul de Alimentare

#### Baterie LiPo

Dispozitivul este alimentat de o baterie LiPo cu tensiune nominală de 3.7V (max 4.2V) și capacitate de 250 mAh.

#### Încărcător — BQ25180

Încărcarea bateriei se realizează prin circuitul BQ25180 care primește alimentare de la conectorul USB-C. Acesta oferă management complet al ciclului de încărcare, protecție termică și reglare a curentului de încărcare pentru bateria LiPo. Comunicarea cu MCU-ul se face prin I2C, permițând configurarea parametrilor de încărcare din software.

#### Convertor DC-DC — RT6160A

Regulatorul buck-boost RT6160A convertește tensiunea variabilă a bateriei (3.0–4.2V) într-o ieșire stabilă de 3.3V. Alegerea unui convertor buck-boost (în loc de un LDO) asigură funcționarea stabilă a întregului sistem inclusiv când bateria este aproape descărcată. Tensiunea de 3.3V alimentează MCU-ul, senzorii, display-ul și toate celelalte periferice.

#### Fuel Gauge — MAX17048

Monitorizarea stării bateriei este realizată de MAX17048, care utilizează algoritmul ModelGauge pentru a estima nivelul de încărcare (State of Charge) fără a necesita rezistență de shunt. Comunicarea se face prin I2C, iar pinul ALERT (conectat la P0.10 pe MCU) poate genera o întrerupere atunci când nivelul bateriei scade sub un prag configurat.

### 3.3 Interfața USB-C

Conectorul USB Type-C (KH-TYPE-C-16P) îndeplinește două roluri: alimentare pentru încărcarea bateriei și comunicație USB 2.0 cu MCU-ul. Liniile de date D+ și D- sunt conectate la pinii USB dedicați ai nRF52840, iar pe liniile CC1/CC2 sunt montate rezistențe pull-down de 5.1kΩ pentru identificarea rolului de device. Protecția ESD pe liniile USB este asigurată de circuitul USBLC6-2SC6Y.

### 3.4 Display E-Paper

Display-ul e-paper de 1.54 inch este conectat prin FPC (conector Molex 503480-2400, 24-pin). Principalele avantaje ale tehnologiei e-paper pentru un smartwatch sunt consumul aproape zero în idle (imaginea persistă fără alimentare) și lizibilitatea excelentă în lumina solară.

Comunicarea cu MCU-ul se face prin **SPI** cu următoarele semnale:
- **MOSI** — transmite date și comenzi către display
- **SCK** — semnal de clock pentru sincronizarea transferului
- **DC** — diferențiază între comenzi și date
- **RST** — resetare hardware a controllerului display-ului
- **BUSY** — semnalizează că display-ul procesează un refresh

Alimentarea display-ului este controlată de un PFET Si2301CDS, permițând oprirea completă a panoului pentru economisirea energiei. Circuitul driver integrat pe placa EPD generează tensiunile înalte necesare actualizării cernelii electronice.

### 3.5 Accelerometru — BMA421

Senzorul BMA421 oferă măsurători de accelerație pe 3 axe și este utilizat pentru:
- detecția mișcării și a gesturilor (ridicare încheietură)
- funcționalitate de pedometru
- activarea ecranului din sleep

Comunicarea se face prin **I2C** (SDA, SCL), iar senzorul dispune de două linii de întrerupere (INT1, INT2) care permit trezirea MCU-ului din modul de consum redus doar când apare un eveniment relevant.

### 3.6 Driver Haptic — DRV2605

Driverul DRV2605 controlează motorul de vibrație (LRA/ERM) și oferă o bibliotecă de peste 100 de efecte haptice predefinite. Configurarea efectelor se realizează prin **I2C**, iar activarea vibrației se poate face și direct printr-un semnal digital pe pinul EN, fără a necesita comunicare I2C continuă.

### 3.7 Butoane

Trei butoane tactile (EVP-AKB31A) sunt conectate la pini GPIO ai MCU-ului cu rezistențe pull-up de 10kΩ. La apăsare, butonul conectează pinul la masă, iar MCU-ul detectează tranziția HIGH→LOW. Fiecare buton are un condensator de debouncing de 1µF.

- **UP** — navigare sus în meniu
- **ENTER** — selecție / confirmare
- **DOWN** — navigare jos în meniu

### 3.8 Interfața SWD

Programarea și debugging-ul firmware-ului se realizează prin interfața SWD (Serial Wire Debug) accesibilă printr-un header Tag-Connect TC2030-IDC (6-pin). Semnalele utilizate: SWDIO, SWDCLK, RESET, VCC, GND.

## 4. Bill of Materials (BOM)

| Componentă | Package | JLC Parts | Datasheet |
|------------|---------|-----------|-----------|
| Antenna (2450AT18B100E) | SMD 3216 | [C5179427](https://jlcpcb.com/partdetail/C5179427) | [Datasheet](https://www.johansontechnology.com/datasheets/2450AT18B100/2450AT18B100.pdf) |
| Charger (BQ25120YBGR) | DSBGA-9 | [C1682423](https://jlcpcb.com/partdetail/C1682423) | [Datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| Haptic Driver (DRV2605LDGSR) | VSSOP-10 | [C527464](https://jlcpcb.com/partdetail/C527464) | [Datasheet](https://www.ti.com/lit/ds/symlink/drv2605l.pdf) |
| Buttons (EVP-AKB31A) | SMD tactile | [C2845028](https://jlcpcb.com/partdetail/C2845028) | [Datasheet](https://industrial.panasonic.com/cdbs/www-data/pdf/ATV0000/ATV0000CE3.pdf) |
| USB Type-C Connector (KH-TYPE-C-16P) | 16-pin SMD | [C2765186](https://jlcpcb.com/partdetail/C2765186) | [Datasheet](https://datasheet.lcsc.com/lcsc/2012121836_Kinghelm-KH-TYPE-C-16P_C2765186.pdf) |
| Fuel Gauge (MAX17048G+T10) | DFN-8 (2x2) | [C2682616](https://jlcpcb.com/partdetail/C2682616) | [Datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17048-MAX17049.pdf) |
| MCU (nRF52840-QIAA-R) | aQFN73 (7x7) | [C190794](https://jlcpcb.com/partdetail/C190794) | [Datasheet](https://docs.nordicsemi.com/) |
| Accelerometer (BMA421) | LGA-12 (2x2) | [C5242966](https://jlcpcb.com/partdetail/C5242966) | [Datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma421-ds004.pdf) |
| FPC Connector (Molex 503480-2400) | 24-pin 0.5mm FPC | [C2857168](https://jlcpcb.com/partdetail/C2857168) | [Datasheet](https://www.molex.com/en-us/products/part-detail/5034802400) |
| Buck-Boost Regulator (RT6160AWSC) | WLCSP-15 | [C7065276](https://jlcpcb.com/partdetail/C7065276) | [Datasheet](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-05.pdf) |
| Vibration Motor (LCM102733605F) | Wire leads | [C7528006](https://jlcpcb.com/partdetail/C7528006) | [Datasheet](https://datasheet.lcsc.com/lcsc/2310131633_LCM-LCM1027B3605F_C7528806.pdf) |
| PFET EPD power (Si2301CDS) | SOT-23 | [C10487](https://jlcpcb.com/partdetail/C10487) | [Datasheet](https://www.vishay.com/docs/68749/si2301cds.pdf) |
| USB ESD Protection (USBLC6-2SC6Y) | SOT-23-6 | [C7519](https://jlcpcb.com/partdetail/C7519) | [Datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| SWD Debug Header (TC2030-IDC) | Tag-Connect 6-pin | - | - |
| Crystal 32 MHz | 2016 | - | - |
| Crystal 32.768 kHz | 3215 | - | - |
| Resistors x 2 10k | 0201 | - | - |
| Resistors x 2 5.1k | 0201 | - | - |
| Capacitors x 4 12pF | 0201 | - | - |
| Capacitors x 5 100nF | 0201 | - | - |
| Capacitors x 5 4.7uF | 0402 | - | - |
| Capacitors x 2 22uF | 0402 | - | - |
| Capacitors x 12 1uF | 0402 | - | - |
| Inductor 0.47uH | 2012 | - | - |
| Inductor 10uH | 0402 | - | - |
| E-Paper Display 1.54 inch | 24-pin FPC | - | - |
| LiPo Battery 250 mAh | - | - | - |

## 5. Alocare Pini nRF52840

### 5.1 Interfața SPI — Display E-Paper

| Pin | Semnal | Rol |
|-----|--------|-----|
| P0.02 | SCK | Clock SPI pentru transfer date către display |
| P0.03 | MOSI | Linie de date master → display |
| P0.15 | DC | Selecție date/comandă |
| P0.16 | RST | Reset hardware controller display |
| P0.17 | BUSY | Stare display (ocupat/liber) |

SPI a fost ales deoarece actualizarea display-ului e-paper necesită transferuri rapide de blocuri mari de date (framebuffer complet). Pinii P0.02 și P0.03 suportă mapare SPI flexibilă pe nRF52840, iar P0.15–P0.17 sunt pini GPIO adiacenți, simplificând rutarea pe PCB.

### 5.2 Magistrala I2C — Senzori și Management Energetic

| Pin | Semnal | Dispozitive conectate |
|-----|--------|-----------------------|
| P0.26 | SDA | BMA421, DRV2605, MAX17048, RT6160A, BQ25180 |
| P0.27 | SCL | BMA421, DRV2605, MAX17048, RT6160A, BQ25180 |

P0.26 și P0.27 sunt pini cu suport hardware I2C nativ (TWIM peripheral) pe nRF52840. Toate cele cinci dispozitive I2C sunt conectate pe aceeași magistrală, fiecare cu adresă unică, reducând numărul de pini utilizați. Pe liniile SDA și SCL sunt montate rezistențe pull-up la 3.3V.

### 5.3 Semnale Suplimentare Senzori

| Pin | Semnal | Rol |
|-----|--------|-----|
| P0.10 | ALERT (MAX17048) | Întrerupere la nivel critic baterie, permite trezirea MCU din sleep |
| P0.22 | EN (DRV2605) | Activare directă efect haptic fără comunicație I2C |

Pinii de întrerupere ai BMA421 (INT1, INT2) sunt de asemenea conectați la MCU, permițând detecția evenimentelor de mișcare fără polling continuu.

### 5.4 Butoane — GPIO

| Pin | Funcție | Detalii |
|-----|---------|---------|
| P0.13 | Button UP | Pull-up 10kΩ, debounce 1µF |
| P0.14 | Button ENTER | Pull-up 10kΩ, debounce 1µF |
| P1.02 | Button DOWN | Pull-up 10kΩ, debounce 1µF |

Pinii sunt configurați ca intrări digitale. Alegerea lor a ținut cont de disponibilitatea pe portul GPIO și de rutarea optimă pe PCB. Apăsarea butonului trage pinul la GND, iar MCU-ul detectează frontul descrescător.

### 5.5 USB

| Pin | Semnal |
|-----|--------|
| D+ (dedicat) | Linie de date USB+ |
| D- (dedicat) | Linie de date USB- |
| VBUS (dedicat) | Detecție alimentare USB |

Pinii USB sunt dedicați pe nRF52840 (nu sunt GPIO remapabili) și sunt conectați la conectorul USB-C prin circuitul de protecție ESD USBLC6-2SC6Y.

### 5.6 Alte Semnale

| Pin | Semnal | Rol |
|-----|--------|-----|
| P0.00 | XL1 | Cristal 32.768 kHz — clock RTC pentru timekeeping în sleep |
| P0.01 | XL2 | Cristal 32.768 kHz — pereche oscilator LFXO |
| SWDIO | SWD Data | Programare și debug firmware |
| SWDCLK | SWD Clock | Clock interfață debug |

P0.00 și P0.01 sunt singurii pini pe nRF52840 dedicați oscilatorului low-frequency (LFXO). Cristalul de 32.768 kHz permite funcționarea RTC-ului cu consum extrem de redus (~2 µA) în modul System ON sleep.

### 5.7 Criterii de Alocare

Distribuția pinilor a fost realizată ținând cont de:
- utilizarea interfețelor hardware native ale MCU-ului (SPI, I2C, USB, LFXO)
- gruparea semnalelor conexe pe pini adiacenți pentru simplificarea traseelor PCB
- disponibilitatea funcțiilor de întrerupere hardware pe pinii aleși
- minimizarea consumului prin posibilitatea de wake-up din sleep pe oricare din pinii GPIO utilizați

## 6. Estimare Consum de Energie

### Consum per componentă

| Componentă | Mod Sleep / Idle | Mod Activ |
|------------|-----------------|-----------|
| nRF52840 | ~2–5 µA (System ON sleep) | ~5–10 mA (CPU activ, radio oprit) |
| nRF52840 (BLE TX) | — | ~4.8 mA (transmisie la 0 dBm) |
| Display E-Paper | ~0 µA (imagine statică) | ~10–20 mA (în timpul refresh-ului) |
| BMA421 | ~14 µA (low-power mode) | ~160 µA (normal mode) |
| MAX17048 | ~50 µA (măsurare continuă) | ~50 µA |
| DRV2605 + Motor | ~0 µA (standby) | ~100 mA (vibrație activă) |
| RT6160A | ~15 µA (quiescent) | depinde de sarcină |
| BQ25180 | ~1 µA (mod ship) | depinde de curentul de încărcare |

### Scenariu tipic de utilizare

În funcționare normală (ecran static, BLE inactiv, fără vibrație), consumul este dominat de MCU în sleep și senzori:

| Componentă | Consum estimat |
|------------|---------------|
| MCU (sleep cu RTC) | ~5 µA |
| BMA421 (low-power) | ~14 µA |
| MAX17048 | ~50 µA |
| RT6160A (quiescent) | ~15 µA |
| **Total idle** | **~84 µA** |

Cu o baterie de 250 mAh, autonomia teoretică maximă în idle este:

**250.000 µAh / 84 µA ≈ 2.976 ore ≈ 124 zile**

Într-un scenariu realist (cu refresh-uri periodice ale ecranului, citiri BLE ocazionale și notificări haptice), consumul mediu estimat este de aproximativ **0.5–1 mA**, rezultând o autonomie de **10–20 zile** — competitivă cu smartwatch-urile e-paper comerciale.

## 7. Jurnal de Design și Fabricație

- **Proiectare PCB:** S-a optat pentru o configurație de 4 straturi (Signal-GND-VCC-Signal) pentru a asigura ecranarea electromagnetică și integritatea semnalului RF.
- **Miniaturizare:** Toate componentele pasive de decuplare din jurul nRF52840 sunt de mărime 0201, reducând dimensiunea finală a plăcii.
- **Design Mecanic:** Carcasa a fost modelată în Fusion 360 pentru a integra PCB-ul și bateria de 250mAh într-un profil compact, utilizând toleranțe strânse pentru montarea prin presare.

## 8. Galerie Randări și Producție

*(Notă: Adăugați aici imaginile proprii generate din Fusion 360)*

- **PCB 3D View:** Placa electronică cu toate componentele populate.
- **Exploded View:** Vedere desfășurată a componentelor în carcasă.
- **Final Assembly:** Randarea smartwatch-ului asamblat complet.
