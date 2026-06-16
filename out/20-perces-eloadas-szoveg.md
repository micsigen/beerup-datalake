# 20 perces előadói szöveg

**Prezentáció:** `data-lake-wo-data-platform.pptx`  
**Téma:** Data Lake adatplatform nélkül - valós idejű aggregáció 1 500 üzenet/mp sebességnél  
**Időkeret:** 20 perc előadás + 5 perc Q&A

> Javasolt használat: ne szó szerint felolvasott szövegként, hanem vezetett speaker scriptként. A ritmus akkor működik jól, ha a technikai részeknél hagysz fél-egy mondatnyi szünetet a diagramok értelmezésére.

---

## Slide 1 - Cím

**Idő:** 0:45

Sziasztok, Balázs vagyok, és ma arról fogok beszélni, hogyan lehet Data Lake-szerű működést megvalósítani klasszikus adatplatform nélkül.

A cím szándékosan kicsit provokatív: *Data Lake adatplatform nélkül*. Nem azt jelenti, hogy nincs adatarchitektúra, vagy hogy nincs tudatos tervezés. Inkább azt jelenti, hogy nem mindig Spark, Flink, Hadoop, nagy compute cluster és külön data engineering platform az első reális válasz.

A konkrét sztori egy telekommunikációs adatfolyamról szól: körülbelül 1 500 üzenet másodpercenként, folyamatosan érkező eszköztelemetria, közel valós idejű aggregációs igényekkel. A fókusz nem egy tankönyvi ideális rendszer, hanem egy olyan megoldás, ami éles környezetben, korlátozott infrastruktúrán működött.

---

## Slide 2 - Hook: Data Lake viselkedés platform nélkül

**Idő:** 1:00

A fő kérdés ez: mi történik akkor, ha Data Lake viselkedésre van szükségünk, de nincs mögötte klasszikus Data Lake platform?

Data Lake alatt itt nem elsősorban egy konkrét terméket értek. Hanem azt a működést, hogy nyers adatot gyűjtünk, nem dobjuk el túl korán, és később többféle szempont szerint tudjuk feldolgozni. Ez nagyon hasznos akkor, amikor az üzleti kérdések változnak, vagy amikor egy hibaminta csak később válik érthetővé.

Csakhogy a valóságban sokszor nincs külön csapat, külön cluster, külön platform, és nincs végtelen mennyiségű üzemeltetési kapacitás. Itt is ez volt a helyzet: Kafka-ból jött az adat, MongoDB volt kéznél, és ebből kellett olyan rendszert építeni, ami nem csak demóban, hanem terhelés alatt is megbízható.

---

## Slide 3 - About Me

**Idő:** 1:10

Röviden rólam, hogy legyen kontextus, honnan nézem ezt a problémát.

Szoftvermérnökként főleg elosztott rendszerekkel és nagy áteresztőképességű backend architektúrákkal foglalkozom. Ezeknél a rendszereknél az érdekes problémák általában nem ott kezdődnek, hogy hogyan írunk meg egy algoritmust, hanem ott, hogy mi történik, amikor az algoritmus folyamatos adatfolyamon, hibákkal, késleltetéssel és üzemeltetési korlátokkal találkozik.

Engineering managerként az is fontos szempont lett számomra, hogy egy megoldás ne csak technikailag legyen elegáns, hanem fenntartható is legyen egy csapat számára. Ha minden incidenshez két órányi detektívmunka kell, akkor az architektúra valójában nem kész.

A konkrét domain pedig telekommunikáció: otthoni eszközök, hálózati topológiák, jelminőség, sok-sok kis adatpont, amelyek önmagukban nem feltétlenül izgalmasak, de időben és tömegben már nagyon is azok.

---

## Slide 4 - Business Requirement

**Idő:** 1:40

Nézzük az üzleti problémát.

Telekommunikációs home gateway eszközökről érkezik telemetria. A home gateway itt az otthoni végberendezés: az optikai jel lakáson belüli terminálási eszköze, amely rendszeresen küld adatot jelminőségről, konfigurációról, hálózati kapcsolatokról és működési metrikákról. A cél az volt, hogy ezekből közel valós időben tudjunk aggregált képet adni.

