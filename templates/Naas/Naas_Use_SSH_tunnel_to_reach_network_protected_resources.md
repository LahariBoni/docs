<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Use_SSH_tunnel_to_reach_network_protected_resources.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Use+SSH+tunnel+to+reach+network+protected+resources:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #ssh #snippet

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

The goal of this notebook is to show you how to create an SSH tunnel in Python and use it to access servers/databases/etc that are not directly publicly accessible.

I am assuming that:
- The ip address of the SSH server is: **1.2.3.4**
- You want to reach a Postgres server on port **5432**
- Postgres private ip address is **10.0.0.2**

## Input

### Install and import libraries


```python
try:
    import sshtunnel
except:
    ! pip install --user sshtunnel paramiko
    import sshtunnel

try:
    import psycopg2
except:
    ! pip install --user psycopg2-binary
    import psycopg2
```

## Model

### Configure an SSH tunnel


```python
tunnel = sshtunnel.open_tunnel(
    ("1.2.3.4", 22),             # The IP address of the SSH server you want to connect to.
    ssh_username="user",         # SSH username
    ssh_pkey="path/to/ssh_key",  # SSH key to use
    remote_bind_addresses=[      # List of remote servers. Lets say the database you
        ('10.0.0.2', 5432)
    ]
)
```

### Open the tunnel


```python
tunnel.start()
tunnel.tunnel_is_up()
```

Example of output:

```
{('0.0.0.0', 42359)}
```

The following output is telling us that we now have an SSH tunnel that we can connect to on `127.0.0.1:42359` this will send us to `10.0.0.2:5432` which is our Postgres instance.



### Store the open port in a variable


```python
local_listening_tunnels = list(tunnel.tunnel_is_up())
local_port = local_listening_tunnels[0][1] # This is to get the local port.
```

### Connect to Postgres


```python
conn = psycopg2.connect(
    dbname="myDatabase",
    user="myUser",
    password="myPassword",
    host="127.0.0.1", # We are going through the tunnel here,
    port=local_port
)

# You can know use your connection
```

### Close SSH Tunnel

It's important to close your tunnel (even though it will be done automatically when the Kernel is killed)


```python
tunnel.close()
```

## Output

As this notebook is more a how to we do not have outputs

🎉 You now know how to create an SSH tunnel on naas.

The fact that we are using `tunnel.tunnel_is_up()` will allow you to have multiple notebooks running on production, opening tunnels to the same SSH server / databases without interferring with each other because the local ports will be random.

If you still need help with this notebook, contact me on [LinkedIn](https://www.linkedin.com/in/maximejublou).
