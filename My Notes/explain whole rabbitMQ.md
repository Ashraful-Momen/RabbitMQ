# RabbitMQ - সম্পূর্ণ বাংলা গাইড

---

## 📖 RabbitMQ কি?

**সংজ্ঞা**: RabbitMQ হলো একটি ওপেন সোর্স **Message Broker** সফটওয়্যার যা বিভিন্ন অ্যাপ্লিকেশনের মধ্যে মেসেজ পাঠাতে এবং গ্রহণ করতে সাহায্য করে।

**সহজ ভাষায়**: 
- ধরুন আপনার একটি রেস্তোরাঁ আছে
- **Customer (প্রোডিউসার)** = অর্ডার দেয়
- **Waiter (RabbitMQ)** = অর্ডার নিয়ে রান্নাঘরে পৌঁছায়
- **Chef (কনজিউমার)** = অর্ডার রান্না করে

RabbitMQ হলো সেই **Waiter** যে নিশ্চিত করে অর্ডার সঠিকভাবে পৌঁছেছে।

---

## 🔄 RabbitMQ এর কাজের ধাপ (Step by Step Workflow)

### ASCII Diagram:

```
┌─────────────┐          ┌──────────────┐          ┌─────────┐          ┌─────────────┐
│  Producer   │─────────>│   Exchange   │─────────>│  Queue  │─────────>│  Consumer   │
│ (প্রোডিউসার)│          │  (এক্সচেঞ্জ)  │          │ (কিউ)    │          │ (কনজিউমার)  │
└─────────────┘          └──────────────┘          └─────────┘          └─────────────┘
   অর্ডার পাঠায়              রুট করে                জমা রাখে              প্রসেস করে
```

### বিস্তারিত ওয়ার্কফ্লো:

```
Step 1: Producer মেসেজ তৈরি করে
        │
        ▼
   ┌─────────────────┐
   │  Message তৈরি   │
   │  "Order #123"   │
   └─────────────────┘
        │
        │ (1) মেসেজ পাঠায়
        ▼
Step 2: Exchange এ মেসেজ পৌঁছায়
        │
        ▼
   ┌─────────────────────────┐
   │      Exchange           │
   │  ┌─────────────────┐    │
   │  │ Routing Logic   │    │
   │  └─────────────────┘    │
   └─────────────────────────┘
        │
        │ (2) Routing Key অনুযায়ী রুট করে
        ▼
Step 3: Queue তে মেসেজ জমা হয়
        │
        ▼
   ┌─────────────────────────┐
   │        Queue            │
   │  ┌───┐ ┌───┐ ┌───┐     │
   │  │ M │ │ M │ │ M │     │
   │  └───┘ └───┘ └───┘     │
   └─────────────────────────┘
        │
        │ (3) Consumer এর জন্য অপেক্ষা করে
        ▼
Step 4: Consumer মেসেজ নেয়
        │
        ▼
   ┌─────────────────────────┐
   │      Consumer           │
   │   প্রসেস করে এবং        │
   │   ACK পাঠায়             │
   └─────────────────────────┘
```

---

## 🏗️ RabbitMQ এর মূল উপাদান (Components)

### 1. **Producer (প্রোডিউসার)**
```
┌─────────────────┐
│   Producer      │
│                 │
│  মেসেজ তৈরি করে │
│  এবং পাঠায়      │
└─────────────────┘
```
**উদাহরণ**: একটি ওয়েবসাইট যেখানে ইউজার অর্ডার দেয়

---

### 2. **Exchange (এক্সচেঞ্জ)**
```
┌───────────────────────────────┐
│        Exchange               │
│  ┌─────────────────────┐      │
│  │  Routing Algorithm  │      │
│  │  কোন Queue তে যাবে?  │      │
│  └─────────────────────┘      │
└───────────────────────────────┘
```
**কাজ**: মেসেজ কোন Queue তে যাবে তা নির্ধারণ করে

**Exchange এর ৪ টি প্রকার**:

#### a) **Direct Exchange** (সরাসরি)
```
Producer ──> Exchange ──[exact routing key]──> Queue
                        
উদাহরণ:
routing_key = "order.new"
    │
    ▼
"order.new" queue
```

#### b) **Fanout Exchange** (সবার কাছে)
```
                    ┌──> Queue 1
Producer ──> Exchange ├──> Queue 2
                    └──> Queue 3
                    
(সব Queue তে কপি পাঠায়)
```

#### c) **Topic Exchange** (প্যাটার্ন ভিত্তিক)
```
routing_key = "order.*.urgent"
    │
    ├──> "order.new.urgent" ✓
    ├──> "order.update.urgent" ✓
    └──> "order.new.normal" ✗
```

