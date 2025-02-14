# Лабораторная работа №3
Тема: Использование принципов проектирования на уровне методов и классов

Цель работы: Получить опыт проектирования и реализации модулей с использованием принципов KISS, YAGNI, DRY, SOLID и др.

## Диаграмма прецедентов
Выбранный вариант использования — Выполнение трансформации вида "Модель-Текст" — выделен на диаграмме прецедентов. 

Описание прецедента: применив к модели разработанные правила трансформации, можно получить программный код на соответствующем языке программирования для реализации визуализации данных по правилам, заданным пользователем, разработавшим модель. 

![Image](https://github.com/user-attachments/assets/1c1c4452-48f3-4e39-84db-6c51440e0d75)

## Диаграмма контейнеров
Основные контейнеры: 
* Подсистема работы с моделями - используется для создания и редактирования моделей, а также осуществления их трансформаций вида "Модель-Текст". 
* Многоаспектная онтология (база знаний) - ядро всей системы. Наличие базы знаний позволяет автоматизировать процесс построения метамоделей DSL посредством проекции онтологии предметной области на онтологию визуальных языков. 
* Браузер моделей - используется для взаимодействия с базой знаний и перевода онтологического представления моделей во внутреннее представление, а также взаимодействия с модулями импорта и экспорта файлов, трансформации и интерпретации моделей. 
* Среда генерации кода - используется для перевода построенной модели в код на языке программирования. Генерация кода необходима для реализации разработанной модели визуализации данных.
  
![Image](https://github.com/user-attachments/assets/a5be3344-a0f6-4cb9-8143-afa2d236232d)
*Контейнеры, которые будут представлены на диаграмме последовательностей, выделены коричневым цветом*

## Диаграмма компонентов
Основные компоненты: 
* Модуль трансформации моделей - используется для трансформации моделей на основе заданных правил трансформации типа "Модель-Текст" (генерация программного кода, реализующего модель). 

![Image](https://github.com/user-attachments/assets/7e268d14-f4f6-4490-a74b-f7b481ba5305)
*Компоненты, которые будут представлены на диаграмме последовательностей, выделены коричневым цветом*

## Диаграмма последовательностей
Диаграмма учитывает альтернативные сценарии, например, ситуацию, когда отсутствуют правила трансформации в многоаспектной онтологии. Это обеспечивает корректное поведение системы.
      
![Image](https://github.com/user-attachments/assets/bf347eb5-418c-4e0d-9038-80da1b95f42b)


## Модель базы знаний
Основные классы и их роли:
* Model – конкретная модель, созданная на основе определённой метамодели.
* MetaModel и Object – представляют метамодель визуального языка и её элементы, которые используются для создания моделей.
* Property – содержит свойства, которыми могут обладать элементы метамодели и сами метамодели в целом.
* TextLanguage и NonTerminalSymbol – описывают текстовые языки (например, Python, R) и их грамматику через нетерминальные символы.
* TransformationRule – связывает элементы метамодели с нетерминальными символами конструкций текстовых языков, формируя правила трансформации. Каждое правило состоит из двух частей – статической и динамической. Статическая часть одинакова для всех моделей, разработанных с использованием языка, определяемого метамоделью, а динамическая использует информацию, «извлеченную» из конкретной модели.

![Image](https://github.com/user-attachments/assets/16e3fc38-b190-418d-b68d-e7ce95c90298)
*На диаграмме отражены только те классы онтологии, которые необходимы для реализации рассматриваемого процедента.*

## Применение основных принципов разработки
### KISS (Keep it stupid simple)
* Принцип: «пусть всё будет простым до безобразия». Простой код и простой дизайн уменьшают риск ошибок.
* **Фрагмент №1**: Простой обход графа модели через вложенную функцию
  ```python
  class TransformationEngine:
    def generate_code(self, model: Model, rule: TransformationRule) -> str:
        code_lines = []
        def traverse(node: GraphNode):
            code_lines.append(rule.apply_to_node(node))
            for child in node.children:
                traverse(child)
        traverse(model.root)
        return "\n".join(code_lines)
  ```

* Объяснение: Вложенная функция ``traverse`` отвечает исключительно за рекурсивный обход графа. Такой подход делает код кратким и легко понимаемым, так как логика обхода сосредоточена в одном месте.
* **Фрагмент №2**: Применение правила трансформации к узлу с простым форматированием
  ```python
  def apply_to_node(self, node: GraphNode) -> str:
    try:
        dynamic_code = self.dynamic_template.format(**node.attributes)
    except KeyError as e:
        dynamic_code = f"<отсутствует значение для {e}>"
    return f"{self.static_template}\n{dynamic_code}\n"
  ```

* Объяснение: Логика формирования динамической части шаблона реализована через простое форматирование строки, что упрощает реализацию и снижает вероятность ошибок.

### YAGNI (You Aren't Gonna Need It)
* Принцип: реализуем только необходимый функционал, избегая избыточных возможностей и ненужных усложнений.
* **Фрагмент №1**: Минимальная реализация динамической части правила трансформации
  ```python
  def apply_to_node(self, node: GraphNode) -> str:
    try:
        dynamic_code = self.dynamic_template.format(**node.attributes)
    except KeyError as e:
        dynamic_code = f"<отсутствует значение для {e}>"
    return f"{self.static_template}\n{dynamic_code}\n"
  ```

* Объяснение: Вместо разработки сложного механизма генерации динамической части кода, используется простое форматирование строки с проверкой наличия всех необходимых атрибутов. Это решает задачу, не усложняя архитектуру.
* **Фрагмент №2**: Локальный обход графа
  ```python
  def generate_code(self, model: Model, rule: TransformationRule) -> str:
    code_lines = []
    def traverse(node: GraphNode):
        code_lines.append(rule.apply_to_node(node))
        for child in node.children:
            traverse(child)
    traverse(model.root)
    return "\n".join(code_lines)
  ```

* Объяснение: Обход графа реализован локально внутри функции ``generate_code``, что исключает избыточное создание отдельных классов или функций для обхода, если задача этого не требует.
  
### DRY (Don't Repeat Yourself)
* Принцип: избегание дублирования кода повышает его читаемость и снижает вероятность ошибок при внесении изменений.
* **Фрагмент №1**: Единая реализация применения правила
  ```python
  class TransformationRule:
    def __init__(self, id, static_template, dynamic_template, left_part, right_part):
        self.id = id
        self.static_template = static_template
        self.dynamic_template = dynamic_template
        self.left_part = left_part
        self.right_part = right_part

    def apply_to_node(self, node: GraphNode) -> str:
        try:
            dynamic_code = self.dynamic_template.format(**node.attributes)
        except KeyError as e:
            dynamic_code = f"<отсутствует значение для {e}>"
        return f"{self.static_template}\n{dynamic_code}\n"
  ```

* Объяснение: Вся логика применения правила к узлу сосредоточена в одном методе ``apply_to_node``, который можно использовать для всех узлов. Это исключает повторение однотипного кода в разных местах.

### SOLID.
#### Single Responsibility (Единая ответственность)
* Принцип: каждый класс или функция выполняет одну конкретную задачу.
* Фрагмент кода:
  ```python
  class TransformationEngine:
    def generate_code(self, model: Model, rule: TransformationRule) -> str:
        # Единственная ответственность: обход графа и объединение сгенерированного кода
        code_lines = []
        def traverse(node: GraphNode):
            code_lines.append(rule.apply_to_node(node))
            for child in node.children:
                traverse(child)
        traverse(model.root)
        return "\n".join(code_lines)
  ```

* Объяснение: ``TransformationEngine`` занимается только обходом графа и сбором кода, а логика применения правила содержится в ``TransformationRule ``.
  
#### Open-Closed (Открытость для расширения, закрытость для модификации)
* Принцип: легко можно добавлять новые типы правил или изменять поведение, не меняя базовой логики.
* Фрагмент кода:
  ```python
  class TransformationRule:
    def apply_to_node(self, node: GraphNode) -> str:
        # Можно переопределить этот метод в наследниках для расширения функциональности
        try:
            dynamic_code = self.dynamic_template.format(**node.attributes)
        except KeyError as e:
            dynamic_code = f"<отсутствует значение для {e}>"
        return f"{self.static_template}\n{dynamic_code}\n"
  ```

* Объяснение: При необходимости можно создать подкласс, который переопределит метод ``apply_to_node``, не затрагивая логику генерации кода в ``TransformationEngine``.
  
#### Liskov Substitution (Принцип подстановки Барбары Лисков)
* Принцип: любой объект, реализующий требуемый интерфейс, может быть использован вместо базового класса.
* Фрагмент кода:
  ```python
  class CustomTransformationRule(TransformationRule):
    def apply_to_node(self, node: GraphNode) -> str:
        # Дополнительная логика для специфичных случаев
        base_code = super().apply_to_node(node)
        return base_code + "// Дополнительный код для специфики\n"
  ```

* Объяснение: Объекты  ``CustomTransformationRule`` можно использовать в ``TransformationEngine`` так же, как объекты базового класса ``TransformationRule``, без изменения логики обхода графа.
  
#### Interface Segregation (Разделение интерфейса)
* Принцип: интерфейсы классов минимальны и сконцентрированы на конкретных задачах.
* Фрагмент кода:
  ```python
  class TransformationEngine:
    def generate_code(self, model: Model, rule: TransformationRule) -> str:
        # Использует только метод apply_to_node, не требуя дополнительных методов
        code_lines = []
        def traverse(node: GraphNode):
            code_lines.append(rule.apply_to_node(node))
            for child in node.children:
                traverse(child)
        traverse(model.root)
        return "\n".join(code_lines)
  ```

* Объяснение: ``TransformationEngine`` зависит только от одного метода (``apply_to_node``), что упрощает интерфейс и повышает гибкость.

#### Dependency Inversion (Инверсия зависимостей)
* Принцип: модули высокого уровня не зависят от модулей низкого уровня, а оба зависят от абстракций.
* Фрагмент кода:
  ```python
  class TransformationEngine:
    def generate_code(self, model: Model, rule: TransformationRule) -> str:
        # Движок зависит от абстрактного метода apply_to_node, а не от конкретной реализации
        code_lines = []
        def traverse(node: GraphNode):
            code_lines.append(rule.apply_to_node(node))
            for child in node.children:
                traverse(child)
        traverse(model.root)
        return "\n".join(code_lines)
  ```

* Объяснение: Движок генерации кода использует абстракцию (метод ``apply_to_node``) для выполнения трансформации, что позволяет подменять конкретные реализации правил без изменения логики генерации.

## Дополнительные принципы разработки
### BDUF (Big Design Up Front)
**Описание:** Масштабное проектирование системы до начала ее реализации, когда архитектурные решения детально продумываются заранее.  
**Применимость.**  
**Обоснование:** Так как проект требует создания методологии, которая позволит вносить дополнительные методы со временем, архитектура должна быть тщательно спроектирована заранее. Если не продумать это на старте, система окажется трудно расширяемой, что снизит ее ценность.  

### SoC (Separation of Concerns)
**Описание:**  Принцип разделения системы на независимые модули, каждый из которых выполняет строго определенную функцию.
**Применимость.**  
**Обоснование:**  Проект включает работу с онтологиями, трансформацию модели в код и визуализацию данных. Разделение ответственности между этими частями повысит гибкость, упростит поддержку, тестирование и последующее расширение системы.

**Пример из кода**:
 ```python
 class ModelTransformer:
    def __init__(self, transformation_rules):
        self.rules = transformation_rules

    def apply_transformation(self, model):
        transformed_code = ""
        for rule in self.rules:
            transformed_code += rule.apply_rule(model)
        return transformed_code

 class CodeGenerator:
    def __init__(self, language):
        self.language = language

    def generate_code(self, transformed_data):
        return f"{self.language} code:\n{transformed_data}"
  ```
(Этот код разделяет трансформацию модели и генерацию кода, реализуя принцип SoC.)

### MVP (Minimum Viable Product)
**Описание:**  Разработка минимально жизнеспособного продукта, который содержит только критически важные функции для проверки гипотез.
**Применимость.**  
**Обоснование:** Основная цель проекта — доказательство осуществимости предлагаемого подхода. Разработка MVP позволит быстро проверить ключевые идеи, продемонстрировать работу системы экспертам и скорректировать архитектуру на ранних этапах.

**Пример из кода**:
 ```python
 # Минимальная реализация трансформации модели в код
 model = Model(id="1", name="SampleModel")
 rule = TransformationRule(id="rule1", staticTemplate="plot(", dynamicTemplate="data)", applyRule=lambda model: 
 "plot(data)")
 transformer = ModelTransformer([rule])
 transformed_code = transformer.apply_transformation(model)
 print(transformed_code)  # Минимальный вывод кода для проверки работоспособности
 ```
(Этот фрагмент кода демонстрирует минимальную реализацию, достаточную для проверки идеи.)

### PoC (Proof of Concept)
**Описание:** Прототип или демонстрационная версия, предназначенная для проверки технологической осуществимости ключевых решений.
**Применимость.**  
**Обоснование:** Проект направлен на исследование и доказательство работоспособности предложенной концепции. PoC позволит выявить возможные ограничения, продемонстрировать ключевые технологии и сформировать основу для дальнейшей разработки.

**Пример из кода**:
 ```python
 # Доказательство концепции генерации кода на Python из модели
 example_model = Model(id="2", name="ExampleModel")
 example_rule = TransformationRule(id="rule2", staticTemplate="plt.", dynamicTemplate="show()", applyRule=lambda model: 
 "plt.show()")
 transformer = ModelTransformer([example_rule])
 generated_code = transformer.apply_transformation(example_model)
 print(f"Generated Python code: {generated_code}")
 ```
(Этот код доказывает, что концепция преобразования модели в код на Python технически реализуема.)

Таким образом, все 4 принципа разработки (BDUF, SoC, MVP, PoC) применимы в рамках проекта. 
