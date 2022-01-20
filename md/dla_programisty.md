# Wstęp

W poniższym rozdziale przedstawiono dokumentację modułów wykorzystywanych w aplikacji oraz omówiono działanie niektórych jej charakterystycznych mechanik.

# Język zapytań dla zestawień
Wyświetlenie wykresu w edytorze raportów wymaga wysłania przez frontendową części aplikacji zapytania do serwera. W takim zapytaniu zawarte są informacje o interesujących użytkownika agregacjach, które mają następnie znaleźć się w wykresie.

Ponieważ zapytania te są opisane w stworzonym specjalnie dla Ankietera+ języku, został on w tej sekcji dokładnie opisany.

W ogólności język stosowany w Ankieterze+ opisuje dobrze znane _zapytania krzyżowe_.
Wszystkie zapytania mają JSONowy format przestrzegający następującego szablonu:
```
{
  "get":    [[column-names...]],
  "as":     [aggregation-names...],
  "by":     [column-names...],
  "if":     [[variable-name/column-number, operator, args...]],
  "except": [[variable-name, operator, args...]],
  "join":   [{"name": column-name, "of": [source-column-names...]}],
  "macro":  [macro-names...]
}
```
Każde zapytanie dzieli się więc na siedem klauzul, z których jedynie dwie pierwsze są wymagane. Część informacji o strukturze zapytań można znaleźć w pliku `grammar.py`.

Klauzula `get` jest _tablicą tablic nazw kolumn_.
Są to nazwy kolumn, na podstawie których obliczane będą wartości funkcji agregujących.
Fakt, że nazwy kolumn zawarte są w podwójnie zagnieżdżonej tablicy może być mylący;
ale dla zrozumienia języka zapytań wystarczy początkowo założyć, że jest to zwykła tablica, a powód podwójnego jej zagnieżdżenia zostanie wyjaśniony później.
Ważne jest, że kolejne nazwy zmiennych klauzuli `get` odpowiadają kolejnym kolumnom wynikowym zestawieniu.

Klauzula `as` jest listą nazw funkcji agregujących, które będą aplikowane do zmiennych położonych na _odpowiadających_ pozycjach w klauzuli `get`.

Klauzula `by` opisuje zmienne, po których grupowane będą wartości kolumn z klauzuli `get`. Wartości należące do tej samej grupy będą podlegały wspólnej agregacji i w efekcie zostaną przekształcone na pojedynczą wartość (np. liczność grupy, średnią albo największą wartość dla grupy itd.). Jeśli wszyscy użytkownicy mają trafić do jednej grupy, aby podsumować jakąś informację na temat całego zbioru, jako nazwa zmiennej grupującej podawana jest _gwiazdka_ (`"by": ["*"]`). Nazwy grup stanowią _nazwy wierszy_ w wynikowych danych. Warto przy tym zauważyć, że wartość w klauzuli `by` jest tablicą, co jest wykorzystywane w sytuacjach, w których w jednym zestawieniu tworzone jest kilka rodzajów grup (np. podział na roczniki + podział na stopień studiów + wszyscy razem). W takiej sytuacji jeden rekord może trafić do kilku grup jednocześnie, ale nie zmienia to formatu wyniku -- zawiera on podsumowania (agregacje) na temat każdej z grup.

Zawartość klauzuli `if` jest tablicą filtrów. Każdy filtr jest tablicą o następującym formacie:
```
[variable-name/column-number, operator, args...]
```

Jeśli pierwszym elementem filtra jest nazwa zmiennej, to filtrowany rekord musi mieć zmienną spełniającą opisany w filtrze wymóg, aby znaleźć się w analizowanym zbiorze. Na przykład:
```
["Kierunek studiów", "in", "Kognitywistyka", "Informatyka", "Fizyka"]
```
lub:
```
["Rok studiów", ">", "3"]
```
Lista filtrów i opis ich działania zostały zawarte w pliku źródłowym `table.py`.

Jeśli jednak pierwszym elementem filtra jest liczba, to musi ona odpowiadać jednej z kolumn zdefiniowanych przez klauzulę `get`. Filter taki nie usuwa całych rekordów, ale usuwa wartości zmiennych, które nie będą podlegały agregacji, np. wartości specjalne nie posiadające interpretacji liczbowej:
```
[3, "!=", "Nie dotyczy"]
```

