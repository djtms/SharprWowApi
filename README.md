###SharprWowApi
SharprWowApi is a strongly typed .NET wrapper library for accessing [Blizzards WoW API](https://dev.battle.net/).

Install using [Nuget](https://www.nuget.org/packages/SharprWowApi/)
````
PM> Install-Package SharprWowApi
````
## Usage example

#####Get achievement By ID

```c#
var client = ApiClient(Region.Eu, Locale.en_GB, ApiKey.Value);
var achievement = client.GetAchievement(2144);
Console.WriteLine(achievement.Title);
```  

#####Get Character by realm and name

```c#
var client = new ApiClient(Region.EU, Locale.en_GB, ApiKey.Value);

var characterOne = client.GetCharacter("CharacterName", CharacterOptions.None, "Realm");
var characterTwo = client.GetCharacter("CharacterName", CharacterOptions.PvP, "Realm");

Console.WriteLine(character.Name);
Console.WriteLine(character.Pvp.Brackets.ArenaBracket2v2.Rating);
```
You can also set __realm__ in the _ApiClient_ and use extention methods if all characters you're getting are from the same realm.

```C#
client = new ApiClient(Region.EU, Locale.en_GB, ApiKey.Value, "Realm");

var characterOne = client.GetCharacter("CharacterName", CharacterOptions.None);
var characterTwo = client.GetCharacter("CharacterName", CharacterOptions.None);
var characterThree = client.GetCharacter("CharacterName", CharacterOptions.None);

Console.WriteLine(characterOne.Name);
Console.WriteLine(characterTwo.Name);
Console.WriteLine(characterThree.Name);
```
#####Create your own Guild & CharacterOptions (v1.0.2)
Can be useful if you need to retrieve more than a single field but less than all.
```C#
var options = GuildOptions.Members | GuildOptions.Challenge;
var guild = client.GetGuild("GuildName", options, "Realm");
...
```
####Usage example ASP.NET MVC
Returns a strongly typed _GuildRoot_ object to the view.
```C#
using SharprWowApi.Models.Guild;

public ActionResult Members()
{
    var client = new ApiClient(Region.EU, Locale.en_GB, ApiKey.Value, "Grim Batol");
    var guild = client.GetGuild("GuildName", GuildOptions.AllOptions);
    
     return View(guild);
}
```
```html
@model SharprWowApi.Models.Guild.GuildRoot

<div class="tab-ctrl" id="tabCtrl">
    <div id="page1" class="main-page">
    
       @foreach (var member in Model.Members.OrderByDescending(m => m.Character.Class))
        {
                Do something
        }
...
```

#####Async
Since some of the data returned by blizzards wow API is quite big (especially auction data & PVP leaderboard), it can be a good idea to use the async methods even though the json parsing is not async. Since it offloads the json deserialization to a new thread.  (https://github.com/JamesNK/Newtonsoft.Json/issues/66). 

```C#
var client = new ApiClient(Region.EU, Locale.en_GB, ApiKey.Value);

var getAuctionFile = client.GetAuctionFile("Realm");
var someCachedValue = "...";

//Check when the auctiondata was last modified (updated)
var lm = from f in getAuctionFile.Files
            select f.LastModified;

  if (!lm.Equals(someCachedValue))
  {
    //await here
    var getAuction = await client.GetAuctionsAsync("Realm");
    var auction = getAuction.Auctions.Auction;
    
  using (auction.GetEnumerator())
  {
       Parallel.ForEach(auction, a =>
       {
         Console.WriteLine(a.Item);
       });
  }
 ...
```
