# Lorch Update Center Agent Changelog

## 1.0.48

- Updatefeed, ZIP-Pakete und XML-Änderungsprotokoll werden dauerhaft über den verifizierten statischen GitHub-Pages-Host `https://updates.lorch-webdesign.com/` ausgeliefert.
- Das Installationsskript migriert die Joomla-Updatequelle nach erfolgreicher Installation auf den neuen kanonischen Feed.
- Der bisherige Feed unter `https://lorch-webdesign.com/updates/` bleibt ausschließlich als einmaliger Bootstrap-Kanal für bereits installierte Versionen 1.0.47 erreichbar.
- Agent-Selbstupdates verwenden weiterhin unverändert Joomlas reguläre Erweiterungs-Updateerkennung und das Update-Modell von `com_installer`.

## 1.0.47

- Agent-Selbstupdates verwenden ausschließlich Joomlas reguläre Erweiterungs-Updateerkennung über `Updater::findUpdates()` und das Update-Modell von `com_installer`.
- Eigenen Feed-Abruf, eigene XML-Auswertung, eigenen ZIP-Download, eigenes Entpacken und den manuellen Installationsaufruf aus dem Agent-Selbstupdate entfernt.
- Die von LUC signiert angeforderte Zielversion wird gegen den von Joomla erzeugten Update-Datensatz geprüft.
- Eigenen Regressionstest ergänzt, der eine Rückkehr zum umgehenden Selbstupdatepfad verhindert.

## 1.0.46

- Reiner Transport- und Migrationstest ohne neue Fachfunktion: Agent-Updatefeed und ZIP-Paket werden ausschließlich über `https://lorch-webdesign.com/updates/` ausgeliefert.
- Temporären Übergangsfeed für Agent-Versionen bis 1.0.44 nach abgeschlossener Migration aller verwalteten Websites entfernt.

## 1.0.45

- Joomla-Updatefeed, Paketdownload und Änderungsprotokoll auf den providerübergreifend getesteten Standardpfad `https://lorch-webdesign.com/updates/` umgestellt.
- Bestehende Agent-Updatequellen werden bei der Installation auf die neue kanonische Feed-Adresse migriert.
- Änderungsprotokoll zusätzlich im von Joomla erwarteten XML-Format bereitgestellt.
- Getrennter Übergangsfeed hält Agent-Versionen bis 1.0.44 mit ihrer bisherigen Domain-Freigabe updatefähig, ohne den neuen kanonischen Feed zurückzustellen.

## 1.0.44

- Persistenter Update-Bridge-State liegt nicht mehr im Joomla-Administrator-Cache, weil `finaliseUpgrade()` dieses Verzeichnis während eines Core-Updates leeren kann.
- Bridge-State und Nonce-Speicher liegen jetzt geschützt im Agent-Verzeichnis und bleiben über Dateiextraktion, Joomla-Finalisierung und Cache-Bereinigung hinweg erhalten.
- Ein Regressionstest leert den Joomla-Cache während `finaliseUpgrade()` vollständig und verlangt trotzdem einen abschließenden, jobgebundenen Erfolgsbeleg.

## 1.0.43

- Joomla-Dateiextraktion vollständig vom Joomla-Bootstrap entkoppelt: Alle mehrteiligen Extraktions- und Extraktionsfinalisierungsschritte laufen über einen eigenständigen, signierten Standalone-Endpunkt.
- Persistenten Bridge-State mit exklusiver Sperre, atomaren Schreibvorgängen, Ablaufzeit und idempotenter Request-ID ergänzt. Verlorene Webapp-Antworten führen nicht mehr zu einem doppelten Extraktionsschritt.
- Update-Passwort und Joomla-Factory-State werden nicht mehr im normalen Runner-State gespeichert.
- Ein Runner-Reset wird blockiert, sobald ein fortsetzbarer Standalone-Update-State existiert.
- Die abschließende Joomla-Datenbankfinalisierung startet erst, nachdem der Standalone-State die vollständig finalisierte Dateiextraktion bestätigt hat.

