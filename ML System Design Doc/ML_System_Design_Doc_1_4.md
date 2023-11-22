# ML System Design Doc - [RU]
## Дизайн ML системы - nanozymes_ai (MVP). вер.1.4 </h1>

Текущий этап проектирования ML системы - выкатывание сервиса в продакшн
Тестирование проведено
Разработка моделей построения эмбеддингов заершена, проведена оптимизация
Архитектура решения спроектирована
Бизнес-требования описаны.
Проблемное интервью проведено

### 1. Цели и предпосылки
#### 1.1. Зачем идем в разработку продукта?
- Бизнес-цель `Product Owner`  Сократить  время ученых и инженеров на изучение больших текстов из доменной области
   - ученых-химиков на изучение статей - на 50%

- Почему станет лучше, чем сейчас, от использования ML `Product Owner` & `Data Scientist` Data Scientist 
В каждой статье от 10 до 50 страниц, на изучение одной статьи тратится до 4 часов. Разрабатываемый сервис должен позволять уменьшать это время до 2 часов за счет information retrieval текста.

- Что будем считать успехом итерации с точки зрения бизнеса `Product Owner` Сокращение до 2 часов времени на выделение из научных статей описания способа получения химического соединения.
  
#### 1.2. Бизнес-требования и ограничения
- Краткое описание БТ и ссылки на детальные документы с бизнес-требованиями `Product Owner` _здесь должна быть ссылка на требования_
Создаем сервис, позволяющий в научных статьях выделять данные, отвечающие на вопрос пользователя. Например, сервис должен найти в тексте статьи инструкцию по синтезу химического вещества, информацию о каталитической активности и т.п.

Ограничения:

Нужно получать информацию только из содержимого научных статей, без подмешивания других данных. Это позволит сохранить достоверность ответа с научной точки зрения.

- Бизнес-ограничения `Product Owner`  Лучше, когда модель не находит нужных данных, чем когда находит их неверно.

- Что мы ожидаем от конкретной итерации `Product Owner`. Сервис может дать ответ на любой вопрос относительно любой статьи из базы данных https://dizyme.aicidlab.itmo.ru/database/
  
- Описание бизнес-процесса пилота, насколько это возможно - как именно мы будем использовать модель в существующем бизнес-процессе? `Product Owner`
  > - Срок тестирования - 2 дня.
  > - Группа тестирования - бизнес-заказчик и команда продукта.
  > - Тестирование UX согласно user case. После каждой итерации распознавания пользователь ставит субъективную оценку точности распознавания от 1 до 10.
  > - Будет протестировано не менее 10 статей.
   
- Что считаем успешным пилотом? Критерии успеха и возможные пути развития проекта `Product Owner`
  > - Количество субъективных оценок распознавания выше 5 баллов составляет не менее 75%.


#### 1.3. Что входит в скоуп проекта/итерации, что не входит
- На закрытие каких БТ подписываемся в данной итерации `Data Scientist`  Система распознавания заданного типа текста (параметры синтеза нанозимов) в научных статьях. 
- Что не будет закрыто `Data Scientist`  Сквозной поиск ответа на запрос по всем статьям из базы данных
- Описание результата с точки зрения качества кода и воспроизводимости решения `Data Scientist`
  Данные, загружаемые пользователем, будут **временно*** храниться в векторной базе данных.
Модель будет **дообучаться*** итеративно. 

Для предикта текст статьи
> - делится на фрагменты длиной N символов 
> - фрагменты передаются модели multilingual-e5-large для построения эмбеддингов
> - строится той же моделью эмбеддинг запроса пользователя
> - выбирается k фрагментов текста, косинусное расстояние между эмбеддингом которыми и эмбеддингом запроса наименьшее
> - к каждому из k фрагментов добавляется q фрагментов до и столько же после
> - k таких наборов данныхв формате JSON по API передаются модели YandexGPT
> - Возвращается ответ в формате JSON или .xml для отображения в области распознанного текста

*****конкретные параметры будут уточнены позднее
  
- Описание планируемого технического долга (что оставляем для дальнейшей продуктивизации) `Data Scientist` Извлечение числовых параметров, Расширение количества форматов документов для распознавания (epub, сканы документов). Классификация других типов статей (не по синтезу химических веществ). 
Проверка прав на использование контента при его загрузке

  
#### 1.4. Предпосылки решения
- Описание всех общих предпосылок решения, используемых в системе – с обоснованием от запроса бизнеса: какие блоки данных используем, горизонт прогноза, гранулярность модели, и др. `Data Scientist`
  Используем только тексты научных статей из базы данных

### 2. Методология Data Scientist
#### 2.1. Постановка задачи
- Что делаем с технической точки зрения `Data Scientist`  Извлечение данных из текста будет производиться на основе языковых моделей
Для извлечения эмбеддингов из фрагментов текста выбрана модель multilingual-e5-large. Критериями выбора являются качество и производительность. Сравнение производилось на лидерборде https://github.com/avidale/encodechka
Бейзлайн для оценки качества модели в текущем проекте - сравнение с качеством работы человека.
Целевые метрики работы модели:
1) Распознавание в тексте статьи описания. Precision - 90%
2) Полнота - количество правильно распознанных слов в описании. Recall - 80%.

- Блок-схема для бейзлайна и основного MVP с ключевыми этапами решения задачи: подготовка данных, построение прогнозных моделей, оптимизация, тестирование, закрытие технического долга, подготовка пилота, другое. `Data Scientist` 
Блок-схема решения (часть 1: схема работы сервиса)
_здесь должна быть схема_1_

Блок-схема решения (часть 2: этапы проекта)

_здесь должна быть схема_2_


#### 2.3. Этапы решения задачи `Data Scientist`  

