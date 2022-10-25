<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Qonto/Qonto_Releve_de_compte_augmente.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Qonto+-+Releve+de+compte+augmente:+Error+short+description">🚨 Bug report</a>

**Tags:** #qonto #bank #statement #naas_drivers #notification #emailbuilder #asset #scheduler #naas #finance #automation #analytics #plotly #email #html #image #excel

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

Recevez un relevé de compte augmenté par email gratuitement, chaque semaine, grâce à un template Naas.ai (moteur de données opensource, 100 crédits offert par mois). 
<br><br>
-Durée de l'installation = 5 minutes<br>
-Support d'installation = Guide vidéo<br>
-Niveau = Facile<br>

## Input
Dans cette section, vous trouverez les informations à configurer pour que ce notebook puisse accéder à vos données via l'API Qonto.

### Import des librairies
Les libraries sont des outils créé dans le language Python qui permettent le fonctionnement du notebook. Aucune action nécessaire. 


```python
from naas_drivers import qonto
from datetime import datetime, timedelta
import pandas as pd
import naas
```

### Configuration des accès API
👇 Veuillez saisir ci-dessous, entre les guillemets, votre identifiant et votre clé secrète récupérés sur la plateforme Qonto.
<a href='https://www.notion.so/naas-official/Qonto-driver-Get-your-credentials-0cc97828b4e7467c8bfbcf704a77e5f4'>Comment récupérer ces accès API ?</a>


```python
QONTO_USER_ID = 'YOUR_USER_ID'
QONTO_SECRET_KEY = 'YOUR_SECRET_KEY'
```


```python
import naas
QONTO_USER_ID = naas.secret.get("QONTO_USER_ID")
QONTO_SECRET_KEY = naas.secret.get("QONTO_API_KEY")
```

### Configuration de l'email
👇 Veuillez saisir ci-dessous, entre les guillemets, le destinataire de l'email (si vous avez plusieurs destinataires, separez d'une virgule les emails en conservants les guillemets)<br>
Vous pouvez aussi changer l'objet de l'email (configuration avancée)


```python
# Destinataire
EMAIL_TO = ["florent.ravenel1@gmail.com"]

# Objet de l'email
EMAIL_SUBJECT = f"🏛️ Qonto - Votre relevé de compte augmenté du {datetime.now().strftime('%d/%m/%Y')}"
```

### Configuration de la période de l'analyse
👇 Veuillez saisir ci-dessous, entre les guillemets, la date de début (et la date de fin de votre analyse.


```python
# Date de début au format AAAA-MM-JJ
DATE_FROM = "2021-01-01"

# Date de fin au format AAAA-MM-JJ (par defaut, c'est la date d'aujoud'hui qui est selectionnée)
DATE_TO = datetime.now().strftime("%Y-%m-%d")

# Nombre de jours de récupération des dernières transactions (doit être un chiffre négatif)
LAST_TRANSACTIONS = -7
```

### Configuration de l'automatisation
Grâce à la formule ci-dessous, le notebook se lancera tous les lundis à 8h.<br>
Si vous souhaitez modifier la fréquence d'envoi, vous devez modifier la synthaxe entre guillemets en  <a href="https://crontab.guru/">suivant la documentation officielle CRON</a> (standard internationnal pour la programmation de tâches automatisées)


```python
naas.scheduler.add(cron="0 8 * * 1")
```

**Configuration des noms de fichiers (avancé)**


```python
GRAPH_FILE = "graph_account_statement.html"
GRAPH_IMG = "graph_account_statement.jpeg"
TABLE_FILE = "account_statement.xlsx"
```

## Model

### Récupération du relevé de compte consolidé


```python
# Colonne to consolidate (DATE already included), if empty return only DATE, AMOUNT, POSITION
to_group = ["TRANSACTION_ID",
            "LABEL",
            "OPERATION_TYPE"]

df_statement = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.get(to_group=to_group,
                                                                             date_from=DATE_FROM,
                                                                             date_to=DATE_TO)
df_statement
```

### Création du graphique "Evolution de la Trésorerie"


```python
barline = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.barline(date_from=DATE_FROM,
                                                                            date_to=DATE_TO)
barline
```

### Récupération des opérations par type


```python
cash_summary = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.summary(summary_type="OPERATION_TYPE",
                                                                                 language="FR",
                                                                                 date_from=DATE_FROM,
                                                                                 date_to=DATE_TO)
cash_summary
```

### Récupération des dernières transactions


```python
df_last = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.transactions(date_from=LAST_TRANSACTIONS,
                                                                                 date_to=DATE_TO)
df_last
```

### Calcule du solde courant


```python
current_position = round(df_statement['POSITION'].tolist()[-1], 2)
current_position
```

## Output

### Sauvegarde du relevé de compte et partage du fichier Excel


```python
df_statement.to_excel(TABLE_FILE)
statement_link = naas.asset.add(TABLE_FILE)
```

### Sauvegarde et partage du graphique en tant qu'image et page html


```python
# Image
barline.write_image(GRAPH_IMG)
graph_img = naas.asset.add(GRAPH_IMG)

# HTML
barline.write_html(GRAPH_FILE)
params = {"inline": True}
graph_link = naas.asset.add(GRAPH_FILE, params=params)
```

### Creation de l'email


```python
email_content = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.email(DATE_FROM,
                                                                                DATE_TO,
                                                                                current_position,
                                                                                graph_img,
                                                                                graph_link,
                                                                                cash_summary,
                                                                                LAST_TRANSACTIONS,
                                                                                df_last,
                                                                                statement_link)
```

### Envoi de l'email


```python
naas.notification.send(EMAIL_TO,
                       EMAIL_SUBJECT,
                       email_content)
```
