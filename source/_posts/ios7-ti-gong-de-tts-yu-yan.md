---
title: ' iOS7提供的TTS语言'
date: 2014-03-03 08:00:00
tags: [iOS]
published: true
hideInList: false
feature: 
---
iOS7发布时增加了Text To Speech API，这里是它支持的语言，做个备忘，方便以后查。
<!-- more -->


iOS7将其封装在AVSpeechSynthesis一系列文件里，可以使用AVSpeechSynthesisVoice来查看它所支持的语言：

```objc
#import <AVFoundation/AVFoundation.h>
#import <AVFoundation/AVSpeechSynthesis.h>

// ...
NSArray* voices = [AVSpeechSynthesisVoice speechVoices];
for (AVSpeechSynthesisVoice* voice in voices) {
    NSLog(@"%@", voice);
}
```

运行结果：

* Language: th-TH
* Language: pt-BR
* Language: sk-SK
* Language: fr-CA
* Language: ro-RO
* Language: no-NO
* Language: fi-FI
* Language: pl-PL
* Language: de-DE
* Language: nl-NL
* Language: id-ID
* Language: tr-TR
* Language: it-IT
* Language: pt-PT
* Language: fr-FR
* Language: ru-RU
* Language: es-MX
* Language: zh-HK
* Language: sv-SE
* Language: hu-HU
* Language: zh-TW
* Language: es-ES
* Language: zh-CN
* Language: nl-BE
* Language: en-GB
* Language: ar-SA
* Language: ko-KR
* Language: cs-CZ
* Language: en-ZA
* Language: en-AU
* Language: da-DK
* Language: en-US
* Language: en-IE
* Language: hi-IN
* Language: el-GR
* Language: ja-JP


支持的语言还是挺多的。默认语音听起来质量并不差，说明它的语音文件应该不算小，假设一个语音文件要200m，那么只存语音文件就需要7个G，而它的系统文件也才几个G，能存储这36种语言的语音文件，实在有够NB。
