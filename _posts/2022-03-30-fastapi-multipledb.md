---
layout: post
emoji: 💽
title: "FastAPI에 2개 이상의 DB연동하기"
date: '2022-03-30 21:33:15'
author: Liam
categories: [ Python, FastAPI ]
image: assets/images/logo-background2.png
---

<br>
<br>

## 💡 Intro

- 원래 flask를 주로 사용하는데 지인분의 추천으로 FastAPI를 알게 되었습니다. FastAPI에서 libuv(node.js 성능의 핵심)를 코어로 사용하는 uvloop가 너무 매력적 이었고, [ASGI](https://asgi.readthedocs.io/en/latest/specs/main.html)를 한번 사용해보고싶어서 FastAPI를 한번 사용해볼까..? 라는 마음으로 사용해 보게 되었습니다.
- FastAPI로 관리자 웹을 만들다가 기존에 연결해 두었던 DB에 한개를 추가로 더 연결해서 MultipleDatabases를 구성해야하는 상황이 발생했습니다. 생각보다 reference가 많이 없었는데, FastAPI의 제작자이신 [tiangolo](https://github.com/tiangolo/fastapi/issues/2592)님의 issues에서 이 문제를 찾을 수 있었습니다. Multipledatabases에 대해 다양한분들의 의견이 잘 정리되어있어서 이를 저의 FastAPI에 적용한것을 정리해보려고 합니다.🙌


<br>
<br>


## 🔎 그전에! FastAPI란? 

[공식 문서](https://fastapi.tiangolo.com/)에는 다음과 같이 설명되어져 있습니다.

FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.6+ based on standard Python type hints.

주요기능:
- Fast: NodeJS 및 Go와 비슷한 성능, 현존하는 파이썬 웹 프레임워크 중 가장 빠르다.
- Fast to code
- Fewer bugs
- Intuitive
- Easy
- Short: 적은버그와 코드중복을 최소화할 수 있고, 각 매개변수 선언에 여러기능을 제공해준다.
- Robust: 문서 자동화 및 쉬운 배포가 가능하다.
- Standards-based: 개방형 API 표준(OpenAPI&JSON)을 기반으로 한다.

이것만 아니라, 현재 아직 많은 레퍼런스가 나와있지는 않지만 그것을 매꿔줄 매우매우 수준높은 공식문서가 잘 마련되어져 있었습니다. 또한 백엔드 엔지니어 입장에서 API를 사용할 수 있도록 만든 문서 작업이 생각보다 많은 시간이 소요되는데, FastAPI는 문서의 자동화를 제공해줌으로써 개발자가 문서 작업에 할애하는 시간을 줄이고 오직 코드에만 집중할 수 있도록 해주어서 업무 효율을 증진 시켜줄 수 있습니다.


<br>
<br>


## 📚 Multiple databases 설정하기 

<br>

### 📕 1. settings.py

```py
import os
from functools import lru_cache
from pydantic import BaseSettings

_VERSION_ = "0.1.0"

###multiple databases-1
class ISettings(BaseSettings):
    db_engine: str = "mysql+pymysql"
    db_host: str = ""
    db_user: str = "root"
    db_password: str = ""
    db_port: int = 3306
    db_database: str = ""

    debug: bool = True

    kakao_rest_key = ""

    @property
    def db_dsn(self):
        dsn = f"{self.db_engine}://{self.db_user}:{self.db_password}@{self.db_host}:{self.db_port}/{self.db_database}"
        return dsn    

class DevelopmentSettings(ISettings):
    pass

class ProductionSettings(ISettings):
    debug = False


###multiple databases-2
class ISettings2(BaseSettings):
    db_engine: str = "mysql+pymysql"
    db_host: str = ""
    db_user: str = "admin"
    db_password: str = ""
    db_port: int = 3306
    db_database: str = ""

    debug: bool = True

    kakao_rest_key = ""

    @property
    def db_dsn(self):
        dsn = f"{self.db_engine}://{self.db_user}:{self.db_password}@{self.db_host}:{self.db_port}/{self.db_database}"
        return dsn    

class DevelopmentSettings2(ISettings2):
    pass

class ProductionSettings2(ISettings2):
    debug = False


@lru_cache()
def get_settings():
    config = os.environ.get("FASTAPI_CONFIG", "default")
    configs = {
        "development": DevelopmentSettings,
        "production": ProductionSettings,
        "default": DevelopmentSettings,
    }

    return configs.get(config, DevelopmentSettings)()

@lru_cache()
def get_settings2():
    config = os.environ.get("FASTAPI_CONFIG", "default")
    configs = {
        "development": DevelopmentSettings2,
        "production": ProductionSettings2,
        "default": DevelopmentSettings2,
    }

    return configs.get(config, DevelopmentSettings2)()
```

<br>

📌 functools - @lru_cache** : LRU(Least Recently Used)캐싱을 사용하기위해 **@lru_cache**를 사용했습니다. @lru_cache 데코레이터는 functools 내장 모듈로 부터 불러올 수 있으며, @lru_cache 를 아무 함수 위에 선언하면 사용하면 그 함수에 넘어온 인자를 키(key)로 함수의 호출 결과를 값(value)으로 LRU캐싱이 적용됩니다.

<br>

### 📘 2. database.py

```py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

from api.settings import get_settings, get_settings2

Base_1 = declarative_base()
Base_2 = declarative_base()

SQLALCHEMY_DATABASE_URL = get_settings().db_dsn
SQLALCHEMY_DATABASE_URL2 = get_settings2().db_dsn

###multiple databases-1
engine1 = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal1 = sessionmaker(autocommit=False, autoflush=False, bind=engine1)

###multiple databases-2
engine2 = create_engine(SQLALCHEMY_DATABASE_URL2)
SessionLocal2 = sessionmaker(autocommit=False, autoflush=False, bind=engine2)

def get_db():
    session = SessionLocal1()
    try:
        yield session
        session.commit()
    finally:
        session.close()
        
def get_db2():
    session = SessionLocal2()
    try:
        yield session
        session.commit()
    finally:
        session.close()
```

📌 sqlalchemy : SQL 문법 없이 개발 중인 언어로 데이터베이스에 접근할 수 있게 해주는 라이브러리를 **ORM(Object Relational Mapping)** 이라고 합니다. 그 중에서도 [**sqlalchemy**](https://www.sqlalchemy.org/)은 python의 대표적인 ORM입니다.

<br>

### 📒 3. model.py

```py
from api.database import Base_1, Base_2 
from api.models.base_model import ModelBase

class DB_Schema1(Base_1, ModelBase):
    __tablename__ = "DB_Schema1_table"

class DB_Schema2(Base_2, ModelBase):
    __tablename__ = "DB_Schema2_table" 
```

<br>

### 📗 4. route.py

```py
from fastapi import APIRouter, Depends, Body
from api.database import get_db, get_db2

router = APIRouter()


@router.get(
    "/items/get_db1", description="multiple DB"
)
async def get_db1(
    db=Depends(get_db),
):
    
    return True
    
    
@router.get(
    "/items/get_db2", description="multiple DB"
)
async def get_db2(
    db=Depends(get_db2),
):
    
    return True
```

<br>
<br>

**[참고자료]**
- [https://blog.neonkid.xyz/253](https://blog.neonkid.xyz/253)
- [pydantic - BaseSettings](https://pydantic-docs.helpmanual.io/usage/settings/)
- [FastAPI - pydantic BaseSettings예제](https://fastapi.tiangolo.com/advanced/settings/)
