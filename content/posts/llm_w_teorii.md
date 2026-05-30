+++
date = '2026-05-20T14:10:19+02:00'
draft = false
title = 'LLM w teorii'
+++

Ostatnio doszedłem do wniosku, że tak naprawdę nie wiem jak działają LLMy. Rozumiem ich działanie od strony praktycznej, ale część teoretyczną rozumiem raczej hasłowo. Uznałem, że to dobry pomysł na aplikację więc rozpisałem zadania i przez weekend zbudowałem ją z pomocą Clauda. Cały projekt jest hostowany na mojej płytce Radxa Q6A, która po raz kolejny okazała się jednym z najlepszych wydatków tego roku. 

Koncepcja jest bardzo prosta - przygotowałem syllabus zawierający 60 tematów pogrupowanych w bloki tematyczne. Codziennie o godzinie 8:30 dostępna jest nowa lekcja zawierająca świeże materiały. 

Dla lepszego kontekstu poniżej znajduje się blok pierwszy, który omawia właśnie podstawy teoretyczne. 

**Fundamentals (10)**
1. Tokenizacja — jak tekst staje się tokenami, BPE, wpływ na koszty
2. Embeddingi — czym są, jak powstają, różnica od tokenów
3. Architektura transformera — encoder vs decoder vs encoder-decoder
4. Attention mechanism — self-attention, multi-head attention, po co istnieje
5. Context window — limity, "lost in the middle", praktyczne konsekwencje
6. Temperature, top-k, top-p — mechanizmy samplingu
7. Base model vs instruction-tuned vs RLHF-aligned
8. Parametry modelu — co oznacza 7B, wpływ na pamięć i szybkość
9. Kwantyzacja — INT4/INT8, GGUF, wpływ na jakość vs szybkość
10. Hallucynacje — skąd się biorą, dlaczego modele kłamią pewnie 

Każdorazowo lekcja generowana jest na podstawie przygotowanego przeze mnie promptu. Korzystam z modelu GPT 5.4, ponieważ generuje moim zdaniem najciekawsze lekcje. Następnie na podstawie lekcji, kolejny prompt generuje quiz składający się z 8 pytań z jedną dobrą odpowiedzią. Dla podwyższenia stawki błędne odpowiedzi nie wyglądają na pierwszy rzut oka na błędne, ponieważ staram się w quizie unikać cytowania 1:1 treści lekcji. 

Po zakończeniu lekcji mogę zaznaczyć czy w kolejnym dniu chcę przejść przez ten sam temat raz jeszcze czy przechodzimy do kolejnego z listy. 

Aplikacja jak widać jest bardzo prosta, a UI czysty. W ramach ciekawostki powiem, że jednym z wymagań było to, aby lekcja oraz quiz prawidłowo działały na moim czytniku ebooków Kobo. Generowana strona to czysty HTML + CSS, ponieważ silnik przeglądarki na Kobo to WebKit, który słabo radzi sobie z JavaScriptem. Po paru iteracjach udało się to osiągnąć więc w trakcie nauki mogę skupić się na treści i nie mam żadnych rozpraszaczy. 

Codzienna nauka, nawet w tak prostej formie jak moja, wyrabia nawyk i siłą rzeczy ułatwia lepsze zrozumienie tego co się dzieje we wnętrzu modelu więc termin black-box testing każdego dnia jest mniej black;) 

![Czytnik-lekcja)](/images/llm_w_teorii/czytnik-lekcja.jpeg)

![Czytnik-quiz)](/images/llm_w_teorii/czytnik-quiz.jpeg)

![Desktop-lekcja)](/images/llm_w_teorii/desktop-lekcja.jpeg)

![Desktop-quiz)](/images/llm_w_teorii/desktop-quiz.jpeg)

## Zobacz też

- [Duże modele są nudne](/posts/duze_modele_sa_nudne/)
- [Czy mały model może zastąpić ify?](/posts/czy_maly_model_moze_zastapic_ify/)