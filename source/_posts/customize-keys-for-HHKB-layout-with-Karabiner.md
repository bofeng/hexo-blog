---
title: Customize keys for HHKB layout with Karabiner
typora-root-url: ../../source
date: 2021-09-04 11:32:19
tags: ["keyboard", "hhkb", "karabiner"]
---



Recently I am using HHKB and Niz Atom 68, especially I really like the typing feel in the Niz Atom68, it is so good. However, typing's muscle memory is hard to change, for example, it is hard to press the ~ key when need to use `cd ~` and the ` key when quote a code block. Also in HHKB layout, hard to press the arrow key.



## Maping Keys

With the software named [Karabiner](https://karabiner-elements.pqrs.org/), we can easily modify the key. Below is a configuration for 3 key modifications for my use:

* Command + Esc, map to Command + ` : this is used for switch window in one application. For example, you have browser both opened in normal mode and incognito mode, and you want to swith these 2 browser windows. In a normal layout keyboard with mac, you can use the "Command + Tilda" key, but in HHKB and Niz Atom 68 layout, the tilda key is way over the top-right corner. So with this modification you will comfortable to press the "command + esc" to swith windows.
* Shift + Esc, map to ~. Same reason like above. With this modification, it is easy to type ~ with your muscle memory, like `cd ~`.
* Left ctrl + hjkl to arrow keys like Vim. I am a Vim user, so this modification let me easily type the "arrow keys" w/o moving my hands. Especially for HHKB, it is hard to press those arrow keys, this one is a saver.



## How to apply

Open the `.config/karabiner/karabiner.json` file, add the below mappings to `profiles.complex_modifications.rules`:

If you don't have the rule section yet, you could open Kababiner, go to "Complex Modifications", click "Import more rules from internet", and find the Left + hjkl to import. Then open the above json file, manually add other 2 modification keys.  Remember to restart your Karabiner after the change.



```json
"rules": [
    {
      "description": "command + Esc, to command + `",
      "manipulators": [
        {
          "from": {
            "key_code": "escape",
            "modifiers": {
              "mandatory": [
                "command"
              ]
            }
          },
          "to": [
            {
              "key_code": "grave_accent_and_tilde",
              "modifiers": [
                "command"
              ]
            }
          ],
          "type": "basic"
        }
      ]
    },
    {
      "description": "shift + Esc, to ~",
      "manipulators": [
        {
          "from": {
            "key_code": "escape",
            "modifiers": {
              "mandatory": [
                "shift"
              ]
            }
          },
          "to": [
            {
              "key_code": "grave_accent_and_tilde",
              "modifiers": [
                "shift"
              ]
            }
          ],
          "type": "basic"
        }
      ]
    },
    {
      "description": "Left ctrl + hjkl to arrow keys Vim",
      "manipulators": [
        {
          "from": {
            "key_code": "h",
            "modifiers": {
              "mandatory": [
                "left_control"
              ],
              "optional": [
                "any"
              ]
            }
          },
          "to": [
            {
              "key_code": "left_arrow"
            }
          ],
          "type": "basic"
        },
        {
          "from": {
            "key_code": "j",
            "modifiers": {
              "mandatory": [
                "left_control"
              ],
              "optional": [
                "any"
              ]
            }
          },
          "to": [
            {
              "key_code": "down_arrow"
            }
          ],
          "type": "basic"
        },
        {
          "from": {
            "key_code": "k",
            "modifiers": {
              "mandatory": [
                "left_control"
              ],
              "optional": [
                "any"
              ]
            }
          },
          "to": [
            {
              "key_code": "up_arrow"
            }
          ],
          "type": "basic"
        },
        {
          "from": {
            "key_code": "l",
            "modifiers": {
              "mandatory": [
                "left_control"
              ],
              "optional": [
                "any"
              ]
            }
          },
          "to": [
            {
              "key_code": "right_arrow"
            }
          ],
          "type": "basic"
        }
      ]
    }
  ]
```



