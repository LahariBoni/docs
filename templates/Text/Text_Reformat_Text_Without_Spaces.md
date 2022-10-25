<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Text/Text_Reformat_Text_Without_Spaces.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Text+-+Reformat++Without+Spaces:+Error+short+description">🚨 Bug report</a>

**Tags:** #text #reformat #snippet #operations #spaces

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

This notebook reformats text that does not contain spaces.

## Input

### Import libraries


```python
try:
    import wordninja
except:
    !pip install wordninja
    import wordninja
```

### Variables


```python
text = """WhenSatoshiNakamotofirstsettheBitcoinblockchainintomotioninJanuary2009,hewas simultaneouslyintroducingtworadicalanduntestedconcepts.Thefirstisthe"bitcoin",adecentralized peer-to-peeronlinecurrencythatmaintainsavaluewithoutanybacking,intrinsicvalueorcentralissuer.So far,the"bitcoin"asacurrencyunithastakenupthebulkofthepublicattention,bothintermsofthepolitical aspectsofacurrencywithoutacentralbankanditsextremeupwardanddownwardvolatilityinprice. However,thereisalsoanother,equallyimportant,parttoSatoshi'sgrandexperiment:theconceptofaproofof work-basedblockchaintoallowforpublicagreementontheorderoftransactions.Bitcoinasanapplicationcan bedescribedasafirst-to-filesystem:ifoneentityhas50BTC,andsimultaneouslysendsthesame50BTCto AandtoB,onlythetransactionthatgetsconfirmedfirstwillprocess.Thereisnointrinsicwayofdetermining fromtwotransactionswhichcameearlier,andfordecadesthisstymiedthedevelopmentofdecentralized digitalcurrency.Satoshi'sblockchainwasthefirstcredibledecentralizedsolution.Andnow,attentionis rapidlystartingtoshifttowardthissecondpartofBitcoin'stechnology,andhowtheblockchainconceptcanbe used for more than just money. Commonlycitedapplicationsincludeusingon-blockchaindigitalassetstorepresentcustomcurrenciesand financialinstruments("coloredcoins"),theownershipofanunderlyingphysicaldevice("smartproperty"), non-fungibleassetssuchasdomainnames("Namecoin")aswellasmoreadvancedapplicationssuchas decentralizedexchange,financialderivatives,peer-to-peergamblingandon-blockchainidentityand reputationsystems.Anotherimportantareaofinquiryis"smartcontracts"-systemswhichautomatically movedigitalassetsaccordingtoarbitrarypre-specifiedrules.Forexample,onemighthaveatreasurycontract oftheform"AcanwithdrawuptoXcurrencyunitsperday,BcanwithdrawuptoYperday,AandBtogether canwithdrawanything,andAcanshutoffB'sabilitytowithdraw".Thelogicalextensionofthisis decentralizedautonomousorganizations(DAOs)-long-termsmartcontractsthatcontaintheassetsand encodethebylawsofanentireorganization.WhatEthereumintendstoprovideisablockchainwithabuilt-in fullyfledgedTuring-completeprogramminglanguagethatcanbeusedtocreate"contracts"thatcanbeused toencodearbitrarystatetransitionfunctions,allowinguserstocreateanyofthesystemsdescribedabove,as well as many others that we have not yet imagined, simply by writing up the logic in a few lines of code."""
```

## Model

### Define function to reformat text


```python
def reformat_text(text):
    sentences = []
    
    for sentence in text.split('.'):
        if len(sentence) > 0:
            sentences.append(" ".join(wordninja.split(sentence)) + '.')
    
    return " ".join(sentences)
```

## Output

### Reformat text


```python
print(reformat_text(text))
```


```python

```
