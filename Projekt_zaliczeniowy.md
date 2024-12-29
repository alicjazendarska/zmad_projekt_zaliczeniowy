# Moduł 8 - Wprowadzenie do programowania obiektowego.

**Wymagania projektu **

**Zadanie 1**  
Projekt polega na stworzeniu modułu, który może być zbiorem funkcji i/lub obiektów z metodami i ta druga 
metoda wydaje się lepszym rozwiązaniem. Moduł będzie operował na wybranym przez studenta datasecie pobranym
ręcznie przez studenta na dysk. Zbiory danych można znaleźć pod adresem https://archive.ics.uci.edu/ml/index.php .
Moduł powinien umożliwiać jego zaimportowanie w interaktywnej konsoli i uruchomienie jego funkcji z wiersza
poleceń.
Funkcjonalność, którą ma dostarczać moduł:
- Wczytanie datasetu – funkcja, która po podaniu ścieżki (nazwa pliku, jeżeli w tym samym folderze) wczyta
dane z pliku do listy (można użyć modułu csv). Dodatkowo funkcja przyjmuje parametr określający czy
pierwszy wiersz pliku zawiera etykiety kolumn czy nie. Jeżeli tak to etykiety wczytywane są do oddzielnej
listy.
- Wypisanie etykiet – funkcja wypisująca etykiety lub komunikat, że etykiet nie było w danym datasecie.
- Wypisanie danych datasetu – funkcja wypisuje kolejne wiersze datasetu. Bez podania parametrów
wypisywany jest cały dataset, ale możliwe też podanie 2 parametrów, które określają przedział, który ma
zostać wyświetlony (na wzór slice),
- Podział datasetu na zbiór treningowy, testowy i walidacyjny. Funkcja przyjmuje 3 parametry określające
procentowo jaka część głównego zbioru danych trafia do poszczególnych zbiorów.
- Wypisz liczbę klas decyzyjnych – wypisanie krotek gdzie pierwsza wartość to wartość klasy (np. nazwa irysa,
dla binarnych 0 lub 1 itd.), a druga to liczebność (kardynalność) tej klasy.
- Wypisz dane dla podanej wartości klasy decyzyjnej – wypisuje wiersze z zadaną wartością klasy decyzyjnej.
- Zapisanie danych do pliku csv – jako parametr przyjmowana jest dowolna lista, która może być podzbiorem
datasetu, zmienną przechowującą dane treningowe, itp. Dodatkowo podawana jest nazwa pliku, do którego
dane zostaną zapisane. 

**Rozwiązanie projektu zaliczeniowego**  

```python
from ucimlrepo import fetch_ucirepo  # Import funkcji do pobierania danych z repozytorium UCI
import csv  # Do zapisu danych
import random  # Do losowego dzielenia danych
from collections import Counter  # Do liczenia unikalnych elementów


class Dataset:
    def __init__(self, dataset_id):
        """
        Inicjalizuje obiekt i pobiera dane z repozytorium UCI.
        :param dataset_id: ID datasetu z repozytorium UCI.
        """
        print(f"Pobieranie datasetu o ID: {dataset_id}")
        self.dataset = fetch_ucirepo(id=dataset_id)  # Pobieranie datasetu
        self.data = self.dataset.data.features.values.tolist()  # Dane jako lista
        self.target = [t[0] if isinstance(t, list) else t for t in
                       self.dataset.data.targets.values.tolist()]  # Klasy decyzyjne (spłaszczone)
        self.labels = list(self.dataset.data.features.columns)  # Etykiety kolumn
        print("Dane z datasetu zostały załadowane.")

    def wypisz_etykiety(self):
        """
        Wypisuje etykiety kolumn, jeśli istnieją.
        """
        if self.labels:
            print("Etykiety kolumn:", self.labels)
        else:
            print("Brak etykiet kolumn w danych.")

    def wypisz_dane(self, start=0, end=None):
        """
        Wypisuje dane w podanym zakresie.
        :param start: Początkowy indeks (domyślnie 0).
        :param end: Końcowy indeks (domyślnie cały dataset).
        """
        if end is None:
            end = len(self.data)
        for row in self.data[start:end]:
            print(row)

    def podziel_dataset(self, treningowy_procent, testowy_procent, walidacyjny_procent):
        """
        Dzieli dane na zbiory treningowy, testowy i walidacyjny.
        :param treningowy_procent: Procent zbioru treningowego.
        :param testowy_procent: Procent zbioru testowego.
        :param walidacyjny_procent: Procent zbioru walidacyjnego.
        :return: Trzy listy: treningowa, testowa, walidacyjna.
        """
        if treningowy_procent + testowy_procent + walidacyjny_procent != 100:
            raise ValueError("Suma procentów musi wynosić 100.")

        combined = list(zip(self.data, self.target))  # Łączy dane i klasy decyzyjne
        random.shuffle(combined)

        total = len(combined)
        train_end = int(treningowy_procent / 100 * total)
        test_end = train_end + int(testowy_procent / 100 * total)

        train = combined[:train_end]
        test = combined[train_end:test_end]
        val = combined[test_end:]

        return train, test, val

    def wypisz_licznosc_klas(self):
        """
        Wypisuje liczebność klas decyzyjnych.
        """
        counts = Counter(self.target)
        for cls, count in counts.items():
            print(f"Klasa: {cls}, Liczebność: {count}")

    def filtruj_klase(self, class_value):
        """
        Filtruje dane według wartości klasy decyzyjnej.
        :param class_value: Wartość klasy decyzyjnej, którą chcemy wyfiltrować.
        :return: Lista wierszy spełniających kryterium.
        """
        filtered_data = [
            row for row, target in zip(self.data, self.target) if target == class_value
        ]
        return filtered_data

    def zapisz_do_csv(self, data, adult_plik):
        """
        Zapisuje dane do pliku CSV.
        :param data: Lista danych do zapisania.
        :param adult_plik: Nazwa pliku wyjściowego.
        """
        with open(adult_plik, 'w', newline='', encoding='utf-8') as csvfile:
            writer = csv.writer(csvfile)
            if self.labels:
                writer.writerow(self.labels + ["target"])  # Nagłówki kolumn + klasa decyzyjna
            writer.writerows(data)  # Dane
        print(f"Dane zapisane do pliku: {adult_plik}")


# Przykład użycia
if __name__ == "__main__":
    # Pobieranie datasetu Adult (ID: 2)
    handler = Dataset(dataset_id=2)

    # Wypisywanie etykiet
    handler.wypisz_etykiety()

    # Wypisywanie pierwszych 5 wierszy danych
    handler.wypisz_dane(start=0, end=5)

    # Podział na zbiory treningowy, testowy i walidacyjny
    train, test, val = handler.podziel_dataset(70, 20, 10)
    print("Zbiór treningowy:", len(train))
    print("Zbiór testowy:", len(test))
    print("Zbiór walidacyjny:", len(val))

    # Wypisanie liczebności klas
    handler.wypisz_licznosc_klas()

    # Filtrowanie danych według klasy decyzyjnej (np. <=50K)
    filtered = handler.filtruj_klase("<=50K")
    print(f"Liczba wierszy z klasą '<=50K': {len(filtered)}")

    # Zapisanie przefiltrowanych danych do pliku CSV
    handler.zapisz_do_csv(filtered, "projekt_zaliczeniowy.csv")

```
