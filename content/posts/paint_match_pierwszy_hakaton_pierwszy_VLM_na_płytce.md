+++
date = '2026-06-09T20:37:38+02:00'
draft = false
title = 'Paint match: pierwszy hakaton, pierwszy VLM na płytce'
+++

> Krótka uwaga zanim zaczniesz: post pisałem w trakcie hakatonu, jeszcze przed fine-tuningiem modelu. Aktualny stan produkcji (z nowym modelem) opisany w sekcji **Aktualizacja** na końcu.

Zdaje sobie sprawę z tego, że miałem opisać kolejne eksperymenty, które przeprowadzam w ramach *EDGE AI*, ale w poprzednią środę na moją skrzynkę trafił mail z Hugging Face z informacją na temat hakatonu [Build Small Hackathon](https://huggingface.co/build-small-hackathon). Muszę przyznać, że ten hakaton jest wyjątkowy, ponieważ motywem przewodnim są małe modele do 32B. Czy mogł trafić się ciekawszy temat? Szczerze wątpię! 

## Skąd pomysł?

Mając kilkanaście lat, uwielbiałem grę [Airfix Dogfighter](https://en.wikipedia.org/wiki/Airfix_Dogfighter) stworzoną przez szwedzkie studia Unique Development Studios i Paradox Entertainment na licencji firmy Airfix. 26 lat po premierze gry dalej jestem blisko związany z firmą Airfix, ponieważ to jeden z moich ulubionych producentów modeli. Tak, jestem modelarzem. Uwielbiam sklejać modele plastikowe, ponieważ to doskonała odskocznia od komputera. Dodatkowo miło zobaczyć fizyczne efekty swojej pracy. 

![Zestaw Airfix](/images/airfix/airfix-pudelko.JPG)

## Co zbudowałem?

Pomimo tego, że lubię modele Airfix, to niestety nie jestem fanem ich standardowych farb Humbrol. To trochę jak z Colą i Pepsi. Każdy ma swoje własne preferencje. Ja od Humbrola wolę farby Tamiya, ale zawsze denerwuje mnie konwersja kolorów. Przykładowo: kolor srebrny z Humbrol to *11 Silver*, ale odpowiednik Tamiya to *X-11 Chrome Silver Gloss*. Dlaczego w takim razie nie ułatwić sobie życia i, automatycznie za pomocą zdjęcia lub screenshotu, konwertować kod Airfix na kod Tamiya? Dodatkowo, motywem przewodnim UI może być gra, w którą zagrywałem się za dzieciaka, i mamy gotowy motyw na hakaton - Paint Match. 

*Poniżej pierwsze logo, które miało trafić do projektu.*

![Pierwsze logo](/images/airfix/logo.png)

## Jak to działa?

HF Space orkiestruje całość, Radxa serwuje wyłącznie model. Aplikacja Gradio na Space przez Cloudflare Tunnel woła Radxę po inferencję, łączy wyniki z inwentarzem z Google Sheets i zwraca do UI.

```
[Telefon/komputer użytkownika]
         ↓
[HF Space - Gradio UI + backend]
    ↓                    ↓ (Cloudflare Tunnel)
[Google Sheets]    [Radxa Q6A - llama-server + InternVL3.5-2B]
 inwentarz farb           inferencja
```

Inwentarz z Sheets pobierany jest raz, przy starcie Space'a. Mógłby to być JSON w repo, ale wtedy każda zmiana wymagałaby commita i redeploy'u - Sheets pozwala zaktualizować stan z telefonu.

![Płytka Radxa w 3D case](/images/airfix/radxa.JPG)

## Jak wybierałem model?

Od początku zakładałem, że projekt będzie budowany z myślą o zdobyciu odznaki "Off the grid", więc wiedziałem, że muszę znaleźć model, który będzie pasował do mojej płytki developerskiej [Radxa Q6A](https://radxa.com/products/dragon/q6a/). Pomimo pięknej strony internetowej, Radxa w moim przypadku to board CPU-only, ponieważ na systemie Armbian, którego używam, biblioteka Vulkan nie jest jeszcze wspierana. Podobnie wygląda sytuacja z modułem NPU - jest wspierany przez oficjalny system producenta Radxa OS, a liczba dostępnych modeli jest bardzo mała.

W każdym razie to pierwszy raz, gdy miałem okazję przetestować na niej model wspierający wizję. Do tej pory korzystałem głównie z modeli tekstowych. Całą pracę zacząłem od researchu możliwych modeli. Finalnie na krótkiej liście znalazły się:

* moondream2-1.5B
* SmolVLM2-2.2B 
* InternVL3.5-2B
* Gemma 4 E4B 

Uważny czytelnik od razu zauważy, że jeden model odstaje od reszty. Przyznaję, że Gemma trafiła na listę, bo była świeża i chciałem ją przetestować.

Testy zostały przeprowadzone na 10 plikach podzielonych na 2 równe grupy: zdjęcia instrukcji oraz screenshoty z oficjalnego sklepu Airfix. W ramach testów zależało mi przede wszystkim na sprawdzeniu 3 aspektów: halucynacje, dokładność oraz latency. Pomimo tego, że na ogół przy benchmarkach staram się podawać konkretne liczby, to tym razem nie mogę tego zrobić dla dwóch testowanych modeli. Modele *moondream2* oraz *SmolVLM2* kompletnie nie radziły sobie z odczytywaniem kodów farb. Na ogół myliły numery z kolorami lub zapętlały się w trakcie generowania odpowiedzi i produkowały halucynacje w postaci wyliczania kolejnych, następujących po sobie numerów. 
W przypadku Gemmy sytuacja była zgoła odmienna, ponieważ ten model doskonale radził sobie z odczytywaniem moich przykładów, ale dopiero po tym, jak zwiększyłem timeout odpowiedzi z 300s na 600s. Uznałem, że tak wysokie latency nie jest tego warte.
Na placu boju pozostał InternVL3.5-2B i to było moje największe zaskoczenie: 

* F1 = 0.87
* P = 0.95 (Precision) 
* R = 0.81 (Recall)

Innymi słowy: model rzadko wymyśla kody, których nie ma, ale czasem nie wyłapuje wszystkich, które są. 
Niestety, z racji CPU-only na płytce, średnie latency procesowania dla wszystkich obrazów to ~194s. 
Sam czas procesowania byłby dłuższy, ale w czasie testów eksperymentowałem z resizem obrazu i wydaje mi się, że znalazłem złoty środek. Screenshoty przesyłane są w rozdzielczości 640px, ale z racji dobrej jakości tak mały rozmiar nie ma większego przełożenia na finalny output. Z kolei zdjęcia robione telefonem też początkowo były skalowane do 640px, ale ten typ obrazu jest podatny na różnego rodzaju zniekształcenia (światło, odległość od instrukcji), więc finalnie ten typ obrazu jest zmniejszany do 960px. Przy tak małym modelu takie szczegóły mają znaczenie. 
Dodatkowym problemem, którym należało się zająć, był format odpowiedzi, ponieważ model bez wprowadzenia sztywnych zasad potrafił za każdym razem generować inny format outputu. Zacząłem od ograniczenia GBNF, ale przy zastosowanej temperaturze 0 model niestety się zapętlał w halucynacjach i nie potrafił wygenerować końcowego tokenu end of stream (EOS). Rozwiązaniem problemu było wdrożenie JSON schema. 

Na koniec tej sekcji wspomnę jeszcze o prompcie, ponieważ było to dla mnie jedno z większych zaskoczeń przy pracy z tym modelem. Finalny prompt brzmi: **List every paint code visible in this image.**. Nie chcę skłamać, ale jest to jeden z najkrótszych promptów jakich używam w swoich aplikacjach. Oczywiście jego długość ma swoje uzasadnienie. Dla przykładu dokładniejszy prompt **List every paint code visible in this image. Output one per line, exactly as `- CODE: NAME`, and nothing else.** psuł jakość outputu i wprowadzał większą randomowość w odpowiedzi modelu. Po paru próbach różnych kombinacji zostawiłem najprostszą wersję. 

## Jak to wygląda? 

Na potrzeby hakatonu przygotowałem dwie wersje UI. Wersja pierwsza korzystała z defaultowych komponentów Gradio i wyglądała jak standardowa aplikacja Gradio: nudno. 

![Pierwsza wersja UI](/images/airfix/UI-v1.png)

Doszedłem do wniosku, że z takim designem świata nie zawojuję, więc przygotowałem wersję drugą, która zdecydowanie bardziej oddawała klimat lat dwutysięcznych i gry Airfix Dogfighter. W tej wersji dodałem również więcej elementów oddających klimat militarny:

* Upload Document - po załadowaniu pliku zmienia się w ramkę "Exhibit A - Recovered Document" ze zdjęciem i pieczątką RECEIVED. Podczas analizy animowana zielona linia skanera przewija się przez dokument.
* Field Report - w stanie bezczynności: animowany radar. Podczas analizy: pulsujące statusy. Po zakończeniu: surowy output z modelu.
* Resupply Manifest - lista farb z linkami do sklepu mojehobby.pl. Posiadane farby oznaczone są zieloną etykietą ✓ ISSUED zamiast linku.

![Druga wersja UI - stan początkowy](/images/airfix/UI-v2.png)

![Druga wersja UI - załadowany plik](/images/airfix/UI-v3.png)

![Druga wersja UI - wyniki](/images/airfix/UI-v4.png)


Całość najlepiej sprawdzić bezpośrednio w aplikacji: [Paint Match HF Space](https://huggingface.co/spaces/build-small-hackathon/paint_match).
Aplikacja jest w pełni responsywna, co jest tu kluczowe - zdjęcia instrukcji robi się głównie telefonem, prosto przy stole modelarskim.

## Co dalej? 

Aplikacja może być rozwijana w dwóch kierunkach. Z technicznego punktu widzenia nie ma co ukrywać: model 2B popełnia błędy, szczególnie w przypadku gorszej jakości zdjęć. Rozwiązaniem byłoby wdrożenie lepszego modelu, ale przy posiadanym hardware nie ma to sensu z powodu wysokiego latency. Sprawdziłem to już z Gemmą E4B - odpadła w testach przez latency. Najlepszym rozwiązaniem byłaby zmiana płytki oraz weryfikacja większych modeli. Bez zmiany sprzętu jedynym kierunkiem jest fine-tuning modelu w celu lepszego odczytywania kodów. 

Po stronie funkcjonalności warto wspomnieć o dwóch rzeczach: dodanie kolejnych producentów (Vallejo, AK, Hataka) oraz wizualne porównanie kolorów po wartościach HEX. Same nazwy potrafią mylić - Humbrol Silver i Tamiya Silver to niekoniecznie ten sam odcień, a właściwy odpowiednik kolorystyczny może nazywać się zupełnie inaczej (np. Silver Steel Matt).

## Podsumowanie

To był mój pierwszy udział w hakatonie i żałuję, że nie zrobiłem tego wcześniej. Zaskoczyła mnie liczba decyzji, które trzeba było podjąć przy tak małym projekcie.
Dodatkowo formuła hakatonu wymaga twardego trzymania się wcześniej ustalonych priorytetów. Wymyślanie nowych funkcjonalności jest bardzo proste, ale ich sensowne wdrożenie to już inna kwestia. Wisienką na torcie było pierwsze użycie modelu wizyjnego - czuję, że nie ostatnie.
Czy jestem zadowolony z finalnego efektu? Jeszcze jak! 

## Aktualizacja: fine-tuning i sponsorska pula nagród

Wymagane kroki hakatonu dopiąłem szybciej niż myślałem, więc do terminu zostało parę dni, które wykorzystałem na dwie dodatkowe rzeczy: fine-tuning (osobna odznaka) i wymianę bazowego modelu na MiniCPM-V-4.6 (osobna pula nagród sponsora) — który teraz siedzi w produkcji na Radxie.

MiniCPM-V-4.6 to model 1B w hybrydowej architekturze Qwen3.5-Mamba — lżejszy niż InternVL3.5-2B i z lepszym F1 jeszcze przed fine-tuningiem na moim benchmarku (0.920 vs. 0.87). Sam fine-tuning to LoRA na Modal.com z H100, 16 minut treningu i kilka dolarów, które poszły z puli którą dostawał każdy uczestnik hakatonu. Zbiór treningowy to 391 przykładów (242 ze screenshotów sklepu Airfix + 192 z papierowych instrukcji po augmentacji), zbiory testowe trzymane osobno.

**Wyniki (po fine-tuningu):**

| Zbiór | F1 |
|---|---|
| Benchmark 10 obrazków | 0.935 (wcześniej 0.920) |
| Przykłady z HF Space (4 obrazki) | 1.000 |
| Zbiór testowy ze sklepu (53 obrazki) | 0.927 |
| Zbiór testowy z papierowych instrukcji (7 obrazków) | 0.928 |

Co najlepsze, latency na Radxie spadło z ~194s do ~60s - trzy razy szybciej niż poprzedni model.

Jest jedno zdjęcie, które model dalej powoduje błędy ekstrakcji: Chinook, na którym numery kalek w czerwonych kwadratach wyglądają identycznie jak kody farb. Biorąc pod uwage ogólny score dla tak małego modelu oraz dużą poprawę latency uznałem, że jest to akceptowalny koszt. 



