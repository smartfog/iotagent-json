# iotagent-mqtt

## Index

* [Overview](#overview)
* [Installation](#installation)
* [Usage](#usage)
* [Configuration] (#configuration)
* [Protocol](#protocol)
* [Command Line Client](#client)
* [Development documentation](#development)

## <a name="overview"/> Overview
This IoT Agent is designed to be a bridge between an MQTT+JSON based protocol and the OMA NGSI standard used in FIWARE.
This project is based in the Node.js IoT Agent library. More information about the IoT Agents can be found in its 
[Github page](https://github.com/telefonicaid/iotagent-node-lib).

## <a name="installation"/> Installation
There are two ways of installing the MQTT IoT Agent: using Git or RPMs.
 
### Using GIT
In order to install the TT Agent, just clone the project and install the dependencies:
```
git clone https://github.com/telefonicaid/iotagent-mqtt.git
npm install
```
In order to start the IoT Agent, from the root folder of the project, type:
```
bin/iotagentMqtt.js
``` 
 
### Using RPM
The project contains a script for generating an RPM that can be installed in Red Hat 6.5 compatible Linux distributions. 
The RPM depends on Node.js 0.10 version, so EPEL repositories are advisable. 

In order to create the RPM, execute the following scritp, inside the `/rpm` folder:
```
create-rpm.sh -v <versionNumber> -r <releaseNumber>
```

Once the RPM is generated, it can be installed using the followogin command:
```
yum localinstall --nogpg <nameOfTheRPM>.rpm
```

The IoTA will then be installed as a linux service, and can ve started with the `service` command as usual:
```
service iotaMQTT start
```
## <a name="usage"/> Usage
In order to execute the MQTT IoT Agent just execute the following command from the root folder:
```
bin/iotagentMqtt.js
```
This will start the MQTT IoT Agent in the foreground. Use standard linux commands to start it in background.

When started with no arguments, the IoT Agent will expect to find a `config.js` file with the configuration in the root
folder. An argument can be passed with the path to a new configuration file (relative to the application folder) to be
used instead of the default one.

## <a name="configuration"/> Configuration
### Overview
All the configuration for the IoT Agent is stored in a single configuration file (typically installed in the root folder).

This configuration file is a JavaScript file and contains two configuration chapters:
* **iota**: this object stores the configuration of the Northbound of the IoT Agent, and is completely managed by the
IoT Agent library. More information about this options can be found [here](https://github.com/telefonicaid/iotagent-node-lib#configuration).
* **mqtt**: this object stores MQTT's specific configuration. A detailed description can be found in the next section.

### MQTT configuration
These are the currently available MQTT configuration options:
* **host**: host of the MQTT broker.
* **port**: port where the MQTT broker is listening.
* **defaultKey**: default API Key to use when a device is provisioned without a configuration.
* **username**: user name that identifies the IOTA against the MQTT broker (optional).
* **password**: password to be used if the username is provided (optional).

## <a name="protocol"/> Protocol

### Configuration retrieval
The protocol offers a mechanism for the devices to retrieve its configuration (or any other value it needs from those
stored in the Context Broker). Two topics are created in order to support this feature: a topic for configuration
commands and a topic to receive configuration information.

#### Configuration command topic 
```
/{{apikey}}/{{deviceid}}/configuration/commands
```
The IoT Agent listens in this topic for requests coming from the device. The messages must contain a JSON document
with the following attributes:

* **type**: indicates the type of command the device is sending. The only currently allowed value is `configuration`.
* **fields**: array with the names of the values to be retrieved from the Context Broker entity representing the device.

This command will trigger a query to the CB that will, as a result, end up with a new message posted to the Configuration
information topic (described bellow).

E.g.:
```
{
  "type": "configuration",
  "fields": [
    "sleepTime",
    "warningLevel"
  ]
}
```

#### Configuration information topic 
```
/{{apikey}}/{{deviceid}}/configuration/values
```
Every device must subscribe to this topic, so it can receive configuration information. Whenever the device requests any
information from the IoTA, the information will be posted in this topic. All published messages are JSON Arrays, containing
one object per requested piece of information, with the following attributes:

* **name**: name of the requested attribute.
* **value**: current value of the attribute. 

E.g.:
```
[
  {
    "name": "sleepTime",
    "value": "200"
  },
  {
    "name": "warningLevel",
    "value": "80"
  }
]
```

### Measure reporting
There are two ways of reporting measures:

* **Multiple measures**: In order to send multiple measures, a device can publish a JSON payload to an MQTT topic with the 
following structure:
```
/{{api-key}}/{{device-id}}/attributes
```
The message in this case must contain a valid JSON object of a single level; for each key/value pair, the key represents
the attribute name and the value the attribute value. Attribute type will be taken from the device provision information.

* **Single measures**: In order to send single measures, a device can publish the direct value to an MQTT topic with
the following structure:
```
/{{api-key}}/{{device-id}}/attributes/<attributeName>
```
Indicating in the topic the name of the attribute to be modified.

In both cases, the key is the one provisioned in the IOTA through the Configuration API, and the Device ID the ID that
was provisioned using the Provisioning API. API Key MUST be present, although can be any string in case the Device was
provisioned without a link to any particular configuration.

### Value conversion
The IoTA performs some ad-hoc conversion for specific types of values, in order to minimize the parsing logic in the
device. This section lists those conversions.

#### Timestamp compression
Any attribute coming to the IoTA with the "timeInstant" name will be expected to be a timestamp in ISO8601 complete basic
calendar representation (e.g.: 20071103T131805). The IoT Agent will automatically transform this values to the extended
representation (e.g.: +002007-11-03T13:18:05) for any interaction with the Context Broker (updates and queries).

## <a name="client"/> Command Line Client 
The MQTT IoT Agent comes with a client that can be used to test its features, simulating a device. The client can be 
executed with the following command:
```
bin/iotaMqttTester.js
```
This will show a prompt where commands can be issued to the MQTT broker. For a list of the currently available commands
type `help`.

The client loads a global configuration used for all the commands, containing the host and port of the MQTT broker and
the API Key and Device ID of the device to simulate. This information can be changed with the `config` command.

In order to use any of the MQTT commands, you have to connect to the MQTT broker first. If no connection is available,
MQTT commands will show an error message reminding you to connect.

The Command Line Client gets its default values from a config file in the root of the project: `client-config.js`. This
config file can be used to permanently tune the MQTT broker parameters, or the default device ID and APIKey.
 
##  <a name="development"/> Development documentation
### Project build
The project is managed using Grunt Task Runner.

For a list of available task, type
```bash
grunt --help
```

The following sections show the available options in detail.

### Testing

[Mocha](http://visionmedia.github.io/mocha/) Test Runner + [Chai](http://chaijs.com/) Assertion Library + [Sinon](http://sinonjs.org/) Spies, stubs.

The test environment is preconfigured to run [BDD](http://chaijs.com/api/bdd/) testing style with
`chai.expect` and `chai.should()` available globally while executing tests, as well as the [Sinon-Chai](http://chaijs.com/plugins/sinon-chai) plugin.

Module mocking during testing can be done with [proxyquire](https://github.com/thlorenz/proxyquire)

#### Requirements

All the tests are designed to test end to end scenarios, and there are some requirements for its current execution:
- Mosquitto v1.3.5 server running
- MongoDB v3.x server running

#### Execution

To run tests, type
```bash
grunt test
```

Tests reports can be used together with Jenkins to monitor project quality metrics by means of TAP or XUnit plugins.
To generate TAP report in `report/test/unit_tests.tap`, type
```bash
grunt test-report
```

### Coding guidelines
jshint, gjslint

Uses provided .jshintrc and .gjslintrc flag files. The latter requires Python and its use can be disabled
while creating the project skeleton with grunt-init.
To check source code style, type
```bash
grunt lint
```

Checkstyle reports can be used together with Jenkins to monitor project quality metrics by means of Checkstyle
and Violations plugins.
To generate Checkstyle and JSLint reports under `report/lint/`, type
```bash
grunt lint-report
```


### Continuous testing

Support for continuous testing by modifying a src file or a test.
For continuous testing, type
```bash
grunt watch
```


### Source Code documentation
dox-foundation

Generates HTML documentation under `site/doc/`. It can be used together with jenkins by means of DocLinks plugin.
For compiling source code documentation, type
```bash
grunt doc
```


### Code Coverage
Istanbul

Analizes the code coverage of your tests.

To generate an HTML coverage report under `site/coverage/` and to print out a summary, type
```bash
# Use git-bash on Windows
grunt coverage
```

To generate a Cobertura report in `report/coverage/cobertura-coverage.xml` that can be used together with Jenkins to
monitor project quality metrics by means of Cobertura plugin, type
```bash
# Use git-bash on Windows
grunt coverage-report
```


### Code complexity
Plato

Analizes code complexity using Plato and stores the report under `site/report/`. It can be used together with jenkins
by means of DocLinks plugin.
For complexity report, type
```bash
grunt complexity
```

### PLC

Update the contributors for the project
```bash
grunt contributors
```


### Development environment

Initialize your environment with git hooks.
```bash
grunt init-dev-env 
```

We strongly suggest you to make an automatic execution of this task for every developer simply by adding the following
lines to your `package.json`
```
{
  "scripts": {
     "postinstall": "grunt init-dev-env"
  }
}
``` 


### Site generation

There is a grunt task to generate the GitHub pages of the project, publishing also coverage, complexity and JSDocs pages.
In order to initialize the GitHub pages, use:

```bash
grunt init-pages
```

This will also create a site folder under the root of your repository. This site folder is detached from your repository's
history, and associated to the gh-pages branch, created for publishing. This initialization action should be done only
once in the project history. Once the site has been initialized, publish with the following command:

```bash
grunt site
```

This command will only work after the developer has executed init-dev-env (that's the goal that will create the detached site).

This command will also launch the coverage, doc and complexity task (see in the above sections).