#### d) **Headers Exchange** (মেটাডেটা ভিত্তিক)
```
headers = {
    "type": "order",
    "priority": "high"
}
```

---

### 3. **Queue (কিউ)**
```
┌─────────────────────────────┐
│         Queue               │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐   │
│  │ 1 │ │ 2 │ │ 3 │ │ 4 │   │
│  └───┘ └───┘ └───┘ └───┘   │
│  (FIFO - First In First Out)│
└─────────────────────────────┘
```
**কাজ**: মেসেজ সংরক্ষণ করে যতক্ষণ না Consumer নেয়

---

### 4. **Consumer (কনজিউমার)**
```
┌─────────────────────────┐
│      Consumer           │
│                         │
│  Queue থেকে মেসেজ নিয়ে │
│  প্রসেস করে             │
└─────────────────────────┘
```
**উদাহরণ**: একটি সার্ভিস যা ইমেইল পাঠায়

---

### 5. **Binding (বাইন্ডিং)**
```
Exchange <──[Binding]──> Queue

Binding = Connection + Routing Rules
```
**কাজ**: Exchange এবং Queue এর মধ্যে সংযোগ তৈরি করে

---

## 📋 সম্পূর্ণ ওয়ার্কফ্লো উদাহরণ (ই-কমার্স অর্ডার সিস্টেম)

```
                                 RabbitMQ Server
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌──────────┐        ┌─────────────────┐                           │
│  │ Producer │        │  Order Exchange │                           │
│  │ (Website)│───────>│   (Direct)      │                           │
│  └──────────┘        └─────────────────┘                           │
│   "Order #123"              │                                       │
│   routing_key:              │                                       │
│   "order.new"               │                                       │
│                             ├─────[order.new]────────┐             │
│                             │                         │             │
│                             ▼                         ▼             │
│                    ┌──────────────┐         ┌──────────────┐       │
│                    │ Order Queue  │         │ Email Queue  │       │
│                    │              │         │              │       │
│                    │ ┌──┐ ┌──┐   │         │ ┌──┐         │       │
│                    │ │M1│ │M2│   │         │ │M1│         │       │
│                    │ └──┘ └──┘   │         │ └──┘         │       │
│                    └──────────────┘         └──────────────┘       │
│                             │                         │             │
└─────────────────────────────│─────────────────────────│─────────────┘
                              │                         │
                              ▼                         ▼
                    ┌──────────────┐         ┌──────────────┐
                    │  Consumer 1  │         │  Consumer 2  │
                    │              │         │              │
                    │ Order        │         │ Email        │
                    │ Processing   │         │ Service      │
                    └──────────────┘         └──────────────┘
```

---

## 🛠️ Step by Step Setup (কোড সহ)

### ধাপ ১: RabbitMQ ইনস্টল করুন

```bash
# Docker দিয়ে (সবচেয়ে সহজ)
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management

# ব্রাউজারে Management UI খুলুন:
# http://localhost:15672
# Username: guest
# Password: guest
```

---

### ধাপ ২: Python Library ইনস্টল করুন

```bash
pip install pika
```

---

### ধাপ ৩: Exchange তৈরি করুন (বিভিন্ন প্রকার)

#### **Example 1: Direct Exchange তৈরি**

```python
import pika

# সংযোগ তৈরি করুন
connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Direct Exchange তৈরি করুন
channel.exchange_declare(
    exchange='order_exchange',    # Exchange এর নাম
    exchange_type='direct',       # প্রকার: direct
    durable=True                  # রিস্টার্টেও থাকবে
)

print("✅ Direct Exchange তৈরি হয়েছে!")
connection.close()
```

---

#### **Example 2: Fanout Exchange তৈরি**

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Fanout Exchange তৈরি করুন
channel.exchange_declare(
    exchange='notification_exchange',
    exchange_type='fanout',      # সব Queue তে পাঠাবে
    durable=True
)

print("✅ Fanout Exchange তৈরি হয়েছে!")
connection.close()
```

---

#### **Example 3: Topic Exchange তৈরি**

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Topic Exchange তৈরি করুন
channel.exchange_declare(
    exchange='logs_exchange',
    exchange_type='topic',       # প্যাটার্ন ভিত্তিক routing
    durable=True
)

print("✅ Topic Exchange তৈরি হয়েছে!")
connection.close()
```

---

### ধাপ ৪: Queue তৈরি করুন

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Queue তৈরি করুন
channel.queue_declare(
    queue='order_processing',     # Queue এর নাম
    durable=True,                 # Permanent storage
    arguments={
        'x-max-priority': 10,     # Priority support
        'x-message-ttl': 3600000  # 1 ঘন্টা TTL
    }
)

