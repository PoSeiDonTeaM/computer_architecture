# Αρχιτεκτονική Υπολογιστών

## Εργαστήριο 1 - Ερωτήματα Πρώτου Μέρους

Σκοπός του τρέχοντος εργαστηρίου είναι η εξοικείωση με το gem5 και τη βιβλιογραφία πάνω στην οποία θα βασιστούμε για την ολοκλήρωση των παραδοτέων.

### 1. Βασικές παράμετροι του συστήματος

|  #  |              Arguement              |       Value       |
|:---:|:-----------------------------------:|:-----------------:|
|  1. |           L1 iCACHE Class:          |    devices.L1I    |
|  2. |           L1 dCACHE Class:          |    devices.L1D    |
|  3. |          Walk Cache Class:          | devices.WalkCache |
|  4. |           L2 Cache Class:           |     devices.L2    |
|  5. |              CPU Type:              |      MinorCPU     |
|  6. |            CPU Frequency:           |        4GHz       |
|  7. |         Number of CPU cores:        |         1         |
|  8. |           Type of Memory:           |   DDR3_1600_8x8   |
|  9. |      Number of Memory channels:     |         2         |
| 10. | Number of Memory ranks per channel: |         2         |
| 11. | Physical Memory size:               |        2GB        |

### 2. Επαλήθευση των παραμέτρων ερωτήματος (1)

Ύστερα από την εκτέλεση του αρχείου `starter_se.py`, δημιουργούνται τα αρχεία `stats.txt`, `config.ini` και `config.json` από τα οποία μπορούμε να επαληθεύσουμε τις παραμέτρους που χρησιμοποιήθηκαν παραπάνω.


**5.** CPU Type:

```
66: [system.cpu_cluster.cpus]
67: type=MinorCPU
```

**6.** CPU Frequency:

```
58: [system.cpu_cluster.clk_domain]
59: type=SrcClockDomain
60: clock=250
```
Το clock ισούται με 250 (picoseconds), τα οποία με τον τύπο F = 1/T προκύπτουν 4GHz, όπως έχουμε ορίσει παραπάνω.

**7.** Number of CPU cores:

```
117: numThreads=1
```

**10.** Number of Memory ranks per channel:

```
1296: ranks_per_channel=2
```

**11.** Physical Memory size:

```
25: mem_ranges= 0:2147483647
```
Συνολικά δηλαδή έχουμε 2.14GB μνήμης

### Σύγκριση του C προγράμματος σε MinorCPU και TimingSimpleCPU

Στα μοντέλα in-order CPUs που χρησιμοποιεί ο gem5 συγκαταλέγονται ο `BaseSimpleCPU` o οποίος χωρίζεται στον `AtomicSimpleCPU` και τον `TimingSimpleCPU`. Μερικές πληροφορίες για αυτούς αναγράφονται παρακάτω.

#### BaseSimpleCPU

Ο BaseSimpleCPU επιτελεί διάφορους σκοπούς:
* Διατηρεί την κατάσταση αρχιτεκτονικής και έχει παρόμοια στατιστικά στα διάφορα μοντέλα SimpleCPU.
* Έχει τη δυνατότητα να ορίζει λειτουργίες (functions) για να ελέγχει για διακοπές (interrupts), να ετοιμάζει ένα αίτημα fetch, ετοιμάζει την ετοιμασία προ-εκτέλεσης (pre-execute setup), διαχειρίζεται τις μετα-εκτέλεσης δραστηριότητες (post-execute actions), και προχωράει τον μετρητή προγράμματος (Program Counter) στην επόμενη εντολή.
* Επιτελεί την ExecContext διεπαφή.

Ο BaseSimpleCPU δεν μπορεί να τρέξει μόνος του. Είναι απαραίτητη η χρήση μίας από τις κλάσεις που κληρονομεί από την BaseSimpleCPU, είτε την AtomicSimpleCPU ή την TimingSimpleCPU.


<p align="center">
  <img src="http://www.gem5.org/docs/html/classAtomicSimpleCPU.png">
</p>


## AtomicSimpleCPU

