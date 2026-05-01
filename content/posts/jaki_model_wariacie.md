+++
date = '2026-04-19T20:58:43+02:00'
draft = false
title = 'Jaki model wariacie?'
+++

Dygresja na początek.

Minęło już trochę czasu od momentu w którym robiłem to co będę opisywać i przyznaję, że nie pamiętam już wszystkich szczegółów. Muszę nad tym popracować bo czuję, że to będzie problem w przyszłości. 

### Problemy wieku dziecięcego

Formalności mamy już za sobą więc przejdźmy do głównego tematu jakim będzie wybór małego modelu, który jest mózgiem mojego asystenta.
Od samego początku chciałem żeby na nowej płytce pracował mały model i wiele obiecywałem sobie po dedykowanym procesorze na Radxa. Niestety jak to w życiu bywa sama specyfikacja techniczna bez oprogramowania nie działa i tak też było w tym przypadku. Oczywiście po części z mojej winy. 
Głównym winowajcą był system operacyjny, którego użyłem czyli [Armbian](https://armbian.com). Nie wiem co mnie podkusiło, że wybrałem ten system zamiast zmodyfikowanej wersji Ubuntu dostarczanej przez producenta. W każdym razie poskutkowało to tym, że procesor NPU nie jest w tym momencie dostępny na żadnym systemie poza tym oferowanym przez Radxa. 

Poniżej wszystkie zebrane problemy, które napotkałem w czasie próby uruchomienia NPU. Nie ukrywam, że moja wiedza na temat szczegółów jest bardzo niska, ponieważ tutaj brylował Claude: 

1. Brak reserved-memory node dla FastRPC w DTB — jądro nie rezerwuje pamięci DMA dla fastrpc; objaw: no reserved DMA memory for FASTRPC w dmesg; wymaga ręcznego patcha DTB.
2. Firmware CDSP niezgodne — wymagany konkretny wariant CDSP.HT.2.5.c3-00130-KODIAK-1; plik cdsp.mbn musi pasować do wersji sterownika.
3. Brak pakietów fastrpc/libcdsprpc na Armbiani — pakiety fastrpc, fastrpc-dev, libcdsprpc1, radxa-firmware-qcs6490 nie są dostępne w repozytoriach Armbiana; są tylko w obrazie Ubuntu 24.04 Radxa.
4. Vulkan niedostępny — Adreno 643L na Linuxie zgłasza uniformAndStorageBuffer16BitAccess = false; Mesa Turnip w tym stanie nie obsługuje compute shaderów potrzebnych do inference.
5. Brak OpenCL ICD dla Adreno — tylko software rusticl, brak sprzętowej akceleracji.

Podczas jednej z prób obejścia problemu bohatersko zbrickowałem płytkę. Na szczęście jestem tą osobą, która robi kopie zapasowe więc przywrócenie płytki do działania było proste. Przy okazji stałem się posiadaczem adaptera do dysków NVMe. 

W każdym razie konkluzja była taka, że powinienem przeinstalować system na oficjalny lub nie będę mógł korzystać z NPU. Muszę przyznać, że nie chciało mi się znowu bawić w stawianie Ubuntu więc postanowiłem spróbować z modelami, które korzystały z samego CPU. Biorąc pod uwagę brak wsparcia dla biblioteki Vulkan był to jedyna sensowna decyzja. 

### Ewaluacja

Duże modele siłą rzeczy są bardzo popularne, ale problem pojawia się podczas wyszukiwania tych zdecydowanie mniejszych. Okazało się, że nie ma żadnego leaderboardu dla SLM. Pomocny jak zwykle okazał się Reddit, dodatkowo trafiłem na ciekawy benchmark małych modeli w [tool callingu](https://github.com/MikeVeerman/tool-calling-benchmark). 

Mój system do ewaluacji miał być prosty. 

1. Ewaluacja w Promptfoo na podstawie 22 pytań, które miały za zadanie weryfikację tool calling.
2. Po selekcji modeli, próba na prawdziwych danych i weryfikacja w jaki sposób dany model obsługuje dłuższe treści. Chodziło tutaj głównie o poprawne wyświetlenie pogody zgodne z szablonem oraz podsumowanie artykułów. 

Moim pierwszym wyborem jeśli chodzi o uruchamianie modeli była Ollama. Od razu mogę zaznaczyć, że jeśli chodzi o LLM to jest to dobry wybór, ale w przypadku małych modeli sytuacja się komplikuje. Po pierwsze wybór małych modeli jest mocno ograniczony. Po drugie modele, które zgodnie z definicją powinny obsługiwać tool calling tego po prostu nie potrafią. Najlepszym przykładem jest model [lfm2.5-350m](https://ollama.com/LiquidAI/lfm2.5-350m), który w opisie reklamuje:

*Ultra-fast edge model for data extraction and tool use*

No i guzik bo po uruchomieniu i próbie odpytania definicja narzędzi okazuje się, że:

`API error: 400 Bad Request {"error":{"message":"registry.ollama.ai/LiquidAI/lfm2.5-350m:latest does not support tools"}}`

Po zajrzeniu w szczegóły modelu za pomocą komendy:

`ollama show LiquidAI/lfm2.5-350m`

okazuje się, że model który jest reklamowany dobrym wsparciem narzędzi w ogóle nie posiada takiej funkcjonalności:
 
*Capabilities: completion*

Dla przykładu informacja z modelu ministral3b:latest, który nie ma z tym problemu:

*Capabilities: completion, tools*

Nie chcąc dalej ryzykować stwierdziłem, że skorzystam z innej opcji i zainstalowałem LM Studio. Tutaj już nie było większych niespodzianek, ale polecam korzystać z modeli od sprawdzonych dostawców takich jak lmstudio-community, Bartowski lub unsloth. 

Mój prywatny komputer to Macbook Air M1 16GB i to na nim odbywały się wszystkie testy. Dla uczciwości zaznaczę, że na 100% miałem problem z LM Studio, ponieważ maksymalna ilość tokenów na sekundę jaką osiągnąłem to 5. Jest to zdecydowanie za niska wartość jak na taką konfigurację, ale doszedłem do wniosku, docelowo model będzie hostowany na innym urządzeniu więc rozwiązywanie problemów z laptopem to strata czasu. Na potrzeby ewaluacji nie miało to znaczenia, ale wydłużało sam proces. 

Przetestowałem łącznie 19 modeli, a podsumowanie przedstawia się następująco:

| Nazwa modelu | Score |
|-------|-------|
| qwen2.5-3b-instruct@q6_k | 18/22 |
| qwen2.5-3b-instruct@q4_k_m | 18/22 |
| llama-3.2-3b-instruct@q6_k | 15/22 |
| llama-3.2-3b-instruct@q4_k_m | 16/22 |
| ministral-3-3b-instruct-2512@q4_k_m | 14/22 |
| mistralai_ministral-3-3b-instruct-2512@q6 | 16/22 |
| qwen3.5-4b@q4_k_m | 20/22 |
| qwen2.5-1.5b-instruct@q6_l | 5/22 |
| llama-3.2-1b-instruct@q8 | 2/22 |
| phi-3.5-mini-instruct@q6_k | 14/22 |
| phi-3.5-mini-instruct@q4_k_m | 17/22 |
| phi-3-mini-4k-instruct@q4 | 2/22 |
| smollm3-3b-mlx | 18/22 |
| smollm3-3b@q6_k_thinking | 18/22 |
| smollm3-3b@q6_k_no_thinking | 20/22 |
| qwen3-0.6b@q8 | 17/22 |
| qwen3-1.7b | 21/22 |
| lfm2.5-1.2b-instruct | 11/22 |

Tak jak widać większość modeli w rozmiarze 3B osiąga przyzwoite wyniki, ale model o takiej wielkości na płytce miałyby problem z szybkością generowania tokenów. Doszedłem do wniosku, że lepiej postawić na mniejszy model kosztem jakości. Po przeanalizowaniu wyników wybór padł na qwen3-0.6, ale z delikatną zmianą. Finalnie wybrałem model w wersji Q4_K_M zamiast Q8. Mniejsza kwantyzacja odbiła się delikatnie na jakości. Dodatkowy test, który przeprowadziłem w promptfoo, dał wynik 15/22. Z drugiej strony model działał poprawnie na typowych pytaniach, a skoro jestem jedynym użytkownikiem swojego asystenta to nie mam problemu z przestrzeganiem pewnej konwencji, która zapewnia jakość i stabilność odpowiedzi. 

Zrezygnowałem również z drugiego kroku flow do ewaluacji. Stwierdziłem, że dalsze próby na moim laptopie nie mają sensu bo i tak nie pokażą faktycznej wydajności modelu na płytce. SLM ma służyć głównie jako router do wywoływania odpowiednich narzędzi więc stwierdziłem, że lepiej sprawdzić konkretne przypadki na Radxa i zweryfikować rzeczywiste osiągi.

### Wdrożenie

Po deploy Qwena zrobiłem pierwszy benchmark i przeżyłem niemiłą niespodziankę. Ilość tokenów na sekundę jakie był w stanie wyprodukować model to zatrważające 3,6. Model z taką wydajnością nie nadawał się nawet jako router do tool callingu. W tym momencie jeszcze bardziej żałowałem, że NPU nie jest dostępne. Na szczęście bywam czasem uparty więc po długiej dyskusji z Claudem udało się znaleźć winowajcę niskiej wydajności. 

Płytka Radxa Q6A posiada 8 rdzeni, które są podzielone w następujący sposób:

* cpu0-3: Cortex-A55 @ 1958 MHz (małe rdzenie efektywnościowe)
* cpu4-6: Cortex-A78 @ 2400 MHz (duże rdzenie wydajnościowe)
* cpu7: Cortex-A78 "Prime" @ 2707 MHz (najszybszy pojedynczy rdzeń)

Początkowe ustawienia llama.cpp wykorzystywały wszystkie rdzenie w trakcie generowania tokenów co powodowało większą konkurencję w dostępie do pamięci. Metodą prób i błędów udało mi się dojść do wyniku, który był bardziej niż zadowalający:

| Konfiguracja | Wydajność |
|---|---|
| wszystkie rdzenie | 3.5 tok/s |
| tylko A78 | 16.9 tok/s |
| tylko Prime A78 | **24.7 tok/s** |

### Podsumowanie

Muszę przyznać, że każdy problem, który napotkałem to kopalnia nowych informacji o których nie miałem wcześniej zielonego pojęcia. Gdybym miał powiedzieć co jest największą zaletą mojego projektu to nie będzie to wcale sam asystent, ale droga i trudności, które napotykam w trakcie pracy nad nim. Dodatkowo dla kogoś kto na co dzień pracuje tylko i wyłącznie z oprogramowaniem aspekt posiadania fizycznego urządzenia jest bardzo przyjemną odskocznią. 