print("✅ Queue তৈরি হয়েছে!")
connection.close()
```

---

### ধাপ ৫: Binding তৈরি করুন (Exchange ও Queue সংযুক্ত করুন)

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Exchange তৈরি করুন
channel.exchange_declare(
    exchange='order_exchange',
    exchange_type='direct',
    durable=True
)

# Queue তৈরি করুন
channel.queue_declare(
    queue='new_orders',
    durable=True
)

# Binding তৈরি করুন (সংযুক্ত করুন)
channel.queue_bind(
    exchange='order_exchange',    # কোন Exchange
    queue='new_orders',           # কোন Queue
    routing_key='order.new'       # কোন routing key
)

print("✅ Binding সফল!")
connection.close()
```

**ভিজুয়াল উপস্থাপনা:**
```
order_exchange ──[routing_key: "order.new"]──> new_orders Queue
```

---

### ধাপ ৬: Producer তৈরি করুন (মেসেজ পাঠান)

```python
import pika
import json

def send_order(order_data):
    # সংযোগ তৈরি
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('localhost')
    )
    channel = connection.channel()
    
    # Exchange declare করুন
    channel.exchange_declare(
        exchange='order_exchange',
        exchange_type='direct',
        durable=True
    )
    
    # মেসেজ পাঠান
    channel.basic_publish(
        exchange='order_exchange',        # কোন Exchange এ
        routing_key='order.new',          # কোন routing key দিয়ে
        body=json.dumps(order_data),      # মেসেজের বডি
        properties=pika.BasicProperties(
            delivery_mode=2,              # Persistent message
            priority=5,                   # Priority
            content_type='application/json'
        )
    )
    
    print(f"✅ অর্ডার পাঠানো হয়েছে: {order_data['order_id']}")
    connection.close()

# অর্ডার পাঠান
order = {
    'order_id': 'ORD-2025-001',
    'customer_name': 'রহিম উদ্দিন',
    'product': 'ল্যাপটপ',
    'price': 45000,
    'quantity': 1
}

send_order(order)
```

**আউটপুট:**
```
✅ অর্ডার পাঠানো হয়েছে: ORD-2025-001
```

---

### ধাপ ৭: Consumer তৈরি করুন (মেসেজ গ্রহণ করুন)

```python
import pika
import json

def process_order(ch, method, properties, body):
    """
    মেসেজ প্রসেস করার ফাংশন
    """
    # মেসেজ পার্স করুন
    order = json.loads(body)
    
    print("=" * 50)
    print("📦 নতুন অর্ডার পাওয়া গেছে!")
    print(f"অর্ডার আইডি: {order['order_id']}")
    print(f"কাস্টমার: {order['customer_name']}")
    print(f"পণ্য: {order['product']}")
    print(f"মূল্য: ৳{order['price']}")
    print(f"পরিমাণ: {order['quantity']}")
    print("=" * 50)
    
    # প্রসেসিং সিমুলেট করুন
    try:
        # এখানে আপনার বিজনেস লজিক
        # যেমন: ডাটাবেসে সেভ, পেমেন্ট প্রসেস ইত্যাদি
        
        print("✅ অর্ডার সফলভাবে প্রসেস হয়েছে!")
        
        # ACK পাঠান (মেসেজ সফল)
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
    except Exception as e:
        print(f"❌ এরর: {e}")
        # NACK পাঠান (মেসেজ ব্যর্থ)
        ch.basic_nack(
            delivery_tag=method.delivery_tag,
            requeue=False  # আবার Queue তে ফেরত দেবে না
        )

def start_consumer():
    """
    Consumer চালু করুন
    """
    # সংযোগ তৈরি
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('localhost')
    )
    channel = connection.channel()
    
    # Queue declare করুন
    channel.queue_declare(
        queue='new_orders',
        durable=True
    )
    
    # QoS সেট করুন (একসাথে কয়টা মেসেজ নেবে)
    channel.basic_qos(prefetch_count=1)
    
    # Consumer শুরু করুন
    channel.basic_consume(
        queue='new_orders',
        on_message_callback=process_order  # কলবাক ফাংশন
    )
    
    print("⏳ অর্ডারের জন্য অপেক্ষা করছি...")
    print("বন্ধ করতে CTRL+C চাপুন")
    
    # মেসেজ শোনা শুরু করুন
    channel.start_consuming()

# Consumer চালু করুন
if __name__ == '__main__':
    start_consumer()
```

**আউটপুট:**
```
⏳ অর্ডারের জন্য অপেক্ষা করছি...
বন্ধ করতে CTRL+C চাপুন
==================================================
📦 নতুন অর্ডার পাওয়া গেছে!
অর্ডার আইডি: ORD-2025-001
কাস্টমার: রহিম উদ্দিন
পণ্য: ল্যাপটপ
মূল্য: ৳45000
পরিমাণ: 1
==================================================
✅ অর্ডার সফলভাবে প্রসেস হয়েছে!
```

