---
layout: post
title:  "Experimental dark mode theme for Azure PlantUML diagrams"
date:   2022-12-21 10:45:00 -0600
categories: azure plantuml diagrams
---

This year, I had the opportunity to contribute to the [Azure PlantUML](https://github.com/plantuml-stdlib/Azure-PlantUML) project, which enables "diagrams-as-code" with icons that support the majority of Azure services. PlantUML has solid integration with VS Code and is a great choice for creating high quality diagrams that integrate seamlessly into git repositories. My work involved modernizing the sprite (icon) generation code as well as adding new icons...and throwing out a few old ones. This gave me a deeper level of understanding of how PlantUML works and how it can be customized.

 Recently, I've been looking at introducing basic theme support, especially given the recent trend of "dark mode" themes across the web. In the past, I've played around with solarizing diagrams:

 [![mde csv diagram](/media/2021-03-20/diagram-mde-csv.png "Data protection CSV example")](/media/2021-03-20/diagram-mde-csv.png)

...which was achieved using a lot of overrides that PlantUML offers natively: directly creating rectangles, specifying sprites, colors, fonts, and text placement. Here's an example:

```cli
rectangle "<size:18><b>Key Vault</b>\n<color:#b58900><$KeyVault></color>\n<i>[Premium: HSM-backed]</i>\n\nAAD-based RBAC" <<KeyVault>> as kv
```

Needless to say, while this works fine its hardly a user-friendly approach. I decided to look into extending Azure PlantUML to include the basic themes that are already included in PlantUML core. Here's an example:

```cli
@startuml sample-diagram-1

' USE EXPERIMENTAL THEME BRANCH 👇🏽
!define AzurePuml https://raw.githubusercontent.com/travisnielsen/Azure-PlantUML/themesupport/dist

' IMPORT THEME HERE 👇🏽
!includeurl AzurePuml/themes/blueprint.puml

!includeurl AzurePuml/Analytics/AzureEventHub.puml
!includeurl AzurePuml/Analytics/AzureStreamAnalyticsJob.puml
!includeurl AzurePuml/Databases/AzureCosmosDb.puml

left to right direction

agent "Device Simulator" as devices

AzureEventHub(fareDataEventHub, "Fare Data", "PK: Medallion HackLicense VendorId; 3 TUs", "testing 1234")
note right: this is a note
AzureEventHub(tripDataEventHub, "Trip Data", "PK: Medallion HackLicense VendorId; 3 TUs")
AzureStreamAnalyticsJob(streamAnalytics, "Stream Processing", "6 SUs")
AzureCosmosDb(outputCosmosDb, "Output Database", "1,000 RUs")

devices --> fareDataEventHub : "test arrow"
devices --> tripDataEventHub
fareDataEventHub --> streamAnalytics
tripDataEventHub --> streamAnalytics
streamAnalytics --> outputCosmosDb

@enduml
```

This produces a diagram based on the PlantUML [Blueprint theme](https://github.com/plantuml/plantuml/blob/master/themes/puml-theme-blueprint.puml):

 [![blueprint theme diagram](/media/2022-12-21/diagram-blueprint.png "Sample diagram blueprint theme")](/media/2022-12-21/diagram-blueprint.png)

This was a nice step forward, but I was looking for more control, especially since the level of detail I was looking for involved more layers: services deployed into subnets, which are inside VNETs, which are part of subscriptions, etc... To support this, I needed more flexibility:

* Use a popular color theme that offers multiple UI layers
* Easy colorization Azure icons at the item level
* Make the *technology* field optional --> Its not relevant for certain services like VNETs, subnets, and subscriptions.

I ended up creating my own [custom theme](https://github.com/travisnielsen/Azure-PlantUML/blob/themesupport/dist/themes/dracula.puml) based the [Dracula](https://github.com/dracula/dracula-theme) color palette. The Dracula theme is found across the web as well as most popular IDEs and offered the kinds of UI layers I needed. As a result, I'm able to produce better looking diagrams that fit nicely into sites that support dark-mode themes like GitHub.

Example: [Island Network Outbound Flow](https://github.com/travisnielsen/azure-island-networking/blob/main/docs/diagram-outbound-flow.puml)

 [![Example Dracula theme diagram](/media/2022-12-21/diagram-outbound-flow.svg "Example Dracula theme diagram")](/media/2022-12-21/diagram-outbound-flow.svg)

In addition to the basic theme itself, I extended the PlantUML Azure entity files to include an additional (optional) 5th parameter for specifying the icon color.

```cli
' COLOR OPTIONS WITH THE DRACULA THEME:
' BACKGROUND, SELECTION, FOREGROUND, COMMENT
' CYAN, GREEN, ORANGE, PINK, PURPLE, RED, YELLOW

AzureFirewall(azfw, "Hub Firewall", " Basic SKU ", "Deny egress (default)", RED)
```

Since most cloud workloads are deployed within one or more platform-specific containers (regions, subscriptions, networks, etc...), the *technology* and *description* fields don't always make sense. However, the icon color must be the 5th parameter. To work around this, I added support for each field to be nullable.

```cli
AzureSubscription(subscriptionIsland, "Business Unit A", null, null, YELLOW) 
```

This provides the icon control I was looking for while keeping the diagram output free of unwanted entity fields.

There are still many quirks with controlling layout in in PlantUML, but I'm happy with this first attempt at supporting better looking diagrams through this initial (experimental) addition of component diagram theme support for Azure PlantUML.