Klauzula `except` działa odwrotnie do `if`. Jeśli rekord spełni _wszystkie_ wymienione w niej wymagania, to rekord _jest usuwany_ z analizowanego zbioru. Pozwala to wybrać podzbiór, którego analiza nie dotyczy.

W klauzuli `join` można tworzyć nowe kolumny na podstawie już istniejących.
Umożliwia to wyliczenie średniej z kilku zmiennych, na przykład:
```
{
  "get":  [["Ocena ogólna"]],
  "as":   ["mean"],
  "by":   ["*"],
  "join": [
    {
      "name": "Ocena ogólna",
      "of": ["Ocena dziekanatu", "Ocena zajęć", "Ocena prowadzących"]
    }
  ],
  "except": ["Rok studiów", "=", "1"]
}
```
Zestawienie to podlicza średnią wartość zmiennej, na którą składają się oceny poszczególnych aspektów pracy wydziału lub uniwersytetu. Zestawienie da pojedyncza wartość dla całego zbioru, wykluczy przy tym studentów 1. roku, którzy nie ukończyli jeszcze żadnych zajęć.

Ostatni klauzula, `macro` zawiera nazwę makra, która jest wykrywana przez funkcje obsługujące zapytania i na jej podstawie zapytanie jest uzupełniane. Jest to najnowszy element języka zapytań używanego przez Ankieter+ i został dodany w ramach optymalizacji aplikacji. Z tego powodu istnieje na razie tylko jedno makro, `count-answers`, które na podstawie kolumn w `get` uzupełnia zapytanie tak, aby zwróciło ono liczbę osób, które odpowiedziały na _przynajmniej jedno_ z tych pytań.

Ponadto, klauzula `get` jest _tablicą tablic_, dzięki temu do zapytania można dołączać wiele list kolumn, o ile każda z nich ma tę samą długość. Takie zapytanie zostanie wykonane jako kilka zapytań podlegających tym samym filtrom i agregacjom. Używane jest to w sytuacjach, kiedy temu samemu zestawieniu podlega kilka podobnych do siebie zmiennych:
```
{
  "get":    [["Ocena dziekanatu"], ["Ocena zajęć"], ["Ocena prowadzących"]],
  "as":     ["mean"],
  "by":     ["Rok studiów"],
  "if":     [0, "!=", "Odmowa odpowiedzi"],
  "except": ["rok studiów", "=", "1"]
}
```

Wyniki zapytania mają następujący format:
```
{
  "index":              [group1-name, group2-name...],
  "aggregated-value-A": [for-group-1, for-group-2, ...],
  "aggregated-value-B": [for-group-1, for-group-2, ...]
}
```
Nazwy wierszy (`index`) pochodzą od nazw zmiennych użytych w klauzuli `by`. Z kolei nazwy kolumn (pozostałe klucze) mają format `aggegator-name variable-name`.

