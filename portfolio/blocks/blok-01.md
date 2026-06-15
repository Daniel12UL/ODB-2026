# Blok 1 – Konzolové aplikace 
## Cíl

Chtěl jsem vytvořit aplikace, které bych mohl i normálně využít, a nebo nějaké, které jsou něčím zajímavé.

---

## Postup

Vždy jsem si prvně sepsal sám nebo s pomocí konzultanta či Ai co od aplikace potřebuji (přesné zadání) a začal jsem psát kod.
Když jsem narazil na chybu a nebo nevěděl jak dál, tak jsem se ptal ostatních --> Ai, konzultant, youtube atp.
S postupem času jsem vytvořil aplikaci

---

## Výstupy
ukázka kousku kodu probability simulatoru
- 


=== Probability simulator / CS2 case opening simulator ===
def load_from_csv(filename: str, mode: str) -> Tuple[List[SimulationOutcome], bool]:
    """Načte data z CSV souboru a provede validaci jednotlivých řádků."""
    outcomes = []
    try:
        with open(filename, mode='r', encoding='utf-8') as f:
            reader = csv.reader(f)
            next(reader, None)  # Přeskočení záhlaví tabulky
           
            for row_num, row in enumerate(reader, start=2):
                if not row or len(row) < 2:
                    print(f"Varování: Přeskočen neúplný řádek {row_num}.")
                    continue
               
                try:
                    name = row[0].strip()
                    value = float(row[1])
                   
                    if mode == "equal":
                        probability = 0.0  # Bude dopočítáno rovnoměrně následně
                    else:
                        if len(row) < 3:
                            print(f"Varování: Na řádku {row_num} chybí definice pravděpodobnosti. Přeskočeno.")
                            continue
                        probability = float(row[2]) / 100.0
                        if probability < 0:
                            print(f"Varování: Záporná pravděpodobnost na řádku {row_num}. Přeskočeno.")
                            continue
                   
                    outcomes.append(SimulationOutcome(name, value, probability))
                except ValueError:
                    print(f"Varování: Chyba formátu dat na řádku {row_num}. Přeskočeno.")







---

## Reflexe

Aplikace funguje tak, jak jsem plánoval. Překvapilo mě, jak moc práce je jen ošetření špatných vstupů – myslel jsem, že to bude jednodušší.

---

## Teoretické pozadí (stručně)

V projektu jsem pracoval s funkcemi, výjimkami (`try/except`) a se soubory. Datové struktury jsem použil hlavně seznam (list) pro historii záznamů a slovník (dict) pro jeden záznam s časem a hodnotou.

Podrobnější vysvětlení pojmů je v souboru `teorie-01.html`.

--
