# Cluster Anomaly Detection in Time Series Data - work title

## Abstract
- téma
- toto jsme udělali
- co řešíme (řešíme problem hledání clusterů dat, ne problem nějakeho algoritmu)
- takto
- takto jsme to otestovali
- toto je výsledek
## Introduction
- start with lots of refs
- describe the problem
- our main goal was to
- general -> specific (describe problem as a whole, then why the problems occurs, then why is it a problem for us, technical details, env. variables)
- constribution
- **toto až nakonec až budeme vědět co vlastně fungovalo**
- **here we describe the domain!! - aneb jak ta data vypadají - co je cílem hlavně vysvětlit že chceme cluster anomalii ne jen anomalie**
 

## Methods

The very first task is to thoroughly analyze the domain of the given problem.
The inappropriate choice of the selected solution could lead to undesirable results.
Having the problem already described, we are now able to analyze and establish a learning process. 

Using the data domain knowledge, some constraints usually arise.
As described in the introductory section, we expect the sensors to produce linear-like data, with minor deviations within the *y* axis.
These deviations do not follow any specific pattern and are completely random.
However, the errors report some kind of observable behavior.
This is usually the case when performing cluster analysis.
The main constraint that is crucial for this task is the cluster forming pattern.
The task could become straightforward if we divide it into subordinate tasks.
First of them is to use the knowledge to separate non-anomalies (not yet clusters).
Doing so, the data that is left are anomalies-only where the task of finding anomaly clusters only becomes less challenging. 

The most straightforward solution when trying to find anomalies in above-shown data would be to use some kind of statistical method that would split the data in a certain ratio.
Figure X shows the mean (straight line) of the given data. 

![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/mean_great_colored.svg) 
> Figure X - Mean of the given dataset with anomalies.

Although this may look positive on the first glance, several problems arise.
The initial one is with the automated distinction.
When the dataset is polluted with anomalies in close to 1:1 ratio, even for human it is close to impossible to differentiate anomalies and regular observation.
The second problem brings up when anomalies are not present at all, making mean method unusable.
Figure X shows the mean method when used on the dataset polluted by very little anomalies.
Obviously, if the dataset contained no anomalies at all, the result would become even more deficient.

![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/mean_wrong_colored.svg) 
> Figure X - Mean of the given dataset with little to zero anomalies.



