# Attribution Reporting API - App-to-Web implementation guide

---
* [Overview](#overview)
* [Register sources with Android OS](#register-sources-with-android-os)
* [Register an app source and web trigger](#register-an-app-source-and-web-trigger)
* [Register a web source and an app trigger](#register-a-web-source-and-an-app-trigger)
* [Campaigns that have both app and web potential destinations](#campaigns-that-have-both-app-and-web-potential-destinations)
* [For apps using Chrome Custom Tabs](for-apps-using-chrome-custom-tabs)
* [For apps using WebView](#for-apps-using-webView)


## Overview

The Attribution Reporting API enables cross app and web attribution for sources and triggers that occur on the same device. Browsers, such as Chrome, can delegate both source and trigger registrations to the Attribution Reporting API for Android instead of handling those registrations locally in the browser. This allows Android to match sources and triggers across both sites and apps.

This guide will teach you how to set up [cross app and web attribution](https://developers.google.com/privacy-sandbox/relevance/attribution-reporting/android/attribution-app-to-web).

## Register sources with Android OS

Cross app and web attribution will only be available if the attribution API is enabled in both the browser and the Android OS on the same device. The availability of the Android attribution reporting API is sent via the Attribution-Reporting-Support header. This header will return os, web, or both, depending on what is available on that device.

The ad tech will need to decide whether to register the source or trigger with the browser or the OS.

- For web only campaigns, ad techs can still register both sources and triggers with Chrome’s attribution reporting API or choose to delegate both to the OS.
- For sources that may result in either an app or a web trigger, the ad tech can choose to delegate web source registration to the Android attribution reporting API.
- Ad techs should avoid registering the source with both the Chrome and Android APIs simultaneously in order to avoid creating duplicate attribution reports.
- For triggers that may have been driven by app based sources, the ad tech can choose to delegate web trigger registration to the Android attribution reporting API.

## Register an app source and web trigger

For some campaigns, the source may occur in an app while the trigger would occur on a website in the mobile browser on the same device.

### Example

A user is reading articles in their favorite news app. They see an ad for cheap flights to Paris and excitedly click to book. The ad tech serving the ad in the news app registers the click source with the Android attribution reporting API. The user is taken to the advertiser’s web page in Chrome where they are able to convert. The ad tech on the advertiser’s site checks if the OS level API is available, and it is. The ad tech registers the conversion trigger by instructing Chrome to delegate the registration to the OS instead of registering it directly with Chrome’s attribution reporting API. The OS-level attribution reporting API is then able to match the app source and web trigger and send out the relevant reports.

### Workflow

Following are further details on how to complete each step:

1. The ad tech from the app registers a source with Android’s attribution reporting API with the following adjustments:
    - To register an app source that is expected to convert on a web site, the Attribution-Reporting-Register-Source response header should include a web destination (eTLD+1) instead of an app destination.

```
Attribution-Reporting-Register-Source: {

  "web_destination": "<https://advertiser.example>",

  "source_event_id": "789",

  "expiry": "120000",

  "priority": "2"

}
```

**Note:** For campaigns that may convert in either a web OR an app destination, see the section on [dual destinations](#2jxsxqh)

- - In some cases, the advertiser may be using multiple measurement providers (e.g., a third-party measurement tool or an analytics tool) via 302 redirect chains. In some cases, the Attribution Reporting API will follow the redirect path specified in the Attribution-Reporting-Redirect header in the background and at the same time the 302 redirect path executes in the foreground for existing navigation requests. These requests will go to the same URL and could result in the third-party measurement provider double counting registrations. In such cases, ad techs can modify redirection behavior to send the Attribution Reporting API registration to an alternative yet deterministic URL.
        - To enable this behavior, ad techs need to include a new HTTP header when responding to a registration request:
            - The header is Attribution-Reporting-Redirect-Config.
            - The header’s value should be redirect-302-to-well-known.

```
"**Attribution-Reporting-Redirect-Config**": "redirect-302-to-well-known"
```

**Note:** All subsequent redirects following this header will have .well-known appended to the beginning of the redirect URI’s path.

- - The rest of the source registration process is the same as a standard app-to-app [source registration](https://developer.android.com/design-for-safety/privacy-sandbox/guides/attribution#register-attribution).
    - Note: If you or the publishing app are using webview to show a native ad in an Android app, refer to the section on [webview](#3j2qqm3).

1. The ad tech registers the trigger on the advertiser’s website by asking Chrome to delegate the registration to the Android attribution reporting API:
    - Once a user converts on a web site, the ad tech will make a request to register the trigger with Chrome.
        - For an app to web use case, the attribution source parameter must be specified directly, either by using the attributionsrc tag or via javascript registration.
        - Below is an example of using the attributionsrc tag to specify the source parameter:

```
<img src="<https://adtech.example/conversionpixel>"
  attributionsrc="<https://adtech.example/register-trigger?purchase=12">>
```

- - - The Attribution-Reporting-Support request header is returned by Chrome to the ad tech. If the API is enabled on both the Chrome browser and the Android device, the header will return ‘os, web’.
            - Note: The attribution reporting API must be enabled on the Android OS by [configuring AdServices](https://developers.google.com/privacy-sandbox/relevance/setup/android/setup-api-access) and [your device](https://developers.google.com/privacy-sandbox/relevance/setup/android/setup-device-access) to successfully measure across app and web.

```
"**Attribution-Reporting-Support**": "os, web"
```

- - The ad tech should then tell Chrome to delegate to the OS via the Attribution-Reporting-Register-OS-Trigger header which 1) tells Chrome to delegate the registration to the OS, and 2) tells the Android attribution reporting API to initiate a secondary API call from Android to the ad tech URI.

"**Attribution-Reporting-Register-OS-Trigger**": "<https://adtech.example/register-trigger>", "<https://other-adtech.example/register-trigger>"

- - To complete the trigger registration, the ad tech’s endpoint should respond to the Android attribution reporting API request via the response header.

```
Attribution-Reporting-Register-Trigger {

  "event_trigger_data": \[{"trigger_data":"1"}\],

  "aggregatable_trigger_data": \[

    {"key_piece":"0x400","source_keys":\["campaignCounts"\]},

    {"key_piece":"0xA80","source_keys":\["geoValue"\]}

  \],

  "aggregatable_values": {"campaignCounts":32768,"geoValue":1664},

  "debug_key": 12345

  "debug_reporting": true

}
```

- - The remainder of the [trigger registration](https://developer.android.com/design-for-safety/privacy-sandbox/guides/attribution#accept-trigger-registration) remains the same.

## Register a web source and an app trigger

For some campaigns, a source may occur on a site in a mobile browser while the trigger occurs in an app on the same device.

### Example

A user is browsing on a site in their Chrome browser on their Android phone. They see an ad for a sweater from one of their favorite stores. They click the ad and are taken to the app they already have downloaded. The ad tech on the website where the ad was served registers the click source by instructing Chrome to delegate the registration to the Android attribution reporting API instead of using Chrome’s native attribution reporting API. The user purchases the sweater in the shopping app. The ad tech in the advertiser’s app then registers the conversion trigger with the Android attribution reporting API. The OS-level attribution reporting API is able to match the web source and app trigger and send out the relevant reports.

### Workflow

Following are further details on how to complete each step:

1. The ad tech on the publisher website registers the source by instructing Chrome to delegate the registration to the Android attribution reporting API:
    - The Attribution-Reporting-Support request header is returned by Chrome to the ad tech. If the API is enabled on both the Chrome browser and the Android device, the header will return ‘os, web’.

```  
"**Attribution-Reporting-Support**": "os, web"
```

**Note:** The attribution reporting API must be enabled on the Android OS by [configuring AdServices](https://developers.google.com/privacy-sandbox/relevance/setup/android/setup-api-access) and [your device](https://developers.google.com/privacy-sandbox/relevance/setup/android/setup-device-access) to successfully measure across app and web.

- - The ad tech should tell Chrome to delegate to the OS-level API via the Attribution-Reporting-Register-OS-Source header which 1) tells Chrome to delegate to the OS, and 2) tells the Android attribution reporting API to initiate a secondary API call from Android to the ad tech URI.

```
"**Attribution-Reporting-Register-OS-Source**": "<https://adtech.example/register-source>"
```

- - To complete the source registration, the ad tech’s endpoint should respond to the Android attribution reporting API request with the response header Attribution-Reporting-Register-Source. The response should also specify an app destination in the destination field.

```
Attribution-Reporting-Register-Source {
  "source_event_id":"123001",
  "destination":"android-app://com.example.advertiser",
```  

- - - To support redirects for source registrations, Chrome will follow the redirects and call the [web context APIs](https://developer.android.com/design-for-safety/privacy-sandbox/attribution-app-to-web#changes-apps) for each redirect hop.
    - The remainder of the [source registration](https://developer.android.com/design-for-safety/privacy-sandbox/guides/attribution#register-attribution) remains the same.

1. The ad tech in the advertiser’s app registers a trigger with the Android attribution reporting API:
    - For triggers that occur in apps, the apps [register triggers](https://developer.android.com/design-for-safety/privacy-sandbox/attribution#register-trigger) with the Android attribution reporting API as normal.

## Campaigns that have both app and web potential destinations

- Setting up dual destinations
  - Some campaigns may be set up to convert in either the advertiser’s app or on the advertiser’s web page depending on various factors such as if the user has the app installed.
  - In these cases, both an app and web destination can be specified in the respective parameters
    - App destination should be in the “destination” field
    - Web destination should be in the “web_destination” field
    
```    
Attribution-Reporting-Register-Source {

  "source_event_id":"123001",

  "destination":"android-app://com.example.advertiser",

  "web_destination": "<https://example.advertiser>"

```

- - The next section on coarse reporting will explain how using dual destinations may impact the noise in your reports.
- Using coarse reporting to reduce noise in event-level reports for dual destination sources:
  - If both an OS and a Web destination were specified in the source registration, event-level reports will specify whether the trigger happened in a web destination or app destination by default. However, to maintain privacy limits, additional noise will be added to these reports.
  - Ad techs can use the coarse_event_report_destinations field under the Attribution-Reporting-Register-Source header to turn on coarse reporting and reduce noise. If a source with the coarse_event_report_destinations field specified wins attribution, the resulting report includes both the app and web destinations without distinction as to where the actual trigger occurred but with less noise than reports where the app or web destination is specified.
  - Aggregate reports remain unchanged.

## For apps using Chrome Custom Tabs

Some apps may use [Custom Tabs](https://developer.chrome.com/docs/android/custom-tabs) to render web content. Custom tabs behave similarly to a regular web page when measuring across apps and mobile web sites.

- Register an app source and Custom Tab trigger:
  - Follow the instructions to [register an app source and web trigger](#4d34og8)
- Register a Custom Tab source and app trigger
  - Follow the above instructions to [register a web source and app trigger](#26in1rg)
- Register a CCT source and CCT trigger
  - This is treated the same as any [site-to-site web attribution](https://developers.google.com/privacy-sandbox/relevance/attribution-reporting/dev-guide) in Chrome

## For apps using WebView

Some apps may use WebView to display content. There are a variety of use cases for WebView, such as rendering ads, hosting web content, or custom app functionality better suited to a web format.

- Only OS-level attribution is available in WebView. The Attribution-Reporting-Support header will return only os, and only if the Android attribution reporting API is available.
  - Note: in order to do attribution across Chrome and WebView, source and trigger registrations in Chrome will need to be delegated to the OS as only OS based attribution is supported for webview.
- registerWebSource differs from registerSource in that registerWebSource determines that the registration is associated with the URL inside the WebView, whereas registerSource determines that the registration is associated with the app where the WebView is rendered. registerWebTrigger and registerTrigger have the same distinction. Whether the WebView uses registerSource or registerWebSource and registerTrigger or registerWebTrigger is set by the app rendering the WebView and is determined on a per WebView basis. Below is a table that outlines different scenarios where a developer would want to configure the different APIs.
- Note: Developers do not need to call registerWebSource and registerWebTrigger directly; instead they will configure whether the OS delegates to registerSource or registerWebSource, and registerTrigger or registerWebTrigger.
- By default, WebView will use registerSource and registerWebTrigger when calling the Android attribution reporting API. This associates sources with the app and triggers with the top-level origin of the WebView when the trigger occurs.
  - If an app requires different behavior, they will need to use a new method setAttributionRegistrationBehavior on the androidx.webkit.WebViewSettingsCompat class. This method will specify whether WebView should call registerWebSource() or registerWebTrigger() rather than registerSource() or registerTrigger().
    - This behavior will need to be set for each WebView that is initiated.
    - If the ad tech SDK is initiating the WebView, the SDK will need to set this default behavior.
    - For apps that would like to use registerWebSource() to associate source registrations with the website in WebView instead of the app, they must join the WebApp allowlist. [Complete this form](https://docs.google.com/forms/d/e/1FAIpQLSeGzZBnytpn-aGZBjCoGV5PDo69QRoBPGcUftTlOgDRl4xrGw/viewform) to join the allowlist. The intent of the allowlist is to mitigate privacy considerations around [establishing trust for web context](https://developers.google.com/privacy-sandbox/relevance/attribution-reporting/android/attribution-app-to-web#establish-trust-web-context).
  - Options for [setAttributionRegistrationBehavior](https://developers.google.com/privacy-sandbox/relevance/attribution-reporting/android/attribution-app-to-web#attribution_source_and_trigger_registration_from_webview)

| **Value** | **Description** | **Example use case** |
| --- | --- | --- |
| APP_SOURCE_AND_WEB_TRIGGER (default) | Allows apps to register app sources (sources associated with the app package name) and web triggers (triggers associated with the eTLD+1) from WebView. | Apps that use WebView to serve ads rather than enable web browsing |
| --- | --- | --- |
| WEB_SOURCE_AND_WEB_TRIGGER | Allows apps to register web sources and web triggers from WebView. | WebView-based browser apps, where ad impressions and conversions could both happen on websites in WebView. |
| --- | --- | --- |
| APP_SOURCE_AND_APP_TRIGGER | Allows apps to register app sources and app triggers from WebView. | WebView-based apps where ad impressions and conversions should always be associated with the app rather than the eTLD+1 of the WebView. |
| --- | --- | --- |
| DISABLED | Disables source and trigger registration from WebView.<br><br>Note that the initial network call to the Attribution Source or Trigger URIs may still happen, but any response is discarded and nothing will be stored on the device. |     |
| --- | --- | --- |

- Source and trigger registrations from WebView
  - Ad techs should respond to source registrations using the Attribution-Reporting-Register-OS-Source header. Based on the set behavior for the WebView, this will either registerSource() or registerWebSource() with the OS and initiate a secondary API call from the Android attribution reporting API to the ad tech URI.
    - To complete the source registration, the ad tech’s endpoint should respond to the Android attribution reporting API request with the response header.

```
Attribution-Reporting-Register-OS-Source {

  "source_event_id":"123001",

  "destination":"android-app://com.example.advertiser",
```

- - - The remainder of the [source registration](https://developer.android.com/design-for-safety/privacy-sandbox/guides/attribution#register-attribution) remains the same.
    - Ad techs should respond to trigger registrations using the Attribution-Reporting-Register-OS-Trigger header. Based on the set behavior for the WebView, this will either registerTrigger() or registerWebTrigger() with the OS and initiate a secondary API call from Rb to the adtech URI.
      - To complete the trigger registration, the ad tech’s endpoint should respond to the Android attribution reporting API request with the response header.

```
Attribution-Reporting-Register-OS-Trigger {

  "event_trigger_data": \[{"trigger_data":"1"}\],

  "aggregatable_trigger_data": \[

    {"key_piece":"0x400","source_keys":\["campaignCounts"\]},

    {"key_piece":"0xA80","source_keys":\["geoValue"\]}

  \],

  "aggregatable_values": {"campaignCounts":32768,"geoValue":1664},

  "debug_key": 12345

  "debug_reporting": true

}
```

- - - The remainder of the [trigger registration](https://developer.android.com/design-for-safety/privacy-sandbox/guides/attribution#accept-trigger-registration) remains the same.