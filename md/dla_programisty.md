# Wstęp

W poniższym rozdziale przedstawiono dokumentację modułów wykorzystywanych na serwerze.


# Jak dodać nowy wykres?
Wykresy są generowane za pomocą biblioteki apache echarts [https://echarts.apache.org/]. Aby dodać nowy wykres, najłatwiej odwiedzić stronę z przykładami [https://echarts.apache.org/examples/en/index.html] i poszukać wykresu zbliżonego. Po wejściu na jego podstronę można eksperymentować, dostosowując interaktywnie różne opcje. Gdy stworzymy już konfigurację taką jaką chcemy, możemy przejść do integracji nowego wykresu do aplikacji. Zapisz gdzieś obiekt JSON options do użycia później.

Tworzenie wykresu jest kilkuetapowe - najpierw dane do serwowane do klasy generującej `options` - obiekt JSONa, następnie jest on ponowanie modyfikowany przez `ColorsGenerator`. Pełny proces generowania nowego wykresu jest zawarty w ` generateChart(series: any, chartElement: ChartReportElement, reportId, namingDictioanry, dictionaryOverrides, localOverrides = undefined,fullQuery=undefined): EChartsOption` w pliku `charts.service.ts`.

Klasa generująca `options` musi dziedziczyć po `AbstractChartGenerator`. Każda taka klasa pochodna musi zostać także zarejestrowana w funkcji: `getGenerator` w `charts.service.ts` przez dopisanie do słownika `strategyType` nowododanej klasy jako wartość. Kluczem niech będzie krótka nazwa `string` używana do identyfikacji klasy. Taki słownik jest workaroundem bo przez niektóre ograniczenia Angulara nie można podać klasy jako first class object w template komponentu.  

## Tutorial 
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
5. W `reports/editor/chart-editor-view.component.ts` ctrl+f ` <nz-tab nzTitle="Wygląd i układ">` 
W elemencie `<section class="query-marker"></section>` dodajemy markup

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
oraz nową funkcję do klasy 

```typescript 
myChartGenerator(options: EChartsOption): EChartsOption {
    return options;
  }
```
która modyfikuje i zwraca obiekt options w celu nadania kolorów i kosmetyki konkretnym seriom lub elementom.

Jeżeli nie jest to potrzebne można nic nie dodawać.

