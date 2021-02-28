# wow-spellLib
A lua library for accessing spelldata for all classes and specilizations in World of Warcraft

# Why did I create this library
When I came back to WoW (stopped playing during with WotLK) I noticed rather quickly noticed the lack of knowledge while playing PvP. All the new spells and talents kept me wondering "Wtf was that? How did I die? How is he still alive?". You know the usual :) I tried to find some kind of addon that would let me read through the abilites and talents of each class and their specializations. Sadly I didn't find any and WoW itslef didn't provide the desired function. I gathered the information I could from third party websites, but wasn't satisfied at all. Everytime I was looking something up I had to minimize the game and fire up Google. So in the end I decided to create my own addon (You can find it <a href="https://github.com/Timsche/wow-classInfo">here</a>). While I was and still am fairly new to lua programming I rather quickly encountered a barrier that seemed to make it outright impossible to create my addon. Meaning information about abilites and talents from other specilizations and classes then your current character could only be accessed via their unique spellIds. Since I didn't plan to code every Id for every ability and talent for every class and specilization into my addon I decided to create a library that would contain the needed information. So I basically create a list with every ability and talent for every class and specilization in the game. Every entry contains the name, id, rank and an indicator showing whether or not the ability is a passive. This way I could systematically get all the data for a certain class or specilization whenever I needed it.

# Usage
The library contains different tables with all spells, talents and pvp-talents for each class and it's specilizations. It's is basically a lookup for spells and their IDs for each class and specilization.

## Explaining the table structure
All three tables (self.classSpells, self.talents, self.talents) are build the same way. The code below shows an excerpt from **classSpells**. You can see that that the first layer is indexed with the classnames in all caps (DEATHKNIGHT, DEMONHUNTER...). Everyone of those contains another table with the indexes "BASIC", "COVENANT" and on for every specilization. Inside those are multiple tables containing the spelldata. Every spell is represented with the following four properties: name (Name of the spell), id (Unique ID of the spell), passive (boolean indicating if the spell is a passive) and rank (the rank of the spell). The tables **talents** and **pvp_talents** follow the same idea. The only difference ofcourse is that fact that the specilizations are the only indexes on the deeper level.
```
self.classSpells = {
  ...
  MAGE = {
    BASIC = {...},
    ARCANE = {
      { name = "Alter Time", id = 342245, passive = 0, rank = 1 },
      { name = "Arcane Barrage", id = 44425, passive = 0, rank = 1 },
      ...
    },
    FIRE = {...},
    FROST = {...},
    COVENANT = {...}
  },
  ...
};
```

## Initializing the library with LibStub and Ace3.0
Since I am using Ace3.0 and LibStub for my Addons I am only showing how to setup the library with these two.
```
function ciAddon:OnInitialize()
  self.spellLib = LibStub("SpellLib-1.0"):New()
  ...
end
```

From that point on the library and data can be accessed by calling **self.spellLib**.

## Accessing spell data directly (not recommended)
*It is highly recommended to use the provided functions below to get the data.*
The basis for this library is a simple lua table. It contains the name, id, passive and rank for each spell. The spells are sorted for each class and their specilizations, e.g. the spelldata for mages is stored like this:

```
self.classSpells = {
  ...
  MAGE = {
    BASIC = {...},
    ARCANE = {
      { name = "Alter Time", id = 342245, passive = 0, rank = 1 },
      { name = "Arcane Barrage", id = 44425, passive = 0, rank = 1 },
      ...
    },
    FIRE = {...},
    FROST = {...},
    COVENANT = {...}
  },
  ...
};
```
### Getting specilization data
This statement gets you all data for frost mages:
```
local frostmageSpells = self.spellLib.classSpells['MAGE']['FROST']
```

Keep in mind that those are only the spells for the specilization itself. The basic spells of the class are not included. For mages these would be accessed like this:
```
local mageSpells = self.spellLib.classSpells['MAGE']['BASIC']
```

Abilities gained from covenants specific to the class are also accessible like this:

### Getting class covenant data
```
local mageSpells = self.spellLib.classSpells['MAGE']['COVENANT']
```
The covenant spells are always listed in the following order:
1. kyrian
2. necrolord
3. night fae
4. venthyr

So to access a mages night fae class ability you could use the following code:
```
local mageNightFae = self.spellLib.classSpells['MAGE']['COVENANT'][3]
```

## Access spell data with provided functions (recommended)
To access certain data from the table you can use the following functions.

### Important! Legal parameter values
These function require you to specify the class and specilization for the spells you want to be returned. Both the class and the specilization need to be in full caps and there are no spaces in the names! E.g. to get the data for a havoc demon hunter the parameters would be "DEMONHUNTER" and "HAVOC". You can also set the specilization to "ALL" to get the data for all of the available specilizations. If you specifiy parameters that are not accepted the library will display an Error in your chat and your addon will most likely create a lua error since the function will return nil.

### Getting Spells for a class and specilization
**SpellLib:GetClassSpecSpells(_class, _spec, _includeBasic, _includeCovenant)** returns a table with all spells for a specilizations
The third and fourth parameters are boolean and default to true. If _spec is set to "ALL" covenant and basic spells with automatically be included.
```
  local classSpells = self.spellLib:GetClassSpecSpells("MAGE", "FROST")
  print(classSpells["FROST"][1]["name"])
```

### Getting talents for a spec
**GetSpecTalents(_class, _spec)** returns a table with all talents for a specilizations
The paramter **_spec** does not allow "ALL" since the talents are specific for every specilization.
```
  local specTalents = self.spellLib:GetSpecTalents("MAGE", "FROST")
  print(specTalents[1]["name"])
```

### Getting pvp talents for a spec
**GetSpecPVPTalents(_class, _spec)** returns a table with all pvp talents for a specilizations
The paramter **_spec** does not allow "ALL" since the pvp talents are specific for every specilization.
```
  local pvpTalents = self.spellLib:GetSpecPVPTalents("MAGE", "FROST")
  print(pvpTalents[1]["name"])
```

### GetClassCovenantSpells(_class)
**GetClassCovenantSpells** returns a table with the four basic covenant abilites
The covenant spells are always listed in the following order:
1. kyrian
2. necrolord
3. night fae
4. venthyr
```
  local mageCovSpells = self.spellLib:GetClassCovenantSpells("MAGE")
  print(mageCovSpells[1]["name"])
```

### Getting Covenant specific spells
**GetGeneralCovenantSpells()** returns a table with the four basic covenant abilites
The covenant spells are always listed in the following order:
1. kyrian
2. necrolord
3. night fae
4. venthyr
```
  local covSpells = self.spellLib:GetGeneralCovenantSpells()
  print(covSpells[1]["name"])
```
