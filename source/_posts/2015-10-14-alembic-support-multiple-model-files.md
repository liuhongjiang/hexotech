title: alembic support multiple model files
date: 2015-10-14 17:22:30
tags:
categories: python
---


Recently, in one of my project, I am using [sqlalchemy](http://www.sqlalchemy.org/) for ORM and [alembic](https://alembic.readthedocs.org/en/latest/) for migration management.

I have find one annoying problem, that the alembic auto generating only work for one module files.

<!--more-->

In the [Auto Generating Migrations Document](http://alembic.readthedocs.org/en/latest/autogenerate.html), it says how to configure autogenerate for alembic.

```
# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
target_metadata = None
```

change to:

```
from myapp.mymodel import Base
target_metadata = Base.metadata
```

But this is only for one module file(mymdel in the example), if you have more than one module file, it won't work.

Here is the workround for multiple models files, improved it based on articles in reference: 

```
from sqlalchemy import engine_from_config, pool, MetaData

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata

from src.models import (feature_vector, table2, table3)


def combine_metadata(*args):
    m = MetaData()
    for metadata in args:
        for t in metadata.tables.values():
            t.tometadata(m)
    return m

target_metadata = combine_metadata(feature_vector.Base.metadata, table2.Base.metadata, table3.Base.metadata)
```


## other issues

As I didn't use any framework, so I need to add the model path to the sys.path to the `env.py`

```
# add the model path to sys.path
import sys
import os

current_path = os.path.dirname(os.path.abspath(__file__))
ROOT_PATH = os.path.join(current_path, '..')
sys.path.append(ROOT_PATH)
```

## reference

[Multiple targetmetadata's in an alembic migration?](https://groups.google.com/forum/#!topic/sqlalchemy/FPvJr564Nn4)

[allow multiple metadata objects for autogenerate](https://bitbucket.org/zzzeek/alembic/issues/38/allow-multiple-metadata-objects-for)
