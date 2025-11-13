

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


### Schwierigkeitsszenario 2 (und irgendwie auch 5)

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

Im Validierungsmodell wird ein Existence Constraint definiert, der prüft, ob die `Nummer` des LFP im Attribut `NBIdent` vorkommt. Es gibt zwei LFP3 in der Datei. Für ein LFP3 ist die Bedingung erfüllt, für die andere nicht. Es muss also ein Fehler gemeldet werden.

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --modeldir "https://geo.so.ch/models;https://geo.so.ch/datahub;https://data.geo.so.ch;https://models.interlis.ch" --refmapping mapping.xtf --scope 2548 lfp3_2548.xtf
```

Läuft durch aber macht noch nix. Eventuell weil gar kein Constraint definiert ist und der Ref-Datensatz nicht benötigt wird? -> Validierungsmodell mit Existence-Constraint erstellen.

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --config T202.ini --modeldir ".;https://models.interlis.ch;https://models.geo.admin.ch" /Users/stefan/Downloads/2548.ch.so.agi.dmav.relational.toleranzstufen.xtf lfp3_2548.xtf
```

Es wird der lokale Toleranzstufen-Datensatz verwendet. Dieser wird auch geprüft (ist also momentan noch kein Referenzdatensatz). Es wird korrekterweise nur ein Fehler gefunden.


Mapping-Variante:
```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --modeldir ".;https://geo.so.ch/models;https://geo.so.ch/datahub;https://data.geo.so.ch;https://models.interlis.ch" --config T202.ini --refmapping mapping.xtf --scope 2548 --allObjectsAccessible lfp3_2548.xtf
```

**-> Scheint nicht zu funktionieren. Es werden zwei Fehler gemeldet.**


Direkte Variante: 
```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --modeldir ".;https://geo.so.ch/models;https://geo.so.ch/datahub;https://data.geo.so.ch;https://models.interlis.ch" --config T202.ini --refdata ilidata:2548.ch.so.agi.dmav.relational.toleranzstufen --allObjectsAccessible lfp3_2548.xtf
```

**-> Scheint nicht zu funktionieren. Es werden zwei Fehler gemeldet.**



**Beobachtungen:**
- Es wird nicht gemeldet, falls scope nicht gefunden wird im mapping file.
- 


### Test: Baskets -- mandatory, optional, banned (2-03)

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --mandatoryBaskets "DMAV_FixpunkteAVKategorie3_V1_0.FixpunkteAVKategorie3;DMAV_Toleranzstufen_V1_0.Toleranzstufe" lfp3_2548.xtf
```

**-> korrekte Meldung, dass Toleranzstufe fehlt**

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --metaConfig mandatory_baskets_meta.ini lfp3_2548.xtf
```

**-> korrekte Meldung, dass Toleranzstufe fehlt**

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --metaConfig banned_baskets_meta.ini lfp3_2548.xtf
```

**-> korrekte Meldung, dass LFP nicht vorhanden sein darf**

### Schwierigkeitsszenario 6 (somehow)

#### Test: GWR-Funktionen (6-01)

```
java -jar /Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar --config T601.ini --plugins ./pluginsdir  Bodenbedeckung_Testdatensatz.xtf
```

**->Funktioniert nicht. "not yet implementet". Ich denke, es liegt daran, dass die Klassen nicht mit "IoxPlugin" enden, nur mit "Plugin".**

**Wir wird sqlite jdbc bekannt gemacht?**

(`java -cp '/Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar:/Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/libs/*' org.interlis2.validator.Main  --config T601.ini --plugins pluginsdir --trace Bodenbedeckung_Testdatensatz.xtf`)

Todo: 
- env




java -cp '/Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/ilivalidator-1.15.0-SNAPSHOT.jar:/Users/stefan/apps/ilivalidator-1.15.0-SNAPSHOT/libs/*' org.interlis2.validator.Main --modeldir "http://models.geo.admin.ch;." control_points.xtf

