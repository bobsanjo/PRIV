# Key/Value Service Use Cases

---
* [Overview](#overview)
* [Use Cases](#use-cases)
* [More Information](#more-information)

## Overview

The Key/Value (KV) server is a TEE-based Key/Value database that can be used to store and integrate user level and campaign level data into Protected Audiences Auctions. 
It can be used for a variety of use cases, including the ones listed below.

## Use Cases

### Real-time creative status retrieval

SSPs can use the Key/Value server for lookup of a previous scan of a creative so they can block ads that violate their policies. When an SSP discovers that an ad violates their policy, 
they can write to the Key/Value server to indicate that the ad should not be presented. The SSP can then check the creative render URL against the key value in the auction code to 
identify if the ad is "allowed", "blocked", or of "unknown" status. More generally, the Key/Value response for a creative could include a list of policy categories the ad falls into 
according to the scan, meaning the SSP’s auction code could allow the creative on some publishers but not others.

### Budgeting

The Key/Value server can be used to store campaign budget strategy such as {full-speed, slow, off} in near real time. When bidding takes place, an ad can check the budget strategy for the 
ad by doing a lookup of render URL to campaign ID and then using the campaign ID to check for the budget strategy. For example, when the DSP places an ad on the browser, 
it stores "campaign": "#123" in the ad’s metadata in the IG, and also stores "spendrate#123" in the IG’s set of keys. Then, when the `computedBid()` function is looking over the IG’s ads, 
it checks the key "spendrate"+ad.campaign and finds the value.

The ad server can also use geographical information to return a value that is, say, country specific. For example, there might be different budgets or different policies in different countries, 
which will give different return values from the Key/Value server.

### Campaign liveness check

The Key/Value server can check whether an ad-tech has paused or unpaused a campaign.

### Real-time bidding information

The Key/Value server can bring real-time bidding information. For example, if the advertiser is running a target cost-per-acquisition (CPA) campaign, the real-time CPA value used for 
bidding would come from the Key/Value server.

### ML Model Evaluation
The contents of the "key" sent to the server is your choice. It can even be some representation of the input data to an ML model. Using the Key/Value server’s User-Defined Functions, 
you can provide real-time data using both the input data stored in the key and the hostname as input signals.

### Brand Safety
Page URL, IP Geo and other information are sent to Key/Value server. They can then be used in the decision of how an ad should be presented next to a visited site.

### Creative Scanning
The SSP server can get a list of render ad URLs to decide if it should present them next to the content. In case an ad is seen for the first time, a mechanism will be available 
to the SSP for creative scanning.

### Refresh IGs
The KV server can be used to send commands to the browser to refresh IGs earlier than the once-a-day frequency.

## More Information
The Key/Value server is a powerful tool that can be used to improve the efficiency and effectiveness of ad tech campaigns. If you have other use cases that involve real-time data in 
the auction, we can discuss your specific needs and determine the best combination of contextual and K/V requests to meet them. If you have any questions or would like to see 
additional use cases supported, contact us via GitHub or your preferred communication channel.