## 1.0.42

- Paket-Chunk-Download im Core-Update-Runner ist toleranter gegen kurzzeitige Range-/Download-Aussetzer.
- Ein einzelner fehlgeschlagener Chunk beendet den Core-Update-Job nicht mehr sofort, sondern wird bis zu fünfmal erneut versucht.

## 1.0.41

- Korrigiert den Runner-State-Reset aus `1.0.40`: Der Agent verwendet nun den bereits validierten Request-Payload statt den JSON-Body erneut zu lesen.
- Behebt dadurch einen Fehler, bei dem der erste Core-Update-Runner-Step mit Reset-Flag sofort als Agent-Transportfehler enden konnte.

## 1.0.40

- Core-Update-Runner kann beim Start eines neuen LUC-Core-Update-Jobs den alten Runner-State auf der Zielseite zurücksetzen.
- Verhindert, dass ein neuer Updateversuch nach Restore oder abgebrochenem Lauf fälschlich bei einem alten `update_runner_extract_step` fortsetzt.

## 1.0.39

- Core-Runner-Finalisierung korrigiert: Nach abgeschlossener Joomla-Extraktion ruft der Runner wieder die lokale `finalizeRestore`-Bridge auf, bevor Joomla `finaliseUpgrade()` ausgeführt wird.
- Wenn die lokale Extraktionsfinalisierung fehlschlägt, stoppt der Runner nun mit Diagnose statt mit einer unvollständigen Joomla-Core-Finalisierung weiterzulaufen.

## 1.0.37

- Agent-Self-Update-Abrufe von XML-Feed und ZIP-Paket erzwingen cURL-seitig einen echten GET ohne Body, HTTP/1.1 und unterdrücken einen eventuell intern gesetzten `Content-Type`-Header.
- Die Agent-Update-Diagnose meldet nun zusätzlich die verwendete Request-Methode, HTTP-Version und dass kein `Content-Type` für den GET gesendet werden soll.

## 1.0.36

- Die lokale Update-Bridge `restore.php` akzeptiert nur noch POST-Aufrufe mit kurzlebiger HMAC-Signatur.
- Externe Agent-Fehlerantworten geben keine internen Exception-Details mehr aus.

## 1.0.35

- Direkte Antworten von `core_runner_dry_run_step` redigieren nun ebenfalls interne Runner-Werte wie Update-Passwort und Factory-State. Der interne State bleibt für Folgephasen erhalten.

## 1.0.34

- Auto-Step und Extract-Start behandeln nun auch den Fall, dass Joomla die Extraktion bereits im `startRestore`-Aufruf vollständig abschließt. Der Runner setzt dann direkt mit `update_runner_finalize` fort.

## 1.0.33

- Finalisierungsstatus um `artifact_cleanup` ergänzt. Der Runner meldet nun nach dem Cleanup explizit, ob Haupt-ZIP, Handoff-ZIP, `update.php`, `restoration.php` und übrige LUC-Dry-Run-ZIPs entfernt wurden.

## 1.0.32

- Neue echte Runner-Phase `update_runner_finalize` ergänzt. Sie läuft erst nach vollständig abgeschlossener Joomla-Extraktion, ruft `finaliseUpgrade()` auf, prüft die installierte Joomla-Version und entfernt LUC-Update-Artefakte.
- Auto-Step erkennt bestehende States, bei denen die Extraktion bereits fertig ist, und setzt diese mit `update_runner_finalize` fort statt neu zu starten.

## 1.0.31

- Externer Runner-Status redigiert nun auch den Joomla-Extractor-`instance`-State, damit interne Extraktionszustände und Serverpfade nicht in der Webapp-Ausgabe erscheinen.

## 1.0.30

- Auto-Step startet nach einem fehlgeschlagenen oder abgeschlossenen Runner-State wieder sauber mit `init`, damit ein Testlauf nach einem `FAIL` nicht versehentlich denselben alten Schritt wiederholt.
- Enthält den Handoff-Fix aus `1.0.29`.