---

## 🎯 সম্পূর্ণ প্রজেক্ট উদাহরণ: ই-কমার্স অর্ডার সিস্টেম

### প্রজেক্ট স্ট্রাকচার:
```
ecommerce-rabbitmq/
├── setup.py          (Setup Exchange, Queue, Binding)
├── producer.py       (অর্ডার পাঠানো)
├── consumer.py       (অর্ডার প্রসেস)
└── config.py         (কনফিগারেশন)
```

---

### **File 1: config.py** (কনফিগারেশন)

```python
# RabbitMQ সেটিংস
RABBITMQ_HOST = 'localhost'
RABBITMQ_PORT = 5672
RABBITMQ_USER = 'guest'
RABBITMQ_PASS = 'guest'

# Exchange সেটিংস
EXCHANGE_NAME = 'ecommerce_exchange'
EXCHANGE_TYPE = 'direct'

# Queue সেটিংস
ORDER_QUEUE = 'order_processing'
EMAIL_QUEUE = 'email_notification'
SMS_QUEUE = 'sms_notification'

# Routing Keys
ROUTE_ORDER = 'order.new'
ROUTE_EMAIL = 'notification.email'
ROUTE_SMS = 'notification.sms'
```

---

### **File 2: setup.py** (সেটআপ স্ক্রিপ্ট)

```python
import pika
from config import *

def setup_rabbitmq():
    """
    RabbitMQ সেটআপ করুন
    """
    # সংযোগ তৈরি
    connection = pika.BlockingConnection(
        pika.ConnectionParameters(host=RABBITMQ_HOST)
    )
    channel = connection.channel()
    
    print("🔧 RabbitMQ সেটআপ শুরু হচ্ছে...")
    
    # Exchange তৈরি
    channel.exchange_declare(
        exchange=EXCHANGE_NAME,
        exchange_type=EXCHANGE_TYPE,
        durable=True
    )
    print(f"✅ Exchange তৈরি: {EXCHANGE_NAME}")
    
    # Queues তৈরি
    queues = [ORDER_QUEUE, EMAIL_QUEUE, SMS_QUEUE]
    for queue in queues:
        channel.queue_declare(
            queue=queue,
            durable=True,
            arguments={
                'x-max-priority': 10,
                'x-message-ttl': 3600000
            }
        )
        print(f"✅ Queue তৈরি: {queue}")
    
    # Bindings তৈরি
    bindings = [
        (ORDER_QUEUE, ROUTE_ORDER),
        (EMAIL_QUEUE, ROUTE_EMAIL),
        (SMS_QUEUE, ROUTE_SMS)
    ]
    
    for queue, routing_key in bindings:
        channel.queue_bind(
            exchange=EXCHANGE_NAME,
            queue=queue,
            routing_key=routing_key
        )
        print(f"✅ Binding: {queue} <-> {routing_key}")
    
    connection.close()
    print("\n🎉 সেটআপ সম্পন্ন!")

if __name__ == '__main__':
    setup_rabbitmq()
```

**চালান:**
```bash
python setup.py
```

---

### **File 3: producer.py** (প্রোডিউসার)

