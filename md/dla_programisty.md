# Wstęp

W poniższym rozdziale przedstawiono dokumentację modułów wykorzystywanych na serwerze.


# Jak dodać nowy wykres?
Wykresy są generowane za pomocą biblioteki apache echarts [https://echarts.apache.org/]. Aby dodać nowy wykres, najłatwiej odwiedzić stronę z przykładami [https://echarts.apache.org/examples/en/index.html] i poszukać wykresu zbliżonego. Po wejściu na jego podstronę można eksperymentować, dostosowując interaktywnie różne opcje. Gdy stworzymy już konfigurację taką jaką chcemy, możemy przejść do integracji nowego wykresu do aplikacji. Zapisz gdzieś obiekt JSON options do użycia później.

Tworzenie wykresu jest kilkuetapowe - najpierw dane do serwowane do klasy generującej `options` - obiekt JSONa, następnie jest on ponowanie modyfikowany przez `ColorsGenerator`. Pełny proces generowania nowego wykresu jest zawarty w ` generateChart(series: any, chartElement: ChartReportElement, reportId, namingDictioanry, dictionaryOverrides, localOverrides = undefined,fullQuery=undefined): EChartsOption` w pliku `charts.service.ts`.

Klasa generująca `options` musi dziedziczyć po `AbstractChartGenerator`. Każda taka klasa pochodna musi zostać także zarejestrowana w funkcji: `getGenerator` w `charts.service.ts` przez dopisanie do słownika `strategyType` nowododanej klasy jako wartość. Kluczem niech będzie krótka nazwa `string` używana do identyfikacji klasy. Taki słownik jest workaroundem bo przez niektóre ograniczenia Angulara nie można podać klasy jako first class object w template komponentu.  

Szablon nowej klasy do wklejenia i uzupełnienia:

```typescript

export class GroupedPercentAndDataChartGenerator extends AbstractChartGenerator {

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

### Funkcje pomocnicze

...