- Для каждого этапа **по результатам EDA** описываем - **отдельно для бейзлайна** и **отдельно для основного MVP** - все про данные и технику решения максимально конкретно. Обозначаем необходимые вводные, технику предполагаемого решения и что ожидаем получить на выходе, чтобы перейти к следующему этапу.  
- Как правило, детальное и структурированное заполнение раздела `2.3` возможно только **по результатам EDA**.  
- Если описание в дизайн доке **шаблонно** - т.е. его можно скопировать и применить к разным продуктам, то оно **некорректно**. Дизайн док должен показывать схему решения для конкретной задачи, поставленной в части 1.  
    
> Примеры этапов:  
> - Этап 2 - Подготовка прогнозных моделей  
> - Этап 3 - Интерпретация моделей (согл. с заказчиком)  
> - Этап 4 - Интеграция бизнес правил для расчета бизнес-метрик качества мрдели  
> - Этап 5 - Подготовка инференса модели по итерациям    
> - Этап 6 - Интеграция бизнес правил  
> - Этап 7 - Разработка оптимизатора (выбор оптимальной итерации)  
> - Этап 8 - Подготовка финального отчета для бизнеса  

*Этап 1 - это обычно, подготовка данных.*  

В этом этапе должно быть следующее:  

- Данные и сущности, на которых будет обучаться ваша модель машинного обучения. Отдельная таблица для целевой переменной (либо целевых переменных разных этапов), отдельная таблица – для признаков.  
  
| Название данных  | Есть ли данные в компании (если да, название источника/витрин) | Требуемый ресурс для получения данных (какие роли нужны) | Проверено ли качество данных (да, нет) |
| ------------- | ------------- | ------------- | ------------- |
| Продажи | DATAMARTS_SALES_PER_DAY  | DE/DS | + |
| ...  | ...  | ... | ... |
 
- Краткое описание результата этапа - что должно быть на выходе: витрины данных, потоки данных, др.  
  
> Чаще всего заполнение раздела невозможно без EDA.

 *Этапы 2 и далее, помимо подготовки данных.*
 
Описание техники **для каждого этапа** должно включать описание **отдельно для MVP** и **отдельно для бейзлайна**:  

- Описание формирования выборки для обучения, тестирования и валидации. Выбор репрезентативных данных для экспериментов, обучения и подготовки пилота (от бизнес-цели и репрезентативности данных с технической точки зрения) `Data Scientist`    
- Горизонт, гранулярность, частоту необходимого пересчета прогнозных моделей `Data Scientist`   
- Определение целевой переменной, согласованное с бизнесом `Data Scientist`   
- Какие метрики качества используем и почему они связаны с бизнес-результатом, обозначенным `Product Owner` в разделах `1` и `3`. Пример - WAPE <= 50% для > 80% категорий, bias ~ 0. Возможна формулировка в терминах относительно бейзлайна, количественно. Для бейзлайна могут быть свои целевые метрики, а может их вообще не быть (если это обосновано) `Data Scientist`   
- Необходимый результат этапа. Например, необходимым результатом может быть не просто достижение каких-либо метрик качества, а включение в модели определенных факторов (флаг промо для прогноза выручки, др.) `Data Scientist`    
- Какие могут быть риски и что планируем с этим делать. Например, необходимый для модели фактор (флаг промо) окажется незначимым для большинства моделей. Или для 50% моделей будет недостаточно данных для оценки `Data Scientist`    
- Верхнеуровневые принципы и обоснования для: feature engineering, подбора алгоритма решения, техники кросс-валидации, интерпретации результата (если применимо).  
- Предусмотрена ли бизнес-проверка результата этапа и как будет проводиться `Data Scientist` & `Product Owner`  
  
### 3. Подготовка пилота  
  
#### 3.1. Способ оценки пилота  
  
- Краткое описание предполагаемого дизайна и способа оценки пилота `Product Owner`, `Data Scientist` with `AB Group` 
  
#### 3.2. Что считаем успешным пилотом  
  
Формализованные в пилоте метрики оценки успешности `Product Owner`   
  
#### 3.3. Подготовка пилота  
  
- Что можем позволить себе, исходя из ожидаемых затрат на вычисления. Если исходно просчитать сложно, то описываем этап расчетов ожидаемой вычислительной сложности на эксперименте с бейзлайном. И предусматриваем уточнение параметров пилота и установку ограничений по вычислительной сложности моделей. `Data Scientist` 

### 4. Внедрение `для production систем, если требуется`    

> Заполнение раздела 4 требуется не для всех дизайн документов. В некоторых случаях результатом итерации может быть расчет каких-то значений, далее используемых в бизнес-процессе для пилота.  
  
#### 4.1. Архитектура решения   
  
- Блок схема и пояснения: сервисы, назначения, методы API `Data Scientist`  
  
#### 4.2. Описание инфраструктуры и масштабируемости 
  
- Какая инфраструктура выбрана и почему `Data Scientist` 
- Плюсы и минусы выбора `Data Scientist` 
- Почему финальный выбор лучше других альтернатив `Data Scientist` 
  
#### 4.3. Требования к работе системы  
  
- SLA, пропускная способность и задержка `Data Scientist`  
  
#### 4.4. Безопасность системы  
  
- Потенциальная уязвимость системы `Data Scientist`  
  
#### 4.5. Безопасность данных   
  
- Нет ли нарушений GDPR и других законов `Data Scientist`  
  
#### 4.6. Издержки  
  
- Расчетные издержки на работу системы в месяц `Data Scientist`  
  
#### 4.5. Integration points  
  
- Описание взаимодействия между сервисами (методы API и др.) `Data Scientist`  
  
#### 4.6. Риски  
  
- Описание рисков и неопределенностей, которые стоит предусмотреть `Data Scientist`   
  