## 1.0.29

- `update_runner_prepare` legt eine Handoff-Kopie des Joomla-Pakets im Joomla-`tmp_path` ab und übergibt Joomla nur den Paket-Basename, damit die Joomla-Extraktion das Archiv korrekt öffnen kann.
- Cleanup entfernt die Handoff-ZIP sowie alte LUC-generierte `update.php`/`restoration.php`-Dateien mit LUC-Paketbezug.

## 1.0.28

- Zwei erste echte Extract-Runner-Phasen ergänzt: `update_runner_extract_start` startet die Joomla-Extraktion einmalig, `update_runner_extract_step` führt genau einen weiteren Extraktionsschritt aus.
- Interner Runner-State wird für Folgephasen roh gelesen, damit Passwort und Factory-State intern erhalten bleiben; der externe Runner-Status bleibt redigiert.
- `finaliseUpgrade` wird weiterhin nicht ausgeführt.

## 1.0.27

- Erste echte, aber noch nicht extrahierende Update-Runner-Stufe `update_runner_prepare` ergänzt. Sie ruft Joomla `createRestorationFile` auf, liest das erzeugte Update-Passwort und schreibt den vorbereiteten State, startet aber keine Extraktion.
- Dry-Run-Cleanup entfernt nur von dieser Phase neu erzeugte `restoration.php`/`update.php`-Dateien wieder; bereits vorhandene Joomla-Dateien bleiben unangetastet.

## 1.0.26

- `restoration_bridge_check` korrigiert: Pflicht sind `restore.php`, `extract.php` und die lokale `Ready`-Antwort. `restoration.php` und `update.php` werden nur noch diagnostisch gemeldet, weil sie bei Joomla 5 nicht zwingend vorhanden sind.

## 1.0.25

- Dry-Run-Chunkgröße von 1 MiB auf 4 MiB erhöht, damit der Paketdownload deutlich weniger Phasen benötigt, ohne wieder auf einen langen Einmal-Request zurückzufallen.
- Nicht-destruktive Runner-Phase `restoration_bridge_check` ergänzt. Sie prüft die lokale Joomla-Restoration-Bridge und notwendige Joomla-Update-Dateien, ohne eine Extraktion zu starten.

## 1.0.24

- Nicht-destruktive Runner-Phase `joomla_package_state` ergänzt. Sie prüft das geladene Joomla-Paket lesend im Joomla-Update-Temp-Verzeichnis und reflektiert das Joomla-Update-Model, ohne zu entpacken, Dateien zu überschreiben oder SQL auszuführen.
- Der phasenbasierte Dry-Run läuft nun nach `verify_package` über `joomla_package_state` und danach erst über `cleanup`.

## 1.0.23

- Phasenbasierten Core-Runner-Dry-Run näher an Joomla geführt: Paketdownloads werden jetzt in `administrator/components/com_joomlaupdate/tmp` abgelegt.
- Core-Runner-Probe prüft zusätzlich das Joomla-Update-Temp-Verzeichnis.
- Cleanup entfernt alte LUC-Dry-Run-ZIP-Artefakte mit festem Prefix aus dem Joomla-Update-Temp-Verzeichnis und dem bisherigen `tmp_path`.

## 1.0.22

- Phasenbasierten Core-Runner-Dry-Run ergänzt: `core_runner_dry_run_step` führt jeweils nur eine kurze Phase aus und schreibt danach sofort State.
- `core_runner_status` ergänzt, damit der lokale Runner-State nach Timeouts separat gelesen werden kann.
- Paketdownloads im phasenbasierten Dry-Run laufen in kleinen Range-Chunks, statt das Joomla-Paket in einem langen Request zu laden.

## 1.0.21

- Experimentelle Core-Runner-Phase `core_runner_dry_run` ergänzt. Sie führt Akeeba Guard aus, lädt das offizielle Joomla-Patchpaket, schreibt es testweise in `tmp_path`, prüft den Hash und entfernt die Datei wieder.
- Der Dry-Run entpackt kein Paket und startet kein Joomla-Core-Update.

