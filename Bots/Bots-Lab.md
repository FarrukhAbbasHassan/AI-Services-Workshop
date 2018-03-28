# Bots
Bots are a new type of App. One which fundamentally changes we the way we interact with our devices. Bots will lead to a world where we are using natural language to interact with device and services. Microsoft are investing heavily in bots with the Microsoft Bot Framework and Azure Bot Service. In this talk we'll look at what a bot is, design patterns for bots and the Microsoft Bot Framework which lets you write a single bot that is published to multiple social channels such as Skype, Facebook and Slack.

In this lab you'll be creating your very own chatbot which is publically accessible and that can understand users using Microsoft Cognitive Services.

# Scenario

Here you will be building a Hospital Bed Booking Bot - that helps hospital staff find and allocate free beds to patients.  They will also be to discover which patients are being discharged from the hospital.

## 1 - Create Azure Bot Service

1. Navigate to the [Azure Portal](https://portal.azure.com)
2. Create a new resource, select `AI + Cognitive Services` and `Web App Bot`
3. Choose a unique Bot name and select a new Resource Group called `RG-BotLab`
4. Set the Location to be `West Europe`
5. Within Bot template select `Node.js`
6. Select `Language understanding` template
7. Change the LUIS App location to `West Europe`
8. Change the Application Insights Location to `West Europe`
9. Complete the other required fields

> Wait a few minutes for the provisioning to complete, then open the resource group

10.  Look at the resources that have been provisioned for you:

> | Type | Value |
> | ----- | ----- |
> | Web App Bot | Azure Bot Service registration |
> | App Service | API Endpoint the Azure Bot Service sends messages to/from |
> | Application Insights | Telemetry & error data for the bot |
> | Storage account | Bot state |

## 2 - Create Language understanding model
A basic bot has now been provisioned for you which already has natural language capability - we just need to add the custom intents to it for our scenario.

1. Navigate to the [Luis portal](https://eu.luis.ai)
2. Open the new model that has been provisioned for you

### Adding utterances to an existing intent
3. Open the `Greeting` intent and add as many new utterances for saying welcome you can think of that aren't already added by default eg.
    * Yo
    * Sup
    * Hey

### Adding a custom intent
4. Add a new custom intent, select `Create new Intent` named `FreeBed`
5. Add as many ways of asking for a free hospital bed you can think of eg:
    * Where is there a free bed?
    * Where can I find a free bed?
    * Is there a bed free?

### Add a custom entity
6. Select Entities - `Create new entity`, with an Entity name of `Ward` and type of `List`
7. Add the following values (notice the Recommend values displayed as you enter these):
    * 1a
    * 1b 
    * 1c
    * 2a
    * 2b
    * 2c
8. Go back to Intents - `FreeBed` and enter the following utterances (notice how the `Ward` entity is automatically tagged as you do this)
    * Is there a free bed on ward 1a?
    * Where can I find a free bed on ward 1b?
    * Is there a bed free on ward 2c?

### Add additional custom entity with prebuilt entity (optional step)
9. Select Entities - `Manage prebuilt entities`, select `datetimev2`
10. Add another new intent named `DischargeDate`, add as many ways of asking for patient discharge information eg:
    * Who is being discharged today?
    * Who is going home tomorrow?
    * Which patients are going home next week?
    * Who is due to go home on Friday?
    * Which patients are due to be discharged?
    * What is the date next Saturday?

### Train and Publish the LUIS model
1. Train the model
2. Publish the model - Within the Publish tab, take a note of the resources and keys that have been automatically setup within the Europe Regions you selected when you created the Azure Resource Group.  Here you can see the LUIS API key and app guid within the endpoint url eg:
https://westeurope.api.cognitive.microsoft.com/api/v2.0/apps/{yourappid}?subscription-key={yourapikey}&verbose=true&timezoneOffset=0&q= 

> As part of the template you've used, the Bot has already been wired up to > the LUIS model, you can check this within the App Settings within the Azure portal for Web App Bot.  You'll notice the same keys from following:

> | App setting | Value |
> | ----- | ----- |
> | LuisAppId | {yourappid} |
> | LuisAPIKey | {yourapikey} |
> | LuisAPIHostName | westeurope.api.cognitive.microsoft.com |

## 3 - Adding business logic within the bot using Azure Portal - App Service Editor
Now we need to handle the LUIS intents that will be resolved by our LUIS model in our bot.  This is the code that will get fired when any LUIS intent is detected.

1. Within Azure Portal, Web App Bot, select the Build tab and `Open online editor`
2. Within the App Service Editor - WWWROOT folder select the `app.js` file

### Adding business logic for custom intent
3. Add the logic for the `FreeBed` intent where it states `.matches('<yourIntent>')`:
```js
.matches('FreeBed', (session, args) => {
     // Check to see if we have a Ward entity resolved from the utterance
     var entities = args.entities;
     var ward = builder.EntityRecognizer.findEntity(entities, 'Ward');    
     if (ward) {
        session.send('Searching for a free bed in the \'%s\' ward', ward.entity);
    }
    else {
        session.send('Searching for a free bed in all wards');
    }
    
    // Simulate some background processing and display results
    session.sendTyping();
    setTimeout(function () {
        var randomnumber = Math.floor((Math.random() * 10) + 1); 
        session.send("%s free beds found", randomnumber);
    }, 3000);
})
```

### Adding business logic for DischargeDate custom intent - (optional step)
4. Add the logic for the `DischargeDate` intent where it states `.matches('<yourIntent>')`:
```js
.matches('DischargeDate', (session, args) => {
     // Check to see if we've been passed a time period
     var entities = args.entities;
     var dateEntity = builder.EntityRecognizer.findEntity(entities, 'builtin.datetimeV2.date');
     var dateOptions = { year: 'numeric', month: 'long', day: '2-digit' };

    if (dateEntity) {
        // Always looking for future dates - might get multiple values back if it's ambiguous
        // https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-reference-prebuilt-entities#builtindatetimev2
        var futureDate = new Date(dateEntity.resolution.values.slice(-1)[0]['value']);
        session.send('Searching patients being discharged %s (%s)..', dateEntity.entity, futureDate.toLocaleDateString("en-GB", dateOptions));
    }
    else {
        session.send('Searching patients being discharged this week..');
    }
    
    // Simulate some background lookup and display results
    session.sendTyping();
    setTimeout(function () {
        session.send(":( no patients leaving");
    }, 3000);
})
```

### Testing the business logic
1. Within the Azure portal, Web App Bot, select the Test in Web Chat tab to bring up the WebChat channel test harness.
2. Type the following utterances to check that the correct intent is resolved
    * Hey
    * Are there any free beds?
    * How many free beds are in 1c ward?