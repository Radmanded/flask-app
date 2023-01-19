# flask-app web front-end with nornir running in the background. When you enter the url the nornir script runs

Install flask, nornir, 
python3 -m pip install scrapli[genie]
python3 -m pip install rich
Get flask bootstrap - Acts as a CDN that uses templates
>Must pip install all the libraries for this to work!

## Import Flask in the python script

```example.py

from flask import Flask, render_template
from nornir import InitNornir
from nornir_scrapli.task import send_command

app = Flask(__name__)

@app.route("/")    # The plan is to map these routes to nornir specific code
def homepage_test():
  return render_template("base.html")  # Name of the template created below
  
@app.route("/newpage.html")
def homepage_test():
  return render_template("newpage.html")
  
# Sharing the different iterations of this route
#1
#@app.route("/hosts/inventory")  # This is the route that will allow nornir 
#def get_all_inventory():
#  nr = InitNornir(config_file="config.yaml")
#  return render_template("inventory.html")
#2
@app.route("/hosts/inventory")  # This is the route that will allow nornir 
def get_all_inventory():
  nr = InitNornir(config_file="config.yaml")
  return render_template("inventory.html", my_list=nr.inventory.hosts.values())  # Added my_list which is a variable you can access from the inventory.html template

@app.route("/all/running")    # Getting config for all devices - MUST go to `run1.py` to run the function
def get_all_running():
  nr = InitNornir(config_file="config.yaml")
  results = nr.run(task=send_command, command="show run")    # Logic behind running the show run to get all device data
  my_list = [v.scrapli_response.result for v in results.value()]
  return render_template("running.html", my_list=my_list)    # The my_list variable is listed in the line above and here
  
@app.route("/all/version")    # Getting version for all devices
def get_all_version():
  nr = InitNornir(config_file="config.yaml")    # This line creates a varible for the InitNornir(config_file="config.yaml")
  results = nr.run(task=send_command, command="show version") 
  my_list = [v.scrapli_response.genie_parse_output() for v in results.values()]  # same logic built in the python script using genie
  return render_template("version.html", my_list=my_list)
  
@app.route("/greeting")
def say_hello():
  return "Hello There!"
  
if __name__ == "__main__":
  app.run(debug=True)
  
```

## Create a `mkdir templates` and create `base.html` file

```template/base.html

<!DOCTYPE • html>
<html>
<head>
    ‹title>Automate All The Things</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-GLhlTQ8iRABdZLl6O3oVMWSktQOp6b7In1Zl3/Jr59b6EGGoI1aFkw7cmDA6j6gD" crossorigin="anonymous">. \\ Adding the CSS bootstrap under the title
</head>
<body>
    <h1>IPvZero Network Automation</h1>
    {% block content %}  # Adding Jinja logic so you don't have to add titles to all pages
    {% endblock content %} # Also, called inheritance 
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js" integrity="sha384-w76AqPfDkMBDXo30jS1Sgez6pr3x5MlQ1ZAGC+nuZB+EYdgRZgiwxhTBTkF7CXvN" crossorigin="anonymous"></script>  // Enter Javascript bootstrap to the bottom
</body>
</html>

```

## Create a `newpage.html` that will inherit from the base.html jinja2 logic

```template/newpage.html
{% extends "base.html" %}
{% block content %}
This is a brand new page
{% endblock content %}
```
  

## Create a `inventory.html` that will inherit from the base.html jinja2 logic

```template/inventory.html
{% extends "base.html" %}
{% block content %}
<h3>Device List</h3>
{% for data in my_list %}    # Accessing the my_list variable from the flask script
<div class="alert alert-info" role="alert">   # Adding a bootstrap div class to add alerts 
<pre>    # Helps to get better formating 
DEVICE: {{ data }}
IP ADDRESS: {{ data.hostname }}
DEVICE PLATFORM: {{ data.platform }}
</pre>   # Close the formating
</div>   # Close the bootstrap alert
{% endfor %}
{% endblock content %}
```

## Get the running config for all devices

```template/running.html
{% extends "base.html" %}
{% block content %}
<h4>Running Configurations of all Devices:</h4>
{% for data in my_list %}   # Links to the my_list variable inside the flask and python script and runs the send_command, show run
<div class="alert alert-dark" role="alert">
<pre>   # Use the pre tag to format the data
{{ data }}
</pre>
</div>
{% endfor %}

{% endblock content %}
```

## Get the running config for all devices

```template/version.html
{% extends "base.html" %}
{% block content %}
<h4>Version information for all Devices:</h4>
{% for data in my_list %}   # Links to the my_list variable inside the flask and python script and runs the send_command, show version
{% if "hour" in data.version.uptime %}
<div class="alert alert-success" role="alert">
<pre>   # Use the pre tag to format the data
  DEVICE: {{ data.version.hostname }}
  VERSION: {{ data.version.version_short }}
  UPTIME: {{ data.platform.uptime }}
  SERIAL NUMBER: {{ data.version.chassis_sn  }}
</pre>
</div>

{% else %}
<div class="alert alert-danger" role="alert">
  <pre>
    DEVICE: {{ data.version.hostname }}
    VERSION: {{ data.version.version_short }}
    UPTIME: {{ data.platform.uptime }}
    SERIAL NUMBER: {{ data.version.chassis_sn  }}
  </pre>   # Use the pre tag to format the data
  </div>
{% endif %}
{% endfor %}
{% endblock content %}
```


## Create a new file `run1.py` that will populate the inventory 
```run1.py
from nornir import InitNornir

nr = InitNornir(config_file-"config.yaml")
  for data in nr.inventory.hosts.values():  # This allows you to access everything within your inventory
  print (data)
  print (data.hostname)
  print (data.platform)
  
```
- Test the script `python3 run1.py` and you should see device hostnames, IP, and platform(ios)


## Create a new file `run2.py` that will get running configs for all devices
```run2.py

from nornir, import InitNornir
from nornir_scrapli.tasks import send_command 
from nornir_utils.plugins.functions import print_result

nr = InitNornir(config_file="config.yaml")

results = nr.run(task=send_command, command="show run")
my_list = [v.scrapli_response.result for v in results.values()]
print(my_list)   # k = dictionary keys, v = value, and scrapli_response.results gives you all the results

print_result(results)
print(results)   # Returns aggregated results, not the whole "show run" results

##########   Alternative method      ############ 

## Printing it in a list like format

from nornir, import InitNornir
from nornir_scrapli.tasks import send_command 
from nornir_utils.plugins.functions import print_result

nr = InitNornir(config_file="config.yaml")

results = nr.run(task=send_command, command="show run")
my_list = []
for v in results.values():
  my_list.append(v.scrapli_response.result)
  
print(my_list)
  
```


## Create a new file `run2.py` that will get running configs for all devices

```
from nornir, import InitNornir
from nornir_scrapli.tasks import send_command 
from nornir_utils.plugins.functions import print_result
from rich import print as rprint

nr = InitNornir(config_file="config.yaml")

results = nr.run(task=send_command, command="show version")
my_list = [v.scrapli_response.genie_parse_output() for v in results.values()]   # Copy and paste this line in the Flask app. Notice we are now using genie_parse_output
for data in my_list:
  rprint(data["version"])         # You can specify the data you want using the dictionary  ex. rprint(data["version"]["uptime"])


```