```python
import pika
import json
from datetime import datetime
from config import *

class OrderProducer:
    def __init__(self):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=RABBITMQ_HOST)
        )
        self.channel = self.connection.channel()
    
    def send_order(self, order_data):
        """
        অর্ডার পাঠান
        """
        # Timestamp যোগ করুন
        order_data['timestamp'] = datetime.now().isoformat()
        
        # মেসেজ পাঠান
        self.channel.basic_publish(
            exchange=EXCHANGE_NAME,
            routing_key=ROUTE_ORDER,
            body=json.dumps(order_data, ensure_ascii=False),
            properties=pika.BasicProperties(
                delivery_mode=2,
                priority=order_data.get('priority', 5),
                content_type='application/json'
            )
        )
        
        print(f"✅ অর্ডার পাঠানো হয়েছে: {order_data['order_id']}")
        return True
    
    def send_notification(self, notification_type, data):
        """
        নোটিফিকেশন পাঠান
        """
        routing_key = ROUTE_EMAIL if notification_type == 'email' else ROUTE_SMS
        
        self.channel.basic_publish(
            exchange=EXCHANGE_NAME,
            routing_key=routing_key,
            body=json.dumps(data, ensure_ascii=False),
            properties=pika.BasicProperties(
                delivery_mode=2,
                content_type='application/json'
            )
        )
        
        print(f"✅ {notification_type} নোটিফিকেশন পাঠানো হয়েছে")
    
    def close(self):
        self.connection.close()

# ব্যবহার উদাহরণ
if __name__ == '__main__':
    producer = OrderProducer()
    
    # অর্ডার তৈরি করুন
    orders = [
        {
            'order_id': 'ORD-2025-001',
            'customer_name': 'করিম মিয়া',
            'customer_phone': '01712345678',
            'customer_email': 'karim@example.com',
            'products': [
                {'name': 'মোবাইল ফোন', 'price': 25000, 'quantity': 1},
                {'name': 'ফোন কেস', 'price': 500, 'quantity': 2}
            ],
            'total': 26000,
            'priority': 8,
            'shipping_address': 'ঢাকা, বাংলাদেশ'
        },
        {
            'order_id': 'ORD-2025-002',
            'customer_name': 'রহিম উদ্দিন',
            'customer_phone': '01898765432',
            'customer_email': 'rahim@example.com',
            'products': [
                {'name': 'ল্যাপটপ', 'price': 55000, 'quantity': 1}
            ],
            'total': 55000,
            'priority': 10,  # High priority
            'shipping_address': 'চট্টগ্রাম, বাংলাদেশ'
        }
    ]
    
    # অর্ডার পাঠান
    for order in orders:
        producer.send_order(order)
        
        # ইমেইল নোটিফিকেশন
        producer.send_notification('email', {
            'to': order['customer_email'],
            'subject': 'অর্ডার কনফার্মেশন',
            'body': f"আপনার অর্ডার {order['order_id']} গ্রহণ করা হয়েছে।"
        })
        
        # SMS নোটিফিকেশন
        producer.send_notification('sms', {
            'to': order['customer_phone'],
            'message': f"অর্ডার {order['order_id']} কনফার্ম হয়েছে। মোট: ৳{order['total']}"
        })
    
    producer.close()
    print("\n🎉 সব মেসেজ পাঠানো হয়েছে!")
```

---

### **File 4: consumer.py** (কনজিউমার)

```python
import pika
import json
import time
from config import *

class OrderConsumer:
    def __init__(self, queue_name, callback):
        self.queue_name = queue_name
        self.callback = callback
        
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=RABBITMQ_HOST)
        )
        self.channel = self.connection.channel()
        
        # QoS সেট করুন
        self.channel.basic_qos(prefetch_count=1)
    
    def start(self):
        """
        Consumer চালু করুন
        """
        self.channel.basic_consume(
            queue=self.queue_name,
            on_message_callback=self.callback
        )
        
        print(f"⏳ {self.queue_name} এ মেসেজের জন্য অপেক্ষা করছি...")
        print("বন্ধ করতে CTRL+C চাপুন\n")
        
        self.channel.start_consuming()

# কলবাক ফাংশন: অর্ডার প্রসেসিং
def process_order(ch, method, properties, body):
    try:
        order = json.loads(body)
        
        print("=" * 70)
        print("📦 নতুন অর্ডার!")
        print(f"অর্ডার আইডি: {order['order_id']}")
        print(f"কাস্টমার: {order['customer_name']}")
        print(f"ফোন: {order['customer_phone']}")
        print(f"ঠিকানা: {order['shipping_address']}")
        print(f"\nপণ্য তালিকা:")
        
        for idx, product in enumerate(order['products'], 1):
            print(f"  {idx}. {product['name']} - ৳{product['price']} x {product['quantity']}")
        
        print(f"\nমোট: ৳{order['total']}")
        print(f"Priority: {order.get('priority', 5)}/10")
        print("=" * 70)
        
        # প্রসেসিং সিমুলেট করুন
        print("⚙️  অর্ডার প্রসেস করা হচ্ছে...")
        time.sleep(2)  # 2 সেক
ন্ড অপেক্ষা
        
        # ডাটাবেসে সেভ (সিমুলেট)
        print("💾 ডাটাবেসে সেভ করা হচ্ছে...")
        time.sleep(1)
        
        # ইনভেন্টরি আপডেট (সিমুলেট)
        print("📊 ইনভেন্টরি আপডেট করা হচ্ছে...")
        time.sleep(1)
        
        print("✅ অর্ডার সফলভাবে প্রসেস হয়েছে!\n")
        
        # ACK পাঠান
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
    except Exception as e:
        print(f"❌ এরর: {e}")
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=False)

# কলবাক ফাংশন: ইমেইল পাঠানো
def send_email(ch, method, properties, body):
    try:
        email_data = json.loads(body)
        
        print("📧 ইমেইল পাঠানো হচ্ছে...")
        print(f"   প্রাপক: {email_data['to']}")
        print(f"   বিষয়: {email_data['subject']}")
        print(f"   বডি: {email_data['body']}")
        
        # ইমেইল পাঠানো সিমুলেট
        time.sleep(1)
        
        print("✅ ইমেইল সফলভাবে পাঠানো হয়েছে!\n")
        
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
    except Exception as e:
        print(f"❌ ইমেইল এরর: {e}")
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=True)

# কলবাক ফাংশন: SMS পাঠানো
def send_sms(ch, method, properties, body):
    try:
        sms_data = json.loads(body)
        
        print("📱 SMS পাঠানো হচ্ছে...")
        print(f"   প্রাপক: {sms_data['to']}")
        print(f"   মেসেজ: {sms_data['message']}")
        
        # SMS পাঠানো সিমুলেট
        time.sleep(1)
        
        print("✅ SMS সফলভাবে পাঠানো হয়েছে!\n")
        
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
    except Exception as e:
        print(f"❌ SMS এরর: {e}")
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=True)

# Consumer চালু করুন
if __name__ == '__main__':
    import sys
    
    if len(sys.argv) < 2:
        print("ব্যবহার: python consumer.py [order|email|sms]")
        sys.exit(1)
    
    consumer_type = sys.argv[1]
    
    if consumer_type == 'order':
        consumer = OrderConsumer(ORDER_QUEUE, process_order)
        print("🛒 অর্ডার প্রসেসর চালু হচ্ছে...")
    elif consumer_type == 'email':
        consumer = OrderConsumer(EMAIL_QUEUE, send_email)
        print("📧 ইমেইল সার্ভিস চালু হচ্ছে...")
    elif consumer_type == 'sms':
        consumer = OrderConsumer(SMS_QUEUE, send_sms)
        print("📱 SMS সার্ভিস চালু হচ্ছে...")
    else:
        print("❌ অবৈধ consumer type!")
        sys.exit(1)
    
    consumer.start()
```

