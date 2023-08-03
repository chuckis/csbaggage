## Один ко многим  [](#one-to-many)

Связь «один ко многим» помещает внешний ключ в дочернюю таблицу, ссылающуюся на родительскую. [relationship()](relationship_api.html#sqlalchemy.orm.relationship)затем указывается в родительском объекте как ссылка на набор элементов, представленных дочерним элементом:

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List["Child"]] = relationship()

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parent_id: Mapped[int] = mapped_column(ForeignKey("parent_table.id"))
```

Чтобы установить двунаправленную связь «один ко многим», где «обратная» сторона — это «многие к одному», укажите дополнительный [relationship()](relationship_api.html#sqlalchemy.orm.relationship)и соедините два с помощью [relationship.back_populates](relationship_api.html#sqlalchemy.orm.relationship.params.back_populates)параметра, используя имя атрибута каждого [relationship()](relationship_api.html#sqlalchemy.orm.relationship) в качестве значения для [relationship.back_populates](relationship_api.html#sqlalchemy.orm.relationship.params.back_populates)другого:

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List["Child"]] = relationship(back_populates="parent")

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parent_id: Mapped[int] = mapped_column(ForeignKey("parent_table.id"))
  parent: Mapped["Parent"] = relationship(back_populates="children")
```

Childполучит parentатрибут с семантикой «многие к одному».
### Использование наборов, списков и других типов коллекций для функции «один ко многим»  [](#using-sets-lists-or-other-collection-types-for-one-to-many)

