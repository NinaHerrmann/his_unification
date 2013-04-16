//Installation
// Entpacken nach $CFG->wwwroot.'\local\lsf_unification' und Moodle Plugin-Installation starten

//unter 'Administrator' -> 'Plugins' -> 'Lokale Plugins' befinden sich zwei Erweiterungen 'LSF Unification Config' und 'LSF Unification Matching'

//LSF Unification Config
zur Konfiguration von HIS-LSF-Server/DB und Zuordnung von Rollen

//LSF Unification Matching
zur Zuordnung von �berschriften aus dem LSF zu Kategorien im Moodle (Zu ber�cksichtigen: LSF-�berschriften werden (zumindest an der WWU) jedes Semester im LSF neu angelegt (jeweils Verweis auf letztsemestrige �berschrift) und muss deshalb hin und wieder aktualisiert werden --> hier fehlt noch der entsprechende Cron-Job



Notwendige Tabellen/Sichten auf Tabellen

###Sicht der Dozenten
### Anmerkung: zivk ist die Nutzerkennung des Dozenten, der sich bei uns einloggt, hier�ber werden die Dozenten gematcht
 SELECT DISTINCT personal.pid, nutzer."login", "replace"(lower("replace"("replace"(peremail.email::text, ' '::text, ''::text), '@uni-muenster.de'::text, ''::text)), 'atuni-muensterdotde'::text, ''::text) AS zivk, personal.akadgrad, personal.vorname, personal.nachname
   FROM personal
   LEFT JOIN r_verpers ON personal.pid = r_verpers.pid
   LEFT JOIN k_verkenn ON k_verkenn.verkennid = r_verpers.verkennid
   LEFT JOIN r_pernutzer ON r_pernutzer.pid = personal.pid
   LEFT JOIN kontakt ON kontakt.tabpk = personal.pid
   LEFT JOIN nutzer ON nutzer.nid = r_pernutzer.nid
   LEFT JOIN peremail ON peremail.pid = personal.pid
  ORDER BY personal.pid, nutzer."login", "replace"(lower("replace"("replace"(peremail.email::text, ' '::text, ''::text), '@uni-muenster.de'::text, ''::text)), 'atuni-muensterdotde'::text, ''::text), personal.akadgrad, personal.vorname, personal.nachname;


### Sicht der Veranstaltungen 
SELECT veranstaltung.veranstid, veranstaltung.veranstnr, veranstaltung.semester, k_semester.ktxt AS semestertxt, k_verart.dtxt AS veranstaltungsart, veranstaltung.dtxt AS titel, veranstaltung.zeitstempel, 'http://uvlsf.uni-muenster.de/qisserver/rds?state=verpublish&status=init&vmfile=no&moduleCall=webInfo&publishConfFile=webInfo&publishSubDir=veranstaltung&publishid='::text || veranstaltung.veranstid::text AS urlveranst
   FROM veranstaltung
   LEFT JOIN k_semester ON k_semester.semid = veranstaltung.semester
   LEFT JOIN k_verart ON k_verart.verartid = veranstaltung.verartid
  WHERE (veranstaltung.semester IN ( SELECT k_semester.semid
   FROM k_semester
  WHERE k_semester.semstatus = 1)) AND (veranstaltung.veranstid IN ( SELECT r_vvzzuord.veranstid
   FROM r_vvzzuord)) AND (veranstaltung.aikz = 'A'::bpchar OR veranstaltung.aikz IS NULL);
   
   
### Sicht Zuordnung Veranstaltung zu Personal
SELECT r_verpers.veranstid, personal.pid, r_verpers.sort, k_verkenn.dtxt AS zustaendigkeit
   FROM personal
   LEFT JOIN r_verpers ON personal.pid = r_verpers.pid
   LEFT JOIN k_verkenn ON k_verkenn.verkennid = r_verpers.verkennid;
   
###Sicht zur Pflege der �berschriften
SELECT r_hierarchie.uebergeord, r_hierarchie.untergeord, r_hierarchie.semester, ueberschrift.zeitstempel, ueberschrift.ueid, ueberschrift.eid, ueberschrift.txt, ueberschrift.quellid, veranstaltung.veranstid
   FROM r_hierarchie
   LEFT JOIN ueberschrift ON ueberschrift.ueid = r_hierarchie.untergeord
   LEFT JOIN r_vvzzuord ON r_vvzzuord.ueid = ueberschrift.ueid
   LEFT JOIN veranstaltung ON veranstaltung.veranstid = r_vvzzuord.veranstid
  WHERE r_hierarchie.tabelle = 'ueberschrift'::bpchar AND r_hierarchie.beziehung = 'U'::bpchar AND ueberschrift.aikz <> 'I'::bpchar
  ORDER BY r_hierarchie.sortierung, ueberschrift.txt;
  
###Sicht Belegung der Studierenden zu einer Veranstaltung, //Wir bekommen �ber unser SSO die Matrikelnr der Studierenden, m�glicherwese bekommen wir bald auch eine Kennung mitgeliefert.
SELECT r_beleg.veranstid, r_beleg.status, r_beleg.tabpk AS mtknr, r_beleg.zeitstempel
   FROM r_beleg;

### Abk�rungen von Status in Eingeschrieben
STATUS, BEDEUTUNG
AN Angemeldet
ZU Zugelassen
AB Abgemeldet
SP Stundenplan (Ist im Stundenplan ver�ffentlicht)
WL Warteliste
ST storniert
SA Selbstablehnung
TE Teilgenommen - erfolgreich
CA canceled (Veranstaltung f�llt aus)
NE Nicht erfolgreich Teilgenommen
YY nachbelegt
WH Wiederholung
zu Dies hiert ist wohl ein Fehler. Sollte "ZU" sein.

Weitere m�gliche Kennzeichen w�ren.
NP Niedrige Priorit�t
MP Niedrige Modulpriorit�t
HP Hohe Priorit�t
TU Termin�berschneidung
XX zum Nachbelegen vorgemerkt
BB Inaktives Modul
VF Malus (Verlust der Fachsemesterpriorit�t)
