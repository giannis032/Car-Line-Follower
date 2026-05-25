# 🤖 Line Follower Robot — Ενσωματωμένα Συστήματα

> Εργασία για το μάθημα **Ενσωματωμένα Συστήματα**  
> Υλοποίηση αυτόνομου ρομπότ παρακολούθησης γραμμής με **Raspberry Pi Pico 2** και έλεγχο **PD**

---

## 📋 Περιγραφή

Το παρόν έργο αφορά τον σχεδιασμό και την υλοποίηση ενός αυτόνομου **Line Follower Robot**, δηλαδή ενός ρομπότ που ακολουθεί μια μαύρη γραμμή πάνω σε λευκό υπόστρωμα.

Ο έλεγχος κίνησης βασίζεται σε **PD controller (Proportional-Derivative)** με δυναμική ανάγνωση 5 υπέρυθρων αισθητήρων και αυτόματη **βαθμονόμηση (calibration)** κατά την εκκίνηση.

---

## 🔧 Υλικά (Bill of Materials)

| Εξάρτημα | Μοντέλο / Περιγραφή |
|---|---|
| **Μικροελεγκτής** | Raspberry Pi Pico |
| **Driver Κινητήρων** | DRV8835 Dual Motor Driver Carrier |
| **Τροφοδοτικό** | Pololu 5V 500mA Step-Down Voltage Regulator D24V5F5 |
| **Αισθητήρες IR** | Waveshare Infrared Line Tracking Tracker Sensor |
| **Κινητήρες** | N20 DC 3V–6V Micro Reduction Motor (Pure Steel Metal Gear) |
| **Μπαταρία** | 7.4V 800mAh LiPo Battery 25C (2S) |
| **Ρόδες** | ZJ327 3PI miniQ N20 Motor Rubber Wheel — Ø34mm × 7mm |
| **Βάση Κινητήρων** | Mounting Bracket for N20 Micro Gear Motors |
| **Σκελετός** | Βάση PVC (λευκό χρώμα) |
| **Καλώδια** | Dupont Female-to-Female / Male-to-Female / Male-to-Male — 10/15/20/30/40cm |

---

## 📐 Κατασκευή — Διαστάσεις Πλαισίου

```
Συνολικό Μήκος : 170 mm
Μέγιστο Πλάτος : 80 mm  (πίσω)
Μεταξόνιο      : 70 mm
Θέση Caster    : 50 mm από μπρος / ύψος 30 mm
```

Το πλαίσιο φιλοξενεί:
- **Θέση Μπαταρίας** — 2S 800mAh LiPo (κεντρικά)
- **Θέση Βάσης Μοτέρ** — πίσω δεξιά
- **Caster** — μπροστά για ισορροπία

---

## 🗂️ Δομή Αποθετηρίου

```
line-follower-robot/
│
├── main.py          # Κύριος κώδικας MicroPython (PD controller)
└── README.md        # Αυτό το αρχείο
```

---

## ⚙️ Αρχιτεκτονική Λογισμικού

### Αισθητήρες
| Αισθητήρας | Τύπος | GPIO Pin |
|---|---|---|
| IR1 (αριστερά άκρο) | Digital (PULL_UP) | GP2 |
| IR2 | Analog ADC | GP28 |
| IR3 (κέντρο) | Analog ADC | GP27 |
| IR4 | Analog ADC | GP26 |
| IR5 (δεξιά άκρο) | Digital (PULL_UP) | GP6 |

### Κινητήρες
| Κινητήρας | IN1 | IN2 |
|---|---|---|
| Motor A (Αριστερά) | GP16 (PWM) | GP17 (PWM) |
| Motor B (Δεξιά) | GP14 (PWM) | GP15 (PWM) |

### Παράμετροι PD Controller
```python
base_speed = 35000   # Βασική ταχύτητα (0–65535)
Kp         = 8000    # Proportional gain
Kd         = 15000   # Derivative gain
```

---

## 🧠 Λειτουργία Αλγορίθμου

```
1. CALIBRATION (3 δευτερόλεπτα)
   └─ Διαβάζει IR3 πάνω σε μαύρο, IR2/IR4 πάνω σε λευκό
   └─ Υπολογίζει THRESHOLD = avg_black + (avg_white - avg_black) × 0.3

2. MAIN LOOP
   ├─ Διαβάζει 5 αισθητήρες → vals[0..4]
   ├─ Αν sum(vals) > 3 → STOP (διασταύρωση / εμπόδιο)
   ├─ Αν active == 0  → στροφή προς last_direction
   └─ Αλλιώς → PD correction
          error     = Σ(weight_i × val_i) / active
          P         = Kp × error
          D         = Kd × (error - last_error)
          left_mot  = base_speed + correction
          right_mot = base_speed - correction
```

### Βάρη Αισθητήρων
```
IR1: -2  |  IR2: -1  |  IR3: 0  |  IR4: +1  |  IR5: +2
```
Αρνητική τιμή → εκτροπή αριστερά → διόρθωση δεξιά, και αντίστροφα.

---

## 🚀 Εγκατάσταση & Εκτέλεση

### Προαπαιτούμενα
- [Thonny IDE](https://thonny.org/) ή `mpremote`
- Raspberry Pi Pico 2 με **MicroPython** firmware

### Βήματα

```bash
# 1. Κατέβασε το αποθετήριο
git clone https://github.com/<username>/line-follower-robot.git
cd line-follower-robot

# 2. Άνοιξε το main.py στο Thonny IDE
# 3. Σύνδεσε το Pico 2 μέσω USB
# 4. Αντέγραψε το main.py στο Pico (Run → Upload current script)
# 5. Τοποθέτησε το ρομπότ στη γραμμή & τρέξε
```

### Διαδικασία Calibration
Κατά την εκκίνηση, το ρομπότ περιμένει **3 δευτερόλεπτα**:
1. Τοποθέτησε τον **κεντρικό αισθητήρα (IR3) πάνω στη μαύρη γραμμή**
2. Τοποθέτησε τους **IR2 & IR4 πάνω στο λευκό**
3. Το threshold υπολογίζεται αυτόματα

---

## 📊 Χαρακτηριστικά

- ✅ Αυτόματο calibration κατά την εκκίνηση
- ✅ PD controller με δυναμική ενίσχυση (1.8× / 2.0× σε έντονες στροφές)
- ✅ Median filter για σταθερές ADC μετρήσεις
- ✅ Αυτόματη ανίχνευση διασταύρωσης / άκρου γραμμής
- ✅ PWM έλεγχος ταχύτητας και κατεύθυνσης κινητήρων
- ✅ Recovery όταν χαθεί η γραμμή (last_direction)

---

## 📚 Μάθημα

| | |
|---|---|
| **Μάθημα** | Ενσωματωμένα Συστήματα |
| **Γλώσσα** | MicroPython |
| **Πλατφόρμα** | Raspberry Pi Pico 2 |

---

## 📄 Άδεια

Το έργο διατίθεται για εκπαιδευτικούς σκοπούς.
