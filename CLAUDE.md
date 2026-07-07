<!--
  Diese Datei liest Claude Code zu Beginn JEDER Sitzung automatisch.
  Sie ist der kurze, verbindliche "Steckbrief" des Projekts – keine ausführliche Doku.
  Regeln: kurz halten, aktuell halten, KEINE Passwörter/API-Keys hier ablegen.
  (Kommentare wie dieser sind nur für Menschen und kosten Claude keinen Platz.)
-->

# CLAUDE.md — BKF-E-Learning-Plattform

## Projekt in Kürze
- **Was:** E-Learning-Plattform für die BKF-Weiterbildung (Berufskraftfahrer, 35-Stunden-Weiterbildung nach BKrFQG).
- **Für wen:** eine bereits anerkannte Fahrschule (Ausbildungsstätte). Die Plattform läuft als **White-Label** unter deren Marke.
- **Zweck:** den gesetzlich erlaubten Online-Anteil (nach aktuellem Stand bis zu 12 der 35 Stunden) rechtskonform digital abbilden. Der Rest bleibt Präsenzunterricht.
- **Oberflächensprache:** Deutsch. Alle für Nutzer sichtbaren Texte auf Deutsch.
- **Status:** in Entwicklung. Vor dem Live-Gang mit echten Teilnehmern sind Behörden-Anerkennung, Sicherheits-/Datenschutz-Check und rechtliche Unterlagen nötig (siehe unten).

## Der rechtliche Kern (WARUM es die Sonderfunktionen gibt)
Diese drei Pflichten treiben fast jede Design-Entscheidung. Baue Features so, dass sie erfüllt werden:
1. **Teilnehmeridentifikation** — es muss belegbar sein, dass die eingeloggte Person wirklich der angemeldete Teilnehmer ist. Umsetzung: Login + Zwei-Faktor (2FA); für höhere Anforderungen zusätzlich Ident-Prüfung (Video-Ident/eID) als anbindbares Modul.
2. **Anwesenheits-/Aktivitätsnachweis** — „Video lief" reicht nicht. Nächsten Abschnitt nur freischalten, wenn Verständnisfragen bestanden sind; in Live-Webinaren regelmäßige Interaktionen protokollieren.
3. **Manipulationssichere Dokumentation** — jeder relevante Schritt (wer, was, wann) wird protokolliert. Das Protokoll ist **append-only** (nur anhängen, nie ändern) und **hash-verkettet** (jeder Eintrag enthält den Hash des vorherigen), sodass nachträgliche Manipulation auffällt. Dieses Protokoll ist die Grundlage der KBA-Meldung.

Zusätzlich: **KBA-Export** — abgeschlossene Weiterbildungen müssen strukturiert exportierbar sein (für die Meldung ans Kraftfahrt-Bundesamt, Schlüsselzahl 95).