Η AtomicSimpleCPU είναι η έκδοση της SimpleCPU όπου χρησιμοποιεί προσβάσεις ατομικής μνήμης. Χρησιμοποιεί τις εκτιμήσεις των καθυστερήσεων μεταφοράς από τις ατομικές προσβάσεις, με σκοπό να εκτιμήσει το συνολικό χρόνο πρόσβασης της cache. Ο AtomicSimpleCPU προέρχεται από τον BaseSimpleCPU και επιτελεί λειτουργίες για να διαβάζει και να γράφει στη μνήμη, όπως επίσης να κάνει τικ, το οποίο ορίζει τι συμβαίνει σε κάθε κύκλο CPU. Προσδιορίζει τη θύρα που χρησιμοποιείται για να κλειδώσει στη μνήμη και συνδέει τη CPU στη μνήμη cache.

## TimingSimpleCPU

Ο TimingSimpleCPU είναι η έκδοση του SimpleCPU που χρησιμοποιεί προσβάσεις μνήμης χρονισμού. Αυτό δημιουργεί καθυστερήσεις στις προσβάσεις της ache και περιμένη το σύστημα μνήμης να ανταποκριθεί πριν προχωρήσει παρακάτω. Όπως ο AtomicSimpleCPU, έτσι και ο TimingSimpleCPU προέρχεται από τον BaseSimpleCPU και επιτελεί το ίδιο σετ λειτουργιών. Προσδιορίζει τη θύρα που χρησιμοποιείται για να κλειδώσει στη μνήμη και συνδέει τον CPU στη μνήμη ache. Επίσης, προσδιορίζει τις απαραίτητες λειτουργίες για τη διαχείριση της απόκρισης από τη μνήμη στις προσβάσεις που έχουν σταλεί έξω.

## MinorCPU

Ο MinorCPU αποτελεί ένα in-order μοντέλο επεξεργαστή με ένα σταθερό pipeline αλλά προσαρμοζόμενες δομές δεδομένων και συμπεριφορά εκτέλεσης. Χρησιμοποιείται σε μοντέλα επεξεργαστών με αυστηρή συμπεριφορά εκτέλεσης και επιτρέπει την απεικόνιση μία θέσης εντολής στο pipeline διαμέσω ενός αρχείου python. Σκοπός του είναι να προσφέρει ένα framework για συσχετίζει μικρο-αρχιτεκτονικά το μοντέλο με έναν συγκεκριμένο, επιλεγμένο επεξεργαστή με παρόμοιες δυνατότητες.

-----------------------------------------------------------------------------------
### α. Σύγκριση χρόνων προσομοίωσης 

Συγκρίνοντας τους χρόνους προσομοίωσης του προγράμματος C που δημιουργήσαμε, παρατηρούμε ότι το πρόγραμμα έχει γρηγορότερο χρόνο εκτέλεσης όταν τρέχει σε τύπο MinorCPU. Παρακάτω παραθέτονται τα αποτελέσματα που βρίσκονται στα αντίστοιχα αρχεία `stats.txt`.

**MinorCPU:**
```
12: sim_seconds   0.000033   # Number of seconds simulated
```
**TimingSimpleCPU:**
```
12: sim_seconds   0.000040   # Number of seconds simulated
```

Το αποτέλεσμα είναι αναμενόμενο, καθώς ο TimingSimpleCPU δε διαθέτει pipeline που σημαίνει ότι όταν αναλαμβάνει να εκτελέσει διάφορες διαδικασίες, αναγκάζεται να τις τρέχει σειριακά ολοκληρώνοντας πρώτα όλα τα βήματα χωρίς να επιτρέπει την επόμενη διεργασία να ξεκινήσει όταν η προηγούμενη έχει ολοκληρωθεί.

### β. Σύγκριση υπόλοιπων στοιχείων του stats.txt

|    Parameter   |   MinorCPU  | TimingSimpleCPU |        Units       |
|:--------------:|:-----------:|:---------------:|:------------------:|
| host_inst_rate |   205,865   |     665,055     | (Instructions/sec) |
|  host_op_rate  |   244,216   |     781,539     | (Operations / sec) |
|  host_seconds  |     0.06    |       0.02      |      (seconds)     |
| host_tick_rate | 581,360,740 |  2,319,062,593  |    (Ticks / sec)   |
|    sim_insts   |    11,517   |      11,460     |          -         |
|   sim_seconds  |   0.000033  |     0.000040    |      (seconds)     |

