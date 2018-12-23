## Detailed analysis of Decred fork resistance

Nie jest już tajemnicą, że sieci oparte jedynie na PoW są podatne na rozwidlanie. Byliśmy świadkami stworzenia wielu monet, które powstały w wyniku mniejszościowego rozłamu sieci , w tym Ethereum Classic, Bitcoin Gold, Bitcoin Cash i Bitcoin SV.

Ten post analizuje, w jaki sposób sieć Decred zapobiega takim mniejszościowym rozłamom. Analiza została pierwotnie [opublikowana na Reddit](https://www.reddit.com/r/decred/comments/7f9ie1/detailed_analysis_of_decred_fork_resistance/) przez davecgh. Opisuje on ważne aspekty hybrydowego systemu konsensusu Decred, które istotne są w tym temacie, a następnie szczegółowo omawia to, co by się stało, gdyby jakakolwiek jednostka próbowała rozszczepić blockchain Decred.

!!!!!!!!!!!Jeśli potrzebujesz przypomnienia, dlaczego takich rozwidleń należy unikać, przeczytaj ten artykuł: (https://blog.goodaudience.com/blockchain-forks-and-chain-splits-why-we-ould-avoid-them-f54c693a90f1)

### Dawka wiedzy wstępnej

Sieć Decred zabezpieczana jest zarówno przez górników PoW, jak i głosujących PoS. System PoS sieci Decred działa poprzez zamykanie większej garści monet w tak zwany bilet do głosowania. Bilety te funkcjonują jako podstawowy element konstrukcyjny całego systemu, który pozwala zainteresowanym stronom uczestniczyć w zarządzaniu Decred.

Na każdy blok jest dostępne tylko 20 biletów. Po nabyciu biletu obowiązuje okres dojrzewania, w którym bilety nie wchodzą jeszcze do puli głównej, wynoszący 256 bloków, po czym bilet zostaje umieszczony w puli biletów gotowych do głosowania. Docelowa wielkość puli gotowych do głosowania biletów to 40960 jednostek, ale rozmiar puli może się powiększać lub zmniejszać w trakcie operacji. Trudność stakingu (cena biletu) jest korygowana na podstawie mechanizmu popytu i podaży, aby próbować utrzymać pulę przy jej docelowym rozmiarze. Proces ten jest dokładniej omówiony w DCP0001 dla czytelników, których interesuje dokładniejszy opis mechanizmu.

Gotowe do głosowania bilety czekają w puli, aby oddać swój głos, a proces selekcji jest niemożliwy do zmanipulowania przez górników PoW. Algorytm, który kontroluje wybór biletów, jest oparty przede wszystkim na haszu poprzedniego bloku, co oznacza, że ​​jest zarówno pseudolosowy, jak i deterministyczny. Jeśli budujesz blok 100 na bloku 99, bilety, które mają być zawarte w bloku 100, są znane każdemu pełnemu węzłowi w sieci. Wybór biletów można zmienić tylko poprzez znalezienie nowego rozwiązania bloku 99 z innym hashem, co z kolei spowodowałoby wybranie nowego zestawu losowych biletów w celu zakwalifikowania ich do głosowania.

W każdym bloku może głosowac do 5 biletów. Przynajmniej 3 z 5 głosów muszą być uwzględnione w bloku; w przeciwnym wypadku, blok zostanie odrzucony przez sieć. Nagroda dla górników PoW jest zmniejszona, jeśli uwzględnione są tylko 3 lub 4 głosy, odpowiednio o 40% i 20%, po to, aby zniechęcić górników do ignorowania głosów, żeby próbować w ten sposób ograć system.

Szczegółowe objaśnienie teorii leżącej u podstaw każdego z tych aspektów wykracza poza zakres tego wpisu, ale dotyczy, przede wszystkim, ochrony przed różnymi przeciwstawnymi i nieprzyjaznymi sytuacjami.

### Scenariusz, założenia i metodologia
Mając to wszystko na uwadze, wyobraźmy sobie scenariusz, w którym jednostka próbuje stworzyć forka, z którym nie zgadza się 75% interesariuszy.

Załóżmy, że obie strony próbowanego forka mają taką samą moc obliczeniową (a więc 50% mocy obliczeniowej na każdej odnodze forka). A zatem, 75% interesariuszy znajduje się na łańcuchu większości, a 25% na łańcuchu mniejszości.

Co więcej, załóżmy, że najnowszym blokiem w momencie forka jest blok 99999. Zatem obie strony łańcucha pracują nad znalezieniem bloku 100000, z jednej strony na zestawie reguł mniejszości, z drugiej strony na zestawie reguł większości.

Wreszcie, w celu uproszczenia opisu i ułatwienia śledzenia logiki, ponieważ tylko 25% interesariuszy znajduje się w łańcuchu mniejszościowym, załóżmy, że co czwarty bilet w puli gotowych biletów jest uczestnikiem łańcucha mniejszości. Innymi słowy, numery biletów 0, 4, 8, 12, 16, 20, ..., 40956 to bilety w puli, które reprezentują interesariuszy w łańcuchu mniejszości, podczas gdy numery biletów 1, 2, 3, 5, 6, 7, 9, ..., 40957, 40958, 40959, są biletami w puli, które reprezentują interesariuszy w łańcuchu większościowym.

Pamiętaj: interesariusze muszą znajdowac się na danej odnodze łańcucha, gdy zostaną wybrane ich bilety, aby oddać głos.

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

... -> [99999] -> (100000*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> (99999a*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu

Innymi słowy, łańcuch większości działa obecnie na bloku 100000, podczas gdy łańcuch mniejszości utknął próbując znaleźć nowe rozwiązanie dla bloku 99999, aby uzyskać nowy zestaw biletów, mając przy tym nadzieję, że tym razem będzie w stanie uzyskać co najmniej 3 głosy. Ponieważ w naszym eksperymencie myślowym oba łańcuchy mają równą moc obliczeniową, możemy bezpiecznie założyć, że  blok 100000 w łańcuchu większościowym, jak i nowy blok 99999 (nazwijmy go 99999a) w łańcuchu mniejszościowym zostaną znalezione średnio w tym samym czasie.

#### Blok 100001
W tym momencie stanie się, co następuje:

Moc obliczeniowa na łańcuchu większościowym będzie próbowała zbudować nowy blok na podstawie bloku 100000 łańcucha większości. Wymagane głosy dla tego bloku to bilety o numerach 563, 6766, 21009, 37394 i 37775.
Tym razem, okazuje się, że wszystkie 5 z tych 5 wylosowanych biletów nalezą do interesariuszy na łańcuchu większościowym, co oznacza, że ​​zamierzają oni oddać swój głos na blok 100000 w łańcuchu większości, co pozwala na zbudowanie bloku 100001.
Łańcuch mniejszości, teraz z nową wersją bloku 99999 (99999a), ma nowy hasz, więc kończy się wylosowaniem biletów o numerach 1069, 8007, 16413, 19172 i 31821.
Łańcuch mniejszości ponownie zdobywa tylko 1 głos (bilet nr 19172), więc musi jeszcze raz wrócić i znaleźć jeszcze jedno nowe rozwiązanie dla bloku 99999 w celu spowodowania wyboru nowego zestawu biletów.
Łańcuchy wyglądają obecnie następująco:


... -> [99999] -> [100000] -> (100001*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> (99999b*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu

Innymi słowy, łańcuch większości pracuje obecnie nad blokiem 100001, podczas gdy łańcuch mniejszości wciąż tkwi przy próbach znalezienia nowego rozwiązania dla bloku 99999, aby otrzymać nowy zestaw biletów, mając nadzieję, że tym razem będą mogli uzyskać co najmniej 3 głosy. Ponieważ w naszym eksperymencie myślowym oba łańcuchy mają równą moc mieszającą, możemy ponownie bezpiecznie przyjąć, blok 100001 w łańcuchu większościowym, jak i nowy blok 99999 (nazwijmy go 99999b) w łańcuchu mniejszościowym zostaną znalezione w około tym samym momencie.

#### Block 100002
W tym momencie stanie się, co następuje:

Moc obliczeniowa na łańcuchu większościowym będzie próbowała zbudować nowy blok na podstawie bloku 100001 łańcucha większości. Wymagane głosy dla tego bloku to bilety o numerach 174, 1999, 12808, 31928, oraz 38317.
Tym razem 3 z tych 5 biletów są interesariuszami na łańcuchu większościowym (bilety o numerach 174, 1999, 38317), co oznacza, że ​​zamierzają oddać swój głos na blok 100001 w łańcuchu większości, co pozwala na zbudowanie bloku 100002.
Łańcuch mniejszości, teraz z nową wersją bloku 99999 (99999b), ma nowy hasz, więc kończy się wymaganiem biletów o numerach 4653, 15211, 29988, 35175 i 35665.
Łańcuch mniejszości jest wciąż w stanie uzyskać tylko 1 głos (bilet nr 29988), więc musi jeszcze raz wrócić i znaleźć kolejne nowe rozwiązanie dla bloku 99999 w celu spowodowania wyboru nowego zestawu biletów.
Łańcuchy wyglądają obecnie następująco:


... -> [99999] -> [100000] -> [100001] -> (100002*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> (99999c*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu

Innymi słowy, łańcuch większości pracuje teraz nad blokiem 100002, podczas gdy łańcuch mniejszości wciąż tkwi przy próbach znalezienia kolejnego nowego rozwiązania dla bloku 99999 w celu uzyskania nowego zestawu biletów, mając nadzieję, że tym razem będą w stanie uzyskać co najmniej 3 głosy.

#### Szybki przeskok do bloku 100010
Proces ten powtarza się, aż w końcu jakiś wariant bloku 99999 na łańcuchu mniejszościowym będzie miał szczęście i wylosuje 3 bilety, które są na łańcuchu mniejszości. Okazuje się, że jest to około 1 na 10 prób. Tak więc, przewijając do przodu stan łańcuchów z naszego eksperymentu, gdy tak się wreszcie stanie, łańcuchy wyglądają następująco:

... -> [99999] -> [100000] -> [100001] -> [100002] -> ... -> [100009] -> (100010*) -
interesariusze większościowi (75%) są na tym łańcuchu

-> [99999j] -> (100000a*) -
interesariusze mniejszościowi (25%) są na tym łańcuchu

Powinno być całkiem jasne, ponieważ oba łańcuchy mają równą moc haszowania, że nie ma możliwości, aby łańcuch mniejszości mógł kiedykolwiek dogonić łańcuch większości. Co więcej, ten sam proces powtórzy się dla bloku 100001 łańcucha mniejszościowego, gdzie będzie musiał wracać i wydobywać ponownie (znajdować nowe rozwiązania) hasze dla swojego bloku 100000 w kółko, dopóki ponownie nie uzyska szczęśliwego losowania, w którym otrzyma przynajmniej 3 wymagane głosy.

W związku z tym, górnicy nie pozostaną na łańcuchu mniejszości, ponieważ nie otrzymują żadnych nagród. Łańcuch mniejszości nigdy nie będzie opłacalny, a zatem cała moc wydobywcza ostatecznie powróci na łańcuch większości.

#### Najczęstsze zastrzeżenia
***Co się stanie, jeśli łańcuch mniejszości uzyska więcej niż 10-krotność mocy obliczeniowej głównego łańcucha?***

Teoretycznie, jeśli łańcuch mniejszości z jedynie 25% poparciem interesariuszy miałby 10x mocy obliczeniowej głównego łańcucha, wówczas mógłby nadążyć za łańcuchem większości, jednak nie jest to realistyczny scenariusz ze względu na ekonomiczne zachęty (incentywy).

Wydobywanie na łańcuchu mniejszości z 10-krotną mocą obliczeniową oznacza, że ​​górnicy otrzymaliby tylko 1/10 nagrody za wydobycie bloku w porównaniu z łańcuchem większości, w oparciu o samą moc obliczeniową. W naszym scenariuszu jest on zredukowany jeszcze bardziej do 1/10 z 60%, ponieważ średnio w blokach uwzględniane są tylko 3 głosy. Innymi słowy, górnicy otrzymaliby tylko 6% nagród, które uzyskaliby, wydobywając bloki na łańcuchu większości. Patrząc na to z innej strony, wydobywanie bloków na łańcuchu mniejszości wiązałoby się z otrzymywaniem nagrody o 94% mniejszej, niż na łańcuchu większościowym.

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

Gdyby tak było, gdyby górnik posiadał 5% całkowitej mocy sieci, mogliby oczekiwać otrzymania około 5% nagrody PoW za blok lub 5% z 13,89 ≈ 0,6945 DCR w obecnym czasie. Jednak w łańcuchu mniejszości, pierwsza nagroda wynosiłaby 60% z 13,89 ≈ 8,344 DCR, a następnie ta 5% moc mieszająca byłaby tylko 0,5% całkowitej mocy mieszania w łańcuchu mniejszości, a więc 0,5% z ~ 8.334 ≈ 0,04167 DCR. Patrząc na liczby, widzimy, że 0,04167 DCR jest rzeczywiście 6% z 0,6945 DCR.

What if the minority chain gets more than 10x the hash power of the main chain?
Theoretically, if the minority chain with only 25% stakeholder approval had 10x the hash power of the main chain, yes, it could keep up with the majority chain. However, this is not a realistic scenario because of the economic incentives.

Mining the minority chain with 10x the hash power effectively means that the miners would only be getting 1/10 of the block reward as they would on the majority chain, based on hash power alone. In our scenario it’s reduced even further to 1/10 of 60% due to only being able to include 3 votes on average. In other words, miners would only receive 6% of the rewards they would by mining the majority chain. Looking at it from another angle, they would receive 94% less by mining the minority chain.

Putting that into numbers, if a miner had, say 5% of the total network hash power, they could expect to receive roughly 5% of the PoW reward per block, or 5% of ~13.89 ≈ 0.6945 DCR at the current time. However, on the minority chain, first the reward would be 60% of ~13.89 ≈ 8.334 DCR, and then that 5% hash power would only be 0.5% of the total hash power on the minority chain, thus 0.5% of ~8.334 ≈ 0.04167 DCR. Looking at the numbers, we can see that 0.04167 DCR is indeed 6% of 0.6945 DCR.

PoW mining is very competitive since it is a zero sum game. Most miners, even those with huge advantages such as free electricity, have thin margins and are often banking on future appreciation to pick up the slack. Given the 94% reduction in income, most miners would actually have to pay in order to mine on the minority chain.

Can’t somebody just change the consensus rules to ignore the stakeholders?
If the minority chain removed or disabled ticket voting for a certain period of time, it would be able to produce blocks and fork away from the majority chain. While it is theoretically possible, doing so would completely destroy the hybrid system and return the forked currency to effectively being a pure PoW network. It would undoubtedly no longer be Decred.

Unlike in pure PoW coins where nobody can say which chain is the “real” one due to the lack of a provable and formalized governance system, Decred has a very clear and well understood governance model. Decred stakeholders make the decision which chain is the real Decred and they do so in an on-chain and cryptographically provable fashion.

Stakeholders sign up for Decred with the expectation that major consensus decisions are made by the stakeholders themselves. Removing the authority of the stakeholders would be akin to removing Proof-of-Work from a pure PoW coin. In other words, it would completely destroy the security properties of the system. How much confidence are holders going to have in a coin that ignores one of the primary characteristics it claims to offer?

### Wnioski
Decred’s hybrid PoW and PoS consensus system makes blockchain forks extremely difficult — if not impossible — without majority stakeholder approval. The walkthrough has demonstrated why a Classic, Gold, or Cash scenario is highly unlikely on the Decred network.

The costs to maintain a minority fork with even 10x of the hash power are substantial; miners can expect a severe reduction in income if they decide to participate. Alternatively, it is possible to remove or disable the PoS system and split the Decred chain like any other PoW network. However, this defeats the purpose of Decred and it is doubtful whether anyone would take such an attempt seriously.

Getting the fundamentals of fork resistance right is critical to longevity. The hybrid PoW and PoS system creates checks and balances to ensure that small groups cannot dominate the flow of transactions or make changes to Decred without agreement among stakeholders. It incentivizes coordination and collaboration, which turns Decred into an uncommonly strong network that is built to last for the long-term.

### Dalsza lektura
This post has covered the important topic of fork resistance, but there is much more to discover. For example, the hybrid PoW and PoS system of Decred is also a superior deterrent to majority (51%) attacks. If you want to know how this works, read this post by Zubair Zia:

Decred’s hybrid protocol, a superior deterrent to majority attacks

This article demonstrates how the unique hybrid protocol of Decred provides superior security against majority attacks.
medium.com
For more advanced topics, you could investigate how Decred can smoothly upgrade its network via voting on consensus rule changes, or how people can submit proposals to the off-chain governance system called Politeia. If you prefer technical details, check out the Decred Documentation.

Pick one of the chat platforms listed here if you want to interact with the Decred community. We are a pragmatic bunch of people — come join us!

### Zasługi
If it wasn’t for the original analysis by davecgh, this post would probably not exist. Furthermore, Artikozel’s review and the constructive comments in the writers room improved this post tremendously. The illustration of the scenario was created by Zubair Zia. Thank you, all!
