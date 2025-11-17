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

# <font color='navy'> MicroPython </font>

S narastajúcim výkonom mikrokontrolérov sa v súčasnom období posúvajú možnosti ich využitia aj do oblastí, ktoré boli pred pár rokmi vyhradené výkonným počítačom alebo výpočtovým strediskám. Z relatívne jednoduchých čipov, často pozostávajúcich len z MCU a nevyhnutných periférií, primárne určených pre obsluhu a jednoduché spracovanie dát zo senzorov sa vyvinuli komplexné zariadenia obsahujúce priamo na čipe univerzálne komponenty potrebné pre tvorbu kompaktného a cenovo veľmi prijateľného systému riadenia, zberu, spracovania a transportu dát zo senzorov. S vývojom technických prostriedkov narastá aj komplexnosť programového vybavenie, na ktoré sú kladené vysoké požiadavky na jeho kvalitu a stabilitu, zariadenia často musia pracovať v ťažkých a komplikovaných podmienkach, bez možnosti údržby počas celej doby ich životnosti. 

Vývoj typickej aplikácie pozostáva z dvoch častí

* Vývoj spojený s obsluhou technických prostriedkov - vstupných zariadení a senzorov, zobrazovacích a signalizačných prvkov a výstupných zariadení. Tento môže mať niekoľko úrovní - od jednoduchého pripojenia štandardného prvku ku normalizovanému rozhraniu (I2C, SPI, ... ) a tvorbe príslušnoho API pre jeho obsluhu až po vývoj vlastného špecializovaného senzora alebo výstupného zariadenia s firmware, komunikačným rozhraním, implementáciou a spracovaním dát a unikátnym API.  