A nagyságrend itt a lényeg. Sok eszköz, folyamatos adatérkezés, nagyjából 1 500 üzenet másodpercenként. Emellett az adatoknak történeti jelentése is van. Nem elég azt tudni, hogy egy eszköznek most rossz a jelszintje. Az üzleti kérdés sokkal inkább az: rossz volt-e tartósan? Két napja is ilyen volt? Ugyanez látszott a topológiában? Ez lokális hiba, vagy hálózati mintázat?

Ezért van szükség nyers adat megtartására és hosszabb időablakban való gondolkodásra. A 48 órás mintadetektálási ablak azt jelenti, hogy az aggregáció nem lehet pusztán pillanatkép. Időbeli viselkedést kell tudnunk értelmezni.

És mindezt úgy, hogy nincs külön compute infrastruktúra, nincs dedikált data platform, hanem a rendelkezésre álló rendszerrel kell dolgozni.

---

## Slide 5 - Topológia / üzleti valóság

**Idő:** 0:55

Ezen a képen az látszik, hogy az adat nem absztrakt sorokból áll egy táblában. Valódi hálózati kapcsolatok, eszközök és topológiák vannak mögötte.

Ez azért fontos, mert a felhasználó nem azt akarja látni, hogy feldolgoztunk X millió dokumentumot. A felhasználó azt akarja tudni, hogy hol van a probléma a hálózatban. Egyetlen eszköz hibásodik? Egy szomszédos csoportban romlik a jel? Egy topológiai változás után jelent meg a hiba?

Itt válik fontossá a Data Lake szemlélet. Ha csak előre definiált aggregátumokat tartunk meg, akkor könnyen elveszítjük azt a részletet, amire később szükség lenne. Ha viszont mindent nyersen tartunk meg túl sokáig, akkor az infrastruktúra törik meg. A rendszernek e kettő között kellett egy működő kompromisszumot találnia.

---

## Slide 6 - Signal level / konkrét üzleti kérdés

**Idő:** 0:55

Itt egy egyszerűbb, de nagyon tipikus példa: signal level, vagyis jelszint.

Egyetlen mérés alapján nem feltétlenül tudjuk, hogy baj van. Lehet átmeneti zaj, lehet mérési késés, lehet olyan állapot, ami önmagában még nem igényel beavatkozást. Az érdekes kérdés az időbeli mintázat: tartósan alacsony-e a jel? Mióta ilyen? Ugyanez történik-e más eszközöknél is?

Ezért az aggregáció nem csak teljesítménykérdés, hanem helyességi kérdés is. Ha késik az aggregáció, a felhasználó régi állapotot lát. Ha rossz időbélyeg alapján aggregálunk, akkor hibás következtetést vonhatunk le. Ha nem látjuk a rendszer késését, akkor nem is tudjuk, hogy a válaszunk friss-e.

Innen indulnak az első próbálkozások.

---

## Slide 7 - First Try: Spark

**Idő:** 1:20

Mielőtt belemegyünk az első próbálkozásba, egy gyors kérdés a teremnek: ki használ AI-t kódírásra? És ki az, aki még nagyrészt kézzel írja a kódjait?

Ezt azért kérdezem, mert ma implementációt nagyon gyorsan lehet építeni. De ha az architekturális irány rossz, az AI nem feltétlenül ment meg - sokszor csak gyorsabban visz el ugyanabba a zsákutcába.

Az első megközelítés technikailag teljesen védhető volt: használjunk Sparkot.

Kafka-ból jön az adat, Spark feldolgozza és aggregálja, MongoDB-ben pedig eltároljuk az eredményt. Papíron ez nagyon szépen néz ki. A Spark jól ismert eszköz nagy adatmennyiség feldolgozására, SQL-ben jól kifejezhetőek az aggregációk, és a distributed processing gondolata elsőre megnyugtató.

Éles környezetben viszont ez nálunk nem működött jól.

Az első probléma az üzemeltetési overhead volt. Memória, executorok, shuffle partíciók, resource allocation - ezek mind olyan paraméterek, amelyeket érteni és folyamatosan gondozni kell. Ha van erre platform és csapat, rendben. Ha nincs, akkor a Spark nem csak megoldás, hanem plusz rendszer, amit üzemeltetni kell.

