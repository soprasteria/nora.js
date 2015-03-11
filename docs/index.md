Documentation Nora.js
==

----------


## Introduction ##

Nora.js is a Soap webservice framework. It's designed to help you to write complex testcases with minimum programmation.

### Features : ###
 - Load properties from property file or generate property from a javascript module
 - XML payload templating engine
 - xpath of full text assertions
 - ...

## Test case definition ##

 A test case is a set of test steps. Each of steps are defined by action, and properties. Nora.js supports several kind of actions.

 - **loadProperties** : Load or generate properties in the Nora.js engine.
 - **makeRequest** : Prepare a SOAP request from a template.
 - **sendRequest** : Send a SOAP request, and get the response.
 - **checkXML** : Test assertions on a XML payload

A testcase is described as a JSON object, in a .json file. See sample below.

## Test case sample ##

The goal of this test case is to test a weather broadcasting service.
The steps of this test are the following.

1. Load the zipcode from a properties generator, host and port of the web services from a properties file
2. Prepare the request from a payload template to setup the zipcode
3. Send request to the web service provider
4. Tests the response (contains Houston, not contains Error and some xpath match)

----------

	[
	{
		"stepName" :	"Load Properties",
		"stepID" : "1",
		"stepAction" : "loadProperties",
		"stepOptions" : [{
			"generator" : "zipcode.js"
				}, {
			"filename" : "getWeather.properties.json"
				}]	
	},{
		"stepName" : "Compute getWeather Request",
		"stepID" : "2",
		"stepAction" : "makeRequest",
		"stepOptions" : {
			"requestID" : "getWeatherRequest",
			"requestTemplate" : "getWeatherTemplate.xml"
			}
	},{
		"stepName" : "Send GetWeather Request",
		"stepID" : "3",
		"stepAction" : "sendRequest",
		"stepOptions" : {
			"requestID" : "getWeatherRequest",
			"protocol" : "http",
			"host" : "${host}",
			"port" : "${port}",
			"path": "/WeatherWS/Weather.asmx",
			"SOAPAction": "http://ws.cdyne.com/WeatherWS/GetCityForecastByZIP",
			"responseID" : "getWeatherResponse"
			},
		"stepReplayOnFailure" : 2,
		"stepWaitBeforeReplay" : 2
	}, {
		"stepName" : "Check GetWeather Response",
		"stepID" : "4",
		"stepAction" : "checkXML",
		"stepOptions" : {
			"xmlID" : "getWeatherResponse",
			"asserts" : [
				{
					"type" : "contains",
					"value" : "City Found"
				},
				{
					"type" : "notContains",
					"value" : "Error"
				},
				{
					"type" : "xpath",
					"xpath" : "//weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity",
					"namespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"},
					"match" : "Houston"
				},
				{
					"type" : "xpath",
					"xpath" : "//weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity",
					"namespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"},
					"match" : "${getWeatherResponse://weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity}",
					"matchNamespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"}
				}
			]
		}
	}
	]


----------

#### Load Properties ####

Sample :

	{
		"stepName" :	"Load Properties",
		"stepID" : "1",
		"stepAction" : "loadProperties",
		"stepOptions" : [{
			"generator" : "zipcode.js"
				}, {
			"filename" : "getWeather.properties.json"
				}]	
	}

##### Properties files #####

A properties file is a json object representing an array of propertyName and propertyValue. Sample below is a set of two properties.

	[
	{
		"propertyName" : "host",
		"propertyValue" : "wsf.cdyne.com"
	},
	{
		"propertyName" : "port",
		"propertyValue" : 80
	}
	]

##### Properties Generator Scripts #####

Sometimes some properties are generated by a script. It have to be a **node.js** module, it will be required "**require(...)**" by the engine of Nora.js. Sample below will generate a property named "zipcode".

	var generateProperties = function() {
		return [{propertyName : "zipcode",  propertyValue : "77001"}];
	};
	
	module.exports = generateProperties;

The **generateProperties** function is the function which will do the job, but don't forget to declare this function as a module entry point with **module.exports = generateProperties;**

####Make Request####

Before sending request, you will have to prepare it. Define a template in wich you can use the properties you defined in **properties file** of **properties generator**.

	{
		"stepName" : "Compute getWeather Request",
		"stepID" : "2",
		"stepAction" : "makeRequest",
		"stepOptions" : {
			"requestID" : "getWeatherRequest",
			"requestTemplate" : "getWeatherTemplate.xml"
			}
	}

To use a property in a template defined in **requestTemplate** just use the **${property}** notation as the sample below.

	<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">
    <Body>
        <GetCityForecastByZIP xmlns="http://ws.cdyne.com/WeatherWS/">
            <ZIP>${zipcode}</ZIP>
        </GetCityForecastByZIP>
    </Body>
	</Envelope>

