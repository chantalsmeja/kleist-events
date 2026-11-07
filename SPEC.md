# SPEC.md · Kleist Events

Quelle: Meeting 1 (Nachr. 1–12) und Meeting 2 (Nachr. 13–20) mit Inhaberin Sabine Kleist, derselbe Chat. Zu jeder Regel steht die Nachricht, aus der sie stammt.

Ausgangslage: Kleist Events plant Hochzeiten, Firmenfeiern, Jubiläen, Produktlaunches, Stadt- und Sommerfeste und Kick-offs, 6–8 Events parallel. Sabine plant allein, ihr Mann macht die Buchhaltung, ihre Tochter hilft bei größeren Events vor Ort. Die Daten liegen in drei Excel-Tabellen, einem Notizbuch und im Mail-Postfach; montags kostet der Überblick eine Stunde, der Budgetstand ist nie auf Abruf (Nachr. 1, 4, 6, 7, 11).

## Entitäten

| Entität | Attribute |
|---|---|
| Kunde | kunden_nr, name, typ (Firma, Privat), ansprechpartner (Hochzeit: beide Auftraggeber), freigabegrenze (Betrag oder „immer Geschäftsführer"), stammkunde, notizen, status (Interessent, Kunde) |
| Event | event_nr, kunde → Kunde, vorlage → Vorlage (optional), titel, eventtyp, datum, location, gaeste_geplant, gaeste_aktuell, budgetrahmen, bestaetigt_am, anzahlung_eingegangen_am, status (Anfrage, Planung, final, durchgeführt, fertig; Seitenausgänge: nicht zustande gekommen, abgesagt) |
| Dienstleister | dienstleister_nr, name, kategorie (Catering, Technik, Musik, Personal, Floristik, Torte, Redner, Location, Agentur), kontakt, stammpartner, rahmenpreis, bewertung, nochmal_nehmen |
| Buchung | buchung_nr, event → Event, dienstleister → Dienstleister, leistung, einkaufspreis, verkaufspreis, status (angefragt, Angebot da, bestätigt, final, storniert), abrechnung (über Kleist, direkt), verauslagt, stornokosten |
| Angebot | angebot_nr, event → Event, version (Erstangebot, Nachkalkulation), honorarart (pauschal, Prozent), honorarwert, rabatt, summe, status (gesendet, wartet auf Freigabe, freigegeben), gesendet_am |
| Aufgabe | aufgabe_nr, event → Event, typ (Erstgespräch, Begehung, Menü, Ablaufplan, Technik-Briefing, Genehmigung), text, faellig_am, erledigt, zustaendig → Nutzer |
| Rechnung | rechnung_nr, event → Event, dienstleister → Dienstleister (nur eingehend), richtung (ein, aus), typ (Anzahlung, Zwischen, Schluss, Storno, Dienstleister), betrag_netto, mwst, faellig_am, status (offen, sachlich geprüft, rechnerisch geprüft, überwiesen) |
| Nutzer | name, rolle (Inhaberin, Buchhaltung, Eventtag) |
| Vorlage (Meeting 2) | vorlage_nr, name, kunde → Kunde (leer = Standard je Eventtyp), eventtyp, umfang (vollständig, Gerüst), gaeste_richtwert, budget_richtwert, honorar_richtwert |

Budget ist keine eigene Entität: geplant = `Event.budgetrahmen`, ist = Summe der Buchungen (`verkaufspreis`) plus Honorar. Heute werden dieselben Beträge in Tabelle 2 und 3 doppelt gepflegt – die Ursache des Quartals mit 130 % Budget (Nachr. 5, 6). Vertrag, Location und Kontaktpersonen sind Attribute, keine Entitäten.

## Beziehungen

- Kunde 1:n Event
- Event 1:n Angebot (Versionen)
- Event n:m Dienstleister über Buchung (einkaufspreis, verkaufspreis, status)
- Event 1:n Aufgabe n:1 Nutzer (zuständig)
- Event 1:n Rechnung
- Dienstleister 1:n Rechnung (eingehend)
- Vorlage 1:n Event (Meeting 2)
- Vorlage n:m Dienstleister (Dienstleister-Set) und Vorlage 1:n Aufgabe (Standardaufgaben) (Meeting 2)

## Business Rules

- **BR1** Anzahlung 30 % des Honorars sofort nach Auftragsbestätigung; verbindlich gebucht wird erst nach Zahlungseingang, Anfragen und Vorrecherche laufen vorher (Nachr. 2, 3, 17, 18, 20)
- **BR2** 21 Tage vor dem Event müssen alle Dienstleister final bestätigt sein, ab dann laufen Stornogebühren; Hochzeit: Floristik, Catering, Musik spätestens 8 Wochen vorher; Stadtfest: Frist aus den Genehmigungen (Nachr. 2, 16)
- **BR3** Gästezahl-Änderung über 10 % muss der Kunde schriftlich melden, Sabine darf nachkalkulieren (Nachr. 5)
- **BR4** Mehrkosten über den Budgetrahmen brauchen die Zustimmung des Kunden; bei Firmen über der kundenspezifischen Freigabegrenze eine interne Genehmigung, bis dahin wartet das Event (Nachr. 5, 9, 19)
- **BR5** Vergabe: Neukunde zwei, höchstens drei Angebote je Gewerk; Stammkunde und Hochzeit (Floristik, Torte) Direktvergabe an Stammpartner ohne Vergleich; Ausnahme bei unerklärtem Preissprung (Nachr. 2, 8, 14, 16)

Weitere Regeln für die Umsetzung:

- Honorar: pauschal bei kleinen, klar umrissenen Events, sonst 8–12 % des Gesamtbudgets; es kommt obendrauf und steckt nicht im Dienstleister-Budget (Nachr. 17, 20)
- Zahlungsfluss: Dienstleister rechnen mit Kleist Events ab; der Kunde erhält Anzahlung, bei über 6 Monaten Vorlauf oder ab 30.000 € eine Zwischenrechnung (30 % des Honorars), zum Schluss die Schlussrechnung; Zahlungsziel immer 14 Tage; auf Kundenwunsch rechnet ein Dienstleister direkt mit dem Kunden ab (Nachr. 3, 17)
- Absage durch den Kunden: Stornogebühren der Dienstleister werden vollständig weitergegeben, die Anzahlung bleibt als Ausfallhonorar, dazu anteiliges Honorar nach Planungsfortschritt (Nachr. 18)
- Rechte: Buchhaltung sieht alle Rechnungen, Zahlungen und vereinbarte Preise, nicht die Planung; Eventtag sieht Ablaufplan und Kontakte, keine Budgetzahlen und Verträge; Einkaufspreis und Marge stehen auf keinem Kundendokument (Nachr. 4, 9, 19)
- Rechnungsprüfung zweistufig: Sabine sachlich, Buchhaltung rechnerisch, dann Freigabe und Überweisung (Nachr. 19)
- Ab 10 Dienstleistern definitiv eine zweite Person am Eventtag, meist auch ab 100 Gästen oder bei schwieriger Location (Nachr. 7)
- Personal: fester Pool von 8–10 Servicekräften zuerst, sonst Agentur (Nachr. 16)
- Hochzeit: beide Auftraggeber unterschreiben, der Vertrag liegt zusätzlich auf Papier (Nachr. 11, 19)

## Widersprüche

- **W1** „Im Alltag bin ich allein … Aufgabenliste, die ich mir selbst geführt habe" (Nachr. 1–3) vs. „mein Mann macht die Buchhaltung und überweist", Tochter am Eventtag (Nachr. 3, 4) → eingeräumt in Nachr. 4, aufgelöst über `Nutzer.rolle` und `Aufgabe.zustaendig`: Planung nur Sabine, Buchhaltung und Eventtag mit eigenen Rechten
- **W2** „Bei einer Firmenfeier entscheidet der Ansprechpartner – fertig" (Nachr. 8) vs. „muss oft noch jemand intern genehmigen" (Nachr. 4) → eingeräumt in Nachr. 9, aufgelöst über `Kunde.freigabegrenze` und `Angebot.status`: operativ entscheidet der Ansprechpartner, Budget über der Grenze genehmigt die Firma intern

Zusätzlich aufgelöst: „Rechnungen weiterleiten" (Nachr. 1) vs. „prüfen, bezahlen" (Nachr. 2) → `Buchung.abrechnung` (Nachr. 3); „bester Anbieter zum besten Preis" (Nachr. 8) vs. immer derselbe Caterer bei TechCorp (Nachr. 13) → `Kunde.stammkunde`, BR5 (Nachr. 14).

## Prioritäten

1. Montag-Überblick: Events mit Termin oder Deadline diese Woche, offene Rechnungen, wartend auf Antwort – in dieser Reihenfolge; Budgetstand je Event auf Abruf (v1, Nachr. 1, 11, 20)
2. Buchungen mit Status und Preisen, Budget gegen Rahmen (v1)
3. Aufgaben je Event mit Fälligkeit und Zuständigkeit (v1)
4. Rechnungen mit Rolle Buchhaltung (v1)
5. Dienstleister-Verzeichnis mit Gewerk, Bewertung, „nochmal nehmen" (v1, Nachr. 20)
6. Vorlagen (später, nach v1 – so von Sabine selbst eingeordnet, Nachr. 12)
7. Interessenten und nicht zustande gekommene Anfragen, Dokumentverweise, mehrstufige Freigaben, Quartals- und Margenauswertung, Stadtfest-Genehmigungen, Kundenzugang (später)

## Neue Anforderung (+1, Meeting 2): Vorlagen

- Zukunftsfrage Nachr. 12: „Was planst du als Nächstes? Kommt etwas Neues dazu?" → Sabine: „Vorlagen anlegen, aus denen ich dann ein neues Event ableite – Dienstleister vorausgewählt, Standard-Aufgaben drin, Richtwert fürs Budget."
- Warum: Stammkunden buchen zwei- bis dreimal im Jahr dieselben Formate, „70 Prozent gleich wie letztes Jahr", alles wird trotzdem neu aufgesetzt (Nachr. 10, 12)
- Vorlage je Kunde und Format oder Standard je Eventtyp; sie liefert Dienstleister-Set, Standardaufgaben mit Ablaufbausteinen, Richtwerte; daraus erst das Angebot, dann das Event (Nachr. 13, 15)
- Übernommen wird nie: Datum, Location, Ansprechpartner (Nachr. 13)
- Dienstleister starten als „angefragt", Verfügbarkeit jedes Mal neu geprüft (Nachr. 13); Preise nie übernommen, jedes Jahr neu angefragt (Nachr. 14)
- Hochzeit nur als Gerüst, Stadtfest keine Vorlage (Nachr. 15)
- Priorität: nach v1 – „kurzfristig ist das Wichtigste, das Patchwork loswerden" (Nachr. 12)

## Annahmen

- Basis der 8–12 %: alle Eventkosten ohne das Honorar selbst („Dienstleister plus alles", Nachr. 20)
- Anteiliges Honorar bei Absage: Vorschlag nach Status (bestätigt = Anzahlung, Planung 50 %, final 80 %), Entscheidung je Fall
- Nachkalkulationen beim Brautpaar brauchen beide Zustimmungen; operative Entscheidungen kann einer treffen
- Stammkunde ist ein Kennzeichen, das Sabine setzt, vorgeschlagen ab dem zweiten Auftrag (Nachr. 15)
- Warnung ab 90 % des Budgetrahmens und bei Abweichung vom Vorjahrespreis; feste Schwellen setzt Sabine
- Dokumente nur als Verweis (Mail-Ordner, Papierakte); Nutzer in v1 nur Inhaberin, Buchhaltung, Eventtag
- Zwischenrechnung: die App schlägt sie vor, wenn die Bedingung aus Nachr. 17 erfüllt ist; ob sie gestellt wird, entscheidet Sabine („manchmal")
- Stadtfeste sind nicht v1; ihre Genehmigungsfristen werden je Event als Aufgabe mit `faellig_am` geführt, nicht als Regel (Nachr. 8, 15, 16)
- Kundennotizen bleiben Freitext und sind nur für die Inhaberin sichtbar; Allergien werden nicht strukturiert gespeichert (DSGVO Art. 9)
- Kein Kundenzugang in v1, Statusupdates und Freigaben laufen weiter per Mail (Nachr. 1, 4, 9)
