## Python Data Class
available in Python 3.6

simplified python class, less boilerplate. extensible and customizable.

good replacement of attrs, idea came from NamedTuples

example:
```
from dataclasses import dataclass
from datetime import datetime

class DC:
  location: str
  datetime: datetime
  yyyymmdd: int = 20180131 #default initialization

  #can be either string or datetime
  date: Union[datetime, str]

  #factory for initialization
  year: int = field(
    default-factory=lambda: datetime.now().year
  )

  #post init run after __init__ finishes
  def __post_init__(self):
    if isintance(self.date, str):
      self.date = datetime.fromisoformat(self.date) #added in Python 3.7
```

### Date Class Utility Functions

```
asdict
astuple
make_dataclass
is_dataclass
replace
```

can be converted to json with asdict then json.dumps

### handy attributes
```
__annotations__
__dataclass_params__
__dataclass_fields__
```