## 1.0.20

- Experimentelle Core-Runner-Phase `core_runner_akeeba_guard` ergänzt. Sie prüft und deaktiviert Akeeba Backup on Update isoliert und verifiziert den Zustand, ohne ein Joomla-Core-Update zu starten.

## 1.0.19

- Experimentelle Core-Runner-Probe ergänzt. Sie führt kein Core-Update aus, sondern prüft Hosting-Voraussetzungen wie State-Verzeichnis, Joomla-Update-Model, ZIP-Entpackung und Joomla-Paketabruf.
- Bestehende Core-Update-, Backup-, Precheck- und Agent-Update-Abläufe bleiben unverändert.

## 1.0.18

- Diagnose für Agent-Self-Updates verbessert: HTML-/Bot-Challenge-Antworten beim Updatefeed werden klar als blockierte Updatequelle gemeldet.
- Updatequelle bleibt unverändert auf `https://updates.lorch-webdesign.com/plg_system_lorchupdate.xml`.

## 1.0.17

- Remote-Abrufe von XML-Feeds und ZIP-Paketen nutzen explizit HTTP/1.1 und senden zusätzlich `Accept: */*`.
- Das reduziert Probleme mit vorgeschalteten OpenResty-/WAF-Setups, die den bisherigen Feed-Abruf mit HTTP 415 ablehnen.

## 1.0.16

- Agent-Anfragen liefern bei ungefangenen PHP-Fatal-Errors eine JSON-Fehlerantwort mit Aktion, Fehlertyp, Datei und Zeile.
- Core-Update-Fehler mit HTTP 500 sind dadurch im LUC nachvollziehbar statt nur als JSON-Syntaxfehler sichtbar.

## 1.0.15

- Joomla-Updatefeed-Name stabilisiert, damit Joomla den Agenten nicht pro Version als neuen Update-Eintrag behandelt.
- Installer räumt doppelte LUC-Agent-Update-Sites für `system/lorchupdate` auf und behält die Updatequelle `https://updates.lorch-webdesign.com/plg_system_lorchupdate.xml`.

## 1.0.14

- Agent-Self-Update-Diagnose erweitert: Feed- und Paketabrufe melden jetzt HTTP-Status, cURL-Fehler, Stream-Fehler und eine gekürzte Antwortprobe zurück.
- Fehler beim Laden des LUC-Agent-Updatefeeds sind dadurch in den Job-Details nachvollziehbar statt nur als generischer Fehlschlag sichtbar.

## 1.0.13

- Core-Update-Fehlerdiagnose erweitert: lokale Update-Bridge, letzter Update-Schritt und begrenzte Joomla-Logauszüge werden ohne Secrets in die Update-Details übernommen.
- Diagnoseausgaben werden gekürzt und sensible Felder wie Passwörter, Tokens, Signaturen, Nonces und Factory-Status werden redigiert.

## 1.0.12

- Agent-Self-Update entpackt das ZIP jetzt vor der Installation über Joomla InstallerHelper und übergibt dem Installer den entpackten Paketpfad.
- Fehlerhafte direkte ZIP-Übergabe an den Joomla Installer korrigiert.

## 1.0.11

- Plugin-Manifestname auf die aktuelle Agent-Version synchronisiert, damit Joomla im Plugin-Manager nicht mehr den alten Namen `v1.0.5` anzeigt.
- Build-Prozess aktualisiert künftig neben `<version>` und der sichtbaren Versions-Note auch den Manifestnamen.

## 1.0.10

- Agent-Self-Update vorbereitet: Der Agent kann sein eigenes Plugin kontrolliert über den bekannten LUC-Updatefeed installieren.
- Der Installer akzeptiert ausschließlich das eigene Plugin `system/lorchupdate` und Pakete von `https://updates.lorch-webdesign.com/`.
- Erfolg eines Agent-Updates gilt erst nach separatem Handshake mit bestätigter neuer Agent-Version.

## 1.0.9

