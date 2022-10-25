<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_age_and_gender_from_profile_picture.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+age+and+gender+from+profile+picture:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #machinelearning #profile #identity #naas_drivers #content #snippet #dataframe

**Author:** [Nikolaj Groeneweg](https://www.linkedin.com/in/njgroene/)

This notebook estimates a person's age and gender based on their LinkedIn profile picture. 

## Prerequisites

This code requires you to download and install a pretrained neural network for it to function correctly. 

Follow these steps:

   1) Create new folder called "models" in the same Naas directory as this noteboook
   
   2) Download the model "res34_fair_align_multi_7_20190809.pt" from the following url
       https://drive.google.com/drive/folders/1F_pXfbzWvG-bhCpNsRj6F_xsdjpesiFu
   
   3) Copy the model to your "models" folder

## References

The code below is based on the following work :
    
*Karkkainen, K., & Joo, J. (2021).*
FairFace: Face Attribute Dataset for Balanced Race, Gender, and Age for Bias Measurement and Mitigation. 
In Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision (pp. 1548-1558).

The paper and all other corresponding information can be found on github:
https://github.com/joojs/fairface

*Disclaimer*: please note that model output and interpretation have been restricted in order to comply to French CNIL regulation regarding the processing and storage of sensitive data (see : https://www.cnil.fr/fr/definition/donnee-sensible)

## Input

### Import libraries


```python
from naas_drivers import linkedin
import urllib 
import naas

from PIL import Image
import pandas as pd
import numpy as np

import torch
import torch.nn as nn
try:
    import torchvision
    from torchvision import datasets, models, transforms
except:
    !pip install torchvision==0.12.0 --user --no-cache-dir
    import torchvision
    from torchvision import datasets, models, transforms
try:
    from torch_mtcnn import detect_faces
    from torch_mtcnn import show_bboxes
except:
    !pip install torch-mtcnn --user
    from torch_mtcnn import detect_faces
    from torch_mtcnn import show_bboxes
```

### Get your cookies

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = "ENTER_YOUR_COOKIE_HERE" # EXAMPLE : "AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2"
JSESSIONID = "ENTER_YOUR_JSESSIONID_HERE" # EXAMPLE : "ajax:8379907400220387585"

# LinkedIn profile url
PROFILE_URL = "ENTER_YOUR_LINKEDIN_PROFILE_HERE" # EXAMPLE "https://www.linkedin.com/in/myprofile/"
```

## Model

### Get image from URL


```python
def get_image_from_url(imgurl):
    urllib.request.urlretrieve(imgurl, "tmp_file")
    return Image.open("tmp_file")
```

### Get data from image


```python
def get_demographics_from_image(img):
    try: 
        bounding_boxes, landmarks = detect_faces(img)
    except Exception as e:
        raise Exception("Image parse error [" + str(e) + "].")
    if len(bounding_boxes) == 0:
        raise Exception("No face in image.")
    if len(bounding_boxes) > 1:
        raise Exception("Multiples faces in image.")
        
    bb = [bounding_boxes[0,0], bounding_boxes[0,1], bounding_boxes[0,2], bounding_boxes[0,3]]
    img_cropped = img.crop(bb)

    device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")

    model_fair_7 = torchvision.models.resnet34(pretrained=True)
    model_fair_7.fc = nn.Linear(model_fair_7.fc.in_features, 18)
    model_fair_7.load_state_dict(torch.load('models/res34_fair_align_multi_7_20190809.pt', map_location=torch.device('cpu')))
    model_fair_7 = model_fair_7.to(device)
    model_fair_7.eval()

    trans = transforms.Compose([
            transforms.Resize((224, 224)),
            transforms.ToTensor(),
            transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
        ])

    face_names = []
    gender_scores_fair = []
    age_scores_fair = []
    gender_preds_fair = []
    age_preds_fair = []

    image = trans(img_cropped)
    image = image.view(1, 3, 224, 224)  # reshape image to match model dimensions (1 batch size)
    image = image.to(device)

    # fair 7 class
    outputs = model_fair_7(image)
    outputs = outputs.cpu().detach().numpy()
    outputs = np.squeeze(outputs)

    gender_outputs = outputs[7:9]
    age_outputs = outputs[9:18]

    gender_score = np.exp(gender_outputs) / np.sum(np.exp(gender_outputs))
    age_score = np.exp(age_outputs) / np.sum(np.exp(age_outputs))

    gender_pred = np.argmax(gender_score)
    age_pred = np.argmax(age_score)

    gender_scores_fair.append(gender_score)
    age_scores_fair.append(age_score)

    gender_preds_fair.append(gender_pred)
    age_preds_fair.append(age_pred)

    result = pd.DataFrame([gender_preds_fair,
                           age_preds_fair]).T

    result.columns = ['gender_preds_fair',
                      'age_preds_fair']
    # gender
    result.loc[result['gender_preds_fair'] == 0, 'gender'] = 'Male'
    result.loc[result['gender_preds_fair'] == 1, 'gender'] = 'Female'

    # age
    result.loc[result['age_preds_fair'] == 0, 'age'] = '0-2'
    result.loc[result['age_preds_fair'] == 1, 'age'] = '3-9'
    result.loc[result['age_preds_fair'] == 2, 'age'] = '10-19'
    result.loc[result['age_preds_fair'] == 3, 'age'] = '20-29'
    result.loc[result['age_preds_fair'] == 4, 'age'] = '30-39'
    result.loc[result['age_preds_fair'] == 5, 'age'] = '40-49'
    result.loc[result['age_preds_fair'] == 6, 'age'] = '50-59'
    result.loc[result['age_preds_fair'] == 7, 'age'] = '60-69'
    result.loc[result['age_preds_fair'] == 8, 'age'] = '70+'

    return [result['gender'][0],result['age'][0]]
```

### Get LinkedIn profile


```python
df = linkedin.connect(LI_AT, JSESSIONID).profile.get_identity(PROFILE_URL)
df.insert(loc=2, column='GENDER', value=None)
df.insert(loc=3, column='AGE', value=None)
```

### Enrich profile with estimates of gender and age range


```python
imgurl = df["PROFILE_PICTURE"][0]
if imgurl != None:
    img = get_image_from_url(imgurl)
    try :
        result = get_demographics_from_image(img)
        df['GENDER'] = result[0]
        df['AGE'] = result[1]
    except Exception as e:
        print("\nCould not perform estimation.\nThe following input error occured : " + str(e)+"\n")
```

## Output

### Display result


```python
df
```


```python

```