---

## 🚀 কিভাবে চালাবেন (Step by Step)

### **ধাপ ১: RabbitMQ সেটআপ করুন**
```bash
# Terminal 1
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```

### **ধাপ ২: প্রজেক্ট সেটআপ করুন**
```bash
# Terminal 2
python setup.py
```

**আউটপুট:**
```
🔧 RabbitMQ সেটআপ শুরু হচ্ছে...
✅ Exchange তৈরি: ecommerce_exchange
✅ Queue তৈরি: order_processing
✅ Queue তৈরি: email_notification
✅ Queue তৈরি: sms_notification
✅ Binding: order_processing <-> order.new
✅ Binding: email_notification <-> notification.email
✅ Binding: sms_notification <-> notification.sms

🎉 সেটআপ সম্পন্ন!
```

### **ধাপ ৩: Consumers চালু করুন (৩টি আলাদা Terminal)**

```bash
# Terminal 3 - Order Processor
python consumer.py order
```

```bash
# Terminal 4 - Email Service
python consumer.py email
```

```bash
# Terminal 5 - SMS Service
python consumer.py sms
```

### **ধাপ ৪: অর্ডার পাঠান**
```bash
# Terminal 6
python producer.py
```

---

## 📊 ভিজুয়াল ফ্লো চার্ট

```
┌─────────────────────────────────────────────────────────────────┐
│                      RabbitMQ সিস্টেম                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐                                               │
│  │  producer.py │                                               │
│  │  (অর্ডার)    │                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│         │ ① মেসেজ পাঠায়                                         │
│         ▼                                                        │
│  ┌─────────────────────────┐                                    │
│  │  ecommerce_exchange     │                                    │
│  │  (Direct Exchange)      │                                    │
│  └─────────┬───────────────┘                                    │
│            │                                                     │
│            │ ② Routing করে                                      │
│            │                                                     │
│  ┌─────────┼──────────┬──────────────┐                         │
│  │         │          │              │                          │
│  ▼         ▼          ▼              ▼                          │
│  order.new │   notification.email   │ notification.sms         │
│            │                         │                          │
│  ┌─────────▼────┐  ┌────────────────▼─┐  ┌──────────────────┐ │
│  │ order_queue  │  │  email_queue     │  │  sms_queue       │ │
│  │              │  │                  │  │                  │ │
│  │ ┌──┐ ┌──┐   │  │ ┌──┐             │  │ ┌──┐            │ │
│  │ │M1│ │M2│   │  │ │M1│             │  │ │M1│            │ │
│  │ └──┘ └──┘   │  │ └──┘             │  │ └──┘            │ │
│  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘ │
│         │                   │                      │            │
└─────────┼───────────────────┼──────────────────────┼────────────┘
          │                   │                      │
          │ ③ Consumer নেয়    │                      │
          ▼                   ▼                      ▼
   ┌──────────────┐    ┌──────────────┐     ┌──────────────┐
   │ consumer.py  │    │ consumer.py  │     │ consumer.py  │
   │   (order)    │    │   (email)    │     │    (sms)     │
   │              │    │              │     │              │
   │ - প্রসেস     │    │ - ইমেইল     │     │ - SMS        │
   │ - সেভ        │    │   পাঠায়     │     │   পাঠায়     │
   │ - আপডেট     │    │              │     │              │
   └──────────────┘    └──────────────┘     └──────────────┘
```

