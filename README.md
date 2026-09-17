# Minesweeper (Jack / Nand2Tetris)

Klasična igra Minesweeper napisana u Jacku za Hack platformu (Nand2Tetris), s ugrađenim
automatskim solverom i mogućnošću traženja pomoći(hinta).

## Struktura projekta

```
MineSweeper/
├── Board/
│ ├── Board.jack — Model ploče: mine, otkrivena/polja s zastavicom/polja s upitnikom, crtanje ploče, hint logika.
│ └── Board.vm — Kompajlirana verzija Board.jack
├── Game/
│ ├── Game.jack — Glavna igra: meni, kontrole, crtanje HUD-a, timer, tijek partije.
│ └── Game.vm — Kompajlirana verzija Game.jack
├── Main/
│ ├── Main.jack — Ulazna točka programa — pokreće Game.
│ └── Main.vm — Kompajlirana verzija Main.jack
├── Maths/
│ ├── Maths.jack — Pomoćna matematička funkcija (mod).
│ └── Maths.vm — Kompajlirana verzija Maths.jack
├── Random/
│ ├── Random.jack — Generator pseudo-slučajnih brojeva (LCG).
│ └── Random.vm — Kompajlirana verzija Random.jack
├── Solver/
│ ├── Solver.jack — Automatski solver — rješava ploču koristeći isključivo javne, upite prema Board-u (nikad ne čita stvarnu poziciju mina).
│ └── Solver.vm — Kompajlirana verzija Solver.jack
└── README.md
```

> Svaka klasa je u vlastitom folderu. Jack kompajler kompajlira folder po folder
> (jedna `.jack` datoteka po pozivu, ili sve datoteke iz jednog direktorija odjednom) —
> ako kompajliraš svaki podfolder posebno, pripazi da se sve dobivene `.vm`
> datoteke na kraju nalaze **zajedno u istom folderu** prije pokretanja u VM
> Emulatoru, jer se program sastoji od više klasa koje se međusobno pozivaju.


## Pokretanje

1. `.jack` datoteke su već kompajlirane u pripadajuće `.vm` datoteke u istom
   folderu (npr. `Solver/Solver.jack` → `Solver/Solver.vm`). Ako mijenjaš
   neki `.jack` fajl, ponovno ga kompajliraj službenim Jack kompajlerom
   (`JackCompiler` iz Nand2Tetris alata) da se `.vm` osvježi.
2. Skupi sve `.vm` datoteke iz svih podfoldera u jedan zajednički folder i
   otvori ga u **VM Emulatoru** (isporučen uz Nand2Tetris alate), pa pokreni.
3. Alternativno, ubaci `.vm` datoteke u puni Hack toolchain (VM translator → assembler)
   za pokretanje na samom Hack računalu/emulatoru.

## Kontrole — meni

| Tipka        | Akcija |
|--------------|--------|
| `1` / `E`    | Easy (9×9, 10 mina) |
| `2` / `M`    | Medium (16×10, 25 mina) |
| `3` / `H`    | Hard (20×10, 44 mina) |
| `D`          | Demo (automatska logička demonstracija solvera) |
| `C` / `4`    | Custom — ručni unos širine, visine i broja mina |
| `Q`          | Izlaz iz igre |

### Custom grid

Pri odabiru custom moda unosi se širina (3–20), visina (3–10) i broj mina.
Ograničenja postoje da ploča uvijek stane na 512×256 Hack ekran uz standardnu
veličinu polja (25×18 px), te da uvijek ostane barem jedna sigurna zona za
otvaranje ploče.

## Kontrole — u igri

| Tipka             | Funkcija tipke |
|--------------------|--------|
| Strelice           | Pomicanje kursora |
| `Space`             | Otkrij polje pod kursorom |
| `F`                 | Ciklus zastavice: skriveno → zastavica → upitnik → skriveno |
| `H`                 | Zatraži hint (otkrije jedno sigurno polje) |
| `S`                 | Jedan korak solvera |
| `A`                 | Auto-solve (solver rješava ploču korak po korak) |
| `R`                 | Restart trenutne razine/custom ploče |
| `D`                 | Pokreni  demo partiju |
| `L`                 | Povratak na meni razina |
| `Q`                 | Izlaz iz igre |

## Napomene o arhitekturi

- **`state[]` vs `mines[]`** — `Board` interno drži dva odvojena niza: `mines[]`
  (stvarna, skrivena pozicija mina) i `state[]` (što je igrač napravio sa
  svakim poljem: skriveno / otkriveno / zastavica / upitnik). `Solver` nikad
  ne čita `mines[]` izravno — sve što zna o ploči dolazi kroz uzak skup javnih
  metoda (`getVisibleState`, `getVisibleNumber`, `isHidden`, `countUnknownCells`,
  `gameWon`, ...) koje namjerno ne otkrivaju poziciju skrivenih mina.
- **Upitnik (`?`)** je čisto vizualna napomena igraču — tretira se identično
  kao skriveno polje u svakoj igrinoj i solverovoj logici (flood-fill otvaranje,
  brojanje nepoznatih susjeda, hint pretraga itd.).
- **Djelomično osvježavanje ekrana** — `Board.draw()` pamti stanje s prošlog
  crtanja (`prevState`, pozicija kursora, pozicija hinta, `showMines` flag) i
  ponovno iscrtava samo ona polja čiji se izgled stvarno promijenio, umjesto
  cijele ploče svaki potez.


