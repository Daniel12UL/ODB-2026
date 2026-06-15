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
-----------------------------------------

```
import csv
import random
from typing import List, Dict, Any, Tuple
 
class SimulationOutcome:
    """Třída reprezentující jeden možný výsledek pokusu."""
    def __init__(self, name: str, value: float, probability: float):
        self.name = name
        self.value = value
        self.probability = probability  # Ukládá se jako desetinné číslo (0.0 až 1.0)
 
    def __str__(self):
        return f"{self.name} (Hodnota: {self.value}, Pravděpodobnost: {self.probability * 100:.1f}%)"
 
 
class ProbabilitySimulator:
    """Hlavní třída pro správu konfigurace a běh simulace."""
    def __init__(self):
        self.cost_per_attempt: float = 0.0
        self.repetitions: int = 0
        self.outcomes: List[SimulationOutcome] = []
 
    def clear_data(self):
        """Vynuluje aktuální nastavení."""
        self.outcomes = []
 
    def validate_probabilities(self) -> bool:
        """Zkontroluje, zda je součet pravděpodobností roven 100 % s tolerancí na zaokrouhlování."""
        total_prob = sum(outcome.probability for outcome in self.outcomes)
        return abs(total_prob - 1.0) < 1e-5
 
    def calculate_theoretical_stats(self) -> Dict[str, float]:
        """Vypočítá teoretické (očekávané) statistiky před spuštěním simulace."""
        if not self.outcomes:
            return {}
       
        # Očekávaná hodnota (EV) = suma (hodnota * pravděpodobnost)
        expected_value = sum(o.value * o.probability for o in self.outcomes)
        expected_return = expected_value - self.cost_per_attempt
        roi_percentage = (expected_value / self.cost_per_attempt * 100) if self.cost_per_attempt > 0 else 0.0
       
        return {
            "expected_value": expected_value,
            "expected_return": expected_return,
            "roi_percentage": roi_percentage
        }
 
    def run_simulation(self) -> Dict[str, Any]:
        """Provede simulaci náhodných pokusů na základě zadaných dat."""
        if not self.outcomes or self.repetitions <= 0:
            raise ValueError("Simulaci nelze spustit. Chybí platná data nebo počet opakování.")
 
        population = self.outcomes
        weights = [o.probability for o in self.outcomes]
       
        counts = {outcome.name: 0 for outcome in self.outcomes}
        total_earned = 0.0
 
        # Generování náhodných výsledků na základě vah pravděpodobnosti
        simulated_results = random.choices(population, weights=weights, k=self.repetitions)
       
        for result in simulated_results:
            counts[result.name] += 1
            total_earned += result.value
 
        total_cost = self.repetitions * self.cost_per_attempt
        net_profit = total_earned - total_cost
 
        return {
            "counts": counts,
            "total_earned": total_earned,
            "total_cost": total_cost,
            "net_profit": net_profit,
            "is_profitable": net_profit > 0
        }
 
 
# --- POMOCNÉ FUNKCE VSTUPŮ A VALIDACE ---
 
def get_positive_float(prompt: str) -> float:
    """Získá od uživatele kladné desetinné číslo s ošetřením chybových stavů."""
    while True:
        try:
            val = float(input(prompt))
            if val < 0:
                print("Chyba: Hodnota nesmí být záporná.")
                continue
            return val
        except ValueError:
            print("Chyba: Zadejte platné numerické číslo.")
 
def get_positive_int(prompt: str) -> int:
    """Získá od uživatele kladné celé číslo větší než nula."""
    while True:
        try:
            val = int(input(prompt))
            if val <= 0:
                print("Chyba: Počet musí být větší než 0.")
                continue
            return val
        except ValueError:
            print("Chyba: Zadejte platné celé číslo.")
 
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
                   
        return outcomes, True
    except FileNotFoundError:
        print(f"Chyba: Soubor '{filename}' nebyl nalezen.")
        return [], False
 
 
# --- HLAVNÍ ŘÍZENÍ APLIKACE ---
 
def main():
    simulator = ProbabilitySimulator()
    print("=======================================")
    print("      PROBABILITY SIMULATOR 2026       ")
    print("=======================================")
   
    simulator.cost_per_attempt = get_positive_float("Zadejte cenu jednoho pokusu (Kč): ")
    simulator.repetitions = get_positive_int("Zadejte počet opakování simulace: ")
   
    print("\nZvolte režim pravděpodobnosti:")
    print("1) Equal Odds (Všechny výsledky mají stejnou šanci)")
    print("2) Custom Odds (Vlastní nastavení pravděpodobností)")
   
    mode_choice = ""
    while mode_choice not in ["1", "2"]:
        mode_choice = input("Vaše volba (1 nebo 2): ").strip()
   
    mode = "equal" if mode_choice == "1" else "custom"
 
    print("\nJak chcete zadat možné výsledky?")
    print("1) Ručně přes konzoli")
    print("2) Načíst ze souboru CSV")
   
    input_choice = ""
    while input_choice not in ["1", "2"]:
        input_choice = input("Vaše volba (1 nebo 2): ").strip()
 
    if input_choice == "1":
        num_outcomes = get_positive_int("Kolik možných výsledků existuje? ")
        for i in range(num_outcomes):
            print(f"\n--- Výsledek č. {i+1} ---")
            name = input("Název výsledku: ").strip()
            value = float(input("Hodnota/Výhra: "))
           
            prob = 0.0
            if mode == "custom":
                while True:
                    prob = float(input("Pravděpodobnost výskytu (v %): "))
                    if 0 <= prob <= 100:
                        prob = prob / 100.0
                        break
                    print("Chyba: Pravděpodobnost musí být v rozmezí 0 až 100 %.")
           
            simulator.outcomes.append(SimulationOutcome(name, value, prob))
    else:
        while True:
            filename = input("Zadejte název nebo cestu k CSV souboru (např. data.csv): ").strip()
            outcomes, success = load_from_csv(filename, mode)
            if success and outcomes:
                simulator.outcomes = outcomes
                break
            elif success and not outcomes:
                print("Varování: Soubor neobsahuje žádná platná data.")
            print("Opakujte zadání názvu souboru.")
 
    if mode == "equal" and simulator.outcomes:
        count = len(simulator.outcomes)
        equal_prob = 1.0 / count
        for outcome in simulator.outcomes:
            outcome.probability = equal_prob
 
    if not simulator.validate_probabilities():
        print("\nKritická chyba: Součet pravděpodobností se nerovná 100 %.")
        print("Program bude ukončen. Zkontrolujte vstupní data.")
        return
 
    stats = simulator.calculate_theoretical_stats()
    print("\n=======================================")
    print("       TEORETICKÉ STATISTIKY           ")
    print("=======================================")
    print(f"Průměrná hodnota výsledku:  {stats['expected_value']:.2f} Kč")
    print(f"Očekávaná návratnost (1x):  {stats['expected_return']:.2f} Kč")
    print(f"Procentuální návratnost:    {stats['roi_percentage']:.2f} %")
   
    input("\nStiskněte klávesu ENTER pro spuštění simulace...")
 
    print("\nProbíhá simulace...")
    results = simulator.run_simulation()
   
    print("\n=======================================")
    print("          VÝSLEDKY SIMULACE            ")
    print("=======================================")
    print("Počet výskytů jednotlivých výsledků:")
    for name, count in results["counts"].items():
        pct = (count / simulator.repetitions) * 100
        print(f" - {name}: {count}x ({pct:.1f}%)")
   
    print("---------------------------------------")
    print(f"Celkové náklady:         {results['total_cost']:.2f} Kč")
    print(f"Celková získaná hodnota: {results['total_earned']:.2f} Kč")
    print(f"Výsledný profit/ztráta:  {results['net_profit']:.2f} Kč")
   
    if results["is_profitable"]:
        print("Stav: Simulace skončila v kladném zisku.")
    else:
        print("Stav: Simulace skončila ve finanční ztrátě.")
    print("=======================================")
 
if __name__ == "__main__":
    main()

```




---

## Reflexe

Aplikace funguje tak, jak jsem plánoval. Překvapilo mě, jak moc práce je jen ošetření špatných vstupů – myslel jsem, že to bude jednodušší.

---

## Teoretické pozadí (stručně)

V projektu jsem pracoval s funkcemi, výjimkami (`try/except`) a se soubory. Datové struktury jsem použil hlavně seznam (list) pro historii záznamů a slovník (dict) pro jeden záznam s časem a hodnotou.

Podrobnější vysvětlení pojmů je v souboru `teorie-01.md`.

--
