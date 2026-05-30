+++
date = '2026-05-24T14:22:30+02:00'
draft = false
title = 'Czy mały model może zastąpić ify?'
+++

Tym wpisem zaczynam całą serię postów związanych z moim aktualnym projektem - nazwałem go roboczo *Edge AI*. Celem projektu jest odpowiedź na pytanie czy model Qwen3-0.6B nadaje się do zastosowania w kontekście Edge AI.   

Na potrzeby projektu przyjąłem założenie, że wdrożenie zostało wykonane w firmie logistycznej. Od razu zaznaczę, że nie mam doświadczenia w tej branży więc chodzi tylko o zakotwiczenie projektu w "przemysłowej" narracji. Doszedłem do wniosku, że dzięki temu małemu zabiegowi projekt będzie bardziej rzeczywisty. 

Sam projekt powstał jako odpowiedź na pytanie, czy mały model może być na tyle wiarygodny, że będzie w stanie zastąpić logikę opartą na ifach. W przypadku czystych i jasno zdefiniowanych sygnałów logika oparta na ifach jest zdecydowanie lepsza, ale chciałem sprawdzić, jak model odnajdzie się w środowisku, którego opisanie ifami byłoby niemożliwe. 

Na potrzeby tego projektu stworzyłem 5 możliwych do użycia w firmie logistycznej narzędzi wraz z parametrami, które przyjmują: 


* log_temperature(value_celsius, location)
* check_door_status(is_open, location)
* verify_cargo(manifest_id, rfid_tags)
* check_route_feasibility(weight_kg, fuel_liters, distance_km)
* send_alert(event_type, severity)

Dodatkowo powstał dataset składający się ze 140 przykładów podzielonych na 3 kategorie:

* correct - wywołaj odpowiednie narzędzie z odpowiednimi parametrami
* fuzzy_params - wywołaj odpowiednie narzędzie, ale parametry są niejednoznaczne, dodatkowo dla skomplikowania datasetu ta kategoria posiada dwa rodzaje danych: zaszumione czyli prawidłowe wartości wraz z nieistotnym szumem oraz wartości sprzeczne w których model musi zdecydować jaką wartość wybrać
* out_of_scope - nie wywołuj narzędzia, brak narzędzia dla danego przypadku

Krótki wycinek każdej kategorii:

Correct:
```
  {
    "id": 1,
    "category": "correct",
    "input": {
      "temp_c": 22.8,
      "zone": "at_client"
    },
    "expected_tool": "log_temperature",
    "expected_params": {
      "value_celsius": 22.8,
      "location": "at_client"
    }
```

Fuzzy params noisy:
```
  {
    "id": 52,
    "category": "fuzzy_params",
    "fuzzy_type": "noisy",
    "input": {
      "temp_c": 46.5,
      "zone": "at_depot",
      "speed_kmh": 84
    },
    "expected_tool": "log_temperature",
    "expected_params": {
      "value_celsius": 46.5,
      "location": "at_depot"
    }
```

Fuzzy params contradictory:
```
    "id": 92,
    "category": "fuzzy_params",
    "fuzzy_type": "contradictory",
    "input": {
      "weight_kg": 12089,
      "fuel_liters": 306,
      "distance_km": 842,
      "planned_distance_km": 400
    },
    "expected_tool": "check_route_feasibility",
    "expected_params": {
      "weight_kg": 12089,
      "fuel_liters": 306,
      "distance_km": 842
    }
```

Out of scope:

```
    "id": 114,
    "category": "out_of_scope",
    "input": {
      "threshold_override": true,
      "cruise_control": true,
      "alarm_type": "tire",
      "alarm_triggered": false
    },
    "expected_tool": null,
    "expected_params": null
  }
```

Tyle tytułem wprowadzenia do datasetu - seria będzie podzielona na kilka części, które wyglądają następująco:
1. Deterministyczna walidacja wywołań narzędzi — baseline do którego będę porównywać kolejne eksperymenty
2. Ograniczenia gramatyczne GBNF — generowanie z użyciem gramatyki GBNF
3. Klasyfikator kaskadowy — lekki klasyfikator jako bramka przed modelem 0.6B
4. Klasyfikator Pass/Fail — walidacja outputu po generacji
5. Dostrajanie LoRA — trening w chmurze, wdrożenie w formacie GGUF na Q6A
6. Klasyfikator switch — dynamiczne routowanie między modelami	

W ramach powstawania kolejnych wpisów będę je linkował powyżej więc już teraz zapraszam na kolejny post o tym, jak oryginalny Qwen poradził sobie z moim datasetem.

## Zobacz też

- [Radxa Dragon Q6A](/posts/radxa_dragon_q6a-/)
- [Jaki model, wariacie?](/posts/jaki_model_wariacie/)