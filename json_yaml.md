# Домашнее задание к занятию «Языки разметки JSON и YAML»

### Цель задания

В результате выполнения задания вы:

* познакомитесь с синтаксисами JSON и YAML;
* узнаете, как преобразовать один формат в другой при помощи пары строк.

### Чеклист готовности к домашнему заданию

1. Установлена библиотека PyYAML для Python 3.

### Инструкция к заданию 

1. Скопируйте в свой .md-файл содержимое этого файла, исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md).
3. Заполните недостающие части документа решением задач — заменяйте `???`, остальное в шаблоне не меняйте, чтобы не сломать форматирование текста, подсветку синтаксиса. Вместо логов можно вставить скриншоты по желанию.
4. Любые вопросы по выполнению заданий задавайте в чате учебной группы или в разделе «Вопросы по заданию» в личном кабинете.

### Дополнительные материалы

1. [Полезные ссылки для модуля «Скриптовые языки и языки разметки».](https://github.com/netology-code/sysadm-homeworks/tree/devsys10/04-script-03-yaml/additional-info)

------

## Задание 1

Мы выгрузили JSON, который получили через API-запрос к нашему сервису:

```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис.

### Ваш скрипт:

```
"info" : "Sample JSON output from our service\\t",
  "elements" : [
    {
     "name" : "first",
     "type" : "server",
     "ip" : "7175" 
    },
    {
     "name" : "second",
     "type" : "proxy",
     "ip" : "71.78.22.43"
    }
  ]
```

1 строка - неэкранированный спецсимвол \t (горизонтальная табуляция)
2 строка - пропущен пробел между : и [ (открытие массива)
1,3,7 строки - после скобки { сделать перенос (для лучшей читаемости)
5 строка - неверная запись ip адреса? недописан и без кавычек
9 строка - пропущены закрывающиеся кавычки ip" и кавычки на значении

---

## Задание 2

В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML-файлов, описывающих наши сервисы. 

Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. 

Формат записи YAML по одному сервису: `- имя сервиса: его IP`. 

Если в момент исполнения скрипта меняется IP у сервиса — он должен так же поменяться в YAML и JSON-файле.

### Ваш скрипт:

```
# !/usr/bin/env python3

import os
import json
import yaml
import time


site_list = ['drive.google.com', 'mail.google.com', 'google.com']
site_dict = {}

def check_list_of_sites(site_list, site_dict):

    for site_url in site_list:
        site_dict=check_site_dns(site_url, site_dict)
    return site_dict


def check_site_dns(site_url, site_dict):
    site_new_ip = []


    result_os = os.popen(f'dig +short {site_url} | grep  -E \'[0-9]\'')

    for result in result_os:
        # For any ip delete \n
        site_new_ip.append(result.replace("\n",""))

    print('site_new_ip: ', site_new_ip)

    if site_dict.get(site_url) != None:

        site_old_ip = site_dict[site_url]
        i = 0
        ip_changed = False
        while i < len(site_old_ip):
            if site_old_ip[i] == site_new_ip[i]:
                print(f'{site_url} - {site_old_ip[i]}.')
            else:
                print(f'[ERROR] {site_url} IP mismatch: {site_old_ip[i]} {site_new_ip[i]}.')                
                ip_changed = True
            i = i + 1

    else:
        #If it's first execution
        site_dict[site_url] = site_new_ip
        print('site_dict==', site_dict)
        ip_changed = True

    if ip_changed == True:
        with open("servers_ip.json", "w") as fp_json:
            json.dump(site_dict, fp_json, indent=2)
        with open("servers_ip.yaml", "w") as fp_yaml:
            yaml.dump(site_dict, fp_yaml, explicit_start=True, explicit_end=True)

    return site_dict

while True:
    site_dict = check_list_of_sites(site_list, site_dict)
    print("site_dict==", site_dict)
    time.sleep(5)
```

### Вывод скрипта при запуске во время тестирования:

```
% python3 test.py 
site_new_ip:  ['142.250.185.78']
site_dict== {'drive.google.com': ['142.250.185.78']}
site_new_ip:  ['142.250.186.101']
site_dict== {'drive.google.com': ['142.250.185.78'], 'mail.google.com': ['142.250.186.101']}
site_new_ip:  ['142.250.184.238']
site_dict== {'drive.google.com': ['142.250.185.78'], 'mail.google.com': ['142.250.186.101'], 'google.com': ['142.250.184.238']}
site_dict== {'drive.google.com': ['142.250.185.78'], 'mail.google.com': ['142.250.186.101'], 'google.com': ['142.250.184.238']}
site_new_ip:  ['142.250.185.78']
drive.google.com - 142.250.185.78.
site_new_ip:  ['142.250.186.101']
mail.google.com - 142.250.186.101.
site_new_ip:  ['142.250.184.238']
google.com - 142.250.184.238.
site_dict== {'drive.google.com': ['142.250.185.78'], 'mail.google.com': ['142.250.186.101'], 'google.com': ['142.250.184.238']}
site_new_ip:  ['142.250.185.78']
drive.google.com - 142.250.185.78.
site_new_ip:  ['142.250.186.101']
mail.google.com - 142.250.186.101.
site_new_ip:  ['142.250.184.238']
google.com - 142.250.184.238.
site_dict== {'drive.google.com': ['142.250.185.78'], 'mail.google.com': ['142.250.186.101'], 'google.com': ['142.250.184.238']}
```

### JSON-файл(ы), который(е) записал ваш скрипт:

```
{
  "drive.google.com": [
    "142.250.185.78"
  ],
  "mail.google.com": [
    "142.250.186.101"
  ],
  "google.com": [
    "142.250.184.238"
  ]
}
```

### YAML-файл(ы), который(е) записал ваш скрипт:

```
---
drive.google.com:
- 142.250.185.78
google.com:
- 142.250.184.238
mail.google.com:
- 142.250.186.101
...
```

---

## Задание со звёздочкой* 

Это самостоятельное задание, его выполнение необязательно.
____

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:

   * принимать на вход имя файла;
   * проверять формат исходного файла. Если файл не JSON или YAML — скрипт должен остановить свою работу;
   * распознавать, какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны;
   * перекодировать данные из исходного формата во второй доступный —  из JSON в YAML, из YAML в JSON;
   * при обнаружении ошибки в исходном файле указать в стандартном выводе строку с ошибкой синтаксиса и её номер;
   * полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов.

### Ваш скрипт:

```
#!/usr/bin/env python3

import yaml
import json
import sys


def check_formats(string):
    f = {}
    try:
        j = json.loads(string)
        f['json'] = "valid"
        y = yaml.safe_load(string)
        f['yaml'] = "json"
    except json.decoder.JSONDecodeError as e:
        try:
            y = yaml.safe_load(string)
            f['yaml'] = "valid"
            try:
                json.dumps(y)
                f['json'] = "yaml"
            except json.decoder.JSONDecodeError as e3:
                m3 = e3.args[0]
                print(m3)
                f['json'] = "err"
            except TypeError as e3:
                m3 = e3.args[0]
                if m3.endswith('not list'):
                    json.dumps(y)
                    f['json'] = "yaml"
        except (yaml.parser.ParserError, yaml.scanner.ScannerError) as e2:
            f['yaml'] = "err"
            f['json'] = "err"
    return f


def yaml_to_file(f_name, d, fmt):
    # print(f'yaml_to_file, fmt = {fmt}')
    if fmt == "yaml":
        stream = yaml.dump(d, indent=2)
    if fmt == "json":
        j = json.loads(d)
        stream = yaml.dump(j, indent=2)
    # print(f"write to: {f_name}\ndata:\n{stream}")
    with open(f_name, 'w') as file:
        file.write(stream)
    print(f'Файл {f_name} записан')


def json_to_file(f_name, d, fmt):
    # print(f'json_to_file, fmt = {fmt}')
    if fmt == "json":
        stream = json.dumps(json.loads(d), indent=4)
    if fmt == "yaml":
        y = yaml.safe_load(d)
        stream = json.dumps(y, indent=2)
    # print(f"write to: {f_name}\ndata:\n{stream}")
    with open(f_name, 'w') as file:
        file.write(stream)
    print(f'Файл {f_name} записан')


def except_pprint(l, c, p, d, ex):
    d = d.split('\n')
    print(f"""
Не получилось разобрать {ex} файл, парсер остановился на строке {l+1}, символ {c+1}.

{d[l]}
{' ' * c}^

Суть проблемы: {p}
""")


args = sys.argv
source_file, ext, name, converted_file = '', '', '', ''
try:
    source_file = args[1]
    ext = source_file.split('.')[-1]
    name = source_file.split(f'.{ext}')[0]
    print(f'Исходный файл {source_file}')
    if ext == 'json':
        converted_file = f'{name}.yaml'
    elif ext == 'yaml' or ext == 'yml':
        converted_file = f'{name}.json'
    else:
        print(f"""Расширение ".{ext}" не поддерживается.\nПожалуйста, укажите файл JSON или YAML""")
except IndexError:
    print('Укажите имя файла в формате JSON или YAML')
    exit()

with open(source_file, 'r', encoding='utf_8') as file:
    data = file.read()
formats = check_formats(data)

if ext == 'json':
    if formats['json'] == "valid":
        yaml_to_file(converted_file, data, 'json')
    elif formats['json'] == "yaml":
        print(f'Файл {source_file} имеет формат yaml, переписываем')
        json_to_file(source_file, data, 'yaml')
        yaml_to_file(converted_file, data, 'yaml')
    else:
        try:
            json.loads(data)
        except json.decoder.JSONDecodeError as e:
            except_pprint(e.lineno-1, e.colno-1, e.args[0], data, ext)
elif ext == 'yaml':
    if formats['yaml'] == "valid":
        json_to_file(converted_file, data, 'yaml')
    elif formats['yaml'] == "json":
        print(f'Файл {source_file} имеет формат json, переписываем')
        yaml_to_file(source_file, data, 'json')
        json_to_file(converted_file, data, 'json')
    else:
        try:
            yaml.safe_load(data)
        except yaml.parser.ParserError as e:
            except_pprint(e.problem_mark.line, e.problem_mark.column, e.problem, data, ext)
        except yaml.scanner.ScannerError as e:
            try:
                except_pprint(e.context_mark.line, e.context_mark.column, e.problem, data, ext)
            except AttributeError:
                except_pprint(e.problem_mark.line, e.problem_mark.column, e.problem, data, ext)


```

### Пример работы скрипта:

???

----

### Правила приёма домашнего задания

В личном кабинете отправлена ссылка на .md-файл в вашем репозитории.

-----

### Критерии оценки

Зачёт:

* выполнены все задания;
* ответы даны в развёрнутой форме;
* приложены соответствующие скриншоты и файлы проекта;
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку:

* задание выполнено частично или не выполнено вообще;
* в логике выполнения заданий есть противоречия и существенные недостатки.  
 
Обязательными являются задачи без звёздочки. Их выполнение необходимо для получения зачёта и диплома о профессиональной переподготовке.

Задачи со звёздочкой (*) являются дополнительными или задачами повышенной сложности. Они необязательные, но их выполнение поможет лучше разобраться в теме.
