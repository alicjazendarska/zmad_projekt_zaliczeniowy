# Projekt zaliczeniowy

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
import csv
from typing import List, Tuple

class DatasetModule:
    def __init__(self):
        self.headers = []  # Lista przechowująca nagłówki kolumn
        self.data = []  # Lista przechowująca wiersze danych

    def wczytanie_bazy(self, file_path: str, has_headers: bool = True):
        with open(file_path, mode='r', newline='', encoding='utf-8') as file:
            reader = csv.reader(file)
            if has_headers:
                self.headers = next(reader)
            self.data = [row for row in reader]
        print("\n Dane z datasetu zostały załadowane.")

    def wypisanie_nagłówków(self) -> List[str]:
        if self.headers:
            print("Etykiety kolumn:", self.headers)


    def wypisanie_danych(self, start: int = None, end: int = None):
        if start is None or end is None:
            return self.data
        return self.data[start:end]

    def podział_dataset(self, train_pct: float, test_pct: float, val_pct: float) -> Tuple[List, List, List]:
        total = len(self.data)
        train_end = int(train_pct * total)
        test_end = train_end + int(test_pct * total)
        return self.data[:train_end], self.data[train_end:test_end], self.data[test_end:]

    def liczba_klas_decyzyjnych(self, class_column: int) -> List[Tuple[str, int]]:
        from collections import Counter
        counts = Counter(row[class_column] for row in self.data)
        return list(counts.items())

    def filtr_klasy_decyzyjnej(self, class_column: int, class_value: str) -> List[List[str]]:
        return [row for row in self.data if row[class_column] == class_value]

    def zapisz_do_csv(self, data: List[List[str]], file_name: str):
        with open(file_name, mode='w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            if self.headers:
                writer.writerow(self.headers)
            writer.writerows(data)

# Testy działania modułu

# Inicjalizacja obiektu modułu
module = DatasetModule()

# 1. Wczytanie danych z pliku CSV
file_path = "sobar-72.csv"  # Ścieżka do pliku CSV
module.wczytanie_bazy(file_path, has_headers=True)

# 2. Wyświetlenie nagłówków
print("\n=== Nagłówki ===")
headers = module.wypisanie_nagłówków()
print(headers)

# 3. Wyświetlenie wybranych wierszy danych
print("\n=== Wybrane wiersze z bazy danych ===")
data_preview = module.wypisanie_danych(6, 18)
for row in data_preview:
    print(row)

# 4. Podział danych na zbiory treningowy, testowy i walidacyjny
print("\n=== Podział danych na zbiory treningowy, testowy i walidacyjny ===")
train_set, test_set, val_set = module.podział_dataset(train_pct=0.7, test_pct=0.2, val_pct=0.1)
print(f"Zbiór treningowy: {len(train_set)} wiersze, Zbiór testowy: {len(test_set)} wiersze, Zbiór walidacyjny: {len(val_set)} wiersze")

# 5. Wyświetlenie liczby klas decyzyjnych w ostatniej kolumnie
print("\n=== Liczebność klas decyzyjnych ===")
class_distribution = module.liczba_klas_decyzyjnych(class_column=-1)
for class_name, count in class_distribution:
    print(f"Class '{class_name}': {count} instances")

# 6. Filtrowanie danych dla konkretnej klasy (np. klasy " 1=ma raka szyjki macicy, 0=nie ma raka szyjki macicy")
print("\n=== Wartość klasy decyzyjnej dla wybranej wartości ===")
filtered_rows = module.filtr_klasy_decyzyjnej(class_column=-1, class_value="1")

print(f"Wybrana ilość wierszy dla wybranej klasy decyzyjnej: {len(filtered_rows)}")
for row in filtered_rows[5:9]:
    print(row)

# 7. Zapisanie filtrowanych danych do nowego pliku CSV
print("\n=== Zapisanie danych do pliku CSV ==")
output_file = "projekt_plik.csv"
module.zapisz_do_csv(filtered_rows, output_file)
print(f"Zapisanie danych do pliku: {output_file}")

```
