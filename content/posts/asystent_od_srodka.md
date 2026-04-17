+++
date = '2026-04-17T13:26:36+02:00'
draft = false
title = 'Asystent od środka'
+++

Zgodnie z obietnicą z poprzedniego posta dzisiaj skupię się na funkcjonalnościach swojego agenta. Od razu zaznaczę, że nie jest tak rozbudowany jak swój pierwowzór oraz nie ma dostępu do katalogu gotowych umiejętności. Każda nowa funkcja musi zostać napisana od nowa. 

Komunikacja z botem odbywa się w najprostszy możliwy sposób czyli przez Telegram. Nie była to moja platforma pierwszego wyboru, ale sposób konfiguracji był na tyle prosty, że nie chciałem na start tego komplikować. 
Funkcjonalności asystenta mógłbym podzielić na dwie grupy: pasywne i aktywne. 
Funkcje pasywne zarządzane są przez CRON, który odpala dwie funkcje: o godzinie 6 wysyła podsumowanie pogody, a o godzinie 6:30 codzienną ciekawostkę.  
Funkcje aktywne to możliwość odpytania o bieżącą pogodę, ciekawostkę, podsumowanie przesłanego artykułu, wyszukanie informacji w internecie oraz nowo dodana funkcja czyli podsumowanie filmów z YT. W każdym momencie można również wysłać bezpośrednie pytanie do LLM-a. Uruchomienie tych funkcji jest możliwe za pomocą "/" i zdefiniowaną nazwą usługi czyli przykładowo "/pogoda" lub za pomocą przykładowej wiadomości "jaka jest dzisiaj pogoda?". 

Podsumowując:

```
CRON: pogoda, ciekawostka
```
```
Interaktywne: pogoda, ciekawostka, podsumowanie artykułu, podsumowanie filmu z YT, wyszukanie w internecie, wiadomość do LLM
```

Krótkie wyznanie: praktycznie nie korzystam z funkcji interaktywnych, a największa korzyść mam z podsumowań zarządzanych przez CRON. Wydaje mi się, że to dobry przykład potwierdzający to, że nie każdy projekt do nauki musi mieć sens. 

Dla lepszego zrozumienia jak działa mój asystent:

* Telegram: "jaka jest pogoda?"
* Asystent: funkcja do zarządzania botem wysyła request typu *call_llm_with_tools*
* LLM: zwraca informację wybierz narzędzie *get_weather*
* Asystent: uruchamia funkcję *tasks/weather.py*, która wykonuje request do API pogodowego
* LLM: otrzymuje dane JSON z opisem pogody i na podstawie prompta przygotowuje je w odpowiednim formacie
* Asystent: odpowiedź z LLM jest przesyłana jako wiadomość na Telegram

Finalna wiadomość z podsumowaniem pogody wygląda następująco:

> Pogoda w Lublinie (15 kwietnia 2026):
> 🌡️ Teraz: 8.3°C (odczuwalna: 6.6°C)
> ☁️ Warunki: Częściowe zachmurzenie
> 💨 Wiatr: 10.1 km/h z południowego wschodu
> 💧 Wilgotność: 61%
> 🌅 Wschód: 05:35 | Zachód: 19:26
> 📅 Prognoza na dziś:
> - Max: 13.9°C | Min: 7.2°C
>
> Dzień zapowiada się dość spokojnie z umiarkowanymi temperaturami. Nie będzie padać, więc parasol możesz zostawić w domu. Warto ubrać się warstwowo, ponieważ poranek jest chłodny, choć w ciągu dnia zrobi się przyjemniej. Cienka kurtka lub sweter będą odpowiednim wyborem na dzisiejszą aurę.

To czego nie ma w powyższym opisie to informacja, że cała sesja jest logowana do Langfuse. Polecam każdemu wdrożenie narzędzia do observability jak najwcześniej, ponieważ dużo łatwiej rozwiązuje się napotkane problemy. W moim przypadku była to podwójna generacja wiadomości z podsumowaniem pogody. Pierwsza iteracja tej funkcjonalności miała o jeden krok za dużo więc podsumowanie wygenerowane w oparciu o mój prompt trafiało po raz kolejny do LLM i traciło wcześniej zdefiniowaną formę. 

Uważny czytelnik pewnie zauważy, że do funkcji *get_weather* nie jest dołączany argument z informacją na temat miasta dla którego ma być pobrana pogoda. Przyznam, że poszedłem tu na łatwiznę i nazwa jest zahardkodowana w funkcji. Nie sprawdzam pogody dla innych miasta więc taki wybór wydawał mi się naturalny.

Na dzisiaj to tyle. W kolejnym poście w końcu przejdziemy do, moim skromnym zdaniem, dużo ciekawszej części związanej z samą płytką oraz małymi modelami, które jak się okazuje mają swoje niespodzianki.