
# Java Application Performance and Memory Management

# Java flags

-XX:+ plus zapína flag
-XX:- mínus vypíná flag

#### -XX:+PrintCompilation
Vytiskne compilované části kódu

| čas od startu v ms | pořadí kompilace | příznaky | compilation level | kompilátor a optimalizace (compile kind) |
| :--- | :--- | --- | --- | --- |
| 57 | 53 | %  | 4 | PrimeNumbers::isPrime @ 2 (35 bytes) |


příznaky:
s - synchronized
n - native
% - compiled to native added to code cache
... 


Java má 2 compilery. C1 a C2

C1 kompiluje do Native 1,2,3
C2 kompiluje do Native 4 a vkládá do code cache (some times)

Ve 32 bitovém prostředí se používá jen C1 kompiler. (říká se mu client compiler)
Ve 64 bitovém prostředí se používají C1 i C2 kompiler. Kompileru C2 se říká server compiler).

Pokud jsme nuceni používat 64 bit jvm, mužeme se vynutit použití pouze client kompilátoru flagem -client. Tento flag je na některých OS ignorován (vypadá to na min. Win10). Zrychluje se startup time.

Flagy:
* -client
* -server
* -d64

Bohužel je potřeba zjistit, jestli na dané platformě dané flagy fungují :confounded:


#### -XX:+UnlockDiagnosticVMOptions -XX:+LogCompilation (musí jít dohromady)
Podobné jako PrintCompilation ale výpis je podrobnější a do souboru.

Code cache má omezenou velikost. Pokud uvidíme ` VM warning: Code cache is full. Compiler has been disabled. ` 

#### -XX:+PrintCodeCache {PrintCodeCache}

> Vytiskne informace o codeCache. Jak můžeme vidět, code cache je **120 MB !**

Nevím, jestli to náhodou není 2x 120 MB ...

``` 
CodeHeap 'non-profiled nmethods': size=120000Kb used=21Kb max_used=21Kb free=119979Kb
 bounds [0x0000017c5f800000, 0x0000017c5fa70000, 0x0000017c66d30000]
CodeHeap 'profiled nmethods': size=120000Kb used=94Kb max_used=94Kb free=119905Kb
 bounds [0x0000017c582d0000, 0x0000017c58540000, 0x0000017c5f800000]
CodeHeap 'non-nmethods': size=5760Kb used=1055Kb max_used=1072Kb free=4704Kb
 bounds [0x0000017c57d30000, 0x0000017c57fa0000, 0x0000017c582d0000]
 total_blobs=384 nmethods=75 adapters=222
 compilation: enabled
              stopped_count=0, restarted_count=0
 full_count=0
```

### Manipulace s code cache
#### -XX:InitialCodeCacheSize
#### -XX:ReservedCodeCacheSize
#### -XX:CodeCacheSizeExpansionSize

```
java -XX:ReservedCodeCacheSize=20k 
java -XX:ReservedCodeCacheSize=20m
java -XX:ReservedCodeCacheSize=20g
```

Otestovat flag můžeme takto:

` java -XX:ReservedCodeCacheSize=20m -XX:+PrintCodeCache java Main `

### Monitorování code cache vzdálené aplikace za použití jconsole

Java ukládá běžící procesy do adresáře hsperfdata_$USER. Pod windows ` C:\Users\user\AppData\Local\Temp\hsperfdata_user\ ` a někdy je potřeba nastavit write práva.

**Kurwa pod Win to zase nejede bo beztak zku... securita.**

Po připojení se jconsoli se code cache zvýší cca o 2 MB. Kompilují se věci kolem připojení s jconsole.

#### -XX:-TieredCompilation

Run in interpreted only ! Je vhodné používat pro rychlé skripty a "jednořádkové" programy.

### Native compilation tuning

Počet vláken použitých ke kompilaci závisí na počtu CPU na počítači.

jps
: Seznam všech běžících aplikaci na počítači.

java -XX:+PrintFlagsFinal
: Zjištění nastavení všech flagů jvm pro daný počítač.
Najdeme CICompilerCount, který zobrazuje počet threadů použitých pro kompilaci.

jinfo -flag CICompilerCount 1234
: Pokud chceme zjistit informaci o specifickém flagu, můžeme použít příkaz jinfo. 1234 je process id, který zjistíme z jps. Můžeme použít jen `jinfo 1234`, který nám vytiskne všechny informace o JVM.

#### -XX:CompileThreshold=n

Počet volání funkce, než bude zkompilována.

# The structure of Java's memory

Stack vs. heap memory.

:exclamation: **Každý thread má svůj stack.** :exclamation:

-XX:+PrintStringTableStatistics
: Vytiskne statistiky. Hledáme StringTable statistics:

`java -XX:+PrintStringTableStatistics Main`

String table je uložena v heap paměti.

```
StringTable statistics:
Number of buckets       :     65536 =    524288 bytes, each 8
Number of entries       :      1725 =     27600 bytes, each 16
Number of literals      :      1725 =    111744 bytes, avg  64.000
Total footprint         :           =    663632 bytes
Average bucket size     :     0.026
Variance of bucket size :     0.026
Std. dev. of bucket size:     0.162
Maximum bucket size     :         2
```
Je dobré monitorovat StringTable jestli není přeplněná

