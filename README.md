# Datadog Config

Persist your DataDog configuration in versioned text files which can be edited locally. Pull down existing config and push changes at will.


## CF Networking Tips and Tricks for Datadog

### Main Screenboards

[Service Discovery PWS Screenboard](https://is.gd/cfnetworkingdatadogsd)

[Networking PWS Screenboard](https://is.gd/cfnetworkingdatadog)

Our environment screenboards can be found by logging into datadog
with either your personal account or cf-container-networking@pivotal.io.

### Items in this repo
In this repository are three main datadog concepts: alerts, screenboards, and dashboards.

Alerts are used to let the team know when a metric is behaving unexpectedly through email.
The alerts are found in the `alert_templates` folder.

Dashboards (also known as timeboards) are boards where all metrics are scoped to the same
time-scale. We do not use dashboards as often as screenboards. However, our dashboards
can be found in the `dashboard_templates` folder.

Screenboards are boards where metrics can be configured to use different time frames. We
prefer to use screenboards and would recommend doing so for new boards. The screenboards
can be found in the `screenboard_templates` folder.

### How this repo works with rake / repo organization
Our scripts for applying our templates to datadog can be found in the `scripts` folder.
The scripts take care of setting up the proper config and location for the templates.

Note that the way the rake tasks work is that they will push any template in the `shared`
folder. This is why our scripts move relevant templates in and out of the shared folder
depending on the environment.

The rake tasks for the environments are created through meta-programming. You should
never need to add a new rake task.

### Making a new dashboard / adding widgets

We have not been able to find any docs on the json templating that datadog uses. We recommend following these steps: 

#### New dashboard

1. Create the new dashboard manually on the datadog website
1. Follow the steps below for importing a datadog template
1. Replace any relavent values with variables
1. Add those variables to the config you are using
1. Delete the manually created dashboard
1. Push the new dashboard using the template and check that it works

#### New widget

1. Create the widget manually on the datadog website
1. Follow the steps below for important a datadog template, but save to a temp file
1. Copy the json for the new widget into the existing dashboard template
1. Delete the manually created widget
1. Push the updated dashboard and see that it works

### Links to config repos

[CI Environment Configs](https://github.com/cloudfoundry/cf-networking-deployments/tree/master/datadog)

[PWS Environment Config](https://github.com/pivotal-cf/pcf-networking-pws/tree/master/datadog)

### Metrics explorer is your friend
If you are ever wondering what a metric is called, or what tags are on the metric, go to Metrics -> Explorer
on the datadog website to find out. No need to create a new graph for it :)


## Docs from original repo

### Updating to version 2.x
[ ]  Add a key to your environment name called 'tags'  
[ ]  Populate an array of tags in this key.   

```
mydeployment:
  tags:
  - aws
  - p-mysql
```

## Usage

### Setup

    $ cp config/config-example.yml config/config.yml
    # Fill in config.yml with your credentials and other details
    $ bundle install

**CAUTION:** This takes a while and will push to everything.

    $ rake <your-env-name>:push

### Rake commands available

```
rake cf_deployment:console                               # A console with some useful variables
rake cf_deployment:delete_unknown                        # Delete dashboards and alerts that are not represented in local templates for Cf_deployment
rake cf_deployment:emit[metric]                          # emit data to datadog for testing purposes (note this WILL affect graphs and alerts)
rake cf_deployment:eval_alert[path]                      # Evaluate the alert at path under the Cf_deployment config and print to stdout
rake cf_deployment:eval_dashboard[path]                  # Evaluate the dashboard at path under the Cf_deployment config and print to stdout
rake cf_deployment:eval_screen[path]                     # Evaluate the screen at path under the Cf_deployment config and print to stdout
rake cf_deployment:get_alert_json_erb[alert_id,path]     # Make a json template for the specified alert at the given file path
rake cf_deployment:get_dashboard_json_erb[dash_id,path]  # Make a json template for the specified dashboard at the given file path
rake cf_deployment:get_screen_json_erb[screen_id,path]   # Make a json template for the specified screen at the given file path
rake cf_deployment:list_unknown                          # List dashboards and alerts that are not represented in local templates for Cf_deployment
rake cf_deployment:push                                  # Push Cf_deployment Datadog Config
rake diego:cf_deployment:push                            # Push cf_deployment Datadog Config
rake diego:push                                          # Push all Diego Datadog Configs
rake garden:cf_deployment:push                           # Push cf_deployment Datadog Config
rake garden:push                                         # Push all Garden Datadog Configs
rake garden_blackbox:cf_deployment:push                  # Push cf_deployment Datadog Config
rake garden_blackbox:push                                # Push all Garden Datadog Configs
rake spec                                                # Run RSpec code examples
```

Note: `cf_deployment`, as used above, is a placeholder for a deployment named in your `config.yml`, such as `prod`.

## Workflow
### Dashboards / Screenboards
#### Creating a new deployment
1. Copy `config-example.yml` to `config.yml` and update it to match your environment, see [config.yml](#configyml) section below for more information on the parameters used therein.

#### Creating a new dashboard by importing from DataDog
1. Make sure your `config.yml` file is populated with necessary values. See [config.yml](#configyml) section below for more information.
2. Create a dashboard on the Datadog web UI (Dashboards -> New Dashboard)
3. Import the dashboard by ID, `https://app.datadoghq.com/dash/85829` where 85829 is the dashboard ID.

        bundle exec rake <environment>:get_dashboard_json_erb[<id number>,<path/to/template.json.erb>]

    - Note: do not add a space between the id number and the path. Rake is weird.
    - Note: the filename must end in `.json.erb` for the rake task to find and push the dashboard.
    - Note: this will pull down the screenboard into the given path, replacing the environment specific deployment with <%= deployment %>, the environment specific bosh deployment with <%= bosh_deployment %>, and putting the corresponding variables for the current environment in path/to/template_thresholds.yml.

4. Commit your changes to source control.

#### Pushing dashboard to datadog
1. Make sure your `config.yml` file is populated with necessary values. See [config.yml](#configyml) section for more information.
2. Push changes to deployment

        bundle exec rake prod:push

### Alerts

#### Creating a new alert from DataDog
Basically the same workflow as dashboards, but with different commands.

        bundle exec rake <environment>:get_alert_json_erb[<id number>,<path/to/template.json.erb>]

#### Pushing alerts to DataDog

        bundle exec rake prod:push

## config.yml
Parameters to the rake tasks and templates are defined in `config/config.yml`.
Each environment can have any key values defined.
These key value pairs are used for parsing downloaded templates. 
Any strings that match the values of these key value pairs will be replaced with ERB syntax.

### Search and replace:
```
search_and_replace:
  my_deployment_name: gobbledygoop
  # For example:
  # datadog.nozzle.mything: {deployment: gobbledygoop } 
  # turns into:
  # datadog.nozzle.mything: {deployment: <%= my_deployment_name %> }
  # 
  # You can also specify distinct search (Regexp) and replace (String) patterns:
  another_key_name:
    search: 'datadog\.nozzle.*\K(gobbledygoop)'
    replace: 'gobbledygoop'
  # This would match the above, but not match:
  # bosh.healthmonitor.mything: { deployment: gobbledygoop }
```

* **deployment**: This is the `name` value in the deployment manifest for your Runtime deployment.  
This can also be found via `bosh deployments`. 
NOTE: for Diego deployments, it's assumed that the name of your Diego deployment is `${name_of_cf-deployment}-diego`
* **bosh_deployment**: If you have a full BOSH deployed in your environment, this is the `name` from its deployment manifest

Threshold values to the templates are defined in `template_thresholds.yml`. These are auto-generated when importing from datadog.

## Folder structure

```
screen_templates/
├── images
├── shared
```

The screen_templates folder contains all of the template and thresholds for screen boards.
Templates in the `shared` folder are pushed to all of the environments, while templates in e.g. the `prod` folder will only be pushed to the prod DataDog.  Move the template json/erb file to the
appropriate folder and move the thresholds yaml to the same folder so that the two files are siblings.
`tags` will contain folders, and if the tag is present in `config.yml`, json/erb files present will be included for that environment.

Edit the resultant json/erb file to make sure that the auto-gsub bit didn't mangle something that wasn't supposed to be static. Check [here for further](lib/screen_synchronizer.rb#L48).

### Metric naming conventions

**PLEASE make sure your units are obvious just from reading the metric name.**

* **Good:** `uptime_seconds` and `free_memory_kilobytes`
* **Bad:** `uptime` and `free_memory`

Your teammates will thank you.

### Using Notes in screenboards
Notes are a fantastic way of creating titles for your various sections.
You can use markdown, meaning that you can have your titles serve the dual purpose of displaying a title and being clickable to allow for deeper inspection.

We have implemented import code that will detect such links and translate them between environments.
However, the code relies on certain semantics to function. When generating the links, use the following format:

```
[Title](/dash/dash/12345) # for dashboards
[Title](/screen/board/12345) # for screenboards
```
Anything else will be left as is.

## Generating fake data to test metrics

Use `rake <env>:emit[some.metric.name]` and follow the command line prompts.
Be careful, as this data may trigger false alarms, so be mindful of what you
are doing.

## Known issues
- Pulling from non-prod results in thresholds file being incorrect. Be careful here, because this is almost by design. If thresholds vary across envionments, a design decision was made to use production as default values, and allow other environments to override as necessary. This is problematic when it's equal across environments. WIP to be smarter about how to handle. Right now, it will just produce broken threshold files if pulling from non-prod.
- If there are no thresholds defined, it will break again. Just remove the thresholds file.
