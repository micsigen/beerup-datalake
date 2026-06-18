# 20 perces előadói szöveg

**Prezentáció:** `data-lake-wo-data-platform.pptx`  
**Téma:** Data Lake adatplatform nélkül - valós idejű aggregáció 1 500 üzenet/mp sebességnél  
**Időkeret:** 18 perc előadás + 5 perc Q&A

> Javasolt használat: ne szó szerint felolvasott szövegként, hanem vezetett speaker scriptként. A ritmus akkor működik jól, ha a technikai részeknél hagysz fél-egy mondatnyi szünetet a diagramok értelmezésére.

---

## Slide 1 - Cím

**Idő:** 0:40
**Visszaszámláló:** 18:00 → 17:20

Sziasztok, Balázs vagyok, és ma arról fogok beszélni, hogyan lehet Data Lake-szerű működést megvalósítani klasszikus adatplatform nélkül.

A cím szándékosan kicsit provokatív: *Data Lake adatplatform nélkül*. Nem azt jelenti, hogy nincs adatarchitektúra, hanem azt, hogy nem mindig Spark, Flink, Hadoop és nagy adatplatform az első reális válasz.

A konkrét sztori egy telekommunikációs adatfolyamról szól: körülbelül 1 500 üzenet másodpercenként, folyamatosan érkező eszköztelemetria, közel valós idejű aggregációs igényekkel. A fókusz nem egy tankönyvi ideális rendszer, hanem egy olyan megoldás, ami éles környezetben, korlátozott infrastruktúrán működött.

---

## Slide 2 - Hook: Data Lake viselkedés platform nélkül

**Idő:** 0:50
**Visszaszámláló:** 17:20 → 16:30

A fő kérdés ez: mi történik akkor, ha Data Lake viselkedésre van szükségünk, de nincs mögötte klasszikus Data Lake platform?

Data Lake alatt itt nem egy konkrét terméket értek, hanem azt a működést, hogy nyers adatot gyűjtünk, nem dobjuk el túl korán, és később többféle szempont szerint tudjuk feldolgozni.

Csakhogy a valóságban sokszor nincs külön csapat, külön fürt, külön platform, és nincs végtelen mennyiségű üzemeltetési kapacitás. Itt is ez volt a helyzet: Kafka-ból jött az adat, MongoDB volt kéznél, és ebből kellett olyan rendszert építeni, ami nem csak demóban, hanem terhelés alatt is megbízható.

---

## Slide 3 - About Me

**Idő:** 1:00
**Visszaszámláló:** 16:30 → 15:30

Röviden rólam, hogy legyen kontextus, honnan nézem ezt a problémát.

Szoftvermérnökként főleg elosztott rendszerekkel és nagy áteresztőképességű backend architektúrákkal foglalkozom. Ezeknél a rendszereknél az érdekes problémák ott kezdődnek, amikor a kód folyamatos adatfolyammal, hibákkal, késleltetéssel és üzemeltetési korlátokkal találkozik.

Engineering managerként az is fontos szempont lett számomra, hogy egy megoldás ne csak technikailag legyen elegáns, hanem fenntartható is legyen egy csapat számára. Ha minden incidenshez két órányi detektívmunka kell, akkor az architektúra valójában nem kész.

A konkrét domain pedig telekommunikáció: otthoni eszközök, hálózati topológiák, jelminőség, sok-sok kis adatpont, amelyek önmagukban nem feltétlenül izgalmasak, de időben és tömegben már nagyon is azok.

---

## Slide 4 - Business Requirement

**Idő:** 1:35
**Visszaszámláló:** 15:30 → 13:55

Nézzük az üzleti problémát.

Telekommunikációs home gateway eszközökről érkezik telemetria. A home gateway itt az otthoni végberendezés: az optikai jel lakáson belüli terminálási eszköze, amely rendszeresen küld adatot jelminőségről, konfigurációról, hálózati kapcsolatokról és működési metrikákról.

