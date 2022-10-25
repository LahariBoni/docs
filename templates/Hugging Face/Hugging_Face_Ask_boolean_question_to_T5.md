<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Hugging%20Face/Hugging_Face_Ask_boolean_question_to_T5.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Hugging+Face+-+Ask+boolean+question+to+T5:+Error+short+description">🚨 Bug report</a>

**Tags:** #huggingface #ml #sales #ai #text

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## T5-base finetuned on BoolQ (superglue task)
This notebook is for demonstrating the training and use of the text-to-text-transfer-transformer (better known as T5) on boolean questions (BoolQ). The example use case is a validator indicating if an idea is environmentally friendly. Nearly any question can be passed into the `query` function (see below) as long as a context to a question is given.

Author: Maximilian Frank ([script4all.com](//script4all.com)) - Copyleft license

Notes:
- The model from [huggingface.co/mrm8488/t5-base-finetuned-boolq](//huggingface.co/mrm8488/t5-base-finetuned-boolq) is used in this example as it is an already trained t5-base model on boolean questions (BoolQ task of superglue).
- Documentation references on [huggingface.co/transformers/model_doc/t5.html#training](//huggingface.co/transformers/model_doc/t5.html#training), template script on [programming-review.com/machine-learning/t5](//programming-review.com/machine-learning/t5)
- The greater the model, the higher the accuracy on BoolQ (see [arxiv.org/pdf/1910.10683.pdf](//arxiv.org/pdf/1910.10683.pdf)):
    t5-small|t5-base|t5-large|t5-3B|t5-11B
    -|-|-|-|-
    76.4%|81.4%|85.4%|89.9%|91.2%

## Loading the model
If here comes an error, install the packages via `python3 -m pip install … --user`.

You can also load a T5 plain model (not finetuned). Just replace the following code
```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
tokenizer = AutoTokenizer.from_pretrained('mrm8488/t5-base-finetuned-boolq')
model = AutoModelForSeq2SeqLM.from_pretrained('mrm8488/t5-base-finetuned-boolq')…
```
with
```python
from transformers import T5Tokenizer, T5ForConditionalGeneration
tokenizer = T5Tokenizer.from_pretrained('t5-small')
model = T5ForConditionalGeneration.from_pretrained('t5-small')
```
where `t5-small` is one of the names in the table above.

## Input

### Install packages


```python
!pip install transformers
!pip install sentencepiece
```

### Import libraries


```python
import json
import torch
from operator import itemgetter
from distutils.util import strtobool
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
```

### Load model


```python
tokenizer = AutoTokenizer.from_pretrained('mrm8488/t5-base-finetuned-boolq')
model = AutoModelForSeq2SeqLM.from_pretrained('mrm8488/t5-base-finetuned-boolq').to(torch.device('cuda' if torch.cuda.is_available() else 'cpu'))
try:model.parallelize()
except:pass
```

## Model

### Training
> **Optional:** You can leave the following out, if you don't have custom datasets. By default the number of training epochs equals 0, so nothing is trained.

> **Warning:** This option consumes a lot of runtime and thus *naas.ai* credits. Make sure to have enough credits on your account.

For each dataset a stream-opener has to be provided which is readable line by line (e.g. file, database). In the array with key `keys` are all dictionary keys which exist in the jsonl-line. So in this example the first training dataset has the keys `question` for the questions (string),`passage` for the contexts (string) and `answer` for the answers (boolean). Adjust these keys to your dataset.

At last you have to adjust the number of epochs to be trained (see comment `# epochs`).


```python
srcs = [
    { 'stream': lambda:open('boolq/train.jsonl', 'r'),
      'keys': ['question', 'passage', 'answer'] },
    { 'stream': lambda:open('boolq/dev.jsonl', 'r'),
      'keys': ['question', 'passage', 'answer'] },
    { 'stream': lambda:open('boolq-nat-perturb/train.jsonl', 'r'),
      'keys': ['question', 'passage', 'roberta_hard'] }
]
model.train()
for _ in range(0): # epochs
    for src in srcs:
        with src['stream']() as s:
            for d in s:
                q, p, a = itemgetter(src['keys'][0], src['keys'][1], src['keys'][2])(json.loads(d))
                tokens = tokenizer('question:'+q+'\ncontext:'+p, return_tensors='pt')
                if len(tokens.input_ids[0]) > model.config.n_positions:
                    continue
                model(input_ids=tokens.input_ids,
                    labels=tokenizer(str(a), return_tensors='pt').input_ids,
                    attention_mask=tokens.attention_mask,
                    use_cache=True
                    ).loss.backward()
model.eval(); # ; suppresses long output on jupyter
```

### Define query function
As the model is ready, define the querying function.


```python
def query(q='question', c='context'):
    return strtobool(
        tokenizer.decode(
            token_ids=model.generate(
                input_ids=tokenizer.encode('question:'+q+'\ncontext:'+c, return_tensors='pt')
            )[0],
        skip_special_tokens=True,
        max_length=3)
    )
```

## Output

### Querying on the task
Now the actual task begins: Query the model with your ideas (see list `ideas`).


```python
if __name__ == '__main__':
    ideas = [ 'The idea is to pollute the air instead of riding the bike.', # should be false
              'The idea is to go cycling instead of driving the car.', # should be true
              'The idea is to put your trash everywhere.', # should be false
              'The idea is to reduce transport distances.', # should be true
              'The idea is to put plants on all the roofs.', # should be true
              'The idea is to forbid opensource vaccines.', # should be true
              'The idea is to go buy an Iphone every five years.', # should be false 
              'The idea is to walk once every week in the nature.', # should be true  
              'The idea is to go buy Green bonds.', # should be true  
              'The idea is to go buy fast fashion.', # should be false
              'The idea is to buy single-use items.', # should be false
              'The idea is to drink plastic bottled water.', # should be false
              'The idea is to use import goods.', # should be false
              'The idea is to use buy more food than you need.', # should be false
              'The idea is to eat a lot of meat.', # should be false
              'The idea is to eat less meat.', # should be false
              'The idea is to always travel by plane.', # should be false
              'The idea is to opensource vaccines.' # should be false
             
            ]
    for idea in ideas:
        print('🌏 Idea:', idea)
        print('\t✅ Good idea' if query('Is the idea environmentally friendly?', idea) else '\t❌ Bad idea' )
```