При использовании аннотированных декларативных сопоставлений тип коллекции, используемый для , [relationship()](relationship_api.html#sqlalchemy.orm.relationship)получается из типа коллекции, переданного [Mapped](internals.html#sqlalchemy.orm.Mapped)типу контейнера. Пример из предыдущего раздела может быть написан так, чтобы для коллекции использовалось a set, а не a :listParent.childrenMapped[Set["Child"]]

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[Set["Child"]] = relationship(back_populates="parent")
```

При использовании неаннотированных форм, включая императивные сопоставления, класс Python для использования в качестве коллекции может быть передан с использованием параметра [relationship.collection_class](relationship_api.html#sqlalchemy.orm.relationship.params.collection_class).

Смотрите также

[Настройка доступа к коллекции](collection_api.html#custom-collections) — содержит дополнительные сведения о конфигурации коллекции, включая некоторые методы сопоставления[relationship()](relationship_api.html#sqlalchemy.orm.relationship) со словарями.
### Настройка поведения удаления для одного ко многим  [](#configuring-delete-behavior-for-one-to-many)

Часто бывает так, что все Childобъекты должны быть удалены, когда их владение Parentудалено. Для настройки этого поведения используется deleteпараметр каскада, описанный при [удалении .](cascades.html#cascade-delete)Дополнительным параметром является то, что Childсам объект может быть удален, когда он отсоединяется от своего родителя. Это поведение описано в [delete-orphan](cascades.html#cascade-delete-orphan) .

Смотрите также

[удалить](cascades.html#cascade-delete)

[Использование каскада внешнего ключа ON DELETE с отношениями ORM](cascades.html#passive-deletes)

[удалить-сирота](cascades.html#cascade-delete-orphan)
## Многие к одному  [](#many-to-one)

Многие к одному помещает внешний ключ в родительскую таблицу, ссылающуюся на дочернюю. [relationship()](relationship_api.html#sqlalchemy.orm.relationship)объявляется на родительском объекте, где будет создан новый скалярный атрибут:

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  child_id: Mapped[int] = mapped_column(ForeignKey("child_table.id"))
  child: Mapped["Child"] = relationship()

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
```

В приведенном выше примере показано отношение «многие к одному», которое предполагает ненулевое поведение; следующий раздел, [Nullable Many-to-One](#relationship-patterns-nullable-m2o) , иллюстрирует версию, допускающую значение null.

Двунаправленное поведение достигается добавлением секунды [relationship()](relationship_api.html#sqlalchemy.orm.relationship) и применением [relationship.back_populates](relationship_api.html#sqlalchemy.orm.relationship.params.back_populates)параметра в обоих направлениях, используя имя атрибута каждого [relationship()](relationship_api.html#sqlalchemy.orm.relationship) в качестве значения для [relationship.back_populates](relationship_api.html#sqlalchemy.orm.relationship.params.back_populates)другого:

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  child_id: Mapped[int] = mapped_column(ForeignKey("child_table.id"))
  child: Mapped["Child"] = relationship(back_populates="parents")

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parents: Mapped[List["Parent"]] = relationship(back_populates="child")
```
### Nullable Many-to-One  [](#nullable-many-to-one)

В предыдущем примере Parent.childотношение не имеет разрешающего типа None; это следует из того, что Parent.child_idсам столбец не может принимать значения NULL, поскольку он набирается с помощью Mapped[int]. Если бы мы хотели Parent.child, чтобы многие к одному допускали **значение NULL**Parent.child_id , мы могли бы установить оба параметра и Parent.childкак be Optional[], и в этом случае конфигурация выглядела бы так:

```
**from** **typing** **import** Optional

**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  child_id: Mapped[Optional[int]] = mapped_column(ForeignKey("child_table.id"))
  child: Mapped[Optional["Child"]] = relationship(back_populates="parents")

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parents: Mapped[List["Parent"]] = relationship(back_populates="child")
```

Выше столбец для Parent.child_idбудет создан в DDL, чтобы разрешить NULLзначения. При использовании [mapped_column()](mapping_api.html#sqlalchemy.orm.mapped_column)с явными объявлениями типизации спецификация эквивалентна установке на для , тогда как эквивалентна установке на . См. статью [mapped_column(), которая получает тип данных и допустимость значений NULL из аннотации Mapped,](declarative_tables.html#orm-declarative-mapped-column-nullability) чтобы узнать об этом поведении.child_id: Mapped[Optional[int]][Column.nullable](../core/metadata.html#sqlalchemy.schema.Column.params.nullable)True[Column](../core/metadata.html#sqlalchemy.schema.Column)child_id: Mapped[int]False[](declarative_tables.html#orm-declarative-mapped-column-nullability)

Кончик

Если вы используете Python 3.10 или выше,[**Синтаксис PEP 604**](https://peps.python.org/pep-0604/) более удобен для указания необязательных типов с помощью, которые в сочетании с | None[**PEP 563**](https://peps.python.org/pep-0563/) отложил оценку аннотаций, чтобы типы в строковых кавычках не требовались, будет выглядеть так:

```
**from** **__future__** **import** annotations

**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  child_id: Mapped[int | **None**] = mapped_column(ForeignKey("child_table.id"))
  child: Mapped[Child | **None**] = relationship(back_populates="parents")

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parents: Mapped[List[Parent]] = relationship(back_populates="child")
```
## Один к одному  [](#one-to-one)

«Один к одному» по сути является отношением [«один ко многим»](#relationship-patterns-o2m) с точки зрения внешнего ключа, но указывает, что в любой момент времени будет только одна строка, которая ссылается на конкретную родительскую строку.

При использовании аннотированных сопоставлений с [Mapped](internals.html#sqlalchemy.orm.Mapped), соглашение «один к одному» достигается за счет применения типа, не относящегося к коллекции, к аннотации [Mapped](internals.html#sqlalchemy.orm.Mapped)с обеих сторон отношения, что означает для ORM, что коллекция не должна использоваться ни с одной из сторон. , как в примере ниже:

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  child: Mapped["Child"] = relationship(back_populates="parent")

**class** **Child**(Base):
  __tablename__ = "child_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parent_id: Mapped[int] = mapped_column(ForeignKey("parent_table.id"))
  parent: Mapped["Parent"] = relationship(back_populates="child")
```

Выше, когда мы загружаем Parentобъект, Parent.childатрибут будет ссылаться на один Childобъект, а не на коллекцию. Если мы заменим значение Parent.childновым Childобъектом, единица рабочего процесса ORM заменит предыдущую Childстроку новой, установив для предыдущего child.parent_idстолбца значение NULL по умолчанию, если только не настроено конкретное [каскадное поведение.](cascades.html#unitofwork-cascades)

Кончик

Как упоминалось ранее, ORM рассматривает шаблон «один-к-одному» как соглашение, в котором предполагается, что при загрузке атрибута Parent.childв Parentобъект возвращается только одна строка. Если будет возвращено более одной строки, ORM выдаст предупреждение.

Однако Child.parentсторона вышеуказанного отношения остается отношением «многие к одному» и остается неизменной, и в самой ORM нет встроенной системы, которая предотвращает создание более одного объекта против одного и того же Childво Parentвремя сохранения. Вместо этого в реальной схеме базы данных могут использоваться такие методы, как [ограничения уникальности](../core/constraints.html#schema-unique-constraint) , чтобы обеспечить такое расположение, где ограничение уникальности для столбца Child.parent_idгарантирует, что только одна Childстрока может ссылаться на конкретную Parentстроку в каждый момент времени.

**Новое в версии 2.0:** конструкция [relationship()](relationship_api.html#sqlalchemy.orm.relationship)может получить действующее значение параметра [relationship.uselist](relationship_api.html#sqlalchemy.orm.relationship.params.uselist) из заданной [Mapped](internals.html#sqlalchemy.orm.Mapped)аннотации.
### Настройка uselist=False для неаннотированных конфигураций  [](#setting-uselist-false-for-non-annotated-configurations)

При использовании [relationship()](relationship_api.html#sqlalchemy.orm.relationship)без преимуществ [Mapped](internals.html#sqlalchemy.orm.Mapped) аннотаций шаблон «один к одному» можно включить с помощью параметра, [relationship.uselist](relationship_api.html#sqlalchemy.orm.relationship.params.uselist)установленного на Falseсторону, которая обычно является «много», как показано в неаннотированной декларативной конфигурации ниже:

```
**class** **Parent**(Base):
  __tablename__ = "parent_table"

  id = mapped_column(Integer, primary_key=**True**)
  child = relationship("Child", uselist=**False**, back_populates="parent")

**class** **Child**(Base):
  __tablename__ = "child_table"

  id = mapped_column(Integer, primary_key=**True**)
  parent_id = mapped_column(ForeignKey("parent_table.id"))
  parent = relationship("Parent", back_populates="child")
```
## Многие ко многим  [](#many-to-many)

Многие ко многим добавляет ассоциативную таблицу между двумя классами. Таблица ассоциаций почти всегда задается как [Table](../core/metadata.html#sqlalchemy.schema.Table)объект Core или другой объект Core, который можно выбрать, например объект [Join](../core/selectable.html#sqlalchemy.sql.expression.Join), и указывается [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary)аргументом [relationship()](relationship_api.html#sqlalchemy.orm.relationship). Обычно [Table](../core/metadata.html#sqlalchemy.schema.Table)используется [MetaData](../core/metadata.html#sqlalchemy.schema.MetaData)объект, связанный с декларативным базовым классом, чтобы директивы [ForeignKey](../core/constraints.html#sqlalchemy.schema.ForeignKey)могли найти удаленные таблицы, с которыми можно связать:

```
**from** **__future__** **import** annotations

**from** **sqlalchemy** **import** Column
**from** **sqlalchemy** **import** Table
**from** **sqlalchemy** **import** ForeignKey
**from** **sqlalchemy** **import** Integer
**from** **sqlalchemy.orm** **import** Mapped
**from** **sqlalchemy.orm** **import** mapped_column
**from** **sqlalchemy.orm** **import** DeclarativeBase
**from** **sqlalchemy.orm** **import** relationship

**class** **Base**(DeclarativeBase):
  **pass**

*# note for a Core table, we use the sqlalchemy.Column construct,*
*# not sqlalchemy.orm.mapped_column*
association_table = Table(
  "association_table",
  Base.metadata,
  Column("left_id", ForeignKey("left_table.id")),
  Column("right_id", ForeignKey("right_table.id")),
)

**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List[Child]] = relationship(secondary=association_table)

**class** **Child**(Base):
  __tablename__ = "right_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
```

Кончик

В приведенной выше «таблице ассоциаций» установлены ограничения внешнего ключа, которые относятся к двум таблицам сущностей по обе стороны отношения. Тип данных каждого из association.left_idи association.right_idобычно выводится из таблицы, на которую ссылаются, и может быть опущен. Также **рекомендуется** , хотя это никоим образом не требуется SQLAlchemy, чтобы столбцы, которые ссылаются на две таблицы сущностей, устанавливались либо в рамках уникального **ограничения** , либо, что чаще встречается, в качестве **ограничения первичного ключа** ; это гарантирует, что повторяющиеся строки не будут сохраняться в таблице независимо от проблем на стороне приложения:

```
association_table = Table(
  "association_table",
  Base.metadata,
  Column("left_id", ForeignKey("left_table.id"), primary_key=**True**),
  Column("right_id", ForeignKey("right_table.id"), primary_key=**True**),
)
```
### Настройка двунаправленной связи «многие ко многим  [](#setting-bi-directional-many-to-many)

Для двунаправленного отношения обе стороны отношения содержат коллекцию. Укажите с помощью [relationship.back_populates](relationship_api.html#sqlalchemy.orm.relationship.params.back_populates), и для каждого [relationship()](relationship_api.html#sqlalchemy.orm.relationship)укажите общую таблицу ассоциаций:

```
**from** **__future__** **import** annotations

**from** **sqlalchemy** **import** Column
**from** **sqlalchemy** **import** Table
**from** **sqlalchemy** **import** ForeignKey
**from** **sqlalchemy** **import** Integer
**from** **sqlalchemy.orm** **import** Mapped
**from** **sqlalchemy.orm** **import** mapped_column
**from** **sqlalchemy.orm** **import** DeclarativeBase
**from** **sqlalchemy.orm** **import** relationship

**class** **Base**(DeclarativeBase):
  **pass**

association_table = Table(
  "association_table",
  Base.metadata,
  Column("left_id", ForeignKey("left_table.id"), primary_key=**True**),
  Column("right_id", ForeignKey("right_table.id"), primary_key=**True**),
)

**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List[Child]] = relationship(
      secondary=association_table, back_populates="parents"
  )

**class** **Child**(Base):
  __tablename__ = "right_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  parents: Mapped[List[Parent]] = relationship(
      secondary=association_table, back_populates="children"
  )
```
### Использование формы с поздней оценкой для «вторичного» аргумента  [](#using-a-late-evaluated-form-for-the-secondary-argument)

Параметр также принимает две разные формы с «поздней оценкой», включая строковое имя таблицы [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary), [relationship()](relationship_api.html#sqlalchemy.orm.relationship)а также вызываемое лямбда-выражение. См. раздел [Использование формы с поздней оценкой для «вторичного» аргумента «многие ко многим»](#orm-declarative-relationship-secondary-eval) для фона и примеров.
### Использование наборов, списков и других типов коллекций для функции «многие ко многим»  [](#using-sets-lists-or-other-collection-types-for-many-to-many)

Конфигурация коллекций для отношения «многие ко многим» идентична настройке « [один ко многим»](#relationship-patterns-o2m) , как описано в [разделе «Использование наборов, списков или других типов коллекций для отношения «один ко многим»](#relationship-patterns-o2m-collection) . Для аннотированного сопоставления с использованием [Mapped](internals.html#sqlalchemy.orm.Mapped)коллекция может быть указана типом коллекции, используемой в [Mapped](internals.html#sqlalchemy.orm.Mapped)универсальном классе, например set:

```
**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[Set["Child"]] = relationship(secondary=association_table)
```

При использовании неаннотированных форм, включая императивные сопоставления, как в случае с «один ко многим», класс Python для использования в качестве коллекции может быть передан с использованием параметра [relationship.collection_class](relationship_api.html#sqlalchemy.orm.relationship.params.collection_class).

Смотрите также

[Настройка доступа к коллекции](collection_api.html#custom-collections) — содержит дополнительные сведения о конфигурации коллекции, включая некоторые методы сопоставления[relationship()](relationship_api.html#sqlalchemy.orm.relationship) со словарями.
### Удаление строк из таблицы «многие ко многим  [](#deleting-rows-from-the-many-to-many-table)

Поведение, уникальное для [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary) аргумента to, [relationship()](relationship_api.html#sqlalchemy.orm.relationship)заключается в том, что [Table](../core/metadata.html#sqlalchemy.schema.Table)указанное здесь свойство автоматически подвергается операторам INSERT и DELETE при добавлении или удалении объектов из коллекции. Нет **необходимости удалять из этой таблицы вручную** . Действие удаления записи из коллекции приведет к удалению строки при сбросе:

```
*# row will be deleted from the "secondary" table*
*# automatically*
myparent.children.remove(somechild)
```

Часто возникает вопрос, как можно удалить строку во «вторичной» таблице, когда дочерний объект передается непосредственно в [Session.delete()](session_api.html#sqlalchemy.orm.Session.delete):

```
session.delete(somechild)
```

Здесь есть несколько возможностей:
- Если есть [relationship()](relationship_api.html#sqlalchemy.orm.relationship)from Parentto Child, но нет обратной связи, которая связывает конкретный объект с each , SQLAlchemy не будет знать, что при удалении этого конкретного объекта необходимо поддерживать «вторичную» таблицу, которая связывает его **с** . Никакого удаления «вторичной» таблицы не произойдет.ChildParentChildParent
- Если существует отношение, которое связывает конкретный объект Childс каждым Parent, предположим, что оно называется Child.parents, SQLAlchemy по умолчанию загрузит Child.parentsколлекцию, чтобы найти все Parentобъекты, и удалит каждую строку из «вторичной» таблицы, которая устанавливает эту связь. Обратите внимание, что эта связь не обязательно должна быть двунаправленной; SQLAlchemy строго следит за каждым, [relationship()](relationship_api.html#sqlalchemy.orm.relationship)связанным с Childудаляемым объектом.
- Здесь более эффективным вариантом является использование директив ON DELETE CASCADE с внешними ключами, используемыми базой данных. Предполагая, что база данных поддерживает эту функцию, можно настроить саму базу данных на автоматическое удаление строк в «дополнительной» таблице по мере удаления ссылающихся строк в «дочерней». Child.parents В этом случае SQLAlchemy можно указать отказаться от активной загрузки коллекции с помощью [relationship.passive_deletes](relationship_api.html#sqlalchemy.orm.relationship.params.passive_deletes) директивы on [relationship()](relationship_api.html#sqlalchemy.orm.relationship); см. [Использование каскада внешнего ключа ON DELETE с отношениями](cascades.html#passive-deletes) ORM для получения более подробной информации об этом.
  
  Еще раз обратите внимание, что это поведение относится *только* к [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary)параметру, используемому с [relationship()](relationship_api.html#sqlalchemy.orm.relationship). При работе с ассоциативными таблицами, которые отображены явно и *не* представлены в [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary) опции соответствующего [relationship()](relationship_api.html#sqlalchemy.orm.relationship), вместо этого можно использовать каскадные правила для автоматического удаления объектов в ответ на удаление связанного объекта — см. [Каскады](cascades.html#unitofwork-cascades) для получения информации об этой функции.
  
  Смотрите также
  
  [Использование каскадного удаления с отношениями «многие ко многим»](cascades.html#cascade-delete-many-to-many)
  
  [Использование внешнего ключа ON DELETE с отношениями «многие ко многим»](cascades.html#passive-deletes-many-to-many)
## Объект ассоциации  [](#association-object)

Шаблон ассоциативного объекта — это вариант «многие ко многим»: он используется, когда ассоциативная таблица содержит дополнительные столбцы помимо тех, которые являются внешними ключами для родительской и дочерней (или левой и правой) таблиц, столбцы, которые наиболее идеально отображаются на свои собственный отображаемый класс ORM. Этот сопоставленный класс сопоставляется с классом [Table](../core/metadata.html#sqlalchemy.schema.Table), который в противном случае был бы отмечен как [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary)при использовании шаблона «многие ко многим».

В шаблоне объекта ассоциации [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary) параметр не используется; вместо этого класс сопоставляется непосредственно с таблицей ассоциаций. Затем две отдельные [relationship()](relationship_api.html#sqlalchemy.orm.relationship)конструкции связывают сначала родительскую сторону с сопоставленным классом ассоциации через один ко многим, а затем сопоставленный класс ассоциации с дочерней стороной через многие к одному, чтобы сформировать однонаправленное отношение объекта ассоциации от родителя к ассоциации , ребенку. Для двунаправленной связи [relationship()](relationship_api.html#sqlalchemy.orm.relationship) используются четыре конструкции, чтобы связать сопоставленный класс ассоциации как с родительским, так и с дочерним в обоих направлениях.

Пример ниже иллюстрирует новый класс Association, который отображается на [Table](../core/metadata.html#sqlalchemy.schema.Table)named association; эта таблица теперь включает дополнительный столбец с именем extra_data, представляющий собой строковое значение, которое сохраняется вместе с каждой связью между Parentи Child. Сопоставляя таблицу с явным классом, рудиментарный доступ из Parentк Childявно использует Association:

```
**from** **typing** **import** Optional

**from** **sqlalchemy** **import** ForeignKey
**from** **sqlalchemy** **import** Integer
**from** **sqlalchemy.orm** **import** Mapped
**from** **sqlalchemy.orm** **import** mapped_column
**from** **sqlalchemy.orm** **import** DeclarativeBase
**from** **sqlalchemy.orm** **import** relationship

**class** **Base**(DeclarativeBase):
  **pass**

**class** **Association**(Base):
  __tablename__ = "association_table"
  left_id: Mapped[int] = mapped_column(ForeignKey("left_table.id"), primary_key=**True**)
  right_id: Mapped[int] = mapped_column(
      ForeignKey("right_table.id"), primary_key=**True**
  )
  extra_data: Mapped[Optional[str]]
  child: Mapped["Child"] = relationship()

**class** **Parent**(Base):
  __tablename__ = "left_table"
  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List["Association"]] = relationship()

**class** **Child**(Base):
  __tablename__ = "right_table"
  id: Mapped[int] = mapped_column(primary_key=**True**)
```

Чтобы проиллюстрировать двунаправленную версию, мы добавляем еще две [relationship()](relationship_api.html#sqlalchemy.orm.relationship) конструкции, связанные с существующими с помощью [relationship.back_populates](relationship_api.html#sqlalchemy.orm.relationship.params.back_populates):

```
**from** **typing** **import** Optional

**from** **sqlalchemy** **import** ForeignKey
**from** **sqlalchemy** **import** Integer
**from** **sqlalchemy.orm** **import** Mapped
**from** **sqlalchemy.orm** **import** mapped_column
**from** **sqlalchemy.orm** **import** DeclarativeBase
**from** **sqlalchemy.orm** **import** relationship

**class** **Base**(DeclarativeBase):
  **pass**

**class** **Association**(Base):
  __tablename__ = "association_table"
  left_id: Mapped[int] = mapped_column(ForeignKey("left_table.id"), primary_key=**True**)
  right_id: Mapped[int] = mapped_column(
      ForeignKey("right_table.id"), primary_key=**True**
  )
  extra_data: Mapped[Optional[str]]
  child: Mapped["Child"] = relationship(back_populates="parents")
  parent: Mapped["Parent"] = relationship(back_populates="children")

**class** **Parent**(Base):
  __tablename__ = "left_table"
  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List["Association"]] = relationship(back_populates="parent")

**class** **Child**(Base):
  __tablename__ = "right_table"
  id: Mapped[int] = mapped_column(primary_key=**True**)
  parents: Mapped[List["Association"]] = relationship(back_populates="child")
```

Работа с шаблоном ассоциации в его прямой форме требует, чтобы дочерние объекты были связаны с экземпляром ассоциации до того, как они будут присоединены к родителю; аналогично доступ от родителя к дочернему идет через объект ассоциации:

```
*# create parent, append a child via association*
p = Parent()
a = Association(extra_data="some data")
a.child = Child()
p.children.append(a)

*# iterate through child objects via association, including association*
*# attributes*
**for** assoc **in** p.children:
  print(assoc.extra_data)
  print(assoc.child)
```

Чтобы улучшить шаблон объекта ассоциации, чтобы прямой доступ к Associationобъекту был необязательным, SQLAlchemy предоставляет расширение [прокси-ассоциации](extensions/associationproxy.html) . Это расширение позволяет настраивать атрибуты, которые будут иметь доступ к двум «переходам» с одним доступом, один «переход» к связанному объекту и второй к целевому атрибуту.

Смотрите также

[Прокси-ассоциация](extensions/associationproxy.html) — обеспечивает прямой доступ в стиле «многие ко многим» между родителем и дочерним элементом для сопоставления объектов ассоциации с тремя классами.

Предупреждение

Избегайте прямого смешивания шаблона объекта ассоциации с шаблоном [«многие ко многим»](#relationships-many-to-many) , так как это создает условия, при которых данные могут считываться и записываться непоследовательным образом без специальных шагов; [прокси ассоциации](extensions/associationproxy.html) обычно используется для предоставления более краткого доступа. Дополнительные сведения о предостережениях, связанных с этой комбинацией, см. в следующем разделе « [Объединение объекта ассоциации с шаблонами доступа «многие ко многим»»](#association-pattern-w-m2m) .
### Объединение объекта ассоциации с шаблонами доступа «многие ко многим  [](#combining-association-object-with-many-to-many-access-patterns)

Как упоминалось в предыдущем разделе, шаблон объекта ассоциации не интегрируется автоматически с использованием шаблона «многие ко многим» для одних и тех же таблиц/столбцов одновременно. Из этого следует, что операции чтения могут возвращать конфликтующие данные, а операции записи также могут пытаться сбросить конфликтующие изменения, вызывая либо ошибки целостности, либо неожиданные вставки или удаления.

Для иллюстрации в приведенном ниже примере настраивается двунаправленная связь «многие ко многим» между Parentи Childчерез Parent.childrenи Child.parents. В то же время также настраивается отношение объекта ассоциации между и :Parent.child_associations -> Association.childChild.parent_associations -> Association.parent

```
**from** **typing** **import** Optional

**from** **sqlalchemy** **import** ForeignKey
**from** **sqlalchemy** **import** Integer
**from** **sqlalchemy.orm** **import** Mapped
**from** **sqlalchemy.orm** **import** mapped_column
**from** **sqlalchemy.orm** **import** DeclarativeBase
**from** **sqlalchemy.orm** **import** relationship

**class** **Base**(DeclarativeBase):
  **pass**

**class** **Association**(Base):
  __tablename__ = "association_table"

  left_id: Mapped[int] = mapped_column(ForeignKey("left_table.id"), primary_key=**True**)
  right_id: Mapped[int] = mapped_column(
      ForeignKey("right_table.id"), primary_key=**True**
  )
  extra_data: Mapped[Optional[str]]

  *# association between Assocation -> Child*
  child: Mapped["Child"] = relationship(back_populates="parent_associations")

  *# association between Assocation -> Parent*
  parent: Mapped["Parent"] = relationship(back_populates="child_associations")

**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)

  *# many-to-many relationship to Child, bypassing the `Association` class*
  children: Mapped[List["Child"]] = relationship(
      secondary="association_table", back_populates="parents"
  )

  *# association between Parent -> Association -> Child*
  child_associations: Mapped[List["Association"]] = relationship(
      back_populates="parent"
  )

**class** **Child**(Base):
  __tablename__ = "right_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)

  *# many-to-many relationship to Parent, bypassing the `Association` class*
  parents: Mapped[List["Parent"]] = relationship(
      secondary="association_table", back_populates="children"
  )

  *# association between Child -> Association -> Parent*
  parent_associations: Mapped[List["Association"]] = relationship(
      back_populates="child"
  )
```

При использовании этой модели ORM для внесения изменений изменения, сделанные в, Parent.childrenне будут скоординированы с изменениями, внесенными в Parent.child_associationsили Child.parent_associationsв Python; в то время как все эти отношения будут продолжать нормально функционировать сами по себе, изменения в одном не будут отображаться в другом до тех пор, пока не [Session](session_api.html#sqlalchemy.orm.Session)истечет срок действия, что обычно происходит автоматически после [Session.commit()](session_api.html#sqlalchemy.orm.Session.commit).

Кроме того, если вносятся конфликтующие изменения, такие как добавление нового Associationобъекта, а также добавление того же самого, связанного Childс Parent.children, это вызовет ошибки целостности, когда продолжится процесс сброса единицы работы, как в примере ниже:

```
p1 = Parent()
c1 = Child()
p1.children.append(c1)

*# redundant, will cause a duplicate INSERT on Association*
p1.child_associations.append(Association(child=c1))
```

Добавление Childнапрямую Parent.childrenтакже подразумевает создание строк в associationтаблице без указания какого-либо значения для association.extra_dataстолбца, который получит NULLза свое значение.

Можно использовать сопоставление, подобное приведенному выше, если вы знаете, что делаете; может быть веская причина использовать отношения «многие ко многим» в случае, когда шаблон «ассоциативный объект» используется нечасто, то есть легче загружать отношения по одному отношению «многие ко многим», что также может оптимизировать немного лучше, как «вторичная» таблица используется в операторах SQL, по сравнению с тем, как используются два отдельных отношения к явному классу ассоциации. По крайней мере, рекомендуется применить [relationship.viewonly](relationship_api.html#sqlalchemy.orm.relationship.params.viewonly)параметр к «вторичному» отношению, чтобы избежать проблемы возникновения конфликтующих изменений, а также предотвратить NULLзапись в дополнительные столбцы ассоциации, как показано ниже:

```
**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)

  *# many-to-many relationship to Child, bypassing the `Association` class*
  children: Mapped[List["Child"]] = relationship(
      secondary="association_table", back_populates="parents", viewonly=**True**
  )

  *# association between Parent -> Association -> Child*
  child_associations: Mapped[List["Association"]] = relationship(
      back_populates="parent"
  )

**class** **Child**(Base):
  __tablename__ = "right_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)

  *# many-to-many relationship to Parent, bypassing the `Association` class*
  parents: Mapped[List["Parent"]] = relationship(
      secondary="association_table", back_populates="children", viewonly=**True**
  )

  *# association between Child -> Association -> Parent*
  parent_associations: Mapped[List["Association"]] = relationship(
      back_populates="child"
  )
```

Приведенное выше сопоставление не будет записывать никаких изменений в базу данных Parent.childrenили Child.parentsв базу данных, предотвращая конфликтные записи. Однако операции чтения Parent.childrenили Child.parentsне обязательно будут совпадать с данными, считанными из Parent.child_associationsили Child.parent_associations, если в эти коллекции вносятся изменения в рамках одной и той же транзакции или [Session](session_api.html#sqlalchemy.orm.Session)когда читаются коллекции только для просмотра. Если отношения ассоциативных объектов используются нечасто и тщательно организованы для кода, который обращается к коллекциям «многие ко многим», чтобы избежать устаревших чтений (в крайних случаях прямое использование для [Session.expire()](session_api.html#sqlalchemy.orm.Session.expire) обновления коллекций в текущей транзакции), образец может быть осуществимым.

Популярной альтернативой приведенному выше шаблону является тот, в котором прямые отношения «многие ко многим» Parent.childrenи Child.parents отношения заменяются расширением, которое будет прозрачно проксировать через Association класс, сохраняя при этом все согласованное с точки зрения ORM. Это расширение известно как [прокси ассоциации](extensions/associationproxy.html) .

Смотрите также

[Прокси-ассоциация](extensions/associationproxy.html) — обеспечивает прямой доступ в стиле «многие ко многим» между родителем и дочерним элементом для сопоставления объектов ассоциации с тремя классами.
## Поздняя оценка аргументов отношений  [](#late-evaluation-of-relationship-arguments)

Большинство примеров в предыдущих разделах иллюстрируют сопоставления, в которых различные [relationship()](relationship_api.html#sqlalchemy.orm.relationship)конструкции ссылаются на свои целевые классы, используя строковое имя, а не сам класс, например [Mapped](internals.html#sqlalchemy.orm.Mapped), при использовании создается прямая ссылка, которая существует во время выполнения только как строка:

```
**class** **Parent**(Base):
  *# ...*

  children: Mapped[List["Child"]] = relationship(back_populates="parent")

**class** **Child**(Base):
  *# ...*

  parent: Mapped["Parent"] = relationship(back_populates="children")
```

Точно так же при использовании неаннотированных форм, таких как неаннотированные декларативные или императивные сопоставления, строковое имя также поддерживается непосредственно конструкцией [relationship()](relationship_api.html#sqlalchemy.orm.relationship):

```
registry.map_imperatively(
  Parent,
  parent_table,
  properties={"children": relationship("Child", back_populates="parent")},
)

registry.map_imperatively(
  Child,
  child_table,
  properties={"parent": relationship("Parent", back_populates="children")},
)
```

Эти строковые имена разрешаются в классы на этапе разрешения преобразователя, который представляет собой внутренний процесс, который обычно происходит после определения всех отображений и обычно запускается при первом использовании самих отображений. Объект [registry](mapping_api.html#sqlalchemy.orm.registry)— это контейнер, в котором эти имена хранятся и разрешаются в отображаемые классы, на которые они ссылаются.

В дополнение к аргументу основного класса для [relationship()](relationship_api.html#sqlalchemy.orm.relationship), другие аргументы, которые зависят от столбцов, присутствующих в пока еще неопределенном классе, также могут быть указаны либо как функции Python, либо, чаще, как строки. Для большинства этих аргументов, за исключением основного аргумента, входные строки оцениваются **как выражения Python с использованием встроенной в Python функции eval()** , поскольку они предназначены для получения полных выражений SQL.

Предупреждение

Поскольку функция Python eval()используется для интерпретации строковых аргументов с поздней оценкой, переданных в [relationship()](relationship_api.html#sqlalchemy.orm.relationship)конструкцию конфигурации преобразователя, эти аргументы **не** следует переназначать таким образом, чтобы они могли получить ненадежный пользовательский ввод; eval()не **защищен** от ненадежного пользовательского ввода.

Полное пространство имен, доступное в этой оценке, включает все классы, сопоставленные для этой декларативной базы, а также содержимое пакета sqlalchemy , включая функции выражений, такие как [desc()](../core/sqlelement.html#sqlalchemy.sql.expression.desc)и sqlalchemy.sql.functions.func:

```
**class** **Parent**(Base):
  *# ...*

  children: Mapped[List["Child"]] = relationship(
      order_by="desc(Child.email_address)",
      primaryjoin="Parent.id == Child.parent_id",
  )
```

В случае, когда более чем один модуль содержит класс с одним и тем же именем, имена строковых классов также могут быть указаны как пути с указанием модуля в любом из этих строковых выражений:

```
**class** **Parent**(Base):
  *# ...*

  children: Mapped[List["myapp.mymodel.Child"]] = relationship(
      order_by="desc(myapp.mymodel.Child.email_address)",
      primaryjoin="myapp.mymodel.Parent.id == myapp.mymodel.Child.parent_id",
  )
```

В примере, подобном приведенному выше, строка, переданная в, [Mapped](internals.html#sqlalchemy.orm.Mapped) может быть устранена из определенного аргумента класса, передав [relationship.argument](relationship_api.html#sqlalchemy.orm.relationship.params.argument)также строку местоположения класса напрямую. Ниже показан импорт только для ввода для Child, в сочетании со спецификатором времени выполнения для целевого класса, который будет искать правильное имя в [registry](mapping_api.html#sqlalchemy.orm.registry):

```
**import** **typing**

**if** typing.TYPE_CHECKING:
  **from** **myapp.mymodel** **import** Child

**class** **Parent**(Base):
  *# ...*

  children: Mapped[List["Child"]] = relationship(
      "myapp.mymodel.Child",
      order_by="desc(myapp.mymodel.Child.email_address)",
      primaryjoin="myapp.mymodel.Parent.id == myapp.mymodel.Child.parent_id",
  )
```

Квалифицированный путь может быть любым частичным путем, который устраняет двусмысленность между именами. Например, чтобы устранить неоднозначность между myapp.model1.Childи myapp.model2.Child, мы можем указать model1.Childили model2.Child:

```
**class** **Parent**(Base):
  *# ...*

  children: Mapped[List["Child"]] = relationship(
      "model1.Child",
      order_by="desc(mymodel1.Child.email_address)",
      primaryjoin="Parent.id == model1.Child.parent_id",
  )
```

Конструкция [relationship()](relationship_api.html#sqlalchemy.orm.relationship)также принимает функции Python или лямбда-выражения в качестве входных данных для этих аргументов. Функциональный подход Python может выглядеть следующим образом:

```
**import** **typing**

**from** **sqlalchemy** **import** desc

**if** typing.TYPE_CHECKING:
  **from** **myapplication** **import** Child

**def** _resolve_child_model():
  **from** **myapplication** **import** Child

  **return** Child

**class** **Parent**(Base):
  *# ...*

  children: Mapped[List["Child"]] = relationship(
      _resolve_child_model,
      order_by=**lambda**: desc(_resolve_child_model().email_address),
      primaryjoin=**lambda**: Parent.id == _resolve_child_model().parent_id,
  )
```

Полный список параметров, которые принимают функции/лямбды или строки Python, которые будут переданы eval():
- [relationship.order_by](relationship_api.html#sqlalchemy.orm.relationship.params.order_by)
- [relationship.primaryjoin](relationship_api.html#sqlalchemy.orm.relationship.params.primaryjoin)
- [relationship.secondaryjoin](relationship_api.html#sqlalchemy.orm.relationship.params.secondaryjoin)
- [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary)
- [relationship.remote_side](relationship_api.html#sqlalchemy.orm.relationship.params.remote_side)
- [relationship.foreign_keys](relationship_api.html#sqlalchemy.orm.relationship.params.foreign_keys)
- [relationship._user_defined_foreign_keys](relationship_api.html#sqlalchemy.orm.relationship.params._user_defined_foreign_keys)
  
  Предупреждение
  
  Как указывалось ранее, вышеуказанные параметры оцениваются [relationship()](relationship_api.html#sqlalchemy.orm.relationship) как **выражения кода Python с использованием eval(). НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЕ ВВОДНЫЕ СВЕДЕНИЯ В ЭТИ АРГУМЕНТЫ.**
### Добавление  отношений  к сопоставленным классам после объявления  [](#adding-relationships-to-mapped-classes-after-declaration)

Следует также отметить, что аналогично тому, как описано в [Добавление дополнительных столбцов к существующему декларативному сопоставленному классу](declarative_tables.html#orm-declarative-table-adding-columns) , любая [MapperProperty](internals.html#sqlalchemy.orm.MapperProperty) конструкция может быть добавлена ​​к декларативному базовому сопоставлению в любое время (отмечая, что аннотированные формы не поддерживаются в этом контексте). Если бы мы хотели реализовать это [relationship()](relationship_api.html#sqlalchemy.orm.relationship)после того, как Address класс стал доступен, мы могли бы также применить его позже:

```
*# first, module A, where Child has not been created yet,*
*# we create a Parent class which knows nothing about Child*

**class** **Parent**(Base):
  ...

*# ... later, in Module B, which is imported after module A:*

**class** **Child**(Base):
  ...

**from** **module_a** **import** Parent

*# assign the User.addresses relationship as a class variable.  The*
*# declarative base class will intercept this and map the relationship.*
Parent.children = relationship(Child, primaryjoin=Child.parent_id == Parent.id)
```

Как и в случае с сопоставленными столбцами ORM, тип аннотации не может [Mapped](internals.html#sqlalchemy.orm.Mapped)участвовать в этой операции; поэтому связанный класс должен быть указан непосредственно внутри [relationship()](relationship_api.html#sqlalchemy.orm.relationship)конструкции либо как сам класс, либо как строковое имя класса, либо как вызываемая функция, которая возвращает ссылку на целевой класс.

Примечание

Как и в случае с сопоставленными столбцами ORM, присвоение сопоставленных свойств уже сопоставленному классу будет работать правильно только в том случае, если используется «декларативный базовый» класс, то есть определяемый пользователем подкласс или динамически сгенерированный класс, [DeclarativeBase](mapping_api.html#sqlalchemy.orm.DeclarativeBase)возвращаемый [declarative_base()](mapping_api.html#sqlalchemy.orm.declarative_base) или [registry.generate_base()](mapping_api.html#sqlalchemy.orm.registry.generate_base). Этот «базовый» класс включает в себя метакласс Python, в котором реализован специальный __setattr__()метод, перехватывающий эти операции.

Назначение во время выполнения атрибутов сопоставленного класса сопоставленному классу **не** будет работать, если класс сопоставлен с помощью декораторов, таких как [registry.mapped()](mapping_api.html#sqlalchemy.orm.registry.mapped) или императивные функции, такие как [registry.map_imperatively()](mapping_api.html#sqlalchemy.orm.registry.map_imperatively).
### Использование формы с поздней оценкой для «вторичного» аргумента «многие ко многим  [](#using-a-late-evaluated-form-for-the-secondary-argument-of-many-to-many)

Отношения «многие ко многим » используют [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary)параметр, который обычно указывает ссылку на обычно не отображаемый [Table](../core/metadata.html#sqlalchemy.schema.Table) объект или другой объект Core, доступный для выбора. Поддерживается поздняя оценка с использованием вызываемого лямбда-выражения или строкового имени, где разрешение строк работает путем оценки заданного выражения Python, которое связывает имена идентификаторов с объектами с теми же именами, которые [Table](../core/metadata.html#sqlalchemy.schema.Table)присутствуют в той же [MetaData](../core/metadata.html#sqlalchemy.schema.MetaData)коллекции, на которую ссылается текущий [registry](mapping_api.html#sqlalchemy.orm.registry).

Для примера, приведенного в [Many To Many](#relationships-many-to-many) , если мы предположили, что association_table [Table](../core/metadata.html#sqlalchemy.schema.Table)объект будет определен в модуле позже, чем сам сопоставленный класс, мы можем написать использование [relationship()](relationship_api.html#sqlalchemy.orm.relationship) лямбда как:

```
**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List["Child"]] = relationship(
      "Child", secondary=**lambda**: association_table
  )
```

Или, чтобы проиллюстрировать поиск того же [Table](../core/metadata.html#sqlalchemy.schema.Table)объекта по имени, имя [Table](../core/metadata.html#sqlalchemy.schema.Table)используется в качестве аргумента. С точки зрения Python это выражение Python оценивается как переменная с именем «association_table», которая разрешается по именам таблиц в [MetaData](../core/metadata.html#sqlalchemy.schema.MetaData)коллекции:

```
**class** **Parent**(Base):
  __tablename__ = "left_table"

  id: Mapped[int] = mapped_column(primary_key=**True**)
  children: Mapped[List["Child"]] = relationship(secondary="association_table")
```

Предупреждение

При передаче в виде строки [relationship.secondary](relationship_api.html#sqlalchemy.orm.relationship.params.secondary)аргумент интерпретируется с использованием eval()функции Python, хотя обычно это имя таблицы. **НЕ ПЕРЕДАВАЙТЕ НЕДОВЕРЕННЫЙ ВВОД В ЭТУ СТРОКУ** .
-