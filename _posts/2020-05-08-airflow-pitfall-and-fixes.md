---
title: Airflow pitfalls and fixes
categories: [tips, operations]
tags: [operations, airflow, tips]
description: Some pitfalls and fixes when I use Airflow.
---

- sqlalchemy.exc.ProgrammingError: (mysql.connector.errors.ProgrammingError) 1305 (42000): FUNCTION rcc_airflow.JSON_VALID does not exist

airflow needs mysql 5.7

- Exception: Global variable explicit_defaults_for_timestamp needs to be on (1) for mysql

set **explicit_defaults_for_timestamp** to 1 in my.cnf

- cryptography not found - values will not be stored encrypted.

{% highlight shell %}
pip install cryptography
{% endhighlight %}

- empty cryptography key - values will not be stored encrypted.

[https://airflow.readthedocs.io/en/stable/howto/secure-connections.html](https://airflow.readthedocs.io/en/stable/howto/secure-connections.html)

generate a key to replace in airflow.cfg

- use **mysql-connector-python** instead of Mysql-python (this proved not working)

in airflow.cfg:

mysql+mysqlconnector://user:password@10.10.10.101:3333/airflow

- Unicodedecodeerror: 'utf-8' codec can't decode byte 0x80 in position 0: invalid start byte

Please install 'apache-airflow[mysql]'

mysql-connector-python is proved not working with Airflow

- BashOperator: TemplateNotFoundthe command needs to follow with a space:

{% highlight python %}
bash_command='{}/backup.sh '.format(backup_dir),
{% endhighlight %}
