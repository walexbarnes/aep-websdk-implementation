# aep-websdk-implementation
Let's implement Adobe Analytics and Adobe Target with the Adobe Experience Platform Web SDK + RT CDP. WITH JUST ONE LOAD RULE. And, let's integrate it with AEM's OOTB data layer. 

# Setting the Stage
Picture an image of the Adobe Debugger pulled up for a site that has Adobe Launch, Adobe Target, and Adobe Analytics. Are you picturing something like this? 

[insert]

What if I told you that actually, this website was sending data to both Adobe Analytics and Adobe Launch? What if I told you it was doing it with only ONE load rule in Launch? Would you believe me? 

[insert]

# What is Going On?
The site in the first screenshot is doing things traditionally. Client side deployment of AppMeasurement.js, at.js, and the Visitor API to deploy Adobe Analytics, Adobe Target, and ECID. 

[insert]

The site in the second screenshot is doing things the new way. One JavaScript file takes care of the deployment of all these technologies, alloy.js, by sending calls to Adobe's Edge Network and then to the relevant Adobe solution.

[insert]

# Why Would I Do It Like This? 

There are several benefits, even if you don't have Adobe Customer Journey Analytics, Customer Journey Orchestration, etc. 

[insert]

# How Does It Work? 

Differently. AppMeasurement does not exist. You aren't calling any s.t()'s nor s.tl()'s. You aren't even sending any data directly to Analytics. You send it via Adobe's XDM schema to Adobe's Edge Network. 

This schema is configured in Adobe Experience Platform's Real Time CDP, so we will start there. 

## Adobe Experience Platform 

### Schema Creation 
XDM Schema is a taxononmy for data collection, not altogether dissimilar from a JSON taxonomy. You define the taxonomy and key value pairs of the fields you want to collect. 

Adobe has predefined schemas we can leverage, one of which being an Adobe Analytics Field Group, whose structure will look familiar to traditionalists.

[insert]


### Dataset Creation 
Now that we have a schema, we have to create a dataset for that schema so we have somewhere to store its contents. 

[insert]

### Datastream Configuration
Last, we need to set up a datastream so that when the Web SDK's data hits Adobe's Edge Network, it knows what Adobe solutions to route it to and how to do it. 
We set up an Analytics component to route the data to the relevant report suite. 
We set up a Target component and include an authenticated namespace for future personalization. 
We set up an AEP component and provide the relevant dataset for ingestion. 

[insert]

## Adobe Launch

### Extensions and Data Elements
With all of that preamble, now we can go to Adobe Launch. 

First, take a look at the extensions we have installed. Notice what isn't there? No Adobe Analytics, ECID, nor Target. 

[insert]

But there is a Web SDK extension. Let's take a look at that. Remember those datastreams we created? You will need them here. 

[insert]

Now configured, this Web SDK extension will recognize our datastream and allow us to reference our XDM schema we created at the start in a data element. 
We populate its individual components with other data elements, presumably from our data layer. 
For example, I am placing the data element for page name in the field for eVar3. 

Traditionally, XDM schema payloads ingest into Analytics as context data, and would need to be mapped in processing rules. 
However, the Adobe Analytics OOTB schema will automatically populate the associated variable of its field. 
I do not have to process eVar3 in processing rules as context data. It will populate the variable automatically. 

[insert]

###Load Rule

Now that we have populated our schema, let's tell Adobe Launch when to deliver it to the Edge Network via a load rule. 

**This entire implementation deploys Adobe Target, Adobe Analytics (all links and page views), ECID, etc. with just one load rule!
**

This is what that load rule looks like. 

[insert]

Yes, it really is that easy. 

This site was built in AEM, which comes with an out of the box integration between [Adobe Launch and AEM Core Components](https://experienceleague.adobe.com/docs/experience-manager-learn/sites/integrations/experience-platform-data-collection-tags/overview.html?lang=en)

With this integration, AEM will automatically deploy the Adobe Client Data Layer, deliver events for page views and clicks, and detail metadata associated with those clicks (think built in activity map tracking). 

The triggering criteria is set up to listen for any pushes made to Adobe's data layer (OOTB page load events, OOTB clicks, or custom pushes) and trigger the load rule when a relevant event occurs. 
Furthermore, by passing the event object as an argument to the trigger(), we can reference that event's context outside of this load rule. 

[insert]

This code will trigger the load rule when the OOTB integration with AEM deploys the cmp:show or cmp:click events to the data layer. 

Now, we just have to deliver that information to Adobe's Edge Network using our XDM schema. 

<img width="532" alt="13_action_deliver" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/fcc362d7-1a80-4df0-819b-790d0e009940">




