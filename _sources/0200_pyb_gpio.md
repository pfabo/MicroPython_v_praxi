---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# <font color='navy'> GPIO </font>

Piny púzdra mikrokontroléra sú štandardne pripojené k paralelnému rozhrania procesora označované ako GPIO (General Purpose Input Output). Piny sú sdržené do skupín označených ako porty, v STM32 port pozostáva zo 16 pinov.
Podľa veľkost púzdra môže mať mikrokontrolér niekoľko portov, pričom na púzdro nemusia byť vyvedené celé porty. Okrem štandardnej funkcie pinu ako GPIO portu majú piny púzdra aj možnosť *alternatívnych funkcií*, kedy sú pripojené na interné periférie mikrokontroléra, napr. časovače, sériové rozhrania a pod. Pri návrhu komplikovanejších zariadení s mikrokontrolérom vyžaduje pridelenie pinov portom a perifériam dobré premyslenie návrhu.  

Časť pinov GPIO je výrobcom označená ako FT, umožňuje ich použitie pre periférie napájané napätím 5V, aj keď mikrokontrolér je napájaný zo zdroja s napätím 3.3V . Toto značne zjednodušuje pripájanie digitálnych periférii pracujúcich s napätím 5V bez inak potrebného prevodníka úrovní.

Základná štruktúra interného zapojenia GPIO pri procesoroch STM32 je na obrázku, podrobný popis je v dokumentácii výrobcu [AN4899](./moduly/AN4899.pdf). 

```{code-cell} ipython3  
:tags: ["remove-cell"]
from cm.utils import *

data = r'''
include(base.ckt)
include(stm32.ckt)

Origin: Here 
d = 2;

move to (0.5, 4);
# vstupne diody
GP: gpio_port(5*d/8,L); {"\sf I/O Pin" at GP.n +(-.3, -.10) above}
dot;
{down_; diode(3*d/4,,R); gnd;}
{up_;   diode(3*d/4,,); power($\sf V_{dd}$); }

line right_ 3*d/4; dot;
{  down_; 
   RUP: resistor(3*d/4,E); rlabel(,\sf R_{PD},); 
   {
	LUP: line from RUP.e + (.25, -0.4) to RUP.s  + (0.25, 0.4);
	line from LUP.c right_ d/4; {"\sf on" above ljust;}; {"\sf off" below ljust;}
   } 
   gnd;
}

{  up_;   
   RD: resistor(3*d/4,E);  llabel(,\sf R_{PU},);
   {
	LPD: line from RD.end + (0.25, -0.4)to RD.start  + (0.25, 0.4);
	line from LPD.c right_ d/4; {"\sf on" above ljust;}; {"\sf off" below ljust;}
   }
   power($\sf V_{dd}$);
}

line right_ d; DT1: dot;

#============================
# digitalna cast

line down_ d+d/4 
line right_ d/2
DT5: dot

line from DT5 up_ d/8
#move to Here + (d/8, 0)
T1: fet_P(d/2,R)

line from DT5 down_ d/8
#move to Here + (d/8, 0)
T2: fet_N(d/2,R)
move to T2.S
gnd

move to T1.D
up_
VD2: tconn(d/4,O); "$\sf V_{dd}$" at VD2.n above; 

La: line from T1.G  right_ d/4
Lb: line from T2.G  right_ d/4
move to (La.end + Lb.end)/2
boxrad=0.1 
OC: box ht d wid 2*d/3
"\sf Output" at OC.c above;
"\sf Control" at OC.c below;

DIN: line <- from OC.e right_ d/2

# digit. komparator

log_init
line from DT1 right_ d;
DOUT: opamp(d/2," ", " ",0.9,)
DOUT: line -> from DOUT.Out to (DIN.end, DOUT.Out)

#------------------------------

# analogova cast
move to DT1 + (d,d)
OP: opamp(d/2,,,1,R)
line from DT1 to (DT1, OP.In1)
right_
single_switch_h(d, OFF)
line from OP.In2 left_ d/4 
line down_ d/2-d/8
ACIN: line -> from Here to (DIN.end, Here)
ACOUT:line -> from OP.Out to (DIN.end, OP.Out)

"\sf Digital Out" at DOUT.end ljust; 
"\sf Digital In" at DIN.end ljust; 
"$\sf V_{comp}$" at ACIN.end ljust; 
"\sf CMP" at ACOUT.end ljust

# ramik okolo analogoveho komparatora
move to DT1 + (d, d/4+d/8)
up_
ABOX: box dashed ht d wid 1.6*d
"\sf Analog Option" at ABOX.n above

'''

_ = cm_compile('./img/gp_003', data,  dpi=600)   
```

```{figure} ./img/gp_003.png
:width: 500px
:name: gp_03

Interné zapojenie pinu mikrokontroléru.
```


Zapojenie každého pinu portu GPIO obsahuje 