A nagyságrend itt a lényeg. Sok eszköz, folyamatos adatérkezés, nagyjából 1 500 üzenet másodpercenként. Emellett az adatoknak történeti jelentése is van. Nem elég azt tudni, hogy egy eszköznek most rossz a jelszintje. Az üzleti kérdés sokkal inkább az: rossz volt-e tartósan? Két napja is ilyen volt? Ugyanez látszott a topológiában? Ez lokális hiba, vagy hálózati mintázat?

Ezért van szükség nyers adat megtartására és hosszabb időablakban való gondolkodásra. A 48 órás mintadetektálási ablak azt jelenti, hogy az aggregáció nem lehet pusztán pillanatkép. Időbeli viselkedést kell tudnunk értelmezni.

És mindezt úgy, hogy nincs külön számítási infrastruktúra, nincs dedikált adatplatform, hanem a rendelkezésre álló rendszerrel kell dolgozni.

---

## Slide 5 - Topológia / üzleti valóság

**Idő:** 0:45
**Visszaszámláló:** 13:55 → 13:10

Ezen a képen az látszik, hogy az adat mögött valódi hálózati kapcsolatok, eszközök és topológiák vannak.

Ez azért fontos, mert az operátor nem azt akarja látni, hogy feldolgoztunk X millió dokumentumot. Azt akarja tudni, hogy egyetlen eszközhöz tartozó hálózati topológia hogyan nézett ki korábban, és hol jelent meg a probléma.

Ezért fontos a Data Lake szemlélet: ha túl korán dobjuk el a részleteket, később nem tudunk új kérdésekre válaszolni. Ha viszont mindent nyersen tartunk meg túl sokáig, az infrastruktúra törik meg.

---

## Slide 6 - Signal level / konkrét üzleti kérdés

**Idő:** 0:45
**Visszaszámláló:** 13:10 → 12:25

Itt egy egyszerűbb, de nagyon tipikus példa: signal level, vagyis jelszint.

Egyetlen mérés alapján nem feltétlenül tudjuk, hogy baj van. Az érdekes kérdés az időbeli mintázat: tartósan alacsony-e a jel, mióta ilyen, és ugyanez történik-e más eszközöknél is?

Ezért az aggregáció nem csak teljesítménykérdés, hanem helyességi kérdés is. Egyéni hibabejelentésnél az operátor ez alapján tud beavatkozást végezni. Ha késik az aggregáció, régi állapotot lát; ha rossz időbélyeg alapján aggregálunk, hibás következtetést vonhatunk le.

Innen indulnak az első próbálkozások.

---

## Slide 7 - First Try: Spark

**Idő:** 1:00
**Visszaszámláló:** 12:25 → 11:25

Az első megközelítés technikailag teljesen védhető volt: használjunk Sparkot.

Kafka-ból jön az adat, Spark feldolgozza és aggregálja, MongoDB-ben pedig eltároljuk az eredményt. Papíron ez szépen néz ki: jól ismert eszköz, SQL-ben kifejezhető aggregációk, elosztott feldolgozás.

Éles környezetben viszont ez nálunk nem működött jól.

Az első probléma az üzemeltetési teher volt. Memória, végrehajtók, partíciók, erőforrás-allokáció - ezeket érteni és folyamatosan gondozni kell. Ha van erre platform és csapat, rendben. Ha nincs, akkor a Spark nem csak megoldás, hanem plusz rendszer.

A második probléma az erőforrás-versengés volt. Ugyanazon az infrastruktúrán futott volna, ahol az alkalmazás többi része is. Terhelés alatt ez kiszámíthatatlan viselkedést eredményezett.

De a legfontosabb tanulság: nem láttuk eléggé, mi történik. Nem volt rendes monitoring, nem volt dashboard, nem volt egyszerű válasz arra, hogy le vagyunk-e maradva.

---

## Slide 8 - First Try: Scheduled Aggregation Jobs

**Idő:** 1:20
**Visszaszámláló:** 11:25 → 10:05

A második próbálkozás sokkal egyszerűbbnek tűnt.

Tegyük be a nyers adatot MongoDB-be, és futtassunk időzített aggregációs jobokat. Például percenként végigolvassuk az új adatokat, kiszámoljuk az aggregátumokat, és kiírjuk az eredményt.