---

## 🎨 বিভিন্ন Exchange Type এর তুলনা

### **1. Direct Exchange**
```
Producer ──["order.new"]──> Exchange ──["order.new"]──> Queue A ✓
                                    ──["order.old"]──> Queue B ✗
```
**ব্যবহার**: যখন নির্দিষ্ট routing key দরকার

---

### **2. Fanout Exchange**
```
                          ┌──> Queue A (অর্ডার)
Producer ──> Exchange ────┼──> Queue B (ইমেইল)
                          └──> Queue C (SMS)
```
**ব্যবহার**: সব Queue তে একই মেসেজ পাঠাতে হলে

**কোড উদাহরণ:**
```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Fanout Exchange
channel.exchange_declare(exchange='broadcast', exchange_type='fanout')

# মেসেজ পাঠান (সব Queue তে যাবে)
channel.basic_publish(
    exchange='broadcast',
    routing_key='',  # Fanout এ routing key দরকার নেই
    body='সিস্টেম মেইন্টেনেন্স নোটিস: আজ রাত ১০টায়'
)

print("✅ সব Queue তে মেসেজ পাঠানো হয়েছে")
connection.close()
```

---

### **3. Topic Exchange**
```
Routing Key Pattern:
- "order.*"        → order.new, order.update
- "order.*.urgent" → order.new.urgent, order.cancel.urgent
- "#.urgent"       → যেকোনো urgent মেসেজ

Producer:
  routing_key = "order.new.urgent"
              │
              ▼
Exchange (Topic)
              │
              ├──> Queue A [pattern: "order.#"] ✓
              ├──> Queue B [pattern: "*.urgent"] ✓
              └──> Queue C [pattern: "email.*"] ✗
```

**কোড উদাহরণ:**
```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Topic Exchange তৈরি
channel.exchange_declare(exchange='logs', exchange_type='topic')

# বিভিন্ন Queue তৈরি
channel.queue_declare(queue='error_logs')
channel.queue_declare(queue='all_logs')

# Binding with patterns
channel.queue_bind(exchange='logs', queue='error_logs', routing_key='*.error')
channel.queue_bind(exchange='logs', queue='all_logs', routing_key='#')

# মেসেজ পাঠান
logs = [
    ('system.error', 'সিস্টেম এরর ঘটেছে'),
    ('database.error', 'ডাটাবেস সংযোগ ব্যর্থ'),
    ('system.info', 'সিস্টেম চালু হয়েছে')
]

for routing_key, message in logs:
    channel.basic_publish(
        exchange='logs',
        routing_key=routing_key,
        body=message
    )
    print(f"✅ পাঠানো: [{routing_key}] {message}")

connection.close()
```

---

### **4. Headers Exchange**
```
Producer:
  headers = {'type': 'order', 'priority': 'high'}
           │
           ▼
Exchange (Headers)
           │
           ├──> Queue A [headers: type=order] ✓
           ├──> Queue B [headers: priority=high] ✓
           └──> Queue C [headers: type=payment] ✗
```

**কোড উদাহরণ:**
```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Headers Exchange
channel.exchange_declare(exchange='headers_ex', exchange_type='headers')

# Queue তৈরি
channel.queue_declare(queue='urgent_queue')

# Headers দিয়ে Binding
channel.queue_bind(
    exchange='headers_ex',
    queue='urgent_queue',
    arguments={
        'x-match': 'all',  # সব headers মিলতে হবে
        'priority': 'high',
        'type': 'order'
    }
)

# মেসেজ পাঠান
channel.basic_publish(
    exchange='headers_ex',
    routing_key='',  # Headers Exchange এ routing key দরকার নেই
    body='জরুরি অর্ডার!',
    properties=pika.BasicProperties(
        headers={'priority': 'high', 'type': 'order'}
    )
)

print("✅ Headers সহ মেসেজ পাঠানো হয়েছে")
connection.close()
```

---

## 🔍 গুরুত্বপূর্ণ বিষয় (ACK, NACK, Retry)

### **Message Acknowledgment (ACK)**

```python
def callback(ch, method, properties, body):
    try:
        # কাজ করুন
        process_message(body)
        
        # সফল হলে ACK পাঠান
        ch.basic_ack(delivery_tag=method.delivery_tag)
        print("✅ ACK পাঠানো হয়েছে")
        
    except Exception as e:
        # ব্যর্থ হলে NACK পাঠান
        ch.basic_nack(
            delivery_tag=method.delivery_tag,
            requeue=True  # আবার Queue তে ফেরত
        )
        print(f"❌ NACK পাঠানো হয়েছে: {e}")
```

