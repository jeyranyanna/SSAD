# Лабораторная работа №6
**Тема:** Использование шаблонов проектирования

**Цель работы:** Получить опыт применения шаблонов проектирования при написании кода программной системы.

## Шаблоны проектирования GoF
### Порождающие шаблоны 
#### 1. Singleton
- Назначение: гарантирует единственный экземпляр класса и глобальный доступ к нему.
- Применение в проекте: используется для управления доступом к базе знаний (многоаспектной онтологии).
- UML диаграмма:
  
  ![image](https://github.com/user-attachments/assets/cdd9c76b-52b0-4823-bdc0-6ebf47027b5a)

- Фрагмент программного кода
  ```python
  class Ontology:
      _instance = None
  
      def __new__(cls):
          if cls._instance is None:
              cls._instance = super(Ontology, cls).__new__(cls)
              cls._instance.data = {}  # База знаний
          return cls._instance
  
      def get_data(self):
          return self.data
  
  # Проверка
  ontology1 = Ontology()
  ontology2 = Ontology()
  
  print(ontology1 is ontology2)  # Выведет: True (один и тот же объект)
  ```

#### 2. Factory Method
- Назначение: определяет интерфейс для создания объектов, позволяя подклассам изменять тип создаваемого объекта.
- Применение в проекте: используется для создания различных моделей визуализации в зависимости от входных данных.
- UML диаграмма:

  ![image](https://github.com/user-attachments/assets/ef1d8a88-20b4-49cb-9fee-5b03fbe1e94f)



  
- Фрагмент программного кода:
  ```python
  class Visualization:
    def render(self):
        raise NotImplementedError

  class BarChart(Visualization):
      def render(self):
          print("Рисуем гистограмму")
  
  class PieChart(Visualization):
      def render(self):
          print("Рисуем круговую диаграмму")
  
  class VisualizationFactory:
      def create_visualization(self):
          raise NotImplementedError
  
  class BarChartFactory(VisualizationFactory):
      def create_visualization(self):
          return BarChart()
  
  class PieChartFactory(VisualizationFactory):
      def create_visualization(self):
          return PieChart()
  
  # Использование
  factory = BarChartFactory()
  chart = factory.create_visualization()
  chart.render()  # Выведет: Рисуем гистограмму
  ```

#### 3. Builder 
- Назначение: пошаговое создание сложных объектов.
- Применение в проекта: используется для пошагового построения сложных моделей визуализации.
- UML диаграмма:
  
  ![image](https://github.com/user-attachments/assets/78356b5f-c1b3-4d93-b345-43aee1e6bf24)

- Фрагмент программного кода:
  ```python
  class Visualization:
    def __init__(self, title, x_label, y_label, data):
        self.title = title
        self.x_label = x_label
        self.y_label = y_label
        self.data = data

    def render(self):
        print(f"Рисуем график: {self.title}")
        print(f"Ось X: {self.x_label}, Ось Y: {self.y_label}")
        print(f"Данные: {self.data}")

  class VisualizationBuilder:
      def set_title(self, title):
          raise NotImplementedError
  
      def set_axes(self, x_label, y_label):
          raise NotImplementedError
  
      def set_data(self, data):
          raise NotImplementedError
  
      def build(self):
          raise NotImplementedError
  
  class ConcreteVisualizationBuilder(VisualizationBuilder):
      def __init__(self):
          self.title = ""
          self.x_label = ""
          self.y_label = ""
          self.data = []
  
      def set_title(self, title):
          self.title = title
          return self
  
      def set_axes(self, x_label, y_label):
          self.x_label = x_label
          self.y_label = y_label
          return self
  
      def set_data(self, data):
          self.data = data
          return self
  
      def build(self):
          return Visualization(self.title, self.x_label, self.y_label, self.data)
  
  # Использование
  builder = ConcreteVisualizationBuilder()
  chart = (builder.set_title("Продажи по месяцам")
                 .set_axes("Месяц", "Доход")
                 .set_data([("Январь", 100), ("Февраль", 150)])
                 .build())
  
  chart.render()
  ```


### Структурные шаблоны 
#### 1. Adapter
- **Назначение**: конвертирует интерфейс класса в другой интерфейс, ожидаемый клиентом. Позволяет классам с разными интерфейсами работать вместе. 
- **Применение в проекте**: используется для адаптации различных форматов моделей (например, JSON, XML) к внутреннему представлению системы.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/0eaff14f-37cf-44c7-be0d-b74c80fe4a47)
- **Фрагмент программного кода**:
  ```python
  class Target:
    def request(self):
        raise NotImplementedError

  class Adaptee:
      def specific_request(self):
          return "Адаптированные данные из XML"
  
  class Adapter(Target):
      def __init__(self, adaptee):
          self.adaptee = adaptee
  
      def request(self):
          return self.adaptee.specific_request()
  
  adaptee = Adaptee()
  adapter = Adapter(adaptee)
  print(adapter.request())  # Выведет: Адаптированные данные из XML
  ```
  
#### 2. Bridge
- **Назначение**: разделяет абстракцию и реализацию так, чтобы они могли изменяться независимо. 
- **Применение в проекте**: используется для разделения логики работы с моделями и конкретных инструментов визуализации.
- **UML диаграмма**:

  ![image](https://github.com/user-attachments/assets/6e9a90c9-35d7-4218-a3d1-db4515fbaa55)
- **Фрагмент программного кода**:
  ```python
  class Renderer:
    def render(self, data):
        raise NotImplementedError

  class MatplotlibRenderer(Renderer):
      def render(self, data):
          print(f"Рисуем график с Matplotlib: {data}")
  
  class SeabornRenderer(Renderer):
      def render(self, data):
          print(f"Рисуем график с Seaborn: {data}")
  
  class Visualization:
      def __init__(self, renderer):
          self.renderer = renderer
  
      def draw(self, data):
          self.renderer.render(data)
  
  viz = Visualization(MatplotlibRenderer())
  viz.draw([1, 2, 3])  # Выведет: Рисуем график с Matplotlib: [1, 2, 3]
  ```
  
#### 3. Decorator
- **Назначение**: динамически предоставляет объекту дополнительные возможности. Представляет собой гибкую альтернативу наследованию для расширения функциональности. 
- **Применение в проекте**: используется для добавления новых свойств визуализации, таких как анимация, интерактивность.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/d44d3521-af6f-4872-9ed0-624d1835cc1c)
- **Фрагмент программного кода**:
  ```python
  class Visualization:
    def draw(self):
        raise NotImplementedError

  class BaseChart(Visualization):
      def draw(self):
          print("Отрисовка базового графика")
  
  class ChartDecorator(Visualization):
      def __init__(self, chart):
          self.chart = chart
  
      def draw(self):
          self.chart.draw()
  
  class InteractiveChart(ChartDecorator):
      def draw(self):
          super().draw()
          print("Добавление интерактивности")
  
  chart = InteractiveChart(BaseChart())
  chart.draw()
  ```
  
#### 4. Facade
- **Назначение**: предоставляет единый интерфейс к группе интерфейсов подсистемы. Определяет высокоуровневый интерфейс, делая подсистему проще для использования. 
- **Применение в проекта**: используется для упрощенного доступа к API трансформации моделей, валидации и экспорта.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/279a548d-86fd-4b60-ba84-736b0f8dbe9d)

- **Фрагмент программного кода**:
  ```python
  class ModelManager:
    def create_model(self, name):
        print(f"Создание модели {name}")

    def validate_model(self, name):
        print(f"Валидация модели {name}")

    def export_model(self, name, format):
        print(f"Экспорт модели {name} в {format}")

  class ModelFacade:
      def __init__(self):
          self.manager = ModelManager()
  
      def process_model(self, name):
          self.manager.create_model(name)
          self.manager.validate_model(name)
          self.manager.export_model(name, "JSON")
  
  facade = ModelFacade()
  facade.process_model("Диаграмма связей")
  ```



### Поведенческие шаблоны 
#### 1. Command 
- **Назначение**: инкапсулирует запрос в виде объекта, позволяя откладывать его выполнение.
- **Применение в проекте**:  используется для управления операциями API над моделями.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/5355553e-b9d3-4b1c-9f64-d16b08b73a1a)
- **Фрагмент программного кода**:
  ```python
  class Command:
    def execute(self):
        raise NotImplementedError

  class CreateModelCommand(Command):
      def execute(self):
          print("Создание модели")
  
  class DeleteModelCommand(Command):
      def execute(self):
          print("Удаление модели")
  
  class Invoker:
      def execute_command(self, command):
          command.execute()
  
  invoker = Invoker()
  invoker.execute_command(CreateModelCommand())  # Выведет: Создание модели
  ```
  