- Akeeba Backup on Update wird jetzt in einem separaten Update-Vorbereitungsrequest deaktiviert und verifiziert.
- Das eigentliche Core-Update startet erst in einem neuen Request, damit ein zuvor geladenes Systemplugin nicht mehr im laufenden Joomla-Dispatcher hängen kann.

## 1.0.8

- Joomla-Core-Patches werden zuerst über den offiziellen maschinenlesbaren Joomla-Updatefeed `https://update.joomla.org/core/list.xml` gegengeprüft.
- Der Agent nutzt cURL mit `file_get_contents` als Fallback, damit externe Joomla-Updatequellen auf mehr Hostingumgebungen erreichbar sind.
- Die HTML-Seite `https://downloads.joomla.org/latest` bleibt nur noch als zusätzliche Reservequelle aktiv.

## 1.0.7

- Aktives Akeeba Backup on Update wird im Precheck als Warnung statt als Fehler gemeldet.
- Die harte Absicherung bleibt direkt vor dem Core-Update bestehen: Wenn die Deaktivierung dort nicht nachweisbar ist, startet das Core-Update nicht.

## 1.0.6

- Akeeba Backup on Update wird im Precheck erkannt und blockiert aktive Core-Updates.
- Der Agent deaktiviert Akeeba Backup on Update unmittelbar vor dem Core-Update und prüft den deaktivierten Zustand nach.
- Joomla-Core-Updates werden zusätzlich gegen die offizielle Joomla-Latest-Seite geprüft, damit neue Patch-Versionen trotz veraltetem Joomla-Updatecache erkannt werden.

## 1.0.5

- Lokales Agent-Rate-Limit auf ein echtes Zeitfenster umgestellt
- Lange Akeeba-Backups mit vielen signierten Backup-Schritten laufen dadurch nicht mehr fälschlich in HTTP 429
- Keine Änderung an Signaturprüfung, Nonce-Schutz oder Backup-Retention

## 1.0.4

- Updatequelle auf `https://updates.lorch-webdesign.com` umgestellt
- Plugin-Name und Versionsanzeige auf den neuen Release-Stand synchronisiert
- Update-Distribution von der LUC-Webapp entkoppelt vorbereitet

## 1.0.3

- Lokales Rate-Limit am Joomla-Agent-Endpunkt ergänzt
- Schutz gegen wiederholte öffentliche Agent-Anfragen verbessert
- Security-Härtung für größere Rollouts vorbereitet

## 1.0.2

- Replay-Schutz in der Webapp gegen wiederverwendete signierte Agent-Requests gehärtet
- Replay-Schutz im Joomla-Agenten ergänzt, damit signierte LUC-Requests nicht erneut abgespielt werden können
- Redirect-Following im signierten Webapp-HTTP-Client deaktiviert
- Sicherheits- und Release-Stand für den Agenten bereinigt

## 1.0.1

- Plugin-Installation ohne Content-Encoding-Fehler stabilisiert
- Plugin-Version wird in den Plugin-Optionen sichtbar angezeigt
- Joomla-Core-Patch-Flow für kontrollierte Einzelfall-Updates stabilisiert
- Backup-Retention löscht ältere LUC-Archivdateien, manuelle Backups bleiben erhalten
- Joomla-Erweiterungsupdate funktioniert jetzt über einen Feed, der sich an einer funktionierenden Plugin-Referenz orientiert

## 1.0.1-rebuild

- Release-ZIP für den Feed neu gebaut
- keine neue Fachfunktion, sondern konsolidierter Paketstand

## 1.0.0-alpha10

- Joomla-Updatefeed für das Plugin ergänzt
- Update-Server im Plugin-Manifest hinterlegt
- Plugin kann damit künftig über das Joomla-Erweiterungsupdate erkannt werden
- bestehender Agent- und Updateflow bleibt unverändert

## 1.0.0-alpha9

- Core-Patch-Updates für Joomla `5.4.0 -> 5.4.5` und `5.4.4 -> 5.4.5` erfolgreich stabilisiert
