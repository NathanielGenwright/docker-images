# Katalon Studio Docker Image 

The following how-to guide is for running Katalon Studio test with Katalon Docker Image (KDI) version 7.2.1 onwards.

### Preconditions

* Katalon Runtime Engine Floating License (See [pricing plan](https://www.katalon.com/pricing/))
* Install and Start Docker on your local machine (See manual for [downloading and installing Docker Desktop](https://docs.docker.com/desktop/#download-and-install))

### Step 1: Pull KDI

* Pull command `docker pull katalonstudio/katalon`

* If you want to check which version of Google Chrome and Mozilla Firefox the KDI supports, use the following command: `docker run -t --rm katalonstudio/katalon cat /katalon/version`. For example, here's the returned output.

```

$ docker run -t --rm katalonstudio/katalon cat /katalon/version

+ echo Entrypoint

Entrypoint

+ '[' -z '' ']'

+ exec cat /katalon/version

Mozilla Firefox 88.0

Google Chrome 92.0.4515.159 

Katalon Studio 
```

### Step 2: Run your test with KDI

**3.1 Prepare your command**

3.1.1 Use the [command builder](https://docs.katalon.com/katalon-studio/docs/console-mode-execution.html#command-builder) to prevent syntax errors. Here’s a sample command generated by the command builder:

`./katalonc -noSplash -runMode=console -projectPath="<Your project path>" -retry=0 -testSuiteCollectionPath="Test Suites/TS_RegressionTestCollection" -apiKey="<Your API key>"`

3.1.2 Replace 

`./katalonc -noSplash -runMode=console -projectPath="<Your project path>"` 

with 

`docker run -t --rm -v "$(pwd)":/tmp/project katalonstudio/katalon katalonc.sh -projectPath=/tmp/project`

**3.2 Run your test with KDI**

Inside your **test project directory**, run the command. Here’s an example:

`docker run -t --rm -v "$(pwd)":/tmp/project katalonstudio/katalon katalonc.sh -projectPath=/tmp/project -retry=0 -testSuiteCollectionPath="Test Suites/TS_RegressionTestCollection" -apiKey="<Your API key>"`

> `katalonc.sh` command will start Katalon Studio and other necessary components. All Katalon Studio console mode arguments are accepted except `-runMode`.

### Configure Proxy

If you need to configure proxy for Katalon Studio, refer to [Proxy Options](https://docs.katalon.com/katalon-studio/docs/console-mode-execution.html#proxy-options) provided on Katalon docs.

Do not forget to put `--config` before the proxy configuration. For example:

```
docker run -t --rm -v "$(pwd)":/katalon/katalon/source katalonstudio/katalon katalonc.sh -projectPath=/katalon/katalon/source -browserType="Chrome" -retry=0 -statusDelay=15 -testSuitePath="Test Suites/TS_RegressionTest" -apikey=<YOUR_API_KEY> --config -proxy.option=MANUAL_CONFIG -proxy.server.type=HTTP -proxy.server.address=192.168.1.221 -proxy.server.port=8888
```

### Prevent user permission issue on your machine

You can also run the test under the current user ID using the environment variable `KATALON_USER_ID`. This will help avoid permission issues when accessing artifacts generated after the test execution.

* Run $ id -u $USER and copy/paste the output in KATALON_USER_ID=`id -u $USER`. Here's an example
```
$ id -u $USER

665056758
```

* Add it to your command. For example: `docker run -t --rm -e KATALON_USER_ID=665056758 -v "$(pwd)":/tmp/project katalonstudio/katalon katalonc.sh -projectPath=/tmp/project`

### Display configuration

This image makes use of Xvfb with the following configurations which are configurable with `docker run`:

```
ENV DISPLAY=:99
ENV DISPLAY_CONFIGURATION=1024x768x24
```

### Jenkins

Please see [the sample `Jenkinsfile`](https://github.com/katalon-studio-samples/ci-samples/blob/master/Jenkinsfile).

### CircleCI

This image is compatible with CircleCI 2.0. Please see [the sample `config.yml`](https://github.com/katalon-studio-samples/ci-samples/blob/master/.circleci/config.yml).

### Sample configurations for CI tools

Please visit https://github.com/katalon-studio-samples/ci-samples for a sample project with configurations for some CI tools.

## Build custom images

The Katalon Runtime Engine's `katalonc` and its companion script `katalonc.sh` were added to `$PATH`. You can make use of these files to build custom images.

## Companion product: Katalon TestOps

[Katalon TestOps](https://analytics.katalon.com) is a web-based application that provides dynamic perspectives and an insightful look at your automation testing data. You can leverage your automation testing data by transforming and visualizing your data; analyzing test results; seamlessly integrating with such tools as Katalon Studio and Jira; maximizing the testing capacity with remote execution.

* Read our [documentation](https://docs.katalon.com/katalon-analytics/docs/overview.html).
* Ask a question on [Forum](https://forum.katalon.com/categories/katalon-analytics).
* Request a new feature on [GitHub](CONTRIBUTING.md).
* Vote for [Popular Feature Requests](https://github.com/katalon-analytics/katalon-analytics/issues?q=is%3Aopen+is%3Aissue+label%3Afeature-request+sort%3Areactions-%2B1-desc).
* File a bug in [GitHub Issues](https://github.com/katalon-analytics/katalon-analytics/issues).

<details><summary><strong>Deprecated - Simple use case for KDI before 7.2.1</strong></summary>
<p>

Inside the test project directory, execute the following command:

```
docker run -t --rm -v "$(pwd)":/katalon/katalon/source katalonstudio/katalon katalon-execute.sh -browserType="Chrome" -retry=0 -statusDelay=15 -testSuitePath="Test Suites/TS_RegressionTest" -apikey=<YOUR_API_KEY>
```

**`katalon-execute.sh`**

This command will start Katalon Studio and other necessary components. All [Katalon Studio console mode arguments](https://docs.katalon.com/display/KD/Console+Mode+Execution) are accepted *except* `-runMode`, `-reportFolder`, and `-projectPath`.

**`/katalon/katalon/source`**

`katalon-execute.sh` will look for the test project inside this directory.

If this bind mount is not used, `katalon-execute.sh` will look for the test project inside the current working directory (defined with `docker run`'s `-w` argument)..

```
docker run -t --rm -v "$(pwd)":/tmp/source -w /tmp/source katalonstudio/katalon katalon-execute.sh -browserType="Chrome" -retry=0 -statusDelay=15 -testSuitePath="Test Suites/TS_RegressionTest" -apikey=<YOUR_API_KEY>
```

**Reports**

Reports will be written to the `report` directory.

> **Docker Toolbox for Windows**
>
> Please make sure directories have been shared and configured correctly https://docs.docker.com/toolbox/toolbox_install_windows/#optional-add-shared-directories.

If bind mount `/katalon/katalon/report` is used, the test reports will be written to that location on the host machine.

</p>
</details>
