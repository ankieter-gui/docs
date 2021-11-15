# Instalacja
Strona serwerowa Ankieter+ do poprawnego działania wymaga języka Python w wersji 3.9.5. Wymagane są także dodatkowe biblioteki języka Python zawarte w pliku "requirements". W celu zainstalowania tych bibliotek należy w wierszu poleceń, będąc w folderze backend/ wykonać polecenie:

    python3 -m pip install -r requirements.txt
Następnie należy zainstalować potrzebną strukture katalogów oraz bazę danych dla aplikacji. Służy do tego plik setup.py:

    python3 setup.py
   Podczas działania skryptu administrator zostanie poproszony o podanie swojego maila oraz peselu, w celu dodania go do bazy danych z uprawnieniami administratora. Po zakończeniu jego działania w bieżącym katalogu zostaną utworzone następujące katalogi oraz pliki:
   

 - surveys/ - zawierający pliki w formacie <id_ankiety>.xml ze strukturami ankiet
 - raw/ - zawiera pliki w formacie <id_ankiety>.csv z wynikami ankiet
 - data/ -zawiera pliki w formacie <id_ankiety>.db ze zmergowanymi oraz zformatowanymi wynikami ankiet
 - report/ - zawiera pliki w formacie <id_raportu>.json z plikami opisującymi utworzone raporty
 - master.db - główna baza danych (## TODO: struktura ##)

Aby uruchomić serwer należy w folderze backend/ wykonać następujące polecenia:

    export FLASK_APP=main.py

    python3 -m flask run

# Wydanie nowej wersji
Kod ankietera znajduje się w repozytoriach.
Aby wydać nową wersję, należy uruchomić skrypt w repozytorium release.
```
update.sh
git push
```
   
# Konfiguracja
W pliku config.py znajdują się następujące parametry, które należy skonfigurować:

# Deployment
W celu uruchomienia aplikacji należy uruchomić skrypt na serwerze.

```
./update.sh
```

# Logika uprawnień
## Użytkownicy
Każdy użytkownik systemu Ankieter+ ma przypisany jedną z następujących ról:

 - "s": superuser - użytkownik z uprawnieniami administratora, ma on wgląd we wszystkie ankiety i raporty utworzone w systemie oraz dostęp do panelu zarządzania użytkownikami i grupami.
 - "u": user - zwykły użytkownik, mogący tworzyć, udostępniać i oglądać/edytować udostępnione mu ankiety.
 - "g": guest - każdy niezalogowany/niedodany do systemu użytkownik, ma wgląd tylko w raporty udostępnione publicznie lub przez link.

## Raporty oraz ankiety
Każdy użytkownik może mieć nadane jedno z następujących rodzajów uprawnień do każdej/każdego z ankiet/raportów w systemie:

 - "o": owner - właściciel ankiety/raportu, posiada prawa do odczytu, edycji, kopiowania, udostępniania oraz usuwania danego obiektu.
 - "w": write - edycja, pozwala na podgląd, edycje oraz kopiowanie raportów
 - "r": read - odczyt, zezwala jedynie na podgląd gotowych raportów
 - "n": none - nie jest to faktyczne uprawnienie, a "operator" służący do usuwania uprawnień nadanych wcześniej. Nie jest uwzględnione w bazie danych.