A második probléma a resource contention volt. Ugyanazon az infrastruktúrán futott volna, ahol az alkalmazás többi része is. Terhelés alatt ez kiszámíthatatlan viselkedést eredményezett.

De a legfontosabb tanulság: nem láttuk eléggé, mi történik. Nem volt rendes monitoring, nem volt dashboard, nem volt egyszerű válasz arra, hogy le vagyunk-e maradva.

---

## Slide 8 - First Try: Scheduled Aggregation Jobs

**Idő:** 1:30

A második próbálkozás sokkal egyszerűbbnek tűnt.

Tegyük be a nyers adatot MongoDB-be, és futtassunk időzített aggregációs jobokat. Például percenként végigolvassuk az új adatokat, kiszámoljuk az aggregátumokat, és kiírjuk az eredményt.

Ez elsőre sokkal barátságosabb megoldás. Nincs Spark, nincs külön cluster, könnyebb debuggolni, minden ismerős technológiából áll. És kisebb terhelésnél tényleg működik is.

A gond ott kezdődik, hogy itt nem csak a frissen beérkező egy percnyi adatot kellett megmozgatni. A 48 órás üzleti ablak miatt két napnyi történeti adatra kellett visszanézni, ezért a feldolgozásnak nem elég az ingesttel azonos tempót hoznia: annál sokkal gyorsabbnak kell lennie.

Ha 1 500 üzenet/mp jön be, de minden futás nagy történeti ablakot olvas és aggregál, a job már eleve nagyobb munkát végez, mint amennyi az új adat mennyisége. Ha egy futás 35 másodpercig tart, mire befejeződik, már 35 másodpercnyi új adat vár, miközben a következő kör megint visszanéz a történeti ablakra.

Ez a scheduled job megközelítés egyik csapdája: egyszerűnek látszik, de folyamatos adatfolyamnál könnyen pull-alapú polling rendszerré válik, amely mindig a múltat próbálja behozni.

És itt ugyanaz volt a gyökérprobléma, mint a Spark esetén: nem volt elég láthatóság. A késést akkor vettük észre igazán, amikor a felhasználók már stale adatokat láttak.

---

## Slide 9 - Az első próbák közös tanulsága

**Idő:** 0:45

Ez a dia a két zsákutca közös tanulságát foglalja össze.

Nem az volt a fő baj, hogy Spark rossz technológia. Nem is az, hogy scheduled jobot soha nem szabad használni. Mindkettőnek megvan a helye.

A baj az volt, hogy a választott megközelítés nem illett jól a konkrét korlátainkhoz, és közben nem volt elég monitoring ahhoz, hogy gyorsan lássuk a rendszer valódi állapotát.

Egy adatfolyam-alapú problémát pull-alapú módon próbáltunk kezelni. Innen jött a fordulat: ne időnként kérdezzük meg az adatbázistól, hogy mi változott, hanem reagáljunk az eseményre akkor, amikor megtörténik.

---

## Slide 10 - What's in the Box?

**Idő:** 1:10

Itt látszik a végül használt stack.

Kafka adja az üzenetfolyamot. MongoDB tárolja a nyers adatot és az aggregált eredményeket. A Change Streams adja azt az eseményvezérelt réteget, amelyen keresztül az insert eseményeket valós időben meg tudjuk figyelni. Node.js-ben fut az alkalmazáslogika és a Kafka olvasás. Prometheus és Grafana adja az observability réteget. Kubernetes pedig az orchestration környezet.

Fontos, hogy itt nincsen Spark, nincsen Flink, nincsen Hadoop. Ez nem technológiaellenes állítás. Inkább arról szól, hogy a megoldást a rendelkezésre álló üzemeltetési modellhez kell igazítani.

Ha egy technológia használatához több infrastruktúra, több szakértelem és több üzemeltetési figyelem kell, mint amennyink van, akkor könnyen az történik, hogy a technológia elvileg skálázható, a rendszerünk viszont gyakorlatilag törékeny lesz.

---

## Slide 11 - Solution

**Idő:** 2:00

A végső megoldás lényege: pull helyett push, polling helyett eseményvezérelt feldolgozás.

