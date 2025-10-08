## Практическая работа 2. Парсинг HTML и консолидация данных

**Студент:** *Нургалеева Гузель*

**Вариант:** №16

**В связи с тем, что запрос к сайту dns-shop.ru выдавал ошибку 401, что, возможно, связано с правами доступа к сайту (*также пробовала спарсить mvideo, technopark, но там была аналогичная ситуация*), анализировала данные с сайта GeekBrains.ru. Это - образовательная платформа, на которой представлены курсы в том числе по аналитике и программированию.**

**Бизнес-кейс:** анализ курсов, предоставляемых GeekBrains, и скрейпинг статей, размещенных на сайте

**Источник:** заменен на: Каталог курсов на gb.ru (3-5 страниц)

**Задача:** собрать данные о названии курсов, предоставялемых скидках на курс, продолжительности обучения. По окончании каких курсов есть услуга по помощи в трудоуствройстве, в описании каких курсов есть информация о потенциальной зарплате и какой разброс зарплат. По собранным статьям определить самые просматриваемые, какие статьи входили в топ-5 в разные годы публикации

## Часть 1. Сбор данных 
### Сбор данных по предоставляемым курсам
<br> Для парсинга используется BeautifulSoup и библиотека запросов requests
<br> каталог курсов находится на странице 'https://gb.ru/courses/all' и имеет вид набора карточек. Разбивки на страницы в разделе курсов нет, как и перехода через кнопку "загрузить еще" (тому подобное). Все курсы размещены на 1 странице. Парсинг на нескольких страницах будет реализован при парсинге статей.

<img width="596" height="407" alt="image" src="https://github.com/user-attachments/assets/6fd9ed30-8c6d-4fb2-9ca4-e0ab13c7b116" />

<br> по карточкам курсов собирается следующая информация:
- наименование
- описание
- продолжительность обучения
- предоставляемая скидка на стоимость обучения

<br> все параметры находятся в своих классах, поэтому собирарлись по отдельности. Для них через интсрументы разработчка были 
определены следующие классы:

        # Находим все карточки вакансий на странице
            cources_name = soup.find_all('a', class_='card_full_link')
            cources_body = soup.find_all('div', class_='direction-card__body')
            cources_info_duration = soup.find_all('div', class_= 'direction-card__info-text ui-text-body--6 --margin-bot')
            cources_info_discount = soup.find_all('div', class_= 'direction-card__info')

<img width="1846" height="876" alt="image" src="https://github.com/user-attachments/assets/323aeaf0-e4e5-47e3-8d58-7824824baa70" />


<img width="1785" height="914" alt="image" src="https://github.com/user-attachments/assets/d72db8e5-1ae8-48a0-94ed-c7f04842b957" />


<br> Далее был проимзведен парсинг по каждому параметру, с последующим сбором в один датасет:

      for card in cources_name:
            # Используем try-except для устойчивости парсера
            try:
                title = card.find('span', class_= 'direction-card__title-text ui-text-body--1 ui-text--medium').text.strip()
            except AttributeError:
                title = 'Не указано'

            cources_title.append(
                title
            )
        for card in cources_body:
            try:
                description = card.find('div', class_= 'direction-card__text') .text.strip()
            except AttributeError:
                description = 'Не указано'

            cources_description.append(
                description
            )

        for card in cources_info_duration:
            try:
                duration = card.find('span', class_= 'ui-text--medium').text.strip() 
            except AttributeError:
                duration = 'Не указано'

            cources_duration.append(
                duration
            )
        for card in cources_info_discount:
            try:
                discount = card.find('div', class_= 'direction-card__info-label ui-text-heading--5 ui-text--medium gb-landings-product-discount').text.strip() 
            except AttributeError:
                discount = 'Не указано'


<img width="682" height="320" alt="image" src="https://github.com/user-attachments/assets/2db4ef11-8629-4cd1-b5fb-c6fb43a05ee5" />

### Сбор данных по опубликованным статьям
<br> Для парсинга используется BeautifulSoup и библиотека запросов requests
<br> статьи находятся на странице 'https://gb.ru/posts' , разнесены по страницам и имеют вид набора карточек. Было прочитано 5 страниц. 

<img width="491" height="425" alt="image" src="https://github.com/user-attachments/assets/d343326e-c865-4b6d-aa30-350954c359cd" />

<br> по статьям собиралась следующая информация:
- заголовок
- количество просмотров
- количество комментариев
- дата публикации

  Для парсинга по страницам кодировка url была приведена к той, которая применяется на сайте

  <img width="272" height="48" alt="image" src="https://github.com/user-attachments/assets/95bd101a-37ac-4f92-a1dc-ed0d731590f8" />

  Собранные данные были оформлены в датасет


  <img width="682" height="320" alt="image" src="https://github.com/user-attachments/assets/b93a3a85-bab6-44f6-a6b1-cf90be76182f" />


  ## Часть 2. Обработка данных
<br> Для того, чтобы можно было произвести анализ данных их нужно сначала обработать (убрать пропуски, убрать лишние символы, преобразовать к нужному типу данных).
<br> Для этого в том числе был использован regex