> **IMPORTANT:** Die genauen technischen Vorgaben stehen in der Berufskraftfahrerqualifikationsverordnung (BKrFQV), die noch finalisiert wird. Gesetzliche Schwellenwerte (z. B. „12 Stunden") und Regeln NICHT fest im Code verstreuen — zentral und **konfigurierbar** halten, damit spätere Anpassungen leicht sind.

## Tech-Stack
- **Basis:** WordPress (PHP).
- **LMS:** LearnDash (Kurse, Lektionen, Quizze, Fortschritt, Zertifikate).
- **Shop/Zahlung:** WooCommerce; Zahlungsanbieter Stripe oder PayPal.
- **Eigene Logik:** ein eigenes WordPress-Plugin namens `bkf-compliance` (hier lebt der gesamte BKF-Spezialcode).
- **Videos:** eingebettet von **Vimeo** (NICHT YouTube).
- **Live-Webinare:** datenschutzfreundliches Webinar-Tool (Anbindung/Verlinkung).
- **Hosting:** deutscher Anbieter, DSGVO-konform, SSL/HTTPS.

## Architektur / wo was liegt
- Standard-WordPress-Installation. **WordPress-Core, LearnDash und WooCommerce NIEMALS direkt ändern** — nur über die offiziellen Erweiterungspunkte (Hooks/Filter) erweitern.
- Aller eigener Code liegt in `wp-content/plugins/bkf-compliance/`.
- Struktur dort: `includes/` (Logik), `admin/` (Dashboard für die Fahrschule), `templates/` (Ausgabe), `assets/` (CSS/JS).

## Aufgaben des `bkf-compliance`-Plugins
- An LearnDash andocken (Lektions-/Quiz-Abschluss auslesen) und den Aktivitätsnachweis erzeugen.
- Das append-only, hash-verkettete Aktivitätsprotokoll führen und speichern.
- Identitäts-Schicht: 2FA beim Login, optionale Ident-Anbindung.
- Teilnahme in Live-Webinaren erfassen.
- Admin-Dashboard: Übersicht „wer steht wo / wer ist fertig".
- KBA-Export (strukturierte Ausgabe abgeschlossener Weiterbildungen).

## Coding-Konventionen
- **WordPress Coding Standards** (PHP) einhalten.
- Nur über **Hooks/Filter** erweitern; keine Fremd- oder Core-Dateien editieren.
- Alles mit `bkf_` / `BKF_` **präfixen** (Funktionen, Klassen, DB-Tabellen, Optionen), um Namenskonflikte zu vermeiden.
- Alle sichtbaren Texte auf **Deutsch** und **übersetzbar** (i18n, Textdomain `bkf-compliance`).
- **Kommentare auf Deutsch**, damit der (nicht-technische) Inhaber den Code nachvollziehen kann.
- In kleinen, nachvollziehbaren Schritten arbeiten und nach jedem Schritt kurz erklären, was gebaut wurde.

## Sicherheit & Datenschutz (DSGVO) — VERBINDLICH
- **YOU MUST** alle Eingaben validieren/säubern und alle Ausgaben escapen (`sanitize_*`, `esc_*`); Datenbankzugriffe nur mit `$wpdb->prepare` (keine handgebauten SQL-Strings).
- **YOU MUST** die Authentifizierung von WordPress nutzen — niemals Passwörter selbst speichern oder eigene Verschlüsselung erfinden.
- **IMPORTANT:** personenbezogene Teilnehmerdaten bleiben in der EU (deutsches Hosting). Solche Daten NIEMALS an Drittdienste außerhalb der EU senden.
- Keine personenbezogenen Daten im Klartext in Log-/Debug-Dateien schreiben.
- **Keine Geheimnisse** (API-Keys, DB-Zugänge) im Code oder in dieser Datei — nur über `wp-config.php`-Konstanten bzw. Umgebungsvariablen.
- Datensparsamkeit; Löschkonzept (Aufbewahrungsfristen) berücksichtigen.
- Vimeo enthält nur Lehrvideos (keine Teilnehmerdaten) — unkritisch.
- Keine lizenzpflichtigen Prüfungsfragen o. Ä. ohne Lizenz einbauen.

## Human Approval Required (erst fragen, dann tun)
Diese Dinge NICHT eigenmächtig ausführen, sondern vorher mit dem Menschen abstimmen:
- Änderungen, die echte **Zahlungen** auslösen oder das Live-Zahlungssystem betreffen.
- Aktionen mit **echten personenbezogenen Teilnehmerdaten** (Import, Export, Löschung).
- **Datenbank-Migrationen** oder Löschen/Umbenennen von Tabellen/Spalten.
- Einbinden **neuer Drittanbieter/Dienste** oder Senden von Daten nach außen.
- Alles, was **live** (in Produktion) geschaltet wird.

## Befehle (Commands)
*Sobald die lokale Entwicklungsumgebung steht, hier die echten Befehle eintragen. Typisch z. B.:*
- Lokale Umgebung starten: *(z. B. Local, DevKinsta oder `wp-env start`)*
- Plugin aktivieren: `wp plugin activate bkf-compliance`
- Tests ausführen: *(sobald eingerichtet)*

## Strategie-Kontext (Hintergrund, nicht Code)
Die ausführliche Markt-, Rechts- und Kostenanalyse steht in den Projektdokumenten „BKF-ELearning-Essay" und „Angebot-BKF-ELearning-Plattform". Bei fachlichen Fragen dort nachsehen. Diese Datei bleibt der knappe technische Steckbrief.

## Vor dem Live-Gang (offene Punkte)
- Finale BKrFQV-Vorgaben mit der zuständigen Anerkennungsbehörde klären.
- Sicherheits-/Datenschutz-Check durch Fachkundige.
- Rechtliche Unterlagen: Datenschutzerklärung, AV-Verträge, Verarbeitungsverzeichnis, Löschkonzept.
