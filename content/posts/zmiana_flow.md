+++
date = '2026-05-03T12:02:46+02:00'
draft = false
title = 'Zmiana flow'
+++

### Zmiana

Nie wiem co mnie podkusiło, ale postanowiłem zmienić flow działania swojego asystenta. Nie znaczy to, że poprzednio działał źle. Model wielkości 0.6B całkiem dobrze radził sobie z moimi zadaniami biorąc pod uwagę, że nie wymagałem od niego nic ponad to co wcześniej sprawdziłem. Miał służyć głównie jako router do tool callingu. 
Pod względem wydajności sama zmiana nie ma za dużo sensu bo dokłada sporo dodatkowej logiki, plus bardzo "usztywniła" flow przepływu danych. Z drugiej strony była ciekawa pod względem edukacyjnym więc jestem zadowolony.

Poniżej krótkie przedstawienie starego i nowego flow:

| Krok | Stare flow | Nowe flow |
|------|-----------|-----------|
| 1 | Wiadomość trafia do `dispatch()` | Wiadomość trafia do `dispatch()` |
| 2 | Buduj messages z historią i system promptem | Sprawdź czy jest URL w wiadomości |
| 3 | Wyślij do OpenRouter z `TOOL_SCHEMAS` | Jeśli URL: wybierz `summarize_article` lub `summarize_youtube` bez LLM |
| 4 | OpenRouter decyduje które narzędzie wywołać | Jeśli brak URL: **klasyfikator MLP** decyduje o narzędziu lokalnie |
| 5 | OpenRouter zwraca nazwę narzędzia + argumenty | Jeśli `get_weather` lub `get_fun_fact`: wywołaj narzędzie bez argumentów |
| 6 | Wykonaj narzędzie | Jeśli `web_search`: llama-server (0.6B + **GBNF**) ekstrahuje argument `query` |
| 7 | Wynik wraca do OpenRouter | Jeśli `none`: llama-server (0.6B) odpowiada bezpośrednio |
| 8 | OpenRouter generuje finalną odpowiedź | Wykonaj narzędzie (jeśli dotyczy) |
| 9 | Zwróć odpowiedź | Zwróć wynik (PASSTHROUGH) lub odpowiedź llama-server |

### Co nowego?

Nowe flow ma dwie ciekawe rzeczy z którymi wcześniej nie miałem do czynienia. Chodzi o klasyfikator MLP i GBNF. 

### *Klasyfikator*

Bardziej szczegółowe flow wywoływania narzędzi wygląda następująco:

* Czy w zapytaniu jest URL? → tak/nie (regex https?://)
* Jeśli tak — czy URL zawiera youtube.com lub youtu.be? → summarize_youtube
* Jeśli tak, ale nie YouTube → summarize_article
* Jeśli nie ma URL → klasyfikator decyduje spośród 4 klas + none

To co najciekawsze dzieje się na ostatnim kroku. Szybkie przypomnienie czym jest klasyfikator MLP:

> Klasyfikator MLP (Multilayer Perceptron) to popularny rodzaj sztucznej sieci neuronowej służący do klasyfikacji danych, czyli przypisywania ich do określonych kategorii. 

W moim przypadku klasyfikator został wytrenowany na dosyć małym zbiorze danych, który obejmował 300 przykładów podzielonych na 4 kategorie:

* get_weather — 75 przykładów
* web_search — 75 przykładów
* get_fun_fact — 75 przykładów
* none — 75 przykładów

Dane treningowe to po prostu możliwe zwroty i ich wariacje, których mogłem użyć w celu wywołania danego narzędzia. Poniżej krótki wycinek dla każdego z nich.

```
{
  "get_weather": [
    "jaka pogoda",
    "pogoda w warszawie",
  ],
  "web_search": [
    "jaka cena zlota dzisiaj",
    "kurs euro do zlotego teraz",
  ],
  "get_fun_fact": [
    "powiedz mi ciekawostke",
    "jakas ciekawostka",
  ],
  "none": [
    "co to jest fotosynteza",
    "jak dziala neuralna siec",
  ]
}
```

Generalnie jak na tak mały dataset jakość klasyfikacji jest moim zdaniem bardzo dobra. Dokładność na zbiorze testowym to 0.93, a dla poszczególnych funkcji tak jak poniżej:

* get_fun_fact - 0.93      
*  get_weather - 1.00      
*         none - 0.87      
*   web_search - 0.93

Zgodnie z tym co było do przewidzenia najmniejszą dokładność ma kategoria NONE, która odnosi się do wiedzy samego modelu, ale wydaje mi się, że to całkiem normalne - takie zapytania są najtrudniejsze do przypisania, a z drugiej strony nie używam małego modelu do ogólnych pytań.  

Podsumowując: dzięki regułom regex, które filtrują funkcje summarize_youtube/summarize_article oraz klasyfikatorowi, filtruję na wejściu wywołania dostępnych narzędzi i upewniam się, że zostanie wywołane prawidłowe.

### *GBNF*  

Co tak właściwie kryje się pod tym tajemniczym skrótem? Przyznam szczerze, że do momentu aż nie zacząłem tego projektu nigdy nie słyszałem o czymś takim.

> GBNF (GGML Backus-Naur Form) to język definiowania gramatyk, stworzony w celu nakładania ograniczeń na wyniki generowane przez duże modele językowe. 

A jak to wygląda w praktyce? Do tej pory wiedziałem, że jestem w stanie kontrolować wyjście modelu za pomocą Pydantic/structured outputs. To, czym się różnią te dwie metody, to moment w którym dane podejście zaczyna działać: Pydantic już po generacji odpowiedzi, a gramatyka *w trakcie* generacji, ponieważ nie pozwala wygenerować odpowiedzi spoza zasad dołączanych do requestu. Jest to swoisty przykład samplowania tokenów tak jak powszechnie znana temperatura. 

### Jak to się sprawdza?

Nie ukrywam, że bardzo dobrze. Czasem zdarzają się potknięcia w przypadku wywołania web_search, ale liczba błędów jest mała. Najważniejsze jest dla mnie to, że praktycznie całość logiki stojącej za wywoływaniem narzędzi jest zlokalizowana lokalnie na płytce. 
Z praktycznego punktu widzenia ta zmiana nie miała sensu, ponieważ przy mojej liczbie narzędzi mogłem skorzystać z dużego modelu i miałbym 100% pewność, że każde wywołanie będzie prawidłowe. Podkreślałem to już pewnie parę razy, ale dalej będę to robić - ten projekt nie musi być sensowny. Jego głównym celem jest nauka i zmiana flow idealnie wpisała się w to założenie. 