One could easily argue that there is an option of using pure clustering algorithms (e.g. ([DBScan](doi/10.5555/3001460.3001507)).
This, however, leads to unpleasant outcome.
Such algorithms tend to view the data as a cluster-only data, despite it being irrelevant in cluster regards.
Figure X shows the performance of the DBScan algorithm on previously non-processed data, where different colors represent different clusters.
Even though the algorithm did find some clusters, it would be demanding to differentiate and find the one with anomalies.
Moreover, due to the gap in the measurement, the DBScan incorrectly split the regular observations into two clusters.
This brings up the idea of algorithm cross-cooperation.
Therefore, our proposed solution separates the anomalies first and then tries to find a cluster amongst them.

![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/DBScanGap.svg) 
> Figure X - DBScan performance

Traditional approaches for anomaly separation consist of either novelty detection or outlier detection.
Novelty detection is an anomaly detection mechanism, where we search for unusual observations, which are discovered due to their differences from the training data.
Novelty detection is a semi-supervised anomaly-detection technique, whereas outlier detection uses unsupervised methods.
This a crucial distinction, due to a fact that whereas the outlier detection is usually presented with data containing both anomalies and regular observation, it then uses mathematical models that try to make distinction between them, novelty detection on the other hand is usually presented data with little to zero anomalies (the proportion of anomalies in the dataset is called a contamination) and later, when conferred with an anomalous observation, it makes a decision.
This means, that if the dataset contains observations which look like anomalies but are still valid, the performance of unsupervised outlier detection in such case is usually unsatisfactory. 

- [ ] TODO: TADY NAPIŠ JAKOBY ŽE ISOLATION FOREST NENI NOVELTY, ALE IDK...

### Isolation Forest
- [ ] TODO: TADY napíšeme něco jakože by nás zajímalo jestli si s tím poradí isolation forest, napíšeme že se často používá právě na outlier detection ale z toho co vysvětlíme bude jasné, že se dá použít na novelty

Isolation Forest ([1](https://doi.org/10.1016/j.engappai.2022.105730 "article 1"), [2](https://doi.org/10.1016/j.patcog.2023.109334 "article 2")) is an outlier detection, semi-supervised ensemble algorithm. 
This approach is well known to successfully isolate outliers by using recursive partitioning (forming a tree-like structure) to decide whether the analyzed particle is an anomaly or not.
The less partitions required to isolate the more probable it is for a particle to be an anomaly.

### todo: tady bychom asi meli vysvětlit ten algoritmus jako takovy, nasledovat budou ruzne pokusy s hyperparametry a až podtím popíšeme že vlastně budeme muset použít IF jako novelty a vysvětlíme dále proč to je možné

- [ ] TODO: Honza tu vysvětlí jak funguje isolation forest, popíše všechny parametry a co dělají
      
#### Isolation Tree
Isolation tree je kořenový binární strom sestaven na základě vybrané podmnožiny $A$ prvků (bodů) o velikosti $s=|A|$.

1. pro sestavení isolation tree není potřeba mít početnou množinu dokonce to může být nežádoucí
2. dobře zvolené malé $s$ může pomoci odstranit *masking* a *swamping*

   masking 
   : problém anomálií v clusteru (aby byli označeny jako anomálie)
   
   swamping
   : problém normal bodů na okraji (normálního clusteru), které se jeví jako anomálie protože ty uvnitř clusteru mají moc velké ohodnocení 
3. isolation tree má dva druhy uzlů
   
   vnitřní
   : obsahuje podmínku (feature a mez) a dva potomky (jeden reprezentuje splněnou podmínku a druhý naopak nesplněnou) 
   
   vnější (list)
   : vzniká pokud podmínky rodičů splňuje (resp. nesplňuje) jeden nebo žádný prvek ze samplu, nebo je dosažena maximální hloubka stromu $l$ zpravidla $l=\ln_2(s)$. Obsahuje ohodnocení $h(x)$ pomocí vzdálenosti od kořene, pokud je dosaženo  max. délky stromu je *vzdálenost* odhadnuta pomocí $h(x)=e+c(n)$, $e$ je vzdálenost od kořene, $n$ je počet prvků ze samplu splňující podmínky rodičů, $c(n)=2\,(H_{n-1}-\frac{n-1}{N})$ a $H_{n-1}$ je $n-1$ harmonické číslo, $N$ je počet prvků celkově. 
   
- [ ] TODO: ověřit $c(n)$ nějak mi to furt nesedí
- [ ] TODO: Budeme to vysvětlovat obecně, nebo jen tak jak mi potřebujeme (2 dimenze x,y)?
- [ ] TODO: Implementace v Pythonu
   + [ ] jak je implementovaná funkce $c(x)$
   + [ ] jak je to s `max-depth` je nastaven na $\ln_2(n)$
   + [ ] jak se stanoví první interval, je $\langle 0, 1\rangle$, nebo $\langle min(data),max(data)\rangle$ nebo jinak
   + [ ] co ovlivňuje contanimation v kódu
- [ ] TODO: další možnost výzkumu (jinej článek) jak udělat isolation forest, 
  když data (features) nebudou hodnoty z intervalu, ale třeba hodnoty z konečné podmnožiny 

Despite its famousness, there are a few drawbacks.

The Scikit-Learn platform (scikit-learn.org) offers several implemented, documented and tested machine-learning open-source algorithms.
Its implementation of Isolation Forest has, in time of writing this text, 5 hyperparameters which need to be explicitly chosen and tuned.
First, the major challenge is setting the contamination parameter itself.
The contamination parameter is to control the proportion of anomalies in the dataset. 
Usually, this has to be known beforehand.
This parameter has a huge impact on the final result of the detection.
However, this is a problem because the anomalies in our dataset appear randomly and hence the proportion varies from 0% to 50%, sometimes even more.
To demonstrate the impact of contamination parameter, we prepared following experiment.
A dataset containing approx. 25% of anomalies is prepared.
Figures below show the differences when using rising values of the contamination parameter.
Note that dataset is generated randomly with each run.


![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/contamination10.svg)
> Figure X Isolation Forest with 10% contamination.

![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/contamination20.svg)
> Figure X Isolation Forest with 20% contamination.


![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/contamination30.svg)
> Figure X Isolation Forest with 30% contamination.

![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/contamination40.svg)
> Figure X Isolation Forest with 40% contamination.


Other notable parameters with huge impact on the result are *number of estimators*, *max samples* and *max features*.
Using similar dataset, we designed the experiment and tested the behavior of the Isolation Forest, with contamination parameter set to 0.25 (=25% anomalies) and varying above-mentioned parameters. 
The result of the experiment shows Figure X.
As we can see the results are quite distinct. 


![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/isolation2.svg)
> Anomaly 2:
n_estimators=50
max_samples= 20
max_features=2
contamination = 0.25

![](https://raw.githubusercontent.com/chazzka/clanekcluster/master/code/figures/isolation3.svg)

> Anomaly 3: 
n_estimators=10
max_samples= 10
max_features=2
contamination = 0.25


This kind of issue is widely known amongst AutoML community.
Some tools have already been implemented that try to deal with the issue of automatic hyperparameter tuning, namely H20 (h2o.ai) or AutoGluon (auto.gluon.ai). 

The last problem is with the unsupervised separation itself.
Consider data polluted by anomalies in close to 1:1 ratio.
Even human will find it nearly impossible to differentiate between these two classes when given plotted dataset.
Finding the line itself is obvious.
Deciding which observations are anomalies, without some domain knowledge on the other hand is close to impossible.

Despite this, one positive thing is that Isolation Forest managed to deal with the gaps in the measurement (seen in Figures above, around X=50). 

The final question is if it is somehow possible to teach Isolation Forest how regular observation look like. 
Can we use Isolation Forest for novelty detection despite it not being primarily novelty detection algorithm? 
To answer these questions, lets thoroughly analyze the Isolation Forest first.


- [ ] TODO: otázka, je teda vubec možné ho použít? Nahoře Honza vysvětlil jak funguje samotný IF a jeho parametry, tady ukážeme a něčím podpoříme, že nám nic nebrání použít IF jako novelty a uděláme na to experimenty

### Experiments using IF as a Novelty detection tool

- tady experimenty

 
### Finding the right clustering algorithm, TUNING OF DB SCAN
- tady už stačí asi že prostě to není těžký ukol, vezmeme jen obyčejný DB scan a bác. oba algoritmy jsou jednoduché ale síla je v jejich kooperaci idk
- možná bychom se tu mohli taky zamyslet nad tím jak funguje ten DB Scan a zkusit trochu potunit aby to dělalo celé clustery

### automl hyperparameter tuning for IF - v jinem članku

## experiments
- [ ] TODO: TADY SE TO UŽ ZKOMBINUJE, ukaž obrázky, ukaž jak jde měnit parametry a bude to mít jiné výslekdy, více se zvýrazní clustery, méně atp...

## Results and discussion
- tady můžeme zkusit tabulku kde budeme ukazovat kolik procent anomálií to našlo apt možná porovnání s nějakým buď expertem nebo nějakými referenčními olabelovanými daty
- zde napíšeme co se povedlo, jak to neni vubec lehke najit dva či více algoritmů které spolu dobře fungují a velký problem je jejich validace a verifikace, zkus navhrnout nějaké řešení verifikace
## Conclusion




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk2MTYwODY1MSwxODM5NTI5MTEwLC0xNT
IzMzc2NTA4LDEzODY0MjE5MjcsNjU2NDUzNSwxNzQ1MzkwNzMx
LDE4ODM3ODU0NTAsNjg3MjA4NjkyLDExNDA2Nzk5NjIsLTE3OD
k4NDIyNzgsNTk1Njg3NDU4LC0xOTQwODE2NDIzLC0xMzQzMTAx
NjY5LC0xMTk4NzI5NDAzLDE2MTQzMjMzMzAsLTU5NDI4OTYyNy
wtNjEzMTE2NTY3LC04NDA4OTcyMDgsOTc2NTQ4NDgsLTE1MzI1
NzQ0MzJdfQ==
-->