#### 2. Interpreter
- **Назначение**: получая формальный язык, определяет представление его грамматики и интерпретатор, использующий это представление для обработки выражений языка.
- **Применение в проекте**: используется для работы с DSL, разбора онтологических запросов.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/e2e908ac-0105-4d93-ba8d-d18402d51449)

- **Фрагмент программного кода**:
  ```python
  class Expression:
    def interpret(self, context):
        raise NotImplementedError

  class TerminalExpression(Expression):
      def interpret(self, context):
          return context.replace("SELECT", "ВЫБРАТЬ")
  
  class NonTerminalExpression(Expression):
      def interpret(self, context):
          return context.upper()
  
  expression = NonTerminalExpression()
  print(expression.interpret("select data"))  # Выведет: SELECT DATA
  ```

#### 3. Mediator
- **Назначение**: обеспечивает централизованное управление взаимодействием объектов.
- **Применение в проекте**: используется для координации взаимодействий между модулями системы.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/94ad2a3a-6696-4fc8-8572-02b4c4fa6875)

- **Фрагмент программного кода**:
  ```python
  class Mediator:
    def notify(self, sender, event):
        raise NotImplementedError

  class ConcreteMediator(Mediator):
      def notify(self, sender, event):
          print(f"Модуль {sender} отправил событие: {event}")
  
  class Component:
      def __init__(self, mediator):
          self.mediator = mediator
  
      def send_event(self, event):
          self.mediator.notify(self, event)
  
  mediator = ConcreteMediator()
  component = Component(mediator)
  component.send_event("Модель обновлена")
  ```