Ez elsőre barátságosabb megoldás: nincs Spark, nincs külön fürt, könnyebb hibát keresni, minden ismerős technológiából áll. Kisebb terhelésnél működik is.

A gond ott kezdődik, hogy itt nem csak a frissen beérkező egy percnyi adatot kellett megmozgatni. A 48 órás üzleti ablak miatt két napnyi történeti adatra kellett visszanézni, ezért a feldolgozásnak nem elég az adatbeérkezéssel azonos tempót hoznia: annál sokkal gyorsabbnak kell lennie.

Ha 1 500 üzenet/mp jön be, de minden futás nagy történeti ablakot olvas és aggregál, a job már eleve nagyobb munkát végez, mint amennyi az új adat mennyisége. Ha egy futás 35 másodpercig tart, mire befejeződik, már 35 másodpercnyi új adat vár, miközben a következő kör megint visszanéz a történeti ablakra.

Ez a scheduled job megközelítés csapdája: egyszerűnek látszik, de folyamatos adatfolyamnál könnyen olyan rendszerré válik, amely mindig a múltat próbálja behozni.

És itt ugyanaz volt a gyökérprobléma, mint a Spark esetén: nem volt elég láthatóság. A késést akkor vettük észre igazán, amikor a felhasználók már nem friss adatokat láttak.

---

## Slide 9 - Az első próbák közös tanulsága

**Idő:** 0:50
**Visszaszámláló:** 10:05 → 09:15

Ez a dia a két zsákutca közös tanulságát foglalja össze.

Nem az volt a fő baj, hogy Spark rossz technológia. Nem is az, hogy scheduled jobot soha nem szabad használni. Mindkettőnek megvan a helye.

A baj az volt, hogy a választott megközelítés nem illett jól a konkrét korlátainkhoz, és közben nem volt elég monitoring ahhoz, hogy gyorsan lássuk a rendszer valódi állapotát.

Itt érdemes egy gyors kérdést feltenni: ki használ AI-t kódírásra, és ki az, aki még nagyrészt kézzel írja a kódjait? Azért jó itt megkérdezni, mert ez a tanulság: ma gyorsan lehet implementálni, de ha az architekturális irány rossz, csak gyorsabban jutunk zsákutcába.

---

## Slide 10 - What's in the Box?

**Idő:** 1:00
**Visszaszámláló:** 09:15 → 08:15

Itt látszik a végül használt technológiai készlet.

Kafka adja az üzenetfolyamot. MongoDB tárolja a nyers adatot és az aggregált eredményeket. A Change Streams adja azt az eseményvezérelt réteget, amelyen keresztül a beszúrási eseményeket valós időben meg tudjuk figyelni. Node.js-ben fut az alkalmazáslogika és a Kafka olvasás. Prometheus és Grafana adja a megfigyelhetőségi réteget. Kubernetes pedig a futtatási környezet.

Fontos, hogy itt nincsen Spark, nincsen Flink, nincsen Hadoop. Nem azért, mert ezek rossz technológiák, hanem mert a megoldást a rendelkezésre álló üzemeltetési modellhez kellett igazítani.

---

## Slide 11 - Solution

**Idő:** 2:00
**Visszaszámláló:** 08:15 → 06:15

A végső megoldás lényege: időszakos lekérdezés helyett eseményvezérelt feldolgozás.

A Kafka-olvasó kötegekben írja be a nyers dokumentumokat MongoDB-be. Ez megtartja a Data Lake-szerű viselkedést: a nyers adat elérhető marad egy ideig, és nem csak az előre kiszámolt nézetek léteznek.

Ezután a MongoDB Change Stream figyeli a beszúrási eseményeket. Amikor új dokumentum kerül a nyers gyűjteménybe, a feldolgozó megkapja ezt az eseményt. Nem kell percenként újraolvasni a gyűjteményt, és nem kell keresni, mi változott.

