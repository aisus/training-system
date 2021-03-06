Cистема сценариев для игр/интерактивных визуализаций на Unity.
Позволяет создавать сложные сценарии и последовательности действий для прохождения уровня в редакторе Unity без написания дополнительного кода/конфигураций.
Реализовано на Unity 2019.4.8f1

[Скачать сборку для Windows x64](https://github.com/aisus/training-system/releases/tag/v1.0)

![alt text](/readme-images/TitleImage.png "TitleImage")

## Принципы работы
### Объекты
Интерактивный сценарий можно разбить на последовательность атомарных операций с объектами в сцене. 
Каждый объект в описанных сценариях может находиться в двух основных состояниях:

* Неактивен (__Inactive__)
* Активен (__Active__)

Примеры: 
* Лампочка. __Inactive__ - выключена, __Active__ - включена.
* Сменная часть устройства. __Inactive__ - лежит на столе, __Active__ - установлена в устройство. 

Операции с объектами производятся кликами мыши, клик переводит объект из неактивного в активное состояние и наоборот, если это позволяется типом объекта.

Типы объектов:
* Одноразовое использование (__UseOnce__). После активации становится недоступным для взаимодействия. Пример - провод, который необходимо перерезать.
* Одна активация (__Trigger__). Активируется один раз. Повторное взаимодействие возможно, но не меняет состояние. Пример - кнопка пожарной сигнализации.
* Переключатель (__Switch__). Сохраняет состояние. Примеры - выключатель, крышка, сменная часть устройства.

### Cценарии 

Сценарий состоит из этапов, каждый из которых подразумевает выполнение определенных операций с объектами в сцене. 
Сценарии строго последовательны, и в текущей реализации не допускают ветвления или отмены этапов. 

Этап сценария содержит условие перехода на следующий этап. Условие представляет собой требуемое состояние объекта или набора объектов. Также, набор разрешенных для взаимодействия объектов на каждом этапе сценария ограничивается. 

Пример сценария:

Открыть защелку на крышке устройства, открыть крышку устройства, нажать красную кнопку, закрыть крышку устройства, закрыть защелку. 
1. Доступно: Защелка (__Switch__). Условие: Защелка открыта (__Active__)
1. Доступно: Защелка, крышка (__Switch__). Условие: Крышка открыта (__Active__)
1. Доступно: Защелка, крышка, десять кнопок разных цветов (__Trigger__). Условие: Красная кнопка нажата (__Active__)
1. Доступно: Защелка, крышка, десять кнопок разных цветов. Условие: Крышка закрыта (__Inactive__)
1. Доступно: Защелка, крышка. Условие: Защелка закрыта (__Inactive__)

На каждом этапе взаимодействие с объектом, не входящим в условия перехода, приводит к провалу сценария и сообщению об ошибочном выполнении.

## Детали реализации проекта

Для перемещения камеры в сцене используется FPS CharacterController из Standard Assets. 

Выделение границ объектов при наведении камеры реализовано шейдерами [UltimateOutline](https://github.com/Shrimpey/UltimateOutline).

Сценарии действий и списки названий объектов настраиваются как ScriptableObject ассеты.

Все ассеты и ресурсы сценариев хранятся в [__\Assets\TrainingSystem\TrainingScenarios__](https://github.com/aisus/training-system/tree/master/Assets/TrainingSystem/TrainingScenarios)



## Создание интерактивных объектов и сценариев

Интерактивными объектами являются GameObject в сцене, имеющие компоненты Collider, Animator и __InteractiveBehaviour__. __InteractiveBehaviour__ связан с __InteractiveObjectEntity__ - данными объекта. В __InteractiveObjectEntity__ задается строковый идентификатор объекта, тип взаимодействия, текущее состояние и доступность для действий. Объект должен иметь слой __Interactive__. 

![alt text](/readme-images/InteractiveBehaviour.png "InteractiveBehaviour")

При произведении действия с __InteractiveBehaviour__ активируются триггеры его Animator - Activate, Deactivate. Рекомендуется создавать AnimationController для новых объектов как AnimationOverrideController. В проекте созданы три базовых Animator для трех типов объектов - AnimatorBase_Trigger, AnimatorBase_Switch, AnimatorBase_UseOnce.

Сценарий создается через CreateAssetMenu.
В созданном ScriptableObject можно настраивать этапы выполнения с соответствующими условиями перехода и списками активируемых/деактивируемых при переходе на этап объектов. Используются строковые ключи, соответствующие заданным в __InteractiveBehaviour__.

![alt text](/readme-images/TrainingScenario.png "TrainingScenario")

Отображаемые при наведении камеры названия объектов задаются в ScriptableObject __DisplayedObjectNames__. 

![alt text](/readme-images/DisplayedObjectNames.png "DisplayedObjectNames")

После подготовки префаба набора интерактивных объектов, сценария и отображаемых названий созданные ассеты объединяются в ScriptableObject __ScenarioPreferences__.

![alt text](/readme-images/Preferences.png "ScenarioPreferences")

__ScenarioPreferences__, добавленные в компонент __TrainingPreferencesInitializer__ в сцене __MainMenu__, становятся доступными для запуска. 

![alt text](/readme-images/Initializer.png "PreferencesInitializer")