#### 4. Observer
- **Назначение**: определяет зависимость "один ко многим" между объектами так, что когда один объект меняет свое состояние, все зависимые объекты оповещаются и обновляются автоматически. 
- **Применение в проекте**: используется для обновления визуализаций при изменении данных.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/7dca652c-3614-45fb-81e2-8c3a8134fdc8)

- **Фрагмент программного кода**:
  ```python
  class Observer:
    def update(self, data):
        raise NotImplementedError

  class ConcreteObserver(Observer):
      def update(self, data):
          print(f"Получены новые данные: {data}")
  
  class Subject:
      def __init__(self):
          self.observers = []
  
      def add_observer(self, observer):
          self.observers.append(observer)
  
      def notify(self, data):
          for observer in self.observers:
              observer.update(data)
  
  subject = Subject()
  observer = ConcreteObserver()
  subject.add_observer(observer)
  subject.notify("Обновление визуализации")
  ```

#### 5. Strategy
- **Назначение**: позволяет выбрать алгоритм поведения во время выполнения.
- **Применение в проекте**: используется для выбора метода визуализации.
- **UML диаграмма**:
  
  ![image](https://github.com/user-attachments/assets/1684f6a8-2031-4733-8c70-af80e526795b)

- **Фрагмент программного кода**:
  ```python
  class Strategy:
    def execute(self):
        raise NotImplementedError

  class MatplotlibStrategy(Strategy):
      def execute(self):
          print("Рисуем график с Matplotlib")
  
  class SeabornStrategy(Strategy):
      def execute(self):
          print("Рисуем график с Seaborn")
  
  class Context:
      def __init__(self, strategy):
          self.strategy = strategy
  
      def set_strategy(self, strategy):
          self.strategy = strategy
  
      def execute_strategy(self):
          self.strategy.execute()
  
  context = Context(MatplotlibStrategy())
  context.execute_strategy()  # Выведет: Рисуем график с Matplotlib
  ```

## Шаблоны проектирования GRASP
### Роли (обязанности) классов
#### 1. Information Expert
- **Проблема**: Кто должен отвечать за предоставление информации о моделях и их свойствах?
- **Решение**: Назначить ответственность за предоставление информации тому классу, который обладает необходимыми данными.
- **Фрагмент кода**:
  (Используем `Ontology` из Singleton, расширив его методами получения данных.)
  ```python
  class Ontology:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(Ontology, cls).__new__(cls)
            cls._instance.data = {}  
        return cls._instance

    def get_data(self, key):
        return self.data.get(key, "Нет данных")

    def add_data(self, key, value):
        self.data[key] = value
  ```
- **Результат**:
  - Ontology отвечает за хранение и предоставление данных → уменьшает зависимость других классов от деталей хранения.
  - Класс выступает как центральное хранилище информации.
- **Связь с другими паттернами**:
  - **Singleton** – гарантирует единственность экземпляра.
  - **Low Coupling** – снижает зависимость других классов от деталей реализации хранения.
  
#### 2. Creator
- **Проблема**: Кто должен создавать объекты визуализаций?
- **Решение**: Назначить создание объекта тому классу, который использует его или владеет его данными.
- **Фрагмент кода**:
  (Добавляем создание модели в `VisualizationFactory` из Factory Method.)
  ```python
  class VisualizationFactory:
    def create_visualization(self, chart_type):
        if chart_type == "bar":
            return BarChart()
        elif chart_type == "pie":
            return PieChart()
        else:
            raise ValueError("Неизвестный тип диаграммы")
  ```
- **Результат**:
  - Централизованное создание объектов → упрощает поддержку кода.
  - Класс не зависит от конкретных типов диаграмм.
- **Связь с другими паттернами**:
  - **Factory Method** – отделяет создание объекта от его использования.
  - **Polymorphism** – позволяет работать с разными типами диаграмм через общий интерфейс.
  
#### 3. Controller
- **Проблема**: Как организовать управление между API и бизнес-логикой?
- **Решение**: Выделить класс-контроллер, который управляет процессами.
- **Фрагмент кода**:
  (Создаем `VisualizationController` для управления созданием диаграмм через API.)
  ```python
  class VisualizationController:
    def __init__(self, factory):
        self.factory = factory

    def create_chart(self, chart_type):
        chart = self.factory.create_visualization(chart_type)
        chart.render()
  ```
- **Результат**:
  - Контроллер управляет процессами → API вызывает только его, не зная деталей создания диаграмм.
- **Связь с другими паттернами**:
  - **Facade** – упрощает взаимодействие с системой.
  - **Indirection** – снижает зависимость API от конкретных классов диаграмм.
  
#### 4. Pure Fabrication
- **Проблема**: Как создать класс, не связанный напрямую с моделями, но обеспечивающий полезную логику?
- **Решение**: Создать отдельный класс, выполняющий логику без изменения бизнес-объектов.
- **Фрагмент кода**:
  (Создаем `ChartExporter`, который экспортирует диаграммы в JSON.)
  ```python
  import json

  class ChartExporter:
      @staticmethod
      def export(chart):
          data = {"type": chart.__class__.__name__, "data": "фейковые данные"}
          return json.dumps(data)
  ```
- **Результат**:
  - Упрощает поддержку → бизнес-объекты не содержат логику экспорта.
  - Можно расширять функционал экспорта без изменения диаграмм.
- **Связь с другими паттернами**:
  - **Decorator** – можно использовать, чтобы динамически добавлять экспорт в объекты диаграмм.
  
#### 5. Indirection
- **Проблема**: Как снизить зависимость классов друг от друга?
- **Решение**: Ввести класс-посредник, который управляет взаимодействием. 
- **Фрагмент кода**:
  (Расширяем `Mediator` для управления взаимодействием между `VisualizationController` и `ChartExporter`.)
  ```python
  class VisualizationMediator:
    def __init__(self, controller, exporter):
        self.controller = controller
        self.exporter = exporter

    def create_and_export_chart(self, chart_type):
        chart = self.controller.create_chart(chart_type)
        return self.exporter.export(chart)
  ```
- **Результат**:
  - Уменьшает жесткие связи между объектами.
  - API теперь взаимодействует только с посредником.
- **Связь с другими паттернами**:
  - **Mediator** – управляет взаимодействием объектов.
  - **Low Coupling** – снижает зависимость API от конкретных диаграмм.
  

### Принципы разработки
#### 1. Polymorphism
- **Используется в:**
  - Factory Method (создание диаграмм через общий интерфейс `Visualization`).
  - Strategy (выбор стратегии визуализации).
- **Решение**: Использование общего интерфейса `Visualization` для разных типов диаграмм.
  
#### 2. Low Coupling
- **Используется в:**
  - Indirection (VisualizationMediator) – API не зависит от конкретных диаграмм.
  - Pure Fabrication (ChartExporter) – диаграммы не зависят от экспорта.
- **Результат**: Компоненты системы слабо связаны → легче вносить изменения.
  
#### 3. High Cohesion
- **Используется в:**
  - `Ontology` – содержит только логику работы с базой знаний.
  - `ChartExporter` – отвечает только за экспорт.
- **Результат**: Классы содержат только свою ответственность → код становится понятнее.


### Свойство программы (цель)
#### 1. Protected Variations
- **Проблема**: Как сделать систему устойчивой к изменениям?
- **Решение**: Использование абстракций (`VisualizationFactory`, `Strategy`, `Observer`) и шаблонов **Polymorphism** и **Low Coupling**.
- **Результат**:
  - Добавление новых типов диаграмм не требует изменений в API.
  - Можно расширять экспорт без изменения диаграмм.