A Kafka reader batch-ekben írja be a nyers dokumentumokat MongoDB-be. Ez továbbra is megtartja a Data Lake-szerű viselkedést: a raw adat elérhető marad egy ideig, és nem csak az előre kiszámolt nézetek léteznek.

Ezután a MongoDB Change Stream figyeli az insert eseményeket. Amikor új dokumentum kerül a raw collectionbe, a Change Stream Processor megkapja ezt az eseményt. Nem kell percenként újraolvasni a collectiont, nem kell keresni, mi változott. Az adatbázis eseményként adja tovább a változást.

A processzor nem minden dokumentumra külön ír aggregátumot. Mikro-batch ablakot használ, jellemzően 200-500 milliszekundum körül. Ez azért fontos, mert így drasztikusan csökkentjük a MongoDB round-tripek számát. Sok kis update helyett bulk write műveleteket tudunk használni, `$inc` és upsert logikával.

A resume token adja a crash-safe működést. Ha a processzor újraindul, tudja, honnan kell folytatni a Change Stream olvasását. Ez kulcskérdés, mert stream processingnél az újraindítás nem lehet adatvesztési pont.

Szándékosan egyetlen processor instance dolgozik az aggregáción. Ez limitációnak hangozhat, de itt előny volt: nincs distributed consensus, nincs shardolt állapot, nincs lockolási bonyolultság. A mért kapacitás így is bőven az aktuális terhelés fölött volt: több mint 6 000 üzenet másodpercenként, nagyjából 1-2 másodperces end-to-end késéssel.

A scheduled jobok nem tűntek el teljesen. Csak más szerepet kaptak: backfill, cleanup, komplex újraszámítás. Nem ők viszik a real-time fő útvonalat.

---

## Slide 12 - Monitoring overview

**Idő:** 1:20

Itt jön a másik nagy tanulság: a monitoring nem kiegészítő funkció. A monitoring az architektúra része.

Ha egy pipeline bármelyik eleménél nincs mérésünk, akkor ott vakfoltunk van. És egy adatfolyam-rendszerben a vakfolt nagyon gyorsan üzleti problémává válik. Nem azt fogjuk látni, hogy "kicsit romlott egy metrika", hanem azt, hogy a felhasználó már nem friss adatot lát, vagy a raw collection túl gyorsan nő, vagy egy processzor csendben lemarad.

Ezért minden pipeline elemnél kell legalább két típusú jel: mennyi adat megy át rajta, és mekkora a késés. Kafka oldalon messages per second és consumer lag. Change Stream oldalon event lag, buffer size, flush frequency, bulk write latency. MongoDB oldalon collection size, TTL delete rate, insert rate.

Az alerting ugyanilyen fontos. A dashboard segít vizsgálódni, de az alert mondja meg, hogy mikor kell egyáltalán odanézni. A cél az, hogy ne a felhasználó legyen az első monitoring rendszerünk.

---

## Slide 13 - Kafka throughput

**Idő:** 0:45

Kafka oldalon az első kérdés egyszerű: folyik-e be adat olyan ütemben, ahogy várjuk?

Az events per second vagy messages per second nem csak forgalmi adat. Ez a pipeline pulzusa. Ha hirtelen leesik, lehet producer probléma. Ha hirtelen megugrik, lehet valós terhelési csúcs, de lehet hibás upstream viselkedés is.

Ezt azért kell a pipeline elején mérni, mert minden későbbi metrika értelmezése ettől függ. Ha az aggregáció keveset dolgozik, az lehet jó hír is, meg rossz hír is. Csak akkor tudjuk eldönteni, ha látjuk, mi érkezik be.

---

## Slide 14 - Kafka reading / consumer lag

**Idő:** 0:45

A második Kafka metrika a consumer lag.

Ez azt mutatja meg, hogy a consumer mennyire van lemaradva a topic legfrissebb offsetjéhez képest. Folyamatos adatfolyamnál ez kritikus jel. Lehet, hogy a rendszer még dolgozik, lehet, hogy a CPU nem magas, lehet, hogy nincsenek hibák a logban - de ha a lag nő, akkor a felhasználói valóság egyre távolodik a tényleges állapottól.

Itt már nem csak throughputról beszélünk, hanem frissességről. A near real-time rendszert nem az teszi real-time jellegűvé, hogy gyors technológiák vannak benne, hanem hogy mérjük és kontrolláljuk a késést.

