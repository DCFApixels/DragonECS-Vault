# DragonECS-Vault
Данный репозитопий это сборник заметок и рекомендаций по коду для фреймворка [DragonECS](https://github.com/DCFApixels/DragonECS). Материал расположенный в этом репозитории не является сводом правил, не обязателен к ознакомлению, и имеет только рекомендательный характер.

# Стиль кода
Начну с объявления аспектов. Аспекты, хоть и могут использоваться одновременно несколькими системами, удобнее всего объявлять для каждой системы свои прямо внутри систем. 

Иименование полей в Аспектах. Поля для кеша пулов называть по названию компонента во множественном числе，например `EcsPool<Health> healths`. Для этого пулы фейково реализуют IEnumerable<T>, чтобы автодополненте IDE предлагало такое наименование.

Модификаторы доступа private/public для членов систем не имеют применения, все взаимодейсвие с системами происходит либо косвенно через данные в компонентах, а если напрямую, то через интерфейсы. Поэтому для сокращения бойлерплейта и улучшения восприятия модификаторы доступа могут быть опушены где это возможно.

Именование. 

Именование аспектов. Многие системы работают только с одним аспектом, так что аспекты можно нызывать просто `Aspect`. Если аспектов несколько то основной также называть `Aspect` а второстепенный с префиксом, например `EventAspect`. 

Именование переменных. По той же причине что системы часто работают только с одним аспектом, возвращаемый запросом `Where` экземпляр аспекта можно называть просто `a`, а сущности внутри `foreach` просто `e`, например: `foreach(var e in _world.Where(out Aspect a)`. Аналогично именованию аспектов, если система работает с несколькими аспектами, то к `a` и `e` добавляется префикс, например: `foreach(var eventE in _world.Where(out EventAspect eventA)`
