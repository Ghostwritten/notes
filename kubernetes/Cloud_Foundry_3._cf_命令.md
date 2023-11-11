

##  1. Chapter Overview
In this module, we take you through the basics of deploying and managing applications in Cloud Foundry:

 - Source paths are mechanism for specifying the location of application content when pushing applications.
 - Manifests provide declarative configuration for an application. This section discusses how to make a manifest, and why they're a good approach when combined with version control.
 - Buildpacks are used to containerize applications. In this section, we briefly look at how buildpacks work in Cloud Foundry.
 - Examine the method for passing environment variables to applications.
 - Metadata allows you to provide additional information to resources with labels and annotations.
 - Apps can be scaled both horizontally and vertically in Cloud Foundry. This section describes the procedure for both.
 - Log output contains lots of useful information about an application,and is essential for troubleshooting.
 - Cloud Foundry is ensuring the correct number of app instances are running. This section demonstrates one of the resiliency features built into the platform.
 - Quotas are named sets of memory, service, and instance usage limits that apply to an org or space. As a Cloud Foundry developer, it's   important to understand what quotas are applied to the space you're working in.
 - Application security groups can be used to restrict egress from both staging and running apps.

##  2. Source Paths: Overview
Source paths let you specify a path to the application when running cf push. It is important to understand what you are pushing to Cloud Foundry.


### 2.1  Using the Current Directory
When you cf push without specifying a path, the contents of the current directory you ran the push command from are pushed up. This is what we did in the first-push exercise. However, you should be careful with this, as it can result in unexpected behavior when you are not in the directory you expect. Therefore, it is always safer to explicitly specify a path.

### 2.2  Specifying a Path
You can use the `--path` (or -p) <directory or file> flag to give the location of the directory containing the application content. The path can also point to a specific application artifact like a .jar or .zip file.

You can also specify the path in the manifest file:

```bash
---
  ...
  path: /path/to/app/bits
```

A quick word on manifests vs the CLI: Broadly, you can think of the CLI commands you run against an app as experimental. Any declarations made via the CLI can, and should be, reflected in the manifest if you'd like them to persist. Manifests live in version control and are shared among developers in a way that CLI commands cannot. Throughout the course, we will use a combination of CLI commands and manifests.

If you declare the -p parameter, this will take precedence over a path declared in the manifest.

If you push without specifying a path in the manifest or a -p parameter, the CLI will push the contents of your current working directory (we did this when we pushed our first-push app earlier in the course). It's a bad idea to get into this habit, as unexpected things can happen (like accidentally pushing a folder of your home movies). Therefore, you should always specify a source path.

### 2.3 Specifying a ZIP File
Cloud Foundry supports pushing .zip files containing your app code. An app packaged as a .zip file exists in the zip-file directory of the course materials. Let's push it, being sure to specify the path to the .zip file.

This app doesn't have a manifest with it, so we'll need to specify some parameters via the CLI.

cf push zip-with-src-path -p app.zip -b staticfile_buildpack -m 32M -k 64M --random-route

If you access the app in a browser, you should see the Cloud Foundry logo.

While specifying a path is best practice for all app deployments, when pushing .zip files, it is mandatory. If you don't pass a path to a .zip file, Cloud Foundry won't unzip the file before deploying it. The implications of this vary depending on the buildpack you're using. Our goal is to avoid any unexpected behavior.

Let's test this out ourselves with the staticfile buildpack and see what happens. Be sure you are in the zip-file directory before pushing.

```bash
cf push zip-no-src-path -b staticfile_buildpack -m 32M -k 64M --random-route
```

If you access this app in a browser, you should see a 403 error. If you access the app's URL and append it with `/app.zip`, the app package will be downloaded to your local machine. When we omitted the path, Cloud Foundry didn't unzip the files. The staticfile buildpack assumed you wanted to pass the entire directory contents during staging, even though it only contains a single file.

Again, make sure you always provide a path when pushing.


