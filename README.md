# Survey Zone List

It's an addon for [The Elder Scroll Online](https://www.elderscrollsonline.com) which display all Blacksmith, Clothier and Woodworking master writ where the style is unknown,
and for each of them, looking for the style price and display it.

## Dependencies

Libraries : 
* [`LibAddonMenu-2.0`](https://esoui.com/downloads/info7-LibAddonMenu.html)
* [`LibAsync`](https://www.esoui.com/downloads/info2125-LibAsync.html)
* [`LibPrice`](https://www.esoui.com/downloads/info2204-LibPrice.html)

And also another addon : [`CraftStore`](https://www.esoui.com/downloads/info1590-CraftStoreStonethorn.html)  
I'm sorry, I not like the idea to depends from another addon, but because a lib about style is missing (and I don't have the time and energy to do it myself currently), I prefer depend from another addon instead of copy all datas already present in it.

## Install it

Into the addon folder (`Elder Scrolls Online\live\AddOns` in your document folder), you need to have a folder `WritStylePrice` and copy all files into it.

So you can :

* Clone the repository in the AddOns folder and name it `WritStylePrice`.
* Or download the zip file started by `esoui-` of the last release in github, and extract it in the AddOns folder.

## In game

You will have access to a new window which will display all master writ (which use a style) where the style is unknown, and for each, the price of the style.  
To display it, you can configure a keybind, or use the command `/writstyleprice`, also, if the command is not already used, you can use the command `/wsp`

*Yes I have keep many masterWrits to do my tests*  
![Screen with the list](https://projects.bulton.fr/teso/WritStylePrice/main_window.jpg)

And I you have the cursor on the price, a tooltip will display you all known price for the style  
![Screen with the list and the price tooltip](https://projects.bulton.fr/teso/WritStylePrice/main_window_price_tooltip.jpg)

When a character is loaded, the addon will scan the current character bag, your bank (not the guild bank) and you storage coffer if you are in your house.
From the settings page, you can configure what will be scanned. If you disable the scan for a things already scanned, all data generated and saved by this addon will be removed.

Also from the settings page, you can configure a preferred source order to obtain and display the price.  
![Screen of the settings page](https://projects.bulton.fr/teso/WritStylePrice/settings.jpg)

## About lua files

There are loaded in order :

* Initialise.lua
* Collect.lua
* Events.lua
* GUI.lua
* Item.lua
* ItemType\MasterWrit.lua
* ItemType\Style.lua
* List.lua
* Price.lua
* Settings.lua
* Run.lua

### Initialise.lua

Declare all variables and the initialise function.

Declared variables :

* `WritStylePrice` : The global table for all addon's properties and methods.
* `WritStylePrice.name` : The addon name
* `WritStylePrice.dirName` : The addon name without space (which correspond to the directory addon name)
* `WritStylePrice.savedVariables` : The `ZO_SavedVars` table which contains saved variable for this addon.
* `WritStylePrice.ready` : If the addon is ready to be used
* `WritStylePrice.async` : The library LibAsync
* `WritStylePrice.LAM` : The library LibAddonMenu2
* `WritStylePrice.charactersMap` : The liste of all your character (not saved)

Methods :

* `WritStylePrice:Initialise` : Module initialiser  
Intiialise savedVariables, settings panel, GUI, the collect and prices systems
* `WritStylePrice:initCharactersMap` : Populate the `charactersMap` property.

### Collect.lua

Table : `WritStylePrice.Collect`

Contain the bag, bank and storage coffers reader system

Constants :

* `TYPE_HOUSE_BANK` : A key used in `list` table to separate house storage coffers items from others.
* `TYPE_BANK` : A key used in `list` table to separate bank items from others.
* `TYPE_CHAR` : A key used in `list` table to separate characters bags items from others.

Properties :

* `list` : The table list of all items.  
The structure is like that : `table[TYPE_constant][bagIdOrCharacterId][bagId_slotIdx] = WritStylePrice.Item object`.  
This list is saved in savedVariables `collect.list`.
* `status` : The read/collected status for each part which can be read/collected
* `savedVars` : All saved variables dedicated to the collect system.

Methods :

* `WritStylePrice.Collect:init` : Initialise the collect system
* `WritStylePrice.Collect:initSavedVarsValues` : Initialise with a default value all saved variables dedicated to the collect system
* `WritStylePrice.Collect:initSavedList` : Initialise the self.list table contents
* `WritStylePrice.Collect:obtainScanCharBag` : To know if the character bag should be scanned or not
* `WritStylePrice.Collect:defineScanCharBag` : Define the value for the var used to know if the character bag should be scanned
* `WritStylePrice.Collect:obtainScanBank` : To know if the bank should be scanned or not
* `WritStylePrice.Collect:defineScanBank` : Define the value for the var used to know if the bank should be scanned
* `WritStylePrice.Collect:obtainScanHouseBank` : To know if the house bank (storrage coffers) should be scanned or not
* `WritStylePrice.Collect:defineScanHouseBank` : Define the value for the var used to know if the house bank should be scanned
* `WritStylePrice.Collect:readAllList` : Read all list of items collected and call the callback for each item
* `WritStylePrice.Collect:removeAllInBagType` : Remove all items in a specific bagType (not bagId !)
* `WritStylePrice.Collect:readSavedList` : Read all item saved in savedVar and instanciate a new Item object for each item.
* `WritStylePrice.Collect:readCharBag` : Read all item in the character bag
* `WritStylePrice.Collect:readBank` : Read all item in the bank
* `WritStylePrice.Collect:readHouseBanks` : Read all item in the house bank (storrage coffers)
* `WritStylePrice.Collect:generateSavedKey` : Generate the key used to save an item in `self.list[][]`
* `WritStylePrice.Collect:obtainBagItemList` : Obtain the table which contain the list item for a specific bagId
* `WritStylePrice.Collect:readItemSlot` : Read a slot index in a bag id
* `WritStylePrice.Collect:newItem` : To add an unknown item (for the list) to the list if it's an item that interests us
* `WritStylePrice.Collect:updateItem` : To update a known item in `self.list[][]`
* `WritStylePrice.Collect:removeItem` : To remove a known item from `self.list[][]`. Also check if it's a motif and if it has been learned.
* `WritStylePrice.Collect:newStyleLearn` : Callback for `self:readAllList` called when a new style has been learned. Used to check if the style is now known or not on each masterWrit in the list.

### Events.lua

Table : `WritStylePrice.Events`

Contain all functions called when a listened event is triggered.

Methods :

* `WritStylePrice.Events.onLoaded` : Called when the addon is loaded
* `WritStylePrice.Events.onLoadScreen` : Called after each load screen. Do the collect if necessary.
* `WritStylePrice.Events.keybindingsToggle` : Called when the keybind is used
* `WritStylePrice.Events.toggleGUI` : Called when the slash command is used
* `WritStylePrice.Events.GuiClose` : Called when the GUI is closed by the top-right cross
* `WritStylePrice.Events.onGuiMoveStop` : Called when the GUI o longer moved to save the current position
* `WritStylePrice.Events.onOpenBank` : Called when the bank is opened to collect it if necessary
* `WritStylePrice.Events.onMoveItem` : Called when a new item is looted, when an item move between bags, or is used.

### GUI.lua

Table : `WritStylePrice.GUI`

Contains all functions to define the GUI container and save GUIItems instances.

Properties :

* `ui` : The main UI control
* `isHidden` : If the UI is hidden or not
* `list` : The object which manage the ui table's content list
* `savedVars` : All saved variables dedicated to the gui.

Methods :

* `WritStylePrice.GUI:init` : Initialise the GUI
* `WritStylePrice.GUI:getIsHidden` : Get the info if the UI is hidden or not
* `WritStylePrice.GUI:refreshList` : Refresh the list of item in the ui table
* `WritStylePrice.GUI:initSavedVarsValues` : Initialise with a default value all saved variables dedicated to the gui
* `WritStylePrice.GUI:toggle` : Show or hide the UI
* `WritStylePrice.GUI:show` : Show the UI and refresh the content list
* `WritStylePrice.GUI:hide` : Hide the UI
* `WritStylePrice.GUI:savePosition` : Save the current position and size of the UI
* `WritStylePrice.GUI:restorePosition` : Redefine position and size of the UI from saved data
* `WritStylePrice.GUI:headerInitCell` : Called on each ui table header cell

### Item.lua

Table : `WritStylePrice.Item`

Contain all info about an item. It's a POO like with one instance of Item for each item.

Constants :

* `ITEM_TYPE_MW` : The itemType value for master writ
* `ITEM_TYPE_SP` : The itemType value for style page
* `ITEM_TYPE_MO` : The itemType value for style books and chapters

Properties :

* `bagId` : The bagId where is the item
* `slotIdx` : The slot index where is the item
* `charId` : If the bagId is a character bag, the character id, else nil
* `itemLink` : The itemLink
* `itemName` : The item name in the language used when the scan is done
* `itemType` : See constants
* `data` : An instance of `WritStylePrice.ItemType.MasterWrit` or `WritStylePrice.ItemType.Style` which contain specialised data about the item

Methods :

* `WritStylePrice.Item:New` : Instanciate a new Item "object"
* `WritStylePrice.Item:initData` : Initialise the data property with a instance of the dedicated object for the item type
* `WritStylePrice.Item:createBaseObj` : Generate and return the base table used to create a new Item object
* `WritStylePrice.Item.checkType` : Check if an item is a type we follow and save (masterWrit or stylePage/motives)

### ItemType/MasterWrit.lua

Table : `WritStylePrice.ItemType.MasterWrit`

Contain all dedicated function to known data about a masterWrit. It's a POO like with one instance of MasterWrit for each masterWrit item followed.  
Note : The object have access to the Item object properties which instanciate it.

Properties :

* `writItemType` : The itemType to craft (axe, stave, legs, etc)
* `styleIdx` : The style id to use for the craft
* `chapterIdx` : The style chapter index corresponding to the itemType
* `styleIsKnown` : If the style is known on one of the characters
* `nbVouchers` : Number of the vouchers given by the masterWrit
* `styleItem.motifLink` : The motif's itemLink
* `styleItem.motifId` : The motif's item id
* `styleItem.motifName` : The motif's item name (in the language used when the scan is done)

Methods :

* `WritStylePrice.ItemType.MasterWrit:New` : Instanciate a new MasterWrit object
* `WritStylePrice.ItemType.MasterWrit:readItem` : Read the item and populate properties
* `WritStylePrice.ItemType.MasterWrit.checkWritType` : Check the type writType to know if it's a masterWrit we need to follow or not
* `WritStylePrice.ItemType.MasterWrit:convertWritItemTypeToCSChapterIdx` : Convert a writ itemType (from info in masterWrit) to CraftStore chapter index
* `WritStylePrice.ItemType.MasterWrit:checkStyleIsKnown` : Check if the style of the masterWrit is known or not
* `WritStylePrice.ItemType.MasterWrit:updatePrices` : Update the motif's price

### ItemType/Style.lua

Table : `WritStylePrice.ItemType.Style`

Contain all dedicated function to known data about a style page. It's a POO like with one instance of Styles for each style item followed.  
Note : The object have access to the Item object properties which instanciate it.

Properties :

* `itemId` : The item ID
* `isKnown` : If the style is known on one of the charactersdone)

Methods :

* `WritStylePrice.ItemType.Style:New` : Instanciate a new Style object
* `WritStylePrice.ItemType.Style:readItem` : Read the item and populate properties
* `WritStylePrice.ItemType.Style.checkStyleIsKnown` : Check if the style is known or not for the current character. Called only if not already known on another character.

### List.lua

Table : `WritStylePrice.List`

Contain all the system to generate the ui table list.
This object extends `ZO_SortFilterList` object

Properties :

* `masterList` : A list of all item to display

Methods :

* `WritStylePrice.List:New` : Instanciate a new ZO_SortFilterList which use us and return it
* `WritStylePrice.List:Initialize` : *inheritdoc*
* `WritStylePrice.List.BuildMasterList` : *inheritdoc*
* `WritStylePrice.List.readItemObj` : Callback for `WritStylePrice.Collect:readAllList`, called for each item in the list. Add all masterWrit whose style is not known to the masterList
* `WritStylePrice.List.convertItemDataToDisplay` : Convert data in Item object to a table used for generate the row
* `WritStylePrice.List.FilterScrollList` : *inheritdoc*
* `WritStylePrice.List.SortScrollList` : *inheritdoc*
* `WritStylePrice.List.SetupItemRow` : *inheritdoc*
* `WritStylePrice.List.createItemTooltip` : Create the item tooltip for the current item in the cell
* `WritStylePrice.List.createPriceTooltip` : Create the tooltip to display all price found for an item

### Price.lua

Table : `WritStylePrice.ItemType.Style`

Contain all dedicated function to obtain the price of an item.

Properties :

* `savedVars` : All saved variables dedicated to the price system.
* `list` : The list of all items with a price.  
Note : This list is not saved in savedVariables.

Methods :

* `WritStylePrice.Price:init` : Initialise data used by the price system
* `WritStylePrice.Price:initSavedVarsValues` : Initialise with a default value all saved variables dedicated to the sort system
* `WritStylePrice.Price.obtainOrder` : Obtain the current order to use
* `WritStylePrice.Price.defineOrder` : Define a new order to use and refresh the UI list
* `WritStylePrice.Price.generatePrices` : Generate the price's table for the itemLink and add it to self.list
* `WritStylePrice.Price.addPrice` : Add a new price source to priceTable arg
* `WritStylePrice.Price.obtainPriceList` : Obtain the price list for a specific itemLink
* `WritStylePrice.Price.obtainPreferredPrice` : Obtain the preferred price for a specific itemLink
* `WritStylePrice.Price.convertSourceKeyToStr` : Convert a price source key to the translated human name

### Settings.lua

Table : `WritStylePrice.Settings`

Contain all function used to build the settings panel

Properties :

* `panelName` : The name of the settings panel

Methods :

* `WritStylePrice.Settings:init` : Initialise the settings panel
* `WritStylePrice.Settings:build` : Build the settings panel
* `WritStylePrice.Settings:buildScanCharBag` : Return info to build the setting panel for "scan character bag"
* `WritStylePrice.Settings:buildScanBank` : Return info to build the setting panel for "scan bank"
* `WritStylePrice.Settings:buildScanHouseBank` : Return info to build the setting panel for "scan house bank"
* `WritStylePrice.Settings:buildPreferredPriceOrder` : Return info to build the setting panel for "preferred price" order

### Run.lua

Define a listener to all used events.