**в случае с курсами:**
- будем рассматривать среднюю, минимальную и максимальную потенциальную зарплату, указанную в карточках.
- среднюю, минимальную и максимальную скидку
- средний, минимальный и максимальный срок обучения.

Для этого в случае с зарплатой нужно извлечь нужные данные из текста, в случае со скидкой и сроком обучения - очистить данные от лишних символов.

Данные по зарплате:

  <img width="953" height="314" alt="image" src="https://github.com/user-attachments/assets/724c4dcb-a97d-4f6f-81f6-1cde4c00fea1" />
  

                # очистка данных в столбце duartion, чтобы можно было посмотреть, оценить курсы по продолжительности
                # убираем при помощи regex все символы в кириллице и пробелы
                cources_df.loc[:,'duration'] = [re.sub('\s[\u0401\u0451\u0410-\u044f]+', '', i) for i in cources_df['duration']] 
                # убираем знак %
                cources_df.loc[:,'discount'] = cources_df['discount'].replace(r'\%', '', regex = True)
                # убираем символы + и -
                cources_df.loc[:,'discount'] = cources_df['discount'].replace(r'[+-]', '', regex = True) 
                
                # также очистим от лишних символов названия курсов
                # в наименовании встречаются служебные символы, убираем их
                cources_df.loc[:,'title'] = cources_df['title'].replace(r'\xa0', ' ', regex = True)
                cources_df.loc[:,'title'] = cources_df['title'].replace(r'\xad', '-', regex = True) 
                
                # также очистим от лишних символов описание курсов
                cources_df['description'] = cources_df['description'].replace(r'\xa0', ' ', regex = True) 
                
                # выведем в отдельный столбец признак 'Помощь с трудоустройством'
                cources_df.loc[cources_df['description'].str.contains("Помощь с трудоустройством: да"), 'job search help'] = 1
                
                # выведем в отдельный столбец данные о потенциальной зарплате
                # при помощи regex ищем 5-6 идущих подряд чисел (например,250000)
                 # Идущих подряд, т.к. ищем в строке из которой убрали все пробелы [re.sub('\s+', '', i) for i in cources_df['description']]
                cources_df['potential_salary'] = [re.findall(r'\d{5,6}', i) for i in [re.sub('\s+', '', i) for i in cources_df['description']]] 
                                                                                         
                # преобразование в число кодом ниже, т.к. код выше дает список строк
                for i in range(len(cources_df)):
                      if len(cources_df['potential_salary'][i]) == 0:
                          cources_df['potential_salary'][i] = int(0)
                      else:
                          cources_df['potential_salary'][i] = int(cources_df['potential_salary'][i][0])

Получили очищенные данные, пригодные для анализа
        
<img width="703" height="300" alt="image" src="https://github.com/user-attachments/assets/c62bde6f-f400-4f4f-91a1-f11d94256c0c" />
        

**в случае со статьями:**
- приведем дату к нужному формату при помощи regex
- 
<br> Выведем в отдельный столбец название месяца на кириллице
<br> Создадим словарь с номерами месяцев
<br> создадим столбец с датой и преобразуем в формат дат
            
        posts_df.loc[:,'months_cyrilc'] = [re.findall('[\u0401\u0451\u0410-\u044f]+', i) for i in posts_df['date']]

        for i in range(len(posts_df)):
            posts_df['months_cyrilc'][i] = posts_df['months_cyrilc'][i][0]
        
        mapping = dict({"января": 1,
                   "февраля": 2,
                   "марта": 3,
                   "апреля": 4,
                   "мая": 5,
                   "июня": 6,
                   "июля": 7,
                   "августа": 8,
                   "сентября": 9,
                   "октября": 10,
                   "ноября": 11,
                   "декабря": 12})
        
        posts_df['months_num'] = posts_df['months_cyrilc'].map(mapping)
        
        posts_df['date_adj'] = 0
        for i in range(len(posts_df)):
            posts_df['date_adj'][i] = posts_df['date'][i].split()[2]+'/'+posts_df['months_num'][i].astype(str)+'/'+posts_df['date'][i].split()[0]
        
        posts_df['date_adj'] = pd.to_datetime(posts_df['date_adj'], format= "mixed")

  ## Часть 3. Анализ и визуализация
  
  **Курсы**


  <img width="855" height="354" alt="image" src="https://github.com/user-attachments/assets/12db641c-1ca1-4ccd-94da-1f3869fbf06e" />

- бОльшая часть курсов имеет продолжитнельность 12 мес
- помощь в поиске работы предлагается почти в половине курсов
- все курсы идут со скидкой, в основном в размере 45%
- всего 69 курсов


<img width="936" height="636" alt="image" src="https://github.com/user-attachments/assets/1b3cd482-7bd9-48af-9bb5-c6054bf44f73" />

**Опубликованные статьи**


<img width="780" height="462" alt="image" src="https://github.com/user-attachments/assets/385907c3-a4bf-401a-8bfb-c1417176d88c" />

#### Вывод
В результате выполнения практической работы были освоены продвинутые техники сбора данных путем
парсинга HTML-страниц, их последующей консолидации из различных источников и проведения комплексного аналитического исследования для
решения прикладных бизнес-задач.


                      