A processzor nem minden dokumentumra külön ír aggregátumot. Mikro-köteg ablakot használ, jellemzően 200-500 milliszekundum körül. Ez azért fontos, mert így drasztikusan csökkentjük a MongoDB felé indított adatbázis-műveletek számát. Sok kis módosítás helyett tömeges írásokat tudunk használni, `$inc` és beszúrás-vagy-frissítés logikával.

A folytatási token adja az újraindításbiztos működést. Ha a processzor újraindul, tudja, honnan kell folytatni a Change Stream olvasását. Ez kulcskérdés, mert adatfolyam-feldolgozásnál az újraindítás nem lehet adatvesztési pont.

Szándékosan egyetlen feldolgozó példány dolgozik az aggregáción. Ez korlátnak hangozhat, de itt előny volt: nincs elosztott konszenzus, nincs szétszabdalt állapot, nincs zárolási bonyolultság. A mért kapacitás így is bőven az aktuális terhelés fölött volt: több mint 6 000 üzenet másodpercenként, nagyjából 1-2 másodperces végponttól végpontig mért késéssel.

A scheduled jobok nem tűntek el, csak más szerepet kaptak: visszatöltés, takarítás, komplex újraszámítás. Nem ők viszik a valós idejű fő útvonalat.

---

## Slide 12 - Monitoring overview

**Idő:** 1:20
**Visszaszámláló:** 06:15 → 04:55

Itt jön a másik nagy tanulság: a monitoring nem kiegészítő funkció. A monitoring az architektúra része.

Ha egy folyamat bármelyik eleménél nincs mérésünk, ott vakfoltunk van. Egy adatfolyam-rendszerben ez gyorsan üzleti problémává válik: a felhasználó nem friss adatot lát, a nyers gyűjtemény túl gyorsan nő, vagy egy feldolgozó csendben lemarad.

Ezért minden elemnél kell legalább két típusú jel: mennyi adat megy át rajta, és mekkora a késés. Kafka oldalon throughput és consumer lag. Change Stream oldalon event lag, buffer size, flush frequency és bulk write latency. MongoDB oldalon gyűjteményméret, TTL törlési sebesség és beszúrási sebesség.

Az alerting ugyanilyen fontos. A dashboard segít vizsgálódni, de az alert mondja meg, hogy mikor kell egyáltalán odanézni. A cél az, hogy ne a felhasználó legyen az első monitoring rendszerünk.

---

## Slide 13 - Kafka throughput

**Idő:** 0:40
**Visszaszámláló:** 04:55 → 04:15

Kafka oldalon az első kérdés egyszerű: folyik-e be adat olyan ütemben, ahogy várjuk?

Az üzenet/mp nem csak forgalmi adat. Ez a folyamat pulzusa. Ha hirtelen leesik, lehet termelő oldali probléma. Ha megugrik, lehet valós terhelési csúcs, de lehet hibás forrásoldali viselkedés is.

Ezt azért kell a folyamat elején mérni, mert minden későbbi metrika értelmezése ettől függ. Ha az aggregáció keveset dolgozik, az lehet jó hír is, meg rossz hír is. Csak akkor tudjuk eldönteni, ha látjuk, mi érkezik be.

---

## Slide 14 - Kafka reading / consumer lag

**Idő:** 0:40
**Visszaszámláló:** 04:15 → 03:35

A második Kafka metrika a consumer lag.

Ez azt mutatja meg, hogy a consumer mennyire van lemaradva a topic legfrissebb offsetjéhez képest. Folyamatos adatfolyamnál ez kritikus jel. Lehet, hogy a rendszer még dolgozik, lehet, hogy a CPU nem magas, lehet, hogy nincsenek hibák a naplóban - de ha a lag nő, akkor a felhasználói valóság egyre távolodik a tényleges állapottól.

Itt már nem csak throughputról beszélünk, hanem frissességről. A közel valós idejű rendszert nem az teszi gyorssá, hogy gyors technológiák vannak benne, hanem hogy mérjük és kontrolláljuk a késést.

---

## Slide 15 - MongoDB Change Stream

**Idő:** 0:50
**Visszaszámláló:** 03:35 → 02:45