Po přidání 10 000 000 prvků. Vypadá to, že se se změní Number of buckets.
Program běžel cca. PT49.4068595S.

```
StringTable statistics:
Number of buckets       :    524288 =   4194304 bytes, each 8
Number of entries       :  10004770 = 160076320 bytes, each 16
Number of literals      :  10004770 = 480307624 bytes, avg  48.008
Total footprsize_t         :           = 644578248 bytes
Average bucket size     :    19.083
Variance of bucket size :    65.677
Std. dev. of bucket size:     8.104
Maximum bucket size     :        41
```
Vypadá to, že dochází k self tuning string table. Ale můžeme tomu pomoci nastavením flagu `-XX:StringTableSize=524231`. Číslo by podle kurzu mělo být ~~prvočíslo~~. Ale podle chování, si Adopt JDK stejně vypočítává své číslo pohybující se kolem 524231. Zjištění: *Adopt používá pro hodnotu Number of buckets násobky 2. Tzn. pokud dáme StringTableSize=524231 pak výsledná hodnota bude 1 048 576. Pokud dáme 1 048 577 pak výsledná hodnota bude 2 097 152.*

# Tuning the size of the heap
-XX:+UnlockDiagnosticVMOptions, -XX:+PrintFlagsFinal
: Zobrazí všechny informace o nastavení JVM, např.: InitialHeapSize,MaxHeapSize. Tyto hodnoty můžeme změnit.

##### Zkratky pro flagy
-XX:MaxHeapSize=512m (or -Xmx512m (nejsem si jist ...))
-XX:InitialHeapSize=32k (or -Xms32k)

# Garbage colections
Paměti která je určená pro GC označujeme jako "eligible for garbage collection". 

## Metoda finalize()
Od javy verze 9 je tato metoda označena jake deprecated a neměla by být používána !

# Section 12: Chapter 12 - Monitoring the heap

## Aplikace pro sledování heap paměti

jvisualvm 
: Aplikace od oracle. Pokud máme OpenJDK, musíme stáhnout. https://visualvm.github.io/


MAT Memory Analyzer (from eclipse)
: http://www.eclipse.org/mat/downloads.php

## Generating a heap dump

-XX:+HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath=someFile
: Flagy zajišťující vytvoření heap dumpu za běhu, kdykoliv dojde pamět.

# Chapter 15 - Garbage Collector tuning & selection

-verbose:gc
: Tisk informací o GC

-XX:-UseAdaptiveSizePolicy
: Vypnutí adaptivního přerozdělení heap paměti mezi Eden, S0 a S1. Většina VM by tento atribut měla mít zapnutý.

## Tunning GC - old and young allocation

-XX:NewRation
: Poměr mezi pamětí old and young (Eden).

## Tunning GC - survivor space allocation

-XX:SurvivorRatio
: Poměr mezi pamětí Eden a S0,S1.  VM může i tak dynamicky měnit poměr, pokud to uzná za vhodné.

## Tunning GC - generations needed to become old

-XX:MaxTenuringThreshold
: Počet přeživšich gc objektu, než se dostane objekt do old paměti. Max. hodnota by měla být 15. Některé VM umožnují hodnotu zvýšit nad 15, pak to ale znamená něco jako (infinity) a objekt nikdy nebude přesunut do části paměti "old", **což se rovná mrhání pamětí**. Není to dobrý nápad. 

## Selecting a garbage collector

Typy gc colektů
* Serial `-XX:+UseSerialGC` - max pro backgroud tasks
* Parallel `-XX:+UseParallelGC`
* Mostly Concurrent používají se 2 typy. Dostupné od javy 8.
`-XX:+UseConcMarkSweepGC` default in java 9
`-XX:+UseG1GC` defailt in java 10>

## Tunning the G1 garbage colector

Hlavní cíl programátorů G1 GC bylo, aby nebylo potřeba ve většině případů GC optimalizovat. Ale pár flagů existuje.

-XX:ConcGCThreads=n
: Užitečný (podle školitele), pokud chceme snížit počet vláken.

-XX:InitialitingHeapOccupancyPercent=n (def 45)
: Přečíst doc... Dobře jsem nerozuměl.

## String de-duplication

Od java8u20 a pouze pro G1 GC.

-XX:UseStringDeDuplication

# Chapter 16 - Using a profiler to analyse application performance

Unlock Java flight recorder

Pro Oracle JVM použijeme `-XX:+UnlockCommercialFeatures -XX:+FlightRecorder`
Pro OpenJDK použijeme pouze `-XX:+FlightRecorder`. Vypadá to, že tato hodnota je defaultně zapnuta v TemurinJDK.

**Příklad spuštění FR z příkazové řádky:**
`-XX:StartFlightRecording=delay=2min,duration=60s,name=Test,filename=recording.jfr,settings=profile`

#Benchmarking with JMH

https://github.com/openjdk/jmh

Více o jmh je potřeba proštudovat dokumentaci.

# Javap - java disasembler




___

# Příklady markdown

`http://www.example.com`

jslkdjfalksjd [^1]

[^1]: This is the first footnote.


```sql
select * from dual;
```

```xml
<head>
  <person name="Roman"/>
</head>
```


```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

jps
: list all running java processes

:joy:

- [x] ksdjfldks
- [ ] sdfsd 

terms
: definition

