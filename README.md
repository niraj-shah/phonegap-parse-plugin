Cordova / PhoneGap Parse.com Plugin
=========================

Cordova / PhoneGap 5.0+ plugin for Parse.com. Updated to support [Open Source ParseServer](https://github.com/ParsePlatform/parse-server/).

Uses:
- Parse Android SDK v1.13.0
- Parse iOS SDK v1.12.0

Using [Parse.com's](http://parse.com) REST API for Push requires the Installation ID, which isn't available in JS

This plugin exposes native API push services to JS:
* <a href="https://www.parse.com/docs/android/api/com/parse/ParseInstallation.html#getInstallationId()">getInstallationId</a>
* <a href="https://www.parse.com/docs/android/api/com/parse/PushService.html#getSubscriptions(android.content.Context)">getSubscriptions</a>
* <a href="https://www.parse.com/docs/android/api/com/parse/PushService.html#subscribe(android.content.Context, java.lang.String, java.lang.Class, int)">subscribe</a>
* <a href="https://www.parse.com/docs/android/api/com/parse/PushService.html#unsubscribe(android.content.Context, java.lang.String)">unsubscribe</a>
* <a href="https://parse.com/docs/osx/api/Classes/PFAnalytics.html#//api/name/trackEvent:dimensions:">trackEvent (iOS only)</a>

As well as other utility methods:
* registerCallback: allows us to get the notification back in javascript
* resetBadge: resets the badge to 0 (iOS only) --this can also be accomplished by setting the badge to 0 in the _Installation table using the <a href="https://parse.com/docs/rest/guide#objects-updating-objects">Parse REST API</a>

Installation
------------

Pick one of these two commands:

```
phonegap local plugin add https://github.com/niraj-shah/phonegap-parse-plugin --variable APP_ID=PARSE_APP_ID --variable CLIENT_KEY=PARSE_CLIENT_KEY
cordova plugin add https://github.com/niraj-shah/phonegap-parse-plugin --variable APP_ID=PARSE_APP_ID --variable CLIENT_KEY=PARSE_CLIENT_KEY
```

If you want to use your own backend Parse server, run the following commands instead:
  
```
phonegap local plugin add https://github.com/niraj-shah/phonegap-parse-plugin --variable APP_ID=PARSE_APP_ID --variable CLIENT_KEY=PARSE_CLIENT_KEY --variable SERVER=https://your-server.com/app
cordova plugin add https://github.com/niraj-shah/phonegap-parse-plugin --variable APP_ID=PARSE_APP_ID --variable CLIENT_KEY=PARSE_CLIENT_KEY --variable SERVER=https://your-server.com/app
```

Remember to point the ```SERVER``` to the location of your Open Source Parse Server. If the ```SERVER``` is not specified, the plugin will detault to Parse.com.

Initializing
-------------

A parsePlugin variable is defined globally (e.g. $window.parsePlugin).

Once the device is ready (see: http://docs.phonegap.com/en/4.0.0/cordova_events_events.md.html#deviceready), call ```parsePlugin.initialize()```. This will register the device with Parse, you should see this reflected in your Parse control panel. After this runs you probably want to save the installationID somewhere, and perhaps subscribe the user to a few channels.

(Note: When using Windows Phone, clientKey must be your .NET client key from Parse. So you will need to set this based on platform i.e. if( window.device.platform == "Win32NT"))

To initialize the plugin, and register the device with Parse, you can call:

```
parsePlugin.initialize( appId, clientKey, function( data ) {
  // ...
}, function(e) {
	alert('error');
});
```

If you are using the Open Source Parse Server, the third parameter can be set to your Server URL instead:

```
parsePlugin.initialize( appId, clientKey, serverUrl, function( data ) {
  // ...
}, function(e) {
	alert('error');
});
```


Initial Setup
-------------

An example of subscribing the user to a Channel is shown below:

```
parsePlugin.initialize(appId, clientKey, function() {

	parsePlugin.subscribe('SampleChannel', function() {

		parsePlugin.getInstallationId(function(id) {

			/**
			 * Now you can construct an object and save it to your own services, or Parse, and correlate users to parse installations
			 *
			 var install_data = {
			  	installation_id: id,
			  	channels: ['SampleChannel']
			 }
			 *
			 */

		}, function(e) {
			alert('error');
		});

	}, function(e) {
		alert('error');
	});

}, function(e) {
	alert('error');
});

```

Alternatively, we can store the user in the installation table and use queries to push notifications.

```
// on sign in, add the user pointer to the Installation
parsePlugin.initialize(appId, clientKey, function() {

  parsePlugin.getInstallationObjectId( function(id) {
    // Success! You can now use Parse REST API to modify the Installation
    // see: https://parse.com/docs/rest/guide#objects for more info
    console.log("installation object id: " + id)
  }, function(error) {
    console.error('Error getting installation object id. ' + error);
  });

}, function(e) {
	alert('Error initializing.');
});

```

To receive notification callbacks, on device ready:


```
parsePlugin.registerCallback('onNotification', function() {

  window.onNotification = function(pnObj) {
    alert('We received this push notification: ' + JSON.stringify(pnObj));
    if (pnObj.receivedInForeground === false) {
    	// TODO: route the user to the uri in pnObj
    }
  };

}, function(error) {
  console.error(error);
});

```

Usage
-----
```
<script type="text/javascript">
    // when using parse.com
	parsePlugin.initialize(appId, clientKey, function() {
		alert('success');
	}, function(e) {
		alert('error');
	});
	
	// when using open source parse server
	parsePlugin.initialize(appId, clientKey, serverUrl, function() {
		alert('success');
	}, function(e) {
		alert('error');
	});
  
    // get installation ID
	parsePlugin.getInstallationId(function(id) {
		alert(id);
	}, function(e) {
		alert('error');
	});

    // get user's subscribed channels
	parsePlugin.getSubscriptions(function(subscriptions) {
		alert(subscriptions);
	}, function(e) {
		alert('error');
	});

    // subscribe user to a channel
	parsePlugin.subscribe('SampleChannel', function() {
		alert('OK');
	}, function(e) {
		alert('error');
	});

    // unsubscribe user from a channel
	parsePlugin.unsubscribe('SampleChannel', function(msg) {
		alert('OK');
	}, function(e) {
		alert('error');
	});

    // reset iOS badge count
	parsePlugin.resetBadge(function() {
    alert('OK');
  }, function(e) {
    alert('error');
  });

    // track event
	parsePlugin.trackEvent(function(name, dimensions) {
    alert('OK');
  }, function(e) {
    alert('error');
  });
</script>
```

Quirks
------

### Android

Parse needs to be initialized once in the `onCreate` method of your application class using the `initializeParseWithApplication` method.

If you don’t have an application class (```App.java```, found in the ```platforms/android/src/my/package/namespace``` folder), you can create one using this template:

```java
package my.package.namespace;

import android.app.Application;
import org.apache.cordova.core.ParsePlugin;

public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        ParsePlugin.initializeParseWithApplication(this);
    }

}
```

And add your application name to `AndroidManifest.xml` in the ```<application>``` tag:

```xml
<application android:name="my.package.namespace.App" ... >...</application>
```

Note: ```my```, ```package``` and ```namespace``` should match the your App ID. For example, if my app has ID: ```com.webniraj.app```, the Application Class  needs to be created in the ```platforms/android/src/com/webniraj/app``` folder. Any occurances of ```my.package.namespace``` in the above code need to be replaced with ```com.webniraj.app```.


Compatibility
-------------
- Phonegap/Cordova > 5.0
- iOS > 8
- Android > 2.3
