---
title: "sledzie-analiza"
author: "Julita Wyrwal"
date: "Friday, May 26, 2018"
output:
  html_document:
    keep_md: yes
---

## SPIS TREŚCI
1. [STRESZCZENIE](#STRESZCZENIE)
2. [WYORZYSTANE BIBLIOTEKI](#WYKORZYSTANE BIBLIOTEKI)
3. [WCZYTYWANIE DANYCH](#WCZYTYWANIE DANYCH)
4. [BRAKUJĄCE WARTOŚCI](#BRAKUJĄCE WARTOŚCI)
5. [ANALIZA ZBIORU DANYCH](#ANALIZA ZBIORU DANYCH)
6. [ANALIZA POSZCZEGÓLNYCH ATRYBUTÓW](#ANALIZA POSZCZEGÓLNYCH ATRYBUTÓW)
7. [ANALIZA KORELACJI](#ANALIZA KORELACJI)
8. [WYKRES](#WYKRES ZMIANY ROZMIARU ŚLEDZIA W CZASIE)
8. [BUDOWA REGRESORA PRZEWIDUJACEGO ROZMIAR ŚLEDZI](#BUDOWA REGRESORA)

##STRESZCZENIE
================================================
Celem projektu jest odpowiedź na pytanie dlaczego na przestrzeni ostatnich lat zauważono stopniowy spadek rozmiaru śledzia oceanicznego wyławianego w Europie,
Analizie poddano dane z ostatnich 60 lat. 

Ponieważ w zbiorze danych dla licznych obserwacji informacja była niepełna, w pierwszym kroku wiersze takie wyrzucono ze zbioru. Po oczyszeniu, w zbiorze zostało 4288 obserwacji.

W kolejnym kroku, podano podstawowe statystyki dla poszczególnych atrybów. Zbadano również ich korelację, która jasno wskazywała zależności występujące pomiędzy długością śledzia a temperaturą przy powierzni wody.

Ostatnim elementem badania, była budowa modelu przewidującego rozmiar śledzia. 
Model oceniono za pomocą metody R kwadrat oraz RMSE. 






## WYKORZYSTANE BIBLIOTEKI


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
library(gridExtra)
```

```
## 
## Attaching package: 'gridExtra'
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

```r
library(corrplot)
```

```
## corrplot 0.84 loaded
```

```r
library(plotly)
```

```
## 
## Attaching package: 'plotly'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     last_plot
```

```
## The following object is masked from 'package:stats':
## 
##     filter
```

```
## The following object is masked from 'package:graphics':
## 
##     layout
```

```r
library(shiny)
library(lattice)
library(caret)
```

## WCZYTANIE DANYCH


```r
sledzie1<- read.csv(file = "http://www.cs.put.poznan.pl/dbrzezinski/teaching/sphd/sledzie.csv", na.strings="?")
```

## BRAKUJĄCE WARTOŚCI - wiersze z nieuzupełnionymi wartościami zostaną usunięte za pomocą kodu:


```r
sledzie<-filter(sledzie1, !is.na(chel2), !is.na(chel1),!is.na(cfin1), !is.na(cfin2), !is.na(lcop1), !is.na(lcop2), !is.na(fbar), !is.na(recr), !is.na(cumf), !is.na(totaln), !is.na(sst), !is.na(sal), !is.na(xmonth), !is.na(nao))
```

##ANALIZA ZBIORU DANYCH

Wymiar tablicy: 

```r
dim(sledzie)
```

```
## [1] 42488    16
```
##Spis oraz informacje o poszczególnych atrybutach:

###Opis atrybutów

* length: długość złowionego śledzia [cm];
* cfin1: dostępność planktonu [zagęszczenie Calanus finmarchicus gat. 1];
* cfin2: dostępność planktonu [zagęszczenie Calanus finmarchicus gat. 2];
* chel1: dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 1];
* chel2: dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 2];
* lcop1: dostępność planktonu [zagęszczenie widłonogów gat. 1];
* lcop2: dostępność planktonu [zagęszczenie widłonogów gat. 2];
* fbar: natężenie połowów w regionie [ułamek pozostawionego narybku];
* recr: roczny narybek [liczba śledzi];
* cumf: łączne roczne natężenie połowów w regionie [ułamek pozostawionego narybku];
* totaln: łączna liczba ryb złowionych w ramach połowu [liczba śledzi];
* sst: temperatura przy powierzchni wody [°C];
* sal: poziom zasolenia wody [Knudsen ppt];
* xmonth: miesiąc połowu [numer miesiąca];
* nao: oscylacja północnoatlantycka [mb].

###Podstawowe statystyki

```r
summary(sledzie)
```

```
##        X             length         cfin1             cfin2        
##  Min.   :    1   Min.   :19.0   Min.   : 0.0000   Min.   : 0.0000  
##  1st Qu.:13233   1st Qu.:24.0   1st Qu.: 0.0000   1st Qu.: 0.2778  
##  Median :26308   Median :25.5   Median : 0.1111   Median : 0.7012  
##  Mean   :26316   Mean   :25.3   Mean   : 0.4457   Mean   : 2.0269  
##  3rd Qu.:39447   3rd Qu.:26.5   3rd Qu.: 0.3333   3rd Qu.: 1.7936  
##  Max.   :52580   Max.   :32.5   Max.   :37.6667   Max.   :19.3958  
##      chel1            chel2            lcop1              lcop2       
##  Min.   : 0.000   Min.   : 5.238   Min.   :  0.3074   Min.   : 7.849  
##  1st Qu.: 2.469   1st Qu.:13.427   1st Qu.:  2.5479   1st Qu.:17.808  
##  Median : 5.750   Median :21.435   Median :  7.0000   Median :24.859  
##  Mean   :10.016   Mean   :21.197   Mean   : 12.8386   Mean   :28.396  
##  3rd Qu.:11.500   3rd Qu.:27.193   3rd Qu.: 21.2315   3rd Qu.:37.232  
##  Max.   :75.000   Max.   :57.706   Max.   :115.5833   Max.   :68.736  
##       fbar             recr              cumf             totaln       
##  Min.   :0.0680   Min.   : 140515   Min.   :0.06833   Min.   : 144137  
##  1st Qu.:0.2270   1st Qu.: 360061   1st Qu.:0.14809   1st Qu.: 306068  
##  Median :0.3320   Median : 421391   Median :0.23191   Median : 539558  
##  Mean   :0.3306   Mean   : 519877   Mean   :0.22987   Mean   : 515082  
##  3rd Qu.:0.4650   3rd Qu.: 724151   3rd Qu.:0.29803   3rd Qu.: 730351  
##  Max.   :0.8490   Max.   :1565890   Max.   :0.39801   Max.   :1015595  
##       sst             sal            xmonth            nao          
##  Min.   :12.77   Min.   :35.40   Min.   : 1.000   Min.   :-4.89000  
##  1st Qu.:13.60   1st Qu.:35.51   1st Qu.: 5.000   1st Qu.:-1.90000  
##  Median :13.86   Median :35.51   Median : 8.000   Median : 0.20000  
##  Mean   :13.87   Mean   :35.51   Mean   : 7.252   Mean   :-0.09642  
##  3rd Qu.:14.16   3rd Qu.:35.52   3rd Qu.: 9.000   3rd Qu.: 1.63000  
##  Max.   :14.73   Max.   :35.61   Max.   :12.000   Max.   : 5.08000
```

##HISTOGRAMY POSZCZEGÓLNYCH ATRYBUTÓW:



```r
l<-ggplot(sledzie, aes(length))+geom_histogram(binwidth=0.5)+theme_bw()+ggtitle('Długość złowionego śledzia')
c_1<-ggplot(sledzie, aes(cfin1))+geom_histogram(binwidth=1)+theme_bw()+ggtitle('dostępność planktonu finmarchicus gat. 1')+xlim(0,6)


grid.arrange(l,c_1, nrow=1)
```

```
## Warning: Removed 6 rows containing non-finite values (stat_bin).
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-1.png)<!-- -->

```r
c_2<-ggplot(sledzie, aes(cfin2))+geom_histogram(binwidth=2)+theme_bw()+ggtitle('dostępność planktonu finmarchicus gat. 2')
chel1<-ggplot(sledzie, aes(chel1))+geom_histogram(binwidth=6)+theme_bw()+ggtitle('dostępność planktonu helgolandicus gat. 1')

grid.arrange(c_2,chel1, nrow=1)
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-2.png)<!-- -->

```r
chel2<-ggplot(sledzie, aes(chel2))+geom_histogram(binwidth=6)+theme_bw()+ggtitle('dostępność planktonu helgolandicus gat. 2')
lcop1<-ggplot(sledzie, aes(lcop1))+geom_histogram(binwidth=12)+theme_bw()+ggtitle('zagęszczenie widłonogów gat. 1')

grid.arrange(chel2,lcop1, nrow=1)
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-3.png)<!-- -->

```r
lcop2<-ggplot(sledzie, aes(lcop2))+geom_histogram(binwidth=6)+theme_bw()+ggtitle('zagęszczenie widłonogów gat. 2')
fbar<-ggplot(sledzie, aes(fbar))+geom_histogram(binwidth=0.1)+theme_bw()+ggtitle('natężenie połowów w regionie')

grid.arrange(lcop2,fbar, nrow=1)
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-4.png)<!-- -->

```r
recr<-ggplot(sledzie, aes(recr))+geom_histogram(binwidth=5000)+theme_bw()+ggtitle('roczny narybek')
cumf<-ggplot(sledzie, aes(cumf))+geom_histogram(binwidth=0.15)+theme_bw()+ggtitle('łączne roczne natężenie połowów w regionie')

grid.arrange(recr,cumf, nrow=1)
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-5.png)<!-- -->

```r
totaln<-ggplot(sledzie, aes(totaln))+geom_histogram(binwidth=500000)+theme_bw()+ggtitle('łączna liczba ryb złowionych w ramach połowu')
sst<-ggplot(sledzie, aes(sst))+geom_histogram(binwidth=0.5)+theme_bw()+ggtitle('temperatura przy powierzchni wody')

grid.arrange(totaln,sst, nrow=1)
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-6.png)<!-- -->

```r
sal<-ggplot(sledzie, aes(sal))+geom_histogram()+theme_bw()+ggtitle('poziom zasolenia wody')
xmonth<-ggplot(sledzie, aes(xmonth))+geom_histogram(binwidth=1)+theme_bw()+ggtitle('miesiąc połowu')

grid.arrange(sal,xmonth, nrow=1)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-7.png)<!-- -->

```r
nao<-ggplot(sledzie, aes(nao))+geom_histogram(binwidth=0.5)+theme_bw()+ggtitle('oscylacja północnoatlantycka')

nao
```

![](Preview-105c1bce7c14_files/figure-html/-wykresy-8.png)<!-- -->


##ANALIZA KORELACJI

Tak wygląda macierz korelacji dla poszczególnych atrybutów:


```r
macierzkorelacji<-as.matrix(cor(sledzie[,2:16]))
macierzkorelacji
```

```
##             length        cfin1        cfin2        chel1        chel2
## length  1.00000000  0.081225529  0.098325146  0.220912259 -0.014307656
## cfin1   0.08122553  1.000000000  0.151196220  0.095406363  0.202015715
## cfin2   0.09832515  0.151196220  1.000000000 -0.003353352  0.308035340
## chel1   0.22091226  0.095406363 -0.003353352  1.000000000  0.285922229
## chel2  -0.01430766  0.202015715  0.308035340  0.285922229  1.000000000
## lcop1   0.23775402  0.122629235 -0.039985515  0.955904783  0.174800520
## lcop2   0.04894328  0.209530271  0.653719980  0.247936562  0.886313212
## fbar    0.25697135 -0.064133894  0.153148152  0.158835712  0.026895710
## recr   -0.01034244  0.118397090 -0.101775278 -0.046069994  0.001370449
## cumf    0.01152544 -0.047894573  0.337704148  0.065693458  0.262898182
## totaln  0.09605811  0.126870210 -0.218376037  0.167614840 -0.376957900
## sst    -0.45167059  0.008365852 -0.238429684 -0.216612861  0.010490540
## sal     0.03223550  0.127142618 -0.084189824 -0.147563946 -0.223746736
## xmonth  0.01371195  0.013110317  0.017475219  0.045507732  0.074408432
## nao    -0.25684475  0.005620173 -0.007106371 -0.505962978 -0.058328698
##               lcop1         lcop2         fbar          recr        cumf
## length  0.237754017  0.0489432816  0.256971351 -0.0103424429  0.01152544
## cfin1   0.122629235  0.2095302707 -0.064133894  0.1183970900 -0.04789457
## cfin2  -0.039985515  0.6537199804  0.153148152 -0.1017752776  0.33770415
## chel1   0.955904783  0.2479365619  0.158835712 -0.0460699939  0.06569346
## chel2   0.174800520  0.8863132119  0.026895710  0.0013704493  0.26289818
## lcop1   1.000000000  0.1502861605  0.093833938  0.0054818043 -0.01420244
## lcop2   0.150286160  1.0000000000  0.052422246 -0.0005897692  0.29208434
## fbar    0.093833938  0.0524222463  1.000000000 -0.2355175205  0.81676389
## recr    0.005481804 -0.0005897692 -0.235517521  1.0000000000 -0.25749903
## cumf   -0.014202439  0.2920843437  0.816763890 -0.2574990346  1.00000000
## totaln  0.267452168 -0.3041919508 -0.508164918  0.3688197501 -0.70834751
## sst    -0.265420185 -0.1197328596 -0.180818504 -0.1959423467  0.02861033
## sal    -0.099884122 -0.1866686213  0.039457960  0.2790502537 -0.10316808
## xmonth  0.030240102  0.0649399271  0.008218476  0.0186172488  0.03589074
## nao    -0.550823722 -0.0445032046  0.066515441  0.0928312133  0.22717146
##             totaln          sst         sal       xmonth          nao
## length  0.09605811 -0.451670590  0.03223550  0.013711953 -0.256844747
## cfin1   0.12687021  0.008365852  0.12714262  0.013110317  0.005620173
## cfin2  -0.21837604 -0.238429684 -0.08418982  0.017475219 -0.007106371
## chel1   0.16761484 -0.216612861 -0.14756395  0.045507732 -0.505962978
## chel2  -0.37695790  0.010490540 -0.22374674  0.074408432 -0.058328698
## lcop1   0.26745217 -0.265420185 -0.09988412  0.030240102 -0.550823722
## lcop2  -0.30419195 -0.119732860 -0.18666862  0.064939927 -0.044503205
## fbar   -0.50816492 -0.180818504  0.03945796  0.008218476  0.066515441
## recr    0.36881975 -0.195942347  0.27905025  0.018617249  0.092831213
## cumf   -0.70834751  0.028610327 -0.10316808  0.035890741  0.227171464
## totaln  1.00000000 -0.286516919  0.14918496 -0.029528694 -0.388978863
## sst    -0.28651692  1.000000000  0.01043136 -0.006810860  0.512119613
## sal     0.14918496  0.010431365  1.00000000 -0.025356696  0.124384763
## xmonth -0.02952869 -0.006810860 -0.02535670  1.000000000 -0.001110339
## nao    -0.38897886  0.512119613  0.12438476 -0.001110339  1.000000000
```

Korelację tę można przedstawić graficznie w następujący sposób:


```r
corrplot(macierzkorelacji)
```

![](Preview-105c1bce7c14_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Wnioski:

Analiza wykresu pozwala stwierdzić że długość śledzi w największym stopniu zależała od temperatury przy powierzni wody.

##WYKRES ZMIANY ROZMIARU ŚLEDZIA W CZASIE
 

```r
 ggplot(sledzie, aes(X, length))+geom_line(linetype = 1)+ggtitle("Zmiana rozmiaru śledzi przez 60 lat")
```

![](Preview-105c1bce7c14_files/figure-html/unnamed-chunk-8-1.png)<!-- -->
 
##BUDOWA REGRESORA PRZEWIDUJĄCEGO ROZMIAR ŚLEDZI ORAZ ANALIZA WAŻNOŚCI ATRYBUTÓW

####PODZIAŁ ZBIORU DANYCH


```r
set.seed(23)
inTraining<-createDataPartition(y = sledzie$length, p = .75, list = FALSE)
training <- sledzie[ inTraining,]
testing  <- sledzie[-inTraining,]
```

####SCHEMAT UCZENIA - POWTÓRZONA OCENA KRZYŻOWA

```r
ctrl <- trainControl(method = "repeatedcv",number = 2,repeats = 5)
```
####UCZENIE - RANDOM FOREST


```r
set.seed(23)
fit <- train(length ~ .,
             data = training,
             method = "rf",
             trControl = ctrl,
             ntree = 10,importance=T)
fit
```

```
## Random Forest 
## 
## 31868 samples
##    15 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (2 fold, repeated 5 times) 
## Summary of sample sizes: 15935, 15933, 15933, 15935, 15934, 15934, ... 
## Resampling results across tuning parameters:
## 
##   mtry  RMSE      Rsquared   MAE      
##    2    1.145208  0.5169196  0.9066151
##    8    1.104919  0.5512185  0.8691451
##   15    1.192487  0.5009169  0.9355418
## 
## RMSE was used to select the optimal model using the smallest value.
## The final value used for the model was mtry = 8.
```

```r
rflen <- predict(fit, newdata = testing)
```

####OCENA WAŻNOŚCI ATRYBUTÓW


```r
ggplot(varImp(fit))
```

![](Preview-105c1bce7c14_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

####REGRESJA


```r
reg<-train(length ~ ., data = sledzie, method = "lm", trControl = ctrl)
reglength <- predict(reg, newdata = testing)
```

####OCENA

```r
postResample(pred=rflen, obs = testing$length)
```

```
##      RMSE  Rsquared       MAE 
## 1.0905859 0.5665011 0.8552131
```

```r
ggplot(varImp(reg))
```

![](Preview-105c1bce7c14_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

```r
postResample(pred = reglength, obs = testing$length)
```

```
##      RMSE  Rsquared       MAE 
## 1.3389883 0.3464418 1.0516093
```

