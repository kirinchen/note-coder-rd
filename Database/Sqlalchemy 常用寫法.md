---
title: Sqlalchemy 常用寫法
tags: [python]

---

# Sqlalchemy 常用寫法


## 只取得一個結果

### 強制查詢結果只有一個,如果有兩個就會跳錯

```python=
session.query(Schema).filter(some where).one()
```

### 有多個只拿第一個

```python=
session.query(Schema).filter(some where).first()
```

## where .. and .. or

### 基本 where

```python=
profiles = session.query(profile.name).filter(profile.email == email).all()
```

### and where 

```python=
profiles = session.query(profile.name).filter(and_(profile.email == email, profile.password == password_hash))
```

### or where
```python=
profiles = session.query(profile.name).filter(or_(profile.email == email, profile.password == password_hash))
```

## 取得Entity Class 的 PK(主鍵) 欄位

### 方法一 : 探詢每個 columns

```python=
def get_entity_pk_key(entity_clazz: type) -> InstrumentedAttribute:
    column_names = entity_clazz.__table__.columns
    for c in column_names:
        if c.primary_key:
            return getattr(entity_clazz, c.name)
    raise KeyError('not find pk')

```
> 如果是複合鍵,就有可能查到兩個

### 方法二 : 透過table info 取得


## query 後的 result 轉 dict
```python=
row = session.query(User.id, User.username, User.email)\
    .filter(and_(User.id == id, User.username == username)).first()

print(dict(zip(row.keys(), row))) # as a dict
```

## 排序

```python=
        routing_list: List[ShipmentRouting] = self.session.query(ShipmentRouting).filter(
            ShipmentRouting.shipment_number.in_(self.shipment_number_list)).order_by(
            asc(ShipmentRouting.routing_sequence))
```

###### tags: `python`