* Vývoj spojený s vlastnou aplikáciou - od jednoduchej komunikačnej aplikácie až po rozsiahly systém komunikujúci cez Internet. Pretože individuálny vývoj od základu je náročný a zdĺhavý, pre implementácie IoT aplikácií bolo v poslednom období vytvorených množstvo operačných systémov, typickým predstaviteľom je projekt [Zephyr](https://www.zephyrproject.org/), ktorý je portovaný na viacej ako 800 platforiem, pričom pre každú platformu má implementovanú obsluhu základných periférií. Súčasťou je mikrokernel alebo nanokoernel v závislosti od výkonu zariadení, podporuje multithreading a non-preemptive a preemptive plánovač (scheduling), programovacím jazykom je C a C++. 

Vývoj programového vybavenia je zvyčajne realizovaný v programovacích jazykoch umožňujúcich jednoduchý prístup k perifériím mikrokontroléra, ako je `C`, `C++`, `Assembler`. Použitie týchto jazukov garantuje vysokú efektivitu programového vybavenia, na druhej strane vyžaduje špeciálne prostriedky podporované samotným mikrokontrolérom pre ladenie kódu a hladanie chýb.

Je zrejmé, že vývoj aplikácie vyžaduje tvorbu a používanie vývojových nástrojov, ktorých príprava a odladenie môže v závislosti od rozsahu trvať nezanedbateľnú dobu. Jednou z možností, ako túto etapu zjednodušiť, je použitie flexibilného univerzálneho nástroja, umožnujúceho interaktívnu prácu a testovanie jednotlivých častí zariadenia. 

`Python` je všeobecne pokladaný za univerzálny programovací jazyk vysokej úrovne ďaleko vzdialený od hardware, `MicropPython` je prepisom referenčnej implementácie CPython jazyka Python pre mikrokontroléry s knižnicami pre prístup k jeho integrovaným perifériám. Implementácia je portovaná pre rôzne cielové platformy, je škálovateľná a open-source. O jeho popularite svedčí aj to, že na githube v súčasnej dobe existuje asi 2000 rôznych vetiev (fork) s rôznymi modifikáciami a úpravami pre rôzne vývojové a experimentálne dosky. Z aplikačného hľadiska je použitie `MicroPython` jednoduché, ako firmware sa nahrá štandardným programovacím software pre konkrétny typ mikrokontroléra do jeho flash pamäte a zvyčajne prostredníctvom emulácie sériového rozhrania komunikuje s terminálovou aplikáciu v počítači. 

Oblasť použitia `MicroPython` a jeho klonov môžeme rozdeliť do niekokých kategórií

* Výuka a vzdelávanie, `MicroPython` poskytuje možnosť interaktívnej komunikácie s mikrokontrolérom v slučke **REPL** (Read–Eval–Print Loop). S priamym prístupom k perifériám mikrokotroléra cez akýkoľvek terminálový emulátor bez potreby písania a vysvetlovania množstva kódu potrebného na inicializáciu a elementárnu komunikáciu je možné na veľkom množstve podporovaných dosák vysvetliť študentom základné princípy zberu a spracovania dát, naviac v jednoduchom programovacom jazyku.

* Vývoj a testovanie periférií a senzorov. `MicroPython` poskytuje odladené a vyskúšané referenčné implementácie rozhraní mikrokontrolérov, čím vývojára zbavuje potreby implementácie celej vertikálnej štruktúry spojenej s komunikáciou a riadením periférneho zariadenia. Moderné integrované periférie komunikujúce cez sériové rozhrania (I2C, SPI, CAN ...) sú riadené zápisom a čítaním hodnôt často z desiatok rôznych registrov, s individuálnym významom jednotlivých bitov a vzájomným previazaním hodnôt. Priamym interaktívnym prístupom v jednoduchom a populárnom jazyku k registrom zariadenia je možné jednoducho overiť funkciu periférií, vyvinúť a odladiť príslušný hardware a algoritmy pre riadenie a zber dát zo zariadenia. Vďaka abstrakcii hardware a univerzálnosti implementácie je možné pre vývoj využiť aj iné platformy, ako bude cieľová a to bez potreby detailnej znalosti jej programovania.

* Monitorovací a konfiguračný nástroj pre komplexné aplikácie. Rozsiahle aplikácie na výkonných mikrokontroléroch a FPGA obsahujú často implementáciu nezávislých pomocných prostriedkov pre monitorovanie a nastavovanie parametrov systému, pri FPGA to býva často SW implementácia niektorého z malých mikroprocesorov (8051, Z80 ...), pomocou programu pre tieto mikroprocesory je možné sledovať a nastavovať parametre a konfiguráciu systému.  Prvé pokusy s implementáciou MicroPython-u na FPGA ukazujú perspektívu daľšieho vývoja. V oblasti operačných systémov existuje experimentálny port `MicroPython` pre monitorovanie a nastavovanie parametrov kernelu Linuxu.   

## <font color='teal'> STM32 </font>

Rodina mikrokontrolérov triedy STM32 zahŕňa triedu populárnych mikrokontrolérov obsahujúcich široké spektrum typov líšiacich sa výkonom, pamäťou perifériami a možnosťami optimalizácie spotreby. Vývoj aplikácií je podporovaný firmou STM pomocou dostupných kitov *Nucleo*, ktoré obsahujú okrem samotného mikrokontroléra aj pomocné obvody a programátor. V distribúcii MicroPython nájdeme podporu pre niektoré typy kitov *Nucleo*, od jednoduchých až po výkonné typy.

```{figure} ./img/nucleo64.png
:width: 400px
:name: cm_010

Vývojový kit *Nucleo-64*
```
Konektory na doskách *Nucleo64* umožňujú použitie vývojových modulov z platformy *Arduino*. Verzia *Nucleo32* je pinovo kompatibilná s platformou *Arduino Nano*.


### <font color='brown'> Prenositeľnosť kódu </font>

Vďaka intenzívnemu využívaniu technológie HAL (Hardware Abstraction Layer) umožnuje pre `MicroPython` prenositeľnosť kódu nielen medzi mikrokontrolérmi v rámci jednej platformy, ale aj naprieč celým spektrom zariadení, na ktorých je portovaný. V nasledujúcom jednoduchom programe sú načítané dáta z teplomera a teplotného komparátora LM92 [6] pripojeného na zbernicu I2C číslo jedna. Dáta sú načítané ako 2 Byte z TEMPERATURE REGISTER na adrese čislo 0, čip obsahuje 7 registrov pre čítanie dát a nastavenie prahových hodnôt komparátorov a hysterézy. 

```Python
import machine

def read_LM92_TR(ic, addr):
    raw = ic.readfrom_mem(addr, 0, 2)  # nacitanie 2 byte z registra 0 
    data = (raw[0] << 8) + raw[1]      # konverzia 16 bit format
    td = data >> 3                     # 2'compl b15-b3 temperature
    TEMP = (-(td & 0x1000) | (td & 0xFFF))* 0.0625       
    L = data & 0x01                    # T_LOW  -> H, TEMP < 10 deg
    H = (data & 0x02) >> 1             # T_HIGH -> H  TEMP > 64 deg
    C = (data & 0x04) >> 2             # T_CRIT -> H  TEMP > 80 deg
    return TEMP, C, H, L
                                       # MCU STM32L432KC
ic=machine.I2C(1)                      # init I2C interface, PA10-SDA, PA9-SCL
print(ic.scan())                       # -> [75]  list of all devices in I2C(1)
print(read_LM92_TR(ic, 75))            # -> (24.5625, 0, 0, 0)
```

Uvedený program bude plne funkčný nielen na všetkých podporovaných mikrokontroléroch z triedy STM32, ale aj na ktomkoľvek zariadení, na ktoré je portovaný `MikroPython` a ktoré na palube obsahuje I2C rozhranie. Ak sú preto na vývoj periférneho zariadenia kladené vyššie požiadavky, môžeme samotný vývoj realizovať na procesore vyššej triedy a pre finálnu aplikáciu využiť jednoduchší procesor.

### <font color='brown'> Modularita </font>

Aby nebolo potrebné po resete mikrokontroléra opakovane nahrávať časti odladeného kódu do prostredia MicroPython-u, tento umožňuje pridanie nového kódu (frozen module) ako knižnice, ktorá sa stane súčasťou firmware. Celý postup je veľmi jednoduchý a spočíva v uložení súboru s kódom knižnice v Pythone do adresáru *./modules* a následnom skompilovaní firmware a jeho nahratí do mikrokontroléra. Knižnica je dostupná pomocou štandardneho príkazu *import*.

Pri mikrokontroléroch s väčšou FLASH pamäťou je možné jej voľnú časť využiť ako pamäťové médium mapované ako file systém. Knižnica *os* poskutuje základné funkcie pre vytváranie adresárov a manipuláciu so súbormi. File systém je dostupný aj mimo prostredia MicroPython-u, pomocou utility *pyboard* je možné do neho ukladať, mazať súbory a vyvárať adresárovú štruktúru. 

### <font color='brown'> Flexibilita vývojovej platformy a škálovateľnosť </font>

Elementárna verzia MikroPython-u bez knižníc má po skompilovaní veľkosť asi 20KB, pri konfigurácii pre jednotlivé platformy sa ale tvorcovia smažili optimálne využiť flash pamäte mikrokontrolérov doplnením čo najväčšieho množstva štandardných knižníc a podporou periférií. V prípade, že mikrokontrolér obsahuje väčšiu pamäť FLASH a RAM, je súčasťou firmware možnosť mapovania pamäte ako súborového systému, do ktorého je možné zaradiť aj externé pamäťové média ako je SD karta a pod. V prípade, že pri vývoji nie sú niektoré knižnice a drivery potrebné, je možné konfiguráciu firmware upraviť podľa aktuálnej potreby. MicroPython pre konfigurácou cieľového firmware pre každú platformu používa súbor *mpconfigboard.h*. Tento obsahuje premenné, pomocou ktorých vieme zaradiť alebo vyradiť z kompilácie firmware zvolené časti zdrojového kódu.

### <font color='brown'> Rozširiteľnosť </font>

Pri vývoji neštandardných periférií nemusí dostupná podpora periférií, ktoré sú súčasťou MicroPython-u, vyhovovať a môže vzniknúť požiadavka na nízkoúrovňovú obsluhu periférie. Podobne ako pri štandardnom Pythone, je možné aj MocroPython rozširovať o natívne moduly napísané v C/C++ a ktoré môžu využívať aj systémové knižnice pre obsluhu periférií mikrokontroléra. Pre vytvorenie modulu musíme rovnako ako v štandardnom Pythone vytvoriť rozhranie medzi natívnym modulom a jeho reprezentáciou v Pythone. Pre štandardný Python existujú generátory kódu, napr. SWIG, ktoré vygenerujú potrebné rozhranie na základe zdrojového kódu modulu. Pri MicroPython-e sa používa opačný postup, môžeme použiť generátor rozhrania (stub), ktorý vytvorí stub pozostávajúci z makier v C na základe deklarácie funkcie v Pythone.
