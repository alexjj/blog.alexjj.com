---
title: Generating lots of tiddlers
date: 2021-12-15
summary: Programmatically creating tiddlers JSON
tags: ["Python", "Tiddlywiki"]
---

For my [Meditations Tiddlywiki](https://alexjj.github.io/Meditations/), I wanted to make each statement an individual tiddler.  Whilst it doesn't add a great deal to the site right now (beyond just viewing the minimum amount, the random tiddler button, and following the tiddlywiki mantra), I think it'll be useful later on. Over time I'd like to add my own thoughts or interpretations to them, or maybe categorise favourite ones with new tags. Plus they might be useful for other people who can export tiddlers or download the file.

I had the site previously but it was one tiddler per book, and that didn't make it very usable beyond just reading.

I find the easiest way to import lots of tiddlers is via a JSON file. Luckily I'd found a JSON file of Meditations some time ago. It's easy to find the text but that's not always as easily parsed as as structured document. The JSON was just book and then each line. I still needed to get the title, tags, and fields sorted out before I could import into a blank Tiddlywiki.

The general structure of JSON to be imported looks like this:

```json
[
    {
        "created": "20211215114254021",
        "text": "More content",
        "tags": "Home [[Book 1]]",
        "title": "2nd tiddler",
        "modified": "20211215114322673",
        "book": "3",
        "section": "12"
    },
    {
        "created": "20211215112427768",
        "text": "Content",
        "tags": "Home",
        "title": "Meditations",
        "modified": "20211215112535190",
        "section": "1",
        "book": "2"
    }
]
```

The `section` and `book` are fields I added and the rest should be obvious. Tags with spaces need to be in `[[ ]]`. So all I'd need to do is iterate over the JSON file and create new entries that matched this structure.

The first issue I came across was knowing how many sections per book there are. I made a new list of the number of sections and stored that in the meditations.json file. Whilst not necessary (as I could've used the code to generate this later on), I thought it might be helpful and more efficient to compute it once and store the results. `data` is the json file as you'll see shortly.

```python
sectionCount = [len(data["meditations"][f"{x}"]) for x in range(1,13)]
```

So now that I have everything it was just a case of building the list in the format I wanted and writing it to a file. Here's the code:

```python
import json

with open("meditations.json", "r") as read_file:
    data = json.load(read_file)

book_names = [
    "Book I",
    "Book II",
    "Book III",
    "Book IV",
    "Book V",
    "Book VI",
    "Book VII",
    "Book VIII",
    "Book IX",
    "Book X",
    "Book XI",
    "Book XII",
    ]

result = []

for book in range(12):
    for section in range(data["sectionCount"][book]):
        book_name = book_names[book]
        title = f"{book_name} - Section {section + 1}"
        tags = f"[[{book_name}]]"
        quote = data['meditations'][f"{book + 1}"][section]

        # Create each tiddler
        entry = dict()
        entry["created"] = "-01800000000000000"  # that's 180BC
        entry["text"] = quote
        entry["tags"] = tags
        entry["title"] = title
        entry["section"] = f"{section + 1}"
        entry["book"] = f"{book + 1}"

        result.append(entry)

# Write to json file
with open("tw_data.json", "w") as write_file:
    json.dump(result, write_file)
```

I started going down the route of converting the book number to roman numerals but given the one off nature of this I just made a list of the names I wanted for the books. 😅

I then just imported into a blank Tiddlywiki, made a few other pages and a table of contents macro tiddler and it's good to go!

[You can see the code and json data here.](https://github.com/alexjj/Meditations)
