# Struktura Google Sheets

Jedna tabulka slouží jako databáze i jako rozhraní pro tým. Scénáře zapisují data, vzorce v tabulce počítají vazby a stavy. Logika, kterou je potřeba často měnit (pravidla kategorií, kurzy, tolerance), je v listech, ne v Make, aby ji šlo upravit bez zásahu do scénářů.

| List | K čemu je | Kdo zapisuje |
|---|---|---|
| **Inbox-faktury** | Fronta nalezených dokladů: datum, odkaz na PDF, schránka, odesílatel, stav zpracování (ne / ano (Claude) / důvod odmítnutí) | Scénáře sběru, Claude |
| **Př.F** (přijaté faktury) | Dodavatel, číslo, VS, vystaveno, splatnost, částka, měna, kategorie, odkaz na PDF | Claude, ručně výjimky |
| **Vyst.F+pohl.** | Vystavené faktury, zaplaceno (SUMIFS z Banky), zbývá, stav | API scénáře, obchodníci (plánované řádky) |
| **Banka_import** | Surová data z bankovního API, beze změn | Bankovní scénáře |
| **Banka** | Vyčištěné pohyby: typ, protistrana, příjem/výdaj, kategorie, VS, stav dokladu | Bankovní scénáře + vzorce |
| **Pravidla** | Protistrana nebo účet → typ, skupina, kategorie; klíčová slova dodavatelů pro párování | Člověk (přes K řešení) |
| **Stripe** | Platby kartou | Stripe scénář |
| **Zálohy** | Zaplacené zálohy studentů, kurz, obchodník, odkaz na lead, způsob shody | Scénáře záloh |
| **K řešení** | Výjimky: nezařazené platby, faktury k úhradě, nejisté shody. Sloupec „Hotovo“ | Scénáře výjimek, člověk |
| **Kontrola** | Zůstatky, čas posledního běhu každého zdroje, kurzy EUR/USD, tolerance párování, stav OK/ALERT, HTML texty reportů | Scénáře + vzorce |

## Párování plateb a dokladů

Vzorec v listu Banka vrací index řádku v Př.F a zároveň úroveň jistoty:

- `index` = shoda podle variabilního symbolu
- `1000 + index` = shoda podle klíčového slova dodavatele, částky (s tolerancí v %) a data (doklad max. 10 dní po platbě, max. 40 dní před ní)
- `2000 + index` = shoda jen podle částky a data (±5 dní)
- `0` = doklad chybí

Ve zveřejněném blueprintu (scénář 03) je vzorec zkrácený pro čitelnost. V provozní verzi navíc platí: doklad, který už byl spárován výše v listu, se znovu nepoužije. Tím se zabrání tomu, aby se dvě stejné platby (např. dvě měsíční předplatná) spárovaly na jednu fakturu.
