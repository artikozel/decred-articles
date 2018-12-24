## Szczegółowa analiza odporności na rozszczepienie łańcucha sieci Decred

Nie jest już tajemnicą, że sieci oparte jedynie na PoW są podatne na rozwidlanie. Byliśmy świadkami stworzenia wielu monet, które powstały w wyniku mniejszościowego rozłamu sieci, w tym Ethereum Classic, Bitcoin Gold, Bitcoin Cash i Bitcoin SV.

Ten post analizuje, w jaki sposób sieć Decred zapobiega takim mniejszościowym rozłamom. Analiza została pierwotnie [opublikowana na Reddit](https://www.reddit.com/r/decred/comments/7f9ie1/detailed_analysis_of_decred_fork_resistance/) przez davecgh. Opisuje ona ważne aspekty hybrydowego systemu konsensusu Decred, które istotne są w tym temacie, a następnie szczegółowo omawia to, co by się stało, gdyby jakakolwiek jednostka próbowała rozszczepić blockchain Decred.

Jeśli potrzebujesz przypomnienia, dlaczego takich rozwidleń należy unikać, przeczytaj [ten artykuł.](https://blog.goodaudience.com/blockchain-forks-and-chain-splits-why-we-ould-avoid-them-f54c693a90f1)

### Dawka wiedzy wstępnej

Sieć Decred zabezpieczana jest zarówno przez górników PoW, jak i głosujących PoS. System PoS sieci Decred działa poprzez zamykanie większej ilości monet w tak zwany bilet do głosowania. Bilety te funkcjonują jako podstawowy element konstrukcyjny całego systemu, który pozwala zainteresowanym stronom uczestniczyć w [zarządzaniu Decred](https://docs.decred.org/governance/introduction-to-decred-governance/).

Na każdy blok jest dostępne tylko 20 biletów. Po nabyciu biletu obowiązuje okres dojrzewania, w którym bilety nie wchodzą jeszcze do puli głównej, wynoszący 256 bloków, po czym bilet zostaje umieszczony w puli biletów gotowych do głosowania. Docelowa wielkość puli gotowych do głosowania biletów to 40960 jednostek, ale rozmiar puli może się powiększać lub zmniejszać w trakcie operacji. Trudność stakingu (cena biletu) jest korygowana na podstawie mechanizmu popytu i podaży, aby próbować utrzymać pulę przy jej docelowym rozmiarze. Proces ten jest dokładniej omówiony w [DCP0001](https://github.com/decred/dcps/blob/master/dcp-0001/dcp-0001.mediawiki) dla czytelników, których interesuje dokładniejszy opis mechanizmu.

Gotowe do głosowania bilety czekają w puli, aby oddać swój głos, a proces selekcji jest niemożliwy do zmanipulowania przez górników PoW. Algorytm, który kontroluje wybór biletów, jest oparty przede wszystkim na haszu poprzedniego bloku, co oznacza, że ​​jest zarówno pseudolosowy, jak i deterministyczny. Jeśli budujesz blok 100 na bloku 99, bilety, które mają być zawarte w bloku 100, są znane każdemu pełnemu węzłowi w sieci. Wybór biletów można zmienić tylko poprzez znalezienie nowego rozwiązania bloku 99 z innym hashem, co z kolei spowodowałoby wybranie nowego zestawu losowych biletów w celu zakwalifikowania ich do głosowania.

W każdym bloku może głosowac do 5 biletów. Przynajmniej 3 z 5 głosów muszą być uwzględnione w bloku; w przeciwnym wypadku, blok zostanie odrzucony przez sieć. Nagroda dla górników PoW jest zmniejszona, jeśli uwzględnione są tylko 3 lub 4 głosy, odpowiednio o 40% i 20%, po to, aby zniechęcić górników do ignorowania głosów, żeby próbować w ten sposób ograć system.

Ważne jest, aby pamiętać, że interesariusze **muszą być obecni na danej odnodze łańcucha**, gdy ich bilety zostają wybrane. Sam akt nabycia biletu nie oznacza, że ​bilet ​automatycznie głosuje, gdyż Twój portfel (lub Twój usługodawca głosowania [ang. *Voting Service Provider*]) musi oddać Twój głos po wybraniu biletu. To rozróżnienie jest kluczowe, ponieważ oznacza, że na ​​pulę gotowych do głosowania biletów na łańcuchu mniejszościowym w dużej mierze składają się bilety bez prawa do głosu, ponieważ ich właściciele znajdują się na innym łańcuchu.

Szczegółowe objaśnienie teorii leżącej u podstaw każdego z tych aspektów wykracza poza zakres tego wpisu, ale dotyczy, przede wszystkim, ochrony przed różnymi przeciwstawnymi i nieprzyjaznymi sytuacjami.

### Scenariusz, założenia i metodologia
Mając to wszystko na uwadze, wyobraźmy sobie scenariusz, w którym jednostka próbuje stworzyć forka, z którym nie zgadza się 75% interesariuszy.

Załóżmy, że obie strony omawianego forka mają taką samą moc obliczeniową (a więc 50% mocy obliczeniowej na każdej odnodze forka). A zatem, 75% interesariuszy znajduje się na łańcuchu większości, a 25% na łańcuchu mniejszości.

Co więcej, załóżmy, że najnowszym blokiem w momencie forka jest blok 99999. Zatem obie strony łańcucha pracują nad znalezieniem bloku 100000, z jednej strony na zestawie reguł mniejszości, z drugiej strony na zestawie reguł większości.

Wreszcie, w celu uproszczenia opisu i ułatwienia śledzenia logiki, ponieważ tylko 25% interesariuszy znajduje się w łańcuchu mniejszościowym, załóżmy, że co czwarty bilet w puli gotowych biletów jest uczestnikiem łańcucha mniejszości. Innymi słowy, numery biletów 0, 4, 8, 12, 16, 20, ..., 40956 to bilety w puli, które reprezentują interesariuszy w łańcuchu mniejszości, podczas gdy numery biletów 1, 2, 3, 5, 6, 7, 9, ..., 40957, 40958, 40959, są biletami w puli, które reprezentują interesariuszy w łańcuchu większościowym.

Pamiętaj: **interesariusze muszą znajdować się na danej odnodze łańcucha**, gdy zostaną wybrane ich bilety, aby oddać głos.

![](https://cdn-images-1.medium.com/max/1000/1*Y8OnkamQAVLjSd8Ch5eQUA.png)
*<p align="center">Ilustracja naszego hipotetycznego scenariusza</p>*
### Opis krok-po-kroku
Poniżej znajduje się sekwencja wydarzeń, która miałaby miejsce w razie wystąpienia scenariusza rozszczepienia sieci, jak opisano i zilustrowano powyżej.

#### Blok 100000
Moc obliczeniowa obu łańcuchów będzie próbowała zbudować nowy blok na bloku 99999.
Aby ten nowy blok został zbudowany na łańcuchu mniejszości, musi zdobyć co najmniej 3 głosy z puli biletów, a wybrane głosy zależą od bloku 99999.
Bilety wymagane do zbudowania bloku 100000, w oparciu o hasz bloku 99999, są biletami o numerach 17113, 17331, 21307, 21328 i 24903.
Jak widać, 4 z tych 5 biletów są interesariuszami na łańcuchu większościowym (numery biletów 17113, 17331, 21307 i 24903), co oznacza, że ​​zamierzają oddać swój głos na blok 100000 na łańcuchu większościowym.
Łańcuch mniejszości jest w stanie zdobyć tylko 1 głos (bilet nr 21328), więc nie może zbudować bloku 100000. Zamiast tego, musi wrócić i znaleźć nowe rozwiązanie dla bloku 99999, aby wybrać nowy zestaw biletów .
W tym momencie łańcuchy wyglądają następująco. Nawiasy z * w tym zapisie wskazują bloki, nad którymi trwają prace.

```
... -> [99999] -> (100000*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> (99999a*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu
```

Innymi słowy, łańcuch większości działa obecnie na bloku 100000, podczas gdy łańcuch mniejszości utknął próbując znaleźć nowe rozwiązanie dla bloku 99999, aby uzyskać nowy zestaw biletów, mając przy tym nadzieję, że tym razem będzie w stanie uzyskać co najmniej 3 głosy. Ponieważ w naszym eksperymencie myślowym oba łańcuchy mają równą moc obliczeniową, możemy bezpiecznie założyć, że  blok 100000 w łańcuchu większościowym, jak i nowy blok 99999 (nazwijmy go 99999a) w łańcuchu mniejszościowym zostaną znalezione średnio w tym samym czasie.

#### Blok 100001
W tym momencie stanie się, co następuje:

Moc obliczeniowa na łańcuchu większościowym będzie próbowała zbudować nowy blok na podstawie bloku 100000 łańcucha większości. Wymagane głosy dla tego bloku to bilety o numerach 563, 6766, 21009, 37394 i 37775.
Tym razem, okazuje się, że wszystkie 5 z tych 5 wylosowanych biletów nalezą do interesariuszy na łańcuchu większościowym, co oznacza, że ​​zamierzają oni oddać swój głos na blok 100000 w łańcuchu większości, co pozwala na zbudowanie bloku 100001.
Łańcuch mniejszości, teraz z nową wersją bloku 99999 (99999a), ma nowy hasz, więc kończy się wylosowaniem biletów o numerach 1069, 8007, 16413, 19172 i 31821.
Łańcuch mniejszości ponownie zdobywa tylko 1 głos (bilet nr 19172), więc musi jeszcze raz wrócić i znaleźć jeszcze jedno nowe rozwiązanie dla bloku 99999 w celu spowodowania wyboru nowego zestawu biletów.
Łańcuchy wyglądają obecnie następująco:


```
... -> [99999] -> [100000] -> (100001*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> (99999b*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu
```

Innymi słowy, łańcuch większości pracuje obecnie nad blokiem 100001, podczas gdy łańcuch mniejszości wciąż tkwi przy próbach znalezienia nowego rozwiązania dla bloku 99999, aby otrzymać nowy zestaw biletów, mając nadzieję, że tym razem będą mogli uzyskać co najmniej 3 głosy. Ponieważ w naszym eksperymencie myślowym oba łańcuchy mają równą moc mieszającą, możemy ponownie bezpiecznie przyjąć, blok 100001 w łańcuchu większościowym, jak i nowy blok 99999 (nazwijmy go 99999b) w łańcuchu mniejszościowym zostaną znalezione w około tym samym momencie.

#### Blok 100002
W tym momencie stanie się, co następuje:

Moc obliczeniowa na łańcuchu większościowym będzie próbowała zbudować nowy blok na podstawie bloku 100001 łańcucha większości. Wymagane głosy dla tego bloku to bilety o numerach 174, 1999, 12808, 31928, oraz 38317.
Tym razem 3 z tych 5 biletów są interesariuszami na łańcuchu większościowym (bilety o numerach 174, 1999, 38317), co oznacza, że ​​zamierzają oddać swój głos na blok 100001 w łańcuchu większości, co pozwala na zbudowanie bloku 100002.
Łańcuch mniejszości, teraz z nową wersją bloku 99999 (99999b), ma nowy hasz, więc kończy się wymaganiem biletów o numerach 4653, 15211, 29988, 35175 i 35665.
Łańcuch mniejszości jest wciąż w stanie uzyskać tylko 1 głos (bilet nr 29988), więc musi jeszcze raz wrócić i znaleźć kolejne nowe rozwiązanie dla bloku 99999 w celu spowodowania wyboru nowego zestawu biletów.
Łańcuchy wyglądają obecnie następująco:


```
... -> [99999] -> [100000] -> [100001] -> (100002*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> (99999c*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu
```

Innymi słowy, łańcuch większości pracuje teraz nad blokiem 100002, podczas gdy łańcuch mniejszości wciąż tkwi przy próbach znalezienia kolejnego nowego rozwiązania dla bloku 99999 w celu uzyskania nowego zestawu biletów, mając nadzieję, że tym razem będą w stanie uzyskać co najmniej 3 głosy.

![](https://cdn-images-1.medium.com/max/800/1*vnZv8fLSnjc6xIu6GbZh3Q.jpeg)

#### Szybki przeskok do bloku 100010
Proces ten powtarza się, aż w końcu jakiś wariant bloku 99999 na łańcuchu mniejszościowym będzie miał szczęście i wylosuje 3 bilety, które są na łańcuchu mniejszości. Okazuje się, że jest to około 1 na 10 prób. Tak więc, przewijając do przodu stan łańcuchów z naszego eksperymentu, gdy tak się wreszcie stanie, łańcuchy wyglądają następująco:

```
... -> [99999] -> [100000] -> [100001] -> [100002] -> ... -> [100009] -> (100010*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> [99999j] -> (100000a*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu
```

Powinno być całkiem jasne, ponieważ oba łańcuchy mają równą moc haszowania, że nie ma możliwości, aby łańcuch mniejszości mógł kiedykolwiek dogonić łańcuch większości. Co więcej, ten sam proces powtórzy się dla bloku 100001 łańcucha mniejszościowego, gdzie będzie musiał wracać i wydobywać ponownie (znajdować nowe rozwiązania) hasze dla swojego bloku 100000 w kółko, dopóki ponownie nie uzyska szczęśliwego losowania, w którym otrzyma przynajmniej 3 wymagane głosy.

W związku z tym, *górnicy nie pozostaną na łańcuchu mniejszości*, ponieważ nie otrzymują żadnych nagród. Łańcuch mniejszości nigdy nie będzie opłacalny, a zatem cała moc wydobywcza ostatecznie powróci na łańcuch większości.

#### Najczęstsze zastrzeżenia
***Co się stanie, jeśli łańcuch mniejszości uzyska więcej niż 10-krotność mocy obliczeniowej głównego łańcucha?***

Teoretycznie, jeśli łańcuch mniejszości z jedynie 25% poparciem interesariuszy miałby 10x mocy obliczeniowej głównego łańcucha, wówczas mógłby nadążyć za łańcuchem większości, jednak nie jest to realistyczny scenariusz ze względu na ekonomiczne zachęty (incentywy).

Wydobywanie na łańcuchu mniejszości z 10-krotną mocą obliczeniową oznacza, że ​​górnicy otrzymaliby tylko 1/10 nagrody za wydobycie bloku w porównaniu z łańcuchem większości, w oparciu o samą moc obliczeniową. W naszym scenariuszu jest on zredukowany jeszcze bardziej do 1/10 z 60%, ponieważ średnio w blokach uwzględniane są tylko 3 głosy. Innymi słowy, górnicy otrzymaliby tylko 6% nagród, które uzyskaliby, wydobywając bloki na łańcuchu większości. Patrząc na to z innej strony, wydobywanie bloków na łańcuchu mniejszości wiązałoby się z otrzymywaniem nagrody o 94% mniejszej, niż na łańcuchu większościowym.

Przekładając to wszystko na liczby można rzec, że gdyby górnik posiadał 5% całkowitej mocy obliczeniowej sieci, mógłby oczekiwać, że otrzyma 5% nagrody PoW za wydobycie bloku, lub 5% z 13,89 ≈ 0,6945 DCR [na daną chwilę](https://www.reddit.com/r/decred/comments/7f9ie1/detailed_analysis_of_decred_fork_resistance/). Jednak na łańcuchu mniejszości, po pierwsze nagroda wynosiłaby 60% z 13,89 ≈ 8,344 DCR, a następnie to 5% mocy obliczeniowej stanowiłoby tylko 0,5% całkowitej mocy obliczeniowej na łańcuchu mniejszości, a więc 0,5% z ~ 8.334 ≈ 0,04167 DCR. Patrząc na liczby, widzimy, że 0,04167 DCR rzeczywiście stanowi 6% z 0,6945 DCR (czyli tego, co otrzymałby górnik z ta samą ilością mocy obliczeniowej na łańcuchu większości).

Wydobycie PoW jest bardzo konkurencyjne, ponieważ jest grą o sumie zerowej. Większość górników, nawet tych cieszących się ogromną przewagę w postaci np. darmowej energii elektrycznej, ma cienkie marże i często liczy na wzrost ceny, która pokryje koszty ich inwestycji i operacji. Biorąc pod uwagę spadek dochodu o 94%, większość górników musiałaby płacić za możliwość wydobywania na łańcuchu mniejszości.

***Czy nie można po prostu zmienić zasad konsensusu tak, żeby zignorować interesariuszy?***

Gdyby łańcuch mniejszości usunął lub zablokował głosowanie biletowe na określony czas, byłby wówczas w stanie produkować bloki i odłączyć się od łańcucha większości. Chociaż teoretycznie jest to możliwe, całkowicie zniszczyłoby to system hybrydowy i w zasadzie przywróciłoby to oddzieloną walutę z powrotem do bycia siecią czysto PoW. Bez wątpienia to nie byłby już Decred.

W przeciwieństwie do monet czysto PoW, gdzie nikt nie może określić, który łańcuch jest "prawdziwy", ze względu na brak udowodnionego i sformalizowanego systemu zarządzania, Decred ma bardzo jasno określony i dobrze zrozumiały model zarządzania. To interesariusze Decred podejmują decyzję, który łańcuch jest prawdziwym Decred i robią to bezpośrednio na łańcuchu, w sposób możliwy do kryptograficznego udowodnienia.

Interesariusze angażują się w Decred oczekując, że główne decyzje odnośnie konsensusu podejmują oni sami jako strony zainteresowane. Usunięcie władzy interesariuszy byłoby podobne do usunięcia Proof-of-Work z czystej monety PoW. Innymi słowy, całkowicie zniszczyłoby to właściwości bezpieczeństwa systemu. Ile zaufania mają pokładać posiadacze monety w projekt, który ignoruje jedną z podstawowych cech, które ma za zadanie oferować?

### Wnioski
Hybrydowy system PoW i PoS projektu Decred sprawia, że ​​rozdwojenie łańcucha jest niezwykle trudne - jeśli nie niemożliwe - bez większego poparcia ze strony interesariuszy. W tej analizie pokazano, dlaczego scenariusz Classic, Gold lub Cash jest bardzo mało prawdopodobny w sieci Decred.

Koszty utrzymania łańcucha mniejszości, nawet z 10-krotnością mocy obliczeniowej, są znaczne; górnicy mogą oczekiwać poważnej redukcji dochodów, jeśli zdecydują się na udział. Ewentualnie, możnaby usunąć lub wyłączyć system PoS i podzielić łańcuch Decred tak, jak każdą inną sieć PoW, jednak to mija się z celem istnienia Decred i wątpliwe jest, czy ktokolwiek potraktowałby taką próbę na poważnie.

Ustanowienie podstaw odporności na podział sieci ma kluczowe znaczenie dla jej długowieczności. Hybrydowy system PoW i PoS tworzy mechanizm kontroli i równowagi, aby małe grupy nie mogły zdominować przepływu transakcji lub wprowadzić zmian w protokole Decred bez porozumienia między interesariuszami. System ten zachęca do koordynacji i współpracy, dzięki której Decred przekształca się w niezwykle silną sieć, której zaprojektowana była właśnie po to, aby przetrwać w dłuższej perspektywie czasu.

### Dalsza lektura
Ten post omówił ważny temat odporności na podział sieci, lecz do odkrycia i omówienia pozostaje dużo więcej, na przykład to, żeby hybrydowy system PoW i PoS sieci Decred jest też najlepszym przeciwśrodkiem na ataki większościowe (ataki 51%). Jeśli chcesz dowiedzieć się, w jaki sposób to osiąga, przeczytaj post autorstwa [Zubair Zia](https://medium.com/@zubairzia):

[Hybrydowy protokół Decred - najlepszy środek obronny przed atakami większościowymi](https://medium.com/decred/decreds-hybrid-protocol-a-superior-deterrent-to-majority-attacks-9421bf486292)


W tym artykule pokazano, w jaki sposób unikalny, hybrydowy protokół Decred zapewnia lepszą ochronę przed atakami większości.

Jeśli jesteś zainteresowany bardziej złożonymi tematami, możesz zbadać, jak Decred jest w stanie płynnie ulepszać swoją sieć poprzez [głosowanie nad zmianami w zasadach konsensusu](https://medium.com/decred/blockchain-governance-how-decred-iterates-upon-bitcoin-3cc7030c655e), lub w jaki sposób użytkownicy mogą składać wnioski do [pozałańcuchowego systemu zarządzania zwanego Politeia](https://docs.decred.org/governance/politeia/). Jeśli interesują Cię szczegóły techniczne, zagłęb się w [dokumentację techniczną Decred](https://docs.decred.org/).

Jeśli chcesz wejść w interakcję ze społecznością Decred, wybierz jedną z wymienionych tutaj platform komunikacyjnych. Jesteśmy grupą pragmatycznych ludzi - dołącz do nas!

### Zasługi
Ten wpis najprawdopodobniej nie powstałby, gdyby nie [oryginalna analiza tematu autorstwa davecgh](https://www.reddit.com/r/decred/comments/7f9ie1/detailed_analysis_of_decred_fork_resistance/). Ponadto, recenzja tego wpisu autorstwa [Artikozel](https://medium.com/@artikozel), oraz konstruktywne komentarze ze strony użytkowników na naszym kanale pisarskim niesamowicie ulepszyły jego jakość. Ilustracja owego scenariusza została stworzone przez [Zubair Zia](https://medium.com/@zubairzia). Dziękuję Wam wszystkim!
