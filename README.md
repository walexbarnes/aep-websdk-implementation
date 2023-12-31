# aep-websdk-implementation
Let's implement Adobe Analytics and Adobe Target with the Adobe Experience Platform Web SDK + RT CDP. WITH JUST ONE LOAD RULE. And, let's integrate it with AEM's OOTB data layer. 

# Setting the Stage
Picture an image of the Adobe Debugger pulled up for a site that has Adobe Launch, Adobe Target, and Adobe Analytics. Are you picturing something like this? 

<img width="451" alt="1_Corporate_Debugger" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/ada28586-87e3-434d-98f0-db6351463c54">

What if I told you that actually, this website was sending data to both Adobe Analytics and Adobe Launch? What if I told you it was doing it with only ONE load rule in Launch? Would you believe me? 

<img width="576" alt="2_wlc_debugger" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/6b048fad-3a20-49d4-b14b-3bd6b85cf510">

# What is Going On?
The site in the first screenshot is doing things traditionally. Client side deployment of AppMeasurement.js, at.js, and the Visitor API to deploy Adobe Analytics, Adobe Target, and ECID. 

<img width="425" alt="3_trad_collection" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/5d6ab138-a312-4592-b9fb-7f90c979503b">


The site in the second screenshot is doing things the new way. One JavaScript file takes care of the deployment of all these technologies, alloy.js, by sending calls to Adobe's Edge Network and then to the relevant Adobe solution.

<img width="458" alt="3_new_collection" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/28ee8649-230e-448d-983b-c9460f3e0802">


# Why Would I Do It Like This? 

There are several benefits, even if you don't have Adobe Customer Journey Analytics, Customer Journey Orchestration, etc. 

<img width="1265" alt="4_benefits" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/c8d970b0-e5d7-4bcb-975d-0293a8f9d7a5">


# How Does It Work? 

Differently. AppMeasurement does not exist. You aren't calling any s.t()'s nor s.tl()'s. You aren't even sending any data directly to Analytics. You send it via Adobe's XDM schema to Adobe's Edge Network. 

This schema is configured in Adobe Experience Platform's Real Time CDP, so we will start there. 

## Adobe Experience Platform 

### Schema Creation 
XDM Schema is a taxononmy for data collection, not altogether dissimilar from a JSON taxonomy. You define the taxonomy and key value pairs of the fields you want to collect. 

Adobe has predefined schemas we can leverage, one of which being an Adobe Analytics Field Group, whose structure will look familiar to traditionalists.

<img width="251" alt="5_xdm_analytics" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/f75729f5-25bc-4529-812b-984027c2e6c3">



### Dataset Creation 
Now that we have a schema, we have to create a dataset for that schema so we have somewhere to store its contents. 

<img width="343" alt="6_dataset" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/580f6a09-555e-4340-b636-37db955c8ea3">


### Datastream Configuration
Last, we need to set up a datastream so that when the Web SDK's data hits Adobe's Edge Network, it knows what Adobe solutions to route it to and how to do it. 
We set up an Analytics component to route the data to the relevant report suite. 
We set up a Target component and include an authenticated namespace for future personalization. 
We set up an AEP component and provide the relevant dataset for ingestion. 

<img width="719" alt="7_datastream" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/31d8f9a5-2d77-4793-aec4-1f2d54e05f73">


## Adobe Launch

### Extensions and Data Elements
With all of that preamble, now we can go to Adobe Launch. 

First, take a look at the extensions we have installed. Notice what isn't there? No Adobe Analytics, ECID, nor Target. 

<img width="776" alt="8_extensions" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/ac98130e-92c5-4648-923e-41a49e738a7c">


But there is a Web SDK extension. Let's take a look at that. Remember those datastreams we created? You will need them here. 

<img width="374" alt="9_WebSDK" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/6ae1efeb-a538-420f-8254-3b36afab2c37">


Now configured, this Web SDK extension will recognize our datastream and allow us to reference our XDM schema we created at the start in a data element. 
We populate its individual components with other data elements, presumably from our data layer. 
For example, I am placing the data element for page name in the field for eVar3. 

Traditionally, XDM schema payloads ingest into Analytics as context data, and would need to be mapped in processing rules. 
However, the Adobe Analytics OOTB schema will automatically populate the associated variable of its field. 
I do not have to process eVar3 in processing rules as context data. It will populate the variable automatically. 

<img width="829" alt="10_xdm" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/93b28b0e-9e25-4649-9633-653a1243d00c">


### Load Rule

Now that we have populated our schema, let's tell Adobe Launch when to deliver it to the Edge Network via a load rule. 

**This entire implementation deploys Adobe Target, Adobe Analytics (all links and page views), ECID, etc. with just one load rule!**

This is what that load rule looks like. 

![11_single_load_rule](https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/d318d82d-0259-46cc-b29d-215dc9f01b59)


Yes, it really is that easy. 

This site was built in AEM, which comes with an out of the box integration between [Adobe Launch and AEM Core Components](https://experienceleague.adobe.com/docs/experience-manager-learn/sites/integrations/experience-platform-data-collection-tags/overview.html?lang=en)

With this integration, AEM will automatically deploy the Adobe Client Data Layer, deliver events for page views and clicks, and detail metadata associated with those clicks (think built in activity map tracking). 

The triggering criteria is set up to listen for any pushes made to Adobe's data layer (OOTB page load events, OOTB clicks, or custom pushes) and trigger the load rule when a relevant event occurs. 
Furthermore, by passing the event object as an argument to the trigger(), we can reference that event's context outside of this load rule. 

<img width="704" alt="12_trigger_criteria" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/d10abed1-549f-43b6-96fa-6ce85d44dc8a">


This code will trigger the load rule when the OOTB integration with AEM deploys the cmp:show or cmp:click events to the data layer. 

Now, we just have to deliver that information to Adobe's Edge Network using our XDM schema. 

<img width="532" alt="13_action_deliver" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/fcc362d7-1a80-4df0-819b-790d0e009940">

The XDM Prod data element is the XDM data element we referenced earlier. But what is mappingEventType? 

That is a data element that will tell the Web SDK if this is an s.t() call or an s.tl() call, basically - if you are coming from the "old way". How does it work? 

<img width="683" alt="mappingtable" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/a15273e9-1982-40d0-8781-06de6e444265">


This data element is a lookup table. It references the event payload from the load rule's triggering criteria (that is why we pass it through as an argument). If that event name is a page view, we will increment page views. If it is not, we will increment link clicks. 


# Debugging

With the hard work behind us, let's take a look at what is happening on the site. 

Remember. Target, Analytics, ECID are not on the page. However, if I log into the AEP Debugger, I can see what we are doing with the Web SDK payload once it hits Edge network. 

<img width="259" alt="14_payload" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/8ca373b1-2dab-4fe4-8e82-531a7003d95f">

We see it delivering the information to Adobe Analytics and deploying Target. If we look a little closer, we can inspect the schema to ensure data collection is taking place properly. 

<img width="950" alt="15_detailedpayload" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/4425300f-ad43-4362-ad58-d4bcb3ee39bc">

You know what else is neat? We can also verify this data in AEP. 

<img width="997" alt="16_data_aep" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/21db6ee6-5704-4e70-bc96-19f668eb0980">

And, I never mentioned this, but if you are populating the identity map field of the schema, when a user authenticates, their profile will be updated in Platform's Real Time CDP and linked accordingly with disparate data sources. 

<img width="649" alt="17_identities" src="https://github.com/walexbarnes/aep-websdk-implementation/assets/59946143/3d09313b-99da-4ea9-8975-55d4227fb85d">