### 2.4  Tidy Up
Before we move on, let's clean up a little. Delete the zip-no-src-path app, as we won't need it anymore.

```bash
cf delete -r zip-no-src-path
```

Rather than deleting the zip-with-src-path app, let's rename it to something more sensible, so that we can use it in later sections:

```bash
cf rename zip-with-src-path static-app
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/dff14452480d430f9d6ea12042446037.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7db17e4a9f4144139ba1c63f004e2ae0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f29f7dc3f1a4acd9c18c4fd4bc26ce9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b907cda1363b48459b67889f3142f8be.png)
### 3.  Manifests Overview
Manifests have been mentioned a handful of times already in this course. In the Source Paths section, the static-app was deployed and configured entirely via command-line arguments. And while this worked fine for our first deployment, if we wanted to deploy it again in another space or share the configuration with a colleague, we'd need to recall the CLI arguments we used. Sharing CLI commands isn't a very reliable way to deploy our app, especially if something changes in our configuration.

Instead, we use a manifest file to specify values for all configurable parameters. Using a manifest file is a good practice because it can be kept in version control. A version-controlled manifest is consistent and can be shared between developers. It also allows manifest updates to be rolled out to multiple deployments quickly and efficiently, especially when integrated as part of a CI/CD pipeline.

### 3.1  Creating Manifests via the CLI
You can write a manifest from scratch manually by following the Cloud Foundry documentation. But that requires a fair amount of YAML wrangling. Instead, Cloud Foundry can create a manifest for you based on the current state of a deployed application. Use the CLI to generate a manifest for the static-app:

```bash
cf create-app-manifest static-app
```

The CLI will generate a manifest file in your current directory.

Note: the name of the file is `static-app_manifest.yml`, not manifest.yml, as it's named after the app. To use this manifest on a push, you need to supply the -f flag and the path to the manifest (or rename the file to manifest.yml, which is the default that the CLI will look for in the current directory).

Looking at the app manifest, you'll notice it includes the stack property. You didn't specify this when you pushed the app, so Cloud Foundry used its default value.

### 3.2  Testing the Manifest
Let's test our new manifest. Start by deleting the `static-app`, then re-pushing it with the manifest.

```bash
cf delete -f -r static-app
cf push -f static-app_manifest.yml
```

Try running cf `delete --help` to see what the flags in the command above do.

If you used the exact commands above, the app will not deploy correctly. When you open the app route in a browser, you will see a 403 error. This is the same error you saw in the Source Paths section. Open the manifest and see if you can figure out what's wrong.

Cloud Foundry can only include the config elements in the generated manifest that it knows about. The path to the `app.zip` file is missing from the generated manifest. Cloud Foundry doesn't know the path you used on your local filesystem. Fix this by adding the path to the manifest, or by pushing with the -p parameter. How you choose to fix this will depend on your use case. For now, let's push with the -p parameter:

```bash
cf push -f static-app_manifest.yml -p app.zip
```

The app should now correctly display the Cloud Foundry logo again.

### 3.3  CLI vs Manifest Precedence
When working with manifests, you can override their values on the command line. You can test this out by re-pushing, but specifying a different disk allocation.

```bash
cf push -f static-app_manifest.yml -p app.zip -k 128M
```

If you want to make this change permanent, you should add it to the manifest. Otherwise, the next time you push the manifest without the `-k 128M` parameter, the current value is overridden.

This reiterates the point made earlier in this section; passing parameters to the CLI is good for experimenting, but permanent changes should be reflected in a manifest that lives in version control.

You can also explicitly ignore the manifest when pushing using the `--no-manifest` flag.

### 3.4  Variables in Manifests
Manifests support the parameterization of values. For example, you likely want to use a different route for the development version of your app than for the production version. You could parameterize the route value and have it set on push. This allows you to use the same manifest for dev and production.

Variables are declared inside double parenthesis: i.e. ((my-variable)). To demonstrate, let's parameterize the number of instances in our manifest by editing the file:

```bash
---
applications:
- name: training-app
  instances: ((instances))
  memory: 64M
  buildpacks:
  - go_buildpack
