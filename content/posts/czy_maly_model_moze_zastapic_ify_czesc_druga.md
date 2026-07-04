+++
date = '2026-07-04T14:05:00+02:00'
draft = false
title = 'Czy_maly_model_moze_zastapic_ify_czesc_druga'
+++

Powinienem napisać tego posta dawno temu, ale nie mogłem się do niego zebrać ponieważ uznałem, że to będzie ostatni post z serii, która początkowo miała liczyć sześć wpisów. Już wyjaśniam skąd taka decyzja. 

Dla przypomnienia - mój eksperyment polegał na sprawdzeniu jak mały model poradzi sobie w podejściu edge AI w kontekście interpretacji docierających sygnałów i wyborze odpowiedniego toola. [Tutaj](https://llmops.pl/posts/czy_maly_model_moze_zastapic_ify/) można poczytać więcej o założeniach. 

Punktem wyjścia dla wszystkich eksperymentów była weryfikacja modelu bazowego, aby mieć punkt odniesienia. Tak jak widać poniżej, model bez żadnych modyfikacji radzi sobie bardzo przyzwoicie osiagając prawie 80% w obydwu kategoriach. Dla przypomnienia używam Qwena w rozmiarze 0.6B. 
Model miał ustawioną flagę /no_think, temperatura wynosiła 0, a średnie latency na jedno zapytanie to ~4s.
Dodatkowo w trakcie testu modelu bazowego zweryfikowałem również opcje z włączonym reasoningiem, ale zysk wzrósł o 3 punkty procentowe przy latency, które zwiększyło się do ~29s. Tak mała różnica w wyniku nie usprawiedliwiała tak dużej różnicy w prędkości. 

| Kategoria | n | tool_correct | params_correct |
|-----------|---|:------------:|:--------------:|
| correct | 40 | 100% | 100% |
| fuzzy_params | 50 | 80% | 97% |
| out_of_scope | 50 | 60% | — |
| **overall** | **140** | **79%** | **79%** |


Kolejnym eksperymentem było 'Ograniczenia gramatyczne GBNF' i tutaj sytuacja wyjaśniła się bardzo szybko. Qwen3-0.6B z użyciem structured output w 100% przypadków poprawnie generował format odpowiedzi, więc uznałem, że dołożenie dodatkowego mechanizmu, który będzie wymuszał format outputu nie ma sensu bo na własne życzenie wprowadzę redundancje. Temat odpuściłem.

Eksperyment trzeci czyli 'Klasyfikator kaskadowy - bramka przed modelem 0.6B' zapowiadał się bardzo ciekawie. 

Pipeline zaprojektowany pod eksperyment składał się z 3 kroków:

* Reguły deterministyczne — dopasowanie kluczy wejściowych do narzędzia, obsługiwał przypadki z kategorii correct 
* Klasyfikator TF-IDF — dla przypadków których reguły nie obsługują czyli przypadki z kategorii fuzzy_params
* Model językowy jako rezerwa — dla pozostałych przypadków

A jak wyglądały rzeczywiste wyniki?

| Kategoria | exp01 | exp03 |
|-----------|-------|-------|
| correct | 40/40 | 40/40 |
| fuzzy_params | 40/50 | 39/50 |
| out_of_scope | 26/50 | 50/50 |

Co jest kluczową informacją z tego eksperymentu? Model językowy angażowany był 5 razy na 140 przypadków zamiast 140 na 140, a średnie latency spadło znacznie poniżej 1s z racji bardzo rzadkiego wykorzystania LLM-a. 

Po zobaczeniu podsumowania tego eksperymentu, a w szczególności liczby requestów do LLM-a, zadałem sobie pytanie czy to co robię ma sens. Teoretycznie eksperyment zakończył się powodzeniem, ponieważ dowiodłem, że użycie bramki poprawiło wyniki oraz zmniejszyło latency. Z drugiej strony ten sam eksperyment pokazał, że w przypadku mojego datasetu mały model jest wywoływany bardzo rzadko, a zdecydowana większość przypadków może być klasyfikowana za pomocą dużo prostszych i szybszych metod. Miałem do wyboru dwie ścieżki: zmienić dataset na trudniejszy, co wymagałoby powtórzenia wszystkich eksperymentów i wymyślenia problemu badawczego od nowa, albo przyznać, że dla tej skali problemu istnieją lepsze metody na analizę sygnałów niż mały model językowy.

Po głębszym zastanowieniu doszedłem do wniosku, że dalsze kontynuowanie tego projektu w obecnej formie nie ma sensu — nie dlatego, że eksperyment się nie udał, tylko dlatego, że udał się zbyt dobrze. Udowodniłem, że kaskada reguł i klasyfikatora TF-IDF załatwia niemal cały ruch, a mały model językowy jest potrzebny tylko jako ostateczne zabezpieczenie na pojedyncze przypadki. To zamyka problem, który sobie postawiłem na starcie, tylko że odpowiedź jest inna niż się spodziewałem: mały LLM rozwiązuje problem mniejszej skali, niż początkowo zakładałem. Mógłbym sprawdzić pozostałe metody z listy, ale przyznam szczerze, że straciłem do tego motywację — lubię robić coś, co przynajmniej w teorii ma praktyczne zastosowanie, a tutaj ta pewność mi się rozmyła.
Wydaje mi się, że moim kluczowym wnioskiem na przyszłość jest to, że przed wyborem narzędzia należy zadać sobie pytanie, czy jest ono odpowiednie do skali problemu.

