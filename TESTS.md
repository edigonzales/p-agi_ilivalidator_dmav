

## Datenrepositories

https://geo.so.ch/models/ilisite.xml

```
<subsidiarySite>
  <IliSite09.RepositoryLocation_>
    <value>https://geo.so.ch/datahub/</value>
  </IliSite09.RepositoryLocation_>
  <IliSite09.RepositoryLocation_>
    <value>https://data.geo.so.ch/</value>
  </IliSite09.RepositoryLocation_>
  <IliSite09.RepositoryLocation_>
    <value>https://geo.so.ch/usabilityhub/</value>
  </IliSite09.RepositoryLocation_>
</subsidiarySite>
```

Ohne Angabe von `--modeldir` werden nach bekannter Reihenfolge (siehe KGK) alle anderen Kanton durchsucht. Landet man beim Kanton Solothurn Haupt-Repo, wird anschliessend das Repo des Datahub durchsucht und dann das Repo des Datenbezugs. Im Datenbezug erscheinen die Daten erst nach dem täglichen Import in die Datenbank etc. Im Datahub-Repo bereits nach erfolgreicher Prüfung.

## Tests

### Schwierigkeitsszenario 1b

Die für die Validierung notwendigen Daten liegen im lokalen Filesystem aber nicht im Format XTF vor (z.B. CSV, GeoPackage).

-> Out-of-scope.


### Schwierigkeitsszenario 2

Entspricht der Stufe 1, aber die für die Validierung benötigten Datensätze liegen in diversen und öffentlichen Daten-Repositories vor.

Die Referenzdaten müssen mit stabilen BasketIDs in Daten-Repositories aktuell vorliegen. Die BasketIDs dienen dazu, den Datensatz als den für die Validierung relevanten zu erkennen.

Der Validator muss für den spezifischen Validierungs-Run wissen, welche Referenzdaten benötigt werden. Es braucht also ein Mapping in welchem pro Modell und Gemeinde, die notwendigen Referenzdatensätze konfiguriert sind (Angabe BasketID zu Referenzdatensatz und Gemeinde). Der Validator muss das Mapping interpretieren können.

#### Test: Validierung von Daten in einem Datenrepository (2-01)

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar ilidata:2548.ch.so.agi.dmav.relational.fixpunkteavkategorie3
```

Reihenfolge von der Repositories verändert:

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --modeldir "https://geo.so.ch/models;https://geo.so.ch/datahub;https://data.geo.so.ch;https://models.interlis.ch" ilidata:2548.ch.so.agi.dmav.relational.fixpunkteavkategorie3
```

#### Test: Validierung von lokalen Daten und Referenzdaten in einem Datenrepository (2-02)

Das Mapping wird benötigt, um einem Validierungs-Run mitzuteilen, wo für welche Gemeinde und welches Modell die notwendigen Referenzdatensätze gespeichert sind. 

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --modeldir "https://geo.so.ch/models;https://geo.so.ch/datahub;https://data.geo.so.ch;https://models.interlis.ch" --refmapping mapping.xtf --scope 2548 lfp3_2548.xtf
```

Läuft durch aber macht noch nix. Eventuell weil gar kein Constraint definiert ist und der Ref-Datensatz nicht benötigt wird? -> Validierungsmodell mit Existence-Constraint "TS2" bei Nummer eintragen.