---

## Slide 15 - MongoDB Change Stream

**Idő:** 1:00

A Change Stream a megoldás központi eleme.

Ez az a pont, ahol az adatbázisban történő insert eseményből feldolgozási esemény lesz. Ahelyett, hogy időnként végigszkennelnénk az új dokumentumokat, feliratkozunk a változásokra, és folyamatosan dolgozzuk fel őket.

Itt két dologra érdemes figyelni. Az egyik, hogy a Change Stream nem varázslat: ugyanúgy lehet lemaradása, ugyanúgy lehet processzor oldali torlódás, és ugyanúgy kell mérni. A másik, hogy a feldolgozás helyességét az event-time szemlélet adja. Nem az számít, mikor dolgoztuk fel az eseményt, hanem hogy az esemény üzletileg melyik időpontra vonatkozik.

Ez különösen fontos történeti adatoknál, ahol a device által küldött időbélyeg és a feldolgozás ideje eltérhet.

---

## Slide 16 - Change Stream lag and buffer

**Idő:** 1:00

Ezen a dián két kulcsjel látszik: Change Stream lag és buffer size.

A Change Stream lag azt mutatja, mennyi idő telik el a raw insert és az aggregált írás között. Ez gyakorlatilag a real-time útvonal késése. Ha ez nő, akkor lehet, hogy az input nőtt meg, lehet, hogy a bulk write lassult be, vagy lehet, hogy a processzor nem flushol elég gyakran.

A buffer size a mikro-batch működés egészségét mutatja. A buffer hasznos, mert csökkenti az adatbázis köröket. De ha túl nagyra nő, akkor már nem optimalizáció, hanem torlódás.

Ezért kell együtt nézni a batch size-t, flush frequency-t és bulk write latency-t. Egyetlen metrika önmagában félrevezető lehet. A pipeline viselkedése a metrikák kapcsolatából derül ki.

---

## Slide 17 - TTL

**Idő:** 0:50

A raw adat megtartása nem ingyen van. Ha másodpercenként 1 500 dokumentum érkezik, akkor a retention nagyon gyorsan tárhely- és teljesítménykérdéssé válik.

MongoDB-ben a TTL index kényelmes megoldásnak tűnik: lejár az adat, az adatbázis takarít. De nagy beérkező volumen mellett nem elég azt feltételezni, hogy a háttérfolyamat majd biztosan lépést tart.

Itt is ugyanaz a kérdés: az insert rate és a delete rate egyensúlyban van-e? Ha több adat érkezik, mint amennyit a TTL takarítás el tud vinni, akkor a collection nőni fog. Először csak tárhelyben, aztán indexméretben, aztán teljesítményben is.

---

## Slide 18 - TTL broken

**Idő:** 0:55

Ez a "broken" TTL állapot tanulsága.

Amikor a takarítás nem tart lépést a beérkező adatmennyiséggel, az nem azonnal látványos alkalmazáshiba. Sokáig úgy tűnhet, hogy minden megy tovább. A dokumentumok jönnek, a processzor dolgozik, a felhasználó talán még lát adatot.

Közben viszont a rendszer belül adósságot halmoz. Nő a collection, nő az index, romolhatnak az írási idők, és egy idő után az egész pipeline késésként vagy instabilitásként adja vissza ezt.

A fontos tanulság: a retention mechanizmusokat ugyanúgy monitorozni kell, mint az ingestet és az aggregációt. Nem elég azt mérni, hogy bejön-e az adat. Azt is mérni kell, hogy kontrolláltan ki tud-e menni a rendszerből.

---

## Slide 19 - Q&A / zárás

**Idő:** 0:15

Összefoglalva: nem az volt a cél, hogy "ne használjunk adatplatformot". Az volt a cél, hogy a problémához, a csapathoz és az infrastruktúrához illő architektúrát építsünk.

A három fő tanulság: stream jellegű problémánál eseményvezérelt feldolgozás kell; a mikro-batch egyszerű, de nagyon erős eszköz; és monitoring nélkül nincs megbízható real-time rendszer.

Köszönöm, és jöhet az 5 perc Q&A - akár most, akár később egy sör mellett.