```

We can then push with:

```bash
cf push -f static-app_manifest.yml -p app.zip --var instances=1
```

While this looks similar to cf push -i 1, variables are required to be passed in during push.

It is good practice to define a vars file for each environment (development, staging, prod, etc) rather than relying on variables on the command line. In this way, we would be declaring (in files in source control) all of the deployment configuration for each environment. We could therefore define a file static-app_dev.yml:

instances: 1

And then push with:

```bash
cf push -f static-app_manifest.yml -p app.zip --vars-file=static-app_dev.yml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/479f3d3956c0438b9496a5d1aba8764e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e7b7ec356384a4ebff89a549f67460b.png)
##  4. Buildpack Basics: Overview
Before we continue, let's briefly discuss this "buildpack" thing we have referenced a few times. Buildpacks are responsible for containerizing your application. Essentially, they provide the runtime environment your application needs. The Python buildpack knows how to construct runtimes for Python apps, the Java buildpack for Java, and Go buildpack for Go-lang, etc. The staticfile buildpack we have used so far uses nginx to serve static content.

Buildpacks construct runtimes during the "staging" phase of an application (more on this later).

### 4.1  Available Buildpacks
So far, we've been using the `staticfile_buildpack` in our push commands. But there are many others.

To view the available buildpacks in your Cloud Foundry instance run:

```bash
cf buildpacks
```

The listed buildpacks are configured by your Cloud Foundry operator. These available buildpacks are referred to as "system" buildpacks, as they are available in the system for all users.

### 4.2  Git Buildpacks
In addition to the system buildpacks, many Cloud Foundry instances allow you to specify the URL of a specific version of a buildpack in a git repository. In this case, Cloud Foundry will download the specific version of the buildpack and use it to stage the application.

Note the use of git buildpacks can be disabled by Cloud Foundry administrators.

If enabled, you could deploy the training-app using a specific version of a git buildpack with:

```bash
cf push -b htt‌ps://github.com/cloudfoundry/staticfile-buildpack.git#v1.5.21
```

Of course, you would want to add this value to your manifest if you planned to use it. We'll look more at buildpacks later in this course.

