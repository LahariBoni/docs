<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/FEC/FEC_Creer_un_dashboard_PowerBI.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=FEC+-+Creer+un+dashboard+PowerBI:+Error+short+description">🚨 Bug report</a>

**Tags:** #fec #powerbi #dataviz #analytics #finance

**Author:** [Alexandre STEVENS](https://www.linkedin.com/in/alexandrestevenspbix/)

Ce Notebook permet de transformer des fichiers FEC de votre entreprise en un tableau de bord Microsoft Power BI.
Le FEC (fichier des écritures comptables) est un export standard des logiciels de comptabilite et une obligation légale en france depuis 2014 afin de déposer ses comptes de manière electronique auprès des services fiscaux.
<br><br>
-Durée de l’installation = 5 minutes<br>
-Support d’installation = [Page Notion](https://www.notion.so/Mode-d-emploi-FECthis-7fc142f2d7ae4a3889fbca28a83acba2/)<br>
-Niveau = Facile<br>

## Input

### Librairie


```python
import pandas as pd
from datetime import datetime, timedelta
import os
import re
import naas
```

### Lien URL vers le logo de l'entreprise


```python
LOGO = "https://landen.imgix.net/e5hx7wyzf53f/assets/26u7xg7u.png?w=400"
COLOR_1 = None
COLOR_2 = None
```

### Lire les fichiers FEC


```python
def get_all_fec(file_regex,
                sep=",",
                decimal=".",
                encoding=None,
                header=None,
                usecols=None,
                names=None,
                dtype=None):
    # Create df init
    df = pd.DataFrame()

    # Get all files in INPUT_FOLDER
    files = [f for f in os.listdir() if re.search(file_regex, f)]
    if len(files) == 0:
        print(f"Aucun fichier FEC ne correspond au standard de nomination")
    else:
        for file in files:
            # Open file and create df
            print(file)
            tmp_df = pd.read_csv(file,
                                 sep=sep,
                                 decimal=decimal,
                                 encoding=encoding,
                                 header=header,
                                 usecols=usecols,
                                 names=names,
                                 dtype=dtype)
            # Add filename to df
            tmp_df['NOM_FICHIER'] = file

            # Concat df
            df = pd.concat([df, tmp_df], axis=0, sort=False)
    return df
```


```python
file_regex = "^\d{9}FEC\d{8}.txt"

db_init = get_all_fec(file_regex,
                      sep='\t',
                      decimal=',',
                      encoding='ISO-8859-1',
                      header=0)
db_init
```

## Model

### Base de donnée FEC

#### Nettoyage des données


```python
db_clean = db_init.copy()

# Selection des colonnes à conserver
to_select = ['NOM_FICHIER',
             'EcritureDate',
             'CompteNum',
             'CompteLib',
             'EcritureLib',
             'Debit',
             'Credit']
db_clean = db_clean[to_select]

# Renommage des colonnes
to_rename = {'EcritureDate': "DATE", 
             'CompteNum': "COMPTE_NUM", 
             'CompteLib': "RUBRIQUE_N3", 
             'EcritureLib': "RUBRIQUE_N4", 
             'Debit': "DEBIT", 
             'Credit': "CREDIT"}
db_clean = db_clean.rename(columns=to_rename)

#suppression des espaces colonne "COMPTE_NUM"
db_clean["COMPTE_NUM"] = db_clean["COMPTE_NUM"].astype(str).str.strip()

# Mise au format des colonnes
db_clean = db_clean.astype({"NOM_FICHIER" : str,
                            "DATE" : str,
                            "COMPTE_NUM" : str,
                            "RUBRIQUE_N3" : str,
                            "RUBRIQUE_N4" : str,
                            "DEBIT" : float,
                            "CREDIT" : float,
                            })

# Mise au format colonne date
db_clean["DATE"] = pd.to_datetime(db_clean["DATE"])

db_clean.head(5)
```

#### Enrichissement de la base


```python
db_enr = db_clean.copy()

# Ajout colonnes entité et période
db_enr['ENTITY'] = db_enr['NOM_FICHIER'].str[:9]
db_enr['PERIOD'] = db_enr['NOM_FICHIER'].str[12:-6]
db_enr['PERIOD'] = pd.to_datetime(db_enr['PERIOD'], format='%Y%m')
db_enr['PERIOD'] = db_enr['PERIOD'].dt.strftime("%Y-%m")

# Ajout colonne month et month_index
db_enr['MONTH'] = db_enr['DATE'].dt.strftime("%b")
db_enr['MONTH_INDEX'] = db_enr['DATE'].dt.month

# Calcul de la valeur debit-crédit
db_enr["VALUE"] = (db_enr["DEBIT"]) - (db_enr["CREDIT"])

db_enr.head(5)
```


```python
# Calcul résultat pour équilibrage bilan dans capitaux propre
db_rn = db_enr.copy()

db_rn = db_rn[db_rn['COMPTE_NUM'].str.contains(r'^6|^7')]

to_group = ["ENTITY", "PERIOD"]
to_agg = {"VALUE": "sum"}
db_rn = db_rn.groupby(to_group, as_index=False).agg(to_agg)

db_rn ["COMPTE_NUM"] = "10999999"
db_rn ["RUBRIQUE_N3"] = "RESULTAT"

# Reorganisation colonne
to_select = ['ENTITY',
             'PERIOD',
             'COMPTE_NUM',
             'RUBRIQUE_N3',
             'VALUE']
db_rn = db_rn[to_select]
db_rn
```

### Base de données FEC aggrégée avec variation

#### Aggrégation RUBRIQUE N3


```python
# Calcul var v = création de dataset avec Period_comp pour merge
db_var = db_enr.copy()

# Regroupement 
to_group = ["ENTITY",
            "PERIOD",
            "COMPTE_NUM",
            "RUBRIQUE_N3"]
to_agg = {"VALUE": "sum"}
db_var = db_var.groupby(to_group, as_index=False).agg(to_agg)

# Ajout des résultats au dataframe
db_var = pd.concat([db_var, db_rn], axis=0, sort=False)

# Creation colonne COMP
db_var['PERIOD_COMP'] = (db_var['PERIOD'].str[:4].astype(int) - 1).astype(str) + db_var['PERIOD'].str[-3:]

db_var
```

#### Création de la base comparable


```python
db_comp = db_var.copy()

# Suppression de la colonne période
db_comp = db_comp.drop("PERIOD_COMP", axis=1)

# Renommage des colonnes
to_rename = {'VALUE': "VALUE_N-1", 
             'PERIOD': "PERIOD_COMP"}
db_comp = db_comp.rename(columns=to_rename)

db_comp.head(5)
```

#### Jointure des 2 tables et calcul des variations


```python
# Jointure entre les 2 tables
join_on = ["ENTITY",
           "PERIOD_COMP",
           "COMPTE_NUM",
           "RUBRIQUE_N3"]
db_var = pd.merge(db_var, db_comp, how='left', on=join_on).drop("PERIOD_COMP", axis=1).fillna(0)

#Création colonne Var V
db_var["VARV"] = db_var["VALUE"] - db_var["VALUE_N-1"]

#Création colonne Var P (%)
db_var["VARP"] = db_var["VARV"] / db_var["VALUE_N-1"]

db_var
```


```python
db_cat = db_var.copy()

# Calcul des rubriques niveau 2
def rubrique_N2(row): 
        numero_compte = str(row["COMPTE_NUM"])
        value = float(row["VALUE"])
        
# BILAN SIMPLIFIE type IFRS NIV2

        to_check = ["^10", "^11", "^12", "^13", "^14"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "CAPITAUX_PROPRES"
        
        to_check = ["^15", "^16", "^17", "^18", "^19"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "DETTES_FINANCIERES"
        
        to_check = ["^20", "^21", "^22", "^23", "^25", "^26", "^27", "^28", "^29"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "IMMOBILISATIONS"
        
        to_check = ["^31", "^32", "^33", "^34", "^35", "^36", "^37", "^38", "^39"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "STOCKS"
        
        to_check = ["^40"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "DETTES_FOURNISSEURS"
        
        to_check = ["^41"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "CREANCES_CLIENTS"
        
        to_check = ["^42", "^43", "^44", "^45", "^46", "^47", "^48", "^49"]
        if any (re.search(x,numero_compte) for x in to_check):
            if value > 0:
                return "AUTRES_CREANCES"
            else:
                return "AUTRES_DETTES"
        
        to_check = ["^50", "^51", "^52", "^53", "^54", "^58", "^59"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "DISPONIBILITES"

        
# COMPTE DE RESULTAT DETAILLE NIV2

        to_check = ["^60"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "ACHATS"
        
        to_check= ["^61", "^62"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "SERVICES_EXTERIEURS"
        
        to_check = ["^63"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "TAXES"        
    
        to_check = ["^64"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "CHARGES_PERSONNEL"    
    
        to_check = ["^65"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "AUTRES_CHARGES"  
    
        to_check = ["^66"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "CHARGES_FINANCIERES"
        
        to_check = ["^67"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "CHARGES_EXCEPTIONNELLES"
        
        to_check = ["^68", "^78"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "AMORTISSEMENTS"
        
        to_check = ["^69"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "IMPOT"
        
        to_check = ["^70"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "VENTES"
        
        to_check = ["^71", "^72"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "PRODUCTION_STOCKEE_IMMOBILISEE"
        
        to_check = ["^74"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "SUBVENTIONS_D'EXPL."
        
        to_check = ["^75", "^791"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "AUTRES_PRODUITS_GESTION_COURANTE"
        
        to_check = ["^76", "^796"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "PRODUITS_FINANCIERS"
        
        to_check = ["^77", "^797"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "PRODUITS_EXCEPTIONNELS"
        
        to_check = ["^78"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "REPRISES_AMORT._DEP."

        to_check = ["^8"]
        if any (re.search(x,numero_compte) for x in to_check):
            return "COMPTES_SPECIAUX"

        
# Calcul des rubriques niveau 1
def rubrique_N1(row):
    categorisation = row.RUBRIQUE_N2
    
# BILAN SIMPLIFIE type IFRS N1

    to_check = ["CAPITAUX_PROPRES", "DETTES_FINANCIERES"]
    if any(re.search(x, categorisation) for x in to_check):
        return "PASSIF_NON_COURANT"
    
    to_check = ["IMMOBILISATIONS"]
    if any(re.search(x, categorisation) for x in to_check):
        return "ACTIF_NON_COURANT"
    
    to_check = ["STOCKS", "CREANCES_CLIENTS", "AUTRES_CREANCES"]
    if any(re.search(x, categorisation) for x in to_check):
        return "ACTIF_COURANT"   
        
    to_check = ["DETTES_FOURNISSEURS", "AUTRES_DETTES"]
    if any(re.search(x, categorisation) for x in to_check):
        return "PASSIF_COURANT"
    
    to_check = ["DISPONIBILITES"]
    if any(re.search(x, categorisation) for x in to_check):
        return "DISPONIBILITES"

    
# COMPTE DE RESULTAT SIMPLIFIE N1
    
    to_check = ["ACHATS"]
    if any(re.search(x, categorisation) for x in to_check):
        return "COUTS_DIRECTS"
    
    to_check = ["SERVICES_EXTERIEURS", "TAXES", "CHARGES_PERSONNEL", "AUTRES_CHARGES", "AMORTISSEMENTS"]
    if any(re.search(x, categorisation) for x in to_check):
        return "CHARGES_EXPLOITATION"
    
    to_check = ["CHARGES_FINANCIERES"]
    if any(re.search(x, categorisation) for x in to_check):
        return "CHARGES_FINANCIERES"
        
    to_check = ["CHARGES_EXCEPTIONNELLES", "IMPOT"]
    if any(re.search(x, categorisation) for x in to_check):
        return "CHARGES_EXCEPTIONNELLES"        

    to_check = ["VENTES", "PRODUCTION_STOCKEE_IMMOBILISEE"]
    if any(re.search(x, categorisation) for x in to_check):
        return "CHIFFRE_D'AFFAIRES"
    
    to_check = ["SUBVENTIONS_D'EXPL.", "AUTRES_PRODUITS_GESTION_COURANTE", "REPRISES_AMORT._DEP."]
    if any(re.search(x, categorisation) for x in to_check):
        return "PRODUITS_EXPLOITATION"
    
    to_check = ["PRODUITS_FINANCIERS"]
    if any(re.search(x, categorisation) for x in to_check):
        return "PRODUITS_FINANCIERS"
        
    to_check = ["PRODUITS_EXCEPTIONNELS"]
    if any(re.search(x, categorisation) for x in to_check):
        return "PRODUITS_EXCEPTIONNELS"      
        

# Calcul des rubriques niveau 0
def rubrique_N0(row):
    masse = row.RUBRIQUE_N1
    
    to_check = ["ACTIF_NON_COURANT", "ACTIF_COURANT", "DISPONIBILITES"]
    if any(re.search(x, masse) for x in to_check):
        return "ACTIF"
    
    to_check = ["PASSIF_NON_COURANT", "PASSIF_COURANT"]
    if any(re.search(x, masse) for x in to_check):
        return "PASSIF"

    to_check = ["COUTS_DIRECTS", "CHARGES_EXPLOITATION", "CHARGES_FINANCIERES", "CHARGES_EXCEPTIONNELLES"]
    if any(re.search(x, masse) for x in to_check):
        return "CHARGES"   
        
    to_check = ["CHIFFRE_D'AFFAIRES", "PRODUITS_EXPLOITATION", "PRODUITS_FINANCIERS", "PRODUITS_EXCEPTIONNELS"]
    if any(re.search(x, masse) for x in to_check):
        return "PRODUITS"  
    
    
# Mapping des rubriques 
db_cat["RUBRIQUE_N2"] = db_cat.apply(lambda row: rubrique_N2(row), axis=1)
db_cat["RUBRIQUE_N1"] = db_cat.apply(lambda row: rubrique_N1(row), axis=1)
db_cat["RUBRIQUE_N0"] = db_cat.apply(lambda row: rubrique_N0(row), axis=1)    


# Reorganisation colonne
to_select = ['ENTITY',
             'PERIOD',
             'COMPTE_NUM', 
             'RUBRIQUE_N0',
             'RUBRIQUE_N1',
             'RUBRIQUE_N2',
             'RUBRIQUE_N3',
             'VALUE',
             'VALUE_N-1',
             'VARV',
             'VARP']
db_cat = db_cat[to_select]

db_cat
```

### Modèles de données des graphiques

#### REF_ENTITE


```python
# Creation du dataset ref_entite
dataset_entite = db_cat.copy()

# Regrouper par entite
to_group = ["ENTITY"]
to_agg = {"ENTITY": "max"}
dataset_entite = dataset_entite.groupby(to_group, as_index=False).agg(to_agg)

# Affichage du modèle de donnée
dataset_entite
```

#### REF_SCENARIO


```python
# Creation du dataset ref_scenario
dataset_scenario = db_cat.copy()

# Regrouper par entite
to_group = ["PERIOD"]
to_agg = {"PERIOD": "max"}
dataset_scenario = dataset_scenario.groupby(to_group, as_index=False).agg(to_agg)

# Affichage du modèle de donnée
dataset_scenario
```

#### KPIS


```python
# Creation du dataset KPIS (CA, MARGE, EBE, BFR, CC, DF)
dataset_kpis = db_cat.copy()

# KPIs CA
dataset_kpis_ca = dataset_kpis[dataset_kpis.RUBRIQUE_N1.isin(["CHIFFRE_D'AFFAIRES"])]

to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N1"]
to_agg = {"VALUE": "sum"}
dataset_kpis_ca = dataset_kpis_ca.groupby(to_group, as_index=False).agg(to_agg)

# Passage value postif
dataset_kpis_ca["VALUE"] = dataset_kpis_ca["VALUE"]*-1


# COUTS_DIRECTS
dataset_kpis_ha = dataset_kpis[dataset_kpis.RUBRIQUE_N1.isin(["COUTS_DIRECTS"])]

to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N1"]
to_agg = {"VALUE": "sum"}
dataset_kpis_ha = dataset_kpis_ha.groupby(to_group, as_index=False).agg(to_agg)

# Passage value négatif
dataset_kpis_ha["VALUE"] = dataset_kpis_ha["VALUE"]*-1


# KPIs MARGE BRUTE (CA - COUTS DIRECTS)
dataset_kpis_mb = dataset_kpis_ca.copy()
dataset_kpis_mb = pd.concat([dataset_kpis_mb, dataset_kpis_ha], axis=0, sort=False)

to_group = ["ENTITY",
            "PERIOD"]
to_agg = {"VALUE": "sum"}

dataset_kpis_mb = dataset_kpis_mb.groupby(to_group, as_index=False).agg(to_agg)
dataset_kpis_mb["RUBRIQUE_N1"] = "MARGE"
dataset_kpis_mb = dataset_kpis_mb[["ENTITY", "PERIOD", "RUBRIQUE_N1", "VALUE"]]


# CHARGES EXTERNES
dataset_kpis_ce = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["SERVICES_EXTERIEURS"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_ce = dataset_kpis_ce.groupby(to_group, as_index=False).agg(to_agg)

# Passage value negatif
dataset_kpis_ce["VALUE"] = dataset_kpis_ce["VALUE"]*-1


# IMPOTS
dataset_kpis_ip = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["TAXES"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_ip = dataset_kpis_ip.groupby(to_group, as_index=False).agg(to_agg)

# Passage value negatif
dataset_kpis_ip["VALUE"] = dataset_kpis_ip["VALUE"]*-1


# CHARGES DE PERSONNEL
dataset_kpis_cp = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["CHARGES_PERSONNEL"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_cp = dataset_kpis_cp.groupby(to_group, as_index=False).agg(to_agg)

# Passage value negatif
dataset_kpis_cp["VALUE"] = dataset_kpis_cp["VALUE"]*-1


# AUTRES_CHARGES
dataset_kpis_ac = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["AUTRES_CHARGES"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_ac = dataset_kpis_ac.groupby(to_group, as_index=False).agg(to_agg)

# Passage value negatif
dataset_kpis_ac["VALUE"] = dataset_kpis_ac["VALUE"]*-1


# SUBVENTIONS D'EXPLOITATION
dataset_kpis_ac = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["SUBVENTIONS_D'EXPL."])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_ac = dataset_kpis_ac.groupby(to_group, as_index=False).agg(to_agg)


# KPIs EBE = MARGE - CHARGES EXTERNES - TAXES - CHARGES PERSONNEL - AUTRES CHARGES + SUBVENTION D'EXPLOITATION
dataset_kpis_ebe = dataset_kpis_mb.copy()
dataset_kpis_ebe = pd.concat([dataset_kpis_ebe, dataset_kpis_ce, dataset_kpis_ip, dataset_kpis_cp, dataset_kpis_ac], axis=0, sort=False)

to_group = ["ENTITY", "PERIOD"]
to_agg = {"VALUE": "sum"}

dataset_kpis_ebe = dataset_kpis_ebe.groupby(to_group, as_index=False).agg(to_agg)
dataset_kpis_ebe["RUBRIQUE_N1"] = "EBE"
dataset_kpis_ebe = dataset_kpis_ebe[["ENTITY", "PERIOD", "RUBRIQUE_N1", "VALUE"]]


# KPIs CREANCES CLIENTS
dataset_kpis_cc = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["CREANCES_CLIENTS"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_cc = dataset_kpis_cc.groupby(to_group, as_index=False).agg(to_agg)

# Renommage colonne
to_rename = {'RUBRIQUE_N2': "RUBRIQUE_N1"}
dataset_kpis_cc = dataset_kpis_cc.rename(columns=to_rename)


# KPIs STOCKS
dataset_kpis_st = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["STOCKS"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_st = dataset_kpis_st.groupby(to_group, as_index=False).agg(to_agg)

# Renommage colonne
to_rename = {'RUBRIQUE_N2': "RUBRIQUE_N1"}
dataset_kpis_st = dataset_kpis_st.rename(columns=to_rename)


# KPIs DETTES FOURNISSEURS
dataset_kpis_df = dataset_kpis[dataset_kpis.RUBRIQUE_N2.isin(["DETTES_FOURNISSEURS"])]
to_group = ["ENTITY", "PERIOD", "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_kpis_df = dataset_kpis_df.groupby(to_group, as_index=False).agg(to_agg)

# Renommage colonne
to_rename = {'RUBRIQUE_N2': "RUBRIQUE_N1"}
dataset_kpis_df = dataset_kpis_df.rename(columns=to_rename)

# Passage value positif
dataset_kpis_df["VALUE"] = dataset_kpis_df["VALUE"].abs()


# KPIs BFR = CREANCES + STOCKS - DETTES FOURNISSEURS
dataset_kpis_bfr_df = dataset_kpis_df.copy()

# Passage dette fournisseur value négatif
dataset_kpis_bfr_df["VALUE"] = dataset_kpis_bfr_df["VALUE"]*-1

dataset_kpis_bfr_df = pd.concat([dataset_kpis_cc, dataset_kpis_st, dataset_kpis_bfr_df], axis=0, sort=False)

to_group = ["ENTITY", "PERIOD"]
to_agg = {"VALUE": "sum"}
dataset_kpis_bfr_df = dataset_kpis_bfr_df.groupby(to_group, as_index=False).agg(to_agg)

# Creation colonne Rubrique_N1 = BFR
dataset_kpis_bfr_df["RUBRIQUE_N1"] = "BFR"

# Reorganisation colonne
dataset_kpis_bfr_df = dataset_kpis_bfr_df[["ENTITY", "PERIOD", "RUBRIQUE_N1", "VALUE"]]


# Creation du dataset final
dataset_kpis_final = pd.concat([dataset_kpis_ca, dataset_kpis_mb, dataset_kpis_ebe, dataset_kpis_cc, dataset_kpis_st, dataset_kpis_df, dataset_kpis_bfr_df], axis=0, sort=False)


# Creation colonne COMP
dataset_kpis_final['PERIOD_COMP'] = (dataset_kpis_final['PERIOD'].str[:4].astype(int) - 1).astype(str) + dataset_kpis_final['PERIOD'].str[-3:]
dataset_kpis_final
```


```python
# creation base comparable pour dataset_kpis
dataset_kpis_final_comp = dataset_kpis_final.copy()

# Suppression de la colonne période
dataset_kpis_final_comp = dataset_kpis_final_comp.drop("PERIOD_COMP", axis=1)

# Renommage des colonnes
to_rename = {'VALUE': "VALUE_N-1", 
             'PERIOD': "PERIOD_COMP"}
dataset_kpis_final_comp = dataset_kpis_final_comp.rename(columns=to_rename)
dataset_kpis_final_comp
```


```python
# Jointure entre les 2 tables dataset_kpis_final et dataset_kpis_vf
join_on = ["ENTITY",
           "PERIOD_COMP",
           "RUBRIQUE_N1"]
dataset_kpis_final = pd.merge(dataset_kpis_final, dataset_kpis_final_comp, how='left', on=join_on).drop("PERIOD_COMP", axis=1).fillna(0)

#Création colonne Var V
dataset_kpis_final["VARV"] = dataset_kpis_final["VALUE"] - dataset_kpis_final["VALUE_N-1"]

#Création colonne Var P (%)
dataset_kpis_final["VARP"] = dataset_kpis_final["VARV"] / dataset_kpis_final["VALUE_N-1"]

dataset_kpis_final
```

#### EVOLUTION CA


```python
# Creation du dataset evol_ca
dataset_evol_ca = db_enr.copy()

# Filtre COMPTE_NUM = Chiffre d'Affaire (RUBRIQUE N1)
dataset_evol_ca = dataset_evol_ca[dataset_evol_ca['COMPTE_NUM'].str.contains(r'^70|^71|^72')]

# Regroupement 
to_group = ["ENTITY",
            "PERIOD",
            "MONTH",
            "MONTH_INDEX",
            "RUBRIQUE_N3"]
to_agg = {"VALUE": "sum"}
dataset_evol_ca = dataset_evol_ca.groupby(to_group, as_index=False).agg(to_agg)

dataset_evol_ca["VALUE"] = dataset_evol_ca["VALUE"].abs()


# Calcul de la somme cumulée
dataset_evol_ca = dataset_evol_ca.sort_values(by=["ENTITY", 'PERIOD', 'MONTH_INDEX']).reset_index(drop=True)
dataset_evol_ca['MONTH_INDEX'] = pd.to_datetime(dataset_evol_ca['MONTH_INDEX'], format="%m").dt.strftime("%m")
dataset_evol_ca['VALUE_CUM'] = dataset_evol_ca.groupby(["ENTITY", "PERIOD"], as_index=True).agg({"VALUE": "cumsum"})

# Affichage du modèle de donnée
dataset_evol_ca
```

#### CHARGES


```python
#Creation du dataset charges
dataset_charges = db_cat.copy()

# Filtre RUBRIQUE_N0 = CHARGES
dataset_charges = dataset_charges[dataset_charges["RUBRIQUE_N0"] == "CHARGES"]

# Mettre en valeur positive VALUE
dataset_charges["VALUE"] = dataset_charges["VALUE"].abs()

# Affichage du modèle de donnée
dataset_charges
```

#### POSITIONS TRESORERIE


```python
# Creation du dataset trésorerie
dataset_treso = db_enr.copy()

# Filtre RUBRIQUE_N1 = TRESORERIE
dataset_treso = dataset_treso[dataset_treso['COMPTE_NUM'].str.contains(r'^5')].reset_index(drop=True)

# Cash in / Cash out ?
dataset_treso.loc[dataset_treso.VALUE > 0, "CASH_IN"] = dataset_treso.VALUE
dataset_treso.loc[dataset_treso.VALUE < 0, "CASH_OUT"] = dataset_treso.VALUE

# Regroupement 
to_group = ["ENTITY",
            "PERIOD",
            "MONTH",
            "MONTH_INDEX"]
to_agg = {"VALUE": "sum",
          "CASH_IN": "sum",
          "CASH_OUT": "sum"}
dataset_treso = dataset_treso.groupby(to_group, as_index = False).agg(to_agg).fillna(0)

# Cumul par période
dataset_treso = dataset_treso.sort_values(["ENTITY", "PERIOD", "MONTH_INDEX"])
dataset_treso['MONTH_INDEX'] = pd.to_datetime(dataset_treso['MONTH_INDEX'], format="%m").dt.strftime("%m")
dataset_treso['VALUE_LINE'] = dataset_treso.groupby(["ENTITY", 'PERIOD'], as_index=True).agg({"VALUE": "cumsum"})

# Mettre en valeur positive CASH_OUT
dataset_treso["CASH_OUT"] = dataset_treso["CASH_OUT"].abs()

# Affichage du modèle de donnée
dataset_treso
```

#### BILAN


```python
# Creation du dataset Bilan
dataset_bilan = db_cat.copy()

# Filtre RUBRIQUE_N0 = ACTIF & PASSIF
dataset_bilan = dataset_bilan[(dataset_bilan["RUBRIQUE_N0"].isin(["ACTIF", "PASSIF"]))]

# Regroupement R0/R1/R2
to_group = ["ENTITY",
            "PERIOD",
            "RUBRIQUE_N0",
            "RUBRIQUE_N1",
            "RUBRIQUE_N2"]
to_agg = {"VALUE": "sum"}
dataset_bilan = dataset_bilan.groupby(to_group, as_index = False).agg(to_agg).fillna(0)


# Mettre en valeur positive VALUE
dataset_bilan["VALUE"] = dataset_bilan["VALUE"].abs()

# Selectionner les colonnes
to_select = ["ENTITY",
             "PERIOD",
             "RUBRIQUE_N0",
             "RUBRIQUE_N1",
             "RUBRIQUE_N2",
             "VALUE"]
dataset_bilan = dataset_bilan[to_select]

# Affichage du modèle de donnée
dataset_bilan
```

## Output

### Sauvegarde des fichiers en csv


```python
def df_to_csv(df, filename):
    # Sauvegarde en csv
    df.to_csv(filename,
              sep=";",
              decimal=",",
              index=False)
    
    # Création du lien url
    naas_link = naas.asset.add(filename)
    
    # Création de la ligne
    data = {
        "OBJET": filename,
        "URL": naas_link,
        "DATE_EXTRACT": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }
    return pd.DataFrame([data])
```


```python
dataset_logo = {
    "OBJET": "Logo",
    "URL": LOGO,
    "DATE_EXTRACT": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
}
logo = pd.DataFrame([dataset_logo])
logo
```


```python
import json

color = {"name":"Color",
         "dataColors":[COLOR_1, COLOR_2]}

with open("color.json", "w") as write_file:
    json.dump(color, write_file)

dataset_color = {
    "OBJET": "Color",
    "URL": naas.asset.add("color.json"),
    "DATE_EXTRACT": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
}
pbi_color = pd.DataFrame([dataset_color])
pbi_color
```


```python
entite = df_to_csv(dataset_entite, "dataset_entite.csv")
entite
```


```python
scenario = df_to_csv(dataset_scenario, "dataset_scenario.csv")
scenario
```


```python
kpis = df_to_csv(dataset_kpis_final, "dataset_kpis_final.csv")
kpis
```


```python
evol_ca = df_to_csv(dataset_evol_ca, "dataset_evol_ca.csv")
evol_ca
```


```python
charges = df_to_csv(dataset_charges, "dataset_charges.csv")
charges
```


```python
treso = df_to_csv(dataset_treso, "dataset_treso.csv")
treso
```


```python
bilan = df_to_csv(dataset_bilan, "dataset_bilan.csv")
bilan
```

### Création du fichier à intégrer dans PowerBI


```python
db_powerbi = pd.concat([logo, pbi_color, entite, scenario, kpis, evol_ca, charges, treso, bilan], axis=0)
db_powerbi
```


```python
df_to_csv(db_powerbi, "powerbi.csv")
```
