### SendingMessages
Отложенное выполнение функции (рассылка сообщений) для группы лиц основываясь на часовом поясе.

## Вводные данные
1.  Предложите решение задачи двумя способами. Выберете один для реализации. Решение должно быть реализовано средствами python.

2.  Есть база данных, в которой имеется следующие данные:

>
| suppliers\_db.phone |  suppliers\_db.name  | district\_db.name  |
|---|---|---|
| +7 000 000 00 00   | ООО «Ромашка»  |  Воронежская  |
| +7 000 000 00 11   | ООО «Пупучик»  |  Амурская  |

3. Есть  модели sqlalchemy



```py
class Supplier(db.Model):
    __tablename__ = 'supplier'
    name = Column(TEXT(None, 'Cyrillic_General_CI_AS'), nullable=True)
    contact_person = Column(TEXT(None, 'Cyrillic_General_CI_AS'), nullable=True)
    inn = Column(String(15, 'Cyrillic_General_CI_AS'), nullable=True)
    storage_address = Column(TEXT(None, 'Cyrillic_General_CI_AS'))
    phone = Column(String(255, 'Cyrillic_General_CI_AS'))
    id = Column(Integer, primary_key=True)
    subscription_cancelled = Column(BIT, nullable=True, comment="Отписан ли от рассылки")
    subscription_admin = Column(BIT, nullable=True, comment="Отписан ли от рассылки админом")
    district_id = Column(ForeignKey('district.id'), nullable=True, comment="id области")
    district = relationship('District')
    area_id = Column(ForeignKey('area.id'), nullable=True, comment="id района")
    area = relationship('Area')
    manager_id = Column(ForeignKey('user.id'), nullable=True)
    manager = relationship('User')
    land_crop = relationship('LandCrop', secondary=supplier_land_crop, backref=backref('suppliers'))
    landuser = Column(String(255, 'Cyrillic_General_CI_AS'), nullable=True)

class District(db.Model):
    __tablename__ = 'district'
    id = Column(Integer, primary_key=True)
    name = Column(String(255, 'Cyrillic_General_CI_AS'), nullable=False, comment="Название области")
```


4. Есть список часовых поясов, в форме словаря {3:3, 7:9} котором имеются следующие данные:

>
| district\_db.id | utc  |
|---|---|
| 3   | 3  |
| 7   | 9  |  

5. Есть функция, которая запускает рассылку по запросу (но не по расписанию) пользователя.

##  Необходимый результат
    

При поступлении команды, необходимо, чтобы на следующий день в 10:00 началась рассылка по всем поставщикам, в зависимости от их часового пояса.

То есть, чтобы каждый поставщик, в зависимости от своей области, получил сообщение в десять часов по местному времени.

Пример:

В Воронежской области 21.10.2021 года, в 21:50 часов дана команда на рассылку.

В Амурской области в это время 22.10.2021 года 03:50 часов.

Получатели сообщения в Амурской области должны получить сообщение 22.10.2021 года в 10:00 часов (то есть 22.10.2021 в 04:00 часов).

Сервер должен продолжать работать и принимать сообщения.

## Решение
Предложим решение двумя способами. Главное отличие которых заключается в реализации отложенного вызова функции рассылки. Отличия приведены ниже.

**Первый способ.**

Для отложенного вызова  функции рассылки будем использовать инструменты: crontab - это хронологический демон-планировщик задач или python модуль sched. Время будем рассчитывать строго в UTC(+0). 

**Второй способ.**

Для отложенного вызова  функции рассылки будем использовать инструменты:
Метод sleep() модуля time. Для этого передадим вычисленное время до заданного часа рассылки от момента вызова. Время будем рассчитывать серверное UTC(+X). 
*(Выбран для реализации)*

Дальнейший функционал идентичен для обоих методов.
