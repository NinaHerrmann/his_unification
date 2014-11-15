---
layout: recipe
title: Notwendige Sichten auf die LSF-Tabellen
---
##Notwendige Tabellen/Sichten auf Tabellen
Die Namen der Sichten können in den Definitionen zu Beginn der lib_his.php Datei manipuliert werden.

###Sicht der Dozenten
(HIS_PERSONAL)
Anmerkung: zivk ist die Nutzerkennung des Dozenten, der sich bei uns einloggt, hierüber werden die Dozenten gematcht
{% highlight sql %}
SELECT DISTINCT personal.pid, nutzer."login", "replace"(lower("replace"("replace"(peremail.email::text, ' '::text, ''::text), '@uni-muenster.de'::text, ''::text)), 'atuni-muensterdotde'::text, ''::text) AS zivk, personal.akadgrad, personal.vorname, personal.nachname
   FROM personal
   LEFT JOIN r_verpers ON personal.pid = r_verpers.pid
   LEFT JOIN k_verkenn ON k_verkenn.verkennid = r_verpers.verkennid
   LEFT JOIN r_pernutzer ON r_pernutzer.pid = personal.pid
   LEFT JOIN kontakt ON kontakt.tabpk = personal.pid
   LEFT JOIN nutzer ON nutzer.nid = r_pernutzer.nid
   LEFT JOIN peremail ON peremail.pid = personal.pid
  ORDER BY personal.pid, nutzer."login", "replace"(lower("replace"("replace"(peremail.email::text, ' '::text, ''::text), '@uni-muenster.de'::text, ''::text)), 'atuni-muensterdotde'::text, ''::text), personal.akadgrad, personal.vorname, personal.nachname;
{% endhighlight %}

### Sicht der Veranstaltungen
(HIS_VERANSTALTUNG)
{% highlight sql %}
SELECT veranstaltung.veranstid, veranstaltung.veranstnr, veranstaltung.semester, veranstaltung.kommentar, veranstaltung.zeitstempel, k_semester.ktxt AS semestertxt, k_verart.dtxt AS veranstaltungsart, veranstaltung.dtxt AS titel, 'http://uvlsf.uni-muenster.de/qisserver/rds?state=verpublish&status=init&vmfile=no&moduleCall=webInfo&publishConfFile=webInfo&publishSubDir=veranstaltung&publishid='::text || veranstaltung.veranstid::text AS urlveranst
   FROM veranstaltung
   LEFT JOIN k_semester ON k_semester.semid = veranstaltung.semester
   LEFT JOIN k_verart ON k_verart.verartid = veranstaltung.verartid
  WHERE (veranstaltung.semester IN ( SELECT k_semester.semid
   FROM k_semester
  WHERE k_semester.semstatus = 1)) AND (veranstaltung.veranstid IN ( SELECT r_vvzzuord.veranstid
   FROM r_vvzzuord)) AND (veranstaltung.aikz = 'A'::bpchar OR veranstaltung.aikz IS NULL);
{% endhighlight %}
   
### Sicht Zuordnung Veranstaltung zu Personal
(HIS_PERSONAL_VERANST)
{% highlight sql %}
 SELECT r_verpers.veranstid, personal.pid, r_verpers.sort, k_verkenn.dtxt AS zustaendigkeit
   FROM personal
   LEFT JOIN r_verpers ON personal.pid = r_verpers.pid
   LEFT JOIN k_verkenn ON k_verkenn.verkennid = r_verpers.verkennid;
{% endhighlight %}
   
###Sicht zur Pflege der Überschriften
(HIS_UEBERSCHRIFT)
{% highlight sql %}
 SELECT r_hierarchie.uebergeord, r_hierarchie.untergeord, r_hierarchie.semester, ueberschrift.zeitstempel, ueberschrift.ueid, ueberschrift.eid, ueberschrift.txt, ueberschrift.quellid, veranstaltung.veranstid
   FROM r_hierarchie
   LEFT JOIN ueberschrift ON ueberschrift.ueid = r_hierarchie.untergeord
   LEFT JOIN r_vvzzuord ON r_vvzzuord.ueid = ueberschrift.ueid
   LEFT JOIN veranstaltung ON veranstaltung.veranstid = r_vvzzuord.veranstid
  WHERE r_hierarchie.tabelle = 'ueberschrift'::bpchar AND r_hierarchie.beziehung = 'U'::bpchar AND ueberschrift.aikz <> 'I'::bpchar
  ORDER BY r_hierarchie.sortierung, ueberschrift.txt;
{% endhighlight %}
   
###Sicht zum Auslesen der Veranstaltungsbeschreibung
(HIS_VERANST_KOMMENTAR)
{% highlight sql %}
 SELECT veranstaltung.veranstid, blobs.txt AS kommentar, r_blob.sprache
   FROM veranstaltung
   LEFT JOIN r_blob ON veranstaltung.veranstid = r_blob.tabpk
   LEFT JOIN blobs ON r_blob.blobid = blobs.blobid
   LEFT JOIN k_semester ON k_semester.semid = veranstaltung.semester
  WHERE r_blob.tabelle::text = 'veranstaltung'::text AND r_blob.spalte::text = 'kommentar'::text AND (veranstaltung.semester IN ( SELECT k_semester.semid
   FROM k_semester
  WHERE k_semester.semstatus = 1));
{% endhighlight %}
   
###Sicht zum Import des Stundenplans
(HIS_STDP)
{% highlight sql %}
 SELECT DISTINCT veransttermin.vtid AS terminid, veranstaltung.veranstid, r_beleg.tabpk AS mtknr, r_beleg.status
   FROM veransttermin, veranstaltung, r_beleg
  WHERE veransttermin.tabpk = veranstaltung.veranstid AND veransttermin.tabpk = r_beleg.veranstid AND veransttermin.tabelle::text = 'veranstaltung'::text AND (veransttermin.parallelid = r_beleg.parallelid OR (veransttermin.parallelid = 0 OR veransttermin.parallelid IS NULL) AND (r_beleg.parallelid = 0 OR r_beleg.parallelid IS NULL)) AND veranstaltung.semester >= (( SELECT s_lsfsys.aktsem
           FROM s_lsfsys)) AND r_beleg.tabelle = 'sospos'::bpchar AND (r_beleg.status = 'AN'::bpchar OR r_beleg.status = 'ZU'::bpchar OR r_beleg.status = 'SP'::bpchar OR r_beleg.status = 'WL'::bpchar) AND veranstaltung.aikz <> 'I'::bpchar
  ORDER BY veransttermin.vtid, veranstaltung.veranstid, r_beleg.tabpk, r_beleg.status;
{% endhighlight %}

Die Namen der Sichten können in den Definitionen zu Beginn der lib_his.php Datei manipuliert werden.