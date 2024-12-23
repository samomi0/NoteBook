---
title: FastapiCORS
date: 2024-07-19
---

Fastapi 跨域组件

<!-- more -->

~~~py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)
~~~