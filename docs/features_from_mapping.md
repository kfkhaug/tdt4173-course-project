## Feature Engineering Pipeline - Dokumentasjon

# Prosjektet
Hydro trenger å predikere hvor mye råmateriale som vil ankomme i fremtiden for å optimalisere produksjonen. Vårt mål er å lage en ML-modell som predikerer kumulative leveranser konservativt (heller litt for lite enn for mye).

# Hva koden gjør
Koden tar rå historiske data og bygger features (variabler) som ML-modeller kan trene på.
Input:
master_data.csv - historiske leveranser per råvare per dag
prediction_mapping.csv - hvilke perioder vi skal predikere for
kernel_purchase_orders.csv - bestillinger som er lagt inn
Output:
features_for_submission.csv - features for Kaggle submission (30,450 rader)
training_features_and_labels.csv - features + labels for å trene modeller (100,000 rader)

# Pipeline-oversikt
RAW DATA → Data Cleaning → master_data.csv → Feature Engineering (VI ER HER) → features CSV-filer → Model Training → Predictions → Kaggle submission

# Hovedsteg
Lese og aggregere data
Les mapping (hva skal predikeres), historiske leveranser, og aggreger til dagsnivå.
Kritisk: complete_calendar() fyller inn manglende datoer med 0 kg, så vi har data for hver dag.
Features per dag

# Rolling window features:

r7_kg, r14_kg, r28_kg, r56_kg, r365_kg: sum vekt siste X dager
r7_days, r14_days, r28_days, r56_days, r365_days: antall dager med leveranser i vinduet
r28_mean_kg_per_day, r56_mean_kg_per_day: gjennomsnittlig vekt per leveringsdag

# Historiske features:

cum_kg: total vekt mottatt noensinne
days_since_last: dager siden siste levering

# Kalender features:

start_month, start_dow, end_month

# Same period last year:

Hvor mye ble levert i samme periode i fjor?
Bruker groupby + loop fordi merge_asof har sorteringsproblemer med store datasett

# Purchase order features:

orders_qty_in_window: totalt kg bestilt for levering i vinduet
orders_lines_in_window: antall unike bestillingslinjer
VIKTIG: Kun bestillinger kjent FØR cutoff_date (unngår data leakage)

# Submission features
For hver rad i prediction_mapping:

Finn cutoff_date = dagen før prediksjon
Hent alle features per den datoen
Beregn same period last year
Legg til purchase orders

Resultat: 30,450 rader klare for prediksjoner.

# Training features (ca. 3 minutter)
Genererer syntetiske prediksjonsvindu fra historien:

For hver råvare, lag sliding windows (hver 28. dag for å redusere datamengde)
Første vindu etter 500 dager historikk (sikrer god historikk)
Sampler 100,000 tilfeldige vindu (for å unngå minneproblemer)
Beregn samme features + faktisk label (y_window_kg)

Viktig: Bruker groupby + loop i label_from_daily() for å unngå merge_asof sorteringsproblemer.
Viktige konsepter
Temporal data leakage
Vi kan IKKE bruke informasjon fra fremtiden. Løsning:

Cutoff date = dagen før prediksjon
Kun bestillinger opprettet FØR cutoff
Historiske features kun basert på data FØR cutoff

# Time-aware validation
Training vinduer genereres kronologisk, ikke random shuffle, fordi tid er viktig i tidsseriedata.
Conservative predictions (P20)
Oppgaven krever underestimering 20% av tiden og overestimering 80%. Bedre å predikere litt for lite enn for mye (produksjonen kan fortsette med mindre materiale, men stopper hvis planlagt materiale ikke kommer).

# Hovedfunksjoner
read_mapping(): Leser prediction_mapping.csv
read_master(): Leser master_data.csv, fjerner duplikater
complete_calendar(): Fyller inn manglende datoer med 0 kg
build_daily(): Aggregerer til dagsnivå og beregner rolling features
read_orders_with_rm(): Leser purchase orders og kobler på rm_id
same_period_last_year(): Beregner leveranser i fjor (groupby + loop for robusthet)
add_orders_features(): Legger til bestillingsdata (kun kjente bestillinger)
make_features_from_mapping(): Bygger komplett feature matrix
build_training_windows_template(): Genererer syntetiske treningsvindu
label_from_daily(): Beregner faktiske leveranser (groupby + loop for robusthet)

# Feature list
Tidsperiode: ID, rm_id, forecast_start_date, forecast_end_date, cutoff_date, window_days
Historiske: cum_kg, r7/14/28/56/365_kg, r7/14/28/56/365_days, r28/56_mean_kg_per_day, days_since_last
Sammenligning: same_period_last_year_kg
Purchase orders: orders_qty_in_window, orders_lines_in_window
Kalender: start_month, start_dow, end_month
Target (kun training): y_window_kg

# Kjøretid
Steg 1-4: 30-40 sekunder
Steg 5: ca. 3 minutter (100,000 treningsvindu)
For raskere kjøring, reduser antall samples

# Quick start
Kjør koden:
python 3b_build_features_from_mapping_final.py
Bruk output:
import pandas as pd
train = pd.read_csv("data/3_processed/training_features_and_labels.csv")
X_train = train.drop(columns=["y_window_kg"])
y_train = train["y_window_kg"]
X_submit = pd.read_csv("data/3_processed/features_for_submission.csv")
Klart for ML-modellering

# Design-beslutninger
complete_calendar: Sikrer data for hver dag, gjør eksakte joins mulige
groupby + loop: Brukes i både same_period_last_year og label_from_daily for robusthet mot merge_asof sorteringsproblemer
cutoff_date: Unngår temporal data leakage
step_days=28: Reduserer antall treningsvindu (var 7, nå 28 for raskere kjøring)
min_history_days=500: Sikrer nok historikk (~16 måneder)
sampling 100k: Unngår minneproblemer (opprinnelig ~3.5M vindu)

# Tekniske utfordringer
Problem: For mange treningsvindu (3.5M) ga minneproblemer (12.8 GB)
Løsning:
Økt step_days fra 7 til 28
Økt min_history_days fra 400 til 500
Sampler 100,000 tilfeldige vindu

Problem: FutureWarning fra pandas groupby.apply
Løsning: Kan ignoreres (deprecated feature, funker fortsatt)

# Veien videre
EDA: Visualiser distribusjoner, sjekk korrelasjon
Feature Selection: Fjern høyt korrelerte features
Model Training: XGBoost, LightGBM, Random Forest
Validation: Time-series split, quantile error (P20)
Tuning: Optimize for quantile loss
Predictions: Tren på all data, generer submission