A Change Stream a megoldás központi eleme.

Ez az a pont, ahol az adatbázisban történő beszúrásból feldolgozási esemény lesz. Ahelyett, hogy időnként végigszkennelnénk az új dokumentumokat, feliratkozunk a változásokra, és folyamatosan dolgozzuk fel őket.

Itt két dologra érdemes figyelni. Az egyik, hogy a Change Stream nem varázslat: ugyanúgy lehet lemaradása, ugyanúgy lehet processzor oldali torlódás, és ugyanúgy kell mérni. A másik, hogy a feldolgozás helyességét az event-time szemlélet adja. Nem az számít, mikor dolgoztuk fel az eseményt, hanem hogy az esemény üzletileg melyik időpontra vonatkozik.

Ez különösen fontos történeti adatoknál, ahol a device által küldött időbélyeg és a feldolgozás ideje eltérhet.

---

## Slide 16 - Change Stream lag and buffer

**Idő:** 0:50
**Visszaszámláló:** 02:45 → 01:55

Ezen a dián két kulcsjel látszik: Change Stream lag és buffer size.

A Change Stream lag azt mutatja, mennyi idő telik el a nyers beszúrás és az aggregált írás között. Ez gyakorlatilag a valós idejű útvonal késése. Ha nő, lehet, hogy a bemeneti adat nőtt meg, az adatbázis írás lassult be, vagy a feldolgozó nem ír ki elég gyakran.

A buffer size a micro-batch működés egészségét mutatja. A buffer hasznos, mert csökkenti az adatbázis köröket. De ha túl nagyra nő, akkor már nem optimalizáció, hanem torlódás.

Ezért kell együtt nézni a batch size-t, a flush frequency-t és a bulk write latency-t. Egyetlen metrika önmagában félrevezető lehet. A folyamat viselkedése a metrikák kapcsolatából derül ki.

---

## Slide 17 - TTL

**Idő:** 0:45
**Visszaszámláló:** 01:55 → 01:10

A nyers adat megtartása nem ingyen van. Ha másodpercenként 1 500 dokumentum érkezik, az adatmegőrzés gyorsan tárhely- és teljesítménykérdéssé válik.

MongoDB-ben a TTL index kényelmes megoldásnak tűnik: lejár az adat, az adatbázis takarít. De nagy beérkező volumen mellett nem elég azt feltételezni, hogy a háttérfolyamat majd biztosan lépést tart.

Itt is ugyanaz a kérdés: a beszúrási sebesség és a törlési sebesség egyensúlyban van-e? Ha több adat érkezik, mint amennyit a TTL takarítás el tud vinni, akkor a gyűjtemény nőni fog. Először csak tárhelyben, aztán indexméretben, aztán teljesítményben is.

---

## Slide 18 - TTL broken

**Idő:** 0:50
**Visszaszámláló:** 01:10 → 00:20

Ez a "broken" TTL állapot tanulsága.

Amikor a takarítás nem tart lépést a beérkező adatmennyiséggel, az nem azonnal látványos alkalmazáshiba. Sokáig úgy tűnhet, hogy minden megy tovább. A dokumentumok jönnek, a processzor dolgozik, a felhasználó talán még lát adatot.

Közben viszont a rendszer belül adósságot halmoz. Nő a gyűjtemény, nő az index, romolhatnak az írási idők, és egy idő után az egész folyamat késésként vagy instabilitásként adja vissza ezt.

A fontos tanulság: az adatmegőrzést ugyanúgy monitorozni kell, mint az adatbeérkezést és az aggregációt. Nem elég azt mérni, hogy bejön-e az adat. Azt is mérni kell, hogy kontrolláltan ki tud-e menni a rendszerből.

---

## Slide 19 - Q&A / zárás

**Idő:** 0:20
**Visszaszámláló:** 00:20 → 00:00

Köszönöm, a részletesebb publikációimat megtaláljátok Mediumon a QR-kódon vagy a `medium.com/@majorbalu` oldalon, és most jöhet az 5 perc Q&A.
