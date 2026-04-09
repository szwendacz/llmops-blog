---
title: "Trudno wymyślić dobry tytuł"
date: 2026-04-09
draft: false
tags: ["llm", "raspberry", "sbc"]
---

Ciężko powiedzieć od czego zacząć. W sierpniu tamtego roku doszedłem do wniosku, że zacznę pisać bloga na temat LLM. Chodziło mi przede wszystkim o uporządkowanie mojej wiedzy. Tydzień po napisaniu pierwszego posta straciłem pracę i siłą rozpędu napisałem jeszcze dwa nowe posty. Patrząc na nie z perspektywy czasu nie były zbyt dobre, a dodatkowo chyba chciałem uchodzić za większego eksperta niż wtedy byłem. Na tym temat się zakończył, a ja zostałem z domeną i serwerem, z którego w ogóle nie korzystałem.

Minęło parę miesięcy, znalazłem nową pracę, w której zajmuję się ewaluacją i testowaniem trzech różnych aplikacji wykorzystujących LLM. Przy okazji praca okazała się być na tyle interesująca i spokojna, że w końcu wróciła mi chęć do siedzenia wieczorami nad swoimi rzeczami.

Tutaj dochodzimy do momentu, w którym pół świata kupowało Mac Mini i instalowało OpenClaw (używam już nazwy ostatecznej, żeby nie wprowadzać zbędnego zamieszania). Pomyślałem, że asystent to na tyle ciekawy temat, że warto go przetestować osobiście i zainstalowałem... PicoClaw. Za opisem z GitHuba: "Tiny, Fast, and Deployable anywhere — automate the mundane, unleash your creativity". Oczywiście przy okazji pomyślałem, że fajnie byłoby, gdyby ten projekt miał jakąś namiastkę fizyczności (ten motyw będzie się powtarzać co jakiś czas), więc zamówiłem rekomendowaną płytkę SBC. Zamówienie było złożone za pośrednictwem AliExpress, więc musiałem cierpliwie odczekać dwa tygodnie.

W końcu listonosz zapukał do moich drzwi i przekazał mi przesyłkę. Nie ukrywam, że trochę się zdziwiłem, gdy po otwarciu paczki w środku nie było płytki, tylko moduł kamerki. Z tego miejsca chciałbym podziękować osobie, która projektowała opcje wyboru w aukcjach na AliExpress. Finalnie zamiast płytki miałem kamerę, która nie była mi potrzebna (teraz trochę żałuję, że ją odesłałem).

Na szczęście w takich momentach ludzka pamięć wykonuje wewnętrzne salto i przypomniałem sobie, że miałem kiedyś w pudełku z kablami Raspberry Pi Zero 2W. Nie zastanawiając się długo ruszyłem na poszukiwania i po chwili byłem znowu w grze. Płytka działała jakby dopiero co została wyprodukowana, a nie przeleżała w szafie ostatnie 2 lata. Sama instalacja PicoClaw z pomocą Claude Code przebiegła bez problemu. Płytka działała lokalnie w mojej sieci domowej. Długo się nie zastanawiając przekazałem pierwsze zadanie mojemu asystentowi — codziennie o 6 rano chciałbym dostawać podsumowanie pogody dla mojego miasta. Agent przyjął zadanie, zapewnił że wszystko gra i pogoda będzie jutro o godzinie 6 rano. Krótko mówiąc — pogoda nigdy do mnie nie dotarła. Nie wiem czy to mój brak zrozumienia założeń tego projektu czy jego złożoność, ale próba egzekucji zadania CRON przekroczyła moje siły i cierpliwość. Poddałem się po paru dniach i nieudanych próbach.

Jakiś czas później trafiłem na artykuł opisujący jak skonfigurować oryginalnego asystenta OpenClaw na Hetzner za całe 5$ miesięcznie. Nie lubię się tak łatwo poddawać, więc doszedłem do wniosku, że to świetna okazja, żeby w końcu zacząć otrzymywać swoje podsumowanie pogody. "Dojrzały" projekt, stabilna infrastruktura oraz duży model LLM wywoływany przez OpenRouter.

Co mogło pójść nie tak? W skrócie: dokładnie to samo co w PicoClaw. Asystent miał zdecydowany problem z kategoryzacją mojego podsumowania pogody i finalnie nie trafiało ono do CRONa (jeśli jakiś w ogóle był — dzisiaj nie jestem tego taki pewien).

W tym momencie byłem już naprawdę blisko porzucenia swojego pomysłu z pogodą, ale spojrzałem na ciągle uruchomioną malinkę i zadałem sobie proste pytanie: czy tak trudno będzie napisać własnego asystenta? Nie będę nikogo trzymać w napięciu — okazało się, że nie.

Z pomocą Claude Code i Pythona stworzyłem własnego asystenta, który jest ubogim krewnym bardziej popularnych projektów. Nie mam możliwości dodawania nowych umiejętności z marketplace, ale wszystkie potrzebne funkcje działają bez żadnego problemu. Co najważniejsze — w końcu codziennie rano o godzinie 6 dostaję podsumowanie pogody. To wszystko działa na Raspberry Pi Zero 2W. W gruncie rzeczy powinienem powiedzieć, że działało — bo od wczoraj (tj. 08.04.2026) mam już nową płytkę SBC, po której obiecuję sobie naprawdę wiele. O szczegółach i funkcjonalności mojego asystenta opowiem w kolejnym poście.
