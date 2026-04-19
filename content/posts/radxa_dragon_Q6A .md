+++
date = '2026-04-18T19:23:56+02:00'
draft = false
title = 'Radxa Dragon Q6A'
tags= ["raspberry", "sbc", "draxa", "zakupy"]
+++

## Zakup

W czasie korzystania z pierwszej wersji swojego asystenta, która chodziła na Raspberry Pi Zero, często łapałem się na tym, że odpowiedź z bota przychodzi z zauważalnym opóźnieniem. Dzięki Langfusowi widziałem, że sam request nie trwał tak długo, więc doszedłem do wniosku, że to musiała być wina samej płytki. Od razu wiedziałem, co to oznacza. Zakup nowego sprzętu.

Nie miałem jasno zdefiniowanych wymagań dotyczących nowego hardware'u poza jednym — chciałem mieć możliwość uruchomienia na nowej platformie lokalnego modelu LLM wielkości 1–4B. W trakcie poszukiwań miałem sporo pomysłów dotyczących docelowej platformy, ale wszystko rozbijało się o cenę. Od dłuższego czasu nie interesowałem się cenami komputerów i przeżyłem niemałe zaskoczenie. Uznaję to trochę za ironię losu, bo ceny sprzętu rosną ze względu na to, że cały świat zachłysnął się LLM. W każdym razie w grę wchodziły laptopy, mini-PC, mocniejsze modele Raspberry oraz bardziej egzotyczne płytki jak Orange Pi. Finalnie stanęło na czymś jeszcze bardziej egzotycznym (przynajmniej dla mnie) — płytce SBC Radxa Q6A.

O tym wyborze zdecydowało parę czynników. Wszystko zaczęło się od newslettera [Unknow News](https://unknow.news). W jednym z wydań pojawił się artykuł pod tytułem [Every Single Board Computer I Tested in 2025](https://bret.dk/every-single-board-computer-i-tested-in-2025/). Model Q6A został opisany takimi słowami:

> The performance-per-dollar here is genuinely impressive.

Wybiegając parę kroków do przodu, znalazł się tam również fragment dotyczący oprogramowania, który powinien dać mi do myślenia, ale nie uprzedzajmy faktów:

> The big question mark is the software ecosystem.

Co więcej, ta sama płytka została pozytywnie opisana w filmie [This is no joke: the SBC hobby is dying](https://www.youtube.com/watch?v=HeX22LnKdFY) na kanale Jeffa Geerlinga. Zaczęło robić się poważnie.

Sama płytka ma bardzo ciekawą [specyfikację](https://radxa.com/products/dragon/q6a/). To, co mnie szczególnie zainteresowało, to 12 TOPS. W tym miejscu na chwilę się zatrzymam, ponieważ do dnia zakupu nie wiedziałem, czym jest TOPS. Poprosiłem Claude o prostą definicję:

> TOPS to „przepustowość" chipa dla matematyki AI. Więcej TOPS → szybszy inference → niższe latency przy generowaniu tekstu.

W skrócie: im więcej, tym lepiej, a płytka SBC z 12 TOPS to całkiem przyzwoity wynik. Dla porównania Raspberry Pi 4 ma 0 TOPS. Taki wynik modelu Q6A jest możliwy dzięki chipowi NPU, który jest dedykowany do operacji matematycznych używanych w AI — dzięki temu inference modelu jest szybszy.

Na koniec cena: płytka w wersji 12 GB RAM kosztowała z wysyłką 620 zł. Prawie jak za darmo przy dzisiejszych cenach kości RAM.

Miłym dodatkiem było to, że płytka jest chłodzona pasywnie.

## Użytkowanie

W ciągu około 2 tygodni od zamówienia płytka była na moim biurku, więc zabrałem się za instalację systemu. Nie do końca ufam oprogramowaniu dostarczanemu przez producenta, więc zdecydowałem się na instalację stabilnej wersji Armbiana — dedykowanej wersji Ubuntu dla płytek z procesorami ARM.

Pierwsze zaskoczenie przyszło dosyć szybko, ponieważ okazało się, że płytka nie ma najnowszej wersji firmware'u, więc środowisko graficzne nie działało. Rozwiązanie było proste, ale niestety wymagało kabla USB-A — USB-A, którego nie posiadałem w swoim składziku, więc doszedłem do wniosku, że póki co sam terminal mi wystarczy.

Sama konfiguracja asystenta była dosyć prosta. Do płytki łączę się przez SSH. Repozytorium jest hostowane na GitHubie. Dzięki użyciu Tailscale i GitHub Actions zmiany po pushu przechodzą przez testy i wykonywany jest deploy bezpośrednio na płytkę. Czysto i elegancko.

Na koniec warto odpowiedzieć sobie na pytanie, czy nowa płytka przyspieszyła działanie asystenta. Po konfiguracji nie odłączyłem od prądu Raspberry Pi i wysłałem zapytanie przez Telegram. Otrzymałem dwie odpowiedzi — pierwsza przyszła po kilku sekundach, druga po kilkunastu. 620 zł. Było warto.