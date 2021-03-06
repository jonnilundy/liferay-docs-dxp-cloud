# Liferay DXP Service [](id=liferay-dxp-service)

The Liferay DXP service is the heartbeat of your project. It runs your 
application's Liferay DXP instance and interacts with other services like the 
Proxy, Elasticsearch, and RDS database. 

![Figure 1: The Liferay DXP service is one of several services available in DXP Cloud.](../../images/services-dxp.png)

## Deployment [](id=deployment)

To install themes, portlets, or OSGi modules, you can include a WAR or JAR file
in the `/deploy/common` folder and then push it to DXP Cloud.

For example, to deploy a custom JAR file, your Liferay DXP source code directory 
could look like this:

    liferay
    ├── deploy
    │ └── common
    │   └── com.liferay.apio.samples.portlet-1.0.0.jar
    └── wedeploy.json

Under the hood, those files are copied to the `$LIFERAY_HOME/deploy` folder and 
deployed on startup. 

## Licenses [](id=licenses)

It's possible to add your own license by creating a `license` folder for the 
license: 

    liferay
    ├── license
    │ └── license.xml
    │ └── license.aatf
    └── wedeploy.json

Depending on the license's format, the license will be copied into either the 
`deploy` or `data` folders of the Liferay Home folder. XML licenses are copied 
to the `deploy` folder, and AATF licenses are copied to `data` folder. 

## Hot Deploy [](id=hot-deploy)

Using hot deploy in DXP Cloud isn't recommended. If you still want to use hot 
deploy, you can do so via the Liferay DXP UI. 

## Configurations [](id=configurations)

To launch new property or OSGI configurations, you can use the `config` folder 
as an extension point. This extension point supports `.cfg`, `.properties`, and
`.config` files.

For example, to deploy a configuration for the Elasticsearch OSGi bundle, your 
folder structure could look like this: 

    liferay
    ├── config
    │ └── com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config
    └── wedeploy.json

Under the hood, all files are copied into the `$LIFERAY_HOME` folder and 
automatically applied on startup. 

## Portal Properties [](id=portal-properties)

There is also a set of files you can use to configure the environment. The 
portal reads these files in the following order: 

`portal-all.properties`: Contains the properties that change Liferay DXP across 
environments.

`portal-env.properties`: Contains the properties that only affect the current 
environment (e.g., credentials and URL endpoints for external services that 
differ from environment to environment).

`portal-clu.properties`: Contains the preconfigured properties for clustering
Liferay DXP on DXP Cloud. You typically don't need to change these properties. 
See the Clustering section for more information. 

`portal-ext.properties`: Contains the final changes to your Liferay DXP 
configuration. Since you should set most of your properties in 
`portal-all.properties` and `portal-env.properties`, this file is typically 
empty or missing altogether. For testing, however, you may find 
`portal-ext.properties` useful. 

## Clustering [](id=clustering)

Clustering Liferay DXP on DXP Cloud is straightforward: set the environment 
variable `WEDEPLOY_PROJECT_LIFERAY_CLUSTER_ENABLED` to `true`. This instructs 
the image startup process to add the configuration to Liferay DXP for 
establishing a cluster. 

Behind the scenes, the image startup process copies the file 
`portal-clu.properties` and `unicast.xml` to the Liferay Home folder. These 
files contain the configuration needed to run a Liferay DXP cluster on DXP 
Cloud. 

## Hotfixes [](id=hotfixes)

To apply hotfixes, add the hotfix ZIP file to the `hotfix` folder. When you 
deploy this change, the hotfix is applied to your application.

    liferay
    ├── hotfix
    │ └── liferay-hotfix-2-7110.zip
    └── wedeploy.json

## Scripts [](id=scripts)

You can use scripts for more extensive customizations. However, use caution when 
doing so. This is the most powerful way to customize Liferay DXP and can cause 
undesired side effects. 

Any `.sh` files found in the `script` folder are run prior to starting your 
service. For example, to include a script that removes all log files, you could 
place it in the following directory structure: 

    liferay
    ├── script
    │ └── remove-log-files.sh
    └── wedeploy.json

## Advanced Monitoring with Dynatrace [](id=advanced-monitoring-with-dynatrace)

To enable advanced monitoring with Dynatrace on Liferay DXP, you must set two
environment variables: 

`WEDEPLOY_PROJECT_MONITOR_DYNATRACE_TENANT`: The tenant value is a string with 
eight characters. It's part of the URL (prefix) of your Dynatrace SaaS product. 

`WEDEPLOY_PROJECT_MONITOR_DYNATRACE_TOKEN`: The token is another string with 22 
characters that you can find in your Dynatrace account at *Deploy Dynatrace* 
&rarr; *Start installation* &rarr; *Set up PaaS monitoring* &rarr; 
*InstallerDownload*. 

## Working with the Document Library

To browse files in your Liferay DXP service, you can use the web file manager 
[Cloud Commander](http://cloudcmd.io/). With Cloud Commander you can access your 
Liferay DXP Document Library folder, and download and/or upload files. 

To add Cloud Commander to your project, first create the `cloudcmd` folder in 
your `wedeploy/` folder. Inside `cloudcmd`, create a `wedeploy.json` file with 
the following configuration: 

    {
        "env": {
            "CLOUDCMD_AUTH": "true",
            "CLOUDCMD_ONE_FILE_PANEL": "true",
            "CLOUDCMD_PASSWORD": "passw0rd",
            "CLOUDCMD_USERNAME": "user"
        },
        "id": "cloudcmd",
        "image": "coderaiser/cloudcmd",
        "port": 8000,
    }

You can replace the value of `CLOUDCMD_USERNAME` or `CLOUDCMD_PASSWORD` with any 
user name you want as well as the password. 

### Configuring Access to the Document Library

For other services in the same environment to access your Liferay DXP service's 
Document Library folder, you must add a volume to that folder. To do this, add a 
`volume` to your Liferay DXP service's `wedeploy.json` as follows: 

    "volumes": {
        "data": "/opt/liferay/data/document_library"
    }

Now that this folder is reachable, you can set it as the Cloud Command root 
folder (the folder Cloud Command displays in a browser). To do this, set the 
`CLOUDCMD_ROOT` environment variable to the folder in your `cloudcmd` service: 

    "env": {
        "CLOUDCMD_ROOT": "/opt/liferay/data/document_library",
    }
    "volumes": {
        "data": "/opt/liferay/data/document_library"
    }
