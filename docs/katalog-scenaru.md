# Katalog scénářů

Celý systém má 27 scénářů. V repozitáři je 7 reprezentativních (označené ★), ostatní fungují na stejných principech.

Časy jsou noční, aby ráno byla data hotová. Pořadí je dané závislostmi: nejdřív sběr, pak čtení, pak banka, pak výjimky a reporty.

## Sběr dokladů (8)

| Čas | Scénář | Co dělá |
|---|---|---|
| 02:00 | Doklady: celý e-mail → PDF (schránka B) | Účtenky bez přílohy (Apple, Google, Meta) převede celé na PDF |
| 02:10 | Doklady: celý e-mail → PDF (schránka C) | Totéž pro druhou schránku |
| 02:20 | Doklady: přeposlané PDF a lektoři → složka + Inbox | PDF, které si tým přeposílá, a faktury lektorů podle štítku |
| 02:25 | Doklady: PDF v přílohách vlastních e-mailů → složka + Inbox | Doklady z vlastních odeslaných e-mailů |
| 02:30 | ★ Přijaté faktury: schránka A | Gmail trigger, OCR, rozdělení firma / jiné |
| 02:35 | Přijaté faktury: schránka B | Totéž |
| 02:40 | Přijaté faktury: schránka C | Totéž |
| 03:00 | ★ Doklady: Claude čte PDF → Přijaté faktury | AI vytěží 10 polí, router zařadí doklad |

## Vystavené faktury (2)

| Čas | Scénář | Co dělá |
|---|---|---|
| 03:35 | ★ Fakturační aplikace 1 (firma) → Vystavené faktury a pohledávky | REST API s API klíčem |
| 03:40 | Fakturační aplikace 2 (OSVČ) → Vystavené faktury a pohledávky | Nejdřív OAuth token (client credentials), pak seznam faktur |

## Banka a platby (6)

| Čas | Scénář | Co dělá |
|---|---|---|
| 03:20 | ★ Bankovní API: firemní účet → Banka | Pohyby za 3 dny, deduplikace, párovací vzorec |
| 03:25 | Bankovní API: účet OSVČ → Banka | Totéž pro druhý účet |
| 04:00 | Záloha bez Stripe (etapa v CRM) | Záloha zaplacená převodem: lead v etapě → řádek v tabulce Zálohy |
| 07:16 | Stripe → list Stripe | Platby kartou do přehledu |
| 07:28 | ★ Rozdělení výplaty obchodníka (fix + provize) | Jedna platba → dva řádky nákladů |
| okamžitě | ★ Zálohy: Stripe webhook → Kommo | Hledání leadu podle e-mailu a jména, úkoly OVĚŘIT |

## Výjimky: list K řešení (5)

| Čas | Scénář | Co dělá |
|---|---|---|
| 07:24 | Nezařazené platby → K řešení | Platby, pro které neplatí žádné pravidlo |
| 07:26 | Přijaté faktury k úhradě → K řešení | Co je potřeba zaplatit |
| 07:30 | Doklady a příjmy k ověření → K řešení | Nejisté shody z párování |
| 07:30 | Ranní e-mail K řešení | Odejde jen tehdy, když je počet položek > 0 |
| 15:00 | Zpracování „Hotovo“ | Člověk vybere kategorii a zaškrtne Hotovo, scénář zapíše pravidlo zpět do Banky |

## Kontrola a reporting (2)

| Čas | Scénář | Co dělá |
|---|---|---|
| 07:30 | ★ Kontrola: zůstatky, alert a týdenní report | Alert jen při nesouladu, v pondělí přehled týmu |
| 1. v měsíci 08:00 | Měsíční report pro účetní | Odkaz na složku měsíce a kontrola před odesláním |

## Archiv a úklid (4)

| Čas | Scénář | Co dělá |
|---|---|---|
| 1. v měsíci 01:00 | Složky podle měsíců | Založí složky dopředu, ID uloží do data store |
| 1. v měsíci 04:30 | Měsíční balíček vystavených faktur (PDF) | Stáhne PDF vystavených faktur do složky měsíce |
| 1. v měsíci 04:45 | Úklid: zbytečné soubory → `_neposílat` | Co není doklad, účetní nedostane |
| 2. v měsíci 05:00 | Soubory dokladů → složka měsíce platby | Doklad patří do měsíce, kdy byl zaplacen |
