# DE32-2rd_team4
# Businness Chatting System
![image](https://github.com/user-attachments/assets/ef2c3426-8eaa-43bf-bbdf-c73f6ce9de71)



## Introduce
<Need add Introduce text>
이 레포지토리는 업무용 채팅, 채팅 감사기능, 챗봇 시스템 등이 결합된 패키지입니다.
보안이 필요한 업무에서 안전하게 다른 부서 직원들과 소통할 수 있고,
그들에게 다양한 기능을 함께하는 챗봇들이 일의 능률을 올려주며,
그들에게 뻗치는 음모를 빠르게 파악 할 수 있습니다.
단지 이 레포지토리를 이용하는 것 만으로요.


## Installation
### For Users
<Need add how to install for user>
사용자 분들은 본 레포지토리를 사용하기 위해 몇가지 준비가 필요합니다.
먼저, 본 레포지토리를 다운로드 해주십시오.
```
git clone https://github.com/DE32-2nd-team4/Business-Chatting-System.git
```

레포지토리를 설치 한 다음 몇 가지 초기 설정과 설치가 필요합니다.

### Airflow

https://airflow.apache.org/docs/apache-airflow/stable/start.html
```
$ pwd
/<your home>

$ vi .zshrc  # zsh

export AIRFLOW_HOME=~/<Your Install Location>
export AIRFLOW__CORE__DAGS_FOLDER=~/<Your Install Location>/dags
export AIRFLOW__CORE__LOAD_EXAMPLES=False
```

### Kafka

https://kafka.apache.org/quickstart
```
$ pwd
/<your Kafka install location>

$ vi config/server.properties
advertised.listeners=PLAINTEXT://<Your Server IP>
or
advertised.listeners=<Your Server IP>
```

# 

위의 과정들이 모두 마무리되었다면, 본 레포지토리 설치를 진행해 주십시오.
```
$ git clone https://github.com/DE32-2nd-team4/Business-Chatting-System.git
$ cd Business-Chatting-System
$ source .venv/bin/activate
$ pip install
```

## Usage
pip install과정까지 마무리된다면 아래 명령어들을 통해 시스템을 실행시킬 수 있습니다.

- bcs -h, --help     : 도움말 표시
- bcs -c, --chat     : 채팅서버 접속
- bcs -b, --bot      : 채팅 봇 활성화
- bcs -i, --ipconfig : 접속 서버 주소 변경

### Chat System
```
$ bcs -c
```
![image](https://github.com/user-attachments/assets/1dcb65b5-800a-457b-827b-b59b25972bd0)

![image](https://github.com/user-attachments/assets/2454408a-6132-4eff-ba41-dcaa952c8a84)

커맨드를 실행시 접속할 대화방과 접속할 닉네임 설정을 물어본다음 채팅방으로 입장됩니다.
화면 하단 마우스 클릭을 통해 채팅 입력창 활성화가 가능하며, Enter를 입력해 채팅을 보낼 수 있습니다.
본인은 빨간색 닉네임으로 표시됩니다.


### Chat Audit System
본 기능은 특수한 목적으로 사용되며 일반 사용자에게는 공개되지 않습니다.
권한 획득을 위해 아래 관리자 인증 안내창으로 이동하여 안내를 따라주십시오.
https://github.com/DE32-2nd-team4/Business-Chatting-System/issues/19#issue-2491436915

### Chat-bot System
```
$ bcs -b
```
명령어를 통해 챗봇을 활성화 할 수 있으며, 활성화한 챗봇들은 다음과 같이 사용할 수 있습니다.

  - 시스템 알림 챗봇
    airflow dags 작업 등을 이용시 특정 작업에 대한 결과를 확인해야 할 때, dags 파일에 삽입해 호출 할 수 있습니다.
    아래 tutorial code를 참조해 이용하십시오.

```
dags/<your_dags>.py

from airflow import DAG
import time
import json
from airflow.operators.python import PythonOperator
from airflow.operators.empty import EmptyOperator
from datetime import datetime
from kafka import KafkaConsumer, KafkaProducer
# DAG 정의
with DAG(
    dag_id='systemAleamTestDag',
    schedule_interval='@once',
    start_date=datetime(2024, 8, 28),
    catchup=False,

    ) as dag:

    def fail_aleam(**context):
        message = "@bot <Your Return Message>"
        topic = "<Your_System_topic>"
        address = "<Your_IP_Address>"
        producer = KafkaProducer(
        bootstrap_servers=address,
        value_serializer=lambda x:json.dumps(x,ensure_ascii=False).encode('utf-8'),
        )
        time_str = context['execution_date'].strftime('%Y-%m-%d %H:%M:%S')
        m_message = {'nickname': '@systembot', 'message': message, 'time': time_str }
        producer.send(topic, value=m_message)
        producer.flush()


    start = EmptyOperator(task_id='start')
    end = EmptyOperator(task_id='end')
    task_a = PythonOperator(
            task_id='task.a',
            python_callable=fail_aleam,
            )

    start >> task_a >> end
```
  ![image](https://github.com/user-attachments/assets/d6bd924b-5d8c-4d01-935e-e25c074cb7fc)

  - 일정 알림 챗봇
    airflow dags를 이용해 특정 시간에 대한 알림이 필요할때 dags파일 작성 후 online설정 시 이용가능합니다.
    아래 tutorial code를 참조해 이용하십시오.

```
dags/<your_dags>.py

from airflow import DAG
import time
import json
from airflow.operators.python import PythonOperator
from airflow.operators.empty import EmptyOperator
from datetime import datetime
from kafka import KafkaConsumer, KafkaProducer
# DAG 정의
with DAG(
    dag_id='systemAleamTestDag',
    schedule_interval='<Want_to_use_time>',
    start_date=datetime(2024, 8, 28),
    catchup=False,

    ) as dag:

    def fail_aleam(**context):
        message = "@bot <Your Return Message>"
        topic = "<Your_System_topic>"
        address = "<Your_IP_Address>"
        producer = KafkaProducer(
        bootstrap_servers=address,
        value_serializer=lambda x:json.dumps(x,ensure_ascii=False).encode('utf-8'),
        )
        time_str = context['execution_date'].strftime('%Y-%m-%d %H:%M:%S')
        m_message = {'nickname': '@systembot', 'message': message, 'time': time_str }
        producer.send(topic, value=m_message)
        producer.flush()


    start = EmptyOperator(task_id='start')
    end = EmptyOperator(task_id='end')
    task_a = PythonOperator(
            task_id='task.a',
            python_callable=fail_aleam,
            )

    start >> task_a >> end
```
  ![image](https://github.com/user-attachments/assets/81e07cd9-f198-467e-91b2-006e65819900)

  - 영화 검색 챗봇
## TODO

  ![image](https://github.com/user-attachments/assets/01e55253-68b0-4ec4-9cc2-91314ebc0ec8)

-

## Reference
- https://kafka.apache.org/quickstart
- https://airflow.apache.org/docs/apache-airflow/stable/start.html
- https://zeppelin.apache.org/docs/latest/quickstart/install.html


