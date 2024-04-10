# DragonECS-Vault
Данный репозитопий это сборник заметок и рекомендаций по коду для фреймворка [DragonECS](https://github.com/DCFApixels/DragonECS). 
> Материал расположенный в этом репозитории не является сводом правил, не обязателен к ознакомлению, и имеет только рекомендательный характер.

# Стиль кода

Пример стиля
```csharp
class ApplyVelocitySystem : IEcsRun, IEcsInject<EcsDefaultWorld>, IEcsInject<TimeService>
{
    class Aspect : EcsAspect
    {
        public EcsPool<Pose> poses = Inc;
        public EcsPool<Velocity> velocities = Inc;
        public EcsTagPool<FreezedTag> freezedTags = Exc;
    }

    public void Inject(EcsDefaultWorld obj) => _world = obj;
    public void Inject(TimeService obj) => _time = obj;

    EcsDefaultWorld _world;
    TimeService _time;

    public void Run()
    {
        foreach (var e in _world.Where(out Aspect a))
        {
            a.poses.Get(e).position += a.velocities.Get(e).value * _time.DeltaTime;
        }
    }
}
```
Тот же пример, но с Auto-Injections：
```csharp
class ApplyVelocitySystem : IEcsRun
{
    class Aspect : EcsAspect
    {
        public EcsPool<Pose> poses = Inc;
        public EcsPool<Velocity> velocities = Inc;
        public EcsTagPool<FreezedTag> freezedTags = Exc;
    }

    [EcsInject] EcsDefaultWorld _world;
    [EcsInject] TimeService _time;

    public void Run()
    {
        foreach (var e in _world.Where(out Aspect a))
        {
            a.poses.Get(e).position += a.velocities.Get(e).value * _time.DeltaTime;
        }
    }
}
```


Начну с объявления аспектов. Аспекты, хоть и могут использоваться одновременно несколькими системами, удобнее всего объявлять для каждой системы свои прямо внутри систем. 

Именование полей в Аспектах. Поля для кеша пулов называть по названию компонента во множественном числе，например `EcsPool<Health> healths`. Для этого пулы фейково реализуют IEnumerable<T>, чтобы автодополненте IDE предлагало такое наименование.

Модификаторы доступа private/public для членов систем не имеют применения, все взаимодейсвие с системами происходит либо косвенно через данные в компонентах, а если напрямую, то через интерфейсы. Поэтому для сокращения бойлерплейта и улучшения восприятия модификаторы доступа могут быть опушены где это возможно.

Именование. 

Именование аспектов. Многие системы работают только с одним аспектом, так что аспекты можно называть просто `Aspect`. Если аспектов несколько то основной также называть `Aspect` а второстепенный с префиксом, например `EventAspect`. 

Именование переменных. По той же причине что системы часто работают только с одним аспектом, возвращаемый запросом `Where` экземпляр аспекта можно называть просто `a`, а сущности внутри `foreach` просто `e`, например: `foreach(var e in _world.Where(out Aspect a)`. Аналогично именованию аспектов, если система работает с несколькими аспектами, то к `a` и `e` добавляется префикс, например: `foreach(var eventE in _world.Where(out EventAspect eventA)`.

# Модули
Группы систем и компонетов объединенные одной логикой или реализующие некую фичу лучше ораганизовывать в модули. Модули можно рассматривать как  отедльные сборки, можно даже разбивать их на отдельные сборки.
Папка одного модуля имеет следующую иерархию:
```
.../
------+
      | ModuleName/
      +------+
      |      | Components/
      |      +------+
      |      |      | SomeComponent1.cs
             |      | SomeComponent2.cs
             |       
             | Events/
             +------+
             |      | SomeEvent1.cs
             |      | SomeSignal1.cs
             |       
             | Systems/
             +------+
             |      | SomeSystem1.cs
             |      | SomeSystem2.cs
             |       
             | ModuleName.cs
```
+ `SomeComponent.cs` - обычные компоненты или компоненты-теги.
+ `SomeSignal.cs` - компонент-событие крепящийся к цели события.
+ `SomeEvent.cs` - компонент-событие используемый в сущности-событие, для создания нескольких событий для одной сущности.
+ `SomeSystem.cs` - Системы.
+ `ModuleName.cs` - Класс реализующий интерфейс `IEcsModule`, добавляющий в пайплайн системы модуля.

Для систем и компонентов присущих одному модулю добавлять мета-атрибут MetaGroup, в качесве корневого каталога группы использовать название модуля. Пример:
```c#
// Слово модуль из SomeModule будет автоматически удаленно, останется только Some
[MetaGroup(nameof(SomeModule))]
public struct SomeComponent : IEcsComponent
{
   //...
}
```