---

### **Retry Logic (পুনরায় চেষ্টা)**

```python
def callback_with_retry(ch, method, properties, body):
    import json
    
    message = json.loads(body)
    retry_count = message.get('retry_count', 0)
    max_retries = 3
    
    try:
        # প্রসেস করুন
        process_order(message)
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
    except Exception as e:
        if retry_count < max_retries:
            # আবার চেষ্টা করুন
            message['retry_count'] = retry_count + 1
            
            ch.basic_publish(
                exchange='',
                routing_key='order_queue',
                body=json.dumps(message)
            )
            
            print(f"🔄 Retry {retry_count + 1}/{max_retries}")
            ch.basic_ack(delivery_tag=method.delivery_tag)
        else:
            # Dead Letter Queue তে পাঠান
            print("❌ Max retries reached. Sending to DLX")
            ch.basic_nack(delivery_tag=method.delivery_tag, requeue=False)
```

---

## 🛡️ Error Handling এবং Dead Letter Exchange

### **Dead Letter Exchange সেটআপ:**

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Dead Letter Exchange তৈরি
channel.exchange_declare(exchange='dlx', exchange_type='fanout', durable=True)

# Dead Letter Queue তৈরি
channel.queue_declare(queue='failed_orders', durable=True)
channel.queue_bind(exchange='dlx', queue='failed_orders')

# Main Queue with DLX
channel.queue_declare(
    queue='main_orders',
    durable=True,
    arguments={
        'x-dead-letter-exchange': 'dlx',  # ব্যর্থ মেসেজ এখানে যাবে
        'x-message-ttl': 300000,          # 5 মিনিট TTL
        'x-max-length': 1000               # সর্বোচ্চ দৈর্ঘ্য
    }
)

print("✅ DLX সেটআপ সম্পন্ন")
connection.close()
```

**ভিজুয়াল:**
```
main_orders Queue ──[ব্যর্থ/TTL শেষ]──> DLX ──> failed_orders Queue
                                                       │
                                                       ▼
                                              Manual Review/Retry
```

---

## 📈 Priority Queue (অগ্রাধিকার কিউ)

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Priority Queue তৈরি
channel.queue_declare(
    queue='priority_orders',
    durable=True,
    arguments={'x-max-priority': 10}  # 0-10 পর্যন্ত priority
)

# High Priority মেসেজ
channel.basic_publish(
    exchange='',
    routing_key='priority_orders',
    body='VIP Customer অর্ডার',
    properties=pika.BasicProperties(
        priority=10,  # সর্বোচ্চ priority
        delivery_mode=2
    )
)

# Normal Priority মেসেজ
channel.basic_publish(
    exchange='',
    routing_key='priority_orders',
    body='সাধারণ অর্ডার',
    properties=pika.BasicProperties(
        priority=5,  # মাঝারি priority
        delivery_mode=2
    )
)

# Low Priority মেসেজ
channel.basic_publish(
    exchange='',
    routing_key='priority_orders',
    body='নন-আর্জেন্ট অর্ডার',
    properties=pika.BasicProperties(
        priority=1,  # কম priority
        delivery_mode=2
    )
)

print("✅ বিভিন্ন priority এর মেসেজ পাঠানো হয়েছে")
connection.close()
```

**প্রসেসিং অর্ডার:**
```
Queue: [P:10] → [P:5] → [P:1]
       VIP     Normal   Low
       প্রথম     দ্বিতীয়  শেষ
```

---

## 🎯 সারসংক্ষেপ

### **RabbitMQ এর মূল কম্পোনেন্ট:**
1. **Producer** - মেসেজ পাঠায়
2. **Exchange** - রাউটিং করে
3. **Queue** - মেসেজ সংরক্ষণ করে
4. **Consumer** - মেসেজ প্রসেস করে
5. **Binding** - Exchange ও Queue সংযুক্ত করে

### **Exchange Types:**
- **Direct** - নির্দিষ্ট routing key
- **Fanout** - সব Queue তে
- **Topic** - Pattern matching
- **Headers** - Metadata ভিত্তিক

### **গুরুত্বপূর্ণ Features:**
- ✅ **Durability** - Restart এ টিকে থাকে
- ✅ **Priority** - গুরুত্ব অনুযায়ী প্রসেস
- ✅ **TTL** - স্বয়ংক্রিয় মেয়াদ শেষ
- ✅ **DLX** - ব্যর্থ মেসেজ পরিচালনা
- ✅ **ACK/NACK** - নিশ্চিতকরণ

---

