# InkTime Smartwatch

**Student:** Ghița Maria, 331CC

## 1. Descriere Generală

Proiectul **InkTime** este un dispozitiv purtabil bazat pe microcontrolerul **nRF52840**, proiectat pentru a oferi o autonomie ridicată prin utilizarea unui ecran de tip E-Paper. Sistemul integrează funcționalități de monitorizare a mișcării, interfață haptică și management avansat al energiei, fiind optimizat pentru dimensiuni reduse și eficiență.

---

## 2. Diagrama Bloc

![Diagrama Bloc InkTime](Images/block_diagram.png)

Arhitectura hardware este structurată în jurul nucleului ARM Cortex-M4F, interconectat cu următoarele module:

- **Sistem de Procesare:** nRF52840 (Bluetooth 5.0, GPIO, SPI, I2C)
- **Afișaj:** E-Paper Display (EPD) cu circuit de drive integrat
- **Senzori & Haptics:** Accelerometru BMA423 și Driver Haptic DRV2605
- **Management Energie:** Încărcător LiPo BQ25180, Fuel Gauge MAX17048 și convertor DC/DC RT610A
- **Interfață Utilizator:** Trei butoane de control (Up, Enter, Down)

---

## 3. Funcționalitate Hardware și Interfețe

- **Microcontroler (nRF52840):** Gestionează stiva de comunicație Bluetooth și coordonează toți perifericii. S-a optat pentru acest model datorită consumului redus în modurile de repaus și a unității de calcul în virgulă mobilă (FPU).
- **Afișaj E-Paper:** Controlat prin interfața SPI. Circuitul de drive (bazat pe inductorul L5 și diodele MBR0530) asigură tensiunile necesare pentru actualizarea pixelilor.
- **Managementul Bateriei:**
  - **BQ25180:** Gestionează încărcarea prin USB-C și monitorizează curentul de intrare.
  - **MAX17048:** Monitorizează tensiunea celulei LiPo și oferă un algoritm de calcul al stării de încărcare (SoC) prin I2C.
  - **RT610A:** Convertor Buck-Boost care stabilizează tensiunea la 3.3V, indiferent de nivelul de descărcare al bateriei.
- **Senzori și Actuatori:**
  - **BMA423:** Senzor de mișcare conectat prin I2C, utilizat pentru pedometru și gesturi.
  - **DRV2605:** Driver pentru motorul de vibrații, capabil să redea efecte haptice complexe.
- **Protecție și RF:** Linia USB-C este protejată de descărcări electrostatice prin USBLC6-2SC6Y. Antena ceramică de 2.4GHz este acordată prin componentele de matching (L4, C9) pentru o performanță optimă la 50Ω.

---

## 4. Alocare Pini (Pinout nRF52840)

| Componentă | Pini MCU | Interfață | Justificare |
|---|---|---|---|
| E-Paper (SPI) | P0.17, P0.20, P0.22 | SPI | Transfer rapid de date pentru imagine |
| I2C Bus (Shared) | P0.26 (SDA), P0.27 (SCL) | I2C | Conectează BMA423, DRV2605 și MAX17048 |
| Butoane Fizice | P0.11, P0.12, P0.24 | GPIO | Intrare digitală pentru navigație meniu |
| Battery Charger | P0.02 | INT / Analog | Monitorizare status și întreruperi încărcare |
| Crystal 32kHz | P0.00, P0.01 | XL1 / XL2 | Referință de timp pentru modurile Low Power |

---

## 5. Fabricație și Fișiere de Producție

Dosarul proiectului include toate documentele necesare fabricării PCB-ului și asamblării (PCBA):

- **BOM (.bom):** Lista completă de componente cu link-uri către furnizori și coduri de comandă LCSC.
- **Gerber Files (.zip):** Straturile de cupru, masca de lipire, silkscreen-ul și fișierele de găurire (Drill).
- **Pick & Place (.cpl):** Coordonatele X-Y și rotația fiecărei componente pentru asamblarea automată.

---

## 6. Bill of Materials (BOM)

| Referință | Componentă | Valoare | Capsulă |
| :--- | :--- | :--- | :--- |
| **U1** | nRF52840 | SoC Bluetooth | QFN73 |
| **U2** | BQ25180 | Charger Li-Ion | DSBGA-9 |
| **U3** | MAX17048 | Fuel Gauge | SOT23-5 |
| **U4** | RT610A | Voltage Regulator | SOT23-5 |
| **U5** | BMA423 | Accelerometru | LGA-12 |
| **U6** | DRV2605 | Haptic Driver | WSON-10 |
| **J1** | Conector E-Paper | 503480-2400 | FPC-24 |
| **J3** | Conector USB-C | KH-TYPE-C-16P | SMD-16 |
| **X1** | Cristal | 32MHz | SMD |
| **X2** | Cristal | 32.768kHz | SMD |

---

## 7. Jurnal de Design și Decizii Tehnice

- **Utilizarea componentelor 0201:** Condensatoarele de decuplare de sub nRF52840 sunt de mărime 0201 pentru a permite o rutare cât mai scurtă către masă și a economisi spațiu pe un PCB dens.
- **Traseele de alimentare și utilizarea VIA-urilor:** S-a optat pentru utilizarea vias-urilor pe traseele de alimentare (V_BAT, 3.3V) pentru a facilita rutarea între straturi în zonele aglomerate. În punctele de distribuție principală s-au folosit mai multe via-uri în paralel pentru a crește capacitatea de transport a curentului și pentru a reduce inductanța parazită, prevenind căderile de tensiune în timpul vârfurilor de consum.
