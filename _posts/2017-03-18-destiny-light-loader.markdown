---
layout: post
title:  "Destiny Light Loader"
date:   2017-03-16 11:58:00 +0000
categories: experiments
comments: true
---

**tldr;** [destiny-light-loader](https://github.com/alexlukelevy/destiny-light-loader) is an experimental application created to explore Bungie's public Destiny REST API.

### Background
Those who know me closely know that when I'm not programming, I'm most likely in front of a video game. November of 2015 was no different. This time around it was the 'MMO' shooter from Bungie; Destiny. My friend and I picked it up (along with the expansions) during a Black Friday sale and have barely put it down since. I could quite happily write a full length blog post on Destiny alone but that can wait for another time!

During one of our many sessions of playing it online, my friend intimated that he was interested in learning to program. We both agreed it would be cool to write an experimental application around our shared passion for Destiny. We wanted to keep it very simple, but to learn more about how the game was constructed whilst making something useful. For those of you not familiar with the game, you take control of a character who has two categories of items; weaponry and armour. Each of these categories are broken down into four sub-categories (primary weapon, chest armour etc.) into which you can place items of varying value. The higher the level of items you hold, the higher the overall level of your character, however there are certain constraints about the combinations of items you can hold. Our aim with [destiny-light-loader](https://github.com/alexlukelevy/destiny-light-loader) was to create a CLI that would look at your character inventory and optimise your loadout to give your character the highest possible level. Little did we know the blood, sweat and tears that would be involved to get this data from Bungie...

### Destiny API
After our initial investigation we were delighted to see that Bungie actually offered a public [REST API](https://www.bungie.net/platform/destiny/help/) for Destiny. What we didn't realise was how complicated they had managed to make it! My typical experience using public REST APIs had been to simply register for an API KEY and get cracking (usually accompanied with some good docs and examples). I can't say my experience with the Destiny API was quite the same. Despite some light-hearted comments in their minimal documentation...

>DEPRECATED - Please use GetAccountSummary instead, pretty please with sugar on top. Seriously, we'll be BFFs 4 evah.

>Don't you want to be a cool kid and use this service instead of GetAccount?

... you are by yourself to discover what each endpoint returns and how to make sense of that data.

We whittled down the API to find what we thought was the most appropriate endpoint to retrieve information about a character's inventory.

##### Endpoint template

`{membershipType}/Account/{destinyMembershipId}/Character/{characterId}/Inventory/`

##### Example response

```json
{
  "Response":{
    "data":{
      "buckets":{
        "Invisible": // array of items
        "Item": // array of items
        "Equippable": // array of items
      },
      "currencies":[
        {
          "itemHash":3159615086,
          "value":0
        },
        {
          "itemHash":2534352370,
          "value":0
        },
        {
          "itemHash":2749350776,
          "value":0
        }
      ]
    }
  },
  "ErrorCode":1,
  "ThrottleSeconds":0,
  "ErrorStatus":"Success",
  "Message":"Ok",
  "MessageData":{

  }
}
```

This appeared to return items although many fields that we had expected to find (item name etc.) were not present and instead an `itemHash` was found. Trawling the somewhat useful Destiny [community](https://www.bungie.net/en/Clan/Forum/39966) for developers, we discovered that these `itemHash`es corresponded to unique identifiers for items and could be resolved to find more details about the item. The somewhat tricky thing was that in order to resolve an `itemHash` the recommended approach was to fetch the latest definitions of items from Bungie and load them into a SQLite DB. This sounded slightly complicated for the purposes of our run-once application, so luckily we stumbled across another method. By appending `?definitions=true` to the end of your request, Bungie would send back the item definitions as a large JSON object. Obviously this approach is not practical if you need to make lots of requests to their API as you are fetching the same definitions all the time. However we only needed to make one call and this seemed a lot easier than starting up a SQLite DB.

A larger issue cropped up though... we were only retrieving the data for items already equipped in our inventory. Where were we going wrong? The endpoint promised to return the full inventory for our character but we weren't seeing it. Having spent hours in the Chrome Debugger Tools, Destiny Item Manager [source code](https://github.com/DestinyItemManager/DIM) and frantic Googling, we discovered that we needed to send additional headers with our requests to the API. Namely, you had to authenticate with a game network provider (PlayStation Network or Xbox Live) and obtain a token that you will send to Bungie. Only after this will you start to see all of the characters inventory. I really wish this information was more easily discoverable on the Web (hopefully this blog helps)! The logic to implement this can be found in our [PsnAuthenticationClass](https://github.com/alexlukelevy/destiny-light-loader/blob/master/src/main/java/auth/PsnAuthenticationService.java). It's somewhat of an eyesore but I hope it will help other aspiring Destiny developers out there! It works as follows:

1. Ask Bungie for a `jsessionid`
2. Log in to PSN, passing the user credentials along with this `jsessionid`
3. Obtain `state` and `X-NP-GRANT-CODE` values from the PSN response
4. Log in to Bungie with these values
5. Obtain the value of the `bungled` cookie that Bungie set

After doing all of this you will be able to use the value of that `bungled` cookie to send an additional header to Bungie; `x-csrf`. Supplying this combined with the `X-API-Key` to a Destiny REST endpoint will now give you all the data you expected.

Bearing in mind I had picked this project as a simple one to teach my friend some programming basics, I didn't really feel like I had succeeded with flying colours!

### Light Level Optimisation
After working with the headache that is the Destiny API, everything else was plain sailing. Deserialsing the items into buckets based on their type was fairly straightforward with the use of the `jackson-databind` library. All that was left was to actually perform the optimisation!

In Destiny each item has a light level value, the mean light level across all of your equipped items determines the light level of your character and in turn the quests available to you. Each item also has a grade; Common, Uncommon, Rare, Legendary and Exotic. Exotic items typically have incredibly useful attributes, so you are limited to only one piece of Exotic weapon and armour at a time. Therefore the role of our app was to keep this constraint in mind and select the correct combinations of items that would produce the highest character light level.

Initially I thought this could be solved as a dynamic programming optimisation problem, since the subproblems would overlap - selecting the next item based on a list of currently unchosen items. We then realised the solution was far simpler, iterate through the item types calculating the light level differential between the top Exotic and non-Exotic item. The optimal combination would be to choose the Exotic item for the item type that had the largest differential and then select the highest non-Exotic items for every other type.

### CLI
We wrapped all the code together and added a minimal CLI that would output the results to the console. You can easily build and run `destiny-light-loader` for yourself.

#### Build
```sh
mvn clean install package
```

#### Run
```sh
./destiny-light-loader.sh <PSN-GAMERTAG> <PSN-ID>
```

This will then print out the optimised loadout for all of your Destiny characters.

>Character: Hunter - 40
>
>Weapons:  
>Ghost: Iron Shell  
>Special Weapons: Antinomy XVI  
>Primary Weapons: Spare Change.25  
>Heavy Weapons: Bolt-Caster  
>
>Armour:  
>Artifacts: Dredgen Yor's Rose  
>Helmet: Warden's Sight  
>Leg Armor: Spektar Aspriet Boots  
>Gauntlets: Spektar Aspriet Grips  
>Class Armor: Takanome Ranger's Hood  
>Chest Armor: Iron Companion Vest  

### Final thoughts
`destiny-light-loader` started off as an educational project that we could learn from and eventually build into something of grander proportions. Sadly we got bogged down with some of the complexity and subtleties of the Destiny REST API. I'm not sure I've convinced my friend to join the world of software development, but I hope this may be useful to some of you out there. Good luck Guardians...