Από το πινακάκι παρατηρούμε ότι παρόλο που ο `TimingSimpleCPU` έχει καλύτερα στατιστικά από τον `MinorCPU`, ο `MinorCPU` εκτελεί γρηγορότερα την προσομοίωση για παραπλήσιο αριθμό εντολών (instructions) με τον `TimingSimpleCPU`.

### γ. Σύγκριση με αλλαγή παραμέτρων (Συχνότητας, Μνήμης)

## Frequency

#### MinorCPU

|    Parameter   |     1GHz    |     2GHz    |         Units        |
|:--------------:|:-----------:|:-----------:|:--------------------:|
| host_inst_rate |   200,162   |   211,447   | (Instructions / sec) |
|  host_op_rate  |   237,435   |   250,779   |  (Operations / sec)  |
|  host_seconds  |     0.06    |     0.05    |       (seconds)      |
| host_tick_rate | 714,675,215 | 596,969,560 |     (Ticks / sec)    |
|    sim_insts   |    11,517   |    11,517   |           -          |
|   sim_seconds  |   0.000041  |   0.000033  |       (seconds)      |

#### TimingSimpleCPU

|    Parameter   |      1GHz     |      2GHz     |         Units        |
|:--------------:|:-------------:|:-------------:|:--------------------:|
| host_inst_rate |    698,218    |    662,492    | (Instructions / sec) |
|  host_op_rate  |    820,565    |    778,805    |  (Operations / sec)  |
|  host_seconds  |      0.02     |      0.02     |       (seconds)      |
| host_tick_rate | 3,518,114,694 | 2,311,165,883 |     (Ticks / sec)    |
|    sim_insts   |     11,460    |     11,460    |           -          |
|   sim_seconds  |    0.000058   |    0.000040   |       (seconds)      |

Παρατηρούμε ότι όταν η συχνότητα ρολογιού του επεξεργαστή αυξάνεται (2GHz > 1GHz), αυξάνεται ο αριθμός των εντολών που μπορεί να τρέχει ανά μονάδα χρόνου και επομένως το πρόγραμμα εκτελείται με μεγαλύτερη ταχύτητα. Για αυτό το λόγο και ο χρόνος προσομοίωσης είναι μικρότερος για τα 2GHz.

## Memory Type

#### MinorCPU

|    Parameter   |  DDR3213388 | DDR42400416 |         Units        |
|:--------------:|:-----------:|:-----------:|:--------------------:|
| host_inst_rate |   206,822   |   202,697   | (Instructions / sec) |
|  host_op_rate  |   245,354   |   240,434   |  (Operations / sec)  |
|  host_seconds  |     0.06    |     0.06    |       (seconds)      |
| host_tick_rate | 558,262,502 | 577,195,923 |     (Ticks / sec)    |
|    sim_insts   |    11,517   |    11,517   |           -          |
|   sim_seconds  |   0.000031  |   0.000033  |       (seconds)      |

#### TimingSimpleCPU

|    Parameter   |   DDR3213388  |  DDR42400416  |         Units        |
|:--------------:|:-------------:|:-------------:|:--------------------:|
| host_inst_rate |    646,721    |    619,259    | (Instructions / sec) |
|  host_op_rate  |    760,367    |    728,341    |  (Operations / sec)  |
|  host_seconds  |      0.02     |      0.02     |       (seconds)      |
| host_tick_rate | 2,204,488,737 | 2,197,820,747 |     (Ticks / sec)    |
|    sim_insts   |     11,460    |     11,460    |           -          |
|   sim_seconds  |    0.000039   |    0.000041   |       (seconds)      |

Παρατηρούμε ότι και στα δυο μοντέλα οι επιδόσεις για τις διαφορετικές μνήμες είναι σχεδόν ίδιες. ´Ομως πρεπει να επισημανθεί ότι η μνήμη DDR3 είναι λίγο γρηγορότερη και στις 2 περιπτώσεις. Αντιφατικό γεγονός καθώς η DDR4 είναι εξέλιξη της DDR3 πιο γρήγορη και συνεπώς θα επρεπε να είναι ταχύτερη. Βέβαια είναι πιθανόν σε μεγάλα προγράμματα να γίνεται φανερό αυτό.