* ochranné diódy voči krátkodobému prepätiu a ESD
* rezistory pre prevádzku portu v režime pull-up, pull-down
* digitálne výstupné obvody s výkonovým budičom 
* digitálne vstupné obvody s komparátorom s hysterézou
* pre vybrané piny portov multiplexer pre pripojenie analogových periférií


## <font color='teal'>  Knižnica pyb.Pin </font>

`MicroPython` obsahuje knižnicu *pyb* s modulom **Pin**. Riadenie pinov je prostredníctvom ich mena, z dôvodu kompatibility sú piny na kitoch *Nucleo* označované podľa modulov Arduina ako aj podľa ich pripojenia k vývodom mikrokontroléra. Na obrázku je zapojenie kitu s procesorom STM32L476, zapojenie kitov s iným mikrokontrolérom môže byť odlišné, podrobnosti sú v [dokumentácii](./moduly/UM1724.pdf). 

Červeno sú označené piny (dutinky) na konektor Morpheo podľa Arduino, modro sú označené piny podľa ich pripojenie k mikrokontroléru. Konverzná tabulka medzi oboma konektormi pre procesor STM32L476 je v súbore  [pins_L476](./moduly/pins_L476.csv).

```{figure} ./img/nucleo-l476.png
:width: 500px
:name: gp_011

Označenie pinov modulu *Nucleo-64* s procesorom STM32L476.
```

```{admonition} Poznámka
V Arduine môže mať jeden pin niekoľko onačení, napríklad pin v Nucleo-64 označený ako **PA5** má v Arduine označenie **D13**, **PA5**, **LED_GREEN**, **LED_ORANGE**, **LED_RED**.
```

## <font color='teal'>  Trieda Pin </font>

**Inicializácia**

```
Pin.init(mode, pull=Pin.PULL_NONE, *, value=None, alt=-1)
```

**Konštanty pre nastavenie funkcie pinu**

```
Pin.IN     - configure the pin for input;
Pin.OUT_PP - configure the pin for output, with push-pull control;
Pin.OUT_OD - configure the pin for output, with open-drain control;
Pin.ALT    - configure the pin for alternate function, input or output;
Pin.AF_PP  - configure the pin for alternate function, push-pull;
Pin.AF_OD  - configure the pin for alternate function, open-drain;
Pin.ANALOG - configure the pin for analog.
```

**Konštanty pre nastavenie typu výstupu**

```
Pin.PULL_NONE - no pull up or down resistors;
Pin.PULL_UP   - enable the pull-up resistor;
Pin.PULL_DOWN - enable the pull-down resistor.
```

**Funkcie pre riadenie portu**

```
Pin.value([value])  Get or set the digital logic level of the pin:
Pin.__str__()       Return a string describing the pin object.
Pin.af()            Returns the currently configured alternate-function of the pin
Pin.af_list()       Returns an array of alternate functions available for this pin.

Pin.gpio()          Returns the base address of the GPIO block associated with this pin.
Pin.mode()          Returns the currently configured mode of the pin.
Pin.name()          Get the pin name.
Pin.names()         Returns the cpu and board names for this pin.
Pin.pin()           Get the pin number.
Pin.port()          Get the pin port.
Pin.pull()          Returns the currently configured pull of the pin
```

## <font color='teal'> Príklady </font>

Preddefinované periférie na doske NUCLEO64 s procesorom STM32L476RG

```{code-cell} ipython3  
:tags: ["remove-cell"]
from cm.utils import *

data = r'''
include(base.ckt)

Origin: Here 
up_
    move to (2,0)
    gnd;
SW: single_switch(2, OFF, V, L);
    dot; {line -> right 2; "\sf PC13" at last line .end ljust}
    resistor(1.5, ,E);
    power($\sf V_{dd}$)
    "\sf Blue Button" at SW.e ljust
    
move to (6,0);
gnd;
DD: diode(2,,R); { em_arrows(,-45, .35) at DD.center +(.3,-.3); }
    dot; {line -> right 2; "\sf PA5" at last line .end ljust}
    resistor(1.5, ,E);
    power($\sf V_{dd}$)
    "\sf Green LED" at DD.center + (.5,0) ljust 
'''

_ = cm_compile('./img/gp_004', data,  dpi=600)   
```

```{figure} ./img/gp_004.png
:width: 300px
:name: gp_04

Zapojenie tlačítka a LED diódy na kite *Nucleo-64*
```





    PC13 - Button           nemapuje sa do Arduina
    PA5  - Green LED        (STM32) PA5 -> D13 (ARD)

**Tlačítko**

Nestlačené tlačítko má hodnotu 1, stlačené 0.

```Python
from pyb import Pin
p = Pin('PC13', Pin.IN)

# tlac hodnoty tlacitka 0/1
print(p.value())  

# definovanie prerusenia a callback funkcie
p.irq(lambda p:print(p.value()))
```

**Green LED**
```Python
from pyb import Pin
p = Pin('D13', Pin.OUT)
p.value(True)
p.value(False) 
```

## <font color='teal'> Úlohy </font>
