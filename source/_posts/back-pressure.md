---
title: 'Back Pressure'
date: 2020-02-23 13:17:54
tags: []
published: true
hideInList: false
feature: 
isTop: false
---

## Resources:

1, [Backpressure explained — the resisted flow of data through software](https://medium.com/@jayphelps/backpressure-explained-the-flow-of-data-through-software-2350b3e77ce7)

2, Flask Author: [I'm not feeling the async pressure](https://lucumr.pocoo.org/2020/1/1/async-pressure/)

3, Answer from Starlette author: [Back pressure?](https://github.com/encode/starlette/issues/802#issuecomment-574606003) :
> However network backpressure isn't actually the only thing we'd like to have in place here. We also want to ensure resource limiting at any potential bottlenecks. The most notable case here would be the number of available database connections.
> 
> Uvicorn currently allows you to set a --limit-concurrency <int> which hard-limits the maximum number of allowable tasks which may be running before 503's will be returned. In the most constrained case you could just set this based on the number of available database connections. 

4, Answer from FastAPI: [Back pressure?](https://github.com/tiangolo/fastapi/issues/857) :
>  the author of starlette has been aware of this issue for (at least) going on two years so I don't think there is any reason to panic
> ... in practice you can usually prevent backpressure issues by just using a rate-limiting load balancer and a good auto-scaling policy. This won't handle all cases, and won't save you from particularly bad design choices, but for 99% of deployed python applications this would already be overkill.

5, [Use sanic with Nginx](https://stackoverflow.com/questions/54454841/performance-decrease-when-using-nginx-as-reverse-proxy-for-sanic-gunicorn)
> Normally, nginx is supposed to control the backpressure for your backend ...

6, [Pymongo's connection pool size](https://pymongo.readthedocs.io/en/3.10.0/api/pymongo/mongo_client.html#pymongo.mongo_client.MongoClient)
> maxPoolSize (optional): The maximum allowable number of concurrent connections to each connected server. Requests to a server will block if there are maxPoolSize outstanding connections to the requested server. Defaults to 100. Cannot be 0.

