# Write-up: My First Field [CORN CTF 2024]

A walkthrough of the "My First Field" challenge, from initial analysis to flag recovery using NBTExplorer.

## Challenge Information

- **CTF:** CornCTF 2025
- **Challenge:** My First Field
- **Category:** Miscellaneous
- **Points:** Dynamic (500 initial, 50 final)

## Problem Description

We were given a single Minecraft world save folder for game version 1.21.4. The prompt set the scene:

> The last farmer walked on an infinite land, with two pieces of wheat. Every step brought him closer to the end, but he did not stop...
>
> He opened a book, wrote a few words with a shaking hand and left it beside him. Shortly after, he closed his eyes forever. They were his last thoughts....
>
> **Game Version:** 1.21.4
>
> **Flag format:** `CORN{.*}`

![SCR-20250622-sono-2](/Users/tanjiazer/Pictures/Screenshots - MAC/SCR-20250622-sono-2.png)
*The initial challenge description.*

## Investigation

### Initial In-Game Analysis

The first step was I did was to load the world in the correct Minecraft version (`1.21.4`). Upon entering, I found myself in a massive superflat world composed entirely of farmland blocks, with some Slimes around. My player inventory was empty, there was nothing given at all. The world was in Hardcore mode, and the infinite size of the world made it clear that finding a single book through manual exploration was not a feasible strategy. This strongly suggested that the challenge was not about in-game discovery but about file forensics.

### Forensic Analysis with NBTExplorer

After some Googling, I proceeded to analyze the world files using **NBTExplorer**, a tool used for reading and editing Minecraft's NBT (Named Binary Tag) data.

![image-20250627185847838](/Users/tanjiazer/Library/Application Support/typora-user-images/image-20250627185847838.png)

After opening the `My_First_Field` folder in NBTExplorer, I began a systematic search for the book.

![image-20250627190010514](/Users/tanjiazer/Library/Application Support/typora-user-images/image-20250627190010514.png)

1.  **Player Data Check:** I first inspected the `playerdata` folder. Expanding the file corresponding to the player's UUID confirmed that both the `Inventory` and `EnderItems` lists were empty.

2.  **The "End" Check:** The phrase "closer to the end" in the prompt seemed like a hint to look into the End's data file. My primary theory was that the flag was hidden within the End dimension. In the world save, the End's data is stored in the `DIM1` folder.

To test this hypothesis, I used NBTExplorer's search function (or`Ctrl+F`) on the `DIM1` folder.

- **Search Attempt 1:** I searched for the **Value** `CORN{`. This would find the flag directly if it were stored as simple text. The search completed with "End of results".
- **Search Attempt 2:** Realizing the flag text might be encoded or part of a complex JSON string, I searched for the item's unique identifier instead. I searched `DIM1` for the **Value** `written_book`. This search also failed.

These results proved that the book was not in the End.

Therefor, the only remaining location for the book was the main Overworld, whose chunk data is stored in the `region` folder.

I repeated my most reliable search, this time targeting the `region` folder:

- **Search Attempt 3:** I searched the `region` folder for the **Value** `written_book`.

![image-20250627190307803](/Users/tanjiazer/Library/Application Support/typora-user-images/image-20250627190307803.png)

**This search was successful.** NBTExplorer immediately located a `written_book` item nested inside a `minecraft:chest` block entity.

![image-20250627190508224](/Users/tanjiazer/Library/Application Support/typora-user-images/image-20250627190508224.png)
**

## Finding the Flag

The search result pinpointed the exact location of the item in the NBT tree. By navigating to the parent `minecraft:chest` tag, I could inspect the chest's full inventory.

The chest contained exactly three items: two stacks of `minecraft:wheat` and one `minecraft:written_book` in slot 13. This discovery perfectly matched the story's description of "two pieces of wheat" and a book, confirming this was the target.

![image-20250627190756486](/Users/tanjiazer/Library/Application Support/typora-user-images/image-20250627190756486.png)

To finally read the flag, I had to expand the NBT data for the book itself. In modern Minecraft versions, specific item data (like a book's text) is stored within the `components` tag. The exact path to the flag was:

```
items` -> `(entry for Slot 13)` -> `components` -> `minecraft:written_book_content` -> `pages
```

Expanding the `pages` tag revealed the text written in the book, which contained the flag.

![image-20250627190845583](/Users/tanjiazer/Library/Application Support/typora-user-images/image-20250627190845583.png)

The text inside the book's `pages` tag revealed the final flag:

```
CORN{I_L0ST_MY_H0M3_4ND_MY_C0RN}
```
