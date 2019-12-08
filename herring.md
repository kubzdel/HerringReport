---
title: "Projekt z analizy danych"
author: "Konrad Kubzdela"
date: "25 listopada 2019"
output: 
  html_document:
    keep_md: yes
    number_sections: yes
    toc: yes
---

# Biblioteki
W projekcie zostały użyte następujące biblioteki

```r
installed.packages()[names(sessionInfo()$otherPkgs), "Version"]
```

```
##       shiny         zoo    corrplot       tidyr   varhandle   heatmaply 
##     "1.4.0"     "1.8-6"      "0.84"     "1.0.0"     "2.0.4"     "1.0.0" 
##     viridis viridisLite      plotly       caret     lattice      GGally 
##     "0.5.1"     "0.3.0"     "4.9.1"    "6.0-84"   "0.20-38"     "1.4.0" 
##    reshape2     ggplot2       dplyr       knitr 
##     "1.4.3"     "3.2.1"     "0.8.3"      "1.26"
```
# Powtarzalność wyników
Kod zapewniający powtarzalność wyników przy tych samych danych

```r
set.seed(22)
knitr::opts_chunk$set(cache=T)
```
# Wczytanie danych
Wszystkie dane wczytuję jako dane numeryczne. Pomijam pierwszą kolumnę zawierającą numer porządkowy rekordu. Znaki zapytania wczytuję jako NA.


```r
data <- read.table("sledzie.csv",header = TRUE,sep=',',na.strings = "?", colClasses=c("NULL", rep(c("numeric"),15)))
```
# Obłsuga brakujących wartości
Najpierw sprawdzam jaki jest stosunek wierszy z brakującymi wartościami do wszystkich wierszy. Wynosi on ponad 20% dlatego nie będę ich pomijał i brakujące wartości zastąpię średnimi z kolumn.

```r
sum(is.na(data))/nrow(data)
```

```
## [1] 0.2102621
```

```r
data<-na.aggregate(data)
```
#Podsumowanie danych
Dane składają się z ponad 52 tysięcy wierszy. Wyliczam cechy statystyczne atrybutów.

```r
dim(data)
```

```
## [1] 52582    15
```

```r
summary(data)
```

```
##      length         cfin1             cfin2             chel1       
##  Min.   :19.0   Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.000  
##  1st Qu.:24.0   1st Qu.: 0.0000   1st Qu.: 0.2778   1st Qu.: 2.469  
##  Median :25.5   Median : 0.1333   Median : 0.7012   Median : 6.083  
##  Mean   :25.3   Mean   : 0.4458   Mean   : 2.0248   Mean   :10.006  
##  3rd Qu.:26.5   3rd Qu.: 0.3603   3rd Qu.: 1.9973   3rd Qu.:11.500  
##  Max.   :32.5   Max.   :37.6667   Max.   :19.3958   Max.   :75.000  
##      chel2            lcop1              lcop2             fbar       
##  Min.   : 5.238   Min.   :  0.3074   Min.   : 7.849   Min.   :0.0680  
##  1st Qu.:13.589   1st Qu.:  2.5479   1st Qu.:17.808   1st Qu.:0.2270  
##  Median :21.435   Median :  7.1229   Median :25.338   Median :0.3320  
##  Mean   :21.221   Mean   : 12.8108   Mean   :28.419   Mean   :0.3304  
##  3rd Qu.:27.193   3rd Qu.: 21.2315   3rd Qu.:37.232   3rd Qu.:0.4560  
##  Max.   :57.706   Max.   :115.5833   Max.   :68.736   Max.   :0.8490  
##       recr              cumf             totaln             sst       
##  Min.   : 140515   Min.   :0.06833   Min.   : 144137   Min.   :12.77  
##  1st Qu.: 360061   1st Qu.:0.14809   1st Qu.: 306068   1st Qu.:13.63  
##  Median : 421391   Median :0.23191   Median : 539558   Median :13.86  
##  Mean   : 520367   Mean   :0.22981   Mean   : 514973   Mean   :13.87  
##  3rd Qu.: 724151   3rd Qu.:0.29803   3rd Qu.: 730351   3rd Qu.:14.16  
##  Max.   :1565890   Max.   :0.39801   Max.   :1015595   Max.   :14.73  
##       sal            xmonth            nao          
##  Min.   :35.40   Min.   : 1.000   Min.   :-4.89000  
##  1st Qu.:35.51   1st Qu.: 5.000   1st Qu.:-1.89000  
##  Median :35.51   Median : 8.000   Median : 0.20000  
##  Mean   :35.51   Mean   : 7.258   Mean   :-0.09236  
##  3rd Qu.:35.52   3rd Qu.: 9.000   3rd Qu.: 1.63000  
##  Max.   :35.61   Max.   :12.000   Max.   : 5.08000
```
#Rozkłąd atrybutów

```r
for (col in 1:ncol(data)) {
   hist(data[,col],main = names(data[col]))
}
```

![](herring_files/figure-html/unnamed-chunk-6-1.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-2.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-3.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-4.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-5.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-6.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-7.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-8.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-9.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-10.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-11.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-12.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-13.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-14.png)<!-- -->![](herring_files/figure-html/unnamed-chunk-6-15.png)<!-- -->

