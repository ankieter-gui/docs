# Pobranie nowej wersji
Kod ankietera znajduje się w repozytoriach pod [tym adresem](https://github.com/ankieter-gui).

Repozytoria te, zawierające frontend, backend oraz dokumentację złożone w jedną strukturę folderów gotową do uruchomienia jako aplikacja, znajdują się w repozytorium [release](https://github.com/ankieter-gui/release).

Aby wykorzystać najnowsze repozytoria źródłowe, należy uruchomić znajdujący się w tym repozytorium skrypt:

```
update.sh -c
```

# Instalacja
Strona serwerowa Ankieter+ do poprawnego działania wymaga języka Python w wersji 3.8. Wymagane są także dodatkowe biblioteki języka Python zawarte w pliku "requirements". W celu zainstalowania tych bibliotek należy w wierszu poleceń, będąc w folderze backend/ wykonać polecenie:

    python3 -m pip install -r requirements.txt

Następnie należy zainstalować potrzebną strukturę katalogów, bazę danych dla aplikacji oraz plik konfiguracyjny. Służy do tego skrypt setup.py:

    python3 setup.py

Podczas działania skryptu administrator zostanie poproszony o podanie adresów URL z których będzie korzystała aplikacja, ścieżki do certyfikatu SSL oraz swojego maila i ewentualnie peselu. Następnie skrypt utworzy konto o podanej nazwie posiadające przywileje administratorskie. Po zakończeniu jego działania w bieżącym katalogu zostaną utworzone następujące katalogi oraz pliki:


 - `survey/` - zawierający pliki w formacie XML zawierające informacje o strukturach ankiet źródłowych,
 - `raw/` - zawiera pliki w formacie CSV z wynikami ankiet,
 - `data/` -zawiera pliki SQLite3 ze wstępnie przetworzonymi danymi zebranymi w ankietach,
 - `report/` - zawiera pliki w formacie JSON opisującymi utworzone raporty,
 - `master.db` - główna baza danych zawierająca dane na temat użytkowników,
 - `config.py` - plik konfiguracyjny zawierający informacje potrzebne aplikacji do startu.

Aby uruchomić serwer należy wykonać następujące polecenia:

    python3 main.py

# Konfiguracja
Plik `config.py` tworzony jest przez skrypt `setup.py`, jednak jego parametry można modyfikować jeszcze po instalacji. Wymaganymi parametrami są:

- `CAS_URL` (str) - adres interfejsu sieciowego wykorzystywanego systemu CAS,
- `CAS_VERSION` (int) - wersja systemu CAS,
- `APP_URL` (str) - adres, na którym hostowany jest Ankieter+, nie powinien on zawierać kończącego ukośnika `/`,
- `APP_PORT` (int) - port wykorzystywany przez aplikację,
- `SSL_CONTEXT` (str, str) - para _(ścieżka do certyfikatu, ścieżka do klucza prywatnego)_

Do niewymaganych należą:
- `GUEST_NAME` (str) - nazwa, którą dostaje konto gościa (domyślnie `'Goście'`),
- `ADMIN_DEFAULT_PERMISSION` (str) - domyślne uprawnienia administratora do _wszystkich_ ankiet (domyślnie `'o'`),
- `DAEMONS_INTERVAL` (int) - sekundowy interwał, co który budzone są demony wykonujące dodatkową pracę dla serwera. Obecnie takie demony nie są dostępne, planuje się jednak demon aktualizujący dane ankiet w bazie oraz demon archiwizujący (domyślnie `5*60`),
- `LOCALHOST` (bool) - ustawiony na `True` powoduje, że aplikacja hostowana jest na adresie _localhost_ (domyślnie `False`),
- `DEBUG` (bool) - ustawiony na `True` pozwala na logowanie się do systemu z pominięciem autoryzacji (domyślnie `False`).

# Logika uprawnień
## Użytkownicy
Każdy użytkownik systemu Ankieter+ ma przypisany jedną z następujących ról:

 - "s": _superuser_ - użytkownik z uprawnieniami administratora, ma on wgląd we wszystkie ankiety i raporty utworzone w systemie oraz dostęp do panelu zarządzania użytkownikami i grupami.
 - "u": _user_ - zwykły użytkownik, mogący tworzyć, udostępniać i oglądać/edytować udostępnione mu ankiety.
 - "g": _guest_ - każdy niezalogowany/niedodany do systemu użytkownik, ma wgląd tylko w raporty udostępnione publicznie lub przez link.

## Raporty oraz ankiety
Każdy użytkownik może mieć nadane jedno z następujących rodzajów uprawnień do każdej/każdego z ankiet/raportów w systemie:

 - "o": _own_ - właściciel ankiety/raportu, posiada prawa do odczytu, edycji, kopiowania, udostępniania oraz usuwania danego obiektu.
 - "w": _write_ - edycja, pozwala na podgląd, edycje oraz kopiowanie raportów
 - "r": _read_ - odczyt, zezwala jedynie na podgląd gotowych raportów
 - "n": _none_ - oznacza brak jakichkolwiek uprawnień. W bazie danych nie istnieje jawna reprezentacja tego uprawnienia, ponieważ polega ono na braku innych uprawnień.
