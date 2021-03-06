ConfigurableOrderPipelineProcessor
====================================

Wish you could change *at runtime* the PCF files that your Commerce Server 2009 Operation Sequence runs?  This would let you have **normal** and **debug** PCFs (for those times) or **awesome** and **plain** PCFs.  You could even have different PCFs for different client types. 

## Installation

Install using the [Enticify.Cs2009.Components Nuget](http://nuget.org/packages/Enticify.Cs2009.Components/).  Type the following at the Package Manager Console:

    PM> Install-Package Enticify.Cs2009.Components 

## Configuration

Replace all ChannelConfiguration.config uses of OrderPipelineProcessor with ConfigurableOrderPipelineProcessor.  You just need to change the **type** attribute...:  

```xml
<!-- From this type ... -->
<Component name="Order Pipelines Processor" type="Microsoft.Commerce.Providers.Components.OrderPipelinesProcessor, Microsoft.Commerce.Providers, Version=1.0.0.0, Culture=neutral,PublicKeyToken=31bf3856ad364e35">
...
</Component>
<!-- To this type ... -->
<Component name="Order Pipelines Processor" type="Enticify.Cs2009.Components.ConfigurableOrderPipelinesProcessor, Enticify.Cs2009.Components, Version=0.1.0.0, Culture=neutral, PublicKeyToken=10ff57ed14d5fefa">
  ...
</Component>
```

## Example: Specifying pipelines to run using PipelineListConfig 

The PipelineListConfig lets you specify a list of pipelines to run.

### Configuration

*  Add all the pipelines you need to the **pipelines** ChannelConfiguration.config.

### Usage

*  Add a reference to **Enticify.Cs2009.Components**.
*  Set up your Basket query as normal.  E.g.:

```csharp
var basketQuery = new CommerceQuery<Basket>();
```
*  Configure the **Model** to use the PipelineListConfig:

```csharp
var config = new PipelineListConfig(){new PipelineConfigurationElementData("basket", OrderPipelineType.Basket)};  
ConfigurableOrderPipelinesProcessor.SetRuntimePipelineConfig(bq.Model, config);
```
*  You're done.

## Example: Adding a secondary basket pipeline using SuffixConfig 

The SuffixConfig lets you specify that you want the same config as in ChannelConfiguration.config, but with a suffix removed from the pipeline names that have it.

### Configuration

*  Choose a suffix for your **Primary** pipelines.  I'm using **_topbanana**.
*  Make a copy of basket.pcf and add your suffix (e.g. basket_topbanana.pcf).
*  basket.pcf is now your *secondary* pipeline so change it as you want (e.g. add debug stuff).
*  Add your new pipeline to the ChannelConfiguration.config pipelines section:  
```xml
<pipelines>
    <!-- Basket now has a primary and secondary -->
    <pipeline name="basket_topbanana" path="basket_banana.PCF" type="OrderPipeline" />
    <pipeline name="basket" path="basket.pcf" type="OrderPipeline" />

    <!-- Total is fine without a secondary -->
    <pipeline name="total" path="total.pcf" type="OrderPipeline" />
</pipelines>
```
*  Update the **Pipeline name** to your primary name:  
```xml
<Component name="Order Pipelines Processor" type="Enticify.Cs2009.Components.ConfigurableOrderPipelinesProcessor, Enticify.Cs2009.Components, Version=0.1.0.0, Culture=neutral, PublicKeyToken=10ff57ed14d5fefa">
  <Configuration
    customElementName="OrderPipelinesProcessorConfiguration"
    customElementType="Microsoft.Commerce.Providers.Components.OrderPipelinesProcessorConfiguration, Microsoft.Commerce.Providers, Version=1.0.0.0, Culture=neutral,PublicKeyToken=31bf3856ad364e35">
    <OrderPipelinesProcessorConfiguration>
      <OrderPipelines>
        <!-- Look Ma, I'm a primary pipeline! -->
        <Pipeline name="basket_topbanana" type="Basket"/>
        <Pipeline name="total" type="Total"/>
      </OrderPipelines>
    </OrderPipelinesProcessorConfiguration>
  </Configuration>
</Component>
```

### Usage - how to switch at runtime

*  Add a reference to **Enticify.Cs2009.Components**.
*  Set up your Basket query.  E.g.:  

```csharp
var basketQuery = new CommerceQuery<Basket>();
```

*  Configure the **Model** to use the SuffixConfig:  

```csharp
ConfigurableOrderPipelinesProcessor.SetRuntimePipelineConfig(bq.Model, SuffixConfig(){Suffix = "_topbanana"});
```

*  You're done.

## How is this happening?

It's very simple.  Look [here](https://github.com/enticify/Enticify.Cs2009.Components/blob/master/src/Enticify.Cs2009.Components/ConfigurableOrderPipelinesProcessor.cs) and [here](https://github.com/enticify/Enticify.Cs2009.Components/blob/master/src/Enticify.Cs2009.Components/RuntimeOrderPipelinesProcessorConfiguration.cs).

##  Tests

The two tests are written in F#.  I also have integration tests that work for me.  Unfortunately, they are currently in the private [Enticify repos](http://www.enticify.com/).

## Why Dude?

To help my lovely customers switch between the Enticify and CS promotion components so they can see the awesome difference :)

## Release Notes

### 0.8.0

* Re-designed the configuration options.  Committed some DTO serialization sins that may change for 3-tier deployments.

### 0.5.0

* Fix: Removed unecessary overrides of specialised CommerceOperation Execute* methods.
* Change: Made configuration methods virtual protected.

