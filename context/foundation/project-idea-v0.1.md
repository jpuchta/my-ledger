# My Ledger - MVP

## Główny problem

Monitorowanie i kategoryzowanie wydatków w domowych w przypadku posiadania wielu kont bankowych oraz wielu kart płatniczych lub kredytowych jest uciążliwe, gdyż wymaga ręcznego przepisywania tranzakcji z wykazów, które każdy z banków dostarcza w innym formacie, a także ręcznego przypisywania kategorii do każdej transakcji.

## Minimalny zestaw funkcjonalności

- System kont użytkowników
- Definiowanie i edycja kategorii tranzakcji
- Manualne dodawanie transakcji
- Przeglądanie i edycja dodanych tranzakcji
- Generowanie raportów miesięcznych w jednym zadanym formacie
- Definiowanie schematów plików CSV z tranzakcjami
- Wczytywanie zestawień transakcji dostarczanych w CSV (różne schematy) z ręcznym ustawianiem kategorii transakcji
- Identyfikacja tranzakcji wewnętrznych
- Identyfikacja duplikatów

## Co NIE wchodzi w zakres MVP

- Import formatów innych niż CSV (czyli PDF, XLSX itp.)
- Migracje wydatków po zmianie systemu kategoryzowania
- Automatyczne przypisywanie kategorii do tranzakcji
- Śledzenie salda kont bankowych
- Aplikacja mobilna (na początek tylko web)

## Kryteria sukcesu
- Użytkownik może zaimportować tranzakcje z wyciągów w formacie csv w przynajmniej dwóch różnych schematach, obejmujących przecinające się okresy czasu, nie pokrywające się z pełnymi miesiącami, a następnie wygenerować zestawienie tranzakcji z wskazanego miesiąca kalendarzowego.