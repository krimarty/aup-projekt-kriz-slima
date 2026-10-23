# AUP projekt – nádrž (TwinCAT 3)

PLC projekt `MPCAUP_Tanky` pro simulátor `maup.NET` (plnění, ohřev, míchání, vypouštění nádrže).

| Větev | Obsah |
|---|---|
| `main` | prázdná šablona ze zadání |
| `solution` | funkční řešení |

## Nastavení TwinCATu pro verzování

TwinCAT 3 ukládá každý POU, DUT a GVL jako samostatný XML soubor, takže se v Gitu dají normálně verzovat. Bez úprav ale do diffů posílá spoustu šumu. Tohle nastavení stačí udělat jednou na každém PC.

### 1. Oddělit LineIDs (nejdůležitější)

Na konci každého `.TcPOU` je blok `<LineIds>` s ladicími informacemi. TwinCAT ho přepisuje při každém uložení, i když se kód nezmění.

**Tools → Options → TwinCAT → PLC Environment → Write options → Separate LineIDs = True**

LineIDs se pak ukládají do souboru `LineIDs.dbg`, který je v `.gitignore`. Po zapnutí dej jednou *Save All*, aby se bloky ze souborů odstranily, a tuhle změnu commitni zvlášť.

### 2. Nezapisovat verzi TwinCATu do souborů

Každý soubor má v hlavičce `ProductVersion="3.1.4024.xx"`. Když každý z nás má jiný build, mění se tahle hlavička při každém uložení.

**Tools → Options → TwinCAT → XAE Environment → File settings → Enable 'Write product version in files' = False**

(Pokud tam tu volbu nemáš, máš starší build. Pak se vyplatí mít v týmu stejnou verzi TwinCATu.)

### 3. Zafixovat verze knihoven

V `PLC_Tank.plcproj` jsou knihovny jako placeholdery s rozlišením `*` (= nejnovější nainstalovaná). Na různých PC se tak projekt může přeložit proti různým verzím.

**PLC_Tank → References → (knihovna) → Properties → Version** nastav na konkrétní verzi místo „Always newest“, nebo použij **Library Manager → Placeholders** a nastav pevné rozlišení.

### 4. Samostatné soubory pro části `.tsproj`

Konfigurace systému (`.tsproj`) je jeden velký soubor. U PLC projektu a I/O zařízení klikni pravým tlačítkem → **Independent Project File**. Každá část se pak uloží do vlastního souboru (`.xti`) a méně se bude konfliktovat.

### 5. Porovnávání pomocí TwinCAT Project Compare

Grafické POU (LD/FBD – `FB_Tank`, `Main_Batch`) a vizualizace jsou v XML nečitelné. TwinCAT obsahuje vlastní nástroj na porovnání a merge:

```
C:\TwinCAT\3.1\Components\TcProjectCompare\TcProjectCompare.exe
```

Ukáže grafické sítě a vizualizace vedle sebe. Nejjednodušší je porovnat dvě složky s projektem, třeba dva checkouty různých větví:

```
git worktree add ../aup-main main
```

Pak v TcProjectCompare otevři `MPCAUP_Tanky.sln` z obou složek.

## Pravidla pro práci v Gitu

- **Před commitem dej v TwinCATu *File → Save All*** (Ctrl+Shift+S). Git vidí jen to, co je uložené na disku.
- **Před `git checkout`, `pull` nebo `merge` zavři solution** (*File → Close Solution*) a pak ho znovu otevři. TwinCAT neumí spolehlivě načíst soubory, které se mu změní pod rukama, a při dalším uložení by je přepsal starou verzí.
- **Grafické POU a vizualizaci needitujte ve dvou najednou.** Merge LD sítí ručně skoro nejde. Práci si dělte po POU, textové ST bloky se mergují bez problémů.
- **Neverzují se vygenerované soubory:** `_Boot/`, `_CompileInfo/`, `_Libraries/`, `*.tmc`, `.vs/` atd. (viz `.gitignore`). TwinCAT si je vytvoří při buildu nebo aktivaci.
- **Trial licence (`TrialLicense.tclrs`) se neverzuje.** Je vázaná na konkrétní PC (SystemId) a platí 7 dní. Každý si ji vygeneruje sám: *SYSTEM → License → 7 Days Trial License*.
- **Konce řádků:** soubory TwinCATu mají CRLF a BOM. Na Windows nech `core.autocrlf=true` (výchozí v Git for Windows), soubory neupravuj editory, které mění kódování.
