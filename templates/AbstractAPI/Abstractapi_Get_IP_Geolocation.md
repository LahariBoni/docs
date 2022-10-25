<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/AbstractAPI/Abstractapi_Get_IP_Geolocation.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=AbstractAPI+-+Abstractapi+Get+IP+Geolocation:+Error+short+description">🚨 Bug report</a>

**Tags:** #api #abstract-api #ip #geolocation #stream #multithread #queues #operations #dataprocessing #automation

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

This notebook enables you to perform efficient queries on Abstract API using streaming techniques that temporarily store results and then transform them into a DataFrame at the end of the process.

## Input

### Import libraries


```python
import threading
import queue
import time
import requests
import pandas as pd
import json
try:
    from ratelimiter import RateLimiter
except:
    ! pip install --user ratelimiter
    from ratelimiter import RateLimiter
```

### Setup Variables


```python
API_KEY = "..."
CALL_PER_SECOND = 500

# Define number of workers.
# You will learn by running this notebook the right numbers for your machine.
MAX_WORKERS = 50
MIN_WORKERS = 45

# File where the results from the api will be streamed.
RESULT_TMP_FILE = "/tmp/results.txt"

# File where we will store the resulting CSV file.
CSV_FILE = "results.csv"
```

### Create list of IP addresses

You can replace this code to load your own dataframe.
Only requirements are:
- The dataframe variable must be named `df`
- The dataframe should have an `ip` columns containing ip addresses.


```python
# Generate a pandas dataframe containing ip addresses
ip_addresses = []

for x in range(1, 41):
    for y in range(1, 251):
        ip_addresses.append({'ip': f'77.205.{x}.{y}'})
        
df = pd.DataFrame(ip_addresses)
df
```

## Model

### Create queues
The goal is to stream events (here IP addresses), instead of batching all of them in one process.


```python
# This queue will contain the list of IP addresses.
job_queue = queue.Queue()

# This queue will contain the results from the workers.
results_queue = queue.Queue()

# This queue will be used to count the number of redrived jobs. (Jobs on error that are pushed back on job_queue for new processing)
redrive_queue = queue.Queue()

# This queue will contain ended jobs (Either successful or pushed in dead_letter_queue)
ended_queue = queue.Queue()

# This queue will be used to send failing job to it, that way we will be able to look at it to see if we have any errors.
dead_letter_queue = queue.Queue()
```

### Set timer

This is the cadence at which we will call the Abstract API.


```python
timer = (1000 / CALL_PER_SECOND) / 1000
timer
```

### Create job worker function

This function will be the worker taking jobs and calling the api.


```python
@RateLimiter(max_calls=1, period=timer)
def limit_me():
    pass

# We pre compute the url to only append the ip in the worker to not "format" the string everytime, doing this is 10 time faster.
pre_computed_url = "https://ipgeolocation.abstractapi.com/v1/?api_key={API_KEY}&ip_address=".format(API_KEY = API_KEY)

def job_worker():
    # Initialize requests session.
    s = requests.Session()
    
    while True:
        # Wait for a job to be availabe.
        item = job_queue.get()
        
        # Limit
        limit_me()
        
        try:
            # Query the api with our existing session.
            response = s.get(pre_computed_url + item)
            
            # Raise for status if we have an error.
            response.raise_for_status()
            
            # Send the result to the results_queue.
            results_queue.put(response.text)
        except Exception as e:
            if response.status_code == 429:
                job_queue.put(item)
                redrive_queue.put(1)
            else:
                print(e)
                dead_letter_queue.put({
                    'item': item,
                    'error': e
                })
                ended_queue.put(1)
        job_queue.task_done()
        

workers = []
        
def start_a_new_worker():
    print('🚀 Starting a new worker')
    workers.append(threading.Thread(target=job_worker, daemon=True).start())

# Start minimal workers count.
for i in range(0, MIN_WORKERS):
    start_a_new_worker()
```

### Create result worker function

This function is the code that will be run by our result worker function.
Its role is to take the results from all workers and store them.

**Note**: If we want to get a lot of jobs done then we will need to offload results to a file instead of keeping everything in memory. We could stream json representations to a file then read it later.


```python
def result_worker():
    f = open(RESULT_TMP_FILE, "w", encoding="utf-8")
    previous_time = None
    while True:
        try:
            result = results_queue.get(block=True, timeout=10)
            f.write(result + '\n')
            results_queue.task_done()
            ended_queue.put(1)
        # We should except _queue.empty here instead of catching a generic exception.
        # It's fine for now.2
        except Exception as e:
            print('Leaving result_worker')
            break
    f.close()

# Start the result worker
threading.Thread(target=result_worker, daemon=True).start()
```

### Create queue monitor function

This function is here to provide live progress during the execution.
It can also start new workers if we are not reaching the `CALL_PER_SECOND` target.


```python
@RateLimiter(max_calls=1, period=1)
def monitor_limiter():
    pass

def queue_monitor():
    prev_queue_size = 0
    while True:
        ended_queue_size = ended_queue.qsize()
        jobs_per_seq = ended_queue_size - prev_queue_size
        
        workers_number = len(workers)
        if jobs_per_seq < CALL_PER_SECOND and workers_number < MAX_WORKERS:
            start_a_new_worker()
            
        print(f"#jobqsize[{job_queue.qsize()}] #resultsqsize[{results_queue.qsize()}] #redriveqsize[{redrive_queue.qsize()}] #deadletterqsize[{dead_letter_queue.qsize()}] | ✅ {ended_queue.qsize()}/{nbr_of_jobs} jobs completed. ⚡ {jobs_per_seq}/sec. 👷‍♂️ Running workers: {workers_number}")
        
        prev_queue_size = ended_queue_size
        
        if ended_queue_size >= nbr_of_jobs:
            print('Jobs completed. Shuting down monitor.')
            return
        
        monitor_limiter()
```

### Run API calls


```python
# Pushing all jobs in the queue.
nbr_of_jobs = 0
for i in df.ip:
    job_queue.put(i)
    nbr_of_jobs += 1

# Start monitor function
threading.Thread(target=queue_monitor, daemon=True).start()

# Block until all tasks are done.
job_queue.join()

# Making sure we have handled all the jobs.
while True:
    if nbr_of_jobs == ended_queue.qsize():
        break
    print(nbr_of_jobs, ended_queue.qsize())
    time.sleep(0.1)

print('🎉 All API calls has been completed!')
```

## Output

Load all results from the `RESULT_TMP_FILE` and load them in a Pandas DataFrame.


```python
results = []

# Open file
with open(RESULT_TMP_FILE, 'r') as results_file:
    # Read line by line
    for line in results_file:
        # Load json into results list
        results.append(json.loads(line))

# Create DataFrame from results
results_df = pd.DataFrame(results)

# Display DataFrame
results_df
```

### Store the results as a CSV file


```python
results_df.to_csv(CSV_FILE)
```
