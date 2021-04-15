---
title: Use `golines` to format Go's long line code
typora-root-url: ../../source
date: 2021-04-15 18:07:06
tags: [golang, golines]
---



VS Code already does a great work to use `gofmt` to auto format your go code, but it won't format your lone line go code. To do it you could install a tool named `golines`.

## Steps:

1, Install `golines`: `go get -u github.com/segmentio/golines`

2, Open VS code, install the plugin named "Run on Save"

![Screen Shot 2021-04-15 at 5.50.28 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618523434482/HUGkXKfCT.png)

3, Go to VS Code settings, search "Run on Save", then click "Edit in settings.json"

![Screen Shot 2021-04-15 at 5.51.03 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618523468428/w70g2IcSK.png)

4, For the `emeraldwalk.runonsave` 's bracket, add `commands` like below:
```javascript
"emeraldwalk.runonsave": {
        "commands": [
            {
                "match": "\\.go$",
                "cmd": "golines ${file} -m 79 -w"
            }
        ]
}
```
You can twist the parameter, for example,`-m 79` means if find a line has more than 79 chars, run `golines` to break/format it. More parameter please check the reference below.


## Reference
* [golines github repo](https://github.com/segmentio/golines)