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



              
