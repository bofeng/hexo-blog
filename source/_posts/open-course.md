---
title: 'Open Course'
date: 2014-09-30 10:30:00
tags: [opencourse,course,education]
published: true
hideInList: false
feature: 
---
I am thinking a ocourse system like *pip* or *npm*, just a thought, might be useful someday.
<!-- more -->


#### Command

ocm stands for "open course manager".

```
ocm list / ls
ocm search
ocm install
ocm update
ocm open
ocm uninstall / remove
ocm init
ocm publish
ocm unpublish
```

#### meta.json
```json
{
    version : "1.0",
    author : {
        name : "Bo",
        portrait: "http://url-to-your-portrait",
        email: "one@example.org"
    }
    language: "English",
    main: "index.md",
    lessons : [
        {
            name : "lesson 1",
            material : [
                {
                    file : "handout1.md",
                    type : "handout",
                },
                {
                    file : "handout1.pdf",
                    type : "pdf",
                }
            ]
        },

        {
        ...
        }
    ]
}
```

And then all files are packed in zip file and upload to ocourse website, or use `ocm publish` command to push all folders there.

#### Other thoughts:

* The lessons part is kinda redundant, maybe we could remove that part, and only use folder instead, like "lesson-1" or "lesson-01", all the materials are put in that folder, e.g. "handout.md", "slides.md"

* course might be packed into a .nw format file, and opened by node-webkit app, this way can make it open in different OS.
