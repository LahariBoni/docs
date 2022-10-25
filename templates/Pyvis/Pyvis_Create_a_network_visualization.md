<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pyvis/Pyvis_Create_a_network_visualization.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pyvis+-+Create+a+network+visualization:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #naas #scheduler #network #snippet #analytics

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/jeremyravenel/)

**Description:** With this notebook, you can create a network graph to visualize the relations between different elements.

**References:** https://pyvis.readthedocs.io/en/latest/tutorial.html

## Input

### Import packages


```python
pip install pyvis
```

### Import library


```python
import naas
from pyvis.network import Network
```

### Setup the global variables for the network


```python
network_name = "pyvis_example"
net = Network(notebook=True, height='750px', width='100%', bgcolor='#222222', font_color='white')
```

### Create master nodes with images and specific properties


```python
# Create master nodes with images and specific properties
net.add_node(3, shape='image', image ="https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/LinkedIn_logo_initials.png/768px-LinkedIn_logo_initials.png")
net.add_node(4, shape='image', image ="https://www.stemmarine.it/wp-content/uploads/2018/08/youtube-logo.png", color="#FF0000")
net.add_node(5, shape='image', image ="https://pnggrid.com/wp-content/uploads/2021/07/Twitter-Logo-Square.png", color="#1DA1F2")
```

### Create other simple nodes


```python

net.add_nodes(
    [1, 2, 6, 7, 8, 9, 10, 11, 12, 13, 14],
    label=["Setup", "Common", "Get post", "Get followers","Get views","Get post", "Get followers","Get views","Get post", "Get followers","Get views"],
    color=["#26cc67", "black","#0072b1","#0072b1", "#0072b1","#1DA1F2", "#1DA1F2", "#1DA1F2","#FF0000","#FF0000","#FF0000",],
)
```

## Model

### Create simple edge


```python
net.add_edge(1, 5)
```

### Create multiple edges


```python
net.add_edges([(1,2),(1, 3), (1, 4), (3,2), (4,2), (5,2), 
               (3,6), (3,7), (3,8), (6,2), (7,2), (8,2), 
              (5,9), (5,10), (5,11),(9,2), (10,2), (11,2),
               (4,12), (4,13), (4,14),(14,2), (13,2), (12,2)])
```

## Output

### Show results


```python
network = net.show(f"{network_name}.html")
network
```

### Share your output


```python
naas.asset.add(f"{network_name}.html",{"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete()
```
