---
title: 'Apache MPM'
date: 2019-03-07 02:27:24
tags: [apache,web]
published: true
hideInList: false
feature: 
---
### View & Change MPM

To see Apache's current mpm mode:

```
apachectl -V | grep -i mpm
```

To switch mpm mode: `mpm_event`, `mpm_worker` and `mpm_prefork`, use:

```
sudo a2dismod mpm_event
sudo a2enmod mpm_prefork
sudo apachectl restart
```

### Reference

> #### Prefork
> With the Prefork module installed, Apache is a non-threaded, pre-forking web server. That means that each Apache child process contains a single thread and handles one request at a time. Because of that, it consumes more resources than the threaded MPMs: Worker and Event.
> 
> Prefork is the default MPM, so if no MPM is selected in EasyApache, Prefork will be selected. It still is the best choice if Apache has to use non-thread safe libraries such as mod_php (DSO), and is ideal if isolation of processes is important.
> 
> #### Worker
> 
> The Worker MPM turns Apache into a multi-process, multi-threaded web server. Unlike Prefork, each child process under Worker can have multiple threads. As such, Worker can handle more requests with fewer resources than Prefork. Worker generally is recommended for high-traffic servers running Apache versions prior to 2.4. However, Worker is incompatible with non-thread safe libraries. If you need to run something that isn’t thread safe, you will need to stick with Prefork.
> 
> #### Event
>
> Each process under Event also can contain multiple threads but, unlike Worker, each is capable of more than one task. Apache has the lowest resource requirements when used with the Event MPM.
> 
> Event, though, is supported only on servers running Apache 2.4. Under Apache 2.2, Event is considered experimental and is incompatible with some modules on older versions of Apache. Nevertheless, on high-traffic Apache 2.2 servers where Apache has experienced issues with memory, upgrading Apache to take advantage of the Event MPM can yield significant results.
> 

Source: https://www.liquidweb.com/kb/apache-mpms-explained/