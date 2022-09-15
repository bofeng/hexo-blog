---
title: 'Fix error in python3-saml: A valid SubjectConfirmation was not found'
typora-root-url: ../../source
date: 2022-09-15 09:53:51
tags: [python, saml]
---



## Problem

If the assertion returned by IdP is encrypted, I am seeing this error:

> A valid SubjectConfirmation was not found on this Response

This problem only happens when the assertion returned is encrypted. If it is plain, then no such problem.

My environment:

* Ubunut: 22.04 LTS
* Python 3.10
* Python3-saml 1.14.0

The error was raised in these 2 lines ([source code >>](https://github.com/onelogin/python3-saml/blob/v1.14.0/src/onelogin/saml2/response.py#L246)):

```python
sc_data = scn.find('saml:SubjectConfirmationData', namespaces=OneLogin_Saml2_Constants.NSMAP)
if sc_data is None:
```

I printed the scn, like `print(tostring(scn))` and the xml clearly shows that it has a child node `<SubjectConfirmationData>`, but the `find` keeps returning `None`. What is more strange, if I recreate the scn node:

```python
new_scn = fromstring(tostring(scn))
sc_data = new_scn.find('saml:SubjectConfirmationData', namespaces=OneLogin_Saml2_Constants.NSMAP)
```

This actually works.

After keep debugging and searching for a whole day, it turns out [it is a problem in python pre-built lxml package](https://github.com/onelogin/python3-saml/issues/292).



## Solution

In my case, I reinstall lxml with no binary option:

```bash
$ pip uninstall lxml
$ pip install --no-binary lxml lxml==4.7.0
```

If using requirements.txt:

```
lxml==4.7.0
--no-binary lxml
python3-saml==1.14.0
```

Currently the [python3-saml](https://github.com/onelogin/python3-saml) repo is currently not under active development, I hope they fix this in the future version.

## Reference

* https://github.com/onelogin/python3-saml/issues/292