# Jak dodać nowy rodzaj wykresu?
Wykresy są generowane za pomocą biblioteki [https://echarts.apache.org/](apache echarts). Aby dodać nowy wykres, najłatwiej odwiedzić stronę z [https://echarts.apache.org/examples/en/index.html](przykładami) i poszukać wykresu zbliżonego. Po wejściu na jego podstronę można eksperymentować, dostosowując interaktywnie różne opcje. Gdy stworzymy już konfigurację taką jaką chcemy, możemy przejść do integracji nowego wykresu do aplikacji. Zapisz gdzieś obiekt JSON options do użycia później.

Tworzenie wykresu jest kilkuetapowe - najpierw dane do serwowane do klasy generującej `options` - obiekt JSONa, następnie jest on ponowanie modyfikowany przez `ColorsGenerator`. Pełny proces generowania nowego wykresu jest zawarty w ` generateChart(series: any, chartElement: ChartReportElement, reportId, namingDictioanry, dictionaryOverrides, localOverrides = undefined,fullQuery=undefined): EChartsOption` w pliku `charts.service.ts`.

Klasa generująca `options` musi dziedziczyć po `AbstractChartGenerator`. Każda taka klasa pochodna musi zostać także zarejestrowana w funkcji: `getGenerator` w `charts.service.ts` przez dopisanie do słownika `strategyType` nowododanej klasy jako wartość. Kluczem niech będzie krótka nazwa `string` używana do identyfikacji klasy. Taki słownik jest workaroundem bo przez niektóre ograniczenia Angulara nie można podać klasy jako first class object w template komponentu.  

Aby dodać nowy wykres:
1. Tworzymy nowy plik o nazwie MyChart.ts.
1. Wklejamy tam szablon klasy generującej `options` wykresu
```typescript

export class MyChartGenerator extends AbstractChartGenerator {

  constructor(series: any, chartElement: ChartReportElement, namingDictionary, public reportsService: ReportsService, dictionaryOverrides) {
    super(series, chartElement, namingDictionary, reportsService, dictionaryOverrides);
  }

  generate(): AbstractChartGenerator {

      return this
  }

  getAllCount(reportId, complimentedQuery: SurveyQuery = undefined) {
  }

  asJSONConfig(): EChartsOption {
    return {} as EChartsOption
  }
}
```
`generate()` jest wywoływane jednorazowo przy każdej zmianie ustawień wykresu w UI przez użytkownika. `asJSONConfig()` zwraca `options` wykresu. Należy wkleić to co wygenerowaliśmy w kreatorze na stronie echarts z dodatkowym uzupełnieniem o serie danych.
Serie danych pochodzą bezośrednio z response z serwera i są dostępne pod `this.rawSeries`

2. Wybieramy tekstowy identyfikator klasy - może to być jej nazwa zapisana w klasie jako `static name = "MyChart"`

3. Otwieramy plik `reports/editor/chart-editor-view.component.ts`
4. Szykujemy obrazek-miniaturkę wykresu i umieszczamy ją w `assets/`
5. W `reports/editor/chart-editor-view.component.ts` ctrl+f ` <nz-tab nzTitle="Wygląd i układ">`  <br>
 W elemencie `<section class="query-marker"></section>` dodajemy markup <br>
 ```html
 <div class="spacer"></div>
<figure class="indicator-card indicator-card-velvet preset"
        nz-tooltip="Użyj tego wykresu aby przedstawić wyniki z pytań wielokrotnego wyboru. Na przykład: dlaczego poleciłbyś UAM"
        (click)="pickPreset('multipleChoice');">
  <div class="indicator-card-inner">
    <div class="indicator-card-header">Wielokrotny wybór</div>
    <div class="indicator-card-content"><img src="./assets/preset2.png" style="width: 100%"></div>
  </div>
</figure>             
```
 Do uzupełniania: tooltip, miniaturka, nazwa i `(click)="pickPreset('MyChart');"`,  gdzie MyChart to nazwa naszego nowego wykresu identyczną z ustaloną wcześniej.

6. Do funkcji pickPreset w tym samym pliku dopisujemy klucz do słownika `fun`.
<br> Wartością jest funkcja bez argumentów która ustawia różne parametry charakterystyczne dla tego rodzaju wykresu i sposób w jaki układane jest zapytanie do bazy danych. <br>
 Możemy schować prawą kolumnę `Grupuj przez` ustawiając `this.hideGrupBy=true;`, schować panel wybierania rodzajów agregeacji ` this.hideData=true;`. <br>
 Zmienić zachowania po kliknięciu jakiegoś elementu interfejsu: ..
`this.onPickQuestion=(question)=>{}`
`this.byPickerClick = (by) => {}` <br>
 Ustawić domyślny typ agregacji: `  this.chartData.dataQuery.as[0]='share'` <br>
 Ustawić domyślny typ grupowania ` this.chartData.dataQuery.by[0] = "*"` <br>
 Lub zrobić cokolwiek innego co jest potrzebne aby umożliwić użytkownikowi wybranie wszystkich potrzebnych opcji.

7. Teraz należy wyświetlić gotowy wykres. Do fragmentu `  <section class="chart-area" *ngIf="['multipleChoice', 'groupedBars', 'multipleBars', 'linearCustomData', 'summary','groupSummary', 'multipleBarsOwnData'].includes(chartData.config.type)  && this.echartOptions">` należy dopisać nasz typ wykresu żeby się wyświetlał w domyślnym trybie - jako prosty wykres generowany na podstawie `options`

8. (Opcjonalnie) Ostatnią czynnością jest kolorowanie wykresu. W `ColorsGenerator.ts` dodajemy do słownika x nowy wpis `[MyChartGenerator.name] = (o) => this.myChartGenerator(o);`
oraz nową funkcję do klasy <br>
```typescript
myChartGenerator(options: EChartsOption): EChartsOption {
    return options;
  }
```
która modyfikuje i zwraca obiekt options w celu nadania kolorów i kosmetyki konkretnym seriom lub elementom. <br>
Jeżeli nie jest to potrzebne można nic nie dodawać.