![在这里插入图片描述](https://img-blog.csdnimg.cn/1fc144504a424a16a34c41ddaedc02fa.png)
## 5. Environment Variables: Overview
Environment variables inject configuration values into applications. As the [12-factor app](https://12factor.net/config) principles state, we should "store config in the environment". In this section, we'll test out passing environment variables to a training-app.

### 5.1  Preparation
The golang application used in this section will display some basic information about the app itself, including its environment variables.

Firstly, let's deploy the `training-app` using the supplied manifest. The application and manifest are in the training-app directory in the course resources:

```bash
cf push training-app -f manifest.yml -p training-app.zip --random-route
```

Access the app in a browser and make sure you see can see the interface. You will notice that at this point the application UI is not displaying any environment variables.

### 5.2  Setting Environment Variables
This app will display any environment variable that starts with TRAINING_. Set some environment variables (but don't restart or restage yet):

```bash
cf set-env training-app TRAINING_KEY_1 training-value-1
cf set-env training-app TRAINING_KEY_2 training-value-2
cf set-env training-app TRAINING_KEY_3 training-value-3
```

Notice the CLI provides a tip: TIP: Use 'cf restage training-app' to ensure your env variable changes take effect. If you access your app in a browser before restarting/restaging, you will see why; the app does not see the new environment variables yet.

Now restart the app and view it in a browser (we will discuss restart versus restage in a later section). You should see the three environment variables you set above.

```bash
cf restart training-app
```
### 5.3 Inspecting Environment Variables
Most apps do not (and should not) show environment variables in their publicly accessible UI. However, you can use cf env to inspect their environment variables and values.

```bash
cf env training-app
```

Be sure you have restaged or restarted your app so that the app sees the same values you see when using cf env.

Before you restage or restart after changing environment variables, cf env will show the latest values, while the app will only see and display the older values in its UI.


### 5.4 Updating Environment Variables
You can change an environment variable value by using cf set-env and providing a new value:

```bash
cf set-env training-app TRAINING_KEY_3 training-value-3_1
```

Remember to restart the app to see the change.

### 5.5  Adding Environment Variables to the Manifest
To ensure the environment variables are configured correctly on every push, be sure to add them to the manifest. If you take a look at the training-app manifest, you'll see that the app already has one environment variable set called `GOPACKAGENAME`.

Update your local copy of the manifest to include the three `TRAINING_KEY_*` values previously set manually via cf set-env.

Delete the existing training-app and re-deploy using the updated manifest, and check that the environment variables are correct.

>   NOTE: You should avoid putting secrets directly in the manifest. Instead, use variables in the manifest and store secrets outside of
> source control.

### 5.6 Unsetting Environment Variables
To remove an environment variable, you use `cf unset-env`. Because you can set variables via the CLI or a manifest, simply removing a value from a manifest and re-pushing will not work. You must unset the value via the CLI and remove it from the manifest.

```bash
cf unset-env training-app TRAINING_KEY_1
```

### 5.7  Platform Environment Variables
When you read the environment variables for an app with cf env, you will have noticed that there were more than the user-provided values you set. Cloud Foundry also injects configuration for your application via environment variables. Two common variables are `VCAP_APPLICATION`, which provides configuration and information about the application instance, and `VCAP_SERVICES`, which provides configuration for accessing service instances. Applications can decide to use these environment variables if necessary (the sample application is reading and displaying some of these values). [A complete list of platform environment variables](https://training.linuxfoundation.cn/courses/play/4) is provided in the Cloud Foundry documentation.

### 5.8  Environment Variable Groups
Environment variable groups are system-wide variables that Cloud Foundry operators can apply to all running apps and all staging apps. Environment variable groups are typically used to provide generally applicable, albeit optional, configuration values for a specific Cloud Foundry instance. Each group consists of a single hash of case-sensitive name-value pairs that are inserted into an app container at either runtime or staging. These values can contain information such as HTTP proxy information.

When creating environment variable groups, only the CF operator can set the value for each group, but authenticated users can view the environment variables assigned to their app. In the event you set an environment variable of the same name, the user-defined value(s) you provide will take precedence over environment variables provided by these groups. So they can effectively be overwritten on an app-by-app basis.

Platform operators configure environmental variable groups. As a developer, you will not be configuring these values. However, you can view any existing environment variable groups with the following commands:

```bash
cf running-environment-variable-group

cf staging-environment-variable-group
```

These commands will retrieve the contents of any running and staging variable groups.

![在这里插入图片描述](https://img-blog.csdnimg.cn/5dc35d26f19a468a861a069b01e24a2f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7c623e87b794555991259b8222aec12.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bed2ac8431d145ec89b7017e893a0562.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fd7fa724336c432bab65a20b6c5d1f54.png)
修改环境变量
![在这里插入图片描述](https://img-blog.csdnimg.cn/d4d7e12f63f74c1c8c695af517d3822a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f45cd8fae30d42ada5720a40b4e901aa.png)

##  6. Metadata: Overview
Cloud Foundry provides the ability to add metadata to resources such as spaces and apps. This metadata provides additional information about the resources in your Cloud Foundry deployment. This can be a useful tool when `operating`, `monitoring`, and `auditing` Cloud Foundry.

There are two types of `metadata: labels` and `annotations`.


### 6.1  Labels
`Labels` are `key-value` pairs that allow you to identify and select Cloud Foundry resources. Labels are searchable. For example, if you have labeled all apps running in development, or all spaces containing internet-facing apps, you can then find those apps by searching for their labels.

Labels can be set via the CLI. Let's add some labels to our training-app:

```bash
cf set-label app training-app env=dev
```

We should be able to see that our labels have been set correctly in the app details:

```bash
cf labels app training-app
```

Now we can use labels as a search function to list all of the apps with the key/value env=dev:

```bash
cf apps --labels 'env=dev'
```

Multiple labels can be used to filter search results. Let's add another:

```bash
cf set-label app training-app sensitive=true
```

Now we can search for both labels using a comma-separated list:

```bash
cf apps --labels 'env=dev,sensitive=true'
```

The only app shown in the output should be the training-app.

Like always, labels should be added to the app manifest.

```bash
metadata:
  labels:
    sensitive: true
```

### 6.2  Annotations
`Annotations` allow you to add non-identifying metadata to CF resources. Unlike labels, you can't query based on annotations, and there are fewer restrictions than for key-value pairs. For example, you could include information about how the resource was deployed (via CI or non-CI), or tool information for debugging purposes.

Annotations cannot be added using a regular CLI command. Instead you'll need to add them via the app manifest. In an app manifest, annotations are included in the metadata block:

```bash
metadata:
  annotations:
    contact: "bob@example.com"
  labels:
    env: dev
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/456c305a927e44ad81b6d1febc89987c.png)
##  7. scale
###  7.1 Horizontal Scaling
When you scale an application horizontally, you manipulate the number of instances of that application up or down, usually in response to an increased or reduced user load.

In Cloud Foundry, horizontal scaling doesn't result in downtime as traffic is only routed to available instances. When new instances are created, traffic isn't routed to those instances until they pass a healthcheck (we'll cover healthchecks later in this course).

Let's start by scaling the training-app up to two instances:

```bash
cf scale -i 2 training-app
```

You can see the instances have been scaled successfully by running `cf app training-app` and observing `instances: 2/2` in the output.

###  7.2 Vertical Scaling
When you scale an application vertically, you are manipulating the resources allocated for each application instance container. In Cloud Foundry, you can change memory and disk allocations.

Let's scale the training-app instances down in both disk and memory.

```bash
cf scale -m 48M -k 256M training-app
```

What happened? Scaling vertically involves downtime as the containers are recreated with different resource allocations. This is an important point to consider when scaling vertically in production. Therefore, you should strive to right-size your resource allocations per instance during development and rely on horizontal scaling in production.

You can view the allocations for an app by running cf app `<app-name>`.

As always, if you'd like to make any scaling changes permanent, these values should be updated in a version-controlled app manifest. For example:

```bash
---
applications:
- name: training-app
  memory: 48M
  disk-quota: 256M
  instances: 2
```


### 7.3  A Quick Word on the App Autoscaler
The scaling techniques described on the previous pages are manual; they must be performed by a human to take effect. The App Autoscaler is an optional component that may be available in the Cloud Foundry instance you are using. This is a Cloud Foundry extension project, implemented as a marketplace service, and automatically scales apps based on performance metrics or a schedule.

Here are a few examples:

 - Automatically scale down the number of app instances over the weekend  and back up Monday morning.
 - Automatically scale up the number of instances when CPU usage exceeds a custom threshold and back down when it drops below a threshold.

The [App Autoscaler](https://github.com/cloudfoundry/app-autoscaler) is outside the scope of this course, but you can learn more online.

![在这里插入图片描述](https://img-blog.csdnimg.cn/1409d70b399a40fdafbe57678a49e5af.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/50aa1ee3a6324fcf943bdf7018a73916.png)

##  8. Logs: Overview
As a developer, you will frequently access application logs to debug your apps. Cloud Foundry uses a component called 'Loggregator' for streaming log output from applications and Cloud Foundry system components (in this course, we'll focus on application logs). Logs are incredibly helpful when troubleshooting. Common issues you might see are apps running out of memory, route collisions, or pushing the wrong application payload.

### 8.1  Following Logs in Real-Time
The CLI can stream new logs to your terminal:

```bash
cf logs training-app
```

You won't see any output until you access the `training-app`. Open the training-app URL and hit refresh a few times. Now that you've generated some traffic, log output should appear. You should see logs from the Cloud Foundry routers tagged with `[RTR/<some-index>`].

You can exit streaming with `CTRL-C`, or you can continue to stream and start a new terminal window.

> Note: The log streaming functionality uses a websocket to stream logs in near real-time. If you cannot stream logs while on a corporate network, you should check with the network administrators to ensure websockets are allowed。

### 8.2  Inspecting Recent Logs
You can also pass the `--recent` parameter to view only the recent logs for an app. This is useful if your application is producing a large volume of logs:

```bash
cf logs --recent training-app
```

The `--recent` buffer is finite in both space and time. If your app has been idle for a long period, you may not see anything in the recent logs.

### 8.3  Log Drains
As mentioned at the start of this section, by default the `Loggregator streams logs` to your terminal. Logs can be drained to a third-party log management service via `syslog` forwarding if you need to persist or analyze logs.

Optional information on [log drains](https://docs.cloudfoundry.org/devguide/services/log-management.html#user-provided) is available online.

###  8.4 Events
There are two main types of Cloud Foundry events: audit events and usage events. Audit events are largely used to monitor actions taken against resources, such as app restarts, stops, scaling, route mapping, and many more.

You can view the most recent events of an app with

```bash
cf events app-name
```

Alongside each event you should be able to see the user that executed that particular event.

Optional information on events is available here: [https://docs.cloudfoundry.org/running/managing-cf/audit-events.html](https://docs.cloudfoundry.org/running/managing-cf/audit-events.html).

 Container Metrics

Metrics for Cloud Foundry application containers can be made available for viewing via CLI plugins (more on these later). While container metrics are outside the scope of this course, it is useful to know of their existence. Optional information on metrics is available here: [https://docs.cloudfoundry.org/loggregator/container-metrics.html](https://docs.cloudfoundry.org/loggregator/container-metrics.html).

![在这里插入图片描述](https://img-blog.csdnimg.cn/5aea05474f164d6aa327c34197952884.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8cd3c0035783422893a13c33e9318fa2.png)
##  9. Resiliency: Overview
By performing cf scale, you asked Cloud Foundry to ensure you have two instances of the app running. Behind the scenes, Cloud Foundry is ensuring this is so. Let's test it.

To see this happen, we can watch the logs. In a terminal window, tail the logs:

```bash
cf logs training-app
```
###  9.1 Killing an App Instance
The application has a `/kill` endpoint that will crash the instance programmatically.

> WARNING: Don't build apps with a /kill endpoint! This is for training purposes only.

Go to your application in a browser. Tack on `/kill` to the URL and hit Enter. You will see a '502 Bad Gateway' error as you will have killed this instance. Quickly switch back to the terminal window where you are tailing the logs. You should see that Cloud Foundry (very quickly) detects an instance is missing and creates a new one.

If you go back to your browser, remove the `/kill` from the URL, you should be able to refresh and see that you are only being routed to the live instance. Once the new instance starts up, Cloud Foundry once again load balances across the healthy instances.

When you are ready, you can stop tailing the logs by hitting `CTRL-C` in the terminal window.

##  9.2 Pretty Cool. So What Happened?
Cloud Foundry is constantly monitoring the actual count of the running application instances and comparing that number to the desired count. If it finds an issue, Cloud Foundry will correct it (in our case, by creating a new instance). And just like during scaling, when the new instance becomes available, Cloud Foundry routes traffic to it. Cloud Foundry knows an instance unhealthy if it fails a health check. We discuss health checks later in the course.

And what didn't happen?
A lot. All apps get this level of care from Cloud Foundry. You don't have to do anything other than use Cloud Foundry.

![在这里插入图片描述](https://img-blog.csdnimg.cn/b0f8049aa4cd4875b2d4217b2b57ffad.png)
##  10. Quotas: Review
Quotas are named sets of memory, service, and instance usage limits. They can be applied at the org level or the space level. Org quotas are mandatory, while space quotas are optional. Org-level quotas are shared across all spaces in that org.

Typically, Cloud Foundry operators will establish quotas for orgs, and OrgManagers will establish quotas for spaces. So while it's not crucial that developers know how to manage quotas, it is important you understand how to view the quotas for your orgs and spaces. After all, resource planning is a collaborative effort.


### 10.1  Org Quotas
Let's take a look at what quota is on your current org:

```bash
cf org <org-name>
```

Once you've retrieved the quota, you can view its details:

```bash
cf org-quota <quota-name>
```

The output should show the resource limits on your org.

##  10.2 Space Quotas
As mentioned previously, quotas can be scoped at both the org and space level. Like org quotas, we can check what quota is applied on a space, by running

```bash
cf space <space-name>
```

If a space doesn't have an associated quota, then this value will be empty in the cf space output. If you have a space quota set, then do

```bash
cf space-quota <quota-name>
```

to see the details of that quota.
![在这里插入图片描述](https://img-blog.csdnimg.cn/3ebea14462464a3bac08b8299d791a18.png)
##  11. Application Security Groups: Overview
Application Security Groups (ASGs) are a collection of egress rules that list protocols, ports, and IP address ranges where an app or task is allowed to connect to (where your app can reach out to). ASGs define 'allow' rules. When ASGs are applied to the same space or deployment, their order of evaluation is unimportant; ASG's are additive.

ASGs can only be created or updated by admins. However, as ASGs dictate the outbound connectivity allowed, it is important to understand them as a developer.

Whenever application security groups are added, updated, or removed, you must restart your app for the changes to take effect.


###  11.1 Staging and Running ASGs
Security groups can be applied globally - to all orgs and spaces - by assigning them to either the staging or running ASG set. ASGs in the staging set determine the egress allowed during app staging, while those in the running set do so for app and task runtime.

When apps and tasks begin staging, they need traffic rules permissive enough to allow them to pull dependencies from the network. A running app or task is likely to have fewer needs of this sort, so ASGs applied during staging are often more permissive than those applied during the running lifecycle.

To view the ASGs for staging:

```bash
cf staging-security-groups
```

And for running ASGs:

```bash
cf running-security-groups
```

Once you've retrieved the name of the ASG you want to inspect, you can view the details of those rules:

```bash
cf security-group <security-group-name>
```

This will output a JSON object like so:

```bash
[
    {
      "protocol": "tcp",
      "destination": "0.0.0.0/0",
      "ports": "53"
    },
    {
      "protocol": "udp",
      "destination": "0.0.0.0/0",
      "ports": "53"
    }
]
```

The above example is one of the two ASGs preconfigured in open-source installations of Cloud Foundry (commercial distributions may have different defaults). The ASG, named dns, permits egress on port 53 over TCP and UDP to any IP address.

The other default ASG, `public_networks`, has the effect of blocking outbound traffic to the following private IP address ranges by omitting them from the ranges that it does permit:

```bash
10.0.0.0 - 10.255.255.255
169.254.0.0 - 169.254.255.255
172.16.0.0 - 172.31.255.255
192.168.0.0 - 192.168.255.255
```

Of course, egress to these IP address ranges will be permitted if another ASG is created allowing egress to them.

###  11.2 Space-scoped ASGs
It is also possible for admins to bind ASGs to a specific space. In such cases, the space-specific ASGs are combined with the platform ASGs to determine the rules for that space.

You will still need an admin to create ASGs that you want to apply to a particular space, but, once created, a Space Manager can bind the ASG to their space with the following command:

```bash
cf bind-security-group <SECURITY-GROUP> <ORG> --space <SPACE>
```

By default, this will apply the ASG as a running security group in the space. To instead set it as a staging security group use the `--lifecycle` flag.

![在这里插入图片描述](https://img-blog.csdnimg.cn/756c948891e14ec6aadbe9e3ee5e1c93.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e709c82af0e445469df7bcb1ca613f70.png)

