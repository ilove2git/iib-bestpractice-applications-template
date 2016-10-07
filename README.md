## The IBM Integration Bus Docker Framework - AppLayer (DeveloperTemplate) *ALPHA*
This repository includes a docker image framework for IBM Integration Bus according to [container best practices](http://developers.redhat.com/blog/2016/02/24/10-things-to-avoid-in-docker-containers/).

The Framework exists of two Layers:
  - [RuntimeLayer](https://github.com/dennisseidel/iib-bestpractice-runtimes): This repository includes the different Runtime Images that are prepared and are the foundation for the the AppLayer the developer is only concerted with the AppLayer. This images should not change often.
  - [AppLayer](https://github.com/dennisseidel/iib-bestpractice-applications-template): This repository include a template for a developer to develop his own immutable image for his applications.



### Use of this Template Repository

This include the *sampleAppname* as a **template** for the developer to develop his own immutable image by doing the following:

1. Check out this repo and rename `sampleAppname` your application name. Update also the build path in the docker-compose file to point to your application.
2. Set the environment variables in the `env` file and run it in your terminal `. ./env`. Including:
  - REPO: the domain of your docker repository you want to push to if empty it used docker hub.
3. update the version number of the appiciation in the docker-compose file e.g. `${REPO}iib-sample-app-with-customconfig:1.0.0`
4. Decide if you need a `customeconfig.sh` because you need to send other setting when starting IIB, by default it creates an instance with the following config:

### Default config of the [runtime images iib-10.0.0.6-mqclient / iib-10.0.0.6](https://github.com/dennisseidel/iib-bestpractice-runtimes)
  - nodename: MYNODE
	- integrationservername: default
	- two users for the webadminui, also to be used when connecting from IIB Toolkit to the Integrationnode:
		- admin:
			- can read, write, execute
			- password must be set by definining it in the IIB_ADMINPW variable
		- observer:
			- can read only
			- password must be set by definining it in the IIB_OBSERVERPW variable

### Image Configuration and Customization Options

- Decide if you want to switch features on or off by setting environment variables:
    - `IIB_TRACEMODE`: this can be set to `on` or `off` to en/disable trace nodes.
    - `IIB_LICENSE`: this must be set to `accept` to indicate that you accepted the IBM License Agreement
    - `IIB_SKIPDEPLOY`: Skip deployment of IIB applications, useful in development
    - `IIB_GLOBALCACHE`: if set to `internal` the global cache on IIB is just enabled. If set to `external` the connection to an external IBM Extreme Scale is configured. This requires the following environment variables to be set:
      - `IIB_GC_USER`: username to connect to IBM Extreme Scale
      - `IIB_GC_PASSWD`: password to connect to IBM Extreme Scale
      - `IIB_GC_CATALOGENDPOINT`: catalogendpoint to connect to IBM Extreme Scale
      - `IIB_GC_GRIDNAME`: gridname to connect to IBM Extreme Scale
		The PKI Infrastructure can be configured as follows by setting the following env variables the iib configuration will be done by the base image automatically if both variables are available and the keystore:
			- `IIB_KEYSTOREPW`: the keystore password, can be set as an environment variable or in the `pw.sh` file mounted under `/secret/pw.sh`.
			- `IIB_TRUSTSTOREPW`: the truststore password, can be set as an environment variable or in the `pw.sh` file mounted under `/secret/pw.sh`.
			- Add a keystore and truststore under `/secret/keystore.jks` and `/secret/truststore.jks` to enable HTTPS or SSL MQ.
- Exposed Ports:
    - `4414`: Port of the IIB Admin WebUi and for remote debugging in IBM Integration Bus Toolkit
    - `7800`: Port of the HTTP Listener


### Build and Staging
7. Build the image with a `docker-compose build`.
8. Start the container locally with `docker-compose up` and test the application locally before checking your code into a
repository like github or a container repository.
9. If everything is ok a developer should check his source code into his git repo and his build tool like jenkins should build
and test and then do a `docker-compose push` to the container repository.
10. Next prepare your stage where you deploy to.
  - Create a stage specific volume/secret under `/secret/pw.sh` into your container (for the content see the example `pw.sh`).
  - Create a stage specific volume/configmap under `/usr/local/bin/customconfig.sh` into your container (for the content see the example customconfig.sh).
  - Create a stage specific volume/configmap under `/usr/local/bin/customconfig.sh` into your container (for the content see the example customconfig.sh).
  - Create stage specific volumes/configmaps for the properties file for every bar file you want to overwrite something under `/iibProperties/` into your container.
    - Properties files are create with the `mqisreadbar -b ./path/to/barfile.bar -r > barfile.properties`.
  - Mount Keystores into ... COMING SOON.

*Remember: When changing something, create new image, stage and test it and deploy it, DON'T change a running container.*

# Development Best Pratices for IIB on a Container Runtime Plattform
1. Don't use [features](http://www.ibm.com/support/knowledgecenter/en/SSMKHH_10.0.0/com.ibm.etools.mft.doc/ah09088_.htm) that require a MQ Queue Manager in direct binding in the same image.
2. Structure you flow logic that it can scaled independently with [CallableFlow](http://www.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.iib.cloud.doc/cl00029_.htm) into the following parts:
  - CallingFlow: This includes one input node, (a mapping to the structure of the callableFlow, this allows for different input node to call the same flow logic), the CallableFlowInvoke, (a mapping to the response structure if necessary) and the output node.
  - CallableFlow: CallableInput, FlowLogic, CallableOutput.  