#Korelacja
Analizując macierz korelacji można wyłonić najbardziej skorelowane pary atrybutów. Są to chel1-lcop1, chel2-lcop2 oraz fbar-cumf. Mocno skorelowane dane wprowadzają niepotrzebną redundancję, dlatego w dalszej analizie wykluczam atrybuty lcop1,lcop2 oraz cumf. Pozbędę się także wiedzy o miesiącu, który jest cykliczny i wykazuje blisko zerową korelację z długością śledzia. Najbardziej skorelowany atrybut z długością to średnia temperatura wody przy powierzchni.

```r
ggcorr(data,label = TRUE,label_round = 1)
```

![](herring_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
data <- subset(data, select = -c(lcop1,lcop2,cumf,xmonth))
```
# Wizualizacja rozmiaru śledzia
Dane pochodzą z ostatich 60 lat i są posortowane chronologicznie, dlatego podzielę je na 60 podzbiorów, a średnia z każdego podzbioru posłuży do wizualizacji rozmiaru śledzia. Oprócz interaktywnego wykresu do wizualizacji rozmiaru, rysuję wykres, na którym jeszcze lepiej widać negatywny trend zaczynający się mniej więcej 40 lat temu.

```r
herring_size<- split(data$length, (ceiling(seq_along(data$length)/(nrow(data)/60))))
herring_size <- lapply(herring_size, function(x) Reduce(`+`, x)/length(x) )
plot(seq(length(herring_size)),herring_size,type = "b",xlab="chunk")
```

![](herring_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


```
## PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.
```

<!--html_preserve--><div style="width: 100% ; height: 400px ; text-align: center; box-sizing: border-box; -moz-box-sizing: border-box; -webkit-box-sizing: border-box;" class="muted well">Shiny applications not supported in static R Markdown documents</div><!--/html_preserve-->
# Model predykcyjny
Dane dzielę na dwa zbiory w proporcji 80-20. Większy zbiór posłuży jako dane uczące i testowe w późniejsze 10-krotnej walidacji krzyżowej. Pozostałe 20% posłuży jako dane walidacyjne, na których zostanie oceniona ostateczna skuteczność modelu. Dodatkowo dane zostały wycentrowane oraz znormalizowane. Model wytrenuję przy użyciu metody extreme gradient boosting. Korzystam z domyślnego testowanie parametrów tej metody.

```r
inTraining <- 
    createDataPartition(
        # atrybut do stratyfikacji
        y = data$length,
        # procent w zbiorze uczącym
        p = .8,
        # chcemy indeksy a nie listę
        list = FALSE)

training <- data[ inTraining,]
testing  <- data[-inTraining,]

X_train = select(training, -length)
y_train = training$length
X_test = select(testing, -length)
y_test = testing$length

trcontrol = trainControl(
  method = "cv",
  number = 2,  
  allowParallel = TRUE,
  verboseIter = FALSE,
  returnData = FALSE)

model <- train(X_train,y_train,
             method = "xgbTree",
             preProcess = c('scale', 'center'),
             trControl = trcontrol
              )
```

Ostateczne wartośći RMSE i R2 dla wytrenowanego modelu i danych walidujących prezentują się następująco:

```r
pred <- predict(model, X_test)
postResample(pred, testing$length)
```

```
##      RMSE  Rsquared       MAE 
## 1.1895059 0.4769302 0.9376172
```

# Analiza ważności atrybutów

Analiza ważności atrybutów z wytrenowanego modelu sugeruje, że zdecydowanie najważniejszym czynnikiem wplywającym na rozmiar śledzia jest temperatura wody przy powierzchni wody. Ten atrybów jest dość silnie ujemnie skorelowany z długością śledzia. Oznacza to, że głównym powodem spadku długości śledzia w czasie był wzrost temperatury przy powierzchni wody.


```r
ggplot(varImp(model))
```

![](herring_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

Potwierdzimy te przypuszczenia rysując jednocześnie jak zmieniały się te dwie wartości w czasie. W tym celu wyciągam średnie z dłuższych okresów dla temperatury przy powierzchi, analogicznie jak wcześniej dla długości śledzia.

```r
temp<- split(data$sst, (ceiling(seq_along(data$sst)/(nrow(data)/60))))
temp <- lapply(temp, function(x) Reduce(`+`, x)/length(x) )
year = seq(length(temp))
df <- do.call(rbind, Map(data.frame, temp=temp, size=herring_size,year = year))
ggplot(df, aes(year)) + 
  geom_line(aes(y = temp)) 
```

![](herring_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
ggplot(df, aes(year)) + 
  geom_line(aes(y = size)) 
```

![](herring_files/figure-html/unnamed-chunk-13-2.png)<!-- -->

Lustrzany obraz wykresów potwierdza stwierdzoną wcześniej ujemną korelację.

#Wnioski
Analiza statystyczna, wizualizacja danych oraz wytrenowanie modelu predykcyjnego pozwoliło potwierdzić spadek w rozmiarze śledzia na przestrzeni lat; określić moment, w którym ten trend nastąpił oraz co miało na niego największy wpływ. Zbadanie korealcji pomiędzy atrybutami pozwoliło usunąć z dalszej analizy mało znaczące atrybuty. 