Be sure that the property (**zipcode**) has been loaded previsiously by a **loadProperty** step.

With a makeRequest step, the **requestID** will be the reference to the xml payload in the Nora.js engine.

####Send Request####

The heart of the engine is the http request engine. 

	{
		"stepName" : "Send GetWeather Request",
		"stepID" : "3",
		"stepAction" : "sendRequest",
		"stepOptions" : {
			"requestID" : "getWeatherRequest",
			"protocol" : "http",
			"host" : "${host}",
			"port" : "${port}",
			"path": "/WeatherWS/Weather.asmx",
			"SOAPAction": "http://ws.cdyne.com/WeatherWS/GetCityForecastByZIP",
			"responseID" : "getWeatherResponse"
			}
	}

- **requestID** is the reference to the xml payload of have defined by a **makeRequest** step.

- **protocol** have to be http or https

- **host** is the server name of the web services provider

- **port** is the port of the web services provider

- **path** is the path path of the URL

- **SOAPAction** is the SOAPAction header of the web service you want to call.

- **responseID** will be the reference if the xml payload of the response in the Nora.js engine.

Note that the **host**, **port**, **path**, **protocol** and **SOAPAction** options can use propeties.

####Wait & Replay####

On every step, you can add wait & replay options. 

	{
		"stepName" : "Send GetWeather Request",
		"stepID" : "3",
		"stepAction" : "sendRequest",
		"stepOptions" : {
			...
			},
		"stepReplayOnFailure" : 2,
		"stepWaitBeforeReplay" : 5
	}

This means that the Nora.js engine will replay 2 times the step while the step is failed, but it will wait 5 seconds between each replay.
Each of this options is not mandatory.


####Check XML####

This will perform asserts on a xml payload. 
	
	{
		"stepName" : "Check GetWeather Response",
		"stepID" : "4",
		"stepAction" : "checkXML",
		"stepOptions" : {
			"xmlID" : "getWeatherResponse",
			"asserts" : [
			...
			]
		}
	}

The reference to the xml payload is **xmlID**. It is mandatory. Then you have to define a set asserts.

#####Assert "contains"#####

Tests if the xml payload contains some value.

	"asserts" : [
				{
					"type" : "contains",
					"value" : "City Found"
				}
				]

In the value, you can also use properties.

#####Assert "notContains"#####

Tests if the xml payload does not contain some value.

	"asserts" : [
				{
					"type" : "notContains",
					"value" : "Error"
				}
				]

In the value, you can also use properties.

#####Assert "xPath match"#####

Tests if a xPath query matches with some value.

				{
					"type" : "xpath",
					"xpath" : "//weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity",
					"namespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"},
					"match" : "Houston"
				}

In the value, you can also use properties and xPath properties.

####xPath Properties####

Anywere you can you properties (template, options, asserts, ...) you can use xPath based properties. It means you can refer to a xPath query with any xml paylod known by the Nora.js engine.

	${getWeatherResponse://weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity}

The notation is 

	${**xml Reference ID**:**xPath query**}

But don't forget you declare namespaces by add namespaces declaration in **stepOptions** in a **makeRequest** step, assert in a **contains** or **notContains** of **checkXML** asserts or add a **matchNamespaces** to use it in a xQuery matching assert.

*makeRequest with namespaces:*

	{
		"stepName" : "Compute getWeather Request",
		"stepID" : "2",
		"stepAction" : "makeRequest",
		"stepOptions" : {
			"requestID" : "getWeatherRequest",
			"requestTemplate" : "getWeatherTemplate.xml",
			"namespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"},
			}
	}

*contains assert with namespaces:*

	"asserts" : [
				{
					"type" : "contains",
					"value" : "${getWeatherResponse://weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity}",
					"namespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"}
				}
				]

*xpath match with namespaces:*

			{
					"type" : "xpath",
					"xpath" : "//weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity",
					"namespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"},
					"match" : "${getWeatherResponse://weather:GetCityForecastByZIPResponse/weather:GetCityForecastByZIPResult/weather:WeatherStationCity}",
					"matchNamespaces": {"weather" : "http://ws.cdyne.com/WeatherWS/"}
				}

----------

##File & Folders Organization##

A test case is a single json file. Since severals files (properties, template, etc...) are pre-requisites to the testcase, their paths have to be relatives to the testcase.

Each times you run the test case, a new directory will be created under the **runs** directory. In this directory, every xml paylod will be saved.

----------

##Command & Options##

	$ node nora -h

  	Usage: nora [options]

  		Options:

    	-h, --help                 output usage information
    	-V, --version              output the version number
    	-t, --testcase [filename]  set the testcase